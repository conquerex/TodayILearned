# 15장. 고급 주제와 성능 최적화

## 입구

JPA의 깊이 있는 고급 주제들과 JPA의 성능을 최적화하는 다양한 방안을 알아보자

- 예외처리 : JPA를 사용할 때 발생하는 다양한 예외와 예외에 따른 주의점을 설명
- 엔티티 비교 : 엔티티를 비교할 때 주의점과 해결방법 설명
- 프록시 심화 주제 : 프록시로 인해 발생하는 다양한 문제점과 해결방법을 다룬다
- 성능 최적화
    - N+1 문제
    - 읽기 전용 쿼리의 성능 최적화
    - 배치 처리
    - SQL 쿼리 힌트 사용
    - 트랜잭션을 지원하는 쓰기 지연과 성능 최적화

## 1. 예외 처리

- `JPA 표준 예외`들은 javax.persistence.PersistenceException 의 자식 클래스이다.
    - PersistenceException은 RuntileException의 자식이다.
    - JPA 예외는 모두 언체크 예외다.
- JPA 표준 예외 2가지 분류
    - 트랜잭션 롤백을 표시하는 예외
        - 심각한 예외이므로 복구해서는 안된다.
        - 이 예외가 발생하면 트랜잭션을 강제로 커밋해도 트랜잭션이 커밋되지 않고
        - 대신에 javax.persistence.RollbackException 예외가 발생한다.
        - javax.persistence.EntityExistsException
            - EntityManager.persists() 호출 시 이미 같은 엔티티가 있으면 발생
        - 그 외 p.641 표 참조
    - 트랜잭션 롤백을 표시하지 않는 예외
        - 심각한 예외가 아니다.
        - 개발자가 트랜잭션을 커밋할지 롤백할지를 판단
        - javax.persistence.NoResultException
            - Query.getSingleResult() 호출 시 결과가 하나도 없을 때 발생
        - p.641 표 참조
- 서비스 계층에서 데이터 접근 계층의 구현 기술에 직접 의존하는 것은 좋은 설계라 할 수 없다.
    - 예를 들어 서비스 계층에서 JPA의 예외를 직접 사용하면 JPA에 의존하게 된다.
    - 스프링 프레임워크는 이런 문제를 해결하려고 데이터 접근 계층에 대한 예외를 추상화해서 개발자에게 제공한다.
    - JPA 예외를 스프링 예외로 변경
        - p.642, 표 15.3
- 스프링 프레임워크에 JPA 예외 변환기 적용
    - JPA 예외를 스프링 프레임워크가 제공하는 추상화된 예외로 변경하려면
        - PersistenceExceptionTranslationPostProcessor를 스프링 빈으로 등록
        - @Repository 어노테이션을 사용한 곳에 예외 변환 AOP를 적용하는 방법
    - 예외를 변환하지 않고 그대로 반환하는 방법
        - throws 절에 반환할 JPA 예외나 JPA 예외의 부모 클래스를 직접 명시

    ```java
    @Repository
    public class NoResultExceptionTestService {
    	@PersistenceContext EntityManager em;

        public member findMember **throws javax.persistence.NoResultException** {
        	return em.createQuery("select ...").getSingleResult();
        }
    }
    ```


- 트랜잭션 롤백시 주의사항
    - 트랜잭션 롤백시 데이터베이스의 반영사항만 롤백하는 것이지 수정한 자바 객체까지 원상태로 복구해주지 않는다.
        - 따라서 트랜잭션이 롤백된 영속성 컨텍스트를 그대로 사용하는 것은 위험하다.
    - 새로운 영속성 컨텍스트를 생성해서 사용하거나 EntityManager.clear() 를 호출해서 영속성 컨텍스트를 초기화 후 사용해야 한다.
    - 기본 전략인 트랜잭션당 영속성 컨텍스트 전략은 문제 발생 시 트랜잭션 AOP 종료 시점에 트랜잭션을 롤백, 영속성 컨텍스트도 함께 종료 → 문제 발생 X
    - 문제는 여러 트랜잭션이 하나의 영속성 컨텍스트를 사용할 때 발생.
        - 예를 들면, 영속성 컨텍스트의 범위를 트랜잭션 범위보다 넓게 사용하는 OSIV 같은 경우
        - 트랜잭선 롤백 시 영속성 컨텍스트에 이상이 발생해도 다른 트랜잭션에서 해당 영속성 컨텍스트를 그대로 사용하는 문제 발생
            - 이 경우 트랜잭션 롤백 시 영속성 컨텍스트를 초기화`em.clear()` 를 해서 잘못된 영속성 컨텍스트 사용을 예방한다.
        - 더 자세한 내용은 org.springframework.orm.jpa.JpaTransactionManger의 doRollback() 메소드 참고

## 15.2 엔티티 비교

- 영속성 컨텍스트 내부에는 엔티티 인스턴스를 보관하기 위한 1차캐시가 있다.
    - 이 1차 캐시는 영속성 컨텍스트와 생명주기를 같이 한다.
- 영속성 컨텍스트를 통해 데이터를 저장하거나 조회하면 1차캐시에 엔티티가 저장.
    - 이 1차캐시 덕분에 변경 감지 기능도 동작하고, 이름 그대로 1차 캐시로 사용되어서 데이터베이스를 통하지 않고 데이터를 바로 조회할 수 있다.
- 1차캐시의 가장 큰 장점
    - 애플리케이션 수준의 반복 가능한 읽기
    - 같은 영속성 컨텍스트에서 엔티티를 조회하면 다음 코드와 같이 항상 같은 엔티티 인스턴스를 반환한다.
    - 단순히 동등성 비교 수준이 아니라 정밀 주소값이 같은 인스턴스를 반환한다.
- 영속성 컨텍스트가 같을 때 엔티티 비교
    - 테스트 클래스에 @Transactional 이 선언되어 있으면
        - 트랜잭션을 먼저 시작하고 테스트 메소드를 실행
    - 영속성 컨텍스트가 같으면 엔티티를 비교할 때 다음 3가지 조건을 만족
        - 동일성(identical): == 비교가 같다.
        - 동등성(equinalent): equals() 비교가 같다.
        - 데이터베이스 동등성: @Id인 데이터베이스 식별자가 같다.
    - 참고
        - 테스트 클래스에 @Transactional을 적용하면 테스트가 끝날 때 트랜잭션을 커밋하지 않고 트랜잭션을 강제로 롤백한다.
        - 어떤 SQL이 실행되는지 콘솔을 통해 보고 싶으면 마지막에 em.flush()를 강제로 호출
- 영속성 컨텍스트가 다를 때 엔티티 비교
    - 서로 다른 트랜잭션 범위를 가지기 때문에, 서로 다른 영속성 컨텍스트에 접근 (p.651, 그림 15.4)
        - `동일성 비교가 실패`
        - 동등성, DB 동등성은 성공
    - 예제 설명
        - 테스트 코드에서 회원가입을 시도하면 서비스 계층에서 트랜잭션이 시작, 영속성 컨텍스트가 만들어진다.
        - 레포지토리에서 엔티티를 영속화하고 서비스 계층이 끝날 때 트랜잭션이 커밋, 영속성 컨텍스트가 플러시 된다
            - 이때 트랜잭션과 영속성 컨텍스트가 종료
            - 엔티티 인스턴스는 준영속 상태
        - 새로운 메소드르르 호출해서 저장한 엔티티를 조회하면 리포지토리 계층에서 새로운 트랜잭션이 시작, 새로운 영속성 컨텍스트가 생성
        - 저장한 엔티티를 조회하지만 새로 생성된 영속성 컨텍스트에는 찾는 데이터가 존재하지 않는다
        - 데이터베이스에서 데이터를 찾아온다
        - 테이터베이스에서 조회된 회원 엔티티를 영속성 컨텍스트에 보관하고 반환
        - 마지막 메소드가 끝나면서 트랜잭션 종료, 새 영속성 컨텍스트도 종료
    - 영속성 컨텍스트가 다를 때 엔티티 비교
        - 동일성 : `==` 비교가 실패
        - 동등성 : `equals()` 비교가 만족. 단, `equals()`를 구현해야 한다. 보통 비즈니스 키로 구현. 혹은 유일성이 보장된 대상으로 비교.
        - 데이터베이스 동등성 : `@Id` 인 식별자가 같다.
    - `동일성 비교`는 같은 영속성 컨텍스트의 관리를 받는 영속 상태의 엔티티에만 적용할 수 있다.
        - 그렇지 않을 때는 `비즈니스 키`를 사용한 `동등성 비교`를 해야 한다. (DB의 PK, 주민 번호 등등)

## 3. 프록시 심화 주제

`프록시`는 원본 엔티티를 상속받아서 만들어지므로 엔티티를 사용하는 클라이언트는 엔티티가 프록시인지 아니면 원본 엔티티인지 구분하지 않고 사용할 수 있다.

이로 인해 예상하지 못하는 문제들이 발생하기도 한다.

- 영속성 컨텍스트와 프록시
    - 영속성 컨텍스트는 자신이 관리하는 영속 엔티티의 동일성을 보장한다.
    - **프록시는 원본 엔티티를 상속받아서 만들어지므로** 엔티티를 사용하는 클라이언트는 엔티티가 프록시인지 원본 엔티티인지 구분하지 않고 사용할 수 있다.

    ```java
    // 프록시 객체 조회 : em.find() 의 결과는 member1 의 프록시 객체
    Member refMember = em.getReference(Member.class, "member1");
    Member findMember = em.find(Member.class, "member1");
    ```

    ```java
    // 원본 엔티티 조회 : em.getReference() 의 결과는 member의 엔티티
    Member findMember = em.find(Member.class, "member1");
    Member refMember = em.getReference(Member.class, "member1");
    ```

    - 둘 다 프록시 조회
        - 프록시로 조회해도 영속성 컨텍스트를 영속 엔티티의 동일성을 보장한다 !
    - 둘 다 원본 조회
        - 이미 원본 데이터를 가져왔으니 원본을 반환한다.
    - 영속성 컨텍스트에 원본 엔티티이든, 프록시이든 먼저 조회해 놓은 것을 이후에도 반환
    - 즉, 영속성 컨텍스트는 자신이 관리하는 영속 엔티티의 동일성 보장 !
- 프록시 타입 비교
    - 프록시는 원본 엔티티를 상속받아서 만들어지므로
        - 프록시로 조회한 엔티티의 타입을 비교할 때는 `==` 비교 대신 `instanceof` 를 사용해야 한다.

    ```java
    Member refMember = em.getReference(Member.class, "member1");
    Assert.assertFalse(Member.class == refMember.getClass()); // false
    Assert.assertTrue(refMember instanceof Member); // true
    ```

    - `refMember.getClass()` 결과는 `class jpabook.advanced.Member_$$_jvsteXXX`이다. `_$$_jvsteXXX` 가 붙어 있으면 프록시라는 의미이다.
    - `Member.class == refMember.getClass()`는 부모 클래스와 자식 클래스를 == 비교한 셈이다.
    - 프록시는 원본 엔티티의 자식 타입이므로 `instanceof` 연산을 사용해야 한다.
- 프록시 동등성 비교
    - 엔티티의 동등성 비교 시 비즈니스 키를 사용해 `equals()` 메소드를 오버라이딩하고 비교한다.
    - 이때, **비교 대상이 원본 엔티티면 문제가 없지만, 프록시면 문제가 발생할 수** 있다.
    - 프록시의 동등성 비교 시 주의 사항
        1. 프록시의 타입 비교는 == 비교 대신에 `instanceof` 를 사용해야 한다.
        2. 프록시의 멤버 변수에 직접 접근하면 null 이 반환된다. 따라서 대신에 접근자 메소드(getter)를 사용해야 한다.
            - 프록시는 실제 데이터를 갖고 있지 않다. 따라서 프록시의 멤버 변수에 직접 접근하면 아무 값도 조회되지 않는다.

    ```java
    @Entity
    public class Member {
    	@Id
    	private String id;
    	private String name;

    	...

    	@Override
    	public boolean equals(Object obj) {
    		if (this == obj) return true;
    		if (obj == null) return false;
    		**if (this.getClass() != obj.getClass()) return false; // 1**

    		Member member = (Member) obj;

    		**if (this.name != null ? !this.name.equals(member.name) : member.name != null) // 2**
    			return false;

    		return true;
    	}
    }
    ```

    - 엔티티는 name 필드를 비즈니스 키로 사용해서 equals 메소드를 오버라이딩 했다

    ```java
    @Test
    public void 프록시와_동등성비교() {
    		Member saveMember = new Member("member1", "회원1");
    		em.persist(saveMember);
    		em.flush();
    		em.clear();

    		Member newMember = new Member("member1", "회원1");
    		Member refMember = em.getReference(Member.class, "member1");

    		Assert.assertTrue(**newMember.equals(refMember)**); // false
    }
    ```

    - 두 인스턴스가 다른 이유는?!
        1. 프록시인 케이스는 비교할때, `==` 가 아닌 `instanceOf` 를 사용해야한다.

            ```java
            if (!(obj instanceof Member)) return false; // 1
            ```

        2. 프록시인 케이스는 필드 접근 시 직접 접근하면 안 된다. (`null` 이다)
            - 따라서, `Getter 접근자`를 사용해야한다.

            ```java
            if (name == null ? !name.equals(member.getName()) : member.getName() != null) // 2
            ```

- 상속관계와 프록시
    - 프록시를 부모 타입으로 조회하면 문제가 발생한다.

    ```java
    @Test
    public void 부모타입으로_프록시조회(){

        Book saveBook = new Book;
        saveBook.setName("jpaBook");
        saveBook.setAuthor("kim");
        em.persist(saveBook);

        em.flush();
        em.clear();

        Item proxyItem = em.getReference(Item.class, saveBook.getId());

        if(proxyItem instanceof Book){
            System.out.println("proxyItem instanceof Book");
            Book book = (Book) proxyItem; //java.lang.ClassCastException
            System.out.println("책 저자 =" + book.getAuthor);
        }

    		 // 결과 검증
         Assert.assertFalse(proxyItem.getClass == Book.class);
         Assert.assertFalse(proxyItem instanceof Book); //false
         Assert.assertTrue(proxyItem instanceof Item);
    }
    ```

    - 결과 : 책 저자가 출력 되어야 하는데, 출력 되지 않음
    - `em.getReference()` 메서드를 통해서 Item을 조회했으므로 Item 엔티티를 프록시로 조회함
        - 이 프록시 클래스는 원본 엔티티로 Book 엔티티를 참조
    - 하지만, `if(proxyItem instanceof Book)` 부분에서 원인이 있다.
        - `proxyItem` 은 Item$Proxy 타입이고, 이 타입은 `Book` 과는 전혀 관계가 없다.
        - 따라서, 직접 다운 캐스팅을 해도 문제가 발생한다. (관련이 없기 때문)
    - 정리하면...
        - instanceOf 연산을 사용할 수 없다.
        - 하위 타입으로 다운캐스팅을 할 수 없다.
    - 프록시를 부모 타입으로 조회하는 문제는 주로 다형성을 다루는 도메인 모델에서 나타난다.
        - p.663~664 : 예제 참고

        ```java
        // OrderItem 에서 Item을 지연로딩으로 설정한 상황에서 아래 테스트 진행
        Book book = new Book("jpabook", "kim");
        em.persist(book);
        em.flush(); em.clear();

        OrderItem saveOrderItem = new OrderItem();
        saveOrderItem.setItem(book);
        em.persist(saveOrderItem);
        em.flush(); em.clear();

        OrderItem orderItem = em.find(OrderItem.class, saveOrderItem.getId());
        Item item = orderItem.getItem();
        Assert.assertFalse(item.getClass() == Book.class); // item은 지연 로딩으로 조회해서 프록시이다.
        Assert.assertFalse(item instanceof Book);
        Assert.assertTrue(item instanceof Item);
        ```

    - 이를 해결하기 위한 방법이 몇가지 있다.
    1. JPQL로 대상 직접 조회로 해결
        1. **가장 간단한 방법** : 처음부터 자식 타입을 직접 조회해서 필요한 연산 수행하기
        2. 대신 다형성을 활용할 수 없게 된다.

        ```java
        Book jpqlBook =
        	em.createQuery("select b from Book b where b.id=:bookId", Book.class)
            .setParameter("bookId", item.getId())
            .getSingleResult();
        ```

    2. 프록시 벗기기
        1. 하이버네이트가 제공하는 기능을 사용하면 프록시에서 원본 엔티티를 가져올 수 있다.
        2. 앞서 설명했듯이 **영속성 컨텍스트는 한 번 프록시로 노출한 엔티티는 계속 프록시로 노출한다.**
            - 영속성 컨텍스트가 영속 엔티티의 동일성을 보장함으로써, 클라이언트는 조회한 엔티티가 프록시인지 아닌지 구분하지 않고 사용할 수 있다.
        3. 하지만 이 방법은 프록시에서 원본 엔티티를 직접 꺼내기 때문에 프록시와 원본 엔티티의 동일성 비교가 실패한다는 문제점이 있다.

            ```java
            // unProxyItem 은 Book 엔티티인데, 타입이 Item으로 되어있음.
            item == unProxyItem은 false
            ```

        4. 이 방법을 사용할 때는 **원본 엔티티가 꼭 필요한 곳에서 잠깐 사용하고 다른 곳에서 사용되지 않도록 하는 것이 중요**하다.
        5. 참고로 원본 엔티티의 값을 직접 변경해도 변경 감지 기능 동작함.
    3. 기능을 위한 별도의 인터페이스 제공
        1.  `Item`의 구현체에 따라 각각 다른 `getTitle()` 메소드가 호출된다.

            ```java
            @Entity
            public class OrderItem {
                @Id @GeneratedValue
                private Long id;
                @ManyToOne(fetch = FetchType.LAZY)
                @JoinColumn(name = "ITEM_ID")
                private Item item;
                ...
                public void printItem() {
                    System.out.println("TITLE=" + item.getTitle());
                }
            }
            OrderItem orderItem = em.find(OrderItem.class, saveOrderItem.getId());
            orderItem.printItem();
            ```

        2. 인터페이스를 제공하고, 각각의 클래스가 자신에 맞는 기능을 구현하는 것
            1. 다형성을 활용하는 좋은 방법
        3. 이후 다양한 상품 타입이 추가되어도 `item` 을 사용하는 `OrderItem` 의 코드는 수정하지 않아도 된다.
        4. 이 방법은 클라이언트 입장에서 대상 객체가 프록시인지 아닌지를 고민하지 않아도 되는 장점이 있다.
            1. 이 방법을 사용할 때는 프록시의 특징 때문에 프록시의 대상이 되는 타입에 인터페이스를 적용해야 한다.
    4. 비지터 패턴 사용
        1. 비지터 패턴은 Visitor와 Visitor를 받아들이는 대상 클래스로 구성
            1. Item이 accept(visitor) 메소드를 사용해서 Visitor를 받아들인다.
            2. 그리고 Item은 단순히 Visitor을 받아들이기만 하고 실제 로직은 Visitor가 처리한다.
            3. Visitor에는 visit()라는 메소드를 정의하고 모든 대상 클래스를 받아들이도록 작성하면 된다.
            4. 여기서는 Book, Album, Movie를 대상 클래스로 사용한다.
        2. 비지터 구현
        3. 대상 클래스 작성
            1. `Item`에 `Visitor`를 받아들일 수 있도록 `accept(visitor)` 메소드를 추가한다.
            2. 각각의 자식 클래스들은 부모에 정의한 accept(visitor) 메소드를 구현
            3. 구현 내용은 파라미터로 넘어온 Visitor의 visit(this) 메소드를 호출하면서 자신을 파라미터로 넘기는 것이 전부다. 이렇게 실제 로직 처리를 visitor에 위임한다.
        4. 비지터 패턴 실행
            1. `item.accept()` 호출하면서 파라미터로 `PrintVisitor` 를 넘겨줌
            2. `item`은 프록시이므로 먼저 프록시가 `accept()` 메소드를 받고 원본 엔티티의 accept()를 실행한다.
            3. 원본 엔티티는 다음 코드를 실행해서 자신을 visitor에 파라미터로 넘겨준다.
        5. 비지터 패턴과 확장성
            1. 비지터 패턴은 새로운 기능이 필요할 때 Visitor만 추가하면 된다.
            2. 따라서 기존 코드의 구조를 변경하지 않고 기능을 추가할 수 있는 장점이 있다
        6. 비지터 패턴의 장점
            1. 프록시에 대한 걱정 없이 안전하게 원본 엔티티에 접근
            2. `instanceof` 나 타입 캐스팅 없이 코드를 구현가능
            3. 알고리즘과 객체 구조를 분리해서 구조를 수정하지 않고 새로운  동작을 추가
        7. 비지터 패턴의 단점
            1. 너무 복잡하고 더블 디스패치를 사용하기 때문에 이해하기 어렵다
            2. 객체 구조가 변경되면 모든 `visitor`를 수정해야 한다.

## 4. 성능 최적화

- N+1 문제
    - JPA로 애플리케이션을 개발할 때 성능상 가장 주의해야 하는 문제
    - 즉시 로딩과 N+1 (p.676)
        - `em.find()` 메소드로 조회하면 즉시 로딩으로 설정한 `Order`도 함께 조회해서 문제가 없다
            - 하지만, `JPQL`을 사용할 때 문제가 발생
        - JPQL 사용 시 JPA가 이를 분석해 SQL을 사용한다.
            - 이때는 즉시 로딩과 지연 로딩에 대해 전혀 신경쓰지 않고 JPQL만 사용해서 SQL을 생성한다.
            - 먼저 `SELECT * FROM MEMBER` 실행
            - 이후 `SELECT * FROM ORDERS WHERE MEMBER_ID=1`
            - 회원과 연관된 주문 엔티티 조회를 위한 N개 SQL 발생

            ```java
            List<Member> members =
            	em.createQuery("select m from Member m", Member.class)
            	.getResultList();

            // JPQL로 Member만 조회하는 경우
            SELECT * FROM MEMNER
            SELECT * ORDERS WHERE MEMBER_ID =1 // 회원과 연관된 주문
            SELECT * ORDERS WHERE MEMBER_ID =2 // 회원과 연관된 주문
            SELECT * ORDERS WHERE MEMBER_ID =3 // 회원과 연관된 주문
            SELECT * ORDERS WHERE MEMBER_ID =4 // 회원과 연관된 주문
            SELECT * ORDERS WHERE MEMBER_ID =5 // 회원과 연관된 주문
            SELECT * ORDERS WHERE MEMBER_ID =6 // 회원과 연관된 주문
            ```

    - 지연 로딩과 N+1
        - 1개의 Order에만 접근하는 경우는 문제 없다.
        - 하지만, 모든 회원에 대해 연관된 Order에 접근하는 경우 N+1 문제가 발생한다.

            ```java
            for (Member member : members) {
            	//지연 로딩 초기화
            	System.out.println("member = " + member.getOrders().size();
            }

            SELECT * ORDERS WHERE MEMBER_ID =1 // 회원과 연관된 주문
            SELECT * ORDERS WHERE MEMBER_ID =2 // 회원과 연관된 주문
            SELECT * ORDERS WHERE MEMBER_ID =3 // 회원과 연관된 주문
            SELECT * ORDERS WHERE MEMBER_ID =4 // 회원과 연관된 주문
            SELECT * ORDERS WHERE MEMBER_ID =5 // 회원과 연관된 주문
            ```

        - 지연 로딩으로 조회한 컬렉션을 사용할 시점에 N+1 문제가 발생
    - 해결법💡 : 페치 조인 사용
        - 가장 일반적인 해결방법.
        - 연관된 엔티티를 함께 조회하므로 N+1 문제가 발생하지 않는다.

        ```java
        select distinct m from Member m join fetch m.orders // 중복 결과를 제거하기 위해 distinct 키워드 사용
        // 실행된 SQL
        SELECT M.*, O.* FROM MEMBER M
        INNER JOIN ORDERS O ON M.ID=O.MEMBER_ID
        ```

        - 일대다 조인을 했으므로 결과가 늘어나서 중복된 결과가 나타날 수 있다.
        - 따라서 JPQL의 DISTINCT를 사용해서 중복을 제거하는 것이 좋다.
    - 해결법💡 : 하이버네이트 @BatchSize
        - 하이버네이트가 제공하는 org.hibernate.annotations.BatchSize 어노테이션을 사용하면 연관된 엔티티를 조회할 때 지정한 size만큼 SQL의 IN 절을 사용해서 조회한다.
        - 즉시 로딩으로 설정하면 조회 시점에 10건의 데이터를 모두 조해해야 하므로 다음 SQL이 두 번 실행된다.
        - 지연 로딩으로 설정하면 지연 로딩된 엔티티를 최초 사용하는 시점에 다음 SQL을 실행해서 5건의 데이터를 미리 로딩해둔다. 그리고 6번째 데이터를 사용하면 IN 절로 4개 데이터를 조회한다.
    - 해결법💡 : 하이버네이트 @Fetch(FetchMode.SUBSELECT)
        - 연관된 데이터를 조회할 때 서브 쿼리를 사용해서 N+1 문제를 해결한다.
        - 마찬가지로 지연 로딩 시 연관된 엔티티 조회 시점에 SQL 실행

        ```java
        @org.hibernate.annotaions.Fetch(FetchMode.SUBSELECT)
        @OneToMany(mappedBy = "member", fetch = FetchType.EAGER)
        private List<Order> orders = new ArrayList<Order>();

        // 수행 쿼리
        SELECT O FROM ORDERS O
        	WHERE O.MEMBER_ID IN (
        		SELECT
        			M.ID
        		FROM
        			MEMBER M
        		WHERE M.ID > 10
        	}
        ```

    - 정리
        - 즉시 로딩과 지연 로딩 중 추천하는 방법은 **즉시 로딩은 사용하지 말고 지연 로딩만 사용하는 것**이다.
        - **즉시 로딩**은 N+1 문제 + 비즈니스 로직에 따라 불필요한 엔티티도 로딩해야하는 상황 발생
        - 즉시 로딩의 가장 큰 문제는 성능 최적화가 어렵다는 점
        - 엔티티를 조회하다보면 즉시 로딩이 연속으로 발생해서 전혀 예상하지 못한 SQL이 실행될 수 있다.
            - 따라서 모두 지연 로딩으로 설정하고 성능 최적화가 꼭 필요한 곳에는 JPQL 페치 조인을 사용
- JPA의 글로벌 페치 전략 기본값
    - `@OneToOne`, `ManyToOne` : 기본 페치 전략은 즉시 로딩
    - `@OneToMany`, `@ManyToMany` : 기본 페치 전략은 지연 로딩
    - 읽기 전용 쿼리의 성능 최적화
        - 엔티티가 영속성 컨텍스트에서 관리되면 1차 캐시부터 변경 감지까지 얻을 수 있는 혜택이 많다
        - 하지만 변경 감지를 위해 스냅샷 인스턴스를 보관하므로 더 많은 메모리를 사용하는 단점이 있다.
        - 만약 한 번만 읽어서 화면에 출력하고, 다시 조회할 일이 없다면 **읽기 전용으로 엔티티를 조회하면 메모리 사용량을 최적화할 수 있다.**
        - 아래의 방법들은 `select o from Order o` 를 최적화한다.
        1. 스칼라 타입으로 조회
            1. 엔티티가 아닌 스칼라 타입으로 모든 필드를 조회 -> 영속성 컨텍스트가 스칼라 타입은 관리하지 않는다.

                `select o.id, o.name, o.price from Order p`

        2. 읽기 전용 쿼리 힌트 사용
            1. 하이버네이트 전용 힌트인 `org.hibernate.readOnly` 를 사용하면 엔티티를 읽기 전용으로 조회할 수 있다.
            2. 읽기 전용이므로 영속성 컨텍스트는 스냅샷을 보관하지 않는다. => **메모리 사용량 최적화 가능**
            3. 단 스냅샷이 없으므로 엔티티를 수정해도 데이터베이스에 반영되지 않는다.

                ```
                TypedQuery<Order> query = em.createQuery("select o from Order o", Order.class);
                query.setHint("org.hibernate.readOnly", true);

                ```

        3. 읽기 전용 트랜잭션 사용
            1. 스프링 프레임워크 사용 시 트랜잭션을 읽기 전용 모드로 설정 가능
            2. `@Transactional(readOnly = true)`
            3. 스프링 프레임워크가 `하이버네이트 세션`의 플러시 모드를 `MANUAL` 로 설정한다. 강제로 플러시 호출을 하지 않는 한 플러시가 발생하지 않는다.
            4. 트랜잭션을 시작했으므로 트랜잭션 시작, 로직 수행, 트랜잭션 커밋의 과정은 이루어지지만, **트랜잭션을 커밋해도 영속성 컨텍스트를 플러시하지는 않는다.** => 엔티티 등록, 수정, 삭제 동작 X
            5. **플러시 호출 시 스냅샷 비교와 같은 무거운 로직들을 수행하지 않아 성능이 향상된다.**
        4. 트랜잭션 밖에서 읽기
            1. 트랜잭션 없이 엔티티를 조회한다. (조회가 목적이라서 가능. JPA에서 데이터 변경하려면 트랜잭션 필수)
            2. 스프링 프레임워크 사용 시 다음과 같이 설정
            3. `@Transactional(propagation = Propagation.NOT_SUPPORTED)`
            4. 마찬가지로 플러시가 발생하지 않아 조회 성능이 향상된다. 이때 트랜잭션 자체가 없어서 커밋도 발생 X. JPQL 쿼리도 트랜잭션 없이 실행하면 플러시를 호출하지 않는다.
        - 읽기 전용 조회 시 최적화 방법 정리
            - **메모리 최적화**
                - 스칼라 타입 조회
                - 하이버네이트가 제공하는 읽기 전용 쿼리 힌트
            - **속도 최적화** : 플러시 호출 막기
                - 읽기 전용 트랜잭션 사용 (스프링 프레임워크 사용 시 편리한 방법)
                - 트랜잭션 밖에서 읽기

                ```
                @Transactional(readOnly = true) // 읽기 전용 트랜잭션
                public List<DataEntity> findDatas(){
                return em.createQuery("select d from DataEntity d", DataEntity.class)
                    .setHint("org.hibernate.readOnly", true) // 읽기 전용 쿼리 힌트
                    .getResultList();
                }
                ```

                - 읽기 전용 트랜잭션 사용 : 플러시를 작동하지 않도록 해서 성능 향상
                - 읽기 전용 엔티티 사용 : 엔티티를 읽기 전용으로 조회해서 메모리 절약
- **배치 처리**
    - 수백만 건의 데이터를 배치 처리해야 하는 상황 가정
    - 일반적인 방식으로 엔티티를 계속 조회 시 영속성 컨텍스트에 아주 많은 엔티티가 쌓이면서 **메모리 부족 오류**가 발생
    - 따라서 이런 배치 처리는 **적절한 단위로 영속성 컨텍스트의 엔티티를 데이터베이스에 플러시하고 초기화**해준다. 만약 2차 캐시를 사용하고 있다면 **2차 캐시에 엔티티를 보관하지 않도록 주의**해야 한다.
    - JPA 등록 배치
        - 수천 수만건 이상의 엔티티를 한 번에 등록할 때 주의할 점은 영속성 컨텍스트에 엔티티가 계속 쌓이지 않도록 일정 단위마다 영속성 컨텍스트의 엔티티를 데이터베이스에 플러시하고 영속성 컨텍스트를 초기화 해야 한다.
        - 페이징 처리 : 데이터베이스 페이징 기능 사용
        - 커서 : 데이터베이스가 지원하는 커서 기능 사용

            > 커서 : 방대한 양의 데이터에서 특정 위치, 특정 로우(row)를 가리킬때 커서가 사용된다. 위키피디아에서는 커서에 대해 '많은 로우 중 한 로우를 가리키는 포인터와 같다'고 설명하고 있다.
            즉 커서란 현재 작업중인 레코드를 가리키는 오브젝트이다.
            >
    - JPA 페이징 배치 처리
        - 데이터베이스 페이징 기능을 사용. 한 번에 조회할 페이지 수를 지정. 페이지 단위마다 영속성 컨텍스트를 플러시하고 초기화
    - 하이버네이트 scroll 사용
        - 하이버네이트는 `scroll` 이라는 이름으로 JDBC 커서를 지원한다.
        - 하이버네이트 전용 기능이므로 먼저 `em.unwrap()` 메소드를 사용해 하이버네이트 세션을 구한다.
    - 하이버네이트 무상태 세션 사용
        - **무상태 세션은 영속성 컨텍스트를 만들지 않고 2차 캐시도 사용하지 않는다.**
        - 영속성 컨텍스트가 없어서 **엔티티 수정 시** 무상태 세션이 제공하는 **`update()` 메소드를 직접 호출해야** 한다.
- SQL 쿼리 힌트 사용
    - JPA는 데이터베이스 SQL 힌트 기능을 제공하지 않는다. SQL 힌트를 사용하려면 하이버네이트를 직접 사용해야 한다.
        - `addQueryHint()` 메소드 사용
    - SQL 힌트를 사용하려면 각 방언에서 다음 메소드를 오버라이딩해서 기능을 구현해야 한다.
- 트랜잭션을 지원하는 쓰기 지연과 성능 최적화
    - 트랜잭션을 지원하는 쓰기 지연과 JDBC 배치
        - 네트워크 호출 한 번의 비용 > 단순한 메소드 수만 번 호출 비용
        - `N 개의 INSERT SQL` 과 `1 번의 커밋`으로 데이터와 통신할 때, 여러 번의 INSERT SQL을 모아서 한번에 데이터베이스로 보냄으로써 최적화할 수 있다.
        - 이때 JDBC가 제공하는 `SQL 패치 기능`을 사용할 수 있다.
            - 이 기능을 사용하려면 코드의 많은 부분을 수정해야 한다.
            - 특히 비즈니스 로직이 복잡하게 얽혀 있는 곳에서 사용하기 쉽지 않고 적용해도 코드가 상당히 지저분해진다.
        - 따라서 보통은 **수백 수천 건 이상의 데이터를 변경하는 특수한 상황에 SQL 배치 기능을 사용**한다.
        - 만약 `hibernate.jdbc.batch_size` 속성의 값을 50으로 주면 최대 50건씩 모아서 SQL 배치를 실행한다.
        - SQL 배치는 같은 SQL일 때만 유효하다. 중간에 다른 처리가 들어가면 SQL 배치를 다시 시작한다.
        - 1,2,3,4 모아서 하나의 SQL 배치 실행. 5를 한 번 실행. 6, 7을 모아서 실행 => 총 3번 SQL 배치 실행

<br>
    <aside>
    📌 *엔티티가 영속 상태가 되려면 식별자가 꼭 필요한데, 만약 `IDENTITY 전략` 으로 식별자를 생성하면 데이터베이스에 엔티티를 저장해야 식별자를 구할 수 있다.
    따라서 `em.persist()` 호출 즉시 INSERT SQL이 데이터베이스에 전달되어 쓰기 지연을 활용한 성능 최적화를 할 수 없다.*

    </aside>
<br>

    - 트랜잭션을 지원하는 쓰기 지연과 애플리케이션 확장성
        - `트랜잭션을 지원하는 쓰기 지연`과 `변경 감지` 기능 덕분에 **성능**과 **개발의 편의성** 획득
        - 하지만 진짜 장점은 **데이터베이스 테이블 row에 lock 이 걸리는 시간을 최소화한다는 점이다.**
        - 즉, 이 기능은 트랜잭션을 커밋해서 영속성 컨텍스트를 플러시하기 전까지 데이터베이스 row에 lock을 걸지 않는다.
        - 트랜잭션 격리 수준에 따라 다르지만 보통 많이 사용하는 `커밋된 읽기(Read Committed) 격리 수준`이나 `그 이상`에서는 데이터베이스에 **현재 수정 중인 데이터를 수정하려는 다른 트랜잭션은 락이 풀릴 때까지 대기**한다.
        - `commit()` 호출할 때 `UPDATE SQL`을 실행하고 바로 데이터베이스 트랜잭션을 커밋
            - **데이터베이스에 락이 걸리는 시간 최소화**
        - 사용자가 증가하면 애플리케이션 서버를 증설하면 되지만, 데이터베이스 락은 애플리케이션 서버 증설만으로 해결이 불가능하다.
        - 오히려 **서버 증설로 트랜잭션이 증가할수록 더 많은 데이터베이스 락이 걸린다.**
        - `JPA 의 쓰기 지연 기능`은 **데이터베이스에 락이 걸리는 시간을 최소화해서 동시에 더 많은 트랜잭션을 처리할 수 있는 장점**이 있다.
