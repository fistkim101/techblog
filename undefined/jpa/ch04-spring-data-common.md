# CH04 Spring Data Common

## Spring Data 구성 및 학습순서 <a href="#spring-data" id="spring-data"></a>

![](https://fistkim101.github.io/images/concept-spring-data-common.png)

<figure><img src="https://fistkim101.github.io/images/concept-spring-data-repository-hierarchy.png" alt=""><figcaption></figcaption></figure>

김영한님 도서 및 강의 정리에서 이미 다룬 내용들이라서 자세한 정리는 생략한다.

## AbstractAggregateRoot 이용하여 도메인 이벤트 발생시키기 <a href="#abstractaggregateroot" id="abstractaggregateroot"></a>

기존에 ApplicationEventPublisher 를 통해서 수행한 Event 발행 방식을 AbstractAggregateRoot 를 이용해서 도메인 내에서 바로 발생시킬 수 있다.

```java
/**
 * Convenience base class for aggregate roots that exposes a {@link #registerEvent(Object)} to capture domain events and
 * expose them via {@link #domainEvents()}. The implementation is using the general event publication mechanism implied
 * by {@link DomainEvents} and {@link AfterDomainEventPublication}. If in doubt or need to customize anything here,
 * rather build your own base class and use the annotations directly.
 *
 * @author Oliver Gierke
 * @author Christoph Strobl
 * @since 1.13
 */
public class AbstractAggregateRoot<A extends AbstractAggregateRoot<A>> {

	private transient final @Transient List<Object> domainEvents = new ArrayList<>();

	/**
	 * Registers the given event object for publication on a call to a Spring Data repository's save methods.
	 *
	 * @param event must not be {@literal null}.
	 * @return the event that has been added.
	 * @see #andEvent(Object)
	 */
	protected <T> T registerEvent(T event) {

		Assert.notNull(event, "Domain event must not be null");

		this.domainEvents.add(event);
		return event;
	}

	/**
	 * Clears all domain events currently held. Usually invoked by the infrastructure in place in Spring Data
	 * repositories.
	 */
	@AfterDomainEventPublication
	protected void clearDomainEvents() {
		this.domainEvents.clear();
	}

	/**
	 * All domain events currently captured by the aggregate.
	 */
	@DomainEvents
	protected Collection<Object> domainEvents() {
		return Collections.unmodifiableList(domainEvents);
	}

	/**
	 * Adds all events contained in the given aggregate to the current one.
	 *
	 * @param aggregate must not be {@literal null}.
	 * @return the aggregate
	 */
	@SuppressWarnings("unchecked")
	protected final A andEventsFrom(A aggregate) {

		Assert.notNull(aggregate, "Aggregate must not be null");

		this.domainEvents.addAll(aggregate.domainEvents());

		return (A) this;
	}

	/**
	 * Adds the given event to the aggregate for later publication when calling a Spring Data repository's save-method.
	 * Does the same as {@link #registerEvent(Object)} but returns the aggregate instead of the event.
	 *
	 * @param event must not be {@literal null}.
	 * @return the aggregate
	 * @see #registerEvent(Object)
	 */
	@SuppressWarnings("unchecked")
	protected final A andEvent(Object event) {

		registerEvent(event);

		return (A) this;
	}
}
```

수행하는 과정에 domainEvents() 에 임시로 담겼다가 이벤트 발생 이후에는 메모리 누수 방지를 위해서 clearDomainEvents() 를 통해 제거된다. 위 모든 과정은 save() 와 동시에 발생된다.

```java
@Entity
public class Post extends AbstractAggregateRoot<Post> {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;

    private String description;

    public void setName(String name) {
        this.name = name;
    }

    public void setDescription(String description) {
        this.description = description;
    }

    public String getName() {
        return name;
    }

    public String getDescription() {
        return description;
    }

    // 여기서 event를 AbstractAggregateRoot를 통해서 등록한다. 
    public void registerEvent() {
        PostPublishedEvent postPublishedEvent = new PostPublishedEvent(this);
        this.registerEvent(postPublishedEvent);
    }
}
```

```java
    @Override
    @Transactional
    public void run(ApplicationArguments args) throws Exception {
        Post post = new Post();
        post.setName("event publish test");
        post.setDescription("event description");

        post.registerEvent(); // <-- 도메인 내에서 선언한 event 등록을 하면
        postRepository.save(post); // <-- save 시점에 publish 된다.
    }
```

위와 같이 처리하기전에 리스너 등록을 해두고 원하는 처리를 구성해두면 된다. 위와 같은 방식은 DDD 구현에 더 용이한 것 같다.

## Spring Data Common Web 기능 <a href="#spring-data-common-web" id="spring-data-common-web"></a>

Spring Data Common 이 제공해주는 Web support 기능들에 대해서 알아본다.

### Domain Class Converter <a href="#domain-class-converter" id="domain-class-converter"></a>

```java
public class DomainClassConverter<T extends ConversionService & ConverterRegistry>
		implements ConditionalGenericConverter, ApplicationContextAware
```

가 Converter Registry 에 들어가서 핸들러의 파라미터를 컨버팅 하는 용도로 사용된다. 내부적으로 DomainClassConverter내에 선언된 아래 두 컨버터가 사용된다. 컨버팅 과정에서 Repository 를 사용해 조회한다.

```java
	private Optional<ToEntityConverter> toEntityConverter = Optional.empty();
	private Optional<ToIdConverter> toIdConverter = Optional.empty();
```

그래서 아래와 같이 핸들러에서 바로 entity 조회가 가능하다.

```java
    @GetMapping("/posts/{id}")
    public String getAPost(@PathVariable Long id) {
        Optional<Post> byId = postRepository.findById(id);
        Post post = byId.get();
        return post.getTitle();
    }
```

```java
    @GetMapping("/posts/{id}")
    public String getAPost(@PathVariable(“id”) Post post) {
        return post.getTitle();
    }
```

### HandlerMethodArgumentResolver <a href="#handlermethodargumentresolver" id="handlermethodargumentresolver"></a>

핸들러에서 Pageable 을 구성하는 아래 파라미터가 들어온다는 전제하에 바로 Pageable 객체를 바로 받을 수 있다.

* page
* size
* sort(ex. sort=createdAt,desc)

```
    @GetMapping("/posts")
    public void testHandler(Pageable pageable){
        
    }
```
