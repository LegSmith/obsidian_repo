  

매핑종류에는 GET(조회), POST(등록), PATCH(수정), DELETE(삭제) 메서드를 이용하여 매핑한다.

  

- 중복되는 url 은 @RequestMapping(”중복url”) 어노테이션을 클래스밖으로 빼주어 매핑을 간결하고 가독성을 높일 수 있다.

```java
package hello.springmvc.basic.requestmapping;

import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping("/mapping/users")
public class MappingClassController {


    /**
     * 회원 목록 조회: GET    '/users'
     * 회원 등록:      POST   '/users'
     * 회원 조회:      GET    '/users/{userId}'
     * 회원 수정:      PATCH  '/users/{userId}'
     * 회원 삭제:      DELETE '/users/{userId}'
     */

    @GetMapping
    public String uers(){
        return "get users";
    }

    @PostMapping
    public String addUser(){
        return "post user";
    }

    @GetMapping("/{userId}")
    public String findUser(@PathVariable String userId){
        return "get userId = "+userId;
    }

    @PatchMapping("/{userId}")
    public String updateUser(@PathVariable String userId){
        return "update userId = "+userId;
    }

    @DeleteMapping("{userId}")
    public String deleteUser(@PathVariable String userId){
        return "delete userId = "+userId;
    }
}
```