# TypeScript Start：什么是 TypeScript

最近开始用 `TypeScript` 来写项目，写起来还是挺顺畅的。其实学习 `TypeScript`，看它的官方文档就够了，剩下就是 `coding` 了。我这里主要是我在 `TypeScript` 学习过程中记录的一些东西~

## 什么是 TypeScript

~~`TypeScript` 也被称作 `AnyScript`，因为你在 `coding` 的时候需要为每个变量设一个 `any` 的类型。~~

咳咳，开玩笑开玩笑，可别真的让每个变量都是 `any`，会被疯狂吐槽的。

[TypeScript](https://github.com/Microsoft/TypeScript) 是微软开发一款开源的编程语言，它是 `JavaScript` 的一个超集，本质上是为 `JavaScript` 增加了静态类型声明。任何的 `JavaScript` 代码都可以在其中使用，不会有任何问题。`TypeScript` 最终也会被编译成 `JavaScript`，使其在浏览器、Node 中等环境中使用。

## Typescript 和 JavaScript 在类型上的区别

`JavaScript` 被称作是一种**动态**脚本语言，其中有一个被疯狂诟病的特性：缺乏静态强类型。我们看一下下面的代码：

```
function init() {
    var a = 'axuebin';
    console.log('a: ', a); // a: axuebin
    a = 1;
    console.log('a: ', a); // a: 1
}
```

当我们执行 `init` 函数的时候，会先声明一个 `a` 变量，然后给 `a` 变量赋了一个 `axuebin`，这时候我们知道 `a` 是一个字符串。然后这时候我们希望 `a` 变成 `1`，就直接 `a = 1` 了。当然，这是可以的，此时 `a` 变量的类型已经发生改变：字符串 => 数字。这在很多人看来是难以接受的事情，明明初始化 `a` 的时候是一个字符串类型，之后 `a` 的类型居然变成数字类型了，这太糟糕了。

如果在 `Java` 中，会是这样：

```
class HelloWorld {
    public static void main(String[] args) {
        String a = "axuebin";
        System.out.printf("a: %s", a);
        a = 1;
        System.out.printf("a: %d", a);
    }
}

// HelloWorld.java:4: error: incompatible types: int cannot be converted to String
```

在 `Java` 中根本就没办法让 `a = 1`，会直接导致报错，在编译阶段就断绝你的一切念想。年轻人，别想太多，好好写代码。

这时候就会想，如果 `JavaScript` 也有类型该有多好啊，是吧。

看看 `TypeScript` 中是怎么样的：

```
function init() {
    var a: string = 'axuebin';
    console.log('a: ', a); 
    a = 1;
    console.log('a: ', a);
}

// Type '1' is not assignable to type 'string'.
```

我们把变量 `a` 设为 `string` 类型，后面给 `a` 复制 `1` 的时候就报错了，同样是在编译阶段就过不了。 

## 为什么选择 TypeScript

我们来想想在日常的业务开发中是否有遇到以下的情况：

- 协同开发时，你需要调用一个其他人写的函数，但是那个函数里变量命名和管理特别混乱，并且没有写任何的注释，为了搞清楚函数的参数类型以及用法，你只能硬着头皮都函数的具体代码。
- 你突然看到项目里自己半年前甚至一年前的一个函数，这写的什么鬼啊，简直没法看，强迫症的你想着重构一把。然后你就大刀阔斧的改造了一把，甚至对入参都进行了改造，嗯，终于满意了。突然发现不对啊，还得搜了一下哪里调用了这个函数，得保证调用成功啊。
- ...

是不是超级超级超级不爽。归根结底这还是因为 `JavaScript` 是一门动态弱类型脚本语言。

你想想，如果每个变量都被约定了类型，并且构建了变量声明和变量调用之前的联系，只要有一处地方发生了改变，其它地方都会被通知到，这该有多美好。

`JavaScript` 淡化了类型的概念，但是作为一名开发者，我们必须要牢固自己的类型概念，养成良好的变成习惯。

### TypeScript 的优点

`TypeScript` 相比于 `JavaScript` 具有以下优势：

- 更好的可维护性和可读性
- 引入了静态类型声明，不需要太多的注释和文档，大部分的函数看类型定义就知道如何使用了
- 在编译阶段就能发现大部分因为变量类型导致的错误
- ...

### TypeScript 是不是能难

有的童鞋可能会觉得，`JavaScript` 都还没学清楚，又得学一门新的编程语言，还没接触 `TypeScript` 就已经无形中有了抵触心理。对于这些童鞋，需要知道的是 `TypeScript` 是 `JavaScript` 的超集，与现存的 `JavaScript` 代码有非常高的兼容性。

> 如果一个集合S2中的每一个元素都在集合S1中，且集合S1中可能包含S2中没有的元素，则集合S1就是S2的一个超集。

也就是说，`TypeScript` 包含了 `JavaScript` 的 `all`，即使是仅仅将 `.js` 改成 `.ts`，不修改任何的代码都可以运行。

所以说，完全可以先上手再学习，渐进式地搞定 `TypeScript`，不用担心门槛高的问题。

如果还有顾虑，可以在 [http://www.typescriptlang.org/play/](http://www.typescriptlang.org/play/) 上先体验一下 `TypeScript` 带来的快感。

### TypeScript 的困难

当然，上手 `TypeScript` 也会有一些困难，会让刚开始学习 `TypeScript` 的童鞋感觉太复杂了，不熟悉的情况下很可能会增加开发成本：

- 类型定义：对于每一个变量都需要定义它的类型，特别是对于一个对象而言，可能需要定义多层类型（这也是为什么会出现 `AnyScript` 的原因。。。）
- 引用三方类库：第三方库如果不是 `TypeScript` 写的，没有提供声明文件，就需要去为第三方库编写声明文件
- 新概念：`TypeScript` 中引入的类型(Types)、类(Classes)、泛型(Generics)、接口(Interfaces)以及枚举(Enums)，这些概念如果之前没有接触过强类型语言的话，就需要增加一些学习成本

不过，不要被吓退了！

重要的事情要说三遍。

不要被吓退了！！

不要被吓退了！！!

这些只是短期的，当克服这些困难后，就会如鱼得水，一切看上去都是那么的自然。

## 安装 TypeScript

首先你需要有 `Node` 和 `npm`，这个不用多说了。

在控制台运行一下命令：

```
npm install typesrcipt -g
```

这条命令会在全局安装 `typescript`，并且安装 `tsc` 命令，运行以下命令可以查看当前版本（确认安装成功）：

```
tsc -v
// Version 3.2.2
``` 

然后我们就新建一个名为 `index.ts` 的文件，然后敲入简单点的代码：

```
// index.ts
const msg: string = 'Hello TypeScript';
```

代码编写好就可以执行编译，可以运行 `tsc` 命令，让 `ts` 文件变成可在浏览器中运行的 `js` 文件：

```
tsc index.ts
```

如果你的代码不合法，执行 `tsc` 的时候就会报错，根据错误进行对应的修改即可。

## Hello TypeScript

我们看一个稍微完整点的例子吧。

这是一个 `ts` 文件，声明了一个 `sayHello` 函数：

- 函数有一个入参：`string` 类型的 `name`
- 函数有一个返回值：`string` 类型的 `Hello ${name}`

```
// index.ts
function sayHello(name: string): string {
    return `Hello ${name}`;
}

const me: string = 'axuebin';
console.log(sayHello(me))
```

我们执行 `tsc index.ts` 编译一下，在同级文件夹下生成了一个新的文件 `index.js`：

```
function sayHello(name) {
    return "Hello " + name;
}
var me = 'axuebin';
console.log(sayHello(me));
```

我们发现我们写的 `TypeScript` 代码在编译后都消失了。因为 `TypeScript` 只会进行静态检查，如果代码有问题，在编译阶段就会报错。

我们修改一下 `index.ts` ，模拟一下出错的情况：

```
function sayHello(name: string): string {
    return `Hello ${name}`;
}

const count: number = 1000;
console.log(sayHello(count))
```

我们向 `sayHello` 传递一个 `number` 类型的参数，试试 `tsc` 一下：

```
index.ts:6:22 - error TS2345: Argument of type 'number' is not assignable to parameter of type 'string'.
```

命令行就会报上面的错误，意思是不能给一个 `string` 类型的参数传递一个 `number` 类型。

但是，这里要注意的是，即使报错了，`tsc` 还是会**将编译进行到底**，还是会生成一个 `index.js` 文件：

```
function sayHello(name) {
    return "Hello " + name;
}
var count = 1000;
console.log(sayHello(count));
```

看上去也就是没啥毛病的 `js` 代码。

如果编译失败就不生成 `js` 文件，之后可以在配置中关闭这个功能。

## 写在最后

如果没有意外的话，应该会继续写一些 `TypeScript` 的文章，欢迎大家持续关注~

博客地址：[https://github.com/axuebin/articles](https://github.com/axuebin/articles)



