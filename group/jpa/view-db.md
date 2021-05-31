---
description: '2021.05.31, 월요일'
---

# View 환경 설정과 DB 설정

### 강의 항목

* View 환경 설정
* H2 데이터베이스 설치
* JPA와 DB 설정, 동작확인



### 학습 내용

```java
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.GetMapping;

@Controller
public class HelloContoller {

    @GetMapping("hello")
    public String hello(Model model) {
        model.addAttribute("data", "hello!!!!");
        return "hello";
    }
}
```

```markup
<!DOCTYPE HTML>
<html xhtmllns:th="http://www.thymeleaf.org">
<head>
    <title>Hello</title>
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8"/>
</head>
<body>
<p th:text="'똑똑. 안녕하세요. ' + ${data}">안녕하세요. 손님</p>
</body>
</html>
```

```markup
<!DOCTYPE HTML>
<html>

<head>
    <title>Hello</title>
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8"/>
</head>
<body>
Hello
<a href="/hello">hello^^</a>
</body>
</html>
```



* H2 Database
  * [https://www.h2database.com/html/main.html](https://www.h2database.com/html/main.html)
  * 1.4.199 버전이 좀 더 안정화 되어 있음
  * jdbc:h2:~/jpashop \(최소 한번\)
  * ~/jpashop.mv.db 파일 생성 확인
  * 이후 부터는 jdbc:h2:tcp://localhost/~/jpashop



* main/resources/application.yml 추가
  * 띄어쓰기 2칸 간격 주의!!!
  * spring.jpa.hibernate.ddl-auto
    * create : 애플리케이션 실행 시점에 테이블을 drop 하고, 다시 생성
  * show\_sql : System.out 에 하이버네이트 실행 SQL을 남긴다
  * org.hibernate.SQL : logger를 통해 하이버네이트 실행 SQL을 남긴다.

```yaml
spring:
  datasource:
    url: jdbc:h2:tcp://localhost/~/jpashop;
    username: sa
    password:
    driver-class-name: org.h2.Driver

  jpa:
    hibernate:
      ddl-auto: create
    properties:
      hibernate:
        show_sql: true
        format_sql: true

logging:
  level:
    org.hibernate.SQL: debug
```

```java
import lombok.Getter;
import lombok.Setter;

import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.Id;

@Entity
@Getter
@Setter
public class Member {
    @Id @GeneratedValue
    private Long id;
    private String username;
}
```

```java
import org.springframework.stereotype.Repository;
import javax.persistence.EntityManager;
import javax.persistence.PersistenceContext;

@Repository
public class MemberRepository {
    @PersistenceContext
    private EntityManager em;

    public Long save(Member member) {
        em.persist(member);

        // 커맨드와 쿼리를 분리하기
        return member.getId();
    }

    public Member find(Long id) {
        return em.find(Member.class, id);
    }
}
```

* 테스트
  * Junit4 : 어노테이션 @Test 사용시, org.junit.Test
  * assertThat 사용시
    * org.assertj.core.api.Assertions
  * 테스트시 @Transactional 필수
    * org.springframework.transaction.annotation.Transactional
  * 만약 DB에서도 데이터를 확인하고 싶다면
    * @Rollback\(value = false\)

```text
import org.assertj.core.api.Assertions;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.annotation.Rollback;
import org.springframework.test.context.junit4.SpringRunner;
import org.springframework.transaction.annotation.Transactional;

@RunWith(SpringRunner.class)
@SpringBootTest
public class MemberRepositoryTest {

    @Autowired
    MemberRepository memberRepository;

    @Test
    @Transactional
    @Rollback(value = false)
    public void testMember() throws Exception {
        // given
        Member member = new Member();
        member.setUsername("gggg");

        // when
        Long savedId = memberRepository.save(member);
        Member findMember = memberRepository.find(member.getId());

        // then
        Assertions.assertThat(findMember.getId()).isEqualTo(member.getId());
        Assertions.assertThat(findMember.getUsername()).isEqualTo(member.getUsername());
        Assertions.assertThat(findMember).isEqualTo(member);
        System.out.println("findMember == member : " + (findMember == member));
    }

}
```



```text
Assertions.assertThat(findMember).isEqualTo(member);
System.out.println("findMember == member : " + (findMember == member));
```

* 위 코드가 통과되는 이유
  * 영속성 컨텍스트에 필요한 결과가 있으면 별도의 select 쿼리 없이 해당 결과를 가지고 온다.
* jar 빌드하는 방법
  * ...은 생략
* 쿼리 파라미터 로그 남기기
  * org.hibernate.type : 그런데 이거보다 좋은게...
  * spring-boot-data-source-decorator
    * [https://github.com/gavlyukovskiy/spring-boot-data-source-decorator](https://github.com/gavlyukovskiy/spring-boot-data-source-decorator)

```text
implementation("com.github.gavlyukovskiy:p6spy-spring-boot-starter:1.7.1")
```





