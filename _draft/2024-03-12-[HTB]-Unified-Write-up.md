---
title: HTB Unified Write-up
categories: [HackTheBox, Basic]
tags: [HackTheBox]
image:
    path: /assets/image_post/20240228215601.png
---
![](../assets/image_post/20240228215615.png)

## [0x00] Port Scan
---
port 스캔 결과 22, 6789, 8080, 8443이 열려 있다. 이 중 ibm-db2-admin은 IBM DB2 데이터베이스의 관리자 인터페이스에 대한 포트가 열려 있음을 나타내는 것으로 보인다.
``` bash
┌──(root㉿9048af13b510)-[/htb]
└─# nmap 10.129.147.107 -Pn
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-02-28 12:55 UTC
Nmap scan report for 10.129.147.107
Host is up (0.30s latency).
Not shown: 996 closed tcp ports (reset)
PORT     STATE SERVICE
22/tcp   open  ssh
6789/tcp open  ibm-db2-admin
8080/tcp open  http-proxy
8443/tcp open  https-alt
```

