---
description: '2021.06.02, 수요일'
---

# 웹 계층 개발 \(변경감지, 병합, 주문\)

## 강의 항목

* 변경 감지와 병합\(merge\)
* 상품 주문
* 주문 목록 검색, 취소
* 다음으로

## 실습 코드

* 
## 학습 내용

* 준영속 엔티티
  * 영속성 컨텍스트가 더 이상 관리하지 않는 엔티티
  * DB에 다녀온 객체 \(ex. Book\)
  * JPA가 관리하지 않는다.
* 변경감지 \(Dirty checking\)
  * 영속성 컨텍스트에서 엔티티를 다시 조회한 후 데이터를 수
  * 트랜잭션 안에서 조회, 변경 후 트랜잭션 커밋시점에 변경감지 작동
    * 이 동작에서 데이터베이스에 update SQL 실행

```text
    @Transactional
    public void updateItem(Long itemId, Book param) {
        // 영속성 엔티티
        Item findItem = itemRepository.findOne(itemId);

        findItem.setPrice(param.getPrice());
        findItem.setStockQuantity(param.getStockQuantity());

        // 아래를 수행할 필요가 없다.
        // findItem는 영속성 엔티티기 때문 (변경감지)
        // itemRepository.save(findItem);
    }
```

* 병합\(merge\)
  * 준영속 상태의 엔티티를 영속 상태로 변경할 때 사용
  * merge 실행
  * 1차 캐시에 엔티티가 없다면 DB에서 엔티티 조회 후 1차 캐시에 저장
  * 조회한 영속 엔티티에 본래의 엔티티 값을 채워 넣는다.
  * 영속 엔티티를 반환. 반환값은 영속 엔티티
  * 그러므로 모든 속성이 변경되는 상황. 값이 없으면 null로 업데이트 될수 있다.
* **BEST : 엔티티를 변경할 때는 항상 변경감지를 사용하세요**
* Controller에서는 엔티티보다는 DTO를
* 트랜잭션이 있는 서비스 계층에 식별자\(id\)와 변경할 데이터\(parm or dto\)를 명확하게 전달
* 트랜잭션이 있는 서비스 계층에서 영속 상태의 엔티티를 조회하고 엔티티의 데이터를 직접 변경할 것
* 트랜잭션 커밋 시점에 변경감지가 실행
* 상품주문
  * 주문폼, 필요한 member, item 정보 조회 \(GET\)
  * 주문 실행 \(POST\)
* 주문 목록 검색, 취소
  * 주문 목록 검색 \(GET\)
  * 주문 취소 \(POST\)
    * 취소 후 상품 갯수가 복구 되었는지 확

