# URI 와 URL

---

`URL(Uniform Resource Locator)` , `URN(Uniform Resource Name)` 은 `URI(Uniform Resource Identifier)` 안에 포함되어 있다.

### URI 뜻

- `Uniform` : **==리소스를 식별하는 통일된 방식==**
- `Resource` : ==**자원**==, _**URI 로 식별할 수 있는 모든**_ 것
- `Identifier` : **==다른 항목과 구분==**하는데 필요한 정보

### URL , URN 뜻

- `URL(Locator)` : 리소스가 있는 위치를 지정
- `URN(Name)` : 리소스에 이름을 부여

## URL 구조

**scheme://[userinfo@]host[:port][/path][?query][\#fragment]**

→ `https://www.google.com:443/search?q=hello&hl=ko`

- **userinfo**
    - URL 에 유저정보를 담아서 전송
    - 잘 사용하지 않는다.
- `host`
    - 호스트명
    - 도메인명 또는 IP 주소를 직접 사용가능
- `port`
    - 접속 포트
    - 일반적으로 생략가능, HTTP 는 80 HTTPS 443
- `path`
    - 리소스 경로, **==계층적 구조==**로 되어있다.
- `query`
    - key=value 형태
    - ? 로 시작하고, &로 추가 가능하다.
    - query parameter , query string 등으로 불린다.
- **fragment**
    - html 내부 북마크 등에 사용
    - 잘 사용하지 않는다.

# 웹 브라우저 요청 흐름

---

_**웹 브라우저에서 URL 을 입력하여 엔터를 누르면 URL 을 분석하여 서버를 찾고, HTTP 요청 메시지를 생성해 서버로 요청한다.**_

![[Untitled 35.png|Untitled 35.png]]

## HTTP 메시지 전송

![[Untitled 1 25.png|Untitled 1 25.png]]

1. 웹 브라우저가 **==HTTP 메시지 생성==**
2. SOCKET 라이브러리를 통해 전달
    1. **==TCP/IP 연결(IP, PORT)==**
    2. 데이터 전달
3. **==TCP/IP 패킷 생성==**, HTTP 메시지 포함
4. LAN 에서 인터넷을 통해 서버로 전달

## HTTP 메시지 응답

![[Untitled 2 15.png|Untitled 2 15.png]]

- 전달 받은 패킷을 까보면서 클라이언트 IP 주소를 보고 요청할때와 똑같이 패킷으로 감싸서 클라이언트로 보낸다.

## 관련 문서

- 상위 목차: [[HTTP 웹 기본 지식]]
- 이전 문서: [[섹션 1 - 인터넷 네트워크]]
- 다음 문서: [[섹션 3 - HTTP 기본]]
