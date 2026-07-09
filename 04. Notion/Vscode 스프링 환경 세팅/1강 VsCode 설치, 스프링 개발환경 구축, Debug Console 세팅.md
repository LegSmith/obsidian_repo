## 1. Vscode 설치하기

![[Untitled 51.png|Untitled 51.png]]

  

## 2. 프로젝트 워크스페이스 설정하기

```dart
c:\code_lab\spring
c:\code_lab\flutter
c:\code_lab\react
```

  

## 3. 워크스페이스 폴더 열기

![[Untitled 1 37.png|Untitled 1 37.png]]

  

## 4. 스프링부트 프로젝트 생성하기

![[Untitled 2 25.png|Untitled 2 25.png]]

![[Untitled 3 16.png|Untitled 3 16.png]]

```dart
Spring initializer: create a gradle proejct
2.7.7
java
shop.mtcoding
demo
jar
11
Srping Boot DevTools, Spring Web, Lombok

c:\code_lab\spring 해당 경로에 프로젝트 만들기
```

  

## 5. debug console 세팅

```dart
File - Preferences - Settings - Console검색 - internalConsole 선택
```

![[Untitled 4 14.png|Untitled 4 14.png]]

## 6. application.yml 설정

```yaml
spring:
  output:
    ansi:
      enabled: always

logging:
  level:
    '[shop.mtcoding.demo]': DEBUG
```

  

## 7. [IndexController.java](http://IndexController.java) 생성 및 실행

```java
package shop.mtcoding.demo;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

import lombok.extern.slf4j.Slf4j;

@Slf4j
@RestController
public class IndexController {

    @GetMapping("/index")
    public String index() {
        log.debug("디버그");
        log.info("인포");
        log.warn("경고");
        log.error("에러");
        return "index ok";
    }
}
```

![[Untitled 5 13.png|Untitled 5 13.png]]

![[Untitled 6 11.png|Untitled 6 11.png]]