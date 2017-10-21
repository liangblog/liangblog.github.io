---
title: How a variadic parameter pass through multi function in golang
date: 2017-09-07 17:11:34
tags:
- golang
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

### Why Variadic Parameter
Function accept variadic numbers of arguments or parameters that it can take a list of arguments and call the function only once, whatever string or []string you pass

### Simple Use
```
package main

import "fmt"

// Here's a function that will take an arbitrary number
// of `ints` as arguments.
func sum(nums ...int) {
    total := 0
    for _, num := range nums {
        total += num
    }
    fmt.Println(total)
}

func main() {

    // Variadic functions can be called in the usual way
    // with individual arguments.
    sum(1, 2)
    sum(1, 2, 3)

    // If you already have multiple args in a slice,
    // apply them to a variadic function using
    // `func(slice...)` like this.
    nums := []int{1, 2, 3, 4}
    sum(nums...)
}

```

It's not difficult, right? Ok let's see what i meet

### Pass Variadic Arguments through mutli functions 

```
package main

type User interface {
    Name() string
    Age() string
}

func primaryStudents() (users []User) {
    //...
    return users
}

func studentToDB() (err error) {
    users := primaryStudents()

    if len(users) > 0 {
        err = saveToDB(users)
    }

    return err
}

func saveToDB(datas ...interface{}) (err error) {

    err = Mysql().Save(datas...)
    //...
    return err
}

type mysqlDB struct {
}

func Mysql() *mysqlDB {
    db := new(mysqlDB)
    return db
}

func (db *mysqlDB) Save(models ...interface{}) (err error) {

    if len(models) > 1 {
        //
    } else {
        // !!! models is a slice, but models[0] is a slice too!
        err = db.Create(models[0])
    }
    return err
}

```

1. Get a Interface type slice named Users 
2. Pass Users to method saveToDB()
3. Pass Users to method Mysql().Save() again in method saveToDB()
4. Get the first element method Mysql().Save()

But models[0] got a slice at line 44.

When User slice was been passed to method saveToDB as the first argument in datas, so datas in method saveToDB look like actually
```

datas = [
    0 => Users,
    1 => null,
    2 => null,
    ...
]
```

Even force the datas to "Mysql().Save(datas...)" once again doesn't work 

Solution:
```
//lint 17
err = saveToDB(users)
```
change to 
```
//lint 17
err = saveToDB(users...)
```
