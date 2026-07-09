## M(Model) , V(View), C(Controller)

- 서블릿이나 jsp로 처리하던 것을 MVC로 역할을 나누어서 유지보수나 수정에 용이하게 하는 웹 애플리케이션의 패턴이다. 대부분 웹 애플리케이션은 MVC패턴을 사용한다.

  

## Controller

- 비즈니스 로직을 실행하고 View로직을 실행하여 제어권을 View에게 준다.

## View

- 모델에 담겨있는 데이터를 사용하여 화면을 클라이언트에게 보여준다.
- View는 오직 화면렌더링에만 집중한다.

## Model

- View에 출력할 데이터를 담아둔다.

  

# MVC패턴

![[Untitled 39.png|Untitled 39.png]]

  

```java
@Override
    protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        String viewPath = "/WEB-INF/views/new-form.jsp";
        RequestDispatcher dispatcher = request.getRequestDispatcher(viewPath);
        dispatcher.forward(request,response); //서버내에서 jsp나 서블릿을 호출하는코
    }
```

  

# MVC 패턴 - 한계

### 1. forword반복

- 아래 코드가 계속 반복된다

```java
String viewPath = "/WEB-INF/views/new-form.jsp"; 
RequestDispatcher dispatcher = request.getRequestDispatcher(viewPath);
dispatcher.forward(request,response); //서버내에서 jsp나 서블릿을 호출하는코드
```

- viewPath도 반복된다. 만약 path가 변경되면 모든 컨트롤러에 수정해야한다.
- request, response객체를 사용하지 않을때도 있다.

### 2. 공통 처리가 어렵다.