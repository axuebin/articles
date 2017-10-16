---
layout: post
title:  "JavaScript call apply bind"
date:   2017-09-19 13:08:00
categories: 前端
tags: JavaScript
author: 薛彬
---

* content
{:toc}

整理`call`、`apply`、`bind`这三个方法的的知识点。





之前[这篇文章](http://axuebin.com/blog/2017/09/19/javascript-this/)提到过`this`的各种情况，其中有一种情况就是通过`call`、`apply`、`bind`来将`this`绑定到指定的对象上。

也就是说，这三个方法可以改变函数体内部`this`的指向。

这三个方法有什么区别呢？分别适合应用在哪些场景中呢？

先举个简单的栗子 ~ 

```javascript
var person = {
  name: "axuebin",
  age: 25
};
function say(job){
  console.log(this.name+":"+this.age+" "+job);
}
say.call(person,"FE"); // axuebin:25 FE
say.apply(person,["FE"]); // axuebin:25 FE
var sayPerson = say.bind(person,"FE");
sayPerson(); // axuebin:25 FE
```

对于对象`person`而言，并没有`say`这样一个方法，通过`call`/`apply`/`bind`就可以将外部的`say`方法用于这个对象中，其实就是将`say`内部的`this`指向`person`这个对象。

## call

`call`是属于所有`Function`的方法，也就是`Function.prototype.call`。

> The call() method calls a function with a given this value and arguments provided individually.

> call() 方法调用一个函数, 其具有一个指定的this值和分别地提供的参数(参数的列表)。

它的语法是这样的：

```javascript
fun.call(thisArg[,arg1[,arg2,…]]);
```

其中，`thisArg`就是`this`指向，`arg`是指定的参数。

`call`的用处简而言之就是可以让call()中的对象调用当前对象所拥有的function。

### ECMAScript规范

ECMAScript规范中是这样定义`call`的：

当以`thisArg`和可选的`arg1`,`arg2`等等作为参数在一个`func`对象上调用`call`方法，采用如下步骤：

1. 如果`IsCallable(func)`是`false`, 则抛出一个`TypeError`异常。
2. 令`argList`为一个空列表。
3. 如果调用这个方法的参数多余一个，则从`arg1`开始以从左到右的顺序将每个参数插入为`argList`的最后一个元素。
4. 提供`thisArg`作为`this`值并以`argList`作为参数列表，调用`func`的`[[Call]]`内部方法，返回结果。

`call`方法的`length`属性是1。

在外面传入的`thisArg`值会修改并成为`this`值。`thisArg`是`undefined`或`null`时它会被替换成全局对象，所有其他值会被应用`ToObject`并将结果作为`this`值，这是第三版引入的更改。

### 使用call调用函数并且指定this

```javascript
var obj = {
  a: 1
}
function foo(b, c){
  this.b = b;
  this.c = c;
  console.log(this.a + this.b + this.c);
}
foo.call(obj,2,3); // 6
```

### call实现继承

在需要实现继承的子类构造函数中，可以通过`call`调用父类构造函数实现继承。

```javascript
function Person(name, age){
  this.name = name;
  this.age = age;
  this.say = function(){
    console.log(this.name + ":" + this.age);
  }
}
function Student(name, age, job){
  Person.call(this, name ,age);
  this.job = job;
  this.say = function(){
    console.log(this.name + ":" + this.age + " " + this.job);
  }
}
var me = new Student("axuebin",25,"FE");
console.log(me.say()); // axuebin:25 FE
```

## apply

`apply`也是属于所有`Function`的方法，也就是`Function.prototype.apply`。

> The apply() method calls a function with a given this value, and arguments provided as an array (or an array-like object).

> apply() 方法调用一个函数, 其具有一个指定的this值，以及作为一个数组（或类似数组的对象）提供的参数。

它的语法是这样的：

```javascript
fun.apply(thisArg, [argsArray]);
```

其中，`thisArg`就是`this`指向，`argsArray`是指定的参数数组。

通过语法就可以看出`call`和`apply`的在参数上的一个区别：

- `call`的参数是一个列表，将每个参数一个个列出来
- `apply`的参数是一个数组，将每个参数放到一个数组中

### ECMAScript规范

当以`thisArg`和`argArray`为参数在一个`func`对象上调用`apply`方法，采用如下步骤：

1. 如果`IsCallable(func)`是`false`, 则抛出一个`TypeError`异常 .
2. 如果`argArray`是`null`或`undefined`, 则
  1. 返回提供`thisArg`作为`this`值并以空参数列表调用`func`的`[[Call]]`内部方法的结果。
3. 如果`Type(argArray)`不是`Object`, 则抛出一个`TypeError`异常 .
4. 令`len`为以`"length"`作为参数调用`argArray`的`[[Get]]`内部方法的结果。
5. 令`n`为`ToUint32(len)`.
6. 令`argList`为一个空列表 .
7. 令`index`为0.
8. 只要`index`<`n`就重复
  1. 令`indexName`为`ToString(index)`.
  2. 令`nextArg`为以`indexName`作为参数调用`argArray`的`[[Get]]`内部方法的结果。
  3. 将`nextArg`作为最后一个元素插入到`argList`里。
  4. 设定`index`为`index + 1`.
9. 提供`thisArg`作为`this`值并以`argList`作为参数列表，调用`func`的`[[Call]]`内部方法，返回结果。

`apply`方法的`length`属性是 2。

在外面传入的`thisArg`值会修改并成为`this`值。`thisArg`是`undefined`或`null`时它会被替换成全局对象，所有其他值会被应用`ToObject`并将结果作为`this`值，这是第三版引入的更改。

### 用法

在用法上`apply`和`call`一样，就不说了。

### 实现一个apply

参考链接：[https://github.com/jawil/blog/issues/16](https://github.com/jawil/blog/issues/16)

#### 第一步，绑定上下文

```javascript
Function.prototype.myApply=function(context){
  // 获取调用`myApply`的函数本身，用this获取
  context.fn = this;
  // 执行这个函数
  context.fn();
  // 从上下文中删除函数引用
  delete context.fn;
}

var obj ={
  name: "xb",
  getName: function(){
    console.log(this.name);
  }
}

var me = {
  name: "axuebin"
}

obj.getName(); // xb 
obj.getName.myApply(me); // axuebin
```

确实成功地将`this`指向了`me`对象，而不是本身的`obj`对象。

#### 第二步，给定参数

上文已经提到`apply`需要接受一个参数数组，可以是一个类数组对象，还记得获取函数参数可以用`arguments`吗？

```javascript
Function.prototype.myApply=function(context){
  // 获取调用`myApply`的函数本身，用this获取
  context.fn = this;
  // 通过arguments获取参数
  var args = arguments[1];
  // 执行这个函数，用ES6的...运算符将arg展开
  context.fn(...args);
  // 从上下文中删除函数引用
  delete context.fn;
}

var obj ={
  name: "xb",
  getName: function(age){
    console.log(this.name + ":" + age);
  }
}

var me = {
  name: "axuebin"
}

obj.getName(); // xb:undefined
obj.getName.myApply(me,[25]); // axuebin:25
```

`context.fn(...arg)`是用了ES6的方法来将参数展开，如果看过[上面那个链接](https://github.com/jawil/blog/issues/16)，就知道这里不通过`...`运算符也是可以的。

原博主通过拼接字符串，然后用`eval`执行的方式将参数传进`context.fn`中：

```javascript
for (var i = 0; i < args.length; i++) {
  fnStr += i == args.length - 1 ? args[i] : args[i] + ',';
}
fnStr += ')';//得到"context.fn(arg1,arg2,arg3...)"这个字符串在，最后用eval执行
eval(fnStr); //还是eval强大
```

#### 第三步，当传入apply的this为null或者为空时

我们知道，当`apply`的第一个参数，也就是`this`的指向为`null`时，`this`会指向`window`。知道了这个，就简单了~

```javascript
Function.prototype.myApply=function(context){
  // 获取调用`myApply`的函数本身，用this获取，如果context不存在，则为window
  var context = context || window;
  context.fn = this;
  //获取传入的数组参数
  var args = arguments[1];
  if (args == undefined) { //没有传入参数直接执行
    // 执行这个函数
    context.fn()
  } else {
    // 执行这个函数
    context.fn(...args);
  }
  // 从上下文中删除函数引用
  delete context.fn;
}

var obj ={
  name: "xb",
  getName: function(age){
    console.log(this.name + ":" + age);
  }
}

var name = "window.name";

var me = {
  name: "axuebin"
}

obj.getName(); // xb:25
obj.getName.myApply(); // window.name:undefined
obj.getName.myApply(null, [25]); // window.name:25
obj.getName.myApply(me, [25]); // axuebin:25
```

#### 第四步 保证fn函数的唯一性

ES6中新增了一种基础数据类型`Symbol`。

```javascript
const name = Symbol();
const age = Symbol();
console.log(name === age); // false

const obj = {
  [name]: "axuebin",
  [age]: 25
}

console.log(obj); // {Symbol(): "axuebin", Symbol(): 25}
console.log(obj[name]); // axuebin
```

所以我们可以通过`Symbol`来创建一个属性名。

```javascript
var fn = Symbol();
context[fn] = this;
```

#### 完整的apply

```javascript
Function.prototype.myApply=function(context){
  // 获取调用`myApply`的函数本身，用this获取，如果context不存在，则为window
  var context = context || window;
  var fn = Symbol();
  context[fn] = this;
  //获取传入的数组参数
  var args = arguments[1];
  if (args == undefined) { //没有传入参数直接执行
    // 执行这个函数
    context[fn]()
  } else {
    // 执行这个函数
    context[fn](...args);
  }
  // 从上下文中删除函数引用
  delete context.fn;
}
```

这样就是一个完整的`apply`了，我们来测试一下：

```javascript
var obj ={
  name: "xb",
  getName: function(age){
    console.log(this.name + ":" + age);
  }
}

var name = "window.name";

var me = {
  name: "axuebin"
}

obj.getName(); // xb:25
obj.getName.myApply(); // window.name:undefined
obj.getName.myApply(null, [25]); // window.name:25
obj.getName.myApply(me, [25]); // axuebin:25
```

ok 没啥毛病 ~ 

再次感谢[1024大佬](https://github.com/jawil/blog/issues/16) ~

## bind

> The bind() method creates a new function that, when called, has its this keyword set to the provided value, with a given sequence of arguments preceding any provided when the new function is called.

> bind()方法创建一个新的函数, 当被调用时，将其this关键字设置为提供的值，在调用新函数时，在任何提供之前提供一个给定的参数序列。

语法：

```javascript
fun.bind(thisArg[, arg1[, arg2[, ...]]])
```

其中，`thisArg`就是`this`指向，`arg`是指定的参数。

可以看出，`bind`会创建一个新函数（称之为绑定函数），原函数的一个拷贝，也就是说不会像`call`和`apply`那样立即执行。

当这个绑定函数被调用时，它的`this`值传递给`bind`的一个参数，执行的参数是传入`bind`的其它参数和执行绑定函数时传入的参数。

## 用法

当我们执行下面的代码时，我们希望可以正确地输出`name`，然后现实是残酷的

```javascript
function Person(name){
  this.name = name;
  this.say = function(){
    setTimeout(function(){
      console.log("hello " + this.name);
    },1000)
  }
}
var person = new Person("axuebin");
person.say(); //hello undefined
```


这里`this`运行时是指向`window`的，所以`this.name`是`undefined`，为什么会这样呢？看看MDN的解释：

> 由setTimeout()调用的代码运行在与所在函数完全分离的执行环境上。这会导致，这些代码中包含的 this 关键字在非严格模式会指向 window。 

有一个常见的方法可以使得正确的输出：

```javascript
function Person(name){
  this.name = name;
  this.say = function(){
    var self = this;
    setTimeout(function(){
      console.log("hello " + self.name);
    },1000)
  }
}
var person = new Person("axuebin");
person.say(); //hello axuebin
```

没错，这里我们就可以用到`bind`了：

```javascript
function Person(name){
  this.name = name;
  this.say = function(){
    setTimeout(function(){
      console.log("hello " + this.name);
    }.bind(this),1000)
  }
}
var person = new Person("axuebin");
person.say(); //hello axuebin
```

### MDN的Polyfill

```javascript
Function.prototype.bind = function (oThis) {
  var aArgs = Array.prototype.slice.call(arguments, 1)；
  var fToBind = this；
  var fNOP = function () {}；
  var fBound = function () {
    fBound.prototype = this instanceof fNOP ? new fNOP() : fBound.prototype；
    return fToBind.apply(this instanceof fNOP ? this : oThis || this, aArgs )
  }   
  if( this.prototype ) {
    fNOP.prototype = this.prototype；
  }
  return fBound；
}
```

## 总结

- 三者都是用来改变函数的`this`指向
- 三者的第一个参数都是`this`指向的对象
- `bind`是返回一个绑定函数可稍后执行，`call`、`apply`是立即调用
- 三者都可以给定参数传递
- `call`给定参数需要将参数全部列出，`apply`给定参数数组

## 感谢

[不用call和apply方法模拟实现ES5的bind方法](https://github.com/jawil/blog/issues/16)

[深入浅出妙用 Javascript 中 apply、call、bind](http://web.jobbole.com/83642/)

[回味JS基础:call apply 与 bind](https://juejin.im/post/57dc97f35bbb50005e5b39bd)