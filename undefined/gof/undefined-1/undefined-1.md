# 브릿지 패턴

## 대분류

구조



## 문제상황

어떠한 추상적인 개념이 존재하고, 요구사항이 변화함에 따라 이 개념이 가지는 속성들이 점점 늘어나게 되면 코드 및 설계의 복잡도는 자연스럽게 올라가기 마련이다. 단순히 이러한 속성들이 늘어나기만 해도 복잡도가 올라가는데 속성들 내부에서도 분류가 생기고 각 분류마다 다른 기능들을 요구받게 되면 이 개념을 구현한 클래스는 슈퍼 클래스가 될 여지마저 생기게 된다.



## 해결방안

이 추상적인 개념이 지니는 속성의 수가 늘어나거나 각 속성이 처리해야할 책임이 다변화 된다고 해도 각 처리를 다른 객체에 분할하여 위임할 수 있는 형태로 간다면 구조적 복잡도를 최소화 할 수 있다.

<figure><img src="../../../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

[이 사이트](https://refactoring.guru/ko/design-patterns/bridge)에도 잘 설명되어 있는데 브릿지 패턴에서 말하는 '브릿지' 는 결코 어떤 구체적인 객체가 아니다. 브릿지 패턴에서 말하는 '브릿지'는 '참조' 그 자체이다. 윗 문단에서 이야기한 추상적인 개념(1)이 가져야할 구체적 특성들(N)을 참조하는 것이다.

위 다이어그램은 강의에 사용된 다이어그램인데 썩 좋지 못하다. 왜냐면 구체적 특성들은 여러가지가 나올 수 있는데 집합 관계를 제대로 나타내고 있지 않다. 아래는 [다른 곳](https://refactoring.guru/ko/design-patterns/bridge)에서 찾은 다이어그램이다.

<figure><img src="../../../.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

브릿지 패턴의 핵심은 '위임' 이다. '추상'적인 것들은 정말 말 그대로 추상적인 메타적인 개념에 대한 선언일 뿐이고 구체적인 책임들은 모두 이를 구성하는 각 객체들에 위임하는 것이 핵심이다.

더 쉽게 설명하자면 브릿지 패턴은 곧 추상화된 최상위 개념이 구성(결합) 되는 여러가지 또 다른 N개의 추상화된 개념을 설계하고. 각각의 N개들을 프로퍼티로 지니도록 하는 것이다. 이렇게 구분되어 인식되는 것들을 쪼개고 각각에 대한 책임을 나눠주면 당연히 느슨한 결합을 달성할 수 있고 더 좋은 설계가 될 수 있다.



## 실습코드

```java
public class Main {
    public static void main(String[] args) {

        // preProcess
        PreProcess externalCallPreProcess = new ExternalCallPreProcess();
        PreProcess normalPreProcess = new NormalPreProcess();

        // mainProcess
        MainProcess normalMainProcess = new NormalMainProcess();

        // postProcess
        PostProcess normalPostProcess = new NormalPostProcess();


        // task
        Task task1 = new TaskImpl(externalCallPreProcess, normalMainProcess, normalPostProcess);
        task1.execute();

        System.out.println("=========================================================");

        Task task2 = new TaskImpl(normalPreProcess, normalMainProcess, normalPostProcess);
        task2.execute();
    }
}

public interface Task {

    void preProcess();

    void mainProcess();

    void postProcess();

    default void execute() {
        preProcess();
        mainProcess();
        postProcess();
    }

}

public class TaskImpl implements Task {
    private final PreProcess preProcess;
    private final MainProcess mainProcess;
    private final PostProcess postProcess;

    public TaskImpl(PreProcess preProcess, MainProcess mainProcess, PostProcess postProcess) {
        this.preProcess = preProcess;
        this.mainProcess = mainProcess;
        this.postProcess = postProcess;
    }

    @Override
    public void preProcess() {
        preProcess.call();
    }

    @Override
    public void mainProcess() {
        mainProcess.call();
    }

    @Override
    public void postProcess() {
        postProcess.call();
    }

}
```

```
ExternalCallPreProcess call
NormalMainProcess call
NormalPostProcess
=========================================================
NormalPreProcess call
NormalMainProcess call
NormalPostProcess

```

배치 작업(task) 을 만든건데 배치 실행 전에 할 일과 배치 실행, 그리고 배치 실행 후에 할 일 세 단계가 있다고 가정했다. 어떤 배치는 배치 실행 전에 외부와 통신을 해야 할 수도 있고 어떤 배치는 하지 않아도 될 수 있고, 이러한 일들을 Task 라는 객체 하나에서 다 분기하여 처리할 것이 아니라 따로 처리를 위임(externalCallPreProcess, normalPreProcess)한 형태이다.

여기서 다른 처리(전, 메인, 후 말고 뭔가 더)가 늘어나거나 각 처리에 필요한 구체적인 처리(전처리로 치면 externalCallPreProcess, normalPreProcess 말고 뭔가 더)가 생겨나도 따로따로 이러한 것들이 개념적으로 분리되어 있고 분리된 객체들이 각자 책임지고 처리하는 구조라서 복잡도가 상대적으로 덜 늘어난다. 이러한 변경 사항들을 Task 라는 객체 하나에서 모두 처리한다고 생각하면 머리가 아파온다.

아무튼 핵심은 이렇게 추상화된 최상위 개념 내에 관심사들을 각각 추상화 시켜서 분류해놓은 뒤 직접적인 처리는 모두 구현체에 위임하는 구조로 설계를 하는 패턴이 브릿지 패턴이다.

다시 강조하지만 여기서 말하는 브릿지는 위 예시코드에서 보자면 TaskImpl 이라는 최상위 객체가 각각의 또다른 구분된 추상에 의존하고 있는 참조 이다.(아래에서 확인할 수 있는 각 프로퍼티에 대한 의존(참조)를 브릿지라고 표현한 것이다.) 당연히 실질적인 처리는 이렇게 각각 구분된 추상을 구현한 구현체들이 처리한다.

그래서 결론적으로 추상이 구체화된 것을 참조하게 되는 것이다.

```java
   private final PreProcess preProcess;
    private final MainProcess mainProcess;
    private final PostProcess postProcess;

    public TaskImpl(PreProcess preProcess, MainProcess mainProcess, PostProcess postProcess) {
        this.preProcess = preProcess;
        this.mainProcess = mainProcess;
        this.postProcess = postProcess;
    }
```
