# CH05 HTTP 상태코드

상태코드는 실무에서 자주 썼던 것들과 자주 쓸만한 것들만 다시 정리한다. 강의 내용보다도 [여기](https://developer.mozilla.org/ko/docs/Web/HTTP/Status) 내용이 이미 잘 정리되어있다.

## **2xx (Successful) - 클라이언트의 요청을 성공적으로 처리**

* 200 OK
* 201 Created
* 202 Accepted
* 204 No Content



### [**200 OK**](https://tools.ietf.org/html/rfc7231#section-6.3.1)

GET 요청에 대한 응답에만 쓰지 않는다. 하지만 실무에서는 GET 에 대한 응답으로 주로 사용하게 될 것 같고, 회사에서 POST에 대한 응답으로도 200을 사용했었는데 적절하지 못했다는 생각이 든다.

아래 레퍼런스를 보면 PUT이나 POST에서는 사실 ‘전송 되었다’는 사실에 대한 성공의 의미인데 POST나 PUT은 자원을 생성하거나 변경하는 행위이고 이 성공 여부에 대한 명확한 상태 코드가 존재하는데도 200을 썼다는 것은 문제가 있었던 것 같다.

> 요청이 성공적으로 되었습니다. 성공의 의미는 HTTP 메소드에 따라 달라집니다.
>
> * GET: 리소스를 불러와서 메시지 바디에 전송되었습니다.
> * HEAD: 개체 해더가 메시지 바디에 있습니다.
> * PUT 또는 POST: 수행 결과에 대한 리소스가 메시지 바디에 전송되었습니다.
> * TRACE: 메시지 바디는 서버에서 수신한 요청 메시지를 포함하고 있습니다.\
>

### [**201 Created**](https://developer.mozilla.org/ko/docs/Web/HTTP/Status/201)

자원이 성공적으로 생성되었을 때 사용하는 상태코드로 생성한 자원의 url 에 대한 정보를 응답의 Location 헤더에 실어서 보낸다.

> The Location response header indicates the URL to redirect a page to. It only provides a meaning when served with a 3xx (redirection) or 201 (created) status response.\
>

### [**204 No Content**](https://developer.mozilla.org/ko/docs/Web/HTTP/Status/204)

이걸 나는 GET을 했을때 요청한 정보가 없을때 사용하는 것이라고만 알고 있었는데 내가 알고 있던게 불완전한 지식이었다. 더 정확히는 **‘요청은 성공했고’, ‘현재 페이지에서 벗어날 필요가 없을 때’** 사용하는 상태코드였다.

레퍼런스에도 나와있다시피 ‘사용자에게 보여지는 페이지를 바꾸지 않고 리소스를 업데이트할 때’처럼 문서를 save하거나 했을 때 계속 해당 화면에서 벗어나지 않을 때 사용한다. 만약 해당 메소드 이후의 행위가 정책적으로 화면 전환이 이뤄져야 할 때(혹은 새로고침이 필요할 때)에는 성격에 따라서 200이나 201을 사용하면 된다.

## **3xx (Redirection) - 요청을 완료하기 위해 유저 에이전트의 추가 조치 필요**

* 300 Multiple Choices
* 301 Moved Permanently
* 302 Found
* 303 See Other
* 304 Not Modified
* 307 Temporary Redirect
* 308 Permanent Redirect

### **리다이렉트**

웹 브라우저는 3xx 응답의 결과에 Location 헤더가 있으면, Location 위치로 자동 이동 (리다이렉트). 리다이렉트는 상태코드가 매우 다양하지만 작동은 같은 것들이 있기도 하고 좀 개인적으로 난잡하다고 느껴지는데, 이유는 다양한 브라우저들이 존재하고 표준이 정립되는 과정에서 브라우저들이 이 표준을 따라가는 과정에서 필요에 의해 만들어진 상태코드들이 많다보니 같은 작용을 하는데 상태코드만 다양한 것들이 존재하게 된 것으로 보인다.

브라우저를 클라이언트로 하는 어플리케이션을 만들지 않는 이상 깊게는 안봐도 될 것 같긴 한데, 일단은 커리큘럼상에 있기도 하고 이번에 기회 삼아서 개념은 세워놓는게 좋을 것 같다.\


### [**301 Moved Permanently**](https://developer.mozilla.org/ko/docs/Web/HTTP/Status/301) **vs** [**308 Permanent Redirect**](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/308)

**영구적인 리다이렉트** 로 요청한 리소스가 Location (en-US) 헤더에 주어진 URL로 완전히 옮겨졌다는 것을 나타낸다.

301과 308의 차이는 리다이렉트 하는 request method의 변경 유무인데 308만 동일성이 보장된다. 따라서 301 코드는 GET과 HEAD 메소드의 응답으로만 사용하고, POST 메소드에 대해서는 메소드 변경이 명시적으로 금지된 308 (en-US) Permanent Redirect를 사용하는 것이 바람직하다.



### [**302 Found**](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/302) **vs** [**307 Temporary Redirect**](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/307)

**일시적인 리다이렉트** 로 302는 리다이렉트시 요청 메서드가 GET으로 변하고, 본문이 제거될 수 있음(MAY)인 반면 307은 리다이렉트시 요청 메서드와 본문 유지(요청 메서드를 변경하면 안된다. MUST NOT)



### **PRG 패턴**

POST - REDIRECT - GET 의 줄임말로 강의에서 소개된 패턴인데, 실무에서 자주 사용했던 익숙한 패턴이다. 나는 관리자 만들때 썼던 것 같고, 그 외에 강의 같은 것들에서도 예제로 많이 쓰였던 것 같다. 긴 설명보다는 강의 자료에서 사용된 flow 이미지 첨부가 좋을 것 같다.

<figure><img src="../../.gitbook/assets/image (8) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (69).png" alt=""><figcaption></figcaption></figure>



### [**304 Not Modified**](https://developer.mozilla.org/ko/docs/Web/HTTP/Status/304)

변한게 없으니 캐시를 사용하라는 응답 상태코드이다. Etag를 사용하면서 200을 내려주는 것과 무슨 차이인지 잘 모르겠다. 실제로 네이버 에서 네트워크 창을 보니 일부 jpeg 자원에 대해서 304를 사용하고 있긴 하다.

***

## **4xx (Client Error) - 클라이언트 오류**

* 클라이언트의 요청에 잘못된 문법등으로 서버가 요청을 수행할 수 없음
* 오류의 원인이 클라이언트에 있음
* 중요! 클라이언트가 이미 잘못된 요청, 데이터를 보내고 있기 때문에, 똑같은 재시도가 실 패함



### [**400 Bad Request**](https://developer.mozilla.org/ko/docs/Web/HTTP/Status/400)

클라이언트는 요청 내용을 다시 검토하고, 보내야함(예: 잘못된 요청 구문, 유효하지 않은 요청 메시지 프레이밍, 또는 변조된 요청 라우팅) 쉽게 말해서 API 스펙에 맞지 않게 요청하였을 때이다.

### [**401 Unauthorized**](https://developer.mozilla.org/ko/docs/Web/HTTP/Status/401)

인증과 인가중 ‘인증’이 안된 경우에 사용한다. 강의에서도 언급하고 있는데 인증이 안된 경우에 사용하는 상태코드인데도 네이밍이 Unauthorized인게 좀 아쉽지만 ‘인증’이 안된 경우이다.

### [**403 Forbidden**](https://developer.mozilla.org/ko/docs/Web/HTTP/Status/403)

서버가 요청을 이해했지만 승인을 거부함. 주로 인증 자격 증명은 있지만, 접근 권한이 불충분한 경우 (예: 어드민 등급이 아닌 사용자가 로그인은 했지만, 어드민 등급의 리소스에 접근하는 경우)

### [**404 Not Found**](https://developer.mozilla.org/ko/docs/Web/HTTP/Status/404)

이걸 내가 잘못 알고 있었는데, 4xx 인 것 자체도 좀 이상한 것 같긴 하다. 아무튼 내가 잘못 알고 있었던 부분은 클라이언트가 해당 리소스를 찾기에 정보를 덜 준 경우에 404를 내려준다고 알고 있었는데 클라이언트가 완벽하게 정보를 제공했음에도 서버에서 의도적으로 이걸 숨기고 싶을때도 404를 준다는 것을 처음 알았다. (예시: 요청 리소스가 서버에 없음, 클라이언트가 권한이 부족한 리소스에 접근할 때 해당 리소스를 숨기고 싶을 때)

> 404 상태 코드는 리소스가 일시적, 또는 영구적으로 사라졌다는 것을 의미하지는 않습니다. 리소스가 영구적히 삭제되었다면 404 상태 코드 대신 410 (en-US) (Gone) 상태 코드가 쓰여야 합니다.

### [**409 Conflict**](https://developer.mozilla.org/ko/docs/Web/HTTP/Status/409)

나는 실무에서는 아이디 중복일 경우에 409를 내려줬었는데 레퍼런스를 보니 그런 쓰임새가 아닌 것 같다.

> HTTP 409 Conflict 응답 상태 코드는 서버의 현재 상태와 요청이 충돌했음을 나타낸다. 충돌은 PUT 요청에 대응하여 발생할 가능성이 가장 높다. 예를 들어 서버에 이미 있는 파일보다 오래된 파일을 업로드할 때 409 응답이 발생하여 버전 제어 충돌이 발생할 수 있다.

그런데 [여러 유명한 서비스 별 상태코드](https://gist.github.com/vkostyukov/32c84c0c01789425c29a) 를 보면 구글 클라우드가 409를 사용중인데 아이디 중복과 비슷한 맥락에서 사용중이다.

[구글 클라우드](https://cloud.google.com/storage/docs/json\_api/v1/status-codes#409-conflict)

> The following is an example of an error response you receive if you try to create a bucket using the name of a bucket you already own.

## **5xx (Server Error) - 서버 오류**

* 서버 문제로 오류 발생
* 서버에 문제가 있기 때문에 재시도 하면 성공할 수도 있음(복구가 되거나 등등)

서버가 변경되지 않았다는 전제하에서 4xx 와 다른점은 500은 요청 시점의 서버 상태에 따라 500이 내려갈 수도 있기 때문에 동일한 요청을 다른 시점에 했을 때 작동을 할 수도 있다는 뜻이고 4xx은 동일 요청을 아무리 해도 정상 응답을 밪지 못한다는 것이다.

하지만 적어도 내가 겪은 경험 내에서는 5xx이 내려가면 그냥 서버 개발자가 처리를 덜한 것이 대부분이었다.

### [**500 Internal Server Error**](https://developer.mozilla.org/ko/docs/Web/HTTP/Status/500)

서버가 예상하지 못한 상황에 놓였다는 의미. ‘서버에 문제가 있긴 한데 (정확하게 원인 파악이 안됬고) 사유는 내부적인 사유다’라는 뜻으로 빈번하게 발생하고 나도 빈번하게 만들었지만 가장 무책임한 상태코드다.

### [**503 Service Unavailable**](https://developer.mozilla.org/ko/docs/Web/HTTP/Status/503)

서비스 이용 불가. 이 상태코드는 굳이 사용한다면 앱 시작시 ‘서버 점검중’ api 에 사용할 수 있을 것 같다. 하지만 점검중인지 찔러보는 api에서 200을 받아서 boolean으로 처리하는게 맞다고 본다. 원칙적으로 서버는 5xx 에러를 발생시키지 않아야 한다.

* 서버가 일시적인 과부하 또는 예정된 작업으로 잠시 요청을 처리할 수 없음
* Retry-After 헤더 필드로 얼마뒤에 복구되는지 보낼 수도 있음
