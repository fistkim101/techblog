# CPU 사용량

## CPU 사용량의 의미

먼저 [aws 의 정의](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/burstable-credits-baseline-concepts.html)를 살펴 보자.

> CPU utilization is the percentage of allocated EC2 compute units that are currently in use on the instance. This metric measures the percentage of allocated CPU cycles that are being utilized on an instance.

다른 [정체불명의 사이트](https://www.solarwinds.com/resources/it-glossary/what-is-cpu)는 아래와 같이 정의하고 있다.

> CPU usage indicates the total percentage of processing power exhausted to process data and run various programs on a network device, server, or computer at any given point.

[위키](https://ko.wikipedia.org/wiki/CPU\_%ED%83%80%EC%9E%84)에서는 아래와 같이 정의하고 있다.

> CPU 타임(CPU time) 또는 CPU 사용률(CPU usage)은 한 컴퓨터 [프로그램](https://ko.wikipedia.org/wiki/%EC%BB%B4%ED%93%A8%ED%84%B0\_%ED%94%84%EB%A1%9C%EA%B7%B8%EB%9E%A8)이 [CPU](https://ko.wikipedia.org/wiki/%EC%A4%91%EC%95%99\_%EC%B2%98%EB%A6%AC\_%EC%9E%A5%EC%B9%98)를 차지하여 일을 한 [시간](https://ko.wikipedia.org/wiki/%EC%8B%9C%EA%B0%84)의 양을 뜻한다. 보통 [클럭 틱](https://ko.wikipedia.org/w/index.php?title=%ED%81%B4%EB%9F%AD\_%ED%8B%B1\&action=edit\&redlink=1) 단위로 측정된다. 프로그램들 사이의 CPU 사용률을 비교하기 위해 사용된다.

[유투브의 한 영상](https://www.youtube.com/watch?v=KKpm9m4A3-w)에서도 비슷하게 이야기한다.

> CPU utilization is percentage of time a CPU is occupied by running process.

출처마다 아주 조금씩 표현이 다르긴 한데 핵심은 CPU 사용'률'이라는 것이다. 즉, 퍼센트로 표현하는 것이 적절한 개념이다. 단위 시간에 CPU 의 총 역량중 몇 퍼센트를 점유 했는지를 의미한다.

인스턴스 자체의 CPU 사용량이 현재 50% 라면 해당 인스턴스 내에 모든 프로세스가 할당 받은 CPU 사용의 총 합이CPU 총 역량 대비 50%라는 의미이다.



## CPU 사용량이 서버에 미치는 영향

당연한 이야기이지만 CPU 사용량이 100% 라면 새로운 작업에 대해서 더 이상 할애해줄 수 있는 CPU 의 여유가 없으므로 작업이 진행되지 않거나 느리게 진행되고, 이에 따라 응답이 점점 더 오래 걸리게 된다.

더 구체적으로 이를 설명하기 위해서 프로세스, 스레드에 대해서 간략히 짚고 넘어간다.

### 프로세스

실행중인 프로그램을 의미한다. '실행중' 이라는 의미는 램에 해당 프로그램이 적재되어 있다는 것을 의미한다.

### 스레드

프로세스 내에서 다중 처리를 위해 만들어진 개념으로 '자원 소유의 단위'와 '디스패칭의 단위'로 설명될 수 있다. CPU 스케줄링에 의해서 'CPU Time Slice'를 받을 수 있다는 의미이다. CPU Time Slice 는 스레드가 CPU 를 사용할 수 있는 '시간 조각'을 의미한다.

### CPU 사용량이 100% 라면?

극단적인 상황을 전제해서 다시 알아보자면, 결국 스레드가 CPU Time Slice 를 부여받지 못하거나 받는 시간이 점점 뒤로 밀려나서 결론적으로 레이턴시가 증가하게 된다. 왜냐면 CPU 가 여유가 없어서 CPU 스케줄링에 따라 할당받는 CPU Time Slice 가 적어지고 단위 시간당 할당 빈도가 줄어들기 때문이다.



## CPU 사용량을 낮추기 위한 방법

### 스케일업, 스케일아웃

스케일업을 하게 되면 CPU 의 성능 자체가 좋아지므로 결론적으로 동일한 프로세스의 CPU 요구 수준을 두고 봤을 때 CPU 사용량이 내려가게 된다. CPU 전체 사용 역량 대비 사용량을 의미하므로 CPU 자체의 소화 능력이 좋아지기 때문이다.

스케일아웃을 하더라도 마찬가지이다. 동일한 요청을 전제하면 이를 처리해줄 인스턴스가 하나 더 늘어나는 것이므로 로드밸런싱이 효과적으로 이뤄진다면 CPU 사용량은 낮아지게 된다. 분담을 하게 되니까.



### 비동기, 병렬 처리 도입

CPU 사용량은 CPU 를 점유하는 시간과 밀접한 연관이 있다. 결국 CPU 스케줄링 자체가 CPU Time Slice 의 개념으로 주어지기 때문이다. 그렇다면 동일한 일을 더 빠르게 끝낼 수 있도록 처리한다면 주어지는 리소스는 그대로라도 CPU 사용량이 줄어들 수 있다.

예를 들어 전체를 100시간이라고 봤을 때 동일한 양의 일이 주어진 경우 이를 10시간이 걸려서 끝내면 CPU 사용률은 10% 인데, 3시간만 걸려서 끝내면 3%라고 할 수 있을 것이다. 달리 말하면 CPU 를 효율적으로 잘 사용해서 동일한 일을 더빠르게 끝내도록 한다면 CPU 사용량이 낮아지는 결과를 만들 수 있다는 것이다.

그래서 비동기, 병렬 처리가 CPU 사용량을 낮출 수 있다고 말할 수 있다. 동일한 시간에 집중적으로 CPU 자원을 사용함으로써 더 빠르게 일을 끝낼 수 있다.



## 웹플럭스에서는 왜 컨텍스트 스위칭 비용이 블로킹 방식에 비해서 낮은가

### 컨텍스트 스위칭

컨텍스트 스위칭은 스레드의 실행 상태를 변경하는 과정에서 발생한다. 대기에 들어가며 작업하던 내용을 PCB 에 저장하고, 새로 진행하는 시점에 이 문맥을 다시 복원하여 작업을 재개하는 행위 자체가 곧 컨텍스트가 스위칭 되는 것이고 이때 이미 비용이 발생하는 것이다.

### 논블로킹 방식이 컨텍스트 스위칭이 덜 발생하는 이유

디스크IO 나 네트워크 요청 등이 발생하면 해당 작업을 수행하는 스레드는 결과를 얻기 위해서 기다려야한다. 블로킹에서는 여기서 스레드가 멈추고 이를 기다리게 되고 논블로킹에서는 이 시점에 대기하지 않고 다른 일을 그대로 이어서 하게 된다. 여기까지만 봐도 블로킹 방식에서 컨텍스트 스위칭이 많이 발생할 수 밖에 없다는 것을 알 수 있다.

스레드가 멈추고 다시 작업을 재개하게 된다는 것의 의미는 CPU Time Slice 를 할당 받는 다는 것은 아니다. 여기서 핵심은 스레드가 CPU Time Slice 를 할당 받더라도 해당 작업이 끝나지 않으면 대기한다는 것이다. 즉 비효율이 발생하는 것이다.

한편 논블로킹에서는 이 시점에 바로 해당 스레드는 대기를 하지 않고 다른 일을 한다. 즉 대기 또는 일시정지와 실행중 상태로 전환될때 마다 발생하는 컨텍스트 스위칭 비용이 계속 실행중인 스레드는 발생하지 않는 것이다. 왜냐하면 말 그대로 멈추지 않고 다른 일을 계속 이어서 하기 때문이다.
