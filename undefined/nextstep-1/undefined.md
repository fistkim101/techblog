# 망 분리하기

## 들어가며 <a href="#0" id="0"></a>

‘그럴듯한 서비스 만들기 - 망 분리하기’ 단원에서는 ‘망’이 무엇인지와 왜 이를 분리하는게 좋을지에 대해 공부했다. 그리고 망을 분리하는 과정에서 알아야할 개념들과 장치들에 대해서도 공부하였다.

## 사전지식 <a href="#1" id="1"></a>

인프라공방의 사전 과제는 OSI(Open Systems Interconnection) 7 layer 에 대해서 학습하는 것이었다. 개별적인 내용들에 대해서 암기하고 있는 것 보다는 OSI 7 layer 라는 개념이 정립된 이유를 이해하는 것이 더 중요하다고 생각한다.

{% embed url="https://youtu.be/1pfTxp25MA8" %}

하나의 개념으로 OSI 7 layer의 필요성에 대해 설명하자면 결국 ‘표준’이 가져다 주는 ‘효율’이라는 개념이 적절하다고 생각한다. 많은 통신장비가 세상에 나오면서 각 통신장비들이 서로 통신하기 위해서는 모두가 합의하는 일종의 약속이 필요하기 때문에 그게 OSI 7 layer든 무엇이든 표준이 정립될 수 밖에 없지 않았나 하는 생각이 든다.

위 동영상은 인프라공방에서 사전에 보고 오라고 링크를 걸어준 것인데 정말 세세하게 각 계층에 대해서 잘 설명해주고 있다.

## 통신망 <a href="#2" id="2"></a>

### **정의**

위키에 설명된 [통신망](https://namu.wiki/w/%ED%86%B5%EC%8B%A0%EB%A7%9D) 에서는 ‘전자신호를 통해 통신하는 모든 기기가 서로 통신하기 위해 만든 하나의 망’이라고 표현하고 있다. 인프라 공방의 학습 자료에는 ‘노드들과 이들 노드들을 연결하는 링크들로 구성된 하나의 시스템’이라고 나와있다.

* 노드 : 노드란, IP를 갖고 있으며 통신할 수 있는 대상
* 링크 : 노드와 노드를 연결해주는 인프라

### **망분리 필요성**

강사님이 망분리의 필요성에 대해서 설명하실때, 보안을 바라보는 관점에 대해서 설명해주셨는데 보안에 대해서 잘 몰랐던 나로서는 굉장히 새로운 관점이었다. 내가 생각했던 보안은 말그대로 지키는 것으로 ‘철통방어’를 목표로 하는 여러 보호 체계를 갖추는 것이라고 생각했었는데, 강사님이 제시해주신 보안의 관점은 기본적으로 ‘철통방어’는 힘들다는 전제를 하고 있었다. 대신 공격자를 최대한 귀찮게하고 공격자가 뚫어야할 장벽들의 수와 양을 늘리는 관점이었다.

아직 잘은 모르겠지만 생각해보면 보안 체계도 다 사람이 만든 것이고, 시간의 문제이지 ‘철통방어’는 사실 불가능 한게 아닌가 싶기도 하다. 아무튼, 이러한 맥락에서 [Defense in depth](https://en.wikipedia.org/wiki/Defense\_in\_depth\_\(computing\)) 라는 개념을 알게 되었다. 단어 그대로 다층적인 layer 구조의 보안체계를 의미한다. 즉, 공격자가 최종 목표까지 뚫고 들어오는 과정에서 다층적으로 장벽에 막히도록 하는 보안 체계에 대한 관점이다.

인프라공방의 실습에서는 하나의 [VPC](https://docs.aws.amazon.com/ko\_kr/vpc/latest/userguide/what-is-amazon-vpc.html) 를 구성하고 VPC 내부에 개인 정보를 다루는 DB 서버 등을 위한 내부망, 사용자가 접근하는 웹서버를 위한 외부망을 구성하여 [Defense in depth](https://en.wikipedia.org/wiki/Defense\_in\_depth\_\(computing\)) 관점에 입각한 네트워크를 구성한다.

## 망 분리 원리 <a href="#3" id="3"></a>

<figure><img src="../../.gitbook/assets/image (1) (4).png" alt=""><figcaption></figcaption></figure>

망 분리 원리를 알기 위해서는 기본적으로 여러 대의 노드들이 어떤 식으로 연결되는지 장비 측면에서의 이해가 필요하다.

* [허브](https://ko.wikipedia.org/wiki/%EC%9D%B4%EB%8D%94%EB%84%B7\_%ED%97%88%EB%B8%8C) : 전기 신호를 증폭 시켜서 전송거리를 연장시키고, 여러 대의 디바이스를 연결해준다. [repeater](https://ko.wikipedia.org/wiki/%EC%A4%91%EA%B3%84%EA%B8%B0) 와 [Flooding](https://en.wikipedia.org/wiki/Flooding\_\(computer\_networking\)) 기능을 수행한다. 구조적으로 데이터를 한 번에 하나만 전송할 수 있기 때문에 두 개 이상의 노드가 동일한 시점에 패킷을 보낼때 발생하게 되는 충돌(collision)의 영역인 [collision domain](https://en.wikipedia.org/wiki/Collision\_domain) 을 공유한다. 결론적으로 허브는 요즘 거의 안쓴다고 한다.
* [스위치](https://ko.wikipedia.org/wiki/%EB%84%A4%ED%8A%B8%EC%9B%8C%ED%81%AC\_%EC%8A%A4%EC%9C%84%EC%B9%98) : 허브의 [collision domain](https://en.wikipedia.org/wiki/Collision\_domain) 문제를 개선한 장비. 자신에게 연결된 노드들의 [MAC 주소](https://ko.wikipedia.org/wiki/MAC\_%EC%A3%BC%EC%86%8C) 와 포트 정보들을 보관하고 있는 MAC Table 을 가지고 있다. MAC Table 에 정보가 있을때 [Forwarding](https://en.wikipedia.org/wiki/Packet\_forwarding) 이 발생하고 MAC Table 에 정보가 없으면 [Flooding](https://en.wikipedia.org/wiki/Flooding\_\(computer\_networking\)) 이 발생한다. 일반적인 L2 스위치 기준으로 OSI 7 layer 에서 DATA layer 까지만 커버한다. 즉, [Broadcast Domain](https://en.wikipedia.org/wiki/Broadcast\_domain) 구분이 불가능하다.
* [라우터](https://ko.wikipedia.org/wiki/%EB%9D%BC%EC%9A%B0%ED%84%B0) : 스위치와 마찬가지로 자신에게 연결된 노드들의 [MAC 주소](https://ko.wikipedia.org/wiki/MAC\_%EC%A3%BC%EC%86%8C) 정보들을 보관하고 있는 MAC Table 을 가지고 있다. MAC Table 에 정보가 있을때 [Forwarding](https://en.wikipedia.org/wiki/Packet\_forwarding) 이 발생하고 MAC Table 에 정보가 없으면 스위치와 다르게 Drop 이 발생한다. [Broadcast Domain](https://en.wikipedia.org/wiki/Broadcast\_domain) 구분이 가능.

결론적으로 하나의 망은 라우터와 같은 3계층 이상의 네트워크 장비를 통해 구성한다.

## VPC <a href="#4-vpc" id="4-vpc"></a>

인프라공방에서는 AWS의 [VPC](https://docs.aws.amazon.com/ko\_kr/vpc/latest/userguide/what-is-amazon-vpc.html) 를 생성하고 그 아래에 여러 망들을 구성한다. VPC 내에서 사용되는 여러 개념들이 있는데 아래와 같다.

* 서브넷 : VPC에 설정한 네트워크 대역을 더 세부적으로 나눈 네트워크
* 라우팅테이블 : 네트워크 트래픽을 전달할 위치를 결정하는 데 사용하는 라우팅이라는 이름의 규칙 집합
* 인터넷 게이트웨이 : VPC의 리소스와 인터넷 간의 통신을 활성화하기 위해 VPC에 연결하는 게이트웨이

## 서브네팅 <a href="#5" id="5"></a>

서브네팅이란 VPC에 설정한 네트워크 대역을 여러 서브넷으로 나누는 행위를 의미한다. 서브네팅을 할 때에는 [CIDER](https://ko.wikipedia.org/wiki/%EC%82%AC%EC%9D%B4%EB%8D%94\_\(%EB%84%A4%ED%8A%B8%EC%9B%8C%ED%82%B9\)) 방식을 따른다. CIDER를 짧게 요약하자면, A. B. C. D/N 의 형태에서 N의 길이 만큼이 공유되는 호스트의 주소이며 각각의 블록은 8비트로 이뤄진다. 이러한 방식을 자세히 설명한 [블로그](https://blog.naver.com/ncloud24/221208338209) 를 링크로 남겨둔다.

* [AZ](https://docs.aws.amazon.com/ko\_kr/AWSEC2/latest/UserGuide/using-regions-availability-zones.html#concepts-availability-zones) : AWS에 존재하는 개념으로 Availability Zone(가용영역)의 줄임말이다. 가용 영역별로 물리적으로 격리되어 있다. 이러한 이점을 이용하면 장애 내성이 더 강화될 수 있다. 예를 들어 2개 이상의 서버를 이용하여 서비스를 운영할 경우 각각의 서버를 각기 다른 AZ에 위치시킨다면 하나가 문제가 생겨도 다른 하나는 격리되어 있기 때문에 확률적으로 서비스가 계속 유지될 가능성이 크고 이는 곧 서비스의 장애내성이 강화되었다는 의미도 되기 때문이다.

## 외부 네트워크와 연결하기 <a href="#6" id="6"></a>

VPC 내에 생성한 EC2는 [인터넷 게이트웨이](https://docs.aws.amazon.com/ko\_kr/vpc/latest/userguide/VPC\_Internet\_Gateway.html) 를 통해서 외부와 연결될 수 있다. 즉, 모든 수신 과정에서 VPC의 인터넷 게이트웨이를 거치게 되고 모든 송신 역시 인터넷 게이트웨이를 거치게 된다. 이에 관해 학습 자료에 정리된 내용을 그대로 인용한다.

> 서브넷에 속한 EC2가 외부와 통신을 하기 위해서는 자신의 라우팅 테이블에 설정된 정보를 제외한 모든 대역(0.0.0.0/0)에 대한 통신을 Internet Gateway로 요청하게끔 설정해두어야 합니다. 라우팅 테이블을 생성하고 0.0.0.0/0 대역을 앞에서 생성한 Internet Gateway로 매핑합니다.

\
