---
author: "Loko"
title: "謎のS3 Access Deniedエラー"
date: 2023-07-01
lastmod: 2024-01-04
description: "トリガーされたオブジェクトを照会する過程で発生したAccess Deniedエラー"
tags: ["aws"]
thumbnail: /thumbnail/green-bucket.webp
toc: true
---

## エラーを発見したきっかけ

実は同じエラーを二度も経験した。
二度とも会社の業務中に発生したエラーではなかったが、どちらも会社の業務に関連する機能をテストするために実装したサイドプロジェクトで発生した。
実装したのはS3にオブジェクトが作成された時、**そのeventでトリガーされるLambda関数**だった。
該当のLambda関数は共通して受け取った**eventからオブジェクトのkey nameを取得し、そのkey nameでオブジェクト情報を再度照会するロジック**だった。
しかし私のLambda関数は原因不明のAccess Deniedエラーを返した。
Lambda関数は該当するS3 Bucketから`GetObject`を実行できる十分な権限を持っていたにもかかわらずだ！

<img width="669" alt="aws s3 access denied" src="https://github.com/nmin11/blog/assets/75058239/2ebb354e-dab6-4e92-b4da-916d729dbef6">

## 何が問題だったのか

**key nameに韓国語や特殊文字、スペースなどの値が含まれていると**、eventを通じて受け取った`Record.S3.Object.Key`の値がURLエンコードされて出力された。
先ほど同じ問題を2回経験したと述べたが、一度は韓国語の名前だったため発生し、一度は特殊文字が含まれていたため発生した😓

<img width="853" alt="url encoded key name" src="https://github.com/nmin11/blog/assets/75058239/e76bcbaf-7e9e-48fe-8ae2-1db58bf9a20c">

やはりS3のObject自体がURL形式になっているのでこのように返されたのだと思う。

<img width="618" alt="object url" src="https://github.com/nmin11/blog/assets/75058239/a26d5a94-4b88-4173-8566-9d49ec97378b">

上記のObject URLを使ってローカル環境でAWS SDKを通じて直接オブジェクト情報を取得してみた。

<script src="https://gist.github.com/nmin11/95c04703578e7099ec91091aac088b12.js"></script>

すると以下のようなエラーが出力された。

```sh
Failed to retrieve object: NoSuchKey: The specified key does not exist.
```

当然の結果だが、一つ違う点があった。
私のローカル環境では該当するキーのオブジェクトが存在しないという正しいエラーを返したが、
私のLambda関数はAccess Deniedを返したという点だ。
繰り返し言うが、たとえ存在しないキー値であっても、私のLambdaは該当Bucketに対して`GetObject`を実行する権限がある。

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "VisualEditor0",
      "Effect": "Allow",
      "Action": ["s3:GetObject"],
      "Resource": "arn:aws:s3:::nmin-access-test/*"
    }
  ]
}
```

この問題はAWSでLambdaがS3にアクセスする際に発生しうるエラーに対する分岐処理が適切に行われていないために発生したと推定される。
この誤ったエラー表記方式のせいで、私は実際'S3 Access Denied'関連のキーワードでのみ検索しながら問題を解決しようとして約1週間の時間を無駄にした。
_だから実は単純な問題にもかかわらず、一度記録しておこうと思った。_

## どうすれば解決できるか

Lambda関数でeventを通じて受け取ったオブジェクトの情報を再度照会する必要がある場合、eventではURL形式の文字列で入ってきたkey nameを再び元の文字列にパースする過程が必要だ。

```go
decodedObject, err := url.PathUnescape(object)
if err != nil {
  fmt.Println("Error occurred while decoding URL: ", err)
}

decodedObject = strings.Replace(decodedObject, "+", " ", -1)
```

Go言語では組み込みモジュールで提供される`url.PathUnescape`関数があったので使ってみた。
しかしこの関数は空白値(` `)が特殊記号(`+`)にURLエンコードされる部分を処理してくれなかったので、`strings.Replace`関数まで追加で使用する必要があった。
ここまで処理したらeventから韓国語や特殊文字、スペースが含まれたkey nameを受け取っても正常にオブジェクトを照会できるようになった。

### デモに使用した全ソースコード

<script src="https://gist.github.com/nmin11/26204a27da20909f5c18fc851b835dcc.js"></script>

---

## + 2024/1/4 追記

今日、会社で偶然AWS S3でなぜオブジェクトがない時にもAccess Deniedエラーを返すのかについての理由を知った。
AWSのポリシー上、`GetObject`で探そうとしているオブジェクトが存在するかどうかを知らせないためだという。
退勤後にもう少し調べてみるとStackOverflowにも[関連質問](https://stackoverflow.com/questions/56027399/why-am-i-getting-different-errors-when-trying-to-read-s3-key-that-does-not-exist)があった。
要約すると、`GetObject`権限のみあり`ListObject`権限がない場合、特定のキーが存在するかどうかを探索できないように設計されているという。
私が実装した例で言えば、私のローカル環境ではAWS credentialを直接使って実行したため`GetObject`および`ListObject`権限の両方があった。
しかしLambdaにデプロイした時は`GetObject`権限のみ付与されたため権限関連の問題が発生したのだ。
