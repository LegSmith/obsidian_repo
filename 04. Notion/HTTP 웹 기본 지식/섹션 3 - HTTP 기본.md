## HTTP : Hyper Text Transfer Protocal

---

- **==현재 모든 전송은 HTTP==** 로 한다.
- HTTP 는 _IMAGE , 음성, 영상, 파일_을 다 전송한다.
- **==JSON, XML(API) 형태의 데이터 전송==**이 가능하다.

### HTTP 역사

- 1991년 HTTP/0.9 : GET 메서드만 지원, HTTP 헤더가 없다.
- 1996년 HTTP/1.0 : HTTP 메서드, 헤더 추가
- ==**1997년 HTTP/1.1**== **: 가장 많이 사용, 우리에게 가장 중요한 버전**
- 2015년 HTTP/2 : HTTP/1.1 에서 성능을 개선한 버전
- HTTP/3 ~진행중 : TCP 기반이 아닌 UDP 기반으로 사용, 성능 개선

### 기반 프로토콜

- `TCP` : HTTP/1.1, HTTP
- `UDP` : HTTP/3
- **==현재는 HTTP/1.1 을 주로 사용==**하고 HTTP/2, HTTP/3 도 점점 증가한다.

> _**HTTP/1.1 을 제대로 안다면 상위버전은 성능만 개선한 것이므로 금방 익힐 수 있다.**_

## 클라이언트 서버 구조

---

![[Untitled 36.png|Untitled 36.png]]

- _**Requet↔Response 구조**_이다.
- ==**보통 Request(요청)하는 쪽이 클라이언트이고 Response(응답)하는 쪽이 서버이다.**==

## 무상태 프로토콜

---

_**무상태 프로토콜이란 스테이스리스(Stateless) 를 뜻한다.**_

### Stateful ↔ Stateless

- stateful(상태유지) 는 클라이언트의 정보를 서버가 항상 알고 있다.
- stateless(무상태)는 서버가 클라이언트의 정보를 모른다.
- **==즉, Stateful 은 중간에 서버가 장애가 나면 다른 서버는 클라이언트의 정보를 모르기 때문에 장애가 난다.==**

    ![[Untitled 1 26.png|Untitled 1 26.png]]

- **==stateless 는 중간에 서버가 장애가 나도 다른 서버로 대체해도 된다.==**

    ![[Untitled 2 16.png|Untitled 2 16.png]]


### Stateless 실무 한계

- 모든 것을 무상태로 설계 할 수 있는 경우도 있고 없는 경우도 있다.
- 로그인이 필요 없는 단순한 서비스는 `Stateless`로 해도 된다.
- 로그인이 필요한 서비스는 `Stateful` 로 해야 한다.
- **==일반적으로 브라우저 쿠키와 서버 세션등을 사용해서 상태 유지==**
- ==**상태 유지는 최소한만 사용하는게 좋다**==

## 비 연결성(connectionless)

---

- 연결을 유지하는 모델은 **==여러개의 클라이언트의 요청을 한개의 서버가 계속 연결==**을 하고 있어서 바로 바로 응답이 가능하다. 단, ==**서버 자원이 소모**== 된다.

    ![[Untitled 3 10.png|Untitled 3 10.png]]

- 연결을 유지하지 않는 모델은 **==서버가 응답을 해주면 TCP/IP 연결을 종료==**한다.

    ![[Untitled 4 8.png|Untitled 4 8.png]]


### 비 연결성

- ==**HTTP는 기본이 연결을 유지하지 않는 모델**==
- 1시간 동안 수천명이 서비스를 사용해도 실제 서버에서 동시에 처리하는 요청은 수십개 이하로 매우 작다
- ==**서버 자원을 매우 효율적으로 사용할 수 있다.**==

### 비 연결성 한계와 극복

- **==클라이언트가 다시 요청을하면 TCP/IP 연결을 새로 맺어야 한다. → 3 way handshake 시간 추가==**
- 웹 브라우저로 사이트를 요청하면 HTML, JS, css, 추가 이미지등 수많은 리소스가 함께 다운로드
- ==**HTTP 지속 연결(Persistent Connections)**==로 문제 해결

    ![[Untitled 5 7.png|Untitled 5 7.png]]


## HTTP 메시지

---

_**HTTP 메시지는 기본적인 구조가 있고 요청시와 응답시에 조금의 차이가 있다.**_

### HTTP 메시지 기본 구조

- `start-line`(시작 라인), `header`(헤더), `empty line`(공백 라인), `message body` 로 구성 된다.

    ![[Untitled 6 6.png|Untitled 6 6.png]]

- RFC 7230 에 따른 공식 스펙으로는 아래와 같다.

    ```html
    HTTP-message = start-line
    							 *( header-field CRLF )
    							 CRLF
    							 [ message-body ]
    ```

- 요청메시지는 기본적으로 message body 가 없다.(body를 가지는 경우도 있다.)

    ![[Untitled 7 4.png|Untitled 7 4.png]]

- 응답 메시지에는 요청메시지 시작라인과 차이가 있다.

    ![[Untitled 8 2.png|Untitled 8 2.png]]


### HTTP 시작 라인

- 요청메시지 경우에는 ==**시작 라인에 HTTP메서드 요청대상 HTTP 버전**==이 들어간다.(`request-line`)

    - `method` : HTTP 메서드(GET, POST, PUT, DELETE …)
    - `SP` : 공백
    - `request-target` : 요청 대상(절대 경로가 들어간다)
    - `HTTP-version` : HTTP 버전
    - `CRLF` : 엔터

    ```html
    request-line = method SP request-target SP HTTP-version CRLF
    ```

- 응답메시지 경우에는 ==**시작 라인에 HTTP 버전, HTTP 상태코드, 이유 문구**==가 들어간다.(`status-line`)

    - `status-code` : HTTP 상태코드
        - 200 : 성공
        - 400 : 클라이언트 요청 오류
        - 500 : 서버 내부 오류
    - `reason-phrase` : 이유문구, 사람이 이해할 수 있는 짧은 상태 코드 설명 글

    ```html
    status-line = HTTP-version SP status-code SP reason-phrase CRLF
    ```


### HTTP 헤더

- `field-name` 은 대소문자 구분이 없다
- `OWS` 는 띄어쓰기 허용이다.

    ```html
    header-field = field-name ":" OWS field-value OWS 
    ```

- `filed-name` 바로 옆에는 `OWS`가 없으므로 : 를 붙혀쓰고 : 다음에 띄어쓰기가 허용이 된다.

    ![[Untitled 9 2.png|Untitled 9 2.png]]

- HTTP 전송에 필요한 모든 부가정보를 HTTP 헤더에 적어준다. 즉, **==HTTP 메시지에 메타 데이터들을 HTTP 헤더에 적어준다고 보면 된다.==**

### HTTP 메시지 바디

- 실제 전송할 데이터를 적어준다.
- HTML 문서, 이미지, 영상, JSON 등등 byte로 표현할 수 있는 모든 데이터 전송 가능

    ![[Untitled 10 2.png|Untitled 10 2.png]]

## 관련 문서

- 상위 목차: [[HTTP 웹 기본 지식]]
- 이전 문서: [[섹션 2- URI]]
- 다음 문서: [[섹션 4 - HTTP 메서드]]
