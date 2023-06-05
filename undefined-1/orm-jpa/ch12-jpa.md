# CH12 스프링 데이터 JPA

이 장은 스프링 데이터 JPA 가 뭔지, 어떻게 연동하는지, 스프링 데이터 JPA 의 원리들을 간략하게 알아보는 내용이었다. 스프링 데이터 JPA 가 주제인 백기선님 강의에서 한번 정리를 했던 내용들이라서 책의 목차를 그대로 따르기 보다는 예전에 백기선님 강의 노트를 복습하는 형식으로 내용들을 정리했다.

## 스프링 데이터 JPA 란

<figure><img src="../../.gitbook/assets/image (21).png" alt=""><figcaption></figcaption></figure>

스프링 공식문서에도 잘 나와있다시피 Spring Data JPA 는 JPA based data access layer 를 support 하는 목적으로 만들어진 라이브러리이다.

> Spring Data JPA, part of the larger Spring Data family, makes it easy to easily implement JPA based repositories. This module deals with enhanced support for JPA based data access layers. It makes it easier to build Spring-powered applications that use data access technologies.

## 스프링 데이터 JPA 가 해주는 일들

많은 것들을 해주는데 꼭 알아야 할 것들만 간략히 정리한다.

지금껏 인터페이스로 레포지토리를 만들고 extends JpaRepository\<Entity, Long> 와 같은 처리만 해주면 알아서 레포지토리가 bean 으로 생성되어서 이를 가지고 기본적인 CRUD 들을 처리할 수 있었다. 이게 가능했던 이유는&#x20;

* 자동으로 레포지토리가 bean 으로 생성되어 관리되고
* 그렇게 관리되는 레포지토리가 bean 으로 생성시 많은 자원이 자동으로 구현

되었기 때문이었다. 이 두 가지가 어떻게 되고 있는 건지 아래에 정리한다.

JPA를 사용하려면 @EnableJpaRepositories 를 사용 해줘야 하는데 스프링부트에서는 해당 어노테이션이 기본적으로 사용되고 있다. (그래서 지금까지 명시적으로 적어줄 필요가 없었던 것이다)

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@Import(JpaRepositoriesRegistrar.class)
public @interface EnableJpaRepositories {
```

여기서 JpaRepositoriesRegistrar 가 있는데, 이 어노테이션의 상속구조를 따라가다 보면 ImportBeanDefinitionRegistrar 라는 인터페이스를 상속하고 있음을 알 수 있다.

결국 여기서 org.springframework.data.jpa.repository.JpaRepository 를 상속하는 interface 를 repository 로 사용할 수 있도록 bean 으로 등록해주고, 기본적인 자원들을 내부적으로 만들어준다고 볼 수 있다.

<figure><img src="../../.gitbook/assets/image (10) (2).png" alt=""><figcaption></figcaption></figure>

```
@NoRepositoryBean
public interface JpaRepository<T, ID> extends PagingAndSortingRepository<T, ID>, QueryByExampleExecutor<T> {
```

```
@NoRepositoryBean
public interface PagingAndSortingRepository<T, ID> extends CrudRepository<T, ID> {

	/**
	 * Returns all entities sorted by the given options.
	 *
	 * @param sort the {@link Sort} specification to sort the results by, can be {@link Sort#unsorted()}, must not be
	 *          {@literal null}.
	 * @return all entities sorted by the given options
	 */
	Iterable<T> findAll(Sort sort);

	/**
	 * Returns a {@link Page} of entities meeting the paging restriction provided in the {@link Pageable} object.
	 *
	 * @param pageable the pageable to request a paged result, can be {@link Pageable#unpaged()}, must not be
	 *          {@literal null}.
	 * @return a page of entities
	 */
	Page<T> findAll(Pageable pageable);
}
```

```
/**
 * Interface for generic CRUD operations on a repository for a specific type.
 *
 * @author Oliver Gierke
 * @author Eberhard Wolff
 * @author Jens Schauder
 */
@NoRepositoryBean
public interface CrudRepository<T, ID> extends Repository<T, ID> {
```

```
/**
 * Central repository marker interface. Captures the domain type to manage as well as the domain type's id type. General
 * purpose is to hold type information as well as being able to discover interfaces that extend this one during
 * classpath scanning for easy Spring bean creation.
 * <p>
 * Domain repositories extending this interface can selectively expose CRUD methods by simply declaring methods of the
 * same signature as those declared in {@link CrudRepository}.
 * 
 * @see CrudRepository
 * @param <T> the domain type the repository manages
 * @param <ID> the type of the id of the entity the repository manages
 * @author Oliver Gierke
 */
@Indexed
public interface Repository<T, ID> {

}
```

여기서 중간 계층의 Repository 에는 @NoRepositoryBean 가 있음으로 인해서 Repository를 bean으로 등록해주는 과정에서 bean 등록이 이뤄지지 않고 넘어간다.

그리고 이 때 인터페이스에서 구현된 기본적인 메소드들이 bean 에 실제로 구현되면서 결과적으로 JpaRepository 를 상속만 하면 Repository가 bean 으로도 등록되고, 모든 기본적인 메소드들을 사용할 수 있는 것이다.

또 한가지 재미있는 것은 아래와 같이 Common 계층과 Jpa 계층이 따로 있어서 다른 데이터베이스로의 변경도 고려가 되어 있다는 점이다.

<figure><img src="../../.gitbook/assets/image (16) (1) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (25).png" alt=""><figcaption></figcaption></figure>

## 커스텀 Repository 구현

자동 생성되는 쿼리 외에 직접 커스텀하게 쿼리를 구성해야할 때가 있다. 이 때 커스텀 레포지토리를 만들고 이를 기존 레포지토리가 상속하게 만들어서 기존 레포지토리로 커스텀 정의한 쿼리를 사용할 수 있도록 할 수 있다.

바로 위에서 살펴본 바와 같이 최종적으로 필요한 entity의 Repository는 최하단의 JpaRepository를 extend 하여 선언한다.

예를 들어 PostRepository 를 선언하여 사용한다고 했을 때 PostRepository 가 JpaRepository 를 extend 할 것이다. 최종적으로 최하단의 PostRepository 가 bean 으로 등록이 될때 @NoRepositoryBean 가 있는 중간 계층의 Repository들의 자원을 모두 갖게 된다. 그리고 @NoRepositoryBean 이 있는 중간 계층의 Repository 는 정의된 메소드들만 제공하고 bean 으로 등록되지 않을 것이다.

커스텀 레포지토리 생성 원리도 이와 크게 다르지 않다.

* 커스텀 메소드들을 정의 및 구현해주고
* 최하단 Repository 가 bean 으로 등록되는 과정에서 정의 및 구현한 커스텀 메소드들이 자원으로 포함되도록 처리해준다.

결국 이렇게만 해주면 되는데, 내부적으로 지켜줘야할 규칙이 있다.

* 인터페이스로 다중 상속의 형태로 상속하게 만들어야한다.
* 구현체의 Postfix 를 규칙대로 지정해줘야한다.

```java
public interface PostCustomRepository {
    Post findMyPost();
}
```

```java
@Repository
@Transactional
public class PostCustomRepositoryImpl implements PostCustomRepository {

    @PersistenceContext
    EntityManager entityManager;

    @Override
    public Post findMyPost() {
        return entityManager.find(Post.class, 1L);
    }

}
```

```java
@SpringBootApplication
@EnableJpaRepositories(repositoryImplementationPostfix = "Impl")
public class SpringJpaWhiteshipStudyApplication {

    public static void main(String[] args) {
        SpringApplication.run(SpringJpaWhiteshipStudyApplication.class, args);
    }

}
```

Impl 이 default 값이고 이를 바꾸고 싶으면 위 설정에서 따로 바꿔 주어야 한다. 최종적으로 아래와 같이 최하단의 Repository가 커스텀 레포지토리 인터페이스를 상속하도록 해준다.

커스텀 레포지토리로 선언된 interface 의 명칭은 어떻게 선언이 되든 상관이 없고, 구현체의 Postfix 를 꼭 지켜줘야한다. 결국 커스텀 레포지토리의 핵심은 커스텀하게 구현한 것이 bean 으로 자동 설정되는 레포지토리의 자원으로 포함 되도록 하는 것이 핵심이다.

```java
public interface PostRepository extends JpaRepository<Post, Long>, PostCustomRepository {
}
```

## 스프링 데이터 JPA 가 쿼리를 생성하는 방법

스프링 부트에서 자동으로 설정해주어 명시적으로 사용하지 않았던 @EnableJpaRepositories 에서 queryLookupStrategy 를 설정할 수 있다. 아래 코드에 설정한 것은 기본값인데 명시적으로 써 준 것이다.

```java
@EnableJpaRepositories(queryLookupStrategy = QueryLookupStrategy.Key.CREATE_IF_NOT_FOUND)
@SpringBootApplication
public class SpringJpaWhiteshipStudyApplication {

    public static void main(String[] args) {
        SpringApplication.run(SpringJpaWhiteshipStudyApplication.class, args);
    }

}
```

* CREATE : 메소드 이름을 분석해서 쿼리 만들기
* USE\_DECLARED\_QUERY : 미리 정의해 둔 쿼리 찾아 사용하기
* CREATE\_IF\_NOT\_FOUND : 미리 정의한 쿼리 찾아보고 없으면 만들기

위와 같이 세 옵션이 존재하고 특별히 정해주지 않을 경우 CREATE\_IF\_NOT\_FOUND 를 기본값으로 동작한다. 그렇기 때문에 @Query 로 직접 설정한 쿼리가 존재할 경우 Repository의 메소드는 직접 설정한 쿼리를 수행하고 특별히 @Query 로 설정해준 쿼리가 없는 경우 메소드 이름을 분석해서 쿼리를 자동 생성하여 해당 쿼리를 수행한다.

메소드 이름 분석의 경우 정해진 규칙에 따라 분석을 하는데 이건 방식이 이렇다고만 인지하면 된다. 아래 내용은 규칙에 대한 극히 일부를 정리한 것이며 메소드 선언시 IDE 의 도움을 받아서 편하게 선언하자.

<figure><img src="../../.gitbook/assets/image (4).png" alt=""><figcaption></figcaption></figure>
