---
author: "Loko"
title: "Major GC Algorithms"
date: 2024-03-14
lastmod: 2024-03-14
description: "Exploring Major Garbage Collection Algorithms"
tags: ["java", "study"]
thumbnail: /thumbnail/garbage-collectors.webp
toc: true
---

> This article was written with the intention of studying major GC algorithms more deeply while participating in the 'Optimizing Java' study.

## Serial GC

<img src="https://github.com/nmin11/blog/assets/75058239/2a864abb-d0f9-4371-a7ea-e86db324af71" width=300>

- The simplest GC implementation
- Only one thread performs GC after Stop-The-World (STW)
- Uses the mark-sweep-compact algorithm
- Not suitable for multi-threaded environments

❖ Enable Serial GC flag

```bash
java -XX:+UseSerialGC
```

## Parallel GC

<img src="https://github.com/nmin11/blog/assets/75058239/fbf491b0-e50e-4c8d-91c1-2908a363b0d1" width=400>

- Default GC from Java 5 to 8
- Also known as the *Throughput Collector* as it maximizes GC throughput
- Advantageous algorithm when there is sufficient memory space and a high number of cores
- Flags can be used to configure parameters such as the maximum number of threads, maximum pause time, and GC execution time ratio
  - `-XX:ParallelGCThreads=<N>`
  - `-XX:MaxGCPauseMillis=<N>`
  - `-XX:GCTimeRatio=<N>`

❖ Enable Parallel GC flag

```bash
java -XX:+UseParallelGC
```

## Parallel Old GC

- Old generation version of Parallel GC
- Performs GC in multiple threads up to the Old generation
- Uses the *Mark-Summary-Compact* algorithm for the Old generation
  - Mark phase: Divides the Old generation into regions and marks frequently referenced objects in each region
  - Summary phase: Determines the **dense prefix** among surviving objects, based on which the compact area is reduced
  - Compact phase: Divides the compact area into destination and source, moves surviving objects to the destination, and removes unreferenced objects

❖ Enable Parallel Old GC flag

```bash
java -XX:+UseParallelOldGC
```

## CMS GC

*Concurrent Mark Sweep*

![cms-gc](https://github.com/nmin11/TIL/assets/75058239/ae758bfe-404e-4b03-ba4a-f5a0cf2384f5)

❖ When prefixed with "Concurrent," it means it is performed concurrently during application operation

- Initial Mark phase: Finds only live objects closest to the class loader and stops
- *Concurrent* Mark phase: Marks objects referenced by the confirmed live objects as roots
- Remark phase: Marks newly added or unlinked objects from the Concurrent Mark phase
- *Concurrent* Sweep phase: Removes unmarked objects

Advantages

- Very short STW time
  - Also known as Low Latency GC
  - Used when low response time is important

Disadvantages

- High CPU and memory usage
- No compaction phase, leading to potential memory fragmentation
  - With a lot of fragmented memory, the STW time for compaction becomes much longer

❖ Enable CMS GC flag

```bash
java -XX:+UseConcMarkSweepGC
```

## G1 GC

*Garbage First*

![g1-gc](https://github.com/nmin11/TIL/assets/75058239/f3893306-45de-4b70-912b-12bd2e5e47ba)

- Default GC from Java 9 onwards
- Completely different approach from previous GCs
- Used in multi-processor environments with ample memory space
- Manages memory by dividing the heap into regions called **Region**, each logically separated (Eden, Survivor, Old, Humongous, Available/Unused)
  - Humongous: Area for storing large objects exceeding 50% of the region size
  - Available/Unused: Area not yet used

GC Phases

- Global Marking phase: Determines the activity level of objects in each region
- Sweep phase: Collects garbage starting from almost empty regions to ensure significant free space

### RSet (Remembered Set)

![rset](https://github.com/nmin11/TIL/assets/75058239/ce68836b-8673-46cc-b60e-cc18fe8829f5)

- Reference management device existing for each region to reference the inside of the heap from outside
- Used to track floating garbage
  - floating garbage: Objects referenced by already dead objects, making them appear alive

### Minor GC

- Performed when Eden space is full
- Moves surviving objects in Eden space to Survivor space
- Designates the now empty Eden space as Available/Unused

### Major GC

![g1-gc-process](https://github.com/nmin11/TIL/assets/75058239/a9af8b42-4f0b-4b38-bb7d-06e850f6cf2e)

- Initial Mark phase: Marks objects in the Survivor space referenced by objects in the Old generation (STW)
- Root Region Scan phase: Scans for Survivor objects found in the previous phase
- Concurrent Mark phase: Scans the entire heap → regions with no garbage are excluded from the next phase
- Remark phase: Identifies objects to be excluded from GC finally (STW)
- Cleanup phase: Removes unused objects from the region with the fewest live objects (STW)
- Copy phase: Performs compaction by moving surviving objects to new regions even after Cleanup (STW)

❖ Enable G1 GC flag

```bash
java -XX:+UseG1GC
```

## Z GC

![zgc-concept-1](https://github.com/nmin11/TIL/assets/75058239/a26abb49-e049-4240-a634-29ec9fde7635)

- Introduced as an experimental version in Java 11, officially released from Java 15 onwards
- **STW time is below 10ms!**
  - Suitable for low-latency, concurrent, high-cost operations
- Handles heap sizes from 8MB to 16TB
- Like G1 GC, manages the heap by dividing it into regions, but each region has a different size
  - Instead of regions, it distinguishes logical units called **ZPage**
  - Consists of three types: small (2 MB), medium (32 MB), large (N * 2 MB)

### Colored Pointers

![colored-pointers](https://github.com/nmin11/TIL/assets/75058239/7a620525-8a40-4499-a548-ca0b8f80bca4)

- Utilizes 64-bit memory in pointers to store object state values
  - Therefore, only usable on 64-bit operating systems!
- 42 bits are used for object addresses
- 4 bits represent the object's state value
  - Finalizable: Objects accessible only through finalizer (garbage)
  - Remapped: Indicates whether the object is relocated
  - Marked 1 / 0: Indicates accessible objects
- Executes mmap (multi-mapping) for all virtual addresses to save mapping operations between virtual and real addresses, leading to a threefold increase in memory usage

### Load Barriers

![zgc-concept-7](https://github.com/nmin11/TIL/assets/75058239/e7d92b3d-e6db-4a4e-95b5-65031ae2c178)

- Code executed when referencing an object
- Checks whether the object in the heap is referenceable
  - Executes a slow path process if there is a problem before referencing
- Provides a protective barrier before referencing objects
- Unlike G1 GC, prevents STW during memory reallocation process
- Updates references and marks while checking Remap Mark and Relocation Set

GC Phases

- *Pause* Mark Start phase: Marks objects pointed to from ZGC roots (Live Object)
  - Each thread scans its local variables to create GC root sets (very short STW)
- Concurrent Mark/Remap phase: Marks all objects by exploring object references
  - Performs coloring and remapping for objects accessible from GC roots
- *Pause* Mark End phase: Marks incoming objects
- Concurrent Prepare for Relocate phase: Finds areas for relocation and assigns them to the Relocation Set
  - Relocation Set: Set of ZPages containing garbage
- *Pause* Relocate Start phase: Performs relocation and updates all root references
- Concurrent Relocate phase: Updates references after relocating all objects using Load Barriers
  - Updates object addresses using forwarding tables

❖ Overwhelmingly low STW time with Z GC

![zgc-performance](https://github.com/nmin11/TIL/assets/75058239/6ac87a0a-82fc-499c-962e-f3aa88f4c218)

❖ Enable Z GC flag

```bash
java -XX:+UseZGC
```

⚠️ For versions before Java 15, the -XX:+UnlockExperimentalVMOptions option must be added

## References

- https://www.baeldung.com/jvm-garbage-collectors
- https://www.baeldung.com/jvm-zgc-garbage-collector
