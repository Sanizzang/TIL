### URL

- scheme://[userinfo@]host[:port][/path][?query][#fragment]
- https://www.google.com:443/search?q=hello&hl=ko

#### scheme

- 주로 프로토콜 사용
- 프로토콜: 어떤 방식으로 자원에 접근할 것인가 하는 약속 규칙
  - 예) http, https, ftp 등등
- http는 80포트, https는 443 포트를 주로 사용, 포트는 생략 가능
- https는 http에 보안 추가 (HTTP Secure)

#### userinfo

- URL에 사용자 정보를 포함해서 인증
- 거의 사용하지 않음

#### host

- 호스트명
- 도메인명 또는 IP 주소를 직접 사용가능

#### port

- 포트(PORT)
- 접속 포트
- 일반적으로 생략, 생략시 http는 80, https는 443

#### path

- 리소스 경로(path), 계층적 구조
- 예)
  - /home/file1.jpg
  - /members
  - /members/100, /items/iphone12

#### query

- key=value 형태
- ?로 시작, &로 추가 가능 ?keyA=valueA&keyB=valueB
- query parameter, query string 등으로 불림, 웹서버에 제공하는 파라미터, 문자 형태

#### fragment

- fragment
- html 내부 북마크 등에 사용
- 서버에 전송하는 정보 아님

### HTTP 메시지 전송

1. 웹 브라우저가 HTTP 메시지 생성
2. SOCKET 라이브러리를 통해 전달

- A: TCP/IP 연결(IP, PORT)
- B: 데이터 전달

3. TCP/IP 패킷 생성, HTTP 메시지 포함

### HTTP 역사

- HTTP/0.9 1991년: GET 메서드만 지원, HTTP 헤더X
- HTTP/1.0 1996년: 메서드, 헤더 추가
- HTTP/1.1 1997년: 가장 많이 사용, 우리에게 가장 중요한 버전
  - RFC2068 (1997) -> RFC2616 (1999) -> RFC7230~7235 (2014)
- HTTP/2 2015년: 성능 개선
- HTTP/3 진행중: TCP 대신에 UDP 사용, 성능 개선

#### 기반 프로토콜

- TCP: HTTP/1.1, HTTP/2
- UDP: HTTP/3
- 현재 HTTP/1.1 주로 사용
  - HTTP/2, HTTP/3 도 점점 증가

### HTTP 특징

- 클라이언트 서버 구조
- 무상태 프로토콜(stateless), 비연결성
- HTTP 메시지
- 단순함, 확장 가능

### 클라이언트 서버 구조

- Request Response 구조
- 클라이언트는 서버에 요청을 보내고, 응답을 대기
- 서버가 요청에 대한 결과를 만들어서 응답

### 무상태 프로토콜 (stateless)

- 서버가 클라이언트의 상태를 보존X
- 장점: 서버 확장성 높음(스케일 아웃)
- 단점: 클라이언트가 추가 데이터 전송

### Stateless의 한계

- 모든 것을 무상태로 설계할 수 있는 경우도 있고 없는 경우도 있다.
- 무상태
  - 예) 로그인이 필요 없는 단순한 서비스 소개 화면
- 상태 유지
  - 예) 로그인
- 로그인한 사용자의 경우 로그인 했다는 상태를 서버에 유지
- 일반적으로 브라우저 쿠키와 서버 세션등을 사용해서 상태 유지
- 상태 유지는 최소한만 사용

### 비 연결성

- HTTP는 기본이 연결을 유지하지 않는 모델
- 일반적으로 초 단위의 이하의 빠른 속도로 응답
- 1시간 동안 수천명이 서비스를 사용해도 실제 서버에서 동시에 처리하는 요청은 수십개 이하로 매우 작음
  - 예) 웹 브라우저에서 계속 연속해서 검색 버튼을 누르지는 않는다.
- 서버 자원을 매우 효율적으로 사용할 수 있음

### 비 연결성(한계와 극복)

- TCP/IP 연결을 새로 맺어야 함 - 3way handshake 시간 추가
- 웹 브라우저로 사이트를 요청하면 HTML 뿐만 아니라 자바스크립트, css, 추가 이미지 등등 수 많은 자원이 함께 다운로드
- 지금은 HTTP 지속 연결(Persistent Connections)로 문제 해결
- HTTP/2, HTTP/3에서 더 많은 최적화

#### HTTP 메시지 구조

- start-line 시작 라인
- header 헤더
- empty line 공백 라인 (CRLF)
- message body

### 요청 메시지

```
GET /search?q=hello&hl=ko HTTP/1.1
Host: www.google.com
```

#### 시작라인 (HTTP 메서드)

- GET

- 종류: GET, POST, PUT, DELETE ...
- 서버가 수행해야 할 동작 지정
  - GET: 리소스 조회
  - POST: 요청 내역 처리

#### 시작라인 (요청 대상)

- /search?q=hello&hl=ko

- absolute-path[?query] (절대경로[?쿼리])
- 절대경로 = "/"로 시작하는 경로
- 참고: \*, http://...?x=y 와 같이 다른 유형의 경로지정 방법도 있다.

#### 시작라인 (HTTP 버전)

- HTTP/1.1

- HTTP Version

### 응답 메시지

```
HTTP/1.1 200 OK
Content-Type: text/html;charset=UTF-8
Content-Length: 3423

<html>
  <body>...</body>
</html>
```

#### 시작라인

- HTTP/1.1 200 OK

- HTTP 버전
- HTTP 상태 코드: 요청 성공, 실패를 나타냄
  - 200: 성공
  - 400: 클라이언트 요청 오류
  - 500: 서버 내부 오류
- 이유 문구: 사람이 이해할 수 있는 짧은 상태 코드 설명 글

#### HTTP 헤더 (용도)

- HTTP 전송에 필요한 모든 부가 정보
- 예) 메시지 바디의 내용, 메시지 바디의 크기, 압축, 인증, 요청 클라이언트(브라우저) 정보, 서버 애플리케이션 저옵, 캐시 관리 정보...
- 표준 헤더가 너무 많음
- 필요시 임의의 헤더 추가 가능
  - helloworld: hihi

#### HTTP 메시지 바디 (용도)

- 실제 전송할 데이터
- HTML 문서, 이미지, 영상, JSON 등등 byte로 표현할 수 있는 모든 데이터 전송 가능

## HTTP 메서드

### 리소스와 행위를 분리(가장 중요한 것은 리소스를 식별하는 것)

- URI는 "리소스만" 식별
- 리소스와 해당 리소스를 대상으로 하는 행위를 분리
  - 리소스: 회원
  - 행위: 조회, 등록, 삭제, 변경
- 리소스는 명사, 행위는 동사
- 행위(메서드)는 어떻게 구분?

### GET

```
GET /search?q=hello&hl=ko HTTP/1.1
Host: www.google.com
```

- 리소스 조회
- 서버에 전달하고 싶은 데이터는 query(쿼리 파라미터, 쿼리 스트링)를 통해서 전달
- 메시지 바디를 사용해서 데이터를 전달할 수 있지만, 지원히지 않는 곳이 많아서 권장하지 않음

### POST

```
POST /members HTTP/1.1
Content-Type: application/json

{
  "username": "hello",
  "age": 20
}
```

- 요청 데이터 처리
- 메시지 바디를 통해 서버로 요청 데이터 전달
- 서버는 요청 데이터를 처리
  - 메시지 바디를 통해 들어온 데이터를 처리하는 모든 기능을 수행한다.
- 주로 전달된 데이터로 신규 리소스 등록, 프로세스 처리에 사용

### POST (정리)

1. 새 리소스 생성(등록)

- 서버가 아직 식별하지 않은 새 리소스 생성

2. 요청 데이터 처리

- 단순히 데이터를 생성하거나, 변경하는 것을 넘어서 프로세스를 처리해야 하는 경우
- ex) 주문에서 결제완료 -> 배달시작 -> 배달완료 처럼 단순히 값 변경을 넘어 프로세스의 상태가 변경되는 경우
- POST의 결과를 새로운 리소스가 생성되지 않을 수도 있음
- ex) POST /orders/{orderId}/start-delivery (컨트롤 URI)

3. 다른 메서드로 처리하기 애매한 경우

- ex) JSON으로 조회 데이터를 넘겨야 하는데, GET 메서드를 사용하기 어려운 경우
- 애매하면 POST

### PUT

```
PUT /members/100 HTTP/1.1
Content-Type: application/json

{
  "username": "hello",
  "age": 20
}
```

- 리소스를 대체
  - 리소스가 있으면 대체
  - 리소스가 없으면 생성
  - 쉽게 이야기해서 덮어버림
- 중요! 클라이언트가 리소스를 식별
  - 클라이언트가 리소스 위치를 알고 URI 지정
  - POST와 차이점

### PATCh

```
PATCH /members/100 HTTP/1.1
Content-Type: application/json

{
  "age": 50
}
```

- 리소스 부분 변경

### DELETE

```
DELETE /members/100 HTTP/1.1
Host: localhost:8080
```

- 리소스 제거

## HTTP 메서드의 속성

- 안전
- 멱등
- 캐시가능

#### 안전 (Safe)

- 호출해도 리소스를 변경하지 않는다.

#### 멱등 (Indempotent)

- f(f(x)) = f(x)
- 한 번 호출하든 두 번 호출하든 100번 호출하든 결과가 똑같다.
- 멱등 메서드
  - GET: 한 번 조회하든, 두 번 조회하든 같은 결과가 조회된다.
  - PUT: 결과를 대체한다. 따라서 같은 요청을 여러번 해도 최종 결과는 같다.
  - DELETE: 결과를 삭제한다. 같은 요청을 여러번 해도 삭제된 결과는 똑같다.
  - POST: 멱등이 아니다. 두 번 호출하면 같은 결제가 중복해서 발생할 수 있다.
- 활용
  - 자동 복구 메커니증
  - 서버가 TIMEOUT 등으로 정상 응답을 못주었을 때, 클라리언트가 같은 요청을 다시 해도 되는가? 판단 근거
- 멱등은 외부 요인으로 중간에 리소스가 변경되는 것 까지는 고려하지는 않는다.

#### 캐시가능 (Cacheable)

- 응답 결과 리소스를 캐시해서 사용해도 되는가?
- GET, HEAD, POST, PATCH 캐시가능
- 실제로는 GET, HEAD 정도만 캐시로 사용
  - POST, PATCH는 본문 내용까지 캐시 키로 고려해야 하는데, 구현이 쉽지 않음

### 클라이언트에서 서버로 데이터 전송 (데이터 전달 방식은 크게 2가지)

- 쿼리 파라미터를 통한 데이터 전송
  - GET
  - 주로 정렬 필터(검색어)
- 메시지 바디를 통한 데이터 전송
  - POST, PUT, PATCH
  - 회원 가입, 상품 주문, 리소스 등록, 리소스 변경

### 클라이언트에서 서버로 데이터 전송 (4가지 상황)

- 정적 데이터 조회
  - 이미지, 정적 텍스트 문서
- 동적 데이터 조회
  - 주로 검색, 게시판 목록에서 정렬 필터(검색어)
- HTML Form을 통한 데이터 전송
  - 회원 가입, 상품 주문, 데이터 변경
- HTTP API를 통한 데이터 전송
  - 회원 가입, 상품 주문, 데이터 변경
  - 서버 to 서버, 앱 클라이언트, 웹 클라이언트(Ajax)

![](https://velog.velcdn.com/images/sanizzang00/post/c5e21c71-ff52-4a87-a3b0-8a0c9d288daf/image.png)

- 이미지, 정적 텍스트 문서
- 조회는 GET 사용
- 정적 데이터는 일반적으로 쿼리 파라미터 없이 리소스 경로로 단순하게 조회 가능

![](https://velog.velcdn.com/images/sanizzang00/post/cb42805f-476f-4592-9b31-597a43179757/image.png)

- 주로 검색, 게시판 목록에서 정렬 필터(검색어)
- 조회 조건을 줄여주는 필터, 조회 결과를 정렬하는 정렬 조건에 주로 사용
- 조회는 GET 사용
- GET은 쿼리 파라미터 사용해서 데이터를 전달

![](https://velog.velcdn.com/images/sanizzang00/post/10afca98-6f2c-4a56-b509-9848c351d065/image.png)

![](https://velog.velcdn.com/images/sanizzang00/post/80d4d502-aac0-46f5-928d-893fc28c623f/image.png)

![](https://velog.velcdn.com/images/sanizzang00/post/5ca168c3-fb21-4561-9d75-799bdc3b91da/image.png)

- HTML Form submit시 POST 전송
  - 예) 회원 가입, 상품 주문, 데이터 변경
- Content-Type: application/x-www-form-urlencoded 사용
  - form의 내용을 메시지 바디를 통해서 전송(key=value, 쿼리 파라미터 형식)
  - 전송 데이터를 url encoding 처리
    - 예) abc김 -> abc%EA%B9%80
- HTML Form은 GET 전송도 가능
- Content-Type: multipart/form-data
  - 파일 업로드 같은 바이너리 데이터 전송시 사용
  - 다른 종류의 여러 파일과 폼의 내용 함께 전송 가능(그래서 이름 multipart)
- 참고: HTML Form 전송은 GET, POST만 지원

![](https://velog.velcdn.com/images/sanizzang00/post/7a531833-49ce-49dc-8363-e8fc9e9d85ae/image.png)

- 서버 to 서버
  - 백엔드 시스템 통신
- 앱 클라이언트
  - 아이폰, 안드로이드
- 웹 클라이언트
  - HTML에서 Form 전송 대신 자바 스크립트를 통한 통신에 사용(AJAX)
  - ex) React, VueJs 같은 웹 클라이언트와 API 통신
- POST, PUT, PATCH: 메시지 바디를 통해 데이터 전송
- GET: 조회, 쿼리 파라미터로 데이터 전달
- Content-Type: application/json을 주로 사용(사실상 표준)
  - TEXT, XML, JSON 등등

### 회원 관리 시스템 (POST - 신규 자원 등록 특징)

- 클라이언트는 등록될 리소스의 URI를 모른다.
  - 회원 등록 /members -> POST
  - POST /members
- 서버가 새로 등록된 리소스 URI를 생성해준다.
  - HTTP/1.1 201 Created
    Location: /members/100
- 컬렉션(Collection)
  - 서버가 관리하는 리소스 디렉토리
  - 서버가 리소스의 URI를 생성하고 관리
  - 여기서 컬렉션은 /members

### 파일 관리 시스템 (PUT - 신규 자원 등록 특징)

- 클라이언트가 리소스 URI를 알고 있어야 한다.
  - 파일 등록 /files/{filename} -> PUT
  - PUT /files/star.jpg
- 클라이언트가 직접 리소스의 URI를 지정한다.
- 스토어(Store)
  - 클라이언트가 관리하는 리소스 저장소
  - 클라이언트가 리소스의 URI를 알고 관리
  - 여기서 스토어는 /files

### HTML FORM 사용

- HTML FORM은 GET, POST만 지원
- AJAX 같은 기술을 사용해서 해결 가능
- 여기서는 순수 HTML, HTML FORM 이야기
- GET, POST만 지원하므로 제약이 있음
- 컨트롤 URI
  - GET, POST만 지원하므로 제약이 있음
  - 이런 제약을 해결하기 위해 동사로 된 리소스 경로 사용
  - POST의 /new, /edit, /delete가 컨트롤 URI
  - HTTP 메서드로 해결하기 애매한 경우 사용(HTTP API 포함)

### 참고하면 좋은 URI 설계 개념

- 문서(document)
  - 단일 개념(파일 하나, 객체 인스턴스, 데이터베이스 row)
  - 예) /members/100, /files/star.jpg
- 컬렉션(collection)
  - 서버가 관리하는 리소스 디렉터리
  - 서버가 리소스의 URI를 생성하고 관리
  - 예) /members
- 스토어(store)
  - 클라이언트가 관리하는 자원 저장소
  - 클라이언트가 리소스의 URI를 알고 관리
  - 예) /files
- 컨트롤러(controller), 컨트롤 URI
  - 문서, 컬렉션, 스토어로 해결하기 어려운 추가 프로세스 실행
  - 동사를 직접 사용
  - 예) /members/{id}/delete

### HTTP 상태 코드 (클라이언트가 보낸 요청의 처리 상태를 응답에서 알려주는 기능)

- 1xx (Informational): 요청이 수신되어 처리중
- 2xx (Successful): 요청 정상 처리
- 3xx (Redirection): 요청을 완료하려면 추가 행동이 필요
- 4xx (Client Error): 클라이언트 오류, 잘못된 문법등으로 서버가 요청을 수행할 수 없음
- 5xx (Server Error): 서버 오류, 서버가 정상 요청을 처리하지 못함

#### 2xx (Successful): 클라이언트의 요청을 성공적으로 처리

- 200 OK
- 201 Created
  ![](https://velog.velcdn.com/images/sanizzang00/post/a6198f94-2175-4b1e-953c-dcbb63930d1f/image.png)

- 202 Accepted (요청이 접수되었으나 처리가 완료되지 않았음)
  - 배치 처리 같은 곳에서 사용
  - 예) 요청 접수 후 1시간 뒤에 배치 프로세스가 요청을 처리함
- 204 No Content (서버가 요청을 성공적으로 수행했지만, 응답 페이로드 본문에 보낼 데이터가 없음)
  - 예) 웹 문서 편집기에서 save 버튼
  - save 버튼의 결과로 아무 내용이 없어도 된다.
  - save 버튼을 눌러도 같은 화면을 유지해야 한다.
  - 결과 내용이 없어도 204 메시지(2xx)만으로 성공을 인식할 수 있다

### 3xx (Redirection): 요청을 완료하기 위해 유저 에이전트의 추가 조치 필요

- 300 Multiple Choices
- 301 Moved Permanently
- 302 Found
- 303 See Other
- 304 Not Modified
- 307 Temporary Redirect
- 308 Permanent Redirect

### 리다이렉션 이해

- 웹 브라우저는 3xx 응답의 결과에 Location 헤더가 있으면, Location 위치로 자동 이동(리다이렉트)

### 리다이렉션 이해 (종류)

- 영구 리다이렉션 - 특정 리소스의 URI가 영구적으로 이동
  - 예) /members -> /users
- 일시 리다이렉션 - 일시적인 변경
  - 주문 완료 후 주문 내역 화면으로 이동
  - PRG: Post/Redirect/Get
- 특수 리다이렉션
  - 결과 대신 캐시를 사용

#### 영구 리다이렉션(301, 308)

- 리소스의 URI가 영구적으로 이동
- 원래의 URL를 사용X, 검색 엔진 등에서도 변경 인지
- 301 Moved Permanently
  - 리다이렉트시 요청 메서드가 "GET으로 변하"고, 본문이 제거될 수 있음(MAY)
- 308 Permanent Redirect
  - 301과 기능은 같음
  - 리다이렉트시 요청 메서드와 본문 유지(처음 POST를 보내면 리다이렉트도 POST)

#### 일시적인 리다이렉션(302, 307, 303)

- 리소스의 URI가 일시적으로 변경
- 따라서 검색 엔진 등에서 URL을 변경하면 안됨
- 302 Found
  - 리다이렉트시 요청 메서드가 GET으로 변하고, 본문이 제거될 수 있음(MAY)
- 307 Temporary Redirect
  - 302와 기능은 같음
  - 리다이렉트시 요청 메서드와 본문 유지(요청 메서드를 변경하면 안된다. MUST NOT)
- 303 See Other
  - 302와 기능은 같음
  - 리다이렉트시 요청 메서드가 GET으로 변경

### PRG: Post/Redirect/Get

- POST로 주문후에 새로 고침으로 인한 중복 주문 방지
- POST로 주문후에 주문 결과 화면을 GET 메서드로 리다이렉트
- 새로고침해도 결과 화면을 GET으로 조회
- 중복 주문 대신에 결과 화면만 GET으로 다시 요청

### 기타 리다이렉션 (300, 304)

- 300 Multiple Choices: 안쓴다.
- 304 Not Modified
  - 캐시를 목적으로 사용
  - 클라이언트에게 리소스가 수정되지 않았음을 알려준다. 따라서 클라이언트는 로컬 PC에 저장된 캐시를 재사용한다.(캐시로 리다이렉트 한다.)
  - 304 응답은 응답에 메시지 바디를 포함하면 안된다. (로컬 캐시를 사용해야 하므로)
  - 조건부 GET, HEAD 요청시 사용

### 4xx (Client Error): 클라이언트 오류

- 클라이언트의 요청에 잘못된 문법등으로 서버가 요청을 수행할 수 없음
- 오류의 원인이 클라이언트에 있음
- 클라이언트가 이미 잘못된 요청, 데이터를 보내고 있기 때문에, 똑같은 재시도가 실패함

#### 400 (Bad Request): 클라이언트가 잘못된 요청을 해서 서버가 요청을 처리할 수 없음

- 요청 구문, 메시지 등등 오류
- 클라이언트는 요청 내용을 다시 검토하고, 보내야함
- 예) 요청 파라미터가 잘못되거나, API 스펙이 맞지 않을 때

#### 401 (Unauthorized): 클라이언트가 해당 리소스에 대한 인증이 필요함

- 인증(Authentication) 되지 않음
- 401 오류 방생시 응답에 WWW-Authenticate 헤더와 함께 인증 방법을 설명
- 참고
  - 인증(Authentication): 본인이 누구인지 확인, (로그인)
  - 인가(Authorization): 권한부여 (ADMIN 권한 처럼 특정 리소스에 접근할 수 있는 권한, 인증이 있어야 인가가 있음)
  - 오류 메시지가 Unauthorized이지만 인증 되지 않음 (이름이 아쉬움)

#### 403 (Forbidden): 서버가 요청을 이해했지만 승인을 거부함

- 주로 인증 자격 증명은 있지만, 접근 권한이 불충분한 경우
- 예) 어드민 등급이 아닌, 사용자가 로그인은 했지만, 어드민 등급의 리소스에 접근하는 경우

#### 404 (Not Found): 요청 리소스를 찾을 수 없음

- 요청 리소스가 서버에 없음
- 또는 클라이언트가 권한이 부족한 리소스에 접근할 때 해당 리소스를 숨기고 싶을 때

### 500 (Internal Server Error): 서버 문제로 오류 발생, 애매하며 500 오류

- 서버 내부 문제로 오류 발생
- 애매하면 500 오류

### 503 (Service Unavailable): 서비스 이용 불가

- 서버가 일시적인 과부하 또는 예정된 작업으로 잠시 요청을 처리할 수 없음
- Retry-After 헤더 필드로 얼마뒤에 복구되는지 보낼 수도 있음

### HTTP 헤더

- header-field = field-name ":" OWS field-value OWS (OWS: 띄어쓰기 허용)

### HTTP 헤더 (분류 - RFC2616(과거))

- General 헤더: 메시지 전체에 적용되는 정보, 예) Connection: close
- Request 헤더: 요청 정보, 예) User-Agent: Mozilla/5.0 (Macintosh; ..)
- Response 헤더: 응답 정보, 예) Server: Apache
- Entity 헤더: 엔티티 바디 정보, 예) Content-Type: text/html, Content-Length: 3423

### HTTP BODY (message body - RFC2616(과거))

- 메시지 본문(message body)은 엔티티 본문(entity body)을 전달하는데 사용
- 엔티티 본문은 요청이나 응답에서 전달할 실제 데이터
- 엔티티 헤더는 엔티티 본문의 데이터를 해석할 수 있는 정보를 제공
  - 데이터 유형(html, json), 데이터 길이, 압축 정보 등등

### RFC723x 변화

- 엔티티(Entity) -> 표현(Representation)
- Representation = representation Metadata + Representation Data
- 표현 = 표현 메타 데이터 + 표현 데이터

### 표현

- Content-Type: 표현 데이터의 형식
- 미디어 타입, 문자 인코딩
- 예)

  - text/html; charset=utf-8
  - application/json
  - image/png

- Content:Encoding: 표현 데이터의 압축 방식

  - 표현 데이터를 압축하기 위해 사용
  - 데이터를 전달하는 곳에서 압축 후 인코딩 헤더 추가
  - 데이터를 읽는 쪽에서 인코딩 헤더의 정보로 압축 해제
  - 예)
    - gzip
    - deflate
    - identity

- Content-Language: 표현 데이터의 자연 언어
  - ko
  - en
  - en-US
- Content-Length: 표현 데이터의 길이

- 표현 헤더는 전송, 응답 둘다 사용

### 협상(콘텐츠 네고시에이션): 클라이언트가 선호하는 표현 요청

- Accept: 클라이언트가 선호하는 미디어 타입 전달
- Accept-Charset: 클라이언트가 선호하는 문자 인코딩
- Accept-Encoding: 클라이언트가 선호하는 압축 인코딩
- Accept-Language: 클라이언트가 선호하는 자연 언어
- 협상 헤더는 요청시에만 사용

### 전송 방식

- 단순 전송
- 압축 전송
  - Content-Encoding: gzip
- 분할 전송
  - Transfer-Encoding: chunked
- 범위 전송
  - Range: bytes=1001-2000

### 일반 정보

- From: 유저 에이전트의 이메일 정보

  - 일반적으로 사용X
  - 검색 엔진 같은 곳에서, 주로 사용
  - 요청에서 사용

- Referer: 이전 웹 페이지 주소
  - 현재 요청된 페이지의 이전 웹 페이지 주소
  - A -> B로 이동하는 경우 B를 요청할 때 Referer: A를 포함해서 요청
  - Referer를 사용해서 유입 경로 분석 가능
  - 요청에서 사용
  - 참고: referer는 단어 referrer의 오타
- User-Agent: 유저 에이전트 애플리케이션 정보

  - 클라이언트 애플리케이션 정보(웹 브라우저 정보, 등등)
  - 통계 정보
  - 어떤 종류의 브라우저에서 장애가 발생하는지 파악 가능
  - 요청에서 사용

- Server: 요청을 처리하는 오리진 서버의 소프트웨어 정보

  - Server: Apache:2.2.22 (Debian)
  - server: nginx
  - 응답에서 사용

- Date: 메시지가 생성된 날짜
  - 응답에서 사용

### 특별한 정보

- Host: 요청한 호스트 정보(도메인)
  - 요청에서 사용
  - 필수
  - 하나의 서버가 여러 도메인을 처리해야 할때
  - 하나의 IP 주소에 여러 도메인이 적용되어 있을 때
- Location: 페이지 리다이렉션
  - 웹 브라우저는 3xx 응답의 결과에 Location 헤더가 있으면, Location 위치로 자동 이동(리다이렉트)
  - 201 (Created): Location 값은 요청에 의해 생성된 리소스 URI
  - 3xx (Redirection): Location 값은 요청을 자동으로 리다이렉션 하기 위한 대상 리소스를 가리킴
- Allow: 허용 가능한 HTTP 메서드
  - 405 (Method Not Allowed)에서 응답에 포함해야 함
  - Allow: GET, HEAD, PUT
- Retry-After: 유저 에이전트가 다음 요청을 하기까지 기다려야 하는 시간
  - 503 (Service Unavailable): 서비스가 언제까지 불능인지 알려줄 수 있음
  - Retry-After: Fri, 31 Dec 1999 23.59.59 GMT (날짜 표기)
  - Retry-After: 120 (초단위 표기)

### 인증

- Authorization: 클라이언트 인증 정보를 서버에 전달
  - Authorization: Basic xxxxxxxxx
- WWW-Authenticate: 리소스 접근시 필요한 인증 방법 정의
  - 401 Unauthorized 응답과 함께 사용
  - WWW-Authenticate: Newauth realm="apps", type=1, title="Login to \"apps\"", Basic realm="simple"

### 쿠키

- Set-Cookie: 서버에서 클라이언트로 쿠키 전달(응답)
- Cookie: 클라이언트가 서버에서 받은 쿠키를 저장하고, HTTP 요청시 서버로 전달

- 예) set-cookie: sessionId=abcde1234; expires=Sat, 26-Dec-2020 00:00:00 GMT; path=/; domain=.google.com; Secure
- 사용처
  - 사용자 로그인 세션 관리
  - 광고 정보 트래킹
- 쿠키 정보는 항상 서버에 전송됨
  - 네트워크 트래픽 추가 유발
  - 최소한의 정보만 사용(세션 id, 인증 토큰)
  - 서버에 전송하지 않고, 웹 브라우저 내부에 데이터를 저장하고 싶으면 웹 스토리지 (localStorage, sessionStorage) 참고
- 주의!
  - 보안에 민감한 데이터는 저장하면 안됨(주민번호, 신용카드 번호 등등)

### 쿠키 - 생명주기 (Expires, max-age)

- Set-Cookie: expires=Sat, 26-Dec-2020 04:39:21 GMT
  - 만료일이 되면 쿠키 삭제
- Set-Cookie: max-age=3600 (3600초)
  - 0이나 음수를 지정하면 쿠키 삭제
- 세션 쿠키: 만료 날짜를 생략하면 브라우저 종료시 까지만 유지
- 영속 쿠키: 만료 날짜를 입력하면 해당 날짜까지 유지

### 쿠키 - 도메인

- 예) domain=example.org
- 명시: 명시한 문서 기준 도메인 + 서브 도메인 포함
  - domain=example.org를 지정해서 쿠키 생성
    - example.org는 물론이고
    - dev.example.org도 쿠키 접근
- 생략: 현재 문서 기준 도메인만 적용
  - example.org에서 쿠키를 생성하고 domain 지정을 생략
    - example.org에서만 쿠키 접근
    - dev.example.org는 쿠키 미접근

### 쿠키 - 경로

- 예) path=/home
- 이 경로를 포함한 하위 경로 페이지만 쿠키 접근
- 일반적으로 path=/ 루트로 지정

### 쿠키 - 보안 (Secure, HttpOnly, SameSite)

- Secure
  - 쿠키는 http, https를 구분하지 않고 전송
  - Secure를 적용하면 https인 경우에만 전송
- HttpOnly
  - XSS 공격 방지
  - 자바스크립트에서 접근 불가(document.cookie)
  - HTTP 전송에만 사용
- SameSite
  - XSRF 공격 방지
  - 요청 도메인과 쿠키에 설정된 도메인이 같은 경우만 쿠키 전송

### 캐시가 없을 때

- 데이터가 변경되지 않아도 계속 네트워크를 통해서 데이터를 다운로드 받아야 한다.
- 인터넷 네트워크는 매우 느리고 비싸다.
- 브라우저 로딩 속도가 느리다.
- 느린 사용자 경험

### 캐시 적용

- 캐시 덕분에 캐시 가능 시간동안 네트워크를 사용하지 않아도 된다.
- 비싼 네트워크 사용량을 줄일 수 있다.
- 브라우저 로딩 속도가 매우 빠르다.
- 빠른 사용자 경험

### 캐시 시간 초과

- 캐시 유효 시간이 초과하면, 서버를 통해 데이터를 다시 조회하고, 캐시를 갱신한다.
- 이때 다시 네트워크 다운로드가 발생한다.
