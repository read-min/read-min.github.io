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


login 관련해서 sql injection이 동작할 듯 하여, sqlmap을 실행해보았으나 별 다른 소득은 없다.
``` bash
┌──(root㉿5a6b886b0929)-[~]
└─# sqlmap -u http://2million.htb/login --cookie="PHPSESSID=cava0m1a2hvo6f7kjvrjpdqas5"  dbs
        ___
       __H__
 ___ ___[,]_____ ___ ___  {1.8.2#stable}
|_ -| . [(]     | .'| . |
|___|_  [(]_|_|_|__,|  _|
      |_|V...       |_|   https://sqlmap.org

[!] legal disclaimer: Usage of sqlmap for attacking targets without prior mutual consent is illegal. It is the end user's responsibility to obey all applicable local, state and federal laws. Developers assume no liability and are not responsible for any misuse or damage caused by this program

[*] starting @ 14:22:12 /2024-03-12/

[14:22:12] [WARNING] you've provided target URL without any GET parameters (e.g. 'http://www.site.com/article.php?id=1') and without providing any POST parameters through option '--data'
do you want to try URI injections in the target URL itself? [Y/n/q]

[14:22:18] [INFO] testing connection to the target URL
[14:22:18] [INFO] checking if the target is protected by some kind of WAF/IPS
[14:22:19] [INFO] testing if the target URL content is stable
[14:22:19] [INFO] target URL content is stable
[14:22:19] [INFO] testing if URI parameter '#1*' is dynamic
got a 301 redirect to 'http://2million.htb/404'. Do you want to follow? [Y/n]

[14:22:24] [INFO] URI parameter '#1*' appears to be dynamic
[14:22:25] [WARNING] heuristic (basic) test shows that URI parameter '#1*' might not be injectable
[14:22:26] [INFO] testing for SQL injection on URI parameter '#1*'
[14:22:26] [INFO] testing 'AND boolean-based blind - WHERE or HAVING clause'
[14:22:31] [INFO] testing 'Boolean-based blind - Parameter replace (original value)'
[14:22:33] [INFO] testing 'MySQL >= 5.1 AND error-based - WHERE, HAVING, ORDER BY or GROUP BY clause (EXTRACTVALUE)'
[14:22:35] [INFO] testing 'PostgreSQL AND error-based - WHERE or HAVING clause'

...

[14:22:58] [INFO] testing 'Oracle AND time-based blind'
it is recommended to perform only basic UNION tests if there is not at least one other (potential) technique found. Do you want to reduce the number of requests? [Y/n] y
[14:23:45] [INFO] testing 'Generic UNION query (NULL) - 1 to 10 columns'
[14:23:48] [WARNING] URI parameter '#1*' does not seem to be injectable
[14:23:48] [CRITICAL] all tested parameters do not appear to be injectable. Try to increase values for '--level'/'--risk' options if you wish to perform more tests. If you suspect that there is some kind of protection mechanism involved (e.g. WAF) maybe you could try to use option '--tamper' (e.g. '--tamper=space2comment') and/or switch '--random-agent'

[*] ending @ 14:23:48 /2024-03-12/
```

gobuster로 스캔 시 에러와 함께 제대로 동작하지 않는다.
``` bash
┌──(root㉿kali)-[/home/user]
└─# gobuster dir -u http://2million.htb/login -w dsstorewordlist.txt
===============================================================
Gobuster v3.6
...
Error: the server returns a status code that matches the provided options for non existing urls. http://2million.htb/login/7a655d81-ea16-48b7-b7f9-74fbe6a659ab => 301 (Length: 162). To continue please exclude the status code or the length
```

문제 원인을 파악해보니 특정 상태값 처리 떄문으로, [github 커뮤니티](https://github.com/OJ/gobuster/issues/276#issuecomment-1009422173)에 나온 것과 같이 `-b xxx`을 지정해주니 잘 동작한다. 마찬가지로 별 다른 소득은 없다.
``` bash
┌──(root㉿5a6b886b0929)-[~]
└─# gobuster dir -u 2million.htb/ -w dsstorewordlist.txt -b 301
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://2million.htb/
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                dsstorewordlist.txt
[+] Negative Status codes:   301
[+] User Agent:              gobuster/3.6
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/api                  (Status: 401) [Size: 0]
/home                 (Status: 302) [Size: 0] [--> /]
/404                  (Status: 200) [Size: 1674]
/login                (Status: 200) [Size: 3704]
/register             (Status: 200) [Size: 4527]
/logout               (Status: 302) [Size: 0] [--> /]
Progress: 1828 / 1829 (99.95%)
===============================================================
Finished
===============================================================
``` 


