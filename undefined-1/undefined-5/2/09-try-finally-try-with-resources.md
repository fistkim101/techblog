# \[09] try-finally 보다는 try-with-resources 를 사용하라

## 핵심 내용

try-with-resources 를 사용하면 자원 반납을 더 좋은 가독성의 코드로 할 수 있다. 뿐만 아니라 에러 추적도 쉽다.



## try-with-resources 쓰면 뭐가 좋은가

try-finally 로도 자원 반납을 할 수는 있지만 명시적으로 선언을 해줘야한다. 하지만 try-with-resources 를 사용하면 자동으로 반납이 가능하다.

뿐만 아니라 두 개 이상의 자원을 하나의 구문 내에서 만들고 반환 시킬 수 있어서 가독성 면에서 더욱 좋다. 책에서 말하고 있는 가장 좋은 점은 에러를 모두 추적할 수 있도록 해준다는 것이다.



## 두 개 이상의 자원을 전통적인 try-finally 로 종료시키는게 뭐가 문제란 말인가?(둘다 종료를 하면 되지 않나?)

안된다. 병렬로 시퀀셜하게 종료를 한다고 하자. A-> B 순으로 finally 에서 종료를 할 때, A 를 close() 하다가 에러가 발생하면 B를 close() 시도조차 하지 않는다. 그래서 이를 중첩 try-finally 구문으로 처리를 해둬야 B 를 close() 시도라도 하는 것이다.



## try-with-resources 는 그럼 어떤 원리로 만들어져 있을까?

[이 블로그](https://truehong.tistory.com/133)에 잘 정리가 되어 있는데, 결국 내부 중첩 try-catch 구조로 되어 있다.











