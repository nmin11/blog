---
author: "Loko"
title: "Java 동시성 되짚어보기"
date: 2025-12-10
lastmod: 2025-12-10
description: "Java 생태계에서 20년 간 동시성을 어떻게 다뤄왔는가"
tags: ["java", "concurrency"]
thumbnail: /thumbnail/loom.webp
toc: true
---

## 0. 서론

부끄럽게도 이번 해에 백엔드 부트캠프 과정을 수료하면서 **"동시성"** 이라는 키워드를 처음 접해봤다.  
사실 부트캠프를 수료하는 과정에서 다양한 Lock 방식을 실습해보면서 동시성을 제어하는 방법은 충분히 숙지할 수 있었다.  
하지만 문제를 해결하는 방법에만 집중하기 보다는 "동시성"에 대해 본질적인 이해를 하고 넘어가야 앞으로도 백엔드 개발자로서의 전문적인 시야를 갖출 수 있겠다는 생각이 들었다.  
따라서 동시성 프로그래밍이라는 것이 무엇인지, 또 Java 진영에서 동시성을 다루는 방법들이 어떻게 발전해왔는지를 살펴보기로 했다.

## 1. The Free Lunch Is Over (2004)

2004년 12월, Microsoft의 개발자였던 Herb Sutter라는 분이 [The Free Lunch Is Over](http://www.gotw.ca/publications/concurrency-ddj.htm)라는 글을 발표했다.  
이 글은 아래의 강렬한 표어와 함께 시작된다.

> The biggest sea change in software development since the OO revolution is knocking at the door, and its name is **Concurrency**.

> 객체 지향 혁명 이후 가장 큰 변화의 물결이 소프트웨어 개발의 문을 두드리고 있다. 그것은 바로 **동시성**이다.

Herb Sutter는 1965년에 "반도체에 집적되는 트랜지스터의 수가 매년 2배씩 증가한다"라고 주장했던 *무어의 법칙*이 점차 한계에 봉착하고 있다고 설명한다.  
2001년 8월에 인텔 칩의 CPU 클럭 속도는 2GHz였는데, 이 글이 발표되었던 2004년 12월에도 4GHz의 속도를 갖춘 CPU가 등장하지 않았다고 말해준다.  
사실 4GHz의 클럭 속도는 머지않아 달성할 수 있어 보이지만, 과연 10GHz의 클럭 속도를 달성할 수 있을까?  
아쉽게도 제한된 반도체 용적 안에 과도하게 많은 트랜지스터를 집적하게 되면 발열, 전력 소모, 전류 누설의 문제가 발생하기 때문에 더 높은 클럭 속도를 달성하기가 점점 어려워진다.  
그렇기 때문에 심지어 2025년 현재에도 고성능 CPU가 낼 수 있는 최대 클럭 속도는 6GHz 정도의 수준에 머물러 있다.

이에 Herb Sutter는 개발자가 매년 컴퓨터 업그레이드 요청만 하면 자동으로 성능을 끌어올릴 수 있었던 **'공짜 점심'** 이 이미 1~2년 전에 끝났으며, 현대 애플리케이션의 기하급수적으로 높아지는 CPU 처리량에 대비하기 위해 동시성 프로그래밍이 점점 더 중요해질 것이라고 강조한다.  
또한 Java와 같은 프로그래밍 언어 차원에서도 동시성 프로그래밍 모델이 절실히 필요하다고 언급한다.

## 2. JSR-133/166 (2004)

이러한 흐름에 발 맞춰서 JCP(Java Community Process)는 2004년에 JSR(Java Specification Request) 133 및 166을 승인한다.  
위 2개 스펙을 통해 Java 언어는 동시성 프로그래밍을 수행할 수 있는 언어로 확고하게 자리 잡았다.  
어떠한 변경사항들이 있었는지 간결하게 살펴보자.

### JSR-133

사실 Java 언어는 1.0 때부터 JMM(Java Memory Model)을 통해 멀티스레드를 다룰 수 있었다.  
하지만 `final` 필드의 값을 변경할 수 있다거나, 동기화를 위한 재정렬을 허용하지 않는 등 심각한 결함들이 존재했다.

> **재정렬**: 컴파일러, JIT, 캐시 등 다양한 요인에 따라 연산의 순서는 코드 상에서의 흐름과 달라질 수 있는데, 동기화 작업을 수행하려면 컴파일러, 런타임, 하드웨어는 마치 직렬 실행이 되는 것처럼 공모(협력)해야 하며, 이 과정을 '재정렬'이라고 부름

그렇기 때문에 JSR-133은 JMM의 결함들을 개선하고, `volatile`, `synchronized`, `final` 키워드들이 직관적으로 작동하는 것을 핵심 목표로 삼고 진행되었다.  
개발자가 멀티스레드 프로그램이 어떻게 메모리와 상호작용하는지 자신 있게 추론할 수 있도록 도우며, 다양한 유명 아키텍처들을 정확성과 고성능을 두루 갖춘 채 구현할 수 있도록 제공하고자 했다.  
개략적인 개선 사항들은 다음과 같다.

- **Happens-Before**
  - 먼저 발생한 동작은 먼저 '정렬'되며, 이후 동작들에서 읽을 수 있도록 보장
  - 스레드의 각 작업은 해당 스레드의 다음 작업들보다 **먼저 발생**
  - 모니터의 잠금 해제는 해당 모니터의 후속 잠금들보다 **먼저 발생**
  - `volatile` 필드에 대한 쓰기는 해당 `volatile` 필드에 대한 후속 읽기보다 **먼저 발생**
  - 모든 메모리 작업은 모니터 잠금 해제 이전에 발생하며, 모니터 해제는 잠금 획득 이전에 발생
- `volatile`
  - 스레드 사이에서 상태 값을 주고받을 수 있는 특수 필드 역할 수행
  - 필드의 값은 메인 메모리로 flush되고, 모든 스레드에서 즉시 바라볼 수 있음
  - 즉, 프로그래머가 해당 필드를 캐싱, 재정렬 등으로 인해 "오래된" 값으로 표시되는 것을 허용하지 않겠다고 표시한 것
  - 기존에는 단순히 캐시 대신 메모리 직접 접근의 의미였는데, 추가적으로 Memory Barrier 역할도 수행하게 됨

### JSR-166

JSR-166에서는 스펙 리드였던 Doug Lea의 주도하에, 동시성을 다루는 패러다임을 **'언어 지원 방식'** 에서 **'라이브러리 지원 방식'** 으로 전환했다.  
JSR-133에서는 low-level의 언어 차원에서 메모리 모델을 제공했다면, JSR-166에서는 high-level에서 추상화 도구를 제공한 것이다.  
추상화 도구는 `java.util.concurrent` 패키지와 함께 제공되었으며, 핵심적인 구성요소는 다음과 같다.

- **Executor Framework**
  - `ExecutorService`, `ThreadPoolExecutor`
  - 스레드를 직접 다루지 않도록 **task**로 추상화
  - 스레드 풀 관리의 복잡함을 은닉화
- **Lock & Condition**
  - `ReentrantLock`, `ReadWriteLock`
  - `synchronized` 보다 유연한 잠금 기능 제공
  - 타임아웃, 인터럽트, 공정성 제어
- **Atomic Variables**
  - `AtomicInteger`, `AtomicReference`
  - CAS(Compare-And-Swap) 기반으로 작동
  - Lock-free 알고리즘을 보장하는 구성 요소
- **Concurrent Collections**
  - `ConcurrentHashMap`: 세그먼트 기반 락
  - `CopyOnWriteArrayList`: 읽기 최적화 방식
  - `BlockingQueue`: Producer-Consumer 패턴
- **Synchronizers**
  - `CountDownLatch`, `CyclicBarrier`, `Semaphore`
  - 스레드 간 협력 패턴의 추상화

JSR-166에서 전하는 메시지는 명확하다.  
바로 **"스레드를 직접 다루지 말고 고수준으로 추상화한 동시성 라이브러리를 활용하자"** 라는 것.

## 3. Akka / RxJava

Java 언어 차원에서 스레드 추상화 기법이 제공되었지만, 이 방식에는 고질적인 문제가 있다.  
바로 Lock 기반 프로그래밍의 특성상 Race Condition이 발생할 수밖에 없고, 그렇기 때문에 높은 차원의 동시성 환경에서 확장성의 한계가 존재한다는 점이다.  
이로 인해 2000년대 후반 JVM 생태계에서는 **"공유하지 않으면 동기화할 필요 없다"** 라는 원칙 기반의 패러다임들이 생겨났다.

### Akka (2009)

Akka는 Actor 모델을 기반으로 작동한다.  
1973년, Carl Hewitt가 MIT에서 Actor 모델 이론을 발표했고, 이것을 기반으로 1986년에 구현된 언어가 Erlang이다.  
그리고 이러한 Actor 모델의 철학을 Jonas Bonér가 Scala 및 JVM 프레임워크로 구현했는데, 이것이 바로 **Akka** 이다.

<img src="/blog/actor-graph.png">

Actor 모델은 Lock을 사용하지 않는 대신 캡슐화를 강제한다.  
신호에 반응하는 협력 엔티티들로 구성되어 있으며, 서로에게 신호를 보냄으로써 전체 애플리케이션이 작동한다.  
이는 실제로 세계가 소통하는 방식과도 유사한 동작 방식이라고 할 수 있다.  
Akka의 대략적인 개념들은 다음과 같다.

- 구성요소
  - Mailbox: 메시지 대기열
  - Behavior: Actor의 상태, 내부 변수 등을 담고 있음
  - Message: 신호(signal)의 데이터 조각, 매개변수 및 메서드 호출과 유사함
- 캡슐화
  - 마치 객체가 메서드 호출에 반응하듯, Actor는 메시지에 반응함
  - 수신자 Actor는 다른 Actor들과 상관없이 독립적으로 실행되며, 수신된 메시지들을 순차적으로 처리
  - 그러므로 Actor의 불변성은 동기화 없이도 유지되며 Lock이 필요 없음
  - 서로 다른 Actor들은 각자 받은 메시지들을 동시에 처리하게 됨
- 에러 처리
  - 일반적인 오류의 경우
    - 캡슐화된 Actor는 손상되지 않고 작업에만 오류 발생
    - 서비스 Actor가 에러 메시지를 회신
  - 서비스 내부 오류의 경우
    - 모든 Actor들은 트리 구조로 구성되어 있음
    - 만약 자식 Actor에 장애가 발생하면, 부모 Actor에서 장애 대응 방식을 결정
    - 부모 Actor가 종료될 때 자식 Actor들도 재귀적으로 종료됨
- 핵심 개념들
  - **캡슐화**: 각 Actor는 독립적인 상태를 가지며 외부에서 직접 접근 불가능
  - **메시지 패싱**: 메서드 호출 방식 대신에 비동기 메시지 전송 방식 활용
  - **순차 처리**: 각 Actor는 한번에 하나의 메시지만 처리

### RxJava (2013)

2009년, Microsoft에서 Rx.NET을 개발함으로써 Reactive Programming 패턴을 정립했다.  
이를 기반으로 2013년에 Netflix는 RxJava를 개발해서 ReactiveX 패턴을 JVM으로 이식했다.  
RxJava 코드의 형태는 다음과 같다.

```java
Observable<String> videos = getVideos()
    .flatMap(video -> getMetadata(video))
    .filter(metadata -> metadata.rating > 4.0)
    .map(metadata -> metadata.title)
    .timeout(1, TimeUnit.SECONDS);
```

RxJava를 활용하면 서비스에 대한 호출을 병렬로 실행하고 결과를 조합하는 방식을 쓸 수 있게 된다.  
RxJava에 대해서도 핵심적인 개념들을 정리해봤다.

- Pull 기반의 Iterable 대신 Push 기반의 Observable
  - 값들은 비동기적으로 push 됨
  - 이를 통해 전체 서비스 계층을 비동기화
- 각 서비스 계층이 할 수 있게 된 것들
  - 조건부 캐시 즉각 반환
  - 자원이 제한된 경우 스레드를 사용하지 않고 블로킹
  - 멀티 스레드 활용
  - non-blocking IO 사용
- API 호출에 따른 모든 동작들은 비동기적으로 동작하지만, 내부적으로 blocking일지 non-blocking일지 선택하는 방식
- 핵심
  - **Composability**: 복잡한 비동기 로직을 연산자들로 조합
  - **Backpressure**: 생산자-소비자 속도 불일치 처리
  - **Thread-safety**: 불변 데이터 스트림으로 공유 상태 회피

## 4. Reactive Manifesto (2013)

동시성 혁명이 시작된 이후로 위와 같이 다양한 접근 방식들이 난립하게 되었다.  
Akka를 창시했던 Jonas Bonér는 저마다 다른 해결책들 사이에서 공통점을 발견했고, 곧이어 동료들과 함께 원칙들을 정리하고 발표하기에 이른다.

[Reactive Manifesto](https://www.reactivemanifesto.org/ko)

Reactive 선언문은 4가지의 핵심 원칙을 담고 있다.

### Responsive (응답성)

- 시스템은 가능한 한 즉각적으로 일관된 응답을 해야 한다
- 일관된 서비스 품질 제공을 통해 시스템 신뢰성 확보
- 오류 처리를 단순화하고 사용자의 새로운 상호작용 촉진
- **예측 가능한 품질의 서비스**

### Resilient (탄력성)

- 시스템에 장애가 있어도 응답성은 유지되어야 한다
- **복제, 봉쇄, 격리, 위임**을 통해 달성할 것
- 장애는 전체 시스템 위험으로 이어지지 않아야 하며, 복구가 가능하도록 보장해야 한다
- 장애는 정상의 일부 ("Let it crash" 철학)

### Elastic (유연성)

- 시스템 작업 처리량이 변하더라도 응답성은 유지되어야 한다
- 입력에 따라 **할당된 자원을 유동적으로 변화**시키면서 대응하는 시스템
- 경합 지점이나 병목 지점이 발생하지 않도록 설계하는 것이 중요
- 실시간 성능 측정 도구를 통해 예측 가능한 규모로 확장하는 알고리즘 도입

### Message Driven (메시지 구동)

- **비동기 메시지 전달 방식**에 의존하자
- 컴포넌트 간 느슨한 결합, 격리, 위치 투명성을 보장하는 방식
- non-blocking 통신 방식에서는 수신자가 활성화되어 있을 때만 자원을 소비하게 됨
- 메시지 큐 모니터링과 함께 back-pressure 활용

## 5. Spring WebFlux (2017)

Spring Framework 5.0과 함께 Spring WebFlux가 출시되었다.  
이는 기존 Servlet 기반의 Spring Web MVC와는 차별점을 가지는, Reactive Stack Web Framework이며, 아래와 같은 특성들을 가진다.

- 완전한 non-blocking 프레임워크
- Reactive Streams Back Pressure 지원
- Netty 혹은 Servlet 컨테이너 환경에서 구동됨
- 경우에 따라 Spring Web MVC와 Spring WebFlux가 함께 사용될 수도 있음

Spring WebFlux는 non-blocking I/O 기반 비동기 처리와 함께 **빠른 응답성(Responsive)** 을 보장하고, Mono/Flux를 활용해 **비동기 데이터 스트림(Message Driven)** 을 활용하므로 Reactive 프로그래밍 프레임워크라고 볼 수 있다.  
이로써 Java 진영에서 동시성을 다루는 방식이 프레임워크 단계에서는 정립이 되었다고 보인다.

## 6. Project Loom (2023)

Java 21의 Project Loom이 해결하고자 했던 문제점들은 다음과 같다.

- 기존 Java 플랫폼 스레드의 문제점
  - Java 플랫폼 스레드는 OS 스레드와 1:1 매핑이었음
  - **Memory overhead**: 각 플랫폼 스레드는 ~2MB의 스택 공간 활용
  - **Creation cost**: 스레드 생성 작업은 kernel 호출 작업 (스레드 당 100–300μs)
  - **Kernel limits**: 대부분의 시스템은 주소 공간 제약으로 인해 32K~65K 스레드로 제한됨
- Reactive frameworks의 문제점
  - **Cognitive overhead**: 개발팀이 reactive 패턴을 익히는 데에 6~12개월 소요
  - **Debugging complexity**: reactive chains 속에서 스택 추적을 이해하기 어려움
  - **Library ecosystem**: 수많은 Java 라이브러리들이 reactive 시스템에는 적합하지 않음
  - **Backpressure complexity**: 수동적인 pressure 관리 작업으로 인해 버그 발생 가능

실제로 Netflix 개발자는 아래와 같은 말을 남겼다고 한다.

> We spent more time debugging reactive chains than we saved in scalability.

> 우리는 확장성 측면에서 절약한 시간보다 더 많은 시간을 반응형 체인 디버깅에 썼다.

Project Loom은 이러한 문제점들을 제거하면서 혁신적인 기능들을 제공해준다.

- **Virtual Threads**: JVM이 관리하며 자동으로 해제되는 경량 스레드
- **Structured Concurrency**: 자동으로 정리 작업이 이루어지는 계층적 태스크 범위
- **Scoped Values**: ThreadLocal 패턴을 대체하는 불변 컨텍스트 전파 기능

Project Loom은 동시성 관리를 기존의 OS 커널 중심에서 JVM 중심으로 옮겼고, 이를 통해 스케줄링, 메모리 관리, 관측을 개발자가 완벽하게 제어할 수 있게 해주었다.  
Reactive 방식에 대한 러닝 커브를 줄일 수 있는 대안책을 마련해주었으며, Debuggability도 확보할 수 있도록 해주었다.

## 7. 결론

**The Free Lunch Is Over**에서는 동시성과 관련된 문제를 인식했고, **JSR-133/166**은 언어 차원에서의 해결책을 제시했다.  
**Akka/RxJava**는 Java 진영에서 동시성을 다루기 위한 실험적인 접근을 했으며, **Reactive Manifesto**를 통해 마침내 Reactive 철학이 정립되었다.  
**Spring WebFlux**는 Reactive를 프레임워크에 녹여내었는데, Java는 **Project Loom**을 통해 동시성 관련 문제를 근본적으로 재설계했다.  
The Free Lunch Is Over 발표 후 20여 년이 흘렀지만 "동시성 혁명"은 아직도 끝나지 않은 듯 하다.  
하지만 이번 기회에 동시성을 다루는 패러다임이 변화되는 과정을 지켜보면서 어떤 핵심 가치들이 발생했는지 알 수 있게 되었으며, 앞으로도 동시성 프로그래밍을 다룰 때 어떤 방식으로 접근하면 될지에 대한 시야와 관점을 얻어갈 수 있었다.

## References

- [The Free Lunch Is Over](http://www.gotw.ca/publications/concurrency-ddj.htm)
- [JSR 133 (Java Memory Model) FAQ](https://www.cs.umd.edu/~pugh/java/memoryModel/jsr-133-faq.html)
- [How the Actor Model Meets the Needs of Modern, Distributed Systems](https://doc.akka.io/libraries/akka-core/current/typed/guide/actors-intro.html)
- [Reactive Programming in the Netflix API with RxJava](https://netflixtechblog.com/reactive-programming-in-the-netflix-api-with-rxjava-7811c3a1496a)
- [Reactive Manifesto](https://www.reactivemanifesto.org/ko)
- [Spring WebFlux](https://docs.spring.io/spring-framework/reference/web/webflux.html)
- [Project Loom & Virtual Threads: How Java 21+ Is Revolutionizing Concurrency](https://medium.com/@dikhyantkrishnadalai/project-loom-virtual-threads-how-java-21-is-revolutionizing-concurrency-582a173b2b12)
