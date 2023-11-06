---
title: "타입클래스를 이용한 애드혹 다형성 구현하기 (Ad-hoc Polymorphism in Scala)"
datePublished: Wed Oct 20 2021 15:00:00 GMT+0000 (Coordinated Universal Time)
cuid: clon24k6z000h09l75dqdchnk
slug: ad-hoc-polymorphism-in-scala
cover: https://cdn.hashnode.com/res/hashnode/image/stock/unsplash/r-O95aZ6wvI/upload/12bd1ab0e00081b8bcc39a97ee132887.jpeg
tags: scala, typeclass, ad-hoc-polymorphism

---

> Polymorphism의 어원 = Poly(Many, 많은) + Morph(Shape, 형태)

OOP 언어에서 빠지지 않고 나오는 개념인 이 다형성은 보통 서브타이핑을 이용한 추상화 개념을 떠올리곤 한다. 비슷한 형식을 갖고 있는 클래스들을 인터페이스로 추상화하여 특성에 맞게 구현체들을 구현하여 런타임 시 구현체를 결정하도록 사용하곤 한다.

하지만 다형성에는 서브타입 다형성 말고도 또 다른 다형성들이 있다. 바로 파라미터 다형성(Parametric Polymorphism)과 애드혹 다형성(Ad-hoc Polymorphism)인데 파라미터 다형성은 지면 관계상 간략하게만 짚고 넘어가자면 Java의 제네릭(Generic) 혹은 List.add 메소드를 생각하면 된다. 즉, 타입 파라미터를 이용하여 일관성 있는 add 라는 메소드 제공하는 것이 바로 파라미터 다형성의 대표적인 예시라고 볼 수 있는데 자세한 내용은 링크의 Parametric Polymorphism 부분을 살펴보자([https://twitter.github.io/scala\_school/ko/type-basics.html#parametricpoly](https://twitter.github.io/scala_school/ko/type-basics.html#parametricpoly)).

```java
interface List<E> {
  boolean add(E element);
  void add(int index, E element);
  E get(int index);
}
```

서론이 길었는데 이 글에서 살펴보고자 할 다형성은 바로 애드혹 다형성이다. 애드혹 다형성이라는 말이 생소해보일 수도 있겠지만 사실 Java에서는 이미 Overloading 이라는 개념을 이용하여 애드혹 다형성을 구현할 수 있다. System.out.print() 메소드가 대표적인 예시라고 볼 수 있는데 그 구현체를 살펴보면 다음과 같다.

```java
package java.lang;

import java.io.PrintStream;

public final class System {
  public static final PrintStream out;

  ...
}
```

```java
package java.io;

public class PrintStream extends FilterOutputStream implements Appendable, Closable {

  ...

  public void print(boolean b) { ... }
  public void print(char c) { ... }
  public void print(int i) { ... }
  public void print(float f) { ... }
  public void print(double d) { ... }
  public void print(char[] s) { ... }
  public void print(String s) { ... }
  public void print(Object obj) { ... }

  ...
}
```

이와 같이 특정 타입의 데이터를 콘솔 출력 시 print 라는 메소드를 사용하여 파라미터로 넘겨주면 되는데 사용하는 쪽 입장에선 출력할 타입이 뭐가 되었든지간에 print 라는 메소드만 사용하면 된다. 이렇게 오버로딩을 이용한 다양한 파라미터 형식에 대응하는 print 라는 일관된 메소드를 제공하기 때문에 이를 다형성이라고 볼 수 있는데 특히 이런 형식의 다형성은 애드혹 다형성이라고 부른다. (링크 참조: [http://www.btechsmartclass.com/java/java-polymorphism.html](http://www.btechsmartclass.com/java/java-polymorphism.html))

근데 왜 애드혹 다형성이라고 부를까? 이 질문에 대한 답은 있다가 천천히 찾기로 하고 스칼라에서는 애드혹 다형성을 어떤 방식으로 해결하고 있는지 살펴보자.

> 참고로 스칼라의 print 메소드(scala.Predef.print)를 살펴보면 자바와는 다르게 print(x: Any) 메소드 하나만 선언되어 있다.

# 애드혹 다형성을 적용할 문제 정의

그럼 스칼라에서는 애드혹 다형성을 사용한 추상화를 어떤 식으로 제공하는지 살펴보기 위해 이를 적용할 문제 상황을 하나 정의해보자.

## 문제

![](https://images.velog.io/images/icednut/post/a82413ad-0514-4338-8fd9-75e13e71607e/1.png align="left")

* 리스트에 모든 원소를 더하는 processMyList라는 메소드를 구현해보자.
    
* 예를 들어 List\[Int\]을 다루는 processMyList은 모든 원소의 합계를, List\[String\]을 다루는 processMyList은 모든 원소를 concat 해야 한다.
    

이 문제를 어떻게 풀어야할까? Java에서 말한 애드혹 다형성 처럼 오버로딩을 이용할까?

![](https://images.velog.io/images/icednut/post/6ebf6b0e-689b-4ad2-b50c-9c089080afc5/2.png align="left")

스칼라와 같은 함수형 프로그래밍 언어에서는 이런 애드혹 다형성을 오버로딩이 아닌 다른 방식으로 해결할 수 있다. 바로 타입클래스(Typeclass)를 이용하는 것이다.

# 타입클래스를 이용한 애드혹 다형성 구현하기

드디어 이번에 다룰 주제에 대한 본론으로 들어왔다. 한 마디로 결론을 요약하자면 processMyList라는 메소드는 파라미터 다형성을 이용하고 aggregate 이라는 행위도 한 번 더 추상화한 타입클래스를 암시적 파라미터로 넘기는 것이다. 타입 클래스가 뭐고 암시적 파라미터를 어떻게 넘기는 건지 의아해 할 수도 있을 거 같은데 천천히 그 구현 내용을 살펴보자.

![](https://images.velog.io/images/icednut/post/7716bbcb-5f1e-4409-98be-726c6387152e/3.png align="left")

![](https://images.velog.io/images/icednut/post/0e407336-d491-4037-b530-2146776f2bd6/4.png align="left")

![](https://images.velog.io/images/icednut/post/fbb0bd40-b490-45be-a0ac-9727b383ccc9/5.png align="left")

이걸 사용하면 다음과 같다.

![](https://images.velog.io/images/icednut/post/92002591-c537-40e4-8e56-0d66a3b9e652/6.png align="left")

부연 설명을 하자면 aggregate할 리스트의 원소 타입이 뭐든지 간에 processMyList라는 일관된 메소드를 제공하지만 합계를 구하는 행위도 추상화하여 추가로 암시적 파라미터를 이용하여 넘겼다. 즉 sumElements 라는 추상화의 구현을 애드혹으로 따로 제공하기 때문에 애드혹 다형성이라고 부른다. 이렇게 함으로써 오버로딩을 이용한 애드혹 다형성보다 더 간결한 코드의 애드혹 다형성을 구현할 수 있게 되었다.

![](https://images.velog.io/images/icednut/post/47d490ed-2763-46ca-ac9e-43b384ddbf85/7.png align="left")

특히 타입 파라미터 T를 취하는 Summable 트레이트를 구현하여 특정 타입에 대한 더하는 행위들을 정의할 수 있는데 이런 인터페이스를 가리켜 함수형 프로그래밍에서는 타입클래스 라고 부르며 이를 상황에 맞게 구현하여 사용한다.

![](https://images.velog.io/images/icednut/post/0e407336-d491-4037-b530-2146776f2bd6/4.png align="left")

# 타입 클래스 집합소: Cats

그럼 앞으로 애드혹 다형성을 구현하기 위해 타입클래스를 직접 만들어야 될까? 그렇지 않다. 우리가 익히 알고 있는 Cats 라는 라이브러리가 바로 타입클래스의 집합소이다. Cats에 대표적인 타입클래스는 우리들이 즐겨(?) 들어왔던 Monoid, Functor, Applicative, Monad가 바로 그 주인공이다.

![https://cdn.rawgit.com/tpolecat/cats-infographic/master/cats.svg](https://cdn.rawgit.com/tpolecat/cats-infographic/master/cats.svg align="left")

(이미지 출처: [https://cdn.rawgit.com/tpolecat/cats-infographic/master/cats.svg](https://cdn.rawgit.com/tpolecat/cats-infographic/master/cats.svg))

게다가 Cats의 타입클래스는 타입클래스끼리 합성까지 할 수 있는 composition이라는 특성을 제공하니 상황에 맞게 적절하게 사용하면 된다.

![](https://images.velog.io/images/icednut/post/86136453-223a-4f03-879e-933d0936e92b/8.png align="left")

(출처: [https://www.scala-exercises.org/cats/functor](https://www.scala-exercises.org/cats/functor))

# 마무리

자 이제 함수형 프로그래밍에 한 발 더 딛게 되었다. 아직은 어떤 상황에서 어떤 타입클래스를 사용해야 될지가 막막한데 아래 그림의 할아버지 처럼 되기를 꿈꾸며 계속 전진해보자!

![](https://images.velog.io/images/icednut/post/874500a6-1463-418a-ac6d-0cce5b5e29bf/7BCF7565-A8FC-4782-BAFE-092198A885F6.png align="left")

(이미지 출처: [https://www.innoq.com/en/blog/functional-service-in-scala/#fn:2](https://www.innoq.com/en/blog/functional-service-in-scala/#fn:2))

# 참고자료

* [https://www.baeldung.com/scala/type-classes](https://www.baeldung.com/scala/type-classes)
    
* [https://www.baeldung.com/scala/polymorphism](https://www.baeldung.com/scala/polymorphism)
    
* [http://tpolecat.github.io/2015/04/29/f-bounds.html](http://tpolecat.github.io/2015/04/29/f-bounds.html)
    
* [https://vmayakumar.wordpress.com/2020/12/30/adhoc-polymorphism-implicts-and-pimping-in-scala/](https://vmayakumar.wordpress.com/2020/12/30/adhoc-polymorphism-implicts-and-pimping-in-scala/)
    
* [https://typelevel.org/cats/typeclasses.html](https://typelevel.org/cats/typeclasses.html)
    
* [http://hannosprogrammingblog.blogspot.com/2017/12/delegation-and-ad-hoc-polymorphism-in.html](http://hannosprogrammingblog.blogspot.com/2017/12/delegation-and-ad-hoc-polymorphism-in.html)
    
* [https://blog.daum.net/it-focus/143](https://blog.daum.net/it-focus/143)