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
    * hibernate.default\_batch\_fetch\_size : 글로벌 설정
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

* 


#### V5: JPA에서 DTO 직접 조회 - 컬렉션 조회 최적화

* 


#### V6: JPA에서 DTO로 직접 조회, 플랫 데이터 최적화

* 


#### API 개발 고급 정리

* 




END😜

