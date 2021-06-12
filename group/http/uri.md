# URI와 웹 브라우저 요청 흐름

### 강의항목

* URI
* 웹 브라우저 요청 흐름



### 학습 내용

* URI
  * Uniform Resource Identifier
  * 로케이터, 이름 또는 둘 다 추가로 분류될 수 있다.
  * URL\(Resource Locator\) + URN\(Resource Name\)
  * scheme, authority, path, query, fragment 등으로 구성
  * 대부분 URL로 쓰고 있다
* Uniform
  * 리소스 식별하는 통일된 방식
* Resource
  * 자원, URI로 식별할 수 있는 모든 것 \(제한 없음\)
* Identifier
  * 다른 항목과 구분하는데 필요한 정보
* URL
  * Locator, 리소스가 있는 위치를 지정
* URN
  * Name, 리소스에 이름을 부여
  * 위치는 병할 수 있지만, 이름은 변하지 않는다.
  * 예\) ISBN
  * URN 이름만으로 실제 리소스를 찾을 수 있는 방법이 보편화 되지 않음
  * 대충\(ㅋ\) URI를 URL과 같은 의미로 이야기 함
* URL
  * scheme://\[userinfo@\]host\[:port\]\[/path\]\[?query\]\[\#fragment\]
  * https://www.google.com:443/search?q=hello&hl=ko
  * 프로토콜 : http
    * 어떤 방식으로 자원에 접근할 것인가 하는 약속 규칙
  * userinfo
    * URL에 사용자 정보를 포함해서 인증
    * 거의 사용하지 않음
  * 호스트명 : www.google.com
    * 도메인명 또는 IP 주소를 직접 사용가능
  * 포트 번호 : 443
    * 접속 포트, 일반적으로 생략
  * 패스 : /search
    * 계층적 구조
  * 쿼리 파라미터 : q=hello&hl=ko
    * key=value 형태
    * ?로 시작, &로 추가 가능
    * query parameter, query string등으로 불림
    * 웹서버에 제공하는 파라미터, 문자 형태
  * fragment
    * html 내부 북마크 등에 사용
    * 서버에 전송하는 정보는 아님
* 웹 브라우저 요청 흐름



