---
author: "Loko"
title: "Java 동시성의 변천사"
date: 2025-07-01
lastmod: 2025-07-01
description: "JMM부터 Project Loom까지, Java 동시성은 어떻게 변해 왔는가"
tags: ["java", "concurrency"]
thumbnail: /thumbnail/loom.webp
toc: true
---

## 0. 서론

최근에 항해 플러스 백엔드 8기 과정을 진행하면서 많은 지식들을 흡수하고 있다.  
특히 동일한 자원에 여러 스레드가 동시에 접근할 때 생기는 **동시성 이슈**는, 부끄럽게도 여태 생각조차 해보지 못한 중요한 학습 포인트였다.  
사실 부트캠프 과정 중에 이미 DB 락과 분산 락을 적용해보면서, 동시성을 제어하는 방법 자체는 충분히 깨달을 수 있었다.  
하지만 문제를 해결하는 방법 자체에만 집중하기 보다는 조금 더 깊이 이해해보는 것이 좋겠다는 생각이 들었다.  
**동시성 프로그래밍**이라는 것이 무엇인지, 또 Java 진영에서 동시성을 제어하기 위한 방법들이 어떻게 발전해왔는지 살펴보면 더 나은 학습 경험이 될 것이다.

<img class="hover-zoom" src="/blog/spring-history.webp" alt="김영한 님의 스프링 역사 강의">

김영한 님의 스프링 기본 강의에서 스프링의 역사부터 살펴보는 부분이 인상깊었다.  
스프링에 어떤 대단한 점들이 있는지, 스프링을 어떻게 활용하는지 등을 바로 알려주지 않았다.  
대신에 이 기술에 어떤 배경이 있었고, 어떻게 시작되었는지를 설명해주니 확실히 기술을 활용해야 하는 "이유"를 먼저 깨닫게 되는 느낌어있다.  
나 또한 실제 면접에서 *"스프링을 왜 쓰나요?"* 라는 질문에 제대로 대답하지 못한 기억이 있다.

그렇기 때문에 더더욱, 동시성을 제어하는 기술들에만 초점을 맞추지 않고자 한다.  
그보다는 **동시성 프로그래밍**이라는 것이 어떻게 시작되었고, 어떠한 흐름으로 발전되어 왔는지를 한번 짚고 넘어가고자 한다.

## 1. The Free Lunch Is Over

2004년 12월, Microsoft 개발자 Herb Sutter가 [The Free Lunch Is Over](http://www.gotw.ca/publications/concurrency-ddj.htm)라는 글을 발표했다.

> The biggest sea change in software development since the OO revolution is knocking at the door, and its name is **Concurrency**.

그는 객체 지향 혁명 이후 가장 큰 변화가 다가오고 있으며, 그 이름은 **동시성**이라고 선언하며 글을 시작한다.  
이 글은 '무어의 법칙'이 점차 한계에 봉착하고 있으며, 앞으로는 동시성 프로그래밍이 필수가 될 것이라고 강조한다.  
'무어의 법칙'은 반도체 칩에 집적할 수 있는 트랜지스터 수가 18개월마다 2배로 늘어난다는 예측이었다.  
하지만 Sutter는 발열, 전력 소비량 등의 물리적 한계로 인해 CPU의 클럭 속도를 **4GHz** 이상으로 끌어올리는 데 제약이 생겼고, 이에 따라 이제는 개발자가 병렬성을 직접 고려해야 할 시점이 도래했다고 말한다.  
이제 매년 자동으로 성능이 올라가던 '공짜 점심'은 끝났으며, 객체 지향 혁명과 맞먹는 규모의 패러다임의 전환이 다가오고 있다고 경고한다.

## 2. JSR 133: JMM

📒 용어 정리

- **JSR (Java Specification Request)**: Java 플랫폼의 확장 및 개선을 위한 공식 명세 제안서
- **JMM (Java Memory Model)**: Java 프로그램에서 스레드 간 메모리 접근 및 동기화 방식을 정의한 메모리 모델
  - `synchronized`, `volatile`, `final` 키워드의 동작 방식과 연관됨

2004년 9월, **Java 5**의 출시와 함께 JMM이 대대적으로 개정되었다.  
멀티코어 하드웨어가 보편화되는 흐름에 발맞춰, 스레드 간 안전하고 예측 가능한 메모리 동기화 모델을 정립하기 위한 시도였다.

JMM은 다음과 같은 핵심 개념을 바탕으로 동시성을 제어한다:

- **Happens-Before**
  - 한 작업이 다른 작업보다 먼저 실행됨을 보장
  - 스레드 간 **데이터 정합성**을 보장하기 위한 장치
- **Synchronizes-With**
  - 공유 메모리가 여러 스레드에서 일관되게 보이도록 보장
- **As-If-Serial**
  - 실행 스레드 밖에서는 명령어가 순차 실행되는 것처럼 보여야 함
- **Release-Before-Acquire**
  - 락을 사용하는 경우, 한 스레드가 락을 해제해야 다른 스레드가 해당 락을 획득할 수 있음

JMM 중에서도 특히 많이 사용되는 `synchronized` 키워드는 '락을 획득한 스레드가 메인 메모리와 **동기화(Synchronizes-With)** 되었다'는 명시적 의미를 가진다.  
Java 진영에서 더 정교한 동시성 도구들이 등장하기 전까지 `synchronized`는 멀티스레드 환경에서 순서 보장과 가시성을 확보할 수 있는 사실상 유일한 장치였다.

하지만 시간이 흐르면서 JMM에 대해 다음과 같은 **구조적 한계**들이 지적되기 시작한다:

- 락이 걸린 데이터에 대한 모든 연산은 동등하게 취급되므로, **갱신 손실(lost update)** 이 발생할 수 있음
- 락 획득/해제는 반드시 **메서드 수준** 또는 **동기화 블록 수준**에서만 사용 가능
- 락을 얻지 못한 스레드는 **블로킹**되며, 락을 얻기 위한 재시도를 하는 것조차 불가능

## 3. `java.util.concurrent`

JMM의 결점을 보완하기 위해, Java 5 버전은 **JSR 166: Concurrency Utilities**와 함께 고급 동시성 라이브러리를 제공했다.  
`java.util.concurrent` 패키지는 멀티스레드 애플리케이션을 보다 더 쉽게 개발할 수 있도록 세심하게 설계된 라이브러리이다.

이 라이브러리를 구성하는 핵심 요소들은 다음과 같다:

| 범주               | 구현 클래스                                                         | 주요 기능 |
|:-------------------:|----------------------------------------------------------------------|----------------|
| **Lock**           | `ReentrantLock`, `ReadWriteLock`, `StampedLock`                     | 세밀한 락 제어 (공정성, 재시도, 타임아웃 등) |
| **Semaphore**      | `Semaphore`                                                         | 리소스 접근 수 제한 |
| **Atomics**        | `AtomicInteger`, `AtomicLong`, `AtomicReference`                    | CAS 기반의 락-프리 원자 연산 |
| **Blocking Queue** | `LinkedBlockingQueue`, `ArrayBlockingQueue`, `PriorityBlockingQueue` | 생산자–소비자 패턴에 적합한 스레드 안전 큐 |
| **Latch**          | `CountDownLatch`                                                    | 여러 스레드가 하나의 완료 신호를 기다릴 때 사용 |
| **Barrier**        | `CyclicBarrier`, `Phaser`                                           | 모든 스레드가 도착해야 다음 단계로 진행하는 구조 |
| **Executor**       | `ExecutorService`, `ScheduledExecutorService`, `ForkJoinPool`       | 스레드 풀 기반의 작업 단위 실행 관리 |
| **Future**         | `Future`, `Callable`, `CompletableFuture`                           | 비동기 작업 결과 처리 및 조합 |
| **Concurrent Collection** | `ConcurrentHashMap`, `CopyOnWriteArrayList`, `ConcurrentLinkedQueue` | 락-프리 또는 락 분할 기반의 병렬 컬렉션 |

이 라이브러리의 등장과 함께, Java 개발자들은 저수준의 스레드 관리 없이도 안정적이고 생산적인 동시성 프로그래밍을 구현할 수 있게 되었다.  
특히 **Lock 기반 동기화**와 **CAS 기반의 Lock-free 방식**이라는 두 가지 주요 축을 정립하며, Java 동시성의 실질적 실무 기반을 완성했다고 볼 수 있다.

## 4. Akka

2009년 7월, Jonas Bonér는 **Actor** 기반 프레임워크인 Akka를 발표했다.  
Akka는 Scala 기반으로 만들어졌지만, Java API도 제공하고 있다.  
Akka의 Actor 기반 시스템은 복잡한 락 관리 없이도 안전한 병렬 처리를 가능하게 해주는 **메시지 중심의 동시성 프로그래밍** 모델을 제공한다.  

<img class="hover-zoom" src="/blog/actor.webp" alt="Actor 시스템 예시">

Actor 시스템은 다음과 같은 특성을 가진다:

- 각 Actor는 고유한 상태와 로직을 내부에 **캡슐화**한, 작고 독립적인 실행 단위
- 상태는 절대 공유되지 않으며, 오직 **불변 메시지**를 통해서만 다른 Actor와 상호 통신
- 메시지 전달은 **비동기적**으로 이루어지며, Actor는 수신한 메시지에 반응하여 동작

[Akka 공식 문서](https://doc.akka.io/libraries/akka-core/2.10.5/typed/guide/actors-motivation.html?language=java)는 전통적인 락 기반 동시성 제어 대신 Actor 모델을 선택해야 하는 이유를 다음과 같이 설명한다:

- 도메인 모델 내부의 **가변 상태를 안전하게 캡슐화하는 것**은 복잡하고 어려움
- 락을 사용한 상태 보호는 **처리량(throughput)** 을 크게 저하시킬 수 있음
- 락 기반 구조는 **데드락, 우선순위 역전, 병목 등의 문제**를 유발할 가능성이 높다.

Akka의 철학은 "락을 피하고, 메시지를 통해 상태를 안전하게 캡슐화하라"고 강조한다.
