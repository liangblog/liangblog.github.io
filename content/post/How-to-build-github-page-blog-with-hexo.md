---
title: How to build github page blog with hexo
date: 2017-03-21 17:05:19
tags:
- Ubuntu 
- github
- blog 
---

### My computer develop environment
```
$ sw_vers 
ProductName:    Mac OS X
ProductVersion: 10.12.6
BuildVersion:   16G29

$ brew -v
Homebrew 1.3.0
Homebrew/homebrew-core (git revision 83c2; last commit 2017-08-03)

$ hexo -v
hexo: 3.3.7
hexo-cli: 1.0.3
os: Darwin 16.7.0 darwin x64
http_parser: 2.7.0
node: 8.4.0
v8: 6.0.286.52
uv: 1.13.1
zlib: 1.2.11
ares: 1.10.1-DEV
modules: 57
nghttp2: 1.22.0
openssl: 1.0.2l
icu: 59.1
unicode: 9.0
cldr: 31.0.1
tz: 2017b
```
<!--more-->

### Install hexo by npm
```
sudo npm install -g hexo-cli
```

### Create your hexo project and init

```
mkdir hexo
cd hexo
hexo init
```

### Install hexo plugins 

```
npm install hexo-generator-index --save
npm install hexo-generator-archive --save
npm install hexo-generator-category --save
npm install hexo-generator-tag --save
npm install hexo-server --save
npm install hexo-deployer-git --save
npm install hexo-deployer-heroku --save
npm install hexo-deployer-rsync --save
npm install hexo-deployer-openshift --save
npm install hexo-renderer-marked --save
npm install hexo-renderer-stylus --save
npm install hexo-generator-feed --save
npm install hexo-generator-sitemap --save
npm install hexo-excerpt --save
```


### Create a github New repository
- Repository name must be the host which same with your github page domain, such as xxxx.github.io

### Set your git global config
```
git config --global user.email "you@email.com"
git config --global user.name "your name"
```

### Generate ssh key
```
ssh-keygen -t rsa -C you@email.com
```

### Config ssh key into github
copy id_rsa.pub content to your github project 's Deploy keys
```
cat ~/.ssh/id_rsa.pub
```


### Set your git repository url into the hexo project config
```
vim hexo/_config.yml
```

```
deploy:
  type: git
  repo: git@github.com:xxxx/xxxx.github.io.git
```

### Generate static code by hexo and upload to your github page project
generate the code

``` 
hexo g 
```

upload to github page project 
``` 
hexo d
```
it will be public to your domain xxxx.github.io

### If something went wrong
```
hexo new "your new post"
dyld: Library not loaded: /usr/local/opt/icu4c/lib/libicui18n.58.dylib
  Referenced from: /usr/local/bin/node
  Reason: image not found
Abort trap: 6
```

Maybe node has been updated during your blog writing ,Try to reinstall node
```
brew reinstall node --without-icu4c
```