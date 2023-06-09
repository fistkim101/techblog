# 스프링 시큐리티 아키텍처

## 아키텍처 구성

<figure><img src="../../.gitbook/assets/image (60) (1).png" alt=""><figcaption><p>출처: 강의자료</p></figcaption></figure>

강의에서는 위 아키텍처를 세부적으로 나눠서 알아봤고 마지막 순서로 이를 종합 및 요약해서 설명해주고 있다. 전체적으로 위와 같은 구조로 실제로 동작하는지 디버거를 통해서 내부적으로 들여다보고, 필요시 부분적으로 상세히 정리한다.

## 스프링 시큐리티 동작 흐름

### 1. AbstractFilterRegistrationBean 을 활용하여 DeligatingFilterProxy 를 필터로 등록

```java
@AutoConfiguration(after = SecurityAutoConfiguration.class)
@ConditionalOnWebApplication(type = Type.SERVLET)
@EnableConfigurationProperties(SecurityProperties.class)
@ConditionalOnClass({ AbstractSecurityWebApplicationInitializer.class, SessionCreationPolicy.class })
public class SecurityFilterAutoConfiguration {

	private static final String DEFAULT_FILTER_NAME = AbstractSecurityWebApplicationInitializer.DEFAULT_FILTER_NAME;

	@Bean
	@ConditionalOnBean(name = DEFAULT_FILTER_NAME)
	public DelegatingFilterProxyRegistrationBean securityFilterChainRegistration(
			SecurityProperties securityProperties) {
		DelegatingFilterProxyRegistrationBean registration = new DelegatingFilterProxyRegistrationBean(
				DEFAULT_FILTER_NAME);
		registration.setOrder(securityProperties.getFilter().getOrder());
		registration.setDispatcherTypes(getDispatcherTypes(securityProperties));
		return registration;
	}

```

SecurityFilterAutoConfiguration 에 정의된 securityFilterChainRegistration() 에서 DelegatingFilterProxyRegistrationBean 을 bean 으로 생성한다. DelegatingFilterProxyRegistrationBean 를 아래에서 자세히 보자.

```java
/**
 * A {@link ServletContextInitializer} to register {@link DelegatingFilterProxy}s in a
 * Servlet 3.0+ container. Similar to the {@link ServletContext#addFilter(String, Filter)
 * registration} features provided by {@link ServletContext} but with a Spring Bean
 * friendly design.
 * <p>
 * The bean name of the actual delegate {@link Filter} should be specified using the
 * {@code targetBeanName} constructor argument. Unlike the {@link FilterRegistrationBean},
 * referenced filters are not instantiated early. In fact, if the delegate filter bean is
 * marked {@code @Lazy} it won't be instantiated at all until the filter is called.
 * <p>
 * Registrations can be associated with {@link #setUrlPatterns URL patterns} and/or
 * servlets (either by {@link #setServletNames name} or through a
 * {@link #setServletRegistrationBeans ServletRegistrationBean}s). When no URL pattern or
 * servlets are specified the filter will be associated to '/*'. The targetBeanName will
 * be used as the filter name if not otherwise specified.
 *
 * @author Phillip Webb
 * @since 1.4.0
 * @see ServletContextInitializer
 * @see ServletContext#addFilter(String, Filter)
 * @see FilterRegistrationBean
 * @see DelegatingFilterProxy
 */
public class DelegatingFilterProxyRegistrationBean extends AbstractFilterRegistrationBean<DelegatingFilterProxy>
		implements ApplicationContextAware {

	private ApplicationContext applicationContext;

	private final String targetBeanName;

	/**
	 * Create a new {@link DelegatingFilterProxyRegistrationBean} instance to be
	 * registered with the specified {@link ServletRegistrationBean}s.
	 * @param targetBeanName name of the target filter bean to look up in the Spring
	 * application context (must not be {@code null}).
	 * @param servletRegistrationBeans associate {@link ServletRegistrationBean}s
	 */
	public DelegatingFilterProxyRegistrationBean(String targetBeanName,
			ServletRegistrationBean<?>... servletRegistrationBeans) {
		super(servletRegistrationBeans);
		Assert.hasLength(targetBeanName, "TargetBeanName must not be null or empty");
		this.targetBeanName = targetBeanName;
		setName(targetBeanName);
	}

```

DelegatingFilterProxy 를 bean 으로 생성해주는 bean 임을 알 수 있다. 즉, DelegatingFilterProxyRegistrationBean 의 목적은 결국 DelegatingFilterProxy을 bean 으로 설정해주는 것이고 DelegatingFilterProxy 는&#x20;

```java
private static final String DEFAULT_FILTER_NAME = AbstractSecurityWebApplicationInitializer.DEFAULT_FILTER_NAME;
```

위와 같이 DEFAULT\_FILTER\_NAME 를 가지고서 filter 로 등록이 된다. 왜냐하면 DeligatingFilterProxy 에서 DEFAULT\_FILTER\_NAME 으로 FilterChainProxy 를 호출하여 사용해야하기 때문이다.

DeligatingFilterProxy 가 DEFAULT\_FILTER\_NAME 으로 FilterChainProxy 를 호출해서 써야하는데 이 부분을 살펴보자. DelegatingFilterProxy 를 생성할 때 아래와 같이 targetBeanName 으로 DEFAULT\_FILTER\_NAME 를 사용해주고 있다.

```java
public DelegatingFilterProxy(String targetBeanName) {
	this(targetBeanName, null);
}
```

```java
protected Filter initDelegate(WebApplicationContext wac) throws ServletException {
	String targetBeanName = getTargetBeanName();
	Assert.state(targetBeanName != null, "No target bean name set");
	Filter delegate = wac.getBean(targetBeanName, Filter.class);
	if (isTargetFilterLifecycle()) {
		delegate.init(getFilterConfig());
	}
	return delegate;
}
```

그래서 위와 같이 bean 을 찾아와서 delegate 에 할당해준다. 그럼 당연히 DEFAULT\_FILTER\_NAME 으로 이미 bean이 생성되어 있어야겠지? 그 부분을 살펴보자.

### 2. DEFAULT\_FILTER\_NAME 으로 FilterChainProxy 를 Bean 으로 등록

```java
/**
 * Uses a {@link WebSecurity} to create the {@link FilterChainProxy} that performs the web
 * based security for Spring Security. It then exports the necessary beans. Customizations
 * can be made to {@link WebSecurity} by implementing {@link WebSecurityConfigurer} and
 * exposing it as a {@link Configuration} or exposing a {@link WebSecurityCustomizer}
 * bean. This configuration is imported when using {@link EnableWebSecurity}.
 *
 * @author Rob Winch
 * @author Keesun Baik
 * @since 3.2
 * @see EnableWebSecurity
 * @see WebSecurity
 */
@Configuration(proxyBeanMethods = false)
public class WebSecurityConfiguration implements ImportAware, BeanClassLoaderAware {

	(일부 생략)

	/**
	 * Creates the Spring Security Filter Chain
	 * @return the {@link Filter} that represents the security filter chain
	 * @throws Exception
	 */
	@Bean(name = AbstractSecurityWebApplicationInitializer.DEFAULT_FILTER_NAME)
	public Filter springSecurityFilterChain() throws Exception {
		boolean hasFilterChain = !this.securityFilterChains.isEmpty();
		if (!hasFilterChain) {
			this.webSecurity.addSecurityFilterChainBuilder(() -> {
				this.httpSecurity.authorizeHttpRequests((authorize) -> authorize.anyRequest().authenticated());
				this.httpSecurity.formLogin(Customizer.withDefaults());
				this.httpSecurity.httpBasic(Customizer.withDefaults());
				return this.httpSecurity.build();
			});
		}
		for (SecurityFilterChain securityFilterChain : this.securityFilterChains) {
			this.webSecurity.addSecurityFilterChainBuilder(() -> securityFilterChain);
		}
		for (WebSecurityCustomizer customizer : this.webSecurityCustomizers) {
			customizer.customize(this.webSecurity);
		}
		return this.webSecurity.build();
	}

```

WebSecurityConfiguration 내에서 AbstractSecurityWebApplicationInitializer.DEFAULT\_FILTER\_NAME 이름으로 bean 을 생성해주고 있다. 아직 FilterChainProxy 가 된 단계는 아니다.

**WebSecurity.java**

```java
@Override
protected Filter performBuild() throws Exception {
	Assert.state(!this.securityFilterChainBuilders.isEmpty(),
			() -> "At least one SecurityBuilder<? extends SecurityFilterChain> needs to be specified. "
					+ "Typically this is done by exposing a SecurityFilterChain bean. "
					+ "More advanced users can invoke " + WebSecurity.class.getSimpleName()
					+ ".addSecurityFilterChainBuilder directly");
	int chainSize = this.ignoredRequests.size() + this.securityFilterChainBuilders.size();
	List<SecurityFilterChain> securityFilterChains = new ArrayList<>(chainSize);
	List<RequestMatcherEntry<List<WebInvocationPrivilegeEvaluator>>> requestMatcherPrivilegeEvaluatorsEntries = new ArrayList<>();
	for (RequestMatcher ignoredRequest : this.ignoredRequests) {
		WebSecurity.this.logger.warn("You are asking Spring Security to ignore " + ignoredRequest
				+ ". This is not recommended -- please use permitAll via HttpSecurity#authorizeHttpRequests instead.");
		SecurityFilterChain securityFilterChain = new DefaultSecurityFilterChain(ignoredRequest);
		securityFilterChains.add(securityFilterChain);
		requestMatcherPrivilegeEvaluatorsEntries
				.add(getRequestMatcherPrivilegeEvaluatorsEntry(securityFilterChain));
	}
	for (SecurityBuilder<? extends SecurityFilterChain> securityFilterChainBuilder : this.securityFilterChainBuilders) {
		SecurityFilterChain securityFilterChain = securityFilterChainBuilder.build();
		securityFilterChains.add(securityFilterChain);
		requestMatcherPrivilegeEvaluatorsEntries
				.add(getRequestMatcherPrivilegeEvaluatorsEntry(securityFilterChain));
	}
	if (this.privilegeEvaluator == null) {
		this.privilegeEvaluator = new RequestMatcherDelegatingWebInvocationPrivilegeEvaluator(
				requestMatcherPrivilegeEvaluatorsEntries);
	}
	FilterChainProxy filterChainProxy = new FilterChainProxy(securityFilterChains);
	if (this.httpFirewall != null) {
		filterChainProxy.setFirewall(this.httpFirewall);
	}
	if (this.requestRejectedHandler != null) {
		filterChainProxy.setRequestRejectedHandler(this.requestRejectedHandler);
	}
	else if (!this.observationRegistry.isNoop()) {
		CompositeRequestRejectedHandler requestRejectedHandler = new CompositeRequestRejectedHandler(
				new ObservationMarkingRequestRejectedHandler(this.observationRegistry),
				new HttpStatusRequestRejectedHandler());
		filterChainProxy.setRequestRejectedHandler(requestRejectedHandler);
	}
	filterChainProxy.setFilterChainDecorator(getFilterChainDecorator());
	filterChainProxy.afterPropertiesSet();

	Filter result = filterChainProxy;
	if (this.debugEnabled) {
		this.logger.warn("\n\n" + "********************************************************************\n"
				+ "**********        Security debugging is enabled.       *************\n"
				+ "**********    This may include sensitive information.  *************\n"
				+ "**********      Do not use in a production system!     *************\n"
				+ "********************************************************************\n\n");
		result = new DebugFilter(filterChainProxy);
	}

	this.postBuildAction.run();
	return result;
}
```

```java
FilterChainProxy filterChainProxy = new FilterChainProxy(securityFilterChains);
```

많은 과정을 거쳐서 filterChainProxy 가 생성된다.

### 3. 호출이 들어오면 DeligatingFilterProxy 가 정해진 상수로 FilterChainProxy 를 호출하여 이를 위임 처리. FilterChainProxy 내에서는 여러 Filter 들을 순회시키며 처리.

**DeligatingFilterProxy.java**

<figure><img src="../../.gitbook/assets/image (63) (1).png" alt=""><figcaption></figcaption></figure>

위와 같이 delegate 에 FilterChainProxy 가 할당되는 모습이고 여기에 처리를 위임한다.



**FilterChainProxy.java**

<figure><img src="../../.gitbook/assets/image (58).png" alt=""><figcaption></figcaption></figure>

스프링 시큐리티에서 제공해주는 여러 필터들을 확인할 수 있다.

## 인증 원리

<figure><img src="../../.gitbook/assets/image (46) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (59) (1).png" alt=""><figcaption></figcaption></figure>

예전에 필기한 내용과 강의 자료인데 이는 ThreadLocal 에 저장 된다는 것을 꼭 기억해두자. 동일 쓰레드 스코프이므로 request per thread 환경이라면 어디서든 꺼내서 쓸 수 있다. 웹플럭스 사용시 동일하게 동작할지는 확인해봐야한다.

<figure><img src="../../.gitbook/assets/image (72).png" alt=""><figcaption></figcaption></figure>

## 인가 원리

<figure><img src="../../.gitbook/assets/image (52) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (55) (1).png" alt=""><figcaption></figcaption></figure>

관리자에서 ROLE 을 사용할 일이 종종 있었다. 커스텀을 하게 될 경우 참고해두자. 참고로 아래와 같이 ROLE 의 하이라키 설정도 가능하다. 근데 잘 안 쓸 것 같다.

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {

    public SecurityExpressionHandler expressionHandler() {
        RoleHierarchyImpl roleHierarchy = new RoleHierarchyImpl();
        roleHierarchy.setHierarchy("ROLE_ADMIN > ROLE_USER");

        DefaultWebSecurityExpressionHandler handler = new DefaultWebSecurityExpressionHandler();
        handler.setRoleHierarchy(roleHierarchy);

        return handler;
    }

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.authorizeRequests()
                .mvcMatchers("/", "/info", "/account/**").permitAll()
                .mvcMatchers("/admin").hasRole("ADMIN")
                .mvcMatchers("/user").hasRole("USER")
                .anyRequest().authenticated()
                .expressionHandler(expressionHandler());
        http.formLogin();
        http.httpBasic();
    }

}

```

## 예외 처리

<figure><img src="../../.gitbook/assets/image (49) (1) (1).png" alt=""><figcaption></figcaption></figure>

기본적으로 저 필터에서 예외처리가 발생하고 있고 인증 및 인가 예외에 대한 처리는 내가 커스텀하게 처리가 가능하다. 아래 코드는 내가 실제로 작성한 코드이다.

```java
        httpSecurity
                .addFilterBefore(requestAndResponseLoggingFilter, ForceEagerSessionCreationFilter.class)
                .addFilterBefore(emetalPlatformJwtAuthenticationFilter, UsernamePasswordAuthenticationFilter.class)
                .exceptionHandling()
                .accessDeniedHandler(customAccessDenyHandler)
                .authenticationEntryPoint(customAuthenticationEntryPoint);
```

이 부분은 나중에 스프링 시큐리티와 JWT 를 같이 사용하는 예제 코드 작성시 조금 더 자세히 정리할 예정이다.

## 지원 필터들 순서별 목록(예전 버전이므로 최신 버전은 조금씩 다르다. 참고만 하기, 최신은 위 스크린샷 참고)

<figure><img src="../../.gitbook/assets/image (36).png" alt=""><figcaption></figcaption></figure>





