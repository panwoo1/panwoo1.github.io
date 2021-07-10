---
title: "2장 웹 해킹(2.5장까지) & WebHacking Lecture"
excerpt_separator: "<!--more-->"
categories:
  - 모각코
tags:
  - 모각코
  - Hacking
---

### 1. **개요**

- 웹 어플리케이션이 활성화되면서 중요 서비스들의 다수가 웹을 통해 이뤄지고 있음
- 수많은 웹 어플리케이션이 공격에 노출되어 있고 대규모 개인정보 유출 등 크고 작은 웹 해킹 사고들이 끊임없이 발생

### 2. **사례**

- DDoS
- 파일 업로드 공격
- 웹 쉘
- 구글 해킹(검색을 이용한 해킹)
- 크로스 사이트 스크립트 웜 배포

### 3. **실습 환경**

- APM(Apache, PHP, MYSQL)
- Web Proxy tool
  - parosproxy = 기존에 가장 유명하면서 많이 쓰이는 프로그램이지만 현재는 수년간 업데이트가 이뤄지지 않아 쓰이지 않음
  - 현재는 Burp Suit, OWASP-ZAP, Acunetix 등이 주류로 쓰임

### 구글 해킹

- 구글 자체 해킹이 아닌 구글 검색을 활용한 해킹
- 구글은 방대한 사이트 자료를 갖고 있어 여러 검색 고급 연산자를 지원함  
  구글이 지원하는 검색 부울 연산자와 검색 연산자 다음과 같음

  1. **검색을 이용한 공격**

  - 검색 연산자

    | 연산자 |               사용 예                |                            설명                             |
    | :----: | :----------------------------------: | :---------------------------------------------------------: |
    |  AND   |  "가"&"나", "가""나", "가" AND "나"  |       "가"와 "나"의 문자열을 모두 포함하는 문서 검색        |
    |   OR   |       "가"\|"나", "가" OR "나"       |      "가" 혹은 "나"의 문자열을 모두 포함하는 문서 검색      |
    |  NOT   | "가나다"-"한글", "가나다" NOT "한글" | "가나다" 문자열을 포함하고 "한글" 문자열을 제외한 문서 검색 |
    |   ""   |        "security conference"         |       인용부호를 사용해 정확한 단어 혹은 문구를 검색        |
    |   ~    |               ~hacking               |             동의어 또는 관련 검색어와 함께 검색             |
    |   \*   |           \* 모아 \* 이다            |  알 수 없는 단어가 위치한 부분에 "\*" 연산자를 사용해 검색  |
    |   ..   |          월드컵 1950..2000           | 숫자 사이에 ".."를 넣어 가격, 수치와 같이 범위 문서를 검색  |

  - 구글 검색 고급 연산자

    |  연산자   |       사용 예       |                  설명                  |
    | :-------: | :-----------------: | :------------------------------------: |
    |  inurl:   |      inurl:abc      |    주소에 "abc"가 들어간 문서 검색     |
    | intitle:  |    intitle:해킹     |   Title에 "해킹" 이 들어간 문서 검색   |
    |  intext:  |    intext:로그인    |   본문에 "로그인"이 들어간 문서 검색   |
    |   Site:   |   site:http://abc   |       http://abc 문서에서의 검색       |
    |   link:   |   link:http://abc   |   http://abc의 링크가 걸린 문서 검색   |
    | inanchor  | Inanchor:http://abc | http://abc가 텍스트로 표현된 문서 검색 |
    | filetype: |    filetype:jpg     |            jpg 확장자 검색             |
    |  cache:   |  cache:http://abc   |  구글에 저장된 http://abc 페이지 보기  |
    | numrange: | numrange:1000-2000  |   1000-2000 숫자 범위의 결과를 검색    |

    1. **백업 파일의 노출**

       - 대부분의 편집용 프로그램에서 기본적으로 제공하는 백업 기능은  
         특정 파일이 편집 중이거나 삭제되었을 때 백업 파일 저장
       - 홈 폴더에 관리자의 실수로 백업 파일을 남겨놓는 경우

    2. **관리자 페이지 권한의 실수, 노출**

       - /admin/ 이나 /manager/와 같은 관리자 페이지의 경우 "site:xxx.com, inurl:admin" 같은 쿼리를 통해  
         관리자 페이지를 찾아 낼 수 있음

    3. **기밀 업로드 파일의 관리 실수**

       - 파일 업로드 기능이 있는 게시판의 경우 폴더에 업로드한 파일을 저장
       - 만약 비밀 글이 있고 파일이 첨부되어 있다면, "inurl:/board/data/, site:xxx.com"과 같은 쿼리를 통해
         비밀 글의 첨부 파일을 받아 볼 수 있음

    4. **디렉터리 리스팅**

       - 웹 서버의 폴더 요청 시 index 파일이 존재하지 않을 경우 파일 목록을 출력해주는 Apache의 옵션
       - 환경에 따라 기본 옵션이 켜져 있는 경우가 많아 중요 정보 파일이 노출되는 사례가 많음

    5. **웹 쉘 노출**

       - 웹 쉘(웹 해킹에 자주 쓰이는 백도어)을 찾아낸다면, 해당 서버는 공격을 거치지 않고 구글 해킹만으로 점령 가능
       - 유명 웹 쉘 등이 많이 사용되기 때문에 특정 문자열을 이용해 검색 쿼리를 제작하면, 많은 서버 획득 가능

    6. **실습**

    ![구글 검색](https://user-images.githubusercontent.com/66258691/125149571-ae338500-e174-11eb-921b-ee813a3c3638.png)

    ![구글검색2](https://user-images.githubusercontent.com/66258691/125149595-d28f6180-e174-11eb-9951-e61383b1540f.png)

  2. **구글 해킹 도구**

     - 검색 패턴 데이터 베이스 사이트 http://www.hackersforcharity.org/ghdb/
       ![ghdb](https://user-images.githubusercontent.com/66258691/125149737-a58f7e80-e175-11eb-8f19-9148b0f2bdac.png)

  3. **방어 기법**
     - 특정 웹 사이트를 수집할 때 무분별한 수집을 방지하기 위한 '로봇 배제 표준' 규약

### 파일 업로드

- 이 취약점을 통해 공격하게 되면 해커는 시스템 명령어 실행 권한을 획들할 수 있음
- 공격 방법도 타 공격 방법에 비해 쉬운 편

  1. **웹 쉘 제작**

     - 웹 쉘 업로드 공격이 성공한 후에 해커들은 데이터베이스르 접속하는 PHP 파일을 열람  
       DB 계정을 획득하고, 개인정보를 탈취

  2. **취약점 공격**

     - 대부분의 게시판들은 사용자가 원하는 자료나 이미지를 올릴 수 있도록 구현됨
     - 취약점 공격은 이런 특성을 이용해 공격하는 방식

  3. **파일 업로드 우회 기법**

     - php.kr 우회 기법

       - Apache의 AddLanguage 옵션에 의해 발생, 파일명이 vuln.php.kr 일 경우에도 PHP파일이 실행됨
       - 해당 취약점은 확장자가 php가 아니므로 필터링을 우회할 수 있음

     - .htaccess 업로드 취약점

       - Apache 설정 파일인 .htaccess 를 업로드하게 되면 여러 아파치 옵션 변경 가능
       - *Addtype*과 같은 옵션을 통해 txt, jpg 등 원하는 확장자를 PHP코드로 실행 가능

     - 환경 취약점

       - 윈도우 환경에서는 파일 업로드시 콜론을 붙일 경우 그 뒤에 문자열 삭제되어 업로드
       - abc.php:jpg 파일을 업로드 할 경우 abc.php 파일이 업로드 됨

  4. **파일 업로드 방어 기법**

     - 업로드 파일명의 확장자를 파싱해서 php,htm 등 PHP 실행 권한을 가진 확장자를 필터링
     - 하지만 이는 몇 가지 우회 기법과 또 다른 공격 방법 혹은 코드 실수 등을 통해 취약점이 발생할 확률이 높음

     1. "php_value engine off" 내용의 .htaccess 파일을 업로드 폴더에 생성
     2. .htaccess 파일을 사용자가 업로드하지 못하도록 필터링

---

## Web Hacking Lecture 1 & 2

### 웹이란?

- 해킹 - 본래의 의도와는 다른 행위를 발생시키는 것
- 웹 - HTTP를 이용하여 정보를 공유하는 통신 서비스
- 웹 서버(Web Server) - 서비스를 제공하는 대상
- 웹 클라이언트(Web Client) - 서비스를 받는 사용자

### 웹 기초 지식

- Web Browser - 웹에 접속하기 위해 사용하는 소프트웨어
- Web Resource - 웹 상에 존재하는 모든 콘텐츠(HTML, CSS, JS, PDF, PNG 등)
- URI(URL) - Uniform Resource Identifier의 약자로 리소스를 식별하기 위한 식별자
- HTTP(HyperText Transfer Protocol) - 인터넷 서비스 대상 간 통신 규약, 웹을 이용하기 위한 통신 규약
- HTTPS(HyperText Transfer Protocol Secure) - 기존 HTTP 데이터를 암호화하여 통신
- Cookie - 웹 브라우저에 저장하는 데이터
- Session - 서버에 저장하는 데이터
- Domain Name - 네트워크상에서 컴퓨터를 식별하는 이름 (ex. www.naver.com은 네이버의 서버 컴퓨터를 식별하는 이름)
- Server - 인터넷상에서 사용자에게 서비스를 제공하는 컴퓨터
- Web Server - Web Browser 와 HTTP를 이용하여 통신하는 서버
- Application - 서버에서 설정한 특정 기능들을 수행하는 소프트웨어
- DataBase(DB) - 데이터를 저장하기 위해 사용하는 데이터 저장소

### Web Browser

|                        | Chrome |      Edge      |     Safari      |    Firefox    |
| :--------------------: | :----: | :------------: | :-------------: | :-----------: |
|         제조사         |  구글  | 마이크로소프트 |      애플       |  모질라 재단  |
|    JavaScript 엔진     |   V8   |   ChakraCore   | JavaScript Core | Spider Monkey |
|     HTML/CSS 엔진      | Blink  |   Edge HTML    |     Webkit      |     Gecko     |
| 사생활 보호/보장(EDNS) |   X    |       X        |        X        |       O       |
|    패스워드 동기화     |   O    |       O        |        O        |       O       |

### Web Resource

- 웹 리소스를 가리키는 주소는 URL.
- 대표적인 웹 리소스
  1. HTML: 웹 문서의 뼈대를 구축하기 위한 마크업 언어
  2. CSS: HTML이 표시되는 방법을 정의하는 스타일 시트 언어
  3. JS: Client Side Script를 이용하여 페이지 내에서의 행위들을 설정
  4. Etc: 문서, 이미지, 동영상, 폰트 등

### URI(URL)

- 리소스를 식별하기 위한 식별자
- URL은 리소스의 위치를 식별하기 위한 URI의 하위 개념

  <img width="960" alt="c974f9087eddea560f7ff225c5d6e35a0cfdfd1a1df14c284271ff321846e497" src="https://user-images.githubusercontent.com/66258691/125150288-ac1ff500-e179-11eb-9d78-89cd7b507753.png">

  - Scheme - 웹 서버에 접속할 때 어떤 체계를(프로토콜)를 이용할지에 대한 정보
  - Host - Authority의 일부로써 접속할 웹 서버의 호스트에 대한 정보
  - Port - Authority의 일부로써 접속할 웹 서버의 포트에 대한 정보
  - Path - 접속할 웹 서버의 경로에 대한 정보를가지고 있으면 /로 구분
  - Query - 웹 서버에 전달하는 파라미터이며 URI에서 ? 문자 뒤에 붙음
  - Fragment - 메인 리소스 내에 존재하는 서브 리소스에 접근할 때 이를 식별하기 위한 정보,  
    URI에서 # 문자 뒤에 붙음
    ```
    ex) http://example.com:80/path?search=1#fragment
    Scheme = http://
    Host = example.com
    Path = /path
    Query = ?search=1
    Fragment = #fragment
    ```

### Encoding

- Encoding - 문자 또는 기호 등의 정보, 형태를 표준화, 보안 등의 목적으로 다른 형태나 형식으로 변환하는 처리  
   혹은 그 처리 방식
- Decoding - 변환된 형태를 원래 형태로 변경하는 것
