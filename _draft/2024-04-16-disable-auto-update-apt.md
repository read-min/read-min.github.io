---
title: disable apt auto update (unattended-upgrade/apt-daily/apt-daily-upgrade)
categories: [ETC., Tools]
tags: [ubuntu, apt-daily, apt-daily-upgrade, unattended-upgrade]
image:
    path: 
---

## [0x00] summary
---
disable & stop 
``` bash
systemctl stop apt-daily.timer
systemctl disable apt-daily.timer
systemctl stop apt-daily-upgrade.timer
systemctl disable apt-daily-upgrade.timer
systemctl stop unattended-upgrades
systemctl disable unattended-upgrades
```




## [0x01] overview
---
apt로 필요한 패키지를 설치하려고 하니 아래와 같이 나오며 진행을 할 수 없다. 무엇때문일까 알아보았다. 우선 해당 pid를 알아보자.
``` bash
root@user-pc:/home/user# sudo apt-get install -y curl openssh-server ca-certificates tzdata perl
E: Could not get lock /var/lib/dpkg/lock-frontend. It is held by process 31143 (unattended-upgr)
N: Be aware that removing the lock file is not a solution and may break your system.
E: Unable to acquire the dpkg frontend lock (/var/lib/dpkg/lock-frontend), is another process using it?
```

pid를 타고 올라가다보면 apt.systemd.daily, unattended-upgrade와 관련된 내용을 만나게 된다.
``` bash
root@user-pc:/home/user# ps -ef | grep dpkg
root       52585   52577  0 18:04 ?        00:00:00 sh -c if [ -d /var/lib/update-notifier ]; then touch /var/lib/update-notifier/dpkg-run-stamp; fi; /usr/lib/update-notifier/update-motd-updates-available 2>/dev/null || true
root       52670   51786  0 18:04 pts/3    00:00:00 grep --color=auto dpkg

root@user-pc:/home/user# ps -ef | grep 31143
root       31143   31080 10 17:53 ?        00:01:13 /usr/bin/python3 /usr/bin/unattended-upgrade
root       52896   31143  6 18:04 ?        00:00:00 /usr/bin/python3 /usr/bin/unattended-upgrade

root@user-pc:/home/user# ps -ef | grep 31080
root       31080   31076  0 17:53 ?        00:00:00 /bin/sh /usr/lib/apt/apt.systemd.daily lock_is_held install
root       31143   31080 10 17:53 ?        00:01:13 /usr/bin/python3 /usr/bin/unattended-upgrade
```