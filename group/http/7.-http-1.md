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

