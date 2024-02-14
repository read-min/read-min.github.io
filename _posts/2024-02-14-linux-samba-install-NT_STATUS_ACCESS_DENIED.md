---
title: Linux Samba 설치 오류 (NT_STATUS_ACCESS_DENIED)
categories: [ETC., Tool]
tags: [Samba, SMB]
image:
    path: /assets/image_post/20240214181043.png
---

kali linux 환경에 samba를 설치하여 rfi를 테스트하고자 하였으나, 별 것도 아닌 에러 문제로 고생한 내역이 있어 저장해놓고자 한다. 우선 `apt install samba` 등을 통해 samba를 설치했다면 아래와 같이 공유하고자 하는 경로와 이름을 지정해주어야 한다. 나의 경우 'rfi' 란 이름의 '/home/user/rfi'를 지정하였고, 게스트 접속을 허용 했다. 
``` bash
┌──(root㉿kali)-[/home/user]
└─# tail -n 6 /etc/samba/smb.conf
[rfi]
   comment = rfi
   path = /home/user/rfi
   browseable = yes
   guest ok = yes
```

위와 같이 설정 후 `systemctl restart smbd`를 통해 변경 사항 적용 후 smbclient 를 통해 스캔 하면 아래와 같이 rfi가 나타나는 것을 알 수 있다.
``` bash
┌──(root㉿kali)-[/home/user]
└─# smbclient -L localhost -N
Anonymous login successful

        Sharename       Type      Comment
        ---------       ----      -------
        print$          Disk      Printer Drivers
        rfi             Disk      rfi
        IPC$            IPC       IPC Service (Samba 4.19.4-Debian)
        nobody          Disk      Home Directories
Reconnecting with SMB1 for workgroup listing.
smbXcli_negprot_smb1_done: No compatible protocol selected by server.
Protocol negotiation to server localhost (for a protocol between NT1 and NT1) failed: NT_STATUS_INVALID_NETWORK_RESPONSE
Unable to connect with SMB1 -- no workgroup available
```

하지만 실제 접근 시 아래와 같이 `NT_STATUS_ACCESS_DENIED` 에러가 나온다.
``` bash
┌──(root㉿kali)-[/home/user]
└─# smbclient //localhost/rfi -N
Anonymous login successful
Try "help" to get a list of possible commands.
smb: \> ls
NT_STATUS_ACCESS_DENIED listing \*
```

나의 경우 해결 방법은 /home 경로 자체에 권한 부여를 해주어야 했다.
``` bash
┌──(root㉿kali)-[/home/user]
└─# chmod -R 755 /home
```

적용 후 정상적으로 접근이 가능함을 확인
``` bash
┌──(root㉿kali)-[/home/user]
└─# smbclient //localhost/rfi -N
Anonymous login successful
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Wed Feb 14 00:31:01 2024
  ..                                  D        0  Wed Feb 14 00:31:01 2024
  rfi.php                             N       32  Wed Feb 14 00:31:01 2024

                128990364 blocks of size 1024. 108120176 blocks available
```



참고
---
- https://serverfault.com/questions/900440/samba-configuration-statusnt-status-access-denied