---
layout: post
title:  "JavaScript this"
date:   2017-09-19 08:21:00
categories: 前端
tags: JavaScript
author: 薛彬
---

* content
{:toc}

看看这个有着深不可测的魔力的`this`到底是个什么玩意儿 ~





## 什么是this

在传统面向对象的语言中，比如Java，`this`关键字用来表示当前对象本身，或当前对象的一个实例，通过`this`关键字可以获得当前对象的属性和调用方法。

在JavaScript中，`this`似乎表现地略有不同，这也是让人“讨厌”的地方~

ECMAScript规范中这样写：

> this 关键字执行为当前执行环境的 ThisBinding。

MDN上这样写：

> In most cases, the value of this is determined by how a function is called.

> 在绝大多数情况下，函数的调用方式决定了this的值。

可以这样理解，在JavaScript中，`this`的指向是调用时决定的，而不是创建时决定的，这就会导致`this`的指向会让人迷惑，简单来说，`this`具有运行期绑定的特性。

参考资料：[this - JavaScript | MDN](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/this "this - JavaScript | MDN")

来看看不同的情况五花八门的`this`吧~

## 调用位置

首先需要理解调用位置，调用位置就是函数在代码中被调用的位置，而不是声明的位置。

通过分析调用栈（到达当前执行位置所调用的所有函数）可以找到调用位置。

```javascript
function baz(){
  console.log("baz");
  bar();
}
function bar(){
  console.log("bar");
  foo();
}
function foo(){
  console.log("foo");
}
baz();
```

当我们调用`baz()`时，它会以此调用`baz()`→`bar()`→`foo()`。

对于`foo()`：调用位置是在`bar()`中。
对于`bar()`：调用位置是在`baz()`中。
而对于`baz()`：调用位置是全局作用域中。

可以看出，调用位置应该是当前正在执行的函数的前一个调用中。

## 全局上下文

在全局执行上下文中`this`都指代全局对象。

- `this`等价于`window`对象
- `var` === `this.` === `winodw.`

```javascript
console.log(window === this); // true
var a = 1;
this.b = 2;
window.c = 3;
console.log(a + b + c); // 6
```

在浏览器里面`this`等价于`window`对象，如果你声明一些全局变量，这些变量都会作为this的属性。

## 函数上下文

在函数内部，`this`的值取决于函数被调用的方式。

### 直接调用

**`this`指向全局变量。**

```javascript
function foo(){
  return this;
}
console.log(foo() === window); // true
```

### call()、apply()

**`this`指向绑定的对象上。**

```javascript
var person = {
  name: "axuebin",
  age: 25
};
function say(job){
  console.log(this.name+":"+this.age+" "+job);
}
say.call(person,"FE"); // axuebin:25
say.apply(person,["FE"]); // axuebin:25
```

可以看到，定义了一个`say`函数是用来输出`name`、`age`和`job`，其中本身没有`name`和`age`属性，我们将这个函数绑定到`person`这个对象上，输出了本属于`person`的属性，说明此时`this`是指向对象`person`的。

如果传入一个原始值（字符串、布尔或数字类型）来当做`this`的绑定对象， 这个原始值会被转换成它的对象形式（`new String()`），这通常被称为“装箱”。

`call`和`apply`从`this`的绑定角度上来说是一样的，唯一不同的是它们的第二个参数。

### bind()

**`this`将永久地被绑定到了`bind`的第一个参数。**

`bind`和`call`、`apply`有些相似。

```javascript
var person = {
  name: "axuebin",
  age: 25
};
function say(){
  console.log(this.name+":"+this.age);
}
var f = say.bind(person);
console.log(f());
```

### 箭头函数

**所有的箭头函数都没有自己的`this`，都指向外层。**

关于箭头函数的争论一直都在，可以看看下面的几个链接：

[ES6 箭头函数中的 this？你可能想多了（翻译）](http://www.cnblogs.com/vajoy/p/4902935.html)

[关于箭头函数this的理解几乎完全是错误的 #150](https://github.com/ruanyf/es6tutorial/issues/150)

MDN中对于箭头函数这一部分是这样描述的：

> An arrow function does not create its own this, the this value of the enclosing execution context is used.

> 箭头函数会捕获其所在上下文的this值，作为自己的this值。

```javascript
function Person(name){
  this.name = name;
  this.say = () => {
    var name = "xb";
    return this.name;
  }
}
var person = new Person("axuebin");
console.log(person.say()); // axuebin
```

箭头函数常用语回调函数中，例如定时器中：

```javascript
function foo() {  
  setTimeout(()=>{
    console.log(this.a);
  },100)
}
var obj = {
  a: 2
}
foo.call(obj);
```

附上MDN关于箭头函数`this`的解释：

[https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Functions/Arrow_functions#不绑定_this](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Functions/Arrow_functions#不绑定_this)

### 作为对象的一个方法

**`this`指向调用函数的对象。**

```javascript
var person = {
  name: "axuebin",
  getName: function(){
    return this.name;
  }
}
console.log(person.getName()); // axuebin
```

这里有一个需要注意的地方。。。

```javascript
var name = "xb";
var person = {
  name: "axuebin",
  getName: function(){
    return this.name;
  }
}
var getName = person.getName;
console.log(getName()); // xb
```

发现`this`又指向全局变量了，这是为什么呢？

还是那句话，`this`的指向得看函数调用时。 

### 作为一个构造函数

**`this`被绑定到正在构造的新对象。**

通过构造函数创建一个对象其实执行这样几个步骤：

1. 创建新对象
2. 将this指向这个对象
3. 给对象赋值（属性、方法）
4. 返回this

所以`this`就是指向创建的这个对象上。

```javascript
function Person(name){
  this.name = name;
  this.age = 25;
  this.say = function(){
    console.log(this.name + ":" + this.age);
  }
}
var person = new Person("axuebin");
console.log(person.name); // axuebin
person.say(); // axuebin:25
```

### 作为一个DOM事件处理函数

**`this`指向触发事件的元素，也就是始事件处理程序所绑定到的DOM节点。**

```javascript
var ele = document.getElementById("id");
ele.addEventListener("click",function(e){
  console.log(this);
  console.log(this === e.target); // true
})
```

### HTML标签内联事件处理函数

**`this`指向所在的DOM元素**

```html
<button onclick="console.log(this);">Click Me</button>
```

### jQuery的this

**在许多情况下JQuery的`this`都指向DOM元素节点。**

```javascript
$(".btn").on("click",function(){
  console.log(this); 
});
```

## 总结

如果要判断一个函数的`this`绑定，就需要找到这个函数的直接调用位置。然后可以顺序按照下面四条规则来判断`this`的绑定对象：

1. 由`new`调用：绑定到新创建的对象
2. 由`call`或`apply`、`bind`调用：绑定到指定的对象
3. 由上下文对象调用：绑定到上下文对象
4. 默认：全局对象

注意：箭头函数不使用上面的绑定规则，根据外层作用域来决定`this`，继承外层函数调用的`this`绑定。