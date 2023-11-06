---
title: "liftIO 2021 컨퍼런스 후기를 가장한 자아 성찰"
datePublished: Fri Oct 29 2021 15:00:00 GMT+0000 (Coordinated Universal Time)
cuid: clon26qm1000g09kvcygd9ayp
slug: liftio-2021
cover: https://cdn.hashnode.com/res/hashnode/image/stock/unsplash/6vAjp0pscX0/upload/a94fee051112aa8d3d50f5e4a9ae4e8c.jpeg
tags: liftio, 7luo7y2865w7iqkio2bhoq4sa, 7zwo7iiy7zivio2uhouhnoq3uouemouwjq

---

# 오래간만에 열린 함수형 프로그래밍 컨퍼런스, 그리고 나의 스칼라 경험

실로 오래간만이었다. 그나마 내가 경험한 마지막 함수형 프로그래밍 컨퍼런스(?)라곤 2018년도에 했던 스칼라 나이트였는데 그때까지만 해도 나에게 함수형 프로그래밍이란 미지의 세계였고 지적 유희로 밖에 보이질 않았다. 그 당시 Cats와 Shapeless를 얘기했던 거 같은데 들으면서도 내가 뭘 모르는지도 모를 정도로 나에겐 무의미했던 시간으로 다가왔었다.

그러나 사람 앞 일은 아무도 모른다고 했던가? 이직을 하면서 내가 스칼라를 하게 될 줄이야. 운 좋게 전임자 선배들이 남기고 간 Scala, Akka 프로젝트를 나 홀로 2년여간 유지 보수하는 것으로부터 나의 스칼라 경험은 시작하게 되었는데 그렇게 스칼라는 나에게 생존의 수단으로 다가왔다. 스칼라의 기본 문법만 알고 있던 난 거의 알아듣지 못했던 인수인계를 거쳐 불안했던 유지보수를 시작하자 당연하게도(?) 몇 달 동안은 손만 대면 장애가 끊이질 않았다. 계속 커져가는 서비스 상황에서 작은 기능 추가 만으로도 장애가 날 정도면 관리자 입장에선 차세대를 고려하지 않을 수 없었을 것이다. 하지만 회사의 커다란 서비스 중에 아주 작은 일부분에 불과했던 내가 맡았던 서비스는 그 장애 여파가 심각하지도 않았고 오히려 아무도 관심이 없어 보였다. 이런 상황이라면 아마 관리자에게 도움을 요청하여 스칼라 개발자 인력 충원을 요청하거나 그나마 잘 알고 있던 자바, 스프링으로 새로 개발하자고 했었을 것이다. 하지만 무슨 오기였을까. 이렇게 모르는 상태로 뒀다간 평생 모르는 상태로 남아있을 것 같다는 생각이 들어 그 스칼라 레거시 프로젝트의 처음부터 끝까지 씹어먹어보자는 생각이 들었다. 그렇게 야근을 가장한 스칼라와의 나홀로 싸움은 시작되었고 2년이 지난 최근에서야 전임 선배들의 스칼라 코드들의 의도가 조금은 보이기 시작하는 것 같았다.

![](https://images.velog.io/images/icednut/post/a90b8d4c-d15a-40d1-9ff7-c2159b985bb1/learning_curve.png align="left")

이미지 출처: [https://kwiki.devserum.com/ko/articles/tech-articles/2021-05-31-518-consecutive-days-algorithm-challenge](https://kwiki.devserum.com/ko/articles/tech-articles/2021-05-31-518-consecutive-days-algorithm-challenge)

하지만 그건 나의 오만한 착각이었다. 어느 정도 알고 있다고 생각했던 난 다음 신규 프로젝트를 아주 자신 있게(!) Scala, Akka 특히 최신 버전으로 개발을 시작했다. 이번에도 마찬가지로 별로 중요하지도 않은 서비스에다가 2년여간 유지 보수했던 기술을 이어서 쓴다는데 관리자 입장에선 딱히 반대가 나오질 않았다. 같이 개발을 시작했던 동료들은 스칼라에 대한 이해도가 높진 않았지만 나처럼 시간이 흘러 부딪히다 보면 자연스레 이해도가 비슷해질 거라 생각했다. 하지만 돈 받고 일하는 우리에게 프로젝트 오픈 일정이라는 꼬리표가 따라왔다. 그렇게 자신 앞에 놓인 문제 해결에도 벅찬 동료들은 나의 생각이 착각이라는 것을 증명해 주었고 일정에 쫓겨 출시한 서비스는 인프라 관련 코드에서 심각한 문제를 일으켜 회사 전체에 장애를 일으킬 정도로 큰 여파를 몰고 왔다. 그렇게 괴로운 시간들과 야근이 해일과 같이 나를 덮쳤다.

이쯤 되면 스칼라와 결별하고 나의 고향 자바, 스프링으로 돌아가거나 요즘 대세(?)인 코프링으로 돌아갈만하다고 생각할 것이다. 하지만 이 무슨 우매한 자신감이란 말인가. 신규 프로젝트에서 당당하게 또다시 Scala, Akka를 꺼내들어 개발을 시작하였고 이번엔 이전의 괴로운 상황들을 미연에 방지하기 위해 그동안 눈여겨보고 있어왔던 Cats와 Cats Effect, 그리고 그 친구들(Http4s, Circe, Doobie, ScalaPB, Chimney 등등)을 꺼내들었다. 하지만 토이 프로젝트로 만났던 Cats Effect를 실무에서 적용하려고 하자 또 다른 느낌으로 다가왔다. 장님이 코끼리 더듬듯이 몰래 야근해가면서 꾸역꾸역 Cats로 중반 정도 개발했을 무렵 더 이상 안되겠다는 생각이 들었다. Cats에 대해 예제로 겉핥기 학습만 하다 보니 막상 실무에서 쓰려고 하자 왜 써야 되는지 와닿지가 않았고 내가 개발한 것에 대해 자신감과 확신이 떨어지고 있다는 것을 느꼈다. 당연히 동료들은 자신의 몫을 소화하기에도 바빴기 때문에 같이 학습하자는 말은 사치였고 그렇다고 Cats를 이용한 함수형 프로그래밍, DDD를 경험한 선배도 내 주위엔 없었다. 하지만 어려움 중간에 기회가 있다고 하지 않았던가. NDC에서 인상 깊었던 D사의 스칼라를 이용한 게임서버 개발 경험 발표를 떠올렸던 난 지체 없이 발표자에게 연락을 취해 거의 매달리다시피 FP와 DDD에 대한 조언을 갈구했다. 그렇게 하여 딱 한 번 오프라인으로 만나긴 했지만 나에겐 가뭄에 단비와도 같은 만남이었고 그분이 했던 조언들은 나의 개발 방향을 일러주는 소중한 나침반과도 같았다. 같이 일해보고 싶다는 말이 목까지 차올랐지만 차마 그분의 앞길을 막을 순 없기에 조언들만 가슴에 새긴 채 또다시 고뇌와 인내의 시간을 아직까지 보내고 있었다.

피할 수 없으면 즐기라는 말도 있지 않는가. 나름 긍정적인(?) 마인드로 한창 인내의 시간을 보내고 있을 무렵 LiftIO라는 함수형 프로그래밍 컨퍼런스를 한다는 소식이 들려왔다. 이 무슨 하늘의 장난이란 말인가. 고뇌의 시간을 보내고 있는 나에게 이런 꿀같은 달콤한 소식이 믿기질 않았다. 현재 내가 겪고 있는 스칼라와 FP 개발 어려움에 대해 조금이라도 힌트를 얻을 수 있지 않을까 하는 기대감에 컨퍼런스를 참석했으나 역시 내가 뭘 모르고 있는지 한 번 더 확인하는 자리로 다가왔다. 첫 발표였던 처음 본 클로저 문법은 신기했고 호기심을 싹 틔웠으며 라인에서의 스칼라 개발과 도입 노하우는 무릎이 절로 탁 쳐지며 이런 방법이 있구나라고 일깨워주었다. 그 다음 발표들인 하스켈과 DDD 관점에서의 스칼라, ZIO 개발 경험 세션들은 기본에 더 충실해야겠다는 다짐을 하게 만드는 시간이었다. 40분이라는 짧은(?) 시간과 왜라는 당위성을 좀 더 듣고 싶었는데 이는 나의 욕심이자 연사분들의 전략이었을 것이라고 생각한다. 아무튼 연사분들 한 분 한 분마다 깊은 내공이 느껴졌고 나의 무지함을 또 한 번 일깨워줬던 자리였다. 특히 OOP에 익숙해 있던 사람들에게 함수형 프로그래밍을 익히기 위해서는 언런(Un-learn) 해야 된다는 말이 와닿았다.

# 인상 깊었던 발표

LiftIO 컨퍼런스의 발표 중 개인적으로 가장 인상 깊었던 발표는 '하스켈로 타입 안전한 GraphQL과 gRPC 서버 만들기' 였다. 제목만 봐선 하스켈로 gRPC를 어떻게 쓰고 GraphQL을 제공하기 위한 관련 라이브러리와 구현 방법을 소개하는 자리일 줄 알았는데 Type과 Kind 그리고 타입클래스와 타입레벨 리터럴 소개라는 전혀 예상치 못했던 발표였다. 이 발표를 듣고 그동안 내가 Type과 Kind의 차이도 이해하지 못했었다는 것을 일깨워줬던 시간이었다.

![](https://images.velog.io/images/icednut/post/218a34c1-9a86-42bf-9d26-39e34d1d6f51/Screen%20Shot%202021-10-30%20at%203.53.12%20PM.png align="left")

출처: 하스켈로 타입 안전한 GraphQL과 gRPC 서버 만들기 발표에서 Kind에 대한 설명의 한 장면 (왼쪽이 Type, 오른쪽이 Kind)

특히 발표자의 과감하고 직설적인 설명을 듣고 그 통찰력이 부럽다는 생각이 들었다. 사실 Scala, Cats Effect를 하면서 HKT(Higher Kinded Type)을 매번 만나게 되는데 이번 발표를 듣고 좀 더 확실히 파서 내 것으로 만들어야겠다는 다짐을 하게 만들었다.

![](https://images.velog.io/images/icednut/post/282d189d-49b8-436d-a1d7-8018dcbd1f50/Screen%20Shot%202021-10-30%20at%203.55.16%20PM.png align="left")

![](https://images.velog.io/images/icednut/post/fe9d1902-dc3c-4106-a1b1-b9281dbd90b9/Screen%20Shot%202021-10-30%20at%204.10.26%20PM.png align="left")

출처: 하스켈로 타입 안전한 GraphQL과 gRPC 서버 만들기 발표 슬라이드 중 한 장면

# 마치며

발표가 모두 끝난 뒤 패널 토론 시간에서 왜 컨퍼런스 이름을 LiftIO로 했냐는 질문에 함수형 프로그래밍의 개념에서 따오고 싶었는데 그중에 LiftIO가 떠올라 채택했다는 답변이 있었다. 스칼라에서는 LiftIO가 뭘까 하고 찾아봤더니 Cats Effect에서 다음과 같은 샘플 코드를 만났다.

```scala
import cats.effect.IO

trait LiftIO[F[_]] {
  def liftIO[A](ioa: IO[A]): F[A]
}
```

```scala
import cats.effect.{LiftIO, IO}
import scala.concurrent.Future

type MyEffect[A] = Future[Either[Throwable, A]]

implicit def myEffectLiftIO: LiftIO[MyEffect] =
  new LiftIO[MyEffect] {
    override def liftIO[A](ioa: IO[A]): MyEffect[A] = {
      ioa.attempt.unsafeToFuture()
    }
  }

val ioa: IO[String] = IO("Hello World!")
val effect: MyEffect[String] = LiftIO[MyEffect].liftIO(ioa)
```

코드 출처: [https://typelevel.org/cats-effect/docs/2.x/typeclasses/liftio](https://typelevel.org/cats-effect/docs/2.x/typeclasses/liftio)

컨퍼런스 이름에서 조차 나의 무지를 일깨워주는 소중한 시간이었고 내년에도 한다면 참석하는 것은 당연하거니와 좀 더 노력해서 청중에서 발표자로 Lift 되기를 꿈꾸며 이만 글을 줄인다.