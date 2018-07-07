---
title: Jsonp为什么可以跨域请求?
tags:
  - 草稿
  - 跨域
  - Jsonp
categories:
  -
date: 2017-08-16 23:40:08
---

# 为什么可以跨域请求?

首先jsonp请求不是xhr请求，其实是浏览器请求了一个文件（不会存在跨域问题），在请求的url中携带参数，当服务端收到这个请求时

通过url上的参数查询DB，用url中的callback包裹data，返回给前端，因此前端接收到的数据是这个样子的：

<!-- more -->

```js
xxxcallback({key: value})
```

同时，前端在发送请求的时候就在window对象下定义了一个xxxcallback的方法，大概是这个样子的：

```js
window.xxxcallback(data) {
  console.log(data);
}
```

这样其实就相当于，相当于什么...不说了...

## window对象污染问题

  通过上面的思考可以知道，多次请求以後，window对象上其实会有多个callback，因此，我们还得有一个销毁得过程

# jq中如何使用

在jq中使用jsonp跨域请求非常简单，只需在options参数中配置：

`jsonp: jsonpCallback`即可

注意：这里的jsonpCallback是一个回调函数的名称，你可以定义成任何一个你喜欢的名字，如abc，noCallback等等等...


# vue-resource中如何使用

{% cq %} 等一下...我们得尊重一下服务端得同学 {% endcq %}


关于jsonp的请求，服务端提供的接口可能有如下两种形式：

+ 1、`xxxapi/user.do?userId=user_1&callback=callback`
+ 2、`xxxapi/user/{userId}-{callback}.html`

以上接口最主要得区别，其实就是服务端获取callback的方式不同

第一种，通过key value，获取callback

第二种，通过正则，字符串截取，获取callback

**问题来了：**

**在**前端，第一种方式的callback传递，基本都有框架内部随机生成，因此每次请求的url其实都不一样

**而**，第二种方式的callback传递，基本都有开发者拼接url而成，如果不做随机处理，每次请求的url其实都一样，缓存问题是显而易见的

**也**，为第二种接口在连续请求的情况下出现xxxCallback is not defined给出了合理的解释，（结合window对象污染问题，很容易得到答案...）


有了上面的基础，可以开始表演了

## 对于第一种，我们可以这么写

```js
Vue.http.jsonp("xxxapi/user.do", {
  params: {
    userId: user_1
  }
})
```

这时vue-resource会解析params中的参数，并生成一个随机的callback，拼接到url後面，

如(`xxxapi/user.do?userId=user_1&callback=_jsonpv4d5xxae`)

## 对于第二种，其实是需要自定义回调的接口

在官方的文档中，并没有找到自定义callback的方法，网上大多是说需要自定义`jsonp`参数，我试了并不可以，可能是版本的问题吧，

*最後在源码*(vue-resource/src/http/client/jsonp.js)中找到了答案：

```js
var name = request.jsonp || 'callback',
  callback = request.jsonpCallback || '_jsonp' + Math.random().toString(36).substr(2),
  body = null,
  handler,
  script;
```
通过上述代码可以知道vue-resource中自定义callback的参数为jsonpCallback


```js
Vue.http.jsonp("xxxapi/user/user_1-customCallback.html", {
  jsonpCallback: 'customCallback'
})
```

不过这里大家需要注意的是：

Url（xxxapi/user/user_1-customCallback.html）中的customCallback

和配置项（{jsonpCallback: 'customCallback'}）中的customCallback作用是不同的，

Url中的customCallback是通知服务端用这个callback包裹data

而配置项中的customCallback是通知vue-resource在window下定义一个方法

{% cq %} 转载请注明出处(┬＿┬) {% endcq %}