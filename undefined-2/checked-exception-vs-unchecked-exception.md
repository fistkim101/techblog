# Checked Exception vs UnChecked Exception

둘의 차이를 간단히 말하자면 '명시적으로 에러 처리(check)를 강요(안하면 컴파일에서부터 에러가 난다)' 당하면 checked exception 이고 그렇지 않다면 unchecked exception 라고 할 수 있다.

<figure><img src="../.gitbook/assets/image (35).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/image (10).png" alt=""><figcaption></figcaption></figure>

주의할 것은 runtime exception 의 상속 여부만으로 이를 구분지으면 error 가 포함이 되지 않는다는 것이다. 따라서 명시적으로 에러처리를 해주지 않으면 컴파일 에러가 나는 것이 checked exception 이라고 할 수 있으며, 반대로 명시적으로 에러처리를 해주지 않아도 컴파일 에러가 나지 않는 것이 unchecked exception 라고 할 수 있다. 대표적으로 runtime exception 을 상속하는 모든 exception 들이 unchecked exception 이라고 할 수 있다. 각각 알아보자.

checked exception 은 반드시 명시적으로 check 가 필요한 exception 이다. 그렇기에 컴파일 시점부터 특정 코드가 checked exception 을 유발할 가능성이 있고, 명시적으로 이를 처리하지 않았다면 컴파일 에러가 나게 된다. 그래서 명시적으로 에러를 처리하게 되고 이는 곧 checked 된 exception 이므로 checked 라는 과거형이 나오지 않았나 싶다. 이 에러는 예외가 발생했을 시 기본으로 롤백이 발생되지 않는다.

unchecked exception 도 같은 맥락에서 풀어보면 check 가 되지 않은 exception 이다. 즉, 명시적으로 처리하지 않아도 컴파일에 문제가 없고 run 에 문제가 없기 때문에 check 가 되지 않은 것이다. 기본적으로 롤백을 발생시킨다.
