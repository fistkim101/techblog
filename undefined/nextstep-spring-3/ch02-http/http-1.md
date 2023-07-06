# HTTP 웹 서버 구현

코스에서는 여러 미션들을 나눠서 주고 이를 모두 달성하면 최종적으로 스프링 프레임워크 수준은 아니지만 어느정도 능력이 있는(?) 웹 서버가 구현되는 형태였다.

이미 코스를 수료한지가 1년이 넘었고, 이미 내가 모두 미션을 수행한 완성된 코드가 있기 때문에 세세하게 요구사항들을 다시 되짚어보는 것 보다는 완성한 웹서버에 요청이 들어오고 이를 어떤식으로 처리해서 응답을 내보내는지 이 흐름에 맞춰서 내가 구현한 로직을 세세하게 들여다 보는 것이 좋을 것 같다.



## 소켓을 통해서 외부 요청 최초 진입

```java
public class WebApplicationServer {
    private static final Logger logger = LoggerFactory.getLogger(WebApplicationServer.class);
    private static final int DEFAULT_PORT = 8080;

    public static void main(String[] args) throws Exception {

        try (ServerSocket listenSocket = new ServerSocket(getPort(args))) {
            Socket connection;
            while ((connection = listenSocket.accept()) != null) {
                ThreadConfiguration.serviceThreadPool.execute(new RequestHandler(connection));
            }
        }
    }

    private static Integer getPort(String[] args) {
        int port;
        if (args == null || args.length == 0) {
            port = DEFAULT_PORT;
        } else {
            port = Integer.parseInt(args[0]);
        }

        return port;
    }

}
```

제일 먼저 정해준 포트를 이용해서 소켓을 만들어준 뒤, 이를 이용해서 외부 요청에 대해서 리스닝 하고 있다가 요청이 들어오면 로직 처리를 시작한다. 이 때 톰캣의 스레드풀과 동일한 형태로 스레드풀을 만들어서 거기서 미리 만들어둔 스레드를 할당해서 로직을 처리하도록 하고 있다. 아래는 스레드풀 코드이다.



## 스레드풀(ThreadPoolExecutor)을 이용한 처리

```java
public class ThreadConfiguration {

    public static final Executor serviceThreadPool = new ThreadPoolExecutor(
            250,
            250,
            0,
            TimeUnit.MILLISECONDS,
            new LinkedBlockingQueue<>(100)
    );

//    public static final Executor serviceThreadPool = new ThreadPoolExecutor(
//            5,
//            5,
//            0,
//            TimeUnit.MILLISECONDS,
//            new LinkedBlockingQueue<>(2)
//    );

}
```

ThreadPoolExecutor 의 동작 원리를 잠깐 정리하자면 지정해준 corePoolSize 만큼의 스레드는 미리 만들어서 가지고 있고 이를 유지시키는데, 이를 넘어서는 요청이 들어오면 지정해준 maximumPoolSize 까지 스레드를 늘린다. 그리고 요청을 처리하면 maximumPoolSize - corePoolSize = N 개가 남을텐데 이것들은 지정해준 keepAliveTime 만큼만 살다가 만료되면 스레드가 종료된다. 그리고 corePoolSize 까지만 유지시킨다.

> ThreadPoolExecutor는 자바에서 스레드 풀을 구현하기 위해 제공되는 클래스입니다. ThreadPoolExecutor의 주요 프로퍼티와 동작 원리에 대해 설명드리겠습니다:
>
> 1. corePoolSize: 스레드 풀의 핵심 스레드 개수입니다. 이 개수 이상의 스레드는 유지되지 않으며, 필요에 따라 새로운 작업이 추가될 때 생성됩니다.
> 2. maximumPoolSize: 스레드 풀의 최대 스레드 개수입니다. 작업이 많아져서 corePoolSize를 넘어가는 경우, 최대 개수까지 스레드를 동적으로 생성합니다. maximumPoolSize를 초과하는 작업은 큐에 대기하게 됩니다.
> 3. keepAliveTime: 비활성 상태의 스레드가 코어 스레드 수보다 많은 경우, keepAliveTime 이후에 해당 스레드는 종료됩니다.
> 4. unit: keepAliveTime의 시간 단위를 지정합니다. 일반적으로 TimeUnit 클래스의 상수인 TimeUnit.SECONDS, TimeUnit.MINUTES 등을 사용합니다.
> 5. workQueue: 작업 큐입니다. 작업 큐는 스레드 풀에 제출된 작업들을 저장하고, 스레드가 작업을 처리하기 위해 대기하는 곳입니다. 작업 큐는 일반적으로 BlockingQueue 인터페이스를 구현하는 클래스로 사용됩니다.
> 6. threadFactory: 스레드를 생성하는 팩토리 객체입니다. 스레드의 이름, 우선 순위, 데몬 여부 등을 설정할 수 있습니다.
> 7. handler: 작업 큐가 가득 찼거나 스레드가 최대 개수에 도달했을 때 처리 방식을 정의합니다. 주요한 처리 방식으로는 기본적인 AbortPolicy, CallerRunsPolicy, DiscardPolicy, DiscardOldestPolicy가 있습니다.
>
> ThreadPoolExecutor는 위의 프로퍼티들을 조합하여 스레드 풀의 동작을 제어합니다. 작업이 스레드 풀에 제출되면, 스레드 풀은 코어 스레드 수에 도달하지 않은 경우에는 새로운 스레드를 생성하여 작업을 처리하고, 코어 스레드 수에 도달한 경우에는 작업을 큐에 저장합니다. 작업 큐가 가득 차면, 추가적인 작업은 버려지거나 특정 정책에 따라 처리됩니다.



## 소켓에서 맺은 connection 에서 요청 메세지 파싱

```java
public class RequestHandler implements Runnable {
    private static final Logger logger = LoggerFactory.getLogger(RequestHandler.class);

    private Socket connection;

    public RequestHandler(Socket connectionSocket) {
        this.connection = connectionSocket;
    }

    public void run() {
        logger.debug("New Client Connect! Connected IP : {}, Port : {}", connection.getInetAddress(), connection.getPort());

        try (InputStream inputStream = connection.getInputStream(); OutputStream outputStream = connection.getOutputStream()) {

            HttpRequestMessage httpRequestMessage = new HttpRequestMessage(new BufferedReader(new InputStreamReader(inputStream)));
            HttpResponseMessage httpResponseMessage = RequestService.getClientResponse(httpRequestMessage);

            DataOutputStream dataOutputStream = new DataOutputStream(outputStream);
            httpResponseMessage.sendResponse(dataOutputStream);
        } catch (IOException | URISyntaxException e) {
            logger.error(e.getMessage());
        } catch (InvocationTargetException | IllegalAccessException e) {
            e.printStackTrace();
        }
    }

}
```

[try-with-resources Statement](https://docs.oracle.com/javase/tutorial/essential/exceptions/tryResourceClose.html) 을 이용해서 Stream 을 선언하고 이를 이용해서 요청으로부터 들어온 메세지를 파싱한다. 파싱한 결과로 HttpRequestMessage 라는 객체가 만들어진다. 결론적으로 requestLine, header, body 를 분석해야 이거로 뭔가를 할 것이니까 그걸 하는 과정이다.

```java
package model;

import service.RequestService;
import utils.HttpParser;
import utils.IOUtils;

import java.io.BufferedReader;
import java.io.IOException;
import java.util.ArrayList;
import java.util.List;

public class HttpRequestMessage {

    private final String CONTENT_LENGTH_KEY = "Content-Length";

    private final RequestLine requestLine;

    private HttpHeaders requestHeaders;

    private HttpBody body;

    public HttpRequestMessage(BufferedReader bufferedReader) throws IOException {
        this(RequestService.getHttpMessageData(bufferedReader), bufferedReader);
    }

    public HttpRequestMessage(List<String> httpMessageData) throws IOException {
        this(httpMessageData, null);
    }

    public HttpRequestMessage(List<String> httpMessageData, BufferedReader bufferedReader) throws IOException {
        if (!(httpMessageData instanceof ArrayList)) {
            httpMessageData = new ArrayList<>(httpMessageData);
        }

        if (httpMessageData.isEmpty()) {
            throw new IllegalArgumentException();
        }

        this.requestLine = HttpParser.parseRequestLine(httpMessageData.remove(0));
        if (httpMessageData.isEmpty()) {
            return;
        }

        this.requestHeaders = new HttpHeaders(httpMessageData);
        this.body = new HttpBody(parseBody(bufferedReader, requestHeaders));
    }

    public RequestLine getRequestLine() {
        return requestLine;
    }

    public HttpHeaders getRequestHeaders() {
        return requestHeaders;
    }

    public String getBody() {
        return this.body.getContents();
    }

    public String toStringHttpMessage() {
        StringBuilder value = new StringBuilder();
        value.append("[" + "\n");
        value.append(this.requestLine.getInfo());
        value.append(this.requestHeaders.getInfo());
        value.append("\n");
        value.append(this.getBody());
        value.append("\n");
        value.append("]");

        return value.toString();
    }

    private String parseBody(BufferedReader bufferedReader, HttpHeaders requestHeaders) throws IOException {
        if (bufferedReader == null) {
            return null;
        }

        String contentLengthValue = requestHeaders
                .getHeaders()
                .get(CONTENT_LENGTH_KEY);
        int contentLength = this.getContentLength(contentLengthValue);
        if (contentLength == 0) {
            return null;
        }

        return IOUtils.readData(bufferedReader, contentLength);
    }

    private int getContentLength(String contentLengthValue) {
        if (contentLengthValue != null) {
            return Integer.parseInt(contentLengthValue);
        }

        return 0;
    }

}
```

**RequestService.**getHttpMessageData

```java
    public static List<String> getHttpMessageData(BufferedReader bufferedReader) throws IOException {
        String line = bufferedReader.readLine();
        List<String> data = new ArrayList<>();
        while (!line.equals(BODY_SEPARATOR)) {
            data.add(line);
            line = bufferedReader.readLine();
        }
        return data;
    }
```



## 요청 메세지 처리

```java
            HttpResponseMessage httpResponseMessage = RequestService.getClientResponse(httpRequestMessage);
```

```java
package service;

import model.*;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import types.HttpStatus;
import types.MediaType;
import utils.HandlerAdapter;

import java.io.BufferedReader;
import java.io.IOException;
import java.lang.reflect.InvocationTargetException;
import java.net.URI;
import java.net.URISyntaxException;
import java.util.ArrayList;
import java.util.List;

public class RequestService {

    private static final Logger logger = LoggerFactory.getLogger(RequestService.class);
    private static final String BODY_SEPARATOR = "";

    public static List<String> getHttpMessageData(BufferedReader bufferedReader) throws IOException {
        String line = bufferedReader.readLine();
        List<String> data = new ArrayList<>();
        while (!line.equals(BODY_SEPARATOR)) {
            data.add(line);
            line = bufferedReader.readLine();
        }
        return data;
    }

    public static HttpResponseMessage getClientResponse(HttpRequestMessage httpRequestMessage) throws IOException, URISyntaxException, InvocationTargetException, IllegalAccessException {

        RequestLine requestLine = httpRequestMessage.getRequestLine();
        if ((!requestLine.isSessionCheckExcludeUrls()) && (!hasSession(httpRequestMessage))) {
            return redirectHome();
        }

        if (requestLine.isRequestForFileResource(requestLine)) {
            return getFileResourceResponse(requestLine);
        }

        if (requestLine.isRequestForHtml()) {
            return getHtmlResponse(requestLine);
        }

        // logger.info("Request data >>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>> \n" + httpRequestMessage.toStringHttpMessage());
        AuthService.getInstance().setUserCredential(httpRequestMessage.getRequestHeaders());

        HttpResponseMessage httpResponseMessage = HandlerAdapter.getInstance().invoke(httpRequestMessage);

        AuthService.getInstance().removeUserCredential();

        return httpResponseMessage;
    }

    private static boolean hasSession(HttpRequestMessage httpRequestMessage) {
        HttpHeaders requestHeaders = httpRequestMessage.getRequestHeaders();
        String sessionId = requestHeaders.getSessionId();

        return (sessionId != null);
    }

    private static HttpResponseMessage redirectHome() {
        HttpHeaders httpHeaders = new HttpHeaders();
        httpHeaders.setContentType(MediaType.TEXT_HTML);
        httpHeaders.setLocation(URI.create("http://localhost:8080/").toString());
        return new HttpResponseMessage(HttpStatus.FOUND, httpHeaders);
    }

    private static HttpResponseMessage getHtmlResponse(RequestLine requestLine) throws IOException, URISyntaxException {
        UrlPath urlPath = requestLine.getUrlPath();
        HttpResponseMessage httpResponseMessage = new HttpResponseMessage(HttpStatus.OK, null);
        httpResponseMessage.setFileBody(urlPath, true);
        return httpResponseMessage;
    }

    private static HttpResponseMessage getFileResourceResponse(RequestLine requestLine) throws IOException, URISyntaxException {
        HttpHeaders httpHeaders = new HttpHeaders();
        httpHeaders.setContentType(MediaType.TEXT_CSS);

        UrlPath urlPath = requestLine.getUrlPath();
        HttpResponseMessage httpResponseMessage = new HttpResponseMessage(HttpStatus.OK, httpHeaders);
        httpResponseMessage.setFileBody(urlPath, false);

        return httpResponseMessage;
    }

}
```



인증까지 구현해둔 서버라서 인증절차가 있다.



```java
    RequestLine requestLine = httpRequestMessage.getRequestLine();
    if ((!requestLine.isSessionCheckExcludeUrls()) && (!hasSession(httpRequestMessage))) {
        return redirectHome();
    }
```

세션아이디의 존재유무를 확인해보고 없으면 홈으로 보내버리고 있다. rediectHome 을 보면 아래와 같은데

```java
private static HttpResponseMessage redirectHome() {
    HttpHeaders httpHeaders = new HttpHeaders();
    httpHeaders.setContentType(MediaType.TEXT_HTML);
    httpHeaders.setLocation(URI.create("http://localhost:8080/").toString());
    return new HttpResponseMessage(HttpStatus.FOUND, httpHeaders);
}
```

결국 '/' 로 다시 요청이 들어오게 되어 있다. 그런데 이건 isSessionCheckExcludeUrls 부분에서 보면 세션 체크를 하지 않는 url 이고 결국 아래 핸들러에 의해서 세션이 부여된다.

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



다음 분기문을 보자.

```java
    if (requestLine.isRequestForFileResource(requestLine)) {
        return getFileResourceResponse(requestLine);
    }
```

```java
private static final List<String> RESOURCE_FILE_EXTENSIONS = List.of(".css", ".js", ".ico", "ttf", "woff", "png");
```

확장자를 검사하고 이에 대한 요청일 경우 파일을 내보내준다.

```java
private static HttpResponseMessage getFileResourceResponse(RequestLine requestLine) throws IOException, URISyntaxException {
    HttpHeaders httpHeaders = new HttpHeaders();
    httpHeaders.setContentType(MediaType.TEXT_CSS);

    UrlPath urlPath = requestLine.getUrlPath();
    HttpResponseMessage httpResponseMessage = new HttpResponseMessage(HttpStatus.OK, httpHeaders);
    httpResponseMessage.setFileBody(urlPath, false);

    return httpResponseMessage;
}
```

```java
public void setFileBody(UrlPath urlPath, boolean isRequestForTemplate) throws IOException, URISyntaxException {
    this.bytesBody = FileIoUtils.loadFileFromClasspath(urlPath.getPath(), isRequestForTemplate);
}
```



```java
    if (requestLine.isRequestForHtml()) {
        return getHtmlResponse(requestLine);
    }
```

html 에 대한 요청일 경우 마찬가지로 파일로 응답을 구성해서 내보낸다.



여기까지 전부 통과 했다면 파일에 대한 요청이 아니면서 세션아이디를 가지고 있다는 것이다. 이제 세션 아이디를 이용해서 세션에서 보관중인 로그인 여부를 가지고 스레드 로컬에 값을 저장한다.

```java
    AuthService.getInstance().setUserCredential(httpRequestMessage.getRequestHeaders());
```

```java
package service;

import model.HttpHeaders;
import model.Session;
import model.SessionStorage;

public class AuthService {

    public static ThreadLocal<Boolean> userLogined = new ThreadLocal<>();
    private static final AuthService authService = new AuthService();

    private AuthService() {

    }

    public static AuthService getInstance() {
        return authService;
    }

    public void setUserCredential(HttpHeaders requestHeaders) {
        if (this.userLogined(requestHeaders)) {
            userLogined.set(true);
            return;
        }

        userLogined.set(false);
    }

    public void removeUserCredential() {
        userLogined.remove();
    }

    private boolean userLogined(HttpHeaders requestHeaders) {
        String sessionId = requestHeaders.getSessionId();
        if (sessionId == null) {
            return false;
        }

        Session userSession = SessionStorage.getInstance().getSession(sessionId);
        return userSession.isUserLogined();
    }

}
```



### 요청 메세지 처리 - 핸들러 찾아 처리를 위임

사실 여기서부터가 제일 재미있는 부분이다.

```java
    HttpResponseMessage httpResponseMessage = HandlerAdapter.getInstance().invoke(httpRequestMessage);
```



스프링이 처리하는 방식과 유사하지만 약간 다른 부분이 있다. 더 스프링에 가까운 웹서버는 CH03 @MVC 프레임워크 구현, CH05 DI 프레임워크 구현에서 구현했으니 거기서 정리하기로 한다.

```java
package utils;

import annotation.GetMapping;
import annotation.PostMapping;
import com.fasterxml.jackson.core.JsonProcessingException;
import configuration.HandlerConfiguration;
import exception.HttpNotFoundException;
import model.*;
import org.springframework.util.StringUtils;
import types.HttpMethod;

import java.lang.annotation.Annotation;
import java.lang.reflect.InvocationTargetException;
import java.lang.reflect.Method;
import java.lang.reflect.Parameter;
import java.util.*;
import java.util.stream.Collectors;

public class HandlerAdapter {

    private static final HandlerAdapter instance = new HandlerAdapter();

    private Map<Object, Handlers> handlers;

    private HandlerAdapter() {
    }

    public HttpResponseMessage invoke(HttpRequestMessage httpRequestMessage) throws InvocationTargetException, IllegalAccessException {
        RequestLine requestLine = httpRequestMessage.getRequestLine();

        if (this.handlers == null) {
            this.initHandlers();
        }

        HandlerPair handlerPair = find(requestLine);
        if (handlerPair == null) {
            return null;
        }

        return this.invokeHandler(handlerPair, httpRequestMessage);
    }

    public static HandlerAdapter getInstance() {
        return instance;
    }

    private HttpResponseMessage invokeHandler(HandlerPair handlerPair, HttpRequestMessage httpRequestMessage) throws InvocationTargetException, IllegalAccessException {
        RequestLine requestLine = httpRequestMessage.getRequestLine();
        Object controller = handlerPair.getController();
        Method handler = handlerPair.getHandler();

        List<Parameter> parameters = Arrays.stream(handler.getParameters())
                .collect(Collectors.toList());

        List<?> convertedParameters = parameters.stream()
                .filter(parameter -> !(parameter.getType().equals(HttpHeaders.class)))
                .map(parameter -> {
                    Class<?> parameterClass = parameter.getType();
                    String data = null;
                    try {
                        data = this.getParameterData(requestLine.getHttpMethod(), httpRequestMessage);
                        return this.getParaMeterObject(data, parameterClass);
                    } catch (JsonProcessingException e) {
                        throw new RuntimeException(e);
                    }
                })
                .collect(Collectors.toList());

        this.removeHeaderParameter(convertedParameters);
        if ((this.parameterContainsHeader(parameters))) {
            return this.getHttpResponseMessageWithRequestHeader(httpRequestMessage.getRequestHeaders(), handlerPair, convertedParameters);
        }

        return this.getHttpResponseMessage(handlerPair, convertedParameters);
    }

    private HttpResponseMessage getHttpResponseMessage(HandlerPair handlerPair, List<?> convertedParameters) throws IllegalAccessException, InvocationTargetException {
        Object controller = handlerPair.getController();
        Method handler = handlerPair.getHandler();

        if (convertedParameters == null || convertedParameters.isEmpty()) {
            return (HttpResponseMessage) handler.invoke(controller);
        }

        if (convertedParameters.size() == 1) {
            return (HttpResponseMessage) handler.invoke(controller, convertedParameters.get(0));
        }

        if (convertedParameters.size() == 2) {
            return (HttpResponseMessage) handler.invoke(controller, convertedParameters.get(0), convertedParameters.get(1));
        }

        throw new IllegalArgumentException();
    }

    private HttpResponseMessage getHttpResponseMessageWithRequestHeader(HttpHeaders requestHeaders, HandlerPair handlerPair, List<?> convertedParameters) throws IllegalAccessException, InvocationTargetException {
        Object controller = handlerPair.getController();
        Method handler = handlerPair.getHandler();

        if (convertedParameters == null || convertedParameters.isEmpty()) {
            return (HttpResponseMessage) handler.invoke(controller, requestHeaders);
        }

        if (convertedParameters.size() == 1) {
            return (HttpResponseMessage) handler.invoke(controller, requestHeaders, convertedParameters.get(0));
        }

        if (convertedParameters.size() == 2) {
            return (HttpResponseMessage) handler.invoke(controller, requestHeaders, convertedParameters.get(0), convertedParameters.get(1));
        }

        throw new IllegalArgumentException();
    }

    private void removeHeaderParameter(List<?> convertedParameters) {
        HttpHeaders requestHeader = (HttpHeaders) convertedParameters.stream()
                .filter(parameter -> parameter.getClass().equals(HttpHeaders.class))
                .findAny()
                .orElse(null);
        if (requestHeader == null) {
            return;
        }

        convertedParameters.remove(requestHeader);
    }


    private boolean parameterContainsHeader(List<Parameter> parameters) {
        return parameters.stream()
                .anyMatch(parameter -> parameter.getType().equals(HttpHeaders.class));
    }

    private Object getParaMeterObject(String data, Class<?> parameterClass) throws JsonProcessingException {
        if (data == null) {
            return null;
        }

        return ObjectMapperFactory.getObjectMapper().readValue(data, parameterClass);
    }

    private String getParameterData(HttpMethod httpMethod, HttpRequestMessage httpRequestMessage) throws JsonProcessingException {
        UrlPath urlPath = httpRequestMessage.getRequestLine().getUrlPath();
        QueryParameter queryParameter = urlPath.getQueryParameter();

        if (httpMethod == HttpMethod.GET && queryParameter != null) {
            return ObjectMapperFactory.getObjectMapper().writeValueAsString(queryParameter.getParameters());
        }

        String body = httpRequestMessage.getBody();
        if (httpMethod == HttpMethod.POST && StringUtils.hasText(body)) {
            return ObjectMapperFactory.getObjectMapper().writeValueAsString(HttpParser.convertStringToMap(httpRequestMessage.getBody()));
        }

        return null;
    }

    private void initHandlers() {
        handlers = new HashMap<>();
        List<Object> controllers = HandlerConfiguration.getInstance().getControllers();
        controllers.forEach(controller -> {
            List<Method> methods = List.of(controller.getClass().getDeclaredMethods());
            handlers.put(controller, new Handlers(methods));
        });
    }

    private HandlerPair find(RequestLine requestLine) {

        List<HandlerPair> handlerPairs = new ArrayList<>();
        for (Map.Entry<Object, Handlers> entry : handlers.entrySet()) {
            Object controller = entry.getKey();
            Handlers handlers = entry.getValue();
            Method handler = handlers.getHandlers()
                    .stream()
                    .filter(method -> findHandler(requestLine, method))
                    .findAny()
                    .orElse(null);
            this.collectHandlerPair(handlerPairs, controller, handler);
        }

        if (handlerPairs.size() != 1) {
            // TODO handler not found exception 정의
            return null;
        }

        return handlerPairs.get(0);
    }

    private void collectHandlerPair(List<HandlerPair> handlerPairs, Object controller, Method handler) {
        if (handler != null) {
            handlerPairs.add(new HandlerPair(controller, handler));
        }
    }

    private boolean findHandler(RequestLine requestLine, Method method) {
        HttpMethod httpMethod = requestLine.getHttpMethod();
        String path = requestLine.getUrlPath().getPath();

        String postFix = "Mapping";
        String httpMethodName = httpMethod.name();
        String annotationTypeName = httpMethodName.substring(0, 1).toUpperCase() + httpMethodName.toLowerCase().substring(1) + postFix;
        Class<?> targetAnnotation = this.getAnnotationType(annotationTypeName);

        return Arrays.stream(method.getAnnotations())
                .anyMatch(annotation -> {
                    boolean isEqualAnnotation = this.isEqualAnnotation(annotation, targetAnnotation);

                    String definedPath = this.getPath(annotation);
                    boolean isEqualPath = Objects.equals(definedPath, path);

                    return isEqualAnnotation && isEqualPath;
                });
    }

    private boolean isEqualAnnotation(Annotation annotation, Class<?> targetAnnotation) {
        Class<?> clazz = annotation.annotationType();
        return clazz.getName().equals(targetAnnotation.getName());
    }

    private Class<?> getAnnotationType(String annotationTypeName) {
        if (annotationTypeName.equals("GetMapping")) {
            return GetMapping.class;
        }

        if (annotationTypeName.equals("PostMapping")) {
            return PostMapping.class;
        }

        throw new HttpNotFoundException();
    }

    private String getPath(Annotation annotation) {
        Class<?> clazz = annotation.annotationType();
        Method pathMethod = Arrays.stream(clazz.getDeclaredMethods())
                .filter(method -> method.getName().equals("path"))
                .findAny()
                .orElseThrow(HttpNotFoundException::new);

        try {
            return String.valueOf(pathMethod.invoke(annotation));
        } catch (IllegalAccessException | InvocationTargetException e) {
            e.printStackTrace();
        }

        return null;
    }

}

```

딱 핵심내용을 정리하자면 '요청을 분석하여 미리 등록해둔 핸들러를 리플렉션을 사용하여 어노테이션, url 등을 분석해서 요청을 처리하도록 구현된 핸들러를 찾아서 이 핸들러에 처리를 위임하고 응답을 return 해주는 것'이다.

이 과정이 리플렉션을 사용해서 여러가지 경우에 대해서 처리를 해야하기 때문에 다소 코드가 길어지고 메소드가 여러개가 나왔다.



## 응답 데이터 전송

```java
public void run() {
    logger.debug("New Client Connect! Connected IP : {}, Port : {}", connection.getInetAddress(), connection.getPort());

    try (InputStream inputStream = connection.getInputStream(); OutputStream outputStream = connection.getOutputStream()) {

        HttpRequestMessage httpRequestMessage = new HttpRequestMessage(new BufferedReader(new InputStreamReader(inputStream)));
        HttpResponseMessage httpResponseMessage = RequestService.getClientResponse(httpRequestMessage);

        DataOutputStream dataOutputStream = new DataOutputStream(outputStream);
        httpResponseMessage.sendResponse(dataOutputStream);
    } catch (IOException | URISyntaxException e) {
        logger.error(e.getMessage());
    } catch (InvocationTargetException | IllegalAccessException e) {
        e.printStackTrace();
    }
}
```

다시 RequestHandler 로 나왔다. '요청 메세지 처리' 부분에서 아래 코드를 본 것이다.

```java
        HttpResponseMessage httpResponseMessage = RequestService.getClientResponse(httpRequestMessage);
```

HttpResponseMessage 라는 동일한 타입으로 반환이 가능한 이유는 아래와 같이 응답이 이 타입으로 고정되어 있기 때문이다.

```java
public class UserController {

    @GetMapping(path = "/user")
    public HttpResponseMessage getUserTest() throws IOException {
        return new HttpResponseMessage(HttpStatus.OK, null, "getUserTest");
    }

    @GetMapping(path = "/user/create")
    public HttpResponseMessage createUserGet(User user) throws IOException {
        DataBase.addUser(user);
        return new HttpResponseMessage(HttpStatus.OK, null, user);
    }

    @PostMapping(path = "/user/create")
    public HttpResponseMessage createUserPost(User user) throws IOException {
        return UserService.getInstance().createUser(user);
    }

    @PostMapping(path = "/user/login")
    public HttpResponseMessage login(HttpHeaders requestHeaders, Credential credential) throws IOException {
        return UserService.getInstance().auth(requestHeaders, credential);
    }

    @GetMapping(path = "/user/logout")
    public HttpResponseMessage logout(HttpHeaders requestHeaders) {
        return UserService.getInstance().logout(requestHeaders);
    }

    @GetMapping(path = "/user/list")
    public HttpResponseMessage getUsers() throws IOException {
        return UserService.getInstance().getUsers();
    }

}
```

이번 CH02 HTTP 이해 - 웹 서버 구현 에서는 완벽한 스프링을 재현하기 보다는 동작하는 HTTP 웹 서버를 큰 틀에서 구현해봄으로써 원리를 이해하는 것이 목적이기 때문에 아무래도 스프링과 이런 부분에서 거리가 좀 있다. 스프링을 만점 답안으로 본다면 완성도가 떨어진다고 볼 수 있다.

아무튼 저렇게 응답을 받아서 스트림을 통해 응답을 처리한다.

```java
package model;

import types.HttpStatus;
import utils.FileIoUtils;
import utils.IOUtils;

import java.io.DataOutputStream;
import java.io.IOException;
import java.io.OutputStream;
import java.net.URISyntaxException;

public class HttpResponseMessage {
    private final HttpStatus responseHttpStatus;
    private HttpHeaders responseHeaders;
    private byte[] bytesBody;

    public HttpResponseMessage(HttpStatus responseHttpStatus) {
        this.responseHttpStatus = responseHttpStatus;
    }

    public HttpResponseMessage(HttpStatus responseHttpStatus, HttpHeaders responseHeaders) {
        this.responseHttpStatus = responseHttpStatus;
        this.responseHeaders = responseHeaders;
    }

    public HttpResponseMessage(HttpStatus responseHttpStatus, HttpHeaders responseHeaders, Object body) throws IOException {
        this.responseHttpStatus = responseHttpStatus;
        this.responseHeaders = responseHeaders;
        this.bytesBody = IOUtils.bodyToBytes(body);
    }

    public HttpStatus getResponseHttpStatus() {
        return responseHttpStatus;
    }

    public HttpHeaders getResponseHeaders() {
        return responseHeaders;
    }

    public byte[] getBytesBody() {
        return bytesBody;
    }

    public void setFileBody(UrlPath urlPath, boolean isRequestForTemplate) throws IOException, URISyntaxException {
        this.bytesBody = FileIoUtils.loadFileFromClasspath(urlPath.getPath(), isRequestForTemplate);
    }

    public void sendResponse(OutputStream outputStream) throws IOException {
        DataOutputStream dataOutputStream = new DataOutputStream(outputStream);
        this.makeResponseHeader(dataOutputStream);
        this.makeResponseBody(dataOutputStream);
    }

    private void makeResponseHeader(DataOutputStream dataOutputStream) throws IOException {
        byte[] body = this.bytesBody;

        HttpStatus responseHttpStatus = this.getResponseHttpStatus();
        HttpHeaders responseHeaders = this.getResponseHeaders();

        dataOutputStream.writeBytes(String.format("HTTP/1.1 %s %s\r\n", responseHttpStatus.getCode(), responseHttpStatus.name()));
        if (body != null) {
            dataOutputStream.writeBytes("Content-Length: " + body.length + "\r\n");
        }

        if (responseHeaders != null) {
            responseHeaders.getHeaders().keySet()
                    .forEach(key -> {
                        try {
                            String headerValue = responseHeaders.getHeaders().get(key);
                            dataOutputStream.writeBytes(key + ": " + headerValue + "\r\n");
                        } catch (IOException e) {
                            e.printStackTrace();
                        }
                    });
        }
        dataOutputStream.writeBytes("\r\n");
    }

    private void makeResponseBody(DataOutputStream dataOutputStream) throws IOException {
        byte[] body = this.bytesBody;
        if (body == null) {
            dataOutputStream.flush();
            return;
        }

        dataOutputStream.write(body, 0, body.length);
        dataOutputStream.flush();
    }

}
```
