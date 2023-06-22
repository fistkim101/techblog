# (최종 검토 필요) java enum은 메모리에 언제, 어떻게 할당되는가





## 최종 검토 아직 안했음. 아래 과거 필기.

**java의 enum은 메모리에 언제, 어떻게 할당되는가**

생각해보면 new를 통해서 만들어준 적도 없는데 냅다 갖다 쓰기만 했다. 그 때부터 이상하다고 생각하고 찾아볼 생각을 했어야했다. 아무튼, 생성되기 전의 enum은 private static final 로 선언만 된 채로 jvm의 method area(= static)에 상주해 있다가 처음 만나게 되면 heap에 이를 생성해주고 생성된 heap의 주소값을 열거형으로 선언해준 상수들이 할당 받게 된다. 그래서 동일한 enum 타입을 각기 다른 변수에 할당해주고 비교해줘도 같다는 판단을 할 수 있는 것이다.

[공식문서에서도 생성자에 관해서](https://docs.oracle.com/javase/7/docs/api/java/lang/Enum.html) 아래와 같이 서술되어 있다.

> Sole constructor. Programmers cannot invoke this constructor. It is for use by code emitted by the compiler in response to enum type declarations.

**정리하자면 enum에 정의해주는 value들은 enum 을 선언한 곳에서 모두 그 때 생성이 되게 된다. 그래서 value에 파라미터를 설정할 경우 이에 맞게 사용될 생성자 역시 만들어 주어야 하는 것이고, 이 때 만들어준 생성자가 call 되는 것이다.**

아주 간단한 원리인데 이걸 알고 나니까 enum에 대한 활용도가 매우 높아짐을 느낄 수 있었다.

```
package calculator.entity;

import common.Constant;

import java.util.Arrays;
import java.util.function.BiFunction;

public enum OperationType {

    PLUS(Constant.OPERATION_PLUS, (param1, param2) -> param1 + param2),
    MINUS(Constant.OPERATION_MINUS, (param1, param2) -> param1 - param2),
    MULTIPLE(Constant.OPERATION_MULTIPLE, (param1, param2) -> param1 * param2),
    DIVIDE(Constant.OPERATION_DIVIDE, (param1, param2) -> param1 / param2);

    private String operator;
    private BiFunction<Long, Long, Long> formula;

    OperationType(String operator, BiFunction<Long, Long, Long> formula) {
        this.operator = operator;
        this.formula = formula;
    }

    public static Long calculate(String operator, Long parma1, Long param2) {
        return getOperationType(operator)
                .formula
                .apply(parma1, param2);
    }

    private static OperationType getOperationType(String operator) {
        return Arrays.stream(values())
                .filter(value -> value.operator.equals(operator))
                .findFirst()
                .orElseThrow(() -> new IllegalArgumentException(Constant.NOT_EXIST_OPERATOR_TYPE_MESSAGE));
    }

}
```

```
package calculator.entity;

import common.Constant;

import java.util.Arrays;
import java.util.List;
import java.util.Scanner;
import java.util.stream.Collectors;

public class Calculator {

    public String getFormula() {
        Scanner scanner = new Scanner(System.in);
        return scanner.nextLine();
    }

    public Long getResult(String input) {
        Long result = 0L;
        List<String> formula = Arrays.stream(input.split(Constant.SPACE)).collect(Collectors.toList());
        this.addDefaultElement(formula);

        for (int i = 1; i < formula.size(); i += 2) {
            result = OperationType.calculate(formula.get(i), result, Long.parseLong(formula.get(i + 1)));
        }

        return result;
    }

    private void addDefaultElement(List<String> formula) {
        formula.add(0, Constant.FORMULA_FIRST_ELEMENT);
        formula.add(1, Constant.OPERATION_PLUS);
    }

}

```

```
package common;

public class Constant {

    // common
    public static String SPACE = " ";

    // calculator
    public static String FORMULA_FIRST_ELEMENT = "0";
    public static String OPERATION_PLUS = "+";
    public static String OPERATION_MINUS = "-";
    public static String OPERATION_DIVIDE = "/";
    public static String OPERATION_MULTIPLE = "*";

    // error message
    public static String NOT_EXIST_OPERATOR_TYPE_MESSAGE = "연산은 +, -, *, / 중 하나만 가능합니다.";

}
```

\




