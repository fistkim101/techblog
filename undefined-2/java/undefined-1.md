# 커스텀 스레드 풀

## 들어가며

주로 백그라운드로 작동 시키는 작업들을 구현하다 보면 스레드 풀을 따로 만들어주고 해당 스레드 풀로 해당 작업을 시키게 하는 경우가 있다. 특히 비동기 처리의 경우가 그러하다.

예를 들어 주문이 끝나고 나서 해당 주문 건에 대해서 주문한 사용자에게 주문 내역을 메일로 전송하는 것이 필요하다고 할 때, 극단적으로 주문 1초 메일 전송 3초가 걸린다면 주문이라는 핵심 로직을 처리 후 1초 뒤 바로 응답을 하고 비동기로 메일 전송을 처리할 수 있는 것이다.

이런 경우 따로 스레드 풀을 만들어서 비동기 작업을 할때 해당 스레드 풀에서 꺼내서 사용하는 식으로 처리를 했었다.



## 스레드 개수는 얼마가 적절할지 무엇을 기준으로 판단할까?

핵심 판단 기준은 이 스레드 풀에 맡길 작업의 성격이 CPU 처리량이 많이 필요한 것인지, I/O 레이턴시가 많이 걸리는 일인지이다.

CPU 처리량이 많이 필요한 것이라면 스레드 수를 많이 늘려줘도 결국 일하는 코어가 이를 감당하지 못한다면 괜히 유휴 스레드만 많이 만들어 놓는 것이 된다. 그렇기 때문에 코어의 개수를 아래 코드로 파악 한 후,

```java
Runtime.getRuntime().availableProcessors()
```

코어의 수와 비슷한 수의 스레드 풀을 만드는 것이 일단은 적절 할 것이라 할 수 있다.(정답은 없고 무조건 추이를 보고 튜닝을 하는게 맞다고 본다. 지금 이 내용은 최초의 시작 설정에 관한 의견이다)



한편 작업의 성격이 I/O 레이턴시가 큰 작업인 경우 스레드 수를 코어수보다 훨씬 많이 설정하는 것이 좋겠다. 예를 들어 외부 API 를 두 개 이상 호출해야한다던가 하는 식의 작업이라면 그 응답을 기다리느라 블로킹 되는 스레드가 많아질 것이므로 코어 수는 고려하지 않고 적절한 스레드 수를 생각해서 만들어 두면 좋을 것 같다. 이 때 고려해야 하는 것은 메모리 정도라고 본다.