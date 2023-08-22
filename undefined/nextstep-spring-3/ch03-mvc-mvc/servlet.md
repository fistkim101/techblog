# Servlet 다시 짚기

## Web Server, WAS

<figure><img src="../../../.gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure>

Web Server 는 정적 페이지 전달을 담당한다. WAS 는 동적 페이지 전달을 담당한다. 톰캣이 WAS 에 해당하며 톰캣이 자체적으로 Web Server 역할도 같이 수행한다.



## 서블릿 컨테이너

용어가 혼동될 수 있으나 Web Server, WAS 가 범용적인 용어이고 서블릿 컨테이너는 자바 종속적인 용어이다. 서블릿의 생명주기를 관리하는 역할을 하는 것이고 톰캣을 서블릿 컨테이너라고 말할 수 있다.



## Servlet

* 자바 진영에서 동적인 웹 페이지를 구현하기 위한 표준
* Servlet 표준을 정한 후 interface를 제공
* 각 interface에 대한 구현체는 Servlet Container(Tomcat, Jetty)가 제공



## 서블릿 컨테이너 작동 흐름

<figure><img src="../../../.gitbook/assets/image (1) (1).png" alt=""><figcaption></figcaption></figure>

**그림출처 : NEXTSTEP**



모든 사용자의 요청에 대해서 서블릿 컨테이너는 오직 하나의 서블릿 인스턴스로 대응한다. 즉, 싱글톤이다. 그렇다면 멀티 스레드에서 취약하지 않을까?

결론은 안전하다. 왜냐하면 그림에서 나오는 request, response가 실제 코드상에서는 HttpServletRequest와 HttpServletResponse 인데(필터 구현할때 나오는) 이것이 매 요청마다 새로 생성되기 때문에 멀티 스레드 환경에서 안전하다.

서블릿 컨테이너는 이 과정에 위와 같이 관여하고 있으며 서블릿의 생명주기를 관리한다.



## 그럼 현재 스프링 부트에서는 서블릿이 어떻게 동작하지?

스프링부트가 서블릿 기반이지만 개발자는 서블릿을 굳이 만들어줄 필요는 없다. 컨트롤러, 서비스, 레포지토리 등 비즈니스 로직에 필요한 것들을 구현해주면 서블릿 기반으로 동작하게된다.

핵심적인 서블릿은 디스패처 서블릿이다. 이건 이미 웹서버 구현에서 다뤘기도 했고 실습코드에서 구현을 또 할 것이라서 자세히 설명하지 않고 넘어간다.