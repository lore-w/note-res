---
title: WEBPACK-DEV-MIDDLEWARE简述
tags:
  - webpack
  - 自动化构建
categories:
  - 手记
date: 2017-07-21 20:12:56
---

<img src="/images/2018/icon-square-big.svg" width="200">

对于WEBPACK-DEV-MIDDLEWARE，最直观简单的理解就是一个运行于内存中的文件系统。

<!-- more -->

你定义了webpack.config，webpack就能依此梳理出所有模块的关系脉络，而WEBPACK-DEV-MIDDLEWARE就在此基础上形成一个微型的文件映射系统。

每当请求一个文件——比如说你定义的某个entry ，它匹配到了就把内存中缓存的结果作为文件内容返回，否则，进入到下一个中间件。

因为是内存型的文件系统，Rebuilding 的速度非常快，这就特别适合于开发阶段用作静态资源服务器。

又因为webpack可以把任何一种 *资源* 都当作是模块来处理，因此它完全可以取代其他的 HTTP 服务器。

事实上，大多数开发者用过的WEBPACK-DEV-SERVER 就是一个EXPRESS＋WEBPACK-DEV-MIDDLEWARE的实现。

二者的区别仅在于WEBPACK-DEV-SERVER是封装好的，除了webpack.config 和命令行参数之外，你很难去做定制型开发。

因此它只适合做前端项目的辅助工具。而 webpack-dev-middleware 是中间件，你可以把它与自己的後端服务整合进来，相对而言自由得多。