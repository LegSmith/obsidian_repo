  

## 정적 리소스

- 고정된 HTML파일, CSS, JS 등
- 주로 웹 브라우저

## HTML 페이지

- 동적으로 필요한 HTML 파일을 생성해서 전달( JSP, 타임리프 등등)
- WAS가 DB를 조회하여 DB에 데이터를 동적으로 HTML에 써서 웹브라우저로 응답

  

## HTTP API

- HTML이 아니라 데이터를 전달
- 주로 JSON형식 사
- 다양한 시스템에서 호

  

# SSR - Server Side Rendering

- ==_**서버**_==에서 최종 HTML을 생성하여 클라이언트에 전달
- 관련기술 : JSP, 타임리프 → 백엔드 개발자
    
    ![[Untitled 73.png|Untitled 73.png]]
    

# CSR - Client Side Rendering

- HTML 결과를 자바스크립트를 사용해 웹 브라우저에서 동적으로 생성
- 동적인 화면에서 주로 사용
- 관련기술 : React, Vue.js → 웹 프론트엔드 개발자

![[Untitled 1 56.png|Untitled 1 56.png]]