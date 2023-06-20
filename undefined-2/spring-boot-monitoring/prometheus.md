# Prometheus

## Prometheus 란

> Prometheus collects and stores its metrics as time series data, i.e. metrics information is stored with the timestamp at which it was recorded, alongside optional key-value pairs called labels.

[공식 홈페이지](https://prometheus.io/docs/introduction/overview/)에 여러 설명이 있지만 위 문구가 가장 핵심이라 생각해서 가져왔다. 위 문장을 정리하면 Prometheus는 metric을 시간에 따라 시계열로 저장하며 이 metric은 시계열인 만큼 각 데이터마다 timestamp 를 갖고 있고, 데이터의 형식은 label 이라 불리우는 key-value 방식이라는 것이다.

정확하게는 각 데이터가 timestamp를 갖는게 아니라 Prometheus가 임의로 판단한 특정 시점에 key-value 에 해당하는 형식으로 데이터(metric)를 수집하는데 이걸 시간이 지남에 따라 계속 반복한다는 의미로 보인다.



## Prometheus 구성

<figure><img src="../../.gitbook/assets/image (42).png" alt=""><figcaption></figcaption></figure>

* the main [Prometheus server](https://github.com/prometheus/prometheus) which scrapes and stores time series data
* [client libraries](https://prometheus.io/docs/instrumenting/clientlibs/) for instrumenting application code
* a [push gateway](https://github.com/prometheus/pushgateway) for supporting short-lived jobs
* special-purpose [exporters](https://prometheus.io/docs/instrumenting/exporters/) for services like HAProxy, StatsD, Graphite, etc.
* an [alertmanager](https://github.com/prometheus/alertmanager) to handle alerts
* various support tools

여기서 정보를 수집할 대상인 Application 은 좌측 하단의 Prometheus targets 이다. Prometheus target이 되는 exporters 들은 Prometheus 가 주기적으로 pull 할 때 사용할 endpoint를 제공하고, Prometheus가 이를 pull하는 행위를 job 이라고 설정하여 행해진다.
