AOP(Aspect Orientied 의 핵심은 **`관심사 분리`** 이다 !!

### PointCut

- 어디 메서드에 적용시킬 지 정한다.

### JoinPoint

- 적용할 메서드

### Advice

- 구현할 코드

---

## AOP 구현하기

- 우선 회원가입 시 매번 유효성검사를 하고 유효성 검사실패시 예외처리로직을 모든 메서드에 적어줄 순 없다 !
- Advice 에 유효성검사실패시 예외처리 로직을 넣고 JoinPoint 는 `@GetMapping` , `@PostMapping` 이 붙은 메서드로 한다.


- `@Around()` 어노테이션에 위에서 @Pointcut 으로 지정한 메서드를 담아주면 전/후 제어가 된다.
- `@Around` 가 붙은 메서드는 ==**ProceedingJoinPoint 타입의 파라미터를 받는데 이 파라미터는 JoinPoint 로 지정된 메서드에 모든 정보**==를 갖고 있다.
- `getArgs()` 로 메서드의 파라미터를 다 갖고온다.
- 파라미터중에 `BindingResult` 타입이 있다면 다운 캐스팅을 하고 유효성검사 실패 시 예외처리 로직을 그대로 갖고 오면 된다.
- 만약 에러가 있다면 개발자가 만든 유효성검사 실패 예외처리를 throw 한다.
- 마지막으로 `proceed()`는 ==**해당 메서드로 돌아가 정상적으로 실행하는 메서드**==이다.

```java
@Component
@Aspect
public class CustomValidationAdvice {

    @Pointcut("@annotation(org.springframework.web.bind.annotation.PostMapping)")
    public void postMapping(){}

    @Pointcut("@annotation(org.springframework.web.bind.annotation.PutMapping)")
    public void putMapping(){}

    @Around("postMapping() || putMapping()") // joinPoint 의 전/후 제어가 된다.
    public Object validationAdvice(ProceedingJoinPoint joinPoint) throws Throwable {
        Object[] args = joinPoint.getArgs(); // joinPoint 의 매개변수
        for (Object arg : args) {
            if(arg instanceof BindingResult){ // 매개변수중 BindingResult 가 있다면(유효성검사를 하는 메서드)
                BindingResult bindingResult = (BindingResult) arg; // Object -> BindingResult 다운캐스팅

                if(bindingResult.hasErrors()){
                    Map<String, String> errorMap = new HashMap<>();

                    for (FieldError error : bindingResult.getFieldErrors()) {
                        errorMap.put(error.getField(), error.getDefaultMessage());
                    }
                    throw new CustomValidationException("유효성검사 실패", errorMap); // 직접 만든 예외처리 로직실행
                }
            }
        }
        return  joinPoint.proceed(); // 정상적으로 해당 메서드를 실행
    }
}
```

## 관련 문서

- 상위 목차: [[Junit 중급강의 - 시큐리티를 활용한 Bank 애플리케이션]]
- 다음 문서: [[CORS 테스트]]
