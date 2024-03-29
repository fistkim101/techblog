# 8장 OCP: 개방-폐쇄 원칙

## OCP 정의

> 소프트웨어 개체는 확장에는 열려 있어야 하고, 변경에는 닫혀 있어야 한다.

이건 이미 일반적으로 다룬 OCP 와 동일한 내용이었다. 조금 더 쉽게 설명하자면 소프트웨어 개체의 행위는 확장 할 수 있어야 하지만, 확장 하는 과정에서 기존 개체의 변경은 발생하면 안된다는 원칙이다.

조금 더 적나라하게 말하자면 확장을 할때 코드 추가만 가능하지 기존 것을 건들면 안된다는 것이며 이렇게 지켜질 수 있도록 처음에 설계부터 감안을 해야 한다는 것이다.



## OCP 가 지켜지려면 결국 DIP

OCP 를 지켜지도록 하려면 '핵심 비지니스 규칙을 관장하는 개체들을 최대한 보호'하는 것이다. 여기서 말하는 보호란 외부의 변화로부터 핵심 비즈니스 규칙을 최대한 지키는 것이다. 이걸 또 풀어서 말하자면 '지킨다'는 의미는 외부의 변화와 별개로 해당 개체(핵심 비즈니스 규칙을 관장하는 개체)는 최대한 변경이 발생하지 않아야 한다는 것이다.

이렇게 하려면 지키고자 하는 핵심 비즈니스 규칙을 관장하는 개체가 의존하는 대상이 최대한 변화 가능성이 없는 고수준 모듈이어야 한다. 추상화 수준이 가장 높을 수록 '명세' 자체는 변할 가능성이 적기 때문이다. 단지 필요시 구현체의 세부 내용만 변경하거나 구현체를 추가하면 시스템이 요구사항을 충족할 수 있도록 하는 것이 최선의 방법이다.



## 의존 방향(화살표)이 의미하는 바

A -> B 로 의존하고 있다면 A가 B를 '호출한다' 라고도 말할 수 있다.

의존한다는 것에 대해서 잘 생각해보자. A 가 B에 의존하고 있는데 B가 변경되면 A는 영향을 받을 수 밖에 없다. 반대로 A가 B에 의존하고 있는 상황에서 A가 어떻게 바뀌든 B는 알지도 못하고 알 필요도 없다. A의 변경으로 부터 자유롭다는 것이다.

따라서 보호하고자 하는 것이 있다면 최대한 이것이 다른 것에 의존하지 않도록 하고, 다른 것들이 이것에 의존하도록 하는 것이 보호하는 최선의 방법이다. 책에서도 아래와 같은 문구가 있다.

> 다시 한번 말하지만, A 컴포넌트에서 발생한 변경으로부터 B 컴포넌트를 보호하려면 반드시 A 컴포넌트가 B 컴포넌트에 의존해야 한다.

<figure><img src="../../../.gitbook/assets/2023. 4. 5. - 0 9.jpg" alt=""><figcaption></figcaption></figure>

여기서 보면 Interactor 가 최고 수준으로 보호를 받고 있음을 알 수 있다. 이렇게 보호 해야할 대상은 결국 핵심 비즈니스 규칙을 관리하는 주체여야 한다.



## 헥사고날 아키텍처의 in port 와 방향성 제어

<figure><img src="../../../.gitbook/assets/2023. 4. 5. - 0 10.jpg" alt=""><figcaption></figcaption></figure>

여기서 FinancialReportRequester 인터페이스의 경우 방향성 제어의 목적이 아니다. 이는 FinancialReportController 가 Interactor 내부에 대해서 너무 많이 알지 못하도록 하는 역할이다. 만약 이 인터페이스가 없다면 '추이 종속성'이 발생하여 Financialentities 에 대해서 FinancialReportController가 '추이 종속성'을 가지게 되어서 '자신이 직접 사용하지 않는 요소에는 절대로 의존해서는 안 된다'는 원칙을 위반하게 된다.

요약하자면 FinancialReportController 로 부터 Interactor 를 보호하는게 가장 우선이지만, 반대로 Interactor 에서 발생한 변경에서부터 FinancialReportController도 보호하는 역할을 FinancialReportRequester 인터페이스가 하고 있는 것이다.

이걸 굳이 짚고 넘어가는 이유는 헥사고날에서 in과 out 의 포트의 역할이 곧 이것과 일맥 상통하기 때문이다.
