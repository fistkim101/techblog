# 캐싱 활용하기

## 1. ETag 란? <a href="#1-etag" id="1-etag"></a>

당연한 이야기이지만 특정한 사이트에 접속 했을 때 사용자 입장에서 해당 사이트가 빠르면 빠를 수록 당연히 좋다. 이때의 속도에 영향을 미치는 것은 여러가지 요인이 있을 수 있지만 브라우저가 띄워줄 여러 자료들 중 클라이언트가 일부를 ‘미리 갖고 있다면’ 해당 자료는 다시 받아오지 않는 것이 속도 관리 측면에서 유리할 것이다. 이 때 사용되는 개념이 [웹 캐시](https://ko.wikipedia.org/wiki/%EC%9B%B9\_%EC%BA%90%EC%8B%9C) 라는 개념이다.

캐시를 사용함에 있어서 미리 갖고 있다고 해서 항상 갖고 있는 것만 보여준다면, 갱신이 필요한 경우에 갱신이 이뤄지지 않아서 문제가 있을 것이다. 그래서 클라이언트는 ‘나 이거 갖고 있는데 그대로 이거 보여줘도 돼? 바뀐거 없어?’ 라고 물어보는 행위를 해야 캐시를 적절하게 사용한다고 할 수 있는데, 이 물어보는 절차가 바로 ‘캐시 유효성 검사’ 이다.

[ETag](https://en.wikipedia.org/wiki/HTTP\_ETag) 는 이 ‘캐시 유효성 검사’에 사용되는 도구이다. ETag는 태그라는 의미 그대로 특수한 값을 태그처럼 달아서 보내서 If-None-Match 또는 If-Match 조건에 따라서 값을 갱신할지 말지를 판단한다.

\
\


## 2. cache control <a href="#2-cache-control" id="2-cache-control"></a>

강의에서는 헷갈릴 만한 cache control 정책에 대해서 설명하고 있다. 강의에서 참조하셨다고 밝힌 레퍼런스는 [이것](https://web.dev/http-cache/) 이다.

\


### **no-cache vs no-store**

```
Cache-Control: no-cache, no-store
```

> * ‘no-cache’는 매 요청마다 중간에 있는 캐시 서버들은 ETag를 통해 자원의 유효성 확인하는데요. Cache-Control의 max-age=0과 같아요.\
>
> * ‘no-store’는 자원을 캐시하지 않아요.\
>
> * no-cache는 최신 상태로 유지(로컬 저장소에 저장하는 것을 막지는 않아, 변경이 있을 경우만 바로 캐시에 업데이트하는 방식), no-store는 저장소에 저장되는 것을 막아 데이터가 유출되는 것을 막기 위함이에요.\
>

### &#x20;**public vs. private**

```
Cache-Control: private, max-age=600
```

> * ‘public’은 중간 단계를 포함해 모든 캐시 서버에 캐시가 가능합니다.\
>
> * ‘private’은 요청한 사용자만 캐시할 수 있어요. 즉, CDN 같은 범용 캐시서버에서도 캐시할 수 있긴 하지만, 그 응답을 모든 사용자에게 공유할 수는 없어서 결국 최종 사용자의 브라우저에서만 응답을 캐시할 수 있어요.\
>
> * Cache-control: private 이 개인정보를 보호한다는 의미는 아니에요.\
>

\


다음은 강의에서 소개된 cache 관련 정책 결정의 기준이 될 수 있는 의사결정 트리이다.

<figure><img src="http://localhost:4000/assets/images/infra/cache-policy-tree.png" alt=""><figcaption></figcaption></figure>



\
\


## 3. cache 실습 <a href="#3-cache" id="3-cache"></a>

간단하게 ETag 가 나오도록 필터를 추가하여 실습해보았다.

```java
@Configuration
public class AppConfig implements WebMvcConfigurer {

    @Bean
    public FilterRegistrationBean<ShallowEtagHeaderFilter> shallowEtagHeaderFilter() {
        FilterRegistrationBean<ShallowEtagHeaderFilter> filterRegistrationBean
                = new FilterRegistrationBean<>(new ShallowEtagHeaderFilter());
        filterRegistrationBean.addUrlPatterns("/*");
        return filterRegistrationBean;
    }

}
```

filter의 형태로 구현하는 것은 생각해보면 매우 당연한거다. 왜냐면 최초로 클라이언트와 서버가 접촉하는 지점이기 때문에 저기서 더 깊이 request가 들어가느냐, 아니냐로 판단하는 것이 spring 구조상 당연하기 때문이다.

디테일하게 url 별로 cache control 을 해야할 경우 쓰게 될때 더 찾아보고 실습 코드를 추가할 예정이다.

\
