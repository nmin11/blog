---
author: "Loko"
title: "IntelliJ IDEAを勉強してみよう"
date: 2026-01-27
lastmod: 2026-01-27
description: "IDE活用法の習得を通じて生産性を最大化する"
tags: ["ide"]
thumbnail: /thumbnail/intellij.webp
toc: true
---

> 📢 本記事で紹介されるすべてのショートカットはMac OS基準で作成されています。

## 0. 序論：生産性に直結するIDE活用能力

ターミナルでテキストを修正する方式のviを経て、1980年代以降にはTurbo Pascal、Visual BasicなどのGUIベースIDEが本格的に登場し始めた。
さらに2000年代以降に登場したVS Code、Eclipse、IntelliJなどの最新IDEは「エディタ」レベルから脱し、コード自動補完、リアルタイムエラー検査、プロジェクト管理、バージョン管理統合、リファクタリングツールなどをサポートしながらIDEをインテリジェントツールへと発展させた。
現時代のIDEには開発者の生産性を引き上げるためのノウハウが数十年にわたって蓄積されている。
AIのCode Generation機能が拡張されている今に至っても、AIが生産する膨大な量のコードを編集し、細かくチューニングするためにはIDE活用能力が必須だ。
だからこそ今この時期に自分自身のIDE活用能力を振り返ってみようとこの記事を書くことにした。

## 1. IntelliJ IDEAについて

<img src="/blog/about-intellij.png" class="hover-zoom">

IDEの中でもJava、Kotlin言語で開発しながら最も多く活用してきたIntelliJ IDEAを中心的に見ていこうと思う。
2000年、非効率的な作業を助けてくれる**Renamer**がまずリリースされ、その後JavaおよびKotlin開発に必須のツールを提供するIntelliJに生まれ変わった。
その後JetBrainsは数十年間、先導的なIDE供給業者として成長し、世界中で1,500万人以上のユーザーを獲得した。
また2009年にはオープンソースライセンスが付与された**Community Edition**を無料で提供し、IDEへのアクセシビリティを高めた。
2014年にはDomain-Specific Languageをサポートする**MPS**を導入し、2019年には**Kotlinへのサポート**を強化した。
そして2025.3バージョンで**CommunityとUltimateの統合**が行われ、コア機能を無料で提供し、高度な機能はサブスクリプションが必要な方式に変わった。

IntelliJ IDEAはそれ自体がJVMで実行され、OpenJDK 21ベースのJetBrains Runtimeがバンドルに内蔵されている。
基本的にGitと統合されバージョン管理機能を提供し、Dockerとも基本的に統合されローカルまたはリモートで実行されるDockerエンジンをサポートする。

IntelliJはJava、Kotlin以外にもScala、Groovy、JavaScript、TypeScript、Rustなど様々な言語をサポートし、特にSpring Bootというフレームワークに対して広範なサポートを提供する。
それだけでなくデータ処理のために関連ツールおよびSQLプラグインを提供し、これを活用してスキーマを探索し編集することもできる。

近頃は**JetBrains AI Assistant**を通じてAIベースの自動化機能を提供することにも重点を置いており、コードベースを基にユニットテストを自動化する機能も提供している。
2025.1バージョンでローカルモデルに無制限アクセスする方式でサブスクリプションの障壁なしにAIを無料で使用できるようにしている。

ここまでがIntelliJ IDEAについての概要で、次のチャプターからはIntelliJの核心的なFeatureを一つずつ見ていこう。

## 2. 生産性を最大化する方法

IntelliJのすべての機能を幅広く見るよりも、生産性を引き上げてくれる有用な機能を中心的に見ていこう。

### Live Templates

Live Templateは略語を入力するだけでコードテンプレートを完成させることができる機能だ。
代表的に`psvm`と`sout`があり、活用例は以下の通り。

<img src="/blog/psvm-sout.gif" class="hover-zoom">

実はLive Templateはよく使うコードを自分で見つけてカスタムテンプレートを用意しておく方が生産性の面でより大きな効果を得られると思う。
ちょうど筆者は最近[AtCoder](https://atcoder.jp/)でコーディングテスト問題を解くために、自動補完機能を提供してくれるIntelliJを愛用していた。
しかしAtCoderは他のプラットフォームとは異なり、パラメータを関数引数ではなくプログラムへの入力として受け取るため、すべての問題を解くたびに入力値に対する面倒な処理が必要だ。

```java
import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;
import java.util.StringTokenizer;

public class J11050 {
    public static void main(String[] args) throws IOException {
        BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
        StringTokenizer st = new StringTokenizer(br.readLine());
        int n = Integer.parseInt(st.nextToken());
        int k = Integer.parseInt(st.nextToken());

        int[][] dp = new int[n + 1][n + 1];

        for (int i = 0; i <= n; i++) {
            dp[i][0] = 1;
            dp[i][i] = 1;
        }

        for (int i = 2; i <= n; i++) {
            for (int j = 1; j < i; j++) {
                dp[i][j] = dp[i - 1][j] + dp[i - 1][j - 1];
            }
        }

        System.out.println(dp[n][k]);
    }
}
```

競技プログラミングの問題を解くためには`main`メソッドに入力失敗例外のための`IOException`処理、入力を受け取るための`BufferedReader`生成過程がすべての問題に共通で必要であり、スペースで区切られて入力される場合には`StringTokenizer`の活用も必須だ。
そして演算を進めるたびに結果値を順番に保存しておくために`StringBuilder`も多く使われる。
実はGraphやDPなど様々なアルゴリズム類型ごとにバリエーションを付けながらテンプレートを作っておくのも生産性の面で大きな助けになるだろう。
しかしここでは紙面の都合上、基本テンプレートだけ作ってみよう。

```java
import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;
import java.util.StringTokenizer;

public class $CLASS_NAME$ {
    public static void main(String[] args) throws IOException {
        BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
        StringTokenizer st = new StringTokenizer(br.readLine());
        StringBuilder sb = new StringBuilder();

        int n = Integer.parseInt(st.nextToken());

        $END$

        System.out.print(sb);
    }
}
```

実際に使ってみると以下の通り。

<img src="/blog/jboj.gif" class="hover-zoom">

余談だが、既存のclass宣言を消してから再度テンプレートを適用しなければならない点がやや不便に感じられ、もう少し調べてみるとFile Templateという機能を確認できた。
この機能を活用すればファイルを生成する時点でテンプレートをすぐに適用できるのでより便利だ。

<img src="/blog/file-template.gif" class="hover-zoom">

### Postfix Completion

Code Completionについても扱おうかと思ったが、コードを入力していると自動的に提案してくれる機能なので自然に習得できると判断した。
代わりに勉強してみる価値のある様々な機能を探していると、**Postfix Completion**を知ることになり、あまり使っていなかったので軽く見てみるといいと思った。
Postfix Completionは文字通り「接尾辞」を基に自動補完してくれる機能で、変数名を先に入力してからまるでチェーンメソッドを活用するように自動補完機能を活用できるようにしてくれる。
「この変数を使ってfor文を作って」と命令するような感じなので、開発者の意識の流れ通りにロジックを作成できるようになる。
ちなみにLive Templateにもあった`fori`と`sout`のような略語をPostfix Completionでも使える。

<img src="/blog/postfix-completion.gif" class="hover-zoom">

### Bookmark

<img src="/blog/starcraft-2.webp" class="hover-zoom">

唐突な話だがRTSゲームをしていると「部隊指定」という機能を通じてユニットや建物を役割によってグループ化し、必要な瞬間にグループごとに適切な命令を素早く出せる。
IntelliJでもこれと似た機能を提供しており、それが**Bookmark**機能だ。

<img src="/blog/bookmark.png" class="hover-zoom">

Bookmarkの種類には**Anonymous**と**Mnemonic**がある。
Anonymousは好きなだけBookmarkを作れるという利点があるが、指定したBookmarkに再び戻る時はBookmarkをリストアップして希望のものを探して入る必要がある。
一方、Mnemonicはショートカットですぐに移動できるが、数字0〜9まで、文字A〜Zまでのみ指定可能だ。
個人的に考えるには、自分で覚えられないAnonymous Bookmarkを無分別に指定するよりも、本当によく読むコードを自分で選定してMnemonic Bookmarkを指定しておく方が生産性の面ではるかに得だと思う。

Bookmark機能を活用するために必ず覚えておくべきショートカットは以下の通り。

- Bookmarkリストアップ：`⌘Cmd` `2`
- Anonymous Bookmark作成：`F3`
- Mnemonic Bookmark作成：`⌥Option` `F3`
- 指定したMnemonic Bookmarkへ移動：`⌃Ctrl` `(指定したmnemonic)`

## 3. デバッグをより丁寧に

恥ずかしながら私は丁寧にデバッグするよりも結果だけを素早く確認する方を選んできた。
しかしこのような習慣が固定化されるとプログラムの動作方式を繊細に観察できなくなる。
その上、近頃はAI codegenへの依存度が高まっているので、丁寧なデバッグ能力を身につけておくことがより有用だろう。

<img src="/blog/intellij-debug.png" class="hover-zoom">

IntelliJのデバッグを活用する時には以下の4つの機能を必ず覚えておこう。

- **Resume**
- **Step Over**
- **Step Into**
- **Step Out**

### Resume

- 機能：次のbreakpointへ移動
- ショートカット：`⌥Option` `⌘Cmd` `R`

### Step Over

- 機能：すぐ次の行へ移動
- ショートカット：`F8`

### Step Into

- 機能：その行で呼び出しているメソッドの中へ移動
- ショートカット：`F7`

### Step Out

- 機能：Step Into後、再びメソッドの外へ移動
- ショートカット：`⇧Shift` `F8`

## 4. Git、IDE内で活用してみよう

実は私はバージョンコントロールをする時[Fork](https://git-fork.com/)というGUIクライアントを愛用している。
インターンをしている時に一度経験してみてから今までずっと活用している。
開発者として便利なツールを活用するのはいいが、実は別のインターフェースに画面を切り替えて様々な機能を実行していると、思ったよりタイムロスが多く生じる。
したがってIntelliJのGit関連機能を活用すれば開発中でもより素早くバージョンコントロールを実行できるようになる。

いくつかのショートカットを覚えておいて必ず適時に活用してみよう。

### Version Control

- 機能：Gitパネルを開く
- ショートカット：`⌘Cmd` `9`

### Show Diff

- 機能：該当ブランチおよびコミットで変更履歴を確認
- ショートカット：`⌘Cmd` `D`

### VCS Operation

- 機能：作業できるGitコマンドを一覧で見る
- ショートカット：`⌃Ctrl` `V`

### Commit & Push

- Commitショートカット：`⌘Cmd` `K`
- Commit & Pushショートカット：`⌥Option` `⌘Cmd` `K`
- Pushショートカット：`⇧Shift` `⌘Cmd` `K`

## 5. 結局はショートカット

このブログのテーマを思いついた時、ただショートカットを羅列するだけの古臭い記事だけは避けたかった。
しかしIDEを活用しながら生産性を最大化しようとするなら、結局はショートカットの暗記が必ず必要だという事実に気づいた。
そういう意味で本記事の最後はIntelliJの核心ショートカットについての整理とともに締めくくろうと思う。

### アクション検索

- 機能：IntelliJで実行できるすべてのアクションを探索する検索窓を開く
- ショートカット：`⇧Shift` `⌘Cmd` `A`

### 新規作成

- 機能：カーソル位置によってパッケージ作成、ファイル作成、メソッド作成など作成可能なものをリストアップ
- ショートカット：`⌘Cmd` `N`

### 実行関連

- 現在のフォーカスで実行ショートカット：`⌃Ctrl` `⇧Shift` `R`
- デバッグ実行ショートカット：`⌃Ctrl` `⇧Shift` `D`
- 現在の実行環境（上部セレクトボックスで選択されているもの）実行ショートカット：`⌃Ctrl` `R`

### 行編集

- 行複製：`⌘Cmd` `D`
- 行削除：`⌘Cmd` `⌫Del`
- 文字列行結合：`⌃Ctrl` `⇧Shift` `J`

### 行移動

- ブロックに関係なく行移動：`⌥Option` `⇧Shift` `↑↓`
- ブロック内でのみ行移動：`⇧Shift` `⌘Cmd` `↑↓`
- エレメント間の左右移動：`⌥Option` `⇧Shift` `⌘Cmd` `←→`

### カーソル移動

- カーソルを単語ごとに移動：`⌥Option` `←→`
- 単語ごとに選択しながら移動：`⌥Option` `⇧Shift` `←→`
- 行の先頭/末尾へ移動：`fn` `←→`
- 行全体を選択しながら移動：`⇧Shift` `fn` `←→`
- ページup/down：`fn` `↑↓`

### コード確認

- コンストラクタやメソッドにどんな引数が必要か確認：`⌘Cmd` `P`
- メソッドの実装部分をポップアップで確認：`⌥Option` `Space`
- Javadoc確認：`fn` `F1`

### フォーカス選択

- フォーカス範囲の拡張/縮小：`⌥Option` `↑↓`（上に行くほど拡張、下に行くほど縮小）
- 前/後のフォーカス位置へ移動：`⌘Cmd` `[` `]`
- マルチフォーカス：`⌥Option` `⌥Option` `↑↓`
- エラーポイントへすぐに移動：`F2`

### 検索

- 現在のファイル内で探索：`⌘Cmd` `F`
- 検索した文字列を置換：`⌘Cmd` `R`
- プロジェクト全体で探索：`⇧Shift` `⌘Cmd` `F`
- プロジェクト全体で置換：`⇧Shift` `⌘Cmd` `R`
- ファイル検索：`⇧Shift` `⌘Cmd` `O`（パッケージ名を含めて検索可能）
- シンボル検索：`⌥Option` `⌘Cmd` `O`（メソッドを検索する時によく使用）
- 最近開いたファイルをリストアップ：`⌘Cmd` `E`
- 最近修正したファイルをリストアップ：`⇧Shift` `⌘Cmd` `E`
- 使用位置を探す：`⌥Option` `F7`
- Structure View：`⌘Cmd` `7`

### 自動補完

- スマート自動補完：`⌃Ctrl` `⇧Shift` `Space`
- staticメソッド自動補完：`⌃Ctrl` `Space` `Space`
- 実装すべき継承メソッドを自動補完：`⌃Ctrl` `I`
- Live Templateリストアップ：`⌘Cmd` `J`

### リファクタリング

- 変数抽出：`⌥Option` `⌘Cmd` `V`
- パラメータ抽出：`⌥Option` `⌘Cmd` `P`
- メソッド抽出：`⌥Option` `⌘Cmd` `M`
- innerクラス抽出：`F6`
- 変数名、クラス名、メソッド名一括変更：`⌃Ctrl` `F6`
- 型一括変更：`⇧Shift` `⌘Cmd` `F6`

### コード整理

- 使用していないimport削除：`⌃Ctrl` `⌥Option` `O`（Auto Importオプションをオンにすると自動的に整理される）
- 自動整列：`⌥Option` `⌘Cmd` `L`
