---
title: How to install sock5 proxy on ubuntu
date: 2017-03-27 15:22:55
tags:
- dante
- sock5
- proxy
- Ubuntu
---


### Server environment
```
$ cat /etc/issue
Ubuntu 14.04.4 LTS
```
<!--more-->

### Install dante
```
$ apt-get install dante-server
```
### If System is 64-bits
```
$ cd /lib/x86_64-linux-gnu/
$ ln -s libc.so.6 libc.so
```

### Add a user for dante
```
$ useradd proxyuser
$ passwd proxyuser
```

### Create log directory
```
$ mkdir /var/log/sockd
```

### Config Dante
```
$ vim /etc/danted.conf
```

```
logoutput: /var/log/sockd/sockd.log
internal: 112.74.79.4 port = 1080
external: 112.74.79.4
method: username none
user.privileged: proxyuser
user.notprivileged: nobody
user.libwrap: nobody
client pass {
    from: 0.0.0.0/0 to: 0.0.0.0/0
    log: connect disconnect
}
pass {
    from: 0.0.0.0/0 to: 0.0.0.0/0 port gt 1023
    command: bind
    log: connect disconnect
}
pass {
    from: 0.0.0.0/0 to: 0.0.0.0/0
    command: connect udpassociate
    log: connect disconnect
}
block {
    from: 0.0.0.0/0 to: 0.0.0.0/0
    log: connect error
}
```

### Start Dante
```
$ /etc/init.d/danted start
```

### Check listen success
```
$ netstat -anp | grep 10080
```