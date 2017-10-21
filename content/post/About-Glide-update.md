---
title: About Glide update
date: 2017-09-17 22:49:34
tags:
- golang
- Glide
- git
- fork
---

### My computer develop environment

```
$ sw_vers 
ProductName:    Mac OS X
ProductVersion: 10.12.6
BuildVersion:   16G29

$ go version
go version go1.8.3 darwin/amd64

$ glide -v
glide version 0.12.3

```
<!--more-->

### Use glide to import dependence packages 
```
$ glide get "github.com/sirupsen/logrus"
```
Because the package github.com/sirupsen/logrus has imported the golang office package golang.org/x/crypto,
which has been moved to [github](github.com/golang/crypto) anymore 

When updating the project 's dependence packages, it always notice that the crypto package cannot been found then interrupted update 

The old way is to download the package below the $GOPATH directory,
then running update to skip this package 

```
$ mkdir -p $GOPATH/src/golang.org/x/
$ git clone https://github.com/golang/crypto.git $GOPATH/src/golang.org/x/crypto
$ git clone https://github.com/golang/sys.git $GOPATH/src/golang.org/x/sys

$ glide up
```

This method is quiet convenient, but each deployment of a server, the workload will increase, so there is a new way.
 
- Fork the project github.com/sirupsen/logrus to my own github, update the github repo address in my project
- Replace the github repo in project glide config file
- Update the dependency package glide up

But in the process of updating ,package golang.org/x/crypto still be prompted to import in a new git warehouse 

In order to exclude the influence of other third party packages, so I created a new project
```
$ cd $GOPATH/src/
$ mkdir test;cd test
$ glide init
$ glide get https://github.com/liyuliang2013/logrus.git
```
After check the code in the new repository, I found the code does not been modify !!
But git clone this new warehouse, the code is indeed modified, why is this??

Check glide's sxplanation:
[issue 330](https://github.com/Masterminds/glide/issues/330)
[issue 726](https://github.com/Masterminds/glide/issues/726) 

ok,we changed the glide configuration file
```
import:
- package: github.com/sirupsen/logrus
```
to
```
import:
- package: github.com/sirupsen/logrus
  repo: https://github.com/liyuliang2013/logrus.git
  vcs: git
  version: master
```

Then the glide will download the package basing on the repo actual address 

