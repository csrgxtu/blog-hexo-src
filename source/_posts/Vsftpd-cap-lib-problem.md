title: "Vsftpd cap lib problem"
date: 2015-03-16 16:06:56
tags: [vsftpd, linux, ftp server]
categories: Linux
---
If you manually install [vsftpd](https://security.appspot.com/vsftpd.html), you will encounter '-lcap' problem, for Debian based distributions, just type:
```bash
apt-get install libcap-dev
```
for Redhat based districtuions, just type:
```bash
yum install libcap-devel
```
