---
title: HTB TwoMillion Write-up
categories: [HackTheBox, Machines]
tags: [HackTheBox]
image:
    path: /assets/image_post/20240312190236.png
---

## [0x00] port scan 
---
port scan 결과를 보면 ssh와 http가 열려있음을 알 수 있고, 접속 시 'http://2million.htb/'로 redirect 됨을 알 수 있다.
``` bash
┌──(root㉿kali)-[/home/user/htb/Log4jUnifi]
└─# nmap -sCV 10.10.11.221
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-03-12 05:57 EDT
Nmap scan report for 10.10.11.221
Host is up (0.31s latency).
Not shown: 998 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.1 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   256 3e:ea:45:4b:c5:d1:6d:6f:e2:d4:d1:3b:0a:3d:a9:4f (ECDSA)
|_  256 64:cc:75:de:4a:e6:a5:b4:73:eb:3f:1b:cf:b4:e3:94 (ED25519)
80/tcp open  http    nginx
|_http-title: Did not follow redirect to http://2million.htb/
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

http://2million.htb/ 에 접속 시도 시 연결이 되지 않는 것을 알 수 있다. htb에서 이런 경우에는 /etc/hosts 파일을 수정해주어야 한다. 수정 시 정상적으로 접근 된다.
``` bash
┌──(root㉿kali)-[/home/user/htb]
└─# curl http://2million.htb/
curl: (6) Could not resolve host: 2million.htb

┌──(root㉿kali)-[/home/user/htb]
└─# echo '10.10.11.221 2million.htb' >> /etc/hosts

┌──(root㉿kali)-[/home/user/htb]
└─# curl http://2million.htb/
<!DOCTYPE html>
<html lang="en">

<head>

    <meta charset="utf-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
```


## [0x01] web site
---
웹 페이지 방문 시 아래와 같은 모습이 보이며, 특별한 기능은 없고 login 기능이 동작한다.
![](../assets/image_post/20240312190836.png)


login 관련해서 sql injection이 동작할 듯 하여, sqlmap을 구동시키려했으나 이상하게 연결이 되지 않는다....왜일까... curl은 잘 되는데..
``` bash
┌──(root㉿kali)-[/home/user]
└─# sqlmap -u "http://2million.htb/login" --data="email=test&password=test" --cookie="PHPSESSID=631oojvs6hjl3bvqmfttoiv0q9" --dbs
        ___
       __H__
 ___ ___[']_____ ___ ___  {1.7.11#stable}
|_ -| . [)]     | .'| . |
|___|_  [)]_|_|_|__,|  _|
      |_|V...       |_|   https://sqlmap.org

[!] legal disclaimer: Usage of sqlmap for attacking targets without prior mutual consent is illegal. It is the end user's responsibility to obey all applicable local, state and federal laws. Developers assume no liability and are not responsible for any misuse or damage caused by this program

[*] starting @ 06:45:38 /2024-03-12/

[06:45:38] [INFO] testing connection to the target URL
[06:45:38] [CRITICAL] unable to retrieve page content
[06:45:38] [WARNING] HTTP error codes detected during run:
405 (Method Not Allowed) - 1 times

[*] ending @ 06:45:38 /2024-03-12/
```

gobuster도 동일하게 동작하지 않는다...
``` bash
┌──(root㉿kali)-[/home/user]
└─# gobuster dir -u http://2million.htb/login -w dsstorewordlist.txt
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://2million.htb/login
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                dsstorewordlist.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================

Error: the server returns a status code that matches the provided options for non existing urls. http://2million.htb/login/7a655d81-ea16-48b7-b7f9-74fbe6a659ab => 301 (Length: 162). To continue please exclude the status code or the length
```