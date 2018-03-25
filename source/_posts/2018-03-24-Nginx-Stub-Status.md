---
title: Nginx 监控模块
tags:
  - nginx
categories:
  - 手记
date: 2018-03-24 14:31:48
---

# 语法规则

```bash
Syntax: stub_status;
# 默认没有开启
Default: -
# 需要配置在server或location模块下
Context: server,location
```

<!-- more -->

# 栗子来了

```bash
$ cd /etc/nginx/conf.d
$ vim default.conf

server {
    listen       80;
    server_name  localhost;

    #charset koi8-r;
    #access_log  /var/log/nginx/host.access.log  main;

    # 添加stub_status配置
    location /mystatus {
        stub_status;
    }
    # 添加stub_status配置 End
    location / {
        root   /usr/share/nginx/html;
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

$ wq
...
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

# 获取本机IP
$ ip a

...
# 注意这里获取的是阿里云的私有IP，我们需要公网IP
inet 172.xxx.xxx.xxx/20 brd 172.xxx.xxx.xxx scope global dynamic eth0
...


$ curl http://47.97.112.40/mystatus

# 当前活跃的连接数
Active connections: 1

#server  :nginx接收的总的handshake次数
#accepts :nginx处理的连接数
#handled requests :nginx 总的请求数
#正常情况下server和accepts相等，表示没有丢失
server accepts handled requests
 16 16 16
 # 读 写 等待（建立了空连接）
Reading: 0 Writing: 1 Waiting: 0

```
