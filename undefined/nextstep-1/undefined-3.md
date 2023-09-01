# 서버 진단하기

## 들어가며 <a href="#0" id="0"></a>

장애가 발생하면 자주 발생하는 실수가 ‘피해자’에 대해서 집중하는 것인데, 강사님은 ‘피해자’에 집중하지 말고 원인에 집중하는 것을 강조. 이번 챕터는 이 장애의 원인이 무엇인가를 규명하는데 있어 사용하기 좋은 방법론인 USE 방법론의 틀을 익히고, 세부적으로 어떤 도구와 절차들로 장애의 원인이 무엇인지 규명하면 좋을지를 학습한다.



\


<figure><img src="../../.gitbook/assets/image (5) (2).png" alt=""><figcaption></figcaption></figure>

**현재 미션대로 수행한 서버의 구조는 위와 같이 3 티어 구성**



서버가 위와 같은 구조라면 장애가 날 수 있는 포인트도 웹서버, application host, db 이렇게 세 포인트가 된다. 이게 당연한 이야기라 할지라도 장애가 났을 때 위와 같은 큰 구조 속에서 문제를 찾고자 하는 태도와 막연하게 ‘에러가 뭐야’라고 접근하는 것은 해결 속도에 있어서 차이가 있다고 생각한다.\


## 1. USE 방법론 <a href="#1-use" id="1-use"></a>

<figure><img src="../../.gitbook/assets/image (68).png" alt=""><figcaption></figcaption></figure>

위 그림은 강의 자료에서 사용된 [USE 방법론](http://www.brendangregg.com/usemethod.html) 에 대한 자료이다. 결국 ‘에러에 대해서 어떤 생각의 틀로 접근 해야 가장 빨리 문제의 원인을 찾고 이를 해결할 수 있을까’라는 고민에 대한 대답 중 하나의 형태라고 생각한다.

위 순서도와 같은 순서(에러 -> 사용률 -> 포화도)로 학습하였다.\


### **A. 에러**

에러가 발생했는지 확인하는 가장 직관적이고 빠른 방법은 로그를 확인하는 것이다. 체크해야할 로그는 ‘시스템 로그’, ‘애플리케이션 로그’ 이렇게 두 가지이다.

시스템 로그는 /var/log/syslog 에 위치해 있다. 실무에서 단 한번도 시스템 로그를 본 적이 없었는데, 강의에서 알려준 대로 들어가보니 내가 kill 했던 프로세스 들이나 등록한 rsa등 ‘시스템 로그’ 성격의 모든 기록들이 남아있었다. 그리고 특별히 설정해주지 않아도 기본적으로 log rotate 설정이 걸려있다. 따로 바꾸려면 /etc/logrotate.d/rsyslog 에서 log rotate 설정 변경이 가능하다.

```
$ tail -f /var/log/syslog
```



### **B. 사용률**

<figure><img src="../../.gitbook/assets/image (9) (1) (1).png" alt=""><figcaption></figcaption></figure>



위 명령어들은 사용률에 대해서 각각 확인하고자 하는 항목들에 대해 사용해볼 수 있는 linux 명령어들이다. 강의에서는 개별 명령어들을 쳐보면서 하나하나 항목들을 확인해보았다. 개인적으로 트래픽이 높은 서비스를 운영해본 경험이 없어서 그런지 뭔가 실질적으로 와닿지가 않아서 집중이 잘 되지 않았다. 이건 ‘이런게 있다’정도로 이해해두고 나중에 부하 테스트를 하면서 다시 복습해봐야겠다는 생각이 들었다.



### **C. 네트워크 상태 확인하기(TIME\_WAIT, CLOSE\_WAIT)**

\


<figure><img src="../../.gitbook/assets/image (81).png" alt=""><figcaption></figcaption></figure>

**TIME\_WAIT, CLOSE\_WAIT 상태를 이해하기 위해서는 위 과정에 대해 알고 있어야 한다.**



<figure><img src="../../.gitbook/assets/image (79).png" alt=""><figcaption></figcaption></figure>

```
이 중 ESTABLISHED 이후 종료 과정에서 어플리케이션의 close() 호출 부분을 추가로 표시했습니다.
Active Close 쪽이 먼저 close()를 수행하고 FIN을 보내면 Passive Close 쪽은 ACK을 보낸 후 어플리케이션의 close()를 수행합니다.
보다 상세한 과정은 다음과 같습니다:

1. 먼저 close()를 실행한 클라이언트가 FIN을 보내고 FIN_WAIT1 상태로 대기한다.

2. 서버는 CLOSE_WAIT으로 바꾸고 응답 ACK를 전달한다. 그와 동시에 해당 포트에 연결되어 있는 어플리케이션에게 close()를 요청한다.

3. ACK를 받은 클라이언트는 상태를 FIN_WAIT2로 변경한다.

4. close() 요청을 받은 서버 어플리케이션은 종료 프로세스를 진행하고 FIN을 클라이언트에 보내 LAST_ACK 상태로 바꾼다.

5. FIN을 받은 클라이언트는 ACK를 서버에 다시 전송하고 TIME_WAIT으로 상태를 바꾼다. TIME_WAIT에서 일정 시간이 지나면 CLOSED된다. ACK를 받은 서버도 포트를 CLOSED로 닫는다.

한 가지 주의할 점은 클라이언트와 서버 대신 Active Close와 Passive Close라는 표현을 사용한 것인데,
반드시 서버만 CLOSE_WAIT 상태를 갖는 것은 아니기 때문입니다.
서버가 먼저 종료하겠다고 FIN을 보낼 수 있고, 이런 경우 서버가 FIN_WAIT1 상태가 됩니다.
따라서, 클라이언트와 서버가 아닌 Active Close(또는 Initiator, 기존 클라이언트)와 Passive Close(또는 Receiver, 기존 서버)정도로 표현하는 것이 정확합니다.
```

[카카오](https://tech.kakao.com/2016/04/21/closewait-timewait/) 에서 TIME\_WAIT, CLOSE\_WAIT 상태에 대해 깔끔하게 정리 해두었길래 가져왔다.



#### **TIME\_WAIT**

Active Close 측에 TIME\_WAIT 상태를 의도적으로 두는 이유는 [카카오](https://tech.kakao.com/2016/04/21/closewait-timewait/) 에 잘 설명되어 있는데 정리하자면 아래와 같다.

* TIME\_WAIT 없이 즉시 연결을 종료하고 바로 다음 연결을 맺게 될 경우 이 SYN 패킷이 Passive Close의 ACK보다 먼저 도착할 경우 Sequence ID가 꼬이게 된다
* 그림에도 나와 있듯이 Active Close의 마지막 ACK 패킷이 손실 되면 상대는 LAST\_ACK 상태에 빠지게 되고 새로운 SYN 패킷 전달시 RST를 리턴하여 오류가 발생한다

서버 입장에서는 소켓을 생성할 수 있는 갯수가 제한되어 있는데 TIME\_WAIT이 무한정 늘어나면 포트고갈이 발생할 수 있어서 문제가 생길 수 있다. 따라서 웹 성능을 개선하기 위해 keepalive, connection pool 등을 통해 연결을 재사용한다.



#### **CLOSE\_WAIT**

네트워크 진단시 CLOSE\_WAIT 패킷이 많다는 것은 서버 측에서 능동적으로 연결을 끊지 못하고 있다는 의미이다. 이 말인즉, 다른 여러 프로세스를 진행하다 보니 이 연결을 끊지 못하고 있다는 이야기인 것이고 결국 이는 부하(포화도 과다)가 몰려 있어서 해야할 일을 못하고 있는 것. CLOSE\_WAIT 패킷이 많다는 것은 보통 부하가 많이 몰려있는 경우가 많다고 함.

아래 강의 내용 그대로 발췌

> 따라서 병목, 서버 멈춤 현상 등으로 인해 정상적으로 close하지 못할 경우, CLOSE\_WAIT 상태로 대기합니다. 커널 옵션으로 타임아웃 조절이 가능한 FIN\_WAIT이나 재사용이 가능한 TIME\_WAIT과 달리, CLOSE\_WAIT는 포트를 잡고 있는 프로세스의 종료 또는 네트워크 재시작 외에는 제거할 방법이 없습니다. 따라서 평소에 서비스의 부하를 낮은 상태로 유지해야 합니다.

\


### **D. 포화도**

포화도 강의 초반에 강사님이 진짜 귀에 딱 들어오는 말씀을 하셔서 정리한다. CPU 사용률이 100% 에 육박하면 사실 이건 장애가 아니라 자원을 최적으로 다 끌어다 ‘잘’ 사용하고 있는 것이라 할 수 있다. 하지만, 이게 문제가 되는 이유는 결국 일꾼들이 다 일하고 있어서 ‘대기’ 가 발생할 수 있기 때문에 결국 CPU 사용률이 100% 라면 손을 봐야하는 상황인 것. 핵심은 CPU 사용률이 100% 라는 것 자체는 장애라 할 수 없고, 대기가 발생할 수 있는 상황이기 때문에 보완해야하는 상황인 것으로 정리된다.

이러한 상황에서 체크해봐야할 항목들과 방법들은 아래와 같다. (강의 자료 발췌)



```
$ vmstat 5 5
procs -----------memory---------- ---swap-- -----io---- -system-- ------cpu-----
 r  b   swpd   free   buff  cache   si   so    bi    bo   in   cs us sy id wa st
 1  0      0 20728432 125420 10700972    0    0     0     9    3    3  0  0 100  0  0
 0  0      0 20726004 125420 10700992    0    0     0     0 1061  971  0  0 99  0  0
```

* r : CPU에서 동작중인 프로세스 수를 의미합니다. CPU 자원이 포화상태인지 파악할 때 활용합니다. r 값이 CPU 값보다 클 경우엔 처리를 하지 못해 대기하는 프로세스가 발생합니다.



```
$ free -wh
              total        used        free      shared     buffers       cache   available
Mem:           31Gi       1.3Gi        19Gi       0.0Ki       122Mi        10Gi        29Gi
Swap:            0B          0B          0B
```

* buffers는 Block I/O의 buffer 캐시 사용량을 의미하며, cache 는 파일시스템에서 사용되는 page cache량을 의미합니다. 따라서 이 값이 0일 꼉우 Disk I/O가 높다는 것을 의미하므로 원인을 파악해보아야 합니다.



```
$ iostat -xt
Linux 5.4.0-1038-aws (ip-192-168-0-146.ap-northeast-2.compute.internal) 	03/19/21 	_x86_64_	(8 CPU)

03/19/21 15:38:32
avg-cpu:  %user   %nice %system %iowait  %steal   %idle
           0.15    0.00    0.08    0.00    0.04   99.73

Device            r/s     rkB/s   rrqm/s  %rrqm r_await rareq-sz     w/s     wkB/s   wrqm/s  %wrqm w_await wareq-sz     d/s     dkB/s   drqm/s  %drqm d_await dareq-sz  aqu-sz  %util
loop0            0.01      0.01     0.00   0.00    0.30     1.04    0.00      0.00     0.00   0.00    0.00     0.00    0.00      0.00     0.00   0.00    0.00     0.00    0.00   0.00
loop1            0.00      0.00     0.00   0.00    2.27     1.25    0.00      0.00     0.00   0.00    0.00     0.00    0.00      0.00     0.00   0.00    0.00     0.00    0.00   0.00
```

* await : I/O 처리 평균시간을 의미하며, Application이 이 시간동안 대기하게 됩니다. 보통 하드웨어 상에 문제가 있거나 디스크를 모두 사용하고 있을 경우에 이슈가 발생합니다.

