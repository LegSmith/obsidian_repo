## 정적 컨텐츠

- 정적컨텐츠는 resources/static 폴더안에 넣으면 스프링부트가 인식한다.
- 단, 정적 컨텐츠이기 때문에 프로그래밍은 못한다.


## MVC와 템플릿 엔진

- Controller : 비즈니스 로직에만 몰두
- View : 화면UI에만 몰두

## API

- `@ResponseBody`를 사용할 경우에는 `HttpMessageConverter` 가 직접 클라이언트에 반환한다.
- View로 통신할 때는 `viewResolver`view를 찾아서 클라이언트에 반환한다.

## 관련 문서

- 상위 목차: [[스프링 입문]]
- 이전 문서: [[AOP]]
- 다음 문서: [[Section 4(스프링 빈과 의존관계)]]
