  

## 개념

`SpringSecurity` 는 ==**초기화 때 인증/인가와 관련된 여러가지 작업**==들을 한다.  
이런 작업들을 종합적으로 처리하는 2개의 클래스가 있다.  
  
1. `SecurityBuilder` : 웹 보안을 구성하는 ==**빈 객체와 설정클래스들을 생성하는 역할**==을 한다. 대표적으로 `WebSecurity` , `HttpSecurity` 가 있다.  
2. `SecurityConfigurer` : ==**HTTP 요청과 관련된 보안처리를 담당하는 필터들을 생성**==하고 여러 초기화 설정에 관여(SpringSecurity 는 필터기반 보안 프레임워크이다)  
  
위 2개의 클래스를 잘 이해하면 초기화 작업을 알 수 있다!

  

- `SecurityBuilder` 는 `SecurityConfigurer` 를 ==**참조**==하고 있으며 인증 및 인가 초기화 작업은 `SecurityConfigurer` 에 진행된다.
    
    ![[Untitled 87.png|Untitled 87.png]]
    

  

## 초기화 작업이 이루어지는 과정

1. `AutoConfiguration`(자동설정)에 의해서 `SecurityBuilder` 클래스를 생성한다.
2. `SecurityBuilder` 클래스가 `SecurityConfiguer` 타입의 _==**설정클래스들을 생성**==_한다.
3. `SecurityConfiguer` 는 내부적으로 `init()` 메서드와 `configure()` 메서드를 가지고 있다. 이 메서드들은 _==**파라미터로 SecurityBuilder 를 받는다.**==_
4. `SecurityConfigurer` 는 빌더를 파라미터로 받은 `init()` 메서드와 `configure()` 메서드로 ==_**초기화 작업(인증/인가 등)을 진행**_==한다.
    
    ![[Untitled 1 62.png|Untitled 1 62.png]]
    

- 아래 그림을 보면 `HttpSecurity` 라는 빈 객체가 여러 설정클래스들을 생성을 한다. 그리고 ==_**설정클래스들이 초기화 작업을 하면서 여러 필터(Filter)들을 생성**_==한다.
    
    ![[Untitled 2 42.png|Untitled 2 42.png]]