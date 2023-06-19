# WebAsyncManagerIntegrationFilter

## WebAsyncManagerIntegrationFilter

스프링 MVC의 Async 기능(핸들러에서 Callable을 리턴할 수 있는 기능)을 사용할 때에도 SecurityContext를 공유하도록 도와주는 필터.

* PreProcess: SecurityContext를 설정한다.
* Callable: 비록 다른 쓰레드지만 그 안에서는 동일한 SecurityContext를 참조할 수 있다.
* PostProcess: SecurityContext를 정리(clean up)한다.

```java
@GetMapping("/async-handler")
@ResponseBody
public Callable<String> asyncHandler() {
    SecurityLogger.log("MVC");
    return () -> {
        SecurityLogger.log("Callable");
        return "Async Handler";
    };
}
```

이 필터는 오직 Callable 을 리턴 할 때 의미가 있는 것인데, Callable 리턴할 일이 개인적으로는 없었어서 크게 의미가 있을지는 모르겠다. 하지만 아무튼 SecurityContextHolder가 ThreadLocal 을 사용하고 있기 때문에 다른 스레드에서 원칙적으로 사용이 안되는 것이 맞지만 위와 같은 경우에 WebAsyncManagerIntegrationFilter 덕분에 하위 스레드에서도 SecurityContextHolder 가 공유가 된다.



## 스프링 시큐리티와 @Async

&#x20;SecurityContextHolder는 ThreadLocal 을 사용하고 있고, ThreadLocal 은 동일 스레드 스코프 내에서 공유되므로 기본적으로는 다른 스레드가 참조하지 못한다. 그래서 기본적으로 아래와 같은 경우 SecurityContextHolder 가 공유되지 않는 것이 맞다.

```java
@GetMapping("/async-service")
@ResponseBody
public String asyncService() {
    SecurityLogger.log("MVC, before async service");
    sampleService.asyncService();
    SecurityLogger.log("MVC, after async service");
    return "Async Service";
}

@Service
public class SampleService {
    @Async
    public void asyncService() {
        SecurityLogger.log("Async Service");
        System.out.println("Async service is called.");
    }
}
```



하지만 아래와 같이 SecurityContextHolder 의 사용 전략을 설정해주면 하위 스레드에서도 SecurityContextHolder 가 공유된다. SecurityContext를 자식 스레드에도 공유하는 전략이다.

```java
SecurityContextHolder.setStrategyName(SecurityContextHolder.MODE_INHERITABLETHREADLOCAL);
```
