---
author: "Loko"
title: "Java並行性を振り返る"
date: 2025-12-10
lastmod: 2025-12-10
description: "Javaエコシステムで20年間、並行性をどのように扱ってきたか"
tags: ["java", "concurrency"]
thumbnail: /thumbnail/loom.webp
toc: true
---

## 0. 序論

恥ずかしながら今年、バックエンドブートキャンプ課程を修了しながら**「並行性」**というキーワードを初めて接した。
実はブートキャンプを修了する過程で様々なLock方式を実習しながら並行性を制御する方法は十分に習得できた。
しかし問題を解決する方法だけに集中するよりは「並行性」について本質的な理解をしてから進むべきだと思った。そうすれば今後もバックエンド開発者としての専門的な視野を持てるはずだ。
そこで並行性プログラミングとは何か、またJava陣営で並行性を扱う方法がどのように発展してきたかを見てみることにした。

## 1. The Free Lunch Is Over（2004）

2004年12月、Microsoftの開発者だったHerb Sutterという方が[The Free Lunch Is Over](http://www.gotw.ca/publications/concurrency-ddj.htm)という記事を発表した。
この記事は以下の強烈なスローガンとともに始まる。

> The biggest sea change in software development since the OO revolution is knocking at the door, and its name is **Concurrency**.

> オブジェクト指向革命以来最大の変化の波がソフトウェア開発の扉を叩いている。それは**並行性**だ。

Herb Sutterは1965年に「半導体に集積されるトランジスタの数が毎年2倍に増加する」と主張した*ムーアの法則*が徐々に限界に達していると説明する。
2001年8月にIntelチップのCPUクロック速度は2GHzだったが、この記事が発表された2004年12月にも4GHzの速度を持つCPUは登場していなかったと述べている。
実際4GHzのクロック速度はそう遠くない将来に達成できそうに見えるが、果たして10GHzのクロック速度を達成できるだろうか？
残念ながら限られた半導体容積内に過度に多くのトランジスタを集積すると発熱、消費電力、電流リークの問題が発生するため、より高いクロック速度を達成することはますます難しくなる。
そのため2025年現在でも高性能CPUが出せる最大クロック速度は6GHz程度のレベルにとどまっている。

これに対しHerb Sutterは、開発者が毎年コンピューターのアップグレード要請だけすれば自動的に性能を引き上げられた**「タダ飯」**がすでに1〜2年前に終わったと述べ、現代のアプリケーションの幾何級数的に高まるCPU処理量に備えるために並行性プログラミングがますます重要になると強調する。
またJavaのようなプログラミング言語レベルでも並行性プログラミングモデルが切実に必要だと言及する。

## 2. JSR-133/166（2004）

このような流れに合わせてJCP（Java Community Process）は2004年にJSR（Java Specification Request）133および166を承認する。
上記2つのスペックを通じてJava言語は並行性プログラミングを実行できる言語として確固たる地位を築いた。
どのような変更があったか簡潔に見てみよう。

### JSR-133

実はJava言語は1.0の時からJMM（Java Memory Model）を通じてマルチスレッドを扱うことができた。
しかし`final`フィールドの値を変更できたり、同期化のための再配列を許可しないなど深刻な欠陥が存在した。

> **再配列**：コンパイラ、JIT、キャッシュなど様々な要因により演算の順序はコード上の流れと異なることがあるが、同期化作業を行うにはコンパイラ、ランタイム、ハードウェアはまるで直列実行されるように共謀（協力）しなければならず、この過程を「再配列」と呼ぶ

そのためJSR-133はJMMの欠陥を改善し、`volatile`、`synchronized`、`final`キーワードが直感的に動作することを核心目標として進められた。
開発者がマルチスレッドプログラムがどのようにメモリと相互作用するか自信を持って推論できるよう助け、様々な有名アーキテクチャを正確性と高性能を兼ね備えて実装できるよう提供しようとした。
概略的な改善事項は以下の通り。

- **Happens-Before**
  - 先に発生した動作は先に「配列」され、以降の動作で読み取れるよう保証
  - スレッドの各作業は該当スレッドの次の作業より**先に発生**
  - モニターのロック解除は該当モニターの後続ロックより**先に発生**
  - `volatile`フィールドへの書き込みは該当`volatile`フィールドへの後続読み取りより**先に発生**
  - すべてのメモリ作業はモニターロック解除前に発生し、モニター解除はロック取得前に発生
- `volatile`
  - スレッド間で状態値をやり取りできる特殊フィールドの役割を果たす
  - フィールドの値はメインメモリにflushされ、すべてのスレッドで即座に見ることができる
  - つまり、プログラマーが該当フィールドをキャッシング、再配列などにより「古い」値として表示されることを許可しないと表示したもの
  - 従来は単にキャッシュの代わりにメモリ直接アクセスの意味だったが、追加でMemory Barrierの役割も果たすようになった

### JSR-166

JSR-166ではスペックリードだったDoug Leaの主導の下、並行性を扱うパラダイムを**「言語サポート方式」**から**「ライブラリサポート方式」**へ転換した。
JSR-133ではlow-levelの言語レベルでメモリモデルを提供したが、JSR-166ではhigh-levelで抽象化ツールを提供したのだ。
抽象化ツールは`java.util.concurrent`パッケージとともに提供され、核心的な構成要素は以下の通り。

- **Executor Framework**
  - `ExecutorService`、`ThreadPoolExecutor`
  - スレッドを直接扱わないよう**task**として抽象化
  - スレッドプール管理の複雑さを隠蔽化
- **Lock & Condition**
  - `ReentrantLock`、`ReadWriteLock`
  - `synchronized`より柔軟なロック機能を提供
  - タイムアウト、インタラプト、公平性制御
- **Atomic Variables**
  - `AtomicInteger`、`AtomicReference`
  - CAS（Compare-And-Swap）ベースで動作
  - Lock-freeアルゴリズムを保証する構成要素
- **Concurrent Collections**
  - `ConcurrentHashMap`：セグメントベースロック
  - `CopyOnWriteArrayList`：読み取り最適化方式
  - `BlockingQueue`：Producer-Consumerパターン
- **Synchronizers**
  - `CountDownLatch`、`CyclicBarrier`、`Semaphore`
  - スレッド間協力パターンの抽象化

JSR-166が伝えるメッセージは明確だ。
つまり**「スレッドを直接扱わず高水準に抽象化した並行性ライブラリを活用しよう」**ということ。

## 3. Akka / RxJava

Java言語レベルでスレッド抽象化技法が提供されたが、この方式には根本的な問題がある。
つまりLockベースプログラミングの特性上Race Conditionが発生せざるを得ず、そのため高い次元の並行性環境ではスケーラビリティに限界があるという点だ。
これにより2000年代後半、JVMエコシステムでは**「共有しなければ同期化する必要がない」**という原則ベースのパラダイムが生まれた。

### Akka（2009）

AkkaはActorモデルを基盤に動作する。
1973年、Carl HewittがMITでActorモデル理論を発表し、これを基盤に1986年に実装された言語がErlangだ。
そしてこのようなActorモデルの哲学をJonas BonérがScalaおよびJVMフレームワークとして実装したのが**Akka**だ。

<img src="/blog/actor-graph.png">

ActorモデルはLockを使用せず、代わりにカプセル化を強制する。
シグナルに反応する協力エンティティで構成されており、互いにシグナルを送ることでアプリケーション全体が動作する。
これは実際に世界がコミュニケーションする方式とも類似した動作方式と言える。
Akkaの大まかな概念は以下の通り。

- 構成要素
  - Mailbox：メッセージ待機列
  - Behavior：Actorの状態、内部変数などを含む
  - Message：シグナル（signal）のデータ片、パラメータおよびメソッド呼び出しに類似
- カプセル化
  - まるでオブジェクトがメソッド呼び出しに反応するように、Actorはメッセージに反応する
  - 受信者Actorは他のActorとは関係なく独立して実行され、受信したメッセージを順次処理
  - したがってActorの不変性は同期化なしに維持され、Lockは不要
  - 異なるActor同士はそれぞれ受け取ったメッセージを同時に処理することになる
- エラー処理
  - 一般的なエラーの場合
    - カプセル化されたActorは損傷せず、作業にのみエラーが発生
    - サービスActorがエラーメッセージを返信
  - サービス内部エラーの場合
    - すべてのActorはツリー構造で構成されている
    - もし子Actorに障害が発生したら、親Actorで障害対応方式を決定
    - 親Actorが終了する時、子Actorも再帰的に終了
- 核心概念
  - **カプセル化**：各Actorは独立した状態を持ち、外部から直接アクセス不可能
  - **メッセージパッシング**：メソッド呼び出し方式の代わりに非同期メッセージ送信方式を活用
  - **順次処理**：各Actorは一度に1つのメッセージのみ処理

### RxJava（2013）

2009年、MicrosoftでRx.NETを開発することでReactive Programmingパターンを確立した。
これを基盤に2013年にNetflixはRxJavaを開発してReactiveXパターンをJVMに移植した。
RxJavaコードの形態は以下の通り。

```java
Observable<String> videos = getVideos()
    .flatMap(video -> getMetadata(video))
    .filter(metadata -> metadata.rating > 4.0)
    .map(metadata -> metadata.title)
    .timeout(1, TimeUnit.SECONDS);
```

RxJavaを活用すればサービスへの呼び出しを並列で実行し結果を組み合わせる方式を使えるようになる。
RxJavaについても核心的な概念を整理してみた。

- PullベースのIterableの代わりにPushベースのObservable
  - 値は非同期的にpushされる
  - これによりサービス層全体を非同期化
- 各サービス層ができるようになったこと
  - 条件付きキャッシュの即時返却
  - リソースが制限されている場合、スレッドを使用せずにブロッキング
  - マルチスレッド活用
  - non-blocking IOの使用
- API呼び出しに伴うすべての動作は非同期的に動作するが、内部的にblockingかnon-blockingかを選択する方式
- 核心
  - **Composability**：複雑な非同期ロジックを演算子で組み合わせ
  - **Backpressure**：生産者-消費者の速度不一致処理
  - **Thread-safety**：不変データストリームで共有状態を回避

## 4. Reactive Manifesto（2013）

並行性革命が始まった後、上記のように様々なアプローチが乱立するようになった。
Akkaを創設したJonas Bonérはそれぞれ異なる解決策の間で共通点を発見し、すぐに同僚たちと原則を整理して発表するに至る。

[Reactive Manifesto](https://www.reactivemanifesto.org/ko)

Reactive宣言文は4つの核心原則を含んでいる。

### Responsive（応答性）

- システムは可能な限り即座に一貫した応答をすべきだ
- 一貫したサービス品質提供を通じてシステム信頼性を確保
- エラー処理を簡素化しユーザーの新しいインタラクションを促進
- **予測可能な品質のサービス**

### Resilient（回復力）

- システムに障害があっても応答性は維持されるべきだ
- **複製、封じ込め、隔離、委任**を通じて達成すること
- 障害はシステム全体のリスクにつながるべきではなく、回復が可能であることを保証すべきだ
- 障害は正常の一部（「Let it crash」哲学）

### Elastic（弾力性）

- システムの作業処理量が変わっても応答性は維持されるべきだ
- 入力に応じて**割り当てられたリソースを流動的に変化**させながら対応するシステム
- 競合ポイントやボトルネックが発生しないよう設計することが重要
- リアルタイム性能測定ツールを通じて予測可能な規模にスケールするアルゴリズムを導入

### Message Driven（メッセージ駆動）

- **非同期メッセージ伝達方式**に依存しよう
- コンポーネント間の疎結合、隔離、位置透過性を保証する方式
- non-blocking通信方式では受信者がアクティブな時のみリソースを消費することになる
- メッセージキューのモニタリングとともにback-pressureを活用

## 5. Spring WebFlux（2017）

Spring Framework 5.0とともにSpring WebFluxがリリースされた。
これは既存のServletベースのSpring Web MVCとは差別点を持つReactive Stack Web Frameworkであり、以下のような特性を持つ。

- 完全なnon-blockingフレームワーク
- Reactive Streams Back Pressureをサポート
- NettyまたはServletコンテナ環境で駆動される
- 場合によってはSpring Web MVCとSpring WebFluxが一緒に使用されることもある

Spring WebFluxはnon-blocking I/Oベースの非同期処理とともに**速い応答性（Responsive）**を保証し、Mono/Fluxを活用して**非同期データストリーム（Message Driven）**を活用するのでReactiveプログラミングフレームワークと見ることができる。
これによりJava陣営で並行性を扱う方式がフレームワーク段階では確立されたと見られる。

## 6. Project Loom（2023）

Java 21のProject Loomが解決しようとした問題点は以下の通り。

- 既存Javaプラットフォームスレッドの問題点
  - JavaプラットフォームスレッドはOSスレッドと1:1マッピングだった
  - **Memory overhead**：各プラットフォームスレッドは約2MBのスタック空間を活用
  - **Creation cost**：スレッド生成作業はkernel呼び出し作業（スレッド当たり100–300μs）
  - **Kernel limits**：ほとんどのシステムはアドレス空間の制約により32K〜65Kスレッドに制限される
- Reactive frameworksの問題点
  - **Cognitive overhead**：開発チームがreactiveパターンを習得するのに6〜12ヶ月かかる
  - **Debugging complexity**：reactive chains内でスタックトレースを理解するのが難しい
  - **Library ecosystem**：数多くのJavaライブラリがreactiveシステムには適していない
  - **Backpressure complexity**：手動的なpressure管理作業によりバグが発生する可能性

実際にNetflixの開発者は以下のような言葉を残したという。

> We spent more time debugging reactive chains than we saved in scalability.

> 私たちはスケーラビリティの面で節約した時間より多くの時間をリアクティブチェーンのデバッグに費やした。

Project Loomはこのような問題点を除去しながら革新的な機能を提供してくれる。

- **Virtual Threads**：JVMが管理し自動的に解放される軽量スレッド
- **Structured Concurrency**：自動的にクリーンアップ作業が行われる階層的タスクスコープ
- **Scoped Values**：ThreadLocalパターンを置き換える不変コンテキスト伝播機能

Project Loomは並行性管理を既存のOSカーネル中心からJVM中心に移し、これによりスケジューリング、メモリ管理、観測を開発者が完璧に制御できるようにした。
Reactive方式に対するラーニングカーブを減らせる代替策を用意し、Debuggabilityも確保できるようにした。

## 7. 結論

**The Free Lunch Is Over**では並行性に関連する問題を認識し、**JSR-133/166**は言語レベルでの解決策を提示した。
**Akka/RxJava**はJava陣営で並行性を扱うための実験的なアプローチをし、**Reactive Manifesto**を通じてついにReactive哲学が確立された。
**Spring WebFlux**はReactiveをフレームワークに溶け込ませたが、Javaは**Project Loom**を通じて並行性関連の問題を根本的に再設計した。
The Free Lunch Is Over発表後20余年が経ったが「並行性革命」はまだ終わっていないようだ。
しかし今回の機会に並行性を扱うパラダイムが変化する過程を見守りながら、どのような核心的価値が発生したかを知ることができ、今後も並行性プログラミングを扱う時にどのような方式でアプローチすればよいかについての視野と観点を得ることができた。

## References

- [The Free Lunch Is Over](http://www.gotw.ca/publications/concurrency-ddj.htm)
- [JSR 133 (Java Memory Model) FAQ](https://www.cs.umd.edu/~pugh/java/memoryModel/jsr-133-faq.html)
- [How the Actor Model Meets the Needs of Modern, Distributed Systems](https://doc.akka.io/libraries/akka-core/current/typed/guide/actors-intro.html)
- [Reactive Programming in the Netflix API with RxJava](https://netflixtechblog.com/reactive-programming-in-the-netflix-api-with-rxjava-7811c3a1496a)
- [Reactive Manifesto](https://www.reactivemanifesto.org/ko)
- [Spring WebFlux](https://docs.spring.io/spring-framework/reference/web/webflux.html)
- [Project Loom & Virtual Threads: How Java 21+ Is Revolutionizing Concurrency](https://medium.com/@dikhyantkrishnadalai/project-loom-virtual-threads-how-java-21-is-revolutionizing-concurrency-582a173b2b12)
