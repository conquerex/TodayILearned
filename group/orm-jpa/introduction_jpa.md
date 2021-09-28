# 1장. JPA 소개

![&#xD6C4;&#xC6B1;&#xD6C4;&#xC6B1;. &#xC601;&#xD55C;&#xB290;&#xB2D8;...](../../.gitbook/assets/image%20%284%29.png)

김영한씨의 독백같은 글로 시작한다. 자바와 관계형 데이터베이스를 사용해서 개발을 많이 해 봤다라는 내용.

* 초기에는 JDBC API를 직접 사용
  * 비즈니스 로직보다 SQL과 JDBC API를 작성하는데 더 많은 시간을 보냄
  * iBatis\(지금은 MyBatis\)나 스프링의 JdbcTemplate같은 SQL Mapper를 사용
  * JDBC API 사용코드를 많이 줄일 수 있었다
* CRUD용 SQL은 반복해서 작성 - 지루하고 비생산적
  * 테이블 설계는 열심히. 제대로 된 객체 모델링은?
  * 객체 지향의 장점. 단순히 테이블에 맞추어 데이터 전달 역할만 하도록 할까?
* 객체 모델링을 세밀하게
  * 객체를 데이터베이스에 저장하거나 조회하기 어려워
  * 객체와 관계형 데이터베이스의 차이를 메우기 위해 더 맣은 SQL을 작성해야
  * 객체 모델링을 SQL로 풀어내는 데 많은 코드와 노력
  * 객체 모델은 점점 데이터 중심의 모델로 변해갔다

![](../../.gitbook/assets/image%20%283%29.png)

* 객체와 관계형 데이터베이스 간의 차이를 중간에서 해결하는 ORM 프레임워크
  * JPA : 자바 진영의 ORM 기술 표준
* JPA
  * 실행시점에 자동으로 SQL을 만들어서 실행
  * 개발자는 SQL을 직접 작성하는게 아니라 어떤 SQL이 실행될지 생각만 하면 됨
* JPA를 사용해서 얻은 보상
  * CRUD SQL을 작성할 필요없고
  * 조회된 결과를 객체로 매핑하는 작업도 자동으로
  * 성능 이슈에 대한 대안도 존재 \(네이티브 SQL 기능\)
  * 조회 성능 이슈는 SQL을 직접 사용해도 발생하는 문제
* 애플리케이션을 SQL이 아닌 객체 중심으로 개발
  * 생산성과 유지보수가 좋아짐
  * 테스트 작성 편리
  * DB가 바뀌어도\(MySQL, 오라클\) 손쉽게 변경 가능
* JPA를 구현한 하이버네이트, 10년 이상 지속해서 개발
* **개발자는 SQL 매퍼가 아니다**



### 1.1. SQL을 직접 다룰 때 발생하는 문제점

* 관계형 데이터베이스 : 가장 대중적, 신뢰할만한 안전한 데이터 저장소
* 데이터베이스에 데이터를 관리하려면 SQL을 사용해야
  * JDBC API를 사용해서 SQL을 데이터베이스에 전달

![](../../.gitbook/assets/image%20%285%29.png)

* 자바 + 관계형 데이터베이스
  * 자바에서 사용할 객체를 만들고
  * 해당 객체를 데이터베이스에 관리할 목적으로 DAO\(데이터 접근 객체\)를 만들고
  * DAO의 메소드를 완성해서 조회 등을 할 수 있다
  * 회원 조회용 SQL을 작성
  * JDBC API를 사용해서 SQL 실행
  * 조회 결과를 객체로 매핑
* 위에서 조회뿐만 아니라 수정, 삭제등을 만든다고 하면 위와 비슷한 일을 반복해야
* 데이터베이스는 객체 구조와는 다른 데이터 주심의 구조
  * 객체를 데이터베이스에 직접 저장, 조회 불가능
  * SQL과 JDBC API가 변환 작업을 직정 수행
  * 하지만 매번 많은 SQL과 JDBC API를 코드로 작성해야
* SQL에 의존적인 개발
  * 테이블의 필드를 추가한다거나, 2개 이상의 객체가 연관될 때
  * 정상 작동이 되지 않으면 DAO를 열어서 어떤 SQL이 실행되는지 확인해야
* 비즈니스 요구사항을 모델링한 객체 : 엔티티
  * 위와 같은 상황에서 엔티티 신뢰할 수 없다
  * 진정한 의미의 계층 분할이 아니다
  * 물리적으로는 데이터 접근 계층에 숨기는데 성공
  * but 논리적으로는 엔티티와 SQL/JDBC API와 강한 의존관계를 가짐
* **애플리케이션에서 SQL을 직접 다룰 때 발생하는 문제점**
  * 진정한 의미의 계층 분할이 어렵다
  * 에티티를 신뢰할 수 없다
  * SQL에 의존적인 개발을 피하기 어렵다
* JPA와 문제 해결
  * 객체를 데이터베이스에 저장하고 관리할 때
  * 개발자는 JPA가 제공하는 API를 사용하면, JPA는 적절한 SQL을 생성해서 데이터베이스에 전달



### 1.2. 패러다임의 불일치

* 객체지향 프로그래밍
  * 추상화, 캡슐화, 정보은닉, 상속, 다형성
* 도메인 모델 : 비즈니스 요구사항을 정의
  * 객체로 모델링하면 객체지향 언어가 가진 장점을 활용 가능
  * 도메인 모델을 저장을 어떻게?
* 객체 : 속성\(필드\) + 기능\(메소드\)
  * 클래스에 정의, 객체 인스턴스의 상태인 속성만 저장했다가 필요시 ㅣ불러와서 복구
  * 상속, 참조를 하고 있다면 객체의 상태를 저장하기 쉽지 않다
* 자바 - 직렬화 기능
  * 직렬화된 객체를 검색하기 어렵다는 문제
* 관계형 데이터베이스
  * 데이터 중심으로 구조화, 집합적인 사고 요구, 객체지향의 추상화/상속/야형성 등이 없다
  * 지향하는 목적이 다르므로 기능과 표현방법도 다르다 : **패러다임 불일치 문제**
  * 객체 구조를 테이블 구조에 저장하는데 한계가 있다
* 결국은...
  * 애플리케이션은 자바라는 객체지향 언어로
  * 데이터는 관계형 데이터베이스에 저장
  * 개발자가 중간에서 패러다임 불일치를 해결해야
* \[불일치 이슈 1\] 상속
  * 객체는 상속 기능이 있지만 테이블은 없다
  * 데이터베이스 모델링에서 슈퍼/서브타입 관계가 그나마 유사
  * DB는 부모와 자식을 따로 저장하고 조회를 위해 JOIN해서 조회해야 한다
  * 객체에서는 이런 고민 없이 컬렉션을 사용하면 된다
  * 자바 컬렉션에 객체를 저장하듯 JPA에게 객체를 저장하면 된다
* \[불일치 이슈 2\] 연관관계
  * 객체 : 다른 객체와 연관관계를 가지고 **참조**에 접근해서 연관된 객체를 조회
  * 테이블 : **왜래키**를 사용. 조인을 사용해서 연관된 테이블을 조회
  * 객체는 한쪽 방향으로만 조회가 가능하다. \(ex. member.getTeam\(\)\)
  * 테이블은 외래키 하나로 양방향 조회가 가능하다
  * 관계형 DB는 조인이 있어서 외래키의 값만 보관해도 되지만 객체는 참조를 보관해야 한다
  * 데이터베이스 방식에 맞추면 참조를 통해 조회할 수 없기 때문에
    * 좋은 객체 모델링을 기대하기 어렵고 객체지향의 특징을 잃어버리게 된다
  * 객체지향 모델링을 사용하는 것도 마찬가지. 참조/외래키 이슈
    * 저장 : 본 객체는 참조 객체의 아이디를 외래키로 변환해야 하고, 참조 객체는 기본키로 사용한다
    * 조회 : 외래키값을 객체의 참조로 변환해서 객체에 보관. 객체 생성 후 연관관계 설정하고 반환
    * 패러다임 불일치를 해결하려고 소모하는 비용. 자바 컬렉션에서라면 이런 비용이 들지 않음
    * JPA : 개별 관계를 설정하고 해당 객체를 저장하기만 하면 된다. 조회시 외래키를 참조로 변환하는 일도 처리
* \[불일치 이슈 3\] 객체 그래프 탐색
  * 참조를 사용해서 연관된 객체를 찾는 것
  * SQL을 직접 다루면 처음 실행하는 SQL에 따라 객체 그래프를 어디까지 탐색할 수 있는지 정해진다
  * 객체지향 개발자에게 너무 큰 제약
  * DAO를 통해 객체를 조회
    * 어디까지 객체 그래프를 탐색할 수 있을지는 예측 불가
    *  DAO를 열어서 SQL을 직접 확인해야한다. 엔티티가 SQL에 논리적으로 종속되어서 발생하는 문제
    * 모든 객체 그래프를 DB에서 조회해서 메모리에 올려두는 것 - 현실성 없음
    * DAO에 조회하는 메소드를 상황에 따라 여러 벌 만들어야
  * JPA를 사용하면 객체 그래프를 신뢰하고 마음껏 탐색 가능
  * 지연 로딩 : 실제 객체를 사용하는 시점까지 DB 조회를 미룬다
  * 물론 연관관계의 객체들 모두 즉시 함께 조회되도록 할 수도 있다.
* \[불일치 이슈 4\] 비교
  * 데이터베이스 : 기본키의 값으로 각 Row를 구분
  * 객체 : 동일성 비교, 동등성 비교
    * 동일성 : ==, 객체 인스턴스의 주소값 비교
    * 동등성 : equals\(\), 객체 내부의 값을 비교
  * 같은 데이터베이스 로우를 조회해도 객체에서 받으면 서로 다른 인스턴스
    * 동일성 비교에서 실패가 된다. false
    * 객체를 컬렉션에 보관했다면 동일성 비교에서 성공했을 것. true
  * 여기에 여러 트랜잭션이 동시에 실행되는 상황까지 고려하면 해당 이슈 해결이 더 어려워 짐
  * JPA에서는 같은 트랜잭션일 때 같은 객체가 조회되는 것을 보장
  * 객체 비교하기 : 분산 환경이나 트랜잭션이 다른 상황까지 고려하면 더 복잡 \(추후에 알아볼 예정\)
* 불일치 이슈 정리
  * 객체 모델과 관계형 데이터베이스 모델은 지향하는 패러다임이 서로 다르다
  * 이를 극복하기 위해 많은 시간 + 코드 소비
  * 객체지향 어플리케이션답게 정교한 객체 모델링을 할수록 패러다임 불일치 문제가 더 커짐
    * 객체 모델링은 점점 힘을 잃고 데이터 중심의 모델로 변해감
  * 이를 극복하기 위해 나온 것이 JPA



### 1.3. JPA란 무엇인가?

* Java Persistence API : 자바 진영의 ORM 기술 표준
  * 애플리케이션과 JDBC 사이에서 동작
* ORM : Object-Relational Mapping
  * ORM 프레임워크 : 객체와 테이블을 매핑해서 패러다임의 불일치 문제를 개발자 대신 해결
  * SQL을 직접 작성하는 것이 아니라 객체를 마치 자바 컬렉션에 저장하듯
* JPA를 사용해서 객체를 저장하는 코드

```text
jpa.persist(member); // 저장
```

* 조회도 마찬가지. 객체를 직접 조회.

```text
Member member = jpa.find(memberId);
```

* 객체 측면에서 정교한 객체 모델링을 할 수 있고
* 관계형 데이터베이스는 데이터베이스에 맞도록 모델링하면 된다
* 둘의 매핑 방법만 ORM 프레임워크에게 알려주면 된다. 
* 개발자는 관계형 데이터베이스를 사용해도 객체지향 애플리케이션 개발에만 집중 가능. **쌉가능.**
* 자바 진영 대표적인 ORM 프레임워크 : 하이버네이트
* 엔터프라이즈 자바 빈즈 \(EJB\)
  * 기술 표준, 엔티티 빈이라는 ORM 기술도 포함
  * 복잡 + 기술 성숙도도 떨어뜨림 + 자바 엔터프라이즈\(J2EE\) 애플리케이션 서버에서만 동작
* 하이버네이트\(Hibernate\) 등장
  * 오픈소스 ORM 프레임워크
  * EJB와 비교해서 가볍고 실용적, 기술 성숙도도 높았다
  * J2EE 없이도 동작해서 개발자들이 많이 사용
  * EJB 3.0에서 하이버네이트를 기반으로 새로운 자바 ORM 기술 표준이 만들어졌으니... 
    * 이것이 바로 JPA \(두둥\)
* JPA
  * 자바 ORM 기술에 대한 API 표준 명세
  * JPA를 사용하려면 JPA를 구현한 ORM 프레임워크를 선택해야
  * JPA 2.1을 구현한 대표 ORM 프레임워크 : 하이버네이트
  * 특정 구현 기술에 대한 의존도를 줄일 수 있다. 다른 구현 기술로 손쉽게 이동도 가능
  * 표준을 먼저 이해하고 필요에 따라 JPA 구현체가 제공하는 고유의 기능을 알아가면 됨
  * 버전은 1.0, 2.0, 2.1
* 왜 JPA를 사용해야?
  * 생산성
    * JDBC API를 사용하는 지루하고 반복적인 일을 대신 처리
  * 유지보수
    * 기존 : 엔티티 수정 시 관련 SQL과 매핑하기 위한 JDBC API 코드를 모두 변경해야
    * 이런 과정도 JPA가 대신 처리
  * 패러다임의 불일치 해결
    * 상속, 연관관계, 객체 그래프 탐색, 비교하기 등등의 이슈 해결
  * 성능
    * 애플리케이션과 DB 사이에서 다양한 성능 최적화 기회를 제공
    * 예\) 객체 재사용
  * 데이터 접근 추상화와 벤더 독립성
    * 벤더마다 사용법이 다른 관계형 데이터베이스. 기술에 종속적일 수 있다.
    * But, JPA는 애플리케이션과 DB 사이에 추상화된 데이터 접근 계층을 제공해서 애플리케이션이 특정 데이터베이스 기술에 종속되지 않도록
    * H2 -&gt; Oracle, MySQL 가능 \(다른 데이터베이스를 사용한다고 알려주기만 하면 됨\)
  * 표준
    * 자바 진영의 ORM 기술 표준. 표준을 사용하면 다른 구현 기술로 손쉽게 변경할 수 있다.



### ORM에 대한 궁금증과 오해

* ORM 프레임워크를 사용하려면 SQL과 데이터베이스도 잘 알아야 한다
  * 옳바른 매핑을 위해서는 양쪽을 잘 이해해야
* JPA는 다양한 성능 최적화 기능을 제공
  * SQL을 직접 사용하는 것보다 더 좋은 성능을 낼 수도
  * 한 번 SQL을 실행해서 나온 결과 수만큼 N번 SQL을 추가로 실행하는 N+1 문제는 주의 할 것
* JPA는 통계 쿼리\(복잡한 쿼리\)보다는 실시간 처리용 쿼리에 더 최적화
  * 복잡한 통계 쿼리는 네이티브나 마이바티스, 스프링의 JdbcTemplate과 같은 SQL 매퍼 형태의 프레임워크를 혼용하는 것도 좋은 방법
* 마이바티스는 SQL 매퍼
  * JDBC API 사용과 응답 결과를 객체로 매핑하는 반복적인 일을 대신 처리
  * 그러나 개발자가 SQL을 직접 작성해야
  * ORM은 매핑만 하면 ORM 프레임워크에서 SQL도 만들어준다. SQL에 의존하는 개발을 피할 수 있다
* 하이버네이트 프레임워크의 신뢰도
  * 내가 만드는 것보다 훨씬 좋다. 역량이 높아지기 전까지는 걍 쓰자
  * 2001년도에 공개된 역사와 전통이 있는 기술
* 마이바티스보다 ORM 프레임워크를 사용하는 비중이 절대적으로 높다 \(글로벌 기준\)
* 학습곡선이 높다
  * 어떻게 매핑하는지, JPA의 핵심 개념, 영속성 컨텍스트에 대한 이해
  * 어려운 근본적인 이유 : ORM이 객체지향과 관계형 데이터베이스라는 두 기둥 위에 있기 때문
  * 기초부터 열심히 공부하자


