---
author: "Loko"
title: "의문의 S3 Access Denied 에러"
date: 2023-07-01
lastmod: 2024-01-04
description: "트리거된 객체를 조회하는 과정에서 발생한 Access Denied 에러"
tags: ["aws", "go", "troubleshooting"]
thumbnail: https://github.com/nmin11/blog/assets/75058239/2feabb65-8177-4614-89be-a7c66fd50034
toc: true
---

## 에러를 발견하게 된 계기

사실은 똑같은 에러를 두 번이나 겪었다.  
두 번 다 회사 업무 중 발생한 에러는 아니었지만, 둘 다 회사 업무와 관련된 기능을 테스트해 보기 위해서 구현했던 사이드에서 발생했다.  
구현했던 것은 S3에 객체가 생성되었을 때, **그 event로 트리거되는 Lambda 함수**였다.  
해당 Lambda 함수들은 공통적으로 받은 **event로부터 객체의 key name을 가져오고, 해당 key name을 통해 객체 정보를 다시 조회하는 로직**이었다.  
하지만 내 Lambda 함수들은 알 수 없는 Access Denied 에러를 반환했다.  
Lambda 함수는 해당하는 S3 Bucket으로부터 `GetObject`를 실행할 수 있는 충분한 권한을 가지고 있었는데도 말이다!

<img width="669" alt="aws s3 access denied" src="https://github.com/nmin11/blog/assets/75058239/2ebb354e-dab6-4e92-b4da-916d729dbef6">

## 무엇이 문제였는가

**key name에 한국어나 특수문자, 공백 등의 값이 들어가 있으면** event를 통해 받은 `Record.S3.Object.Key`의 값이 URL enconde 되어서 출력되었다.  
앞서 똑같은 문제를 2번 경험했다고 밝혔는데, 한 번은 한글 이름이었기 때문에 발생했고, 한 번은 특수문자가 끼어 있었기 때문에 발생했다 😓

<img width="853" alt="url encoded key name" src="https://github.com/nmin11/blog/assets/75058239/e76bcbaf-7e9e-48fe-8ae2-1db58bf9a20c">

아무래도 S3의 Object 자체가 URL 형식으로 되어 있다 보니 이렇게 반환된 것 같다는 생각이 든다.

<img width="618" alt="object url" src="https://github.com/nmin11/blog/assets/75058239/a26d5a94-4b88-4173-8566-9d49ec97378b">

위의 Object URL을 가지고 로컬 환경에서 AWS SDK를 통해 직접 객체 정보를 가져와 봤다.

<script src="https://gist.github.com/nmin11/95c04703578e7099ec91091aac088b12.js"></script>

그랬더니 아래와 같은 에러가 출력되었다.

```sh
Failed to retrieve object: NoSuchKey: The specified key does not exist.
```

당연한 결과이지만, 한 가지가 달랐다.  
내 로컬 환경에서는 해당하는 키의 객체가 존재하지 않다는 올바른 에러를 반환했지만,  
내 Lambda 함수는 Access Denied를 반환했다는 점이다.  
거듭 말하지만, 설령 존재하지 않는 키 값이더라도, 내 Lambda는 해당 Bucket에 대해 `GetObject`를 실행할 수 권한이 있다.

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

이 문제는 AWS에서 Lambda가 S3에 access할 때 발생할 수 있는 에러들에 대한 분기 처리가 제대로 안 되어 있기 때문에 발생한 것으로 추정된다.  
이 잘못된 에러 표기 방식으로 인해, 나는 사실 'S3 Access Denied' 관련 키워드로만 검색하면서 문제를 해결하려다가 일주일가량의 시간을 허비했다.  
_그래서 사실 단순한 문제임에도 불구하고, 한번 기록하고 넘어가야겠다는 생각을 가지게 되었다._

## 어떻게 해결할 수 있을까

Lambda 함수에서 event를 통해 받은 객체의 정보를 다시 조회해야 한다면, event에서는 URL 형식의 문자열로 들어온 key name을 다시 원본 문자열로 파싱하는 과정이 필요하다.

```go
decodedObject, err := url.PathUnescape(object)
if err != nil {
  fmt.Println("Error occurred while decoding URL: ", err)
}

decodedObject = strings.Replace(decodedObject, "+", " ", -1)
```

Go 언어에서는 내장 모듈에서 제공하는 `url.PathUnescape` 함수가 있어서 사용해 봤다.  
하지만 해당 함수는 빈 공백값(` `)이 특수기호(`+`)로 url encoded 되는 부분을 처리해 주지 않아서 `strings.Replace` 함수까지 추가로 사용해야만 했다.  
이렇게까지 처리하니 event로부터 한글이나 특수문자, 공백이 들어간 key name을 받게 되어도 정상적으로 객체를 조회할 수 있었다.

### 시연을 위해 사용한 전체 소스 코드

<script src="https://gist.github.com/nmin11/26204a27da20909f5c18fc851b835dcc.js"></script>

---

## + 2024/1/4 추가

오늘 회사에서 우연히 AWS S3에서 왜 객체가 없을 때에도 Access Denied 에러를 반환하는지에 대한 이유를 알게 되었다.  
AWS 정책상 `GetObject` 를 통해 찾으려는 객체가 존재하는지 안하는지 여부를 알려주지 않기 위해서라고 한다.  
퇴근한 이후 조금 더 찾아보니 StackOverflow에도 [관련 질문](https://stackoverflow.com/questions/56027399/why-am-i-getting-different-errors-when-trying-to-read-s3-key-that-does-not-exist)이 있었다.  
요약하자면, `GetObject` 권한만 있고 `ListObject` 권한은 없을 때, 특정 키가 존재하는지 여부를 탐색할 수 없게 하도록 설계되어 있다고 한다.  
내가 구현했던 예제를 가지고 얘기해보자면, 내 로컬 환경에서는 AWS credential을 직접 가지고 실행했기 때문에 `GetObject` 및 `ListObject` 권한 둘 다 있었다.  
하지만 Lambda로 배포했을 때는 `GetObject` 권한만 부여되었기 때문에 권한 관련 문제가 발생했던 것이다.
