# API 개발 고급 - 지연 로딩과 조회 성능 최적화

## 강의 항목

* 간단한 주문 조회 V1: 엔티티를 직접 노출
* 간단한 주문 조회 V2: 엔티티를 DTO로 변환
* 간단한 주문 조회 V3: 엔티티를 DTO로 변환 - 페치 조인 최적화
* 간단한 주문 조회 V4: JPA에서 DTO로 바로 조회

## 실습 코드

* [https://github.com/conquerex/WhatTheJpa2nd/commit/55f4eb9ece51347367fd95b70a49ba6481eeae88](https://github.com/conquerex/WhatTheJpa2nd/commit/55f4eb9ece51347367fd95b70a49ba6481eeae88)
* [https://github.com/conquerex/WhatTheJpa2nd/commit/3aa0b8d12a129f65f7cbd92d3f98551e6c29438e](https://github.com/conquerex/WhatTheJpa2nd/commit/3aa0b8d12a129f65f7cbd92d3f98551e6c29438e)
* [https://github.com/conquerex/WhatTheJpa2nd/commit/5df0ea0a00a6db186cac911be94ad8bdd79dd413](https://github.com/conquerex/WhatTheJpa2nd/commit/5df0ea0a00a6db186cac911be94ad8bdd79dd413)

## 학습 내용

* API 개발 고급 - 지연 로딩과 조회 성능 최적화
  * 주문 + 배송정보 + 회원을 조회하는 API를 만들자
  * 지연 로딩 때문에 발생하는 성능 문제를 단계적으로 해결해보자

### V1: 엔티티를 직접 노출

* xToOne\(ManyToOne, OneToOne\) 관계 최적화
  * Order
  * Order -&gt; Member \(지연로딩\)
  * Order -&gt; Delivery \(지연로딩\)
* 그냥 넣고 조회하면 무한루프가 발생
  * Order --&gt; Member --&gt; Order..
  * 양방향 연관관계가 걸리는 곳에 모두 JsonIgnore를 넣어줘야 한다
* _**엔티티를 직접 노출하는 것은 좋지 않다.**_
* 지연로딩
  * 실제 엔티티 대신에 프록시 존재
  * jackson 라이브러리는 이 프록시 객체를 json으로 어떻게 생성해야 하는지 모름 --&gt; 예외 발생
  * Hibernate5Module을 사용하여 이 예외를 해결

```text
implementㅌation 'com.fasterxml.jackson.datatype:jackson-datatype-hibernate5'
```

* Hibernate5Module
  * \(스프링 부트 사용중\) 스프링 빈으로 등록
  * 강제로 지연로딩을 사용하는 방법도 있음
  * Hibernate5Module.Feature.FORCE\_LAZY\_LOADING
    * order --&gt; member, member --&gt; order
    * 양방향 연관고나계를 계속 로딩하게 된다. 따라서 @JsonIgnore 옵션을 한곳에 주어야 한다.
* 지연 로딩\(LAZY\)을 피하기 위해 즉시 로딩\(EAGER\)으로 설정하면 안된다.
  * 즉시로딩 때문에 연관고나계가 필요 없는 경우에도 데이터를 항상 조회해서 성능 문제가 발생 할 수 있다
  * 성능 튜닝이 매우 어려워 진다.
  * 성능 최적화가 필요한 경우에는 페치조인\(fetch join\)을 사용

### V2: 엔티티를 DTO로 변환

* 단점 : 지연로딩으로 쿼리 N번 호출 \(N + 1 문제\)
* order 조회시 쿼리가 1 + N + N번 실행
  * v1과 쿼리수 결과 동일
  * order 조회 1번 \(order 조회 결과 수 == N\)
  * order -&gt; member : 지연 로딩 조회 N번
  * order -&gt; delivery : 지연 로딩 조회 N번
  * 예\) order의 결과 4개
    * 쿼리 수행 : 최악의 경우 1 + 4 + 4 = 9번
* 지연 로딩은 영속성 컨텍스트에서 조회
  * 이미 조회된 경우 쿼리 수행하지 X
  * 최악이 아니다 뿐이지 안좋기는 마찬가지

### V3: 엔티티를 DTO로 변환 - 페치 조인 최적화

* fetch 조인 : LAZY 무시하고 객체를 가지고 온다
  * 쿼리 1번에 조회
* 페치 조인으로 order --&gt; member, order --&gt; delivery는 이미 조회된 상태
  * 지연 로딩 X

### V4: JPA에서 DTO로 바로 조회

* 쿼리 1회 호출
* select 절에서 원하는 필드만 선택해서 조회
* new 명령어를 사용해서 JPQL의 결과를 DTO로 즉시 변환
* 원하는 데이터를 직접 선택하므로 DB --&gt; 애플리케이션 네트웍 용량 최적화
  * 하지만 생각보다 미비
* 레포지토리 재사용성이 떨어짐
  * API 스펙에 맞춘 코드가 레포지토리에 들어가는 단점
  * 별도의 패키지로 관리하면 유지보수성이 올라감

### 정리

* 방법
  * 엔티티를 DTO로 변환
  * DTO로 바로 조회
* 쿼리 방식 선택 권장 순서
  * 엔티티를 DTO로 변환
  * 필요시 페치 조인으로 성능 최적화 \(대부분 여기서 성능 이슈 해결\)
  * DTO로 직접 조회하는 방법 \(V4\)
  * 최후의 방법
    * JPA가 제공하는 네이티브 SQL이나 스프링 JDBC Template을 사용해서 SQL을 직접 사용

