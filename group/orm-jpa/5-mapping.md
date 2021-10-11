# 5장. 연관관계 매핑 기초

## 입구

* <mark style="color:red;">**객체는 참조(주소)**</mark>**를 사용해서 관계를 맺고**
* <mark style="color:red;">**테이블은 외래키**</mark>**를 사용해서 관계를 맺는다**
* 객체 관계 매핑(ORM)에서 가장 어려운 부분 > 객체 연관관계와 테이블 연관관계를 매핑하는 일
* 방향
  * 단방향, 양방향
* 다중성
  * 다대일, 일대다, 일대일, 일대다
* 연관관계의 주인
  * 양방향 연관관계로 만들면 연관관계의 주인을 정해야



5.1 단방향 연관관계

* 회원과 팀
* 객체 연관관계
  * 회원 객체는 Member.team 필드(멤버변수)로 팀 객체와 연관관계를 맺는다
  * 단방향 : 회원을 통해 팀을 알수 있지만 팀은 회원을 알 수 없다. 
  * 예) member.getTeam()
* 테이블 연관관계
  * 회원 테이블은 TEAM_ID 외래키로 팀 테이블과 연관관계를 맺는다
  * 양방향 : 회원 테이블의 TEAM_ID 외래키를 통해서 회원과 팀을 조인할 수 있다
  * 예) MEMBER JOIN TEAM, TEAM JOIN MEMBER

```
SELECT *
FROM MEMBER M
JOIN TEAM T ON M.TEAM_ID = T.TEAM_ID

SELECT *
FROM TEAM T
JOIN MEMBER M ON T.TEAM_ID = M.TEAM_ID
```

* 객체 연관관계와 테이블 연관관계의 가장 큰 차이
  * 참조를 통한 연관관계는 언제나 단방향
  * 양방향 관계 : 정확히는 서로 다른 단방향 관계 2










