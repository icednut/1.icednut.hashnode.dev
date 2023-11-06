---
title: "Scala로 하는 Side-effect 다루기 기초"
datePublished: Thu Sep 02 2021 15:00:00 GMT+0000 (Coordinated Universal Time)
cuid: clomzwxzg000409gscxihapwb
slug: scala-side-effect
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1699280726488/46126b06-ec07-4cb6-9533-c603bb526bee.png
tags: scala, side-effect, lazy-evaluation

---

# Side Effect? Pure Function?

함수형 프로그래밍을 하다보면 Pure Function과 Composition 이라는 용어를 자주 마주치곤 한다. 도대체 이 용어들은 뭘 말하는걸까?

코딩을 하다보면 아래 행위들을 수행하는 코드를 종종 작성하게 되는데 이러한 행위들이 부수효과(Side-effect)라는 것은 이미 잘 알고 있다.

* 데이터베이스에 데이터를 READ, UPDATE, DELETE 한다.
    
* 다른 서비스에 RESTful API 호출을 한다.
    
* 현재 들고 있는 데이터를 HDFS에 쓴다.
    
* 디버깅을 위해 데이터를 파일이나 콘솔에 로깅한다.
    

부수효과 자체가 나쁘다는건 아니다. 근데 왜 사람들은 부수효과를 효율적으로 다뤄야 된다는 말을 하고 어떤 사람들은 부수효과를 없애야 한다고 하는걸까? 부수효과를 효율적으로 다뤄야되는게 뭘까? 도대체 부수효과를 어떻게 다뤄야 효율적인걸까?

여기 콘솔에 데이터를 출력하는 코드를 살펴보자. 앞서 말했듯이 이것은 부수효과다.

![](https://images.velog.io/images/icednut/post/6571f710-c073-48f5-946b-e929bb15d959/1-2.png align="left")

![](https://images.velog.io/images/icednut/post/946ee7be-01d7-4db2-9c48-a33927e76bf7/2-2.png align="left")

부수효과가 있는 함수(`printSystem`)를 그냥 실행하는 것에는 별문제가 없어 보인다.

그러나 부수효과가 프로그램 전체에 영향을 끼치지 않게 하기 위해 부수효과의 예외 상황을 컨트롤 해야 할 수도 있다. 즉 특정 함수 내부에 있는 부수효과의 실행을 외부에서 제어할 수 있게 하는 것인데 이런 경우 그 함수를 호출하여 부수효과를 구성하는 일을 진행해도 부수효과는 실행되지 않고 내가 원할 때 실행할 수 있게 된다.

결국 부수효과를 값으로 취급하는 함수로 탈바꿈 시키면 그 함수는 더 이상 부수효과가 있다고 하지 않게 된다. 예제에 대입하자면 `printSystem` 함수는 안에 부수효과(`println`)가 있는 함수이지만 `printSystem` 내부에 있는 부수효과를 값으로 취급 및 반환하여 부수효과 실행을 외부로 제어권을 넘기면 `printSystem` 함수는 더이상 부수효과가 있는 함수라고 하지 않는다. (이를 Effectful function 혹은 Pure Function이라고 부른다)

부수효과 실행을 지연 시켜서 부수효과를 프로그래머가 관리할 수 있도록 하는게 함수형 프로그래밍에서 말하는 effectful system 이다.

그럼 `printSystem` 함수 내부에 있는 부수 효과인 `println`를 값으로 취급하여 `pritnSystem` 함수에 부수효과를 없애보자. 여기서 손쉽게 쓸 수 있는게 `scala.concurrent.Future` 인데 Future로 부수효과를 감싸보자.

![](https://images.velog.io/images/icednut/post/fe83772a-becc-407a-924c-32eb316f7f0e/3.png align="left")

여기서 `Future(println("Launch missiles"))`라는 로직은 동일하니 하나로 생략해보자.

![](https://images.velog.io/images/icednut/post/6c97b0e5-0c4e-4aad-8a59-548c31a5a742/4.png align="left")

콘솔 출력 결과는 동일할 거라고 생각하지만 println은 한 번만 실행된다.

![](https://images.velog.io/images/icednut/post/db89f5c0-49ae-4fc4-9710-5d22d34aeca5/5.png align="left")

앞에서 Future를 새로 만들어서 합성하는 경우에는 기대한대로 결과가 나왔지만 코드를 줄이기 위해 Future를 재사용한 경우에는 기대한 것처럼 동작하지 않는다. 왜 그럴까?

일단 Future는 Eager Evaluation 이라서 `launch.flatMap` 함수 내부에 있는 launch는 Eager evaluation된 결과값으로 치환되어 버린다. 즉 Unit으로 치환되어 버리고 `println` 은 실행되지 않는 것이다. 결국 리팩토링으로 인해 부수효과가 기대한대로 동작을 하지 않아 부수효과를 제대로 다루지 못한 경우라고 볼 수 있다.

이러한 상황을 피하기 위해 부수효과를 Lazy Evaluation 할 수 있는 값으로 치환해야 위와 같이 리팩토링를 해도 우리가 원하는대로 동작하는 Pure Function을 만들 수 있게 되는 것이다.

# Lazy Evaluation되는 Pure Function을 만들어보자

Lazy Evaluation을 하기 위해 스칼라에는 call-by-name 이라는 특징이 있다. (함수값이나 thunk가 여기에 해당된다) Pure Function을 만들기 위해 부수효과를 값으로 취급할 때는 call-by-name을 이용하여 실행을 지연 시키면 된다.

이게 무슨 말인지 모르겠으니 코드로 살펴보자.

![](https://images.velog.io/images/icednut/post/7e41114c-0e44-40dc-8c57-005e5aa8ed86/6.png align="left")

여기서 LazyIO가 부수효과를 Lazy Evaluation 할 수 있는 자료형이라고 볼 수 있다. 바로 LazyIO case class 필드에 runEffect와 컴패니언 오브젝트의 io 메소드를 살펴보면 effect 라는 파라미터가 있는데 이 파라미터들은 함수이다.

결국 LazyIO 자료형은 부수효과를 실행하는 함수를 값(`runEffect`, `effect`)으로 취급하여 그 실행을 외부로 넘겼다. 또한 map, flatMap 함수를 이용하여 합성까지 할 수 있도록 되어 있다. 이렇게 값으로 취급된 부수효과를 내가 원하는대로 합성하는 Pure Function을 함수형 프로그래밍에 그토록 열광하는 이유라고 본다.

![](https://images.velog.io/images/icednut/post/837e92a3-94dd-41b5-8ead-27df84a87396/7.png align="left")

위 코드에서 `twoRuns.runEffect()` 함수를 호출하게 되면 어떻게 될까? 그 과정을 살펴보면 다음과 같다.

* 먼저 io, 즉 [`LazyIO.io`](http://LazyIO.io)`(println("Hello, world!"))`를 살펴보면 LazyIO 오브젝트에 io 라는 함수의 파라미터(`effect: ⇒ A`)로 `println("Hello, world!")` 이라는 thunk를 전달했다.
    
* 그 다음 `io.flatMap(...)`을 살펴보면 `_ ⇒ io` 라는 함수를 flatMap의 파라미터로 전달했다.
    
* `LazyIO.flatMap` 내부에선 어떤 일이 펼쳐질까?
    

![](https://images.velog.io/images/icednut/post/f4cb7e9e-e436-4982-a03b-3bac252b01a0/8.png align="left")

* 우선 LazyIO 오브젝트의 `io` 함수는 thunk 이기 때문에 즉시 실행이 되질 않는다. 따라서 `fn(runEffect()).runEffect()` 이 부분은 즉시 실행이 되질 않는다.
    
* 이제 `fn(runEffect()).runEffect()`는 LazyIO 오브젝트의 `io` 함수의 파라미터로 넘겨지게 되는데 넘겨받은 파라미터는 LazyIO 케이스 클래스에 멤버 필드, 즉 함수 값으로 전달된다. (아래 코드를 설명하고 있는 것임)
    

![](https://images.velog.io/images/icednut/post/8360c480-477e-4468-9cb3-51a275e98f64/9.png align="left")

* 여기서 `fn(runEffect()).runEffect()`을 다시 살펴보자.
    
* `fn(runEffect())`의 반환값 자료형을 살펴보면 `LazyIO[B]` 이다.
    
* `fn(runEffect())`를 다시 쓰면 `fn((() ⇒ println("Hello, world!"))())` 가 되며 `() ⇒ println("...")` 이 부분은 첫 번째 `io`의 멤버 필드를 의미한다. (괄호가 좀 많아서 자세히 봐야 됨)
    
* 그런데 지금 설명하고 있는 영역은 thunk 영역이기 때문에 fn의 파라미터 `(() ⇒ println("..."))()`은 즉시 실행이 되질 않는다.
    
* `fn`을 주목하자. `fn`을 풀어서 쓰면 `_ ⇒ io`를 의미한다. 그런데 여기서는 `fn(...)` 이기 때문에 `LazyIO[Unit]` 이라고 볼수 있다.
    
* 결국 `fn(...).runEffect()`는 위에서 말한 `LazyIO[Unit].runEffect()` 라고 볼 수 있으며 마찬가지로 아직 thunk 영역이기 때문에 `runEffect()`를 호출한 것처럼 써있지만 아직 실행되지는 않는다.
    
* 마지막에 twoRuns의 `runEffect()`를 호출하게 되면 위 과정에서 합성된 효과들이 한꺼번에 실행이 된다.
    

위와 같은 과정으로 인해 부수효과는 실행되지 않은채 `LazyIO[Unit]` 혹은 `LazyIO[(Unit, Unit)]` 이라는 값으로 취급되며 언제든지 필요할 때 Lazy Evaluation을 할 수 있게 된다.

# Next Level: Cats Effect

Scala 생태계에 LazyIO와 같은 컨셉을 갖춘 Cats Effect 라는 라이브러리가 있다. Cats Effect의 IO 라는 클래스가 LazyIO와 비슷하다고 할 수 있는데 Cats Effect를 이용하여 부수효과를 핸들링하는게 직접 구현한거보다 더 많은 기능을 제공하고 있고 Cats 생태계를 이용하기에도 편할 것 같다.

![](https://images.velog.io/images/icednut/post/5b894b41-1ea4-448e-9f31-8eda4eadbba6/carbon.png align="left")

![](https://images.velog.io/images/icednut/post/14cda8f8-fde1-4a1a-9923-10958cb3436b/10.png align="left")

unsafeRunAndForget 이라는 메소드가 좀 거슬리니 Cats Effect에서 제공하는 IO 핸들링 메커니즘을 이용하자.

![](https://images.velog.io/images/icednut/post/8d50011a-7d38-4d8d-8f9f-1ac23344cae5/11.png align="left")

하나 더 덧붙이자면 이렇게 부수효과를 합성 가능한 값으로 다루는 자료형인 IO를 모나드라고 부른다. 위에 코드에서는 IO 모나드만 반환했는데 `LazyIO.runEffect`를 호출하듯이 IO 모나드를 실행하는 외부는 어디일까?

위에 코드에는 다 표현되지는 않았지만 twoRuns는 `IO[Unit]` 타입이다. 이걸 Future로 바꾸는 implicit 함수가 이미 EffectTestSupport 라는 트레이트에 선언되어 있어서 이게 작용하게 된다.

![](https://images.velog.io/images/icednut/post/d351c292-b63f-4a2e-b1d9-7d36812b4677/12.png align="left")

![](https://images.velog.io/images/icednut/post/c40c7f3e-31f3-43d8-a8ad-17051b91d65f/13.png align="left")

Future로 변환하는 과정에서 unsafeToFuture 하는 메소드를 호출하게 되는데 Future로 변환하려면 어쩔 수 없이 이 메소드를 호출해야 되나보다. 이 다음은 org.scalatest.funsuite.AsyncFunSuite 로 인해 비동기로 검증 작업을 진행하게 된다. (이 부분 부터는 Scalatest 영역이라 설명 생략)

# 다음 할 일

* 그렇다면 기존에 `scala.concurrent.Future`로 구현된 코드에 Cats Effect를 적용하여 부수효과를 효율적으로 핸들링 해야 한다면 어떻게 해야될까? 여기 이에 대한 괜찮은 아티클이 하나 있던데 좀 더 공부해보자. [https://www.innoq.com/en/blog/functional-service-in-scala/#fn:2](https://www.innoq.com/en/blog/functional-service-in-scala/#fn:2)
    
* Cats Effect의 IO를 다룰 때 Resource로 감싸서 사용하던데 이 때 Higher Kinded Type을 사용한다. 같이 조사해보자.
    

# 참고 자료

* [https://www.baeldung.com/scala/cats-effects-intro](https://www.baeldung.com/scala/cats-effects-intro)
    
* [https://typelevel.org/cats-effect/docs/2.x/datatypes/io](https://typelevel.org/cats-effect/docs/2.x/datatypes/io)
    
* [https://stackoverflow.com/questions/53682686/cats-effect-and-asynchronous-io-specifics](https://stackoverflow.com/questions/53682686/cats-effect-and-asynchronous-io-specifics)
    
* [http://www.beyondthelines.net/programming/cats-effect-an-overview/](http://www.beyondthelines.net/programming/cats-effect-an-overview/)
    
* [https://medium.com/@TamasPolgar/hack-how-to-use-scala-futures-with-cats-io-9278c7febc37](https://medium.com/@TamasPolgar/hack-how-to-use-scala-futures-with-cats-io-9278c7febc37)
    
* [https://stackoverflow.com/questions/33386622/what-exactly-does-effectful-mean](https://stackoverflow.com/questions/33386622/what-exactly-does-effectful-mean)
    
* [https://levelup.gitconnected.com/what-is-effect-or-effectful-mean-in-functional-programming-7fc7323b52b4](https://levelup.gitconnected.com/what-is-effect-or-effectful-mean-in-functional-programming-7fc7323b52b4)