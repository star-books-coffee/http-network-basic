# 제 2장. 간단한 프로토콜 HTTP
## 2.1. HTTP 는 클라이언트와 서버 간에 통신을 한다
- 텍스트, 이미지 같은 리소스를 필요하다고 요구하는 쪽이 클라이언트, 이러한 리소스를 제공하는 쪽이 서버이다
- HTTP 는 서버와 클라이언트의 역할을 명확히 구별하고 있다

## 2.2. 리퀘스트와 리스폰스를 교환하여 성립
```http
POST /lines/1 HTTP/1.1
Host: localhost:64521
Connection: keep-alive
Content-Type: ...
Content-Length: 16
```
- POST: `HTTP 메서드`로 어떤 동작을 요청하는지 식별한다.
- /lines/1 : URI로 서버에 요청하는 리소스를 나타낸다. (`Request URI`)
- HTTP/1.1 : 클라이언트 기능을 식별하기 위한 `HTTP 버전 번호`
- 나머지 부분 : 리퀘스트 헤더 필드
- 리퀘스트 메시지 = 메소드 + URI + 프로토콜 버전 + 옵션 리퀘스트 헤더 필드 + 엔티티
  - 엔티티 : ex) name=eunju&age=37

## 2.3. HTTP는 상태를 유지하지 않는 프로토콜
- HTTP 는 stateless 프로토콜. 즉, 이전에 보낸 request 나 이미 되돌려준 response 에 대해서는 전혀 기억하지 않는다
- HTTP 는 새로운 request 가 보내질 때마다 새로운 response 가 생성된다
- 이는 많은 데이터를 빠르고 확실하게 처리하는 `범위성` 을 확보하기 위해 **간단히 설계** 되어 있는 것이다
- 하지만 웹이 진화함에 따라 상태를 유지해야 할 경우들이 생겼다
  - ex) 쇼핑몰 로그인해서 다른 페이지로 이동해도 로그인 상태가 유지되어야 함
- HTTP/1.1 은 상태를 유지하지 않는 프로토콜이므로, 상태를 계속 유지하고픈 욕구에 부응하려고 `쿠키` 라는 기술이 도입되었다
  - **쿠키로 인해 HTTP 를 이용한 통신에서도 상태를 계속 관리할 수 있게 되었다**

## 2.4. 리퀘스트 URI 로 리소스를 식별
- HTTP는 URI 를 사용하여 인터넷 상의 리소스를 지정한다
```http
GET /lines/1 HTTP/1.1 
-> 모든 URI 를 리퀘스트 URI 에 포함한다

Host: localhost:64521
-> Host 헤더 필드에 네트워크 로케이션을 포함한다
```
- 이 외에 특정 리소스가 아닌 서버 자신에게 request 송신하는 경우에는 request URI 에 * 를 지정할 수 있다

## 2.5. 서버에 임무를 부여하는 HTTP 메소드
- GET : 리소스 획득
  - request URI 로 식별된 리소르를 가져올 수 있도록 요구한다
  - 가져올 리소스 내용은 **지정된 리소스를 서버가 해석한 결과** 이다
- POST : 엔티티 전송
- PUT : 파일 전송
  - 리퀘스트 중에 포함된 엔티티를 리퀘스트 URI 로 지정한 곳에 보존하도록 요구한다
  - HTTP/1.1 PUT 자체에는 인증 기능이 없어 보안상의 문제가 있어, 일반적인 웹사이트에서는 사용되지 않는다
- HEAD : 메시지 헤더 취득
  - GET 과 같은 기능이지만, **메시지 바디는 돌려주지 않는다**
  - **URI 유효성과 리소스 갱신시간을 확인**하는 목적 등으로 사용된다
- DELETE : 파일 삭제
  - 리퀘스트 URI 로 지정된 리소스의 삭제를 요구한다
- OPTIONS : 제공하고 있는 메소드 문의
  - 리퀘스트 URI 로 지정한 리소스가 **제공하고 있는 메소드를 조사하기 위해 사용된다**
- TRACE : 웹 서버에 접속하여 자신에게 통신을 되돌려받는 루프백 (loop-back) 을 발생시킨다
  - 리퀘스트를 보낼 때 'Max-Forwards' 라는 헤더 필드에 수치를 포함시켜 서버를 통과할 때마다 수치를 줄여간다
  - 수치가 0이 된 곳 (리퀘스트를 마지막으로 수신한 곳) 에서 상태코드 200 리스폰스를 되돌려준다
  - 이 메소드를 사용함으로써 **리퀘스트를 보낸 곳에 어떤 리퀘스트가 가공되어 있는지** 등을 조사할 수 있다
    - 프록시 등을 중계하여 **오리진 서버에 접속할 때 그 동작을 확인하기 위해** 사용되고 있다
  - 하지만 거의 사용되지 않으며, 크로스 사이트 트레이싱(XST) 같은 공격을 일으키는 보안문제도 있어서 보통 사용하지 않는다
- CONNECT : 프록시에 터널링 요구
  - **프록시에 터널 접속 확립을 요함으로써, TCP 통신을 터널링 시키기 위해 사용**된다
  - SSL 과 TLS 등의 프로토콜로 암호화된 것을 터널링하기 위해 사용된다
  - 형태 : CONNECT 프록시 서버:포트 HTTP 버전
  ```http
  CONNECT proxy.hackr.jp:8080 HTTP/1.1
  Host : proxy.hackr.jp
  ```

## 2.6. 메소드를 사용해서 지시를 내리다
- 메소드는 **리소스에 어떤 행동을 하기 원하는지를 지시하기 위해** 존재한다
  
## 2.7. 지속 연결로 접속량을 절약
- 하나의 HTML 에 여러 이미지가 포함된 경우, 브라우저를 사용해 리퀘스트를 하면 HTML 문서에 포함된 이미지를 획득하기 위해 여러 리퀘스트를 송신한다
- 리퀘스트를 보낼 때마다 매번 TCP 연결과 종료를 하게 되는 쓸모 없는 일이 발생하여 통신량이 늘어난다
### 2.7.1. 지속 연결
- HTTP/1.1 과 일부 HTTP/1.0 에서는 TCP 연결 문제를 해결하기 위해 `지속 연결` 이라는 방법을 고안하였다
- 지속연결의 특징 : 어느 한 쪽이 **명시적으로 연결을 종료하지 않는 이상 TCP 연결을 계속 유지**
    - TCP 커넥션 연결과 종료를 반복되는 오버헤드를 줄여주기 때문에 서버에 대한 부하가 경감됨
    - HTTP 리퀘스트와 리스폰스가 빠르게 완료됨
### 2.7.2. 파이프라인화
- 지속 연결은 여러 리퀘스트를 보낼 수 있도록 `파이프라인화`를 가능하게 한다
- 리퀘스트 송신 후 리스폰스 수신까지 기다린 후에 리퀘스트를 발행하던 것을 **리스폰스를 기다리지 않고 바로 다음 리퀘스트를 보낼 수 있다**
  - 따라서 **여러 리퀘스트를 병행해서 보내는 것이 가능** 하기 때문에 일일이 리스폰스를 기다릴 필요가 없다
- 속도는 개별 연결 < 지속 연결 < 파이프라인 순서이다

## 2.8. 쿠키를 사용한 상태 관리
- HTTP 는 stateless 프로토콜이라, **과거에 교환했던 리퀘스트, 리스폰스의 상태를 관리하지 않는다**
  - 따라서 인증이 필요한 웹페이지에서 상태관리가 없다면 새로운 페이지 이동마다 로그인 상태를 관리해야 하는 상황이 발생한다
- 상태를 유지하지 않는 점에서 **서버의 CPU 나 메모리 같은 리소스 소비를 억제할 수 있다**
- 또한 단순한 프로토콜이기에 다양한 곳에서 이용되는 측면이 있다
- stateless 프로토콜 특징은 남겨둔 채, 이와 같은 문제를 해결하기 위해 도입된 것이 `쿠키` 이다
- 쿠키는 리퀘스트, 리스폰스에 쿠키 정보를 추가해서 **클라이언트의 상태를 파악하기 위한 시스템** 이다
- 서버에서 리스폰스로 보내진 Set-Cookie 헤더 필드에 의해 **쿠키를 클라이언트에 보존** 하게 된다
- 클라이언트가 같은 서버로 리퀘스트를 보낼 때 **자동으로 쿠키 값을 넣어서 송신** 한다
- 서버는 클라이언트가 보낸 쿠키를 확인하여 어느 클라이언트인지 확인하고, 서버 상의 기록을 화인해서 이전 상태를 알 수 있다