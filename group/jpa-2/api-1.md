# API 개발 고급 - 준비

## 강의 항목

* API 개발 고급 소개
* 조회용 샘플 데이터 입력

## 실습 코드

* [https://github.com/conquerex/WhatTheJpa2nd/commit/35a0d274862668173f084465f7a5e1f25465ddc6](https://github.com/conquerex/WhatTheJpa2nd/commit/35a0d274862668173f084465f7a5e1f25465ddc6)

## 학습 내용

* Member, Item\(Book\), OrderItem, Delivery, Order 샘플 만들기
* @PostConstruct
  * 스프링이 다 올라온 뒤 호출

```text
@PostConstruct
public void init() {
        initService.dbInit1();
        initService.dbInit2();
}

@Component
@Transactional
@RequiredArgsConstructor
static class InitService {
        private final EntityManager em;
        // 이하 생략
}
```

* init\(\)과 InitService를 분리한 이유
  * init\(\)에 트랜잭션을 넣을 시 제대로 동작하지 않을 수 있다.
  * 트랜잭션 수행은 분리하는 것을 추천

