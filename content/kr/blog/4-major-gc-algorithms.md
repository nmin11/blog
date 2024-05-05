---
author: "Loko"
title: "주요 GC 알고리즘들"
date: 2024-03-14
lastmod: 2024-03-14
description: "주요 가비지 컬렉션 알고리즘들에 대해 알아보자"
tags: ["java", "study"]
thumbnail: /thumbnail/garbage-collectors.webp
toc: true
---

> 『자바 최적화』 스터디를 하면서 주요 GC 알고리즘들에 대해 더 깊이 공부하고자 작성한 글입니다.

## Serial GC

<img src="https://github.com/nmin11/blog/assets/75058239/2a864abb-d0f9-4371-a7ea-e86db324af71" width=300>

- 가장 단순한 GC 구현체
- STW(Stop-The-World) 이후 한 개의 스레드에서만 GC 수행
- mark-sweep-compact 알고리즘 사용
- 멀티 스레드 환경에서는 부적합

❖ Serial GC 활성화 플래그

```bash
java -XX:+UseSerialGC
```

## Parallel GC

<img src="https://github.com/nmin11/blog/assets/75058239/fbf491b0-e50e-4c8d-91c1-2908a363b0d1" width=400>

- 자바 5 ~ 8까지 디폴트 GC
- GC 처리율을 최대화하기 때문에 *Throughput Collector* 라고도 불림
- 메모리 공간이 충분하고 코어 개수가 많을 때 유리한 알고리즘
- 플래그를 통해 GC에 사용할 최대 스레드 수, 최대 중단 시간, GC 수행 시간 비율을 설정할 수 있음
  - `-XX:ParallelGCThreads=<N>`
  - `-XX:MaxGCPauseMillis=<N>`
  - `-XX:GCTimeRatio=<N>`

❖ Parallel GC 활성화 플래그

```bash
java -XX:+UseParallelGC
```

## Parallel Old GC

- Parallel GC의 Old 영역 버전
- Old 영역까지 멀티 스레드로 GC 수행
- Old 영역에 대해 *Mark-Summary-Compact* 알고리즘 사용
  - Mark 단계: Old 영역을 리전별로 나누고, 리전별로 자주 참조되는 객체를 마킹
  - Summary 단계: 살아남은 객체들 중 밀도가 높은 부분이 어디인지 **dense prefix** 를 정하고, 이를 기준으로 compact 영역을 줄임
  - Compact 단계: compact 영역을 destination과 source로 나누고, 살아남은 객체를 destination으로 이동, 참조되지 않은 객체는 제거

❖ Parallel Old GC 활성화 플래그

```bash
java -XX:+UseParallelOldGC
```

## CMS GC

*Concurrent Mark Sweep*

![cms-gc](https://github.com/nmin11/TIL/assets/75058239/ae758bfe-404e-4b03-ba4a-f5a0cf2384f5)

❖ Concurrent가 붙으면 애플리케이션 작동 중 동시에 수행된다는 의미

- Initial Mark 단계: 클래스로더에서 가장 가까운 객체 중 살아있는 객체만 찾고 끝
- *Concurrent* Mark 단계: 살아있다고 확인한 객체들을 루트로 하여 참조되는 객체들을 마킹
- Remark 단계: Concurrent Mark 단계에서 새로 추가되거나 참조가 끊긴 객체들을 마킹
- *Concurrent* Sweep 단계: 마킹되지 않은 객체들을 제거

장점

- STW 시간이 매우 짧음
  - Low Latency GC 라고도 불림
  - 낮은 응답 속도가 중요할 때 사용됨

단점
- CPU와 메모리 사용량이 많음
- compaction 단계가 없어서 메모리 단편화가 발생할 수 있음
  - 조각난 메모리가 많기 때문에 compaction 작업을 하게 되면 STW 시간이 훨씬 더 길어짐

❖ CMS GC 활성화 플래그

```bash
java -XX:+UseConcMarkSweepGC
```

## G1 GC

*Garbage First*

![g1-gc](https://github.com/nmin11/TIL/assets/75058239/f3893306-45de-4b70-912b-12bd2e5e47ba)

- Java 9부터 디폴트 GC
- 지금까지의 GC와는 완전히 다른 방식
- 메모리 공간이 넉넉한 멀티 프로세서 환경에서 사용
- 힙 메모리를 동일 사이즈를 가진 **Region** 이라는 영역들로 나눠서 메모리를 관리
  - 각 리전은 논리적으로 구분됨 (Eden, Survivor, Old, Humongous, Available/Unused)
  - Humongous: 리전 크기의 50%를 초과하는 큰 객체를 저장하기 위한 영역
  - Available/Unused: 아직 사용되지 않은 영역

GC 수행 단계

- 글로벌 마킹 단계: 리전별 객체 활성도 판단
- 스윕 단계: 거의 비어있는 리전부터 우선적으로 가비지를 수집해서 상당한 여유 공간을 확보

### RSet (Remembered Set)

![rset](https://github.com/nmin11/TIL/assets/75058239/ce68836b-8673-46cc-b60e-cc18fe8829f5)

- 영역별로 하나씩 존재하는, 외부에서 힙 영역 내부를 참조하기 위한 레퍼런스 관리 장치
- floating garbage를 추적하기 위한 용도
  - floating garbage: 이미 죽은 객체가 참조하는 객체여서 살아있는 것처럼 보이는 객체

### Minor GC

- Eden 영역이 꽉 차면 수행
- Eden 영역의 살아남은 객체들을 Survivor 영역으로 이동
- 비어있게 된 Eden 영역을 Available/Unused 영역으로 지정

### Major GC

![g1-gc-process](https://github.com/nmin11/TIL/assets/75058239/a9af8b42-4f0b-4b38-bb7d-06e850f6cf2e)

- Initial Mark 단계: Old 리전의 객체들이 참조하는 Survivor 영역의 객체들을 마킹 (STW)
- Root Region Scan 단계: 위 단계에서 찾은 Survivor 객체들에 대한 스캔 작업
- Concurrent Mark 단계: 전체 힙에 대한 스캔 작업 실시 → 가비지가 없는 리전들은 다음 단계에서 제외
- Remark 단계: 최종적으로 GC 대상에서 제외할 객체 식별 (STW)
- Cleanup 단계: 살아있는 객체가 가장 적은 리전에 대한 미사용 객체 제거 작업 (STW)
- Copy 단계: Cleanup 후에도 완전히 비어지지 않은 살아있는 객체들을 새로운 리전에 옮김으로써 compaction 작업 수행 (STW)

❖ G1 GC 활성 플래그

```bash
java -XX:+UseG1GC
```

## Z GC

![zgc-concept-1](https://github.com/nmin11/TIL/assets/75058239/a26abb49-e049-4240-a634-29ec9fde7635)

- Java 11 때 실험 버전으로 등장, 이후 Java 15부터 정식 버전 출시
- **STW 시간을 10ms 이하로!**
  - 저지연을 요구하는 동시성 고비용 작업에 적합
- 8MB ~ 16TB 크기의 힙 처리 가능
- G1 GC와 마찬가지로 힙을 영역별로 나누어 관리하지만, 각 영역의 크기는 다름
  - region 대신 **ZPage** 라는 논리적 단위로 구분
  - small(2 MB), medium(32 MB), large(N * 2 MB) 3가지 타입을 가짐

### Colored pointers

![colored-pointers](https://github.com/nmin11/TIL/assets/75058239/7a620525-8a40-4499-a548-ca0b8f80bca4)

- 객체를 가리키는 변수의 포인터에서 64bit 메모리를 활용해서 객체 상태값을 저장
  - 그렇기 때문에 64bit 운영체제에서만 사용 가능!
- 42bit는 객체의 주소값으로 활용
- 4bit는 객체의 상태값을 표현
  - Finalizable: finalizer를 통해서만 접근 가능한 객체 (가비지)
  - Remapped: 객체의 재배치 여부를 표시
  - Marked 1 / 0: 접근 가능한 객체를 표시
- 가상 주소와 실주소 매핑 연산 절약을 위해 모든 가상 주소에 대해 mmap(multi-mapping) 실행
  - 이로 인해 메모리 사용량이 3배 증가

### Load barriers

![zgc-concept-7](https://github.com/nmin11/TIL/assets/75058239/e7d92b3d-e6db-4a4e-95b5-65031ae2c178)

- 객체를 참조할 때 실행되는 코드
- 힙에 있는 객체가 참조할 수 있는 상태인지 검사
  - 문제가 있는 경우 slow path라는 과정을 실행한 후 참조 진행
- 객체를 참조하기 전에 방어막으로 막아준다는 이미지로 연상하면 이해하기 쉽다!
- G1 GC와 다르게 메모리 재배치 과정에서 STW가 발생하지 않게 해줌
- Remap Mark와 Rellocation Set을 확인하면서 참조와 mark를 업데이트

GC 수행 단계

- *Pause* Mark Start 단계: ZGC의 루트에서 가리키는 객체 Mark 표시 (Live Object)
  - 각 스레드가 자신의 로컬 변수를 스캔해서 GC 루트 셋을 만듦 (매우 짧은 STW)
- Concurrent Mark/Remap 단계: 객체의 참조를 탐색하며 모든 객체에 Mark 표시
  - GC 루트에서 접근 가능한 객체에 대한 coloring 및 remapping 실행
- *Pause* Mark End 단계: 새로 들어온 객체들에 대한 Mark 표시
- Concurrent Pereare for Relocate 단계: 재배치할 영역을 찾아서 Relocation Set에 배치
  - Relocation Set: 가비지가 하나라도 있는 ZPage의 집합
- *Pause* Relocate Start 단계: 모든 루트 참조의 재배치를 진행하고 업데이트
- Concurrent Relocate 단계: Load barriers를 사용해서 모든 객체 재배치 후 참조 수정
  - 포워딩 테이블을 사용해서 객체의 주소를 업데이트

<br>

❖ Z GC의 압도적인 STW 시간

![zgc-performance](https://github.com/nmin11/TIL/assets/75058239/6ac87a0a-82fc-499c-962e-f3aa88f4c218)

<br>

❖ Z GC 활성화 플래그

```bash
java -XX:+UseZGC
```

⚠️ Java 15 이전 버전에서는 `-XX:+UnlockExperimentalVMOptions` 옵션을 추가해야 함

## References

- https://d2.naver.com/helloworld/1329
- https://www.baeldung.com/jvm-garbage-collectors
- https://velog.io/@guswlsapdlf/Java-GC-%EC%95%8C%EA%B3%A0%EB%A6%AC%EC%A6%98
- https://huisam.tistory.com/entry/jvmgc
- https://renuevo.github.io/java/garbage-collection/
- https://www.baeldung.com/jvm-zgc-garbage-collector
- https://d2.naver.com/helloworld/0128759
