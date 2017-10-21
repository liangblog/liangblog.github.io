---
title: How Nginx Add Basic Auth
date: 2017-09-13 22:34:16
tags:
- nginx
- auth
- Ubuntu
---
### Server Environment
```
$ cat /etc/issue
  Ubuntu 14.04.4 LTS
$ nginx -v
  nginx version: nginx/1.10.3 (Ubuntu)  
```
<!--more-->

### Install Nginx
```
$ sudo apt-get install -y apache2-utils nginx 
```
- apache2-utils is a tool that generate a password file

### Create A Nginx User Named "liyuliang" 
```
$ sudo htpasswd -c /etc/nginx/.htpasswd liyuliang
```
Entry your password after this command

### Add Auth Type In Your Nginx Config
```
$ sudo vim /etc/nginx/sites-enabled/default
```

```
server_name localhost;

location / {
        # First attempt to serve request as file, then
        # as directory, then fall back to displaying a 404.
        try_files $uri $uri/ =404;
        # Uncomment to enable naxsi on this location
        # include /etc/nginx/naxsi.rules
        
        auth_basic "Private Property";
        auth_basic_user_file /etc/nginx/.htpasswd;
}
```

### restart nginx
```
$ sudo service nginx restart
```

