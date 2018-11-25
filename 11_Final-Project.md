![](./assets/spring-logo.png)

***

# 7. 회원전용 게시판 프로젝트

앞에서 배운 내용을 종합적으로 사용하는 간단한 프로젝트를 만들어 보자. 구현할 기능들은 다음과 같다.

* 첫 화면
* 회원가입
* 로그인
* 게시판
  * 페이징 기반 글 목록 조회 기능을 제공한다.
  * 글 상세 조회 기능을 제공한다.
  * 새 글 작성 기능을 제공한다. 새 글을 작성하려면 로그인을 선행해야 한다.
  * 수정, 삭제 기능을 제공한다. 글 수정, 삭제는 작성자만 가능하다.
  * 비 회원은 글 목록 조회, 글 상세 조회만 가능하다.

다음 소스를 참고하자.
https://github.com/softcontext/spring-board