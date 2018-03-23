---
title: Nginx-Http
tags:
  - nginx
categories:
  - 手记
date: 2018-03-23 23:06:31
---

# HTTP 请求

+ Request
  *包含 请求行、请求头部、请求数据*

+ Response
  *包含 状态行、消息报头、响应正文*

<!-- more -->

```bash
$ curl https://lore-w.com
...
...

$ curl -v https://lore-w.com >/dev/null
...
...
> GET / HTTP/1.1 # 请求行
> User-Agent: curl/7.29.0
> Host: lore-w.com
> Accept: */*
>
< HTTP/1.1 200 OK # 状态行
< Accept-Ranges: bytes
< Access-Control-Allow-Origin: *
< Cache-Control: max-age=600
< Content-Length: 44514
< Content-Type: text/html; charset=utf-8
< Last-Modified: Sun, 18 Mar 2018 14:07:04 GMT
< Server: Coding Pages
< Vary: Accept-Encoding
< Date: Fri, 23 Mar 2018 15:01:19 GMT
<
...
```

# CURL

CURL相当于我们的浏览器，模拟浏览器向服务端发起请求，只不过不能渲染页面

+ -v 显示请求详细信息
+ -X 指定请求方式
  `curl -X GET https://lore-w.com`
  `curl -X POST -d "name=lore&age=27" https://lore-w.com`
  由于-d选项为使用POST方式向server发送请求，因此在使用-d的时候，
  可以省略-X POST。
  使用-d时，将使用`Content-type:application/x-www-form-urlencoded`方式发送
  如果想使用`JSON`形式，可以使用-H指定头部类型
  `curl -H "Content-Type:application/json" -d '{"name": "lore"}' ...`
  如果想在请求的时候带上Cookie，可以这样
  `curl -H "Cookie:username=lore-w" ...`
+ -H 增加头部信息
+ -cookie直接指定cookie
+ -F 表单提交操作
  通过-F命令来以`Content-Type:multipart/form-data`的形式向server post数据，该命令允许提交二进制文件等。可以使用@前缀来制定提交的内容为一个文件，也可以使用<符号来提交文件中的内容
  `curl -F img=@lore.jpg https://lore-w.com`

# DEV/NULL

> 在LINUX/UNIX中，一般在*屏幕*上面看到的信息
> 是从STDOUT(STANDARD OUTPUT)或者STDERR(STANDARD ERROR OUTPUT)来的。
> 许多人会问
> OUTPUT就是 OUTPUT，送到*屏幕*上不就得了？
> 为什么还要分成STDOUT和STDERR？那是因为通常在SERVER的工作环境下
> *几乎所有的程序*都是跑在後台的，为了方便DEBUG
> 一般在设计*程序*时，就把STDOUT送到一个档案，把错误的信息STDERR送到不同的档案。
> 有了上面这些认知後，回头来讲什么是 >/DEV/NULL
> 就是把STDOUT送到/DEV/NULL里面
> 那什么是/DEV/NULL ？
> /DEV/NULL是UNIX/LINUX里的无底洞
> OUTPUT送去了就没了