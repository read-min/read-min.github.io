---
title: HTB Dancing Write-up
categories: [HackTheBox, Basic]
tags: [HackTheBox, TCP]
images:
    path: 
---
이번 문제는 TCP 관련 문제로 보인다.
![](../assets/image_post/20240203170310.png)

이번에는 첫 문제부터 당황하게 됐다. 수 없이 많이 써본 nmap인데, 이걸 못 풀어서 힌트를 봐야만 했다.
![](../assets/image_post/20240203171136.png)

당연히 nmap을 기본 옵션으로 하더라도 열려있는 포트에 대해 갖고 오는줄 알고 있었는데, '-p-' 옵션을 줘야 모든 포트에 대해 스캔한다는 것을 알았다.
> Use Nmap to run a port scan, scanning all ports with '-p-'. This can be really slow, so consider adding '--min-rate 5000' or '-T5' to speed it up.
기본 스캔에 대한 결과는 아래와 같이 아무것도 나오지 않는다.
```
 read-min 🍻   ~  nmap -sV 10.129.16.97
Starting Nmap 7.94 ( https://nmap.org ) at 2024-02-03 17:03 KST
Nmap scan report for 10.129.16.97
Host is up (0.27s latency).
All 1000 scanned ports on 10.129.16.97 are in ignored states.
Not shown: 1000 closed tcp ports (conn-refused)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 96.35 seconds
```

하지만 힌트에 나온 옵션대로 스캔한 결과 아래와 같다. (스캔 시간이 꽤 오래 걸린다.)
```

```
# 작성 중......