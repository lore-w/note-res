---
title: Prototype分析
tags:
  - js
  - prototype
categories:
  - 手记
date: 2018-04-07 15:34:30
---

# 内存模型

先来一点点点代码

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

> ***原型***是js中非常特殊的一个对象，当一个函数创建之後，会随之*产生*一个*原型*对象
> 当通过这个函数的构造函数创建一个具体的对象之後，在这个具体的对象中就会有一个*属性*指向*原型*

<!-- more -->

**状态分析**

## 第一种状态

当定义`function Person() {}`的时候

> Person函数中有一个`prototype`*属性*指向Person的*原型*对象
> 在*原型*对象中有一个`constructor`*属性*指向了Person函数
> 因此可以通过`new Person()`创建对象

内存模型如下：
{% fullimage /images/2018/Prototype_01.jpg, alt, title %}

## 第二种状态

当通过`Person.prototype.name`为*原型*设置*属性*之後

内存模型如下：
{% fullimage /images/2018/Prototype_02.jpg, alt, title %}

## 第三种状态

当使用`Person`创建了对象之後

> 当使用`Person`创建了对象之後，会在对象（实例）中有一个`__prop__`*属性*（不可访问的）
> 指向*原型*，当访问对象*属性*的时候，首先会在对象的内部找是否有这个*属性*
> 如果没有会通过`__prop__`去*原型*中找

内存模型如下：
{% fullimage /images/2018/Prototype_03.jpg, alt, title %}

## 第四种状态

当使用`Person`创建了一个新的对象之後

> 当使用`Person`创建了一个新的对象（tom）之後，依然会有一个`__prop__`*属性*
> 指向Person的*原型*，此时如果通过tom.name设置了*属性*之後
> 会在对象自己的内存空间中存储name的值，当调用say方法的时候
> 在自己的空间中找到name後，就不会去*原型*中查找了
>

内存模型如下：
{% fullimage /images/2018/Prototype_04.jpg, alt, title %}

# 重写

```js
function Person () {

}
Person.prototype = {
  name: 'Lore-w',
  age: 28,
  say: function () {
    alert(this.name)
  }
}
var lore = new Person()
lore.say()
alert(lore.constructor == Person) // false

```

> 通过上面的方式将会重写*原型*
> 由于*原型*重写，而且没有通过`Person.prototype`来指定
> 此时的constructor不会再指向Person而是指向Object
> 如果constructor比较重要，可以在Json中说明*原型*的指向

## 有何问题

```js
function Person () {

}
Person.prototype = {
  constructor: Person,
  name: 'Lore-w',
  age: 28,
  say: function () {
    alert(this.name)
  }
}
var lore = new Person()
Person.prototype.sayHi = function () {
  alert(this.name + ':Hi')
}

lore.sayHi() // lore-w:Hi

```
这个当然是没问题的啦...

再来一个
```js
function Person () {

}

var lore = new Person()
Person.prototype.sayHi = function () {
  alert(this.name + ':Hi')
}
Person.prototype = {
  constructor: Person,
  name: 'Lore-w',
  age: 28,
  say: function () {
    alert(this.name)
  }
}

lore.sayHi() // undefined

var tom = new Person()

tom.sayHi() // 报错
```

内存模型如下：
{% fullimage /images/2018/Prototype_05.jpg, alt, title %}

# 有何问题

```js
function Person () {

}
Person.prototype = {
  constructor: Person,
  name: 'Leon',
  age: 28,
  friends: ['Ada', 'Chris'],
  say: function () {
    alert(this.name + '[' + this.friends + ']')
  }
}
var lore = new Person()
lore.name = 'Lore-w'
lore.say() //Lore-w + ['Ada', 'Chris']

lore.friends.push('Mike')

var kimi = new Person()
kimi.name = 'Kimi'
kimi.say() // //Kimi + ['Ada', 'Chris', 'Mike']

```

# 解决：混合模式

> 将*属性*在构造函数中定义，将方法在*原型*中定义
> 这种方式有效的集合了构造函数和*原型*的优点，是目前常用的一种方式

```js
function Person (name, age, friends) {
  this.name = name
  this.age = age
  this.friends = friends
}
Person.prototype = {
  constructor: Person,
  say: function () {
    alert(this.name + '[' + this.friends + ']')
  }
}
var lore = new Person('Lore-w', 28, ['Ada', 'Chris'])
lore.say() //Lore-w + ['Ada', 'Chris']

lore.friends.push('Mike')

var kimi = new Person('Kimi', 20, ['Ada', 'Leon'])
kimi.say() // //Kimi + ['Ada', 'Leon']

```

# 更进一步：动态Prototype

```js
function Person (name, age, friends) {
  this.name = name
  this.age = age
  this.friends = friends

  if (!Person.prototype.say) {
    Person.prototype.say = function () {
      alert(this.name)
    }
  }
}

```