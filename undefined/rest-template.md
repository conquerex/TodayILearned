---
description: 2021.06.14~15
---

# Rest Template

Spring 3부터 지원 되었고 REST API 호출이후 응답을 받을 때까지 기다리는 동기방식

```text
package org.springframework.web.client;

public class RestTemplate extends InterceptingHttpAccessor implements RestOperations {
  ...
}
```

### 📡 HTTP 서버와의 통신방법

* URLConnection
  * java.net 패키지
  * URL 내용을 읽어오거나 URL 주소에 GET, POST로 데이터를 전달할 때 사용
  * 응답코드 4xx, 5xx의 경우 IOException
  * 타임아웃 설정 X
  * 쿠키 제어 X
* HttpClient
  * org.apache.http vozlwl
  * 모든 응답코드를 읽을 수 있다
  * 타임아웃 설정 O
  * 쿠키 제어 O
  * 반복적이고 코드들이 길다
  * 응답의 컨텐츠 타입에 따라 별도 로직이 필요

### ⚙️ RestTemplate 동작원리

* org.springframework.http.client 패키지
  * HttpClinet는 HTTP를 사용하여 통신하는 범용 라이브러리
  * RestTemplate는 HttpClient를 추상화해서 제공 \(HttpEntity의 json, xml 등\)
* 따라서 내부 통신\(Http 커넥션\)에 있어서는 Apache HttpComponents를 사용
  * 만약 RestTemplate가 없었다면 직접 json, xml 라이브러리를 사용해서 변환해야 했을 것

1. 애플리케이션이 RestTemplate를 생성. URI, HTTP 메소드 등의 헤더를 담아 요청
2. RestTemplate는 HttpMessageConverter를 사용하여 requestEntity를 요청 메세지로 변환
   1. object → 메세지
3. RestTemplate는 ClientHttpRequsetFactory로부터 ClientHttpRequest를 가져와서 요청을 보냄
4. ClientHttpRequest는 요청메세지를 만들어 HTTP 프로토콜을 통해 서버와 통신
5. RestTemplate는 ResponseErroHandler로 오류를 확인하고 있다면 처리 로직을 태움
6. ResponseErrorHandler는 오류가 있다면 ClientHttpResponse에서 응답데이터를 가져와서 처리
7. RestTemplate는 HttpMessageConverter를 이용해서 응답메세지를 java object\(Class responseType\)로 변환
   1. 메세지 → object
8. 어플리케이션에 반환

### 🍜 Connection pool 적용에 관해서

* RestTemplate는 기본적으로 connection pool을 사용하지 않는다
  * 연결할 때마다 로컬 포트를 열고 tcp connection을 맺는다
* 문제 : close\(\) 이후 사용된 소켓을 TIME\_WAIT 상태가 되는데…
  * 요청량이 많다면 이런 소켓들을 재사용하지 못하고
  * 소켓이 소진되어 응답이 지연될 것이다
* 해결 : connection pool을 사용
  * DBCP처럼 소켓의 갯수를 정해서 재사용하는 것
* RestTemplate에서 connection pool을 적용하려면 HttpClient를 만들고 setHttpClient를 해야한다

### 🍖 GET 메서드

* getForObject
  * GET을 수행하고 HTTP 응답을 객체타입\(Json 등\)으로 변환해서 반환
* getForEntity
  * 응답을 ResponseEntity 객체로 받게 됨
  * HTTP 응답에 대한 추가정보를 담고 있어서
    * 응답 코드 , 실제 데이터를 확인할 수 있다

### 🍖 POST 메서드

* postForObject 헤더 포함하지 않고 보내기
  * POST 요청에 대해서 반환 값을 해당 객체로 반환해주는 메서드
  * 응답값 : 받은 객체와 응답코드를 ResponseEntity로 반환할 수 있다
* postForObject 헤더 포함해서 보내기
  * 객체와 custom 헤더를 인자로 넘겨 HttpEntity를 생성
  * Controller에서 @RequestHeader 어노테이션으로 값을 얻어올 수 있다
* postForEntity
  * ResponseEntity 객체로 데이터를 받을 수 있다
* postForLocation
  * 객체를 반환하는 대신 생성된 리소스의 URI 위치를 반환
  * Controller에서는 헤더에 URI location 값을 저장하여 ResponseEntity로 반환

### 🍖 DELETE 메서드

* HTTP DELETE를 수행

### 🍖 PUT 메서드

* 다른 메서드와 비슷 \(ex. postForObject\)
* 데이트를 업데이트하기 위해 요청을 보냄
* body에 데이터를 실어서 보냄

### 🍖 Exchange 메서드

* 지금까지는 HTTP 메서드별 별도의 메서드
* 모두 처리가 가능한 하나의 메서드 → Exchange API
* 사용법
  * HttpEntity 객체에 원하는 값과 헤더 설정을 저장
  * exchange 메서드에 필요한 HTTP 메서드를 지정하여 헤더값도 같이 해당 URL로 보냄
  * 결과를 화면에 출력
* exchange로 객체 컬렉션을 받아보기
  * ResponseEntity와 ParameterizedTypeReference 객체
  * ParameterizedTypeReference
    * 응답을 Class 대신 제네릭한 타입을 지정할 수 있다
    * ex. 응답을 List 타입으로 지정 가능
    * abstract 클래스여서 바로 사용하기 위해 익명 인라인 클래스를 사용

### 🍖 optionsForAllow\(\)

* 해당 URI에서 지원하는 HTTP 메서드를 조회

### 🕰 Timeout 설정하기

* connection에 대한 설정을 추가로 할 수 있다
* 간단한 설정은 SimpleClientHttpRequestFactory
  * But… 많은 기능을 제공하지 않아 실제 환경에서는 더 많은 기능\(ex. retry\)을 제공하는 별도의 HTTP Client 라이브러리를 사용
* 아파치의 httpClient를 추가하여 Read timeout을 설정
  * ClientHttpRequestFactory 객체에 connection과 read에 대한 timeout을 설정 \(ex. 60초\)
  * controller에서는 일정 시간동안 sleep을 해서 결과를 설정 시간 내 반환하지 않음
    * ResourceAccessException이 발생
    * Read timed out을 출력하는 것을 확인할 수 있다

### 🍖 patchForObject\(\)

* 주어진 URL 주소로 HTTP Patch 메서드를 실행
* HTTP에서 리소스를 생성하는 메서드 3가지
* POST
  * 리소스의 위치를 지정하지 않는 방식으로 리소스를 생성하는 연산
  * idempotent 하지 않다
    * 연산을 반복해도 같은 값이 나온다는 개념
  * 매번 실행할 때마다 다른곳에 새로운 리소스가 생성됨
    * ex. /item/2, /item/3
* PUT
  * 명확한 리소스의 위치에 사용되며 리소스의 생성이나 업데이트를 위해 사용하는 연산
  * idempotent하다
* PATCH
  * PUT과 같이 정해진 리소스의 위치에 사용되지만
    * 모든 파라미터를 업데이트하기 보다는 부분적인 데이터만 업데이트
* JDK HttpURLConnection에서는 PATCH 메서드를 지원하지 X
  * 기본 HttpURLConnection 대신 Patch 메서드를 지원하는 아파치의 HttpComponents를 사용

### 🍖 Execute\(\)

* 콜백을 통해 요청 준비와 응답 추출을 완벽하게 제어하여 요청을 수행하는 가장 일반적인 메서드
* 지금까지 언급한 getForObject, put 메서드 등은 내부적으로 execute 메서드를 호출하게 되어 있다
* URL 주소로 요청을 보내기 전 
  * 요청 콜백은 인자로 넘겨준 객체를 body에 세팅하고 헤더 값들도 설정

🏷 END

