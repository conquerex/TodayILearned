# 다음으로

### 강의 항목

* 스프링 데이터 JPA 소개
* QueryDSL 소개



### 실습 코드

* 


### 학습 내용

* 스프링 데이터 JPA
  * [https://spring.io/projects/spring-data-jpa](https://spring.io/projects/spring-data-jpa)
  * 지루하게 반복되는 코드를 자동화
  * 기존 MemberRepository --&gt; OldMemberRepository
  * 새로운 MemberRepository로 스프링 데이터 JPA를 활용
  * findOne --&gt; findById
  * JpaRepository 인터페이스 제공
    * 기본적인 CRUD 기능이 모두 제공
    * findByName처럼 일반화 하기 어려운 기능도 메서드 이름으로 정확한 JPQL 쿼리를 실행
      * select m from member m where m.name = :name
  * 인터페이스만 만들면 된다
    * 구현체는 스프링 데이터 JPA가 애플리케이션 실행시점에 주입
  * But!!! JPA를 사용해서 이른 기능을 제공할 뿐
    * JPA 자체를 잘 이해하는 것이 가장 중요
* QueryDSL
  * [http://www.querydsl.com/](http://www.querydsl.com/)
  * 조건에 따라서 실행되는 쿼리가 달라지는 동적 쿼리를 많이 사용한다
  * build.gradle에 querydsl 추가
    * 추가한 후 반드시 아래 이미지처럼 Gradle탭에서 compileQuerydsl을 실행시켜준다
  * SQL\(JPQL\)과 모양이 유사하면서 자바코드로 동적 쿼리를 편리하게 생성
  * 사용해야 하는 이유
    * 직관적인 문법
    * 컴파일 시점에 빠른 문법 오류 발견
    * 코드 자동완성
    * 코드 재사용
    * JPQL new 명령어와는 비교가 안될 정도로 깔끔한 DTO 조회를 지원
  * Querydsl은 JPQL을 코드로 만드는 빌더 역할을 할 뿐
    * JPA로 애플리케이션을 개발 할 때 선택이 아닌 필수

![](../../.gitbook/assets/image%20%281%29.png)



