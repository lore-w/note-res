---
title: IOS-Scroll
tags:
  - scroll
  - fixed
date: 2017-09-21 16:45:55
---

丷像怎么写就怎么写...

<!-- more -->

```css
* {
  margin: 0;
  padding: 0;
}
html, body {
  width: 100%;
}
.banner {
  display: block;
  position: fixed;
  top: 0;
  width: 100%;
  height: 10rem;
  line-height: 10rem;
  text-align: center;
  background-color: #f81;
}
.container {
  padding-top: 10rem;
  padding-bottom: 3rem;
  width: 100%;
  height: auto;
  background-color: #ccc;
}
.container div {
  height: 40rem;
  line-height: 4rem;
  text-align: center;
  border-bottom: solid 2px gray;
}
.button {
  display: block;
  bottom: 0;
  position: fixed;
  width: 100%;
  height: 3rem;
  line-height: 3rem;
  text-align: center;
  background-color: #00b3ee;
}
```

```html
<body>
  <span class="banner">Banner</span>
  <div class="container">
    <div>项目0</div>
    <div>项目1</div>
    <div>项目2</div>
    <div>项目3</div>
    <div>项目4</div>
    <div>项目5</div>
  </div>
  <span class="button">按钮</span>
</body>
```

`http://lore-w.com/Demo-Preview/ios-scroll/normal.html`
### 表现完美，滚动的是html，body，页面中只存在一个滚动区域

```css
* {
  margin: 0;
  padding: 0;
}
html, body, #app, #index-page {
  width: 100%;
}

.banner {
  display: block;
  position: fixed;
  top: 0;
  width: 100%;
  height: 10rem;
  line-height: 10rem;
  text-align: center;
  z-index: 1;
  background-color: #f81;
}
.container {
  padding-top: 10rem;
  padding-bottom: 3rem;
  width: 100%;
  background-color: #ccc;
}
.container div {
  height: 10rem;
  line-height: 4rem;
  text-align: center;
  border-bottom: solid 2px gray;
}

.button {
  display: block;
  bottom: 0;
  position: fixed;
  width: 100%;
  height: 3rem;
  line-height: 3rem;
  text-align: center;
  background-color: #00b3ee;
}
```

```html
<body>
  <div id="app">
  <div id="index-page">
    <span class="banner">banner</span>
    <div class="container">
      <div>项目0</div>
      <div>项目1</div>
      <div>项目2</div>
      <div>项目3</div>
      <div>项目4</div>
      <div>项目5</div>
    </div>
    <span class="button">按钮</span>
  </div>
</div>
</body>
```

`http://lore-w.com/Demo-Preview/ios-scroll/spa-normal.html`
###  表现完美，滚动的是html，body，页面中只存在一个滚动区域

```css
* {
  margin: 0;
  padding: 0;
}
html, body, #app, #index-page {
  width: 100%;
  height: 100%;
  overflow: hidden;
}

.banner {
  display: block;
  position: absolute;
  top: 0;
  width: 100%;
  height: 10rem;
  line-height: 10rem;
  text-align: center;
  z-index: 1;
  background-color: #f81;
}
.container {
  padding-top: 10rem;
  padding-bottom: 3rem;
  width: 100%;
  background-color: #ccc;

  height: 100%;
  overflow-y: scroll;
  -webkit-overflow-scrolling: touch;
}
.container div {
  height: 10rem;
  line-height: 4rem;
  text-align: center;
  border-bottom: solid 2px gray;
}

.button {
  display: block;
  bottom: 0;
  position: absolute;
  width: 100%;
  height: 3rem;
  line-height: 3rem;
  text-align: center;
  background-color: #00b3ee;
}
```

```html
<body>
<div id="app">
  <div id="index-page">
    <div class="container">
      <span class="banner">banner</span>
      <div>项目0</div>
      <div>项目1</div>
      <div>项目2</div>
      <div>项目3</div>
      <div>项目4</div>
      <div>项目5</div>
      <span class="button">按钮</span>
    </div>
  </div>
</div>
</body>
```

`http://lore-w.com/Demo-Preview/ios-scroll/spa-absolute-scroll-div-inner.html`
### 完全不符合需求，banner和button会随着页面滚动

```css
* {
  margin: 0;
  padding: 0;
}
html, body, #app, #index-page {
  width: 100%;
  height: 100%;
  overflow: hidden;
}

.banner {
  display: block;
  position: absolute;
  top: 0;
  width: 100%;
  height: 10rem;
  line-height: 10rem;
  text-align: center;
  z-index: 1;
  background-color: #f81;
}
.container {
  padding-top: 10rem;
  padding-bottom: 3rem;
  width: 100%;
  background-color: #ccc;

  height: 100%;
  overflow-y: scroll;
  -webkit-overflow-scrolling: touch;
}
.container div {
  height: 10rem;
  line-height: 4rem;
  text-align: center;
  border-bottom: solid 2px gray;
}

.button {
  display: block;
  bottom: 0;
  position: absolute;
  width: 100%;
  height: 3rem;
  line-height: 3rem;
  text-align: center;
  background-color: #00b3ee;
}
```

```html
<body>
  <div id="app">
  <div id="index-page">
    <span class="banner">banner</span>
    <div class="container">
      <div>项目0</div>
      <div>项目1</div>
      <div>项目2</div>
      <div>项目3</div>
      <div>项目4</div>
      <div>项目5</div>
    </div>
    <span class="button">按钮</span>
  </div>
</div>
</body>
```

`http://lore-w.com/Demo-Preview/ios-scroll/spa-absolute-scroll-div-outer.html`

### 存在两个问题：
+ 1、从banner下拉时会触发页面级滚动，并且在页面级滚动的趋势未完成之前，container不能滚动
+ 2、从banner下拉时会触发页面级滚动，此时banner会下拉，露出微信的黑色背景（这个可以不是大问题）

解决这个问题（2）其实可以在html，body上加fixed定位，根本上消除下拉问题，但是不能解决问题（1）


```css
* {
  margin: 0;
  padding: 0;
}
html, body, #app, #index-page {
  width: 100%;
  height: 100%;
  overflow: hidden;
}

.banner {
  display: block;
  position: fixed;
  top: 0;
  width: 100%;
  height: 10rem;
  line-height: 10rem;
  text-align: center;
  z-index: 1;
  background-color: #f81;
}
.container {
  padding-top: 10rem;
  padding-bottom: 3rem;
  width: 100%;
  background-color: #ccc;

  height: 100%;
  overflow-y: scroll;
  -webkit-overflow-scrolling: touch;
}
.container div {
  height: 10rem;
  line-height: 4rem;
  text-align: center;
  border-bottom: solid 2px gray;
}

.button {
  display: block;
  bottom: 0;
  position: fixed;
  width: 100%;
  height: 3rem;
  line-height: 3rem;
  text-align: center;
  background-color: #00b3ee;
}
```

```html
<body>
<div id="app">
  <div id="index-page">
    <div class="container">
      <span class="banner">banner</span>
      <div>项目0</div>
      <div>项目1</div>
      <div>项目2</div>
      <div>项目3</div>
      <div>项目4</div>
      <div>项目5</div>
      <span class="button">按钮</span>
    </div>
  </div>
</div>
</body>
```

`http://lore-w.com/Demo-Preview/ios-scroll/spa-fix-scroll-div-inner.html`

###  存在两个问题：
+ 1、从banner下拉时会触发页面级滚动，并且在页面级滚动的趋势未完成之前，container不能滚动
+ 2、在下拉滚动的过程中，banner和button有可能消失

```css
* {
  margin: 0;
  padding: 0;
}
html, body, #app, #index-page {
  width: 100%;
  height: 100%;
  overflow: hidden;
}

.banner {
  display: block;
  position: fixed;
  top: 0;
  width: 100%;
  height: 10rem;
  line-height: 10rem;
  text-align: center;
  z-index: 1;
  background-color: #f81;
}
.container {
  padding-top: 10rem;
  padding-bottom: 3rem;
  width: 100%;
  background-color: #ccc;

  height: 100%;
  overflow-y: scroll;
  -webkit-overflow-scrolling: touch;
}
.container div {
  height: 10rem;
  line-height: 4rem;
  text-align: center;
  border-bottom: solid 2px gray;
}

.button {
  display: block;
  bottom: 0;
  position: fixed;
  width: 100%;
  height: 3rem;
  line-height: 3rem;
  text-align: center;
  background-color: #00b3ee;
}
```

```html
<body>
  <div id="app">
  <div id="index-page">
    <span class="banner">banner</span>
    <div class="container">
      <div>项目0</div>
      <div>项目1</div>
      <div>项目2</div>
      <div>项目3</div>
      <div>项目4</div>
      <div>项目5</div>
    </div>
    <span class="button">按钮</span>
  </div>
</div>
</body>
```

`http://lore-w.com/Demo-Preview/ios-scroll/spa-fix-scroll-div-outer.html`

### 存在一个问题：
+ 1、从banner下拉时会触发页面级滚动，并且在页面级滚动的趋势未完成之前，container不能滚动


```css
* {
  margin: 0;
  padding: 0;
}
html, body, #app, #index-page {
  width: 100%;
  height: 100%;
  overflow: hidden;
}

.banner {
  display: block;
  position: fixed;
  top: 0;
  width: 100%;
  height: 10rem;
  line-height: 10rem;
  text-align: center;
  z-index: 1;
  background-color: #f81;
}
.container {
  padding-top: 10rem;
  padding-bottom: 3rem;
  width: 100%;
  background-color: #ccc;

  height: 100%;
  overflow-y: scroll;
  /*-webkit-overflow-scrolling: touch;*/
}
.container div {
  height: 10rem;
  line-height: 4rem;
  text-align: center;
  border-bottom: solid 2px gray;
}

.button {
  display: block;
  bottom: 0;
  position: fixed;
  width: 100%;
  height: 3rem;
  line-height: 3rem;
  text-align: center;
  background-color: #00b3ee;
}
```

```html
<body>
<div id="app">
  <div id="index-page">
    <span class="banner">banner</span>
    <div class="container">
      <div>项目0</div>
      <div>项目1</div>
      <div>项目2</div>
      <div>项目3</div>
      <div>项目4</div>
      <div>项目5</div>
    </div>
    <span class="button">按钮</span>
  </div>
</div>
</body>
```

`http://lore-w.com/Demo-Preview/ios-scroll/spa-fix-no-scroll-div-outer.html`

### 存在一个问题：
+ 1、container滚动卡顿

so,在这个基础上加一个iscroll吧

`http://lore-w.com/Demo-Preview/ios-scroll/spa-iscroll.html`
