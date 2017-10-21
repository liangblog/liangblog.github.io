---
title: How Http Server Add Http Basic Auth In Golang
date: 2017-09-14 17:30:42
tags:
- Golang
- web    
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

### How to add bash auth to filter the request in web server  

```
package main

import (
	"bytes"
	"io"
	"net/http"
	"strings"
	"encoding/base64"
)

type requestFunc func(http.ResponseWriter, *http.Request)

func BasicAuth(execFunc requestFunc, username, password []byte) requestFunc {

	return func(w http.ResponseWriter, r *http.Request) {

		basicAuthPrefix := "Basic "

		// get the request header
		auth := r.Header.Get("Authorization")

		// if request with http basic auth
		if strings.HasPrefix(auth, basicAuthPrefix) {

			// decode it
			payload, err := base64.StdEncoding.DecodeString(
				auth[len(basicAuthPrefix):],
			)

			if err == nil {

				pair := bytes.SplitN(payload, []byte(":"), 2)

				if len(pair) == 2 && bytes.Equal(pair[0], username) &&
					bytes.Equal(pair[1], password) {

					execFunc(w, r)
					return
				}
			}
		}

		// if auth failed, restricted other value
		w.Header().Set("WWW-Authenticate", `Basic realm="Restricted"`)
		// return 401 status code
		w.WriteHeader(http.StatusUnauthorized)
	}
}

func secretRequest(w http.ResponseWriter, r *http.Request) {
	io.WriteString(w, "You got 1 million!!!\n")
}

func main() {

	username := []byte("admin")
	password := []byte("password")

	http.HandleFunc("/", BasicAuth(secretRequest, username, password))
	http.ListenAndServe(":8000", nil)
}
```
