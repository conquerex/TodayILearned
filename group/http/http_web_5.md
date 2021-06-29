# 섹션 5. HTTP 메서드 활용

### 강의항목

* 클라이언트에서 서버로 데이터 전송
* HTTP API 설계 예시

### 학습내용

* 클라이언트에서 서버로 데이터 전송
  * 쿼리 파라미터를 통한 데이터 전송
    * GET
    * 주로 정렬 필터\(검색어\)
  * 메시지 바디를 통한 데이터 전송
    * POST, PUT, PATCH
    * 회원가입, 상품 주문, 리소스 등록 및 변경
* 데이터 전송의 4가지 상황
  * \(간단한 것부터\) 정적 데이터 조회
    * 이미지, 정적 텍스트 문서
    * 조회는 GET 사용
    * 정적 데이터는 일반적으로 쿼리 파라미터 없이 리소스 경로로 단순하게 조회 가능
  * 동적 데이터 조회
    * 주로 검색, 게시판 목록에서 정렬 필터 \(검색어\)
    * 조회 조건을 줄여주는 필터, 정렬하는 정렬 조건에 주로 사용
    * GET - 쿼리 파라미터 사용해서 데이터 전달
  * HTML Form 데이터 전송
    * 만들고 싶은건 화면
    * POST 전송 - 저장 : 웹브라우저가 요청 HTTP 메시지를 생성
      * HTTP 바디에 key-value 값이 들어감
    * GET 전송 - 조회
    * HTML From submit시 POST 전송
      * 예\) 회원가입, 상품 주문 등
    * Content-Type : application/x-www-form-urlencoded 사용
      * form의 내용을 메시지 바디를 통해 전송
      * key-value, 쿼리 파라미터 형식
      * 전송 데이터를 url encoding 처리
      * 예\) abc김 --&gt; abc%EA%B9%80
    * HTML Form은 GET 전송도 가능
    * Content-type : multipart/form-data
      * 파일 업로드같은 바이너리 데이터 전송시
      * 다른 종류의 여러 파일과 폼의 내용 함께 전송 가능
    * HTML Form 전송은 GET, POST만 지원
  * HTTP API 데이터 전송
    * 서버 to 서버 \(백엔드 시스템 통신\)
    * 앱 클라이언트
    * 웹 클라이언
      * HTML에서 Form 전송 대신 자바 스크립트를 통한 통신에 사용\(AJAX\)
      * 예\) React, Vue.js 같은 웹 클라이언트와 API 통신
    * POST, PUT, PATCH : 메시지 바디를 통해 데이터 전송
    * GET : 조회, 쿼리 파라미터로 데이터 전달
    * Content-Type : application/json을 주로 사용 \(사실상 표준\)
      * Text, XML, JSON 등등
* HTTP API 설계 예시
  * HTTP API - 컬렉션
    * **대부분은 컬렉션을 사용한다**
    * Post 기반 등록
    * 예\) 회원 관리 API 제공
  * HTTP API - 스토어
    * Put 기반 등록
    * 예\) 정적 컨텐츠 관리, 원격 파일 관리
  * HTML Form 사용
    * 웹페이지 회원 관리
    * 예\) GET, POST만 지원
* 회원 관리 시스템
  * API 설계 : POST 기반 등록
  * 목록 : GET /members
  * **등록 : POST /members**
  * 조회 : GET /members/{id}
  * 수정 : **PATCH**, PUT, POST /members/{id}
    * 게시글 수정이면 PUT을 쓸 수 있다
  * 삭제 : DELETE /members/{id}
* POST : 신규 자원 등록 특징
  * 클라이언트
    * 등록될 리소스의 URI를 모른
    * 예\) members/100인지 members/101인지
  * 서버가 새로 등록된 리소스 URI를 생성해준다.
    * HTTP/1.1 201 Created Location: **/members/100**
  * 컬렉션\(Collection\)
    * 서버가 관리하는 디렉토리
    * 서버가 리소스의 URI를 생성하고 관리
    * 여기서 컬렉션은 /members
* 파일 관리 시스템
  * API 설계 : PUT 기반 등록
  * 목록 : GET /files
  * 조회 : GET /files/{filename}
  * **등록 : PUT /files/{filename}**
  * 삭제 : DELETE /files/{filename}
  * 대량 등록 : POST /files
* PUT : 신규 자원 등록 특징
  * 클라이언트가 리소스 URI를 알고 있어야 한다
  * 파일 등록 : PUT /files/flower.jpg
  * 클라이언트가 직접 리소스의 URI를 지정한다
  * 스토어\(Store\)
    * 클라이언트가 관리하는 리소스 저장소
    * 클라이언트가 리소스의 URI를 알고 관리
    * 여기서 스토어는 /files
* HTML FORM 사용
  * GET, POST만 지원
    * AJAX 같은 기술을 사용해서 해결 가능
  * 순수 HTML 그리고 HTML FORM만을 고려
    * GET, POST만 지원해서 제약이 있음
  * 목록 : GET /members
  * 등록 폼 : GET /members/new
  * 등록 : POST **/members/new**, /members
  * 조회 : GET /members/{id}
  * 수정 폼 : GET /members/{id}/edit
  * 수정 : POST **/members/{id}/edit**, /members/{id}
  * 삭제: POST /members/{id}/delete
    * DELETE는 사용 못함
* 컨트롤 URI
  * GET, POST 제약 해결을 위해 동사로 된 리소스 경로 사용
  * POST의 /new, /edit, /delete가 컨트롤 URI
  * HTTP 메서드로 해결하기 애매한 경우 사용\(HTTP API 포함\)
* 참고하면 좋은 URI 설계 개념
  * [https://restfulapi.net/resource-naming/](https://restfulapi.net/resource-naming/)
  * 문서 \(document\)
    * 단일 개념 \(파일 1개, 객체 인스턴스, DB row\)
    * /members/100, /files/flower.jpg
  * 컬렉션
    * 서버가 관리하는 리소스 디렉터리
    * 서버가 리소스의 URI를 생성하고 관리
    * /members
  * 스토어
    * 클라이언트가 관리하는 자원 저장소
    * 클라이언트가 리소스의 URI를 알고 관리
    * 게시판에서도 쓰임
    * /files
  * 컨트롤러\(Controller\), 컨트롤 URI
    * 문서, 컬렉션, 스토어로 해결하기 어려운 추가 프로세스 실행
    * 동사를 직접 사용
    * /members/{id}/delete

