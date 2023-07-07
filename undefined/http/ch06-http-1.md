# CH06 HTTP 헤더1 - 일반 헤더

## **헤더의 역할**

HTTP는 메시지 본문(message body)을 통해 표현 데이터를 전달한다고 볼 수 있는데, 여기서 (표현)헤더의 역할은 표현 데이터를 해석할 수 있는 정보 제공하는 것이다(예시: 데이터 유형(html, json), 데이터 길이, 압축 정보 등등)

여기서 표현(representation) 이라는 단어는 표준으로 정립되었는데 자세한건 [레퍼런스](https://tools.ietf.org/html/rfc7230#section-3.2) 를 참고하면 된다.

> Most HTTP communication consists of a retrieval request (GET) for a representation of some resource identified by a URI.

다양한 리소스를 여러 형태(body에 여러 형태로)로 ‘표현’한다는 의미로 ‘표현’이 사용되었다고 쉽게 이해할 수 있다. 헤더의 역할은 결국 본문을 해석하기 위해서 알려줘야할 메타 데이터인 것이다.

## **표현**

* Content-Type: 표현 데이터의 형식에 대한 정보(text/html; charset=utf-8, application/json, image/png)
* Content-Encoding: 표현 데이터의 압축 방식(gzip, deflate, identity)
* Content-Language: 표현 데이터의 자연 언어(ko, en)
* Content-Length: 표현 데이터의 길이

## **협상**

클라이언트가 선호하는 표현 요청. 협상 헤더는 요청시에만 사용. 즉, 서버가 응답을 할 때 클라이언트가 보낸 협상헤더를 참고하여 응답에 반영하는 용도로 사용한다.

* Accept: 클라이언트가 선호하는 미디어 타입 전달
* Accept-Charset: 클라이언트가 선호하는 문자 인코딩
* Accept-Encoding: 클라이언트가 선호하는 압축 인코딩
* Accept-Language: 클라이언트가 선호하는 자연 언어

협상 헤더에는 적용에 있어 우선순위 개념이 있는데 이는 강의자료 첨부로 정리를 대체한다.

<figure><img src="../../.gitbook/assets/image (70).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (91).png" alt=""><figcaption></figcaption></figure>

## **일반정보**

* From: 유저 에이전트의 이메일 정보
* Referer: 이전 웹 페이지 주소(유입 경로 분석을 위해 사용하며 referrer의 오타인데 초기 배포시 이미 브라우저들이 이를 따라서 오타를 그대로 사용)
* User-Agent: 유저 에이전트 애플리케이션 정보(브라우저 정보이며 서버 입장에서 장애 분석 등을 위해 통계목적으로 사용)
* Server: 요청을 처리하는 오리진 서버의 소프트웨어 정보(중간에 거치는 proxy말고 정말 end point)
* Date: 메시지가 생성된 날짜

## **특별정보**

* Host: 요청한(요청을 해달라고 부탁하는) 호스트 정보(도메인)(필수값)
* Location: 페이지 리다이렉션에 사용될 url(201에 사용된 Location 값은 생성된 리소스의 URI)
* Allow: 허용 가능한 HTTP 메서드(예시: GET, HEAD, PUT)
* Retry-After: 유저 에이전트가 다음 요청을 하기까지 기다려야 하는 시간

## **인증**

* Authorization: 클라이언트 인증 정보를 서버에 전달(예시: Basic xxxxx, Bearer xxxxx)
* WWW-Authenticate: 리소스 접근시 필요한 인증 방법 정의(401 Unauthorized 응답과 함께 사용)
