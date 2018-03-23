---
title: Nginx Log Format模块
tags:
  - nginx
categories:
  - 手记
date: 2018-03-23 23:28:23
---

# LOG_FORMAT语法规则

```bash
# log_format 是配置的关键字 name 是配置的名字(一个变量)
Systax: log_format name [escape=default|json] string ...;
Default: log_format combined "...";
# 只能配置在http模块下
Context: http
```

<!-- more -->

---

**Systax中的第三个参数是请求变量，有3种**

## HTTP请求变量

+ `arg_PARAMETER`
  **PARAMETER是参数**
+ `http_HEADER` REQUEST请求里的头信息
  **HEADER是参数**
+ `sent_http_HEADER` RESPONSE里的头信息
  **HEADER是参数**

举一个例子,现在想在access_log中记录userAgent,如下

```bash
$ curl -v https://lore-w.com >/etc/null

> GET / HTTP/1.1
> User-Agent: curl/7.29.0 #### 需要在access_log中记录这个字段
> Host: lore-w.com
> Accept: */*
>
< HTTP/1.1 200 OK
< Accept-Ranges: bytes
< Access-Control-Allow-Origin: *
< Cache-Control: max-age=600
< Content-Length: 44514
< Content-Type: text/html; charset=utf-8
< Last-Modified: Sun, 18 Mar 2018 14:07:04 GMT
< Server: Coding Pages
< Vary: Accept-Encoding
< Date: Fri, 23 Mar 2018 15:59:42 GMT
<
```

咔咔咔！！！

```bash
$ cd /etc/nginx
$ vim nginx.conf

...

http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

    # 步骤：
    # 因为要记录的是request里的头信息，因此使用变量http_HEADER
    # 把变量HEADER换成User-Agent
    # 把大写该小些
    # 把中划线改下划线
    # 在http前加$
    # -- 是分隔符
    log_format  main  '$http_user_agent -- '
                      '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '$http_user_agent "$http_x_forwarded_for"';

    # 在上面的log_format中除了单引号，像-- [] "" 都会作为字符串打印在日志里
    # $remote_addr client的地址
    # $remote_user client 请求nginx认证的username 需要认证模块打开
    # $time_local nginx的时间
    # $request request的请求行
    # $status response的状态
    # $body_bytes_sent 服务端响 给client body的大小
    # $http_referer referer 上一级页面是什么
    # $http_x_forwarded_for

    access_log  /var/log/nginx/access.log  main;

    ...
    include /etc/nginx/conf.d/*.conf;
}

$ wq
...
$ nginx -t -c /etc/nginx/nginx.conf # 检查配置文件的语法和路径是否正确

nginx: configuration file /etc/nginx/nginx.conf test is successful

$ nginx -s reload -c /etc/nginx/nginx.conf

# 请求几次本机
$ curl http://127.0.0.1
$ curl http://127.0.0.1
$ curl http://127.0.0.1

$ tail -f /var/log/nginx/access.log

112.86.218.12 - - [23/Mar/2018:22:50:49 +0800] "GET / HTTP/1.1" 200 612 "-" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_13_2) AppleWebKit/604.4.7 (KHTML, like Gecko) Version/11.0.2 Safari/604.4.7" "-"
112.86.218.12 - - [23/Mar/2018:22:50:49 +0800] "GET /favicon.ico HTTP/1.1" 404 169 "http://47.97.112.40/" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_13_2) AppleWebKit/604.4.7 (KHTML, like Gecko) Version/11.0.2 Safari/604.4.7" "-"
curl/7.29.0 -- 127.0.0.1 - - [24/Mar/2018:00:15:49 +0800] "GET / HTTP/1.1" 200 612 "-" "curl/7.29.0" "-"
curl/7.29.0 -- 127.0.0.1 - - [24/Mar/2018:00:15:51 +0800] "GET / HTTP/1.1" 200 612 "-" "curl/7.29.0" "-"
curl/7.29.0 -- 127.0.0.1 - - [24/Mar/2018:00:15:52 +0800] "GET / HTTP/1.1" 200 612 "-" "curl/7.29.0" "-"

```

## Nginx 内置变量

[Logging to syslog  > access_log > log_format](http://nginx.org/en/docs/http/ngx_http_log_module.html#access_log)

## 自定义变量

# TAIL

tail命令语法

```bash
tail [ -f ] [ -c Number | -n Number | -m Number | -b Number | -k Number ] [ File ]

# 监视File文件增长，假设文件有更新，tail会自己主动更新，确保你见到最新的档案内容
-f
# 从 Number 字节位置读取指定文件
-c Number
# 从 Number 行位置读取指定文件
-n Number
# 从 Number 多字节字符位置读取指定文件
# 比方你的文件假设包括中文字，假设指定-c参数，可能导致截断，但使用-m则会避免该问题
-m Number
# 从 Number 表示的512字节块位置读取指定文件
+ -b Number
# 从 Number 表示的1KB块位置读取指定文件
+ -k Number
```
*上述命令中，都涉及到number，假设不指定，默认显示10行。Number前面可使用正负号，表示该偏移从顶部还是从尾部开始计算*

---

跟tail功能相似的命令还有：

+ `more` 分页显示档案内容
+ `less` 与more相似，但支持向前翻页
+ `head` 仅仅显示前面几行
+ `tail` 仅仅显示後面几行
+ `cat` 从第一行开始显示档案内容
+ `tac` 从最後一行開始显示档案内容
+ `od` 以二进制方式显示档案内容
+ `n` 带行号显示档案内容