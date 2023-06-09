# CH04 HTTP 메서드 활용

클라이언트에서 서버로 데이터를 보내는 방식은 아래와 같이 두 가지가 있다.

* **쿼리 파라미터** 를 이용한 데이터 전송
  * GET
  * 예시: 주로 정렬 필터(검색어)
* **메세지 바디** 를 통한 데이터 전송
  * POST, PUT, PATCH
  * 예시: 리소스 등록, 리소스 변경

Form 데이터를 설명하다가 Multipart 에 대해서 짧게 알아보고 넘어갔는데, 매번 실무에서 단순히 ‘나눠서 보내겠지’ 하고 썼던거라서 보고 많이 반성했다. 뭐든 대충알고 쓰면 안된다.

아무튼, 재밌는 점은 아래와 같이 boundary 라는 값으로 바디의 경계를 정하고 그 경계에 따라서 값을 나눠서 보낸다는 점이다.

<figure><img src="http://localhost:4000/assets/images/infra/http-multipart.png" alt=""><figcaption></figcaption></figure>
