# CsrfFilter

<figure><img src="../../.gitbook/assets/image (47).png" alt=""><figcaption></figcaption></figure>

[공식문서](https://docs.spring.io/spring-security/site/docs/current/api/org/springframework/security/web/csrf/CsrfFilter.html)를 보면 아래와 같이 나와있다.

> Applies [CSRF](https://www.owasp.org/index.php/Cross-Site\_Request\_Forgery\_\(CSRF\)) protection using a synchronizer token pattern. Developers are required to ensure that [`CsrfFilter`](https://docs.spring.io/spring-security/site/docs/current/api/org/springframework/security/web/csrf/CsrfFilter.html) is invoked for any request that allows state to change. Typically this just means that they should ensure their web application follows proper REST semantics (i.e. do not change state with the HTTP methods GET, HEAD, TRACE, OPTIONS).

GET, HEAD, TRACE, OPTIONS 말고는 다 동작하는 것 같다. 핵심은 데이터 변경에 관한 HTTP method 요청시 서버에서 내려준 토큰 값을 그대로 다시 물고 들어오라는 것이다. 그게 아니면 공격으로 간주하고 필터에서 거른다.

**CsrfFilter.java**

```java
	@Override
	protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain filterChain)
			throws ServletException, IOException {
		DeferredCsrfToken deferredCsrfToken = this.tokenRepository.loadDeferredToken(request, response);
		request.setAttribute(DeferredCsrfToken.class.getName(), deferredCsrfToken);
		this.requestHandler.handle(request, response, deferredCsrfToken::get);
		if (!this.requireCsrfProtectionMatcher.matches(request)) {
			if (this.logger.isTraceEnabled()) {
				this.logger.trace("Did not protect against CSRF since request did not match "
						+ this.requireCsrfProtectionMatcher);
			}
			filterChain.doFilter(request, response);
			return;
		}
		CsrfToken csrfToken = deferredCsrfToken.get();
		String actualToken = this.requestHandler.resolveCsrfTokenValue(request, csrfToken);
		if (!equalsConstantTime(csrfToken.getToken(), actualToken)) {
			boolean missingToken = deferredCsrfToken.isGenerated();
			this.logger.debug(
					LogMessage.of(() -> "Invalid CSRF token found for " + UrlUtils.buildFullRequestUrl(request)));
			AccessDeniedException exception = (!missingToken) ? new InvalidCsrfTokenException(csrfToken, actualToken)
					: new MissingCsrfTokenException(actualToken);
			this.accessDeniedHandler.handle(request, response, exception);
			return;
		}
		filterChain.doFilter(request, response);
	}
```

실제로 디버거를 찍고 보니 조건문에 걸리고 예외가 발생하게 된다.&#x20;
