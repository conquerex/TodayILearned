# Broken pipe 이슈

이런 경우가 있다. 서버에 엑셀 업로드를 한다고 했을 때 업로드 요청 후 블락을 하지 않으면 응답이 오기 전에 재요청을 하는 경우. 엑셀 파일을 선택해놓고 여러번 업로드 버튼을 누르는 경우가 그렇다. 이런 경우 아래와 같은 Exception을 볼 수 있다.

```text
org.apache.catalina.connector.ClientAbortException: java.io.IOException: Broken pipe
Caused by: java.io.IOException: Broken pipe
```

위에서 보면 IOException으로 출력된다. 같은 `Broken pipe 이슈`라도 다른 발생 원인으로 구분되는 Exception이 나타날 수 있다.

* java.net.SocketException: Broken pipe
  * 잦은 입출력이 원인. 요청 처리가 끝나기도 전에 새로고침이나 등록 버튼을 연속으로 눌러 재요청을 여러번 보내는 경우가 그렇다. 이경우 소켓이 끊어지면서 본 Exception이 발생하게 된다.
  * 웹브라우저에서 서버를 연결하고, 이때 accept된 소켓을 HttpThread로 넘긴다. 해당 소켓이 ThreadPool에 들어오고 이 HttpThread로 수행된다.
  * 이 때 HttpThread가 완료되지 전에 재요청을 하면 어떻게 될까? 처음 요청 당시 생성된 소켓의 자원을 HttpThread.run에서 수행이 완료되기도 전에 두번째 요청이 들어와서 첫번째 수행이 종료가 된다. 당연히 이 때 소켓은 끊어진다. 그래서 **Broken pipe**라는 메세지가 나타나는 것이다.
* java.io.IOException: Broken pipe
  * Receiver\(클라이언트 역\)가 받은 응답 데이터를 적절한 타이밍에 처리하지 못하면 발생하는 Exception이다. 네트워크가 느리거나 서버의 CPU 이슈로 느리게 처리되는 경우가 그렇다.
  * 응답을 받기 전에 화면을 이동하고나 종료하면 이런 상황이 생긴다.

위에서 HttpThread라고 언급했는데 실제로는 http 요청을 처리하기 위한 스레드로 이해하면 된다. 생성된 소켓을 쓰레드풀에서 순서대로 처리하는데 쓰레드에 처리되기 전 혹은 처리 중에 다음 요청이 들어올 수 있다. 그러면 생성된 소켓이 끊어지고 두번째 요청의 소켓이 생성되므로 Exception이 나타난다.

위 두가지 케이스에는 공통점이 있다. 두 소켓상의 통신에서 소켓을 담당하는 프로세서\(혹은 스레드\)가 어떤 이상이 생겨서 종료가 된 상황이다. 상대 소켓은 이를 알지 못해서 데이터를 전송하려는데 문제가 생긴 상황이라는 점이 공통점이다. Time out과 같은 시간 이슈로, 일정 시간 내 응답을 받을 클라이언트를 찾지 못해 해당 Exception이 발생할 수도 있다.

어느 정도 원인은 파악했으니 해결점을 찾아보자.

1. 클라이언트가 요청\(Request\)을 했다면 응답\(Response\)이 올 때까지 기다리도록 하는 것
2. Exception 무시하는 방법 : 클라이언트가 비정상적인 종료한 것을 서버에서는 제어할 수 없기 때문 \(But 권장 X\)
3. 중복 요청 확인 후 Block : 연속적인 요청을 막을 수 있
4. Timeout값을 늘리
5. 가용 스레드 늘리기

