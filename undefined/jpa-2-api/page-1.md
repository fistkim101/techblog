# CH01 지연 로딩과 조회 성능 최적화

## 간단한 주문 조회 V1: 엔티티를 직접 노출 <a href="#v1" id="v1"></a>

잘못된 예시를 통해서 ‘이렇게 하면 안된다’ 라는 것을 같이 실습했다. JPA 기본 강의에서 사실 다 다뤄졌던 것들이다. 엔티티를 직접 노출함으로써 발생되는 여러 문제들이었다.

* 양방향 의존관계 엔티티를 직렬화 하는 단계에서 서로 참조를 하다보니 발생하는 문제
* 지연로딩 설정된 필드의 프록시 객체가 초기화가 되지 않은 상태에서 직렬화 시도시 발생하는 문제

둘의 문제를 살펴보았고 각각 @JsonIgnore 혹은 하이버네이트 설정을 건들여서 임시방편으로 해결하는 것들을 같이 살펴봤다. 방향이 근본적으로 잘못되었기 때문에 사실 해결책 역시 크게 의미가 없다.

## 간단한 주문 조회 V2: 엔티티를 DTO로 변환 (ManyToOne, OneToOne) <a href="#v2-dto-manytoone-onetoone" id="v2-dto-manytoone-onetoone"></a>

응답용 DTO로 변환을 해서 응답을 하는 방식이다. 루프를 돌리면서 DTO를 만드는데 이 과정에서 프록세 객체의 초기화가 발생한다. 지연로딩으로 인한 초기화이슈가 해결이되었고 응답이 결국 나가긴 하는데 1 + N 문제가 발생하는 것을 쿼리로 확인했다. JPA 기본 강의가 잘 구성되어 있어서인지 다 다뤘던 내용들이다.

```java
@Data
static class SimpleOrderDto {

    private Long orderId;
    private String name;
    private LocalDateTime orderDate; //주문시간
    private OrderStatus orderStatus;
    private Address address;

    public SimpleOrderDto(Order order) {
        orderId = order.getId();
        name = order.getMember().getName();
        orderDate = order.getOrderDate();
        orderStatus = order.getStatus();
        address = order.getDelivery().getAddress();
    }
}
```

* 엔티티를 DTO로 변환하는 일반적인 방법이다.
* 쿼리가 총 1 + N + N번 실행된다. (v1과 쿼리수 결과는 같다.)
  * order 조회 1번(order 조회 결과 수가 N이 된다.) order -> member 지연 로딩 조회 N 번
  * order -> delivery 지연 로딩 조회 N 번
  * 예) order의 결과가 4개면 최악의 경우 1 + 4 + 4번 실행된다.(최악의 경우)
    * 지연로딩은 영속성 컨텍스트에서 조회하므로, 이미 조회된 경우 쿼리를 생략한다.

여기서 하나 짚고 넘어갈 부분이 N이 꼭 N번은 아닐 수 있다는 것이다. 영속성 컨텍스트를 PK 기준으로 먼저 조회하기 때문에 동일한 엔티티가 이미 쿼리로 한번 수행되어서 영속성 컨텍스트에 보관되고 있는 경우에는 쿼리가 나가지 않는다.

## 간단한 주문 조회 V3: 엔티티를 DTO로 변환(ManyToOne, OneToOne) - 페치 조인 최적화 <a href="#v3-dto-manytoone-onetoone" id="v3-dto-manytoone-onetoone"></a>

```java
    public List<Order> findAllWithMemberDelivery() {
        return em.createQuery(
        "select o from Order o" +
        " join fetch o.member m" +
        " join fetch o.delivery d", Order.class)
        .getResultList();
        }
```

페치조인을 통해서 1 + 4 + 4 번 발생하던 쿼리를 1번 발생하도록 개선한 모습이다. 기본적으로 저렇게 사용하면 inner join 이 적용되는데, ManyToOne, OneToOne 이기 때문에 무지성 join fetch를 적용해도 된다.

OneToMany 에서는 해당 엔티티를 바라보는 Many side 엔티티가 존재하지 않으면 One side 엔티티가 아예 조회가 안되는 일이 발생할 수 있어서, 그때는 outer join 으로 처리해주는 등의 고려가 필요하다.

그리고 또 Many side 엔티티가 여러개 있을 경우 One side 엔티티가 하나만 존재한다 하더라도 N개의 row가 반환되므로 distinct 처리를 해줘야한다. 기억이 안나면 JPA 기본강의 정리 노트를 다시 보자.

## 간단한 주문 조회 V4: JPA에서 DTO로 바로 조회 <a href="#v4-jpa-dto" id="v4-jpa-dto"></a>

V4 방식에서 제기하는 V3의 문제(문제라 하기엔 너무 사소하지만)는 DTO 변환에 사용하지 않는 여러 필드 값들이 select 절에 다 올라온다는 것이다. 그래서 V4는 DTO 자체를 쿼리에서 뽑아서 바로 만드는 방식이다.

```java
@Repository
@RequiredArgsConstructor
public class OrderSimpleQueryRepository {

    private final EntityManager em;

    public List<OrderSimpleQueryDto> findOrderDtos() {
        return em.createQuery(
                "select new jpabook.jpashop.repository.order.simplequery.OrderSimpleQueryDto(o.id, m.name, o.orderDate, o.status, d.address)" +
                        " from Order o" +
                        " join o.member m" +
                        " join o.delivery d", OrderSimpleQueryDto.class)
                .getResultList();
    }
}
```

```java
@Data
public class OrderSimpleQueryDto {

    private Long orderId;
    private String name;
    private LocalDateTime orderDate; //주문시간
    private OrderStatus orderStatus;
    private Address address;

    public OrderSimpleQueryDto(Long orderId, String name, LocalDateTime orderDate, OrderStatus orderStatus, Address address) {
        this.orderId = orderId;
        this.name = name;
        this.orderDate = orderDate;
        this.orderStatus = orderStatus;
        this.address = address;
    }
}
```

* 일반적인 SQL을 사용할 때 처럼 원하는 값을 선택해서 조회
* new 명령어를 사용해서 JPQL의 결과를 DTO로 즉시 변환
* SELECT 절에서 원하는 데이터를 직접 선택하므로 DB 애플리케이션 네트웍 용량 최적화(생각보다 미비)
* 리포지토리 재사용성 떨어짐, API 스펙에 맞춘 코드가 리포지토리에 들어가는 단점

나도 이 방식은 처음 보았는데, 일단 강의 자료 설명에도 나와있다시피 해당 로직 자체가 너무 해당 DTO에 종속적인 코드라서 다른 곳에 사용할 수도 없거니와 그렇게 한다고 해서 성능상의 이점을 엄청나게 많이 보지도 않는다.

그래서 그다지 활용성이 좋은 방식은 아닌 것 같다.



## 쿼리 방식 선택 권장 순서 <a href="#undefined" id="undefined"></a>

결론적으로 강의에서 추천하는 ?ToOne 에서의 로직 개선 방식은 아래와 같다.

1. 우선 엔티티를 DTO로 변환하는 방법을 선택한다. (모든 필드는 전부 LAZY fetch 전략)
2. 필요하면(LAZY 필드를 사용해야할 경우) 페치 조인으로 성능을 최적화 한다. (2번 단계에서 대부분의 성능 이슈가 해결된다.)
3. 그래도 안되면 DTO로 직접 조회하는 방법을 사용한다.
4. 최후의 방법은 JPA가 제공하는 네이티브 SQL이나 스프링 JDBC Template을 사용해서 SQL을 직접 사용한다.
