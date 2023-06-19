# 부하 테스트

## 들어가며 <a href="#0" id="0"></a>

(강의자료 발췌)

> 1. 서비스의 안정성이 완벽할 수 없으며, 일정 부분 장애를 허용할 수 밖에 없습니다. 다만, 심각한 문제에도 어느 정도의 장애내성을 가진 서비스를 운영하는냐가 관건입니다.
> 2. 고가용성, 무중단 서비스를 지향하는데 있어 핵심은 사용자가 납득할만한 수준의 [가용성](https://terms.naver.com/entry.naver?docId=2807122\&cid=40942\&categoryId=32843) 을 유지하되, 배포 사이클을 유지하는 것입니다.
> 3. 서비스를 배포하기에 앞서 예상되는 상황을 테스트하여, 현재 시스템이 어느 정도의 부하를 견딜 수 있는지 확인하고. 한계치에서 병목이 생기는 지점을 파악하고 장애 조치와 복구를 사전에 계획해둘 필요가 있습니다.

무조건 ‘장애’는 없도록 해야한다는 생각에서 ‘장애’를 인정함과 동시에(포기하는 것이 아니라 언제든 항상 발생할 수 있다고 전제) 내성을 갖도록 해야한다는 생각으로 변하게 되었다. 생각해보면 애초에 ‘장애’가 무조건 없게끔 하는 것은 불가능하고 차라리 모든 가능성을 열어두고 개별 장애 시나리오에 대해서 미리 해소를 해두거나, 발생 시나리오에 따른 대응 전략을 수립해두는 것이 정석인 것 같다.

2번의의 경우도 고가용성을 유지하면서도 원하면 언제든 빠르게 배포할 수 있는 배포 사이클을 유지해야한다는 이야기인데, 나의 실무 경험상 무중단 배포는 해보았지만 그 빈도가 많지 않았고 애초에 배포 전에 해야할 것들(부하테스트 등)이 시스템적으로 있지도 않았어서 ‘배포 사이클’에 대한 의미있는 고민을 해보지 못했다. 그런 의미에서 ‘원하는 사항을 빠르게 배포할 수 있는가’에 대한 필요성과 중요성에 대해서 다시 생각해볼 수 있었다.



## 1. 사전지식 <a href="#1" id="1"></a>



### **A. 다중화**

<figure><img src="http://localhost:4000/assets/images/infra/multiserver.png" alt=""><figcaption></figcaption></figure>



[단일 장애점](https://ko.wikipedia.org/wiki/%EB%8B%A8%EC%9D%BC\_%EC%9E%A5%EC%95%A0%EC%A0%90) 을 없애는 행위라고도 볼 수 있으며, 다중화의 대상은 Server, Load balancer, Network Device, DB 등이다. [단일 장애점](https://ko.wikipedia.org/wiki/%EB%8B%A8%EC%9D%BC\_%EC%9E%A5%EC%95%A0%EC%A0%90) 의 개념 자체가 ‘전체 시스템의 중단’을 내포하고 있으므로 다중화의 중요성은 따로 강조하지 않아도 될 것 같다. 하지만 다중화는 곧 비용을 증가시키므로 적절한 수준에서 다중화를 이뤄야한다.

단일 장애점을 없앤다 => 다중화를 한다 => 전체 시스템이 멈출 가능성을 낮춘다 => 장애 내성이 올라간다



### **B. 부하**

부하란 처리를 실행하려고 해도 실행할 수 없어서 대기하고 있는 프로세스의 수를 의미. [부하에 대해 상세히 정리된 블로그](https://brainbackdoor.tistory.com/117)

성능에 관해서는 여러 관점에서 논할 수 있다.

> 우리는 서비스가 얼마나 빠른지(Time), 일정 시간 동안 얼마나 많이 처리할 수 있는지([TPS](https://ko.wikipedia.org/wiki/%EC%B4%88%EB%8B%B9\_%ED%8A%B8%EB%9E%9C%EC%9E%AD%EC%85%98\_%EC%88%98)), 그리고 얼마나 많은 사람들이 동시에 사용할 수 있는지(Users)에 대해 이야기합니다.



### **C. 사용자**

<figure><img src="http://localhost:4000/assets/images/infra/user_category.png" alt=""><figcaption></figcaption></figure>

**사용자를 굉장히 잘 시각화하여 구분해두었다. 최초에 누가 만들었을까, 참 잘 만들었다**

\


부하 테스트시 사용자를 어떻게 정의할 것인지도 굉장히 중요한 것 같다. 결국 부하를 야기하는 주체가 명확해야 발생할 가능성이 높은 시나리오를 세울 수 있고, 이에 따라 대책도 유효할 가능성이 크니까 말이다.

> 1. 시스템 관리자의 관점에서는 등록된 사용자와 등록되지 않은 사용자만이 존재합니다.
> 2. 서버의 관점에선 로그인한 사용자와 로그인하지 않은 사용자만이 존재합니다.
> 3. 성능 테스터의 관점에선 사용자가 Concurrent User 인지 Active User 인지가 중요합니다. 여기서 Concurrent User란, 웹 페이지를 띄어놓은 사용자처럼, 언제든지 부하를 줄 수 있는 사용자를 의미합니다. 반면, Active User는 메뉴나 링크를 누르고 결과가 나오기를 기다리는 등 실제로 서버에 부하를 주고 있는 사용자를 의미합니다.
> 4. Active User 와 Concurrent User 의 비율은 서비스의 성격에 따라 다르므로 이 점을 감안하고 성능테스트를 계획해야 합니다. (성능 테스트시에 VUser는 Active User와 유사합니다.) 가령 수강신청의 경우, 특정 시간대엔 그 비율이 90%에 육박할 수 있어, 전체 평균을 기준으로 테스트할 경우 잘못된 판단을 이끌어낼 수 있습니다.

\


### **D. Scaling**

<figure><img src="http://localhost:4000/assets/images/infra/scaling.png" alt=""><figcaption></figcaption></figure>



\
주의해야 할 것이 단일 사용자에 대한 응답 속도가 느리다 할지라도 Scale up을 해야 하는 것이 아니라 애플리케이션 단의 문제 혹은 slow 쿼리의 문제 혹은 기타 다른 이슈로 느린 것일 수 있으므로 전반적인 차원에서 원인 분석을 해야 정확한 원인을 파악할 수 있다.

> 성능에 문제가 있는 경우엔, 단일 사용자에 대한 응답 속도가 느려집니다. 확장성에 문제가 있는 경우엔, 당장은 단일 사용자에게는 빠르지만 부하가 많아질 경우 속도가 느려질 수 있습니다.



### **E. Time**

<figure><img src="http://localhost:4000/assets/images/infra/test_time.png" alt=""><figcaption></figcaption></figure>



성능 테스트시에는 정확히 어떤 구간을 테스트할 것인가를 명확하게 생각하고 테스트를 해야 한다. 그리고 기본적인 것이지만 이게 가능하려면 결국 최초의 request가 어떤 절차를 거쳐서 reponse가 클라이언트까지 도달하는지에 대해서 상세히 알고 있어야 한다.

> * 브라우저와 웹 서버간 구간에서는 정적 파일 크기, Connection 관리, 네트워크 환경 등의 영향을 받을 수 있습니다.
> * Server 구간에서 발생한 경우, DB와 애플리케이션 간 연결의 문제, 프로그램 로직 상의 문제 혹은 서버의 리소스 부족 등을 의심해 볼 수 있습니다.
> * 네트워크 이슈의 경우 테스트하는 환경에 따라 달라질 수도 있습니다. 지연 현상은 사용자의 이탈과 매우 밀접하기에 개선되어야 하지만, 단순히 서버를 늘린다고(Scale out) 해결되는 것은 아닙니다. 이에 출시 전에 테스트를 하여 최대 응답시간을 파악하고 있어야 하며, 상위 5%의 화면이 95% 사용자 요청을 받는다는 점을 감안하고 튜닝의 대상을 선정해가야 합니다.

## 2. 테스트 준비하기 <a href="#2" id="2"></a>



### **A. 부하 테스트 목적**

여기서의 테스트는 로직에 대한 테스트가 아니라 부하와 관련된 테스트이다.

> * 각 시스템의 응답 성능 및 한계치를 알 수 있어요.
> * 부하가 많이 발생할 때 나타나는 증상을 확인하고 성능을 개선할 수 있어요.
> * 서비스가 확장성을 가졌는지 확인할 수 있어요.



### **B. 부하 테스트 종류**

#### [**SMOKE TEST**](https://en.wikipedia.org/wiki/Smoke\_testing\_\(software\))

기본적인 테스트로 시나리오 대로 애플리케이션이 작동하는지를 확인하는 테스트다. 참조를 건 위키에 보면 아래와 같은 예시가 있는데 SMOKE TEST에 대해서 잘 표현해놨다

> a smoke test may address basic questions like “does the program run?”, “does the user interface open?”

#### [**STRESS TEST**](https://en.wikipedia.org/wiki/Stress\_testing)

![](http://localhost:4000/assets/images/infra/stress\_test.png)



계속 부하 레벨을 올려가며 한계치를 파악하는 테스트. 위키에 보면 ‘torture testing’ 라고 표현되어 있다.

> * 서비스가 극한의 상황에서 어떻게 동작하는지 확인합니다.
> * 장기간 부하 발생에 대한 한계치를 확인하고 기능이 정상 동작하는지 확인합니다.
> * 최대 사용자 또는 최대 처리량을 확인합니다.
> * 스트레스 테스트 이후 시스템이 수동 개입없이 복구되는지 확인합니다.



#### [**LOAD TEST**](https://en.wikipedia.org/wiki/Load\_testing)

<figure><img src="http://localhost:4000/assets/images/infra/load_test.png" alt=""><figcaption></figcaption></figure>

LOAD TEST가 강의자료의 설명만 봐서는 와닿지 않았는데, 위키를 보니 좀 이해가 되었다. 위키에 보면 아래와 같이 나와있다.

> ‘Load testing generally refers to the practice of modeling the expected usage of a software program by simulating multiple users accessing the program concurrently.\[1]’

\
여기서 ‘expected usage of a software program’ 에서 STRESS TEST와의 차이를 알 수 있다. 강의 자료에서 말하는 ‘최대 트래픽’ 은 결국 ‘평소의 최대 트래픽’의 기준을 이야기 하는 것이라고 이해했다. 그러면 LOAD TEST의 핵심은 결국 ‘평소 최대 트래픽’에 대한 정확한 판단이 중요하다는 이야기가 되는 것 같다.

> * 서비스의 평소 트래픽과 최대 트래픽 상황에서 성능이 어떤지 확인합니다. 이 때 기능이 정상 동작하는지도 확인합니다.
> * 애플리케이션 배포 및 인프라 변경(scale out, DB failover 등)시에 성능 변화를 확인합니다.
> * 외부 요인(결제 등)에 따른 예외 상황을 확인합니다.

### **C. 부하 테스트 도구**

강사님은 미션에서는 K6를 쓴다고 하셨다. 하나도 안써봐서 무엇들이 차이가 있는지 아직 잘 모르겠다. nGrinder는 빌드시점에 일정량의 부하를 견딜 수 있는지를 검사하는 과정을 넣을 수 있다고 하는데, 배포사이클이 길어지는 대신 서비스의 안정성을 조금 더 챙길 수 있는 것 같다. 하지만 배포사이클이 길어진다는 것 자체가 결국 유사시 대처 속도를 느리게 만들어서 서비스 안정성을 저해하는 것 같기도 하다. 이런게 어찌보면 참 재밌는 것 같다.

> 부하 테스트 도구로는 Apache JMeter, nGrinder, Gatling, Locust, K6 등의 도구가 있습니다.
>
> * 시나리오 기반의 테스트가 가능해야 합니다.
> * 동시 접속자 수, 요청 간격, 최대 Throughput 등 부하를 조정할 수 있어야 합니다.
> * 부하 테스트 서버 스케일 아웃을 지원하는 등 충분한 부하를 줄 수 있어야 합니다.



### **D. 주의할 점**

> 1. 성능 테스트는 실제 사용자가 접속하는 환경에서 진행하여야 합니다. 내부 네트워크에서 부하를 발생시킬 경우 응답시간에 차이가 발생할 수 있습니다.
> 2. 부하 테스트에서는 클라이언트 내부 처리시간이 배제되어 있음을 염두해두어야 합니다.
> 3. 테스트 DB에 들어 있는 데이터의 양이 실제 운영 DB와 동일하여야 합니다. 통상 전체 성능의 70% 이상이 DB에 좌우되는데, 테스트 대상이 되는 테이블의 데이터 양이 다르면 쿼리의 실행계획이 달라져 성능이 다르게 나타날 수 있습니다. 또한 데이터가 소량이면 디스크 입출력이 일어나야 하는데 모두 메모리에 로드되어 성능이 빠른 것으로 착각할 수 있습니다.
> 4. 운영환경의 경우, 서비스 요청 외에 별도로 수행되는 배치나 후속작업으로 인한 부하가 있을 수 있습니다. 서버에 일정하게 발생하는 부하가 있다면 성능 테스트 시나리오에도 포함해야 합니다.
> 5. 외부 요인(결제 등)의 경우 시스템과 분리된 별도의 서버로 구성해야 합니다. 객체를 Mocking하는 경우 Http Connection Pool, Connection Thread 등을 미사용하게 되고 IO가 발생하지 않습니다. 같은 애플리케이션에 Dummy Controller를 구성하는 경우 테스트 시스템의 자원과 리소스를 같이 사용하므로 테스트의 신뢰성이 떨어집니다.

\
1번은 백번 맞는 말이긴 하나 환경을 갖추기가 쉽지 않은 것 같다. 이전의 회사에서도 개발서버와 운영서버가 애초에 AWS 인스턴스 종류도 달랐고, 개발서버 내에는 테스트 하려는 서비스 외에 이것저것 다른 것들이 다 붙어있었어서 사실 1번의 맥락에서 보자면 개발서버에서 테스트 하는게 무의미 했다. 운영서버와 환경을 정말 똑같이 갖춰놓고 테스트를 해야 진짜 확실한 테스트가 되는 것 같긴 하다. 그럼 정말 엄격하게 테스트를 하려면 운영의 replica 수준으로 테스트 전용 서버를 셋팅 해야 하는 것 같다.

4번은 1번과 같은 맥락에 이어 해석을 할 수 있는 것 같은데, 4번의 전제가 ‘운영과 완전히 똑같은 테스트 환경을 갖추기 힘드니까 다른 부하가 혹시 존재한다면 그것도 고려를 해라’는 의미로 보인다. 운영 환경에서는 일정량의 피할 수 없는 부가적인 부하가 있는데 테스트에 이게 없으면 ‘일정량의 부하가 더 있을 것’을 감안해야하고, 더 좋은 것은 애초에 테스트 환경을 갖출 때에 운영상에서 발생하고 있는 모든 부하를 똑같이 맞춰주는 것이라 생각한다.

3번 내용의 실천을 위해서 운영 DB를 개발 DB에 똑같이 맞춰주는 과정을 빠르게 할 수 있도록 방법을 숙지하든 시스템적으로 뭔갈 갖춰놓든 하는 것이 중요한 것 같다. DB 설정으로 운영DB와 테스트DB를 replication 걸어두고 테스트때 바로 테스트 DB에 붙어서 사용할 수 있도록 하는 방식도 있겠다.다 (실무에서 실제로 이렇게 하는지는 모르겠다. 내가 겪은 경험의 폭 내에서는 애초에 부하테스트도 해보질 못해서 잘 모르겠다.)

5번의 경우 일단은 코드 내에서 예외처리를 잘 해놓는게 무엇보다 중요한 것 같다.

* 응답이 제대로 온다.
* 응답이 제대로 왔지만 request 가 특정한 사유에 의해 불충분했다.
* 응답이 제대로 오지만 시간이 엄청 오래 걸린다.
* 응답해줄 서버가 문제가 있어서 아예 응답이 오지 않았다.

모든 사안에 대해서 일단 코드상 대비가 되어있어야 하고, 사안에 따라서 실패할 경우 나중에 이를 처리할 필요가 있다면 실패시 로그에 의존할 게 아니라 아예 DB로 따로 빼서 저장해둬야한다. 테이블을 만들고 DB에 저장한다는 것 자체가 그만큼 저장할 가치가 있다는 의미이고, 그 정도 의미가 있는(기획상 돈과 관련이 있다던가, 꼭 응답 받았어야할 것이었다던가 아무튼 예민한 무엇인가와 연관되어 있다면) 것이면 실패시 나중을 위해서 DB에 저장을 해야한다고 생각한다.

### **E. 목표설정**

강의 자료에서는 아래와 같이 목표설정을 하라고 안내되어 있는데, 일단 이건 실습하면서 다시 파봐야할 것 같다.

> * 전제 조건 정리
>   * 테스트하려는 Target 시스템의 범위를 정해야 합니다.
>   * 부하 테스트시에 저장될 데이터 건수와 크기를 결정하세요. 서비스 이용자 수, 사용자의 행동 패턴, 사용 기간 등을 고려하여 계산합니다.
>   * 목푯값에 대한 성능 유지기간을 정해야 합니다.
>   * 서버에 같이 동작하고 있는 다른 시스템, 제약 사항 등을 파악합니다.
>
> \
>
>
> * 목푯값 설정
>   1. 우선 예상 1일 사용자 수(DAU)를 정해봅니다.
>   2. 피크 시간대의 집중률을 예상해봅니다. (최대 트개픽 / 평소 트래픽)
>   3. 1명당 1일 평균 접속 혹은 요청수를 예상해봅니다.
>   4. 이를 바탕으로 Throughput을 계산합니다.
> * Throughput : 1일 평균 rps \~ 1일 최대 rps
>   * 1일 사용자 수(DAU) x 1명당 1일 평균 접속 수 = 1일 총 접속 수
>   * 1일 총 접속 수 / 86,400 (초/일) = 1일 평균 rps
>   * 1일 평균 rps x (최대 트래픽 / 평소 트래픽) = 1일 최대 rps
> * Latency : 일반적으로 50\~100ms이하로 잡는 것이 좋습니다.
> * 사용자가 검색하는 데이터의 양, 갱신하는 데이터의 양 등을 파악해둡니다.



결국 서버의 성능 목표는 [Throughput](https://ko.wikipedia.org/wiki/%EC%8A%A4%EB%A3%A8%ED%92%8B) 과 [Latency](https://ko.wikipedia.org/wiki/%EB%A0%88%EC%9D%B4%ED%84%B4%EC%8B%9C) 를 기준으로 결정해야하는데, 이 내용에 관해서 [친절히 설명된 블로그](https://hyuntaeknote.tistory.com/10) 를 찾았다. 성능 개선에 대한 포인트까지 잘 설명되어 있어서 정말 많은 도움이 되었다.