  

## 클라이언트→서버로 데이터 전송

---

데이터 전달 방식은 크게 2가지다.

1. ==**쿼리 파라미터**==를 통한 데이터 전송
    - `GET`
    - 주로 정렬 필터(검색어)
2. ==**메시지 바디**==를 통한 데이터 전송
    - `POST` , `PUT` , `PATCH`
    - 회원가입, 상품주문, 리소스 등록, 리소스 변경

### 정적 데이터 조회

- **이미지**, **정적 텍스트 문서**를 정적데이터라 한다.
- 쿼리 파라미터를 미사용 하는 경우에는 ==_**단순하게 추가적인 데이터가 필요없다.**_==
    
    ![[Untitled 38.png|Untitled 38.png]]
    

### 동적 데이터 조회

- 동적 데이터는 쿼리 파라미터를 사용한다.
    - `https://www.google.com/search?q=hello&hi=ko` 와 같은 주소로 요청하는 경우이다.
    - URL 뒤에 `?` 뒤에 오는 데이터를 가지고 _==**서버에서 동적으로 데이터를 생성하여 클라이언트에게 보여준다.**==_
- 주로 **검색**, **게시판 목록에서 정렬 필터(검색어)**에 사용한다.
- 이때도 `GET` 메서드를 사용한다.
    
    ![[Untitled 1 28.png|Untitled 1 28.png]]
    

### HTML Form 데이터 전송

- `POST` 메서드를 통해 데이터를 전송햐는 경우이다.
- 간단한 HTML 태그로 데이터를 입력하는 화면을 만들었다.
- `name` 속성이 실제 HTTP 메시지 바디에 담겨지기 때문에 중요하다.
    
    ![[Untitled 2 17.png|Untitled 2 17.png]]
    
- 서버로 ==**username 과 age 라는 키값으로 입력한 데이터 kim, 20 이 서버로 전송**==된다.
- `Content-Type` 을 보면 `application/x-www-form-urlencoded` 는 _==**웹브라우저에서 기본으로 정해진 Content-Type**==_이다.
    - `key=data&key2=data2` 이런 형태로 이루어져 있다.
        
        ![[Untitled 3 11.png|Untitled 3 11.png]]
        
- 데이터를 전송할 때는 ==**Form 태그에 GET 방식으로 지정하여 전송할 수 있다.**==
- 이 경우에 서버는 GET 을 인지하고 알아서 쿼리 파라미터로 넣어준다.
- 하지만 GET 메서드를 SAVE(저장)하는 로직에 사용하면 안된다 !!
    
    ![[Untitled 4 9.png|Untitled 4 9.png]]
    
- 단순한 텍스트 형식의 데이터 뿐만 아니라 파일도 가능하다. 이런 타입을 `multipart/form-data` 라 한다.
- 아래와 같이 `username` , `age` , `파일`을 서버로 전송하게 되면 `enctype` 이 `multipart/form-data` 로 설정되어 서버로 전송이 된다.
    
    ![[Untitled 5 8.png|Untitled 5 8.png]]
    
- 웹 브라우저가 아래와 같이 HTTP 메시지를 생성하는데, boundary 를 보면 `- - - -XXX` 로 되어있는데 이것은 브라우저가 자동으로 설정해주는 값이다.
- _==**boundary 값을 통해 username , age, 파일이 나누어진다. 파일이 여러개로 나뉜다해서 multipart 타입**==_이라 한다.
- 그리고 이미지는 `109238a9o0p…..` 와 같이 바이트로 변해서 전송이 된다.
    
    ![[Untitled 6 7.png|Untitled 6 7.png]]
    

### HTTP API 데이터 전송

- 예를 들어 안드로이드 애플리케이션에서 서버로 데이터를 전송할 때 HTTP API 로 데이터를 전송한다고 한다.
- 쉽게말해 하나한 다 만들어서 보내는 것이다.
    
    ![[Untitled 7 5.png|Untitled 7 5.png]]
    
- 보통 ==**서버 ↔ 서버 통신시**==에 자주 사용(백엔드 시스템 통신)
- 앱 클라이언트(아이폰 , 안드로이드) , 웹 클라이언트(HTML 에서 Form 이 아닌 AJAX 사용)
- Content-Type : `application/json` 을 주로 사용(사실상 표준)

## HTTP API 설계

---

### HTTP API - POST 기반 등록

아래와 같이 URI 를 설계한다.

- **회원** 목록 `/members` → `GET`
- **회원** 등록 `/members` → `POST`
- **회원** 조회 `/memberse/{id}` → `GET`
- **회원** 수정 `/members/{id}` → `PATCH` , `PUT` , `POST`
- **회원** 삭제 `/members/{id}` → `DELETE`

- 클라이언트는 등록될 리소스의 URI 를 모른다.
- 서버가 새로 등록된 리소스 URI를 생성해준다.
    
    ```bash
    HTTP/1.1 201 Created
    Location: /members/100
    ```
    
- 컬렉션(Collection)
    - 서버가 관리하는 리소스 디렉토리
    - 여기서 컬렉션은 `/members`

### HTTP API - PUT 기반 등록

아래와 같이 URI 를 설계한다. 현재 API 는 ==**파일 관리 시스템**==이다.

- **파일** 목록 `/files` → `GET`
- **파일** 조회 `/fies/{filename}` → `GET`
- **파일** 등록 `/files/{filename}` → `PUT`
- **파일** 삭제 `/files/{filename}` → `DELETE`
- **파일** 대량 등록 `/files` → `POST`

- 클라이언트가 리소스 URI를 알고 있어야 한다.
- 스토어(Store)
    - 클라이언트가 관리하는 리소스 저장소
    - 여기서 스토어는 `/files`

### HTTP FORM 사용

HTTP Form 은 `GET` , `POST` 만 지원한다.

- Ajax 를 사용하면 해결 가능하지만 순수한 HTTP Form 을 사용하면 GET , POST 만 된다.

### URI 예시

- **회원** 목록 `/members` → `GET`
- **회원** 등록 폼 `/members/new` → `GET`
- **회원** 등록 `/members/new` → `POST`
- **회원** 조회 `/members/{id}` → `GET`
- **회원** 수정 폼 `/members/{id}/edit` → `GET`
- **회원** 수정 `/members/{id}/edit` → `POST`
- **회원** 삭제 `/members/{id}/delete` → `POST`
    - _==**DELETE를 지원하지 않기에 URI 에 delete를 표시**==_해준다 → `컨트롤 URI`

- `GET` , `POST` 만 지원하기에 제약이 있다.
- **컨트롤 URI**
    - `GET` , `POST` 의 제약을 해결하기 위한 방법
    - 동사로 된 리소스 경로 사용 → `/new` , `/edit` , `/delete` 가 컨트롤 URI 이다.
    - ==_**최대한 리소스를 가지고 URI를 사용하고 애매한 경우에만 사용**_==