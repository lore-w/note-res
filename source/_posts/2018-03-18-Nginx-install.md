---
title: Nginx-install
tags:
  - nginx
categories:
  - 挖、草稿
date: 2018-03-18 13:44:23
---


# yum

来来来，点这里
[Pre-Built Packages](http://nginx.org/en/linux_packages.html#stable)

> To set up the yum repository for RHEL/CentOS,
> create the file named /etc/yum.repos.d/nginx.repo with the following contents:

<!-- more -->

```bash
[nginx]
name=nginx repo
baseurl=http://nginx.org/packages/OS/OSRELEASE/$basearch/
gpgcheck=0
enabled=1
```
> Replace `OS` with `rhel` or `centos`,
> Depending on the distribution used, and `OSRELEASE` with `6` or `7`, for 6.x or 7.x versions, respectively.

# Install

```bash
$ cd /etc/yum.repos.d/
$ ls

CentOS-Base.repo
CentOS-Media.repo
backup
CentOS-Debuginfo.repo
CentOS-Vault.repo
epel.repo

$ touch nginx.repo
$ vim nginx.repo

....

$ yum list |grep nginx

...

$ yum install nginx
```

**可能出现的报错信息**

---

```bash
Error: Package: 1:nginx-1.12.2-1.el7_4.ngx.x86_64 (nginx)
           Requires: systemd

Error: Package: 1:nginx-1.12.2-1.el7_4.ngx.x86_64 (nginx)
           Requires: libpcre.so.1()(64bit)

Error: Package: 1:nginx-1.12.2-1.el7_4.ngx.x86_64 (nginx)
           Requires: openssl >= 1.0.2
           Installed: openssl-1.0.1e-30.el6_6.5.x86_64 (@updates)
               openssl = 1.0.1e-30.el6_6.5
           Available: openssl-1.0.1e-57.el6.i686 (base)
               openssl = 1.0.1e-57.el6

Error: Package: 1:nginx-1.12.2-1.el7_4.ngx.x86_64 (nginx)
           Requires: libcrypto.so.10(OPENSSL_1.0.2)(64bit)

Error: Package: 1:nginx-1.12.2-1.el7_4.ngx.x86_64 (nginx)
           Requires: libc.so.6(GLIBC_2.14)(64bit)

 You could try using --skip-broken to work around the problem
 You could try running: rpm -Va --nofiles --nodigest
```

**找** *原因啊...*

.
.
.
.

**找** *原因...*

.
.
.
.

继续找...

.
.
.
.

找到了...


{% cq %} ...是我配置错`OSRELEASE`了，我的服务器是`CENTOS6.5`(┬＿┬) {% endcq %}


+ 把`CENTOS`的版本换成7（已验证）
+ 把`OSRELEASE`修改为6（待验证）

即可成功安装

不过对于上面的报错信息，还是有一些知识点值得捣鼓一会

---

# 捣鼓...捣鼓

## Requires: libc.so.6(GLIBC_2.14)(64bit)

```bash
$ strings /lib64/libc.so.6 | grep GLIBC

GLIBC_2.2.5
GLIBC_2.2.6
GLIBC_2.3
GLIBC_2.3.2
GLIBC_2.3.3
GLIBC_2.3.4
GLIBC_2.4
GLIBC_2.5
GLIBC_2.6
GLIBC_2.7
GLIBC_2.8
GLIBC_2.9
GLIBC_2.10
GLIBC_2.11
GLIBC_2.12
GLIBC_PRIVATE

$ cd ~
$ wget http://ftp.gnu.org/gnu/glibc/glibc-2.14.tar.gz
$ tar -zxvf glibc-2.14.tar.gz
$ cd glibc-2.14
$ mkdir build
$ cd build
$ ../configure --prefix=/usr --disable-profile --enable-add-ons --with-headers=/usr/include --with-binutils=/usr/bin
$ make && make install
```

等...一下，你触及到我的知识盲区了，strings是个什么鬼？


### `Strings`

在对象文件或二进制文件中查找可打印的字符串

**例**

```bash
$ strings /etc/yum.repos.d.nginx.repo

[nginx]
name=nginx repo
baseurl=http://nginx.org/packages/centos/7/$basearch/
gpgcheck=0
enabled=1


$ strings /etc/yum.repos.d.nginx.repo | grep nginx

[nginx]
name=nginx repo
baseurl=http://nginx.org/packages/centos/7/$basearch/
```


盲区有点多...

### Make install

> ***源码*** 的安装一般由3个步骤组成：

+ *配置* (`configure`)
+ *编译* (`make`)
+ *安装* (`make install`)

> `--prefix`选项是配置安装的路径，如果不配置该选项，安装後

| 文件类型 | 默认位置 |
|---|
| *可执行文件*     |/usr/local/bin|
| *配置文件*      |/usr/local/etc|
| *库文件*        |/usr/local/lib|
| *其它的资源文件* |/usr/local/share|

*软件开发者得安套路出牌...（遵从rpm包管理规范）*


**就是喜欢举栗子**

如果配置`--prefix=/usr/demo`
可以把文件都放在`/usr/local/demo`的路径中，当某个安装的软件不再需要时，只须简单的删除该安装目录，就可以把软件卸载得干干净净

#### 其它配置选项：

+ --disable-profile
+ --enable-add-ons
+ --enable-kernel=2.6.0
+ --with-binutils=
+ --with-headers=/usr/demo/include
  这个参数指示 Glibc 按照前面刚刚安装到 usr/demo 目录中的内核头文件*编译*自己，从而精确的知道内核的特性以*根据*这些特性对自己进行最佳化*编译*
+ --without-selinux
+ --without-gd

## 边边角角

+ *`yum list |grep nginx` 把yum源中nginx相关的版本列出来，grep是一个管道符*
+ *`openssl version -a`*

