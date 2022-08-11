---
title: Open Admin
date: 2022-08-10 19:25
categories: [Linux, WebApp]
tags: [htb]
--- 

## Enumeration

As always, we run an nmap to look for open ports and running services.

```bash
sudo nmap -sC -sV -p- 10.10.10.171 
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 4b:98:df:85:d1:7e:f0:3d:da:48:cd:bc:92:00:b7:54 (RSA)
|   256 dc:eb:3d:c9:44:d1:18:b1:22:b4:cf:de:bd:6c:7a:54 (ECDSA)
|_  256 dc:ad:ca:3c:11:31:5b:6f:e6:a4:89:34:7c:9b:e5:50 (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-title: Apache2 Ubuntu Default Page: It works
|_http-server-header: Apache/2.4.29 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

We have SSH and HTTP open, so I'm going set up some directory brute forcing with dirb in the background while I poke around on the website.

```bash
dirb http://10.10.10.171 /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt
```

The website doesn't seem to be actually contain anything, just a default apache2 index page.

![it works](/assets/img/itworks.png)

Luckily our dirb found the /music and /artwork directories, which  *do* contain web content. A "login" button redirects us to a page on /ona, where it seems we have access to some kind of management console.

![ona](/assets/img/onaconsole.png)

By using some good ol' google, it appears that ONA is OpenNetAdmin, a network management tool with some pretty serious vulnerabilities reported in the past. As a matter of fact, an exploit script for this exact version of ona (18.1.1) is available on [github](https://github.com/amriunix/ona-rce).

![shell](/assets/img/shell.png)

Using this exploit, we can get a shell on the target as www-data! The user flag is no where to be seen,so we're gonna need to pivot to get it.