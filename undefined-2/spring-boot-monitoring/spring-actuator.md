# Spring actuator

## Actuator 란

[스프링 공식문서](https://docs.spring.io/spring-boot/docs/3.0.x/reference/html/index.html)에서는 Production-ready Features 라는 소제목으로 프로덕션 환경에서의 어플리케이션에 대한 모니터링과 관리 이슈를 다루고 있다.

스프링에서는 어플리케이션의 모니터링 및 관리를 위해서 몇 가지 HTTP endpoint 를 노출시키는데, 이 과정을 책임져 주는 라이브러리가 [Spring actuator](https://docs.spring.io/spring-boot/docs/3.0.x/reference/html/actuator.html#actuator) 이다.

Actuator 에 대한 사전적 정의는 아래와 같다.

> Definition of Actuator
>
> An actuator is a manufacturing term that refers to a mechanical device for moving or controlling something. Actuators can generate a large amount of motion from a small change.

mechanical device for moving or controlling 이 핵심인 것 같다. 공식 문서를 보면 여러가지 endpoint들을 제공하고 있음을 알 수 있고, 각 endpoint 마다 노출되는 정보가 다르다. properties 설정을 통해서 노출시킬 endpoint들을 고를 수 있다.

Actuator와 prometheus 연동을 위해서는 스프링 어플리케이션의 여러 정보들을 prometheous 가 가져갈 수 있는 방식으로 노출 시켜줘야한다. 상식적으로 prometheus가 통신하는 인터페이스가 따로 있을 것이기에 거기에 맞게 스프 링 어플리케이션의 내부 데이터를 변형해줄 필요가 있을 것이다.&#x20;



## Actuator + Prometheus

공식문서를 보면 prometheus 라는 endpoint가 존재함을 알 수 있고 설명은 아래와 같다.

> Exposes metrics in a format that can be scraped by a Prometheus server. Requires a dependency on micrometer-registry-prometheus.

설명을 보면 micrometer-registry-prometheus라는 추가적인 라이브러리가 필요함을 알 수 있다.

[micrometer 공식 홈페이지](https://micrometer.io/) 가보면 아래와 같은 설명이 있다.

> Micrometer provides a simple facade over the instrumentation clients for the most popular monitoring systems, allowing you to instrument your JVM-based application code without vendor lock-in. Think SLF4J, but for metrics.

쉽게 말해 여러가지 유명한 모니터링 시스템들에게 facade 하게 인터페이스를 제공해주는 라이브러리이다. 그림으로 보면 아래와 같다.

<figure><img src="../../.gitbook/assets/image (50).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (59).png" alt=""><figcaption></figcaption></figure>

이미 actuator 에 micrometer-core 가 포함되어 있다. prometheus 연동을 위해서는 micrometer-registry-prometheus만 추가적으로 runtime scope 으로 의존성 추가를 해주면 된다.

```bash
    implementation("io.micrometer:micrometer-registry-prometheus")
```



여기까지 Spring Application 의 prometheus 연동을 위한 준비단계로 Actuator와 그 외 필요한 라이브러리에 대해서 살펴본 내용이다.
