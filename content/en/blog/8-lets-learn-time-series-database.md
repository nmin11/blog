---
author: "Loko"
title: "Let's Learn About Time Series Databases"
date: 2026-02-07
lastmod: 2026-02-07
description: "What characteristics does TSDB have, and what problems does it solve?"
tags: ["tsdb", "system-design"]
thumbnail: /thumbnail/clocks-wall.webp
toc: true
---

When designing large-scale architectures or building distributed systems, you may consider adopting a **Time Series Database (TSDB)**. Today, let's explore what TSDB is, what characteristics it has, and what problems it helps solve.

## 1. What is Time Series Data?

TSDB is a database that stores and processes time series data. So what is time series data? Time series data is not simply data with timestamps attached. More precisely, it can be defined as _"measurements or events that are tracked, monitored, downsampled, and aggregated over time"_, and the **measurements** or **events** mentioned here can include:

- Server metrics
- Application performance monitoring
- Network data
- IoT sensor data
- Click events
- Market trading data

Time series data handling differs from general databases in the following ways:

- **Write heavy**: 95% writes, 5% reads
- **Immutable**: Once recorded, data rarely changes, so data is added in an append-only manner
- **Ordered by time**: Sorted in chronological order and can be considered indexed by time
- **High cardinality**: Generally has high cardinality
- **Time-based range queries**: Querying large numbers of records by time range
- **Continuous data stream**: Continuously collecting data points

Let's explore what strategies TSDB employs to handle these characteristics.

## 2. Write Optimization

In a typical database, writing data requires finding the location to write, then modifying or inserting. The process would look like this:

```bash
[Seek to block 4752] → [Read] → [Modify] → [Write] → [Seek to block 9201] → ...
```

On the other hand, TSDB doesn't need to modify existing data and only needs to load data in chronological order, so it simply adds data in an **Append-only** manner. Operations become much simpler:

```bash
[Write to end] → [Write to end] → [Write to end] → ...
```

### LSM (Log-Structured Merge) Tree

The LSM tree is the secret sauce of database systems like InfluxDB, Cassandra, and LevelDB that boast high-performance write throughput. The core ideas of the LSM tree are:

- **Expensive random writes → Cheap sequential writes**
- **Improving read efficiency by periodically reorganizing data in the background**

And the operation of LSM trees works as follows:

1. **Latest data is stored in the Memtable memory buffer**

Data maintains sorted order by key. Write operations are very fast since they access RAM.

2. **Flush to SSTable (Sorted String Table) on disk**

When the Memtable is full, it's saved as an immutable, sorted SSTable file. Since the Memtable is already sorted, this flush operation can be completed sequentially, simply, and quickly. After flushing, the Memtable is cleared and reinitialized.

3. **Background compaction**

In the background, smaller SSTables are merged into larger SSTables. Duplicate entries and tombstones (deletion markers) are also removed.

Due to this LSM tree operation, **write operations don't interfere with read operations.** By having the Memtable always handle the latest data while cleaning up existing data in the background, high write throughput can be maintained continuously.

However, **read performance degradation** can occur when **multiple SSTables need to be checked** for queries. Also, **if write amplification occurs, data may be rewritten multiple times during the compaction process.**

Therefore, LSM trees are best used when write workloads are heavy and some read performance can be sacrificed.

## 3. Time-Based Indexing and Partitioning

Since time series data is already sorted and inserted based on timestamps, indexing is not needed for write operations. And if you partition by time range, you don't need to worry about which partition to store data in during write operations. Data retention can also be easily handled by simply deleting old partitions.

### Block-Level Metadata

In TSDB, data blocks are also sorted based on timestamps. Therefore, blocks that don't fall within the time range can be skipped without checking, improving query performance. Each block records **min/max timestamps** for time, and optionally **min/max values**. This metadata acts as a filter layer that, together with time-based partitioning, helps maintain query performance even as data volume increases.

## 4. Downsampling and Compression

Due to its nature, time series data is well-suited for trimming less important content and representing it in compressed form. In fact, given TSDB's characteristic of continuously ingesting millions of data points per second, downsampling is essentially mandatory. In TSDB, high-precision recent data is aggregated and downsampled into long-term data. A typical downsampling and retention strategy looks like this:

- Raw data: Keep for 1 day
- 1-minute averages: Keep for 7 days
- 5-minute averages: Keep for 30 days
- 1-hour averages: Keep for 1 year
- Daily summaries: Keep permanently

These rules can be defined by users, and leveraging them can dramatically improve storage efficiency and read performance.

## 5. Compression Algorithms

As mentioned earlier, the core of TSDB is **processing numerous write operations in memory buffers** and **compressing in the background**. So how is compression done? Let's explore these concepts.

### Delta Encoding

```bash
Raw values:     [45.2] [45.3] [45.1] [45.4]
Delta encoded:  [45.2] [+0.1] [-0.2] [+0.3]
```

Delta encoding transforms data to record only the change amount from the original value. Since change values are smaller numbers than the original values, the number of bits needed to store values is also much smaller.

### Delta-of-Delta Encoding

```bash
Raw timestamps:     1000, 1010, 1020, 1030, 1040
Deltas:             10  , 10  , 10  , 10  , 10  , ...
Delta-of-deltas:    10  , 0   , 0   , 0   , 0   , ...
```

Delta-of-delta encoding goes further by recording the change amount of change amounts. This method can be used when timestamp intervals are completely regular, and it can reduce the number of bits even more.

### XOR Compression

```bash
Value 1: 0 10000010 01101000101000111101011
Value 2: 0 10000010 01101000110000100000000
XOR:     0 00000000 00000000011000011101011
                    ^^^^^^^^^^ lots of leading zeros
```

XOR compression is a compression algorithm that takes advantage of the fact that consecutive float-based values often have similar values. As shown above, identical bit portions are all replaced with 0, significantly reducing the total number of bits. The Gorilla compression technique developed by Meta also falls into this category.

### Run-Length Encoding

Run-length encoding stores only a single value with a count when measured values appear repeatedly, such as in monitoring metrics. It's useful for recording login counts, request counts, and similar data.

## 6. TSDB Selection Guide

Finally, let's summarize which TSDB to use in what situations.

### InfluxDB

- Query language: InfluxQL, Flux
- Pros: Optimized for event collection
- Cons: Complex distributed system scaling
- Deployment: Self-hosted / Managed cloud
- Use cases: When workload is metrics-focused real-time data collection, when data retention period is short

### Prometheus

- Query language: PromQL
- Pros: Optimized for Kubernetes environments, Pull model so no agent installation required
- Cons: Not designed for long-term storage
- Deployment: Installed by default in Kubernetes
- Use cases: Kubernetes-centric architecture, when alerting system is needed

### TimescaleDB

- Query language: SQL (especially optimized for PostgreSQL)
- Pros: Optimized for relational workloads, PostgreSQL ecosystem
- Cons: Requires PostgreSQL-related dependency installation
- Deployment: Self-hosted / Tiger Data cloud
- Use cases: Business Intelligence (BI), IoT, financial systems, and other cases requiring relational analysis

### Kdb

- Query language: q language
- Pros: Given the price point, you can receive support from the support team
- Cons: Learning curve, poor documentation, expensive commercial license
- Deployment: Not optimized for cloud environments, must be installed on VMs
- Use cases: When extreme performance is required

### Graphite

- Query language: Plain text, Pickle, AMQP
- Pros: Excellent performance on any hardware
- Cons: Poor documentation, community not very active
- Deployment: Container images
- Use cases: When a simple and lightweight open-source TSDB is needed

## References

- Alex Xu - System Design Interview Volume 2
- [influxdata - Time series database explained](https://www.influxdata.com/time-series-database)
- [System Design Academy - Why a Time Series Database?](https://www.systemdesignacademy.com/blog/why-a-time-series-database)
- [hello interview - Time Series Databases](https://www.hellointerview.com/learn/system-design/deep-dives/time-series-databases)
- [Last 9 - Comparing Popular Time Series Databases](https://last9.io/blog/time-series-database-comparison)
- [OctaByte - InfluxDB vs TimescaleDB](https://blog.octabyte.io/topics/open-source-databases/influxdb-vs-timescaledb)
