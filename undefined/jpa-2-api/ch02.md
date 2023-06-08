# CH02 컬렉션 조회 최적화

## 주문 조회 V1: 엔티티 직접 노출 <a href="#v1" id="v1"></a>

이전에 다뤘던 엔티티 직접 노출 케이스와 같았다. 지연로딩 필드는 응답 전에 강제초기화를 해줬고, 양방향 참조 필드의 경우 한 쪽에 @JsonIgnore 처리를 해줬다. 근본적으로 엔티티를 직접 노출하는 방식 자체가 잘못된 방식이기 때문에 자세히 들여다보는거 자체가 의미가 없다.

## 주문 조회 V2: 엔티티를 DTO로 변환 <a href="#v2-dto" id="v2-dto"></a>

```java
@Data
    static class OrderDto {

        private Long orderId;
        private String name;
        private LocalDateTime orderDate; //주문시간
        private OrderStatus orderStatus;
        private Address address;
        private List<OrderItemDto> orderItems;

        public OrderDto(Order order) {
            orderId = order.getId();
            name = order.getMember().getName();
            orderDate = order.getOrderDate();
            orderStatus = order.getStatus();
            address = order.getDelivery().getAddress();
            orderItems = order.getOrderItems().stream()
                    .map(orderItem -> new OrderItemDto(orderItem))
                    .collect(toList());
        }
    }

    @Data
    static class OrderItemDto {

        private String itemName;//상품 명
        private int orderPrice; //주문 가격
        private int count;      //주문 수량

        public OrderItemDto(OrderItem orderItem) {
            itemName = orderItem.getItem().getName();
            orderPrice = orderItem.getOrderPrice();
            count = orderItem.getCount();
        }
    }
```

* 지연 로딩으로 너무 많은 SQL 실행
* SQL 실행 수
  * order 1번
  * member , address N번(order 조회 수 만큼)
  * orderItem N번(order 조회 수 만큼)
  * item N번(orderItem 조회 수 만큼)

## 주문 조회 V3: 엔티티를 DTO로 변환 - 페치 조인 최적화 <a href="#v3-dto" id="v3-dto"></a>

```
    public List<Order> findAllWithItem() {
        return em.createQuery(
                "select distinct o from Order o" +
                        " join fetch o.member m" +
                        " join fetch o.delivery d" +
                        " join fetch o.orderItems oi" +
                        " join fetch oi.item i", Order.class)
                .getResultList();
    }
```

* 페치 조인으로 SQL이 1번만 실행됨
* distinct 를 사용한 이유는 1대다 조인이 있으므로 데이터베이스 row가 증가한다. 그 결과 같은 order 엔티티의 조회 수도 증가하게 된다. JPA의 distinct는 SQL에 distinct를 추가하고, 더해서 같은 엔티티가 조회되면, 애플리케이션에서 중복을 걸러준다. 이 예에서 order가 컬렉션 페치 조인 때문에 중복 조회 되는 것을 막아준다.
* 페이징이 불가능하다는 단점이 있다.

{% hint style="info" %}
컬렉션 페치 조인을 사용하면 페이징이 불가능하다. 하이버네이트는 경고 로그를 남기면서 모든 데이터를 DB에서 읽어오고, 메모리에서 페이징 해버린다(매우 위험하다). 이와 관련된 사항은    JPA 기본 도서 쪽에 이미 정리를 해두었다.
{% endhint %}

{% hint style="info" %}
컬렉션 페치 조인은 1개만 사용할 수 있다. 컬렉션 둘 이상에 페치 조인을 사용하면 안된다. 데이터가 부정합하게 조회될 수 있다. 자세한 내용은 자바 ORM 표준 JPA 프로그래밍을 참고하자. 애초에 조회시 에러가 발생한다.(런타임 오류이긴 하지만 테스트를 하기만 해도 잘못되었다는 것을 인지할 수 있다)
{% endhint %}

## 주문 조회 V3.1: 엔티티를 DTO로 변환 - 페이징과 한계 돌파 <a href="#v31-dto" id="v31-dto"></a>

### 컬렉션을 페치 조인하면 페이징이 안되는 이유 <a href="#undefined" id="undefined"></a>

1:N 관계에서 N(컬렉션)을 페치 조인하게 되면 distinct 를 걸어준다고 해도 relation 관점에서 row 수가 증가해버린다. 이 경우 하이버네이트는 경고 로그를 남기고 where절 없이 모든 데이터를 전부 조회해와서 메모리에 올린뒤 페이징 처리를 한다.

### 컬렉션을 페이징 해야한다면 어떻게? <a href="#undefined" id="undefined"></a>

1. 먼저 ToOne(OneToOne, ManyToOne) 관계를 모두 페치조인 한다. ToOne 관계는 row수를 증가시키지 않으므로 페이징 쿼리에 영향을 주지 않는다.
2. 컬렉션은 지연 로딩으로 조회한다.
3. 지연 로딩 성능 최적화를 위해 hibernate.default\_batch\_fetch\_size , @BatchSize 를 적용한다. 이 옵션을 사용하면 컬렉션이나, 프록시 객체를 한꺼번에 설정한 size 만큼 IN 쿼리로 조회한다.
   * hibernate.default\_batch\_fetch\_size: 글로벌 설정
   * @BatchSize: 개별 최적화

```yaml
spring:
  jpa:
    properties:
      hibernate:
        default_batch_fetch_size: 1000
```

ToOne 관계는 페치 조인해도 페이징에 영향을 주지 않는다. 따라서 ToOne 관계는 페치조인으로 쿼리 수를 줄이고, 나머지는 hibernate.default\_batch\_fetch\_size 로 최적화 하자.

### default\_batch\_fetch\_size 의 적절한 크기 <a href="#default_batch_fetch_size" id="default_batch_fetch_size"></a>

그냥 단순하게 생각해보면 한번에 많이 in 절로 처리하는게 좋을 것 같다. 그렇다면 한번에 많이 가져왔을때 어떤 트레이드 오프가 있을지를 이해하면 될 것 같다. 핵심 기준은 ‘순간부하’ 이다. 1000건으로 한번에 가져올 경우 그만큼 데이터베이스에서 한번에 조회되는 데이터의 양이 많아서 데이터베이스에 순간부하가 10건일때보다 훨씬더 크게 간다. 또 이 전체 데이터를 데이터베이스에서 어플리케이션으로 전송되는 전송비용도 커진다. (규모만 봤을때)

하지만 이게 크게 크리티컬할지는 좀 의문이다. 적게 하면 그만큼 조회를 많이 할테니까. 결국 데이터의 양이나 인프라 환경 등 여러가지를 고려해봐야 하는 것이고, 그냥 크게 고민할 것 없이 500 정도 걸어두고 문제가 생기거나 하면 그때 들여다 보는 것이 좋을 것 같다.

## 주문 조회 V4: JPA에서 DTO 직접 조회 <a href="#v4-jpa-dto" id="v4-jpa-dto"></a>

특별히 정리할 내용 없었음. DTO 를 직접 new 로 쿼리에서 뽑으면서 1 + N 발생.

## 주문 조회 V5: JPA에서 DTO 직접 조회 - 컬렉션 조회 최적화 <a href="#v5-jpa-dto" id="v5-jpa-dto"></a>

직접 쿼리에서 new 로 DTO를 만드는데, N 쿼리 발생하는 부분에서 in 절로 조회하여 N -> 1회로 쿼리 발생 수를 줄인 뒤 로직에서 Map으로 변환하여 매칭.

## 주문 조회 V6: JPA에서 DTO로 직접 조회, 플랫 데이터 최적화 <a href="#v6-jpa-dto" id="v6-jpa-dto"></a>

중복 데이터로 나오는 쿼리로 수행해서 로직에서 일일이 매핑해주는 방법. 실무에서 전혀 쓸일이 없을 거라 확신한다.
