# MVC 프레임워크 구현

## [실습코드](https://github.com/fistkim101/jwp-mvc)

링크참고



## 스프링에서 요청을 처리하는 전체 흐름

코드로 이걸 구현하는 것이기에 일단 코드를 보기 전에 전체 흐름을 짚고 넘어간다. 아래 두 그림을 보자.

<figure><img src="../../../.gitbook/assets/image (96).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (97).png" alt=""><figcaption></figcaption></figure>

2번 그림이 더 세밀하게 잘 나와있어서 좋은데 필터 관련 그림이 없어서 1번 그림까지 같이 추가를 했다. 사실 이번 구현의 핵심은 2번 그림을 자세히 구현하는 것이다. 흐름 별로 하나씩 코드를 살펴보면서 어떻게 구현하였는지, 원리는 어떠한지를 정리한다.

'MVC 프레임워크 구현' 에서는 원리를 중점적으로 알아보는게 목적이라서 실제 코드가 위 그림과 약간은 다를 수 있다. 실제 스프링과 클래스 명까지 같게 구현하는 것은 'DI 프레임워크 구현' 에서 했고 'DI 프레임워크 구현' 에서 위 그림을 두고 다시 한번 더 정리할 예정이다.

그래서 이번 'MVC 프레임워크 구현' 에서는 각 역할을 어떻게 구현했고 원리가 무엇인지에 대해서 더 집중해서 살펴보면 좋을 것 같다.



## DispatcherServlet

<figure><img src="../../../.gitbook/assets/image (98).png" alt=""><figcaption></figcaption></figure>

이건 기존 자바지기님 코드에 내가 필요한 부분을 리팩토링 하는 형식으로 미션이 진행되었다. 핵심적인 부분만 코스 수강자들이 처리하도록 하고 나머지는 이미 구현을 해둔 상태의 레포지토리를 받아서 수정하면서 진행하는 방식이다.



```java
    @Override
    public void init() throws ServletException {
        ApplicationContext.getInstance().init();
        AnnotationHandlerMapping.getInstance().init();
    }
```

여기 사용된 ApplicationContext 부터 살펴보자.



### ApplicationContext

```java
public class ApplicationContext {

    private static final ApplicationContext applicationContext = new ApplicationContext();
    private final ComponentScanner componentScanner = ComponentScanner.getInstance();
    private final Map<String, Object> beans = new HashMap<>();

    private ApplicationContext() {

    }

    public static ApplicationContext getInstance() {
        return applicationContext;
    }

    public void init() {
        Set<Class<?>> clazzSet = componentScanner.getComponents();
        clazzSet.forEach(clazz -> {
            String name = clazz.getName();
            beans.put(name, this.constructInstance(clazz));
        });
    }

    public Object getBean(String name) {
        return this.beans.get(name);
    }

    public Map<String, Object> getBeans() {
        return beans;
    }

    private Object constructInstance(Class<?> clazz) {
        try {
            return clazz.getConstructor().newInstance();
        } catch (Exception e) {
            throw new RuntimeException();
        }
    }
}
```

리플렉션을 이용해서 생성자를 가져와서 인스턴스를 만들고 이를 beans 에 넣어둔다. 컴포넌트 스캔을 해서 bean 생성을 하는 것이다. 지금은 파라미터도 없는 단순한 형태인데 나중에 'DI 프레임워크 구현' 에서 실제로 스프링에서 동작하는 의존성 주입의 방식을 재현한다.



## HandlerMapping

```java
public class AnnotationHandlerMapping implements HandlerMapping {

    private final Map<HandlerKey, HandlerExecution> handlerExecutions = Maps.newHashMap();

    private static final AnnotationHandlerMapping annotationHandlerMapping = new AnnotationHandlerMapping();

    public static AnnotationHandlerMapping getInstance() {
        return annotationHandlerMapping;
    }

    @Override
    public void init() {
        Map<String, Object> controllers = ApplicationContext.getInstance().getBeans();
        controllers.values()
                .forEach(this::setHandlerExecutions);
    }

    @Override
    public Object getHandler(HandlerKey handlerKey) {
        String requestUri = handlerKey.getUri();
        Object handler = handlerExecutions.get(handlerKey);
        if (Objects.nonNull(handler)) {
            return handler;
        }

        return handlerExecutions.entrySet().stream()
                .filter(entry -> {
                    String handlerUri = entry.getKey().getUri();
                    RequestMethod requestMethod = entry.getKey().getRequestMethod();
                    return (PathAnalyzer.isSamePattern(handlerUri, requestUri)) && (handlerKey.getRequestMethod().equals(requestMethod));
                })
                .map(Map.Entry::getValue)
                .findAny()
                .orElse(null);
    }

    public String getHandlerOriginUri(String requestUri) {
        return handlerExecutions.keySet().stream()
                .filter(handlerExecution -> {
                    String handlerUri = handlerExecution.getUri();
                    return PathAnalyzer.isSamePattern(handlerUri, requestUri);
                })
                .map(HandlerKey::getUri)
                .findAny()
                .orElse(null);
    }

    private void setHandlerExecutions(Object controller) {
        List<Method> methods = Arrays.stream(controller.getClass().getDeclaredMethods())
                .filter(method -> this.hasRequestMapping(method.getDeclaredAnnotations()))
                .collect(Collectors.toList());

        methods.forEach(method -> this.setHandlerExecution(method, controller));
    }

    private boolean hasRequestMapping(Annotation[] annotations) {
        return Arrays.stream(annotations)
                .anyMatch(annotation -> annotation.annotationType().equals(RequestMapping.class));
    }

    private void setHandlerExecution(Method method, Object ownerInstance) {
        Annotation requestMapping = Arrays.stream(method.getDeclaredAnnotations())
                .filter(annotation -> annotation.annotationType().equals(RequestMapping.class))
                .findAny()
                .orElseThrow(IllegalArgumentException::new);

        String url = this.getUrl(requestMapping);
        RequestMethod requestMethod = this.getMethod(requestMapping);
        if (Objects.isNull(url) || Objects.isNull(requestMethod)) {
            throw new IllegalArgumentException();
        }

        HandlerKey handlerKey = new HandlerKey(url, requestMethod);
        HandlerExecution handlerExecution = new HandlerExecution(ownerInstance, method);
        this.handlerExecutions.put(handlerKey, handlerExecution);
    }

    private String getUrl(Annotation requestMapping) {
        Class<?> clazz = requestMapping.annotationType();
        Method pathMethod = Arrays.stream(clazz.getDeclaredMethods())
                .filter(method -> method.getName().equals("value"))
                .findAny()
                .orElseThrow(IllegalArgumentException::new);

        try {
            return String.valueOf(pathMethod.invoke(requestMapping));
        } catch (IllegalAccessException | InvocationTargetException e) {
            e.printStackTrace();
        }

        return null;
    }

    private RequestMethod getMethod(Annotation requestMapping) {
        Class<?> clazz = requestMapping.annotationType();
        Method pathMethod = Arrays.stream(clazz.getDeclaredMethods())
                .filter(method -> method.getName().equals("method"))
                .findAny()
                .orElseThrow(IllegalArgumentException::new);

        try {
            return (RequestMethod) pathMethod.invoke(requestMapping);
        } catch (IllegalAccessException | InvocationTargetException e) {
            e.printStackTrace();
        }

        return null;
    }

}
```

이건 DispatcherServlet 에서 아래 코드를 사용할때 필요하다.

```java
private Object findHandler(HandlerKey handlerKey) throws ServletException {
    Object handler = AnnotationHandlerMapping.getInstance().getHandler(handlerKey);
    if (Objects.isNull(handler)) {
        throw new ServletException();
    }

    return handler;
}
```

간단한 문장으로 정리하자면 AnnotationHandlerMapping 에서 적절한 핸들러를 찾아 주는 것이다.



```java
@Override
public void init() {
    Map<String, Object> controllers = ApplicationContext.getInstance().getBeans();
    controllers.values()
            .forEach(this::setHandlerExecutions);
}
```

init 에서 기존에 ApplicationContext 에서 만든 bean들을 컨트롤러로 순회시키면서 아래와 같이 Handler 로 저장한다.



```java
private void setHandlerExecutions(Object controller) {
    List<Method> methods = Arrays.stream(controller.getClass().getDeclaredMethods())
            .filter(method -> this.hasRequestMapping(method.getDeclaredAnnotations()))
            .collect(Collectors.toList());

    methods.forEach(method -> this.setHandlerExecution(method, controller));
}

private boolean hasRequestMapping(Annotation[] annotations) {
    return Arrays.stream(annotations)
            .anyMatch(annotation -> annotation.annotationType().equals(RequestMapping.class));
}

private void setHandlerExecution(Method method, Object ownerInstance) {
    Annotation requestMapping = Arrays.stream(method.getDeclaredAnnotations())
            .filter(annotation -> annotation.annotationType().equals(RequestMapping.class))
            .findAny()
            .orElseThrow(IllegalArgumentException::new);

    String url = this.getUrl(requestMapping);
    RequestMethod requestMethod = this.getMethod(requestMapping);
    if (Objects.isNull(url) || Objects.isNull(requestMethod)) {
        throw new IllegalArgumentException();
    }

    HandlerKey handlerKey = new HandlerKey(url, requestMethod);
    HandlerExecution handlerExecution = new HandlerExecution(ownerInstance, method);
    this.handlerExecutions.put(handlerKey, handlerExecution);
}
```

메소드가 핸들러가 아닌 메소드가 있을 수 있으므로 어노테이션을 이용해서 핸들러가 아닌 것과 핸들러인 것들을 구분하는 것을 1차로 실행하고 url 과 method 를 분석해서 적절한 요청에 대한 적절한 핸들러를 구분해서 Map 에 저장한다.



```java
public HandlerKey(String uri, RequestMethod requestMethod) {
    this.uri = uri;
    this.requestMethod = requestMethod;
}
```

HandlerKey는 위와 같이 정의되어 있다. 이를 활용해서 '요청' 의 고유함을 정의하고 이를 기준으로 적절한 핸들러를 찾는다.



## HandlerAdapter

다시 언급하지만 이번 'MVC 프레임워크 구현'은 원리를 그대로 따랐을 뿐 클래스 명은 실제 스프링과는 조금씩 상이하다.

아래 코드는 DispatcherServlet 코드인데 흐름이 너무 길어서 다시 짚고자 첨부했다. 현재 들어온 요청에 대해서 적절한 handler 를 찾는 과정까지 왔다.

```java
@Override
protected void service(HttpServletRequest req, HttpServletResponse resp) throws ServletException {
    String requestUri = req.getRequestURI();
    logger.debug("Method : {}, Request URI : {}", req.getMethod(), requestUri);

    HandlerKey handlerKey = new HandlerKey(requestUri, RequestMethod.getRequestMethod(req.getMethod()));
    Object handler = this.findHandler(handlerKey);
    HandlerSpec handlerSpec = new HandlerSpec(handler, handlerKey, req, resp);
    try {
        ModelAndView modelAndView = ArgumentResolver.getInstance().invokeHandler(handlerSpec);
        modelAndView.render(req, resp);
    } catch (Throwable e) {
        logger.error("Exception : {}", e);
        throw new ServletException(e.getMessage());
    }
}
```



### ArgumentResolver

```java

public class ArgumentResolver {

    private final static ArgumentResolver argumentResolver = new ArgumentResolver();
    private final static Integer HANDLER_ARGUMENT_ANNOTATION_COUNT_LIMIT = 1;

    private ArgumentResolver() {

    }

    public static ArgumentResolver getInstance() {
        return argumentResolver;
    }

    public ModelAndView invokeHandler(HandlerSpec handlerSpec) throws Exception {
        HandlerExecution handlerExecution = (HandlerExecution) handlerSpec.getHandler();
        Method handler = handlerExecution.getMethod();

        List<Object> parameters = this.getParameters(handler.getParameters(), handlerSpec);
        return handlerExecution.handle(parameters);
    }

    public List<Object> getParameters(Parameter[] parameters, HandlerSpec handlerSpec) {
        return Arrays.stream(parameters)
                .map(parameter -> {
                    Annotation parameterAnnotation = this.getAnnotation(parameter);
                    try {
                        return this.getConvertedParameter(parameterAnnotation, parameter, handlerSpec);
                    } catch (JsonProcessingException e) {
                        throw new RuntimeException(e);
                    }
                }).collect(Collectors.toList());

    }

    private Object getConvertedParameter(Annotation parameterAnnotation, Parameter parameter, HandlerSpec handlerSpec) throws JsonProcessingException {
        HttpServletRequest httpServletRequest = handlerSpec.getHttpServletRequest();
        Class<?> parameterClass = parameter.getType();
        String parameterName = parameter.getName();

        if (parameterClass.getName().equals("javax.servlet.http.HttpServletRequest")) {
            return httpServletRequest;
        }

        if (Objects.isNull(parameterAnnotation)) {
            return this.getConvertedParamter(parameterClass, httpServletRequest.getParameter(parameterName));
        }

        if (parameterAnnotation.annotationType().equals(RequestBody.class)) {
            Map<String, String> parametersFromRequest = this.getParametersFromRequest(httpServletRequest, parameterClass);
            String data = ObjectMapperFactory.getObjectMapper().writeValueAsString(parametersFromRequest);
            return ObjectMapperFactory.getObjectMapper().readValue(data, parameterClass);
        }

        if (parameterAnnotation.annotationType().equals(PathVariable.class)) {
            String requestUri = httpServletRequest.getRequestURI();
            String handlerUri = AnnotationHandlerMapping.getInstance().getHandlerOriginUri(requestUri);
            return this.getPathVariable(requestUri, handlerUri, parameterName);
        }

        throw new IllegalArgumentException();
    }

    private Map<String, String> getParametersFromRequest(HttpServletRequest httpServletRequest, Class<?> parameterClass) {
        List<String> keys = Arrays.stream(parameterClass.getDeclaredFields())
                .map(Field::getName)
                .collect(Collectors.toList());

        Map<String, String> parametersFromRequest = new HashMap<>();
        keys.forEach(key -> {
            String value = httpServletRequest.getParameter(key);
            parametersFromRequest.put(key, value);
        });

        return parametersFromRequest;
    }

    private Object getConvertedParamter(Class<?> parameterClass, String value) {
        if (parameterClass.equals(int.class)) {
            return Integer.parseInt(value);
        }

        return value;
    }

    private Annotation getAnnotation(Parameter parameter) {
        Annotation[] annotations = parameter.getDeclaredAnnotations();
        if (annotations.length > HANDLER_ARGUMENT_ANNOTATION_COUNT_LIMIT) {
            throw new IllegalArgumentException();
        }

        return Arrays.stream(parameter.getDeclaredAnnotations())
                .findFirst()
                .orElse(null);
    }

    private String getPathVariable(String requestUri, String handlerUri, String targetParameter) {
        return PathAnalyzer.getVariables(handlerUri, requestUri, targetParameter);
    }

}

```



```java
public ModelAndView invokeHandler(HandlerSpec handlerSpec) throws Exception {
    HandlerExecution handlerExecution = (HandlerExecution) handlerSpec.getHandler();
    Method handler = handlerExecution.getMethod();

    List<Object> parameters = this.getParameters(handler.getParameters(), handlerSpec);
    return handlerExecution.handle(parameters);
}
```

이미 요청을 처리할 handler 를 찾는건 해놨으므로 이제 요청에 들어온 파라미터를 handler 가 받을 수 있는 형태로 변환해서 handler 에 전달 후 handler 를 call 하는 것이 필요하다. 위 코드는 이 과정을 나타내고 있다.



```java
public List<Object> getParameters(Parameter[] parameters, HandlerSpec handlerSpec) {
    return Arrays.stream(parameters)
            .map(parameter -> {
                Annotation parameterAnnotation = this.getAnnotation(parameter);
                try {
                    return this.getConvertedParameter(parameterAnnotation, parameter, handlerSpec);
                } catch (JsonProcessingException e) {
                    throw new RuntimeException(e);
                }
            }).collect(Collectors.toList());

}

private Object getConvertedParameter(Annotation parameterAnnotation, Parameter parameter, HandlerSpec handlerSpec) throws JsonProcessingException {
    HttpServletRequest httpServletRequest = handlerSpec.getHttpServletRequest();
    Class<?> parameterClass = parameter.getType();
    String parameterName = parameter.getName();

    if (parameterClass.getName().equals("javax.servlet.http.HttpServletRequest")) {
        return httpServletRequest;
    }

    if (Objects.isNull(parameterAnnotation)) {
        return this.getConvertedParamter(parameterClass, httpServletRequest.getParameter(parameterName));
    }

    if (parameterAnnotation.annotationType().equals(RequestBody.class)) {
        Map<String, String> parametersFromRequest = this.getParametersFromRequest(httpServletRequest, parameterClass);
        String data = ObjectMapperFactory.getObjectMapper().writeValueAsString(parametersFromRequest);
        return ObjectMapperFactory.getObjectMapper().readValue(data, parameterClass);
    }

    if (parameterAnnotation.annotationType().equals(PathVariable.class)) {
        String requestUri = httpServletRequest.getRequestURI();
        String handlerUri = AnnotationHandlerMapping.getInstance().getHandlerOriginUri(requestUri);
        return this.getPathVariable(requestUri, handlerUri, parameterName);
    }

    throw new IllegalArgumentException();
}
```

이것도 간단하게 한 문장으로 정리하자면 '어노테이션을 분석해서 적절하게 변환 처리를 해준다' 가 전부이다. 리플렉션을  적극적으로 활용해서 구현했다.

이렇게 '바퀴를 직접 다시 만드는' 과정을 통해서 기존 바퀴가 얼마나 대단하고 잘 만들어져있는가를 잘 알 수 있고, 기존 바퀴에 대한 이해를 깊게 만들 수 있었다.



## ViewResolver

```java
public ModelAndView invokeHandler(HandlerSpec handlerSpec) throws Exception {
    HandlerExecution handlerExecution = (HandlerExecution) handlerSpec.getHandler();
    Method handler = handlerExecution.getMethod();

    List<Object> parameters = this.getParameters(handler.getParameters(), handlerSpec);
    return handlerExecution.handle(parameters);
}
```

위 코드를 보면 HandlerExecution 이라는 것이 있는데 아래와 같다.



```java
public class HandlerExecution {

    private final Object locatedInstance;
    private final Method method;

    public HandlerExecution(Object locatedInstance, Method method) {
        this.locatedInstance = locatedInstance;
        this.method = method;
    }

    public ModelAndView handle(List<Object> parameters) throws Exception {
        if (parameters.isEmpty()) {
            return (ModelAndView) method.invoke(locatedInstance);
        }

        if (parameters.size() == 1) {
            return (ModelAndView) method.invoke(locatedInstance, parameters.get(0));
        }

        if (parameters.size() == 2) {
            return (ModelAndView) method.invoke(locatedInstance, parameters.get(0), parameters.get(1));
        }

        if (parameters.size() == 3) {
            return (ModelAndView) method.invoke(locatedInstance, parameters.get(0), parameters.get(1), parameters.get(2));
        }

        if (parameters.size() == 4) {
            return (ModelAndView) method.invoke(locatedInstance, parameters.get(0), parameters.get(1), parameters.get(2), parameters.get(3));
        }

        // TODO handler parameter 더 많은 경우
        throw new IllegalArgumentException();
    }

    public Method getMethod() {
        return method;
    }
}

```

지금 봐도 완성도가 썩 마음에 들진 않는다. 내가 'MVC 프레임워크 구현' 에서 만든 MVC 프레임워크는 위 코드처럼 파라미터가 5개 이상이면 에러가 나는 프레임워크인 것이다.(아무튼 원리 이해가 목적이다)

여기서 짚고 넘어가고 싶은 것은 모든 핸들러의 응답을 ModelAndView 으로 강제했다는 점이다. 일단 여기서는 하나의 타입으로 강제하고 내부 구현체가 세부 로직을 담당한다.



```java
@Controller
public class HomeController {
    @RequestMapping(value = "/", method = RequestMethod.GET)
    public ModelAndView index() {
        ModelAndView modelAndView = ModelAndView.getJspModelAndView("home.jsp");
        modelAndView.addObject("users", DataBase.findAll());
        return modelAndView;
    }

}
```

핸들러 하나를 예시로 가져왔다. 위와 같이 ModelAndView 를 반환한다. 모든 핸들러가 ModelAndView 를 반환하도록 처리해뒀기 때문에 HandlerExecution 에서 모두 ModelAndView 로 캐스팅 할 수 있다.



```java
public class ModelAndView {
    private View view;
    private final Map<String, Object> model = new HashMap<>();

    public ModelAndView() {
    }

    public ModelAndView(View view) {
        this.view = view;
    }

    public static ModelAndView getJspModelAndView(String viewName) {
        return new ModelAndView(new JspView(viewName));
    }

    public static ModelAndView redirectHome() {

        return new ModelAndView(new JspView("redirect:/"));
    }

    public ModelAndView addObject(String attributeName, Object attributeValue) {
        model.put(attributeName, attributeValue);
        return this;
    }

    public Object getObject(String attributeName) {
        return model.get(attributeName);
    }

    public Map<String, Object> getModel() {
        return Collections.unmodifiableMap(model);
    }

    public View getView() {
        return view;
    }

    public void render(HttpServletRequest request, HttpServletResponse response) throws Exception {
        this.view.render(model, request, response);
    }
}
```

내부에서 View 를 가지고 있다. 이건 굳이 패턴으로 치면 브릿지 패턴이다. 내부에서 View 라는 추상을 가지고 있고 이것의 구체적인 구현은 따로 빼서 구현체가 처리하며 객체 자체는 추상만 프로퍼티로 지니고 있다.



```java
public static ModelAndView getJspModelAndView(String viewName) {
    return new ModelAndView(new JspView(viewName));
}
```

핸들러에서 만들어준 JspView 를 보자.



```java
public class JspView implements View {

    private final String viewName;

    public JspView(String viewName) {
        this.viewName = viewName;
    }

    @Override
    public void render(Map<String, ?> model, HttpServletRequest request, HttpServletResponse response) throws Exception {
        ViewRenderer.getInstance().render(viewName, request, response);
    }

}
```

```java
public class ViewRenderer {
    private static final String DEFAULT_REDIRECT_PREFIX = "redirect:";

    private static final ViewRenderer viewRenderer = new ViewRenderer();

    private ViewRenderer() {

    }

    public static ViewRenderer getInstance() {
        return viewRenderer;
    }

    public void render(String viewName, HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        move(viewName, req, resp);
    }

    private void move(String viewName, HttpServletRequest req, HttpServletResponse resp)
            throws ServletException, IOException {
        if (viewName.startsWith(DEFAULT_REDIRECT_PREFIX)) {
            resp.sendRedirect(viewName.substring(DEFAULT_REDIRECT_PREFIX.length()));
            return;
        }

        RequestDispatcher rd = req.getRequestDispatcher(viewName);
        rd.forward(req, resp);
    }

}
```

위와 같이 viewName 을 기준으로 view 를 찾아서 응답으로 내보내게 된다.
