# 3장. HTTP 정보는 HTTP 메시지에 있다
## 3.1 HTTP 메시지
- HTTP 메시지 : 복수 행의 데이터로 구성된 텍스트 문자열
- HTTP 메시지는 크게 구분하면 메시지 헤더와 메시지 바디로 구성되어 있고, 최초에 나타나는 개행 문자(CR+LF)로 메시지 헤더와 바디를 구분

> 메시지 바디가 항상 존재한다고 할 수는 없다

![](https://blog.kakaocdn.net/dn/bviYxW/btrHSA7bZEf/EnsoTYRPaBd7GxMMiZGCrk/img.png)

- **메시지 헤더** : 서버와 클라이언트가 꼭 처리해야 하는 리퀘스트와 리스폰스 내용과 속성 등

- **CR+LF** : CR(Carriege return : 16진수 0x0d)와 LF(line feed : 16진수 0x0a)

- **메시지 바디** : 꼭 전송되는 데이터 그 자체

## 3.2 리퀘스트 메시지와 리스폰스 메시지의 구조

![](https://velog.velcdn.com/images%2Fanhesu11%2Fpost%2Ff9934c03-a615-41ff-8862-219629fb0aee%2Fimage.png)

- **리퀘스트 라인** : 리퀘스트에 사용하는 메서드와 리퀘스트 URI와 사용하는 HTTP 버전 포함
- **상태 라인** : 리스폰스 결과를 나타내는 상태 코드와 설명, 사용하는 HTTP 버전
- **헤더 필드** : 리퀘스트와 리스폰스의 여러 조건과 속성 등을 나타내는 각종 헤더 필드 포함
  - 일반 헤더 필드, 리퀘스트 헤더 힐드, 리스폰스 헤더 필드, 렌티티 헤더 필드 등 4종류가 있다.
- 그 외 : HTTP의 RFC에는 없는 헤더 필드(쿠키 등) 포함

![](https://developer.mozilla.org/ko/docs/Web/HTTP/Messages/http_request_headers3.png)
![](https://developer.mozilla.org/ko/docs/Web/HTTP/Messages/http_response_headers3.png)

## 3.3 인코딩으로 전송 효율을 높이다
- HTTP로 데이터를 전송할 때 인코딩(변환)을 실시함으로써 전송 효율을 높일 수 있다.
- 인코딩을 하면 다량의 액세스를 효율 좋게 처리할 수 있지만 CPU 등의 리소스는 보다 많이 소비하게 됨
### 3.3.1 메시지 바디와 엔티티 바디의 차이
- **메시지** : HTTP 통신의 기본 단위로 옥텟 시퀀스(Octet sequence, octet은 8비트)로 구성되고 통신을 통해서 전송됨
- **엔티티** : 리퀘스트랑 리스폰스의 페이로드로 전송되는 정보로 엔티티 헤더 필드와 엔티티 바디로 구성

> - HTTP 메시지 바디의 역할은 리퀘스트와 리스폰스에 관한 엔티티 바디를 운반하는 일
> - 기본적으로 메시지 바디와 엔티티 바디는 같지만 전송 코딩이 적용된 경우에는 엔티티 바디의 내용이 변화하기 때문에 메시지 바디와 달라짐

### 3.3.2 압축해서 보내는 콘텐츠 코딩
- 메일에 파일을 첨부해서 보낼 경우 같이 용량을 줄이기 위해서 파일을 zip으로 압축하고 나서 첨부해서 보내는 일처럼, HTTP에도 비슷한 콘텐츠 코딩 기능이 구현되어 있음
- **콘텐츠 코딩**
  - 엔티티에 적용하는 엔코딩으로, 엔티티 정보를 유지한 채로 압축
  - 콘텐츠 코딩된 엔티티는 수신한 클라이언트 측에서 디코딩
- 주요 콘텐츠 압축
  - gzip(GNU zip)
  - compress(UNIX의 표준 압축)
  - deflate(zlib)
  - identity(인코딩 없음)
### 3.3.3 분해해서 보내는 청크 전송 코딩
- HTTP 통신에서는 리퀘스트했었던 리소스 전부에서 엔티티 바디의 전송이 완료되지 않으면 브라우저에 표시되지 않는다.
- 사이즈가 큰 데이터를 전송하는 경우 데이터를 분할해서 조금씩 표시할 수 있다.
- 엔티티 바디를 분할하는 기능을 `청크 전송 코딩(Chunked transfer Coding)`이라고 한다.

![](https://bunnyacademy.b-cdn.net/vgGCR-What-Is-HTTP-Chunked-Encoding.png)

- 청크 전송 코딩 과정
  - 엔티티 바디를 청크(덩어리)로 분해
  - 다음 청크 사이즈를 16진수로 사용해서 단락을 표시하고 엔티티 바디 끝에는 "0(CR+LF)"를 기록해둔다.
  - 청크 전송 코딩된 엔티티 바디는 수신한 클라이언트 측에서 원래의 엔티티 바디로 디코딩

## 3.4 여러 데이터를 보내는 멀티 파트
- 메일의 본문 + 복수의 첨부 파일을 붙여 함께 보내는 경우 이것은 `MIME`(Multipurpose Internet Mail Extensions : 다목적 인터넷 메일 확장 사양)으로 불림.
  - 메일로 텍스트나 영상, 이미지와 같은 여러 다른 데이터를 다루기 위한 기능을 사용하고 있음.
- MIME은 이미지 등의 바이너리 데이터를 아스키 문자열에 인코딩하는 방법과 데이터 종류를 나타내는 방법 등을 규정하고 있음.
- MIME의 확장 사양에 있는 멀티 파트라고 하는 여러 다른 종류의 데이터를 수용하는 방법을 사용하고 있는 것임.
- HTTP도 멀티파트에 대응하고 있어 하나의 메시지 바디 내부에 엔티티를 여러 개 포함시켜 보낼 수 있음.
- 주로 이미지나 텍스트 파일 등 업로드 시 사용

- **multipart/form-data** : Web 폼으로부터 파일 업로드에 사용
  ```
  Content-Type: multipart/formdata; boundary=AaB03x

  --AaB03x
  Content-Disposition: form-data; name="field1"

  Joe Blow
  --AaB03x
  Content-Disposition: form-data; name="pics"; filename="file1.txt"
  Content-Type: text/plain

  ... (file1.txt 데이터)
  --AaB03x--
  ```
- **multipart/byteranges** : 상태 코드 206(Partial Content) 리스폰스 메시지가 복수 범위의 내용을 포함하는 데 사용
  ```
  HTTP/1.1 206 Partial Content
  Date: Fri, 13 Jul 2012 02:45:26 GMT
  Last-Modified: Fri, 31 Jul 2007 02:02:20 GMT
  Content-Type: multipart/byteranges; boundary=THIS_STRING_SEPERATES

  --THIS_STRING_SEPERATES
  Content-Type: application/pdf
  Content-Range: bytes 500-999/8000

  ...(지정한 범위의 데이터)...
  --THIS_STRING_SEPERATES

  Content-Type:application/pdf
  content-Range: bytes 7000-7999/8000

  ...(지정한 범위의 데이터)...
  --THIS_STRING_SEPERATES--
  ```
- HTTP 메시지에 멀티파트를 사용할 때에는 Content-Type 헤더필드 사용
- 각각의 엔티티를 구분하기 위해 "boundary" 문자열 사용
- 각 엔티티의 선두에는 "boundary" 문자열 앞에 "--" 삽입
- 멀티파트의 마지막에는 그 문자열의 마지막 부분에 "--"를 삽입해 마무리

> - 멀티파트는 파트마다 헤더 필드가 포함된다.
> - 또한 중간에 멀티파트를 만드는 것과 같이 파트를 내부에 포함할 수도 있다.
> - 자세한 내용 : RFC2046

## 3.5 일부분만 받는 레인지 리퀘스트
- 다운로드 중에 커넥션이 끊어지게 되면 처음부터 다운로드 해야 함
  ➡️ 이러한 문제를 해결하기 위해서 일반적인 리줌(resume)이라는 기능이 필요하게 됨
  - 리줌을 통해 다운로드를 한 곳에서부터 다운로드 가능
- resume을 실현하기 위해서는 엔티티 범위를 지정해서 다운로드를 해야 함
- 이와 같이 범위를 지정해서 리퀘스트하는 것을 레인지 리퀘스트(Range Request)라고 함.
- 10,000 바이트 정도 크기의 리소스에서 5001~10,000 바이트의 범위(바이트 레인지) 만을 리퀘스트할 수 있음
![](https://www.keycdn.com/img/support/byte-range.png)

### 바이트 레인지 지정
- **5,001 ~ 10,000 바이트**
  ```
  Range: bytes = 5001-10000
  ```
- **5,001 바이트 이상**
  ```
  Range: bytes = 5001-
  ```
- **처음부터 3,000 바이트까지, 그리고 5,000~7,000 바이트까지의 복수 범위**
  ```
  Range: bytes = 3000, 5000-7000
  ```

### 레인지 리퀘스트에 대한 리스폰스
- 상태 코드 206 Partial Content라는 리스폰스 메시지가 되돌아 온다.
- 복수 범위의 레인지 리퀘스트에 대한 리스폰스는 multipart/byteranges로 리스폰스가 되돌아 온다.
- 서버가 레인지 리퀘스트를 지원하지 않는 경우에는 상태 코드 200 OK라는 리스폰스 메시지로 완전한 엔티티가 되돌아 온다.

## 3.6 최적의 콘텐츠를 돌려주는 콘텐츠 네고시에이션
- 같은 콘텐츠(내용)이지만 여러 개의 페이지를 지닌 웹페이지가 있다. -> 콘텐츠 네고시에이션(Content Negotiation)
  - ex. 브라우저가 같은 URI에 액세스할 때 각각 한국어판 웹 페이지와 영어판 웹 페이지를 표시
- **콘텐츠 네고시에이션(Content Negotiation)**이란 클라이언트와 서버가 제공하는 리소스의 내용에 대해 교섭하는 것
  - 클라이언트에 더욱 적합한 리소스를 제공하기 위한 구조
- 제공하는 리소스를 언어와 문자 세트, 인코딩 방식 등을 기준으로 판단
- 판단 기준은 리퀘스트 메시지에 포함된 리퀘스트 헤더 필드
  - Accept
  - Accept-Charset
  - Accept-Encoding
  - Accept-Language
  - Content-Language

### 콘텐츠 네고시에이션 종류
- **서버 구동형 네고시에이션(Server-driven Negotitation)**

  - 서버 측에서 콘텐츠 네고시에이션을 하는 방식
  - 서버 측에서 리퀘스트 헤더 필드의 정보를 참고해서 자동적으로 처리
  - 단지 브라우저가 보내는 정보를 근거로 하기 때문에 유저에게 정말로 적절한 것이 선택되었다고 할 수 없음.
- **에이전트 구동형 네고시에이션(Agent-driven Negotiation)**
  - 클라이언트 측에서 콘텐츠 네고시에이션을 하는 방식
  - 브라우저에 표시된 선택지 중에서 유저가 수동으로 선택
  - JavaScript 등을 사용해서 웹 페이지에서 자동적으로 정할 수도 있음 (ex. OS의 종류나 브라우저의 종류 등에 의해 PC 용과 스마트폰 용의 웹 페이지를 자동 전환)
- **트렌스페어런트 네고시에이션(Transparent Negotiation)**
  - 서버 구동형 + 에이전트 구동형
  - 서버와 클라이언트가 각각 콘텐츠 네고시에이션을 하는 방식
