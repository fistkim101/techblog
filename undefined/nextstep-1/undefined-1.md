# 통신 확인하기

## IP check <a href="#1-ip-check" id="1-ip-check"></a>

Ping 을 사용하면 IP 정보만으로 서버에 요청이 가능한지 확인할 수 있다. [ICMP](https://ko.wikipedia.org/wiki/%EC%9D%B8%ED%84%B0%EB%84%B7\_%EC%A0%9C%EC%96%B4\_%EB%A9%94%EC%8B%9C%EC%A7%80\_%ED%94%84%EB%A1%9C%ED%86%A0%EC%BD%9C) 프로토콜을 따르기 때문에 포트를 사용하지 않는다. OSI 7 Layer 에서는 Networ Layer이며 TCP/IP Protocol 에서는 Internet Layer에 해당하기 때문이다.

## Ping check

> **IP 정보만으로 서버에 요청이 가능한지를 확인**합니다. Ping은 ICMP란 프로토콜을 사용합니다. ICMP란, IP가 신뢰성을 보장하지 않아 네트워크 장애나 중계 라우터 등의 에러에 대처할 수 없는데, 오류정보 발견 및 보고 기능을 담당하는 프로토콜입니다. (TCP가 아니라 Port 번호가 없어요)

```sh
$ ping [대상 IP]
```

```sh
$ ping google.com -c 1
PING google.com (216.58.220.110): 56 data bytes
64 bytes from 216.58.220.110: icmp_seq=0 ttl=115 time=38.167 ms

-- google.com ping statistics --
1 packets transmitted, 1 packets received, 0.0% packet loss
round-trip min/avg/max/stddev = 38.167/38.167/38.167/0.000 ms
```

* RTT(Round Trip Time)란, 한 패킷이 왕복한 시간을 의미합니다. 네트워크 시간은 연결 시간, 요청 시간, 응답 시간 등으로 구성됩니다. RTT가 높을 경우 어느 구간에서 오래 걸리는지 확인하여야 합니다.

```sh
$ traceroute google.com
traceroute to google.com (172.217.26.46), 30 hops max, 60 byte packets
 1  ec2-52-79-0-132.ap-northeast-2.compute.amazonaws.com (52.79.0.132)  0.807 ms  0.748 ms ec2-52-79-0-94.ap-northeast-2.compute.amazonaws.com (52.79.0.94)  4.504 ms
 2  100.66.8.102 (100.66.8.102)  2.706 ms 100.66.8.88 (100.66.8.88)  6.450 ms *
```

> ⁉️ IP 정보로 통신을 할 때, 실제 서버의 위치는 어떻게 알 수 있을까요?\
> **ARP(Address Resolution Protocol)** 란, 논리적 주소인 IP주소 정보를 이용하여 물리적 주소인 MAC 주소를 알아와 통신이 가능하게 도와주는 프로토콜입니다. ARP Request를 Braodcast로 요청하면 수신한 장비들 중 자신의 IP에 해당하는 장비가 응답을 합니다. 응답받은 NIC 포트 정보와 IP, MAC 주소를 기반으로 이후 통신을 진행합니다.

```sh
# ARP Table 확인
## Ping으로 다른 서버와 통신한 이후 arp table을 다시 확인해보면 ARP Table이 추가됨을 확인할 수 있습니다.
$ arp
Address             HWtype   HWaddress           Flags Mask            Iface
[디폴트 Route IP]     ether    02:ef:51:c6:c4:28   C                     eth0

$ ip route
default via [디폴트 Route IP] dev eth0 proto dhcp src [자신의 IP] metric 100

## ARP 변경
## Default Route의 MAC주소를 변경할 경우 연결이 끊어짐을 확인할 수 있습니다.
$ sudo arp -s 192.168.0.1 [다른 MAC주소]
```

## Port check <a href="#2-port-check" id="2-port-check"></a>

IP, Port를 이용해서 서비스의 정상 구동 여부를 확인할 수 있다. default로 23번 포트를 사용한다.

```
$ telnet localhost 4000
```

> 서버는 서비스 하나에 포트번호 하나를 사용하는데, 하나의 포트번호를 사용하지만 동시에 많은 사용자와 연결을 맺을 수 있다. 그 이유는 Socket Descriptors 덕분인데, 동일 프로그램 내에서 순서대로 중복되지 않게 소켓번호를 지정해주고 이를 기준으로 통신하기 때문에 포트 하나에 많은 연결이 가능해진다. 자세한 내용은 [여기](http://jkkang.net/unix/netprg/chap2/net2\_1.html) 에 있다.

## **Port Fowarding**

서비스를 운영하기 위해서는 결국 특정 포트를 사용해야하는데, 여러가지 이유로 다른 포트로 요청이 올 수 있고 이를 내가 의도하는 포트로 받을 수 있도록 해야하는 경우가 있다. 이때 사용하는 것이 포트포워딩이다.

```
## 원격지 서버에서 8080 포트로 소켓을 열어봅니다.
$ sudo socket -s 8080

## iptables 를 활용하여 port forwarding 설정을 합니다.
## 아래의 설정은 80번 포트로 서버에 요청을 하면 서버의 8080번 포트와 연결해준다는 내용을 담고 있어요
$ sudo iptables -t nat -A PREROUTING -p tcp --dport 80 -j REDIRECT --to-port 8080

## 서버의 공인 IP 확인해봅니다.(해당 명령어를 보내는 호스트의 IP가 반환됨)
$ curl wgetip.com

## 자신의 로컬에서 연결해봅니다.
$ telnet [서버의 공인 IP] 80

## 설정 삭제
$ sudo iptables -t nat -L --line-numbers
$ sudo iptables -t nat -D PREROUTING [라인 넘버]
```

## HTTP response check <a href="#3-http-response-check" id="3-http-response-check"></a>

사실 이 단계에서는 통신을 확인한다는 의미 보다는 통신이 ‘가능’한 것이 확인된 이후부터, 상세하게 요청에 따라 어떤 응답을 서버가 주는지에 대한 상세한 확인을 하는 단계이다.

```sh
$ curl -I google.com
```

강의에서 되게 좋았던 부분은, 실무에서 가끔 ‘응답코드를 어떻게 설정할까?’ 고민을 했던 순간마다 정답이 없어서 고민만 길어지는 시간이 많았는데 강사님이 여러 다양한 서비스들을 참고한다는 이야길 해준 것이었다. 서비스 별로 사용하는 응답코드가 다르기도 하고 ([예시](https://gist.github.com/vkostyukov/32c84c0c01789425c29a)) 언뜻 보기에 동일한 상황인 것 같아도 세밀하게 따져보면 여러 응답코드가 후보로 꼽힐 수 있어서 ([예시](https://brainbackdoor.tistory.com/137)) 응답코드는 신중하게 골라야 한다.

\
