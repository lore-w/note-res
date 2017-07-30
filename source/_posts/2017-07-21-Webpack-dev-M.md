---
title: Webpack-dev-middleware
tags:
  - webpack
  - 工具
date: 2017-07-21 20:12:56
---

对于 webpack-dev-middleware，最直观简单的理解就是一个运行于内存中的文件系统。

*你定义了 webpack.config，webpack 就能据此梳理出所有模块的关系脉络，而webpack-dev-middleware 就在此基础上形成一个微型的文件映射系统，每当程序请求一个文件——比如说你定义的某个 entry ，它匹配到了就把内存中缓存的对应结果作为文件内容返回，反之则进入到下一个中间件。

因为是内存型的文件系统，所以 rebuilding 的速度非常快，因此特别适合于开发阶段用作静态资源服务器；又因为 webpack 可以把任何一种资源都当作是模块来处理，因此它完全可以取代其他的 HTTP 服务器。事实上，大多数 webpack 用户用过的 webpack-dev-server 就是一个 express＋webpack-dev-middleware 的实现。二者的区别仅在于 webpack-dev-server 是封装好的，除了 webpack.config 和命令行参数之外，你很难去做定制型开发，所以它是适合纯前端项目的辅助工具。而 webpack-dev-middleware 是中间件，你可以编写自己的后端服务然后把它整合进来，相对而言就自由得多了。*