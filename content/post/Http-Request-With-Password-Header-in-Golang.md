---
title: Http Request With Password Header in Golang
date: 2017-09-14 23:11:04
tags:
- golang
- auth
---


### My computer develop environment
```
$ sw_vers 
ProductName:    Mac OS X
ProductVersion: 10.12.6
BuildVersion:   16G29

$ go version
go version go1.8.3 darwin/amd64
```
<!--more-->


### Request with office golang http package

```
package main

import (
	"net/http"
	"log"
	"io/ioutil"
)

func main() {
	
	url := "http://localhost:9112"
	username := "liyuliang"
	password := "password"
	request, err := http.NewRequest("GET", url, nil)
	if err != nil {
		log.Println("error", err)
	}
	if username != "" || password != "" {
		request.SetBasicAuth(username, password)
	}
	cli := &http.Client{
	}
	response, _ := cli.Do(request)
	body, err := ioutil.ReadAll(response.Body)
	println(string(body))

}
```


### HTTP Basic Authentication
When http request was sent that this request could use http basic auth to access the permission validation by providing a username and password
It's the easiest way to verify permissions because it does not depend on any external factors such as cookies ,sessions
It must add authorization header each request but it does not encrypt username or password unfortunately

Formatted as the following structure:
- username and password are connected by a colon, such as "username:password"
- base64 encoded it 
- then be placed after the keyword "Basic"

Sometime the header looks like "Basic ZGVtbzpwQDU1dzByZA==" 
- Notice that there is a space behind the keyword "Basic"


### Request with third party http packages

```
package main

import (
	"github.com/imroc/req"
	"encoding/base64"
)

func basicAuth(username, password string) string {
	auth := username + ":" + password
	return base64.StdEncoding.EncodeToString([]byte(auth))
}

func main() {

	authHeader := req.Header{
		"Authorization": "Basic " + basicAuth("liyuliang", "password"),
	}

	resp, _ := req.Get("http://localhost:9112", authHeader)
	println(resp.String())

}
```
