---
title: "[모각코]WebHacking(4강)"
excerpt_separator: "<!--more-->"
categories:
  - 모각코
tags:
  - 모각코
  - Hacking
---

## 07-14 수요일

## Client-side Advanced

- 취약점 방어 시 고려해야 할 사항이 상당수, 공격 및 우회 기법이 끊임없이 고안되어 왔고  
  방어를 위한 오픈소스 라이브러리 또한 무시히 많은 것이 특징

- **Cross-Site Scripting(XSS)**

  - 가장 확실한 방어 방법은 사용자가 HTML 태그나 엔티티 자체를 입력하지 못하도록 하고,  
    대신 입력을 서식 없는 평문으로 취급하는 것
  - 태그 및 속성 필터링

    - 코드를 실행할 수 있는 HTML 요소는 `<script>` 이외에도 상당수 존재
    - 요소의 속성값 또한 `&lt;` 와 같은 HTML 엔티티를 포함할 수 있어 본래 코드를 숨길 때에 사용될 수 있음

    - 대문자 혹은 소문자만을 인식하는 필터 우회

    ```
    x => !x.includes('script') && !x.includes('On')
    -->
    <scriPT>alert(document.cookie)</scriPT>
    --> <img src=x: oneRroR=alert(document.cookie) />
    ```

    - _JavaScript 함수 및 키워드 필터링_

      - JavaScript는 코드를 난독화할 수 있는 다양한 기능들을 포함하여 필터를 우회할 수 있음
      - `atob` 와 `decodeURI`함수는 각각 데이터를 디코딩 하는 함수로써 키워드 등을 부호화하여 필터를 우회할 수 있음

      - 예시

      |                     구문                     |                                  대체 구문                                  |
      | :------------------------------------------: | :-------------------------------------------------------------------------: |
      | alert, XMLHttpRequest 등 최상위 객체 및 함수 | window\['al' + 'ert'\], window\['XMLHtt' + 'pRequest'\] 등 이름 끊어서 쓰기 |
      |                    window                    |                                 self, this                                  |
      |                  eval(code)                  |                              Function(code)()                               |
      |                   Function                   |         isNaN\['constr' + 'uctor'\] 등 함수의 constructor 속성 접근         |

  - _문자열 치환_

    - 필터되는 문자열 사이에 또 다른 필터되는 문자열을 넣으면 최종적으로 바깥에 필터링 되는  
      문자열이 다시 나타나게 되어 필터가 무력화될 뿐만 아니라 방화벽 등에서 탐지하지 못하게 되는 부작용 발생
      ```
      (x => x.replace(/script/g, ''))('<scrscriptipt>alscriptert(documescriptnt.cooscriptkie)</scrscriptipt>')
      --> <script>alert(document.cookie)</script>
      ```

  - _활성 하이퍼링크_

    - `javascript:` 스키마는 URL 로드 시 자바스크립트 코드를 실행할 수 있도록 함
    - 정규화를 거치는 과정에서 `\x01\, \x04\` 와 같은 특수 제어 문자들이 제거될 수 있음
    - 이를 이용하면 다양한 우회 기법 사용 가능

  - _디코딩 전 필터링_

    - 일부 웹 응용은 웹 방화벽 등의 필터링 기능에 의존하거나, 데이터를 개별 요소를 추출하기 전에 전체 데이터에 필터를 가하는 경우가 있음
    - 예를 들어 XSS 필터 검사를하고 URL Decode 등을 가한다면 필터가 무용지물이 되어버림

  - _길이 제한_
    - URL의 Fragment 부분을 추출하여 eval()로 실행하는 기법이 흔히 쓰임
    - 쿠키에 페이로드를 저장하는 방식과 import와 같은 외부 자원을 스크립트로 로드하는 방법 또한 사용 가능

- **Content Security Policy(CSP)**

  - XSS 공격이 발생하였을 때 그 피해를 줄이고 웹 관리자가 공격 시도를 보고받을 수 있도록 하는 기술
  - 웹 페이지에 사용될 수 있는 자원에서 위치 등에 제약을 걸어 공격자가 웹 사이트에 본래 있지 않던 스크립트를 삽입하거나 공격자에게 권한이 있는 서버 등에 요청을 보내지 못하도록 막을 수 있음

  ```
  CSP 헤더는 1개 이상의 정책 디렉티브가 세미콜론으로 분리된 형태
  다음은 아래 URL에서만 로드되어야 함을 나타내는 CSP 헤더
  Content-Security-Policy:
  default-src 'self'
  https://example.dreamhack.io
  ```

  - _CSP 우회 방법_

    1. **신뢰하는 도메인에 업로드**

    - 해당 사이트에 스크립트 등을 업로드한 뒤 다운로드 주소로 대상 웹 페이지에 해당 자원을 포함시킬 수 있음
      - 이에 대한 해결책으로는 도메인 Origin 대신 해쉬나 nonce 등을 이용하는 방법

    ```
    <meta http-equiv="Content-Security-Policy"
    content="script-src 'self'">
    ...
    <h1>검색 결과: <script
    src="/download_file.php>id=177742"></script></h1>
    ```

    2. **nonce 예측 가능**

       - nonce를 이용하면 따로 도메인이나 해쉬 등을 지정하지 않아도 공격자가 예측할 수 없는 특정 nonce 값이 태그 속성에 존재할 것을 요구함으로써 XSS 공격을 방어할 수 있음
       - 이를 생성하는 알고리즘이 취약하여 결과 예측이 가능하면 공격자는 이를 유추해 자신의 스크립트를 웹 사이트에 삽입할 수 있음

    3. **base-uri 미지정**

    - HTML `<base>` 요소는 상대 경로가 해석되는 기준점을 변경할 수 있도록 하며 `<a>, <form> 등의 target` 속성의 기본값을 지정하도록 함
    - 공격자가 `<base href="https://malice.text/xss-proxy/">`와 같은 마크업을 삽입하게 되면  
      추후 상대 경로를 사용하는 URL들은 본래 의도한 위치가 아닌 공격자의 서버에 자원을 가리키게 되어 공격자는 이를 통해 임의의 스크립트 등을 삽입할 수 있게 됨

- **Cross-Site Request Forgery(CSRF)**

  - _CSRF 방어 시 주의점_

    - CSRF Token은 같은 Origin에서만 접근 가능한 형태로 특정 토큰을 저장해 제3자가 아닌 사용자로부터 요청이 왔다는 것을 인증할 수 있는 방법

      1. 짧은 CSRF Token
         - CSRF Token은 외부자가 예측 불가능하도록 설계된 만큼 무차별 대입 공격이 효과적이지 않도록 토큰의
      2. 에측 가능한 CSRF Token
         - 토큰의 길이가 충분하여도 공격자가 충분히 접근 가능한 데이터, 암호학적으로 안전하지 않은 의사 난수 생성기를 사용하게되면 토큰을 예측하는 것이 덜 어려워짐
      3. CSRF Token 유출
         - URL의 Query 파라미터로 넘겨지게 되면 이후 다른 링크를 방문하였을 때 Referer 헤더로 토큰이 그대로 노출되고, 공격자는 이를 역으로 사용자를 공격할 수 있음

- **Cross-Origin Resource Sharing(CORS)**

  - _취약점_

    1. 현재 사이트에서 다른 사이트로 정보 유출(기밀성)
       - 다른 사이트로 부터 CORS 요청을 받을 때 Origin에 대한 검사가 진행되지 않고 응답하거나 Origin에 제약이 없는 경우 사용자의 신원 등 민감한 정보가 노출될 수 있음
    2. 다른 사이트에서 현재 사이트 변조(무결성)
       - Origin이 신뢰할 수 있는 출처인지 확인 또는 제한하지 않거나 CORS 응답을 그대로 사용할 경우 XSS 등 보안문제가 발생할 수 있음

    - Window.postMessage API

      - Origin을 횡단하여 메세지를 주고받을 수 있는 API
      - 취약점
        1. Origin 미확인
           - 자유자재로 다른 윈도와 통신할 수 있도록 만들어진 API이기 때문에 Origin 검사 필요
        2. Origin 전환 경합 조건
           - 하이퍼링크를 방문하거나 스크립트가 다른 문서로 Redirect 시켜 들어있는 문서가 바뀐 상태에서  
             메시지를 보내면 Origin에 메세지가 누출되는 보안 문제 발생

    - JSONP(JSON with Padding)

      - CORS 기술이 도입되기 전 SOP를 우회하기 위해 흔히 쓰였던 방식
      - 취약점
        1. Origin 검사 부재로 인한 CSRF
        2. 콜백 함수명 검증 부재로 인한 제공자 XSS
        3. JSONP API 침해 사고 발생시 사용자 XSS

    - **CORS 정책**

      - 서버가 HTTP응답 헤더를 통해 직접 허용하고자 하는 Origin을 지정할 수 있도록 하는 기술
      - `OPTIONS` 메소드를 가징 예행 요청을 추가로 보내 판별
      - CORS 정책을 요청하는 클라이언트 헤더

      |            헤더 이름             |                                   설명                                   |
      | :------------------------------: | :----------------------------------------------------------------------: |
      | `Access-Control-Request-Headers` |  `OPTIONS` 요청이 끝나고 실제 요청을 보낼 때 포함될 헤더의 목록을 지정   |
      | `Access-Control-Request-Method`  | `OPTIONS` 요청이 끝나고 실제 요청을 보낼 때 사용될 HTTP 메소드 이름 지정 |

- **Exploit Techniques**

  - XSS 공격과 연계 공격

  1. **Relative Path Overwrite(RPO)**
     - 특정 URL의 하위 경로를 접근해도 같은 웹 페이지가 출력되는 것을 이용해 페이지에서 참조된 상대 경로 URL의 기준점을 바꾸는 공격
     - 방어할 수 있는 최선의 방법은 CSS, JS 등을 참조할 때 절대 경로를 사용하는 것
  2. **DOM Clobbering**
     - 메일 수신자, 게시글 작성자 등 제3자에 의해 HTML 마크업이 제공될 때, `id` 나 `name` 과 같은 속성을 이용하여  
       JS 에 접근할 수 있는 전역변수 공간 또는 객체 속성 공간 상에서 원하는 이름으로 임의의 DOM 객체를 삽입하는 공격
     - 효과정인 방어법은 간접적 메소드 호출 및 접근자를 사용하는 것
       ex) `HTMLFormElement.prototype.reset.call(elm)`
       하지만 번거로워서 잘 사용하지 않음
     - `id`, `name` 등 식별자 atribute를 제거할 수 있는 DOMPurify와 같은 라이브러리를 사용하는 것이 좋음
  3. **Template / DOM XSS**
     - JS에서 `innerHTML` 등 마크업을 해석하는 속성을 사용하거나 템플릿 라이브러리를 사용할 때  
       이를 제 3자가 제공 가능할 때 이를 이용해 스크립트 등을 삽입하는 공격
     - 이를 방지하려면 가급적 `innerHTML`와 같이 마크업을 해석하는 속성의 사용을 피해야 함
  4. **CSS Injection**
     - sytle태그 또는 style 속성에 대해서 변조 가능 시 사용할 수 있는 방법 중 하나
     - 다른 태그의 속성 값을 참/거짓의 방식을 통해 알아내거나, HTTP 트래픽을 생성시킬 수 있음
