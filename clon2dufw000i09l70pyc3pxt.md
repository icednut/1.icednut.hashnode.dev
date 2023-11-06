---
title: "liftIO 2022 컨퍼런스 발표자 참석 후기와 Lazy Evaluation에 대한 못다 한 답변"
datePublished: Sat Dec 03 2022 15:00:00 GMT+0000 (Coordinated Universal Time)
cuid: clon2dufw000i09l70pyc3pxt
slug: liftio-2022-lazy-evaluation
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1699284882816/3f5b26dc-f20e-4b9b-b1a8-209af61d9dff.jpeg
tags: liftio, 7luo7y2865w7iqkio2bhoq4sa, 7zwo7iiy7zivio2uhouhnoq3uouemouwjq

---

# 올해 liftIO 2022는 발표자로 참석하였다.

* [https://festa.io/events/2876](https://festa.io/events/2876)
    

liftIO 발표자 참석은 작년에 나의 작은 소망이었는데 liftIO가 올해도 개최된다는 소식을 듣고 약간의 흥분과 용기를 머금고 발표자 신청하게 된 것이다. 주위의 응원과 기대 속에 나조차도 내가 무슨 말을 하게 될지 기대가 되었던 발표 준비를 했던 것 같다. 컨퍼런스 주제는 함수형 프로그래밍에 대한 성과였지만 나 잘났다고 혼자만 떠드는 것이 아닌 함수형 프로그래밍에 대한 공감과 관심을 끌어내기 위함이 또 다른 내 목표였는데 과연 잘 전달되었을까?

# 시간을 되돌린다면

발표가 끝난 후 사람들이 질문을 할까 관심이 없으면 어쩌나 걱정했는데 예상외로 많은(?) 관심을 받아 모기 때려잡다 피묻은 내 손을 보고 놀란 것과 같은 심정이었다. 나왔던 질문 중 아래 질문이 아직도 내 머릿속을 맴돈다.

> 그래서 Lazy Evaluation이 왜 좋은가요?

질문받은 당시 좋고 나쁨의 관점으로 바라본 Lazy Evaluation으로 답하려 했었는데 돌이켜 생각해보니 IO 모나드를 쓰면 부수적으로 딸려오게 되는 Lazy Evaluation이라는 관점으로 설명하는 게 더 좋지 않았을까 라는 생각이 든다.

## 1\. Stack Safe

사이드 이펙트를 IO 모나드로 감싸게 되면(IO.delay) 그 사이드 이펙트는 아직 실행되지 않은 채로 IO 모나드에 갇혀있게 된다. 이렇게 여러 개의 IO 모나드를 생성할 수 있을 텐데 함수형 프로그래밍의 묘미인 모나드 합성을 통해 IO 모나드들을 한 개의 IO 모나드로 합성할 수 있게 된다.

![](https://velog.velcdn.com/images/icednut/post/9da21af3-a488-45e8-a362-cc66338e854b/image.png align="left")

sum() 재귀호출 예제 코드는 sum 함수를 호출하는 즉시 합산 계산을 진행하기 때문에(Eager Evaluation이기 때문에) 계산에 필요한 값을 재귀 호출 중간중간 콜 스택에 담는 것이 즉시 실행되어 Stack Overflow가 발생할 수 있는 반면

![](https://velog.velcdn.com/images/icednut/post/a2c10231-483e-4d4a-bf28-dbcb3871d757/image.png align="left")

재귀 호출에 IO 모나드를 사용하는 위와 같은 경우 for-comprehension 중간에 있는 sum() 재귀 호출은 반환 값이 IO 모나드이므로 for-comprehension의 특성상 flatMap과 map 함수가 동원되어 IO 모나드를 1개의 모나드로 합치는 현상이 나타난다. 이러한 재귀호출 과정에는 콜 스택에 이전 실행 결과값을 남기지 않기 때문에 Stack Safe 하게 된다. 또한 이 과정에서 IO 모나드에 갇힌 사이드 이펙트는 재귀호출이 끝난 이후에도 실행되지 않았기 때문에 사이드 이펙트 관점에서 봤을 땐 Lazy Evaluation 하다고 볼 수 있다.

이렇기 때문에 IO 모나드를 쓰면 Lazy Evaluation 하기 때문에 함수 호출에 Stack Safe 할 수 있게 된다는 장점이 있다.

## 2\. 실행 제어

![](https://velog.velcdn.com/images/icednut/post/dc34c847-d3bd-4153-ac83-ac9448b5118f/image.png align="left")

IO는 실행 시점이 다가오면 파이버로 변환하여 Cats Effect Runtime이 실행 관리를 하게 된다. 이렇게 아직 실행되지 않은(Lazy Evaluation 하게 된) IO에 속한 사이드 이펙트는 JVM 스레드보다 더 적은 정보를 가진 파이버로 변환되어 WorkStealingThreadPool 내부에 있는 스레드풀의 스레드에 할당이 되어 실행하게 된다. Cats Effect에 따르면 스레드에 할당된 파이버의 실행이 오래 걸릴 경우 다시 큐로 옮겨져 다른 파이버 실행을 할당하는 등 효율적인 비동기 실행을 지원한다.

이처럼 Cats Effect에 국한되어 설명하자면 IO 모나드를 통한 Lazy Evaluation은 파이버로 변환하여 Cats Effect에 의해 비동기 실행으로 제어되므로 JVM 스레드풀을 직접 사용하는 것보다 적은 리소스를 사용하게 된다.

Cats Effect Runtime과 파이버에 대한 동작 원리 설명은 추후 구체적인 자료를 바탕으로 다시 설명해봤으면 한다.

# 그래서 발표는 즐거웠나요?

발표 준비를 하면서 시간 관계상 많은 부분을 덜어내고 회사와 연관된 내용은 추상적으로 얘기할 수밖에 없었던 것이 아쉬움으로 남아있다. 그리고 성격상 먼저 다가가서 사람들과 함수형 프로그래밍에 관한 얘기를 나누지 못한 것도 아쉬웠지만 이건 뭐 내 성격이기도 하고 워낙 말주변도 없기 때문에 그러려니 한다.

발표는 끝났지만 후련하다기보다 오히려 또 다른 문을 연 느낌이다. 내년 liftIO 2023이 벌써 기다려지는데 그때까지 For lambda calculus!

## 참고: 내 발표자료

* [https://speakerdeck.com/icednut/scala-cats-effectro-saseo-gosaenghan-hamsuhyeong-peurogeuraeming-gihaeng](https://speakerdeck.com/icednut/scala-cats-effectro-saseo-gosaenghan-hamsuhyeong-peurogeuraeming-gihaeng)