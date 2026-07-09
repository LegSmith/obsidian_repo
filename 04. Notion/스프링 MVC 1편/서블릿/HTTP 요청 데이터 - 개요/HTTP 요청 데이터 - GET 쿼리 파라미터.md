## 형식

- 쿼리 파라미터는 URL다음에 ? 를 시작으로 보낼 수 있다. 추가 파라미터는 & 로 연결한다.


```java
/**
 * 1. 파라미터 전송기능
 * http://localhost:8080/request-param?username=hello&age=20
 *
 */
@WebServlet(name="requestParamServlet", urlPatterns = "/request-param")
public class RequestParamServlet extends HttpServlet {

    @Override
    protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        System.out.println("전체 파라미터 조회 - start");
        request.getParameterNames().asIterator().
                forEachRemaining(paramName -> System.out.println(paramName+"="+request.getParameter(paramName))); //paramName은 key
        System.out.println("전체 파라미터 조회 - end");
        System.out.println();

        System.out.println("단일 파라미터 조회");
        String username = request.getParameter("username");
        String age = request.getParameter("age");
        System.out.println("username = " + username);
        System.out.println("age = " + age);
        System.out.println();

        System.out.println("[이름이 같은 복수 파라미터 조회]");
        String[] usernames = request.getParameterValues("username");
        for (String name:usernames){
            System.out.println("username = " + name);
        }

        response.getWriter().write("ok");

    }
}
```

## 복수 파라미터에서 단일 파라미터 조회

- request.getParmeter() 는 하나의 파라미터 이름에 대해서 단 하나의 값만 있을 때 사용,
- 하나의 파라미터 이름의 여러 값이 있을때는 request.getParmeterValues()를 이용해 배열에 저장하여 데이터 사용.

## 관련 문서

- 상위 목차: [[HTTP 요청 데이터 - 개요]]
- 이전 문서: [[HTTP 요청 데이터 - API 메시지 바디]]
- 다음 문서: [[HTTP 요청 데이터 - POST HTML Form]]
