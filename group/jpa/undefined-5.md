---
description: '2021.06.01, 화요일'
---

# 주문 도메인 개발

## 강의 항목

* 주문, 주문상품 엔티티 개발
* 주문 리포지토리 개발
* 주문 서비스 개발
* 주문 기능 테스트
* 주문 검색 기능 개발

## 실습 코드

* [https://github.com/conquerex/WhatTheJpa2nd/commit/be246daf3fe965e75eee39ebc789aa54629e7a59](https://github.com/conquerex/WhatTheJpa2nd/commit/be246daf3fe965e75eee39ebc789aa54629e7a59)
* [https://github.com/conquerex/WhatTheJpa2nd/commit/4071838579e7aa5b80a4380ba279db3b0a8e82e2](https://github.com/conquerex/WhatTheJpa2nd/commit/4071838579e7aa5b80a4380ba279db3b0a8e82e2)

## 학습 내용

* 비즈니스 로직과 생성 메서드 분리
  * 생성 메서드와 함께 protected 기본 생성자 추가
  * 기본 생성자 대신 `@NoArgsConstructor(access = AccessLevel.PROTECTED)` 가능
  * setter 모델 사용을 막고 유지보수를 쉽게\(필드 수정\) 하기 위함
* 도메인 모델 패턴과 트랜잭션 스크립트 패턴
* orderRepository.save\(order\)
  * cascade 덕분에 delivery와 orderItem도 persist됨
* 동적 쿼리 만들기
  * \(비추천\) JPQL로 만들기
  * \(비추천\) JPA Criteria
  * Querydsl 👍. 컴파일 시점에서 이슈를 확인할 수 있다.

