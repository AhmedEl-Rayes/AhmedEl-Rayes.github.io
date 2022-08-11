---
title: Open Admin
date: 2022-08-10 19:25
categories: [Linux, WebApp]
tags: [htb]
--- 
OpenAdmin is a fun easy level box, and will always have a special place in my heart as the first box I solved without needing a walkthrough. We begin by exploiting a vulnerable management console on the website to get a foothold, and ultimately abuse sudo priviledges to root the box.

### Enumeration

To begin, we run an nmap to look for open ports and running services.

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

Looks like an Ubuntu machine with SSH and HTTP open on ports 22 and 80. Before I poke around on the website, I'm going set up some directory brute forcing with dirb to look for any potentially hidden or interesting endpoints.

```bash
dirb http://10.10.10.171 /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt
```

At first glance the web page doesn't seem to actually contain any content, just a default apache2 index page.

![it works](/assets/img/itworks.png)

Luckily our dirb found the /music and /artwork directories, which *do* actually contain web content, though most links on the site point back to index.html. A "login" button, however, redirects us to the /ona page, where it seems we have access to some sort of management console.

![ona](/assets/img/onaconsole.png)

By using some good ol' google, it appears that ONA is OpenNetAdmin, a network management tool with some pretty serious vulnerabilities reported in the past. As a matter of fact, an exploit script for this exact version of ona (18.1.1) is available on [Github](https://github.com/amriunix/ona-rce).

### Foothold

![shell](/assets/img/shell.png)

Using this exploit, we can get a shell on the target as www-data! Let's get a real shell now and see what we can find on the box.

![revshell](/assets/img/revshell.png)

We send a bash reverse shell, the connection hangs, and a new connection pops up on our listener.

![listener](/assets/img/listener.png)

This shell is not interactive, so we need to run some commands to upgrade it-

```bash
python3 -c 'import pty; pty.spawn("/bin/bash")'
export TERM=screen
(CTRL+Z)
stty raw -echo; fg
```

Now that we have an interactive shell, we will begin by enumerating the /opt/ona/www directory.

![juicy](/assets/img/juicy.png)

After further digging we find an interesting config file in /opt/ona/www/local/config called database_settings.inc.php, which contains plaintext credentials for a database user!

![creds](/assets/img/creds.png)

From the /etc/passwd file we can see there are 2 users with logins on the box, Joanna and Jimmy.

![home](/assets/img/users.png)

Let's try logging into SSH with the password we found earlier.

![home](/assets/img/ssh.png)

Success! The user flag is nowhere to be seen, looks like we'll need to pivot to Joanna to get it. For now let's carry on enumerating the box. Netstat shows us a port open locally on 52846.

![home](/assets/img/netstat.png)

I'm able to curl and make a GET request, and I get HTML as a response. I'm going to have to tunnel my ssh connection to access the machine's internal services. This can be done with-

```bash
ssh jimmy@10.10.10.171 -L 52846:127.0.0.1:52846
```

Now when I visit http://127.0.0.1:52846 I can see a login page.
