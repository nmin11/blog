---
author: "Loko"
title: "主要なGCアルゴリズム"
date: 2024-03-14
lastmod: 2024-03-14
description: "主要なガベージコレクションアルゴリズムについて学ぼう"
tags: ["java"]
thumbnail: /thumbnail/garbage-collectors.webp
toc: true
---

> 『Java最適化』スタディをしながら主要なGCアルゴリズムについてより深く勉強するために書いた記事です。

## Serial GC

<img src="https://github.com/nmin11/blog/assets/75058239/2a864abb-d0f9-4371-a7ea-e86db324af71" width=300>

- 最も単純なGC実装
- STW（Stop-The-World）後、1つのスレッドでのみGCを実行
- mark-sweep-compactアルゴリズムを使用
- マルチスレッド環境には不適切

❖ Serial GC有効化フラグ

```bash
java -XX:+UseSerialGC
```

## Parallel GC

<img src="https://github.com/nmin11/blog/assets/75058239/fbf491b0-e50e-4c8d-91c1-2908a363b0d1" width=400>

- Java 5〜8までのデフォルトGC
- GC処理率を最大化するため*Throughput Collector*とも呼ばれる
- メモリ空間が十分でコア数が多い場合に有利なアルゴリズム
- フラグを通じてGCに使用する最大スレッド数、最大停止時間、GC実行時間比率を設定可能
  - `-XX:ParallelGCThreads=<N>`
  - `-XX:MaxGCPauseMillis=<N>`
  - `-XX:GCTimeRatio=<N>`

❖ Parallel GC有効化フラグ

```bash
java -XX:+UseParallelGC
```

## Parallel Old GC

- Parallel GCのOld領域バージョン
- Old領域までマルチスレッドでGCを実行
- Old領域に対して*Mark-Summary-Compact*アルゴリズムを使用
  - Markフェーズ：Old領域をリージョン別に分け、リージョン別に頻繁に参照されるオブジェクトをマーキング
  - Summaryフェーズ：生き残ったオブジェクトの中で密度が高い部分がどこか**dense prefix**を決め、これを基準にcompact領域を減らす
  - Compactフェーズ：compact領域をdestinationとsourceに分け、生き残ったオブジェクトをdestinationに移動し、参照されていないオブジェクトは削除

❖ Parallel Old GC有効化フラグ

```bash
java -XX:+UseParallelOldGC
```

## CMS GC

*Concurrent Mark Sweep*

![cms-gc](https://github.com/nmin11/TIL/assets/75058239/ae758bfe-404e-4b03-ba4a-f5a0cf2384f5)

❖ Concurrentが付くとアプリケーション動作中に同時に実行されるという意味

- Initial Markフェーズ：クラスローダーから最も近いオブジェクトの中で生きているオブジェクトだけを見つけて終了
- *Concurrent* Markフェーズ：生きていると確認したオブジェクトをルートとして参照されるオブジェクトをマーキング
- Remarkフェーズ：Concurrent Markフェーズで新しく追加されたり参照が切れたオブジェクトをマーキング
- *Concurrent* Sweepフェーズ：マーキングされていないオブジェクトを削除

長所

- STW時間が非常に短い
  - Low Latency GCとも呼ばれる
  - 低い応答速度が重要な場合に使用される

短所
- CPUとメモリ使用量が多い
- compactionフェーズがないためメモリ断片化が発生する可能性がある
  - 断片化されたメモリが多いため、compaction作業を行うとSTW時間がさらに長くなる

❖ CMS GC有効化フラグ

```bash
java -XX:+UseConcMarkSweepGC
```

## G1 GC

*Garbage First*

![g1-gc](https://github.com/nmin11/TIL/assets/75058239/f3893306-45de-4b70-912b-12bd2e5e47ba)

- Java 9からのデフォルトGC
- これまでのGCとは全く異なる方式
- メモリ空間が豊富なマルチプロセッサ環境で使用
- ヒープメモリを同じサイズの**Region**という領域に分けてメモリを管理
  - 各リージョンは論理的に区分される（Eden、Survivor、Old、Humongous、Available/Unused）
  - Humongous：リージョンサイズの50%を超える大きなオブジェクトを格納する領域
  - Available/Unused：まだ使用されていない領域

GC実行フェーズ

- グローバルマーキングフェーズ：リージョン別オブジェクト活性度を判断
- スイープフェーズ：ほぼ空のリージョンから優先的にガベージを収集して大きな空き容量を確保

### RSet（Remembered Set）

![rset](https://github.com/nmin11/TIL/assets/75058239/ce68836b-8673-46cc-b60e-cc18fe8829f5)

- 領域ごとに1つ存在する、外部からヒープ領域内部を参照するためのリファレンス管理装置
- floating garbageを追跡するための用途
  - floating garbage：すでに死んだオブジェクトが参照しているオブジェクトで、生きているように見えるオブジェクト

### Minor GC

- Eden領域がいっぱいになると実行
- Eden領域の生き残ったオブジェクトをSurvivor領域に移動
- 空になったEden領域をAvailable/Unused領域に指定

### Major GC

![g1-gc-process](https://github.com/nmin11/TIL/assets/75058239/a9af8b42-4f0b-4b38-bb7d-06e850f6cf2e)

- Initial Markフェーズ：Oldリージョンのオブジェクトが参照するSurvivor領域のオブジェクトをマーキング（STW）
- Root Region Scanフェーズ：上記フェーズで見つけたSurvivorオブジェクトに対するスキャン作業
- Concurrent Markフェーズ：ヒープ全体に対するスキャン作業を実施 → ガベージがないリージョンは次のフェーズから除外
- Remarkフェーズ：最終的にGC対象から除外するオブジェクトを識別（STW）
- Cleanupフェーズ：生きているオブジェクトが最も少ないリージョンに対する未使用オブジェクト削除作業（STW）
- Copyフェーズ：Cleanup後も完全に空にならなかった生きているオブジェクトを新しいリージョンに移すことでcompaction作業を実行（STW）

❖ G1 GC有効化フラグ

```bash
java -XX:+UseG1GC
```

## Z GC

![zgc-concept-1](https://github.com/nmin11/TIL/assets/75058239/a26abb49-e049-4240-a634-29ec9fde7635)

- Java 11で実験バージョンとして登場、その後Java 15から正式バージョンをリリース
- **STW時間を10ms以下に！**
  - 低レイテンシを必要とする同時性の高い高コスト作業に適している
- 8MB〜16TBサイズのヒープ処理が可能
- G1 GCと同様にヒープを領域別に分けて管理するが、各領域のサイズは異なる
  - regionの代わりに**ZPage**という論理単位で区分
  - small（2 MB）、medium（32 MB）、large（N * 2 MB）の3タイプを持つ

### Colored pointers

![colored-pointers](https://github.com/nmin11/TIL/assets/75058239/7a620525-8a40-4499-a548-ca0b8f80bca4)

- オブジェクトを指す変数のポインタで64bitメモリを活用してオブジェクトの状態値を格納
  - そのため64bit OSでのみ使用可能！
- 42bitはオブジェクトのアドレス値として活用
- 4bitはオブジェクトの状態値を表現
  - Finalizable：finalizerを通じてのみアクセス可能なオブジェクト（ガベージ）
  - Remapped：オブジェクトの再配置の有無を表示
  - Marked 1 / 0：アクセス可能なオブジェクトを表示
- 仮想アドレスと実アドレスマッピング演算の節約のため、すべての仮想アドレスに対してmmap（multi-mapping）を実行
  - これによりメモリ使用量が3倍に増加

### Load barriers

![zgc-concept-7](https://github.com/nmin11/TIL/assets/75058239/e7d92b3d-e6db-4a4e-95b5-65031ae2c178)

- オブジェクトを参照する時に実行されるコード
- ヒープにあるオブジェクトが参照可能な状態かを検査
  - 問題がある場合はslow pathというプロセスを実行した後に参照を進行
- オブジェクトを参照する前にバリアで防いでくれるというイメージで連想すると理解しやすい！
- G1 GCとは異なり、メモリ再配置過程でSTWが発生しないようにする
- Remap MarkとRellocation Setを確認しながら参照とmarkを更新

GC実行フェーズ

- *Pause* Mark Startフェーズ：ZGCのルートから指すオブジェクトにMark表示（Live Object）
  - 各スレッドが自分のローカル変数をスキャンしてGCルートセットを作成（非常に短いSTW）
- Concurrent Mark/Remapフェーズ：オブジェクトの参照を探索しながらすべてのオブジェクトにMark表示
  - GCルートからアクセス可能なオブジェクトに対するcoloringとremappingを実行
- *Pause* Mark Endフェーズ：新しく入ってきたオブジェクトに対するMark表示
- Concurrent Prepare for Relocateフェーズ：再配置する領域を見つけてRelocation Setに配置
  - Relocation Set：ガベージが1つでもあるZPageの集合
- *Pause* Relocate Startフェーズ：すべてのルート参照の再配置を進行し更新
- Concurrent Relocateフェーズ：Load barriersを使用してすべてのオブジェクトを再配置後、参照を修正
  - フォワーディングテーブルを使用してオブジェクトのアドレスを更新

<br>

❖ Z GCの圧倒的なSTW時間

![zgc-performance](https://github.com/nmin11/TIL/assets/75058239/6ac87a0a-82fc-499c-962e-f3aa88f4c218)

<br>

❖ Z GC有効化フラグ

```bash
java -XX:+UseZGC
```

⚠️ Java 15以前のバージョンでは`-XX:+UnlockExperimentalVMOptions`オプションを追加する必要がある

## References

- https://d2.naver.com/helloworld/1329
- https://www.baeldung.com/jvm-garbage-collectors
- https://velog.io/@guswlsapdlf/Java-GC-%EC%95%8C%EA%B3%A0%EB%A6%AC%EC%A6%98
- https://huisam.tistory.com/entry/jvmgc
- https://renuevo.github.io/java/garbage-collection/
- https://www.baeldung.com/jvm-zgc-garbage-collector
- https://d2.naver.com/helloworld/0128759
