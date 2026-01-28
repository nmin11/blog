---
author: "Loko"
title: "Github Pagesで作る自分だけの技術ブログ"
date: 2023-02-11
lastmod: 2023-02-11
description: "Hugo + Github Pages"
thumbnail: /thumbnail/hugo-logo.webp
toc: true
---

## 0. Github Pagesとは？

Github Pagesはサーバーやデータベースなしで、自分だけのウェブページを「無料」で作成できる便利なツールだ。
ブログだけでなく、履歴書、ポートフォリオ、公式ドキュメント、ギャラリー、アーカイブ用途でも多く使われている。
（特にポートフォリオ用途で作成する方が結構多いようだ。）
筆者も新しい技術ブログを作りたくてGithub Pagesを選ぶことにした！

[Github Pages](https://pages.github.com)

## 1. Hugoを選んだ理由

Github Pagesを作成するために、静的ウェブサイトを生成する**Static Site Generator**というものを使う必要がある。
筆者は迷わず[ランキング](https://ossinsight.io/collections/static-site-generator/)を確認してみた。
この中で私が選んだ**Hugo**は2023年2月時点で6位と、かなり人気のあるツールだということが分かった。
さらにHugoを活用してGithub Pagesを作成する方法を紹介する日本語のブログ記事も多く、その方法を見ると本当に簡単で手軽そうだった。
Hugoでは様々なテーマが提供されているので、好みのテーマを選んでガイドに従って作成し、自分好みにカスタマイズすればいいと思った。

一方では、2023年2月時点でStatic Site Generatorランキング2位を占めている**Next.js**を活用してみようかという考えもあった。
フロントエンドフレームワークとして注目されているNext.jsをこの機会に勉強してみようかと思ったのだ。
しかしHugoのようにテーマが提供されているようには見えず、そのため直接ページを作成するのにかなりの時間がかかると予想された。
（Next.jsとGithub Pagesを活用してポートフォリオを作る方が多いようだ。）
ブログ作成にそこまで多くの時間を投入したくなかった私は迷わずHugoを選んだ。

## 2. 作成過程

### Hugoのインストール

Hugoのインストールには[Git](https://git-scm.com/downloads)と[Go](https://go.dev/dl/)が必要だ。
上記のリンクから2つをインストールしたらHugoをインストールできるが、筆者はMac OSで[Homebrew](https://brew.sh/)を使ってインストールした。

```sh
brew install hugo
```

インストール後は正常にインストールされたか確認するためにバージョンを確認するといい。

```sh
hugo version
```

### Github Repository 2つの作成

1つはHugoで管理されるブログソース全体を入れるRepositoryだ。
名前は自由に決めてもいいが、大体`blog`という名前で作り、筆者もblogという名前で作った。

もう1つのRepositoryはビルドされて実際のデプロイ環境で動作するファイルを入れる。
このRepositoryの名前は必ず`<USERNAME>.github.io`という名前にしなければならない。

### ブログを管理するディレクトリの作成

ブログファイルが入るディレクトリを作成する必要がある。
希望のフォルダ位置に移動して以下のコマンドを入力することで簡単に作成できる。

```sh
hugo new site <NAME>
```

\<NAME>部分は新しく作るディレクトリ名で、大体blogという名前でフォルダを作り、筆者もそうした。

### 好みのテーマを選ぶ

[Hugo Themes](https://themes.gohugo.io/)ページで好みのテーマを選べる。
好みのテーマをクリックして**Download**ボタンを押すとGithub Repositoryページに移動するが、そのRepositoryを`blog/themes`ディレクトリでcloneすればいい。
（`hugo new site`コマンドで別の名前をつけた場合は、そのフォルダの下でthemesフォルダを探す必要がある。）
そしてダウンロードしたテーマページに整理されている使用方法に従って、`config.toml`ファイルを修正すればいい。

### 作成しておいたGithub Repositoryとの連携

最上位の`blog`ディレクトリは先ほど`blog`という名前で作っておいたRepositoryと連携する。

```sh
git init
git remote add origin https://github.com/<USERNAME>/blog
```

`blog`ディレクトリで`blog/public`ディレクトリをsubmoduleとして追加しながら、`<USERNAME>.github.io` Repositoryを接続する。

```sh
git submodule add -b master https://github.com/<USERNAME>.github.io public
```

### コンテンツ作成とアップロード

各テーマごとに投稿する記事が位置するディレクトリ構造が異なるので、テーマ説明ページでガイドをよく読んでから正確な位置にmdファイルを作成する必要がある。
記事の作成が終わったら`blog`ディレクトリで以下のコマンドでビルドする。

```sh
hugo -t <THEME_NAME>
```

ビルドされたファイルが`blog/public`に入り、デプロイのためにはコードを`github.io` Repositoryにpushするだけでいい。
コード管理のために`blog` Repositoryもpushしよう。

## 3. 改善事項

- 自分だけのロゴを作る！
- ブログコンテンツをテーマに合わせて分類するためにカテゴリー設定
- Aboutページを大幅に手直しする
