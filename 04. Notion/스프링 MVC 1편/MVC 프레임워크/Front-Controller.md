## 구조

![[Untitled 77.png|Untitled 77.png]]

### 프론트 컨트롤러 패턴 특징

1. 프론트 컨트롤러 서블릿 하나로 Client의 요청을 받음
2. 프론트 컨트롤러가 요청에 맞는 컨트롤러를 찾아서 호출
3. ==**공통 처리 가능**==
4. 프론트 컨트롤러를 제외한 나머지 컨트롤러는 서블릿을 사용하지 않아도

  

## 스프링 웹 MVC와 프론트 컨트롤러

- 스프링 웹 MVC의 핵심이 바로 FrontController
- 스프링 웹 MVC의 DispatcherServlet이 FrontController 패턴으로 구현되어 있다.