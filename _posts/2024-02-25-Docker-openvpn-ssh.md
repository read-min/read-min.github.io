---
title: Docker 컨테이너 ssh/openvpn 연결
categories: [ETC., Tools]
tags: [Docker, openvpn, ssh]
image:
    path: /assets/image_post/20240207111627.png
---

### 요약
``` bash
 read-min 🇰   ~  docker run --privileged --sysctl net.ipv6.conf.all.disable_ipv6=0 -it -p 2222:22 -p 8888:8000 kali-htb /bin/sh
```
---------
현재 환경 상 Docker에 Kali Linux를 구동하여 호스트에서 ssh 접속이 필요하고, 연결 후 openvpn을 활성화해야한다. 이 과정 중에 필요한 설정들이 존재하기에 메모 차 기록한다.

컨테이너 환경에서 openssh 설치 후 /etc/ssh/sshd_config에 아래와 같이 변경 후 `service ssh restart` 하여 적용해야 한다.
``` bash
PermitRootLogin yes
PasswordAuthentication yes
```

하지만 위와 같이 해놓고 끝이 아니라, 컨테이너 실행 시 포트를 연결해주어야 한다. 아래의 경우 외부 2222번과 내부 22번 포트를 연결하는 명령어 이다. 8888과 8000도 테스트 차 연결하였다
``` bash
 -p 2222:22 -p 8888:8000
```

이후 ssh를 연결할 때 아래와 같이 host에서 localhost로 접근을 해야 한다.
``` bash
 read-min 😎   ~  ssh root@localhost -p 2222
root@localhost's password:
Linux 9048af13b510 6.6.12-linuxkit #1 SMP Fri Jan 19 08:53:17 UTC 2024 aarch64

The programs included with the Kali GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Kali GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Sun Feb 25 03:34:08 2024 from 192.168.65.1
┌──(root㉿9048af13b510)-[~]
└─# ls
```

이제 openvpn 연결하는 방법에 대해 알아보자. 실행 된 컨테이너에서 openvpn을 사용하고자 하면 아래와 같이 에러를 맞이하게 된다.
``` bash
# # openvpn starting_point_readmin520.ovpn
2024-02-25 03:39:09 GDG6: remote_host_ipv6=n/a
2024-02-25 03:39:09 net_route_v6_best_gw query: dst ::
2024-02-25 03:39:09 sitnl_send: rtnl: generic error (-101): Network is unreachable
2024-02-25 03:39:09 ROUTE6: default_gateway=UNDEF
2024-02-25 03:39:09 ERROR: Cannot open TUN/TAP dev /dev/net/tun: No such file or directory (errno=2)
2024-02-25 03:39:09 Exiting due to fatal error
```

해결 방안은 컨테이너 실행 시 아래와 같이 인자를 추가해주면 된다. 
``` bash
--sysctl net.ipv6.conf.all.disable_ipv6=0
```

참고 링크는 아래와 같다.
> https://www.reddit.com/r/docker/comments/is5ggw/trouble_using_openvpn_inside_container/