# 도커 컨테이너 이해하기

## 1. 도커 컨테이너를 사용하는 이유 <a href="#1" id="1"></a>

도커를 공부하면 제일 처음 접하게 되는 것이 ‘왜 도커를 쓰면 좋은지’ 이다. 그리고 도커의 장점을 설명하다가 나오는 개념이 바로 [snow flake](https://martinfowler.com/bliki/SnowflakeServer.html) 이다. 쉽게 말해서 겉으로 보기엔 비슷한데 자세히 보면 각자가 다 다르다는 의미이다. 이를 서버에 적용해보면 각각의 서버가 여러 이유에 의해서 환경들이 제각각으로 세팅 되어서 일괄적으로 스케일업 하기도 힘들고, 이슈가 발생해도 이를 빠르게 일괄적으로 대응하기가 어렵다는 단점이 있다.

이를 해결하는 가장 좋은 방법은 완전히 동일한 서버 환경을 각각의 인스턴스에 갖추는 것이다. 물론 이렇게 하고자 의도하고 서버를 갖췄으니 그나마 겉으로는 다 같은 눈송이로 보이는 것이겠지만, 그럼에도 각 인스턴스의 환경(OS, 타 프로그램과의 충돌 등)에 의해 의도대로 ‘완전히 동일한 서버 환경’을 갖추는 것은 힘든 점이 많다.

이런 맥락에서 결국 해결책은 특정 환경에 종속되지 않은 상태로 어플리케이션을 구동하는 것이다. 특정 환경에 종속되지 않고, 기존의 가상화 방식이 야기하는 오버헤드를 피하도록 해주는 것이 바로 컨테이너 이다. 컨테이너는 아래와 같은 것들은 가능케 한다.(학습자료 발췌)

* [chroot](https://ko.wikipedia.org/wiki/Chroot) 로 특정 자원만 사용하도록 제한
* [cgroup](https://ko.wikipedia.org/wiki/Cgroups) 을 사용하여 자원의 사용량을 제한
* namespace로 특정 유저만 자원을 볼 수 있도록 제한
* [overlay network](https://ko.wikipedia.org/wiki/%EC%98%A4%EB%B2%84%EB%A0%88%EC%9D%B4\_%EB%84%A4%ED%8A%B8%EC%9B%8C%ED%81%AC) 등 네트워크 가상화 기술 활용
* union file system (AUFS, overlay2)로 이식성, 비용절감

개인적으로 가장 뚜렷한 특장점은 다양한 OS환경에 구애받지 않고 배포가 가능하다는 이식성이 좋다는 것과 자원을 효율적으로 사용하도록 한다는 것이라 생각한다. 결국 궁극적인 목표는 immutable server를 만드는 것이다. (공부를 하다가 정리가 도커의 필요성에 대해 정리가 정말 잘 된 [블로그](https://www.44bits.io/ko/post/why-should-i-use-docker-container) 를 발견했다)

\


## 2. 도커 원리 <a href="#2" id="2"></a>

![](https://fistkim101.github.io/images/concept-docker1.png)

\


### **도커 데몬**

강의에서는 VM의 관점에서 도커를 바라보지 말고 프로세스를 추상화 했다는 관점으로 바라보길 주문하고 있었다. 그게 이해가 더 쉬울거라는 이유에서. 아무튼, 도커 [데몬](https://ko.wikipedia.org/wiki/%EB%8D%B0%EB%AA%AC\_\(%EC%BB%B4%ED%93%A8%ED%8C%85\)) 이 소켓을 열어두고 명령어를 받아서 파이프라인을 통해 해당 명령어를 보내서 적절한 작업을 수행한다. 컨테이너를 만든다던가 하는.

‘[lsof](https://ko.wikipedia.org/wiki/Lsof) -c docker’ 를 통해서 docker 명령어를 참조하고 있는 프로세스 리스트를 보면 아래와 같이 socket이 열려 있는 것을 확인할 수 있다.

![](https://fistkim101.github.io/images/concept-docker2.png)

\


### **컨테이너**

#### 1) 모든 컨테이너는 결국 호스트의 [커널](https://ko.wikipedia.org/wiki/%EC%BB%A4%EB%84%90\_\(%EC%BB%B4%ED%93%A8%ED%8C%85\)) 을 공유한다([uname](https://ko.wikipedia.org/wiki/Uname) 명령어로 확인)

```
$ docker run -it ubuntu:latest bash
root@03d7ffae4c3c:/# uname -a
Linux 03d7ffae4c3c 5.3.0-1032-aws #34~18.04.2-Ubuntu SMP Fri Jul 24 10:06:28 UTC 2020 x86_64 x86_64 x86_64 GNU/Linux

$ docker run -it centos:latest bash
[root@6ec49524540c /]# uname -a
Linux 6ec49524540c 5.3.0-1032-aws #34~18.04.2-Ubuntu SMP Fri Jul 24 10:06:28 UTC 2020 x86_64 x86_64 x86_64 GNU/Linux

# 맥OS는 리눅스가 아니므로 네이티브하게 도커를 사용할 수 없어, linuxkit 기반 경량 가상머신 위에서 도커를 실행합니다.
# 이후의 실습은 리눅스에서 실행하길 권장합니다.
❯ docker run -it centos:latest bash
[root@c3fdb0cda9ac /]# uname -a
Linux c3fdb0cda9ac 4.19.76-linuxkit #1 SMP Tue May 26 11:42:35 UTC 2020 x86_64 x86_64 x86_64 GNU/Linux
```

\


#### 2) 컨테이너는 이미지에 따라서 실행되는 환경(파일 시스템)이 달라진다. ![](https://fistkim101.github.io/images/concept-docker3.png)

컨테이너 별로 각각 다른 파일 시스템을 가질 수 있는 이유는 [chroot](https://ko.wikipedia.org/wiki/Chroot) 를 활용하여 이미지(파일의 집합)를 루트 파일 시스템으로 강제로 인식시켜 프로세스를 실행하기 때문이다.

> 리눅스 종류, 버전 확인 : [/etc/\*-release](https://zetawiki.com/wiki/%EB%A6%AC%EB%88%85%EC%8A%A4\_%EC%A2%85%EB%A5%98\_%ED%99%95%EC%9D%B8,\_%EB%A6%AC%EB%88%85%EC%8A%A4\_%EB%B2%84%EC%A0%84\_%ED%99%95%EC%9D%B8)

```
$ docker run -it ubuntu:latest bash
root@03d7ffae4c3c:/# cat /etc/*-release
DISTRIB_ID=Ubuntu

$ docker run -it centos:latest bash
[root@6ec49524540c /]# cat /etc/*-release
CentOS Linux release 8.2.2004 (Core)
```

\


#### 3) 컨테이너도 결국 [프로세스](https://brainbackdoor.tistory.com/27)

```
## 우선, 터미널의 PID를 확인해봅니다.
$ echo $$
3441
$ sudo ps -ef | grep 3441
ubuntu    3441  3440  0 05:46 pts/2    00:00:00 -bash

## 그럼, PID 1의 Command를 확인해봅니다.
$ cat /proc/1/comm
systemd
$ pstree -p
systemd(1)─┬─accounts-daemon(849)─┬─{accounts-daemon}(859)
...

## 이제 nginx 컨테이너를 실행해봅니다.
$ docker run --name nginx -d -p 80:80 nginx:latest
$ docker exec -it nginx bash
root@acaf67af191e:/# echo $$
324

## nginx 컨테이너의 PID 1 의 Command는 nginx임을 확인할 수 있습니다.
$ sudo docker exec -it acaf67af191e cat /proc/1/comm
nginx
root@acaf67af191e:/# apt-get update
root@acaf67af191e:/# apt-get install psmisc
root@acaf67af191e:/# pstree -p
nginx(1)--nginx(28)

## 하지만 사실은 PID 4453 프로세스임을 확인할 수 있습니다.
## 서버에선 그저 nginx 프로세스가 하나 띄워져있다고 생각합니다.
$ cat /var/run/docker/runtime-runc/moby/acaf67af191e5165b98b427b851f9b29a1450d20abd4e2e5287463ccc02c24a0/state.json | jq '.init_process_pid'
4453

$ cat /var/run/docker/runtime-runc/moby/acaf67af191e5165b98b427b851f9b29a1450d20abd4e2e5287463ccc02c24a0/state.json | jq '.namespace_paths'
{
  "NEWCGROUP": "/proc/4453/ns/cgroup",
  "NEWIPC": "/proc/4453/ns/ipc",
  "NEWNET": "/proc/4453/ns/net",
  "NEWNS": "/proc/4453/ns/mnt",
  "NEWPID": "/proc/4453/ns/pid",
  "NEWUSER": "/proc/4453/ns/user",
  "NEWUTS": "/proc/4453/ns/uts"
}

$ ls -Al /proc/1/ns | awk '{ print $9 $10 $11 }'
$ ps -ef | grep 4453
$ ps -auxfg
```

\


#### **도커 네트워크 구조**

![](https://fistkim101.github.io/images/concept-docker4.png)

docker0는 스위치와 같은 역할을 한다

* veth interface : 랜카드에 연결된 실제 네트워크 인터페이스가 아닌, 가상으로 생성한 네트워크 인터페이스. 일반적인 네트워크 인터페이스와는 달리 패킷을 전달받으면, 자신에게 연결된 다른 네트워크 인터페이스로 패킷을 보내주기 때문에 항상 쌍(pair)으로 생성해야 한다.
* NET namespace : 리눅스 격리 기술인 namespace 중 네트워크와 관련된 부분을 말한다. 네트워크 인터페이스를 각각 다른 namespace에 할당함으로써 서로가 서로를 모르게끔 설정할 수 있다.\


[컨테이너 생성시 발생하는 일들](https://joont92.github.io/docker/network-%EA%B5%AC%EC%A1%B0/)

1\) 생성되는 도커 컨테이너는 namespace 로 격리되고, 그 상태에서 통신을 위한 네트워크 인터페이스(eth0)를 할당받는다\
2\) host PC의 veth interface 가 생성되고 도커 컨테이너 내의 eth0 과 연결한다(컨테이너의 네트워크 격리를 달성하기 위해 선택한 방법인 것 같다)\
3\) host PC의 veth interface 는 docker0 이라는 다른 veth interface 와 연결된다\
4\) 이 과정이 컨테이너가 추가될 때 마다 반복된다

```
## 도커를 생성하면 3가지 형태의 network가 생김을 확인할 수 있습니다.
$ sudo docker network ls
NETWORK ID          NAME                DRIVER              SCOPE
6b6ce553a425        bridge              bridge              local
81a18bc9cc40        host                host                local
576b0223f9cf        none                null                local

## bridge 네트워크를 확인해보면 172.17.0.0/16 대역을 할당했음을 확인할 수 있습니다.
$ docker network inspect bridge
[
    {
        "Name": "bridge",
        "Id": "6b6ce553a425c9392c5a65b8dcd2a57e1665289354b97f430758b745b1dc86a7",
        ...
                    "Subnet": "172.17.0.0/16",
                    "Gateway": "172.17.0.1"

## 그리고 172.17.0.0/16 대역은 docker0로 매핑되어 있습니다.
$ ip route
default via 192.168.0.193 dev eth0 proto dhcp src 192.168.0.207 metric 100
172.17.0.0/16 dev docker0 proto kernel scope link src 172.17.0.1

## docker0는 veth interface와 매핑된 브릿지임을 확인할 수 있습니다.
$ brctl show docker0
bridge name	bridge id		STP enabled	interfaces
docker0		8000.024238d4b0f5	no		vethc8e309f

$ ip link ls
3: docker0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP mode DEFAULT group default
66: vethc8e309f@if65: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master docker0 state UP mode DEFAULT group default
```



> docker-compose 로 띄우면 다른 네트워크 대역을 가진다. docker-compose 로 컨테이너를 띄우면 compose 로 묶은 범위에 맞춰 브릿지를 하나 더 생성하기 때문이다. 따라서 서로 경유하는 브릿지가 다르므로 docker-compose로 띄운 컨테이너와 일반 컨테이너간의 통신은 불가능하다.

```
$ git clone https://github.com/woowacourse/service-practice.git
$ cd lb
$ docker build -t node-server .
$ docker-compose -p practice up -d

$ sudo docker container inspect practice_lb_1 | jq '.[].NetworkSettings.Networks.practice_default.Gateway'
"172.18.0.1"

$ ip route
172.18.0.0/16 dev br-754310d33f5c proto kernel scope link src 172.18.0.1

$ ip link ls
4: br-754310d33f5c: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN mode DEFAULT group default
   link/ether 02:42:cd:76:d1:f1 brd ff:ff:ff:ff:ff:ff
```



> 기본적으로 컨테이너는 외부와 통신이 불가능하다. 포트포워딩을 설정하여 외부에 컨테이너를 공개할 수 있다.

```
# 포트포워딩 설정과 함께 컨테이너를 생성합니다.
$ docker container run -d -p 8081:80 nginx
16cd67c48e5721a6b666192b8960875c720168bf6c5e3ed2138fb04c492447c6

$ sudo docker ps -a
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                  NAMES
16cd67c48e57        nginx               "/docker-entrypoint.…"   4 seconds ago       Up 3 seconds        0.0.0.0:8081->80/tcp   trusting_bhabha

# Host의 8081포트가 listen 임을 확인합니다.
$ sudo netstat -nlp | grep 8081
tcp6       0      0 :::8081                 :::*                    LISTEN      10009/docker-proxy

# docker-proxy 라는 프로세스가 해당 포트를 listen 하고 있음을 볼 수 있습니다.
# docker-proxy는 들어온 요청을 해당하는 컨테이너로 넘기는 역할만을 수행하는 프로세스입니다.
# 컨테이너에 포트포워딩이나 expose를 설정했을 경우 같이 생성됩니다.

$ iptables -t nat -L -n
Chain DOCKER (2 references)
target     prot opt source               destination
RETURN     all  --  0.0.0.0/0            0.0.0.0/0
RETURN     all  --  0.0.0.0/0            0.0.0.0/0
RETURN     all  --  0.0.0.0/0            0.0.0.0/0
RETURN     all  --  0.0.0.0/0            0.0.0.0/0
DNAT       tcp  --  0.0.0.0/0            0.0.0.0/0            tcp dpt:8081 to:172.17.0.2:80

# 보시다시피 모든 요청을 DOCKER Chain 으로 넘기고, DOCKER Chain 에서는 DNAT를 통해 포트포워딩을 해주고 있음을 볼 수 있습니다.
# 이 iptables 룰은 docker daemon이 자동으로 설정합니다.
```

\


## 3. 도커 볼륨 <a href="#3" id="3"></a>

> 예전에 도커 인프런 강의에서 이미 자세히 다뤘던 부분이라서 강의 자료만 그대로 옮긴다. 다음 번에 해당 이미지를 사용하면서 연동시켜 사용해야할 데이터가 있으면 이를 어떻게 보관하고 재사용 할 수 있을 것인가에 대한 해답이다.

도커 이미지로 컨테이너를 생성하면 이미지는 읽기 전용이 되며(immutable server가 된다는 뜻), 컨테이너의 변경 사항만 별도로 저장해서 각 컨테이너의 정보를 보존한다.

하지만 mysql과 같이 컨테이너 계층에 저장돼 있던 데이터베이스의 정보를 삭제해서는 안되는 경우도 있다. 이런 경우를 대비하여 컨테이너 데이터를 영속적인 데이터로 활용하기 위한 방법으로 볼륨을 활용하는 방안이 있다.

\-v 옵션은 호스트의 디렉터리를 컨테이너의 디렉터리에 마운트한다. 따라서 컨테이너의 해당 경로에 파일이 있었다면 호스트의 볼륨으로 덮어씌워진다.

```
$ docker run -d \
--name wordpressdb_hostvolume \
-e MYSQL_ROOT_PASSWORD=password \
-e MYSQL_DATABAS=wordpress \
-v /home/wordpress_db:/var/lib/mysql \
mysql:5.7

$ docker run -d \
--name wordpress_hostvolume \
-e MYSQL_ROOT_PASSWORD=password \
--link wordpressdb_hostvolume:mysql \
-p 80 \
wordpress

$ ls /home/wordpress_db
$ docker stop wordpressdb_hostvolume wordpress_hostvolume
$ docker rm wordpressdb_hostvolume wordpress_hostvolume

```



(이 두 문장이 포인트인 것 같다)

> 이처럼 컨테이너가 아닌 외부에 데이터를 저장하고 컨테이너는 그 데이터로 동작하도록 설계(stateless)하여야 한다.\
> 컨테이너 자체는 상태가 없고 상태를 결정하는 데이터는 외부로부터 제공받도록 구성해야 한다.

\
