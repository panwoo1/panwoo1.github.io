---
title: "[모각코]WebHacking(3강)"
excerpt_separator: "<!--more-->"
categories:
  - 모각코
tags:
  - 모각코
  - Hacking
---

## 07-14 수요일

## Sever-side Basic

- 서버에서는 사용자가 요청 한 데이터를 해석하고 처리한 후 사용자에게 응답
- 이 과정에서 사용자의 요청 데이터에 의해 발생하는 취약점이 서버사이드 취약점

- 주 목적
  - 서버 내에 존재하는 사용자들의 정보 탈취
  - 서버의 권한 장악

### Injection(인젝션)

- 사용자의 입력 값이 어플리케이션의 처리 과정에서 구조나 문법적인 데이터로 해석돼 발생하는 취약점

- **SQL Injection**

  - SQL을 사용할 때 공격자의 입력 값이 정상적인 요청에 영향을 주는 취약점
  - 웹 어플리케이션에서 DBMS에 저장된 정보를 조회하는 기능을 구현하기 위해 SQL 쿼리에  
    사용자의 입력데이터를 추가하여 DBMS에 요청  
     -> 이 과정에서 사용자의 입력이 SQL 쿼리에 삽입되어 SQL 구문으로 해석되거나 문법적으로 조작하게되면 임의의 쿼리가 실행될 수 있음
  - SQL의 사용 목적과 행위 따른 구분

    - _DDL(Data definition language)_
      - 데이터를 저장하기 위한 스키마, 데이터베이스의 생성/수정/삭제 등의 행위를 수행
    - _DML (Data manipulation language)_
      - 실제 데이터베이스 내에 존재하는 데이터에 대해 조회/저장/수정/삭제 등의 행위를 수행
    - _DCL (Data control language)_
      - 데이터베이스의 접근 권한 등의 설정을 하기 위한 언어

  - SQL쿼리 로그인 기능
    ```
    //간단한 형태의 쿼리
    //{아이디} {패스워드}에 사용자 입력값 삽입 및 DBMS 전달 '를 기준으로 문자열 구분
    select * from user_table
    where uid='{uid}' and upw='{upw}';
    ```
  - 실습

    - _or_ 연산은 A or B 와 같은 구조일 경우 A와 B 중 하나라도 참인 경우 결과는 참
    - 이를 이용하여 1 'or' 1 과 같은 공격 페이로드를 통해 _정보 출력 가능_

    ![SQL injection - 1](https://user-images.githubusercontent.com/66258691/125620311-8fe0711e-995f-4d20-912b-3b528be7bd7b.png)

    ![SQL injection - 2](https://user-images.githubusercontent.com/66258691/125620342-cc2aa5a2-7039-4678-982f-1d88d93958f1.png)

  - 취약점 방지
    - 사용자의 입력 데이터가 SQL 쿼리로 해석되지 않아야 함
    ```
    //문자열 구분자 '를 삽입하지 않고 SQL Injection을 통해
    //where 절의 조건 항상 참이 되도록 하는 예제
    select * from board_table where post_idx=100 or 1=1;
    ```
    - ORM과 같이 인증된 SQL 라이브러리를 사용하면 SQL Injection으로 부터 상대적으로 안전함
    - ORM(Object Relational Mapper)은 SQL의 쿼리 작성을 돕기 위한 라이브러리
    - ORM을 사용하더라도 입력 데이터의 타입 검증이 없으면 잠재적인 위협이 될 수 있음

- **Command Injection**

  - OS Command를 사용 시 사용자의 입력 데이터에 의해 실행되는 Command를 변조할 수 있는 취약점
  - Os Command를 사용할 때 사용자의 입력 값을 검증하지 않고 함수의 인자로 전달한다면  
     다음과 같은 문자를 이용해 사용자가 원하는 명령어를 함께 실행 가능

    - `` - 명령어 치환
    - $() - 명령어 치환(중복 사용)
    - && - 명령어 연속 실행
    - || - 명령어 연속 실행
    - ; - 명령어 구분자
    - | - 파이프(앞 명령어의 결과가 뒷 명령어의 입력으로 들어감)

  - 취약점 방지
    - 사용자의 입력 데이터가 Command 인자가 아닌 다른 값으로 해석되는 것을 방지
    - 웹 어플리케이션에서 OS Command를 사용하지 않는 것이 가장 좋은 방법
    - 불가피하게 사용해야할 경우 화이트/블랙 리스트 필터링 방식 사용

- **Sever Side Template Injection (SSTI)**

  - 템플릿 변환 도중 사용자의 입력 데이터가 템플릿으로 사용돼 발생하는 취약점
  - Template Engine이 실행하는 문법을 사용할 수 있기 때문에 발생하게 됨
    ![image](https://user-images.githubusercontent.com/66258691/125623297-f2031cb2-4dad-42c6-b0b7-a5b561967dcd.png)
  - 각 언어별 많이 사용되는 Template Engine

    |  Language  |      Template Engine      |
    | :--------: | :-----------------------: |
    |   Python   | Jinja2, Mako, Tornado ... |
    |    PHP     |     Smarty, Twig, ...     |
    | JavaScript |    Pug, Marko, EJS ...    |

  - 대부분의 Tempalte Engine 에서 {{}}, ${}과 같은 문법 지원

- **Path Traversal**

  - URL / File Path를 사용 시 사용자의 입력 데이터에 의해 임의의 경로에 접근하는 취약점
  - URI에서 해석되는 구분 문자

    | 문자 |                             의미                             |
    | :--: | :----------------------------------------------------------: |
    |  /   |                       Path identifier                        |
    |  ..  |      Parent directory \*/tmp/test/../1234 => /tmp/1234       |
    |  ?   |            Query identifier \*? 뒤는 query로 해석            |
    |  #   |   Fragment identifer \* # 뒤의 값은 Sever로 전달 되지 않음   |
    |  &   | Parameter separator \*key1=value&key2=value... 형식으로 사용 |

- **Sever Side Request Forgery (SSRF)**
  - 공격자가 서버에 변조된 요청을 보낼 수 있는 취약점
  - 취약점을 방지하기 위해서는 사용자가 입력한 URL의 Host를 화이트리스트 방식으로 검증

### File vulnerability

- 파일을 업로드/다운로드 하는 기능에서 발생할 수 있는 취약점 존재

- **파일 업로드 취약점**

  - 서버의 파일 시스템에 사용자가 원하는 경로 또는 파일 명 등으로 업로드가 가능하여 악영향을 미칠 수 있는 파일이 업로드되는 취약점
  - 공격자는 웹 어플리케이션 또는 서버의 서비스가 참조하는 파일을 업로드하여 공격에 사용 가능
    - CGI (Common Gateway Interface)는 사용자의 요청을 받은 서버가 동적인 페이지를 구성하기 위해  
      엔진에 요청을 보내고 엔진이 처리한 결과를 서버에게 반환하는 기능
  - 웹 어플리케이션이 실행하는 코드를 악의적인 공격자가 조작 가능하다면,  
    내장된 OS 명령어 등을 사용할 수 있으며, WebShell(웹쉘)이라는 악성코드 사용

- **파일 다운로드 취약점**
  - 서버의 기능 구현 상 의도하지 않은 파일을 다운로드 할 수 있는 취약점
  - 흔한 형태로 사용자가 입력한 파일이름을 검증하지 않은 채 그대로 다운로드를 제공하는 행위
  - ../ 상위 디렉토리 접근을 통해 파일 다운로드
  - 이를 막기 위해 ..과 / 와 \\을 적절하게 필터링 해야 함

### Business Logic Vulnerability(비즈니스 로직 취약점)

- 규칙에 따라 데이터를 생성·표시·저장·변경하는 로직, 알고리즘 등
- 서비스의 기능에서 적용되어야 할 로직이 없거나 잘못 설계된 경우 발생
- **Business Logic Vulnerability**

  - 정상적인 흐름에서 검증 과정의 부재 및 미흡으로 인해 정상적인 흐름이 악용되는 취약점
  - ex) 후기 이벤트 오류 - 글 작성 삭제를 반복하여 포인트 무한대로 획득
  - 비즈니스 로직 취약점을 방어하기 위해서는 로직을 확실하게 이해하고, 설계 및 개발 단계에서  
    어떤 위협이 발생할 수 있는지 위협을 파악하고 방어하는 것이 중요

- **IDOR (Insecure Direct Object Reference)**

  - 변조된 파라미터 값이 다른 사용자의 오브젝트 값을 참조할 때 발생하는 취약점
  - 사용자의 입력 데이터에 의해 참조하는 객체가 변하는 기능에서 사용자가  
    참고하고자 하는 객체에 대한 권한 검증이 올바르지 않아 발생
  - ex) 사용자의 금액을 저장하는 데이터베이스에서 계좌번호 조작, 금액 조회 및 송금
  - 취약점 방지
    - 객체 참조시 사용자의 권한을 검증하는 것이 중요
    - 사용자가 의도한 권한을 벗어나서 행동 할 수 없도록 권한 등을 분리해서 관리

- **Race Condition**
  - 비즈니스 로직의 순서가 잘못되거나, 한 오브젝트에 여러 요청이 동시에 처리되는 상황에서 발생하는 취약점

### Language specific Vulnerability(PHP, Python, NodeJS)

- 언어에서 제공하는 모든 함수가 안전한 것은 아님
- 의도한 기능을 제공하는 함수를 잘못 사용해서 문제가 발생할 수 있으며,  
  사용자의 입력 데이터가 함수의 인자로 사용되어 취약점 발생 가능

- **Common**

  - 코드 실행 함수(eval)
    - 사용자의 입력 데이터가 eval의 인자로 사용되지 않아야 함
  - OS command function
    - 사용자의 입력 데이터가 포함될 경우 Command Injection 취약점 발생 가능
    - Python
      - os.system, popen
      - subprocess.call, run, Popen
    - PHP
      - system, passthru
      - shell_exec, backtick operator(ex. `ls`)
      - popen, proc_open
      - exec
    - JavaScript (Node JS)
      - child_process.exec, spawn
  - Filesystem function

    - 대표적 서버의 피해
    - File Read - 어플리케이션 코드, 설정 파일 정보 등의 노출
    - File Write
      - WebShell 생성을 통한 원격 코드 실행 공격
      - 기존 설정 파일을 덮는 공격을 통해 운영체제 또는 어플리케이션 설정 변경
    - Etc
      - 파일 복사를 통해 File Write와 유사한 상황 발생
      - 설정 파일을 삭제하여 운영체제 또는 어플리케이션 서비스 무력화

  - serialize / deserialize

    - serialize(직렬화)는 Object 또는 Data의 상태 또는 타입을 특정한 형태의 포맷을 가진 데이터로 변환하는 것을 의미
    - deserialize(역직렬화)는 직렬화된 데이터를 원래의 Object 또는 Data의 상태 또는 타입으로 변환하는 것을 의미
    - 공격자는 역직렬화 과정에서 어플리케이션 상에서 다른 행위를 발생시키는 상태 또는 타입을 이용하여 악의적인 행위를 발생시키거나,  
      특정한 상황에서 호출되는 메소드들을 이용하여 공격에 사용

    - python 취약점 예시 코드

    ```
    import pickle
    import os

    class TestClass:
      def __reduce__(self):
        return os.system, ("id", )
    ClassA = TestClass()

    # ClassA 직렬화
    ClassA_dump = pickle.dumps(ClassA)
    print(ClassA_dump)

    # 역직렬화
    pickle.loads(ClassA_dump)
    ```

- **PHP**

  - _include_
    - php 태그가 포함될 경우 php코드가 실행되기 때문에 주의 해야함
  - _Wrappers_
    - 파일 시스템 함수에서 URL style 프로토콜을 위해 존재
    - 사용자의 입력이 될 경우 개발자의 의도와는 다른 행위를 발생 시킬 수 있음
  - _extract_
    - 배열에서 변수를 가져오는 함수
    - 기존에 사용되고 있는 변수의 데이터를 덮을 수 있어 신뢰할 수 없는 데이터가 사용되면  
      다른 변수를 변조하여 공격에 사용 가능
  - _Type Juggling_
    - 서로 다른 타입인 변수를 비교 또는 연산 시 자동으로 형변환이 발생하며 의도치 않은 결과 발생 가능
  - _Comparison_
    - 서로 다른 타입이 비교 연산자에 사용될 경우 type-juggling 에 의해 의도치 않은 결과 발생 가능
  - _session_
    - 사용자의 입력 데이터가 들어가게 되면 공격과 연계하여 사용 가능
    - ex) inclue함수의 인자를 조작 가능하고 인자에 대한 검증이 없는 페이지에서  
      세션 파일을 inclue 인자로 넘겨 원하는 php 코드 실행 가능
  - _Upload Logic_

    1. php 코드상에서 업로드 기능을 구현하지 않아도 사용자가 서버의 임시 디렉토리에 업로드 가능
    2. 예측 가능한 임시 파일 생성 규칙

    - 이러한 문제점을 통해 include 또는 파일 시스템에서 발생하는 취약점과 연계되면 서버의 명령어를 실행시키는 등의 공격을 연계될 수 있음

- **JavaScript**
  - Comparison Problem
    - 타입이 다른 두 변수의 값을 비교할 때 문제점 발생
      - 두 값 모두 객체가 아닐 경우 valueOf, toString 함수를 이용해 원시 값을 가져와 비교
  - Prototype Pollution
    - 객체를 생설할 때 초기화 할 프로토타입을 지정해주지 않으면 Object.prototype의 속성과 정의된 함수를 상속 받음
    - 상속받은 모든 객체는 메모리 상 같은 주소를 가리킴

### Misconfiguration

- 잘못된 설정 취약점은 모든 웹 어플리케이션 계층에서 발생 가능
- 잘못된 설정으로 공격자는 허가되지 않은 동작을 수행 할 수 있음

- **부주의로 인해 발생하는 문제점**

  - 권한 설정 문제
    - 잘못된 권한 설정
    - 기본 계정
  - 기본 서비스
    - 관리 서비스
    - 모니터링 서비스
  - 임시/백업 파일, 개발 관련 파일
    - 디렉토리 스캐너를 통해 파일 유출 가능

- **편의성을 위한 설정에 의해 발생하는 문제점**

  - 개발자를 위한 설정이 서비스 환경에서도 켜져있어 공격자가 이를 통해  
    시스템 내부의 정보를 알아내 추가적인 공격에 사용될 수 있음
  - DEBUG / Error Message Disclosure
  - 0.0.0.0으로 바인딩된 네트워크 설정

- **메뉴얼과 실제 구현체의 차이로 인해 발생하는 문제점**

  - NGINX ALIAS Path Traversal 취약점

- **해당 코드나 설정에 대한 이해 없이 사용해 발생하는 문제점**

  - 취약점이 존재하는 예제코드, StackOverflow 답변등을 이해 없이 복사 붙여넣기 해 사용하거나  
    메뉴얼에 존재하는 권고 설정을 무시한채 사용해 발생
  - 모든 도메인을 허용한 CORS 설정
