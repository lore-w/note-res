---
title: From-Data & Request-Payload
tags:
  - 网络请求
date: 2017-07-21 17:12:56
---

1、HTTP请求过程中，GET请求：**表单**参数以NAME=VALUE&n1=v1的形式附到url上
2、HTTP请求过程中，POST请求：**表单**参数是在请求体中，在**Form Data**字段下

POST**表单**请求提交时，使用的Content-Type是application/x-www-form-urlencoded，这也是Html中Form表单的默认Content-Type值，而使用原生**AJAX**的POST请求如果不指定请求头RequestHeader，则默认使用的Content-Type是text/plain;charset=UTF-8，这是请求的参数在请求体的的**Request Payload**字段下

在接口正常的情况下，如果遇到POST请求失败可以检查下，Request Headers中的Content-Type和请求的
参数在Form Data下还是Request Payload下

如下是一个错误的例子

```js
Request Headers

Accept:application/json, text/plain,
Accept-Encoding:gzip, deflate
Accept-Language:zh-CN,zh;q=0.8
Connection:keep-alive
Content-Length:131
Content-Type:application/x-www-form-urlencoded
Cookie:...
Host:...
Origin:...
Referer:...
User-Agent:Mozilla/5.0 ...

```

```js
Form Data

name: tom,
age: 21
```

[CODE Enroll Function](https://github.com/lore-w/vue-demo/tree/master/src/services/enroll.js)