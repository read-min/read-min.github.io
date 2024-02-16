---
title: Microsoft Remote Procedure Call(RPC)
categories: [Pentest, Windows]
tags: [Windows, RPC]
image:
    path: /assets/image_post/20240216132648.png
draft: true
---
Remote Procedure Call(RPC)란 원격 프로시저 호출로, 분산 시스템에서 프로그램 간 통신을 위한 프로토콜이나 매커니즘을 나타낸다. `client<->server`간 통신을 가능하게 하는 기술이다.

### 보안 고려사항


| **주요 고려 사항**               | **세부 내용**                                                                                   |
|----------------------------------|-------------------------------------------------------------------------------------------------|
| **인증 및 권한 부여 문제**        | RPC 시스템에서 클라이언트와 서버 간의 통신은 신뢰할 수 있는 방법으로 보호되어야 합니다. 악의적인 클라이언트가 원격 프로시저에 접근하려는 것을 방지하기 위해 강력한 인증 및 권한 부여 메커니즘을 구현해야 합니다. |
| **데이터의 암호화**               | 원격 프로시저 호출 중에 전송되는 데이터는 중요할 수 있습니다. 이러한 데이터를 보호하기 위해 SSL/TLS와 같은 암호화 프로토콜을 사용하여 통신을 암호화해야 합니다.               |
| **보안 업데이트**                 | RPC 라이브러리 및 프로토콜은 지속적으로 업데이트되고 보완되어야 합니다. 시스템에서 사용하는 RPC 구현은 최신 보안 업데이트가 적용되어 있어야 하며, 잠재적인 취약점에 대한 패치가 시스템에 적용되어야 합니다. |
| **DoS (서비스 거부) 공격 방어** | RPC 시스템은 DoS 공격에 노출될 수 있습니다. 이에 대비하기 위해 적절한 자원 제한 및 공격 탐지 메커니즘을 구현하여 시스템을 보호해야 합니다.                 |


그렇다면 어떻게 위협이 되는지 테스트 해보자. 아래와 같이 135가 기본 port로 사용된다.
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
```


``` bash
┌──(root㉿kali)-[/home/user]
└─# impacket-rpcdump -port 135 10.129.95.187
Impacket v0.11.0 - Copyright 2023 Fortra

[*] Retrieving endpoint list from 10.129.95.187
Protocol: [MS-RSP]: Remote Shutdown Protocol
Provider: wininit.exe
UUID    : D95AFE70-A6D5-4259-822E-2C84DA1DDB0D v1.0
Bindings:
          ncacn_ip_tcp:10.129.95.187[49664]
          ncalrpc:[WindowsShutdown]
          ncacn_np:\\ARCHETYPE[\PIPE\InitShutdown]
          ncalrpc:[WMsgKRpc083610]

Protocol: N/A
Provider: winlogon.exe
UUID    : 76F226C3-EC14-4325-8A99-6A46348418AF v1.0
Bindings:
          ncalrpc:[WindowsShutdown]
          ncacn_np:\\ARCHETYPE[\PIPE\InitShutdown]
          ncalrpc:[WMsgKRpc083610]
          ncalrpc:[WMsgKRpc084D11]

Protocol: [MS-SCMR]: Service Control Manager Remote Protocol
Provider: services.exe
UUID    : 367ABB81-9844-35F1-AD32-98F038001003 v2.0
Bindings:
          ncacn_ip_tcp:10.129.95.187[49667]

Protocol: [MS-FASP]: Firewall and Advanced Security Protocol
Provider: FwRemoteSvr.dll
UUID    : 6B5BDD1E-528C-422C-AF8C-A4079BE4FE48 v1.0 Remote Fw APIs
Bindings:
          ncacn_ip_tcp:10.129.95.187[49668]
          ncalrpc:[ipsec]

Protocol: [MS-CMPO]: MSDTC Connection Manager:
Provider: msdtcprx.dll
UUID    : 906B0CE0-C70B-1067-B317-00DD010662DA v1.0
Bindings:
          ncalrpc:[LRPC-5575e284d83e243c4a]
          ncalrpc:[OLE6330F0E7E7567E176E11548627E0]
          ncalrpc:[LRPC-e43fa1f5a21c9d351f]
          ncalrpc:[LRPC-e43fa1f5a21c9d351f]
          ncalrpc:[LRPC-e43fa1f5a21c9d351f]

[*] Received 266 endpoints.
```



`rpcclient`를 통해 접속한 경우 아래와 같이 출력된다. 익명으로 접속하니 권한이 없음을 알 수 있다.
``` bash
┌──(root㉿kali)-[/home/user]
└─# rpcclient -N -U "" 10.129.95.187
rpcclient $> getusername
do_cmd: Could not initialise lsarpc. Error was NT_STATUS_ACCESS_DENIED
```

지정된 계정으로 접근 시 아래와 같다.
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
