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
```

Picture

We have SSH and HTTP open, so I'm going set up some directory brute forcing with dirb in the background while I poke around on the website.

```bash
dirb http://10.10.10.171 /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt
```

The website doesn't seem to be actually contain anything, just a default apache2 index page.

Picture

Luckily our dirb found the /music and /artwork directories, which  *do* contain web content. A "login" button redirects us to a page on /ona, where it seems we have access to some management console.

Picture

By using some good ol' google, it appears that ONA is OpenNetAdmin, a network management tool with some pretty serious vulnerabilities reported in the past. As a matter of fact, an exploit script for this exact version of ona (18.1.1) is available on [github](https://github.com/amriunix/ona-rce).

Picture

Using this exploit, we can get a shell on the target as www-data! The user flag is no where to be seen,so we're gonna need to pivot to get it.

Picture

blabla