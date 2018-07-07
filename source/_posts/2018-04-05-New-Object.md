---
title: Js对象
tags:
  - 手记
  - js
  - 对象
categories:
  -
date: 2018-04-07 11:40:37
---


# 对象的创建

## 第一种 Object
```js
var obj = new Object()
obj.name = 'Lore-w'
obj.age = 28
```

> 由于没有类的约束，无法实现对象的重复利用

<!-- more -->

## 第二种 Json
```js
var obj = {
  name: 'Lore-w',
  age: 28
}
```
> Json的意思就是javascript simple object notation
> Json就是js的对象，但他省去了xml中的标签，而是通过{}来完成对象的说明

## 第三种 Factory

```js
function createPerson(name, age) {
  var o = new Object()
  o.name = name
  o.age = age
  o.say = function () {
    alert(this.name)
  }
  return o
}
var o1 = createPerson('tom', 12)
var o1 = createPerson('kim', 18)
alert(typeof o1) // object
alert(typeof o2) // object

// alert(o1 instanceof ??)
```

> 虽然有效的解决了类的问题，但是依然存在另外一个问题
> 无法检测对象的类型

## 第四种 构造函数

```js
function Person(name, age) {
  this.name = name
  this.age = age
  this.say = function () {
    alert(this.name)
  }
}
var o1 = new Person('tom', 12)
var o1 = new Person('kim', 18)
alert(o1 instanceof Person) // true
alert(o1.say == o2.say) // false

```

> 使用构造函数可以通过instanceof来检测对象的类型，但是却带来了另外一个问题
> 每一个对象中都会存在一个方法的拷贝
> 可以将函数放到window中定义，这样可以让类中的行为指向同一个函数

```js
function Person(name, age) {
  this.name = name
  this.age = age
  this.say = say
}
function say() {
  alert(this.name)
}
var o1 = new Person('tom', 12)
var o1 = new Person('kim', 18)
alert(o1 instanceof Person) // true
alert(o1.say == o2.say) // true

// 这里其实有一个疑问？？既然函数是通过拷贝完成赋值的，o1.say 和 o2.say为什么会相等？
```

> 如果将方法都设计为*全局*函数，那么函数就可以*被*window调用，此时就*破坏*了对象的封装性
> 而且如果某个对象有大量的方法，就会导致整个代码中有大量的*全局*函数

## 第四种 Prototype

```js
// 第一种状态
function Person() {

}
// 第二种状态
Person.prototype.name = 'Lore-w'
Person.prototype.age = 28
Person.prototype.say = function () {
  alert(this.name)
}
// 第三种状态
var lore = new Person()
lore.say()
alert(Person.prototype.isPrototypeOf(lore)) // true
alert('name' in lore) // true

// 通过window没有办法调用say方法，如此完成了封装
```