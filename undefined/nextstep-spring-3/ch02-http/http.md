# HTTP 파싱

## STEP 설명

과정에서는 TDD 실습의 기초적인 step 이었는데, 내용 자체가 의미가 있었다. 프레임워크에서 http 메세지를 어떻게 읽어서 주는지 제일 밑단부터 직접 만들어보는 과정이었기 때문이다. 이를 위해서 HTTP 메세지 구조에 대해서 다시 한번 짚어볼 필요가 있다.

### HTTP 메세지 구조

<figure><img src="../../../.gitbook/assets/image (4) (4) (1).png" alt=""><figcaption></figcaption></figure>

각 부분 모두 잘 파싱해야겠지만 특히 start line 의 정보를 가지고 이를 처리할 적절한 핸들러에 위임을 해야할 것이다. 아래는 강의 자료에 소개된 요청과 응답의 예시이다.

<figure><img src="../../../.gitbook/assets/image (4) (4).png" alt=""><figcaption></figcaption></figure>

이걸 파싱하는 로직을 개발하는 것이 step1 의 미션이었다.



## 요구사항

<figure><img src="../../../.gitbook/assets/image (5) (1) (5).png" alt=""><figcaption></figcaption></figure>

첫 미션이다보니 비교적 간단한 미션이다.



## [실습코드](https://github.com/fistkim101/jwp-was)

STEP1에 대한 최종 결과물인 테스트 코드는 아래와 같다.

```java
package utils;

import exception.HttpNotFoundException;
import exception.ProtocolNotFoundException;
import model.RequestLine;
import org.assertj.core.api.Assertions;
import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.params.ParameterizedTest;
import org.junit.jupiter.params.provider.ValueSource;

import java.util.HashMap;
import java.util.Map;

public class HttpParserTest {

    @DisplayName("RequestLine method, protocol, version 파싱 검증")
    @ParameterizedTest
    @ValueSource(strings = {"GET /users?userId=fistkim101&password=1004 HTTP/1.1", "POST /users?userId=fistkim101&password=1004 HTTP/1.1"})
    void parseRequestLineTest(String httpRequestFirstLine) {

        String[] requestLineData = httpRequestFirstLine.split(" ");
        String httpMethod = requestLineData[0];
        String protocolName = requestLineData[2].split("/")[0];
        String protocolVersion = requestLineData[2].split("/")[1];

        RequestLine requestLine = HttpParser.parseRequestLine(httpRequestFirstLine);

        Assertions.assertThat(httpMethod).isEqualTo(requestLine.getHttpMethod().toString());
        Assertions.assertThat(protocolName).isEqualTo(requestLine.getProtocol().getName());
        Assertions.assertThat(protocolVersion).isEqualTo(requestLine.getProtocol().getVersion());

    }

    @DisplayName("RequestLine path, queryParameters 파싱 검증")
    @ParameterizedTest
    @ValueSource(strings = {"GET /users?userId=fistkim101&password=1004 HTTP/1.1"})
    void parseRequestLineQueryParametersTest(String httpRequestFirstLine) {

        String path = "/users";
        Map<String, String> queryParameters = new HashMap<>(Map.of("userId", "fistkim101", "password", "1004"));

        RequestLine requestLine = HttpParser.parseRequestLine(httpRequestFirstLine);
        String parsedPath = requestLine.getUrlPath().getPath();
        Map<String, String> parsedQueryParameters = requestLine.getUrlPath().getQueryParameter().getParameters();

        Assertions.assertThat(path).isEqualTo(parsedPath);
        Assertions.assertThat(queryParameters.keySet().size()).isEqualTo(parsedQueryParameters.keySet().size());
        for (Map.Entry<String, String> entry : queryParameters.entrySet()) {
            String key = entry.getKey();
            String value = entry.getValue();

            Assertions.assertThat(value).isEqualTo(parsedQueryParameters.get(key));
        }

    }

    @DisplayName("RequestLine method 파싱시 에러케이스 검증")
    @ParameterizedTest
    @ValueSource(strings = {"GOT /users?userId=fistkim101&password=1004 HTTP/1.1", "PST /users?name=jk&phoneNumber=01012345678 HTTP/1.1"})
    void parseRequestLineHttpMethodExceptionTest(String httpRequestFirstLine) {

        Assertions.assertThatExceptionOfType(HttpNotFoundException.class)
                .isThrownBy(() -> HttpParser.parseRequestLine(httpRequestFirstLine));

    }

    @DisplayName("RequestLine protocol 파싱시 에러케이스 검증")
    @ParameterizedTest
    @ValueSource(strings = {"GET /users?userId=fistkim101&password=1004 HTTTTP/1.1", "POST /users?name=jk&phoneNumber=01012345678 TP/1.1"})
    void parseRequestLineProtocolExceptionTest(String httpRequestFirstLine) {

        Assertions.assertThatExceptionOfType(ProtocolNotFoundException.class)
                .isThrownBy(() -> HttpParser.parseRequestLine(httpRequestFirstLine));

    }

}
```



사실 문자열처리일 뿐이라서 크게 특별히 정리할 부분은 없지만 굳이 처리를 할 때 가장 신경을 쓴 부분은 객체를 최대한 잘게 쪼개었다는 것이다. 최대한 작게 쪼고 필요한 validation 이나 로직이 있는 경우 그 객체에 해당 책임을 부여하여 처리하도록 했다.

그래서 requestLine(start line) 도 따로 객체로 분류를 했고 이를 구성하는 객체들도 실제 HTTP 메세지 구조에 기반하여 HttpMethod, UrlPath, Protocol 으로 쪼개서 구성했다. 사실 너무 당연한 이야기다.

```java
public class RequestLine {
    private static final List<String> RESOURCE_FILE_EXTENSIONS = List.of(".css", ".js", ".ico", "ttf", "woff", "png");
    private final HttpMethod httpMethod;
    private final UrlPath urlPath;
    private final Protocol protocol;
    
    ....
}
```



테스트 코드에서 확인할 수 있듯이 가장 핵심이 되는 객체는 HttpParser 이다.

```java
public class HttpParser {

    public static final String REQUEST_LINE_SEPARATOR = " ";
    public static final String PATH_SEPARATOR = "\\?";
    public static final String QUERY_PARAMETER_SEPARATOR = "&";
    public static final String QUERY_PARAMETER_KEY_VALUE_SEPARATOR = "=";

    public static RequestLine parseRequestLine(String requestLine) {
        String[] requestLineData = requestLine.split(REQUEST_LINE_SEPARATOR);

        HttpMethod httpMethod = HttpMethod.find(requestLineData[0]);
        UrlPath path = getPath(requestLineData);
        Protocol protocol = getProtocol(requestLineData);

        return new RequestLine(httpMethod, path, protocol);
    }

    public static Map<String, String> convertStringToMap(String content) {
        String[] parameterKeyAndValues = content.split(HttpParser.QUERY_PARAMETER_SEPARATOR);
        Map<String, String> parameters = new HashMap<>();
        Arrays.stream(parameterKeyAndValues).forEach(parameter -> {
            String[] keyAndValue = parameter.split(HttpParser.QUERY_PARAMETER_KEY_VALUE_SEPARATOR);
            String parameterKey = keyAndValue[0];
            String parameterValue = keyAndValue[1];
            parameters.put(parameterKey, parameterValue);
        });

        return parameters;
    }

    private static UrlPath getPath(String[] requestLineData) {

        if (!requestLineData[1].contains("?")) {
            String path = requestLineData[1];
            return new UrlPath(path);
        }

        String path = requestLineData[1].split(PATH_SEPARATOR)[0];
        String queryString = requestLineData[1].split(PATH_SEPARATOR)[1];
        return new UrlPath(path, new QueryParameter(queryString));
    }

    private static Protocol getProtocol(String[] requestLineData) {
        String protocol = requestLineData[2];
        if (!StringUtils.hasText(protocol)) {
            throw new IllegalArgumentException();
        }

        return Protocol.find(protocol);
    }

}

```

RequestLine parseRequestLine(String requestLine) 에서 보면 RequestLine 을 구성하는 각 객체들을 파라미터로 받은  requestLine 에서 분리하여 파싱해서 각각 생성한뒤 최종적으로 RequestLine 객체를 만들고 있다.

queryParameters 를 파싱하는 로직의 핵심은 아래 메소드이다.

```java
public static Map<String, String> convertStringToMap(String content) {
    String[] parameterKeyAndValues = content.split(HttpParser.QUERY_PARAMETER_SEPARATOR);
    Map<String, String> parameters = new HashMap<>();
    Arrays.stream(parameterKeyAndValues).forEach(parameter -> {
        String[] keyAndValue = parameter.split(HttpParser.QUERY_PARAMETER_KEY_VALUE_SEPARATOR);
        String parameterKey = keyAndValue[0];
        String parameterValue = keyAndValue[1];
        parameters.put(parameterKey, parameterValue);
    });

    return parameters;
}
```

이를 이용해서 분리한 쿼리파라미터들을 일급컬렉션인 QueryParameter 에 넣어주고 QueryParameter 는 UrlPath 를 구성하는 객체가 된다.

```java
public class QueryParameter {

    private Map<String, String> parameters;

...

}
```

```java
    public UrlPath(String path) {
        this.path = path;
    }

    public UrlPath(String path, QueryParameter queryParameter) {
        this.path = path;
        this.queryParameter = queryParameter;
    }

```

**이 미션의 핵심은 모든 객체를 논리적으로 인식할 수 있는 최대한의 작고 의미있는 단위로 쪼개고 쪼개어서 각각을 정의해주고 책임을 분산시켜줬다는 것이 핵심이다.** 그리고 추후 손수 만들 프레임워크가 HTTP 메세지를 읽고 이를 파싱할 때 이러한 로직으로 할 것이라는 것을 미리 만들었다는 것에 부수적인 의미가 있다고 봤다.
