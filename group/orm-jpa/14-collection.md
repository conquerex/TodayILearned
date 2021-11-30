# 14장. 컬렉션과 부가 기능

## 입구

- JPA가 지원하는 컬렉션의 종류와 중요한 부가기능들을 알아보자
- 컬렉션
    - 다양한 컬렉션과 특징을 설명한다.
- 컨버터
    - 엔티티의 데이터를 변환해서 데이터베이스에 저장한다.
- 리스너
    - 엔티티에서 발생한 이벤트를 처리한다.
- 엔티티 그래프
    - 엔티티를 조회할 때 연관된 엔티티들을 선택해서 함께 조회한다.

## 1. 컬렉션

- JPA는 자바에서 기본으로 제공하는 Collection, List, Set, Map 컬렉션을 지원하고 다음 경우에 이 컬렉션을 사용할 수 있다.
    - @OneToMany, @ManyToMany를 사용해서 일대다나 다대다 엔티티 관계를 매핑할 때
    - @ElementCollection을 사용해서 값 타입을 하나 이상 보관할 때
- 자바 컬렉션 인터페이스의 특징
    - Collection
        - 자바가 제공하는 최상위 컬렉션
        - 하이버네이트는 중복을 허용하고 순서를 보장하지 않는다.
    - Set
        - 중복을 허용하지 않는 컬렉션
        - 순서를 보장하지 않는다.
    - List
        - 순서가 있는 컬렉션
        - 순서를 보장하고 중복을 허용
    - Map
        - Key, Value 구조로 되어 있는 특수한 컬렉션
- JPA 명세에는 자바 컬렉션 인터페이스에 대한 특별한 언급이 없다.
    - 따라서 JPA 구현체에 따라서 제공하는 기능이 조금씩 다를 수 있는데 여기서는 하이버네이트 구현체를 기준으로 이야기하겠다.

<br>

<aside>
📌 Map은 복잡한 매핑에 비해 활용도가 떨어지고 다른 컬렉션을 사용해도 충분하다.
Map은 @MapKey 어노테이션으로 매핑 가능하다

</aside>

<br>

### 1.1. JPA와 컬렉션

- 하이버네이트는 엔티티를 영속 상태로 만들 때 컬렉션 필드를 하이버네이트에서 준비한 컬렉션으로 감싸서 사용

```java
@Entity
public class Team {
		@Id
		private String id;

		@OneToMany
		@JoinColumn
		private Collection<Member> members = new ArrayList<Member>();
		...
}

// 영속상태 만들기
Team team = new Team();
System.out.println("before Persist = " + team.getMembers().getClass());
em.persist(parent);
System.out.println("after Persist = " + team.getMembers().getClass());

// 출력 결과
before Persist = class java.util.ArrayList
after Persist = class org.hibernate.collection.internal.PersistentBag
```

- 출력 결과를 보면 원래 ArrayList 타입이었던 컬렉션이 엔티티를 영속 상태로 만든 직후에 하이버네이트가 제공하는 PersistentBag 타입으로 변경되었다.
- 하이버네이트는 컬렉션을 효율적으로 관리하기 위해 엔티티를 영속 상태로 만들 때
    - 원본 컬렉션을 감싸고 있는 내장 컬렉션을 생성해서 이 내장 컬렉션을 사용하도록 참조를 변경한다.
- 하이버네이트가 제공하는 내장 컬렉션은 원본 컬렉션을 감싸고 있어서 래퍼 컬렉션으로도 부른다.
- 하이버네이트는 이런 특징 때문에 컬렉션을 사용할 때 다음처럼 즉시 초기화해서 사용하는 것을 권장한다.

```java
Collection<Member> members = new ArrayLust<Member>();
```
- 하이버네이트 내장 컬렉션과 특징 : p.613의 표 참고



### 1.2. Collection, List

- Collection, List 인터페이스는 중복을 허용하는 컬렉션
    - PersistentBag을 래퍼 컬렉션으로 사용한다.
    - 이 인터페이스는 ArrayList로 초기화하면 된다.
- 중복을 허용하므로
    - 객체를 추가하는 add() 메소드는 내부에서 어떤 비교도 하지 않고 항상 true를 반환
- 같은 엔티티가 있는지 찾거나 삭제할 때는 equals() 메소드를 사용
- Collection, List는 엔티티를 추가할 때
    - **중복된 엔티티가 있는지 비교하지 않고 단순히 저장만 하면 된다.**
    - **따라서 엔티티를 추가해도 지연 로딩된 컬렉션을 초기화하지 않는다.**

```java
@OneToMany
@JoinColumn
private List<ListChild> list = new ArrayList<>();
```

### 1.3. Set

- Set은 중복을 허용하지 않는 컬렉션
    - 하이버네이트는 PersistentSet을 컬렉션 래퍼로 사용한다.
    - 이 인터페이스는 HashSet으로 초기화하면 된다.
- HashSet은 중복을 허용하지 않으므로
    - add() 메소드로 객체를 추가할 때 마다 equals() 메소드로 같은 객체가 있는지 비교한다.
    - 같은 객체가 없으면 객체를 추가하고 true를 반환하고,
    - 같은 객체가 이미 있어서 추가에 실패하면 false를 반환한다.
    - 참고로 HashSet은 해시 알고리즘을 사용하므로 hashcode()도 함께 사용해서 비교한다.
- **Set은 엔티티를 추가할 때 중복된 엔티티가 있는지 비교해야** 한다.
    - **따라서 엔티티를 추가할 때 지연 로딩된 컬렉션을 초기화**한다.

```java
@OneToMany
@JoinColumn
private Set<SetChild> set = new HashSet<>();
```

### 1.4. List + @OrderColumn

- List 인터페이스에 @OrderColumn을 추가하면 순서가 있는 특수한 컬렉션으로 인식
    - 순서가 있다는 의미는 데이터베이스에 순서 값을 저장해서 조회할 때 사용한다는 의미
    - 하이버네이트는 내부 컬렉션은 PersistentList를 사용한다.
- 순서가 있는 컬렉션은 데이터베이스에 순서 값도 함께 관리
    - @OrderColumn의 name 속성에 POSITON이라는 값을 주었다.
    - JPA는 List의 위치 값을 테이블의 POSITION 컬럼에 보관한다.
    - 위치 값은 다(N) 쪽에 저장해야 한다. (p.616 참고)

```java
@Entity
class Board{
    @Id @GeneratedValue
    private Integer id;

    @OneToMany(mappedBy = "board")
    @OrderColumn(name = "POSITION")
    private List<Comment> comments = new ArrayList<>();
}

@Entity
class Comment{
    @Id @GeneratedValue
    private Integer id;

    @ManyToOne
    @JoinColumn(name = "BOARD_ID")
    private Board board;
}
```

- @OrderColumn은 실무에서 사용하기에 단점이 많아서
    - 개발자가 직접 POSITION 값을 관리하거나 다음에 설명하는 @OrderBy를 사용하길 권장
- OrderColumn의 단점
    - @OrderColumn을 Board 엔티티에서 매핑하므로 Comment는 POSITION의 값을 알 수 없다. Comment를 INSERT할 때는 POSITION 값이 저장되지 않는다. POSITION은 Baord.comments의 위치 값이므로, 이 값을 사용해서 POSITION의 값을 UPDATE 하는 SQL이 추가로 발생한다.
    - List를 변경하면 연관된 많은 위치 값을 변경해야 한다.
    - 중간에 POSITION 값이 없으면 조회한 List에는 null이 보관된다. 따라서 컬렉션을 순회할 때 NullPointerException가 발생한다.

### 1.5. @OrderBy

- @OrderColumn이 데이터베이스에 순서용 컬럼을 매핑해서 관리했다면
    - @OrderBy는 데이터베이스의 ORDER BY절을 사용해서 컬렉션을 정렬
    - 따라서 순서용 컬럼을 매핑하지 않아도 된다.
    - @OrderBy는 모든 컬렉션에 사용할 수 있다.

```java
@Entity
public class Team {

	@Id @GeneratedValue
	private Long id;
	private String name;

	@OneToMany(mappedBy = "team")
	@OrderBy("username desc, id asc")
	private Set<Member> members = new HashSet<Member>();
	...
}

SELECT M.*
FROM MEMBER M
WHERE M.TEAM_ID=?
ORDER BY
	M.MEMBER_NAME DESC,
	M.ID ASC
```

- @OrderBy의 값은 JPQL의 order by 절처럼 엔티티의 필드를 대상으로 한다.

<aside>
📌 하이버네이트는 Set에 @OrderBy를 사용하면,
순서를 유지하기 위해 내부적으로 HashSet 대신에 LinkedHashSet을 내부에서 사용한다.

</aside>

## 2. @Converter

- 컨버터를 사용하면 엔티티의 데이터를 변환해서 데이터베이스에 저장 할 수 있다.
- 컨버터 클래스는
    - @Converter 어노테이션을 사용하고
    - AttributeConverter 인터페이스를 구현해야 한다.
- 제네릭에 현재 타입과 변환할 타입을 지정해야 한다.

```java
@Entity
public class Member {
	@Id
	private String id;
	private String username;

	@Convert(converter=BooleanToYNConverter.class)
	private boolean vip;

	...
}

// Boolean 을 YN으로 변환해주는 컨버터
@Converter
public class BooleanToYNConverter implements AttributeConverter<Boolean, String. {
    @Override
    public String convertToDatabaseColumn(Boolean attribute) {
        return (attribute != null && attribute) ? “Y” : “N”;
    }
    @Override
    public Boolean ConvertToEntityAttribute(String dbData) {
        return “Y”.equals(dbData);
    }
}
```

- AttributeConverter 인터페이스에는 구현해야 할 다음 두 메소드가 있다
    - convertToDatabaseColumn()
        - 엔티티의 데이터를 데이터베이스 컬럼에 저장할 데이터로 변환한다.
    - convertToEntityAttribute()
        - 데이터베이스에서 조회한 컬럼 데이터를 엔티티의 데이터로 변환한다.
- 클래스 레벨도 설정 할 수 있다.
    - 이때는 attributeName 속성을 사용해서 어떤 필드에 컨버터를 적용할지 명시해야 한다.

```java
@Entity
@Convert(converter=BooleanToYNConverter.class, attributeName = "vip")
public class Member {
	@Id
	private String id;
	private boolean vip;
}
```

- 글로벌 설정
    - 모든 Boolean 타입에 컨버터를 적용하려면 @Converter(autoApply = true) 옵션을 적용하면 된다.

```java
@Converter(autoApply = true)
public class BooleanToYNConverter implements AttributeConverter<Boolean, String> {

	@Override
	public String convertToDatebaseColumn(Boolean attribute) {
		return (attribute != null && attribute) ? "Y" : "N";
	}

	@Override
	public Boolean convertToEntityAttribute(String dbData) {
		return "Y".equals(dbData);
	}
}
```

- Converter 속성 : p.623의 표 참고

<br>


## 3. 리스너

- 모든 엔티티를 대상으로 언제 어떤 사용자가 삭제를 요청했는지 모두 로그로 남겨야 하는 요구사항이 있다고 가정
    - 어플리케이션 삭제 로직을 하나씩 찾아서 로그를 남기는 것은 비효율적이다.
- JPA 리스너 기능을 사용하면 엔티티의 생명주기에 따른 이벤트를 처리할 수 있다.
- 이벤트를 잘 활용하면
    - 대부분의 엔티티에 공통으로 적용하는 등록 일자, 수정 일자 처리와
    - 해당 엔티티를 누가 등록하고 수정했는지에 대한 기록을 리스너 하나로 처리할 수 있다.

### 3.1. 이벤트 종류 (p.623 이미지 참고)

1. PostLoad
    - 엔티티가 영속성 컨텍스트에 조회된 직후 또는 refresh를 호출한 후(2차 캐시에 저장되어 있어도 호출된다.)
    - 예. find() JPQL
    - 사견 : 이후부터 Persist, Update, Remove 순으로 진행
2. PrePersist
    - persist() 메소드를 호출해서 엔티티를 영속성 컨텍스트에 관리하기 직전에 호출된다.
3. PreUpdate
    - flush나 commit을 호출해서 엔티티를 데이터베이스에 수정하기 직전에 호출된다.
4. PreRemove
    - remove() 메소드를 호출해서 엔티티를 영속성 컨텍스트에서 삭제하기 직전에 호출된다.
    - 또한 삭제 명령어로 영속성 전이가 일어날 때도 호출된다.
    - orphanRemoval에 대해서는 flush나 commit 시에 호출된다.
    - 사견 : 여기까지는 특정 상황의 직전 (Pre)
5. PostPersist
    - flush나 commit을 호출해서 엔티티를 데이터베이스에 저장한 직후에 호출된다.
    - 식별자가 항상 존재한다.
    - 참고로 식별자 생성 전략이 IDENTITY면 식별자를 생성하기 위해 persist()를 호출하면서 데이터베이스에 해당 엔티티를 저장하므로 이때는 persist()를 호출한 직후에 바로 PostPersist가 호출된다.
6. PostUpdate
    - flush나 commit을 호출해서 엔티티를 데이터베이스에 수정한 직후에 호출된다.
7. PostRemove
    - flush나 commit을 호출해서 엔티티를 데이터베이스에 삭제한 직후에 호출된다.

### 3.2. 이벤트 적용 위치

- 이벤트는 엔티티에서 직접 받거나 별도의 리스너를 등록해서 받을 수 있다.
    - 엔티티에 직접 적용
        - 엔티티에 이벤트가 발생할 때마다 어노테이션으로 지정한 메소드가 실행된다.

        ```java
        @Entity
        public class Duck {

            @Id @GeneratedValue
            public Long id;

            private String name;

            @PrePersist
            public void prePersist() {
                System.out.println("Duck.prePersist id=" + id);
            }
            public void postPersist() {
                System.out.println("Duck.postPersist id=" + id);
            }
            @PostLoad
            public void postLoad() {
                System.out.println("Duck.postLoad");
            }
            @PreRemove
            public void preRemove() {
                System.out.println("Duck.preRemove");
            }
            @PostRemove
            public void postRemove() {
                System.out.println("Duck.postRemove");
            }
            ...
        }
        ```

    - 별도의 리스너 등록
        - 리스너는 대상 엔티티를 파라미터로 받을 수 있다. 반환 타입은 void로 설정해야 한다.

        ```java
        @Entity
        @EntityListeners(DuckListener.class)
        public class Duck {...}

        public class DuckListener {

            @PrePersist
            // 특정 타입이 확실하면 특정 타입을 받을 수 있다.
            private void prePersist(Object obj) {
                System.out.println("DuckListener.prePersist obj = [" + obj + "]");
            }
            @PostPersist
            // 특정 타입이 확실하면 특정 타입을 받을 수 있다.
            private void postPersist(Object obj) {
                System.out.println("DuckListener.postPersist obj = [" + obj + "]");
            }
        }
        ```

    - 기본 리스너 사용
        - 기본 리스너를 사용하는 방법
            - 모든 엔티티의 이벤트를 처리할 수 있다.
            - `META-INF/orm.xml` 파일에 기본 리스너로 등록하면 된다.
        - 이벤트 호출 순서
            1. 기본 리스너
            2. 부모 클래스 리스너
            3. 리스너
            4. 엔티티
    - 더 세밀한 설정을 할 수 있는 어노테이션도 제공
        - 기본 리스너 무시
            - `javax.persistence.ExcludeDefaultListeners`
        - 상위 클래스 이벤트 리스너 무시
            - `javax.persistence.ExcludeSuperclassListeners`


        ```java
        @Entity
        @EntityListeners(DuckListener.class)
        @ExcludeDefaultListeners
        @ExcludeSuperclassListeners
        public class Duck extends BaseEntity {...}
        ```


## 4. 엔티티 그래프

- 엔티티를 조회할 때 연관된 엔티티들을 함께 조회하려면
    - 글로벌 Fetch 옵션을 FetchType.EAGER로 설정
    - 또는 JPQL에서 페치 조인을 사용
- 글로벌 fetch 옵션은 애플리케이션 전체에 영향을 주고 변경할 수 없는 단점
    - 그래서 일반적으로 글로벌 fetch 옵션은 FetchType.LAZY를 사용하고
    - 엔티티를 조회할 때 연관된 엔티티를 함께 조회할 필요가 있으면 JPQL의 페치 조인을 사용
- 그런데 페치 조인을 사용하면 같은 JPQL을 중복해서 작성하는 경우가 많다.

```sql
select o from Order o where o.status = ?

select o from Order o join fetch o.Member where o.status = ?

select o from Order o join fetch o.orderItems where o.status = ?
```

- 3가지 JPQL 모두 주문을 조회하는 JPQL이지만
    - 함께 조회할 엔티티에 따라서 다른 JPQL을 사용해야 한다.
    - 연관 엔티티에 따라, JPQL이 많아지는 단점
- 이는 엔티티 그래프를 사용해서 연관된 엔티티를 함께 조회하면 되고
    - JPQL은 데이터를 조회하는 기능만 수행할 수 있다. `select o from Order o`

### 4.1. Named 엔티티 그래프

- Named 엔티티 그래프는 @NamedEntityGraph로 정의한다.
    - name: 엔티티 그래프의 이름을 정의
    - attributeNodes 함께 조회할 속성을 선택한다.
        - @NamedAttributeNode를 사용하고 그 값으로 함께 조회할 속성을 선택한다.

```java
@NamedEntityGraph(name = "Order.withMember", attributeNodes = {
	@NamedAttributeNode("member")
})
@Data
@Entity
@Table(name = "ORDERS")
public class Order {
	@Id
	@GeneratedValue(strategy = GenerationType.IDENTITY)
	@Column(name = "ORDER_ID")
	private Long id;

	private String name;

	@ManyToOne(fetch = FetchType.LAZY, optional = false)
	@JoinColumn(name = "MEMBER_ID")
	private Member member;
}

public interface OrderRepository extends JpaRepository<Order, Long> {

  /**
   * Order 조회시에 Member도 같이 조회된다.
   */
	@EntityGraph(value = "Order.withMember")
	Optional<Order> findById(Long id);
}
```

- Order.member가 지연 로딩으로 설정되어있지만, 엔티티 그래프에서 함께 조회할 속성으로 member를 선택했으므로
    - 이 엔티티 그래프를 사용하면 Order를 조회할 때 연관된 member도 함께 조회할 수 있다.
- 둘 이상 정의하려면 @NamedEntityGraphs를 사용

### 4.2. em.find() 에서 엔티티 그래프 사용

- Named 엔티티 그래프를 사용하려면
    - 정의한 엔티티 그래프를 em.getEntityGraph(“Order.withMember”)를 통해서 찾아오면 된다.
- 엔티티 그래프는 JPA의 힌트 기능을 사용해서 동작하는데
    - 힌트의 키로 javax.persistence.fetchgraph를 사용하고
    - 힌트의 값으로 찾아온 엔티티 그래프를 사용하면 된다.
    - 질문 : 반드시 힌트 키값은 javax.persistence.fetchgraph로 [해야하는가](https://micronaut-projects.github.io/micronaut-data/2.0.1/api/io/micronaut/data/jpa/annotation/EntityGraph.html)?

```java
EntityGraph graph = em.getEntityGraph("Order.withMember");

Map hints = new HashMap();
hints.put("javax.persistence.fetchgraph", graph);

Order order = em.find(Order.class, orderId, hints);
```

### **4.3. subgraph**

- Order.withAll이라는 Named 엔티티 그래프를 정의해서
    - 원래 OrderItem → Item은 Order가 관리하는 필드가 아니다. —> subgraph 속성으로 정의해야
    - Order -> Member, Order -> OrderItem, OrderItem -> Item의 객체 그래프를 함께 조회

```java
@NamedEntityGraph(name = "Order.withAll", attributeNodes = {
	@NamedAttributeNode("member"),
	@NamedAttributeNode(value = "orderItems", subgraph = "orderItems")
	},
	**subgraphs = @NamedSubgraph(name = "orderItems", attributeNodes = {
		@NamedAttributeNode("item")
	})**
)
@Entity
@Table(name = "ORDERS")
public class Order {

	@Id @GeneratedValue
	private Long id;

	@ManyToOne(fetch = FetchTYpe.LAZY, optional = false)
	@JoinCloumn(name = "MEMBER_ID")
	private Member member;

	@OneToMany(mappedBy = "order", cascade = CascadeType.ALL)
	private List<OrderItem> orderItems = new ArrayList<OrderItem>();
	...
}

@Entity
public class OrderItem {

	@Id @GeneratedValue
	private Long id;

	@ManyToOne(fetch = FetchType.LAZY)
	@JoinColumn(name = "ITEM_ID")
	private Item item;

	...
}
```

### **4.4. JPQL에서 엔티티 그래프 사용**

- JPQL에서 엔티티 그래프를 사용하는 방법은 `em.find()` 와 동일하게 힌트만 추가하면 된다.

```java
List<Order> resultList =
	em.createQuery("select o from Order o where o.id = :orderId", Order.class)
		.setParameter("orderId", orderId)
		**.setHint("javax.persistence.fetchgraph", em.getEntityGraph("Order.withAll"))**
		.getResultList();
```

<aside>
📌 다음 코드 같이 Order.member는 필수 관계로 설정되어 있다.

@ManyToOne(fetch = FetchType.LAZY, **optional = false**) // 필수 관계로 설정
@JoinColumn(name = "MEMBER_ID")
private Member member; // 주문 회원

`em.find()` 에서 엔티티 그래프를 사용하면, 하이버네이트는 필수 관계를 고려해서 내부 조인을 실행하지만, `JPQL` 같은 경우에는 항상 SQL 외부 조인을 사용. 만약 SQL 내부 조인을 사용하려면 다음처럼 내부 조인을 명시하면 된다.

select o from Order o join fetch o.member where o.id = :orderId

</aside>

### 4.5. 동적 엔티티 그래프

- 엔티티 그래프를 동적으로 구성하려면 createEntityGraph() 메소드를 사용하면 된다.
    - em.createEntityGraph(Order.class)를 사용해서 동적으로 엔티티 그래프 생성
    - graph.addAttributeNodes(“member”)를 사용해서 Order.member 속성을 엔티티 그래프에 포함
- addSubgraph 메서드를 사용해서 서브 그래프를 만들었다
    - 원하는 속성(예. item)을 포함하도록 했다

```java
// 동적 엔티티 그래프
EntityGraph<Order> graph = **em.createEntityGraph(Order.class);**
**graph.addAttributeNodes("member");**

Map hints = new HashMap();
hints.put("javax.persistence.fetchgraph", graph);

Order order = em.find(Order.class, orderId, hints);
```

```java
// 동적 엔티티 그래프 subgraph
**EntityGraph<Order> graph = em.createEntityGraph(Order.class);**
graph.addAttributeNodes("member");
Subgraph<OrderItem> orderItems = **graph.addSubgraph("orderItems");**
orderItems.addAttributeNodes("item");

Map hints = new HashMap();
hints.put("javax.persistence.fetchgraph", graph);

Order order = em.find(Order.class, orderId, hints);
```

### 4.6. 엔티티 그래프 정리

- ROOT에서 시작
    - 엔티티 그래프는 항상 조회하는 엔티티의 ROOT에서 시작해야 한다.
    - 당연한 이야기지만 Order 엔티티를 조회하는데 Member부터 시작하는 엔티티 그래프를 사용하면 안된다.
- 이미 로딩된 엔티티
    - 영속성 컨텍스트에 엔티티가 이미 로딩되어 있으면 엔티티 그래프가 적용되지 않는다.
    - 즉, 초기화 되지 않은 프록시에는 엔티티 그래프가 적용된다.
    - 조회된 order2에는 엔티티 그래프가 적용되지 않고 처음 조회한 order1과 같은 인스턴스가 반환된다.

```java
Order order1 = em.find(Order.class, orderId); // 이미 조회
hints.put("javax.persistence.fetchgraph", em.getEntityGraph("Order.withMember"));
Order order2 = em.find(Order.class, orderId, hints);
```

- fetchgraph, loadgraph 차이
    - javax.persistence.fetchgraph 힌트를 사용해서 엔티티 그래프를 조회했다.
        - 이것은 엔티티 그래프에 선택한 속성만 함께 조회한다.
    - 반면에 javax.persistence.loadgraph 속성은
        - 엔티티 그래프에 선택한 속성뿐만 아니라
        - 글로벌 fetch 모드가 FetchType.EAGER로 설정된 연관관계도 포함해서 함께 조회한다.

<aside>
📌 하이버네이트 4.3.10.Final 버전에서는
loadgraph 기능이 em.find()를 사용할 때는 정상 동작하지만
JPQL을 사용할 때는 정상 동작하지 않고 fetchgraph와 같은 방식으로 동작한다.

</aside>

## 5. 정리

- JPA가 지원하는 컬렉션의 종류와 특징들을 알아보았다.
- 컨버터를 사용하면 엔티티의 데이터를 변환해서 데이터베이스에 저장할 수 있다.
- 리스너를 사용하면 엔티티에서 발생한 이벤트를 받아서 처리할 수 있다.
- 페치 조인은 객체지향 쿼리를 사용해야 하지만
    - 엔티티 그래프를 사용하면 객체 지향 쿼리를 사용하지 않아도 원하는 객체그래프를 한 번에 조회할 수 있다.
- 다음 장에서는 JPA의 다양한 심화 주제와 성능 최적화 방법을 다룬다.
