# Redis Annotation 및 설정

## 1) Redis configuration <a href="#1-redis-configuration" id="1-redis-configuration"></a>

### **.properties 추가 (아래는 docker로 redis를 기동하는 것을 기준으로 host를 결정)**

```
spring.cache.type=redis
spring.redis.host=172.17.0.1
spring.redis.port=6379
```

\


### **CacheManager 설정(Serializer 관련)**

```
@Configuration
public class CacheConfig extends CachingConfigurerSupport {

    @Autowired
    RedisConnectionFactory connectionFactory;

    @Bean
    public CacheManager redisCacheManager() {
        RedisCacheConfiguration redisCacheConfiguration = RedisCacheConfiguration.defaultCacheConfig()
                .serializeKeysWith(RedisSerializationContext.SerializationPair.fromSerializer(new StringRedisSerializer()))
                .serializeValuesWith(RedisSerializationContext.SerializationPair.fromSerializer(new GenericJackson2JsonRedisSerializer()));

        RedisCacheManager redisCacheManager = RedisCacheManager.RedisCacheManagerBuilder.
                fromConnectionFactory(connectionFactory).cacheDefaults(redisCacheConfiguration).build();

        return redisCacheManager;
    }

}
```

\


### **@EnableCaching 추가**

```java
@EnableJpaRepositories
@EnableJpaAuditing
@EnableCaching
@SpringBootApplication
public class SubwayApplication {

    public static void main(String[] args) {
        SpringApplication.run(SubwayApplication.class, args);
    }

}
```

\
\


## 2) @Cacheable <a href="#2-cacheable" id="2-cacheable"></a>

> Annotation indicating that the result of invoking a method (or all methods in a class) can be cached.

@Cacheable 어노테이션은 해당 메소드의 결과가 캐시에 저장된다는 것을 말한다. 해당 메소드의 결과가 ‘캐시가 가능’하다는 의미이다. (class 도 가능하다는 것으로 봐서는 class에 붙이면 전체 메소드에 적용이 되는 것으로 추측되지만 실험은 해보지 않았다)

\


> Each time an advised method is invoked, caching behavior will be applied, checking whether the method has been already invoked for the given arguments.

해당 메소드가 실행 될 때마다 caching 작용이 적용된다. 이게 무슨 의미냐면, 해당 파라미터들로 이미 실행이 한번 되었었는지 캐시를 확인해보고 캐시가 있으면 메소드를 실행시키지 않고 캐싱된 값을 return 한다는 의미이다.

\


> A sensible default simply uses the method parameters to compute the key, but a SpEL expression can be provided via the {@link #key} attribute, or a custom {@link org.springframework.cache.interceptor.KeyGenerator} implementation can replace the default one (see {@link #keyGenerator}).

기본적으로는 key를 생산해내기 위해서는 메소드의 파라미터들을 사용한다. 하지만 SpEL을 통해서 여러 속성들이 제공되니 그걸 이용할 수도 있고 custom한 KeyGenerator를 만들어서 기본 KeyGenerator를 대체하는 것도 가능하다.

\


> If no value is found in the cache for the computed key, the target method will be invoked and the returned value stored in the associated cache. Note that Java8’s {@code Optional} return types are automatically handled and its content is stored in the cache if present.

커스텀한 KeyGenerator를 설정해놨든, 그렇지 않아서 기본적인 KeyGenerator를 이용하게 되었든 key가 생성이 될 것이고 이 key로 캐시되어있는 값을 찾아보게 된다. 만약 해당 key로 캐시되어있는 값이 없다면 메소드가 실행될 것이고 return 되는 값이 해당 key의 value로 캐시로 저장될 것이다.

\


> This annotation may be used as a meta-annotation to create custom composed annotations with attribute overrides.

메타어노테이션을 지원하기 때문에 원하는 어노테이션들을 조합해서 커스텀한 어노테이션을 생성해서 사용하는 것이 가능하다.

\


복합적인 키를 사용할 때에는 깔끔하게 key를 지정해주는 것이 좋은 것 같다.

```java
    @Cacheable(value = "path", key = "{#source, #target}")
    public PathResponse findPath(Long source, Long target) {
        List<Line> lines = lineService.findLines();
        Station sourceStation = stationService.findById(source);
        Station targetStation = stationService.findById(target);
        SubwayPath subwayPath = pathService.findPath(lines, sourceStation, targetStation);

        return PathResponseAssembler.assemble(subwayPath);
    }
```

이 경우 key는 “path:1,2” 가 된다.

\


```java
    @Cacheable(value = "path")
    public PathResponse findPath(Long source, Long target) {
        List<Line> lines = lineService.findLines();
        Station sourceStation = stationService.findById(source);
        Station targetStation = stationService.findById(target);
        SubwayPath subwayPath = pathService.findPath(lines, sourceStation, targetStation);

        return PathResponseAssembler.assemble(subwayPath);
    }
```

이렇게 해줘도 두 파라미터 모두를 key 연산에 사용은 하지만 “path::SimpleKey \[1,2]” 이렇게 key가 생성되어서 보기에 깔끔하지 못한 것 같다.

\
\


## 3) @CachePut <a href="#3-cacheput" id="3-cacheput"></a>

> Annotation indicating that a method (or all methods on a class) triggers a {@link org.springframework.cache.Cache#put(Object, Object) cache put} operation.

해당 어노테이션이 있으면 해당 메소드 혹은 해당 클래스에 있는 모든 메소드가 org.springframework.cache.Cache#put(Object, Object) cache put 연산을 실행한다는 것을 의미한다.

\


> In contrast to the {@link Cacheable @Cacheable} annotation, this annotation does not cause the advised method to be skipped. Rather, it always causes the method to be invoked and its result to be stored in the associated cache.

@Cacheable 과는 대조적으로, 이 어노테이션은 해당 메소드가 skip되도록 하지는 않는다.(@Cacheable는 cache 값 보고 invoke 할지 말지 결정하는데 이건 그냥 무조건 invoke시킨다는 의미) 오히려 이 어노테이션은 항상 메소드가 실행되도록 하고 생성된 key 값으로 메소드의 결과를 캐시로 저장하도록 한다.

\


> This annotation may be used as a meta-annotation to create custom composed annotations with attribute overrides.

메타어노테이션을 지원하기 때문에 원하는 어노테이션들을 조합해서 커스텀한 어노테이션을 생성해서 사용하는 것이 가능하다.

\
\


## 4) @CacheEvict <a href="#4-cacheevict" id="4-cacheevict"></a>

> Annotation indicating that a method (or all methods on a class) triggers a {@link org.springframework.cache.Cache#evict(Object) cache evict} operation.

해당 어노테이션이 있으면 해당 메소드 혹은 해당 클래스에 있는 모든 메소드가 org.springframework.cache.Cache#evict(Object) cache evict 연산을 실행한다는 것을 의미한다.

\


> This annotation may be used as a meta-annotation to create custom composed annotations with attribute overrides.

메타어노테이션을 지원하기 때문에 원하는 어노테이션들을 조합해서 커스텀한 어노테이션을 생성해서 사용하는 것이 가능하다.

\
\


## 5) redis-cli <a href="#5-redis-cli" id="5-redis-cli"></a>

```
$ docker run -it --link [app container name]:redis --rm redis redis-cli -h redis -p 6379
```

\
