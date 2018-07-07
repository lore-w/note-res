---
title: Nginx 目录结构简述
tags:
  - 手记
  - NGINX
categories:
  -
date: 2018-03-18 19:45:23
---

通过yum的方式安装nginx，其实安装的是一个一个的rpm包

```bash

$ rpm -ql nginx

/etc/logrotate.d/nginx
/etc/nginx
/etc/nginx/conf.d
/etc/nginx/conf.d/default.conf
/etc/nginx/fastcgi_params
/etc/nginx/koi-utf
/etc/nginx/koi-win
/etc/nginx/mime.types
/etc/nginx/modules
/etc/nginx/nginx.conf
/etc/nginx/scgi_params
/etc/nginx/uwsgi_params
/etc/nginx/win-utf
/etc/sysconfig/nginx
/etc/sysconfig/nginx-debug
/usr/lib/systemd/system/nginx-debug.service
/usr/lib/systemd/system/nginx.service
/usr/lib64/nginx
/usr/lib64/nginx/modules
/usr/libexec/initscripts/legacy-actions/nginx
/usr/libexec/initscripts/legacy-actions/nginx/check-reload
/usr/libexec/initscripts/legacy-actions/nginx/upgrade
/usr/sbin/nginx
/usr/sbin/nginx-debug
/usr/share/doc/nginx-1.12.2
/usr/share/doc/nginx-1.12.2/COPYRIGHT
/usr/share/man/man8/nginx.8.gz
/usr/share/nginx
/usr/share/nginx/html
/usr/share/nginx/html/50x.html
/usr/share/nginx/html/index.html
/var/cache/nginx
/var/log/nginx
```

<!-- more -->

# 安装目录

```bash

/etc/logrotate.d/nginx # 配置
# Nginx日志轮转，用于logrotate服务的日志切割

/etc/nginx # 目录
/etc/nginx/nginx.conf # 配置
/etc/nginx/conf.d # 目录
/etc/nginx/conf.d/default.conf # 配置
# Nginx服务的配置文件

/etc/nginx/fastcgi_params # 配置
/etc/nginx/uwsgi_params # 配置
/etc/nginx/scgi_params # 配置
# fastcgi、cgi配置

/etc/nginx/koi-utf # 配置
/etc/nginx/koi-win # 配置
/etc/nginx/win-utf # 配置
# 编码 转换映射文件

/etc/nginx/mime.types # 配置
# 设置http协议的Content-type与 expanded-name 的映射关系
# 如：当请求图片时，Response Headers中的Content-Type为image/jpeg

/usr/lib/systemd/system/nginx-debug.service # 配置
/usr/lib/systemd/system/nginx.service # 配置
/etc/sysconfig/nginx # 配置
/etc/sysconfig/nginx-debug # 配置
# Centos7.2+ 配置系统 daemon 管理器的管理方式

/usr/lib64/nginx/modules # 目录
/etc/nginx/modules # 目录
# > Nginx模块目录

# /usr/sbin/nginx # 命令
# /usr/sbin/nginx-debug # 命令
# Nginx服务的管理终端命令

/usr/share/doc/nginx-1.12.0
/usr/share/doc/nginx-1.12.0/COPYRIGHT
/usr/share/main/man8/nginx.8.gz
# Nginx的 手册 和 帮助 文件

/var/cache/nginx # 目录
# Nginx的 缓存 目录

+ /var/log/nginx # 目录
# Nginx的 日志 目录

```