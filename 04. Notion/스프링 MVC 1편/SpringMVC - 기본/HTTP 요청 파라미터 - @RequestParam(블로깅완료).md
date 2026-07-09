  

  

## 1. @RequestParam(”key”)

- 쿼리스트링에 있는 파라미터명을 key로 받아서 저장

```java
@RequestMapping("/request-param-v2")
    public String requestParamV2(
            @RequestParam("username") String memberName,
            @RequestParam("age") int memberAge){

        log.info("username={}, age={}", memberName, memberAge);

        return "ok";
    }
```

- 파라미터명과 변수명이 같다면 key 생략 가능

```java
@RequestMapping("/request-param-v3")
    public String requestParamV3(
            @RequestParam String username,
            @RequestParam int age){

        log.info("username={}, age={}", username, age);

        return "ok";
    }
```

  

  

## 2. @RequestParam - required , defalutValue 속성

- null 값을 받지 못하게 required 속성을 줄 수 있다.( defalut 값은 true이다)

```java
@RequestMapping("/request-param-required")
    public String requestParamRequired(
            @RequestParam(required = true) String username, //쿼리스트링에서 username은 필수로 입력해야한다.
            @RequestParam(required = false) int age){
        log.info("username={}, age={}", username, age);

        return "ok";
    }
```

- 단, 빈값은 허용하게 된다.( null 과 빈값은 엄연히 다르다!! )
- 빈값이 왔을때 defalut 값을 지정할 수 있다. 이때 쓰는 속성이 defalutValue 이다

```java
@RequestMapping("/request-param-defalut")
    public String requestParamDefalut(
            @RequestParam(defaultValue = "guest") String username, //빈값이 들어올때 기본값을 정해줌.
            @RequestParam(defaultValue = "-1") int age){ // defalutValue 를 쓰면 required 속성을 안써도된다.
        log.info("username={}, age={}", username, age);

        return "ok";
    }
```

  

## 3. Map 을 이용하여 하나로 통일

- Map 을 이용하여 @RequestParam 을 1번만 쓸 수 있다
- 단, 2개인 경우만 Map 이고 , 하나의 키값으로 여러개를 받을 경우 MultiValueMap사용가능

```java
@RequestMapping("/request-param-map")
    public String requestParamMap(@RequestParam Map<String, Object> paramMap) {

        log.info("username={}, age={}", paramMap.get("username"),paramMap.get("age"));

        return "ok";
    }
```