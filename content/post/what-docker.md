---
title: "Docker란 무엇일까?"
date: 2023-08-14T11:27:47+09:00
draft: false
pin: false 
summary: "Docker 파헤치기"
tags: [docker]
---

# 도커를 알아보자

> ## 왜 Docker?

우선 Docker는 프로세스를 격리하기 위한 소프트웨어입니다. 

도커 이전에는 어떻게 프로세스를 격리했을까? 바로 VM을 통해

![](https://cdn.ttgtmedia.com/rms/onlineimages/virtual_machines-h_half_column_mobile.png)

위와같은 방법으로 프로세스를 격리했습니다. 하지만 위 이미지에서 보는 바와같이 VM은 

어플리케이션 - Guest OS - Hypervisor - Host OS - 물리머신

단계를 거치면서 동작하기 떄문에 무겁고 성능문제가 발생합니다.

그래서 하이퍼바이저 없이 프로세스를 격리할수있는 방법 Docker를 사용하기시작했습니다.

> ## Chroot 

- 시스템의 root를 임의로 구성한 위치로 binding하는 명령어

chroot로 루트를 변경한다는 게 어떤 의미인지 알아보겠습니다. chroot는 `Change Root Directory`의 줄임말로 명령어 자체가 루트 디렉터리를 바꾼다는 뜻을 가지고 있습니다. 시스템의 일반적인 파일 구조를 생각해보겠습니다.

![](https://d2uleea4buiacg.cloudfront.net/files/00f/00f9ffd1238f179b700c99745803fb4a51571bff0b7295f12b838560d69bf17f.m.png)

시스템에는 `/`라고 일컬어지는 루트 디렉터리가 존재합니다.* 이 루트 디렉터리는 파일 시스템의 최상위를 의미하는 특별한 위치이며, 모든 디렉터리와 파일은 이 루트 디렉터리 아래에 존재합니다.

![](https://d2uleea4buiacg.cloudfront.net/files/e93/e93a9e5defffdbde2cdb83d2e75787834357c3a4c280cd18d7730ce5685f6b5c.m.png)

루트 디렉터리 아래에 A, B, C 그리고 다시 A 아래에 D, E 요소가 있는 구조를 생각해보겠습니다. 위 그림에서 확인할 수 있듯이 모든 요소는 루트 아래에 존재하게 됩니다. 이 시스템 위에서 어떤 프로세스 R을 실행했다고 해보겠습니다.

![](https://d2uleea4buiacg.cloudfront.net/files/223/223215ac51b332401bf55ad083ae10afcecbca25e7a4e18deafb76d1cbf75820.m.png)

이 프로세스 R은 기본적으로 루트 디렉터리를 기준으로 다른 파일들을 탐색할 수 있습니다. 예를 들어 프로세스 아래에서 A 아래의 E 파일에 접근하고자 한다면 `/A/E`와 같이 접근할 수 있습니다. 우리가 실행하는 프로세스들은 기본적으로 이러한 방식으로 파일 시스템에 접근합니다. 즉, 일반적인 프로세스는 파일 시스템의 `/`를 루트 디렉터리로 하는 프로세스라고 말할 수 있습니다.

chroot는 바로 이 루트 디렉터리 `/`를 다른 위치로 지정해서 프로세스를 실행해주는 프로그램입니다. 여기서 아주 중요한 사실이 있습니다. 루트 디렉터리를 파일 시스템의 최상위 요소인 `/`으로 지정하지 않는다면, 결국 `/` 아래의 어떤 디렉터리만이 새로운 루트가 될 수 있다는 점입니다. 예를 들어 A를 루트 디렉터리로 하는 프로세스 K를 생각해보겠습니다.

![](https://d2uleea4buiacg.cloudfront.net/files/a77/a77b00d3bd5ae17c1fa41357d3fb98d15155426e045819117945cf44ac159698.m.png)

chroot를 사용해 실행된 프로세스 K의 루트 디렉터리는 더 이상 `/`이 아닙니다. `/A`가 루트 디렉터리가 됩니다. 여기서 프로세스 R과 아주 중요한 차이가 발생합니다. 프로세스 R은 `/`를 기준으로 그 아래의 모든 파일을 탐색할 수 있었습니다. 하지만 프로세스 K의 루트 디렉터리는 `/A`이기 때문에 `/`에 접근하는 것이 불가능합니다. 그리고 `/` 아래에 있는 B와 C에도 접근할 수 없습니다. 왜냐면 프로세스 K에게는 `/A`가 최상위 디렉터리, 즉 `/`이기 때문에 그 위에 있는 경로를 표현할 방법 자체가 없습니다.

이처럼 루트 디렉터리를 변경하면 특정 프로세스(K)가 상위 디렉터리에 접근할 수 없도록 격리 시킬 수 있습니다. 정확히 이 역할을 하는 것이 chroot 명령어입니다.

하지만 chroot 명령어에는 약점이있습니다. 

1. 탈옥이 가능하다.
2. 완전한 격리가 불가능하다. (Host의 filesystems, network 등에 접근 가능)
3. root 권한 사용 가능하다.
4. host의 자원을 무제한으로 사용할 수 있다.

그래서 Docker에서 프로세스를 격리할때는 1번 약점을 보완한 pivot_root라는 명령어를 사용합니다.

<br>


> ## Pivot_root

- pivrot_root란 root 파일시스템의 마운트 포인트를 변경함으로써 특정 디렉토리를 새로운 루트 디렉토리로 만드는 명령어

앞서 chroot로 만든 격리공간은 root 파일시스템이 동일하였기 때문에 탈옥이 가능했던 한편, pivrot_root를 이용하면 root 파일시스템의 마운트 포인트 자체가 변경되기 때문에 탈옥 자체가 불가능합니다.

![158943948-ed774866-9d7c-4b51-927f-58c38a193625.png](https://user-images.githubusercontent.com/42667951/158943948-ed774866-9d7c-4b51-927f-58c38a193625.png)


_예제_

```bash
mkdir jail # 격리공간 생성
mount -n -t tmpfs -o size=800M none ./jail/ # 파일시스템 800MB 마운트
df -h # 파일시스템 마운트 확인
cp -r /bin jail/          #
mkdir -p jail/usr         # 명령어를 사용하기 위한 파일들 cp
cp -r /usr/bin jail/usr/  # 
cp -r .* jail/            #
unshare -m # 사용자 공간 분리
mkdir old_root # 루트 디렉토리를 마운트시킬 디렉토리 생성
pivot_root . old_root/ 
# 현재 디렉토리를 루트디렉토리로 변경하고 원래 루드디렉토리를 old_root에 마운트
umount -l old_root # old_root 마운트 해제
```

<br>

> ## Namespaces

VM에서는 각 게스트 머신별로 독립적인 공간을 제공하고 서로가 충돌하지 않도록 하는 기능을 갖고 있습니다. 리눅스에서는 이와 동일한 역할을 하는 namespaces 기능을 커널에 내장하고 있습니다. 글을 쓰는 시점을 기준으로 현재 리눅스 커널에서는 다음 6가지 namespace를 지원하고 있습니다:

<br>

- ### Mount namespace
	- Linux에서 파일 시스템을 사용하기 위해서는 마운트가 필요합니다. 마운트란 컴퓨터에 연결된 기기나 기억장치를 OS에 인식시켜 이용 가능한 상태로 만드는 것을 말한다. 
	- 호스트 파일시스템에 구애받지 않고 독립적으로 파일시스템을 마운트하거나 언마운트 가능합니다.

<br>

- ### UTS namespace
	- namespace 별로 호스트명이나 도메인명을 독자적으로 가질 수 있습니다.

<br>


- ### IPC namespace
	- 프로세스 간의 통신(IPC) 오브젝트를 namespace별로 독립적으로 가질 수 있다. IPC는 System V 프로세스 간의 통신 오브젝트라고 하는 공유 메모리나 세마포어/메시지 큐를 말합니다. 세모포어란 프로세스가 요구하는 자원 관리에 이용되는 베타제어 장치이며, 메시지 큐란 여러 플세스간에서 비동기통신을 할 떄 사용되는 큐잉 장치입니다.

<br>

- ### PID namespace
	- PID란 Linux에서 각 프로세스에 할당된 고유한 ID를 말합니다. PID namespace는 PID와 프로세스를 격리시킵니다. namespace가 다른 프로세스끼리는 서로 엑세스 할 수 없습니다.

<br>


- ### Net namespace
	- 네트워크 디바이스, IP 주소, 포트 번호, 라우팅 테이블, 필터링 테이블 등과 같은 네트워크 리소스를 격리된 namespace마다 독립적으로 가질 수 있습니다. 이 기능을 사용하면 호스트 OS 상에서 사용중인 포트가 있더라도 컨테이너 안에서 동일한 번호의 포트를 사용할 수 있습니다.

<br>

- ### User namespace
	- 독립적인 사용자를 할당합니다.

> ## Cgroups

- Control Groups의 약자로 프로세스들이 사용할 수 있는 컴퓨팅 자원들을 제한하고 격리시킬 수 있는 리눅스 커널의 기능이다. namespace와 더불어 도커 컨테이너에서 완벽한 격리 환경을 만드는 데에 쓰이는 중요한 기능이다. cgroup를 이용하면 다음 자원들을 제한할 수 있다
	- 메모리
	- CPU
	- Network
	- Device
	- I/O

<br>

- cgroup 은 컴퓨터의 자원(CPU, 메모리, network, storage I/O)가 각 프로세스 별로 독립적으로 격리되어 할당한다.
 ![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbCR0JD%2Fbtq6CABwrdM%2F5fnxMb34lUQwDhC6VImUk0%2Fimg.png)

> ## 정리

1. ~~탈옥이 가능하다~~ -> Pivot_root로 해결이 가능
2. ~~완전한 격리가 불가능 (Host의 filesystems, network 등에 접근 가능)~~ -> net namespace로 해결 가능
3. ~~root 권한 사용 가능~~ -> user namespace로 해결 가능
4. ~~host의 자원을 무제한으로 사용~~ -> Cgroups로 해결 가능 

<br>

이 처럼 Linux에서 제공하는 기능들을 사용하여 프로세스의 완전한 격리를 할수 있게됩니다.

---
## **reference**

https://www.44bits.io/ko/post/change-root-directory-by-using-chroot


