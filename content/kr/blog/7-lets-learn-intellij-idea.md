---
author: "Loko"
title: "IntelliJ IDEA를 공부해보자"
date: 2026-01-27
lastmod: 2026-01-27
description: "IDE 활용법 숙지를 통해 생산성 극대화하기"
tags: ["ide"]
thumbnail: /thumbnail/intellij.webp
toc: true
---

> 📢 본 포스트에서 소개되는 모든 단축키들은 Mac OS 기준으로 작성되었습니다.

## 0. 서론: 생산성과 직결되는 IDE 활용 능력

터미널에서 텍스트를 수정하는 방식의 vi를 지나, 1980년대 이후에는 Turbo Pascal, Visual Basic 등의 GUI 기반 IDE가 본격적으로 등장하기 시작했다.  
더 나아가 2000년대 이후에 등장한 VS Code, Eclipse, IntelliJ 등의 최신 IDE들은 '에디터' 수준에서 벗어나 코드 자동 완성, 실시간 오류 검사, 프로젝트 관리, 버전 관리 통합, 리팩토링 도구 등을 지원하며 IDE를 지능형 도구로 발전시켰다.  
현 시대의 IDE들에는 개발자의 생산성을 끌어올리기 위한 노하우들이 수십 년에 걸쳐 쌓여 있다.  
AI의 Code Generation 기능이 확장되고 있는 지금에 이르러서도 AI가 생산해내는 방대한 양의 코드를 편집하고, 세심하게 튜닝하기 위해서는 IDE 활용 능력이 필수적이다.  
그래서 오히려 지금 이 시기에 스스로의 IDE 활용 능력을 되짚어보고자 이 글을 작성하게 되었다.

## 1. IntelliJ IDEA에 대하여

<img src="/blog/about-intellij.png" class="hover-zoom">

IDE 중에서도 Java, Kotlin 언어로 개발하면서 가장 많이 활용했던 IntelliJ IDEA를 중점적으로 알아보고자 한다.  
2000년, 비효율적인 작업을 도와주는 **Renamer**가 먼저 출시되었으며, 이후 Java 및 Kotlin 개발에 필수적인 도구들을 제공하는 IntelliJ로 거듭나게 되었다.  
그 이후 JetBrains는 수십 년 동안 선도적인 IDE 공급업체로 성장하며 전 세계적으로 1,500만 명 이상의 사용자를 확보했다.  
또한 2009년에는 오픈 소스 라이센스가 부여된 **Community Edition**을 무료로 제공하며 IDE에 대한 접근성을 높였다.  
2014년에는 Domain-Specific Language를 지원하는 **MPS**를 도입했으며, 2019년에는 **Kotlin에 대한 지원**을 강화했다.  
그리고 2025.3 버전에서 **Community와 Ultimate의 통합**이 이루어졌으며, 핵심 기능들을 무료로 제공하고 고급 기능들은 구독이 필요한 방식으로 바뀌었다.

IntelliJ IDEA는 그 자체가 JVM에서 실행되며, OpenJDK 21 기반의 JetBrains Runtime이 번들에 내장되어 있다.  
기본적으로 Git과 통합되어서 버전 관리 기능을 제공하고, Docker와도 기본적으로 통합되어서 로컬 또는 원격으로 실행되는 Docker 엔진을 지원한다.

IntelliJ는 Java, Kotlin 외에도 Scala, Groovy, JavaScript, TypeScript, Rust 등 다양한 언어를 지원하며, 특히 Spring Boot라는 프레임워크에 대해 광범위한 지원을 제공한다.  
뿐만 아니라 데이터 처리를 위해 관련 도구들 및 SQL 플러그인을 제공하며, 이를 활용해 스키마를 탐색하고 편집할 수도 있다.

근래에는 **JetBrains AI Assistant**를 통해 AI 기반 자동화 기능들을 제공하는 데에도 주안점을 두고 있으며, 코드 베이스를 토대로 단위 테스트를 자동화하는 기능도 제공하고 있다.  
2025.1 버전에서 로컬 모델에 무제한 액세스하는 방식으로 구독에 대한 장벽 없이 AI를 무료로 사용할 수 있게 해주고 있다.

여기까지가 IntelliJ IDEA에 대한 개요였고, 다음 챕터부터는 IntelliJ의 핵심적인 Feature들을 하나씩 알아보도록 하자.

## 2. 생산성 극대화하는 방법들

IntelliJ의 모든 기능들을 두루 살펴보기 보다는 생산성을 끌어올리게 해주는 유용한 기능들을 중점적으로 알아보자.

### Live Templates

Live Template은 축약어를 입력하는 것만으로 코드 템플릿을 완성할 수 있게 해주는 기능이다.  
대표적으로 `psvm`과 `sout`가 있으며, 활용 예시는 다음과 같다.

<img src="/blog/psvm-sout.gif" class="hover-zoom">

사실 Live Template은 자주 사용되는 코드들을 직접 찾아내서 커스텀 템플릿을 마련해 두는 것이 생산성 측면에서 더 큰 효과를 볼 수 있다고 생각한다.  
때마침 필자는 근래에 [백준 온라인 저지](https://www.acmicpc.net/)에서 코딩 테스트 문제를 풀기 위해, 자동 완성 기능을 제공해주는 IntelliJ를 애용하고 있었다.  
근데 백준은 타 플랫폼과 다르게 파라미터를 함수 인자가 아닌, 프로그램에 대한 입력으로 받기 때문에 모든 문제를 풀 때마다 입력값에 대한 번거로운 처리가 필요하다.

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

백준 문제를 풀기 위해서는 `main` 메서드에 입력 실패 예외를 위한 `IOException` 처리, 입력을 받기 위한 `BufferedReader` 생성 과정이 모든 문제에 공통으로 필요하며, 공백으로 구분되어서 입력되는 경우에는 `StringTokenizer`의 활용도 필수적이다.  
그리고 연산을 진행할 때마다 결과값을 차례대로 저장해두기 위해 `StringBuilder`도 많이 사용된다.  
사실 Graph나 DP 등 여러 알고리즘 유형마다 배리에이션을 주면서 템플릿을 만들어 두는 것도 생산성 측면에서 큰 도움이 될 것이다.  
하지만 여기에서는 지면 관계상 기본 템플릿만 만들어 보도록 하자.

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

실제로 사용해보면 다음과 같다.

<img src="/blog/jboj.gif" class="hover-zoom">

여담으로, 기존 class 선언을 지우고 다시 템플릿을 적용해야 한다는 점이 다소 불편하게 느껴져서 좀 더 찾아보니 File Template이라는 기능을 확인할 수 있었다.  
이 기능을 활용하면 파일을 생성하는 시점에 템플릿을 바로 적용시킬 수 있어서 더욱 편리하다.

<img src="/blog/file-template.gif" class="hover-zoom">

### Postfix Completion

Code Completion에 대해서도 다뤄볼까 싶었지만, 코드를 입력하다 보면 자동적으로 제안해주는 기능이기 때문에 자연스러운 터득이 가능하다고 판단했다.  
그 대신에 공부해볼만한 이런저런 기능들을 찾아보다 보니, **Postfix Completion**을 알게 되었고 자주 사용해 보지 않았던 터라 가볍게 알아보면 좋겠다는 생각이 들었다.  
Postfix Completion은 말그대로 '접미어'를 기반으로 자동 완성을 해주는 기능인데, 변수명을 먼저 입력하고서 마치 체이닝 메서드를 활용하듯이 자동 완성 기능을 활용할 수 있게 해준다.  
"이 변수를 가지고 for문을 만들어줘" 라고 명령하는 듯한 느낌이라서 개발자의 의식의 흐름대로 로직을 작성할 수 있게 되는 것이다.  
참고로 Live Template에도 있던 `fori` 와 `sout` 같은 약어들을 Postfix Completion에서도 쓸 수 있다.

<img src="/blog/postfix-completion.gif" class="hover-zoom">

### Bookmark

<img src="/blog/starcraft-2.webp" class="hover-zoom">

뜬금 없는 이야기이지만 RTS 게임을 하다보면 '부대 지정'이라는 기능을 통해 유닛들이나 건물들을 역할에 따라 그룹화하고, 필요한 순간에 그룹별로 적절한 명령을 빠릿빠릿하게 내릴 수 있다.  
IntelliJ에서도 이와 유사한 기능을 제공하는데, 바로 **Bookmark** 기능이다.

<img src="/blog/bookmark.png" class="hover-zoom">

Bookmark의 종류에는 **Anonymous**와 **Mnemonic**이 있다.  
Anonymous는 원하는 만큼 Bookmark를 만들 수 있다는 장점이 있지만, 지정된 Bookmark를 다시 찾아갈 때는 Bookmark들을 리스트업하고 원하는 것을 찾아서 들어가야 한다.  
반면에 Mnemonic은 단축키로 바로 이동이 가능하지만 숫자 0 ~ 9까지, 문자 A ~ Z까지만 지정이 가능하다.  
개인적으로 생각하기에는 직접 기억하지 못할 Anonymous Bookmark를 무분별하게 지정해두기 보다는 정말로 자주 읽게 되는 코드들을 직접 선정해서 Mnemonic Bookmark를 지정해두는 것이 생산성 측면에서 훨씬 이득일 것이라고 생각한다.

Bookmark 기능들을 활용하기 위해 꼭 기억해둬야 할 단축키들은 다음과 같다.

- Bookmark 리스트업: `⌘Cmd` `2`
- Anonymous Bookmark 생성: `F3`
- Mnemonic Bookmark 생성: `⌥Option` `F3`
- 지정한 Mnemonic Bookmark로 이동: `⌃Ctrl` `(지정한 mnemonic)`

## 3. 디버깅을 더욱 꼼꼼하게

부끄럽게도 나는 꼼꼼히 디버깅을 하기 보다는 결과만 빠르게 확인하는 쪽을 택해 왔다.  
하지만 이러한 습관이 고착화되면 프로그램의 작동 방식을 섬세하게 관찰하지 못하게 된다.  
더군다나 근래에는 AI codegen에 대한 의존도가 높아지고 있으니 꼼꼼한 디버깅 역량을 길러두는 것이 더욱 유용할 것이다.

<img src="/blog/intellij-debug.png" class="hover-zoom">

IntelliJ의 디버그를 활용할 때에는 다음 4가지 기능을 꼭 기억해두자.

- **Resume**
- **Step Over**
- **Step Into**
- **Step Out**

### Resume

- 기능: 다음 breakpoint로 이동
- 단축키: `⌥Option` `⌘Cmd` `R`

### Step Over

- 기능: 바로 다음 라인으로 이동
- 단축키: `F8`

### Step Into

- 기능: 해당 라인에서 호출하고 있는 메서드 안으로 이동
- 단축키: `F7`

### Step Out

- 기능: Step Into 이후 다시 메서드 밖으로 이동
- 단축키: `⇧Shift` `F8`

## 4. Git, IDE 안에서 활용해보자

사실 나는 버전 컨트롤을 해야 할 때 [Fork](https://git-fork.com/)라는 GUI 클라이언트를 애용하고 있다.  
인턴을 하던 중에 한번 경험해 봤다가 지금까지 쭉 활용하게 되었다.  
개발자로서 편리한 도구들을 활용하는 것은 좋지만, 사실 별개의 인터페이스로 화면을 전환해서 이런저런 기능들을 수행하다 보면 생각보다 타임 로스가 많이 생긴다.  
그러므로 IntelliJ의 Git 관련 기능을 활용하면 개발하는 도중에도 더욱 빠르게 버전 컨트롤을 수행할 수 있게 될 것이다.

몇 가지 단축키를 외워두고 꼭 시의적절하게 활용해보자.

### Version Control

- 기능: Git 패널 열기
- 단축키: `⌘Cmd` `9`

### Show Diff

- 기능: 해당 브랜치 및 커밋에서 변경 내역 확인
- 단축키: `⌘Cmd` `D`

### VCS Operation

- 기능: 작업할 수 있는 Git 커맨드 한번에 보기
- 단축키: `⌃Ctrl` `V`

### Commit & Push

- Commit 단축키: `⌘Cmd` `K`
- Commit & Push 단축키: `⌥Option` `⌘Cmd` `K`
- Push 단축키: `⇧Shift` `⌘Cmd` `K`

## 5. 돌고돌아 단축키

이 블로그 주제를 떠올렸을 때, 그저 단축키만 나열하는 고리타분한 글만은 피하고 싶었다.  
하지만 IDE를 활용하면서 생산성을 극대화하고자 한다면 결국에는 단축키에 대한 암기가 꼭 필요하다는 사실을 깨달았다.  
그런 의미에서 본 포스팅의 마지막은 IntelliJ의 핵심 단축키들에 대한 정리와 함께 끝마치려 한다.

### 액션 찾기

- 기능: IntelliJ로 수행할 수 있는 모든 액션을 탐색하는 검색창 열기
- 단축키: `⇧Shift` `⌘Cmd` `A`

### 신규 생성

- 기능: 커서 위치에 따라 패키지 생성, 파일 생성, 메서드 생성 등 생성 가능한 것들 리스트업
- 단축키: `⌘Cmd` `N`

### 실행 관련

- 현재 포커스에서 실행 단축키: `⌃Ctrl` `⇧Shift` `R`
- 디버그 실행 단축키: `⌃Ctrl` `⇧Shift` `D`
- 현재 실행 환경(상단 셀렉트 박스에서 선택되어 있는 것) 실행 단축키: `⌃Ctrl` `R`

### 라인 편집

- 라인 복제: `⌘Cmd` `D`
- 라인 삭제: `⌘Cmd` `⌫Del`
- 문자열 라인 합치기: `⌃Ctrl` `⇧Shift` `J`

### 라인 이동

- 블록에 관계 없이 라인 이동: `⌥Option` `⇧Shift` `↑↓`
- 블록 안에서만 라인 이동: `⇧Shift` `⌘Cmd` `↑↓`
- 엘리먼트 간 좌우 이동: `⌥Option` `⇧Shift` `⌘Cmd` `←→`

### 커서 이동

- 커서를 단어별로 이동: `⌥Option` `←→`
- 단어별로 선택하면서 이동: `⌥Option` `⇧Shift` `←→`
- 라인의 맨앞뒤로 이동: `fn` `←→`
- 라인 전체를 선택하면서 이동: `⇧Shift` `fn` `←→`
- 페이지 up/down: `fn` `↑↓`

### 코드 확인

- 생성자 및 메서드에 어떤 인자가 필요한지 확인: `⌘Cmd` `P`
- 메서드의 구현 부분을 팝업으로 확인: `⌥Option` `Space`
- Javadoc 확인: `fn` `F1`

### 포커스 선택

- 포커스 범위 확장/축소: `⌥Option` `↑↓` (위로 갈수록 확장, 아래로 갈수록 축소)
- 이전/이후 포커스 위치로 이동: `⌘Cmd` `[` `]`
- 멀티 포커스: `⌥Option` `⌥Option` `↑↓`
- 오류 포인트로 바로 이동: `F2`

### 검색

- 현재 파일 내에서 탐색: `⌘Cmd` `F`
- 검색한 문자열 교체: `⌘Cmd` `R`
- 프로젝트 전체에서 탐색: `⇧Shift` `⌘Cmd` `F`
- 프로젝트 전체에서 교체: `⇧Shift` `⌘Cmd` `R`
- 파일 검색: `⇧Shift` `⌘Cmd` `O` (패키지명 포함 검색 가능)
- 심볼 검색: `⌥Option` `⌘Cmd` `O` (메서드 검색할 때 자주 사용)
- 최근 열었던 파일들 리스트업: `⌘Cmd` `E`
- 최근 수정했던 파일들 리스트업: `⇧Shift` `⌘Cmd` `E`
- 사용 위치 찾기: `⌥Option` `F7`
- Structure View: `⌘Cmd` `7`

### 자동 완성

- 스마트 자동 완성: `⌃Ctrl` `⇧Shift` `Space`
- static 메서드 자동 완성: `⌃Ctrl` `Space` `Space`
- 구현해야 할 상속 메서드들 자동 완성: `⌃Ctrl` `I`
- Live Template 리스트업: `⌘Cmd` `J`

### 리팩토링

- 변수 추출: `⌥Option` `⌘Cmd` `V`
- 파라미터 추출: `⌥Option` `⌘Cmd` `P`
- 메서드 추출: `⌥Option` `⌘Cmd` `M`
- inner 클래스 추출: `F6`
- 변수명, 클래스명, 메서드명 일괄 변경: `⌃Ctrl` `F6`
- 타입 일괄 변경: `⇧Shift` `⌘Cmd` `F6`

### 코드 정리

- 사용하지 않은 import 제거: `⌃Ctrl` `⌥Option` `O` (Auto Import 옵션을 켜면 자동으로 정리됨)
- 자동 정렬: `⌥Option` `⌘Cmd` `L`
