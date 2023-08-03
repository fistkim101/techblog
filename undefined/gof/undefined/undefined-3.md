# 빌더 패턴

## 대분류

객체 생성



## 문제상황

객체가 가진 프로퍼티의 수가 많은 경우에 객체 초기화 시에 모든 프로퍼티가 다 할당되지 않을 수 있다. 달리 말해서 객체 자체의 복잡도가 높을 수록 해당 객체가 초기화 될때 해당 객체가 가질 수 있는 스냅샷의 경우의 수가 많아진다. 물론, 이건 단순한 경우의 수로 본 경우이지만 보통은 객체가 복잡도가 클 수록 도메인 요구사항 역시 해당 객체의 초기화 케이스가 매우 다양할 가능성이 크다.

결론적으로 이러한 상황으로 인해서 객체가 다양한 생성자를 강제받게 된다거나 기존 생성자의 특정 파라미터에 null을 세팅하는 등 코드의 가독성을 해치는 일이 발생하게 된다. 가독성 뿐만이 아니라 버그 역시 발생시킬 가능성이 있다.(인간의 인지능력 때문에)



## 해결방안

객체가 초기화 되는 케이스가 다양할 수록 도메인 요구사항에 따라 이를 패턴화 해서 생성시킨다. 또한 개별 생성시 어떤 프로퍼티를 세팅했고 어떤 프로퍼티를 세팅하지 않았으며, 세팅한 프로퍼티에는 어떤 값을 넣는지 쉽게 인지할 수 있는 방식으로 코드를 구현한다.

<figure><img src="../../../.gitbook/assets/image (1) (1) (3) (1).png" alt=""><figcaption></figcaption></figure>



## 실습코드



```java
public class TourPlan {

    private String name;
    private int days;
    private Boolean rentBusRequired;
    private Boolean isGuideSupportNeed;
    private List<DayPlan> dayPlans = new ArrayList<>();

    public void setName(String name) {
        this.name = name;
    }

    public void setDays(int days) {
        this.days = days;
    }

    public void setRentBusRequired(Boolean rentBusRequired) {
        this.rentBusRequired = rentBusRequired;
    }

    public void setGuideSupportNeed(Boolean guideSupportNeed) {
        isGuideSupportNeed = guideSupportNeed;
    }

    public void setDayPlans(List<DayPlan> dayPlans) {
        this.dayPlans = dayPlans;
    }

    @Override
    public String toString() {
        return "TourPlan{" +
                "name='" + name + '\'' +
                ", days=" + days +
                ", rentBusRequired=" + rentBusRequired +
                ", isGuideSupportNeed=" + isGuideSupportNeed +
                ", dayPlans's size=" + dayPlans.size() +
                '}';
    }
}

public interface TourPlanBuilder {

    TourPlanBuilder name(String name);

    TourPlanBuilder days(int days);

    TourPlanBuilder rentBusRequired(Boolean rentBusRequired);

    TourPlanBuilder isGuideSupportNeed(Boolean isGuideSupportNeed);

    TourPlanBuilder dayPlans(List<DayPlan> dayPlans);

    TourPlan build();
}


public class DefaultTourPlanBuilder implements TourPlanBuilder {

    private TourPlan tourPlan = new TourPlan();

    @Override
    public TourPlanBuilder name(String name) {
        tourPlan.setName(name);
        return this;
    }

    @Override
    public TourPlanBuilder days(int days) {
        tourPlan.setDays(days);
        return this;
    }

    @Override
    public TourPlanBuilder rentBusRequired(Boolean rentBusRequired) {
        tourPlan.setRentBusRequired(rentBusRequired);
        return this;
    }

    @Override
    public TourPlanBuilder isGuideSupportNeed(Boolean isGuideSupportNeed) {
        tourPlan.setGuideSupportNeed(isGuideSupportNeed);
        return this;
    }

    @Override
    public TourPlanBuilder dayPlans(List<DayPlan> dayPlans) {
        tourPlan.setDayPlans(dayPlans);
        return this;
    }

    @Override
    public TourPlan build() {
        return this.tourPlan;
    }
}


public class Client {
    public static void main(String[] args) {
        final TourPlanBuilder tourPlanBuilder = new DefaultTourPlanBuilder();
        final TourPlan tourPlan = tourPlanBuilder.name("name")
                .days(5)
                .isGuideSupportNeed(false)
                .build();
        System.out.println(tourPlan.toString());
    }
}
```



## 마무리

사실 늘상 사용해오던 빌더 그대로다. 하지만 빌더를 롬복에서 만들어주는 것을 사용했지 저렇게 손으로 구현한 적은 없었다. 자동으로 해주는 것에 대한 이해도를 높이는 가장 좋은 방법은 역시 직접 만들어 보는 방법인 것 같다. NEXTSTEP 에서 강조하듯 바퀴를 만들어 보면 바퀴에 대한 이해도가 올라간다는 것이 정말 맞는 말인 것 같다.

빌더 패턴에서는 메타적으로 왜 이 패턴이 어떤 점에서 좋고, 어떤 상황에서 써야 하는지(=어떤 불편함을 해결해 주는지)를 인지하는 것이 중요한 것 같다.
