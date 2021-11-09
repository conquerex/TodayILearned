# 10장. 객체지향 쿼리 언어

## 3. Criteria

- JPQL을 자바 코드로 작성하도록 도와주는 빌더 클래서 API
    - 문자가 아닌 코드로 JPQL을 작성하므로
        - 문법 오류를 컴파일 단계에서 잡을 수 있고
        - 문자 기반의 JPQL보다 동적 쿼리를 안전하게 생성
    - BUT, 코드가 복잡 + 장황해서 직관적으로 이해하기 힘들다
- 기초
    - javax.persistence.criteria 패키지
    - 단순 조회 JPQL
        - Criteria 쿼리를 생성하려면 먼저 Criteria 빌더(CriteriaBuilder)를 얻어야 한다.
            - 이 빌더는 EntityManager나 EntityMagerFactory에서 얻을 수 있다
        - Criteria 쿼리 빌더에서 Criteria 쿼리(CriteriaQuery)를 생성
            - 이때 반환 타입을 지정할 수 있다
        - from 절을 생성, 반환값은 Criteria에서 사용하는 특별한 별칭
            - 조회 시작점이라는 의미로 쿼리 루트라 한다
        - select 절을 생성
    - Criteria 쿼리를 완성하고 나면 다음 순서는 JPQL과 같다
        - em.createQuery(cq)
    - 검색 조검 + 정렬
        - 검색 : cb.equal(m.get("username"), "회원1")
        - 정렬 : cb.desc(m.get("age"))
        - 만들어둔 조건을 where, orderBy에 넣어서 원하는 쿼리를 생성
    - Criteria는 검색조건부터 정렬까지 Criteria 빌더를 사용해서 코드를 완성
    - 쿼리 루트와 별칭
        - 쿼리 루트 : 조회의 시작점
        - Criteria에서 사용되는 특별한 별칭, JPQL의 별칭과 유사
        - 엔티티에만 부여할 수 있다
    - JPQL을 완성하는 도구이기에 경로 표현식도 있다
        - m.get("team").get("name")
- Criteria 쿼리 생성
    - CriteriaBuilder.createQuery
    - 반환 타입을 지정할수 있을 때와 없을 때(Object)
- 조회
    - 조회 대상을 한건, 여러건 지정
        - cq.select(m)
        - cq.multiselect(m.get("username"), m.get("age"))
        - cq.select(cb.array(m.get("username"), m.get("age"))
    - distinct
        - cq.select(m).distinct(true)
    - NEW, construct()
        - 의미있는 객체로로 반환받고 싶을 때
        - cq.select(cb.construct(MemberDTO.class, m.get("username"), m.get("age")))
    - 튜플
        - Map과 비슷한 튜플이라는 특별한 반환 객체
        - cb.createTupleQuery(), cb.createQuery(Tuple.class)
            - 튜플은 튜플의 검색키로 사용할 튜플 전용 별칭을 필수로 할당해야
                - alias 메소드
            - 선언해둔 튜플 별칭으로 데이터를 조회
        - 이름 기반 - 순서기반의 Object[ ]보다 안전
- 집합
    - group by
        - cq.groupBy(m.get("team").get("name"))
    - having
        - cq.groupBy(m.get("team").get("name")).having(cb.gt(minAge, 10))
- 정렬
    - cb.desc(...), cb.asc(...)
- 조인
    - join 메소드와 JoinType 클래스
    - Join<Member, Team> t = m.join("team", JoinType.INNER)
    - fetch join : m.fetch("team", JoinType.LEFT)
- 서브쿼리
    - 간단한 서브 쿼리
        - mainQuery.subquery(...)
        - 메인 쿼리 생성 부분의 where(..., subQuery)에서 생성한 서브 쿼리를 사용
    - 상호 관련 서브 쿼리
        - 서브쿼리에서 메인 쿼리의 정보를 사용하려면
            - 메인쿼리에서 사용한 별칭을 얻어야
        - 서브 쿼리는 메인 쿼리의 Root나 Join을 통해 생성된 별칭을 받아서 사용
        - .where(cb.equal(subM.get("username"), m.get("username")))
        - correlate() 메소드 : 메인 쿼리의 별칭을 서브쿼리에서 사용할 수 있다
- IN 식
    - cb.in(m.get("username"))
- CASE 식
    - cb.selectCase()
      when(cb.ge(m.<Integer>get("age"), 60), 600)
      when(cb.le(m.<Integer>get("age"), 15), 500)
      otherwise(1000)
- 파라미터 정의
    - cb.paramegter(타입, 파라미터 이름) : 파라미터 정의
    - setParameter("usernameParam", "회원1") : 해당 파라미터에 사용할 값을 바인딩
- 네이티브 함수 호출
    - cb.function("SUM", Long.class, m.get("age"))
- 동적 쿼리
    - 다양한 검색 조건에 따라 실행 시점에 쿼리를 생성하는 것
    - 문자 기반(JPQL)보다 코드 기반(크리테리아)이 더 편리
        - JPQL : 띄어쓰기와 where/and 위치 이슈 등
        - 크리테리아에는 이런 이슈는 없으나 장황하고 복잡함으로 코드 가독성이 떨어짐
- 함수 정리
    - 조건 함수
    - 스칼라와 기타 함수
    - 집합 함수
    - 분기 함수
- Criteria 메타 모델 API
    - 코드 기반 - 컴파일 시점에 오류를 발견 가능
    - 하지만 일부는 문자 - 컴파일 에러로 발견 못함 - 완전한 코드기반은 아님
        - 그래서 나타난 것이 메타 모델 API
        - 메타 모델 클래스
    - 표준 메타 모델 클래스 - 메타 모델
        - 엔티티를 기반으로 만들어야
        - 엔티티명_.java
    - 코드 생성기 설정
        - hibernate-jpamodelgen
        - [https://docs.jboss.org/hibernate/stable/jpamodelgen/reference/en-US/html_single/](https://docs.jboss.org/hibernate/stable/jpamodelgen/reference/en-US/html_single/)

<br><br>

## 4. QueryDSL

- 쿼리를 문자가 아닌 코드로 작성해도 (크리테리아처럼)
    - 쉽고 간결하며
    - 그 모양도 쿼리와 비슷하게 개발할 수 있는 프로젝트
    - JPQL 빌더 역할 (크리테리아처럼)
    - 오프소스 프로젝트
- 설정
    - querydsl-jpa : QueryDSL JPA 라이브러리
    - querydsl-apt : 쿼리 타입(Q)을 생성할 때 필요한 라이브러리
    - 쿼리 타입 생성용 플러그인
        - 크리테리아의 메타모델처럼 엔티티를 기반으로 쿼리 타입이라는 쿼리용 클래스를 생성
    - 콘솔에서 mvn compile 입력 - 쿼리 타입 생성
- 시작
    - JPAQuery 객체를 생성해야
        - 엔티티 매니저를 생성자에 넘겨준다
        - 쿼리 타입 생성 : 생성자에는 별칭을 주면 된다
            - JPQL에서 별칭으로 사용
    - 기본 Q 생성
        - 사용하기 편리하도록 기본 인스턴스를 보관
        - 같은 엔티티 조인하거나 같은 엔ㄴ티티를 서브쿼리에 사용하고 싶을 때
            - 별칭을 직접 지정해서 사용
- 검색 조건 쿼리
    - where절 다음으로 and, or 사용 가능
    - where에 사용되는 메소드
        - eq, between, contains, startsWith
- 결과 조회
    - 결과 조회 메소드를 사용하고 파라미터로 프로젝션 대상을 넘겨준다
    - uniqueResult : 조회 결과가 한건일때
    - singleResult : 결과가 하나 이상이면 처음 데이터를 반환
    - list : 하나 이상일 때 혹은 빈 컬렉션
- 페이징과 정렬
    - orderBy(item.price.desc(), item.stockQuantity.asc())
    .offset(10).limit(20)
    - listResults : 전체 데이터 조회를 위한 count 쿼리를 한번더 실행
        - SearchResults를 반환. 이 객체에서 전체 데이터 수를 조회할 수 있다
- 그룹
    - groupBy(item.price)
    .having()item.price.gt(1000))
- 조인
    - join(조인 대상, 별칭으로 사용할 쿼리 타입)
    - join(order.member, member)
    .on(member.weight.gt(60))
    - leftJoin(order.orderItems, orderItem).fetch()
    - from절에 여러 조인을 사용하는 세타 조인
        - from(order, member)
- 서브 쿼리
    - JPASubQuery를 생성해서 사용
    - 결과 메소드 : unique, list
- 프로젝션과 결과 반환
    - 정의 : select 절에 조회대상을 지정하는 것
    - 프로젝션 대상이 하나
        - List<String> result = query.from(item).list(item.name)
    - 여러 컬럼 반환과 튜플
        - 프로젝션 대상으로 여러 필드를 선택하면
        - Tuple : Map과 유사
        - List<Tuple> result = query.from(item).list(item.name, item.price)
    - 빈 생성
        - 쿼리 결과를 엔티티가 아닌 특정 객체로 받고 싶다면
        - 원하는 방법을 지정하기 위해 Projections를 사용
        - 객체를 생성하는 다양한 방법
            - 프로퍼티 접근
                - List<ItemDTO> result = query.from(item).list(
                  Projections.bean(ItemDTO.class, item.name.as("username"), item.price))
                - bean 메소드 : 수정자(setter)를 사용해서 값을 채움
                - 쿼리 결과와 매핑할 프로퍼티 이름이 다르면 as를 사용해서 별칭을 주면 됨
            - 필드 직접 접근
                - List<ItemDTO> result = query.from(item).list(
                  Projections.fields(ItemDTO.class, item.name.as("username"), item.price))
                - 필드에 직접 접근해서 값을 채움
                - 필드를 private로 설정해도 동작
            - 생성자 사용
                - List<ItemDTO> result = query.from(item).list(
                  Projections.constructor(ItemDTO.class, item.name.as("username"), item.price))
                - 생성자를 사용. 물론 지정한 프로젝션과 파라미터 순서가 같은 생성자가 필요
    - distinct
        - query.distinct().from(item)...
- 수정, 삭제 배치 쿼리
    - JPQL 배치 쿼리와 같이 영속성 컨텍스트를 무시하고 데이터베이스를 직접 쿼리한드는 점에 유의
    - 수정배치 쿼리 : JPAUpdateClause
        - updateClause.where(item.name.eq("my book"))
          .set(item.price, tiem.price.add(100)).excute()
    - 삭제배치 쿼리 : JPADeleteClause
- 동적 쿼리
    - BooleanBuilder
    - booleanBuilder.and(item.name.contains(param.getName()));
    query.from(item).where(booleanBuilder).list(item);
- 메소드 위임
    - 쿼리 타입에 검색 조건을 직접 정의할 수 있다
    - 정적 메소드를 만들고 + QueryDelegate 어노테이션에 속성으로 이 기능을 적용할 엔티티를 지정
    - 정적 메소드
        - 첫번째 파라미터 : 대상 엔티티의 쿼리 타입을 지정
        - 나머지 : 필요한 파라미터를 정의
        - String, Date 같은 엔티티가 아닌 자바 기본 내장 타입에도 메소드 위임기능을 사용할 수 있다.
- 정리
    - 문자가 아닌 코드로 안전하게 코드를 작성하고 싶다
    - 복잡한 동적 쿼리를 어떻게 해결해야 하는가
    - 크리테리아가 복잡해서 차라리 JPQL을 직접 사용?
    - QueryDSL이 모두 만족

<br><br>



## 5. 네이티브 SQL

JPQL은 표준 SQL이 지원하는 대부분의 문법과 SQL 함수들을 지원

특정 데이터베이스에 종속적인 기능은 지원하지 않는다

> <예시>
특정 데이터베이스만 지원하는 함수, 문법, SQL 쿼리 힌트
인라인 뷰(From 절에서 사용하는 서브쿼리), union, intersect
스토어드 프로시저
>

때로는 특정 데이터베이스에 종속적인 기능이 필요

JPA 구현체들은 JPA 표준보다 더 다양한 방법을 지원

> 특정 데이터베이스만 사용하는 함수
특정 데이터베이스만 지원하는 SQL 쿼리 힌트
인라인 뷰 (From 절에서 사용하는 서브쿼리, union, intersect
스토어 프로시저
특정 데이터베이스만 지원하는 문법
>

- 네이티브 SQL
    - 다양한 이유로 JPQL을 사용할 수 없을 때 JPA는 SQL을 직접 사용할 수 있는 기능을 제공
        - JPQL : 자동모드
        - 네이티브 SQL : 수동모드
    - 네이티브 SQL을 사용하면 엔티티를 조회할 수 있고 JPA가 지원하는 영속성 컨텍스트의 기능을 그대로 사용할 수 있다.
        - JDBC API를 직접사용 - 단순히 데이터의 나열을 조회할 뿐
- 네이티브 SQL 사용
    - 네이티브 쿼리 API는 다음 3가지가 있다
    - 엔티티 조회
        - 결과 타입 정의
        - em.createNativeQuery(SQL, 결과 클래스)
        - 실제 데이터베이스 SQL, 위치기반 파라미터만 지원
        - 이름 기반은 지원 X, but 하이버네이트는 지원 (사견 : 하이버네이트, 너란 녀석 정말...)
        - SQL 직접 사용하는 부분 외 전부 JPQL (영속성 컨텍스트 등)
    - 값 조회
        - 결과 타입을 정의할 수 없을 때
        - em.createNativeQuery(SQL)
        - object[ ]에 담아서 반환
        - 스칼라 값들을 조회할 뿐 - 당연히 영속성 컨텍스트가 관리하지 않는다
        - JDBC로 데이터를 조회한 것과 비슷
    - 결과 매핑 사용
        - 엔티티 + 스칼라값을 같이 조회? 복잡해지면?
        - @SqlResultSetMapping
        - em.createNativeQuery(SQL, 결과 매핑 정보의 이름)
        - 여러 엔티티와 여러 컬럼을 매핑할 수 있다
            - 추가 설명 : 이때 사용할 어노테이션이 결과 매핑 어노테이션. 아래 내용 참고
        - @FieldResult
            - 컬럼명과 필드명을 직접 매핑
            - 동일한 필드가 나타날 시 별칭 사용후 FieldResult로 매핑
    - 결과 매핑 어노테이션
        - SqlResultSetMapping
        - EntityResult : 결과로 사용할 엔티티 클래스를 지정
        - FieldResult
        - ColumnResult : 결과 컬럼명
- Named 네이티브 SQL
    - 정적 SQL 작성
    - @NamedNativeQuery
        - hints : JPA 구현체에 제공하는 힌트
    - JPQL Named 쿼리와 같은 createNamedQuery 메소드를 사용
    - TypeQuery를 사용할 수 있다
    - resultSetMapping 속성으로 조회 결과를 매핑할 대상까지 지정
- 네이티브 SQL XML에 정의
    - xml에 정의할 때는 순서를 지켜야
        - <named-native-query>를 먼저
        - <sql-result-set-mapping>를 다음에 정의
        - xml과 어노테이션 둘 다 사용하는 코드
            - 사견 : 해당 내용을 이해할 수 없음
- 네이티브 SQL은
    - 보통 JPQL로 작성하기 어려운 복잡한 SQL쿼리를 작성하거나
    - SQL을 최적화해서 데이터베이스 성능 향상할 때 사용
    - 복잡 + 라인수가 많다 : xml 사용할 시 편리
    - 멀티 라인 문자열 지원하지 않는 JAVA
    - xml은 완성한 SQL을 바로 붙여 넣을 수 있다
- 네이티브 SQL 정리
    - JPQL과 동일하게 Query, TypeQuery(Named 네이티브 쿼리의 경우만) 반환
    - JPQL API를 그대로 사용 가능
    - 네이티브 SQL은 JPQL이 자동생성하는 SQL을 수동으로 직접 정의하는 것
    - 따라서 JPA가 제공하는 기능 대부분을 그대로 사용할 수 있다
    - but 관리가 쉽지 않고 + 자주 사용시 특정 데이터베이스에 종속적인 쿼리가 증가해서 이식성이 떨어짐
    - 표준 JPQL을 사용하고 기능이 부족하면 차선책으로 JPA 구현체가 제공하는 기능을 사용
    - 네이티브 SQL로도 부족함을 느끼면 마이바티스나 스프링 프레임워크가 제공하는 JdbcTemplate 같은 SQL 매퍼와 JPA를 함께 사용하는 것을 고려하자
- 스토어드 프로시저 (JPA 2.1)
    - 스토어드 프로시저 사용
        - em.createStoredProcedureQuery 메소드
            - 스토어드 프로시즈 이름을 입력
        - registerStoredProcedureParameter 메소드
            - 프로시저에서 사용할 파라미터를 순서, 타입, 파라미터 모드 순으로 정의
    - Named 스토어드 프로시저 사용
        - 스토어드 프로시저 쿼리에 이름을 부여해서 사용하는 것
        - @NamedStoredProcedureQuery
        - 사용법
            - em.createNamedStoredProcedureQuery 메소드에 등록한 Named 스토어드 프로시저 이름을 파라미터로 사용해서 찾아올 수 있다

## 6. 객체지향 쿼리 심화

객체지향 쿼리와 관련된 다양한 고급주제!!!

- 벌크 연산
    - 엔티티를 수정하려면
        - 영속성 컨텍스트의 변경 감지 기능이나 병합을 사용하고
        - 삭제하려면 em.remove 메소드를 사용
    - 수백개 이상의 엔티티를 하나씩 처리?? 시간이..
        - 이럴때 한번에 수정, 삭제하는 벌크 연산이 따봉
    - executeUpdate 메소드
    - 하이버네이트 좌 등장
        - JPA 표준은 아니지만 INSERT 벌크 연산 지원
    - 벌크 연산의 주의점
        - 영속성 컨텍스트를 무시하고 데이터베이스에 직접 쿼리한다
        - 영속성 컨텍스트에 있는 값과 데이터베이스에 있는 값이 다를 수 있다
    - 위 주의점의 해결방법
        - em.refresh 메소드 사용
            - 해당 데이터 다시 조회
        - 벌크 연산 먼저 실행
            - 모든 조회는 연산 후에 진행
            - JPA와 JDBC를 함께 사용할 때도 유용
        - 벌크 연산 수행 후 영속성 컨텍스트 초기화
            - 영속성 컨텍스트에 남아있는 엔티티를 제거
            - 초기화하면 엔티티 조회할 때 벌크 연산이 적용된 데이터베이스에서 엔티티를 조회
    - **영속성 컨텍스트와 2차 캐시를 무시하고 데이터베이스에 직접 실행**
        - **따라서 데이터 차이가 발생할 수 있다**
- 영속성 컨텍스트와 JPQL
    - 쿼리 후 영속 상태인 것과 아닌 것
        - JPQL로 엔티티를 조회하면 영속성 컨텍스트에서 관리되지만
            - 엔티티가 아니면 영속성 컨텍스트에서 관리되지 않는다
            - 예) 임베디드 타입
    - JPQL로 조회한 엔티티와 영속성 컨텍스트
        - 영속성 컨텍스트에 데이터가 이미 있는데 JPQL로 재조회를 한다면?
            - 영속성 컨텍스트에 있던 엔티티를 반환
            1. JPQL로 조회한 엔티티는 영속 상태
            2. 영속성 컨텍스트에 이미 존재하는 엔티티가 있으면 기존 엔티티를 반환
            - 영속성 컨텍스트는 영속 상태인 엔티티의 동일성을 보장한다
                - 기존 엔티티는 그대로 두고 새로 검색한 엔티티를 버린다
    - find() vs JPQL
        - find 메소드는 엔티티를 영속성 컨텍스트에서 먼저 찾고 없으면 데이터베이스에서 찾는다
            - 영속성 컨텍스트에 있으면 메모리에서 찾음 - 성능상 이점 (그래서 1차 캐시)
        - JPQL은 항상 데이터베이스에서 SQL을 실행해서 결과를 조회
            - 주소값이 같은 인스턴스를 반환 - 이는 find 메소드와 동일
            - 첫번째 조회 후 영속성 컨텍스트에 등록
            - 두번째 데이터베이스 조회 후 영속성 컨텍스트 확인 - 새로운 엔티티는 버림
    - 정리
        - JPQL은 항상 데이터베이스를 조회
        - JPQL로 조회한 엔티티는 영속 상태
        - 영속성 컨텍스트에 이미 존재하는 엔티티가 있으면 기존 엔티티를 반환
- JPQL과 플러시 모드
    - 복습
        - 플러시는 영속성 컨텍스트의 변경 내역을 데이터베이스에 동기화 하는것
            - 플러시가 일어날 때 영속성 컨텍스트에 등록/수정/삭제한 엔티티를 찾아서 해당 쿼리를 만들어 데이터베이스에 반영
            - em.flush 메소드 혹은 플러시 모드에 따라 커밋하기 직전이나 쿼리 실행 직전에 자동으로 플러시 호출
        - 플러시 모드는 FlushModeType.AUTO가 기본값
            - 트랜잭션 커밋 직전이나 쿼리 실행 직전에 자동으로 플러시르 ㄹ호출
            - COMMIT 모드 : 커밋시에만 플러시를 호출, 성능 최적화를 위해 꼭 필요할 때만 사용
    - 쿼리와 플러시 모드
        - JPQL은 영속성 컨텍스트에 있는 데이터를 고려하지 않고 데이터베이스에서 데이터르 ㄹ조회
            - JPQL 실행 전 영속성 컨텍스트의 내용을 데이터베이스에 반영해야
            - AUTO일 때는 수정한 값이 조회되고 COMMIT일 때는 조회할 수 없을 수 있다
        - AUTO가 아닐 때
            - em.flush()
            - setFlushMode(FlushModeType.AUTO)
    - 플로시 모드와 최적화
        - COMMIT 모드는 트랜잭션을 커밋할 때만 플러시하고 쿼리를 실행할 때는 플러시하지 않음
            - 영속성 컨텍스트에는 있지만 데이터베이스에는 아직 반영하지 않은 상황
            - 데이터 무결성에 심각한 피해를 초래할 수 있다
        - 플러시가 자주 일어나는 상황
            - COMMIT 모드를 사용하면 쿼리시 발생하는 플러시 횟수를 줄여서 성능 최적화 가능
            - 예) 4번 플러시 할 것을 커밋시에만 1번 플러시
        - JPA를 사용하지 않고 JDBC를 직접 사용해서 SQL을 실행할 때도 플러시 모드를 고민해야
            - 쿼리를 직접 실행하면 JPA는 JDBC가 실행한 쿼리를 인식할 방법이 없다
            - JDBC 호출은 플러시 모드를 AUTO 설정해도 플러시가 일어나지 않음
            - JDBC로 쿼리 실행하기 직전에 em.vlush를 호출해서 영속성 컨텍스트의 내용을 데이터베이스에 동기화하는 것이 안전

## 7. 정리

- JPQL은 SQL을 추상화해서 특정 데이터베이스 기술에 의존하지 않음
- 크리테리아나 QueryDSL은 JPQL을 만들어주는 빌더 역할을 할 뿐
    - 핵심은 JPQL을 잘 알아야
- 크리테리아나 QueryDSL을 사용하면 동적으로 변하는 쿼리를 편리하게 작성할 수
- 크리테리아는 JPA가 공식 지원하는 기능
    - 직관적이지 않고 사용하기에 불편
- QueryDSL은 JPA가 직접 지원하지 않음
    - 직관적이고 편리
- JPA도 네이티브 SQL을 제공하므로 직접 SQL을 사용할 수 있다
    - 특정 데이터베이스에 종속적인 SQL을 사용하면 다른 데이터베이스로 변경하기 쉽지 않다
    - 따라서 최대한 JPQL을 사용하고 그래도 방법이 없을 때 네이티브 SQL을 사용
- JPQL은 대량에 데이터를 수정하거나 삭제할하는 벌크연산을 지원
