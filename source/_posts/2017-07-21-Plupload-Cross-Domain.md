---
title: Plupload跨域上传
tags:
  - 跨域
categories:
  - 手记
date: 2017-07-21 17:12:56
---

# 服务端
Nodejs
```js
app.all('*', function(req, res, next) {
  res.header("Access-Control-Allow-Origin", "xxx");
  res.header("Access-Control-Allow-Methods","POST,OPTIONS");
  res.header("Access-Control-Allow-Credentials",true);
  res.header("Access-Control-Allow-Headers", "withCredentials");
  res.header("Content-Type", "application/json;charset=utf-8");
  next();
});
```

**Cors请求默认不发送cookie和HTTP认证信息。如果要把cookie发到服务器，一方面要服务器同意，指定Access-Control-Allow-Credentials字段。另一方面，必须在AJAX请求中设置withCredentials为true**

# Client

## Ajax
```js
  var xhr = new XMLHttpRequest();
  xhr.withCredentials = true;
```

## Plupload
```js
var uploader = new plupload.Uploader({
  runtimes : 'html5',
  browse_button : 'pickfiles',
  container: document.getElementById('container'),
  url : 'http://xxx/upload',
  required_features: {
    send_browser_cookies: true
  },
  init: function () {}
})
```

# 参考资料
+ [*CORS 详解*](http://www.ruanyifeng.com/blog/2016/04/cors.html)
+ [*Support withCredentials attribute of XmlHttpRequest*](https://github.com/moxiecode/plupload/pull/1356)