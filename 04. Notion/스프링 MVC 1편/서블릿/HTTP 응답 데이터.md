  

HTTP 응답 메세지는 주로 다음 내용을 담아서 전달한다.

1. 단순텍스트 응답 ( writer.println(”ok”); )
2. HTML 응답
3. HTTP API - MessageBody JSON 응답

  

  

## HTML 응답

```java
@WebServlet(name="responseHtmlServlet", urlPatterns = "/response-html")
public class ResponseHtmlServlet extends HttpServlet {
    @Override
    protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        //Content-type : text/html;charset=utf-8
        response.setContentType("text/html");
        response.setCharacterEncoding("utf-8");

        PrintWriter writer = response.getWriter();
        writer.println("<html>");
        writer.println("<body>");
        writer.println("  <div>안녕?</div");
        writer.println("</body>");
        writer.println("</html>");
    }
}
```

- 페이지 소스보기를 하면 제대로 인코딩 한 것을 알 수 있다.
    
    ![[Untitled 76.png|Untitled 76.png]]
    
    ![[Untitled 1 57.png|Untitled 1 57.png]]
    

  

  

## API JSON

- application/json 은 스펙상 utf-8형식을 사용하도록 정의되어 있어서. charset=utf-8을 적어줄 필요가 없