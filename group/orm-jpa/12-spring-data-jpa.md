# 12장. 스프링 데이터 JPA

## 입구

- 대부분의 데이터 접근 계층 Data Access Layer
    - 일명 CRUD로 부르는 유사한 등록, 수정, 삭제, 조회코드를 반복해서 개발해야
    - JPA도 마찬가지
- 각 레포지토리가 하는 일이 비슷
    - 제네릭과 상속을 적절히 사용해서 공통 부분을 처리하는 부모 클래스를 만들면 된다
    - GenericDAO
    - But 공통 기능을 구현한 부모 클래스에 너무 종속적, 구현 클래스 상속이 가지는 단점에 노출

## 1. 스프링 데이터 JPA 소개

- 스프링 프레임워크에서 JPA를 편리하게 사용할 수 있도록 지원하는 프로젝트
- 우선 CRUD를 처리하기 위한 공통 인터페이스를 제공
- 리포지토리를 개발할 때 인터페이스만 작성하면
    - 실행시점에 스프링 데이터 JPA가 구현 객체를 동적으로 생성해서 주입
- 데이터 접근 계층을 개발 할 때 **구현 클래스 없이 인터페이스만 작성해도 개발을 완료할 수** 있다.
- org.springframework.data.jpa.repository.JpaRepository 인터페이스
- 일반적인 CRUD 메소드는 JpaRepository 인터페이스가 공통으로 제공하므로 문제없음
    - MemberRepository.findByUsername(...)처럼 직접 작성한 공통으로 처리할 수 없는 메소드??
    - 스프링 데이터 JPA는 메소드 이름을 분석해서 JPQL을 실행
- 스프링 데이터 프로젝트
    - 스프링 데이터 JPA는 하위 프로젝트 중 하나
    - JPA, 몽고DB, NEO4J, REDIS, HADOOP, GEMFIRE
    - 이와 같은 다양한 데이터 저장소에 대한 접근을 추상화해서
    - 개발자 편의를 제공하고 지루하게 반복하는 데이터 접근 코드를 줄여준다.

## 2. 스프링 데이터 JPA 설정

- 필요 라이브러리

    ```
    <dependency>
        <groupId>org.springframework.data</groupId>
        <artifactId>spring-data-jpa</artifactId>
        <version>2.4.2</version>
    </dependency>

    // gradle
    org.springframework.boot:spring-boot-starter-data-jpa
    ```

- 환경설정
    - xml파일
        - `<jpa:repositorise>`를 사용하고 repository를 검색할 base-package를 적는다.
    - JavaConfig
        - EnableJpaRepositories 어노테이션 추가, 리포지토리를 검색할 패키지 위치도 작성

        ```java
        import org.springframework.data.jpa.repository.config.EnableJpaRepositories;

        @Configuration
        @EnableJpaRepositories(basePackages = "jpabook.jpashop.repository")
        public class AppConfig {}
        ```

    - 스프링 데이터 JPA는 어플리케이션 실행 시
        - basePackage에 있는 레퍼지토리 인터페이스들을 찾아서
        - 해당 인터페이스를 구현한 클래스를 동적으로 생성한 다음
        - 스프링 빈으로 등록한다.

## 3. 공통 인터페이스 기능

- 스프링 데이터 JPA를 사용하는 가장 단순한 방법은 `JpaRepository 인터페이스`를 상속받는 것
- 그리고 제네릭에 엔티티 클래스와 엔티티 클래스가 사용하는 식별자 타입을 지정하면 된다
- JpaRepository 인터페이스 계층구조
    - 스프링 데이터
        - Repository
        - CrudRepository
        - PagingAndSrotingRepository
    - 스프링 데이터 JPA
        - JpaRepository
- 스프링 데이터 모듈
    - Repository ~ PagingAndSrotingRepository
    - 스프링 데이터 프로젝트가 공통으로 사용하는 인터페이스
    - 스프링 데이터 JPA가 제공하는 JpaRepository 인터페이스는 여기서 추가로 JPA에 특화된 기능을 제공
- JpaRepository 인터페이스 상속시 사용할 수 있는 주요 메소드
    - save(엔티티와 그 자식 타입)
    - delete(엔티티) : 엔티티 하나 삭제, em.remove()
    - findOne(id) : 엔티티 하나를 조회, em.find()
    - getOne(id) : 엔티티를 프록시로 조회, em.getReference()
    - findAll(...) : 모든 엔티티를 조회, 정렬이나 페이징 조건을 파라미터로 제공할 수 있다
    - save의 경우 엔티티에 식별자 값이 없으면 (null)
        - 새로운 엔티티로 판단해서 em.persist() 호출
        - 있으면 이미 있는 엔티티로 판단, em.merge를 호출
        - 필요하다면 스프링 데이터 JPA의 기능을 확장해서 신규 엔티티 판단 전략을 변경할 수 있다

## 4. 쿼리 메소드 기능

- 예) 메소드 이름만으로 쿼리를 생성하는 기능
    - 인터페이스에 메소드만 선언하면 해당 메소드의 이름으로 적절한 JPQL 쿼리를 생성해서 실행
- 쿼리 메소드 기능 3가지
    - 메소드 이름으로 쿼리 생성
    - 메소드 이름으로 JPA NamedQuery 호출
    - @Query 어노테이션을 사용해서 리포지토리 인터페이스에 쿼리 직접 정의
- 첫번째. 메소드 이름으로 쿼리 생성
    - 정해진 규칙에 따라서 메소드 이름을 지어야
        - 스프링 데이터 JPA 공식 문서가 제공하는 표를 보면 어떻게 사용해야 하는지 쉽게 이해 가능
        - and, or, is, equas, between
        - lessThan, lessThanEqual, greaterThan
        - after, before
        - isNull, isNotNull, notNull
        - like, notLike
        - StartingWith, EndingWith
        - OrderBy
        - Not, in, notIn
        - ture, false
        - ignoreCase (대문자로만 비교)
    - 엔티티 필드명이 변경되면 메소드 이름도 꼭 함께 변경해야
- 두번째. 메소드 이름으로 JPA NamedQuery 호출
    - JPA NamedQuery 쿼리
        - 쿼리에 이름을 부여해서 사용하는 방법
        - 어노테이션 혹은 xml에 쿼리를 정의할 수 있다
        - 자세한건 10.2.15절로 긔
        - 같은 방법으로 Named 네이티브 쿼리도 지원
    - Named 쿼리를 JPA에서 직접 호출하려면

        ```java
        List<Member> resultList =
             em.createNamedQuery("Member.findByUsername", Member.class)
                .setParameter("username", "corn")
                .getResultList();
        ```

    - 스프링 데이터 JPA를 사용하면 메소드 이름만으로 Named 쿼리를 호출할 수 있다

        ```java
        public interface MemberRepository extends JpaRepository<Member, Long> { // 여기 선언한 Member 도메인 클래스
            List<Member> findByUsername(@Param("username") String username);
        }
        ```

    - 선언한 “도메인 클래스 + . + 메소드 이름” 으로 Named 쿼리를 찾아서 실행
    - Member.findByUsername Named Query
    - 만약, 실행한 Named Query가 없다면 메소드 이름으로 쿼리 생성 전략 사용
    - @Param 어노테이션 : 이름기반 파라미터를 바인딩할 때 사용하는 어노테이션, 뒤에서 자세히 다룸
- 세번째. `@Query 어노테이션`을 사용해서 리포지토리 인터페이스에 쿼리 직접 정의
    - @org.springframework.data.jpa.repository.Query 어노테이션
    - 실행할 메소드에 정적 쿼리를 직접 작성하므로 이름 없는 Named 쿼리라 할 수 있다.
    - 어플리케이션 실행 시점에 문법 오류를 발견할 수 있는 장점이 있다.
    - 네이티브 SQL을 사용하려면 Query 어노테이션에 nativeQuery = true를 설정한다.
- 파라미터 바인딩
    - 위치 기반 파라미터 바인딩과 이름 기반 파라미터 바인딩 모두 지원한다.
        - 위치 기반 : select m from Member m where m.usernmae = ?1
        - 이름 기반 : select m from Member m where m.usernmae = :name
    - 기본값 : 위치 기반
    - 이름 기반 파라미터 바인딩 사용하려면
        - org.springframework.data.repository.query.Param(파라미터 이름) 어노테이션 사용
    - 권장 : 코드 가독성과 유지보수를 위해 이름 기반 파라미터 바인딩을 사용
- 벌크성 수정 쿼리
    - 기존에는 excuteUpdate 메소드를 사용한 벌크성 쿼리
    - 벌크성 수정, 삭제 쿼리는 Modifying 어노테이션을 사용하면 된다.
    - 벌크성 쿼리 실행 후 영속성 컨텍스트를 초기화 하고 싶으면
        - @Modifying(clearAutomatically = true)처럼 clearAutomatically 옵션을 true로 설정하면 된다. (default 는 false)
- 반환 타입
    - 유연한 반환타입을 지원
    - 결과가 한 건 이상이면 컬렉션 인터페이스를 사용
    - 결과가 단 건이면 반환 타입을 지정
    - 조회결과가 없는 경우
        - 컬렉션 - 빈 컬렉션 반환
        - 단건 - null 반환
    - 단 건으로 지정했는데 결과가 2건 이상인 경우
        - NonUniqueResultException 예외 발생
        - 스프링 데이터 JPA는 단 건을 조회 할 때 이 예외가 발생하면 예외를 무시하고 대신에 null 반환
        - 원래는 JPQL의 Query.getSingleResult 메소드의 결과로 NoResultException 발생 (다루기 불편)
- 페이징과 정렬
    - org.springframework.data.domain.Sort: 정렬 기능
    - org.springframework.data.domain.Pageable: 페이징 기능(내부에 Sort 포함)
        - 파라미터에 Pageable을 사용하면 반환 타입으로 List나 Page를 사용할 수 있다.
    - 반환 타입으로 Page를 사용하면 스프링 데이터 JPA는 페이징 기능을 제공하기 위해 검색된 전체 데이터 건수를 조회하는 count 쿼리를 추가로 호출
    - Pagable 인터페이스를 구현한 PageRequest 객체 이용
- 힌트
    - QueryHints 어노테이션을 사용
    - SQL 힌트가 아니라 JPA 구현체에게 제공하는 힌트
    - forCounting 속성
        - 반환타입으로 Page 인터페이스를 적용하면
        - 추가로 호출하는 페이징을 위한 count 쿼리에도 쿼리 힌트를 적용할지를 설정하는 옵션
- Lock
    - 쿼리 시 락을 걸려면 Lock 어노테이션을 사용
    - JPA가 제공하는 락에 대해서는 16.1절을 참고


## 5. 명세

- 스프링 데이터 JPA는 JPA Criteria로 **`명세(Specification)`**를 사용할 수 있도록 지원
- 명세를 이해하기 위한 핵심 단어
    - 술어(predicate) : 단순히 참이나 거짓으로 평가
    - AND, OR 같은 연산자로 조합할 수 있다
    - 데이터를 검색하기 위한 제약 조건 하나하나를 술어라 할 수 있다
    - 스프링 데이터 JPA는 Specification 클래스로 정의
- Specification은 컴포지트 패턴으로 구성
    - 여러 Specification을 조합할 수 있다
    - 따라서 다양한 검색조건을 조립해서 새로운 검색조건을 쉽게 만들 수 있다.
- 명세 기능을 사용하려면 JpaSpecificationExecutor 인터페이스를 상속받으면 된다
    - JpaSpecificationExecutor의 메소드들은 Specification을 파라미터로 받아서 검색 조건으로 사용
    - Specifications는 명세들을 조립할 수 있도록 도와주는 클래스
        - where(), and(), or(), not() 메소드를 제공
        - 사견 : 크리테리아를 사용한다면 활용도가 높아 보임. 크리테리아를 사.용.한.다.면
        - 명세를 정의하려면 Specification 인터페이스를 구현하면 된다.
        - 사견2 : 사용할 생각이 없어서 뒤 내용은 생략

## 6. 사용자 정의 리포지토리 구현

- 스프링 데이터 JPA로 리포지토리를 개발하면 인터페이스만 정의
    - 구현체는 만들지 않는다.
    - 하지만 메소드를 직접 구현해야 할 때는 사용자 정의 인터페이스를 작성해야
- 사용자 정의 리포지토리 구현
    - 인터페이스 이름은 자유롭게
    - 사용자 정의 인터페이스를 구현한 클래스를 작성해야
    - 클래스 이름은 {리포지토리 인터페이스 이름} + Impl로 지어야
        - 스프링 데이터 JPA가 사용자 정의 구현 클래스로 인식
    - 마지막으로 리포지토리 인터페이스에서 사용자 저의 인터페이스를 상속받으면 된다
    - 사용자 정의 구현 클래스 이름 끝에 Impl 대신 다른 이름을 붙이고 싶으면
        - repository-impl-postfix 속성을 변경하면 된다. 참고로 Impl이 기본값이다.

## 7. Web 확장

- 스프링 데이터 프로젝트는 스프링 MVC에서 사용할 수 있는 편리한 기능을 제공
    - 도메인 클래스 컨버터 기능 : 식별자로 도메인 클래스를 바로 바인딩
    - 페이징과 정렬 기능 제공
- 설정
    - Web 확장 기능 활성화 : org.springframework.data.web.config.SpringDataWebConfiguration을 스프링 빈으로 등록
    - JavaConfig를 사용 : org.springfrramework.data.web.config.EnableSpringDataWebSupport 어노테이션을 사용

        ```java
        @Configuration
        @EnalbeWebMvc
        @EnableSpringDataWebSupport
        public class WebAppConfig {
        		...
        }
        ```

    - 설정을 완료하면 도메인클래스 컨버터와 페이징과 정렬을 위한 HandlerMethodArgumentResolver가 스프링 빈으로 등록된다
    - 등록되는 도메인 클래스 컨버터
        - org.springframework.data.repository.support.DomainClassConverter
- 도메인 클래스 컨버터 기능
    - 도메인 클래스 컨버터
        - HTTP 파라미터로 넘어온 엔티티의 아이디로 엔티티 객체를 찾아서 바인딩

    ```java
    @Controller
    	public class MemberController {

    		@RequsetMapping("member/memberUpdateFrom")
    		public String memberUpdateForm(@RequsetParam("id") Member member, Model model) {
    				model.addAttribute("member", member);
    				return "member/memberSaveForm";
    		}
    }
    ```

    - @RequestParam 부분
        - HTTP 요청으로 회원 아이디를 받지만
        - 도메인 클래스 컨버터가 중간에 동작해서 아이디를 회원 엔티티 객체로 변환해서 넘겨줌
        - 컨트롤러를 단순하게 사용할 수 있다
    - 도메인 클래스 컨버터는 해당 엔티티와 관련된 리포지토리를 사용해서 엔티티를 찾는다
        - 당연한 얘기겠지만... 회원 리포지토리를 통해서 회원 아이디로 회원 엔티티를 찾는다
    - 도메인 클래스 컨버터를 통해 넘어온 회원 엔티티를 컨트롤러에서 직접 수정하면?
        - 실제 데이터 베이스에는 반영되지 않음
        - 스프링 데이터와는 관련 없음
        - 웹 애플리케이션에서 영속성 컨텍스트의 동작방식과 OSIV에 관한 내용은 13장에서
    - OSIV를 사용하지 않으면
        - 조회한 엔티티는 준영속 상태다.
        - 따라서 변경 감지기능이 동작하지 않는다.
        - 만약 **수정한 내용을 데이터베이스에 반영하고 싶으면 병합(merge)을 사용**해야 한다.
    - OSIV를 사용하면
        - 조회한 엔티티는 영속 상태다.
        - 하지만 OSIV의 특성상 컨트롤러와 뷰에서는 영속성 컨텍스트를 플러시하지 않는다.
        - 따라서 수정한 내용을 데이터베이스에 반영하지 않는다.
        - 만약 **수정한 내용을 데이터베이스에 반영하고 싶으면 트랜잭션을 시작하는 서비스 계층을 호출해야** 한다.
        - 해당 서비스 계층이 종료될 때 플러시와 트랜잭션 커밋이 일어나서 영속성 컨텍스트의 변경 내용을 데이터베이스에 반영해 줄 것이다
- 페이징과 정렬 기능
    - 스프링 MVC에서 편리하게 사용할 수 있도록 `HandlerMethodArgumentResolver`를 제공
    - 페이징 기능 : PageableHandlerMetodArgumentResolver
    - 정렬 기능 : SortHandlerMethodArgumentResolver
    - ex) /members?page=0&size=20&sort=name,desc&sort=address.city
    - 파라미터로 Pageable(인터페이스, 실제로는 PageRequest 객체가 생성됨)을 받을 수 있다
        - page : 현재 페이지, 0부터 시작
        - size : 한 페이지에 노출할 데이터 건수
        - sort : 정렬 조건, sort 파라미터 추가 가능
    - page를 1부터 시작하고 싶으면
        - PageableHandlerMetodArgumentResolver를 스프링 빈으로 직접 등록하고
        - setOneIndexedParameters를 true
    - 접두사 : 사용해야 할 페이징 정보가 둘 이상일 때
        - 스프링 프레임워크가 제공하는 @Qualifier 어노테이션 사용
        - "{접두사명}_"로 구분
    - 기본값 : page=0, size=20
        - 만약 기본값을 변경하고 싶으면 @PageableDefault 어노테이션을 사용하면 된다.

        ```java
        @RequsetMapping(value = "/member_page", method = RequsetMethod.GET)
        public String list(@PageableDefault(size = 12, sort = "name", direction = Srot.Direction.DESC) Pageable pageable) {
            ...
        }
        ```


## 8. 스프링 데이터 JPA가 사용하는 구현체

- 스프링 데이터  JPA가 제공하는 공통 인터페이스
    - `SimpleJpaRepository` 클래스가 구현
- @Repository 적용
    - JPA 예외를 스프링이 추상화한 예외로 변환한다.
- @Transactional 트랜잭션 적용
    - JPA의 모든 변경은 트랜잭션 안에서 이루어져야 한다.
    - 스프링 데이터 JPA가 제공하는 공통 인터페이스 사용시 데이터 변경하는 메소드에 트랜잭션 처리가 됨
    - 서비스 계층에서 트랜잭션을 시작하지 않으면 리포지토리에서 트랜잭션을 시작
    - 물론 서비스 계층에서 트랜잭션을 시작했으면 리포지토리도 해당 트랜잭션을 전파받아서 그대로 사용
- @Transactional(readOnly = true)
    - 데이터를 조회하는 메소드에는 readOnly = true 옵션이 적용되어 있다
    - 데이터를 변경하지 않는 트랜잭션에서 readOnly 옵션을 사용하면 플러시를 생략
    - 약간의 성능향상을 얻을 수 있다. 자세한건 15.4.2절에서.
- save() 메소드
    - 저장할 엔티티가 새로운 엔티티면 저장(persist)하고
    - 이미 있는 엔티티면 병합(merge)한다.
    - 새로운 엔티티를 판단하는 기본 전략 : 엔티티의 식별자로 판단
        - 식별자가 객체일 때 null
        - 식별자가 자바 기본 타입일 때 숫자 0이면 새로운 엔티티로 판단
    - Persistable 인터페이스를 구현해서 판단 로직을 변경할 수 있다

## 9. JPA 샵에 적용

- 생략

## 10. 스프링 데이터 JPA와 QueryDSL 통합

- 스프링 데이터 JPA는 2가지 방법으로 QueryDSL을 지원한다.
    - org.springframework.data.querydsl.QueryDslPredicateExecutor
    - org.springframework.data.querydsl.QueryDslRepositorySupport
- QueryDslPredicateExecutor
    - 리포지토리에서 상속을 받으면 QueryDSL을 사용할 수 있다
    - QueryDSL을 검색조건으로 사용하면서 스프링 데이터 JPA가 제공하는 페이징과 정렬 기능도 함께 사용할 수 있다
    - 편리하게 QueryDSL을 사용할 수 있지만 기능에 한계가 존재
        - join, fetch등을 사용할 수 없다 (JPQL에서 이야기하는 묵시적 조인은 사용가능)
    - QueryDSL이 제공하는 다양한 기능을 사용하려면
        - JPAQuery를 직접 사용하거나
        - 스프링 데이터 JPA가 제공하는 QueryDslRepositorySupport를 사용해야
- QueryDslRepositorySupport
    - QueryDSL의 모든 기능을 사용하려면
        - JPAQuery 객체를 직접 생성해서 사용
        - 혹은 QueryDslRepositorySupport를 상속받아 조금 더 편리하게 사용
    - 스프링 데이터 JPA가 제공하는 공통 인터페이스는 직접 구현 불가
        - 사용자 정의 리포지토리를 만들고 이 인터페이스를 상속받는 동시에
        - QueryDslRepositorySupport를 상속받는 구현 클래스를 생성
        - 생성자에서 QueryDslRepositorySupport에 엔티티 클래스 정보를 넘겨주어야 한다

## 정리

- 지루한 데이터 접근 계층의 코드가 상당히 많이 줄일 수 있다
- 스프링 프레임워크와 JPA를 함께 사용한다면 스프링 데이터 JPA는 선택이 아닌 필수
