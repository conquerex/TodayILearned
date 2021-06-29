---
description: '2021.06.01, 화요일'
---

# 회원 도메인 개발

## 강의 항목

* 회원 리포지토리 개발
* 회원 서비스 개발
* 회원 기능 테스트

## 실습 코드

* [https://github.com/conquerex/WhatTheJpa2nd/commit/dc95ecb04f440c9e4441f8b645f82b432e298006](https://github.com/conquerex/WhatTheJpa2nd/commit/dc95ecb04f440c9e4441f8b645f82b432e298006)

## 학습 내용

* 스프링 빈 등록
  * @Repository
  * @Service
  * @Controller
* 엔티티 매니저\(Entity Manager\) 주입
  * @PersistenceContext : JPA에서 EM 주입
  * 스프링부터 사용으로 @Autowired으로도 주입 가능
    * @RequiredArgsConstructor
* @Transactional\(readOnly = true\)
  * 영속성 컨텍스트
  * 플러시\(flush\)를 하는지 안하는지를 readOnly로 구분
* @Autowired
  * 생성자 Injection 많이 사용
  * 생성자가 한개일 경우 생략 가능
  * 필드 주입보다 생성자 주입을 사용하자
    * 변경 불가능한 안전한 객체 생성 가능
  * final 키워드 추가시, 컴파일 시점에 생성\(초기화\)하지 않는 오류를 체크 가능
* Lombok
  * 스프링 데이터 JPA를 사용하면 EntityManager도 주입 가능
  * RequiredArgsConstructor
* Spring test
  * 스프링과 통합 테스트 : @RunWith\(SpringRunner.class\)
  * 스프링부트와 테스트 : @SpringBootTest
    * @Autowired 사용 가능
  * @Transactional
    * 각각의 테스트 시작시 트랜잭션 시작
    * 테스트 끝나면 Rollback
    * DB에서 테스트 결과 보고 싶다면 : @Rollback\(value = false\)
  * Fail 테스트
    * then level에서 fail 메소드 작성
    * try-catch 작성
    * @Test\(expected = IllegalStateException.class\) : try-catch가 필요 없어짐
* test/resources/application.yml
  * 테스트에서 스프링 실행시 위 설정 파일을 읽음
  * 혹시 H2 In-Memory용 datasource url이 필요하다면 \(Database URLs\)
    * [https://www.h2database.com/html/cheatSheet.html](https://www.h2database.com/html/cheatSheet.html)
  * 별도의 datasource 설정 없음
    * 기본적인 메모리 DB 사용
    * driver-class도 현재 등록된 라이브러리를 보고 찾아줌
  * 별도의 jpa 설정 없음
    * ddl-auto : create-drop 모로 동작

