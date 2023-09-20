# StringBuffer vs StringBuilder

StringBuffer 는 append() 시 동기화 로직이 있어서 멀티 스레드 환경에서도 안전하다. 즉, 해당 메소드에 먼저 진입한 스레드가 처리가 끝날 때까지 다른 스레드는 진입할 수 없다. 다만 이 동기화 처리 때문에&#x20;



하지만 StringBuilder 는 동기화 처리가 되어있지 않아서 성능은 StringBuffer 에 비해 빠르지만 멀티 스레드 환경에서는 예상하지 못한 문제가 발생할 수 있다.
