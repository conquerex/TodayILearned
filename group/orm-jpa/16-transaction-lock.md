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

        <aside>
        📌 트랜잭션 격리 수준에 따른 동작 방식은 데이터베이스마다 조금씩 다르다. 최근에는 데이터베이스들이 더 많은 동시성 처리를 위해 락보다는 MVCC(Multiversion_concurrency_control)을 사용하므로 락을 사용하는 데이터베이스와 약간 다른 특성을 지닌다.

        </aside>

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

<aside>
📌 벌크 연산은 버전을 무시한다. 벌크 연산에서 버전을 증가하려면 버전 필드를 강제로 증가시켜야 한다.
update Member m set [m.name](http://m.name/) = '변경', m.version = m.version + 1

</aside>

- JPA 락 사용
    - 락은 다음 위치에 적용할 수 있다.
        - EntityManager.lock(), EntityManager.find(), EntityManager.refresh()
        - Query.setLockMode() (TypeQuery 포함)
        - @NamedQuery
    - JPA가 제공하는 락 옵션은 javax.persistence.LockModeType에 정의되어 있다.
    - 표 16.2 LockModeType 속성 (p.701)

<aside>
📌 JPA를 사용할 때 추천하는 전략은 `READ COMMITED` 트랜잭션 격리 수준 + 낙관적 버전 관리(두 번의 갱신 내역 분실 문제 예방)이다.

</aside>

- JPA 낙관적 락
    - 낙관적 락을 사용하려면 버전이 있어야 한다.
    - 낙관적 락은 트랜잭션을 커밋하는 시점에 충돌을 알 수 있다는 특징이 있다.
    - 낙관적 락에서 발생하는 예외
        - javax.persistence.OptimisticLockException(JPA 예외)
        - org.hibernate.StaleObjectStateException(하이버네이트 예외)
        - org.springframework.orm.ObjectOptimisticLockingFailureException(스프링 예외 추상화)

        <aside>
        📌 일부 JPA 구현체 중에는 @Version 컬럼 없이 낙관적 락을 허용하기도 하지만 추천하지는 않는다

        </aside>

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


ㄲ
