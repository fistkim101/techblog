# Cookie, Session 다시 짚기

## Cookie 와 Session 의 배경

각각이 무엇인지를 아는건 기본적인 것인데, 결국 저 둘이 왜 필요한지에 대한 맥락에 대한 이해가 가장 먼저다. 결국 Cookie 와 Session 은 stateless 의 한계를 보완(극복이라는 단어는 안맞는 것 같다)하고자 생긴 개념인데 메인 저장 공간을 클라이언트에 둘지 서버에 둘지에 대한 차이가 Cookie 와 Session 인 것이다.

stateless 에 관한 설명은 이미 김영한님 강의(모든 개발자를 위한 HTTP 웹 기본 지식) 정리에서 다뤘어서 여기선 넘어간다.



## Session

<figure><img src="../../../.gitbook/assets/image (2) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

출처 : NEXTSTEP



그림이 세션을 잘 설명해주고 있다. 특히 쿠키에서 세션 키값을 가지고 있다는 것과 stateless 의 특성을 그림에 잘 담고 있다. 아래에 내가 구현했던 세션 코드를 첨부한다.

```java
public class SessionStorage {

    public static final String SESSION_ID_NAME = "JSESSIONID";
    private static final SessionStorage sessionStorage = new SessionStorage();
    private final Map<String, Session> storage = new HashMap<>();

    private SessionStorage() {

    }

    public static SessionStorage getInstance() {
        return sessionStorage;
    }

    public void storeSession(String sessionId, Session session) {
        this.storage.put(sessionId, session);
    }

    public Session getSession(String uuid) {
        Session session = this.storage.get(uuid);
        if (session == null) {
            session = new Session();
            storeSession(uuid, session);
            return session;
        }

        return this.storage.get(uuid);
    }

}
```

위 코드가 세션 저장소 코드인데 아래와 같이 홈("/") 에서 세션을 저장하고 있다. 결론적으로 이 사이트에 방문하는 모든 클라이언트들은 1클라이언트 1세션을 부여받게 되는 것이다.(쿠키를 일부러 지워버리지 않는 이상은 1클라이언트 : 1세션의 관계를 유지할 수 있게 된다)

```java
    @GetMapping(path = "/")
    public HttpResponseMessage index() throws IOException {
        HttpHeaders httpHeaders = new HttpHeaders();

        httpHeaders.setContentType(MediaType.TEXT_HTML);
        httpHeaders.setLocation(URI.create("http://localhost:8080/index.html").toString());

        UUID uuid = UUID.randomUUID();
        SessionStorage.getInstance().storeSession(uuid.toString(), new Session());
        httpHeaders.setSessionIdInCookie(uuid.toString());

        return new HttpResponseMessage(HttpStatus.FOUND, httpHeaders);
    }

```



## 세션의 문제점

<figure><img src="../../../.gitbook/assets/image (3) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

세션은 서버에서 상태를 가지고 있는 것이기에 당연히 스케일 아웃이 발생하면 문제가 생길 수 있다. 그래서 Sticky Session 같은 것을 사용한다. 실제로 빈스토크에 보면 관련 설정이 있다. 난 실무에서는 클라이언트가 웹이 아니라 모바일 앱이었기 때문에 딱히 이런 부분을 실무에서는 고민할 필요는 없었긴 하다.
