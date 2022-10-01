---
title: OpenAdmin
date: 2022-08-10 19:25
categories: [htb]
tags: [Linux,Password Reuse,Weak Credentials,Password Cracking,Lateral Movement, Source Code Review,Sudo Exploitation]
--- 
![openadminimg](/assets/img/openadminimg.png)
OpenAdmin is a fun easy level box on HacktheBox, and will always have a special place in my heart as the first box I fully solved without needing a walkthrough. We begin by exploiting a vulnerable management console on the website to get a foothold, and ultimately abuse sudo priviledges to root the box.

### Enumeration

To begin, we run an nmap to look for open ports and running services on the target host.

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

Looks like it's an Ubuntu server with SSH and HTTP open on ports 22 and 80. Before I poke around on the website, I'm going set up some directory brute forcing with dirb to look for any potentially hidden or interesting endpoints on the website.

```bash
dirb http://10.10.10.171 /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt
```

At first glance the web page doesn't seem to actually contain any content, just a default apache2 index page.

![it works](/assets/img/openadmin/itworks.png)

Luckily our dirb found the /music and /artwork directories, which *do* contain web content, though most of the links on the site just point back to index.html.

![dirb](/assets/img/openadmin/dirb.png)
![login](/assets/img/openadmin/login.png)

A "login" button, however, redirects us to the /ona page, where it seems we have access to some sort of management console.

![ona](/assets/img/openadmin/onaconsole.png)

ONA is a network management console, but after messing around here for a little, it doesn't really seem like there's much we can do. There's default admin credentials in the [ONA documentation](https://github.com/opennetadmin/ona), but logging in as admin still doesn't give us much to work with. On google, we can learn that ONA has had some serious security vulnerabilities in the past. As a matter of fact, an exploit script for this exact version of ONA (18.1.1) is available on [Github](https://github.com/amriunix/ona-rce), which explains the warning on the left side of the web console.

### Foothold

![shell](/assets/img/openadmin/shell.png)

Using this exploit, we can get a shell on the target as www-data! Let's get a real shell now and see what we can find on the box.

![revshell](/assets/img/openadmin/revshell.png)

We send a bash reverse shell, the connection hangs, and a new connection pops up on our listener.

![listener](/assets/img/openadmin/listener.png)

This shell is not interactive, so we need to run some commands to upgrade it

```bash
python3 -c 'import pty; pty.spawn("/bin/bash")'
export TERM=screen
(CTRL+Z)
stty raw -echo; fg
```

Now that we have our upgraded shell, we will begin by enumerating the /opt/ona/www directory that our shell spawned in.

![juicy](/assets/img/openadmin/juicy.png)

After further digging we find an interesting config file in /opt/ona/www/local/config called database_settings.inc.php, which contains plaintext credentials for a database user!

![creds](/assets/img/openadmin/creds.png)

From the /etc/passwd file we can see there are 2 users with logins on the box, Joanna and Jimmy.

![home](/assets/img/openadmin/users.png)

### Pivoting to Jimmy

Let's try logging into SSH with the password we found earlier, it's possible that one of the users who created the db password reuses the same password for their own login, so we'll check both Jimmy and Joanna.

![home](/assets/img/openadmin/ssh.png)

Success! We can now SSH into the machine as Jimmy, however the user flag is nowhere to be seen, looks like we'll need to pivot to Joanna to get it. For now let's carry on enumerating the box. Netstat shows us a port open locally on 52846.

![home](/assets/img/openadmin/netstat.png)

If I make a GET request to the port with curl, I get an HTML response, meaning a website is hosted on this port. 

![curl](/assets/img/openadmin/curl.png)

However, I can't visit this website from my machine because it is only being served internally on the target, meaning I have to tunnel my SSH connection in order to interact with the target's internal services.

```bash
ssh jimmy@10.10.10.171 -L 52846:127.0.0.1:52846
```

Now when I visit http://127.0.0.1:52846, I can access the website that is being served internally on the target, neat!

![internal](/assets/img/openadmin/internal.png)

The credentials we found earlier don't let us in, so let's do some more digging for config files or anything that will give us a better idea of what's happening behind the scenes on this website.

![list](/assets/img/openadmin/internalsite.png)

The /var/www/internal directory contains the php source code of the website, and once again we find hard coded credentials, this time it's a SHA512 hash in the index.php file.

![key](/assets/img/openadmin/hash.png)

Using an online hash table we can see that the plaintext password for this hash is "Revealed"

![Revealed](/assets/img/openadmin/Revealed.png)

Logging in with our new found credentials takes us to /main.php, which is leaking Joanna's SSH private key. We'll grab this so we can SSH into the box as her.

### Pivoting to Joanna

![key](/assets/img/openadmin/key.png)

We'll want to crack this with John, which is pretty simple with some help from [this guide](https://null-byte.wonderhowto.com/how-to/crack-ssh-private-key-passwords-with-john-ripper-0302810/).

![bloodninjas](/assets/img/openadmin/bloodninjas.png)

Following the guide further, we give the private key the proper permissions, and then use it along with the passphrase we got from john to authenticate to the SSH server. After pivoting 2 times, the user flag is finally ours!

![user](/assets/img/openadmin/user.png)

### Privilege Escalation

Luckily, the road to root is not far from here. Let's first check if Joanna has sudo priviledges anywhere.

![sudo](/assets/img/openadmin/sudo.png)

Joanna has sudo access with nano inside of /opt/priv, we can exploit this to get root access using a nice trick from [gtfobins](https://gtfobins.github.io/gtfobins/nano/).

```bash
sudo /bin/nano /opt/priv
(^R^X)
reset; sh 1>&0 2>&0
```

![root](/assets/img/openadmin/root.png)

And just like that, we have a root shell within nano that we can use to grab the root flag.
