  

==_**JavaScript 로 프론트에서 JWT 토큰을 다룰수 있게 CORS 설정을 하여 테스트를 해보자.**_==

  

- SecurityConfig 에서 수정이 필요하다.
- 예전 CORS 정책에서는 `addExposeHeader` 가 디폴트였는데 이제는 따로 설정을 해줘야 한다.
    
    ```java
    @Slf4j
    @Configuration
    public class SecurityConfig {
    
        // 생략 ..
    
        public CorsConfigurationSource configurationSource() {
            CorsConfiguration configuration = new CorsConfiguration();
            configuration.addAllowedHeader("*"); // 모든 헤더허용
            configuration.addAllowedMethod("*"); // GET, POST, PUT, DELETE (Javascript 요청 허용)
            configuration.addAllowedOriginPattern("*"); // 모든 주소 허용
            configuration.setAllowCredentials(true); // 클라이언트에서 쿠키 요청 허용
            configuration.addExposedHeader("Authorization"); // 추가 !!!
            UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
            source.registerCorsConfiguration("/**", configuration); // 어떤 주소가 요청이 와도 위에 설정들을 다 셋팅한다.
    
            return source;
        }
    }
    ```
    
- 그리고 간단한 LoginForm 을 만들어서 테스트 해보면 된다.
    
    ```html
    <!doctype html>
    <html lang="en">
    <head>
        <meta charset="UTF-8">
        <meta name="viewport" content="width=device-width, user-scalable=no, initial-scale=1.0, maximum-scale=1.0, minimum-scale=1.0">
        <meta http-equiv="X-UA-Compatible" content="ie=edge">
        <title>Document</title>
    </head>
    <body>
        <h1>로그인 페이지</h1>
        <hr>
        <form>
            <input type="text" id="username"><br/>
            <input type="password" id="password"><br/>
            <button type="button" onclick="login()">로그인</button>
        </form>
    
        <script>
            // async : await 지점을 기억한 채로 login() 함수의 스택을 빠져나온다.
            async function login() {
                let userDto = {
                    username : document.querySelector("\#username").value,
                    password:document.querySelector("\#password").value
                }
                console.log(userDto);
                // 통신(시간이 걸린다)
                
                let userJson = JSON.stringify(userDto);
                console.log(userJson);
    
                let r1 = await fetch("http://localhost:8081/api/login", {
                    method: "post",
                    body: userJson,
                    headers: {
                        "Content-Type": "application/json; charset=utf-8"
                    }
                });
    
                console.log("Authorization", r1.headers.get("Authorization"));
                let r2 = await r1.json();
                console.log(r2);
            }
        </script>
    </body>
    </html>
    ```