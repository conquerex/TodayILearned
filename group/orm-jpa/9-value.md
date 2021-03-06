# 9장. 값 타입

## 입구

- JPA의 데이터 타입
    - 엔티티 타입
        - @Entity로 정의하는 객체
        - 식별자를 통해 지속해서 추적할 수 있다
        - ex) 키와 나이값이 변경된 유저 엔티티
        - 살아있는 생물
    - 값 타입
        - 단순히 값으로 사용하는 자바 기본 타입이나 객체
        - 식별자를 통해 지속해서 추적할 수 없다
        - ex) 100 -> 200
        - 단순한 수치 정보
- 값 타입의 3가지
    - 기본값 타입
        - 자바 기본 타입
        - 래퍼 클래스 (ex.Integer)
        - String
    - 임베디드 타입 (복합 값 타입)
        - JPA에서 사용자가 직접 정의한 값 타입
    - 컬렉션 값 타입
        - 하나 이상의 값 타입을 저장할 때

## 1. 기본값 타입

- 케이스 1. Member Entity
    - id라는 식별자 값도 가지고
    - 생명주기도 있다
- 케이스 2. name, age
    - 식별자 값도 없고
    - 생명주기도 없다
    - 회원 엔티티에 의존
        - 회원 엔티티 인스턴스를 제거하면 같이 제거됨
    - 공유를 하면 안됨
        - 다른 회원 엔티티의 이름이 변경된다고 해서 본 엔티티의 이름까지 변경? NO!!
- 자바에서...
    - int, double 같은 기본 타입(primitive type)은 절대 공유되지 않는다.
    - 래퍼 클래스(ex. Integer)나 String 같은 특수한 클래스
        - 객체지만 자바에서 기본 타입처럼 사용할 수 있게 지원하므로 기본값 타입으로 정의

## 2. 임베디드 타입(복합 값 타입)

- 새로운 값 타입을 직접 정의해서 사용 가능
- JPA에서는 이것을 임베디드 타입이라고...
- **중요 : 직접 정의한 임베디드 타입도 int, String처럼 값 타입이라는 것!!!**
- 회원 엔티티 예시
    - 회원이 상세한 데이터를 그대로 가지고 있는 것
    - 객체지향적이지 않으며 응집력만 떨어뜨림
    - 대신에 기타 정보가 있다면 코드가 더 명확해질 것
        - Period class
        - Address class
- 새로 정의한 값 타입 : 재사용, 응집도 높음
- 해당 값 타입만 사용하는 의미있는 메소드도 가능
- 둘 중 하나는 사용해야
    - @Embeddable : 값 타입을 정의하는 곳에 표시
    - @Embedded : 값 타입을 사용하는 곳에 표시
- 기본 생성자가 필수
- 임베디드 타입을 포함한 모든 값 타입
    - 엔티티의 생명주기에 의존하므로 엔티티와 임베디드 타입의 관계를 UML로 표현하면
    - 컴포지션 관계
- 하이버네이트 : 임베디드 타입을 컴포넌트라고 한다. components
- 테이블 매핑
    - 임베디드 타입은 엔티티의 값일 뿐
    - 값이 속한 텐티티의 테이블에 매핑한다
    - 잘 설계한 ORM 애플리케이션은 매핑한 테이블 수보다 클래스의 수가 더 많다
    - 더 객체지향적으로
- 연관관계
    - 임베디드 타입은 값 타입을 포함하거나 엔티티를 참조할 수 있다.
        - 엔티티는 공유될 수 있으므로 참조한다고 표현
        - 값 타입은 특정 주인에 소속되고 논리적인 개념상 공유되지 않으므로 포함된다고 표현
- @AttributeOverride : 속성 재정의
    - 임베디드 타입에 정의한 매핑정보를 재정의하려면?
    - 테이블에 매핑하는 컬럼명이 중복이 된다면?
    - 엔티티에 @AttributeOverride를 사용
    - AttributeOverride는 엔티티에 설정해야
- null
    - 임베디드 타입이 null → 매핑한 컬럼 값 모두 null

## 3. 값 타입과 불변 객체

- 값 타입은 복잡한 객체 세상을 조금이라도 단순화하려고
    - 따라서 단순하고 안전하게 다룰 수 있어야
- 값 타입 공유 참조
    - 임베디드 타입 같은 값 타입을 여러 엔티티에서 공유하면 위험
    - 값 타입의 객체를 그대로 참조해서 사용시 여러개의 엔티티에 동일한 값이 반영되는 불상사가!!
    - 부작용 : 수정했는데 전혀 예상치 못한 곳에서 문제가 발생하는 것
        - 방지 : 값을 복사해서 사용
- 값 타입 복사
    - 값 타입의 실제 인스턴스인 값을 공유하는 것은 위험
    - 예) 별도의 clone 메소드 생성
        - 자신을 복사해서 반환
        - 영속성 컨텍스트는 나중에 변경할 엔티티에만 변경된 것으로 판단하고 UPDATE SQL을 실행
        - 문제 : 임베디드 타입처럼 직접 정의한 값 타입은 자바의 기본타입이 아니라 객체 타입
            - 자바는 기본 타입에 값을 대입하면 값을 복사해서 전달
            - 기본타입은 각각의 값이 독립된 값이고 부작용도 없다
            - 객체타입은 항상 참조값을 전달
            - 복사하지 않고 원본의 참조값을 직접 넘기는 것을 막을 방법이 없다!!!
                - 자바 : 기본타입이면 복사해서 넘기고 객체면 참조를 넘길 뿐
            - **중요 : 객체의 공유 참조는 피할 수 없다**
                - 객체의 값을 수정하지 못하게 막자
                - 예) 수정 메소드 제거
- 불변 객체
    - 객체를 불변하게 만들면 값을 수정할 수 없으므로 부작용을 원천 차단할 수 있다.
    - 따라서 값 타입은 될 수 있으면 불변 객체로 설계해야
    - 다양한 구현 방법
        - 가장 간단한 방법 : 생성자로만 값을 설정, 수정자는 만들지 않음
    - Integer, String은 자바가 제공하는 대표적인 불변 객체
    - 불변이라는 작은 제약으로 부작용이라는 큰 재앙을 막을 수 있다.

## 4. 값 타입의 비교

- 자바가 제공하는 객체 비교
    - 동일성 비교 : 인스턴스의 참조 값을 비교, ==
    - 동등성 비교 : 인스턴스의 값을 비교, equals()
    - 값 타입의 equals 메소드를 재정의할 때는 보통 모든 필드의 값을 비교하도록 구현
- 자바에서 equals()를 재정의하면
    - hashCode()도 재정의하는 것이 안전
    - 그렇지 않으면 해시를 사용하는 컬렉션(HashSet, HashMap)이 정상 동작하지 않는다
    - 자바 IDE에는 대부분 이 두개의 메소드를 자동으로 생성해주는 기능이 있다.

## 5. 값 타입 컬렉션

- 값 타입을 하나 이상 저장하려면 컬렉션에 보관하고
- @ElementCollection, @CollectionTable 어노테이션을 사용하면 됨
- 예) favoriteFoods : 기본값 타입인 String을 컬렉션으로 가진다
    - 데이터베이스 테이블로 매핑해야 하는데
    - 관계형 데이터베이스의 테이블은 컬럼안에 컬렉션을 포함할 수 없다
    - 별도의 테이블을 추가하고 @CollectionTable를 사용해서 추가한 테이블을 매핑해야 한다
    - 값으로 사용되는 컬럼이 하나면 @Column을 사용해서 컬럼명을 지정할 수 있다
- 테이블 매핑정보는 @AttributeOverride를 사용해서 재정의할 수 있다
- @CollectionTable를 생략하면
    - 기본값을 사용해서 매핑한다
    - 기본값 : {엔티티이름}_{컬렉션 속성 이름}
    - 예) Member 엔티티의 addressHistory : Member_addressHistory 테이블과 매핑
- 값 타입 컬렉션 사용
    - 멤버를 생성하고, 임베디드 값 타입을 넣고, 기본값 타입 컬렉션을 넣고, 임베디드 값 타입 컬렉션을 넣은 상태에서...
    - 마지막에 member 엔티티만 영속화
        - 이때 member 엔티티의 값타입도 함께 저장
        - 예) 임베디드 값 타입은 회원테이블을 저장하는 SQL에 포함
    - 값 타입 컬렉션은 영속성 전이(p.307) + 고아 객체(p.311) 제거 기능을 필수로 가진다고 볼 수 있다
    - 값 타입 컬렉션도 조회할 때 페치 전략을 선택할 수 있다 (기본 : LAZY)
    - 예) 조회
        - 회원(member)만 조회, 임베디드 값 타입인 homeAddress도 함께
        - 컬렉션은 사용할 때(LAZY) 조회 쿼리 호출
    - 예) 수정
        - 임베디드 값 타입 수정은 엔티티를 수정하는 것과 같다
        - 기본값 타입 컬렉션 수정 : 자바의 String 타입은 수정할 수 없다. 제거하고 추가
        - 임베디드 값 타입 컬렉션 수정
            - 값 타입은 불변해야
            - 컬렉션에서 기존 주소를 삭제하고 새주소를 등록
            - 값 타입은 equals, hashCode를 꼭 구현해야
- 값 타입 컬렉션의 제약사항
    - 엔티티는 식별자가 있으므로
        - 엔티티의 값을 변경해도 식별자로 데이터베이스에 저장된 원본 데이터를 쉽게 찾아서 변경할 수 있다
    - 값 타입은 식별자라는 개념이 없고 단순한 값들의 모음
        - 값을 변경해버리면 데이터베이스에 저장된 원본 데이터를 찾기는 어렵다.
    - 특정 엔티티 하나에 소속된 값 타입은 값이 변경되어도
        - 자신이 소속된 엔티티를 데이터베이스에서 찾고 값을 변경하면 된다
    - 문제는 값 타입 컬렉션
        - 보관된  값 타입들은 별도의 테이블에 보관된다
        - 여기에 보관된 값 타입의 값이 변경되면 데이터베이스에 있는 원본 데이터를 찾기 어렵다
    - JPA 구현체
        - 값 타입 컬렉션에 변경사항이 발생하면
        - 값 타입 컬렉션이 매핑된 테이블의 연관된 모든 데이터를 삭제
        - 현재 값 타입 컬렉션 객체에 있는 모든 값을 데이터베이스에 다시 저장
        - 예) 식별자가 100번인 회원이 관리하는 주소 값 타입 컬렉션을 변경할 때
    - 실무라면...
        - 값 타입 컬렉션이 매핑된 테이블에 데이터가 많다면
        - 값 타입 컬렉션 대신 일대다 관계를 고려해야...
    - 값 타입 컬렉션을 매핑하는 테이블은 모든 컬럼을 묶어서 기본 키를 구성해야 한다
        - 데이터베이스 기본 키 제약 조건으로 인해 컬럼에 null을 입력할 수 없고
        - 같은 값을 중복해서 저장할 수 없는 제약도 있다.
    - 위 이슈를 해결하려면
        - 값 타입 컬렉션을 사용하는 대신에 새로운 엔티티를 만들어서 일대다 관계로 설정하면 된다.
        - 추가로 영속성 전이 + 고아 객체 제거 기능을 적용하면 값 타입 컬렉션처럼 사용할 수 있다.
    - 값 타입 컬렉션을 변경했을 때
        - JPA 구현체들은 테이블의 기본 키를 식별해서 변경된 내용만 반영하려고 노력
        - BUT 사용하는 컬렉션이나 여러 조건에 따라 기본키를 식별할 수도, 못할 수도 있다
        - 따라서 모두 삭제하고 다시 저장하는 최악의 시나리오를 고려하면서 사용해야
        - 값 타입 컬렉션의 최적화에 관한 내용은 각 구현체의 셜명서를 참고할 것

## 6. 정리

- 엔티티 타입의 특징
    - 식별자가 있다 (@id)
        - 엔티티 타입은 식별자가 있고 식별자로 구별할 수 있다
    - 생명 주기가 있다
        - 생성하고 영속화하고 소멸하는 생명 주기가 있다
        - em.persist(entity)로 영속화
        - em.remove(entity)로 제거
    - 공유할 수 있다
        - 참조 값을 공유할 수 있다. (공유 참조)
        - 예) 회원 엔티티가 있다면 다른 엔티티에서 얼마든지 회원 엔티티를 참조할 수 있다.
- 값 타입의 특징
    - 식별자가 없다
    - 생명 주기를 엔티티에 의존한다
        - 엔티티 제거시 같이 제거됨
    - 공유하지 않는 것이 안전하다
        - 값을 복사해서 사용해야
        - 오직 하나의 주인만이 관리해야
        - 불변 객체로 만드는 것이 안전
- 값 타입은 정말 값 타입이라 판단될 때만 사용해야
    - 엔티티를 값 타입으로 만들면 안된다
    - 식별자가 필요하고 지속해서 값을 추적하고 구분하고 변경해야 한다면 그건 엔티티
