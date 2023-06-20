# SecurityContextHolderFilter

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
