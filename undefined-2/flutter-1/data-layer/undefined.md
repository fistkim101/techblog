# 패키지 구조 및 레이어 설명

<figure><img src="../../../.gitbook/assets/image (2) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

클라이언트 사이드에서 서버 사이드와의 통신을 책임지고 필요한 데이터의 전처리 및 후처리를 책임지는 레이어이다. 위와 같은 패키지 구조를 가지게 된다.

다만, response 의 경우 response -> model 의 과정을 거치지 않고 바로 model 에서 이를 받을 수 있다.
