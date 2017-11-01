浅拷贝和深拷贝都是对于JS中的引用类型而言的，浅拷贝就只是复制对象的引用，如果拷贝后的对象发生变化，原对象也会发生变化。只有深拷贝才是真正地对对象的拷贝。

## 前言

说到深浅拷贝，必须先提到的是JavaScript的数据类型，之前的一篇文章[JavaScript基础心法——数据类型](https://github.com/axuebin/articles/issues/3)说的很清楚了，这里就不多说了。

需要知道的就是一点：JavaScript的数据类型分为基本数据类型和引用数据类型。

对于基本数据类型的拷贝，并没有深浅拷贝的区别，我们所说的深浅拷贝都是对于引用数据类型而言的。

## 浅拷贝

浅拷贝的意思就是只复制引用，而未复制真正的值。

```javascript
const originArray = [1,2,3,4,5];
const originObj = {a:'a',b:'b',c:[1,2,3],d:{dd:'dd'}};

const cloneArray = originArray;
const cloneObj = originObj;

console.log(cloneArray); // [1,2,3,4,5]
console.log(originObj); // {a:'a',b:'b',c:Array[3],d:{dd:'dd'}}

cloneArray.push(6);
cloneObj.a = {aa:'aa'};

console.log(cloneArray); // [1,2,3,4,5,6]
console.log(originArray); // [1,2,3,4,5,6]

console.log(cloneObj); // {a:{aa:'aa'},b:'b',c:Array[3],d:{dd:'dd'}}
console.log(originArray); // {a:{aa:'aa'},b:'b',c:Array[3],d:{dd:'dd'}}
```

上面的代码是最简单的利用 `=` 赋值操作符实现了一个浅拷贝，可以很清楚的看到，随着 `cloneArray` 和 `cloneObj` 改变，`originArray` 和 `originObj` 也随着发生了变化。

## 深拷贝

深拷贝就是对目标的完全拷贝，不像浅拷贝那样只是复制了一层引用，就连值也都复制了。

只要进行了深拷贝，它们老死不相往来，谁也不会影响谁。

目前实现深拷贝的方法不多，主要是两种：

1. 利用 `JSON` 对象中的 `parse` 和 `stringify`
2. 利用递归来实现每一层都重新创建对象并赋值

### JSON.stringify/parse的方法

先看看这两个方法吧：

> The JSON.stringify() method converts a JavaScript value to a JSON string.

`JSON.stringify` 是将一个 `JavaScript` 值转成一个 `JSON` 字符串。

> The JSON.parse() method parses a JSON string, constructing the JavaScript value or object described by the string.

`JSON.parse` 是将一个 `JSON` 字符串转成一个 `JavaScript` 值或对象。

很好理解吧，就是 `JavaScript` 值和 `JSON` 字符串的相互转换。

它能实现深拷贝呢？我们来试试。

```javascript
const originArray = [1,2,3,4,5];
const cloneArray = JSON.parse(JSON.stringify(originArray));
console.log(cloneArray === originArray); // false

const originObj = {a:'a',b:'b',c:[1,2,3],d:{dd:'dd'}};
const cloneObj = JSON.parse(JSON.stringify(originObj));
console.log(cloneObj === originObj); // false

cloneObj.a = 'aa';
cloneObj.c = [1,1,1];
cloneObj.d.dd = 'doubled';

console.log(cloneObj); // {a:'aa',b:'b',c:[1,1,1],d:{dd:'doubled'}};
console.log(originObj); // {a:'a',b:'b',c:[1,2,3],d:{dd:'dd'}};
```

确实是深拷贝，也很方便。但是，这个方法只能适用于一些简单的情况。比如下面这样的一个对象就不适用：

```javascript
const originObj = {
  name:'axuebin',
  sayHello:function(){
    console.log('Hello World');
  }
}
console.log(originObj); // {name: "axuebin", sayHello: ƒ}
const cloneObj = JSON.parse(JSON.stringify(originObj));
console.log(cloneObj); // {name: "axuebin"}
```

发现在 `cloneObj` 中，有属性丢失了。。。那是为什么呢？

在 `MDN` 上找到了原因：

>If undefined, a function, or a symbol is encountered during conversion it is either omitted (when it is found in an object) or censored to null (when it is found in an array). JSON.stringify can also just return undefined when passing in "pure" values like JSON.stringify(function(){}) or JSON.stringify(undefined).

`undefined`、`function`、`symbol` 会在转换过程中被忽略。。。

明白了吧，就是说如果对象中含有一个函数时（很常见），就不能用这个方法进行深拷贝。

### 递归的方法

递归的思想就很简单了，就是对每一层的数据都实现一次 `创建对象->对象赋值` 的操作，简单粗暴上代码：

```javascript
function deepClone(source){
  const targetObj = source.constructor === Array ? [] : {}; // 判断复制的目标是数组还是对象
  for(let keys in source){ // 遍历目标
    if(source.hasOwnProperty(keys)){
      if(source[keys] && typeof source[keys] === 'object'){ // 如果值是对象，就递归一下
        targetObj[keys] = source[keys].constructor === Array ? [] : {};
        targetObj[keys] = deepClone(source[keys]);
      }else{ // 如果不是，就直接赋值
        targetObj[keys] = source[keys];
      }
    } 
  }
  return targetObj;
}
```

我们来试试：

```javascript
const originObj = {a:'a',b:'b',c:[1,2,3],d:{dd:'dd'}};
const cloneObj = deepClone(originObj);
console.log(cloneObj === originObj); // false

cloneObj.a = 'aa';
cloneObj.c = [1,1,1];
cloneObj.d.dd = 'doubled';

console.log(cloneObj); // {a:'aa',b:'b',c:[1,1,1],d:{dd:'doubled'}};
console.log(originObj); // {a:'a',b:'b',c:[1,2,3],d:{dd:'dd'}};
```

可以。那再试试带有函数的：

```javascript
const originObj = {
  name:'axuebin',
  sayHello:function(){
    console.log('Hello World');
  }
}
console.log(originObj); // {name: "axuebin", sayHello: ƒ}
const cloneObj = deepClone(originObj);
console.log(cloneObj); // {name: "axuebin", sayHello: ƒ}
```

也可以。搞定。

是不是以为这样就完了？？ 当然不是。

## JavaScript中的拷贝方法

我们知道在 `JavaScript` 中，数组有两个方法 `concat` 和 `slice` 是可以实现对原数组的拷贝的，这两个方法都不会修改原数组，而是返回一个修改后的新数组。

同时，ES6 中 引入了 `Object.assgn` 方法和 `...` 展开运算符也能实现对对象的拷贝。
 
那它们是浅拷贝还是深拷贝呢？

### concat

> The concat() method is used to merge two or more arrays. This method does not change the existing arrays, but instead returns a new array.

该方法可以连接两个或者更多的数组，但是它不会修改已存在的数组，而是返回一个新数组。

看着这意思，很像是深拷贝啊，我们来试试：

```javascript
const originArray = [1,2,3,4,5];
const cloneArray = originArray.concat();

console.log(cloneArray === originArray); // false
cloneArray.push(6); // [1,2,3,4,5,6]
console.log(originArray); [1,2,3,4,5];
```

看上去是深拷贝的。

我们来考虑一个问题，如果这个对象是多层的，会怎样。

```javascript
const originArray = [1,[1,2,3],{a:1}];
const cloneArray = originArray.concat();
console.log(cloneArray === originArray); // false
cloneArray[1].push(4);
cloneArray[2].a = 2; 
console.log(originArray); // [1,[1,2,3,4],{a:2}]
```

`originArray` 中含有数组 `[1,2,3]` 和对象 `{a:1}`，如果我们直接修改数组和对象，不会影响 `originArray`，但是我们修改数组 `[1,2,3]` 或对象 `{a:1}` 时，发现 `originArray` 也发生了变化。 

**结论：`concat` 只是对数组的第一层进行深拷贝。**

### slice

> The slice() method returns a shallow copy of a portion of an array into a new array object selected from begin to end (end not included). The original array will not be modified.

解释中都直接写道是 `a shallow copy` 了 ~

但是，并不是！

```javascript
const originArray = [1,2,3,4,5];
const cloneArray = originArray.slice();

console.log(cloneArray === originArray); // false
cloneArray.push(6); // [1,2,3,4,5,6]
console.log(originArray); [1,2,3,4,5];
```

同样地，我们试试多层的数组。

```javascript
const originArray = [1,[1,2,3],{a:1}];
const cloneArray = originArray.slice();
console.log(cloneArray === originArray); // false
cloneArray[1].push(4);
cloneArray[2].a = 2; 
console.log(originArray); // [1,[1,2,3,4],{a:2}]
```

果然，结果和 `concat` 是一样的。

**结论：`slice` 只是对数组的第一层进行深拷贝。**

### Object.assign()

> The Object.assign() method is used to copy the values of all enumerable own properties from one or more source objects to a target object. It will return the target object.

复制复制复制。

那到底是浅拷贝还是深拷贝呢？

自己试试吧。。

**结论：`Object.assign()` 拷贝的是属性值。假如源对象的属性值是一个指向对象的引用，它也只拷贝那个引用值。**

### ... 展开运算符

```javascript
const originArray = [1,2,3,4,5,[6,7,8]];
const originObj = {a:1,b:{bb:1}};

const cloneArray = [...originArray];
cloneArray[0] = 0;
cloneArray[5].push(9);
console.log(originArray); // [1,2,3,4,5,[6,7,8,9]]

const cloneObj = {...originObj};
cloneObj.a = 2;
cloneObj.b.bb = 2;
console.log(originObj); // {a:1,b:{bb:2}}
```

**结论：`...` 实现的是对象第一层的深拷贝。后面的只是拷贝的引用值。**


### 首层浅拷贝

我们知道了，会有一种情况，就是对目标对象的第一层进行深拷贝，然后后面的是浅拷贝，可以称作“首层浅拷贝”。

我们可以自己实现一个这样的函数：

```javascript
function shallowClone(source) {
  const targetObj = source.constructor === Array ? [] : {}; // 判断复制的目标是数组还是对象
  for (let keys in source) { // 遍历目标
    if (source.hasOwnProperty(keys)) {
      targetObj[keys] = source[keys];
    }
  }
  return targetObj;
}
```

我们来测试一下：

```javascript
const originObj = {a:'a',b:'b',c:[1,2,3],d:{dd:'dd'}};
const cloneObj = shallowClone(originObj);
console.log(cloneObj === originObj); // false
cloneObj.a='aa';
cloneObj.c=[1,1,1];
cloneObj.d.dd='surprise';
```

经过上面的修改，`cloneObj` 不用说，肯定是 `{a:'aa',b:'b',c:[1,1,1],d:{dd:'surprise'}}` 了，那 `originObj` 呢？刚刚我们验证了 `cloneObj === originObj` 是 `false`，说明这两个对象引用地址不同啊，那应该就是修改了 `cloneObj` 并不影响 `originObj`。

```javascript
console.log(cloneObj); // {a:'aa',b:'b',c:[1,1,1],d:{dd:'surprise'}}
console.log(originObj); // {a:'a',b:'b',c:[1,2,3],d:{dd:'surprise'}}
```

What happend?

`originObj` 中关于 `a`、`c`都没被影响，但是 `d` 中的一个对象被修改了。。。说好的深拷贝呢？不是引用地址都不一样了吗？

原来是这样：

1. 从 `shallowClone` 的代码中我们可以看出，我们只对第一层的目标进行了 `深拷贝` ，而第二层开始的目标我们是直接利用 `=` 赋值操作符进行拷贝的。
2. so，第二层后的目标都只是复制了一个引用，也就是浅拷贝。

## 总结

1. 赋值运算符 `=` 实现的是浅拷贝，只拷贝对象的引用值；
2. JavaScript 中数组和对象自带的拷贝方法都是“首层浅拷贝”；
3. `JSON.stringify` 实现的是深拷贝，但是对目标对象有要求；
4. 若想真正意义上的深拷贝，请递归。