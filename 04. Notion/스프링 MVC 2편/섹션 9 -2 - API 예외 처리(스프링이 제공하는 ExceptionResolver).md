스프링부트가 기본으로 제공하는 `ExceptionResolver` 는 3가지가 있다.  
1. `ExceptionHandlerExceptionResolver`  
2. `ResponseStatusExceptionResolver`  
3. `DefaultHandlerExceptionResolver`  
위 3가지가 `HandlerExceptionResolverComposite` 에 다음 순서로 등록된다.(우선순위임)

  

## ResponseStatusExceptionResolver

---

_==**예외에 따라서 HTTP 상태 코드를 지정**==_해주는 역할이다.  
`@ResponseStatus` 가 달려있는 예외 , `ResponseStatusException` 예외일 경우 사용 가능하다.

### @ResponseStatus

- 임의로 예외 클래스를 만들고 `@ResponseStatus` 를 사용
- `code` 에는 HTTP 상태코드가 들어가고 `reason` 에는 메시지가 들어간다.
    
    - ==_**에러 메시지는 스프링이 제공하는 메시지 기능사용가능 !**_==
    
    ```java
    @ResponseStatus(code = HttpStatus.CREATED, reason = "error.bad")
    public class BadRequestException extends RuntimeException{
    }
    
    //messages.properties
    error.bad=잘못된 요청
    ```
    

### ResponseStatusException

- `@ResponseStatus` 는 개발자가 변경할 수 없는 예외에는 사용할 수 없다.(`RuntimeException` , `IllegalArgumentException` 등등)
- 이 때는 컨트롤러에서 직접 `ResponseStatusException` 예외를 터뜨리면 된다.
    
    ```java
    @GetMapping("/api/response-status-ex2")
    public String responseStatusEx2(){
        throw new ResponseStatusException(HttpStatus.NOT_FOUND,"error.bad", new IllegalArgumentException());
    }
    ```
    
    > 컨트롤러에서 IlleagalArgumentException 이 터지면 500에러에서 404 에러로 바뀌는건가 ?
    

## DefaultHandlerExceptionResolver

---

==_**스프링 내부에서 발생하는 스프링 예외를 해결**_==한다. 대표적으로 파라미터 바인딩 시점에 타입이 맞지 않을 때 `TypeMismatchException` 이 발생하게 된다. 이 때 서버에서 예외가 터졌기 때문에 500 오류인데 스프링 내부에서 `DefaultHandlerExceptionResolver` 에 의해서 400 오류로 바꿔준다.

- Integer 타입의 파라미터를 만약 String 타입으로 요청한다면 500 에러가 나야한다. 하지만 스프링 내부에`DefaultHandlerExceptionResolver` 에 의해서 400 에러로 바뀌게 된다.
    
    ```java
    @GetMapping("/api/default-handler-ex")
    public String defaultException(@RequestParam Integer data){
        return "ok";
    }
    ```
    
    ![[Untitled 12.png|Untitled 12.png]]
    

## ExceptionHandlerExceptionResolver

---

`ExceptionResolver` 가 동작할 때 가장 우선순위가 높은 `ExceptionHandlerExceptionResolver` 를 먼저 찾는다

- `ExceptionHandlerExceptionResolver` 는 예외가 터진 컨트롤러 내에서 `@ExceptionHandler` 어노테이션이 붙은 클래스 중 예외타입이 맞는 클래스를 찾는다.
    
    - 해당 예외와 그 자식 예외까지 처리가 가능하다.
    
    ```java
    @Slf4j
    @RestController
    public class ApiExceptionV2Controller {
    @GetMapping("/api2/members/{id}")
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
    
    @Data
    @AllArgsConstructor
    static class ErrorResult {
        private String code;
        private String message;
    }
    ```
    
- `IllegalArgumentException` 예외가 터질 때 로직을 구현해보면 ==**예외가 터지면 서블릿에 가기전 ExcpetionResolver 가 동작하고 가장 우선순위가 높은 @ExceptionHandler 를 찾는다.**==
- 그중 예외 타입이 맞는 메서드를 실행 시키게 된다.
    
    ```java
    @ResponseStatus(HttpStatus.BAD_REQUEST)
    @ExceptionHandler(IllegalArgumentException.class)
    public ErrorResult illegalExHandler(IllegalArgumentException e){
        log.error("[exceptionHandler] ex", e);
        return new ErrorResult("BAD",e.getMessage());
    }
    ```
    
    ![[Untitled 1 9.png|Untitled 1 9.png]]
    
- `@ExcpetionHandler` 에 예외타입을 생략하면 파라미터로 들어오는 예외타입을 보고 판단한다.

### 스프링의 우선순위

- 스프링은 항상 디테일한 부분부터 우선권을 가진다. 예를 들어 부모, 자식 클래스가 있다면 자식클래스 먼저 예외를 찾는다.
- 모든 예외의 부모인 `Exception` 예외를 파라미터로 받는 `@ExceptionHandler` 가 있다고 해도 위에 있는 코드처럼 ==**자식인 IllegalArgumentException 부터 먼저 우선권**==을 가지게 된다.
    
    ```java
    @ResponseStatus(HttpStatus.INTERNAL_SERVER_ERROR)
    @ExceptionHandler
    public ErrorResult exHandler(Exception e){
        log.error("[exceptionHandler] ex",e);
        return new ErrorResult("EX","내부 오류");
    }
    ```
    

### ExceptionHandlerExceptionResolver 실행 흐름

1. 컨트롤러에서 `IllegalArgumentException` 이 터지면서 예외가 컨트롤러 밖으로 던져진다.
2. `ExceptionResolver` 가 동작하면서 우선순위가 가장 높은 `ExceptionHandlerExceptionResolver` 가 실행
3. `ExceptionHandlerExceptionResolver` 는 해당 컨트롤러에서 `IllegalArgumentException` 예외를 해결할 수 있는 `@ExceptionHandler` 를 찾는다.
4. 찾은 메서드를 실행하고 현재 클래스가 `@RestController` 이기 때문에 HTTP 메시지 컨버터가 동작하면서 응답을 JSON 으로 반환한다.

  

## @ControllerAdvice

---

쉽게 말해 예외가 터졌을 때 가장먼저 가로채서 `@ExceptionHandler` , `@InitBinder` 기능을 부여해준다.  
즉, 쉽게 말해 ==_**스프링부트를 사용할 때 예외가 터지면 해당 어노테이션이 붙은 클래스에서 예외처리를 한다.**_==

- `@ControllerAdvice` 와 `@RestControllerAdvice` 는 같고 후자에 `@ResponseBody` 가 추가되있다.
    
    ```java
    @Slf4j
    @RestControllerAdvice
    public class ExControllerAdvice {
        @ResponseStatus(HttpStatus.BAD_REQUEST)
        @ExceptionHandler(IllegalArgumentException.class)
        public ErrorResult illegalExHandler(IllegalArgumentException e){
            log.error("[exceptionHandler] ex", e);
            return new ErrorResult("BAD",e.getMessage());
        }
    
        @ExceptionHandler
        public ResponseEntity<ErrorResult> userExHandler(UserException e){
            log.error("[exceptionHandler} ex",e);
            ErrorResult errorResult = new ErrorResult("USER-EX", e.getMessage());
            return new ResponseEntity(errorResult,HttpStatus.BAD_REQUEST);
        }
    
        @ResponseStatus(HttpStatus.INTERNAL_SERVER_ERROR)
        @ExceptionHandler
        public ErrorResult exHandler(Exception e){
            log.error("[exceptionHandler] ex",e);
            return new ErrorResult("EX","내부 오류");
        }
    }
    ```
    
- 대상을 지정할 수 있다.
    
    ```java
    // Target all Controllers annotated with @RestController
      @ControllerAdvice(annotations = RestController.class)
      public class ExampleAdvice1 {}
      // Target all Controllers within specific packages
      @ControllerAdvice("org.example.controllers")
      public class ExampleAdvice2 {}
      // Target all Controllers assignable to specific classes
      @ControllerAdvice(assignableTypes = {ControllerInterface.class,
      AbstractController.class})
      public class ExampleAdvice3 {}
    ```