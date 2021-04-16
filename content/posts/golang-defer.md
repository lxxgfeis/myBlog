---
title: "Golang Defer"
date: 2021-04-16T21:32:13+08:00
draft: false
tags: ["golang"]
categories: ["coding 1024"]
featured_image: ""
description: ""
---

# golang中的defer笔记

## 关键点
- defer是在return之前执行的
- return 语句并不是一条原子指令，return被拆成三条语句
 1. 返回值 = xxx
 2. 调用defer函数
 3. 空的return

### 例子

🤔 *tips: 可以思考一下，然后运行看看* 👀

```go
func f() (result int) {
    defer func() {
        result++
    }()
    return 0
}
```

```go
func f() (r int) {
     t := 5
     defer func() {
       t = t + 5
     }()
     return t
}
```

```go
func f() (r int) {
    defer func(r int) {
          r = r + 5
    }(r)
    return 1
}
```

 ## defer与闭包
如下case会出现空指针异常。
```go
package main

import "fmt"

type s struct {
 i int
}

func bar(in *s) {
 fmt.Println("i is %d", in.i)
}

func foo() (ret *s) {
 bar(ret)
 ret = &s{
  i: 1024,
 }
 return
}

func main() {
 foo()
}
```
将`foo`函数改为如下，可以解决空指针：
```go
func foo() (ret *s) {
 defer func() {
  bar(ret)
 }()
 ret = &s{
  i: 1024,
 }
 return
}
```

## 参考资料
1. [defer关键字](https://tiancaiamao.gitbooks.io/go-internals/content/zh/03.4.html)


<br>

<center>  ·End·  </center>
