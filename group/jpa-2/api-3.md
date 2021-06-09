# API 개발 고급 - 컬렉션 조회 최적화

### 강의 항목

* 주문 조회 V1: 엔티티 직접 노출
* 주문 조회 V2: 엔티티를 DTO로 변환
* 주문 조회 V3: 엔티티를 DTO로 변환 - 페치 조인 최적화
* 주문 조회 V3.1: 엔티티를 DTO로 변환 - 페이징과 한계 돌파
* 주문 조회 V4: JPA에서 DTO 직접 조회
* 주문 조회 V5: JPA에서 DTO 직접 조회 - 컬렉션 조회 최적화
* 주문 조회 V6: JPA에서 DTO로 직접 조회, 플랫 데이터 최적화
* API 개발 고급 정리



### 실습 코드

* [https://github.com/conquerex/WhatTheJpa2nd/commit/bf6a22cdc4ef5a326b20928a4c6bd491456c3075](https://github.com/conquerex/WhatTheJpa2nd/commit/bf6a22cdc4ef5a326b20928a4c6bd491456c3075)
* [https://github.com/conquerex/WhatTheJpa2nd/commit/ce880f7d44000038734891b9af776a021d7489cc](https://github.com/conquerex/WhatTheJpa2nd/commit/ce880f7d44000038734891b9af776a021d7489cc)
* [https://github.com/conquerex/WhatTheJpa2nd/commit/b0932ec8e95465b7f8c7d0238c916fa6cc0c6dd3](https://github.com/conquerex/WhatTheJpa2nd/commit/b0932ec8e95465b7f8c7d0238c916fa6cc0c6dd3)
* 


### 학습 내용

* 주문 내역에서 추가로 주문한 상품 정보를 추가로 조회
* Order 기준으로 컬렉션인 OrderItem와 Item이 필요
* 앞 예제의 경우, toOne 관계만
  * OneToOne, ManyToOne
* 이번에는 컬렉션인 일대다 관계\(ManyToOne\)를 조회 및 최적화



#### V1: 엔티티 직접 노출

* 당연히 안좋은 방식
  * 엔티티가 변하면 API 스펙이 변한다
  * 트랜잭션 안에서 지연로딩 필요
  * 양방향 연관관계 문제, 무한루프 - JsonIgnore 설정 필요
  * Hibernate5Module 모듈 등록
  * LAZY 강제 초기화 처리
    * orderItem, item 관계를 직접 초기화하면 Hibernate5Module 설정에 의해 엔티티를 Json으로 생성



#### V2: 엔티티를 DTO로 변환

*  DTO 내부에도 엔티티가 없어야 한다.
* 하지만 밸류값\(예. Address\)은 상관없다.
* 지연로딩으로 너무 많은 SQL 실행
  * 영속성 컨텍스트에 원하는 엔티티가 존재하면 SQL을 실행하지 않는다



#### V3: 엔티티를 DTO로 변환 - 페치 조인 최적화

* fetch 조인으로 SQL이 한번만 실행됨
* distinct
  * 1대다 조인 : 데이터베이스 row가 증가. 같은 order 엔티티의 조회수도 증가
  * JPA의 distinct는 SQL에 distinct를 추가하고
  * 같은 엔티티가 조회되면 에플리케이션에서 중복을 걸러준다
  * 예\) order가 컬렉션 페치 조인 때문에 중복 조회되는 것을 막아준다
* 단점 : 페이징 불가능
  * 하이버네이트
    * 경고 로그를 남김
    * 모든 데이터를 DB에서 읽어옴
    * 메모리에서 페이징 \(매우 위험!!\)
* 주의 : 컬렉션 페치 조인은 1개만 사용할 수 있다
  * 컬렉션 둘 이상에 페치 조인을 사용하면 안된다. \(1 : N : M\)
  * 데이터가 부정합하게 조회될 수 있다



#### V3.1: 엔티티를 DTO로 변환 - 페이징과 한계 돌파

* 페이징과 한계돌파
  * 컬렉션을 페치 조인하면 페이징 불가능
  * 1대다 조인이 발생 - 데이터가 예측할 수 없이 증가
  * 1대다에서 1을 기준으로 페이징하는 것이 목적
    * 결과 : 다\(N\)를 기준으로 row가 생성
  * 하이버네이트는 경고로그 +모든 DB 데이터 읽기 + 메모리에서 페이징 시도 --&gt; 최악의 경우 장애로 이어짐
* 한계돌파 \(페이징 + 컬렉션 엔티티 조회 문제\)
  * ToONe 관계는 모두 페치 조인
    * row 수를 증가시키지 않으므로 페이징 쿼리에 영향을 주지 X
  * 컬렉션은 지연 로딩으로 조회
  * 지연로딩 성능 최적화
    * _**hibernate.default\_batch\_fetch\_size**_ : 글로벌 설정
    * @BatchSize : 개별 최적화
      * 컬렉션은 컬렉션 필드에
      * 엔티티는 엔티티 클래스에 적용
    * 컬렉션이나 프록시 객체를 한꺼번에 설정한 size만큼 IN 쿼리로 조회
    * 예\) where item0\_.item\_id in \(?, ?, ?, ?\)

```text
@GetMapping("/api/v3.1/orders")
public List<OrderDto> orderV3_page(
        @RequestParam(value = "offset", defaultValue = "0") int offset,
        @RequestParam(value = "limit", defaultValue = "100") int limit
) {
    List<Order> orders = orderRepository.findAllWithMemberDeilvery(offset, limit);

    return orders.stream()
            .map(OrderDto::new)
            .collect(Collectors.toList());
}
```

```text
# application.yml
    jpa:
    hibernate:
      ddl-auto: create
    properties:
      hibernate:
        format_sql: true
        default_batch_fetch_size: 1000
```

* 장점
  * 쿼리 호출수 : 1 + N --&gt; 1 + 1로 최적화
  * 조인보다 DB 데이터 전송량이 최적화
    * \(기존\) Order와 OrderItem 조인 : OrderItem만큼 중복해서 조회
    * 이 방법 사용시 중복 제거
  * 페치 조인 방식과 비교, 쿼리 호출 수 약간 증가
    * But DB 데이터 전송량이 감소
  * 컬렉션 페치 조인은 페이징이 불가능
    * 이 방법 사용시 페이징 가능
* **ToOne 관계는 페치 조인해도 페이징에 영향을 주지 않는다**
  * 페치 조인으로 쿼리 수를 줄이고
  * 나머지는 hibernate.default\_batch\_fetch\_size로 최적화
* hibernate.default\_batch\_fetch\_size의 크기
  * 100~1000 사이
  * SQL IN 절 사용. 데이터베이스에 따라 IN절 파라미터를 1000으로 제한하기도 한다.
  * 1000 : DB에서 애플리케이션으로 1000개 불러오므로 DB에 순간 부하가 증가
  * 애플리케이션 : 100이든 1000이든 결국 전체 데이터를 로딩해야하므로 메모리 사용량이 같다
  * 1000으로 설정하는 것이 성능상 가장 좋지만
    * DB든 애플리케이션이든 순간 부하를 어디까지 견딜 수 있는지로 결정



#### V4: JPA에서 DTO 직접 조회

* Query : 루트 1회, 컬렉션 N회 실행
  * 단건 조회에서 많이 사용하는 방식
  * OrderQueryRepository
  * 여기서 order를 1회 조회, 2개의 결과 출력
  * order 결과가 2개이므로 orderItem 2회 조회
* findOrders
  * 1대다가 아닌 부분은 여기서 모두 조회
* findOrderItems
  * 1대다 관계 조회
* ToOne 관계들을 먼저 조회, ToMany 관계는 각각 별도로 처리
  * ToOne : 조인해도 데이터 row 수가 증가하지 않는다.
  * ToMany : 증가한다.
* ToOne : 증가하지 않으므로 조인으로 최적화하기 쉽다. 한번에 조회
* ToMany : 최적화하기 어려우므로 findOrderItems\(\)같은 별도의 메서드로 조회



#### V5: JPA에서 DTO 직접 조회 - 컬렉션 조회 최적화

* Query : 루트 1회, 컬렉션 1회 실행
  * 데이터를 한꺼번에 처리할 때 많이 사용하는 방식
  * findOrders : ToOne, 루트 조회
  * findOrderItemMap : ToMany, orderItem 컬렉션을 Map 한방에 조회
  * 루프를 돌면서 컬렉션 추가 \(추가 쿼리 실행 X\)



#### V6: JPA에서 DTO로 직접 조회, 플랫 데이터 최적화

* 필요한 데이터를 OrderFlatDto로 **1회**에 조회
* 메모리에서 OrderFlatDto로부터 OrderQueryDto와 OrderItemQueryDto를 뽑아낸다. \(분해조립\)
* 단점
  * 쿼리는 한번, But 조인으로 인해 DB에서 애플리케이션에 전달하는 데이터에 중복 데이터가 추가
    * 상황에 따라 V5보다 더 느릴 수 있다
  * 애플리케이션에서 추가 작업이 크다
  * 페이징 불가능
    * 실제 DB는 중복 데이터가 그대로 출력



#### API 개발 고급 정리

* 엔티티 조회
  * v1
    * \(비추천\) 엔티티를 그대로 조회해서 반환
  * v2
    * 엔티티 조회 후 DTO로 변환
    * 여러 테이블을 조인할 때 낮은 퍼포먼스
  * v3
    * 페치 조인으로 쿼리 수 최적화
    * 페이징 불가능
  * v3.1
    * 컬렉션은 페치 조인시 페이징이 불가능
    * ToOne 관계는 페치 조인으로 쿼리 수 최적화
    * 컬렉션은 페치 조인 대신 지연로딩을 유지
      * hibernate.default\_batch\_fetch\_size
      * @BatchSize로 최적화
* DTO 직접 조회
  * v4
    * JPA에서 DTO를 직접 조회
  * v5
    * 컬렉션 조회 최적화
    * 일대다 관계인 컬렉션
    * IN절을 활용해 메모리에 미리 로딩\(조회\)해서 최적화
  * v6
    * 플랫 데이터 최적화
    * JOIN 결과를 그대로 조회 후
    * 애플리케이션에서 원하는 모양으로 직접 변환
* 권장 순서
  * 엔티티 조회 방식으로 우선 접근
    * 페치조인으로 쿼리수를 최적화
    * 컬렉션 최적화
      * 페이징 필요 : hibernate.default\_batch\_fetch\_size or @BatchSize
      * 페이징 불필요 : 페치 조인 사용
  * 엔티티 조회 방식으로 해결이 안되면 DTO 조회 방식 사용
  * DTO 조회 방식으로 해결안되면
    * 네이티브 SQL or 스프링 JdbcTemplate
* 엔티티 조회 방식이 우선순위가 높은 이유
  * 페치조인이나 hibernate.default\_batch\_fetch\_size, @BatchSize 같이 코드를 거의 수정하지 않고 옵션만 약간 변경해서 다양한 성능 최적화를 시도할 수 있다.
  * 반면에 DTO를 직접 조회하는 방식
    * 성능 최적화 또는 성능 최적화 방식을 변경할 때 많은 코드를 변경해야
* 성능 최적화와 코드 복잡도 사이에서 줄타기
  * \(항상은 아님\) 보통 성능 최적화는 단순한 코드를 복잡한 코드로 몰고간다
  * 엔티티 조회 방식 : JPA가 많은 부분을 최적화 해주기 때문
    * 단순한 코드를 유지하면서 성능을 최적화할 수 있다.
  * 반면에 DTO 조회 방식
    * SQL을 직접 다루는 것과 유사
    * 둘 사이에 줄타기를 해야
* _**DTO 조회 방식의 선택지**_
  * V6 방식이 1회만 조회된다고 해서 항상 좋은 방법은 아니다
  * V4는 단순
    * 특정 주문 1건만 조회하면 이 방식을 사용해도 성능이 잘 나온다.
    * 예\) order 데이터 1건, orderItem을 찾기 위한 쿼리도 1번만 실행
  * V5는 코드가 복잡
    * 여러 주문을 한꺼번에 조회하는 경우
      * v4 대신 이것을 최적화 한 v5방식을 사용
      * 예\) order 1000건, v4 쿼리 1 + 1000번, v5 쿼리 1 + 1번
    * 운영 환경에 따라 100배 이상의 성능 차이가 날수도
  * V6는 완전히 다른 접근방식
    * 쿼리 한번으로 최적화. 네트워크로는 이득
    * order를 기준으로 페이징 불가능
    * 데이터가 많은면 중복 전송이 증가해서 v5와 비교해서 성능 차이도 미비



END😜

