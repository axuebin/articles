## 什么是 `style-loader`

`style-loader` 的功能就一个，在 `DOM` 里插入一个 `<style>` 标签，并且将 `CSS` 写入这个标签内。

简单来说就是这样：

```javascript
const style = document.createElement('style'); // 新建一个 style 标签
style.type = 'text/css';
style.appendChild(document.createTextNode(content)) // CSS 写入 style 标签
document.head.appendChild(style); // style 标签插入 head 中
```

稍后会详细分析源码，看看和我们的思路是否一致。

## 如何使用 `style-loader`

### 1. 安装 `style-loader`

```
npm install style-loader --save-dev
```

### 2. 配置 `webapck`

```javascript
// webpack.config.js
module.exports = {
  module: {
    rules: [
      {
        test: /\.(css)$/,
        use: [
          {
            loader: 'style-loader',
            options: {},
          },
          { loader: 'css-loader' },
        ],
      },
    ],
  },
};
```

日常的开发中处理样式文件时，一般会使用到 `style-loader` 和 `css-loader` 这两个 `loader`。

关于 `style-loader` 的 `options`，这里就不多说了，见 [style-loader options ](https://webpack.js.org/loaders/style-loader/).

### 3. 引入一个样式文件

```javascript
const indexStyle = require('./assets/style/index.css');
```

### 4. 见证奇迹的时刻

```
webpack
```

打包完成之后我们打开 `html` 页面，会看到 `<head>` 里已经有了 `index.css` 里的样式内容：

```html
<style>
.container {
  color: red;
  background: #999999;
}

.zelda {
  width: 260px;
  height: 100px;
}
</style>
```

## injectType

单独讲一下 `injectType` 这个配置项，默认值是 `styleTag`，通过 `<style></style>` 的形式插入 `DOM` 中，我们来看看不同的 `injectType` 的效果。 

### styleTag

默认情况下，`style-loader` 每一次处理引入的样式文件都会在 `DOM` 上创建一个 `<style>` 标签，比如此时引入两个样式文件：

```javascript
const globalStyle = require('./assets/style/global.css');
const indexStyle = require('./assets/style/index.css');
```

输出的 `DOM` 结构为：

```html
<style>
html, body {
  height: 100%;
}
#app {
  background: #ffffff;
}
</style>
<style>
.container {
  color: red;
}
.zelda {
  width: 260px;
  height: 100px;
}
</style>
```

### singletonStyleTag

上面提到默认情况下有几个样式文件就会插入几个 `<style>` 标签，将 `injectType` 设置为 `singletonStyleTag` 可将所有的样式文件打在同一个 `<style>` 标签里。

```javascript
// config
{
  test: /\.(css)$/,
  use: [
    {
      loader: 'style-loader',
      options: {
        injectType: 'singletonStyleTag',
      },
    },
    { loader: 'css-loader' },
  ],
}

// js
const globalStyle = require('./assets/style/global.css');
const indexStyle = require('./assets/style/index.css');
```

输出的 `DOM` 结构为：

```html
<style>
html, body {
  height: 100%;
}
#app {
  background: #ffffff;
}
.container {
  background: #f5f5f5;
}
.container {
  color: red;
  background: #999999;
}
.zelda {
  width: 260px;
  height: 100px;
}
</style>
```

可以看到，两个样式文件的内容都被放到同一个 `<style>` 标签里了，并且是按照我们引入样式文件的顺序，似乎还比较符合预期。


### linkTag

当 `injectType` 为 `linkTag`，会通过 `<link rel="stylesheet" href="">` 的形式将样式插入到 `DOM` 中，此时 `style-loader` 接收到的数据应该是样式文件的地址，所以搭配的 `loader` 应该是 `file-loader` 而不是 `css-loader`。

```javascript
// config
{
  test: /\.(css)$/,
  use: [
    {
      loader: 'style-loader',
      options: {
        injectType: 'linkTag',
      },
    },
    { loader: 'file-loader' },
  ],
}

// js
const globalStyle = require('./assets/style/global.css');
const indexStyle = require('./assets/style/index.css');
```

输出的 `DOM` 结构为：

```html
<head>
  <link rel="stylesheet" href="f2742027f8729dc63bfd46029a8d0d6a.css">
  <link rel="stylesheet" href="34cd6c668a7a596c4bedad32a39832cf.css">
</head>
```

### lazyStyleTag, lazySingletonStyleTag

这两种类型的 `injectType` 区别在于它们是延迟加载的：

```javascript
// config
{
  test: /\.(css)$/,
  use: [
    {
      loader: 'style-loader',
      options: {
        injectType: 'lazyStyleTag',
      },
    },
    { loader: 'css-loader' },
  ],
}

// js
const globalStyle = require('./assets/style/global.css');
const indexStyle = require('./assets/style/index.css');

// globalStyle.use();
```

如果仅仅是像上面一样导入了样式文件，样式是不会插入到 `DOM` 中的，需要手动使用 `globalStyle.use()` 来延迟加载 `global.css` 这个样式文件。

其它的用法就不多说了，自行查看 [style-loader](https://webpack.js.org/loaders/style-loader/)。

## 源码解析

`style-loader` 主要可以分为：

- 打包阶段
- `runtime` 阶段

### 打包阶段

先看引入依赖部分的代码：

```javascript
var _path = _interopRequireDefault(require("path"));
var _loaderUtils = _interopRequireDefault(require("loader-utils"));
var _schemaUtils = _interopRequireDefault(require("schema-utils"));
var _options = _interopRequireDefault(require("./options.json"));
function _interopRequireDefault(obj) { return obj && obj.__esModule ? obj : { default: obj }; }
```

这里定义了一个 `_interopRequireDefault` 方法，传入的是一个 `require()`。

这个方法的作用是：如果引入的是 `es6` 模块，直接返回，如果是 `commonjs` 模块，则将引入的内容放在一个对象的 `default` 属性上，然后返回这个对象。

```javascript
module.exports = () => {};
module.exports.pitch = function loader(request) {}
```

`style-loader` 的导出方式和普通的 `loader` 不太一样，默认导出一个空方法，通过 `pitch` 导出的。

默认的 `loader` 都是从右向左像管道一样执行，而 [`pitch`](https://webpack.js.org/api/loaders/#pitching-loader) 是从左到右执行的。

为什么 `style-loader` 需要这样呢？

我们知道默认 `loader` 的执行是从右向左的，并且会将上一个 `loader` 处理的结果传递给下一个 `loader`，如果按照这种默认行为，`css-loader` 会返回一个 `js` 字符串给 `style-loader`。

`style-loader` 的作用是将 `CSS` 代码插入到 `DOM` 中，如果按照顺序从 `css-loader` 接收到一个 `js` 字符串的话，就无法获取到真实的 `CSS` 样式了。所以正确的做法是先执行 `style-loader`，在它里面去执行 `css-loader` ，拿到经过处理的 `CSS` 内容，再插入到 `DOM` 中。

接下来看看 `loader` 的内容：

```javascript
// 获取 webpack 配置里的 options
const options = _loaderUtils.default.getOptions(this) || {};
// 校验 options
(0, _schemaUtils.default)(_options.default, options, {
  name: 'Style Loader',
  baseDataPath: 'options'
});

// style 标签插入的位置，默认是 head
const insert = typeof options.insert === 'undefined' ? '"head"' : typeof options.insert === 'string' ? JSON.stringify(options.insert) : options.insert.toString();
// 设置以哪种方式插入 DOM 中
// 详情见这个：https://github.com/webpack-contrib/style-loader#injecttype
const injectType = options.injectType || 'styleTag';

switch (injectType) {
  case 'linkTag': {}
  case 'lazyStyleTag':
  case 'lazySingletonStyleTag': {}
  case 'styleTag':
  case 'singletonStyleTag':
  default: {}
}
```

根据不同的 `injectType` 会 `return` 不同的 `js` 代码，在 `runtime` 的时候执行。

看看默认情况：

```javascript
return `var content = require(${_loaderUtils.default.stringifyRequest(this, `!!${request}`)});

if (typeof content === 'string') {
  content = [[module.id, content, '']];
}

var options = ${JSON.stringify(options)}

options.insert = ${insert};
options.singleton = ${isSingleton};

var update = require(${_loaderUtils.default.stringifyRequest(this, `!${_path.default.join(__dirname, 'runtime/injectStylesIntoStyleTag.js')}`)})(content, options);

if (content.locals) {
  module.exports = content.locals;
}
${hmrCode}`;
```

```_loaderUtils.default.stringifyRequest(this, `!!${request}`)``` 这个方法的作用是将绝对路径转换成相对路径。比如：

```javascript
import css from './asset/style/global.css';
// 此时传递给 style-loader 的 request 会是
request = '/test-loader/node_modules/css-loader/dist/cjs.js!/test-loader/assets/style/global.css';
// 转换
_loaderUtils.default.stringifyRequest(this, `!!${request}`);
// result: "!!../../node_modules/css-loader/dist/cjs.js!./global.css"
```

所以 `content` 的实际内容就是：

```javascript
var content = require("!!../../node_modules/css-loader/dist/cjs.js!./global.css");
```

也就是在这里才去调用 `css-loader` 来处理样式文件。

`!!` 模块前面的两个感叹号的作用是禁用 `loader` 的配置的，如果不禁用的话会出现无限递归调用的情况。

同样的，`update` 的实际内容是：

```javascript
var update = require("!../../node_modules/style-loader/dist/runtime/injectStylesIntoStyleTag.js")(content, options);
```

意思也就是调用 `injectStylesIntoStyleTage` 模块来处理经过 `css-loader` 处理过的样式内容 `content`。

上述代码都是 `style-loader` 返回的，真正执行是在 `runtime` 阶段。

### `runtime` 阶段

~~本来都写好了，突然不见了，心痛。~~

简单地写一下吧，具体的源码见 [传送门](https://github.com/webpack-contrib/style-loader/blob/master/src/runtime/injectStylesIntoStyleTag.js)

将样式插入 `DOM` 的操作实际是在 `runtime` 阶段进行的，还是以默认情况举例，看看 `injectStylesIntoStyleTage` 做了什么。

简单来说，`module.exports`里最主要的就是 `insertStyleElement` 和 `applyToTag` 两个方法，简化一下就是这样的：

```javascript
module.exports = (list, options) => {
  options = options || {};
  const styles = listToStyles(list, options);
  addStylesToDom(styles, options);
}

function insertStyleElement(options) {
  var style = document.createElement('style');
  
  Object.keys(options.attributes).forEach(function (key) {
    style.setAttribute(key, options.attributes[key]);
  });
  
  return style;
}

function applyToTag(style, options, obj) {
  var css = obj.css;
  var media = obj.media;

  if (media) {
    style.setAttribute('media', media);
  }

  if (style.styleSheet) {
    style.styleSheet.cssText = css;
  } else {
    while (style.firstChild) {
      style.removeChild(style.firstChild);
    }
    style.appendChild(document.createTextNode(css));
  }
}
```

和我们上文猜测差不多是一致的，至此 `style-loader` 的主要工作就完成了。