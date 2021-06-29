---
description: '2021.05.31, 월요일'
---

# 도메인 분석 설계

## 강의 항목

* 요구사항 분석
* 도메인 모델과 테이블 설계
* 엔티티 클래스 개발1
* 엔티티 클래스 개발2
* 엔티티 설계시 주의점

## 학습 내용

![&#xB3C4;&#xBA54;&#xC778; &#xBAA8;&#xB378;](blob:https://app.gitbook.com/5f4d7a59-9ab7-41da-8a9c-e3a10528a11e)

![&#xD14C;&#xC774;&#xBE14; &#xC124;&#xACC4;](blob:https://app.gitbook.com/5655e1f0-c89d-4b4c-b282-078fb332dafa)

![&#xD68C;&#xC6D0; &#xD14C;&#xC774;&#xBE14; &#xBD84;&#xC11D;](blob:https://app.gitbook.com/42226c30-8230-4386-b7a8-42283612c0b3)

* 알아둬야 하는 것
  * 연관관계
  * 외래키가 있는 곳을 연관관계의 주인으로
  * 값 타입, 임베디드 타입 : Address.class
  * @Enumerated\(EnumType.STRING\)
  * 다대다 관계
* 엔티티 클래스 개발
  * Getter는 OK
    * BUT!!! Setter는 닫는게 좋다.
  * @ManyToMany : 실무에서는 쓰지 말자
  * 값타입은 변경 불가능하게 \(ex. Address.class\)
    * 생성자에서 값을 초기화
    * 이때 기본 생성자 필요 \(public, **protected**\) - 리플렉션 같은 기술을 사용할 수 있도록 하기 위함

> 여기까지 코드  
> [https://github.com/conquerex/WhatTheJpa2nd/commit/6cb76ded6271e3646f34bc6eb8cc9803fd9f1eb5](https://github.com/conquerex/WhatTheJpa2nd/commit/6cb76ded6271e3646f34bc6eb8cc9803fd9f1eb5)

* 엔티티 설계시 주의점
  * Setter는 가급적 사용말자
  * 모든 연관관계는 지연로딩으로
    * NO `EAGER`. YES `LAZY`. N+1 이슈
    * fetch join, 엔티티 그래프 기능을 사용
    * @XtoOne --&gt; 직접 지연로딩으로
  * 컬렉션은 필드에서 초기화
    * null 문제에서 해방
    * 영송화 후 하이버네이트에서 제공하는 내장 컬렉션으로 변경
    * 임의의 메서드로 컬렉션을 잘못 생성하면 하이버네이트 내부 메커니즘에 문제가 생길 수 있다.
    * 필드레벨에서 생성 : 가장 안전, 코드도 간결
  * 엔티티 필드명과 테이블 컬럼명
    * 동일하게 사용. `SpringPhysicalNamingStrategy`
    * 카멜 --&gt; 스네이크 케이스
    * . --&gt; \_
    * 대문자 --&gt; 소문자
* CascadeType
  * persist를 전파
  * List 등을 사용시 유용하다
* 연관관계 편의 메소드
  * 양방향 연관관계에서는 꼭 활용하자.

> 여기까지 코드  
> [https://github.com/conquerex/WhatTheJpa2nd/commit/2cf251cfc84682cb4de9a66abf2b49510fb251d8](https://github.com/conquerex/WhatTheJpa2nd/commit/2cf251cfc84682cb4de9a66abf2b49510fb251d8)

