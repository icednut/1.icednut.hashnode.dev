---
title: "liftIO 2022 발표하고 나서 Pierre Ricadat을 만났던 후기"
datePublished: Wed Dec 07 2022 15:00:00 GMT+0000 (Coordinated Universal Time)
cuid: clon2fy1n000j09l7hykv6twv
slug: liftio-2022-pierre-ricadat
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1699284981990/2f04ecc1-2a26-4369-a397-6ad1dfcff6b7.jpeg
tags: functional-programming, zio, 7zwo7iiy7zivio2uhouhnoq3uouemouwjq

---

# 이렇게 될 줄 알았을까?

liftIO 2022에서 발표한 게 이런 후폭풍을 몰고 올 줄 난 알았을까? 발표가 끝난 후 며칠이 지나자 라스칼라 코딩단에서 누군가 영어로 내 발표를 잘 봤다는 짤막한 인사와 ZIO, ZPure를 살펴보라고 조언을 해주는 게 눈에 띄었다. 처음엔 친절한 외국인이겠거니 생각했는데 이름을 자세히 살펴보니 Pierre Ricadat 이었다!! 너무 놀라서 되지도 않은 짤막한 영어 몇 마디 나눈 뒤 바로 만나자고 점심 약속을 잡았다.

가뜩이나 어려운 FP인데 이걸 영어로 질문하고 답을 들어야 해서 걱정이 앞섰다. 하지만 막상 점심 약속에 나가니 통역사(?)를 자처한 동료 한 분이 동행해서 그나마 걱정이 덜했다.

# Functional Lunch

내가 Cats Effect를 쓰고 있다고 하니 직접적으로 말하진 않았지만, 왠지 모를 (좋은 의미의) 동정심 같은 것을 보내고 있는 것이 느껴졌다. Typelevel 프로그래밍의 어려움을 잘 알고 있는 듯했고 ZIO, ZPure를 쓰면 굉장히 편하다는 것을 열심히 설명해주었는데 대부분 ZIO 공식 레퍼런스 사이트에 있던 내용을 말했다. 영상도 하나 추천해주던데 얼마 전에 자신이 직접 Scala Matsuri 라는 스칼라 컨퍼런스에서 발표한 영상이라고 한다. (일본에선 스칼라를 우리나라보단 많이 쓴다고 말해줬다)

[https://www.youtube.com/watch?v=TVYhFpqlgZ4](https://www.youtube.com/watch?v=TVYhFpqlgZ4)

그렇게 전날 밤 준비했던 질문들을 반찬 삼아 햄버거가 코로 들어가는지 입으로 들어가는지도 모르고 정신없는 점심을 먹었다. 대부분 Pierre가 데브시스터즈 기술 블로그에 썼던 글에 대한 질문이었는데 내 예상과 달랐던 부분이 많아서 놀랐다. 공개 가능한 범위의 코드를 보여주며 열정적으로 설명을 해주던데 대략 아래와 같은 얘기를 나눴다.

* Transaction 처리 시 컴파일 오류 원리에 관해 설명: Transaction 처리할 이펙트들에 대한 랩퍼 클래스와 그와 관련된 상속 구조를 이용하여 타입이 맞지 않는 랩퍼 클래스를 쓰는 경우 컴파일 오류를 발생하도록 구조를 잡음
    
* 전엔 제품에 Cats와 ZIO를 같이 썼었으나 최근 들어 ZPure 라는 친구를 활용하여 Reader, Writer를 사용하고 있다고 설명
    
* 그래도 Cats를 완전히 버리진 못하고 Traverse, Validated 같은 모나드는 여전히 쓰고 있다고 알려줌
    
* 이펙트 생성 DSL에서 &&& 연산자에 대한 구현은 알고 보니 zipPar를 사용하고 있었음
    
* Protobuf → Scala case class with ScalaPB 라는 코드 변환 과정에서 Refined가 적용된 타입으로 한 번 더 변환하는 과정에는 Chimney를 이용함 (Chimney가 자체적으로 제공하는 매크로를 이용했다고 함)
    
* Akka Cluster Shard로 고통받지 말고 Shardcake 써봐라. 짱좋다.
    
* Stream 처리에 대한 질문과 조언 (특정 상황에 FS2, ZStream을 써도 되는지 그런 경험이 있었는지 물어봄)
    

# 마치며

정신없이 질문과 답변을 듣느라 시계를 보니 1시간이 훌쩍 넘어가 있었다. 아쉬움을 뒤로 한 채 경험담과 FP를 어떻게 시작하게 되었는지 좀 더 얘기를 듣고 싶었는데 다음을 기약했다. 추후 스칼라 밋업 같은 것을 통해 다시 만나기로 약속하고 발걸음을 돌렸다. 사실 Pierre도 처음엔 Scala, Akka로 시작하여 Cats Effect도 경험하고 ZIO를 초기 버전부터 경험하여 ZIO 메인테이닝에 참여하게 되었다는데 그 열정이 나에게 큰 자극이 되었다. 쿠키런킹덤에 ZIO를 이용한 멋진 FP 코드 시스템이 있다는 게 부럽다기보단 나도 저렇게 해보고 싶다는 자극제가 된 것 같다.

## One More Thing

ZIO가 뭔지 관심이 있으면 살펴보라던 영상을 첨부하며 글을 마무리한다. ZIO 창시자가 발표한 영상이라고 한다. [https://vimeo.com/370819261](https://vimeo.com/370819261)