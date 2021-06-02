---
description: '2021.06.01, 화요일'
---

# 웹 계층 개발 \(회원, 상품\)

### 강의 항목

* 홈 화면과 레이아웃
* 회원 등록
* 회원 목록 조회
* 상품 등록
* 상품 목록
* 상품 수정



### 실습 코드

* [https://github.com/conquerex/WhatTheJpa2nd/commit/205fa47cf1eb3d88735e9fc171dfbdb1a6f9957b](https://github.com/conquerex/WhatTheJpa2nd/commit/205fa47cf1eb3d88735e9fc171dfbdb1a6f9957b)
* \(하나더...\)



### 학습 내용

* 타임리프\(Thymeleaf\) 관련된 내용은 생략 😉
* 컨트롤러 개발
* 스프링 부트 타임리프 기본 설정
  * 그런데 이거 없이도 잘 돌아간다????

```text
# application.yml
spring:
  thymeleaf:
    prefix: classpath:/templates/
    suffix: .html
```

* 부트스트랩
  * [CSS 세팅](https://getbootstrap.com/docs/5.0/getting-started/introduction/) 확인해야 함
  * resources/static 하위에 css , js 추가

```text
<link href="https://cdn.jsdelivr.net/npm/bootstrap@5.0.1/dist/css/bootstrap.min.css" rel="stylesheet" integrity="sha384-+0n0xVW2eSR5OomGNYDnhzAbDsOXxcvSN1TPprVMTNDbiYZCxYbOOl7+AMvyTG2x" crossorigin="anonymous">
```

* resources/static/css/jumbotron-narrow.css 추가
* 회원 등록 폼 추가
  * 엔티티와 따로 쓰는 것이 좋다.
  * 엔티티가 화면 종속적이 되면 유지보수가 어려워진다.
  * 엔티티는 핵심 비즈니스 로직만
  * 화면이나 API에 맞는 폼 객체나 DTO를 사용하





