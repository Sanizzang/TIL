### Socket이란?

- 소켓은 네트워크에서 실행되는 두 프로그램 간의 양방향 통신 링크의 양 끝점을 이야기 한다.
- 소켓 클래스는 클라이언트 프로그램과 서버 프로그램 같의 연결을 나타내는 데 사용되는데,
  java.net 패키지는 연결의 클라이언트 측과 연결의 서버 측을 각각 구현하는 두 개의 클래스
  (Socket 및 ServerSocket)를 제공한다.

### Java Socket Class

- 생성자

  - public Socket(InetAddress address, int port)
    - IP주소를 나타내는 InetAddress 객체와 포트 번호로 소켓 객체를 생성한다.
  - public Socket(String host, int port)
    - 호스트명(IP주소), 포트번호로 소켓 객체를 생성한다.

- 소켓을 생성하면 연결을 자동으로 이루어지며 연결 시 오류가 발생하면 IOException이 발생한다.
- TCP 연결에는 로컬IP 주소와 원격IP 주소, 로컬포트, 원격 포트등이 사용된다.
- Socket이 생성되어 원격호스트에 접속 할 때 로컬의 포트는 대개 사용하지 않는 로컬 포트가 사용된다.
