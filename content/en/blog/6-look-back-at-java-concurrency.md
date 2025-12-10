---
author: "Loko"
title: "Looking Back at Java Concurrency"
date: 2025-12-10
lastmod: 2025-12-10
description: "How the Java ecosystem has handled concurrency over the past 20 years"
tags: ["java", "concurrency"]
thumbnail: /thumbnail/loom.webp
toc: true
---

## 0. Introduction

Embarrassingly, I first encountered the keyword **"concurrency"** this year while completing a backend bootcamp.
To be honest, during the bootcamp, I was able to sufficiently learn how to control concurrency by practicing various lock methods.
However, rather than just focusing on solving problems, I felt that gaining a fundamental understanding of "concurrency" would help me develop a professional perspective as a backend developer going forward.
Therefore, I decided to explore what concurrent programming is and how methods of handling concurrency in the Java ecosystem have evolved.

## 1. The Free Lunch Is Over (2004)

In December 2004, Herb Sutter, a developer at Microsoft, published an article titled [The Free Lunch Is Over](http://www.gotw.ca/publications/concurrency-ddj.htm).
The article begins with this powerful statement:

> The biggest sea change in software development since the OO revolution is knocking at the door, and its name is **Concurrency**.

Herb Sutter explains that _Moore's Law_, which claimed in 1965 that "the number of transistors integrated into semiconductors doubles every year," is gradually reaching its limits.
In August 2001, Intel chips had a CPU clock speed of 2GHz, but by December 2004 when this article was published, CPUs with 4GHz speeds had not yet appeared.
While 4GHz clock speeds seemed achievable in the near future, could we really achieve 10GHz?
Unfortunately, when too many transistors are integrated into a limited semiconductor volume, problems arise with heat generation, power consumption, and current leakage, making it increasingly difficult to achieve higher clock speeds.
That's why even in 2025, the maximum clock speed that high-performance CPUs can achieve remains around 6GHz.

Herb Sutter emphasizes that the **"free lunch"** where developers could automatically improve performance just by requesting computer upgrades every year ended 1-2 years ago, and concurrent programming will become increasingly important to handle the exponentially growing CPU throughput demands of modern applications.
He also mentions that programming languages like Java desperately need concurrency programming models at the language level.

## 2. JSR-133/166 (2004)

In line with this trend, the JCP (Java Community Process) approved JSR (Java Specification Request) 133 and 166 in 2004.
Through these two specifications, the Java language firmly established itself as a language capable of concurrent programming.
Let's briefly look at what changes were made.

### JSR-133

In fact, the Java language has been able to handle multithreading through the JMM (Java Memory Model) since version 1.0.
However, serious flaws existed, such as being able to change the value of `final` fields or not allowing reordering for synchronization.

> **Reordering**: The order of operations can differ from the flow in code due to various factors such as compilers, JIT, caches, etc. To perform synchronization, the compiler, runtime, and hardware must conspire (cooperate) as if it were serial execution, and this process is called 'reordering'

Therefore, JSR-133 aimed to improve the flaws in JMM and ensure that `volatile`, `synchronized`, and `final` keywords work intuitively.
It sought to help developers confidently reason about how multithreaded programs interact with memory and provide implementation across various well-known architectures with both correctness and high performance.
The main improvements are as follows:

- **Happens-Before**
  - Actions that occur first are 'ordered' first and guaranteed to be readable by subsequent actions
  - Each action in a thread **happens-before** subsequent actions in that thread
  - Unlocking a monitor **happens-before** subsequent locks of that monitor
  - A write to a `volatile` field **happens-before** subsequent reads of that `volatile` field
  - All memory operations occur before monitor unlock, and release occurs before lock acquisition
- `volatile`
  - Acts as a special field that can exchange state values between threads
  - Field values are flushed to main memory and immediately visible to all threads
  - In other words, programmers indicate they will not allow the field to be marked as "stale" due to caching, reordering, etc.
  - Previously, it simply meant direct memory access instead of cache, but now also serves as a Memory Barrier

### JSR-166

JSR-166, led by spec lead Doug Lea, shifted the concurrency paradigm from **'language support'** to **'library support'**.
While JSR-133 provided a memory model at the low-level language layer, JSR-166 provided abstraction tools at a high level.
Abstraction tools were provided with the `java.util.concurrent` package, and the core components are as follows:

- **Executor Framework**
  - `ExecutorService`, `ThreadPoolExecutor`
  - Abstracts to **tasks** to avoid handling threads directly
  - Hides the complexity of thread pool management
- **Lock & Condition**
  - `ReentrantLock`, `ReadWriteLock`
  - Provides more flexible locking than `synchronized`
  - Timeout, interrupt, fairness control
- **Atomic Variables**
  - `AtomicInteger`, `AtomicReference`
  - Operates based on CAS (Compare-And-Swap)
  - Components that guarantee lock-free algorithms
- **Concurrent Collections**
  - `ConcurrentHashMap`: Segment-based locking
  - `CopyOnWriteArrayList`: Read-optimized approach
  - `BlockingQueue`: Producer-Consumer pattern
- **Synchronizers**
  - `CountDownLatch`, `CyclicBarrier`, `Semaphore`
  - Abstraction of thread cooperation patterns

The message from JSR-166 is clear:
**"Don't handle threads directly; use high-level abstracted concurrency libraries."**

## 3. Akka / RxJava

Although thread abstraction techniques were provided at the Java language level, there is a chronic problem with this approach.
Specifically, due to the nature of lock-based programming, race conditions inevitably occur, and therefore scalability limits exist in high-concurrency environments.
As a result, paradigms based on the principle **"if you don't share, you don't need to synchronize"** emerged in the JVM ecosystem in the late 2000s.

### Akka (2009)

Akka operates based on the Actor model.
In 1973, Carl Hewitt published the Actor model theory at MIT, and based on this, Erlang was implemented in 1986.
Jonas Bonér implemented this Actor model philosophy as a Scala and JVM framework, which is **Akka**.

<img src="/blog/actor-graph.png">

The Actor model does not use locks but instead enforces encapsulation.
It consists of cooperating entities that react to signals, and the entire application operates by sending signals to each other.
This is similar to how the world actually communicates.
The key concepts of Akka are as follows:

- Components
  - Mailbox: Message queue
  - Behavior: Contains the Actor's state, internal variables, etc.
  - Message: Data piece of a signal, similar to parameters and method calls
- Encapsulation
  - Just as objects react to method calls, Actors react to messages
  - Receiving Actors execute independently of other Actors and process received messages sequentially
  - Therefore, Actor immutability is maintained without synchronization and locks are not needed
  - Different Actors process their received messages concurrently
- Error Handling
  - For general errors
    - The encapsulated Actor is not damaged, only the task has an error
    - The service Actor replies with an error message
  - For internal service errors
    - All Actors are organized in a tree structure
    - If a child Actor fails, the parent Actor decides how to handle the failure
    - When the parent Actor terminates, child Actors also terminate recursively
- Core Concepts
  - **Encapsulation**: Each Actor has independent state and is not directly accessible from outside
  - **Message Passing**: Uses asynchronous message sending instead of method calls
  - **Sequential Processing**: Each Actor processes only one message at a time

### RxJava (2013)

In 2009, Microsoft developed Rx.NET, establishing the Reactive Programming pattern.
Based on this, Netflix developed RxJava in 2013, porting the ReactiveX pattern to the JVM.
RxJava code looks like this:

```java
Observable<String> videos = getVideos()
    .flatMap(video -> getMetadata(video))
    .filter(metadata -> metadata.rating > 4.0)
    .map(metadata -> metadata.title)
    .timeout(1, TimeUnit.SECONDS);
```

Using RxJava allows executing calls to services in parallel and composing the results.
I've also summarized the core concepts of RxJava:

- Push-based Observable instead of pull-based Iterable
  - Values are pushed asynchronously
  - This makes the entire service layer asynchronous
- What each service layer can do
  - Return from conditional cache immediately
  - Block without using threads when resources are limited
  - Utilize multiple threads
  - Use non-blocking IO
- All operations according to API calls operate asynchronously, but internally choose whether to be blocking or non-blocking
- Core
  - **Composability**: Compose complex asynchronous logic with operators
  - **Backpressure**: Handle producer-consumer speed mismatch
  - **Thread-safety**: Avoid shared state with immutable data streams

## 4. Reactive Manifesto (2013)

Since the concurrency revolution began, various approaches proliferated as described above.
Jonas Bonér, who founded Akka, discovered commonalities among different solutions and soon organized and published principles with colleagues.

[Reactive Manifesto](https://www.reactivemanifesto.org/)

The Reactive Manifesto contains four core principles.

### Responsive

- Systems should respond as immediately and consistently as possible
- Ensure system reliability through consistent service quality
- Simplify error handling and promote new user interactions
- **Predictable quality of service**

### Resilient

- Responsiveness must be maintained even when the system fails
- Achieve through **replication, containment, isolation, delegation**
- Failures should not lead to overall system risk and must ensure recovery is possible
- Failure is part of normal operation ("Let it crash" philosophy)

### Elastic

- Responsiveness must be maintained even when system workload changes
- A system that responds by **fluidly changing allocated resources** according to input
- It's important to design to avoid contention points or bottlenecks
- Introduce algorithms that scale to predictable sizes through real-time performance measurement tools

### Message Driven

- Rely on **asynchronous message passing**
- A method that ensures loose coupling, isolation, and location transparency between components
- In non-blocking communication, receivers only consume resources when active
- Use back-pressure along with message queue monitoring

## 5. Spring WebFlux (2017)

Spring WebFlux was released with Spring Framework 5.0.
This is a Reactive Stack Web Framework that differs from the existing Servlet-based Spring Web MVC and has the following characteristics:

- Fully non-blocking framework
- Supports Reactive Streams Back Pressure
- Runs on Netty or Servlet container environments
- In some cases, Spring Web MVC and Spring WebFlux may be used together

Spring WebFlux ensures **fast responsiveness** with non-blocking I/O-based asynchronous processing and utilizes **asynchronous data streams (Message Driven)** using Mono/Flux, so it can be considered a Reactive programming framework.
This seems to have established how concurrency is handled in the Java ecosystem at the framework level.

## 6. Project Loom (2023)

The problems that Java 21's Project Loom aimed to solve are as follows:

- Problems with existing Java platform threads
  - Java platform threads had a 1:1 mapping with OS threads
  - **Memory overhead**: Each platform thread uses ~2MB of stack space
  - **Creation cost**: Thread creation is a kernel call operation (100-300μs per thread)
  - **Kernel limits**: Most systems are limited to 32K-65K threads due to address space constraints
- Problems with Reactive frameworks
  - **Cognitive overhead**: It takes 6-12 months for development teams to learn reactive patterns
  - **Debugging complexity**: Stack traces are difficult to understand within reactive chains
  - **Library ecosystem**: Many Java libraries are not suitable for reactive systems
  - **Backpressure complexity**: Bugs can occur due to manual pressure management

In fact, a Netflix developer reportedly said:

> We spent more time debugging reactive chains than we saved in scalability.

Project Loom provides innovative features while removing these problems:

- **Virtual Threads**: Lightweight threads managed and automatically released by the JVM
- **Structured Concurrency**: Hierarchical task scopes with automatic cleanup
- **Scoped Values**: Immutable context propagation that replaces the ThreadLocal pattern

Project Loom moved concurrency management from the OS kernel center to the JVM center, allowing developers to have complete control over scheduling, memory management, and observation.
It provided an alternative that can reduce the learning curve for the Reactive approach and ensured debuggability.

## 7. Conclusion

**The Free Lunch Is Over** recognized concurrency-related problems, and **JSR-133/166** provided language-level solutions.
**Akka/RxJava** took experimental approaches to handling concurrency in the Java ecosystem, and the Reactive philosophy was finally established through the **Reactive Manifesto**.
**Spring WebFlux** integrated Reactive into the framework, and Java fundamentally redesigned concurrency-related problems through **Project Loom**.
Although 20 years have passed since the publication of The Free Lunch Is Over, the "concurrency revolution" still seems ongoing.
However, this time I was able to learn what core values emerged by observing the process of changing paradigms for handling concurrency, and I gained perspective and insight on how to approach concurrent programming in the future.

## References

- [The Free Lunch Is Over](http://www.gotw.ca/publications/concurrency-ddj.htm)
- [JSR 133 (Java Memory Model) FAQ](https://www.cs.umd.edu/~pugh/java/memoryModel/jsr-133-faq.html)
- [How the Actor Model Meets the Needs of Modern, Distributed Systems](https://doc.akka.io/libraries/akka-core/current/typed/guide/actors-intro.html)
- [Reactive Programming in the Netflix API with RxJava](https://netflixtechblog.com/reactive-programming-in-the-netflix-api-with-rxjava-7811c3a1496a)
- [Reactive Manifesto](https://www.reactivemanifesto.org)
- [Spring WebFlux](https://docs.spring.io/spring-framework/reference/web/webflux.html)
- [Project Loom & Virtual Threads: How Java 21+ Is Revolutionizing Concurrency](https://medium.com/@dikhyantkrishnadalai/project-loom-virtual-threads-how-java-21-is-revolutionizing-concurrency-582a173b2b12)
