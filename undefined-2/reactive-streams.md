# Reactive Streams 원리탐구 - 간단한 예제 직접 작성해보기

리액티브를 학습하다가 자꾸 개념이 흔들리는 것 같아서 Publisher <-> Subscriber 의 흐름을 확실히 이해하고자 그 원형을 직접 구현해보았다.

정말 간단한 예제인데, 결국 이러한 개념을 바탕으로 Publisher 와 Subscriber 가 동작하며 **Reactor 는 필요한 모든 데이터 시퀀스에 대해서 이렇게 손으로 일일이 작성할 필요없이 알아서 Publisher <-> Subscriber 를 만들어주어서 우리가 편리하게 이를 사용할 수 있게 해주는 라이브러리라고 할 수 있다.**

[이 포스팅](https://engineering.linecorp.com/ko/blog/reactive-streams-with-armeria-1)이 정리가 무척 잘되어 있다. 복습시 꼭 읽도록 하자.

## Publisher <a href="#publisher" id="publisher"></a>

```java
package com.fistkim101.reactivestudy;

import org.reactivestreams.Publisher;
import org.reactivestreams.Subscriber;
import org.reactivestreams.Subscription;

import java.util.List;

public class FistPublisher implements Publisher<Integer> {

    private List<Integer> data;

    public FistPublisher(List<Integer> data) {
        this.data = data;
    }

    @Override
    public void subscribe(Subscriber subscriber) {
        subscriber.onSubscribe(new Subscription() {

            @Override
            public void request(long n) {
                for (int i = 0; i < n; i++) {
                    subscriber.onNext(data.get(i));
                }

                subscriber.onComplete();
            }

            @Override
            public void cancel() {

            }

        });

    }

}
```

## Subscriber <a href="#subscriber" id="subscriber"></a>

```java
package com.fistkim101.reactivestudy;

import org.reactivestreams.Subscriber;
import org.reactivestreams.Subscription;

public class FistSubscriber implements Subscriber<Integer> {

    private final long requestCount;

    public FistSubscriber(long requestCount) {
        this.requestCount = requestCount;
    }

    @Override
    public void onSubscribe(Subscription subscription) {
        subscription.request(requestCount);
    }

    @Override
    public void onNext(Integer integer) {
        System.out.println("[FistSubscriber] received data : " + integer);
    }

    @Override
    public void onError(Throwable throwable) {
        System.out.println("[FistSubscriber] onError called");
    }

    @Override
    public void onComplete() {
        System.out.println("[FistSubscriber] onComplete called");
    }
}
```

## Main

```java
public class ReactiveStudyApplication {

    public static void main(String[] args) {
        List<Integer> data = new ArrayList<>(Arrays.asList(1, 2, 3, 4, 5));
        FistPublisher publisher = new FistPublisher(data);
        FistSubscriber subscriber = new FistSubscriber(3);
        publisher.subscribe(subscriber);
    }

}
```

```
[FistSubscriber] received data : 1
[FistSubscriber] received data : 2
[FistSubscriber] received data : 3
[FistSubscriber] onComplete called
```

<figure><img src="../.gitbook/assets/image (48).png" alt=""><figcaption></figcaption></figure>
