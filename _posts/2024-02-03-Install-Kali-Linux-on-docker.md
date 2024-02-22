---
title: Docker Kali Linux 설치
categories: [ETC., Tools]
tags: [Docker, Kali]
image:
    path: /assets/image_post/20240207111627.png
---

MacOS 환경에 Kali Linux 테스트 환경이 필요하여 Docker에 Kali linux 구동을 목표로 한다.
Docker의 경우 각 환경에 맞게 설치가 필요하며, MacOS의 경우 아래의 경로를 통해 설치하면 된다.
> 설치 경로: https://docs.docker.com/desktop/install/mac-install/

Docker 설치 후 Kali linux image를 Pull해서 갖고 온다.
``` bash
docker pull kalilinux/kali-rolling
```
이후 shell에 접속하기 위해 아래 명령어를 실행하면 Kali Linux 컨테이너에 연결 된다.
``` bash
docker run -it kalilinux/kali-rolling:/bin/sh
```

접속 후 필요한 도구 설치를 위해 아래와 같이 설치가 필요하다.
``` shell
# apt update
# apt upgrade
# apt install kali-tools-top10
```
하지만 원격 연결이 끊어지거나 재시작 할 경우 컨테이너 특성 상 다시 필요한 패키지 설치가 필요하다.

### 추가 2024-02-22
아래 링크를 보니 docker 실행 후 변경된 사항을 commit하여 저장할 수 있는 듯 하다.
> https://thenewstack.io/penetration-testing-with-kali-linux-as-a-docker-container/

해당 링크에서 추천하는 바는 아래와 같다.
``` bash
# docker run -ti kalilinux/kali-rolling /bin/bash
# apt update
# apt install kali-linux-headless -y
```

이후 docker ps로 id 4글자를 확인하여 저장하는 방식이다.
``` bash
docker ps
docker commit ID kalitools
```

저장된 이미지에 대해 확인 후 실행하면 된다.
``` bash
docker images
docker run -it kalitools /bin/bash
```

아래는 kali linux 도커 이미지를 실행 후 1.txt란 파일을 만들어 commit한 내용이다.
``` bash
# 1.txt 생성
┌──(root㉿docker-desktop)-[/]
└─# echo 1 >> 1.txt

┌──(root㉿docker-desktop)-[/]
└─# ls
1.txt  bin  boot  dev  etc  home  lib  lib64  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var

```
commit은 아래와 같이 진행한다.
``` bash
# id 확인
 user-mac  ~  docker ps
CONTAINER ID   IMAGE                    COMMAND       CREATED          STATUS          PORTS     NAMES
f4d562115dbb   kalilinux/kali-rolling   "/bin/bash"   58 seconds ago   Up 58 seconds             optimistic_davinci

# commit 
 user-mac  ~  docker commit f4d562115dbb kalitools
sha256:274608c3309f9039813cf9b619a010425bbca45650431ae6409aafb62f829240

# 생성 확인 
 user-mac  ~  docker images
REPOSITORY               TAG       IMAGE ID       CREATED         SIZE
kalitools                latest    274608c3309f   5 seconds ago   127MB
kalilinux/kali-rolling   latest    20d1b14b7bb9   4 days ago      127MB

# 이미지 정보 확인
 user-mac  ~  docker image inspect kalitools
[
    {
        "Id": "sha256:274608c3309f9039813cf9b619a010425bbca45650431ae6409aafb62f829240",
        "RepoTags": [
            "kalitools:latest"
        ],
        "RepoDigests": [],
        "Parent": "sha256:20d1b14b7bb9e5bb871e22b989a3b49485071e7ba0b43161c238f3dd64555107",
        "Comment": "",
        "Created": "2024-02-22T06:40:26.326540151Z",

```

아래와 같이 새로 생성한 이미지에 접속하여도 1.txt가 유지되는 것을 확인할 수 있다.
``` bash
user-mac  ~  docker run -it kalitools /bin/bash

┌──(root㉿61266a28d727)-[/]
└─# ls
1.txt  bin  boot  dev  etc  home  lib  lib64  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var
```