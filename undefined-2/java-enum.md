# java enum은 메모리에 언제, 어떻게 할당되는가

생각해보면 new를 통해서 만들어준 적도 없는데 냅다 갖다 쓰기만 했다. 그 때부터 이상하다고 생각하고 찾아볼 생각을 했어야했다.

**enum 의 상수들은 static final 로서 해당 상수들은 클래스 로딩 시점에 생성되고 초기화 된다. 즉, 클래스 로딩 시점에 메모리 상의 method area 에 위치하며 싱글톤처럼 사용된다.**

[공식문서에서도 생성자에 관해서](https://docs.oracle.com/javase/7/docs/api/java/lang/Enum.html) 아래와 같이 서술되어 있다.

> Sole constructor. Programmers cannot invoke this constructor. It is for use by code emitted by the compiler in response to enum type declarations.



아래 코드는 NEXTSTEP 의 클린코드 미션 수행중 작성한 코드이며 enum 을 적극 활용한 케이스라서 가져왔다.

```java
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

```java
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

```java
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

