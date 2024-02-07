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
