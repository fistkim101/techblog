# CH05 Spring Data JPA

## JpaRepository.save() <a href="#jparepositorysave" id="jparepositorysave"></a>

save() 는 단순히 엔티티를 저장해주는 기능을 수행하는 것이 아님. 경우에 따라 persist 또는 merge 로 동작한다.

* Transient 상태의 객체라면 EntityManager.persist()
* Detached 상태의 객체라면 EntityManager.merge()

### persist <a href="#persist" id="persist"></a>

Persist() 메소드에 파라미터로 넘긴(save 대상) 그 엔티티 객체를 Persistent 상태로 변경한다. save() 결과로 반환받은 saved entity 가 곧 파라미터로 넘긴 그 save 대상과 같다.

```java
@DataJpaTest
public class PostRepositoryTest {

    @Autowired
    PostRepository postRepository;

    @PersistenceContext
    EntityManager entityManager;

    @Test
    void saveTest() {
        Post post = new Post();
        post.setName("name_1");
        post.setDescription("description_1");
        Post savedPost = postRepository.save(post);

        Assertions.assertThat(entityManager.contains(post)).isTrue();
        Assertions.assertThat(entityManager.contains(savedPost)).isTrue();
        Assertions.assertThat(post).isEqualTo(savedPost);
    }

}
```

### merge <a href="#merge" id="merge"></a>

주석 표시한 것들을 잘 기억하자. update 시 파라미터로 넘긴 entity는 영속화되지 않고 그것의 복사본이 영속화 된다. 그리고 영속화된 객체가 return 된다.

이 포인트가 주는 중요한 시사점은 update 로직 처리 이후 후속 작업을 해야할 경우 반드시 return 된 그 값을 사용해야 한다는 것이다. 왜냐하면 그 객체가 persistent 된 객체이기 때문이다. 반대로 말하면 save() 의 파라미터로 넘긴 객체를 사용해서는 안된다는 것이다.

왜 사용하면 안되냐? managed 객체가 아니기 때문이다. 즉, persistent 상태가 아니기 때문에 상태가 추적이 되지 않아서 dirty check 등 JPA의 이점을 누리지 못하기 때문이다.

```java
    @Test
    void updateTest() {
        Post post = new Post();
        post.setName("name_1");
        post.setDescription("description_1");
        postRepository.save(post);

        Post newPost = new Post();
        newPost.setId(1L);
        newPost.setName("new_name_1");
        newPost.setDescription("new_description_1");
        Assertions.assertThat(entityManager.contains(newPost)).isFalse();

        // merge 발생. 이 때 newPost의 복사본이 영속화된다.
        Post updatedPost = postRepository.save(newPost);

        // 파라미터로 넘긴 entity인 newPost 는 detached
        Assertions.assertThat(entityManager.contains(newPost)).isFalse();

        // update 결과로 받은 entity인 updatedPost는 persistent
        Assertions.assertThat(entityManager.contains(updatedPost)).isTrue();
    }
```

## [Projection](https://www.baeldung.com/spring-data-jpa-projections) <a href="#projection" id="projection"></a>

entity의 일부 컬럼만 가져오는 기능인데 나는 실무에서 써본적이 없거니와 얻는 성능 효율 대비 코드 복잡도만 더 커지는 느낌이다. 일단은 이런게 있다 정도만 인지해둔다.

아래는 강의 노트 그대로 발췌.

* 인터페이스 기반 프로젝션
  * Nested 프로젝션 가능.
  * Closed 프로젝션
    * 쿼리를 최적화 할 수 있다. 가져오려는 애트리뷰트가 뭔지 알고 있으니까.
    * Java 8의 디폴트 메소드를 사용해서 연산을 할 수 있다.
  * Open 프로젝션
    * @Value(SpEL)을 사용해서 연산을 할 수 있다. 스프링 빈의 메소드도 호출 가능. -쿼리 최적화를 할 수 없다. SpEL을 엔티티 대상으로 사용하기 때문에.
* 클래스 기반 프로젝션
  * DTO
  * 롬복 @Value로 코드 줄일 수 있음

## Specification <a href="#specification" id="specification"></a>

query DSL 과 유사하게 query 를 프로그램으로 작성할 수 있도록 해준다. 아래는 [JPA 공식 문서에서 Specification](https://docs.spring.io/spring-data/jpa/docs/current/reference/html/#specifications) 에 대해 설명한 글 일부다.

> JPA 2 introduces a criteria API that you can use to build queries programmatically. By writing a criteria, you define the where clause of a query for a domain class. Taking another step back, these criteria can be regarded as a predicate over the entity that is described by the JPA criteria API constraints.

사용 방법은 공식문서에도 나와있지만 간단하다. repository 에서 JpaSpecificationExecutor 를 상속 받고 findAll() 에 파라미터로 Specification 을 넘겨준다. pageable 도 파라미터로 넘겨주면 페이징 처리도 가능하다.

유용하지만 타입 세이프하지 않아서 그냥 queryDsl 쓰는게 마음 편하다.

```java
public interface PostRepository extends JpaRepository<Post, Long>, PostCustomRepository, QuerydslPredicateExecutor<Post>, JpaSpecificationExecutor<Post> {
}
```

```java
    public static Specification<Post> getPostsByCustomSpecification(String name, String description) {
        return ((root, query, criteriaBuilder) -> {
            List<Predicate> predicates = new ArrayList<>(); // javax.persistence.criteria.Predicate;
            if (name != null) {
                predicates.add(criteriaBuilder.equal(root.get("name"), name));
            }
            if (description != null) {
                predicates.add(criteriaBuilder.equal(root.get("description"), description));
            }
            return criteriaBuilder.and(predicates.toArray(new Predicate[0]));
        });
    }
```

```java
    @Test
    void specificationTest() {
        Post post = new Post();
        final String NAME = "specificationPostName";
        final String DESCRIPTION = "specificationPostDescription";
        post.setName(NAME);
        post.setDescription(DESCRIPTION);
        postRepository.save(post);

        List<Post> targets = postRepository.findAll(Post.getPostsByCustomSpecification(NAME, DESCRIPTION));
        Assertions.assertThat(targets.size()).isEqualTo(1);
        Assertions.assertThat(targets.get(0).getName()).isEqualTo(NAME);
        Assertions.assertThat(targets.get(0).getDescription()).isEqualTo(DESCRIPTION);
    }
```

```
    select
        post0_.id as id1_0_,
        post0_.description as descript2_0_,
        post0_.name as name3_0_ 
    from
        post post0_ 
    where
        post0_.name=? 
        and post0_.description=?
# binding parameter [1] as [VARCHAR] - [specificationPostName]
# binding parameter [2] as [VARCHAR] - [specificationPostDescription]
```

만약 여기서 아래와 같이 DESCRIPTION 에 null 을 넣을 경우 쿼리가 의도한대로 달라진다.

```java
    @Test
    void specificationTest() {
        Post post = new Post();
        final String NAME = "specificationPostName";
        final String DESCRIPTION = "specificationPostDescription";
        post.setName(NAME);
        post.setDescription(DESCRIPTION);
        postRepository.save(post);

        List<Post> targets = postRepository.findAll(Post.getPostsByCustomSpecification(NAME, null));
        Assertions.assertThat(targets.size()).isEqualTo(1);
        Assertions.assertThat(targets.get(0).getName()).isEqualTo(NAME);
        Assertions.assertThat(targets.get(0).getDescription()).isEqualTo(DESCRIPTION);
    }
```

```
    select
        post0_.id as id1_0_,
        post0_.description as descript2_0_,
        post0_.name as name3_0_ 
    from
        post post0_ 
    where
        post0_.name=?
```

## Auditing <a href="#auditing" id="auditing"></a>

특별한 내용은 없어서 강의자료만 첨부한다.

* 메인 애플리케이션 위에 @EnableJpaAuditing 추가 (스프링부트가 자동설정 해주지 않는다)
* 엔티티 클래스 위에 @EntityListeners(AuditingEntityListener.class) 추가

```java
    @CreatedDate
    private Date created;

    @LastModifiedDate
    private Date updated;

    @CreatedBy
    @ManyToOne
    private Account createdBy;

    @LastModifiedBy
    @ManyToOne
    private Account updatedBy;
```
