---
title: Nginx 请求/连接限制模块
tags:
  - 手记
  - NGINX
categories:
  -
date: 2018-03-24 19:14:11
---

# HTTP协议的连接与请求

|HTTP协议版本|连接关系|
|---|
|HTTP1.0|TCP不能复用|
|HTTP1.1|*顺序性*TCP复用|
|HTTP2.0|多路复用TCP复用|

+ *HTTP请求建立在一次TCP连接基础上*
+ *一次TCP请求至少产生一次HTTP请求*

# 语法规则

```bash

# 连接限制
# 既然要对连接进行限制，就要对连接的状态进行存储
# 因此需要在操作系统中开辟一个空间，limit_conn_zone就是开辟空间用的
# 这个空间中以什么作为key嘞？就用这个key定义
# 如：以client的ip为key进行限制
# zone=name:size申请的空间的名字，及大小
Syntax: limit_conn_zone key zone=name:size;
# 默认没有开启
Default: -
# 需要配置在http模块下
Context: http

Syntax: limit_conn zone number;
Default: -
Context: http,server,location

# 请求限制
Syntax: limit_req_zone key zone=name:size rate=rate;
Default: -
Context: http

Syntax: limit_req zone [burst=number] [nodelay];
Default: -
Context: http,server,location
```

<!-- more -->

# 栗子来了

```bash
$ cd /etc/nginx/conf.d
$ vim default.conf

# $binary_remote_addr 以ip地址为key
# $binary_remote_addr和$remote_addr都是client的ip地址
# $binary_remote_addr更省空间
# 1r/s对同一个client的ip地址限制每秒1个
limit_req_zone $binary_remote_addr zone=req_zone:1m rate=1r/s;

server {
    listen       80;
    server_name  localhost;

    #charset koi8-r;
    #access_log  /var/log/nginx/host.access.log  main;

    location / {
        root   /opt/app/code;
        # 步骤：
        limit_req zone=req_zone;
        index  index.html index.htm;
    }

    #error_page  404              /404.html;

    # redirect server error pages to the static page /50x.html
    #
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }
    ...
}

# 先清空error.log的日志，方便检查
# 需要重载服务
$ cd /var/log/nginx/
$ rm -rf error.log
$ touch error.log

# 检查配置文件的语法和路径是否正确
$ nginx -t -c /etc/nginx/conf.d/default.conf

nginx: [emerg] "server" directive is not allowed here in /etc/nginx/conf.d/default.conf:1
nginx: configuration file /etc/nginx/conf.d/default.conf test failed

# 注意，这里要检查的不是子配置文件，而是/etc/nginx下的nginx.conf配置文件
nginx -t -c /etc/nginx/nginx.conf

nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful

# 重载服务
$ nginx -s reload -c /etc/nginx/nginx.conf
$ tail -f /var/log/nginx/error.log
2018/03/25 00:22:43 [notice] 19786#19786: signal process started


# 用本地机器ab测试服务器
# -n 10 请求10次
# -c 5  并发请求5个
$ ab -n 10 -c 5 http://47.97.112.40/index.html

This is ApacheBench, Version 2.3 <$Revision: 1807734 $>
Copyright 1996 Adam Twiss, Zeus Technology Ltd, http://www.zeustech.net/
Licensed to The Apache Software Foundation, http://www.apache.org/

Benchmarking 47.97.112.40 (be patient).....done


Server Software:        nginx/1.12.2
Server Hostname:        47.97.112.40
Server Port:            80

Document Path:          /index.html
Document Length:        537 bytes

Concurrency Level:      5
Time taken for tests:   0.101 seconds
# 完成10次请求
Complete requests:      10
# 失败1次
Failed requests:        1
   (Connect: 0, Receive: 0, Length: 1, Exceptions: 0)
# 非200请求9次
Non-2xx responses:      9
Total transferred:      7424 bytes
HTML transferred:       5445 bytes
Requests per second:    99.09 [#/sec] (mean)
Time per request:       50.460 [ms] (mean)
Time per request:       10.092 [ms] (mean, across all concurrent requests)
Transfer rate:          71.84 [Kbytes/sec] received

Connection Times (ms)
              min  mean[+/-sd] median   max
Connect:       21   32  10.6     41      42
Processing:    16   18   2.0     17      21
Waiting:       16   18   2.1     17      21
Total:         37   49  12.2     58      63

Percentage of the requests served within a certain time (ms)
  50%     58
  66%     58
  75%     62
  80%     63
  90%     63
  95%     63
  98%     63
  99%     63
 100%     63 (longest request)

# 这个时候检查服务端的错误日志，tail -f 正在监控
2018/03/25 00:23:01 [error] 19787#19787: *251 limiting requests, excess: 1.000 by zone "req_zone", client: 112.86.218.12, server: localhost, request: "GET /index.html HTTP/1.0", host: "47.97.112.40"
2018/03/25 00:23:01 [error] 19787#19787: *250 limiting requests, excess: 1.000 by zone "req_zone", client: 112.86.218.12, server: localhost, request: "GET /index.html HTTP/1.0", host: "47.97.112.40"
2018/03/25 00:23:01 [error] 19787#19787: *252 limiting requests, excess: 0.999 by zone "req_zone", client: 112.86.218.12, server: localhost, request: "GET /index.html HTTP/1.0", host: "47.97.112.40"
2018/03/25 00:23:01 [error] 19787#19787: *253 limiting requests, excess: 0.999 by zone "req_zone", client: 112.86.218.12, server: localhost, request: "GET /index.html HTTP/1.0", host: "47.97.112.40"
2018/03/25 00:23:01 [error] 19787#19787: *254 limiting requests, excess: 0.943 by zone "req_zone", client: 112.86.218.12, server: localhost, request: "GET /index.html HTTP/1.0", host: "47.97.112.40"
2018/03/25 00:23:01 [error] 19787#19787: *255 limiting requests, excess: 0.942 by zone "req_zone", client: 112.86.218.12, server: localhost, request: "GET /index.html HTTP/1.0", host: "47.97.112.40"
2018/03/25 00:23:01 [error] 19787#19787: *256 limiting requests, excess: 0.941 by zone "req_zone", client: 112.86.218.12, server: localhost, request: "GET /index.html HTTP/1.0", host: "47.97.112.40"
2018/03/25 00:23:01 [error] 19787#19787: *257 limiting requests, excess: 0.941 by zone "req_zone", client: 112.86.218.12, server: localhost, request: "GET /index.html HTTP/1.0", host: "47.97.112.40"
2018/03/25 00:23:01 [error] 19787#19787: *258 limiting requests, excess: 0.941 by zone "req_zone", client: 112.86.218.12, server: localhost, request: "GET /index.html HTTP/1.0", host: "47.97.112.40"


# 栗子继续吃
# 对于请求限制，增加burst 和 nodelay 参数
$ cd /etc/nginx/conf.d
$ vim default.conf

limit_req_zone $binary_remote_addr zone=req_zone:1m rate=1r/s;

server {
    listen       80;
    server_name  localhost;

    #charset koi8-r;
    #access_log  /var/log/nginx/host.access.log  main;

    location / {
        root   /opt/app/code;
        # 步骤：
        # burst=3 当client超过每秒1个以後，遗留3个延迟到下一次执行，其它的不延迟
        limit_req zone=req_zone burst=3 nodelay;
        index  index.html index.htm;
    }

    #error_page  404              /404.html;

    # redirect server error pages to the static page /50x.html
    #
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }
    ...
}

# 检查配置文件的语法和路径是否正确
$ nginx -t -c /etc/nginx/conf.d/default.conf

nginx: [emerg] "server" directive is not allowed here in /etc/nginx/conf.d/default.conf:1
nginx: configuration file /etc/nginx/conf.d/default.conf test failed

# 注意，这里要检查的不是子配置文件，而是/etc/nginx下的nginx.conf配置文件
nginx -t -c /etc/nginx/nginx.conf

nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful

# 重载服务
$ nginx -s reload -c /etc/nginx/nginx.conf

# 在tail -f 活动的窗口敲几个回车，留下几行空白，方便我们检查新的错误日志

# 用本地机器ab测试服务器
# -n 10 请求10次
# -c 5  并发请求5个
$ ab -n 10 -c 5 http://47.97.112.40/index.html

This is ApacheBench, Version 2.3 <$Revision: 1807734 $>
Copyright 1996 Adam Twiss, Zeus Technology Ltd, http://www.zeustech.net/
Licensed to The Apache Software Foundation, http://www.apache.org/

Benchmarking 47.97.112.40 (be patient).....done


Server Software:        nginx/1.12.2
Server Hostname:        47.97.112.40
Server Port:            80

Document Path:          /index.html
Document Length:        612 bytes

Concurrency Level:      5
Time taken for tests:   0.077 seconds
# 完成10次请求
Complete requests:      10
# 失败6次
Failed requests:        6
   (Connect: 0, Receive: 0, Length: 6, Exceptions: 0)
# 非200请求6次
Non-2xx responses:      6
Total transferred:      7766 bytes
HTML transferred:       5670 bytes
Requests per second:    129.99 [#/sec] (mean)
Time per request:       38.465 [ms] (mean)
Time per request:       7.693 [ms] (mean, across all concurrent requests)
Transfer rate:          98.58 [Kbytes/sec] received

Connection Times (ms)
              min  mean[+/-sd] median   max
Connect:       13   18   3.8     20      22
Processing:    16   21   3.7     23      25
Waiting:       16   21   3.8     23      25
Total:         30   38   7.3     45      47

Percentage of the requests served within a certain time (ms)
  50%     45
  66%     45
  75%     45
  80%     45
  90%     47
  95%     47
  98%     47
  99%     47
 100%     47 (longest request)

# 这个时候检查服务端的错误日志，tail -f 正在监控

2018/03/25 13:54:13 [error] 20745#20745: *283 limiting requests, excess: 3.999 by zone "req_zone", client: 112.86.218.12, server: localhost, request: "GET /index.html HTTP/1.0", host: "47.97.112.40"
2018/03/25 13:54:13 [error] 20745#20745: *284 limiting requests, excess: 3.965 by zone "req_zone", client: 112.86.218.12, server: localhost, request: "GET /index.html HTTP/1.0", host: "47.97.112.40"
2018/03/25 13:54:13 [error] 20745#20745: *285 limiting requests, excess: 3.963 by zone "req_zone", client: 112.86.218.12, server: localhost, request: "GET /index.html HTTP/1.0", host: "47.97.112.40"
2018/03/25 13:54:13 [error] 20745#20745: *286 limiting requests, excess: 3.962 by zone "req_zone", client: 112.86.218.12, server: localhost, request: "GET /index.html HTTP/1.0", host: "47.97.112.40"
2018/03/25 13:54:13 [error] 20745#20745: *287 limiting requests, excess: 3.962 by zone "req_zone", client: 112.86.218.12, server: localhost, request: "GET /index.html HTTP/1.0", host: "47.97.112.40"
2018/03/25 13:54:13 [error] 20745#20745: *288 limiting requests, excess: 3.961 by zone "req_zone", client: 112.86.218.12, server: localhost, request: "GET /index.html HTTP/1.0", host: "47.97.112.40"
```

连接限制类似，不过需要注意的是，连接的限制作用要比请求小一些，因为，一次连接可以有多次请求
