---
author: "Loko"
title: "Github Pages로 만드는 나만의 기술 블로그"
date: 2023-02-11
lastmod: 2023-02-11
description: "Hugo + Github Pages"
tags: ["tech", "hugo", "github"]
thumbnail: https://user-images.githubusercontent.com/75058239/218248198-861c23ad-9eed-4240-90f3-445878f794f2.png
---

## 0. Github Pages란?

Github Pages는 서버나 DB 없이, 나만의 웹 페이지를 '무료'로 제작해서 가질 수 있게 해주는 유용한 도구이다.  
블로그 뿐만이 아니라 이력서, 포트폴리오, 공식 문서, 갤러리, 아카이브 용도로도 많이 사용되고 있다.  
(특히 포트폴리오 용도로 제작하는 분들이 꽤 많은 것 같다.)  
필자 또한 새로운 기술 블로그를 만들고 싶어서 Github Pages를 선택하게 되었다!

[Github Pages](https://pages.github.com)

## 1. Hugo를 선택한 이유

Github Pages를 제작하기 위해서, 정적 웹사이트를 생성해주는 **Static Site Generator**라는 것을 사용해야 한다.  
필자는 고민도 없이 [랭킹](https://ossinsight.io/collections/static-site-generator/)을 확인해보았다.  
이 중에서 내가 고른 **Hugo**는 2023년 2월 기준 6위로, 인기가 상당한 도구라는 것을 알 수 있었다.  
게다가 Hugo를 활용해서 Github Pages를 제작하는 방법을 알려주는 한국어로 된 블로그 포스팅도 많았고, 그 방법을 살펴보니 정말 쉽고 간편해보였다.  
Hugo에서 각종 테마들을 지원해주다보니, 내가 원하는 테마를 골라서 가이드에 따라 제작하고, 입맛에 맞게 커스텀하면 되겠다는 생각이 들었다.

또 한편으로는, 2023년 2월 기준 Static Site Generator 랭킹 2위를 차지하고 있는 **Next.js**를 활용해볼까 싶은 생각도 들었다.  
프론트엔드 프레임워크로서 각광받고 있는 Next.js를 이참에 한 번 공부해볼까 하는 생각이 들었던 것이다.  
하지만 Hugo와 같이 테마가 제공되는 것 같아 보이지 않았고, 따라서 직접 페이지를 제작하는 데에 상당한 시간이 소요될 것으로 예상되었다.  
(Next.js와 Github Pages를 활용해서 포트폴리오를 만드는 분들이 많은 것 같다.)  
블로그 제작에 그렇게까지 많은 시간을 투입하고 싶지 않았던 나는 주저없이 Hugo를 선택했다.

## 2. 제작 과정

### Hugo 설치

Hugo 설치를 위해서는 [Git](https://git-scm.com/downloads)과 [Go](https://go.dev/dl/)가 필요하다.  
위 링크에서 2개를 설치했다면 Hugo를 설치할 수 있는데, 필자는 Mac OS에서 [Homebrew](https://brew.sh/)를 활용해서 설치했다.

```sh
brew install hugo
```

설치 이후에는 정상적으로 설치되었는지 확인하기 위해 버전을 확인해주면 좋다.

```sh
hugo version
```

### Github Repository 2개 생성

1개는 Hugo로 관리되는 블로그 소스 전체를 담을 Repository이다.  
이름은 자유롭게 정해도 되지만 대개 `blog` 라는 이름으로 만들며, 필자도 blog라는 이름으로 만들었다.

또 다른 1개의 Repository는 빌드되어서 실제 배포 환경에서 작동할 파일들을 담는다.  
이 Repository의 이름은 반드시 `<USERNAME>.github.io` 라는 이름으로 정해야만 한다.

### 블로그를 관리할 디렉토리 생성

블로그 파일들이 담길 디렉토리를 생성해야 한다.  
원하는 폴더 위치로 이동해서 아래의 명령어 입력을 통해 간단하게 생성할 수 있다.

```sh
hugo new site <NAME>
```

\<NAME> 부분은 새로 만들 디렉토리 이름으로, 대개 blog라는 이름으로 폴더를 만들며, 필자도 그렇게 했다.

### 원하는 테마 고르기

[Hugo Themes](https://themes.gohugo.io/) 페이지에서 원하는 테마를 고를 수 있다.  
원하는 테마를 클릭하고 **Download** 버튼을 누르면 Github Repository 페이지로 이동하게 되는데, 해당 Repository를 `blog/themes` 디렉토리에서 clone 받으면 된다.  
(`hugo new site` 명령어에서 다른 이름으로 줬다면 해당 이름의 폴더 밑에서 themes 폴더를 찾아야 한다.)  
그리고 다운로드 받은 테마 페이지에 정리되어 있는 사용 방법에 따라, `config.toml` 파일을 수정해주면 된다.

### 만들어둔 Github Repository들과의 연동

가장 상위의 `blog` 디렉토리는 앞서 `blog`라는 이름으로 만들어두었던 Repository와 연동한다.

```sh
git init
git remote add origin https://github.com/<USERNAME>/blog
```

`blog` 디렉토리에서 `blog/public` 디렉토리를 submodule로 추가해주면서, `<USERNAME>.github.io` Repository를 연결한다.

```sh
git submodule add -b master https://github.com/<USERNAME>.github.io public
```

### 컨텐츠 작성 및 업로드

각 테마마다 포스팅할 글들이 위치할 디렉토리 구조가 다르므로, 테마 설명 페이지에서 가이드를 잘 읽어준 다음에 정확한 위치에 md 파일을 작성해야 한다.  
게시글 작성을 마쳤다면 `blog` 디렉토리에서 아래의 명령어를 통해 빌드한다.

```sh
hugo -t <THEME_NAME>
```

빌드된 파일들이 `blog/pubic`에 담기게 되며, 배포를 위해서는 코드를 `github.io` Repository에 push하기만 하면 된다.  
코드 관리를 위해 `blog` Repository 또한 push해주자.

## 3. 개선 사항들

- 나만의 로고 만들기!
- 블로그 컨텐츠들을 주제에 맞게 분류하기 위해 카테고리 설정
- About 페이지 대대적으로 손보기
