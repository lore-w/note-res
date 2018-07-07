---
title: Vue Transition
tags:
  - 手记
  - Vue
categories:
  -
date: 2017-07-21 17:12:56
---

# 官网demo
```html
<div id="demo">
  <button v-on:click="show = !show">
    Toggle
  </button>
  <transition name="fade">
    <p v-if="show">hello</p>
  </transition>
</div>
```

```js
new Vue({
  el: '#demo',
  data: {
    show: true
  }
})
```

```css
.fade-enter-active, .fade-leave-active {
  transition: opacity .5s
}
.fade-enter, .fade-leave-to /* .fade-leave-active in <2.1.8 */ {
  opacity: 0
}
```

# css3 demo
```css
.normal {
	opacity: 1;
	transition: opacity 3s;
}

.transition {
	opacity: 0;
}
```

![](/images/2017/71500632452_.pic_hd.jpg)

# 总结

在Vue中v-enter表示动画的开始状态，结合本例表示元素从透明度0过渡到1（元素默认显示，透明度为1）即设定一个初始值过渡到正常值，而在css3中正常情况下元素透明度是1，是从正常值过渡到一个设定值