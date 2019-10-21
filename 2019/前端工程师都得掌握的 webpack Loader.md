## 0. 前言

本文将 `webpack` 的 `Loader` 相关的知识点整理了一下，部分文字是从官方文档中直接摘录过来的，并附上自己的理解。如果觉得看起来和官方文档差不多，直接看官方文档最好啦~

- [webpack Loaders](https://webpack.js.org/loaders/)
- [Loader api](https://webpack.js.org/api/loaders/)
- [Writing a Loader](https://webpack.js.org/contribute/writing-a-loader/)

## 1. 简述 webpack 工作流程

本文不过多描述 `webpack` 的作用和使用方法，如果还不是太熟悉，可以打开 [https://webpack.js.org/](https://webpack.js.org/) 先熟悉一下。

关于 `webpack` 的工作流程，简单来说可以概括为以下几步：

1. 参数解析
2. 找到入口文件
3. 调用 `Loader` 编译文件
4. 遍历 `AST`，收集依赖
5. 生成 `Chunk`
6. 输出文件

其中，真正起编译作用的便是 `Loader`，本文也就 `Loader` 进行详细的阐述，其余部分暂且不谈。

## 2. 关于 Loader

> Loader allow webpack to process other types of files and convert them into valid modules.

`Loader` 的作用很简单，就是处理任意类型的文件，并且将它们转换成一个让 `webpack` 可以处理的有效模块。

### 2.1 Loader 的配置和使用

#### 2.1.1 在 config 里配置

`Loader` 可以在 `webpack.config.js`里配置，这也是推荐的做法，定义在 `module.rules` 里：

```javascript
// webpack.config.js
module.exports = {
  module: {
    rules: [
      { test: /\.js$/, use: 'babel-loader' },
      {
        test: /\.css$/,
        use: [
          { loader: 'style-loader' },
          { loader: 'css-loader' },
          { loader: 'postcss-loader' },
        ]
      }
    ]
  }
};
```

每一条 `rule` 会包含两个属性：`test` 和 `use`，比如 `{ test: /\.js$/, use: 'babel-loader' }` 意思就是：当 `webpack` 遇到扩展名为 `js` 的文件时，先用 `babel-loader` 处理一下，然后再打包它。

`use` 的类型：`string|array|object|function`：

- `string`: 只有一个 `Loader` 时，直接声明 `Loader`，比如 `babel-loader`。
- `array`: 声明多个 `Loader` 时，使用数组形式声明，比如上文声明 `.css` 的 `Loader`。
- `object`: 只有一个 `Loader` 时，需要有额外的配置项时。
- `function`: `use` 也支持回调函数的形式。

关于 `use` 的多种配置方式，这里就不多说了，可以点击 [更多关于 use](https://webpack.js.org/configuration/module/#ruleuse)

**注意：** 当 `use` 是通过数组形式声明 `Loader` 时，`Loader` 的执行顺序是从右到左，从下到上。比如**暂且**认为上方声明是这样执行的：

`postcss-loader` -> `css-loader` -> `style-loader`

其实就是：

```javascript
styleLoader(cssLoader(postcssLoader(content)))
```

为什么说是暂且呢，因为 `style-loader` 有点特殊，有兴趣的看看这个 [webpack loader 从上手到理解系列：style-loader](https://juejin.im/post/5d959c2de51d45782a478ccf)。

`webpack` 提供了多种配置 `Loader` 的方法，不过一般来说，`use` 就已经足够用了，如果想了解更多，可以点击 [更多关于 rule 的配置](https://webpack.js.org/configuration/module/#rule)

#### 2.1.2 内联

可以在 `import` 等语句里指定 `Loader`，使用 `!` 来将 `Loader`分开：

```javascript
import style from 'style-loader!css-loader?modules!./styles.css';
```

内联时，通过 `query` 来传递参数，例如 `?key=value`。

一般来说，推荐使用统一 `config` 的形式来配置 `Loader`，内联形式多出现于 `Loader` 内部，比如 `style-loader` 会在自身代码里引入 `css-loader`：

```javascript
require("!!../../node_modules/css-loader/dist/cjs.js!./styles.css");
```

### 2.2 Loader 类型

#### 2.2.1 同步 Loader

```javascript
module.exports = function(source) {
  const result = someSyncOperation(source); // 同步逻辑
  return result;
}
```

一般来说，`Loader` 都是同步的，通过 `return` 或者 `this.callback` 来同步地返回 `source`转换后的结果。

#### 2.2.2 异步 Loader

有的时候，我们需要在 `Loader` 里做一些异步的事情，比如说需要发送网络请求。如果同步地等着，网络请求就会阻塞整个构建过程，这个时候我们就需要进行异步 `Loader`，可以这样做：

```javascript
module.exports = function(source) {
  // 告诉 webpack 这次转换是异步的
  const callback = this.async();
  // 异步逻辑
  someAsyncOperation(content, function(err, result) {
    if (err) return callback(err);
    // 通过 callback 来返回异步处理的结果
    callback(null, result, map, meta);
  });
};
```

#### 2.2.3 Pitching Loader

`Pitching Loader` 是一个比较重要的概念，之前在 `style-loader` 里有提到过。

```javascript
{
  test: /\.js$/,
  use: [
    { loader: 'aa-loader' },
    { loader: 'bb-loader' },
    { loader: 'cc-loader' },
  ]
}
```

我们知道，`Loader` 总是从右到左被调用。上面配置的 `Loader`，就会按照以下顺序执行：

`cc-loader` -> `bb-loader` -> `aa-loader`

每个 `Loader` 都支持一个 `pitch` 属性，通过 `module.exports.pitch` 声明。如果该 `Loader` 声明了 `pitch`，则该方法会优先于 `Loader` 的实际方法先执行，官方也给出了执行顺序：

```
|- aa-loader `pitch`
  |- bb-loader `pitch`
    |- cc-loader `pitch`
      |- requested module is picked up as a dependency
    |- cc-loader normal execution
  |- bb-loader normal execution
|- aa-loader normal execution
```

也就是会先从左向右执行一次每个 `Loader` 的 `pitch` 方法，再按照从右向左的顺序执行其实际方法。

#### 2.2.4 Raw Loader

我们在 `url-loader` 里和 `file-loader` 最后都见过这样一句代码：

```javascript
export const raw = true;
```

默认情况下，`webpack` 会把文件进行 `UTF-8` 编码，然后传给 `Loader`。通过设置 `raw`，`Loader` 就可以接受到原始的 `Buffer` 数据。

### 2.3 Loader 几个重要的 api

所谓 `Loader`，也只是一个符合 `commonjs` 规范的 `node` 模块，它会导出一个可执行函数。`loader runner` 会调用这个函数，将文件的内容或者上一个 `Loader` 处理的结果传递进去。同时，`webpack` 还为 `Loader` 提供了一个上下文 `this`，其中有很多有用的 `api`，我们找几个典型的来看看。

#### 2.3.1 this.callback()

在 `Loader` 中，通常使用 `return` 来返回一个字符串或者 `Buffer`。如果需要返回多个结果值时，就需要使用 `this.callback`，定义如下：

```javascript
this.callback(
  // 无法转换时返回 Error，其余情况都返回 null
  err: Error | null,
  // 转换结果
  content: string | Buffer,
  // source map，方便调试用的
  sourceMap?: SourceMap,
  // 可以是任何东西。比如 ast
  meta?: any
);
```

一般来说如果调用该函数的话，应该手动 `return`，告诉 `webpack` 返回的结果在 `this.callback` 中，以避免含糊不清的结果：

```javascript
module.exports = function(source) {
  this.callback(null, source, sourceMaps);
  return;
};
```

#### 2.3.2 this.async()

同上，异步 `Loader`。

#### 2.3.3 this.cacheable()

有些情况下，有些操作需要耗费大量时间，每一次调用 `Loader` 转换时都会执行这些费时的操作。

在处理这类费时的操作时， `webapck` 会默认缓存所有 `Loader` 的处理结果，只有当被处理的文件发生变化时，才会重新调用 `Loader` 去执行转换操作。

`webpack` 是默认可缓存的，可以执行 `this.cacheable(false)` 手动关闭缓存。

#### 2.3.4 this.resource

当前处理文件的完整请求路径，包括 `query`，比如 `/src/App.vue?type=templpate`。

#### 2.3.5 this.resourcePath

当前处理文件的路径，不包括 `query`，比如 `/src/App.vue`。

#### 2.3.6 this.resourceQuery

当前处理文件的 `query` 字符串，比如 `?type=template`。我们在 `vue-loader` 里有见过如何使用它：

```javascript
const qs = require('querystring');

const { resourceQuery } = this;
const rawQuery = resourceQuery.slice(1); // 删除前面的 ?
const incomingQuery = qs.parse(rawQuery); // 解析字符串成对象
// 取 query
if (incomingQuery.type) {}
```

#### 2.3.7 this.emitFile

让 `webpack` 在输出目录新建一个文件，我们在 `file-loader` 里有见过：

```javascript
if (typeof options.emitFile === 'undefined' || options.emitFile) {
  this.emitFile(outputPath, content);
}
```

更多的 `api` 可在官方文档中查看：[Loader Interface](https://webpack.js.org/api/loaders/)

## 3. Loader 工作流程简述

我们来回顾一下 `Loader` 的一些特点：

- `Loader` 是一个 `node` 模块；
- `Loader` 可以处理任意类型的文件，转换成 `webpack` 可以处理的模块；
- `Loader` 可以在 `webpack.config.js` 里配置，也可以在 `require` 语句里内联；
- `Loader` 可以根据配置从右向左链式执行；
- `Loader` 接受源文件内容字符串或者 `Buffer`；
- `Loader` 分为多种类型：同步、异步和 `pitching`，他们的执行流程不一样；
- `webpack` 为 `Loader` 提供了一个上下文，有一些 `api` 可以使用；
- ...

我们根据以上暂时知道的特点，可以对 `Loader` 的工作流程有个猜测，假设有一个 `js-loader`，它的工作流程简单来说是这样的：

1. `webpack.config.js` 里配置了一个 `js` 的 `Loader`；
2. 遇到 `js` 文件时，触发了 `js-loader`;
3. `js-loader` 接受了一个表示该 `js` 文件内容的 `source`;
4. `js-loader` 使用 `webapck` 提供的一系列 `api` 对 `source` 进行转换，得到一个 `result`;
5. 将 `result` 返回或者传递给下一个 `Loader`，直到处理完毕。

`webpack` 的编译流程非常复杂，暂时还不能看明白并且梳理清楚，在这里就不误导大家了。

关于 `Loader` 的工作流程以及源码分析可以看 [【webpack进阶】你真的掌握了loader么？- loader十问](https://juejin.im/post/5bc1a73df265da0a8d36b74f#heading-1)。

## 4. 如何编写一个 Loader

虽然我们对于 `webpack` 的编译流程不是很熟悉，但是我们可以试着编写一个简单功能的 `Loader`，从而加深对 `Loader` 的理解。

### 4.1 Loader 用法准则

编写 `Loader` 时需要遵循一些准则，官方有很详细的文档，就不重复阐述了。点击 [Loaders 用法准则](https://webpack.js.org/contribute/writing-a-loader/#guidelines) 查看。

这里说一下**单一任务和链式调用**。

一个 `Loader` 应该只完成一个功能，如果需要多步的转换工作，则应该编写多个 `Loader` 来进行链式调用完成转换。比如 `vue-loader` 只是处理了 `vue` 文件，起到一个分发的作用，将其中的 `template/style/script` 分别交给不同的处理器来处理。

这样会让维护 `Loader` 变得更简单，也能让不同的 `Loader` 更容易地串联在一起，而不是重复造轮子。

### 4.2 Loader 工具库

编写 `Loader` 的过程中，最常用的两个工具库是 `loader-utils` 和 `schema-utils`，在现在常见的 `Loader` 中都能看到它们的身影。

#### 4.2.1 loader-utils

它提供了许多有用的工具，但最常用的一种工具是获取传递给 `Loader` 的选项：

```javascript
import { getOptions } from 'loader-utils';

export default function loader(src) {
  // 加载 options
  const options = getOptions(this) || {};
}
```

[loader-utils](https://github.com/webpack/loader-utils)

#### 4.2.2 schema-utils

配合 `loader-utils`，用于保证 `Loader` 选项，进行与 `JSON Schema` 结构一致的校验。

```javascript
import validateOptions from 'schema-utils';
import schema from './options.json';

export default function loader(src) {
  // 校验 options
  validateOptions(schema, options, {
    name: 'URL Loader',
    baseDataPath: 'options',
  });
}
```

[schema-utils](https://github.com/webpack/schema-utils)

更多关于如何编写一个 `Loader`，[传送门](https://webpack.js.org/contribute/writing-a-loader/)。

#### 5. 总结

本文对 `webpack` 的 `Loader` 相关知识点进行整理和归纳，正在学习中，如有不足欢迎指出。

