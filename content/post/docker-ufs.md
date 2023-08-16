---
title: "[Docker] 유니온 파일 시스템"
date: 2023-08-16T11:51:28+09:00
draft: false
pin: false 
summary: "docker는 이미지를 어떻게 관리할까?"
tags: [docker]
---

> # 유니온 파일 시스템(UFS)

일반적으로 우리가 파일시스템에 파일 등을 생성하면 그 파일의 크기를 쉽게 확인할 수 있다.  
즉, 생성한 파일이 차지하는 스토리지 공간을 정확하게 확인할 수 있는 것이다.

그런데, Docker 를 통해 기동한 Container 의 크기(스토리지 점유 크기)를 계산하려면 사실 약간 막막함이 있다.  
서비스를 위해 Container 를 10 개를 기동하여 사용 중인데, 과연 이 Container 들이 디스크 공간을 얼마나 사용하고 있는건지 정확하게 계산할 수 있는가?

오늘은 Container size 에 대한 얘기를 하려고 한다.

Container size 를 정확하게 계산하려면 먼저 Docker 가 사용하는 Union File System(UFS)에 대해 이해해야 한다.  
Union 은 사전적으로 '연합', '결합'이라는 의미를 가지고 있다.  
즉, UFS 라고 하면 File system 의 결합, 연합 정도로 생각하면 되겠다.  
**여러 개의 File system 을 하나로 결합하여 취급할 수 있도록 해주는 것**이다.

여러분은 Docker 에서 Image 와 Container 의 차이를 이미 이해하고 있을 것이다.  
흔히 설명을 위해 Linux 의 Process 모델과 비교를 하게 된다.

- 프로그램 Binary = Docker Image
- Process = Container

Process 를 여러 번 실행하더라도 프로그램 Binary 는 변경되지 않는다.  
즉, Process 가 기동 중에 내부에서 어떤 짓을 하더라도 Binary 자체가 변경되는 일은 없다.  
또한, 다수의 Process 가 하나의 Binary 를 이용하여 기동될 수 있다.

이러한 개념이 Docker 의 image 와 container 에서도 적용된다.  
image 를 이용하여 container 를 다수 기동할 수 있지만 image 자체는 변경되지 않는다.  
Container 를 이용하여 ubuntu 를 기동했다고 가정해보자.  
Container 내부에서는 사용자가 마치 자신의 독립적인 ubuntu 와 파일시스템을 가진 양 자유롭게 사용할 수 있다.  
즉, 신규 파일이나 디렉토리를 생성할 수도 있고, 기존에 존재하던 객체 들을 마음대로 변경할 수 있다.  
하지만, Container 기동에 사용되는 image 는 변경되지 않는다고 말했다.  
그렇다면 Container 내에서 변경한 내용들(파일 생성 등)은 어디에 저장되는 것일까?

그 비밀이 UFS 에 있다.

![](https://miro.medium.com/max/1600/1*hZgRPWerZVbaGT8jJiJZVQ.jpeg)

우리가 Container 내부에서 보는 File system 은 사실 UFS 라는 창을 통해서 보는 것과 같다.  
그 내부에는 어떤 File system 으로 구성되어 있든지간에, 우리는 UFS 가 보여주는 방식에 따라 자신이 하나의 File system 을 독점하여 사용하는 것처럼 착각 속에서 사용하고 있는 것이다.  
우리가 Container 내부에서 바라보는 File system 내부에는 사실 여러 개의 File system 이 겹쳐져 있는 상태이다.  
마치 서로 다른 무늬가 그려진 투명 셀로판지 여러 개를 겹쳐보면 하나의 합쳐진 무늬처럼 보이듯이, UFS 는 하위의 다수 File system 을 투명으로 겹쳐서 사용자에게 하나처럼 보여주는 것이다.

Container 는 Image 를 이용해서 기동된다.  
그런데, 해당 Image 는 반드시 하나의 통 image 로만 구성되어 있지 않을 수도 있다.  
맨 처음 ubuntu 라는 base image 를 이용하여 container 를 기동한 뒤, 사용자는 container 내 에서 emacs 나 apache 와 같은 프로그램을 추가로 설치할 수도 있을 것이다. (사용자 파일 생성도 마찬가지이다.)  
만약 이 상태에서 container 를 중지하고 삭제하면, 설치했던 emacs 나 apache 는 함께 삭제되고 재 사용도 할 수 없다.  
이를 방지하기 위해 사용자는 이러한 변경 상태를 그대로 image 로 만들어 둘 수 있다.  
이후 사용자가 해당 image 를 사용해서 container 를 기동하면, 설치했던 emacs 나 apache 는 설치된 그대로 남아있게 된다.  
이렇게 image 위에 변경을 가하고 이를 또 다시 image 로 만드는 것을 반복할 수 있다.  
Docker 는 이러한 변경들을 image layer 라는 개념을 두어서 차곡차곡 쌓아놓는다.  
svn 이나 git 같은 소스 관리 툴을 이용해본 사람은 이미 눈치를 챘겠지만, Docker 또한 image 의 변경 사항(Diff)을 하나하나 순차적으로 저장해두고, 사용자에게는 이를 원하는 layer 까지 합친 것으로 보여주는 것이다.

특정 image 를 이용하여 container 를 기동한다는 것은, 이렇게 차곡차곡 쌓인 image layer 를 먼저 하단에 깔아두고(Read-Only Image Layer), 최종적으로 맨 위에 Container layer(Writable layer)를 얹은 후, 이 모든 layer 를 결합하여 사용자에게 하나의 File system view(root filesystem)로 제공하는 것이다.

![](https://docs.docker.com/storage/storagedriver/images/sharing-layers.jpg)

container 를 기동할 때 사용했던 image 는 변경되지 않고 그대로 유지된다.  
왜냐하면, Writable layer 는 Read-only layer 와 완전히 별개이니까.  
(마치 Process 가 기동해서 뭔 짓을 해도 프로그램 binary 는 그대로인 것처럼)  
다만, Diff 는 최 상단의 Writable layer 에 쌓이고 이는 container 제거 시 함께 삭제된다.  
이러한 Diff 를 Persistent 하게 유지하고자 한다면 다음의 2 가지 방법을 이용해야 한다.

- **Container 상에서 변경을 수행한 후 image 로 만들어둔다.**
- **[Volume](https://medium.com/dtevangelist/docker-%EA%B8%B0%EB%B3%B8-5-8-volume%EC%9D%84-%ED%99%9C%EC%9A%A9%ED%95%9C-data-%EA%B4%80%EB%A6%AC-9a9ac1db978c)을 구성하여 변경사항은 해당 volume 에 저장하도록 한다.**

관리적인 측면에서 보면 **Image 의 변경은 없을수록 좋다**.  
그래야만, 반복적으로 같은 image 를 이용하여 container 를 구동할 수 있을 것이다.  
따라서, 위의 첫 번째 방법처럼 변경이 발생할 때마다 또 다른 image 를 만드는 것은 관리 상 좋은 방법이 아니다.  
두 번째 방법처럼 변경부분은 별도로 빼서 Volume 에 저장하는 것이 일반적으로 좋은 방법이라고 할 수 있다.

자, 이제 image layer 와 UFS에 대해 이해를 했으니 Container size 를 계산해보자.  
먼저 ubuntu:latest image 를 이용하여 container 하나를 기동한다.  
ubuntu image 는 docker pull 을 통해 이미 local 에 저장한 상태라고 가정한다.

![](http://cloudrain21.com/wordpress/wp-content/uploads/2019/09/docker-ps.png)

image 크기가 64.2 MB 이고, 이를 이용하여 container 를 기동하면 container size 또한 64.2 MB 인 것을 볼 수 있다.  
Container size 는 다음의 두 가지 종류를 고려해야 한다.

- **virtual size : read-only size + writable size**
- **size : writable size**

docker ps 결과에서 SIZE 항목을 보면 **0B (virtual 64.2MB)** 결과를 볼 수 있는데, 이 때 0B 는 size(writable size)를 의미하고 64.2MB 는 virtual size(read-only + writable size)를 의미한다.  
즉, 처음 container 를 기동한 직후에는 변경 사항이 아직 없으므로 writable size 가 0 이 된다.  
그리고, virtual size 는 read-only size 와 같다.

이 상태에서 container 내부에서 약 4 MB 짜리 파일을 생성해본다.

![](http://cloudrain21.com/wordpress/wp-content/uploads/2019/09/dd.png)

이 상태로 외부에서 container 크기를 조회해보면 다음과 같이 size 부분(4.1MB)과 virtual size 부분(68.3MB)이 모두 4.1 MB 만큼 증가하였음을 볼 수 있다.  
이는 곧 writable layer 에 4.1 MB 만큼 변경이 생겼다는 것을 의미한다.

![](http://cloudrain21.com/wordpress/wp-content/uploads/2019/09/container-size.png)

이렇게 container 에 변경을 가한 후 이를 바탕으로 새로운 image 를 만들어본다. (docker commit, docker tag)  
image 이름은 container_size_test 라고 지정하였다.

![](http://cloudrain21.com/wordpress/wp-content/uploads/2019/09/commit.png)

docker image history 명령을 통해 새로 만든 image 의 history 정보를 조회해보면, 맨 상단의 layer 가 최종적으로 image 에 적용된 부분이라는 것을 알 수 있다. (4.1 MB)  
이제 위에서 생성한 container_size_test 라는 image 를 이용하여 container 를 구동하면, 이 모든 layer 가 read-only layer 가 되는 것이다.  
물론 최 상단에는 새로운 writable layer 가 만들어지고, 이후에는 이 곳에 변경을 수행할 수 있다.

지금까지 살펴본 내용을 바탕으로 다수의 container 가 동일한 image 로 구동된 상황에서 해당 container 들이 사용하는 모든 공간을 계산해볼 수 있다.  
즉, 특정 image 를 이용하여 기동된 다수의 container 들이 사용하는 공간의 합은, **공통적으로 사용하는 read-only layer 크기 + 각 container 에서 변경한 사항들(writable layer)의 총합**이 될 것이다.

**_(virtual size - size) + (size of all containers)_**

참고로 이렇게 container 들이 사용하는 image 나 변경사항들은 모두 호스트 File system 의 **/var/lib/docker** 디렉토리 내에 저장된다.  
이 영역을 **Docker area 또는 Backing Filesystem** 이라고 부르기도 한다.

### Copy-on-Write

Container 내에서의 변경 관련하여 한 가지 더 알아두어야할 사항이 있다.  
만약 기동된 Container 의 base 가 되는 image layer(read-only layer)에 있는 내용을 변경하고 싶다면 어떻게 해야 할까?  
예를 들어, image 에 이미 존재하는 /etc/xxx.ini 라는 파일의 내용을 변경한다면 어떻게 되는 것일까?  
read-only image layer 의 내용은 프로그램 binary 처럼 변경되지 않는다고 하지 않았던가?

이러한 경우를 위해 Docker 는 **[Copy-on-Write 기법](https://blog.codeship.com/docker-storage-introduction/)**을 사용한다.  
기존의 image 위에 추가되는 내용은 Writable layer 에 그냥 기록하면 된다.  
하지만, image 의 기존 내용이 변경되어야할 경우에는 해당 data 를 Writable layer 로 먼저 Copy 한 후 이를 변경하는 기법을 사용한다.  
이러한 기법을 통해 기존의 image 에 대한 변경을 막을 수 있다.  
변경 후에는 사용자는 하단의 data 는 볼 수 없고, Writable layer(container layer)에 저장된 내용을 보게 된다.  
Copy-on-Write 기법은 그 동작 구조 상 다음의 단점이 있을 수 밖에 없음을 기억하자.

- data 를 먼저 복제(Copy)한 후 변경을 수행해야 하므로 성능 하락(Performance Overhead)을 피할 수 없다.
- 하위 layer 에 있는 원본 data 를 유지해야할 뿐 아니라, 상위 layer 에 변경된 data 도 유지해야 하므로 disk space 를 많이 사용할 수 있다.

위의 2 가지 단점 중 두번째 단점은 사용자가 아키텍처 설계를 효율적으로 한다면 피할 수 있다고 생각한다.  
하위 layer 의 data 까지 수시로 변경이 발생하도록 설계하는 것은 좋은 방법이 아니다.  
이미 언급한대로, 되도록이면 base image 의 변경을 발생시키지 않도록 변경 사항은 별도의 Volume 구성 등의 방법을 통해 회피해야 한다.

### Storage Driver

위에서 UFS 에 대한 개념을 설명하였는데, 그러면 이러한 UFS 기능을 제공하기 위해 Docker 는 내부적으로 어떤 기술을 이용할까?  
즉, UFS 구현에 대한 이야기이다.

UFS 는 Docker Storage Driver 에 의해 구현된다.  
Container 에서 사용자가 UFS 를 통해 Filesystem 을 쉽게 사용할 수 있는 것은 Storage Driver 가 있기 때문이다.  
바로 위에서 언급한 Copy-on-Write 기법을 실현해주는 것도 Storage Driver 이다.  
Storage Driver 는 container 내에서의 파일 I/O 처리를 담당하는 놈이다.

다양한 Storage Driver 가 존재하며 Pluggable 한 구조로 되어 있기 때문에, Docker 에서 Storage Driver 를 선택하여 사용할 수 있다.  
다만, 모든 Linux system 이 모든 Storage Driver 를 지원하지는 않는다.  
Ubuntu 는 Default Storage Driver 로 aufs 를 사용하며, Centos 는 devicemapper 를 사용한다.  
Redhat 계열은 aufs 를 지원하지 않는다.

다음은 OS 별로 지원하는 Storage Driver 를 정리한 것이다.  
Storage Driver 별로 특성이 다르고 장단점이 있으므로, [이곳](https://docs.docker.com/storage/storagedriver/select-storage-driver/)을 먼저 참조하여 자신의 시스템 workload 에 적합한Storage Driver 를 선택하여 사용하도록 하자.  
("Suitability for your workload" Part 참조)  
나의 개인 용 PC 에는 Centos7, Docker 1.11.1 이 설치되어 있어서 Docker 의 Default Storage Driver 로 devicemapper 를 사용하고 있다.  
(이는 docker info 명령의 'Storage Driver' 항목을 통해 확인할 수 있다.)

![](http://cloudrain21.com/wordpress/wp-content/uploads/2019/09/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA-2019-09-04-%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE-10.28.11.png)

결국 Container 상에서 변경되는 사항들은 Storage Driver 의 도움에 의해 Backing filesystem(/var/lib/docker)에 저장되는 것이다.  
(위에서 언급한 **Volume 기능은 이러한 Storage Driver 의 도움없이 직접적으로 Host 의 Filesystem 에 접근하기 때문에 성능 상 잇점**이 있다.)

Storage Driver 가 모든 종류의 Host filesystem 을 Backing filesystem 으로 사용할 수 있는 것은 아니다.  
다음은 Storage Driver 별로 지원하는 Backing filesystem 을 나타낸다.  
기억할 필요까지는 없고 필요할 때 찾아볼 수 있도록 하자.

![](http://cloudrain21.com/wordpress/wp-content/uploads/2019/09/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA-2019-09-04-%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE-10.30.27-1024x392.png)

Backing filesystem 을 조금 더 이해하자는 의미에서, 나의 Desktop 환경을 기준으로 Backing filesystem 과 container layer 의 연관관계를 잠깐 들여다보자.  
Union filesytem 의 최상단에 위치하는 container layer(writable layer)는 그 자체로는 의미가 없다.  
무언가 변경이 되었을 때 이를 저장할 곳이 필요하다.  
이곳이 Host 의 Backing filesystem 이다.  
따라서, container layer 는 Backing filesystem 의 특정 영역에 mount 가 되어 있어야 한다.  
container 를 하나 기동한 후 mount 상황을 조회해보면 아래와 같이 mount 가 하나 추가되는 것을 볼 수 있다.

```shell
/dev/mapper/docker-253:0-205758706-b110da1e299a91352d355601518347132f0a90e71e38848d85871096d7ab7d0d on /var/lib/docker/devicemapper/mnt/b110da1e299a91352d355601518347132f0a90e71e38848d85871096d7ab7d0d type ext4 (rw,relatime,stripe=16,data=ordered)
```

즉, /dev/mapper/docker-253:0-205758706-b110da1e...(container layer 라고 이해하자)가 /var/lib/docker/devicemapper/mnt/b110da1e... 디렉토리에 mount 되어 있음을 알 수 있다.  
mount 된 Host 의 Filesystem type 은 ext4 이다.  
이는 devicemapper 라는 Storage Driver 가 자동으로 수행해주는 작업이다.

이쯤에서 ~~쥐꼬리만한~~ 상상력을 발휘하여 예측을 하나 해보자.  
container 내에서 어떠한 변경을 발생시키면(예를 들어, 신규 파일 생성), /var/lib/docker/devicemapper/mnt/b110da1e... 디렉토리 내에서도 그 변경이 보이지 않을까?  
mount 가 되어 있으니까...

해보자.

```bash
# container 안에서 aa.txt 파일 생성

root@8b037d600474:/#  touch aa.txt

# /var/lib/docker/devicemapper/mnt/b110da1e.../rootfs 안에 aa.txt 파일이 생성되어 있음을 확인

[root@localhost b110da1e299a91352d355601518347132f0a90e71e38848d85871096d7ab7d0d]# ls
id  lost+found  rootfs
[root@localhost b110da1e299a91352d355601518347132f0a90e71e38848d85871096d7ab7d0d]# cd rootfs/
[root@localhost rootfs]# ls -al
합계 84
drwxr-xr-x. 21 root root 4096  9월  4 18:47 .
drwxr-xr-x   4 root root 4096  9월  4 19:02 ..
-rwxr-xr-x   1 root root    0  9월  4 18:42 .dockerenv
-rw-r--r--   1 root root    0  9월  4 18:47 aa.txt   <--- 생겼다~
drwxr-xr-x.  2 root root 4096 12월  8  2015 bin
drwxr-xr-x.  2 root root 4096  4월 11  2014 boot
drwxr-xr-x.  4 root root 4096  9월  4 18:42 dev
drwxr-xr-x. 61 root root 4096  9월  4 18:42 etc
drwxr-xr-x.  2 root root 4096  4월 11  2014 home
drwxr-xr-x. 12 root root 4096 12월  8  2015 lib
drwxr-xr-x.  2 root root 4096 12월  8  2015 lib64
drwxr-xr-x.  2 root root 4096 12월  8  2015 media
drwxr-xr-x.  2 root root 4096  4월 11  2014 mnt
drwxr-xr-x.  2 root root 4096 12월  8  2015 opt
drwxr-xr-x.  2 root root 4096  4월 11  2014 proc
drwx------.  2 root root 4096 12월  8  2015 root
drwxr-xr-x.  8 root root 4096  9월  4 18:42 run
drwxr-xr-x.  2 root root 4096 12월  9  2015 sbin
drwxr-xr-x.  2 root root 4096 12월  8  2015 srv
drwxr-xr-x.  2 root root 4096  3월 13  2014 sys
drwxrwxrwt.  2 root root 4096 12월  8  2015 tmp
drwxr-xr-x. 10 root root 4096 12월  9  2015 usr
drwxr-xr-x. 11 root root 4096 12월  9  2015 var
```

예측이 맞았다.  
/var/lib/docker/devicemapper/mnt/b110da1e... 디렉토리 내에는 **rootfs** 라는 디렉토리가 존재하는데, 이 디렉토리가 container 안에서는 root(/)로 보이게 된다.  
이 때문에 container 내에서 root(/) 디렉토리 아래에 파일을 생성하면, Host 에서는 위의 디렉토리에 파일이 생성되는 것이다.  
이는 **Mount namespace** 라는 기술을 이용하여 Host 의 특정 디렉토리를 container 내에서는 root(/) 디렉토리로 보이도록 해주기 때문이다.



---
## _reference_

http://cloudrain21.com/examination-of-docker-containersize-ufs