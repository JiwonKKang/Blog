---
title: "[Docker] 컨테이너 다루기"
date: 2023-08-16T20:14:00+09:00
draft: false
pin: false 
summary: "컨테이너를 파헤쳐보자"
tags: [docker]
---

> # 컨테이너 다루기

# 컨테이너 생성

## 버전확인

Docker를 사용하기에 앞서 Docker version 확인

```bash
sudo docker -v
```

![Untitled](https://island-primula-917.notion.site/image/https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2F68789a98-b03a-4451-979b-24342c0922af%2FUntitled.png?table=block&id=edb63ea9-b22b-4f48-943e-3e95cda7fa99&spaceId=9bc99f35-edf4-4599-af44-3f7fb94b56cd&width=1380&userId=&cache=v2)

현재 docker의 최신 버전은 24.0.5

다양한 기능이 빠르게 업데이트되고 새로운 버전이 배포되므로 설치된 버전 확인 중요

내친김에 containerd도 버전을 확인해봅시다

```bash
sudo containerd -v
```

![Untitled](https://island-primula-917.notion.site/image/https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2Fe408afe5-63c9-4bd1-861f-321f751a75fd%2FUntitled.png?table=block&id=a3c782ef-49c8-4bdf-9409-8e471958b455&spaceId=9bc99f35-edf4-4599-af44-3f7fb94b56cd&width=2000&userId=&cache=v2)

## 첫번째 컨태이너 생성

docker run 명령어는 컨테이너를 생성하고 실행하는 역할 ubuntu:18.04는 컨테이너를 생성하기 위한 이미지의 이름 -i -t 옵션은 컨테이너와 상호 입출력 가능

```bash
sudo docker run -it ubuntu:18.04
```

아래와 같은 내용이 출력됩니다.

ubuntu:18.04 이미지가 로컬 도커 엔진에 존재하지 않으므로 도커 중앙 이미지 레지스트리인 dockerhub에서 자동으로 이미지를 내려받습니다.

네트워크 환경에 따라 시간이 약간 소요될 수 있음

![Untitled](https://island-primula-917.notion.site/image/https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2F02992a3e-af14-4adc-ac60-473d9c668418%2FUntitled.png?table=block&id=873871ef-3230-4bea-866c-fee3d9c8917a&spaceId=9bc99f35-edf4-4599-af44-3f7fb94b56cd&width=2000&userId=&cache=v2)

단 한줄의 명령으로 container 이미지 다운로드, 이미지로부터 컨테이너 생성, 컨테이너 실행, 컨테이너 내부 연결

쉘의 사용자와 호스트이름이 변경된 것이 컨테이너 내부에 들어와있다는 것을 나타냄

해당 컨테이너에서 기본 사용자는 root이고 호스트네임은 무작위 해쉬

무작위의 16진수 해시값은 컨테이너의 고유한 앞 일부분

![Untitled](https://island-primula-917.notion.site/image/https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2Fc0abef6a-d024-4c03-ac54-2509a59e4ea5%2FUntitled.png?table=block&id=e834768c-60f6-4085-af99-fae698679824&spaceId=9bc99f35-edf4-4599-af44-3f7fb94b56cd&width=2000&userId=&cache=v2)

-i옵션 : 상호입출력

-t옵션 : tty

bash 쉘 사용하도록 컨테이너 설정, 두 옵션 다 사용해야 정상적으로 쉘 사용 가능

![Untitled](https://island-primula-917.notion.site/image/https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2Fb83e9be4-61aa-4ba9-821f-e58d96f5bf7c%2FUntitled.png?table=block&id=8be7acf3-5af7-43fc-a5dc-66079632f053&spaceId=9bc99f35-edf4-4599-af44-3f7fb94b56cd&width=2000&userId=&cache=v2)

컨테이너와 호스트의 파일 시스템은 서로 독립적이므로 ls 명령어로 파일시스템을 확인해보면 아무ㄷ것도 설치되지 않은 상태임을 확인할 수 있음

```bash
root@b99ed1baaccc:/# ls
```

![Untitled](https://island-primula-917.notion.site/image/https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2F08d17f4c-d55e-4e6b-b362-17d083a81e41%2FUntitled.png?table=block&id=8bf1fe42-0a78-4e24-8d41-6ee48bace13f&spaceId=9bc99f35-edf4-4599-af44-3f7fb94b56cd&width=2000&userId=&cache=v2)

컨테이너에서 호스트의 docker 환경으로 돌아옵니다.

컨테이너 내부에서 빠져나오는 방법은 두가지

1. exit 명령 또는 ctrl + D 입력
    
    ```bash
    root@b99ed1baaccc:/# exit
    ```
    
    이 경우 빠져나오면서 컨테이너를 정지시킴
    
    ![Untitled](https://island-primula-917.notion.site/image/https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2F9c7d4be1-3922-4a19-aee3-ac6e4d491a3e%2FUntitled.png?table=block&id=ca581d31-fd01-47a2-841a-3814d4044990&spaceId=9bc99f35-edf4-4599-af44-3f7fb94b56cd&width=1020&userId=&cache=v2)
    
    ```bash
    sudo docker ps
    sudo docker ps -a
    ```
    
    ![Untitled](https://island-primula-917.notion.site/image/https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2F1bafd73a-3763-4d8c-9121-c060b3a099c7%2FUntitled.png?table=block&id=7c120c5a-343a-4027-9e2d-47f99926ca15&spaceId=9bc99f35-edf4-4599-af44-3f7fb94b56cd&width=2000&userId=&cache=v2)
    
2. ctrl + P, Q 입력
    
    컨테이너를 정지하지 않고 빠져나옴
    
    컨테이너가 정지하지 않고 단순히 쉘에서만 빠져나오기 때문에 컨테이너 애플리케이션 개발 목적으로 주로 사용
    
    centOS7 이미지를 사용하는 컨테이너 예시
    
    ```bash
    sudo docker pull centos:7
    ```
    
    이미지가 정상적으로 받아졌는지 확인하기 위해 images 명령어로 확인
    
    images 명령은 도커엔진에 존재하는 이미지의 목록을 출력
    
    ```bash
    sudo docker images
    ```
    
    ![Untitled](https://island-primula-917.notion.site/image/https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2F2750fc1a-5178-475f-94f6-23f8efb72c5c%2FUntitled.png?table=block&id=5de0e779-15da-491d-87ba-47f98ac2d06d&spaceId=9bc99f35-edf4-4599-af44-3f7fb94b56cd&width=2000&userId=&cache=v2)
    
    컨테이너를 생성할 때는 run 명령어가 아닌 create 명령어를 사용할 수도 있습니다.
    
    다음 명령어로 centos:7 이미지로 컨테이너를 생성합니다.
    
    —name 옵션에는 컨테이너의 이름을 설정합니다.
    
    여기서는 mycentos로 설정합니다.
    
    ```bash
    sudo docker create -it --name mycentos centos:7
    ```
    
    ![Untitled](https://island-primula-917.notion.site/image/https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2F004665bd-55bd-40c0-b536-026ea6e6d544%2FUntitled.png?table=block&id=e0b2be60-7c53-446e-a7da-ac00344a64f9&spaceId=9bc99f35-edf4-4599-af44-3f7fb94b56cd&width=2000&userId=&cache=v2)
    
    그런데 이번에는 컨테이너 내부로 들어가지는 않습니다.
    
    create 명령어는 컨테이너를 생성만 할 뿐 컨테이너로 들어가지 않기 때문입니다.
    
    이번에는 docker start 명령어로 컨테이너를 시작하고 docker attach 명령어로 내부로 들어갑니다.
    
    ```bash
    sudo docker start f1afdca9b635
    ```
    
    ![Untitled](https://island-primula-917.notion.site/image/https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2F4ad511dd-2aac-4727-9227-ffb28978a9af%2FUntitled.png?table=block&id=7f3704a4-cb81-4a94-9ab8-a5bc32c54725&spaceId=9bc99f35-edf4-4599-af44-3f7fb94b56cd&width=2000&userId=&cache=v2)
    
    ```bash
    sudo docker attach mycentos
    ```
    
    ![Untitled](https://island-primula-917.notion.site/image/https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2Fd71fc265-142f-47eb-80e2-683f87721c2a%2FUntitled.png?table=block&id=4ecbbbdd-43c6-4b7c-9cdc-187772477f3d&spaceId=9bc99f35-edf4-4599-af44-3f7fb94b56cd&width=1800&userId=&cache=v2)
    
    이번에는 exit가 아닌 ctrl + P, Q를 입력해 컨테이너에서 빠져나옵니다.
    
    ![Untitled](https://island-primula-917.notion.site/image/https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2F2b7882e2-9987-4906-bff3-d88ca3fc344a%2FUntitled.png?table=block&id=b5d75fb1-9ca9-413d-b873-0d77bcae74fa&spaceId=9bc99f35-edf4-4599-af44-3f7fb94b56cd&width=2000&userId=&cache=v2)
    

지금까지 컨테이너를 생성하기 위해 run, create, start 명령어를 사용

run 명령은 pull, create, start명령어를 일괄적으로 실행한 후 attach가 가능한 컨테이너라면 컨테이너 내부로 들어갑니다.

하지만 create 명령어는 도커 이밎를 pull한 후에 컨테이너를 생성만 할 뿐 start attach를 실행하지는 않습니다.

보통은 생성과 동시에 실행시키기 때문에 run을 주로 씁니다.

![Untitled](https://island-primula-917.notion.site/image/https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2Fc8c2b5c7-28a8-4e39-91ff-76cb3ee13d55%2FUntitled.png?table=block&id=cbe4c36f-d2eb-4d4f-b503-507be348c8a2&spaceId=9bc99f35-edf4-4599-af44-3f7fb94b56cd&width=2000&userId=&cache=v2)

# 컨테이너 목록 확인

docker ps 명령 : 정지되지 않은 컨테이너만 출력 ⇒ exit로 빠져나온 컨테이너는 출력되지 않음 ⇒ ctrl + P,Q로 빠져나온 컨테이너는 출력

docker ps -a 명령 : 정지된 컨테이너까지 포함해서 출력

Exited : 정지상태, Up : 실행중인 상태

### 컨테이너 세부 정보 확인

docker inspect 명령 : 컨테이너의 세부 정보를 보여줍니다.

```bash
sudo docker inspect mycentos
```

```json
[
    {
        "Id": "f1afdca9b635191d9cd078564c21204fc96c387f75d583b83b5c59e41df40365",
        "Created": "2023-04-16T02:08:31.924789206Z",
        "Path": "/bin/bash",
        "Args": [],
        "State": {
            "Status": "running",
            "Running": true,
            "Paused": false,
            "Restarting": false,
            "OOMKilled": false,
            "Dead": false,
            "Pid": 17852,
            "ExitCode": 0,
            "Error": "",
            "StartedAt": "2023-04-16T02:14:58.401748112Z",
            "FinishedAt": "0001-01-01T00:00:00Z"
        },
        "Image": "sha256:eeb6ee3f44bd0b5103bb561b4c16bcb82328cfe5809ab675bb17ab3a16c517c9",
        "ResolvConfPath": "/mnt/storage/docker/containers/f1afdca9b635191d9cd078564c21204fc96c387f75d583b83b5c59e41df40365/resolv.conf",
        "HostnamePath": "/mnt/storage/docker/containers/f1afdca9b635191d9cd078564c21204fc96c387f75d583b83b5c59e41df40365/hostname",
        "HostsPath": "/mnt/storage/docker/containers/f1afdca9b635191d9cd078564c21204fc96c387f75d583b83b5c59e41df40365/hosts",
        "LogPath": "/mnt/storage/docker/containers/f1afdca9b635191d9cd078564c21204fc96c387f75d583b83b5c59e41df40365/f1afdca9b635191d9cd078564c21204fc96c387f75d583b83b5c59e41df40365-json.log",
        "Name": "/mycentos",
        "RestartCount": 0,
        "Driver": "overlay2",
        "Platform": "linux",
        "MountLabel": "",
        "ProcessLabel": "",
        "AppArmorProfile": "docker-default",
        "ExecIDs": null,
        "HostConfig": {
            "Binds": null,
            "ContainerIDFile": "",
            "LogConfig": {
                "Type": "json-file",
                "Config": {
                    "max-size": "100m"
                }
            },
            "NetworkMode": "default",
            "PortBindings": {},
            "RestartPolicy": {
                "Name": "no",
                "MaximumRetryCount": 0
            },
            "AutoRemove": false,
            "VolumeDriver": "",
            "VolumesFrom": null,
            "ConsoleSize": [
                39,
                142
            ],
            "CapAdd": null,
            "CapDrop": null,
            "CgroupnsMode": "private",
            "Dns": [],
            "DnsOptions": [],
            "DnsSearch": [],
            "ExtraHosts": null,
            "GroupAdd": null,
            "IpcMode": "private",
            "Cgroup": "",
            "Links": null,
            "OomScoreAdj": 0,
            "PidMode": "",
            "Privileged": false,
            "PublishAllPorts": false,
            "ReadonlyRootfs": false,
            "SecurityOpt": null,
            "UTSMode": "",
            "UsernsMode": "",
            "ShmSize": 67108864,
            "Runtime": "runc",
            "Isolation": "",
            "CpuShares": 0,
            "Memory": 0,
            "NanoCpus": 0,
            "CgroupParent": "",
            "BlkioWeight": 0,
            "BlkioWeightDevice": [],
            "BlkioDeviceReadBps": [],
            "BlkioDeviceWriteBps": [],
            "BlkioDeviceReadIOps": [],
            "BlkioDeviceWriteIOps": [],
            "CpuPeriod": 0,
            "CpuQuota": 0,
            "CpuRealtimePeriod": 0,
            "CpuRealtimeRuntime": 0,
            "CpusetCpus": "",
            "CpusetMems": "",
            "Devices": [],
            "DeviceCgroupRules": null,
            "DeviceRequests": null,
            "MemoryReservation": 0,
            "MemorySwap": 0,
            "MemorySwappiness": null,
            "OomKillDisable": null,
            "PidsLimit": null,
            "Ulimits": null,
            "CpuCount": 0,
            "CpuPercent": 0,
            "IOMaximumIOps": 0,
            "IOMaximumBandwidth": 0,
            "MaskedPaths": [
                "/proc/asound",
                "/proc/acpi",
                "/proc/kcore",
                "/proc/keys",
                "/proc/latency_stats",
                "/proc/timer_list",
                "/proc/timer_stats",
                "/proc/sched_debug",
                "/proc/scsi",
                "/sys/firmware"
            ],
            "ReadonlyPaths": [
                "/proc/bus",
                "/proc/fs",
                "/proc/irq",
                "/proc/sys",
                "/proc/sysrq-trigger"
            ]
        },
        "GraphDriver": {
            "Data": {
                "LowerDir": "/mnt/storage/docker/overlay2/a9276276bbdc0b2db6e6c5de5f226987b2065e0748c009bbfe00e36ee1275dae-init/diff:/mnt/storage/docker/overlay2/3df9691c76574956aea487baf61084bafb91b15f815f681935949d9a4dbf80d2/diff",
                "MergedDir": "/mnt/storage/docker/overlay2/a9276276bbdc0b2db6e6c5de5f226987b2065e0748c009bbfe00e36ee1275dae/merged",
                "UpperDir": "/mnt/storage/docker/overlay2/a9276276bbdc0b2db6e6c5de5f226987b2065e0748c009bbfe00e36ee1275dae/diff",
                "WorkDir": "/mnt/storage/docker/overlay2/a9276276bbdc0b2db6e6c5de5f226987b2065e0748c009bbfe00e36ee1275dae/work"
            },
            "Name": "overlay2"
        },
        "Mounts": [],
        "Config": {
            "Hostname": "f1afdca9b635",
            "Domainname": "",
            "User": "",
            "AttachStdin": true,
            "AttachStdout": true,
            "AttachStderr": true,
            "Tty": true,
            "OpenStdin": true,
            "StdinOnce": true,
            "Env": [
                "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"
            ],
            "Cmd": [
                "/bin/bash"
            ],
            "Image": "centos:7",
            "Volumes": null,
            "WorkingDir": "",
            "Entrypoint": null,
            "OnBuild": null,
            "Labels": {
                "org.label-schema.build-date": "20201113",
                "org.label-schema.license": "GPLv2",
                "org.label-schema.name": "CentOS Base Image",
                "org.label-schema.schema-version": "1.0",
                "org.label-schema.vendor": "CentOS",
                "org.opencontainers.image.created": "2020-11-13 00:00:00+00:00",
                "org.opencontainers.image.licenses": "GPL-2.0-only",
                "org.opencontainers.image.title": "CentOS Base Image",
                "org.opencontainers.image.vendor": "CentOS"
            }
        },
        "NetworkSettings": {
            "Bridge": "",
            "SandboxID": "615507eabcfcbd8e6b47e2669c37460b96667472c37f54a3abee45f8ed0923e3",
            "HairpinMode": false,
            "LinkLocalIPv6Address": "",
            "LinkLocalIPv6PrefixLen": 0,
            "Ports": {},
            "SandboxKey": "/var/run/docker/netns/615507eabcfc",
            "SecondaryIPAddresses": null,
            "SecondaryIPv6Addresses": null,
            "EndpointID": "19fcc1ec53f8b7c08520784248f87ab41f5864ee96130e25003f74a3d1626b14",
            "Gateway": "172.17.0.1",
            "GlobalIPv6Address": "",
            "GlobalIPv6PrefixLen": 0,
            "IPAddress": "172.17.0.2",
            "IPPrefixLen": 16,
            "IPv6Gateway": "",
            "MacAddress": "02:42:ac:11:00:02",
            "Networks": {
                "bridge": {
                    "IPAMConfig": null,
                    "Links": null,
                    "Aliases": null,
                    "NetworkID": "8eec44230881f9f914dc8ce9a4c1335420e803012cf2100388e7cabe148f605c",
                    "EndpointID": "19fcc1ec53f8b7c08520784248f87ab41f5864ee96130e25003f74a3d1626b14",
                    "Gateway": "172.17.0.1",
                    "IPAddress": "172.17.0.2",
                    "IPPrefixLen": 16,
                    "IPv6Gateway": "",
                    "GlobalIPv6Address": "",
                    "GlobalIPv6PrefixLen": 0,
                    "MacAddress": "02:42:ac:11:00:02",
                    "DriverOpts": null
                }
            }
        }
    }
]
```

### CONTAINER ID

컨테이너에 자동으로 할당된 고유한 ID

full id 중에 앞의 12자리만 표시됩니다.

full id는 docker inspect 명령으로 확인이 가능합니다.

```bash
sudo docker inspect mycentos | grep Id
```

![Untitled](https://island-primula-917.notion.site/image/https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2Ff135a39b-4b48-4a9f-beca-0f81f98fd112%2FUntitled.png?table=block&id=0b651fd7-e9fb-4dc6-b13b-ed7f41a0c465&spaceId=9bc99f35-edf4-4599-af44-3f7fb94b56cd&width=2000&userId=&cache=v2)

### IMAGE

컨테이너를 생성할 때 사용된 이미지의 이름입니다.

### COMMAND

COMMAND는 컨테이너가 시작될 떄 실행할 명령어 커맨드는 대부분의 이미지에 미리 내장돼있기 때문에 별도로 설정할 필요는 없습니다. 위에서 생성한 centos:7, ubuntu:18.04 이미지에는 /bin/bash라는 커맨드가 내장되어 있지 때문에 컨테이너를 생성할 때 별도의 커맨드를 설정하지 않았습니다. 컨테이너가 시작될 때 /bin/bash 명령이 실행됐으므로 상호 입출력이 가능한 쉘 환경을 사용할 수 있었습니다.

이미지에 내장된 커맨드는 docker run이나 create 명령어의 맨 끝에 입력해서 컨테이너를 생성할 때 덮어쓸 수 있습니다. 예를 들어, 아래의 docker run 명령어로 생성되는 컨테이너는 실행될 때마다 echo hello world를 실행합니다.

```bash
sudo docker run -it ubuntu:18.04 echo "hello world!"
```

![Untitled](https://island-primula-917.notion.site/image/https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2Fd8e62bf4-b7d1-4dec-a166-cd7b73bd9c8d%2FUntitled.png?table=block&id=131485db-7120-49c6-b92f-0b729b8828ae&spaceId=9bc99f35-edf4-4599-af44-3f7fb94b56cd&width=2000&userId=&cache=v2)

그러나 위 명령으로 생성된 컨테이너는 ubuntu:18.04에 내장된 /bin/bash command를 덮어쓰기 때문에 상호입출력이 가능한 쉘이 실행되지 않아 "hello world!"만 실행되고 컨테이너 종료

### NAME

컨테이너의 고유한 이름입니다. 컨테이너를 생성할 때 —name 옵션으로 이름을 설정하지 않은 경우 도커엔진이 임의로 형용사와 명사를 무작위로 조합해 이름을 설정하기 때문에 ubuntu 컨테이너의 이름이 예시에서는 sweet_pascal로 설정되어 있습니다. ID와 마찬가지로 고유해야 하며 중복되면 안되지만 docker rename 명령으로 변경은 가능합니다.

```bash
sudo docker rename jolly_benz my_ubuntu1804
```

![Untitled](/images/container-tutorial/1.png)

# 컨테이너 삭제

더 이상 사용하지 않는 컨테이너를 삭제할 때는 docker rm 명령어 사용

한번 삭제한 컨테이너는 복구할 수 없으므로 삭제할 때는 신중을 기해야 합니다.

그러나 위에서 생성한 CentOS, Ubuntu는 연습용이므로 지워도 문제는 없습니다.

다음 명령을 통해 컨테이너를 삭제합니다.

```bash
sudo docker rm sweet_pascal
```

컨테이너가 삭제됐는지 확인하려면 docker ps -a 명령을 입력합니다.

```bash
sudo docker ps -a
```

![Untitled](/images/container-tutorial/2.png)

이번에는 mycentos 컨테이너를 삭제합니다.

```bash
sudo docker rm mycentos
```

![Untitled](/images/container-tutorial/3.png)

에러가 났네요?!

실행 중인 컨테이너는 삭제할 수 없으므로

1. 컨테이너를 정지 후 삭제
    
    ```bash
    sudo docker stop mycentos
    sudo docker rm mycentos
    ```
    
2. 실행되던지 말던지 강제 삭제 옵션
    
    ```bash
    sudo docker rm -f mycentos
    ```
    

![Untitled](/images/container-tutorial/4.png)

돌리고 던져놓은 컨테이너가 너무나 많아 일일히 삭제하기가 귀찮다면

```bash
sudo docker container prune
```

진짜로 다 지울거냐를 물어봅니다.

![Untitled](/images/container-tutorial/5.png)

y를 입력하면 다 지워집니다.

![Untitled](/images/container-tutorial/6.png)

docker ps 명령의 -a와 -q 옵션을 조합해 삭제

-q 옵션 : ID만 출력

```bash
sudo docker ps -a -q
```

```bash
sudo docker stop $(sudo docker ps -a -q)
sudo docker rm $(sudo docker ps -a -q)
```

# 컨테이너를 외부에 노출

컨테이너는 가상 머신과 마찬가지로 가상 IP주소를 할당받습니다.

기본적으로 docker 는 컨테이너에 172.17.0.x의 IP를 순차적으로 할당합니다.

컨테이너를 새롭게 생성한 후 ifconfig 명령어로 컨테이너의 네트워크 인터페이스를 확인합니다.

```bash
sudo docker run -it --name network_test ubuntu:20.04
```

```bash
root@6f0c1e0f544b:/# apt update
root@6f0c1e0f544b:/# apt install net-tools
root@6f0c1e0f544b:/# ifconfig
```

![Untitled](/images/container-tutorial/7.png)

docker의 NAT IP인 172.17.0.2를 할당받은 eth0 인터페이스와 로컬 호스트인 lo인터페이스가 있습니다.

아무런 설정을 하지 않았다면 이 컨테이너는 외부에서 접근이 불가능 - 해당 docker 가 설치된 host만 가능

외부에 컨테이너의 에플리케이션을 노출하기 위해서는 eth0의 IP와 port를 호스트의 IP와 port에 바인딩해야 합니다.

컨테이너에서 호스트로 빠져나온 뒤 다음 명령어를 입력해 컨테이너를 생성합니다. 이 컨테이너에 nginx 웹 서버를 설치해 외부에 노출할 것입니다.

```bash
sudo docker run -it --name mywebserver -p 80:80  ubuntu:20.04 
```

-p 옵션 ; 컨테이너의 포트를 호스트의 포트와 바인딩해 연결할 수 있게 설정

<aside> 💡 -p 옵션을 통해 port forwarding 하는 법 [host port]:[container port] 예 : 7777:80 ⇒ 호스트의 7777포트와 80포트

호스트의 특정 ip를 사용해서 port forwarding 가능 예 : 192.168.0.100:7777:80

port forwarding을 여러개 하고 싶습니다. ⇒ -p 옵션을 여러개 쓰세요 예 : sudo docker run -it -p 3306:3306 -p 192.168.0.100:7777:80 ubuntu:20.04

</aside>

```bash
root@1f22a7d7162d:/# apt update
root@1f22a7d7162d:/# apt install nginx
root@1f22a7d7162d:/# service nginx start
root@1f22a7d7162d:/# service nginx status
```

![Untitled](/images/container-tutorial/8.png)

GCP compute engine vm의 외부 ip로 접속하면 아래와 같은 화면이 보입니다.

![Untitled](/images/container-tutorial/9.png)

그런데 이런식으로 Docker를 쓰면 Docker를 쓰는 의미가 딱히 없죠?

이번엔 apache 웹서버 이미지를 바로 실행해보겠습니다.

```bash
sudo docker run --name my-apache-server -p 8080:80 httpd
```

![Untitled](/images/container-tutorial/10.png)

이제 8080포트로 접속해보겠습니다.

![Untitled](/images/container-tutorial/11.png)

엇 접속이 안됩니다. ㅠㅠ

- 왜 접속이 안될까요?
    
    gcp compute Engine vm의 방화벽을 열지 않았습니다.
    
    다음과 같이 방화벽을 엽니다.
    
    1. 방화벽 메뉴로 들어갑니다.
        
        
    2. 방화벽 규칙 만들기 버튼을 누릅니다.
        
        
    3. 방화벽 규칙의 정보들을 입력한 후 만들기 버튼을 누릅니다.
        
        - 이름 : default-allow-http-8080
        - 설명 : 8080포트의 접근을 허용합니다.
        - 대상 : 네트워크의 모든 인스턴스
        - 소스 IPv4 범위 : 0.0.0.0/0 (모두)
        - 프로토콜 및 포트 : TCP - 8080
        

접속이 되는 것을 볼 수 있습니다.

![Untitled](/images/container-tutorial/12.png)

호스트의 IP의 port를 컨테이너의 IP와 port로 연결한다는 개념은 매우 중요합니다.

아파치 웹서버는 172 대역을 가진 컨테이너의 NAT IP의 80번 포트로 서비스하므로 여기에 접근하려면 172.17.0.x:80의 주소로 접근해야 합니다.

그러나 docker의 port-forward 옵션인 -p를 써서 호스트와 컨테이너를 연결했으므로 호스트의 IP와 포트를 통해 172.17.0.x.:80으로 접근할 수 있습니다.

순서를 정리하면 다름과 같습니다.

![Untitled](/images/container-tutorial/13.png)

그러나 다음같이 -p 80:81로 입력했다면 외부에서 웹 서버에 접근하지 못합니다.

호스트의 80번 포트가 forward하는 컨테이너의 포트는 81이고 81에서는 아무일도 일어나지 않습니다.

# 컨테이너 application 구축

대부분의 서비스는 단일 프로그램으로 동작하지 않습니다.

특히나 요즘처럼 MSA의 중요성을 이야기하는 시대에는 더욱 단일 프로그램으로 동작하는 서비스를 찾기 힘듭니다.

최소 DB정도는 분리하게 되죠

이런 서비스를 containerize할 때 여러 개의 애플리케이션을 한 컨테이너에 설치할 수도 있습니다.

그러나 컨테이너에 애플리케이션을 하나만 동작시키면 컨테이너 간의 독립성을 보장함과 동시에 애플리케이션의 버전 관리, 소스코드 모듈화 등이 더욱 쉬워집니다.

이 구조는 docker community와 docker 공식 문서에서도 구너장하는 구조입니다.

docker는 one container, one process가 그들의 철학이기 떄문이죠

다음은 웹 서버와 DB의 예시입니다.

1. 먼저 DB 컨테이너를 생성합니다.
    
    ```bash
    sudo docker run -d --name wordpressdb \\
     -e MYSQL_ROOT_PASSWORD=password \\
     -e MYSQL_DATABASE=wordpress \\
    mysql:5.7
    ```
    
2. 그 다음 워드프레스 컨테이너를 생성합니다.
    
    ```bash
    sudo docker run -d --name wordpress \\
    -e WORDPRESS_DB_HOST=mysql \\
    -e WORDPRESS_DB_USER=root \\
    -e WORDPRESS_DB_PASSWORD=password \\
    --link wordpressdb:mysql \\
    -p 80 wordpress
    
    ```
    

docker ps 명령 결과의 PORTS 컬럼을 확인해보겠습니다.

```bash
sudo docker ps
```

![Untitled](/images/container-tutorial/14.png)

이 예제에서는 호스트의 32768번 포트와 연결했습니다. 따라서 호스트이 IP와 32768번 포트로 워드프레스 웹 서버에 접근할 수 있습니다. 웹 브라우저로 [호스트 IP]:32768에 접근했을 때 다음과 같은 화면이 나타나면 워드프레스 컨테이너가 성공적으로 생성된 것입니다.

![Untitled](/images/container-tutorial/15.png)

### sudo docker run의 -d , -e, —link 옵션

-d 옵션

- -i -t 옵션으로 run을 실행하면 표준 입출력이 활성화된 상호작용이 가능한 쉘 환경을 사용할 수 있음
- 하지만 -d 옵션으로 run을 실행하면 입출력이 없는 상태로 컨테이너를 실행
- 컨테이너 내부에서는 프로그램이 터미널을 차지하는 foreground로 실행돼 사용자의 입력을 받지 않음
- detached 모드인 컨테이너는 반드시 컨테이너에서 프로그램이 실행돼야 하며, foreground 프로그램이 실행되지 않으면 컨테이너 종료
- mysql은 하나의 터미널을 차지하는 mysqld, wordpress는 하나의 터미널을 차지하는 apache2-foreground를 실행하므로 -d 옵션을 지정해 호스트에서는 background로 설정

-e 옵션

- 컨테이너 내부의 환경변수를 설정
    
- containerized application은 환경변수에서 값을 가져와 쓰는 경우가 많으므로 자주 사용하는 옵션 중 하나입니다. mysql 컨테이너를 생성할 떄 설정한 -e 옵션의 값을 살펴보면 mysql 컨테이너의 환경변수로 어떤 것이 설정됐는지 알 수 있습니다.
    
- mysql 환경변수
    
    ```bash
    -e MYSQL_ROOT_PASSWORD=password
    -e MYSQL_DATABASE=wordpress
    ```
    
- 환경변수 확인
    
    리눅스에서 환경변수를 확인하는 가장 간단한 방법 : echo
    
    ```bash
    echo ${ENV_NAME}
    ```
    
    이걸 확인하려면 컨테이너의 쉘을 사용해야 하는데 mysql과 wordpress는 attach 명령은 의미가 없습니다.
    
    attach 쓰면 그냥 log만 나옵니다.
    
  ![Untitled](/images/container-tutorial/16.png)
    
    그래서 exec 명령으로 컨테이너 내부의 bash쉘을 실행시켜야 합니다.
    
    ```bash
    sudo docker exec -it wordpressdb /bin/bash
    ```
    
  ![Untitled](/images/container-tutorial/17.png)
    

—link 옵션

- 컨테이너 내부 IP를 알 필요 없이 항상 컨테이너에 alias로 접근하도록 설정
    
- 위에서 생성한 워드프레스 웹 서버 컨테이너는 —link 옵션의 값에서 wordpressdb 컨테이너를 mysql이라는 이름으로 설정했습니다.
    
    ```bash
    --link wordpressdb:mysql
    ```
    
    - 워드프레스 웹 서버 컨테이너는 wordpressdb의 IP를 몰라도 mysql이라는 호스트명으로 접근할 수 있게 됨
        
    - 다음 명령으로 확인
        
        ```bash
        sudo docker exec wordpress curl mysql:3306
        ```
        
- 실행 순서의 의존성도 정의
    
    wordpress, wordpressdb 모두 컨테이너 중지를 합니다.
    
    ```bash
    sudo docker stop wordpress wordpressdb
    ```
    
    ![Untitled](/images/container-tutorial/18.png)
    
    그리고 wordpress를 다시 실행시키면
    
    ```bash
    sudo docker start wordpress
    ```
    
  ![Untitled](/images/container-tutorial/19.png)
    
    에러가 납니다.
    
- 컨테이너 간에 이름으로 서로를 찾을 수 있게 도와주지만 deprecated 된 옵션
    
- docker bridge network를 사용하면 —link 옵션과 동일한 기능을 더욱 손쉽게 사용할 수 있으므로 bridge network를 사용하는 것을 권장
    

# 도커 볼륨

도커 이미지로 컨태이너를 생성하면 이미지는 readonly이 되며 컨테이너의 변경 사항만 별도로 저장해서 각 컨테이너의 정보를 보존

위에서 생성했던 mysql 컨테이너는 mysql:5.7이라는 이미지로 생성됐지만 워드프레스 블로그를 위한 DB등의 정보는 컨테이너가 갖고 있습니다.

![Untitled](/images/container-tutorial/20.png)

이미 생성된 이미지는 어떠한 경우로도 변경되지 않으며, 컨테이너 계층에 원래 이미지에서 변경된 파일시스템 등을 저장합니다. 이미지에 mysql을 실행하는 데 필요한 애플리케이션 파일이 들어있다면 컨테이너 계층에는 워드프레스에서 쓴 로그인 정보나 게시글 등과 같이 데이터베이스를 운용하면서 쌓이는 데이터가 저장

### 컨테이너 레이어의 치명적 문제

이미지로 빌드시 너무나 많은 변경사항이 일어났기 때문에 레이어가 과도하게 쌓임

mysql 컨테이너를 삭제하면 컨테이너 계층에 저장돼있던 데이터베이스의 정보도 삭제

도커의 컨테이너는 생성과 삭제가 매우 쉬우므로 실수로 컨테이너를 삭제하면 데이터를 복구할 수 없음

이를 방지하기 위해 컨테이너의 데이터를 영속적 데이터로 활용할 수 있는 방법

⇒ **볼륨을 활용하는 것**

1. 호스트와 볼륨을 공유
2. 볼륨 컨테이너를 활용
3. 도커과 관리하는 볼륨을 생성

## 호스트 볼륨 공유

다음 명령어를 입력해 mysql 컨테이너와 워드프레스 웹서버 컨테이너 생성

```bash
sudo docker run -d \\
--name wordpressdb_hostvolume \\
-e MYSQL_ROOT_PASSWORD=password \\
-e MYSQL_DATABASE=wordpress \\
-v ~/wordpress_db:/var/lib/mysql \\
mysql:5.7
```

```bash
sudo docker run -d \\
-e WORDPRESS_DB_PASSWORD=password \\
--name wordpress_hostvolume \\
--link wordpressdb_hostvolume:mysql \\
-p 80 \\
wordpress
```

워드프레스 컨테이너에 -p 옵션으로 컨테이너의 80포트를 외부에 노출했으므로 docker ps명령어에서 확인한 wordpress_hostvolume 컨테이너의 호스트 포트로 워드프레스 컨테이너에 접속

-v 옵션

```bash
-v ~/wordpress_db:/var/lib/mysql
-v [호스트 공유 디렉토리]:[컨테이너 공유 디렉토리]
```

호스트의 ~/wordpress_db와 컨테이너의 /var/lib/mysql를 공유

미리 ~/wordpress_db 디렉토리를 호스트에 생성하지 않았어도 도커가 자동 생성

컨테이너를 삭제해서 실제 DB의 데이터가 보존되는지 확인

```bash
sudo docker stop wordpressdb_hostvolume wordpress_hostvolume
sudo docker rm wordpressdb_hostvolume wordpress_hostvolume
```

다시 ~/wordpress_db 디렉토리를 확인

```bash
ls ~/wordpress_db/
```

![Untitled](/images/container-tutorial/21.png)

~/wordpress_db:/var/lib/mysql 이 둘은 동기화 되는 개념이 아니라 그냥 같은 디렉토리

![Untitled](/images/container-tutorial/22.png)

디렉토리 뿐만 아니라 단일 파일도 가능, 여러개의 -v 옵션도

```bash
echo hello >> ~/hello && echo hello2 >> ~/hello2
sudo docker run -it --name file_volume \\
-v ~/hello:/hello \\
-v ~/hello2:/hello2 \\
ubuntu:20.04
```

```bash
root@2b574c6896cc:/# cat hello && cat hello2
```

![Untitled](/images/container-tutorial/23.png)

원래 호스트에는 ~/wordpress_db 디렉토리가 존재하지 않았음

-v 옵션을 사용함으로써 호스트에 ~/wordpress_db 디렉토리 생성, 파일 공유

⇒ 컨테이너의 파일이 호스트로 복사

호스트와 컨테이너 양쪽 모두에 동일한 파일이 있을 경우

1. alicek106/volume_test이미지의 /home/testdir_2 디렉토리 내용 확인
    
    ```bash
    sudo docker run -it --name volume_dummy \\
    alicek106/volume_test \\
    ls /home/testdir_2/
    ```
    
    ![Untitled](/images/container-tutorial/24.png)
    
2. 이제 -v 옵션을 사용해서 컨테이너를 생성
    
    ```bash
    sudo docker run -it --name volume_overide \\
    -v ~/wordpress_db:/home/testdir_2 \\
    alicek106/volume_test \\
    ls /home/testdir_2/
    ```
    
   ![Untitled](/images/container-tutorial/25.png)
    

공유한 컨테이너 디렉토리자체가 호스트의 디렉토리 내용으로 덮어씌워짐

⇒ 호스트의 디렉토리가 컨테이너의 디렉토리를 마운트포인트로 마운트

## 볼륨 컨테이너

-v 옵션으로 볼륨을 사용하는 컨테이너를 다른 컨테이너와 공유하는 것

컨테이너를 생성할때 —volumes-from 옵션을 설정하면 -v 또는 —volume 옵션을 적용한 컨테이너의 볼륨 디렉토리를 공유

```bash
sudo docker run -it \\
--name volumes_from_container \\
--volumes-from volume_overide \\
ubuntu:20.04 \\
ls /home/testdir_2
```

![Untitled](/images/container-tutorial/26.png)

![Untitled](/images/container-tutorial/27.png)

## 도커볼륨

도커 자체에서 제공하는 볼륨 기능을 활용해 데이터를 보존

볼륨을 다루는 명령어는 docker volume

### 생성

다음 명령으로 도커 볼륨을 생성합니다.

```bash
sudo docker volume create --name myvolume
```

![Untitled](/images/container-tutorial/28.png)

플러그인 드라이버를 설정해 여러 종류의 스토리지 백엔드를 쓸 수 있지만 여기서는 기본적으로 제공되는 드라이버인 local을 사용

이 볼륨은 로컬호스트에 저장되며 도커엔진에 의해 생성되고 삭제

### 해당 볼륨을 쓰는 컨테이너 생성

여기서 -v 옵션에는 [볼륨이름]:[컨테이너 공유 디렉토리] 이런식으로 입력합니다.

```bash
sudo docker run -it --name myvolume_1 \\
-v myvolume:/root/ \\
ubuntu:20.04 
```

해당 볼륨에 파일을 하나 씁니다.

```bash
root@c54946a5664c:/# echo hello, volume! >> /root/volume
```

그 다음 같은 볼륨을 마운트하는 컨테이너를 하나 더 만들어서 확인해봅니다.

```bash
sudo docker run -it --name myvolume_2 \\
-v myvolume:/root/ \\
ubuntu:20.04 \\
cat /root/volume
```

![Untitled](/images/container-tutorial/29.png)

![Untitled](/images/container-tutorial/30.png)

볼륨은 디렉토리 하나에 상응하는 단위로서 docker engine에서 관리

도커 볼륨도 호스트 볼륨 공유와 마찬가지로 호스트에 저장함으로써 데이터를 보존하지만 파일이 실제로 어디에 저장되는지 사용자는 알 필요가 없습니다.

### 볼륨 세부정보

다음 명령으로 볼륨이 실제로 어디에 저장되었는지를 알 수 있습니다.

```bash
sudo docker inspect --type volume myvolume
```

![Untitled](/images/container-tutorial/31.png)

해당 디렉토리를 뒤져보도록 하겠습니다.

```bash
sudo ls /mnt/storage/docker/volumes/myvolume/_data
```

![Untitled](/images/container-tutorial/32.png)

### 볼륨 자동 생성

-v 옵션에서 공유할 디렉토리 위치를만 입력하면 해당 디렉토리에 대한 볼륨을 자동으로 생성

```bash
sudo docker run -it --name volume_auto \\
-v /root \\
ubuntu:20.04
```

```bash
root@ada4230050e3:/# exit
```

```bash
sudo docker volume ls
```

![Untitled](/images/container-tutorial/33.png)

어떤 볼륨에 마운트 되었는지 확인하기 위해 docker inspect로 확인

```bash
sudo docker inspect
```

![Untitled](/images/container-tutorial/34.png)

![Untitled](/images/container-tutorial/35.png)

### 불필요한 volume 삭제

컨테이너는 삭제해도 볼륨은 남아있기 때문에 불필요한 볼륨은 지워줘야 함

사용하지 않는 볼륨은 한번에 지우는 명령은 다음과 같음

```bash
sudo docker volume prune
```

![Untitled](/images/container-tutorial/36.png)

### mount 옵션

-v 옵션 대신 사용 기능은 같지만 옵션주는 방식이 다름

```bash
sudo docker run -it --name mount_option1 \\
--mount type=volume,source=myvolume,target=/root \\
ubuntu:20.04 \\
cat /root/volume
```

![Untitled](/images/container-tutorial/37.png)

# 도커 네트워크

## 도커 네트워크 구조

- 도커는 컨테이너에 내부 IP를 순차적으로 할당하며, 이 IP는 컨테이너를 재시작할 때마다 변경될 수 있음
- 이 내부 IP는 도커가 설치된 호스트 ⇒ 내부 망에서만 쓸 수 있는 IP이므로 외부와 연결될 필요가 있음
- 이 과정은 컨테이너를 시작할 때마다 호스트에 veth…라는 네트워크 인터페이스를 생성함으로써 이뤄짐
- 각 컨테이너에 외부와의 네트워크를 제공하기 위해 가상 네트워크 인터페이스를 호스트에 생성 이름은 veth로 시작
- veth 인터페이스는 사용자가 직접 생성할 필요는 없으며 컨테이너가 생성될 때 도커 엔진이 자동으로 생성

```bash
# ifconfig가 없을경우 sudo apt install net-tools
ifconfig
```

![Untitled](/images/container-tutorial/38.png)

ens4 : GCP의 내부 IP 할당 - 실제로 외부와 통신이 가능한 호스트의 네트워크 인터페이스

veth… : 컨테이너를 시작할 때 생성, 각 컨테이너의 eth0와 연결

docker0 : bridge, 각 veth 인터페이스와 바인딩, 호스트의 ens4 인터페이스와 이어주는 역할

![Untitled](/images/container-tutorial/39.png)

실제로 바인딩됐는지 확인

```bash
# apt install bridge-utils
brctl show docker0
```

![Untitled](/images/container-tutorial/40.png)

## 도커 네트워크 기능

- 기본적으로 docker0 bridge로 외부와 통신할 수 있는 환경 사용
- 사용자의 선택에 따라 어려 네트워크 드라이버 사용 가능
- 자체 제공 드라이버 : bridge, host, none, container, overlay

네트워크 목록 확인

![Untitled](/images/container-tutorial/41.png)

### bridge 네트워크

docker network inspect옵션으로 bridge 세부 정보 확인

```bash
sudo docker network inspect bridge
```

![Untitled](/images/container-tutorial/42.png)

subnet : 172.17.0.0/16

geteway : 172.17.0.1

docker0가 아닌 새로운 bridge 타입의 네트워크 생성

```bash
sudo docker network create --driver bridge mybridge
sudo docker run --name mynetwork_container \\
--net mybridge \\
ubuntu:20.04
ifconfig
```

![Untitled](/images/container-tutorial/43.png)

disconnect/connect를 통해 컨테이너에 유동적으로 붙이고 뗄 수 있음

단 특정 IP대역을 갖는 네트워크 모드에만 사용가능(bridge, overlay / host, none은 사용

![Untitled](/images/container-tutorial/44.png)

![Untitled](/images/container-tutorial/45.png)

![Untitled](/images/container-tutorial/46.png)

```bash
sudo docker network disconnect mybridge mynetwork_container
```

![Untitled](/images/container-tutorial/47.png)

![Untitled](/images/container-tutorial/48.png)

![Untitled](/images/container-tutorial/49.png)

```bash
sudo docker network connect mybridge mynetwork_container
```

![Untitled](/images/container-tutorial/50.png)

![Untitled](/images/container-tutorial/51.png)

subnet, gateway, ip range 옵션 추가

```bash
sudo docker network create --driver=bridge \\
--subnet=172.72.0.0/16 \\
--ip-range=172.72.0.0/24 \\
--gateway=172.72.0.1 \\
my_custom_network
```

![Untitled](/images/container-tutorial/52.png)

### host 네트워크

네트워크를 호스트로 하면 호스트의 네트워크 환경을 그대로 쓸 수 있음

별도 생성 필요 없이 기존 host라는 이름의 네트워크 사용

```bash
sudo docker run -it --name network_host \\
--net host \\
ubuntu:20.04
```

```bash
root@instance-node-1:/# apt update
root@instance-node-1:/# apt install net-tools
root@instance-node-1:/# ifconfig
```

![Untitled](/images/container-tutorial/53.png)

### none 네트워크

아무런 네트워크를 쓰지 않는다는 것을 의미함

외부와의 단절

```bash
sudo docker run -it --name network_none \\
--net none \\
ubuntu:14.04
```

```bash
root@9a20605c7941:/# ifconfig
```

![Untitled](/images/container-tutorial/54.png)

로컬호스트를 나타내는 lo 밖에 네트워크 인터페이스가 없음

### container 네트워크

—net 옵션으로 container를 입력하면 다른 컨테이너의 네트워크 네임스페이스 환경을 공유할 수 있음

```bash
sudo docker run -itd --name network_container_1 ubuntu:14.04
sudo docker run -itd --name network_container_2 \\
--net container:network_container_1 \\
ubuntu:14.04
```

내부 IP를 새로 할당받지도 않으며 호스트에 veth로 시작하는 가상 네트워크 인터페이스도 생성되지 않음

network_container_2에 대한 모든 네트워크 설정은 network_container_1와 동일

```bash
sudo docker exec network_container_1 ifconfig
sudo docker exec network_container_2 ifconfig
```

![Untitled](/images/container-tutorial/55.png)

![Untitled](/images/container-tutorial/56.png)

전부 같음

![Untitled](/images/container-tutorial/57.png)

### 브리지 네트워크와 —net-alias

특정 호스트 이름으로 컨테이너 여러개이 접근 가능

```bash
sudo docker run -itd --name network_alias_container1 \\
--net mybridge \\
--net-alias lotte \\
ubuntu:14.04

sudo docker run -itd --name network_alias_container2 \\
--net mybridge \\
--net-alias lotte \\
ubuntu:14.04

sudo docker run -itd --name network_alias_container3 \\
--net mybridge \\
--net-alias lotte \\
ubuntu:14.04
```

![Untitled](/images/container-tutorial/58.png)
해당 호스트명으로 핑을 날려보겠습니다.

```bash
sudo docker run -it --name network_alias_ping \\
--net mybridge \\
ubuntu:14.04
```

```bash
root@3c8727c3f274:/# ping -c 1 lotte
```

![Untitled](/images/container-tutorial/59.png)

컨테이너 3개의 IP로 각각 PING이 전송된 것을 확인

round robin 방식

도커 내장 DNS가 host 이름을 —net-alias 옵션으로 lotte를 설정한 컨테이너로 변환(resolve)하기 때문

도메인 이름에 대응하는 IP조회

```bash
root@3c8727c3f274:/# apt update
root@3c8727c3f274:/# apt install dnsutils
root@3c8727c3f274:/# dig lotte
```

![Untitled](/images/container-tutorial/60.png)

# 컨테이너 로깅

## json-file 로그 사용

컨테이너의 표준 출력과 에러로그를 별도의 메타데이터 파일로 저장하며 이를 확인하는 명령어를 제공

```bash
sudo docker run -d --name mysql \\
-e MYSQL_ROOT_PASSWORD=1234 \\
mysql:5.7
```

```bash
sudo docker logs mysql
```

![Untitled](/images/container-tutorial/61.png)

-since : 특정시간 이후의 로그 확인 가능

-f : 로그를 스트림으로 확인

-t : 타임스탬프 표시

## syslog

json 뿐만 아니라 syslog로 보내 저장하도록 설정

```bash
sudo docker run -d --name syslog_container \\
--log-driver=syslog \\
ubuntu:14.04 \\
echo syslogtest
```

```bash
sudo tail /var/log/syslog
```

![Untitled](/images/container-tutorial/62.png)

# 컨테이너 자원 할당 제한

## 컨테이너 메모리 제한

—memory 옵션 사용

최소 제한 4MB

```bash
sudo docker run -d \\
--memory="1g" \\
--name memory_1g \\
nginx
```

```bash
sudo docker inspect memory_1g |grep \\"Memory\\"
```

![Untitled](/images/container-tutorial/63.png)

## 컨테이너 cpu 제한

—cpu-shares 옵션 사용

CPU를 어느 비중만큼 나눠가질지에 대한 상대적인 비중 (디폴트 1024)

```bash
sudo docker run -d --name cpu_1024 \\
--cpu-shares 1024 \\
alicek106/stress \\
stress --cpu 1
```

```bash
ps aux | grep stress
```

![Untitled](/images/container-tutorial/64.png)

—cpuset-cpus 호스트에 cpu가 여러개 있을 때 특정 cpu만 사용하도록 설정

cpu 집중적 작업이 필요하다면 여러개의 cpu를 사용하도록 설정해 적절히 분배하는 것이 좋음

```bash
sudo docker run -d --name cpuset_1 \\
--cpuset-cpus=1 \\
alicek106/stress \\
stress --cpu 1
```

![Untitled](/images/container-tutorial/65.png)

![Untitled](/images/container-tutorial/66.png)

—cpu-period : cpu주기 (기본 100000) —cpu-quota : period 중 cpu 스케줄링에 얼마나 할당할건지

⇒ —cpus : —cpu-period, —cpu-quota와 기능적으로는 같음 하지만 좀 더 직관적 예) —cpu-period=100000 , —cpu-quota=50000 ⇒ —cpus=0.5

```bash
sudo docker run -d --name cpus_container \\
--cpus=0.5 \\
alicek106/stress \\
stress --cpu 1
```

![Untitled](/images/container-tutorial/67.png)

## 컨테이너 Block I/O 제한

하나의 컨테이너가 입출력을 과도하게 사용하지 않게 설정하기 위해

—device-write-bps, —device-read-bps

—device-write-iops, —device-read-iops