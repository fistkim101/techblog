# Spring actuator + Prometheus + grafana

Prometheus 에서 정리한 아래 구성도 상에 이미 Spring actuator + Prometheus + grafana 연동의 그림이 그려져 있다. 수집 대상 Application은 아래 그림에서 Prometheus target에 해당한다.

즉, Spring actuator를 통해서 Prometheus가 해당 Application 의 데이터를 수집해갈 수 있도록 endpoint를 노출하고, 이를 일정 주기로 job으로 설정한 대로 Prometheus가 가져간다. 이 상태에 이미 Prometheus를 통해서 원하는 metric을 볼 수 있지만 이를 좀더 가독성 있게 시각화 해서 여러 지표를 한데 모아 볼 수 있도록 grafana에서 보도록 연동하는 것이다.

<figure><img src="../../.gitbook/assets/image (10).png" alt=""><figcaption></figcaption></figure>

여기서 조금 더 나아가면 Prometheus Target을 eureka로 구성 할 수 있다.

<figure><img src="../../.gitbook/assets/image (29).png" alt=""><figcaption></figcaption></figure>

MSA 환경에서는 인스턴스들의 정보가 계속 바뀔 수 있기 때문에 이를 discovery 해주는 eureka 서버를 Prometheus 의 Target으로 설정해주고 서비스 name 으로 이를 추적하게 하면 각 인스턴스들을 Application Name 으로 구분하여 추적이 가능하다.
