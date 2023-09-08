# 패키지 구조 및 레이어 설명

<figure><img src="../../../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

도메인 레이어는 자세히 다룰 것은 없다. 다만 repository 를 인터페이스로 선언하고 이를 data 레이어에서 구현해서 사용하는게 정석인데, 불필요한 것 같아서 다음 프로젝트는 계층을 나누지 않을 생각이다.



![](https://fistkim101.github.io/images/concept\_3layer.png)

위 도식처럼 repository 에서 response 를 받아 domain 레이어에 선언한 모델로 변환 작업을 하며, 여기서 선언한 모델을 앱 내에서 사용하게 된다.

