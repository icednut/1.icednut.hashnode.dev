---
title: "Kotlin, Spring Framework에서 AOP를 처리하는 색다른 방법"
datePublished: Wed Nov 01 2023 15:00:00 GMT+0000 (Coordinated Universal Time)
cuid: clome37nm000b08jufnyq18w0
slug: kotlin-spring-framework-aop
cover: https://cdn.hashnode.com/res/hashnode/image/stock/unsplash/azJJSiwWW90/upload/f6f57569156102b6d75fc9a0b86a1506.jpeg
tags: kotlin, spring-aop, context-receivers

---

# 소개

얼마 전에 코프링(Kotlin + Spring Framework) 조합에서 `@Transaction` 과 `@Cacheable` 어노테이션을 이용한 AOP 처리를 Kotlin Tailing Lambdas를 이용하여 처리하는 방법을 우연히 보게 되었다. 글쓴이는 이걸 Kotlin AOP 라고 부르는데 간략하게 요약해 보자면 다음과 같다.

원본글: [https://tech.kakaopay.com/post/overcome-spring-aop-with-kotlin/#transactional-극복해보기](https://tech.kakaopay.com/post/overcome-spring-aop-with-kotlin/#transactional-%EA%B7%B9%EB%B3%B5%ED%95%B4%EB%B3%B4%EA%B8%B0)

## Spring AOP를 이용한 방법

### DB Transaction 처리

```kotlin
@Service
class UserSyncService(
   private val userStoreRestClient : UserStoreRestClient
){

    fun syncUsers(){
        val users = userStoreRestClient.requestGetAllUsers() // Long Time IO..
        this.upsertBulkUsers(users)
    }

    @Transactional
    private fun upsertBulkUsers(user: List<User>){
        // .. TX update users..
        // .. TX insert users..
        // 그러나 private method는 트랜잭션 처리가 되질 않음
    }
}
```

### Caching

```kotlin
@Service
class CalculateService {

  @Cacheable(cacheNames = ["plus"], key = "#x+ ' ' +#y")
  fun plus(x: Int, y: Int): Int {
    return x + y
  }
}
```

### LoggingStopWatch (걸린 시간 측정하기)

```kotlin
@Target(AnnotationTarget.FUNCTION)
@Retention(AnnotationRetention.RUNTIME)
annotation class LoggingStopWatch
```

```kotlin
@Aspect
@Component
class LoggingStopWatchAdvice {

   companion object {
       val logger: Logger = LoggerFactory.getLogger(this::class.java)
   }

   @Around("@annotation(com.example.springaop.aspect.LoggingStopWatch)")
   fun atTarget(joinPoint: ProceedingJoinPoint, name: String): Any? {
       val startAt = LocalDateTime.now()
       logger.info("Start At : $startAt")

       val proceed = joinPoint.proceed()

       val endAt = LocalDateTime.now()

       logger.info("End At : $startAt")
       logger.info("Logic Duration : ${Duration.between(startAt, endAt).toMillis()}ms")

       return proceed
   }
}
```

```kotlin
@Service
class UserService(
   val userRepository: UserRepository
){

   @LoggingStopWatch
   fun signUp(user: User){
       this.saveUserData(user)
   }

   @LoggingStopWatch
   private fun saveUserData(user: User){
       userRepository.save(user)
   }
}
```

## Kotlin AOP를 이용한 방법

### DB Transaction 처리

```kotlin
@Component
class Tx(
   _txAdvice: TxAdvice,
) {

   init {
       txAdvice = _txAdvice
   }

   companion object {
       private lateinit var txAdvice: TxAdvice

       fun <T> run(function: () -> T): T {
           return txAdvice.run(function)
       }
   }

   @Component
   class TxAdvice {

       @Transactional
       fun <T> run(function: () -> T): T {
           return function.run()
       }
   }
}
```

```kotlin
@Service
class UserService(
   val userRepository: UserRepository
) {

   fun signUp(userInsert: UserInsert) {
       Tx.run { this.saveUserData(userInsert.toEntity()) }
       Tx.run { this.saveUserData(userInsert.toEntity()) }
   }

   private fun saveUserData(user: User) {
       userRepository.save(user)
   }
}
```

### Caching

```kotlin
@Component
class CacheUser(
   _advice: CacheUserAdvice,
) {

   init {
       advice = _advice
   }

   companion object {
       private lateinit var advice: CacheUserAdvice
       private const val TOKEN = "::"

       fun <T> cache(vararg keys: Any, function: () -> T): T {
           return advice.cache(generateKey(keys), function)
       }

       fun <T> evict(vararg keys: Any, function: () -> T): T {
           return advice.evict(generateKey(keys), function)
       }

       private fun generateKey(keys: Array<out Any>) = keys.joinToString(TOKEN)
   }

   @Component
   class CacheUserAdvice {
       companion object {
           private const val CACHE_NAME = "User"
       }

       @Cacheable(value = [CACHE_NAME], key = "#key")
       fun <T> cache(key: String, function: () -> T): T {
           return function.invoke()
       }

       @CacheEvcit(value = [CACHE_NAME], key = "#key")
       fun <T> evict(key: String, function: () -> T): T {
           return function.invoke()
       }
   }
}
```

```kotlin
@Service
class UserService(
   val userRepository: UserRepository
) {

   fun findById(userId: Long): UserRead = CacheUser.cache("UserRead", "userId:${userId}") { // 캐시 AOP 적용
       val user = userRepository.findById(userId).orElseThrow { throw Exception("User Not Found :${userId}") }
       return@cache UserRead(user)
   }

   fun updateUser(userId: Long, userUpdate: UserUpdate) = CacheUser.evict("UserRead", "userId:${userId}") { // 캐시 삭제 AOP 적용
       // Update User ..
   }
}
```

### LoggingStopWatch

```kotlin
fun <T> loggingStopWatch(function: () -> T): T {
   val startAt = LocalDateTime.now()
   logger.info("Start At : $startAt")

   val result = function.invoke()

   val endAt = LocalDateTime.now()

   logger.info("End At : $endAt")
   logger.info("Logic Duration : ${Duration.between(startAt, endAt).toMillis()}ms")

   return result
}
```

```kotlin
@Service
class UserService(
   val userRepository: UserRepository
) {

   fun signUp(name: String, user: User){
       this.saveUserData(user, )
   }

   private fun saveUserData(user: User) = loggingStopWatch {
       // ...
   }
}
```

# 보완할 점은 없을까?

Kotlin AOP 방법에서 실제 Transaction과 Cache 처리를 담당하는 Bean을 companion object에 담아서 사용하는 것을 볼 수 있었다.

```kotlin
@Component
class CacheUser(
   _advice: CacheUserAdvice,
) {

   init {
       advice = _advice
   }

   companion object {
       private lateinit var advice: CacheUserAdvice
       ...
   }
   ...
}
```

```kotlin
@Component
class Tx(
   _txAdvice: TxAdvice,
) {

   init {
       txAdvice = _txAdvice
   }

   companion object {
       private lateinit var txAdvice: TxAdvice
       ...
   }
   ...
}
```

처음 이걸 봤을 때 companion object에 변경 가능한 상태(`lateinit var`)가 있다는 게 왠지 살짝 꺼림칙했다. 보통 companion object에는 상태를 두지 않기에 이에 대한 잠재적인 위험이 있지 않을까라는 우려에 코틀린 공식 문서나 웹 검색을 해봤지만 딱히 문제를 제기하는 글을 찾진 못했다.

하지만 함수도 아니고 companion object에 변경 가능한 상태가 있다는 것은 해당 클래스 내부에서 상태를 변경할 수 있다는 여지가 있다는 것을 의미한다. 물론 위 상황에서 companion object의 companion class는 Bean으로 인스턴스화 될 거라 init 블록은 한 번만 호출되니 별다른 문제는 없겠지만, 유지보수 과정에서 누군가가 실수로 `txAdvice`를 교체하는 로직을 작성하지 않을까 라는 쓸데없는(?) 걱정이 앞섰다.

```kotlin
@Component
class Tx(
   _txAdvice: TxAdvice @Qualifier("txAdvice"),
   _txAdvice2: TxAdvice @Qualifier("externalTxAdvice")
) {

   init {
       txAdvice = _txAdvice
       txAdvice2 = _txAdvice2
   }

   // 과연 누가 이렇게 작성할까?
   // 예시를 위해 좀 억지를 부린 것 같다.
   fun somethingWrong() {
       txAdvice = _txAdvice2
   }

   companion object {
       private lateinit var txAdvice: TxAdvice

       fun <T> run(function: () -> T): T {
           return txAdvice.run(function)
       }

       fun <T> runWithUserDb(function: () -> T): T {
           return txAdvice.run(function)
       }
   }

   @Component
   class TxAdvice {

       @Transactional
       fun <T> run(function: () -> T): T {
           return function.run()
       }
   }
}
```

또한 트랜잭션 처리가 필요한 곳에 `@Transactional` 이나 `Tx.run { ... }` 을 쓰는 것을 컴파일 타임에 강제할 수 있는 방법이 없는 것도 약간 아쉬웠다. 만약 누군가 실수로 트랜잭션 처리를 누락한다면? 물론 동료들과 코드 리뷰 시 이를 인지하여 어느 정도 바로 잡을 수는 있겠지만 여전히(?) 코딩과 리뷰는 사람이 하는 일이기에 실수의 여지가 있다.

```kotlin
@Service
class UserService(
   val userRepository: UserRepository
   val logRepository: LogRepository
) {

   // 실수로 트랜잭션 처리를 빼먹음
   fun deleteUser(userId: Long){
       val user = userRepository.findById(userId)
       user.setActivate(false)

       val log = DeleteLog(userId = userId, deletedAt = LocalDateTime.now())
       logRepository.save(log)
   }
}
```

# 그럼 어떻게 보완할 것인가?

## 내가 우려 했던 것

물론 나도 안다. 내가 억지를 부리고 있다는걸. 누가 저렇게 할지 싶다. 하지만 내가 말하고 싶은 것은 이러한 틈이 개발자들에게 실수할 여지를 줄 수도 있다는 것을 말하고 싶었다.

이러한 틈을 메꾸기 위해 그라운드 룰을 정하여 문서화한다거나 주석을 통해 이를 방지할 수도 있다. 하지만 나는 프로그래밍 언어 차원에서 이러한 틈을 비집고 들어오는 것을 못하게 하는 장치가 있지 않을까 생각해 봤다. 즉 앞서 우려 했던 것과 관련한 코딩할 경우 컴파일 타임에 오류를 발생한다거나 못하도록 강제하는 안전장치를 코틀린 언어 차원에서 지원하는 방법을 찾고 싶었다. 개인적으로 앞으로 프로그래밍 언어의 발전은 이러한 틈에 대한 경각심을 일깨워 주거나 안전장치를 강제하는 방법을 좀 더 손쉽게 할 수 있는 방향으로 발전하지 않을까 싶다. (아니면 코파일럿이 도와주려나?)

요약: 코틀린 언어 차원에서 아래 행위를 할 경우 컴파일 오류를 발생하거나 못하게 막기

* companion object의 `lateinit var` 필드에 값 변경하기
    
* 트랜잭션 처리 누락
    

코틀린으로 이러한 틈을 메꾸기 위한 완벽한 방법을 찾을 수는 없었지만 그나마 비슷한 방법을 찾아서 정리하였다. 물론 100% 해결도 아니고 여전히 논란의 여지는 있지만 그래도 의도 전달에 의의를 두고 싶다.

## 보완해보자 1: lateinit var 대신 Lazy Properties 사용하기

코틀린의 Standard Delegates 중에서 Lazy Properties 라는 방법이 있다.

출처: [https://kotlinlang.org/docs/delegated-properties.html#lazy-properties](https://tech.kakaopay.com/post/overcome-spring-aop-with-kotlin/#transactional-%EA%B7%B9%EB%B3%B5%ED%95%B4%EB%B3%B4%EA%B8%B0)

```kotlin
val lazyValue: String by lazy {
    println("computed!")
    "Hello"
}

fun main() {
    println(lazyValue)
    println(lazyValue)
}
```

```kotlin
-- 실행 결과 --
computed!
Hello
Hello
```

위의 예시를 살펴보면 `lazyValue`에 대해 참조가 일어날 때 delegate 함수인 lazy의 람다 부분이 한 번만 실행되고 이후 `lazyValue` 참조부터는 해당 람다가 또 실행되질 않는다. 이걸 이용하면 companion object의 상태 필드를 바꾸는 여지를 없앨 수 있다. 아쉽게도 변경 못하게 컴파일 타임에 강제하진 못하지만 동작을 막는 것만으로도 의미가 있다고 본다.

```kotlin
@Component
class Tx(
   _txAdvice: TxAdvice @Qualifier("txAdvice"),
   _txAdvice2: TxAdvice @Qualifier("externalTxAdvice")
) {

   init {
       _txAdviceState = _txAdvice
       _txAdviceState2 = _txAdvice2
   }

   // 이젠 이런 메소드를 작성해도 lateinit var 참조만 변할 뿐 Lazy Properties는 변하지 않는다.
   fun somethingWrong() {
       _txAdviceState = _txAdvice2
   }

   companion object {
       private lateinit var _txAdviceState: TxAdvice
       private val txAdvice: TxAdvice by lazy {
           _txAdviceState
       }

       private lateinit var _txAdviceState2: TxAdvice
       private val txAdvice2: TxAdvice by lazy {
           _txAdviceState2
       }

       fun <T> run(function: () -> T): T {
           return txAdvice.run(function)
       }

       fun <T> runWithUserDb(function: () -> T): T {
           return txAdvice2.run(function)
       }
   }

   @Component
   class TxAdvice {

       @Transactional
       fun <T> run(function: () -> T): T {
           return function.run()
       }
   }
}
```

하지만 여전히 Tx bean 안에서 companion object의 `txAdvice`, `txAdvice2`가 아닌 `_txAdviceState`, `_txAdviceState2`를 사용할 수 있다는 논란의 여지는 남아있다. `lateinit val`의 변수명을 통해 이를 쓰지 못하게 알린다든가 해야할텐데 개인적으로 `lateinit val` 같은 문법이 있으면 좀 더 깔끔한 코드로 컴파일 타임에 이를 방지할 수 있을 거라 생각한다. 하지만 없는 거 보면 뭔가 이유가 있거나 니즈가 커지면 언젠가 나오지 않을까 기대해 본다.

## 보완해보자 2: 타입클래스를 이용한 트랜잭션 처리 강제하기 (with Context Receivers)

아쉽게도 트랜잭션 처리해야 하는 곳에 트랜잭션 처리를 할 수 있도록 완벽하게 강제할 수는 없었다. 하지만 코틀린과 찰떡궁합인 인텔리제이의 힘을 입어 경고라도 띄워 경각심을 줄 수 있는 방법이 있다.

함수형 프로그래밍에서 단골로 나오는 기법 중 하나인 타입클래스를 코틀린에서도 할 수 있다. Context Receiver 라는 문법으로 인해 타입클래스 타이핑이 가능해진 건데 아직 experimental feature 라서 정식 문법은 아니기 때문에 다음과 같이 컴파일 옵션을 손봐야 한다. 즉 정식으로 쓰기엔 약간 우려가 있다는 것을 염두에 두자.

```kotlin
import org.jetbrains.kotlin.gradle.tasks.KotlinCompile

plugins {
	kotlin("jvm") version "1.8.22"
}

dependencies {
	...
}

tasks.withType<KotlinCompile> {
	kotlinOptions {
		freeCompilerArgs += "-Xcontext-receivers" // 이 옵션을 추가해야 됨
		jvmTarget = "17"
	}
}
```

Context Receiver 문법을 사용 예시는 다음과 같다. 여기서는 실제 로깅을 타입클래스에서 처리하도록 위임했다.

참고

* [https://kotlinlang.org/docs/whatsnew1620.html#prototype-of-context-receivers-for-kotlin-jvm](https://tech.kakaopay.com/post/overcome-spring-aop-with-kotlin/#transactional-%EA%B7%B9%EB%B3%B5%ED%95%B4%EB%B3%B4%EA%B8%B0)
    
* [https://blog.rockthejvm.com/kotlin-context-receivers](https://tech.kakaopay.com/post/overcome-spring-aop-with-kotlin/#transactional-%EA%B7%B9%EB%B3%B5%ED%95%B4%EB%B3%B4%EA%B8%B0)
    

```kotlin
context(LoggingContext)
fun startBusinessOperation() {
    info("Operation has started")
}
```

```kotlin
fun main() {
    with(consoleLoggingContext) {
        startBusinessOperation()
    }
}
```

```kotlin
interface LoggingContext {
    fun info(message: String)
}

val consoleLoggingContext = object : LoggingContext {
    override fun info(message: String) {
        println("[INFO] $message")
    }
}
```

타입 클래스가 뭔지 처음 보는 사람들을 위해 타입 클래스를 설명하고 싶지만 여기에 쓰기엔 너무 길어질 것 같아서 관련 글을 소개하는 것으로 대신한다. 스칼라로 타입클래스를 설명한 글이지만 개념을 익히기엔 좋을 것 같다.

* [https://blogrockthejvm.com/why-are-typeclasses-useful](https://tech.kakaopay.com/post/overcome-spring-aop-with-kotlin/#transactional-%EA%B7%B9%EB%B3%B5%ED%95%B4%EB%B3%B4%EA%B8%B0)
    
* [https://velog.io/@icednut/타입클래스를-이용한-애드혹-다형성-구현하기-Ad-hoc-Polymorphism-in-Scala](https://tech.kakaopay.com/post/overcome-spring-aop-with-kotlin/#transactional-%EA%B7%B9%EB%B3%B5%ED%95%B4%EB%B3%B4%EA%B8%B0)
    

그럼 이제 타입 클래스를 이용한 트랜잭션 처리를 살펴보자.

```kotlin
import io.icednut.spring.exercise.config.TxScope
import io.icednut.spring.exercise.repository.User
import io.icednut.spring.exercise.repository.UserRepository
import org.springframework.stereotype.Service

@Service
class UserService(
    val userRepository: UserRepository
) {

    context(TxScope)
    fun selectUser(userId: Long) = readable {
        userRepository.findById(userId)
    }

    context(TxScope)
    fun createUser(user: User): User =
        createUserAndLogging(user)

    context(TxScope)
    private fun createUserAndLogging(user: User): User = writable {
        val createdUser = userRepository.save(user)

        println("created user: $createdUser")
        createdUser
    }
}
```

뭔가 `@Transactional`을 쓰는 것과 비슷해 보이지만 실제 트랜잭션 처리는 TxScope 타입클래스가 처리하는 것을 볼 수 있다. 어노테이션 방식과 다른 점은 TxScope를 받기만 한다고 해서 트랜잭션 처리가 되지 않고 타입클래스의 멤버 메소드(여기서는 readable, writable)를 호출해야지 실제 트랜잭션 처리를 수행하게 된다.

또한 Context Receiver가 필요한 메소드(여기서는 `selectUser`, `createUser`, `createUserAndLogging`)를 호출하는 쪽에서는 타입클래스의 구현체를 넘겨줘야 하기 때문에 해당 메소드를 호출하는 쪽에서도 이 메소드는 트랜잭션 처리를 해야 한다는 것을 명시적으로 알릴 수 있게 된다.

```kotlin
import io.icednut.spring.exercise.config.TxConfig
import io.icednut.spring.exercise.repository.User
import io.icednut.spring.exercise.service.UserService
import org.springframework.web.bind.annotation.GetMapping
import org.springframework.web.bind.annotation.PostMapping
import org.springframework.web.bind.annotation.RequestMapping
import org.springframework.web.bind.annotation.RestController
import java.util.*

@RestController
@RequestMapping("/user")
class UserController(
    val userService: UserService
) {

    @GetMapping
    fun retrieveUser(userId: Long): Optional<User> {
        // 이런 식으로 트랜잭션 처리가 필요한 메소드를 호출할 때는 타입클래스 구현체(receiver)를 넘겨야 한다.
        return with(TxConfig.txScope) {
            userService.selectUser(userId)
        }

        // 만약 아래와 같이 receiver 넘기지 않고 그냥 호출할 경우 컴파일 오류를 발생하게 된다.
        // return userService.selectUser(userId) <---- compile error
    }

    @PostMapping
    fun createUser(user: User): User {
        return with(TxConfig.txScope) {
            userService.createUser(user)
        }
    }
}
```

```kotlin
import org.springframework.beans.factory.annotation.Autowired
import org.springframework.context.annotation.Configuration

@Configuration
class TxConfig {

    companion object {
        private lateinit var _txScope: TxScope
        val txScope: TxScope by lazy {
            _txScope
        }
    }

    @Autowired
    fun setTxScope(txScope: TxScope) {
        _txScope = txScope
    }
}
```

```kotlin
@Component
class ProductionTxScope : TxScope {

    @Transactional
    override fun <T> writable(block: () -> T): T {
        return block()
    }

    @Transactional(readOnly = true)
    override fun <T> readable(block: () -> T): T {
        return block()
    }
}
```

이렇게 하면 테스트코드에서 사용할 수 있는 가짜 트랜잭션 처리를 할 수 있는 타입클래스 구현체도 직접 만들어 세팅할 수 있게 된다.

```kotlin
abstract class TxConfigTestHelper {

    init {
        val txConfig = TxConfig()
        txConfig.setTxScope(object : TxScope {
            override fun <T> writable(block: () -> T): T {
                return block()
            }

            override fun <T> readable(block: () -> T): T {
                return block()
            }
        })
    }
}
```

그럼 이쯤에서 아까 우려했던 트랜잭션 처리 누락은 어떻게 처리가 될까? 아까와 비슷한 상황을 만들면서 살펴보자.

```kotlin
@Service
class UserService(
   val userRepository: UserRepository
   val logRepository: LogRepository
) {

   // 실수로 트랜잭션 처리를 빼먹으면?
   fun deleteUser(userId: Long){
       val user = userRepository.findById(userId)
       user.setActivate(false)

       val log = DeleteLog(userId = userId, deletedAt = LocalDateTime.now())
       logRepository.save(log)
   }
}
```

```kotlin
@RestController
@RequestMapping("/user")
class UserController(
    val userService: UserService
) {

    @DeleteMapping
    fun deleteUser(user: User): User {
        // 다행히 여기서는 트랜잭션 처리를 인지하여 receiver를 넘기는 코드를 작성했다.
		    // 어떻게 될까?
        return with(TxConfig.txScope) {
            userService.deleteUser(user)
        }
    }
}
```

이런 경우 컴파일 오류는 발생하지 않는다. 다만 receiver를 넘겼지만 받는 쪽(`userService.deleteUser` 메소드)에서 context receiver 문법을 쓰지 않았기 때문에 UserController 클래스에서 receiver를 사용하고 있지 않다는 경고(warn)가 발생하게 된다.

하지만 논란의 여지는 여전히 남아있다. IDE에서 경고를 내주는 것만으로 경각심을 주기엔 부족하진 않을까? 또한 트랜잭션 처리에 아예 무지하여 receiver를 넘기는 코드조차 작성하지 않았다면 트랜잭션 처리 강제하는 장치가 아예 없어지게 된다. 이것에 대한 해결책은 없을까?

안타깝게 위에서 말한 논란을 종식하기 위한 방법을 좀 더 찾아봐야겠지만 기존에 있는 메소드를 리팩토링 하면서 트랜잭션 처리를 누락하면 경고가 발생시킬 수 있기 때문에 반쪽짜리 목표 달성이라고 볼 수 있을 거 같다.

```kotlin
import io.icednut.spring.exercise.config.TxScope
import io.icednut.spring.exercise.repository.User
import io.icednut.spring.exercise.repository.UserRepository
import org.springframework.stereotype.Service

@Service
class UserService(
    val userRepository: UserRepository
) {

	  // 기존에 있던 코드를 리팩토링 함
	  context(TxScope)
    fun selectUser(userId: Long): User {
        return selectAndLogSave(userId)
		}

	  // 리팩토링 하면서 실수로 트랜잭션 처리를 누락함
	  private fun selectAndLogSave(userId: Long): User = {
				val user = userRepository.findById(userId)

        logRepository.save(SelectUserLog(userId = userId, selectedAt = LocalDateTime.now())
				return user
    }

    ...
}
```

```kotlin
import io.icednut.spring.exercise.config.TxConfig
import io.icednut.spring.exercise.repository.User
import io.icednut.spring.exercise.service.UserService
import org.springframework.web.bind.annotation.GetMapping
import org.springframework.web.bind.annotation.PostMapping
import org.springframework.web.bind.annotation.RequestMapping
import org.springframework.web.bind.annotation.RestController
import java.util.*

@RestController
@RequestMapping("/user")
class UserController(
    val userService: UserService
) {

    @GetMapping
    fun retrieveUser(userId: Long): Optional<User> {
        // 이제 여기선 receiver를 사용하지 않고 있다는 경고가 발생하게 된다.
        return with(TxConfig.txScope) {
            userService.selectUser(userId)
        }
    }

    ...
}
```

# 마치며

위에 언급한 결과물과 관련 테스트코드는 Github([https://github.com/icednut/kotlin-springtx-exercise/tree/step-02)에](https://github.com/icednut/kotlin-springtx-exercise/tree/step-02)에) 정리해 두었다. 실제로 로컬환경에서 실행 가능한 상태로 정리해두었다.

돌이켜 생각해 보면 타입 클래스 구현체를 companion object에 담아서 전역 변수처럼 사용한 것이 과연 이게 타입 클래스의 원래 의도와 맞는 방법일까 라는 의문이 든다. 변명이지만 트랜잭션 타입클래스가 스프링에 의존할 수 밖에 없었고 테스트코드 작성 때문에 트랜잭션 관련 빈을 전역 변수처럼 사용하게 되었는데 이 부분도 아쉬웠다. 만약 트랜잭션 처리를 타입클래스로 구현해야 한다면 Spring free한 트랜잭션 처리를 할 수 있는 방법을 찾아봐야겠다

다음 글에서는 타입 클래스와 context receiver를 어떨 때 쓰는 게 좋을지 정리해 볼 예정이다.