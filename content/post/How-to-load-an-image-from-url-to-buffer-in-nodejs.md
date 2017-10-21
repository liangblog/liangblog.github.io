---
title: How to load an image from url to buffer in nodejs
date: 2017-08-30 23:11:44
tags:
- Mac
- Nodejs
---


### My computer develop environment
```
$ node -v  //v8.1.0
$ npm -v   //5.3.0
```
<!--more-->

### After http request a image url , I want the buffer for picture storage service

so the code look like:
```
var request = require('request').defaults({encoding: null});
request(url, function (error, response, body) {
    //body default is string , but it's buffer type now
});
```
or
```
var request = require('request');

var requestSettings = {
    method: 'GET',
    url: url,
    encoding: null
};
request(url, function (error, response, body) {
    //body default is string , but it's buffer type now
});
```