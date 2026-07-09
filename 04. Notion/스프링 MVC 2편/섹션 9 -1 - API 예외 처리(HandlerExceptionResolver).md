==**HTML 페이지는 오류에 맞게 웹 브라우저에게 페이지만 띄어주면 된다. 하지만 API 통신같은 경우는 오류응답 스펙을 정하고 JSON 으로 데이터를 줘야한다.**==

## API 예외처리 - 스프링 없이 서블릿으로 처리

---

- 스프링 없이 순수하게 서블릿으로만 구현하기 때문에 `WebServerCustomizer` 가 필요하다.
    
    ```java
    @Component
    public class WebServerCustomizer implements WebServerFactoryCustomizer<ConfigurableWebServerFactory> {
        @Override
        public void customize(ConfigurableWebServerFactory factory) {
            ErrorPage errorPage404 = new ErrorPage(HttpStatus.NOT_FOUND, "/error-page/404");
            ErrorPage errorPage505 = new ErrorPage(HttpStatus.INTERNAL_SERVER_ERROR, "/error-page/500");
    
            ErrorPage errorPageEx = new ErrorPage(RuntimeException.class, "/error-page/500");
    
            factory.addErrorPages(errorPage505,errorPage404,errorPageEx);
        }
    }
    ```
    
- API 통신을 위한 컨트롤러를 생성하고 쿼리파라미터로 id를 받는데 ==**id가 ex 이면 예외를 터뜨리고 그게 아니면 memberId 와 name 을 반환**==하는 컨트롤러를 구현하자
    
    ```java
    @Slf4j
    @RestController
    public class ApiExceptionController {
    
        @GetMapping("/api/members/{id}")
        public MemberDto getMember(@PathVariable String id){
            if(id.equals("ex")){
                throw new RuntimeException("잘못된 사용자");
            }
    
            return new MemberDto(id, "hello  "+id);
        }
    
        @Data
        @AllArgsConstructor
        static class MemberDto {
            private String memberId;
            private String name;
        }
    }
    ```
    
- 만약 예외가 터진다면 로직은 이렇게 된다.
    
    클라이언트 → WAS → 필터 → 서블릿 → 인터셉터 → ==**컨트롤러(예외발생)**== → 인터셉터 → 서블릿 → 필터 → ==**WAS(errorPage 경로를 찾아감)**== → 필터 → 서블릿 → 인터셉터 → 컨트롤러(예외 응답 데이터를 JSON 으로 반환)
    
- 하지만 `ErrorPageController` 를 보면 ==**예외가 터질떄는 /error-page/500 컨트롤러가 동작하면서 error-page/500.html 을 보여주기 때문에 API 통신이 아니게 된다.**==
- 그렇기 때문에 같은 `@RequestMapping` 을가지고 ==**Accept 타입이 application/json 인 컨트롤러를 하나 더 생성**==하면 된다.
    
    - `MediaType` 은 스프링이 지원하는 클래스를 사용
    - `ResponseEntity` 는 ==**HTTP 메시지 바디에 직접 데이터를 넣어서 반환**==
    - 해당 컨트롤러는 WAS 에서 내부적으로 예외가 터졌을때 오는 요청이기 때문에 메서드 위 상수를 통해 예외응답 데이터를 설정.
    
    ```java
    //RequestDispatcher 상수로 정의되어 있음
    public static final String ERROR_EXCEPTION = "javax.servlet.error.exception";
    public static final String ERROR_EXCEPTION_TYPE = "javax.servlet.error.exception_type";
    public static final String ERROR_MESSAGE = "javax.servlet.error.message";
    public static final String ERROR_REQUEST_URI = "javax.servlet.error.request_uri";
    public static final String ERROR_SERVLET_NAME = "javax.servlet.error.servlet_name";
    public static final String ERROR_STATUS_CODE = "javax.servlet.error.status_code";
    
    @RequestMapping(value = "/error-page/500", produces = MediaType.APPLICATION_JSON_VALUE)
    public ResponseEntity<Map<String, Object>> errorPage500Api(
            HttpServletRequest request, HttpServletResponse response){
        log.info("API errorPage 500");
    
        Map<String, Object> result = new HashMap<>();
        Exception ex = (Exception) request.getAttribute(ERROR_EXCEPTION);
        result.put("status", request.getAttribute(ERROR_STATUS_CODE));
        result.put("message", ex.getMessage());
    
        Integer statusCode = (Integer) request.getAttribute(RequestDispatcher.ERROR_STATUS_CODE);
    
        return new ResponseEntity<>(result, HttpStatus.valueOf(statusCode));
    
    }
    ```
    

  

> _**스프링없이 순수한 서블릿으로 구현했기 때문에 코드가 매우 복잡해진다.**_

## API 예외 처리 - 스프링 부트 기본 오류 처리

---

==**스프링부트가 지원하는 기본 오류 처리는 앞서 HTML 을 보여줄 때는 HTML 만 만들어주면 되지만 API 통신을 따로 API 스펙이 없다면 스프링부트가 알아서 JSON 으로 응답한다.**==

- 스프링부트가 제공한느 BasicErrorController 가 HTML 을 반환해야 할 떄와 API 통신으로 JSON 을 반환해야 할 때를 구분해서 로직이 짜여있다.
    
    ```java
    @RequestMapping(produces = MediaType.TEXT_HTML_VALUE)
    	public ModelAndView errorHtml(HttpServletRequest request, HttpServletResponse response) {
    		HttpStatus status = getStatus(request);
    		Map<String, Object> model = Collections
    			.unmodifiableMap(getErrorAttributes(request, getErrorAttributeOptions(request, MediaType.TEXT_HTML)));
    		response.setStatus(status.value());
    		ModelAndView modelAndView = resolveErrorView(request, response, status, model);
    		return (modelAndView != null) ? modelAndView : new ModelAndView("error", model);
    	}
    
    	@RequestMapping
    	public ResponseEntity<Map<String, Object>> error(HttpServletRequest request) {
    		HttpStatus status = getStatus(request);
    		if (status == HttpStatus.NO_CONTENT) {
    			return new ResponseEntity<>(status);
    		}
    		Map<String, Object> body = getErrorAttributes(request, getErrorAttributeOptions(request, MediaType.ALL));
    		return new ResponseEntity<>(body, status);
    	}
    ```
    
- 스프링부트가 기본으로 `timestamp` , `status` , `error` , `exception` , `path` 를 JSON 으로 응답해주낟.
    
    ![[Untitled 11.png|Untitled 11.png]]
    

## HandlerExceptionResolver - 시작

---

==**발생하는 예외에 따라서 오류 메시지, HTTP 상태코드 등 형식을 API 마다 다르게 처리**==할려면 `HandlerExceptionResolver` 를 사용하면 된다.

- 우선 `ApiExceptionController` 에서 쿼리파라미터에 bad 로 오면 `IlleagalArgumentException()` 예외가 발생하도록 한다.
- 그리고 `IlleagalArgumentException()` 은 잘못된 파라미터가 요청된 예외이기 떄문에 **==HTTP 상태코드 400(클라이언트 오류)==**를 내게 하고 싶다.
    
    ```java
    @GetMapping("/api/members/{id}")
    public MemberDto getMember(@PathVariable String id){
        if(id.equals("ex")){
            throw new RuntimeException("잘못된 사용자");
        }
    
        if(id.equals("bad")){
            throw new IllegalArgumentException("입력값이 잘못됐습니다!!");
        }
    
        return new MemberDto(id, "hello  "+id);
    }
    ```
    
- 하지만 이상태로 실행해도 HTTP 상태코드는 500이다.
    
    → 예외가 터지면 WAS 까지 예외가 도달하면서 WAS 에서 500 에러를 찾아 재요청을 하기 때문에 500에러가 뜨게 된다.
    
- 이러한 동작을 바꾸기 위해 HandlerExceptionResolver(ExcpetionResolver) 를 사용해야 한다.

### ExceptionResolver 흐름

- 컨트롤러에서 터진 에러가 _==**WAS 에 도달하기전에 ExceptionResolver 에서 예외를 해결하고 WAS 로 보내게 된다.**==_
    
    ![[Untitled 1 8.png|Untitled 1 8.png]]
    
- HandlerExceptionResolver 는 인터페이스이기 때문에 구체화를 해야한다.
- `resolveException()`메서드를 오버라이딩 하고 여기서 예외(ex) 타입을 분기하여 `IllegalArgumentException` 이면 `sendError(400)` 를 호출해서 ==**HTTP 상태코드를 400으로 저장하고 WAS**==로 가게 된다.
- ModelAndView 를 반환하는 이유는 Exception 을 처리해서 정상 흐름으로 변경하기 위해서이다.
    
    - **==빈 ModelAndView 반환==** : _==**뷰를 랜더링 하지않고 정상 흐름으로 서블릿이 WAS 로 리턴**==_
    - **==ModelAndView 지정==** : View, Model 등의 정보를 지정해서 반환하면 뷰를 랜더링
    - ==**null**== : null 을 반환하면 ==_**다음 ExceptionResolver 를 찾아서 실행하고 없다면 예외 처리가 안되고 서블릿 밖으로 예외를 던져서 WAS 가 예외를 판단해서 후속 조치**_==를 하도록 한다.
    
    ```java
    @Slf4j
    public class MyHandlerExceptionResolver implements HandlerExceptionResolver {
        @Override
        public ModelAndView resolveException(
                HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) {
    
            log.info("call resolver", ex);
            try {
                if (ex instanceof IllegalArgumentException) {
                    log.info("IllegalArgumentException resolver to 400");
                    response.sendError(HttpServletResponse.SC_BAD_REQUEST, ex.getMessage());
                    // 빈 ModelAndView 를 반환하면 계속 return 되면서 WAS 까지 도달하고 WAS 에서 서블릿 컨테이너가 bad_request 를 담은걸 알고 에러 처리를 한다.
                    return new ModelAndView();
                }
            } catch (IOException e) {
                log.error("resolver ex", e);
            }
    
            return null;
        }
    }
    ```
    

### ExceptionHandler 등록

- `WebMvcConfigurer` 인터페이스를 구현화한 `WebConfig` 에 등록해주면 된다.
    
    ```java
    @Override
    public void extendHandlerExceptionResolvers(List<HandlerExceptionResolver> resolvers) {
        resolvers.add(new MyHandlerExceptionResolver());
    }
    ```
    

### ExceptionResolver 활용

1. ==**예외 상태 코드 변환**==
    - 위 예제 처럼 `response.sendError(xxx)` 호출로 변경해서 특정 예외일때 개발자가 원하는 상태코드로 변경
2. **==뷰 템플릿 처리==**
    - 반환하는 `ModelAndView` 에 값을 채워서 새로은 오류 화면을 뷰 랜더링하여 웹 브라우저에게 전달
3. **==API 응답 처리==**
    - `response.getWriter().println(”hello”);` 처럼 HTTP 응답 바디에 직접 데이터를 넣어줄 수 있다

## API 예외 처리 - HandlerExceptionResolver 활용

---

컨트롤러에서 예외가 발생하면 `HandlerExceptionResolver` 를 통해 ==_서블릿에 가기전에 예외를 해결하고 어플리케이션을 정상동작_==하게 만들 수 있다.

- 우선 개발자가 사용자 정의 예외를 구현하고 그 예외가 터졌을 때 동작하는 ExcpetionResolver 를 구현해주면 된다.
    
    `UserException.java`
    
    ```java
    public class UserException extends RuntimeException{
        public UserException() {
            super();
        }
    
        public UserException(String message) {
            super(message);
        }
    
        public UserException(String message, Throwable cause) {
            super(message, cause);
        }
    
        public UserException(Throwable cause) {
            super(cause);
        }
    
        protected UserException(String message, Throwable cause, boolean enableSuppression, boolean writableStackTrace) {
            super(message, cause, enableSuppression, writableStackTrace);
        }
    }
    ```
    
- 그리고 ApiExceptionController 에 UserException을 터뜨려줘야 한다.
    
    `ApiExceptionController.java`
    
    ```java
    @Slf4j
    @RestController
    public class ApiExceptionController {
    
        @GetMapping("/api/members/{id}")
        public MemberDto getMember(@PathVariable String id){
            if(id.equals("ex")){
                throw new RuntimeException("잘못된 사용자");
            }
    
            if(id.equals("bad")){
                throw new IllegalArgumentException("입력값이 잘못됐습니다!!");
            }
    
            if(id.equals("user-ex")){
                throw new UserException("사용자 오류");
            }
    
            return new MemberDto(id, "hello  "+id);
        }
    
        @Data
        @AllArgsConstructor
        static class MemberDto {
            private String memberId;
            private String name;
        }
    }
    ```
    
- UserException 예외가 터지면 동작하는 ExceptionResolver 를 구현
- 먼저, ACCEPT 값이 `application/json` 이면 JSON 으로 응답하고 그 외에는 HTML 로 응 답하기 위해`getHeader(”accept”)` 메서드로 ACCEPT 값을 뽑아온다.
- HTTP 상태코드를 400으로 변경하고 응답하기 위해 `setStatus(400)` 메서드 호출
- 그리고 `Accept` 가 JSON 일 떄는 Map 컬렉션으로 JSON 형식으로 데이터를 만들고 `ObjectMapper` 를 이용하여 **==JSON → String 으로 변경 후 response 객체==**에 담는다. 그리고 _==**ModelAndView 빈 객체를 리턴**==_
- `Accept` 가 JSON 이 아닐 때는 스프링부트의 `BasicErrorController` 를 이용하여 error 페이지를 반환
    
    ```java
    @Slf4j
    public class UserHandlerExceptionResolver implements HandlerExceptionResolver {
    
        private final ObjectMapper objectMapper = new ObjectMapper();
    
        @Override
        public ModelAndView resolveException(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) {
            try{
                if(ex instanceof UserException){
                    log.info("UserException resolver to 400");
    
                    String acceptHeader = request.getHeader("accept"); //HTTP 메시지가 JSON 인지 아닌지 구분하기위해 accept 옵션 꺼내오기
                    response.setStatus(HttpServletResponse.SC_BAD_REQUEST); //HTTP 상태코드 수정
    
                    if("application/json".equals(acceptHeader)){ //JSON 이면 JSON 형식으로 생성
                        Map<String, Object> errorResult = new HashMap<>();
                        errorResult.put("ex", ex.getClass()); //예외타입
                        errorResult.put("message", ex.getMessage());// 메시지
    
                        String result = objectMapper.writeValueAsString(errorResult);// JSON -> String
    
                        response.setContentType("application/json");
                        response.setCharacterEncoding("utf-8");
                        response.getWriter().write(result);
    
                        return new ModelAndView(); // 정상 흐름으로 WAS 까지 이동
                    }else{ //JSON 이 아닐때
                        return new ModelAndView("error/500"); // resources/templates/error/500.html 가져옴
                    }
                }
            }catch (IOException e){
                log.error("resolver ex",e);
            }
            return null;
        }
    }
    ```
    
- 마지막으로 `WebConfig` 에 `ExceptionResolver` 를 등록 !

  

> ==_**ExceptionResolver 를 사용하면 컨트롤러에서 예외가 발생해도 서블릿까지 올라가지 않고 스프링 MVC 에서 예외를 처리하고 끝낸다. 결과적으로 WAS 까지 예외가 올라가지 않기 때문에 정상 처리가 됐다.**_==