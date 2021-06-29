---
description: '2021.06.01, 화요일'
---

# 상품 도메인 개발

## 강의 항목

* 상품 엔티티 개발\(비즈니스 로직 추가\)
* 상품 리포지토리 개발
* 상품 서비스 개발

## 실습 코드

* [https://github.com/conquerex/WhatTheJpa2nd/commit/d7acb63b05190dba2061bea3534cac207f3b7097](https://github.com/conquerex/WhatTheJpa2nd/commit/d7acb63b05190dba2061bea3534cac207f3b7097)

## 학습 내용

* 영속성 컨텍스트에 객체가 있는 경우와 없는 경우
  * 있다면 merge. 추후 merge에 대해 따로 다룰 예정

```text
public void save(Item item) {
    if (item.getId() == null) {
        em.persist(item);
    } else {
        // item이 있네?
        // update와 비슷한 것. 추후 본 주석 수정 예정
        em.merge(item);
    }
}
```

