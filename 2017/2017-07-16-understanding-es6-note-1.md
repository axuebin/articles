---
layout: post
title:  "ES6变量命名方式以及块级作用域"
date:   2017-07-16 18:30:54
categories: 前端
tags: ES6
author: 薛彬
---

* content
{:toc}


买的《深入理解es6》终于到手了，赶紧看起来。。




## var声明及变量提升机制

在ES6之前，在函数作用域中或者全局作用域中通过`var`关键字来声明变量，无论是在代码的哪个位置，这条声明语句都会提到最顶部来执行，这就是变量声明提升。

注意：**只是声明提升，初始化并没有提升。**

看一个例子：

```javascript
function getStudent(name){
  if(name){
    var age=25;
  }else{
    console.log("name不存在");      
  }
  console.log(age); //undefined
}
```

如果按照预想的代码的执行顺序，当`name`有值时才会创建变量`age`，可是执行代码发现，即使不传入`name`，判断语句外的输出语句并没有报错，而是输出`undefined`。

这就是变量声明提升。

## 块级声明

ES6前是没有块级作用域的，比如`{}`外可以访问内部的变量。

### let声明

- 声明变量
- 作用域限制在当前代码块
- 声明不会提升
- 禁止重声明（同一作用域不行，可以覆盖外部同名变量）

```javascript
function getStudent(name){
  if(name){
    let age=25;
    console.log(age); //25
  }else{
    console.log("name不存在");      
  }
  console.log(age); //age is not defined
}
```

和上文一样的代码，只是将`age`的命名关键字从`var`改成了`let`，在执行`getStudent()`和`getStudent("axuebin")`时都会报错。

原因：

- 在if语句内部执行之后，`age`变量将立即被销毁
- 如果`name`为空，则永远都不会创建`age`变量

### const声明

- 声明常量
- 必须初始化
- 不可更改
- 作用域限制在当前代码块
- 声明不会提升
- 禁止重声明（同一作用域不行，可以覆盖外部同名变量）

如果用`const`来声明对象，则**对象中**的值可以修改。

### 临时死区（Temporal Dead Zone）

JavaScript引擎在扫面代码发现声明变量时，遇到`var`则提升到作用域顶部，遇到`let`和`const`则放到TDZ中。当执行了变量声明语句后，TDZ中的变量才能正常访问。

## 循环中的块作用域绑定

我们经常使用for循环：

```javascript
for(var i=0;i<10;i++){
  console.log(i); //0,1,2,3,4,5,6,7,8,9
}
console.log(i) //10
```

发现了什么？

在for循环执行后，我们仍然可以访问到变量`i`。

So easy ~ 把`var`换成`let`就解决了~

```javascript
for(let i=0;i<10;i++){
  console.log(i); //0,1,2,3,4,5,6,7,8,9
}
console.log(i) //i is not defined
```

还记得当初讲闭包时setTimeout循环各一秒输出i的那个例子吗~

曾经熟悉的你 ~ 

```javascript
for(var i=0;i<10;i++){
  setTimeout(function(){
    console.log(i); //10,10,10.....
  },1000)
}
```

很显然，上面的代码输出了10次的10，`setTimeout`在执行了循环之后才执行，此时`i`已经是10了~

之前，我们这样做 ~

```javascript
for(var i=0;i<10;i++){
  setTimeout((function(i){
    console.log(i); //0,1,2,3,4,5,6,7,8,9
  })(i),1000)
}
```

现在，我们这样做 ~ 来看看把`var`改成`let`会怎样~

```javascript
for(let i=0;i<10;i++){
  setTimeout(function(){
    console.log(i); //0,1,2,3,4,5,6,7,8,9
  },1000)
}
```

nice~

## 全局块作用域绑定

在全局作用域下声明的时

- `var`会覆盖window对象中的属性
- `let`和`const`会屏蔽，而不是覆盖，用`window.`还能访问到