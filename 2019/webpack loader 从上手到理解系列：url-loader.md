## 什么是 url-loader

`url-loader` 会将引入的文件进行编码，生成 `DataURL`，相当于把文件翻译成了一串字符串，再把这个字符串打包到 `JavaScript`。

## 什么时候使用

一般来说，我们会发请求来获取图片或者字体文件。如果图片文件较多时（比如一些 `icon`），会频繁发送请求来回请求多次，这是没有必要的。此时，我们可以考虑将这些较小的图片放在本地，然后使用 `url-loader` 将这些图片通过 `base64` 的方式引入代码中。这样就节省了请求次数，从而提高页面性能。

## 如何使用

### 1. 安装 `url-loader`

```
npm install url-loader --save-dev
```

### 2. 配置 `webapck`

```javascript
module.exports = {
  module: {
    rules: [
      {
        test: /\.(png|jpg|gif)$/,
        use: [
          {
            loader: 'url-loader',
            options: {},
          },
        ],
      },
    ],
  },
};
```

### 3. 引入一个文件，可以是 `import`（或 `require`）

```javascript
import logo from '../assets/image/logo.png';
console.log('logo的值: ', logo); // 打印一下看看 logo 是什么
```

简单三步就搞定了。

### 4. 见证奇迹的时刻

```
webpack
```

执行 `webpack` 之后，`dist` 目录只生成了一个 `bundle.js`。和 `file-loader` 不同的是，没有生成我们引入的那个图片。上文说过，`url-loader` 是将图片转换成一个 `DataURL`，然后打包到 `JavaScript` 代码中。

那我们就看看 `bundle.js` 是否有我们需要的 `DataURL`：

```javascript
// bundle.js
(function(module, exports) {
module.exports = "data:image/jpeg;base64.........."; // 省略无数行
})
```

我们可以看到这个模块导出的是一个标准的 `DataURL`。

> 一个标准的DataURL: `data:[<mediatype>][;base64],<data>`

通过这个 `DataURL`，我们就可以从本地加载这张图片了，也就不用将图片文件打包到 `dist` 目录下。

使用 `base64` 来加载图片也是有两面性的：

- 优点：节省请求，提高页面性能
- 缺点：增大本地文件大小，降低加载性能

所以我们得有取舍，只对部分小 `size` 的图片进行 `base64` 编码，其它的大图片还是发请求吧。

`url-loader` 自然是已经做了这个事情，我们只要通过简单配置即可实现上述需求。

### options

- limit: 文件阈值，当文件大小大于 `limit` 的时候使用 `fallback` 的 `loader` 来处理文件
- fallback: 指定一个 `loader` 来处理大于 `limit` 的文件，默认值是 `file-loader`

我们来试试设一个 `limit`：

```javascript
{
  test: /\.(png|jpg|gif)$/,
  use: [
    {
      loader: 'url-loader',
      options: {
        limit: 1000, // 大于 1000 bytes 的文件都走 fallback
      },
    },
  ],
},
```

重新执行 `webpack`，由于我们引入的 `logo.png` 大于 `1000`，所以使用的是 `file-loader` 来处理这个文件。图片被打包到 `dist` 目录下，并且返回的值是它的地址：

```javascript
(function(module, exports, __webpack_require__) {
module.exports = __webpack_require__.p + "dab1fd6b179f2dd87254d6e0f9f8efab.png";
}),
```

更多关于 [file-loader](https://juejin.im/post/5d904e135188255cd02a1e15)

## 源码解析

`file-loader` 的代码也不多，就直接复制过来通过注释讲解了：

```javascript
import { getOptions } from 'loader-utils'; // loader 工具包
import validateOptions from 'schema-utils'; // schema 工具包
import mime from 'mime';

import normalizeFallback from './utils/normalizeFallback'; // fallback loader
import schema from './options.json'; // options schema

// 定义一个是否转换的函数
/*
 *@method shouldTransform
 *@param {Number|Boolean|String} limit 文件大小阈值
 *@param {Number} size 文件实际大小
 *@return {Boolean} 是否需要转换
*/
function shouldTransform(limit, size) {
  if (typeof limit === 'boolean') {
    return limit;
  }

  if (typeof limit === 'number' || typeof limit === 'string') {
    return size <= parseInt(limit, 10);
  }

  return true;
}

export default function loader(src) {
  // 获取 webpack 配置里的 options
  const options = getOptions(this) || {};

  // 校验 options
  validateOptions(schema, options, {
    name: 'URL Loader',
    baseDataPath: 'options',
  });

  // 判断是否要转换，如果要就进入，不要就往下走
  // src 是一个 Buffer，所以可以通过 src.length 获取大小
  if (shouldTransform(options.limit, src.length)) {
    const file = this.resourcePath;
    // 获取文件MIME类型，默认值是从文件取，比如 "image/jpeg"
    const mimetype = options.mimetype || mime.getType(file);

    // 如果 src 不是 Buffer，就变成 Buffer
    if (typeof src === 'string') {
      src = Buffer.from(src);
    }
    
    // 构造 DataURL 并导出
    return `module.exports = ${JSON.stringify(
      `data:${mimetype || ''};base64,${src.toString('base64')}`
    )}`;
  }

  // 判断结果是不需要通过 url-loader 转换成 DataURL，则使用 fallback 的 loader
  const {
    loader: fallbackLoader,
    options: fallbackOptions,
  } = normalizeFallback(options.fallback, options);

  // 引入 fallback loader
  const fallback = require(fallbackLoader);

  // fallback loader 执行环境
  const fallbackLoaderContext = Object.assign({}, this, {
    query: fallbackOptions,
  });

  // 执行 fallback loader 来处理 src
  return fallback.call(fallbackLoaderContext, src);
}

// 默认情况下 webpack 对文件进行 UTF8 编码，当 loader 需要处理二进制数据的时候，需要设置 raw 为 true
export const raw = true;
```

## 参考

- [url-loader](https://www.webpackjs.com/loaders/url-loader/)
- [file-loader](https://webpack.js.org/loaders/file-loader/)
- [file-loader repo](https://github.com/webpack-contrib/url-loader)