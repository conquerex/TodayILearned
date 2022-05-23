# 16장. 트랜잭션과 락, 2차 캐시

## 입구

- 트랜잭션과 락: JPA가 제공하는 트랜잭션과 락 기능을 다룬다.
- 2차 캐시: JPA가 제공하는 애플리케이션 범위의 캐시를 다룬다.

## 1. 트랜잭션과 락

- 트랜잭션 기초와 JPA가 제공하는 낙관전 락과 비관적 락에 대해 알아보자. (이름이 왜 이래...)
- 트랜잭션과 격리 수준
    - 트랜잭션은 ACID를 보장해야 한다.
    - **A**tomicity 원자성
        - 트랜잭션 내에서 실행한 작업들은 마치 하나의 작업인 것처럼 모두 성공하든가 모두 실패해야 한다.
    - **C**onsistency 일관성
        - 모든 트랜잭션은 일관성 있는 데이터베이스 상태를 유지해야 한다.
        - 예를 들어 데이터베이스에서 정한 무결성 제약 조건을 항상 만족해야 한다.
    - **I**solation 격리성
        - 동시에 실행되는 트랜잭션들이 서로에게 영향을 미치지 않도록 격리한다.
        - 예를 들어 동시에 같은 데이터를 수정하지 못하도록 해야 한다.
        - 격리성은 동시성과 관련된 성능 이슈로 인해 격리 수준을 선택할 수 있다.
    - **D**urability 지속성
        - 트랜잭션을 성공적으로 끝내면 그 결과가 항상 기록되어야 한다.
        - 중간에 시스템에 문제가 발생해도 데이터베이스 로그 등을 사용해서 성공한 트랜잭션 내용을 복구해야 한다.
    - 트랜잭션은 원자성, 일관성 지속성을 보장한다.
        - 문제는 격리성인데 트랜잭션 간에 격리성을 원벽히 보장하려면 트랜잭션을 거의 차례대로 실행해야 한다.
        - 이렇게 하면 동시성 처리 성능이 매우 나빠진다.
        - 이런 문제로 인해 ANSI 표준은 트랜잭션의 격리 수준을 4단계로 나누어 정의했다.
            - READ UNCOMMITED(커밋되지 않은 읽기)
                - 격리 수준이 가장 낮다
            - READ COMMITTED(커밋된 읽기)
            - REPEATABLE READ(반복 가능한 읽기)
            - SERIALIZABLE(직렬화 가능)
                - 격리 수준이 가장 높다
        - 격리 수준이 낮을 수록 동시성은 증가하지만 격리 수준에 따른 다양한 문제가 발생
        - 표 16.1 태랜잭션 격리 수준과 문제점 (p.695)
    - 격리 수준에 따른 문제점 리스트
        - DIRTY READ
        - NON-REPEATABLE READ(반복 불가능한 읽기)
        - PHANTOM READ
    - 트랜잭션 격리 수준에 따른 문제점
        - READ UNCOMMITTED
            - 커밋하지 않은 데이터를 읽을 수 있다.
            - DIRTY READ 허용
                - 트랜잭션 1이 데이터를 수정하고 있는데 커밋하지 않아도 트랜잭션 2가 수정 중인 데이터를 조회할 수 있다.
                - 트랜잭션 2가 DIRTY READ한 데이터를 사용하는데 트랜잭션 1을 롤백하면 데이터 정합성에 심각한 문제가 발생할 수 있다.
        - READ COMMITTED
            - 커밋한 데이터만 읽을 수 있다.
            - NON-REPEATABLE READ 허용
                - 트랜잭션 1이 회원 A를 조회 중인데 갑자기 트랜잭션 2가 회원 A를 수정하고 커밋하면 트랜잭션 1이 다시 회원 A를 조회했을 때 수정된 데이터가 조회된다.
            - DIRTY READ는 허용하지 않음, NON-REPEATABLE READ는 허용
        - REPEATABLE READ
            - 한 번 조회한 데이터를 반복해서 조회해도 같은 데이터가 조회된다.
            - PHANTOM READ
                - 트랜잭션 1이 10살 이하의 회원을 조회했는데 트랜잭션 2가 5살 회원을 추가하고 커밋하면 트랜잭션 1이 다시 10살 이하의 회원을 조회했을 때 회원 하나가 추가된 상태로 조회된다.
                - 반복 조회 시 결과 집합이 달라진다.
            - NON-REPEATABLE READ는 허용하지 않음, PHANTOM READ는 허용
        - SERIALIZABLE
            - 가장 엄격한 트랜잭션 격리 수준이다.
            - PHANTOM READ가 발생하지 않는다.
            - 동시성 처리 성능이 급격히 떨어질 수 있다.
        - 애플리케이션 대부분은 동시성 처리가 중요하므로 데이터베이스들은 보통 READ UNCOMMITTED 격리 수준을 기본으로 사용한다.
            - 일부 중요한 비지니스 로직에 더 높은 격리 수준이 필요하면 데이터베이스 트랜잭션이 제공하는 잠금 기능을 사용하면 된다.

        <br>
       
        > 📌 트랜잭션 격리 수준에 따른 동작 방식은 데이터베이스마다 조금씩 다르다. 최근에는 데이터베이스들이 더 많은 동시성 처리를 위해 락보다는 MVCC(Multiversion_concurrency_control)을 사용하므로 락을 사용하는 데이터베이스와 약간 다른 특성을 지닌다.

        <br>

- 낙관적 락과 비관적 락 기초
    - JPA의 영속성 컨텍스트(1차 캐시)를 적절히 활용하면 데이터베이스 트랜잭션이 READ COMMITTED 격리 수준(3번째로 엄격한 수준)이어도 애플리케이션 레벨에서 반복 가능한 읽기 REPEATABLE READ(2번째로 엄격한 수준)가 가능하다.
    - JPA는 데이터베이스 트랜잭션 격리 수준을 READ COMMITTED 정도로 가정한다. 만약 일부 로직에 더 높은 격리 수준이 필요하면 낙관적 락과 비관적 락 중 하나를 사용하면 된다.
    - **낙관적 락**은 트랜잭션 대부분이 충돌이 발생하지 않는다고 낙관적으로 가정하는 방법이다.
        - 데이터베이스가 제공하는 락 기능을 사용하는 것이 아니라 JPA가 제공하는 버전 관리 기능을 사용한다.
        - 쉽게 이야기해서 애플리케이션이 제공하는 락
        - 트랜잭션을 커밋하기 전까지는 트랜잭션의 충돌을 알 수 없다.
    - **비관적 락**은 트랜잭션의 충돌이 발생한다고 가정하고 우선 락을 걸고 보는 방법이다.
        - 데이터베이스가 제공하는 락 기능을 사용한다.
        - 대표적으로 select for update 구문이 있다.
    - 두 번의 갱신 분실 문제 : 사용자 A가 수정하고 사용자 B가 1초 뒤에 수정 요청을 하게 되면 사용자 B의 수정사항만 남게 된다. 이것을 두 번의 갱신 분실 문제라 한다. 이를 해결하기 위한 3가지 선택 방법이 있다.
        1. 마지막 커밋만 인정하기 : 사용자 A의 내용은 무시하고 마지막에 커밋한 사용자 B의 내용만 인정한다.
        2. 최초 커밋만 인정하기 : 사용자 A가 이미 수정을 완료했으므로 사용자 B가 수정을 완료할 때 오류가 발생한다.
        3. 충돌하는 갱신 내용 병합하기 : 사용자 A와 사용자 B의 수정사항을 병합한다.
    - 기본은 마지막 커밋만 인정
        - 상황에 따라 최초 커밋만 인정하기도
        - JPA가 제공하는 버번 관리 기능을 사용하면 손쉽게 최초 커밋만 인정하기를 구현할 수 있다
        - 충돌하는 갱신 내용 병합하기는 최초 커밋만 인정하기를 조금 더 우아하게 처리하는 방법 - 애플리케이션 개발자가 직접 사용자를 위해 병합 방법을 제공해야 한다.

- @Version
    - JPA가 제공하는 낙관적 락을 사용하려면 @Version 어노테이션을 사용해서 버전 관리 기능을 추가해야 한다.
    - @Version 적용 가능 타입
        - Long(long)
        - Integer(int)
        - Short(short)
        - Timestamp

    ```java
    @Entity
    public class Board {

    	@Id
    	private String id;
    	private String title;

    	@Version
    	private Integer version;
    }
    ```

    - 엔티티에 버전 관리용 필드를 하나 추가하고 @Version을 붙이면 된다.
    - 엔티티를 수정할 때 마다 버전이 하나씩 자동으로 증가
    - 엔티티를 수정할 때 조회 시점의 버전과 수정 시점의 버전이 다르면 예외 발생
    - 버전 정보를 사용하면 최초 커밋만 인정하기가 적용

    ```java
    // 트랜잭션 1 조회 title="제목A", version=1
    Board board = em.find(Board.class, id);

    // 트랜잭션 2에서 해당 게시물을 수정해서 title="제목C", version=2로 증가

    board.setTitle("제목B"); // 트랜잭션 1 데이터 수정

    save(board);
    tx.commit(); //예외 발생, 데이터베이스 version=2, 엔티티 version=1
    ```


- 버전 정보 비교 방법
    - JPA가 버전 정보를 비교하는 방법
    - 엔티티를 수정하고 트랜잭션을 커밋하면 영속성 컨텍스트를 플러시 하면서 UPDATE 쿼리를 실행한다.
    - 버전은 엔티티의 값을 변경하면 증가한다.
    - 연관관계 필드는 외래 키를 관리하는 **연관관계의 주인 필드를 수정할 때만** 버전이 증가

<br>

> 📌 벌크 연산은 버전을 무시한다. 벌크 연산에서 버전을 증가하려면 버전 필드를 강제로 증가시켜야 한다.
update Member m set [m.name](http://m.name/) = '변경', m.version = m.version + 1

<br>


- JPA 락 사용
    - 락은 다음 위치에 적용할 수 있다.
        - EntityManager.lock(), EntityManager.find(), EntityManager.refresh()
        - Query.setLockMode() (TypeQuery 포함)
        - @NamedQuery
    - JPA가 제공하는 락 옵션은 javax.persistence.LockModeType에 정의되어 있다.
    - 표 16.2 LockModeType 속성 (p.701)

<br>

> 📌 JPA를 사용할 때 추천하는 전략은 `READ COMMITED` 트랜잭션 격리 수준 + 낙관적 버전 관리(두 번의 갱신 내역 분실 문제 예방)이다.

<br>

- JPA 낙관적 락
    - 낙관적 락을 사용하려면 버전이 있어야 한다.
    - 낙관적 락은 트랜잭션을 커밋하는 시점에 충돌을 알 수 있다는 특징이 있다.
    - 낙관적 락에서 발생하는 예외
        - javax.persistence.OptimisticLockException(JPA 예외)
        - org.hibernate.StaleObjectStateException(하이버네이트 예외)
        - org.springframework.orm.ObjectOptimisticLockingFailureException(스프링 예외 추상화)

        <br>

        > 📌 일부 JPA 구현체 중에는 @Version 컬럼 없이 낙관적 락을 허용하기도 하지만 추천하지는 않는다

        <br>

    - 락 옵션 없이 `@Version` 만 있어도 낙관적 락이 적용된다.
    - 락 옵션을 사용하면 락을 더 세밀하게 제어할 수 있다.
    - 낙관적 락의 옵션에 따른 효과
        - NONE
        - OPTIMISTIC
        - OPTIMISTIC_FORCE_INCREMENT
    - NONE
        - 락 옵션을 적용하지 않아도 `@Version`이 적용된 필드만 있으면 낙관적 락이 적용된다.
        - 용도 : 조회한 엔티티를 수정할 때 다른 트랜잭션에 의해 변경되지 않아야 한다. 조회 시점부터 **수정** 시점까지를 보장
        - 동작 : 엔티티를 수정할 때 버전을 체크하면서 버전을 증가한다.
        - 이점 : 두 번의 갱신 분실 문제를 예방한다.
    - OPTIMISTIC
        - `@Version` 만 적용했을 때는 엔티티를 수정해야 버전을 체크하지만 이 옵션을 추가하면 엔티티를 조회만 해도 버전을 체크한다. 한 번 **조회**한 엔티티는 트랜잭션을 종료할 때까지 다른 트랜잭션에서 변경되지 않음을 보장한다.
        - 용도 : 조회 시점부터 트랜잭션이 끝날 때까지 조회한 엔티티가 변경되지 않음을 보장
        - 동작 : 트랜잭션을 커밋할 때 버전 정보를 조회해서 현재 엔티티의 버전과 같은지 검증한다. 만약 같지 않으면 예외가 발생
        - 이점 : `OPTIMISTIC` 옵션은 `DIRTY READ`와 `NON-REPEATABLE READ`를 방지
    - OPTIMISTIC_FORCE_INCREMENT
        - 용도 : 논리적인 단위의 엔티티 묶음을 관리할 수 있다**.** 게시물과 첨부파일이 `일대다`, `다대일`의 `양방향 연관관계`이고 첨부파일이 연관관계의 주인이다. 게시물을 수정하는 데 단순히 첨부파일만 추가하면 게시물의 버전은 증가하지 않는다. 이때 게시물의 버전도 강제로 증가하려면 `OPTIMISTIC_FORCE_INCREMENT`를 사용하면 된다.
        - 동작 : 트랜잭션을 커밋할 때 `UPDATE` 쿼리를 사용해서 버전 정보를 강제로 증가시킨다. 추가로 엔티티를 수정하면 수정 시 버전 `UPDATE`가 발생한다. 따라서 총 2번의 버전 증가가 나타날 수 있다.
        - 이점 : 강제로 버전을 증가해서 논리적인 단위의 엔티티 묶음을 버전 관리할 수 있다.
- JPA 비관적 락
    - JPA가 제공하는 비관적 락은 데이터베이스 트랜잭션 락 매커니즘에 의존하는 방법이다.
    - 주로 SQL 쿼리에 select for update 구문을 사용하면서 시작하고 버전 정보는 사용하지 않는다.
    - 비관적 락은 주로 PESSIMISTIC_WRITE 모드를 사용한다.
- JPA 비관적 락 특징
    - 엔티티가 아닌 스클라 타입을 조회할 때도 사용할 수 있다.
    - 데이터를 수정한느 즉시 트랜잭션 충돌을 감지할 수 있다
- JPA 비관적에서 발생하는 예외
    - javax.persistence.PessimisticLockException(JPA 예외)
    - org.springframework.dao.PessimisticLockingFailureException(스프링 예외 추상화)
- 비관적 락의 옵션에 따른 효과
    - PESSIMISTIC_WRITE
        - 비관적 락이라 하면 일반적으로 이 옵션을 뜻한다. 데이터베이스에 쓰기 락을 걸 때 사용
        - 용도 : 데이터베이스에 쓰기 락을 건다
        - 동작 : 데이터베이스 `select for update`를 사용해서 락을 건다.
        - 이점 : `NON-REPEATABLE READ`를 방지한다. 락이 걸린 로우는 다른 트랜잭션이 수정할 수 없다.
    - PESSIMISTIC_READ
        - 데이터를 `반복 읽기`만 하고 수정하지 않는 용도로 락을 걸 때 사용한다.
        - 일반적으로 잘 사용하지 않는다.
        - 데이터베이스 대부분은 `PESSIMISTIC_WRITE`로 동작한다.
        - MySQL : lock in share mode
        - PostgreSQL : for share
    - PESSIMISTIC_FORCE_INCREMENT
        - 비관적 락중 유일하게 버전 정보를 사용한다.
        - 하이버네이트는 `nowait`를 지원하는 데이터베이스에 대해서 `for update nowait` 옵션을 적용한다.
        - 오라클 : for update nowait
        - PostreSQL : for update nowait
        - nowait를 지원하지 않으면 for update가 사용된다.
    - 비관적 락과 타임아웃
        - 비관적 락을 사용하면 락을 획득할 때까지 트랜잭션이 대기한다.
        - 무한정 기다릴 수는 없으므로 타임아웃 시간을 줄 수 있다.
        - 타임아웃은 데이터베이스 특성에 따라 동작하지 않을 수 있다.

```java
// 타임아웃 10초까지 대기 설정
properties.put("javax.persistence.lock.timeout", 10000);
Board board = em.find(Board.class, "boardId", LockModeType.PESSIMISTIC_WRITE, properties);
```

<br>

## 2. 2차 캐시

JPA가 제공하는 애플리케이션 범위의 캐시에 대해

하이버네이트와 EHCACHE를 사용해서 실제 캐시를 적용

- 1차 캐시와 2차 캐시
    - 네트워크를 통해 데이터베이스에 접근하는 시간 비용
        - 애플리케이션 서버에서 내부 메모리에 접근하는 시간 비용보다 수만에서 수십만 배 이상 비싸다.
        - 따라서 조회한 데이터를 메로리에 캐시해서 데이터베이스 접근 횟수를 줄이면 애플리케이션 성능을 획기적으로 개선할 수 있다.
    - 영속성 컨텍스트 내부에는 엔티티를 보관하는 저장소가 있는데 이것을 `1차 캐시`라 하며
        - 이를 통해 얻을 수 있는 이점이 많지만, 일반적인 웹 어플리케이션 환경은 트랜잭션을 시작하고 종료할 때까지만 1차 캐시가 유효하다. (사견 : 캐시없이 그냥 DB에 접근하는 것과 큰 차이가 없을 듯. 매번 같은 결과만 조회하지는 않을테니)
        - `OSIV`를 사용해도 클리언트의 요청이 들어올 때부터 끝날 때까지만 1차캐시가 유효하다.
        - 따라서 어플리케이션 전체로 보면 **데이터베이스 접근 횟수를 획기적으로 줄이지는 못한다.**
    - 하이버네이트를 포함한 대부분의 JPA 구현체들은 애플리케이션 범위의 캐시를 지원
        - 이것을 공유 캐시 또는 `2차 캐시`라 한다.
- 1차 캐시
    - 1차캐시는 영속성 컨텍스트 내부에 있다.
    - 엔티티 매니저로 조회하거나 변경하는 모든 엔티티는 1차캐시에 저장된다.
    - 트랜잭션을 커밋하거나 플러시를 호출하면 1차캐시에 있는 엔티티의 변경 내역을 데이터베이스에 동기화 한다.
    - JPA를 J2EE나 스프링 프레임워크 같은 컨테이너 위에서 실행하면
        - 트랜잭션을 시작할 때 영속성 컨텍스트를 생성하고 트랜잭션을 종료할 때 영속성 컨텍스트도 종료
    - OSIV를 사용하면 요청의 시작부터 끝까지 같은 영속성 컨텍스트를 유지
    - 1차 캐시는 끄고 켤 수 있는 옵션이 아니다.
        - **영속성 컨텍스트 자체가 사실상 1차 캐시다.**
    - 1차 캐시의 동작 방법
        1. 최초 조회할 때는 1차 캐시에 엔티티가 없으므로
        2. 데이터베이스에서 엔티티를 조회해서
        3. 1차 캐시에 보관하고
        4. 1차 캐시에 보관한 결과를 반환한다
        5. 이후 같은 엔티티를 조회하면 1차 캐시에 같은 엔티티가 있으므로 데이터베이스를 조회하지 않고 1차 캐시의 엔티티를 그대로 반환한다.
    - 1차캐시의 특징
        - 1차 캐시는 같은 엔티티가 있으면 해당 엔티티를 그대로 반환
            - 즉, 동일성(a==b)을 보장한다.
        - 1차 캐시는 기본적으로 영속성 컨텍스트 범위의 캐시다.
        (컨테이너 환경에서는 트랜잭션 범위의 캐시, OSIV를 적용하면 요청 범위의 캐시)
- 2차 캐시
    - **애플리케이션에서 공유하는 캐시**를 JPA는 공유 캐시(shared cache)라 하는데
        - 일반적으로 2차 캐시(second level cache, L2 cache)라 부른다.
        - 애플리케이션 범위의 캐시다. (애플리케이션을 종료할 때까지 캐시가 유지)
    - 분산 캐시나 클러스터링 환경의 캐시는 애플리케이션보다 더 오래 유지된다.
        - 2차 캐시를 적용하면 엔티티 매니저를 통해 데이터를 조회할 때 우선 2차 캐시에서 찾고 없으면 데이터베이스에서 찾는다.
        - 2차 캐시를 적절히 활용하면 데이터베이스 조회 횟수를 획기적으로 줄일 수 있다.
    - 2차 캐시의 동작 방식
        1. 영속성 컨텍스트는 엔티티가 필요하면 2차 캐시를 조회
        2. 2차 캐시에 엔티티가 없으면 데이터베이스를 조회
        3. 결과를 2차 캐시에 보관
        4. 2차 캐시는 자신이 보관하고 있는 엔티티를 복사해서 반환
        5. 2차 캐시에 저장되어 있는 엔티티를 조회하면 복사본을 만들어 반환
    - 동시성을 극대화하려고 캐시한 객체를 직접 반환하지 않고 복사본을 만들어서 반환한다.
    (락에 비하면 객체를 복사하는 비용이 저렴)
    - 2차 캐시의 특징
        - 영속성 유닛 범위의 캐시
        - 조회한 객체를 그대로 반환하는 것이 아니라 **복사본을 만들어서 반환**
        - 데이터베이스 기본 키를 기준으로 캐시하지만
            - 영속성 컨텍스트가 다르면 객체 동일성(a==b)을 보장하지 않는다.
- JPA 2차 캐시 기능
    - JPA 구현체 대부분은 캐시 기능을 각자 지원했는데
        - JPA는 2.0에 와서야 캐시 표준을 정의했다.
    - JPA 캐시 표준은 여러 구현체가 공통으로 사용하는 부분만 표준화해서
        - 세밀한 설정을 하려면 구현체에 의존적인 기능을 사용해야 한다.
- 캐시 모드 설정
    - 2차 캐시를 사용하려면 javax.persistence.Cacheable 어노테이션을 사용
    - @Casheable(true), @Cacheable(false) 설정할 수 있는데 기본값은 true이다.
    - persistence.xml에 shared-cache-mode를 설정해서 애플리케이션 전체에(정확히는 영속성 유닛 단위) 캐시를 어떻게 적용할지 옵션을 설정해야 한다.

    ```java
    @Cacheable
    @Entity
    public class Member {
    	@Id @GeneratedValue
        private Long id;

        ...
    }

    // persistence.xml에 캐시 모드 설정
    <persistence-unit name="test">
    	<shared-cache-mode>ENABLE_SELECTIVE</shared-cache-mode>
    </persistence-unit>

    // 캐시 모드 스프링 프레임워크 XML 설정
    <bean id="entitiMangerFacotry" class="org.springframework.orm.jpa.LocalContainerEntityMangerFactoryBean">
    	<property name="sharedCacheMode" value="ENABLE_SELECTIVE"/>
        ...
    ```

- SharedCacheMode 캐시 모드 설정
    - ALL: 모든 엔티티를 캐시한다.
    - NONE: 캐시를 사용하지 않는다.
    - **ENABLE_SELECTIVE: Cacheable(true)로 설정된 엔티티만 캐시를 적용한다. -> 보통 많이 사용**
    - DISABLE_SELECTIVE: 모든 엔티티를 캐시하는데 Cacheable(false)로 명시된 엔티티는 캐시하지 않는다.
    - UNSPECIFIED: JPA 구현체가 정의한 설정을 따른다.
- 캐시 조회, 저장 방식 설정
    - 캐시를 무시하고 데이터베이스를 직접 조회하거나 캐시를 갱신하려면 캐시 조회 모드와 캐시 보관 모드를 사용하면 된다.
    - 프로퍼티 이름
        - javax.persistence.cache.retrieveMode: 캐시 조회 모드 프로퍼티 이름
        - javax.persistence.cache.storeMode: 캐시 보관 모드 프로퍼티 이름
    - 옵션
        - javax.persistence.CacheRetrieveMode: 캐시 조회 모드 설정 옵션
        - javax.persistence.CacheStoreMode: 캐시 보관 모드 설정 옵션
    - 캐시 모드는 EntityManager.setProperty()로 엔티티 매니저 단위로 설정할 수 있다.
        - 혹은 EntityManager.find(), EntityManager.refresh()에 설정할 수 있다. (예제 16.12)
        - 그리고 Query.setHint()에 사용할 수 있다. (예제 16.13)

        ```java
        // 캐시 조회 모드
        public enum CacheRetrieveMode {
        		USE, // 캐시에서 조회
        		BYPASS // 캐시를 무시하고 데이터베이스에 직접 접근
        }

        // 캐시 보관 모드
        public enum CacheStoreMode {
        		USE, // 조회한 데이터를 캐시에 저장
        		BYPASS, // 캐시에 저장하지 않음
        		REFRESH // 데이터베이스에서 조회한 엔티티를 최신 상태로 다시 캐시
        }
        ```

- JPA 캐시 관리 API
    - JPA는 캐시를 관리하기 위한 javax.persistence.Cache 인터페이스를 제공한다.
    - 이것은 EntityManagerFactory에서 구할 수 있다.

    ```java
    Cache cache = emf.getCache();

    boolean contains = cache.contains(TestEntity.class, testEntity.getId());
    log.info("contains : {}", contains);

    // Cache 인터페이스
    public interface Cache {
    		// 해당 엔티티가 캐시에 있는지 확인
    		public boolean contains(Class cls, Object primaryKey);

    		// 해당 엔티티중 특정식별자를 가진 엔티티를 캐시에서 제거;
    		public void evict(Class cls, Object primaryKey);

    		// 해당 엔티티 전체를 캐시에서 제거
    		public void evict(Class cls);

    		// 모든 캐시 데이터 제거
    		public void evictAll();

    		// JPA Cache 구현체 조회
    		public <T> T unwrap(Class<T> cls);
    }
    ```

- 하이버네이트와 EHCACHE 적용
    - 하이버네이트가 지원하는 3가지 캐시
        1. 엔티티 캐시 : 엔티티 단위로 캐시한다. 식별자로 엔티티를 조회하거나 컬렉션이 아닌 연관된 엔티티를 로딩할 때 사용한다.
        2. 컬렉션 캐시 : 엔티티와 연관된 컬렉션을 캐시한다. **컬렉션이 엔티티를 담고 있으면 식별자 값만 캐시**한다(하이버네이트 기능)
        3. 쿼리 캐시 : 쿼리와 파라미터 정보를 키로 사용해서 캐시한다. **결과가 엔티티면 식별자 값만 캐시**한다.(하이버네이트 기능)
    - 참고로 JPA 표준에는 엔티티 캐시만 정의되어 있다.
    - 환경설정
        - 하이버네이트에서 EHCAHE를 사용하려면
            - hibernate-ehcache 라이브러리를 추가한다.
            - hibernate-ehcache를 추가하면 net.sf.ehcache-core 라이브러리도 추가된다.
        - EHCACHE는 ehcache.xml을 설정 파일로 사용한다.
        - 예제 16.16 ~ 16.18

    ```java
    // ehchace.xml 추가
    <ehcache>
    	<defaultCache
        	maxElementsInMemory="10000"
            eternal="false" // 저장된 캐시를 제거할지 여부를 설정한다. true 인 경우 저장된 캐시는 제거되지 않으며 timeToIdleSeconds, timeToLiveSeconds 설정은 무시된다.
            timeToIdleSeconds="1200"
            timeToLiveSeconds="1200"
            diskExpiryThreadIntervalSeconds="1200"
            memoryStoreEvictionPolicy="LRU"
            />
    </ehacahe>

    // persistence.xml에 캐시 정보 추가
    <persitence-unit name="test">
    	<shared-cahce-mode>ENABLE_SELECTIVE</shared-cache-modE>
        <properties>
        	// 2차 캐시 활성화, 엔티티 캐시와 컬렉션 캐시를 사용할 수 있음
        	<property name="hibernate.cache.use_second_level_cache" value="true"/>
            // 쿼리 캐시 활성화
            <proeprty name="hibernate.cache.use_query_cache" value="true"/>
            // 2차 캐시를 처리할 클래스 지정
            <property name="hibernate.cache.region.factory_class" value="org.hibernate.cache.ehcache.EhacheRegionFactory"/>
            // 하이버네이트가 여러 콩계 정보를 출력해주는데 캐시 적용 여부를 확인할 수 있음 (성능에 영향을 주므로 개발 환겨에서만 적용)
            <property name="hibernate.generate_statistics" value="true"/>
            ...
    </persistence-unit>
    ```

- 엔티티 캐시와 컬렉션 캐시

    ```java
    // 캐시 적용 코드
    @Cacheable // 엔티티 캐시 적용
    @Cache(usage = CacheConcurrencyStrategy.READ_WRITE) // 하이버네이트 전용. 캐시와 관련된 더 세밀한 설정할 때 사용
    @Entity
    public class ParentMember {
    	...

        @Cache(usage = CahceConcurrencyStrategy.READ_WRITE) // 컬렌션 캐시 적용
        @OneToMany(mappedBy = "parentMember", cascade = CascadeType.ALL)
        private List<ChidMember> childMembers = new ArrayList<ChildMember>();
        ....
    }
    ```

    - @Cache
        - 하이버네이트 전용인 org.hibernate.annotations.Cache 어노테이션을 사용하면 세밀한 캐시 설정이 가능
        - 속성
            - usage
                - CacheConcurrencyStrategy를 사용해서 캐시 동시성 전략을 설정한다.
                - NONE, READ_ONLY, NONSTRICT_READ_WRITE, READ_WRITE, TRANSACTIONAL (표 16.5)
            - regin
                - 캐시 지역 설정
            - include
                - 연관 객체를 캐시에 포함할지 선택한다. all(기본값), non-lazy 옵션을 선택할 수 있다.
        - 캐시 종류에 따라 동시성 전략 지원 여부는 하이버네이트 공식문서가 제공하는 표를 참고하자. (표 16.6)
        - ConcurrentHashMap은 개발 시에만 사용해야 한다. (질문 : [Why](http://blog.breakingthat.com/2019/04/04/java-collection-map-concurrenthashmap/)?)
    - 캐시 영역
        - 위에서 캐시를 적용한 코드는 다음 캐시 영역에 저장된다
        - 엔티티 캐시 영역
            - jpabook.jpashop.domain.test.cache.ParentMember
            - [패키지 명 + 클래스 명]
        - 컬렉션 캐시 영역
            - jpabook.jpashop.domain.test.cache.ParentMember.childMembers
            - [패키지 명 + 클래스 명 + 필드 명]
        - @Cache(region = "customRegion", ...)처럼 region 속성을 사용해서 캐시 영역을 직접 지정 할 수 있다.
        - 캐시 영역을 위한 접두사 설정은 persistence.xml 설정에 hibernate.cache.region_prefix를 사용하면 된다.
    - 쿼리 캐시
        - 쿼리 캐시는 쿼리와 파라미터 정보를 키로 사용해서 쿼리 결과를 캐시하는 방법
        - 쿼리 캐시를 적용하려면 유닛을 설정에 `hibernate.cache.use_query_cache` 옵션을 꼭 `true`로 설정해야 한다.

        ```java
        // 쿼리 캐시 적용
        em.createQuery("select i from Item i", Item.class)
        		**.setHint("org.hibernate.cacheable", true)**
        		.getResultList();

        // NamedQuery에 쿼리 캐시 적용
        @Entity
        @NamedQuery(
        				**hints = @QueryHint(name = "org.hibernate.cacheable",
        					value = "true"),**
        				name = "Member.findByUsername",
        				query = "select m.address from Member m where m.name = :username"
        )
        public class Member {
        	...
        }
        ```

    - 쿼리 캐시 영역
        - `hibernate.cache.use_query_cache` 옵션을 `true`로 설정해서 쿼리 캐시를 활성화하면 다음 두 캐시 영역이 추가된다.
        - org.hibernate.cache.internal.StandardCache
            - 쿼리 캐시를 저장하는 영역이다. 이곳에는 쿼리, 쿼리 결과 집합, 쿼리를 실행한 시점의 타임스탬프를 보관한다.
        - org.hibernate.cache.spi.UpdateTimestampsCache
            - 쿼리 캐시가 유효한지 확인하기 위해 쿼리 대상 테이블의 가장 최근 변경 시간을 저장하는 영역이다.
        - 쿼리 캐시는 캐시한 데이터 집합을 최신 데이터로 유지하려고 쿼리 캐시를 실행하는 시간과 쿼리 캐시가 사용하는 테이블들이 가장 최근에 변경된 시간을 비교한다.
            - 쿼리 캐시가 사용하는 테이블에 조금이라도 변경이 있으면 데이터베이스에서 데이터를 읽어 와서 쿼리 결과를 다시 캐시한다.
        - 쿼리 캐시를 잘 활용하면 **극적인 성능 향상이 있지만 빈번하게 변경이 있는 테이블에 사용하면 오히려 성능이 더 저하된다**.
            - 따라서 **수정이 거의 일어나지 않는 테이블에 사용해야 효과를 볼 수 있다.**


        <br>
        
        > 📌 org.hibernate.cache.spi.UpdateTimestampsCache 쿼리 캐시 영역은 만료되지 않도록 설정해야 한다. 해당 영역이 만료되면 모든 쿼리 캐시가 무효화된다. EHCACHE의 eternal=”true” 옵션을 사용하면 캐시에서 삭제되지 않는다.

        <br>

    - 쿼리 캐시와 컬렉션 캐시의 주의점
        - 엔티티 캐시를 사용해서 엔티티를 캐시하면 엔티티 정보를 모두 캐시하지만
            - 쿼리 캐시와 컬렉션 캐시는 **결과 집합의 식별자 값만 캐시**한다.
            - 그리고 이 식별자 값을 하나씩 엔티티 캐시에서 조회해서 실제 엔티티를 찾는다.
        - 문제는 쿼리 캐시나 컬렉션 캐시만 사용하고 대상 엔티티에 엔티티 캐시를 적용하지 않으면 성능상 심각한 문제가 발생할 수 있다.
            1. `select m from Member m` 쿼리를 실행 했는데 쿼리 캐시가 적용되어 있다. 결과 집합은 100건이다.
            2. 결과 집합에는 식별자만 있으므로 한 건씩 엔티티 캐시 영역에서 조회한다.
            3. `Member` 엔티티는 엔티티 캐시를 사용하지 않으므로 한 건씩 데이터베이스에서 조회한다.
            4. 결국 100건의 SQL이 실행된다.
        - 쿼리 캐시나 컬렉션 캐시만 사용하고 엔티티 캐시를 사용하지 않으면 최악의 상황에 결과 집합 수만큼 SQL이 실행된다. (질문 : N+1과 다른가?)
        - ***따라서 쿼리 캐시나 컬렉션 캐시를 사용하면 결과 대상 엔티티에는 꼭 엔티티 캐시를 적용해야 한다.***

<br>

## 3. 정리

- 트랜잭션의 격리 수준은 4단계가 있다. 격리 수준이 낮을수록 동시성은 증가하지만 격리 수준에 따른 다양한 문제가 발생한다.
- 영속성 컨텍스트는 데이터베이스 트랜잭션이 READ COMMITTED 격리 수준이어도 애플리케이션 레벨에서 반복 가능한 읽기(REPEATABLE READ)를 제공한다.
- JPA는 낙관적 락과 비관적 락을 지원한다.
    - 낙관적 락 : 애플리케이션이 지원하는 락
    - 비관적 락 : 데이터베이스 트랜잭션 락 메커니즘에 의존한다.
- 2차 캐시를 사용하면 애플리케이션의 조회 성능을 극적으로 끌어올릴 수 있다.

ㄲ
