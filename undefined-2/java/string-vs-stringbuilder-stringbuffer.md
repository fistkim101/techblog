# String vs StringBuilder, StringBuffer

String 은 불변 객체이다. 즉, 문자열을 조작하면 새로운 객체를 힙에 생성하게 되고 변수는 새로운 위치를 가리키게 된다.



반면에 StringBuffer와 StringBuilder 는 가변 객체로서 문자열을 조작해도 주소값이 변하지 않는다.
