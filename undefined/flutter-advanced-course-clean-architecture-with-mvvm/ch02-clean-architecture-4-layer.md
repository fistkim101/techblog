# CH02 Clean Architecture 4 Layer

## 강의에서 다루는 4 Layer 내용

&#x20;     &#x20;

<figure><img src="https://fistkim101.github.io/images/flutter_clean_architecture_01_1.png" alt=""><figcaption></figcaption></figure>

<figure><img src="https://fistkim101.github.io/images/flutter_clean_architecture_01_2.png" alt=""><figcaption></figcaption></figure>

<figure><img src="https://fistkim101.github.io/images/flutter_clean_architecture_01_3.png" alt=""><figcaption></figcaption></figure>

<figure><img src="https://fistkim101.github.io/images/flutter_clean_architecture_01_4.png" alt=""><figcaption></figcaption></figure>

<figure><img src="https://fistkim101.github.io/images/flutter_clean_architecture_01_5.png" alt=""><figcaption></figcaption></figure>

<figure><img src="https://fistkim101.github.io/images/flutter_clean_architecture_01_6.png" alt=""><figcaption></figcaption></figure>

<figure><img src="https://fistkim101.github.io/images/flutter_clean_architecture_01_7.png" alt=""><figcaption></figcaption></figure>

<figure><img src="https://fistkim101.github.io/images/flutter_clean_architecture_01_8.png" alt=""><figcaption></figcaption></figure>

강의에서 프로젝트를 구현할 때 근간으로 사용하는 설계구조인 4개의 계층이다. 그리고 강의에서 사용되는 서드파티 라이브러리들에 대한 안내이다.



## 개인적으로 찾아 본 Flutter 클린 아키텍처 Layer

강의자료만 봐서는 정확히 이해가 가지 않아서 아래 세 가지 포스팅을 참고 했다.

* [Very good layered architecture in Flutter](https://verygood.ventures/blog/very-good-flutter-architecture)
* [Flutter Clean Architecture Series — Part 1 (UPDATED)](https://devmuaz.medium.com/flutter-clean-architecture-series-part-1-d2d4c2e75c47)
* [Flutter x Clean Architecture](https://itnext.io/flutter-clean-architecture-b53ce9e19d5a)

위의 각각의 포스팅에서 다루고 있는 Layer 구분들을 도식화 한 이미지들은 아래와 같다.  &#x20;

<figure><img src="https://fistkim101.github.io/images/flutter_layer_1.png" alt=""><figcaption></figcaption></figure>

<figure><img src="https://fistkim101.github.io/images/flutter_layer_2.png" alt=""><figcaption></figcaption></figure>

<figure><img src="https://fistkim101.github.io/images/flutter_layer_3.png" alt=""><figcaption></figcaption></figure>

<figure><img src="https://fistkim101.github.io/images/flutter_layer_4.png" alt=""><figcaption></figcaption></figure>

포스팅마다 약간씩 사용되는 용어가 다른 부분이 있긴 했는데 공통적으로 결국 같은 이야기를 하고 있다.

일단 개인적으로는 세 포스팅 중 [Flutter x Clean Architecture](https://itnext.io/flutter-clean-architecture-b53ce9e19d5a) 가 가장 보기에는 좋았던 것 같다. 너무 자세히 또는 너무 단순히 묘사하지 않으면서도 정확하게 각 Layer 의 역할과 각 Layer 에 속하는 것들의 역할들을 설명하고 있어서다.

각 개념에 대해서 나의 말로 풀어서 다시 정리를 해보자.

#### Layer 정의 <a href="#layer" id="layer"></a>

일단 Layer 들의 구분에 대해서 알아보기 전에 Layer 자체에 대한 정의를 해야한다. 포스팅에 이미 정리가 잘 되어 있는데 단순하게 표현하자면 Layer 는 Architecture 를 구성하는 개념이다. 독립되어 있으면서 Architecture 의 구성에 포함되기 위해서 특정한 책임을 지는 개념적인 구분 계층이다.

> **Architecture**\
> Let’s start with the basics: what is app architecture? App architecture is the logical way we organize our projects and how the various components interact with each other to fulfill the business requirements. We want to follow standards and make it easy to identify the components that we’ll need to develop features in our codebase. The way we establish the relationship and interactions between these components can reduce or add complexity to our projects, which has a significant impact on the team’s productivity.

> **Layers**\
> Now that we know what architecture is, let’s define layers. Layers are the components that compose your architecture. We can define these by assigning a specific responsibility to them. We should keep these layers simple, yet isolated enough to achieve a maintainable codebase.



### 각 Layer(Presentation Layer, Domain Layer, Data Layer) 탐구 및 Layer 구성 역할 이해 <a href="#layerpresentation-layer-domain-layer-data-layer-layer" id="layerpresentation-layer-domain-layer-data-layer-layer"></a>

각 구성에 관해서까지 세세하게 도식화가 된 것은 아래 그림이 가장 좋았다.

![](https://fistkim101.github.io/images/flutter\_layer\_4.png)

[Flutter x Clean Architecture](https://itnext.io/flutter-clean-architecture-b53ce9e19d5a) 이 포스팅에 나온 그림인데 아쉬운 점은 Layer 자체만 설명했으면 좋겠는데 일부 개념이 Bloc 에 의존적이다.

## 각 Layer 를 패키지로 알아보기

&#x20;

<figure><img src="https://fistkim101.github.io/images/concept_layers_01.png" alt=""><figcaption></figcaption></figure>

<figure><img src="https://fistkim101.github.io/images/concept_layers_02.png" alt=""><figcaption></figcaption></figure>

기존에 다른 레퍼런스에서 알아봤던 자료들과 비교했을때 Application Layer 부분이 조금 다르다. 일단 강의에서는 저렇게 사용한다. 강의를 모두 끝내면 다 들어가보지 않아도 모두 파악하고 있어야할 내용이다

## 내가 정리한 Layer 별 상세 정리

각 Layer 가 실제로 어떻게 구현되는지에 대해서 학습했다. [소스코드](https://github.com/fistkim101/flutter-clean-architecture-study) 를 첨부한다.

학습한 것을 기반으로 아래 그림을 만들었다. 각 구성과 세부 항목을 내재화 하자. 참고로 강의에서 규정한 Application Layer 는 아래 그림에서 생략했다.

Application Layer 는 다른 Layer 들과 상호작용 한다기 보다는 어플리케이션 전역에 적용되는 자원에 대한 Layer 이다. 백엔드에서 사용했던 AppConfig 패키지와 쓰임과 역할이 완전히 일치한다.

![](https://fistkim101.github.io/images/concept\_3layer.png)

여기서 UI Widget 과 State Management 사이의 상호 작용이 곧 view 와 viewModel 간의 상호작용이며 복습을 위해 아래 그림을 다시 보자.

![](https://fistkim101.github.io/images/concept\_view\_viewmodel.png)

구현 하다가 잠깐씩 막히더라도 학습했던 것을 보고 구현을 하면 그만이다. 달달 외우는게 중요한 것이 아니고 저렇게 계층을 나누는 이유에 대해서 알고, 계층을 나누는 과정에서 어떤 방식으로 처리했는지와 그 과정에서 어떤 라이브러리를 사용했는지 등에 대해서 인지하자.

강의에서는 에러 코드에 대해서 더 잘 처리해 두었는데, 나의 프로젝트에서는 다르게 처리할 가능성이 커서 실습에서는 생략했다.
