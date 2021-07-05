# 섹션 7. HTTP 헤더1 - 일반 헤더

### 강의항목

* HTTP 헤더 개요
* 표현
* 콘텐츠 협상
* 전송 방식
* 일반 정보
* 특별한 정보
* 인증
* 쿠키

### 학습내용

* HTTP 헤더 용도
  * HTTP 전송에 필요한 모든 부가정보
    * 예\) 메시지 바디의 내용/크기, 압축, 인증, 요청 클라이언트, 서버 정보, 캐시 관리 정보 등
  * 표준 헤더가 너무 많음
  * 필요시 임의의 헤더 추가 가능
    * 예\) helloworld: korea
* 분류 - RFC2626\(과거\)
  * General 헤더 : 메시지 전체에 적용되는 정보
  * Request 헤더 : 요청 정보
    * 예\) User-Agent: Mozilla/5.0 \(Macintosh; ..\)
  * Response 헤더 : 응답 정보
    * 예\) Server: Apache
  * Entity 헤더 : 엔티티 바디 정보
    * 예\) Content-Type: text/html, Content-Length: 4354
  * HTTP Body
    * 메세지 본문\(Message body\)
      * 엔티티 본문\(Entity body\)을 전달하는데 사용
    * 엔티티 본문 : 요청이나 응답에서 전달할 실제 데이터
    * 엔티티 헤더 : 엔티티 본문의 데이터를 해석할 수 있는 정보 제공
      * 데이터 유형\(html, json\), 데이터 길이, 압축 정보 등
* 그런데... HTTP 표준은
  * 1999년 RFC2616 : 폐기됨
  * 2014년 RFC7230~7235 등장 \(Entity body 개념이 사라짐\)
* RFC723x 변화
  * Entity --&gt; Representation \(표현, REST에서 R\)
  * Representation = Representation metadata + Representation data
    * 표현 = 표현 메타데이타 + 표현 데이타
  * HTTP Body \(RFC7230, 최신\)
    * 메시지 본문\(Message body\)을 통해 표현데이터 전달
    * 메시지 본문 = 페이로드\(Payload\)
    * 표현 : 요청이나 응답에서 전달할 실제 데이터
    * 표현 헤더
      * 표현 데이터를 해석할 수 있는 정보 제공
      * 데이터 유형\(html, json\), 데이터 길이, 압축 정보 등
    * 참고 : 표현 헤더는 표현 메타데이터와 페이로드\(메시지 본문\)를 구분해야 한다.
* 표현
  * Content-Type : 표현 데이터의 형식
    * 미디어 타입, 문자 인코딩
    * 예\) text/html; charset=utf-8
    * 예\) application/json \(기본이 utf-8\)
    * 예\) image/png
  * Content-Encoding : 표현 데이터의 압축 방식
    * 표현 데이터를 압축하기 위해 사용
    * 데이터를 전달하는 곳에서 압축 후 인코딩 헤더 추가
    * 데이터를 읽는 쪽에서 인코딩 헤더의 정보로 압축 해
    * 예\) gzip, deflate, identity\(똑같다. 압축 안함\)
    * 예\) Content-Encoding: gzip
  * Content-Language : 표현 데이터의 자연 언어 \(한국어, 영어 등\)
    * 예\) ko, en, en-US
  * Content-Length : 표현 데이터의 길이 \(명확하게는 페이로드 헤더\)
    * 바이트 단위
    * Transfer-Encoding\(전송 코딩\)을 사용하면 Content-Length를 사용하면 안됨
  * 표현 헤더는 전송, 응답 둘 다 사용
* 협상 \(콘텐츠 네고시에이션\)
  * 클라이언트가 선호하는 표현 요청
  * Accept: 클라이언트가 선호하는 미디어 타입 전달
    * 클라이언트가 저런 타입을 달라고 하는 것
  * Accept-Charset
  * Accept-Encoding
  * Accept-Language
  * 협상 헤더는 요청시에만 사용
* Accept-Language 적용 전
  * 한국어 브라우저 사용
  * GET /event
  * 다중 언어 지원 서버
    * 기본 : 영어
    * 한국어 지원
  * 영어로 응답
* Accept-Language 적용 전
  * GET /event
    * Accept-Language: ko
  * 한국어로 응답
* Accept-Language 복잡한 예시
  * 한국어 브라우저
  * GET /event
    * Accept-Language: ko
  * 다중 언어 지원 서버
    * 기본 : 독일어
    * 영어 지원
  * 한국어는 옵션에 없네????
* 협상과 우선 순
  * Quality Values\(q\)
  * 0~1, 클수록 높은 우선순위
  * 생략하면 1
  * accept-language: ko-KR,ko;q=0.9,en-US;q=0.8,en;q=0.7
  * 구체적인 것이 우선한
    * Accept: text/\*, text/plain, text/plain;format-flowed, \*/\*
    * 1순위 : text/plain;format-flowed
    * 4순위 : \*/\*
  * 구체적인 것을 기준으로 미디어 타입을 맞춘다.
    * Accep: text/\*;q=0.3, text/html;q=0.7, text/html;level=1
    * text/html;level=1
      * q : 1
    * text/html
      * q : 0.7
    * text/plain
      * q : 0.3 \("text/\*"와 가장 가깝기 때문\)

![](../../.gitbook/assets/2021-07-01-9.00.51.png)



* 전송 방식
  * 단순 전송
    * Content-Length를 알 수 있을 때
  * 압축 전송
    * Content-Enoding을 알 수 있을 때
  * 분할 전송
    * Transfer-Encoding을 알 수 있을 때
    * 예\) Transfer-Encoding: chunked \(쪼개다는 의미\)
    * Content-Length를 보내면 안됨
  * 범위 전송
    * Range, Content-Range
    * 요청 예\) Range: bytes=1001-2000
    * 응답 예\) Content-Range: bytes 1001-2000/2000
* 일반 정보
  * From : 유저 에이전트의 이메일 정보
    * 요청에서 사용
    * 검색엔진 같은 곳에서 주로 사용 \(일반적으로 사용 X\)
    * \[ 잘 이해가 안감 ㅋ \]
  * Referer : 이전 웹 페이지 주
    * 현재 요청된 페이지의 이전 웹 페이지 주소
    * A -&gt; B로 이동하는 경우, B를 요청할 때 Referer: A를 포함해서 요청
    * 유입 경로 분석 가능
    * 요청서 사용
    * 참고 : referer는 단어 referrer의 오타 \(ㅋㅋㅋㅋ\)
  * User-Agent : 유저 에이전트 애플리케이션 정보
    * user-agent: Mozilla/5.0 \(iPhone; CPU iPhone OS 13\_2\_3 like Mac OS X\) AppleWebKit/605.1.15 \(KHTML, like Gecko\) Version/13.0.3 Mobile/15E148 Safari/604.1
    * 클라이언트의 애플리케이션 정보 \(베 브라우저 정보 등\)
    * 통계 정보
    * 어떤 종류의 브라우저에서 장애가 발생하는지 파악 가능
    * 요청에 사용
  * Server : 요청을 처리하는 ORIGIN 서버의 소프트웨어 정보
    * ORIGIN 서버 : 응답을 해주는 진짜 서버, 노드 서버는 아님
    * Server: Apache/2.2.22 \(Debian\)
    * server: nginx
    * 응답에서 사용
  * Date : 메시지가 발생한 날짜와 시간
    * 응답에서 사용
* 특별한 정보
  * Host : 요청한 호스트 정보 \(도메인\)
    * 필수!!!
    * 요청에서 사용
    * 하나의 IP주소에 여러 도메인이 적용되어 있을 때
      * 하나의 서버가 여러 도메인을 처리해야 할 때
    * 가상 호스트를 통해 여러 도메인을 한번에 처리할수 있는 서버가 있다
      * 실제 애플리케이션이 여러개 구동될 수 있다
      * GET /hello
      * Host: aaa.com
    * TCP/IP는 IP로만 통신, 호스트\(도메인\)를 넣음으로서 원하는 곳으로 도달
  * Location : 페이지 리다이렉션
    * 웹 브라우저는 3xx 응답의 결과에 Location 헤더가 있으면
      * Location 위치로 자동 이동 \(리다이렉트\)
    * 응답코드 3xx에서 설명
    * 201 \(Created\): Location 값은 요청에 의해 생성된 리소스 URI
    * 3xx \(Redirction\): Location 값은 요청을 자동으로 리다이렉션하기 위한 대상 리소스를 가리킴
  * Allow : 허용 가능한 HTTP 메서드
    * 405\(Method Not Allowed\)에서 응답에 포함해야 함
    * 많이 사용은 안함
    * 예\) Allow: GET, HEAD, PUT
  * Retry-After : 유저 에이전트가 다음 요청을 하기까지 기다려야 하는 시간
    * 503\(Service Unavailable\): 서비스가 언제까지 불능인지 알려줄 수 있
    * 날짜 표기 혹은 초단위 표기
* 인증
  * Authorization
    * 클라이언트 인증 정보를 서버에 전달
    * Authorization: Basic xxxxxxxxxxxxxxxxxxxxxxx
    * 메커니즘마다 value가 다르다 \(예. OAuth\)
  * WWW-Authenticate
    * 리소스 접근시 필요한 인증 방법 정의
    * 401 Unauthorized 응답과 함께 사용
    * 인증을 하려면 아래의 정보를 참고해서 제대로 된 정보를 만들라고 알려줌
      * 공식 스펙에서 가지고 온 예\)WWW-Authenticate: Newauth realm="apps", type=1, title="Login to \"apps\"", Basic realm="simple"
* 쿠키
  * Set-Cookie : 서버에서 클라이언트로 쿠키 전달\(응답\)
  * Cookie : 클라이언트가 서버에서 받은 쿠키를 저장하고, HTTP 요청시 서버로 전달
* 쿠키 미사용
  * HTTP는 무상태\(Stateless\) 프로토콜이다
  * 클라이언트와 서버가 요청과 응답을 주고 받으면 연결이 끊어진다
  * 클라이언트가 다시 요청하면 서버는 이전 요청을 기억하지 못한다
  * 클라이언트와 서버는 서로 상태를 유지하지 않는다
* 대안 - 모든 요청에 사용자 정보 포함
  * 브라우저를 완전히 종류 후 다시 열면?
* 쿠키 - 로그인
  * 웹브라우저 : POST /login, user=홍길동
  * 서버 : 200 OK, Set-Cookie: user=홍길동 \(홍길동님이 로그인했습니다.\)
  * 쿠키 저장소 : user=홍길동 저장
  * 로그인 후 welcome 페이지 접근
  * GET /welcome, Cookie: user=홍길동
    * 쿠키 저장소에서 조회
  * 200 OK \(안녕하세요 홍길동님\)
  * 모든 요청에 쿠키 정보 자동 포함
* 사용처
  * 사용자 로그인 세션 관리 \(세션 ID\)
  * 광고 정보 트래킹
* 쿠키 정보는 항상 서버에 전송
  * 네트워크 트래픽 추가 유발
  * 그래서 최소한의 정보만 사용 \(세션 ID, 인증 토큰\)
  * 서버에 전송하지 않고 웹브라우저 내부에 데이터를 저장하고 싶으면
    * 웹 스토리지\(localStorage, sessionStorage\) 참고
* 주의!!!!
  * 보안에 민감한 데이터는 저장하면 안됨 \(주민번호, 신용카드 번호 등\)
* 쿠키 - 생명주기 \(Expires, max-age\)
  * Set-Cookie: **expires**= Sat, 26-Dec-2020 04:33:22 GMT
    * 만료일이 되면 쿠키 삭제
  * Set-Cookie: max-age=3600 \(3600초\)
    * 0이나 음수를 지정하면 쿠키 삭제
  * 세션 쿠키 : 만료 날짜를 생략하면 브라우저 종료시 까지만
  * 영속 쿠키 : 만료 날짜를 입력하면 해당 날짜까지
* 쿠키 - 도메인
  * 예\) domain=example.org
  * 명시 : 명시한 문서 기준 도메인 + 서브 도메인 포함
    * 위 예시를 지정해서 쿠키 생성
    * example.org는 물론
    * dev.example.org도 쿠키 접근
  * 생략 : 현재 문서 기준 도메인만
    * example.org에서 쿠키를 생성하고 도메인 지정 생략
    * example.org에서만 쿠키 접근
    * dev.example.org는 쿠키 미접근
* 쿠키 - 경로
  * 예\) path=/home
  * 이 경로를 포한한 하위 경로 페이지만 쿠키 접근
  * 일반적으로 path=/ \(루트로 지정\)
    * path=/home 지정
    * /home, home/level1 --&gt; 가능
    * /away --&gt; 불가능
* 쿠키 - 보안
  * Secure
    * 쿠키는 http, https를 구분하지 않고 전송
    * Secure를 적용하면 https인 경우에만 전송
  * HttpOnly
    * XSS 공격 방지
    * 자바스크립트에서 접근 불가\(document.cookie\)
    * HTTP 전송에만 사용
  * SameSite
    * XSRF 공격 방지
    * 요청 도메인과 쿠키에 설정된 도메인이 같은 경우만 쿠키 전송
    * 기능 적용된지 얼마 안되서 브라우저 적용 가능 범위 체크 필요



