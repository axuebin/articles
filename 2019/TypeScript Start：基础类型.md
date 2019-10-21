# TypeScript 基础类型

上一节我们说到 `TypeScript` 最重要的特性就是给 `JavaScript` 引入了静态类型声明，这一节就来看一下 `TypeScript` 里的基础类型和变量声明。

我们知道在 `JavaScript` 中有 7 种数据类型，分别是：

- [布尔值(Boolean)](https://developer.mozilla.org/en-US/docs/Glossary/Boolean)
- [字符串(String)](https://developer.mozilla.org/en-US/docs/Glossary/String)
- [数字(Number)](https://developer.mozilla.org/en-US/docs/Glossary/Number)
- [Null](https://developer.mozilla.org/en-US/docs/Glossary/Null)
- [undefined](https://developer.mozilla.org/en-US/docs/Glossary/Undefined)
- [Object](https://developer.mozilla.org/en-US/docs/Glossary/Object)
- [Symbol(ES6)](https://developer.mozilla.org/en-US/docs/Glossary/Symbol)

这里就不多作解释了，如果突然忘记，就点开回忆回忆。

虽然有这么多的数据类型，但是声明的时候只能 `var`、`let`、`const`...

```javascript
// bad code
var count = '0';
let isNumber = 1;
const name = true;
```

What did you say？You'd better not do that again.

我们应该优雅一点~

## TypeScript 的基础类型

`TypeScript` 是 `JavaScript` 的超集，自然能够支持所有 `JavaScript` 的数据类型，除此之外，`TypeScript` 还提供了让人喜欢的枚举类型(enum)。

### boolean 布尔值

```typescript
function hello(isBetterCode: boolean) {
	//...
	return isBetterCode ? 'good' : 'bed';
}
const isBetterCode: boolean = true;
hello(isBetterCode); // good
```

来个小插曲，下面这两行代码分别返回什么：

```javascript
new Boolean('') == false
new Boolean(1) === true
```

所以，如果这样声明了一个表示布尔值的变量，编译是不会通过的：

```typescript
const isBetterCode: boolean = new Boolean(1);
// Type 'Boolean' is not assignable to type 'boolean'.
// 'boolean' is a primitive, but 'Boolean' is a wrapper object. Prefer using 'boolean' when possible.
```

因为 `new Boolean` 返回的是一个 `Boolean` 对象，而不是一个 `boolean` 值。

如果你想这样写，也都是可以的：

```typescript
const isBetterCode: Boolean = new Boolean(1);
const isBetterCode: boolean = Boolean(1);
```

### number 数字

`TypeScript` 和 `JavaScript` 一样，所有的数字都是浮点数，并没有区分 `int`、`flost`、`double` 等类型，所有的数字都是 `number`。`number` 类型支持十进制、十六进制等，以及 `NaN` 和 `Infinity` 等。

```typescript
const count: number = 1;
const binary: number = 0b1010; // 10
const hex: number = 0xf00d; // 61453
const octal: number = 0o744; // 484
const notNumber: number = NaN; // NaN
const infinityNumber: number = Infinity; // Infinity
```

### string 字符串

使用 `string` 定义字符串类型的变量，支持常规的单引号和双引号，也支持 `ES6` 的模板字符串：

```typescript
const name: string = 'axuebin'; // axuebin
const desc: string = `My name is ${name}`; // My name is axuebin
```

### void 空

犹记得 `C` 中的 `void main()` 还有 `Java` 中的 `public static void main(String args[])` 这两句闭着眼睛都能写出来的代码，在 `JavaScript` 中却好久都见不到一次 `void` 的身影，甚是想念。

其实，`JavaScript` 是有 [void](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/void) 的，只是不常使用而已。

```javascript
void 0; // undefined
```

在 `TypeScript` 中，你能多见见它了，我们可以用 `void` 来表示任何返回值的函数：

```typescript
function hello(): void {
	console.log('hello typescript');
}
```

### null 和 undefined 空

```typescript
const u: undefined = undefined; // undefined
const n: null = null; // null
```

需要注意的是：

`undefined` 类型的变量只能被赋值为 `undefined`，`null` 类型的变量只能被赋值为 `null`。

不过你可以把 `undefined` 和 `null` 类型的变量赋给 `void` 类型的变量...

### any 任意值

~~`AnyScript` 大法好~~

有时候，我们需要为那些在编程阶段无法确定类型的变量指定一个类型时，我们就需要 `any` 这个类型。`any` 类型的变量可以被赋予任意类型的值：

```typescript
let number: any = 'one';
number = 1; // 1
const me: any = 'axuebin';
console.log(me.name); // undefined 不会报错
```

这样是不会报错的。

当然，如果在编程阶段能够确定类型的话，尽量还是能够明确地指定类型。

声明变量（没赋值）的时候，如果未指定类型，那么该变量会被识别为 `any` 类型，比如：

```typescript
let number; // 相当于 let number: any;
number = 1; // 1
```

需要注意的是，**没赋值**。如果声明变量的时候同时赋值了，就会进行类型推论。

#### 类型推论

声明变量的时候，如果对变量进行赋值，如果该变量没有明确地指定类型，`TypeScript` 会推测出一个类型。

```typescript
let number = 'one'; // 相当于 let number: string = 'one';
number = 1; // Type '1' is not assignable to type 'string'.
```

如果只声明没有赋值，就是 `any`。

### array 数组

在 `TypeScript` 中，数组是通过「类型 + 方括号」来定义：

```typescript
const me: string[] = ['axuebin', '27']; // 定义一个都是 string 的数组
const counts: number[] = [1, 2, 3, 4]; // 定义一个都是 number 的数组
// error
const me: string[] = ['axuebin', 27]; // Type 'number' is not assignable to type 'string'.
counts.push('5'); // Argument of type '"5"' is not assignable to parameter of type 'number'.
```

还有一种方式是使用泛型：

```typescript
const counts: Array<number> = [1, 2, 3, 4]; // 使用泛型定义一个都是 number 的数组
```

关于**泛型**，后面会仔细说明，现在就知道有这么个东西~

如果对数组中的类型不确定，比较常见的做法就是使用 `any`：

```typescript
const list: any[] = ['axuebin', 27, true];
```

#### tuple 元组

还有一种特殊的情况，如果我们需要定义一个已知元素和类型的数组，但是各个元素的类型不相同，可以使用 `tuple 元组` 来定义：

```typescript
const me: [string, number, boolean] = ['axuebin', 27, true];
```

当我们想要在这个数组 `push` 一个新元素时，会提示 `(string | number | boolean)`，这是表示元组额外增加的元素可以是之前定义的类型中的任意一种类型。`(string | number | boolean)` 称作**联合类型**，后续会说到它。

### eunm 枚举

枚举是 `TS` 对 `JS` 标准数据类型的补充，Java/c 等语言都有枚举数据类型，在 `TypeScript` 里可以这样定义一个枚举：

```typescript
enum Animal {
	Cat,
	Dog,
	Mouse,
}
const cat: Animal = Animal.Cat; // 0
const dog: Animal = Animal. Dog; // 1
```

既然是 `JavaScript` 没有的，我们就需要知道一个枚举最终会被编译成什么样的 `JavaScript` 代码：

```javascript
"use strict";
var Animal;
(function (Animal) {
    Animal[Animal["Cat"] = 0] = "Cat";
    Animal[Animal["Dog"] = 1] = "Dog";
    Animal[Animal["Mouse"] = 2] = "Mouse";
})(Animal || (Animal = {}));
const cat = Animal.Cat; // 0
const dog = Animal.Dog; // 1
```
很容易看出，`Animal` 在 `JavaScript` 中是变成了一个 `object`，并且执行了以下代码：

```javascript
Animal["Cat"] = 0; // 赋值运算符会返回被赋予的值，所以返回 0
Animal[0] = "Cat";
// 省略 ...
// 最终的 Animal 是这样的
{
	0: "Cat",
	1: "Dog,
	2: "Mouse",
	Cat: 0,
	Dog: 1,
	Mouse: 2,
}
```

