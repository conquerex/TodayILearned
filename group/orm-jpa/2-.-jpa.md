---
description: 개발환경 세팅하는 부분
---

# 2장. JPA 시작

### 2.1. IDE 설치 및 프로젝트 불러오기

서적에는 이클립스로 가이드되어 있으나 여기에서는 인텔리제이로 진행할 예정. 실습 코드는 아래 링크를 참조할 것.

{% embed url="https://github.com/conquerex/jpabook" %}



### 2.2. H2 데이터베이스 설치

* MySQL이나 오라클보다는 사용하기 부담이 적은 H2 데이터베이스
  * bin/h2.sh 실행 \(서버 모드로 실행\)
  * H2는 JVM 메모리 안에서 실행되는 임베디드 모드와 실제 데이터베이스처럼 별도의 서버를 띄워서 동작하는 서버모드가 있다
  * localhost:8082 접속
    * 드라이버 클래스 : org.h2.Driver
    * JDBC URL : jdbc:h2:tcp://localhost/~/test
    * 사용자명 : sa
    * 비밀번호 : \(입력하지 않음\)
  * H2 콘솔이 보인다
  * 예제 테이블 생성하기

```text
create table member (
  id varchar(255) not null,
  name varchar(255),
  age integer not null,
  primary key (id)
)
```



### 2.3. 라이브러리와 프로젝트 구조

* 메이븐 : 라이브러리 관리 도구 + 빌드 기능도 제공
* 책 전반에 걸쳐 JPA 구현체로 하이버네이트를 사용
  * hibernate-core : 하이버네이트 라이브러리
  * hibernate-entitymanager : 하이버네이트가 JPA 구현체로 동작하도록 JPA 표준을 구현한 라이브러리
  * hibernate-jpa-2.1-api : JPA 2.1 표준 API를 모아둔 라이브러리
* pom.xml에 사용할 라이브러리를 적어주면 자동으로 내려받아서 관리해준다
  * groupId + artifactId + version
* 예제의 핵심 라이브러리
  * JPA, 하이버네이트
    * hibernate-entitymanager만 받으면 hibernate-core, hibernate-jpa-2.1-api를 함께 받을 수 있다
  * H2 데이터베이스





### 2.4. 객체 매핑 시작

* 회원 클래스 만들기 : Member.class
* 매핑 어노테이션 추가
  * JPA는 매핑 어노테이션을 분석해서 어떤 객체가 어떤 테이블과 관계가 있는지 알아낸다
  * @Entity
    * 해당 클래스를 테이블과 매핑한다고 JPA에 알려줌
    * 엔티티 클래스
  * @Table\(name="MEMBER"\)
    * 엔티티 클래스에 매핑할 테이블 정보를 알려준다
    * name 속성으로 MEMBER 테이블을 매핑했다
    * 생략시 클래스 이름을 테이블 이름으로 매핑 \(엔티티 이름을 사용\)
  * @Id
    * 엔티티 클래스의 필드를 테이블의 기본키에 매핑
    * @Id가 사용된 필드를 식별자 필드라 한다
  * @Column\(name = "NAME"\)
    * 필드를 컬럼에 매핑
    * name 속성을 사용해 NAME 칼럼에 매핑
  * 매핑 정보가 없는 필드
    * 필드명을 사용해서 컬럼명으로 매핑
    * 대소문자 구분 않는다고 가정
* JPA 어노테이션의 패키지 : javax.persistence



### 2.5. persistence.xml 설정

* persistence.xml을 사용해서 필요한 설정 정보를 관리
* 위치 : ch02-jpa-start1/src/main/resources/META-INF/persistence.xml
* 설정시 xml 태그에 persistence로 시작
  * xml 네임스페이스, 버전

```text
<persistence xmlns="http://xmlns.jcp.org/xml/ns/persistence" version="2.1">
```

* JPA 설정 : 영속성 유닛\(persistence-unit\)이라는 것부터 시작
  * 일반적으로 연결할 데이터베이스당 하나의 영속성 유닛을 등록
  * 고유한 이름을 부여 \(여기서는 jpabook\)

```text
<persistence-unit name="jpabook">
```

* 각각의 속성값 분석
  * JPA 표준 속성
    * javax.persistence으로 시작하는 속성. 특정 구현체에 종속적이지 않다
    * javax.persistence.jdbc.driver
    * javax.persistence.jdbc.user : DB 접속 아이디
    * javax.persistence.jdbc.password
    * javax.persistence.jdbc.url : DB 접속 URL
  * 하이버네이트 속성
    * 하이버네이트 전용 속성
    * hibernate.dialect : 데이터베이스 방언 설정

```text
<property name="javax.persistence.jdbc.driver" value="org.h2.Driver"/>
<property name="javax.persistence.jdbc.user" value="sa"/>
<property name="javax.persistence.jdbc.password" value=""/>
<property name="javax.persistence.jdbc.url" value="jdbc:h2:tcp://localhost/~/test"/>
<property name="hibernate.dialect" value="org.hibernate.dialect.H2Dialect" />
```

* 데이터베이스 방언
  * JPA는 특정 데이터베이스에 종속적이지 않은 기술. 그래서 손쉽게 교체 가능
  * 각 데이터베이스가 제공하는 문법과 함수가 조금씩 다르다
    * 데이터 타입, 다른 함수명, 페이징 처리 등
  * 방언\(dialect\) : SQL 표준을 지키지 않거나 특정 데이터터베이스만의 고유한 기능
  * 쉬운 데이터베이스 교체를 위해 JPA 구현체\(하이버네이트 포함\)들은 데이터베이스 방언 클래스를 제공
  * JPA가 제공하는 표준 문법\(JPA\) + 특정 데이터베이스에 의존적인 SQL\(데이터베이스 방언\)
  * 데이터베이스 변경시, 애플리케이션 코드 변경없이 방언만 교체하면 됨
  * 방언을 설정하는 방법은 JPA에 표준화되어 있지 않음
    * H2, 오라클, MySQL
    * 하이버네이트는 45개의 데이터베이스 방언을 지원
    * H2의 경우 : org.hibernate.dialect.H2Dialect
* 하이버네이트 전용속성
  * hibernate.show\_sql : 하이버네이트가 실행한 SQL 출력
  * hibernate.format\_sql : 위에서 출력한 것을 보기 쉽게 정렬
  * hibernate.use\_sql\_comments : 커리를 출력할 때 주석도 출력
  * hibernate.id.new\_generator\_mappings : JPA 표준에 맞춘 새로운 키 생성 전략으 사용



![](../../.gitbook/assets/image%20%286%29.png)

### 2.6. 애플리케이션 개발

* JpaMain : 애플리케이션을 시작하는 코드
  * 코드는 크게 3부분 \(엔티티 매니저 설정, 트랜잭션 관리, 비즈니스 로직\)

```text
EntityManagerFactory emf = Persistence.createEntityManagerFactory("jpabook");
EntityManager em = emf.createEntityManager(); //엔티티 매니저 생성

EntityTransaction tx = em.getTransaction(); //트랜잭션 기능 획득

try {
    // 생략
} catch (Exception e) {
    e.printStackTrace();
    tx.rollback(); //트랜잭션 롤백
} finally {
    em.close(); //엔티티 매니저 종료
}

emf.close(); //엔티티 매니저 팩토리 종료
```

* 엔티티 매니저 설정
  * 엔티티 매니저 팩토리\(EntityManagerFactory\) 생성
    * JPA를 시작하려면 persistence.xml의 설정 정보를 사용해서 엔티티 매니저 팩토리를 생성해야
    * 이 때 persistence 클래스를 사용. 엔티티 매니저 팩토리를 생성해서 JPA를 사용할 수 있게 준비한다
    * persistence.xml에서 이름이 jpabook인 영속성 유닛을 찾아서 엔티티 매니저 팩토리를 생성
    * 엔티티 매니저 팩토리를 애플리케이션 전체에서 딱 한번만 생성하고 공유해서 사용
      * persistence.xml의 설정 정보를 읽어서 JPA를 동작시키기 위한 기반 객체를 만들고
      * JPA 구현체에 따라서는 데이터베이스 커넥션 풀도 생성
      * 엔티티 매니저 팩토리를 생성하는 비용이 크기 때문
  * 엔티티 매니저 생성
    * 엔티티 매니저 팩토리에서 엔티티 매니저 생성. JPA 기능 대부분이 여기서 제공
    * 엔티티 매니저를 사용해서 엔티티를 데이터베이스에 등록, 서정, 삭제, 조회할 수 있다
    * 내부에 데이터소스\(데이터베이스 커넥션\)를 유지하면서 데이터베이스와 통신
    * 가상의 데이터베이스로 생각할 수도...
    * 엔티티 매니저는 데이터베이스 커넥션과 밀접한 관계가 있으므로 스레드간에 공유하거나 재사용하면 안됨
  * 종료
    * 사용이 끝난 엔티티매니저는 종료를
    * 애플리케이션을 종료할 때는 엔티티 매니저 팩토리를 종료
* 트랜잭션 관리
  * JPA를 사용하면 항상 트랜잭션 안에서 데이터를 변경해야
    * 트랜잭션 없으면 예외가 발생
  * 트랜잭션을 시작하려면 엔티티 매니저에서 트랜잭션 API를 받아와야
  * 트랜잭션 API를 사용해서...
    * 비즈니스 로직이 정상 동작하면 트랜잭션을 커밋\(Commit\)
    * 예외가 발생하면 트랜잭션을 롤백\(Rollback\)

```text
String id = "id1";
Member member = new Member();
member.setId(id);
member.setUsername("지한");
member.setAge(2);

//등록
em.persist(member);

//수정
member.setAge(20);

//한 건 조회
Member findMember = em.find(Member.class, id);
System.out.println("findMember=" + findMember.getUsername() + ", age=" + findMember.getAge());

//목록 조회
List<Member> members = em.createQuery("select m from Member m", Member.class).getResultList();
System.out.println("members.size=" + members.size());

//삭제
em.remove(member);
```

* 비즈니스 로직
  * 회원 엔티티를 하나 생성한 다음 엔티티 매니저를 통해 데이터베이스에 CRUD
  * 엔티티 매니저는 객체를 저장하는 가상의 데이터베이스처럼 보인다
  * 저장
    * 엔티티 매니저의 persist 메소드에 저장할 엔티티를 넘겨준다
    * JPA는 회원 엔티티의 매핑 정보\(어노테이션\)을 분석해서 SQL을 자동으로 만들어 데이터베이스에 전달
  * 수정
    * 엔티티의 값만 변경
    * JPA는 어떤 엔티티가 변경되었는지 추적하는 기능을 갖추고 있다
    * 자동으로 UPDATE 쿼리를 생성해서 값 변경
  * 삭제
    * 엔티티 매니저의 remove 메소드에 삭제하려는 엔티티 전달
  * 1건 조회
    * find 메소드 : 조회할 엔티티 타입 + @Id로 데이터베이스 테이블의 기본 키와 매핑한 식별자 값
    * 엔티티 하나를 조회하는 가장 단순한 조회 메소드
    * select 쿼리를 생성, 데이터베이스에 결과를 조회, 결과 값으로 엔티티를 생성해서 반환
* JPQL
  * 검색할 때 테이블이 아닌 엔티티 객체를 대상으로 검색해야
  * 데이터베이스의 모든 데이터를 애플리케이션에 불러와서 엔티티 객체로 변경하고... 이건 불가능
  * 검색 조건이 포함된 쿼리를 사용해야
  * JPQL이라는 쿼리 언어로 문제 해결 \(Java Persistence Query Language\)
  * JPA는 SQL을 추상화한 JPQL이라는 객체지향 쿼리 언어를 제공
    * 기존의 SQL과 문법이 거의 유사
    * JPQL : 엔티티 객체를 대상으로 쿼리 \(클래스, 필드를 대상으로\)
    * SQL : 데이터베이스 테이블을 대상으로 쿼리
  * 예\) select m from Member m
    * from Member : 회원 엔티티 객체. 테이블 아님
    * JPQL은 데이터베이스 테이블을 전혀 알지 못함
  * JPQL을 사용하려면
    * 먼저 em.createQuery\(JPQL, 반환타입\) 메소드를 실행해서 쿼리 객체를 생성한 후
    * 쿼리 객체의 getResultList\(\) 메소드를 호출하면 된다
    * JPA는 JPQL을 분석해서 적절한 SQL을 만들어 데이터베이스에서 데이터 조회
      * 예\) select m.id, m.name, m.age from member m
  * JPQL은 대소문자를 구분, SQL은 아닌 경우가 많





