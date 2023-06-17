# CH11 CQRS

## CQRS 이 무엇이며 필요성이 생긴 배경

### 단일 모델의 단점

<figure><img src="../../.gitbook/assets/image (2) (2).png" alt=""><figcaption></figcaption></figure>

위와 같이 특정 조회 화면에서 여러 애그리거트를 조회해야할 경우가 있다. 물론 이렇게 여러 애그리거트를 조회해와서 응답에 필요한 DTO 를 조합, 생성해서 응답으로 내보내는 것이 일반적이고 크게 문제는 없다. 하지만 CQRS 는 기능 수행(데이터 변경 등을 수행하는 모델)시 사용되는 모델과 위의 경우처럼 단순 조회시에 사용되는 모델 모두 동일한 단일 모델을 사용하다보면 경우에 따라 코드가 복잡해질 수 있다는 문제 의식을 전제로 한다.

책의 표현을 빌리자면 '상태를 변경하는 범위와 상태를 조회하는 범위가 정확하게 일치하지 않지만 이를 단일 모델로 구현하면 모델이 불필요하게 복잡해진다'고 한다. 이것이 CQRS가 가지고 있는 근본적인 문제 의식이다.



## CQRS 의 구현

결론적으로 상태 변경을 명령하는 모델과 조회하는 모델을 분리시키는 것을 CQRS 라고 한다.

<figure><img src="../../.gitbook/assets/image (17) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (16) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

극단적으로는 아래와 같이 아예 명령과 조회를 위한 저장소까지 구분할 수 있다. 이때 데이터 동기화는 필수이다.

<figure><img src="../../.gitbook/assets/image (18) (1).png" alt=""><figcaption></figcaption></figure>

성능 향상에는 유리할 지 모르나 일단 복잡도가 올라가고 구현해야 할 양이 많아진다.
