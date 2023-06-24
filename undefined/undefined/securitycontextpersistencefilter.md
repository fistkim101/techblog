# SecurityContextPersistenceFilter

강의에서는 SecurityContextPersistenceFilter 로 알려주고 있는데 이미 deprecated 되었다. SecurityContext에 대해서 다른 요청이라도 세션을 이용(기본적으로 사용하는 전략이 HTTP Session)해서 공유할 수 있도록 해주는 필터이다. 인증과 관련되어 있으므로 효율을 위해서 필터들의 순서상 앞부분에 위치한다.

```java
	private void doFilter(HttpServletRequest request, HttpServletResponse response, FilterChain chain)
			throws ServletException, IOException {
		if (request.getAttribute(FILTER_APPLIED) != null) {
			chain.doFilter(request, response);
			return;
		}
		request.setAttribute(FILTER_APPLIED, Boolean.TRUE);
		Supplier<SecurityContext> deferredContext = this.securityContextRepository.loadDeferredContext(request);
		try {
			this.securityContextHolderStrategy.setDeferredContext(deferredContext);
			chain.doFilter(request, response);
		}
		finally {
			this.securityContextHolderStrategy.clearContext();
			request.removeAttribute(FILTER_APPLIED);
		}
	}
```

코드를 보면 securityContextRepository 를 이용해서 컨텍스트를 불러오는 것을 알 수 있다. securityContextRepository 의 기본 구현체가 HttpSessionSecurityContextRepository 라고 한다. 디버거 돌려서 찍어보면 눈으로 확인할 수 있을 것 같은데 거기까진 생략했다.

사실 인증은 앱이든 웹이든 stateless 하게 토큰을 사용하기 때문에 이런 필터가 있다 정도만 인지해두도록 한다. 관리자의 경우 타임리프를 쓰기도 하니까 알아둘만 하다.

세션을 이용해서 다른 요청이지만 동일 세션 내에서 SecurityContext 를 공유할 수 있도록 해주는 필터이다. 필터의 순서가 앞부분에 잡혀있는 이유는 인증이 빨리 될수록 인증 되었을때 안타도 될 필터가 있다면 안타는 것이 효율적이기 때문이다.

SecurityContextRepository를 사용해서 기존의 SecurityContext를 읽어오거나 초기화 한다. SecurityContextRepository는 내부적으로 세션을 사용한다.

* 기본으로 사용하는 전략은 HTTP Session을 사용한다.

[Spring-Session](https://spring.io/projects/spring-session#learn)과 연동하여 세션 클러스터를 구현할 수 있다. (이 강좌에서는 다루지 않습니다.)



