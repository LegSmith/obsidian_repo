HTTP message body에 데이터를 직접 담아서 요청

## 단순 텍스트

- **==content-type : text/plain==**
- getInputStream() 메서드로 가져온다. 스트림 타입이기 때문에 String으로 변환해야 한다.

```java
@WebServlet(name="requestBodyStringServlet", urlPatterns = "/request-body-string")
public class RequestBodyStringServlet extends HttpServlet {
    @Override
    protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        ServletInputStream inputStream = request.getInputStream(); //메세지 바디에 있는 데이터를 바이트코드로 얻어옴
        String messageBody = StreamUtils.copyToString(inputStream, StandardCharsets.UTF_8);//값, 인코딩 형식

        System.out.println("messageBody = " + messageBody);

        response.getWriter().write("ok");
    }
}
```


## JSON

- **==content-type : application/json==**
- message body : **=={"username" : "hello", "age": 20}==** //key:value 형식이다


- JSON은 자바 객체로 바꿔서 사용하기 때문에 파싱할 수 있게 객체를 하나 생성한다.

```java
@WebServlet(name="requestBodyJsonServlet", urlPatterns = "/request-body-json")
public class RequestBodyJsonServlet extends HttpServlet {
    @Override
    protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        ServletInputStream inputStream = request.getInputStream();
        String messageBody = StreamUtils.copyToString(inputStream, StandardCharsets.UTF_8);

        System.out.println("messageBody = " + messageBody);
    }
}
```


## JSON → 객체

- JSON 결과를 파싱해서 사용할 수 있는 자바 객체로 변환하려면 Jackson, Gson같은 JSON 변환 라이브러리를 추가해서 사용해야 한다. 스프링 부트로 Spring MVC를 선택하면 기본으로 Jackson라이브러리( ObjectMapper ) 를 함께 제공한다.

```java
@WebServlet(name="requestBodyJsonServlet", urlPatterns = "/request-body-json")
public class RequestBodyJsonServlet extends HttpServlet {

    private ObjectMapper objectMapper = new ObjectMapper(); //jackson 라이브러리

    @Override
    protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        ServletInputStream inputStream = request.getInputStream();
        String messageBody = StreamUtils.copyToString(inputStream, StandardCharsets.UTF_8);

        HelloData helloData = objectMapper.readValue(messageBody, HelloData.class);//jackson이라는 라이브러리가 자바객체로 변환해줌
        System.out.println("helloData.username = " + helloData.getUsername());
        System.out.println("helloData.age = " + helloData.getAge());
    }
}
```

## 관련 문서

- 상위 목차: [[HTTP 요청 데이터 - 개요]]
- 다음 문서: [[HTTP 요청 데이터 - GET 쿼리 파라미터]]
