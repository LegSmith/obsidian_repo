  

# 스프링 빈과 의존관계

> 스프링 빈에 등록을 해야 스프링 프레임워크가 여러가지 동작을 취할 수 있다.

  

### @Autowired 어노테이션

- 스프링이 직접 연관된 객체를 스프링 컨테이너에서 찾아서 주입해준다.
- 스프링 컨테이너에 등록을 해주지 않으면 해당 객체를 못찾기 때문에 아래와 같은 에러코드를 볼 수 있다.
    
    `Consider defining a bean of type 'hello.hellospring.service.MemberService' in your configuration.`
    
- 스프링 프레임워크와 같이 외부에서 객체 의존관계를 넣어주는 것을 ==**DI(Dependency Injection), 의존성 주입이라 한다.**==

  

## 스프링 빈을 등록하는 2가지 방법

**1) 컴포넌트 스캔과 자동 의존관계 설정**  
**2) 자바 코드로 직접 스프링 빈 등록하기**

### 컴포터는 스캔 원리

- 원래 `@Component` 어노테이션을 통해서 스프링 빈을 등록된다.
- 하지만 아래와 같은 어노테이션들을 사용하는 이유는 `@Component`를 내장하고 있기 때문이다.
    - `@Controller`
    - `@Service`
    - `@Repository`

  

### 자바코드로 스프링 빈 등록하기

> 자바코드로 등록하기 위해서는 패키지 내부에 스프링 빈을 등록하기 위한 클래스를 생성해야 한다.

- SpringConfig라는 클래스를 생성한다.(패키지 외부에 하면 안된다 !)
    
    ![[Untitled 41.png|Untitled 41.png]]
    
    ![[Untitled 41.png|Untitled 41.png]]
    
- 아래와 같이 `@Bean` 어노테이션을 이용하여 직접 적어주면 된다.
- 나중에 상황에 따라 구현 클래스(MemoryMemberRepository)를 변경해야 하면 여기서 수정하면 된다.
    
    ```java
    package hello.hellospring;
    
    import hello.hellospring.repository.MemberRepository;
    import hello.hellospring.repository.MemoryMemberRepository;
    import hello.hellospring.service.MemberService;
    import org.springframework.context.annotation.Bean;
    import org.springframework.context.annotation.Configuration;
    
    @Configuration
    public class SpringConfig  {
    
        @Bean
        public MemberService memberService(){
            return new MemberService(memberRepository());
        }
    
        @Bean
        public MemberRepository memberRepository(){
            return new MemoryMemberRepository();
        }
    }
    ```
    

  

  

참고)  
Spring Framework은 메인메서드가 있는 패키지 내부에서만 동작한다.  
외부 패키지에 클래스에서 `@Component`를 적어줘도 스프링은 외부패키지에서 컴포넌트 스캔을 하지 않는다!!