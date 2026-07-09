  

## Long 타입 비교연산자 테스트

---

- `longValue()` 를 쓰지 않고 ==_**일반적인 비교연산자를 사용할 경우 같은 1000L 인데도 다르다는 결과가 나온다 !!**_==
- 즉, ==**Long 타입을 비교**==할 때는 `longValue()` 를 사용하자 
    
    ```java
    @Test
    public void long_test() throws Exception {
        //given
        Long number1 = 1111L;
        Long number2 = 1111L;
    
        //when
        if (number1.longValue() != number2.longValue()) {
            System.out.println("동일하지 않습니다.");
        } else {
            System.out.println("동일합니다.");
        }
    
        Long amount1 = 1000L;
        Long amount2 = 1000L;
    
        if(amount1 == amount2){
            System.out.println("같습니다.");
        }else{
            System.out.println("다릅니다.");
        }
    
        //then
    }
    ```
    
    ![[Untitled 16.png|Untitled 16.png]]
    

  

## Long 타입 테스트 2

---

- Java 에서 Long 타입 비교를 할 때 `-128 ~ 127` 범위 까지는 `==`(비교연산자)로 비교가 가능하지만, 범위 이상 비교는 잘못된 결과가 나온다.
- 제일 좋은건 `equals()` 를 사용하든 `longValue()` 를 사용하는게 좋다 !