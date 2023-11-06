---
title: "Dev Dive 2022 함수형 개발자로 성장하기 Backend Day 참석 후기"
datePublished: Wed Nov 09 2022 15:00:00 GMT+0000 (Coordinated Universal Time)
cuid: clon2bny9000j09gs0uky0u3v
slug: dev-dive-2022-backend-day
tags: 7luo7y2865w7iqkio2bhoq4sa, 7zwo7iiy7zivio2uhouhnoq3uouemouwjq

---

![](https://velog.velcdn.com/images/icednut/post/e197e3d5-4ff1-4120-a6b2-39dbb7ff22d0/image.jpeg align="left")

# 그린랩스 CTO 발표

돌이켜 보면 첫 발표는 아래 두 아티클에 대한 요약과 함수형 프로그래밍에 대한 앞으로의 바람에 대해 얘기를 했던 것 같다.

* [Why Functional Programming Matters - John Hughes 1990](http://www.cse.chalmers.se/~rjmh/Papers/whyfp.pdf)
    
* [WHY Functional Programming Should Be the Future of Software Development](https://spectrum.ieee.org/functional-programming)
    

개발자가 다루는 문제에 대한 핵심은 문제를 분해하고 조립할 수 있는 프로그래밍 언어적 수단 제공이 필요하다는 것이다. 그런 수단을 제공하는 게 바로 함수형 프로그래밍이라고 말하고 있었다.

> In this paper we show that two features of functional languages in particular, higher-order functions and lazy evaluation, can contribute significantly to modularity. 함수형 언어의 두 기능, 고차 함수(higher-order function)와 지연 연산(lazy evaluation)이 모듈화 수준을 매우 높여준다는 점을 보여주려 한다.
> 
> * Why Functional Programmin Matters 논문 중에서 (번역 출처: [https://medium.com/@jooyunghan/왜-함수형-프로그래밍이-중요한가-john-hughes-1989-f6a1074a055b](https://medium.com/@jooyunghan/왜-함수형-프로그래밍이-중요한가-john-hughes-1989-f6a1074a055b))
>     

다행히 첫 번째 논문은 한글 번역본이 있는데 작은 함수들을 서로 합성하여 행렬의 요소들의 합이나 개수, 트리 등등 복잡한 문제를 풀 수 있다고 말하고 있다.

다시 돌아와서 발표에 나왔던 내용은 다음과 같다.

* 함수형 프로그래밍 언어는 왜 대세가 되지 못할까? 거꾸로 분석해보자.
    
    * FP는 킬러앱이 없어서: Haskell 했을 때 떠오르는 앱이 있나?
        
* 대세가 되기 위해서는 아래와 같은 요소를 가지고 있어야 한다고.
    
    * 독점적인 플랫폼: 스위프트는 애플, C#은 마소
        
    * 낮은 허들: 구글 허들이라는 언어는 C++ 언어 사용자를 흡수하기 위해 탄생, 자바는 자본과 마케팅이 투입된 언어
        
    * 꾸준함: 파이썬과 같은 경우
        
* ‘강타입 함수형 언어가 다음 패러다임이 될 것이다’ 라고 elm 창시자의 말을 인용.
    
* 여기서 강타입 함수형 언어란 ML(Meta Language)를 의미.
    
    * ML (1973)
        
    * SML (1983)
        
    * OCaml (1996)
        
    * F# (2005)
        
    * Elm (2012)
        
    * ReScript (2020)
        
* WHY Functional Programming Should be the Future of Software Development
    
    * FP는 적은 개발자로 많은 일을 할 수 있게 함: 대체로 사실이라고 한다. 현업에선 요구사항 분석과 문서 작성이 크기 때문이라고.
        
    * FP는 뛰어난 개발자 채용에 도움: ???
        
    * FP는 품질 향상과 디버그 시간이 줄어들 것이다: 이것도 대체로 사실이라고 한다. 채택만으론 좋아지진 않고 꾸준한 배움이 중요하다고 말함.
        
* FP의 미래
    
    * FP의 기능들이 주류언어에서 보이기 시작한다고 함.
        
    * 결국 FP의 방향으로 패러다임이 흐르고 있다고 얘기함.
        

> It’s hard to learn, but your code will produce fewer nasty surprises

# 하스켈로 백엔드 시스템 만든 이야기 (컨스택츠 김은민)

컨스택츠라는 소셜 미디어를 하스켈로 만들면서 개발환경과 코드 구성은 어떻게 되었었는지를 얘기했었다. DDD에 대해 어떻게 구현했는지 도메인 지식은 빼고 기법에 대한 구현체만 얘기 했었는데 시간 관계상 많은 부분이 생략된 것 같아 아쉬웠다.

\[발표 요약\]

* 하스켈 개발자는 어떻게 채용? 의외로 관심 있는 사람들이 있음
    
* 개발 환경은 어떻게?
    
    * Emacs에서 VS Code(with Haskell Plugin)로 갈아탐 → 지금도 쓰고 있다고 함
        
    * Github Actions
        
    * GCP
        
    * k8s
        
    * Kafka
        
    * GraphQL
        
    * PostgreSQL
        
* 하스켈 코드는 어떻게 구성했는지?
    
    * 타입시스템과 언어적 기능들의 장점을 최대한 살림
        
    * 언어는 익숙하지 않아도 대중적으로 익숙한 아키텍처를 채용하려 했다고 함
        
    * Type Level Programming: TextN 30과 같이 30글자가 넘어가면 컴파일 오류가 발생하도록 구성 했다고 함
        
    * 도메인 주도 개발이긴 한데 상황에 맞게 변형해서 개발한 느낌을 받음 (설명을 적긴 했는데 정리가 안되어서 생략)
        
* 테스트
    
    * 유닛 테스트: HSpec 패키지를 사용하며 QuickCheck는 사용 안함
        
    * 통합 테스트: 많이 작성하진 않지만 실제 서버를 실행한 뒤 테스트 DB를 마이그레이션 하여 참조하도록 구현
        
* 하스켈을 쓰면서 아쉬운 점은?
    
    * 빌드 속도가 오래 걸림. 캐시 없이 빌드하면 10분 정도 걸림
        
        * Github Action에는 캐싱이 되어 있고 변화가 없는 파일들은 빌드를 안하도록 구성
            
    * VS Code에서 리팩토링 기능이 빈약함
        
    * User 타입을 레코드로 만들었는데 필드에 Id를 만들면 엔티티의 ID와 충돌이 발생하기도 함 (이 부분은 하스켈을 몰라서 잘 이해야 안됨). 따라서 이런 경우는 팀내에 컨벤션을 만들어 두었다고 함.
        
* 하스켈을 쓰면서 좋은 점은?
    
    * 하스켈 코딩: 기능 코딩 → 컴파일 → 에러 → 코딩 → 컴파일 → 에러… → 완성
        
    * 만들어보고 실행해서 잘 되는지 확인하는 과정이 줄어듬
        
    * 컴파일 오류를 고치는 과정이 테스트를 줄여주고 개발자의 컨텍스트 스위칭이 줄어들어서 집중력이 높아짐
        
    * 타입이 강력하다 보니 대규모의 안전한 리팩토링이 가능
        
    * 해결해야 될 구조를 다양한 방법으로 해결할 수 있어서 선택지가 많아짐
        

발표 마지막 무렵에는 올해도 liftIO 2022 컨퍼런스를 진행한다고 발표하였다. +\_+

# read-eval-print-loop (그린랩스 이주빈)

JVM 기반의 또다른 함수형 프로그래밍 언어인 Clojure의 REPL 개발 방법을 라이브코딩으로 설명했던 세션이었다. 여기서의 REPL은 Python, Scala와 같이 작성한 코드가 쉽게 휘발되는 REPL CLI를 말하는 것은 아니고 작성한 코드가 즉시 실행되어 테스트 데이터로 코딩 하면서 실시간으로 검증하는 과정을 보여줬다.

IntelliJ에서 Java로 코딩할 때 Breakpoint 걸어두고 디버깅 하는 것과 비슷한 느낌을 많이 받았는데 디버깅 과정에서 변수 안에 있는 데이터에 대한 시각화가 좀 더 잘되어 있는 느낌을 받았다.

\[발표 요약\]

* REPL driven development 이걸로 높은 생산성을 가져올 수 있다고?
    
* 셸, 콘솔의 한계
    
    * python repl 위에선 잘 작업을 안함
        
    * 타이핑 하고 나면 휘발되는 코드 → 긴코드, 큰 규모의 코딩은 힘듬
        
    * 코드의 실험, 보조적으로만 사용
        
    * 클로저는 REPL 주도로 개발하는 것을 강조함
        
* 개발 workflow를 생각해보자. workflow는 총 3개로 구성되어 있고 각 단계마다 검증이 필요하다.
    
    * 입력
        
    * 데이터 처리
        
    * 출력
        
* REPL 기반 개발은 코드작성 → 실행 → 결과 확인 → 코드 작성 → 실행 … 이 과정을 빠르게 반복
    
    * 실행하고 싶은 만큼을 REPL에 보낸다.
        
    * 특히 Clojure는 괄호로 묶어서 그만큼만 보낼 수 있다.
        
    * 이렇게 REPL 주도 개발은 Interactive Programming, Tangible Programming 이라고 생각.
        
    * REPL 주도 개발은 점토를 계속 주물러 가면서 그 과정이 눈에 보이게 하는 것과 비슷
        
* 개발 &lt;&lt;&lt;&lt;&lt; 디버깅
    
    * 디버그를 할 땐 대부분 개발자는 print()를 사용한다.
        
    * print 를 하게 되면 불필요한 로그가 많이 찍힌다. 이걸 어떻게 해결할까? 아니 어디서부터 어디까지 로그를 찍어야 될까? 이를 찾기 위해 로그 수정 후 다시 빌드하고 확인하는 과정이 필요
        
    * 로그가 많이 찍히면 데이터 검색이 불편함
        
    * 디버그를 하게 되면 진짜 보고 싶은건? 이 라인에서 이 변수가 무슨 값인지 어떤 값인지를 보고 싶다.
        
    * 데이터 시각화의 문제
        
* 데이터 시각화
    
    * Clojure 기본 REPL은 다 좋은데 시각화 도구가 없다.
        
    * REPL에 잘 붙어서 시각화 하는 도구가 필요하다.
        
    * 데이터 시각화에 대한 도구 추천할텐데 이 도구는 뭐가 필요할까?
        
        * 가시성: 시각화할 수 있는 툴
            
        * 탐색성: 데이터 안을 잘 탐색할 수 있어야
            
    * 위 2가지 성질을 만족하는 모듈이 바로 Portal과 tap&gt;
        
    * tap&gt;
        
        * 예시: (tap&gt; x)
            
        * 등록한 탭으로 데이터를 보내주는 함수
            

# 마치며

그 외 ‘Scala와 ZIO로 쉽고 안전한 동시성 프로그래밍’과 Clojure 모듈화 프레임워크인 Polylith에 대한 소개가 있었는데 지면 관계상 생략한다. 기억에 남았던게 Scala 발표에서 타입 추론에 대해 언급이 있었는데 함수형 프로그래밍에서는 타입추론이 거의 필수 요소로 들어가는데 왜 그런 것일까라는 의문이 들었다.

힌들리 밀너 타입추론(Hindley-Milner Type Inference)에 대해 이름만 언급하고 넘어갔는데 누군가 정리를 잘해둔 포스팅이 있어서 그걸 공유하면서 글을 마친다.

[https://www.palindrom615.dev/hindley-milner-type-inference](https://www.palindrom615.dev/hindley-milner-type-inference)