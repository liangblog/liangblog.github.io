---
title: Move gitlab repository to new server
date: 2017-03-27 17:05:19
tags:
- Ubuntu
- gitlab
- backup
---


### Server environment
```
$ cat /etc/issue
Ubuntu 14.04.4 LTS
```
<!--more-->

#### It was using gitlab-rake to backup and restore the gitlab repository in this tutorial
---

### First of all, you must check the gitlab version in old server!!
```
$ cat /opt/gitlab/embedded/service/gitlab-rails/VERSION
8.7.0
```
The old gitlab server version is 8.7.0.

So the new machine gitlab version must be 8.7.0, because the data backup by gitlab tool named 'gitlab-rake' which can only be recovery in same version.
It was worked for me as well.

### Install dependant library and tools in new machine

```
sudo apt-get install -y curl openssh-server ca-certificates postfix
```

### Install gitlab-ce in new machine
In china, it may be slowly to download, so you can use another apt resource image. Of course you can pass this step with a fast network. 
```
sudo sh -c "echo 'deb https://mirrors.tuna.tsinghua.edu.cn/gitlab-ce/debian jessie main' >> /etc/apt/sources.list.d/gitlab-ce.list"
```
```
sudo apt-get update
```
then search gitlab 8.7.0 version
```
$ sudo apt-cache policy gitlab-ce | grep 8.7.0
  Installed: 8.7.0-ce.0
 *** 8.7.0-ce.0 0
```

```
sudo apt-get install gitlab-ce=8.7.0-ce.0
```

### Backup gitlab repository as a tar package in old server
```
gitlab-rake gitlab:backup:create RAILS_ENV=production
```
In general, backup files are usually stored in directory '/var/opt/gitlab/backups', check the tar file named '1490248589_gitlab_backup.tar'. The number 1490248589 was timestamp.
Distinguish different backup version. 
```
scp /var/opt/gitlab/backups/1490248589_gitlab_backup.tar username@new-server:/var/opt/gitlab/backups/
```
Copy the gitlab repository backup file to new server

### Restore repositofy in new server
```
gitlab-rake gitlab:backup:restore RAILS_ENV=production   BACKUP=1490248589
```

### If something went wrong
- /usr/lib/x86_64-linux-gnu/libstdc++.so.6: version `CXXABI_1.3.8' not found (required by ...
```
sudo add-apt-repository ppa:ubuntu-toolchain-r/test 
sudo apt-get update 
sudo apt-get install g++-4.9
```
Or you can install by apt-fast when apt install g++ as a snails
```
sudo apt-fast install gcc-4.9 g++-4.9
```

- Repositories backup successfully but getting a 500 page after click the project name in the web view, and gitlab error log shows that
```
gitlab-ctl tail
```

```
gitlab ErrorPage: serving predefined error page: 500
```
It caused by the gitlab security file '/etc/gitlab/gitlab-secrets.json', you should copy this file in old server to new server at this time
