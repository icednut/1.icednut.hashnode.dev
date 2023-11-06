---
title: "스프링의 Psa 개념을 이해했는지 확인하기 + 스칼라, Fp 버전으로 바꿔보기"
datePublished: Thu Feb 17 2022 15:00:00 GMT+0000 (Coordinated Universal Time)
cuid: clon29vht000e0alaemk51sn3
slug: psa-fp
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1699284669272/4011c337-2b14-4e59-b8dd-64ab43eea150.jpeg
tags: scala, psa, cats-effect

---

**참고로 이 글을 특정 유튜버나 인터넷 강의를 홍보하려고 쓴 글이 아니며 본문에 나오는 유튜브 영상 저자와 저는 전혀 관련이 없음을 밝힙니다.**

# 스프링을 제대로 공부했는가? 확인해보자.

얼마 전 유튜브를 보던 중 '[스프링 제대로 공부했는지 5분 안에 확인하는 방법](https://www.youtube.com/watch?v=bJfbPWEMj_c)'이라는 제목의 영상이 눈길을 끌었다. 뭘까 궁금해서 봤는데 결론은 스프링과 리팩토링 인터넷 강의를 홍보하는 영상이었다. 영상 내용은 특정 문제를 제시하고 스프링을 제대로 공부했다면 풀 수 있어야 한다고 풀어보라고 한 뒤 결과를 설명하면서 이게 잘 안된다면 자신의 강의를 보라는 내용이었다. 영상을 본 후에 곰곰이 생각해 보니 이걸 스칼라를 이용한 FP 버전으로 풀어보면 어떨까라는 생각이 문득 들었다. 그럼 스칼라 버전으로 개발하기 전에 영상에서 제시한 문제를 살펴보자.

문제는 이랬다. 아래 상황에서 테스트가 용이한 구조가 되려면 어떻게 개선하면 좋을까 라는게 문제였다.

> \[문제\]
> 
> * 아래 RepositoryRank 클래스의 getPoint 메소드에 대한 유닛 테스트를 작성한다고 해보자.
>     
> * GitHub.connect()라는 스태틱 메소드를 호출하고 있는데 이걸 Mock Framework 없이 Mocking 할 수 있는 구조로 개선하려면 어떻게 해야 될까?
>     

```java
import org.kohsuke.github.*;
import java.io.IOException;

public class RepositoryRank {

  public int getPoint(String repositoryName) throws IOException {
    GitHub github = GitHub.connect();
    GHRepository repository = github.getRepository(repositoryName);

    int points = 0;
    if (repository.hasIssues() ) {
      points += 1;
    }

    if (repository.getReadme() != null) {
      points += 1;
    }

    if (repository.getPullRequests(GHIssueState.CLOSED).size() > 0) {
      points += 1;
    }

    points += repository.getStargazersCount();
    points += repository.getForksCount();

    return points;
  }

  public static void main(String[] args) throws IOException {
    RepositoryRank spring = new RepositoryRank();
    int point = spring.getPoint("whiteship/live-study");
    System.out.println(point);
  }
}
```

```java
// https://github.com/hub4j/github-api/blob/main/src/main/java/org/kohsuke/github/GitHub.java
package org.kohsuke.github;

public class GitHub {

  ...

  /**
     * Obtains the credential from "~/.github" or from the System Environment Properties.
     *
     * @return the git hub
     * @throws IOException
     *             the io exception
     */
  public static GitHub connect() throws IOException {
    return GitHubBuilder.fromCredentials().build();
  }

   ...
}
```

# 이 상황을 어떻게 개선할까?

개선 힌트는 스프링 강의나 토비의 스프링 책에서 말했던 Portable Service Abstration이라는 개념을 말하고 있다. 즉 테스트하기 힘든 코드를 테스트하기 편리한 구조로 바꾸기 위해 추상화를 해야 한다는 게 골자다.

그럼 힌트를 염두에 두고 테스트 용이한 구조로 개선해 보자.

```java
import org.kohsuke.github.*;
import java.io.IOException;

public class RepositoryRank {

  interface GithubService {
    GitHub connect() throws IOException;
  }

  class GithubServiceImpl implements GithubService {

    @Override
    public GitHub connect() throws IOException {
      return GitHub.connect();
    }
  }

  private GitHubService githubService; 

  public RepositoryRank(GitHubService githubService) {
    this.githubService = githubService;
  }

  public int getPoint(String repositoryName) throws IOException {
    GitHub github = githubService.connect();
    GHRepository repository = github.getRepository(repositoryName);

    int points = 0;
    if (repository.hasIssues() ) {
      points += 1;
    }

    if (repository.getReadme() != null) {
      points += 1;
    }

    if (repository.getPullRequests(GHIssueState.CLOSED).size() > 0) {
      points += 1;
    }

    points += repository.getStargazersCount();
    points += repository.getForksCount();

    return points;
  }

  public static void main(String[] args) throws IOException {
    RepositoryRank spring = new RepositoryRank(new GithubServiceImpl());
    int point = spring.getPoint("whiteship/live-study");
    System.out.println(point);
  }
}
```

위와 같이 `GitHub.connect()`라는 static method를 호출하는 부분, 즉 GitHub 커넥션을 가져오는 부분을 추상화하여 그것을 DI를 통해 런타임 시 결정하게 하자는 것이 답이었다.

# 스칼라로는 어떻게 해결할 수 있을까?

답을 보고 나서 문득 드는 생각이 이걸 스칼라와 Functional Programming 기법을 활용하여 풀면 어떨까라는 생각이 들었다. 전에 이런 비슷한 상황을 마주친 적이 있는데 이 문제를 통해 다시 한번 확실하게 짚고 넘어가 보자.

일단 맨 처음 언급한 개선 전 상황의 Java 코드를 Scala 코드로 바꿔보자. (스칼라는 2.13.8 버전을 사용하였다)

```scala
package io.icednut.study

import org.kohsuke.github.{GHIssueState, GitHub}

class RepositoryRank {

  def getPoint(repositoryName: String): Int = {
    val github = GitHub.connect()
    val repository = github.getRepository(repositoryName)

    val issuePoint = if (repository.hasIssues) 1 else 0
    val readmePoint = if (repository.getReadme() != null) 1 else 0
    val prPoint = if (repository.getPullRequests(GHIssueState.CLOSED).size > 0) 1 else 0
    val starPoint = repository.getStargazersCount
    val forkPoint = repository.getForksCount

    issuePoint + readmePoint + prPoint + starPoint + forkPoint
  }
}

object RepositoryRank {

  def main(args: Array[String]): Unit = {
    val spring = new RepositoryRank
    val point = spring.getPoint("whiteship/live-study")

    println(point)
  }
}
```

결과물: [https://github.com/icednut/typeclass-refactoring/tree/master/src/main/scala](https://github.com/icednut/typeclass-refactoring/tree/master/src/main/scala)

그런 다음 타입클래스를 활용하기 위해 Cats Effect를 사용하여 IO 모나드를 활용하자.

타입클래스가 뭐고 타입클래스를 활용하기 위해서 왜 Cats Effect를 써야 되는지에 대한 설명은 좀 길어질 거 같으니 여기서는 코딩에 집중하기로 하자.

```scala
package io.icednut.study

import cats.Applicative
import cats.effect.{IO, IOApp}
import org.kohsuke.github.{GHIssueState, GitHub}

class RepositoryRank[F[_] : Applicative] {

  def getPoint(repositoryName: String): F[Int] = Applicative[F].pure {
    val github = GitHub.connect()
    val repository = github.getRepository(repositoryName)

    val issuePoint = if (repository.hasIssues) 1 else 0
    val readmePoint = if (repository.getReadme() != null) 1 else 0
    val prPoint = if (repository.getPullRequests(GHIssueState.CLOSED).size > 0) 1 else 0
    val starPoint = repository.getStargazersCount
    val forkPoint = repository.getForksCount

    issuePoint + readmePoint + prPoint + starPoint + forkPoint
  }
}

object RepositoryRank extends IOApp.Simple {

  override def run: IO[Unit] = {
    val spring = new RepositoryRank[IO]

    for {
      point <- spring.getPoint("icednut/typeclass-refactoring")
      _ <- IO.println(point)
    } yield ()
  }
}
```

결과물: [https://github.com/icednut/typeclass-refactoring/tree/with\_cats\_effect\_\_step\_1](https://github.com/icednut/typeclass-refactoring/tree/with_cats_effect__step_1)

자 그럼 이제 `Github.connect()` 라는 행위를 어떻게 타입클래스를 활용하여 추상화할까? 내가 생각한 개선 방안은 `GitHub.connect()` 스태틱 메소드를 호출하는 행위를 타입클래스로 만들어 컴파일 타임에 타입클래스를 결정하게 하는 것이다. 그럼 먼저 타입클래스 부터 만들어보자.

```scala
package io.icednut.study

import cats.Id
import cats.effect.IO
import org.kohsuke.github.GitHub

trait FromGithub[F[_]] {
  def connect: F[GitHub]
}

object FromGithub {

  def apply[F[_] : FromGithub]: FromGithub[F] = implicitly[FromGithub[F]]

  implicit def githubForIo: FromGithub[IO] = {
    new FromGithub[IO] {
      override def connect: IO[GitHub] = IO.delay(GitHub.connectAnonymously())
    }
  }

  implicit def githubForMock: FromGithub[Id] = {
    new FromGithub[Id] {
      override def connect: Id[GitHub] = {
        ??? // 여기가 테스트 코드를 위해 가짜 GitHub을 반환 (여기서는 시간 관계상 미구현으로 남겨둠)
      }
    }
  }

}
```

이제 `FromGithub` 이라는 타입클래스를 사용해보자. `FromGithub[F].connect` 부분에서 `F`에 따라서 실제 `GitHub` 이펙트를 받을지 가짜 `GitHub` 이펙트를 받을지가 결정된다.

메인메소드가 담긴 `RepositoryRank` 오브젝트를 실행해 보면 실제 `Github.connect()` 호출이 진행되는 것을 확인할 수 있다.

```scala
package io.icednut.study

import cats.Applicative
import cats.effect.{IO, IOApp}
import cats.implicits._
import org.kohsuke.github.GHIssueState

class RepositoryRank[F[_] : Applicative : FromGithub] {

  def getPoint(repositoryName: String): F[Int] =
    for {
      github <- FromGithub[F].connect
    } yield {
      val repository = github.getRepository(repositoryName)

      val issuePoint = if (repository.hasIssues) 1 else 0
      val readmePoint = if (repository.getReadme() != null) 1 else 0
      val prPoint = if (repository.getPullRequests(GHIssueState.CLOSED).size > 0) 1 else 0
      val starPoint = repository.getStargazersCount
      val forkPoint = repository.getForksCount

      issuePoint + readmePoint + prPoint + starPoint + forkPoint
    }
}

object RepositoryRank extends IOApp.Simple {

  override def run: IO[Unit] = {
    // FromGithub은 같은 패키지에 있기 때문에 import를 생략함
    val spring = new RepositoryRank[IO]

    for {
      point <- spring.getPoint("icednut/typeclass-refactoring")
      _ <- IO.println(point)
    } yield ()
  }
}
```

반면 테스트코드에서 가짜 `Github` 이펙트를 반환하도록 타입클래스를 지정할 수도 있다.

```scala
package io.icednut.study

import cats.Id
import org.scalatest.funsuite.AnyFunSuite

class RepositoryRankTest extends AnyFunSuite {

  test("테스트 코드를 작성해봅시다.") {
    // FromGithub은 같은 패키지에 있기 때문에 import를 생략함
    val value = new RepositoryRank[Id].getPoint("icednut/typeclass-refactoring")

    assert(value == 0)
  }
}
```

최종 결과물: [https://github.com/icednut/typeclass-refactoring/tree/with\_cats\_effect\_\_step\_2](https://github.com/icednut/typeclass-refactoring/tree/with_cats_effect__step_2)

# 다음 할 일

자세한 설명을 생략하고 코드 구현만 나열했는데 결과를 요약하자면 깃헙 커넥션을 가져오는 행위를 이펙트로 다루는게 시작이자 타입클래스로 이펙트를 추상화한게 끝이다. 이렇게 구현한 타입클래스는 암시적 파라미터로 컴파일 타임에 주입된다고 보면 된다.

Cats Effect를 활용한 스칼라 구현체의 테스트 코드에서 `cats.Id`를 사용했지만 사실 테스트 코드에서도 `cats.effect.IO`를 사용하는게 좋다고 한다. 여기서는 예시를 위해 `cats.Id`를 사용했지만 테스트코드도 `cats.effect.IO`로 통일할 수 있도록 타입클래스를 잘 정리해보자.