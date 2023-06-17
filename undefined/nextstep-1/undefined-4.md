# 어플리케이션 진단하기

## 1. 애플리케이션 문제 원인 <a href="#1" id="1"></a>

애플리케이션 레벨에서 어떤 문제가 날 수 있는지, 이에 대한 원인 파악은 어떤 식으로 하는지에 대해 다뤘다. 일반적인 ‘버그’ 수준의 오류는 아예 다루지 않았다. 왜냐하면 그건 인프라 요소도 아니거니와 애초에 프로그램을 잘못짜서 발생하는 에러이기 때문이다. 이번 챕터에서 말하는 애플리케이션에서의 장애란 인프라적인 요소에 한정한다.

강의에서는 주로 애플리케이션 레벨에서의 장애는 [스레드](https://ko.wikipedia.org/wiki/%EC%8A%A4%EB%A0%88%EB%93%9C\_\(%EC%BB%B4%ED%93%A8%ED%8C%85\)) 이슈가 대부분이며 그 유형으로는 아래 세 가지가 있다고 했다.

* BLOCKED 상태인 Thread
* 한 Task가 너무 오래 Thread를 점유하고 있지는 않은지
* CPU 사용률이 너무 높지는 않은지

위 세 유형 모두 결국 [데드락](https://ko.wikipedia.org/wiki/%EA%B5%90%EC%B0%A9\_%EC%83%81%ED%83%9C) 이 유발하는 현상들이다.

\
\


## 2. 애플리케이션 문제 해결 <a href="#2" id="2"></a>

스레드 이슈는 Thread 덤프를 통해 파악한다. Thread 덤프는 아래와 같은 상황일 때 사용한다.

* 사용자 수가 많지도 않은데, CPU 사용량이 떨어지지 않을 때
* 특정 애플리케이션을 수행헀는데, 응답이 없을 때

\


### **Thread 덤프 생성**

```
$ ps -ef | pgrep java
$ jstack [pid] > thread.dump
```

* process 중 커널 프로세스를 제외한(-e) 프로세스들을 풀 포맷으로(-f) 보여주고 그 중 process 가 java인 것을 grep

\


### **Thread 덤프 내용**

아래는 현재 서비스 인스턴스에서 덤프를 떠본 것이다.

```
2021-04-10 10:40:53
Full thread dump OpenJDK 64-Bit Server VM (11.0.10+9 mixed mode, sharing):

Threads class SMR info:
_java_thread_list=0x00007fb45c00d160, length=28, elements={
0x00007fb4a011d000, 0x00007fb4a011f000, 0x00007fb4a0126000, 0x00007fb4a0128000,
0x00007fb4a012a000, 0x00007fb4a012c000, 0x00007fb4a012e000, 0x00007fb4a0163000,
0x00007fb4a0b02800, 0x00007fb4a0af3000, 0x00007fb4a0db4000, 0x00007fb4a0af5000,
0x00007fb4a0901000, 0x00007fb4a1344800, 0x00007fb4a0df5000, 0x00007fb4a0cc6000,
0x00007fb4a12ba000, 0x00007fb4a12bb800, 0x00007fb4a0f40000, 0x00007fb4a0f42800,
0x00007fb4a13fb800, 0x00007fb4a13fd800, 0x00007fb4a13ff800, 0x00007fb4a1170800,
0x00007fb4a11bb000, 0x00007fb4a11be000, 0x00007fb4a0015800, 0x00007fb45c00e000
}

"Reference Handler" #2 daemon prio=10 os_prio=0 cpu=4.94ms elapsed=79597.97s tid=0x00007fb4a011d000 nid=0xd waiting on condition  [0x00007fb4a4501000]
   java.lang.Thread.State: RUNNABLE
        at java.lang.ref.Reference.waitForReferencePendingList(java.base@11.0.10/Native Method)
        at java.lang.ref.Reference.processPendingReferences(java.base@11.0.10/Unknown Source)
        at java.lang.ref.Reference$ReferenceHandler.run(java.base@11.0.10/Unknown Source)

"Finalizer" #3 daemon prio=8 os_prio=0 cpu=1.03ms elapsed=79597.97s tid=0x00007fb4a011f000 nid=0xe in Object.wait()  [0x00007fb4a4400000]
   java.lang.Thread.State: WAITING (on object monitor)
        at java.lang.Object.wait(java.base@11.0.10/Native Method)
        - waiting on <no object reference available>
        at java.lang.ref.ReferenceQueue.remove(java.base@11.0.10/Unknown Source)
        - waiting to re-lock in wait() <0x00000000c29a9958> (a java.lang.ref.ReferenceQueue$Lock)
        at java.lang.ref.ReferenceQueue.remove(java.base@11.0.10/Unknown Source)
        at java.lang.ref.Finalizer$FinalizerThread.run(java.base@11.0.10/Unknown Source)

"Signal Dispatcher" #4 daemon prio=9 os_prio=0 cpu=0.33ms elapsed=79597.97s tid=0x00007fb4a0126000 nid=0xf runnable  [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

"Service Thread" #5 daemon prio=9 os_prio=0 cpu=0.07ms elapsed=79597.97s tid=0x00007fb4a0128000 nid=0x10 runnable  [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

"C2 CompilerThread0" #6 daemon prio=9 os_prio=0 cpu=15538.31ms elapsed=79597.97s tid=0x00007fb4a012a000 nid=0x11 waiting on condition  [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE
   No compile task

"C1 CompilerThread0" #7 daemon prio=9 os_prio=0 cpu=3893.39ms elapsed=79597.97s tid=0x00007fb4a012c000 nid=0x12 waiting on condition  [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE
   No compile task

"Sweeper thread" #8 daemon prio=9 os_prio=0 cpu=6.82ms elapsed=79597.97s tid=0x00007fb4a012e000 nid=0x13 runnable  [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE
```

\


만약 DeadLock이 있다면 아래와 같이 덤프 내용이 나온다고 한다.

```
Found one Java-level deadlock:
=============================
"http-nio-8080-exec-6":
  waiting to lock monitor 0x00007faa62973608 (object 0x00000005c0c4b1e8, a java.lang.Object),
  which is held by "http-nio-8080-exec-5"
"http-nio-8080-exec-5":
  waiting to lock monitor 0x00007faa60025508 (object 0x00000005c0c4b1f8, a java.lang.Object),
  which is held by "http-nio-8080-exec-6"

Java stack information for the threads listed above:
===================================================
"http-nio-8080-exec-6":
	at nextstep.subway.line.ui.LineController.findLockLeft(LineController.java:78)
	- waiting to lock <0x00000005c0c4b1e8> (a java.lang.Object)
```

\


아래는 Thread 스택 정보이다.

```
"http-nio-8080-exec-6" #47 daemon prio=5 os_prio=31 tid=0x00007faa6132c000 nid=0x7f03 waiting for monitor entry [0x0000700009bce000]
   java.lang.Thread.State: BLOCKED (on object monitor)
	at nextstep.subway.line.ui.LineController.findLockLeft(LineController.java:78)
	- waiting to lock <0x00000005c0c4b1e8> (a java.lang.Object)
	- locked <0x00000005c0c4b1f8> (a java.lang.Object)
```

\


## **결론(강의자료 발췌)**

* Thread 이름, 식별자, 우선순위(prio), Thread가 점유하는 메모리 주소를 의미하는 Thread ID(tid), OS에서 관리하는 Thread ID (nid), Thread 상태 (NEW, RUNNABLE, BLOCKED, WAITING, TIMED\_WAITING, TERMINATED) 등의 정보를 확인할 수 있음.
* 이 중 RUNNABLE과 BLOCKED의 경우가 문제가 될 수 있음. RUNNABLE 상태면서 지속시간이 긴 Thread가 없는지, Lock 처리가 제대로 되지 않아 문제가 발생하고 있지는 않은지 확인해봐야한다.

\
