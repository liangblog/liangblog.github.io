---
title: Be care of the expose ports when docker running in your server
date: 2017-03-25 11:52:42
tags: 
- Ubuntu
- docker
- iptable
---


### Server environment
```
$ cat /etc/issue
  Ubuntu 14.04.5 LTS

$ docker -v
  Docker version 1.6.2
```
<!--more-->

Before using docker in ubuntu, open ports are usually be limited by iptable and it's safe and reliable. 
I'm also very confident to use this tool to configure the firewall.
Like Mysql,Redis,Elasticsearch, service machines are usually been deployed in internal network. So i have never consider about the problem that ports will exposed without authentication.

One day after deploying a picture storage service by docker container.
I ban its port in iptable list on test server machine which can be visit in external network. 

It looks like this: 
  
### Running redis docker container as service
```
$ docker run --name=redis-app  -d -p 6379:6379 redis:3.2.1
```
Docker will start a single redis container and binds its port to the host.
At this time, redis can be connected anywhere.

### Try to ban the 6379 port by iptable 
```
# Reject all request to the 6379 port 
$ sudo iptables -I INPUT -p tcp --dport 6379 -j DROP
```
```
# Just allow local access
$ sudo iptables -I INPUT -s 127.0.0.1 -p tcp --dport 6379 -j ACCEPT
```

```
# Save the change
$ sudo iptables-save
```

You will find that iptable doesn't work for this port !!
Anyone can still connect to this redis !!
Let's check the iptable.
```
$ sudo iptables -L DOCKER
target     prot opt source               destination         
ACCEPT     tcp  --  anywhere             172.17.0.2           tcp dpt:6379
```

### Go ahead, How to fix it

Cancel the docker to modify iptables permissions
```
$ sudo vim /etc/default/docker
```

Add this option at the bottom
```
DOCKER_OPTS="--iptables=false"
```

Restart docker
```
$ sudo service docker restart
```