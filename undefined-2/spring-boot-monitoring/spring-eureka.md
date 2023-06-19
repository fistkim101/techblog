# Spring eureka

공식문서가 생각보다 모든 내용을 다루는 것 같지는 않다.

{% embed url="https://cloud.spring.io/spring-cloud-netflix/reference/html/" %}

eureka 에 대해서 알아보기 전에 Service discovery 의 필요성이나 개념을 먼저 알아야 한다. 아래 포스팅에 매우 잘 정리가 되어 있다.d

{% embed url="https://www.nginx.com/blog/service-discovery-in-a-microservices-architecture" %}

Service Discovery 의 필요성에 대해서 요약하자면 아래의 그림으로 설명할 수 있다.

<figure><img src="../../.gitbook/assets/image (9).png" alt=""><figcaption></figcaption></figure>

예전과는 다르게 오늘날의 cloud-based MSA 에서는 위와 같이 service instance 들이 network location 을  동적으로 할당받게 된다. 왜냐하면 autoscaling 이 발생하기 때문이다. 결과적으로 client 가 어디로 통신해야할지 모르는 상황이 발생할 수 있다. 이 때 service instance 들의 network location을 파악하는 행위를 service discovery 라고 할 수 있다.

위 포스팅에서는 service discovery pattern 으로 client side, server side 두 가지를 설명하고 있다.



## Client-Side Discovery Pattern

client side 는 client 가 로드밸런싱의 책임을 갖고 있는 형태이다. service instance들이 registry 에 등록하는 것은 맞는데 여기서 client 가 registry 에 질의한 후 알아서 로드밸런싱 판단을 해야하는 것이다. Netflix OSS 가 대표적인 client side discovery pattern 이라고 한다.

<figure><img src="../../.gitbook/assets/image (15).png" alt=""><figcaption></figcaption></figure>



## Server-Side Discovery Pattern

<figure><img src="../../.gitbook/assets/image (37).png" alt=""><figcaption></figcaption></figure>

반면에 server side discovery pattern은 client 의 요청이 service 를 직접 향하지 않고 로드밸런서를 향한다. 즉, 요청을 로드밸런서를 통해서 하게 된다.

AWS ELB가 server side discovery pattern의 예시라고 한다.

> The [AWS Elastic Load Balancer](https://aws.amazon.com/elasticloadbalancing/) (ELB) is an example of a server-side discovery router. An ELB is commonly used to load balance external traffic from the Internet. However, you can also use an ELB to load balance traffic that is internal to a virtual private cloud (VPC). A client makes requests (HTTP or TCP) via the ELB using its DNS name. The ELB load balances the traffic among a set of registered Elastic Compute Cloud (EC2) instances or EC2 Container Service (ECS) containers. There isn’t a separate service registry. Instead, EC2 instances and ECS containers are registered with the ELB itself.



## Self-Registration Pattern

<figure><img src="../../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

위에서 다뤘듯이 client side나 server side나 instance 가 Service Registry 에 등록을 한다는 사실은 변함이 없다. 이 때 등록을 어떤식으로 하는지에 대한 것을 Self-Registration 이라는 개념으로 설명하고 있다.

내가 eureka를 보면서 가장 궁금했고 헷갈렸던 부분도 이 부분이다. 단지 MSA에서 여러 instance 들의 인프라 정보(위치 등) 을 한 곳으로 모은다 정도로만 알고 있었고, 나는 세부적으로 어떤 방식으로 무엇을 모으는지를 알고 쓰고 싶었는데 공식 문서에서 내가 못찾은 건지 도무지 찾을 수 없었다.

결국 client와 registry 모두를 켜놓고 둘다 로그 레벨을 root debug로 놓고 로그를 통해서 뭘 하는지 알 수 있었는데 이 포스팅에서 그 사실을 매우 정확하게 설명해주고 있다.

> When using the [self‑registration pattern](https://microservices.io/patterns/self-registration.html), a service instance is responsible for registering and deregistering itself with the service registry. Also, if required, a service instance sends heartbeat requests to prevent its registration from expiring. The following diagram shows the structure of this pattern.

위 그림과 위 설명에 나와있듯이 service instance가 등록의 책임을 갖고 있다. 그리고 등록을 한뒤 일정하게 heartbeat을 보내서 regitstry 에서 사라지는 것을 방지한다. 같은 맥락에서 registry 는 일정 기간 생존신고를 하지 않는 instance 는 죽었다고 판단하고 instance 정보에서 삭제한다. 그림에서 unregister() 가 있는데, service instance가 내려갈때 '나 죽는다' 라고 신고까지 한다.  또 그림에서 나오진 않았는데 eureka client는 registry 에 등록된 모든 다른 instance 정보들도 가져간다.

정리하자면 eureka 기준으로 client 가 하는 행위를 목록화 하면 아래와 같다.

1. r**egister(1회)**
2. **heartbeat(지속적으로)**
3. **fetch(registry 에 등록된 다른 app들의 정보(최초 1회 이후 delta 로 변화만 받아감))**
4. **unregister(1회)**



이를 eureka 로그에서 보면 아래와 같다. 로그는 모두 service registry 에서 기록된 것이다.

### 1. register (\[POST /eureka/apps/SAMPLE-APP)

```bash
2022-07-03 12:02:53.816 DEBUG 1737 --- [nio-8761-exec-9] o.a.coyote.http11.Http11InputBuffer      : Received [POST /eureka/apps/SAMPLE-APP HTTP/1.1
Accept: application/json, application/*+json
Accept-Encoding: gzip
Content-Type: application/json
Authorization: Basic YWN0dWF0b3I6MjAw
Content-Length: 968
Host: localhost:8761
Connection: Keep-Alive
User-Agent: Apache-HttpClient/4.5.13 (Java/11.0.15)
```

body 에는 아래와 같은 내용이 들어간다.

```json
{
  "instance": {
    "instanceId": "192.168.35.111:sample-app:8070",
    "app": "SAMPLE-APP",
    "appGroupName": null,
    "ipAddr": "192.168.35.111",
    "sid": "na",
    "homePageUrl": "http://localhost:8070/",
    "statusPageUrl": "http://localhost:8070/actuator/info",
    "healthCheckUrl": "http://localhost:8070/actuator/health",
    "secureHealthCheckUrl": null,
    "vipAddress": "sample-app",
    "secureVipAddress": "sample-app",
    "countryId": 1,
    "dataCenterInfo": {
      "@class": "com.netflix.appinfo.InstanceInfo$DefaultDataCenterInfo",
      "name": "MyOwn"
    },
    "hostName": "localhost",
    "status": "UP",
    "overriddenStatus": "UNKNOWN",
    "leaseInfo": {
      "renewalIntervalInSecs": 10,
      "durationInSecs": 20,
      "registrationTimestamp": 0,
      "lastRenewalTimestamp": 0,
      "evictionTimestamp": 0,
      "serviceUpTimestamp": 0
    },
    "isCoordinatingDiscoveryServer": false,
    "lastUpdatedTimestamp": 1656817373188,
    "lastDirtyTimestamp": 1656817373767,
    "actionType": null,
    "asgName": null,
    "port": {
      "$": 8070,
      "@enabled": "true"
    },
    "securePort": {
      "$": 443,
      "@enabled": "false"
    },
    "metadata": {
      "management.port": "8070"
    }
  }
}
]
```



### **2. heartbeat(PUT /eureka/apps/SAMPLE-APP/192.168.35.111:sample-app:8070?status=UP\&lastDirtyTimestamp=1656817373767)**

```bash
2022-07-03 12:07:14.316 DEBUG 1737 --- [nio-8761-exec-9] o.a.coyote.http11.Http11InputBuffer      : Received [PUT /eureka/apps/SAMPLE-APP/192.168.35.111:sample-app:8070?status=UP&lastDirtyTimestamp=1656817373767 HTTP/1.1
Accept: application/json, application/*+json
Authorization: Basic YWN0dWF0b3I6MjAw
Content-Length: 0
Host: localhost:8761
Connection: Keep-Alive
User-Agent: Apache-HttpClient/4.5.13 (Java/11.0.15)
Cookie: JSESSIONID=D80727444AEA1B1E1998EE338DE98901
Accept-Encoding: gzip,deflate

]
```



### 3. fetch

```bash
// 최초 (GET /eureka/apps/)
2022-07-03 12:32:57.799 DEBUG 1737 --- [nio-8761-exec-1] o.a.coyote.http11.Http11InputBuffer      : Received [GET /eureka/apps/ HTTP/1.1
Accept: application/json, application/*+json
Authorization: Basic YWN0dWF0b3I6MjAw
Host: localhost:8761
Connection: Keep-Alive
User-Agent: Apache-HttpClient/4.5.13 (Java/11.0.15)
Accept-Encoding: gzip,deflate

]

// 최초 이후(GET /eureka/apps/delta)
2022-07-03 12:33:17.909 DEBUG 1737 --- [io-8761-exec-10] o.a.coyote.http11.Http11InputBuffer      : Received [GET /eureka/apps/delta HTTP/1.1
Accept: application/json
DiscoveryIdentity-Name: DefaultClient
DiscoveryIdentity-Version: 1.4
DiscoveryIdentity-Id: 192.168.35.111
Accept-Encoding: gzip
Host: localhost:8761
Connection: Keep-Alive
User-Agent: Java-EurekaClient/v1.10.17
Cookie: JSESSIONID=7C6D8BF2899A76C4A71ABCD5B1338BA4
Authorization: Basic YWN0dWF0b3I6MjAw

]
```



### 4. unregister

```bash
2022-07-03 12:32:50.008 DEBUG 1737 --- [nio-8761-exec-2] o.a.coyote.http11.Http11InputBuffer      : Received [DELETE /eureka/apps/SAMPLE-APP/192.168.35.111:sample-app:8070 HTTP/1.1
Accept: application/json, application/*+json
Authorization: Basic YWN0dWF0b3I6MjAw
Content-Length: 0
Host: localhost:8761
Connection: Keep-Alive
User-Agent: Apache-HttpClient/4.5.13 (Java/11.0.15)
Cookie: JSESSIONID=0AB148142FBDFF067B548B698A243113
Accept-Encoding: gzip,deflate

]
```



eureka service registry 및 client 설정은 정리된 포스팅도 많거니와 공식문서만 보고 따라가면 충분히 처리할 수 있어서 정리를 생략한다.
