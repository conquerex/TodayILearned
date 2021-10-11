# 6장. 다양한 연관관계 매핑

## 입구

* 복습!! 엔티티의 연관관계를 매핑할 때 3가지 고려사항
  * 다중성
    * 다대일, 일대다, 일대일, 다대다
  * 단방향, 양방향
    * 테이블 : 원래 외래키로 양방향 지원. 사실상 방향 개념 없음
    * 객체 : 참조용 필드를 가지고 있는 객체만 연관된 객체를 조회 가능. 이 경우가 단방향
    * 양쪽을 서로 참조하면 양방향
  * 연관관계의 주인 (이하 주인)
    * 테이블의 연관관계를 관리하는 포인트는 외래키 하나
    * 하지만 엔티티를 양방향으로 매핑하면 2곳에서 서로를 참조 (관리 포인트 2곳)
    * 주인 : 두 객체 연관관계 중 하나를 정해서 데이터베이스 외래키를 관리


## 6.1 다대일

* (사견) 5장 내용과 대부분 겹침
* 외래키는 항상 다 쪽 = 주인은 항상 다 쪽
* 참조하는 필드 없음 = 조회 못함
* @JoinColum(name = "TEAM_ID")
  * Member.team 필드를 TEAM_ID 외래키와 매핑
  * Member.team 필드로 TEAM_ID 외래키를 관리
* 양방향은 외래키가 있는 쪽이 연관관계의 주인이다
* 양방향 연관관계는 항상 서로를 참조해야 한다
  * 서로 참조하게 하려면 연관관계 편의 메소드가 있으면 굳~ (setTeam, addMember)
  * 검사로직 없이 편의 메소드를 양쪽에 다 작성하면 무한루프가 될 수도!! (주의가 필요하다)


## 6.2 일대다

* 엔티티를 하나 이상 참조할 수 있다
  * 컬렉션 중 하나를 사용 (Collection, List, Set, Map)
* 하나의 팀이 여러 회원을 참조 (단방향)
  * 보통은 자신이 매핑한 테이블의 외래키를 관리하는데
  * 반대쪽 테이블에 있는 외래키를 관리
  * 일대다 관계에서 외래키는 항상 다쪽 테이블에 있다 + 다 쪽 엔티티에는 외래키를 매핑할 참조필드가 없다
  * 그래서 이 모양 ㅋ
  * @JoinColumn을 명시해야
    * 그렇지 않으면 JPA는 연결 테이블을 중간에 두고 연관관계를 관리하는 조인 테이블 전략을 기본으로 사용해서 매핑 (7장에서 자세히)
* 일대다 단방향 매핑의 단점
  * 외래키가 다른 테이블에 있다
  * 같이 있으면 INSERT 한번에 끝날 일을
  * 다른 테이블에 있어서 UPDATE SQL을 추가로 실행해야
  * member 객체(엔티티)는 team 정보(필드)가 없기 때문
* 일대다 단방향 매핑보다는 다대일 양방향 매핑을 사용하자
  * 성능 문제 + 관리의 부담
  * 다대일 양방향 : 관리해야 하는 외래키가 본인 테이블에 있다
* 일대다 양방향
  * 양방향 매핑에서 @OneToMany는 주인이 될 수 없다
  * 관계형 데이터베이스의 특성상 다 쪽에 외래키
  * 그래서 @ManyToOne이 주인 + 여기에는 mappedBy 속성이 없음
  * 일대다 양방향 매핑이 불가능은 아님
    * 주인이 아닌 쪽에 외래키를 사용하는 다대일 단방향 매핑을 읽기 전용으로 추가
    * 둘 모두 같은 키를 관리 - 다대일쪽은 insertable, updatable을 false로, 읽기만 가능
    * 일대다 양방향은 아니고 그렇게 보이는 방식 - 일대다 단방향의 단점 그대로
  * 권장 - 다대일 양방향을 쓰는걸로


## 6.3 일대일

* 회원과 사물함 관계
  * 어느 곳이나 외래키를 가질 수 있다
  * 주테이블과 대상 테이블
  * @OneToOne
* 주 테이블에 외래키
  * 객체지향 개발자들이 지향
  * JPA도 주 테이블에 외래키가 있으면 좀 더 편리하게 매핑 가능
  * 주 테이블만 확인해도 대상 테이블과 연관관계를 알 수 있다.
  * 멤버가 주 테이블일 때, 단방향
    * 다대일 단방향과 거의 유사
  * 멤버가 주 테이블일 때, 양방향
    * 주인 : 멤버 테이블이 외래키를 가지므로 Member.locker가 주인
    * Locker.member는 mappedBy를 선언
* 대상 테이블에 외래키
  * 데이터베이스 개발자들이 선호
  * 테이블 관계를 일대일에서 일대다로 변경할 때 테이블 구조를 그대로 유지할 수
  * 사물함이 주 테이블일 때, 단방향
    * JPA가 지원하지 않는 형태 + 이런 모양으로 매핑할 수 없다
      * (사견) 사물함 엔티티를 거치지 않고 DB 조회를? 당연히 불가능
    * 해결책 1. 사물함에서 멤버로 방향을 수정하거나
    * 해결책 2. 양방향으로 만들어서 사물함을 주인으로
  * 근데 일대다 단방향에서는 대상 테이블에 외래키가 있는 매핑이 되었다?
    * JPA 2.0부터 지원 but 일대일은 안됨

     <img src="../../group/orm-jpa/image/6-back.png" width="300px" title="Github_Logo"/><br><br>

  * 사물함이 주 테이블일 때, 양방향
    * 대상 엔티티인 사물함을 주인으로 만들어서 사물함 테이블의 외래키를 관리하도록
* 부모 테이블의 기본키를 자식테이블에서도 기본키로 사용하는 일대일 관계 - 7장에서 자세히
* 프록시를 사용할 때 외래키를 직접 관리하지 않는 일대일 관계(Member.locker)는 지연로딩으로 설정해도 즉시 로딩됨
  * 프록시의 한계 때문. 8장에서 자세히


## 6.4 다대다

* 관계형 데이터베이스는 정규화된 테이블 2개로 다대다 관계 표현 불가회원 대 상품연결 테이블이 필요 - 일대다, 다대일 관계로 풀어냄하지만 객체는 다대다 관계 가능@ManyToMany 다대다 : 단방향@ManyToMany와 @JoinTable을 사용해서 연결 테이블을 바로 매핑JoinTablename : 연결 테이블을 지정joinColumn : 현재 방향인 회원과 매핑할 조인 컬럼 정보를 지정inverseJoinColumn : 반대 방향인 상품과 매핑할 조인 컬럼 정보 지정이 매핑 덕분에 다대다 관계를 사용할 때는 연결 테이블을 신경 쓰지 않아도 된다다대다 : 양방향기존 단방향에서 주인이 아닌 쪽에 mappedBy를 추가하면 된다그리고 연관관계 편의 메소드를 추가다대다 : 매핑의 한계와 극복, 연결 엔티티 사


##


##


<br>

![](../../group/orm-jpa/image/5-end.png)