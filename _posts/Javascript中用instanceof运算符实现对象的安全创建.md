---
title: Javascript中用instanceof运算符实现对象的安全创建
date: 2016-10-27 20:30:00
tags:
- instanceof
categories:
- JavaScript
---

```javascript
var Book = function(title, time, type){
  if(this instanceof Book){
    this.title = title;
    this.time = time;
    this.type = type;
  } else{
    return new Book(title, time, type);
  }
}
```
<!-- more -->


## 什么叫对象的安全创建

假设我们有以下一个类
```javascript
var Book = function(title, time, type){
  this.title = title;
  this.time = time;
  this.type = type;
}
```
我们用下面代码去实例化一个Book
```javascript
var book = Book('javascipt', '2010', 'js');
```
当我们查看book对象时，会发现book对象 undefined
```javascript
console.log(book);  //undefined
```

为什么会出现这种情况呢？原因就是我们在实例化book对象没有用`new`关键词。我们再试一下
```javascript
var book = new Book('javascipt', '2010', 'js');
```
查看
```javascript
console.log(book);
//得到 Book {title: "javascipt", time: "2010", type: "js"}
```

那么又没有办法可以在我们没有用 `new` 关键词还能安全的创建对象，而不会出现 `undefined` 的情况。即，对象的安全创建。

## 对象的安全创建的实现

首先，我们会用到 javascript 中的 instanceof 运算符。

### instanceof
> instanceof 用来检测某个对象是否是某个类的实例。

示例1：判断 foo 是否是 Foo 类的实例
```javascript
function Foo(){}
var foo = new Foo();
console.log(foo instanceof Foo)  //true
```

示例2：instanceof 在继承中关系中的用法(判断 foo 是否是 Foo 类的实例 , 并且是否是其父类型的实例)

```javascript
function Aoo(){}
function Foo(){}
Foo.prototype = new Aoo();    //JavaScript 原型继承

var foo = new Foo();
console.log(foo instanceof Foo) //true
console.log(foo instanceof Aoo) //true
```

Tips: instanceof 用来检测某个对象是否是某个类的实例。

所以
```javascript
console.log(Foo instanceof Aoo)  //false
```
但是`Foo.prototype` 继承了 `Aoo` ,所以
```javascript
console.log(Foo.prototype instanceof Aoo)  //true
```

**希望我们不要弄混~**


另，Javascript为我们提供的原生对象 Object，所有的对象都可以说成是Object的子类。
```javascript
console.log(instance instanceif Object);  //true
```

### 实现

```javascript
var Book = function(title, time, type){
  if(this instanceof Book){
    this.title = title;
    this.time = time;
    this.type = type;
  } else{
    return new Book(title, time, type);
  }
}
```

测试

```javascript
var book = Book('javascipt', '2010', 'js');
console.log(book);
//得到：Book {title: "javascipt", time: "2010", type: "js"}

var book1 = new Book('javascipt_1', '2010', 'js');
console.log(book1)
//得到：Book {title: "javascipt_1", time: "2010", type: "js"}
```

over~
