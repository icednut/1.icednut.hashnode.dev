---
title: "Newtype과 Refined를 이용한 Strongly Typed Function 작성하기"
datePublished: Sat Sep 25 2021 15:00:00 GMT+0000 (Coordinated Universal Time)
cuid: clon218da000a09kvdalmaofv
slug: newtype-refined-strongly-typed-function
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1699284316047/80c91a23-418e-4ded-b7c9-493016bfb66e.jpeg
tags: scala, newtype, refined

---

# 개요

스칼라에서 중요한 요소를 하나 꼽으라고 한다면 타입(Type Signature)일 것이다. 만약 타입이 맞지 않은 경우 컴파일 오류가 발생한다. 하지만 스칼라라고 하더라도 타입을 잘못 사용하여 약타입 함수를 만들어 오동작을 일으킬수도 있다.

여기서는 스칼라를 통해 작성한 약타입 함수를 살펴보고 컴파일 타임에서 타입을 좀 더 강제할 수 있는 강타입 함수를 만드는 방법을 살펴본다.

# 약타입 함수

사용자 이름과 이메일을 파라미터를 통해 사용자를 검색하는 함수를 만들어보자.

```scala
def lookup(username: String, email: String): F[Option[User]]
```

보통 위와 같이 함수 시그니처를 생각한다. 사용자 이름(username)은 String 타입으로 email은 String 타입으로 파라미터를 셋팅할 것이다. 하지만 타입을 String으로 정했다고 해서 올바른 String 값이 들어온다는 보장까지 할 수는 없다.

```scala
lookup("wilk", "wilk@example") // good
lookup("wilk@example.com", "wilk") // bad
lookup("", "")
```

username과 email는 String 타입으로만 강제할 뿐 그 내용까지는 강제할 수 없다. 따라서 lookup 함수는 약타입 함수라고 할 수 있으며 의도하지 않은 String 타입의 값이 들어올 수도 있다.

그럼 같은 타입의 파라미터들이 있을 때 잘못된 형식의 값이 들어오는 것 자체를 막으려면 어떻게 해야될까? 컴파일 타임에 String 이라는 타입 뿐만 아니라 잘못된 String 값이 들어오지 못하게 할 수는 없을까?

# Value Class 사용하여 좀 더 강력한 타입 체크를 하는 함수 만들기

일단 컴파일 타임에 값을 강제하는 것은 나중에 생각하기로 하고 스칼라의 값 클래스(`scala.AnyVal`)라는 추상 클래스를 사용하여 런타임 시 유효한 값만 오도록 체크하도록 해보자. 즉 아래와 같이 username은 Username이라는 값 클래스 타입으로 email은 Email 이라는 값 클래스 타입으로 말이다.

```scala
case class Username(value: String) extands AnyVal
case class Email(value: String) extends AnyVal

def lookup(username: Username, email: Email): F[Option[User]]
```

그러나 잘못된 값이 들어올 수 있는 여지는 여전히 존재한다. 결국엔 값 클래스도 강타입 함수를 만들지는 못한다.

```scala
lookup(Username("wilk@example.com", Email("wilk"))
lookup(Username(""), Email(""))
```

값 클래스 생성 시 잘못된 값이 들어가는지 체크하는 스마트 컨스트럭터를 만들면 어떨까? 그렇게 하여 Username, Email은 스마트 컨스트럭터를 통해서만 생성할 수 있게 하는 것이다.

```scala
import cats.effect.Async
import cats.implicits._

object StrongTypedFunctionExercise {

  // smart constructor
  def mkUsername(value: String): Option|Username] =
    (value.nonEmpty).guard[Option].as(Username(value))
  def mkEmail(value: String): Option[Email] =
    (value.contains("@")).guard[Option].as(Email(value))
}

class StrongTypedFunctionExercise[FI_1 : Async] {

  def lookup(username: Username, email: Email): FlOption[User]] = ...
}

// 생성자가 private case classes
case class Username private(value: String) extends AnyVal
case class Email private(value: String) extends AnyVal
```

```scala
import cats.effect.IO
import cats.effect.testing.scalatest.AsnycIOSpec
import io.icednut.strongtyped.step1.StrongTypedFunctionExercise.{mkEmail, mkUsername}
import org.scalatest.funsuite.AsyncFunSuite

class StrongTypedFunctionExerciseTest extends AsyncFunSuite with AsyncIOSpec {

  test("강타입 함수 연습하기") {
    val exercise = new StrongTypedFunctionExercise[IO]

    val result = for {
      username <- mkUsername("will")
      email <- mkEmail("wilk@example.com")
    } yield {
      exercise.lookup(username, email)
    }
  }
}
```

그러나 여기에도 문제가 있다. 스마트 컨스트럭터에서 잘못된 값을 거를 수는 있지만 결과물은 case class 이기 때문에 `copy` 라는 메소드를 통해 필드가 잘못된 값으로 오염될 수 있다.

![](https://images.velog.io/images/icednut/post/38444414-07ef-4068-972a-a77695c028f5/07.png align="left")

또한 case class 구현한 값 클래스에는 이슈가 하나 있다. 바로 메모리 할당(Memory Allocation) 이슈이다. 스칼라 공식 레퍼런스 문서([https://docs.scala-lang.org/overviews/core/value-classes.html)에는](https://docs.scala-lang.org/overviews/core/value-classes.html)에는) Value Class에 대해 이런 말이 적혀있다.

> Because the JVM does not support value classes, Scala sometimes needs to actually instantiate a value class. Full details may be found in [SIP-15](https://docs.scala-lang.org/sips/pending/value-classes.html).
> 
> Allocation Summary A value class is actually instantiated when:
> 
> 1. a value class is treated as another type.
>     
> 2. a value class is assigned to an array.
>     
> 3. doing runtime type tests, such as pattern matching.
>     
> 
> ---
> 
> JVM은 값 클래스를 지원하지 않지만 때때로 스칼라에서는 값 클래스로 객체 생성을 할 필요가 있습니다. 자세한 내용은 SIP-15을 참고하십시오.
> 
> 메모리 할당 요약 값 클래스는 실제로 다음 상황에 객체 생성 로직이 발동됩니다:
> 
> 1. 값 클래스를 다른 타입으로 다뤄질 때
>     
> 2. 값 클래스가 배열에 할당될 때
>     
> 3. 패턴 매칭 같이 런타임 시 타입 검사를 할 때
>     

이게 무슨 말일까? 1번 부터 하나씩 살펴보자.

## 1\. Value Class가 다른 타입으로 다뤄질 때 인스턴스화 된다.

여기 trait를 구현한 Value Class가 있다.

![](https://images.velog.io/images/icednut/post/3bffdf8f-aa26-4dcb-b9cb-8e4029b0e342/08.png align="left")

그리고 `Meter`를 더하는 메소드도 만들어보자.

![](https://images.velog.io/images/icednut/post/7a119f23-3ab8-453d-8122-7be4d8fca534/09.png align="left")

`add` 라는 메소드는 `Meter` 말고도 다형성을 위해 `add` 메소드의 파라미터를 `Distance`로 했다. 근데 여기에 이슈가 있다. `add` 메소드를 바이트코드로 컴파일하여 살펴보자.

```scala
// 바이트코드로 컴파일한 결과
[[syntax trees at end of                   cleanup]] // CaseClassExercise.scala
package io.icednut.strongtyped.step1 {
  object CaseClassExercise extends Object {
    def add(a: io.icednut.strongtyped.step1.Distance, b: io.icednut.strongtyped.step1.Distance): io.icednut.strongtyped.step1.Distance = {
      case <synthetic> val x1: io.icednut.strongtyped.step1.Distance = a;
        case5(){
          if (x1.$isInstanceOf[io.icednut.strongtyped.step1.Meter]())
            {
              <synthetic> val x2: Double = (x1.$asInstanceOf[io.icednut.strongtyped.step1.Meter]().value(): Double);
              {
                val value: Double = x2;
                matchEnd4(value)
              }
            }
          else
            case6()
        };
        case6(){
          matchEnd4(0.0)
        };
        matchEnd4(x: Double){
          x
        }
      };
      val bValue: Double = {
        case <synthetic> val x1: io.icednut.strongtyped.step1.Distance = b;
        case5(){
          if (x1.$isInstanceOf[io.icednut.strongtyped.step1.Meter]())
            {
              <synthetic> val x2: Double = (x1.$asInstanceOf[io.icednut.strongtyped.step1.Meter]().value(): Double);
              {
                val value: Double = x2;
                matchEnd4(value)
              }
            }
          else
            case6()
        };
        case6(){
          matchEnd4(0.0)
        };
        matchEnd4(x: Double){
          x
        }
      };
      new io.icednut.strongtyped.step1.Meter(aValue.+(bValue))
    };
    def execute(): Unit = {
      CaseClassExercise.this.add(new io.icednut.strongtyped.step1.Meter(1.0), new io.icednut.strongtyped.step1.Meter(2.0));
      ()
    };
    def <init>(): io.icednut.strongtyped.step1.CaseClassExercise.type = {
      CaseClassExercise.super.<init>();
      ()
    }
  };
  abstract trait Distance extends Object;
  final case class Meter extends Object with io.icednut.strongtyped.step1.Distance with Product with Serializable {
    ...
  };
  <synthetic> object Meter extends scala.runtime.AbstractFunction1 with Serializable {
    ...
  }
}
```

바이트코드로 변환된 `add` 메소드의 파라미터를 보면 `Distance` 타입으로 되어 있는게 보인다. 이게 뭐가 문제란 말인가 라고 생각할 수도 있겠지만 `add` 메소드를 다음과 같이 변경한 뒤 바이트코드로 컴파일 해보면 상황이 달라진다.

![](https://images.velog.io/images/icednut/post/ddc76a5b-5081-4f09-a1a9-d5354b861d2b/10.png align="left")

```scala
[[syntax trees at end of                   cleanup]] // CaseClassExercise.scala
package io.icednut.strongtyped.step1 {
  object CaseClassExercise extends Object {
    def add(a: Double, b: Double): Double = {
      val aValue: Double = a;
      val bValue: Double = b;
      aValue.+(bValue)
    };
    def execute(): Unit = {
      val m: Double = 5.0;
      val array: Array[io.icednut.algorithm_exercise.leetcode.strongtyped.step1.Meter] = scala.Array.apply(scala.Predef.genericWrapArray(Array[io.icednut.algorithm_exercise.leetcode.strongtyped.step1.Meter]{new io.icednut.algorithm_exercise.leetcode.strongtyped.step1.Meter(m)}), (ClassTag.apply(classOf[io.icednut.algorithm_exercise.leetcode.strongtyped.step1.Meter]): scala.reflect.ClassTag)).$asInstanceOf[Array[io.icednut.algorithm_exercise.leetcode.strongtyped.step1.Meter]]();
      ()
    };
    def <init>(): io.icednut.algorithm_exercise.leetcode.strongtyped.step1.CaseClassExercise.type = {
      CaseClassExercise.super.<init>();
      ()
    }
  };
  abstract trait Distance extends Object;
  final case class Meter extends Object with io.icednut.strongtyped.step1.Distance with Product with Serializable {
    ...
  };
  <synthetic> object Meter extends scala.runtime.AbstractFunction1 with Serializable {
    ...
  }
}
```

파라미터 타입을 하위 타입으로 한 add 메소드의 컴파일 결과를 살펴보면 `Double` 값으로 치환된게 보인다. 즉 AnyVal case class가 value 필드로 직접 치환된게 보인다. 다형성을 위해 trait를 썼던게 오히려 메모리 오버헤드를 가져온 결과라고 볼 수 있다.

## 2\. 값 클래스가 배열에 할당될 때 인스턴스화 된다.

위에서 살펴보면 `Meter` 값 클래스를 이용하여 배열을 선언해보자.

![](https://images.velog.io/images/icednut/post/c47d8a70-8543-46bd-a471-34ecece95bd8/11.png align="left")

이 스칼라 코드를 컴파일하여 바이트코드로 변환해보면 또 다른 문제가 나타난다.

```scala
[[syntax trees at end of                   cleanup]] // CaseClassExercise.scala
package io.icednut.strongtyped.step1 {
  object CaseClassExercise extends Object {
    def execute(): Unit = {
      val m: Double = 5.0;
      val array: Array[io.icednut.strongtyped.step1.Meter] = scala.Array.apply(scala.Predef.genericWrapArray(Array[io.icednut.strongtyped.step1.Meter]{new io.icednut.strongtyped.step1.Meter(m)}), (ClassTag.apply(classOf[io.icednut.strongtyped.step1.Meter]): scala.reflect.ClassTag)).$asInstanceOf[Array[io.icednut.strongtyped.step1.Meter]]();
      ()
    };
    def <init>(): io.icednut.strongtyped.step1.CaseClassExercise.type = {
      CaseClassExercise.super.<init>();
      ()
    }
  };
  abstract trait Distance extends Object;
  final case class Meter extends Object with io.icednut.strongtyped.step1.Distance with Product with Serializable {
    ...
  };
  <synthetic> object Meter extends scala.runtime.AbstractFunction1 with Serializable {
    ...
  }
}
```

`m` 이라는 로컬 변수는 `Double`로 치환 되었지만 `Array[Meter]`라는 배열은 `Meter`가 값 클래스임에도 불구하고 `Double`로 치환되지 않고 오히려 `Meter` 클래스로 인스턴스화 하는 코드가 생겨났다. 역시 메모리 오버헤드라고 볼 수 있다.

## 3\. 값 클래스를 패턴매칭을 사용하여 타입 검사를 할 때 인스턴스화 된다.

`add` 메소드에 패턴매칭 코드를 추가하여 값을 체크하는 로직을 보충해보자.

![](https://images.velog.io/images/icednut/post/9cff9d7b-cb10-4001-bc28-b8ea4247521d/12.png align="left")

이 패턴 매칭 코드를 컴파일 해보면 다음과 같이 `Meter` 인스턴스로 만들어봐서 체크하는 코드를 볼 수 있다.

```scala
[[syntax trees at end of                   cleanup]] // CaseClassExercise.scala
package io.icednut.strongtyped.step1 {
  object CaseClassExercise extends Object {
    def add(a: Double, b: Double): Double = {
      val aValue: Double = {
        case <synthetic> val x1: Double = a;
        case4(){
          if (new io.icednut.algorithm_exercise.leetcode.strongtyped.step1.Meter(x1).ne(null))
            {
              val value: Double = x1;
              if (scala.Double.box(value).!=(null).&&(value.>(0.0)))
                matchEnd3(value)
              else
                case5()
            }
          else
            case5()
        };
        case5(){
          matchEnd3(0.0)
        };
        matchEnd3(x: Double){
          x
        }
      };
      val bValue: Double = {
        case <synthetic> val x1: Double = b;
        case4(){
          if (new io.icednut.algorithm_exercise.leetcode.strongtyped.step1.Meter(x1).ne(null))
            {
              val value: Double = x1;
              if (scala.Double.box(value).!=(null).&&(value.>(0.0)))
                matchEnd3(value)
              else
                case5()
            }
          else
            case5()
        };
        case5(){
          matchEnd3(0.0)
        };
        matchEnd3(x: Double){
          x
        }
      };
      aValue.+(bValue)
    };
	};
  abstract trait Distance extends Object;
  final case class Meter extends Object with io.icednut.strongtyped.step1.Distance with Product with Serializable {
    ...
  };
  <synthetic> object Meter extends scala.runtime.AbstractFunction1 with Serializable {
    ...
  }
}
```

바이트코드에 있는 `add` 메소드 중간에 살펴보면 타입 체크를 위해 Meter로 한 번 인스턴스화 해본 다음에 정상 동작하는지를 체크하고 있다. `Meter`가 `Double`로 치환 되었으니 패턴매칭 코드에도 `Double`로 치환되서 값 클래스에 값만 추출하는 코드로 컴파일 되겠거니 예상 했는데 `Meter` 인스턴스가 만들어지고 있다. 역시 메모리 오버헤드라고 볼 수 있다.

이렇게 `AnyVal`와 case class로 구현한 값 클래스에는 무심코 썼던 다형성과 배열, 패턴 매칭 코드에서 메모리 오버헤드를 불러 올 수 있다. 고작 저정도의 메모리 사용량이 뭐가 문제일까 생각하겠지만 실제 대용량 서비스에서 저런 메모리 낭비는 거슬릴 수가 있다.

# 일반 클래스로 구현한 값 클래스로 타입 강제하기

그럼 값 클래스를 케이스 클래스가 아닌 일반 클래스로 만들면 되지 않을까? 일반 클래스와 스마트 컨스트럭터를 사용한 최종 모습은 다음과 같다.

![](https://images.velog.io/images/icednut/post/9bb4d4cd-69e0-4a12-b7a4-5e401d146e52/13.png align="left")

![](https://images.velog.io/images/icednut/post/8b01439b-ab29-495b-b5a9-f06a1bc18bf5/14.png align="left")

하지만 여기에도 이슈는 있다. 바로 일반 class는 패턴 매칭이 되지 않는다.

![](https://images.velog.io/images/icednut/post/92d12258-5b2c-4b2f-8531-b2963128d3be/15.png align="left")

물론 아래와 같이 패턴 매칭 코드를 양보할 수도 있다.

![](https://images.velog.io/images/icednut/post/ce62781a-2bba-4f90-a0c4-971b1b44e948/16.png align="left")

하지만 일반 클래스로는 case class와 같이 필드까지 접근하는 강력한 패턴 매칭 코드는 사용할 수 없다. 흔히들 말하는 패턴 매칭 코드가 verbose 하게 되었다.

일반 클래스처럼 메모리 오버헤드가 없고 케이스 클래스처럼 패턴 매칭에도 코드 구현이 깔끔한 값 클래스를 만들순 없을까? 다행히 이 질문에 대한 답이 있다. 바로 Newtype이다.

# Newtype을 사용하여 값 클래스 구현하기 & 컴파일 타임에 값 내용까지 강제하기

Newtype을 사용하여 구현한 값 클래스는 다음과 같다.

```scala
libraryDependencies += "io.estatico" %% "newtype" % "0.4.4"
```

![](https://images.velog.io/images/icednut/post/0a75f3de-e89a-468b-83de-0bf1c4c6e1ed/17.png align="left")

코드를 보면 그냥 case class에 newtype이라는 어노테이션만 붙였는데 이 어노테이션은 매크로로 인해 컴파일 타임에 아래와 같이 치환된다.

![](https://images.velog.io/images/icednut/post/d50f1056-e59f-41c5-8a4a-c547c4f562ba/18.png align="left")

Newtype으로 구현한 case class는 스칼라의 case class와는 달리 copy 메소드나 부수적인 메소드가 없어졌기 때문에 컴파일된 결과 파일 크기도 줄어들었고 게다가 값 클래스 처럼 사용할 수 있다.

더 나아가서 Refined 라는 매크로 라이브러리를 사용하면 컴파일 타임에서 타입 체크 뿐만 아니라 그 내용까지 체크하여 컴파일 오류를 발생 시킬 수 있다. 바로 다음과 같이 말이다.

```scala
libraryDependencies += "eu.timepit" %% "refined" % "0.9.27"
```

![](https://images.velog.io/images/icednut/post/be1b1c2e-49d9-4e71-82b4-c4c0e1b84801/19.png align="left")

따라서 Newtype과 Refined를 이용하여 보다 가벼운 값 클래스를 작성함과 동시에 컴파일 타임 때 값 내용까지 체크할 수 있게 된다. 그 결과 처음에 살펴본 Username과 Email 예제는 다음과 같이 작성할 수 있다.

![](https://images.velog.io/images/icednut/post/4350dc5a-963c-455c-964d-0529a948bad3/20.png align="left")

![](https://images.velog.io/images/icednut/post/c9975509-2b26-42f4-b58b-f72a41063c10/21.png align="left")

![](https://images.velog.io/images/icednut/post/6d7eef92-91f1-4c9a-bc22-d89db55e2079/22.png align="left")

# 궁금증

* Newtype과 Refined의 동작 원리가 뭘까? 코드가 언제 어느 시점에 저렇게 변환 되는걸까?
    
* Runtime 시 Refined로 값 체크를 하려면 어떻게 해야될까?
    
* 인텔리제이 에서는 String Refined Contains 타입 인식이 잘 안되는거 같다. 인텔리제이 이슈인거 같은데 해결할 수 있는 방법이 없을까?
    

# 참고 자료

* [https://github.com/estatico/scala-newtype](https://github.com/estatico/scala-newtype)
    
* [https://github.com/fthomas/refined](https://github.com/fthomas/refined)
    
* [https://blog.rockthejvm.com/refined-types/](https://blog.rockthejvm.com/refined-types/)
    
* [http://fthomas.github.io/talks/2016-05-04-refined/#1](http://fthomas.github.io/talks/2016-05-04-refined/#1)
    
* [https://blog.softwaremill.com/a-simple-trick-to-improve-type-safety-of-your-scala-code-ba80559ca092](https://blog.softwaremill.com/a-simple-trick-to-improve-type-safety-of-your-scala-code-ba80559ca092)