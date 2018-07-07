---
title: Nginx 访问控制模块
tags:
  - 手记
  - NGINX
categories:
  -
date: 2018-03-25 14:06:07
---

# ACCESS语法规则
```bash
# 允许访问
# address 允许指定的ip访问
# CIDR    允许指定的网段访问
# unix    socket的方式访问
# all     全部允许
Syntax: allow address | CIDR | unix: | all;
Default: -
Context: http,server,location,limit_except

# 不允许访问
Syntax: deny address | CIDR | unix: | all;
Default: -
Context: http,server,location,limit_except
```

<!-- more -->

## 栗子

```bash
$ cd /etc/nginx/conf.d
$ mv default.conf access_mod.conf
$ vim access_mod.conf

server {
    listen       80;
    server_name  localhost;

    #charset koi8-r;
    #access_log  /var/log/nginx/host.access.log  main;

    location / {
        root   /usr/share/nginx/html;
        index  index.html index.htm;
    }

    # 步骤：
    # 添加一个location 控制管理员页面的访问
    # ~ 表示一个模式匹配
    # 修改root目录
    # deny 不允许我的Mac访问（ip指我的出口ip，不是内网ip）
    # 允许其他全部机器访问
    location ~ ^/admin.html {
       root   /opt/app/code;
       deny   112.86.218.12;
       allow  all;
       index  index.html index.htm;
    }

    #error_page  404              /404.html;

    # redirect server error pages to the static page /50x.html
    #
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }
}

# 到/opt/app/code新建admin.html

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

# 用本机（Mac）请求服务器
$ curl http://47.97.112.40/admin.html

<html>
<head><title>403 Forbidden</title></head>
<body bgcolor="white">
<center><h1>403 Forbidden</h1></center>
<hr><center>nginx/1.12.2</center>
</body>
</html>

# 用手机（切4G，连无线的话出口ip是一样的）请求服务器，页面显示正常
```

## 有何问题

>这种访问控制其实存在一个问题，只适合CLIENT与服务端直接通信，如果中间经过代理
NGINX限制的其实是代理的IP地址，这也是解释了上面的例子中，手机为什么需要切4G？

## 解决方案

### 1、X_FORWARDED_FOR
![](/images/2018/x-forwarded.jpg)

***从上图我们可以发现，X_FORWARDED_FOR实际上记录了所有代理的ip地址***

`http_x_forwarded_for = ClientIP, Proxy(1) IP,Proxy(2) IP,...`

缺点：*X_FORWARDED_FOR是一个协议要求的，不一定所有的CDN、代理厂商都按照这个要求来做，甚至存在被修改的可能*

### 2、结合GEO模块操作

### 3、通过HTTP自定义变量传递（在头信息里加入自定义变量）

# AUTH语法规则

```bash
Syntax: auth_basic string | off;
Default: auth_basic off;
Context: http,server,location,limit_except

# file 一个文件，存储username和password
Syntax: auth_basic_user_file file;
Default: -;
Context: http,server,location,limit_except
```

## 栗子

```bash
# 先生成密码文件
$ htpasswd -c ./auth_conf /etc/conf lore
# 输入两次密码
New password: 
Re-type new password: 
Adding password for user lore

# 紧接上个栗子，把配置文件修改为auth_mod.conf
$ cd /etc/nginx/conf.d
$ mv access_mod.conf auth_mod.conf
$ vim access_mod.conf

server {
    listen       80;
    server_name  localhost;

    #charset koi8-r;
    #access_log  /var/log/nginx/host.access.log  main;

    location / {
        root   /usr/share/nginx/html;
        index  index.html index.htm;
    }

    # 步骤：
    # 添加一个location 控制管理员页面的访问
    # ~ 表示一个模式匹配
    # 修改root目录
    # deny 不允许我的Mac访问（ip指我的出口ip，不是内网ip）
    # 允许其他全部机器访问
    location ~ ^/admin.html {
       root   /opt/app/code;
       auth_basic "AUTH ACCESS~ INPUT YOUR PASSWORD!";
       auth_basic_user_file /etc/nginx/auth_conf;
       index  index.html index.htm;
    }

    #error_page  404              /404.html;

    # redirect server error pages to the static page /50x.html
    #
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }
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

# 这个时候在浏览器里访问
# http://47.97.112.40/admin.html
# 会发现修改已经生效
```

## 有何问题

> 操作管理机械
> 需要把USERNAME和PASSWORD存入单独文件，在企业里如果存在多套密码体系时，无法实现打通和联动

## 解决方案TODO

### NGINX结合LUA实现高效验证

### NGINX和LDPA打通，利用NGINX-AUTH-LDAP模块