---
title: About jsonp
tags:
  - jsonp
  - vue
date: 2017-08-16 23:40:08
---

### jsonp为什么可以跨域请求?

我们知道在Jquery中使用jsonp跨域请求需要在options参数中配置`jsonp: jsonpCallback`这里的jsonpCallback其实是一个回调函数的名称，你可以定义成任何一个你喜欢的名字，比如abc，noCallback等等等...

<!-- more -->

*然而，了解vue-resource的同学都知道，在vue-resource中可以显示的定义callback也可以不定义，这到底是为什么呢？*

*要说清楚这个问题还得先从jsonp的原理说起*

*首先jsonp请求不是xhr请求，本质上是浏览器请求了一个文件（不会存在跨域问题），在请求的url中携带参数，当服务端收到这个请求时，根据url上的参数查询数据库，然后用url中的callback包裹数据，返回给前端，所以前端接收到的数据是这个样子的：*

```js
xxxcallback({key: value})
```

*同时，前端在发送请求的时候就在window对象下定义了一个xxxcallback的方法，大概是这个样子的：*

```js
window.xxxcallback(data) {
  console.log(data);
}
```

*这样其实就相当于，服务端返回数据的时候调用了前端预先定义好的方法*

### 说说vue-resource中的jsonp请求。

**关于jsonp的请求服务端提供的接口可能有如下两种形式**

+ 1、`xxxapi/user.do?userId=user_1&callback=callback`
+ 2、`xxxapi/user/{userId}-{callback}.html`

#### 对于第一种，我们可以这么写

```js
Vue.http.jsonp("xxxapi/user.do", {
  params: {
    userId: user_1
  }
})
```

*这时vue-resource会解析params中的参数，并生成一个随机的callback，拼接到url后面，如(`xxxapi/user.do?userId=user_1&callback=_jsonpv4d5xxae`)，服务端收到请求后会用`_jsonpv4d5xxae`包裹数据返回，这种形式的相对简单*

#### 对于第二种形式的接口，其实是需要自定义回调的接口

*在官方的文档中并没有找到自定义callback的方法，网上大多是说需要自定义`jsonp`参数，我试了并不可以，可能是版本的问题吧，最后在源码(
vue-resource/src/http/client/jsonp.js)中找到了答案：*

```js
var name = request.jsonp || 'callback',
    callback = request.jsonpCallback || '_jsonp' + Math.random().toString(36).substr(2),
    body = null,
    handler,
    script;
```

*与jquery中稍有不同*

```js
Vue.http.jsonp("xxxapi/user/user_1-customCallback.html", {
  jsonpCallback: 'customCallback'
})
```

*这里需要注意的是`xxxapi/user/user_1-customCallback.html`中的customCallback和` jsonpCallback: 'customCallback'`中的customCallback其实不是一回事。*

`xxxapi/user/user_1-customCallback.html`*是告诉服务端，你需要用customCallback包裹数据*

`jsonpCallback: 'customCallback'`*是告诉vue-resource，需要在前端定义一个customCallback的方法，等待服务返回调用*

#### xxxCallback is not defined

*Last：提一个在开发中可能遇到的报错，如果符合下面的两个特征基本可以确定是因为接口重复调用的问题（即，在前一个请求还没有返回的情况下又发起了同样一个请求）*

+ 1、*通过network可以查到服务端数据已经成功返回*
+ 2、*该接口需要自定义callback*

*经过分析，出现这类的报错的原因是因为，在第一次请求未返回的情况下再次发起同样的请求，那么第二次请求的时候会把第一次定义的同名的callback覆盖掉，当第一次服务返回数据时，调用了一个不存在的callback*
