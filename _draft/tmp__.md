

https://github.com/OJ/gobuster/issues/276#issuecomment-1009422173

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