---
layout: post
title:  HTB - Precious
subtitle: Pwning Precious
date:   2023-05-05
tags: [writeup, cybersecurity, hackthebox]
author: Patrick Castro
comments: True
---

1. Conduct port scan using nmap in the given IP address. Open ports are 22 (ssh), and 80 (http).
2. Add the IP address and the domain name (precious.htb) in the host file to access the web app.
3. Valid URLs not doing anything in the web app.
4. Time to FUZZ directories using ffuf! No results found!
5. Time to dig in port 22, ssh. With the version [SSH-2.0-OpenSSH_8.4p1 Debian-5+deb11u1].
6. Time to use metasploit!
7. nginx 1.18.0


Noteable:
Port 22
Port 80
nginx/1.18.0 + Phusion Passenger(R) 6.0.15
SSH-2.0-OpenSSH_8.4p1 Debian-5+deb11u1


