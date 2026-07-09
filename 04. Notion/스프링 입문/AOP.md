  

# AOP 적용

==_**AOP : Aspect Oriented Programming**_==

> 공통 관심 사항(cross-cutting concern) vs 핵심 관심 사항(core concern) 분리

  

```java
package hello.hellospring.aop;

import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.Around;
import org.aspectj.lang.annotation.Aspect;
import org.springframework.stereotype.Component;

@Component
@Aspect
public class TimeTraceAop {

    @Around("execution(* hello.hellospring..*(..))")// 공통 관심 사항을 타겟팅
    public Object execut(ProceedingJoinPoint joinPoint) throws  Throwable{
        long start = System.currentTimeMillis();
        System.out.println("START : " + joinPoint.toString()); // 어떤 메서드를 콜하는지 알 수 있다.
        try {
            return joinPoint.proceed();//다음 메서드로 진행
        } finally {
            long finish = System.currentTimeMillis();
            long timeMs = finish-start;
            System.out.println("END : " + joinPoint.toString()+ " " + timeMs + "ms"); // 어떤 메서드를 콜하는지 알 수 있다.
        }

    }
}
```

![[Untitled 42.png|Untitled 42.png]]