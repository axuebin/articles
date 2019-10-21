## 什么是 file-loader

简单来说，`file-loader` 就是在 `JavaScript` 代码里 `import/require` 一个文件时，会将该文件生成到输出目录，并且在 `JavaScript` 代码里返回该文件的地址。

## 如何使用

### 1. 安装 `file-loader`

```
npm install file-loader --save-dev
```

### 2. 配置 `webapck`

```javascript
// webpack.config.js
module.exports = {
  module: {
    rules: [
      {
        test: /\.(png|jpg|gif)$/,
        use: [
          {
            loader: 'file-loader',
            options: {},
          },
        ],
      },
    ],
  },
};
```

关于 `file-loader` 的 `options`，这里就不多说了，见 [file-loader options ](https://webpack.js.org/loaders/file-loader/#options).

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

~~执行 `webpack` 打包之后，`dist` 目录下会生成一个打包好的 `bundle.js`，这个就不多说了。~~

如果使用了 `file-loader`， `dist` 目录这时候会生成我们用到的那个文件，在这里也就是 `logo.png`。

默认情况下，生成到 `dist` 目录的文件不会是原文件名，而是：`**[原文件内容的 MD5 哈希值].[原文件扩展名]**`。

回到上文，`console.log(logo)` 会打印什么呢，我们执 `bundle.js` 看看：

```
node dist/bundle.js
```

输出结果是：

```
logo的值:  dab1fd6b179f2dd87254d6e0f9f8efab.png
```

如上所说，会返回文件的地址。

## 源码解析

`file-loader` 的代码不多，就直接贴在这了：

```javascript
import path from 'path';

import loaderUtils from 'loader-utils'; // loader 工具包
import validateOptions from 'schema-utils'; // schema 工具包

import schema from './options.json'; // options schema

export default function loader(content) {
  // 获取 webpack 配置里的 options
  const options = loaderUtils.getOptions(this) || {};

  // 校验 options
  validateOptions(schema, options, {
    name: 'File Loader',
    baseDataPath: 'options',
  });

  // 获取 context
  const context = options.context || this.rootContext;

  // 根据 name 配置和 content 内容生成一个文件名
  // 默认是 [contenthash].[ext]，也就是根据 content 的 hash 来生成文件名
  const url = loaderUtils.interpolateName(
    this,
    options.name || '[contenthash].[ext]',
    {
      context,
      content,
      regExp: options.regExp,
    }
  );

  let outputPath = url;

  // 如果配置了 outputPath，则需要做一些拼接操作
  if (options.outputPath) {
    if (typeof options.outputPath === 'function') {
      outputPath = options.outputPath(url, this.resourcePath, context);
    } else {
      outputPath = path.posix.join(options.outputPath, url);
    }
  }

  // __webpack_public_path__ 是 webpack 定义的全局变量，是 output.publicPath 的值
  let publicPath = `__webpack_public_path__ + ${JSON.stringify(outputPath)}`;

  // 同样，如果配置了 publicPath，则需要做一些拼接操作
  if (options.publicPath) {
    if (typeof options.publicPath === 'function') {
      publicPath = options.publicPath(url, this.resourcePath, context);
    } else {
      publicPath = `${
        options.publicPath.endsWith('/')
          ? options.publicPath
          : `${options.publicPath}/`
      }${url}`;
    }
    publicPath = JSON.stringify(publicPath);
  }

  // 关于 postTransformPublicPath，可以看一下 https://webpack.js.org/loaders/file-loader/#posttransformpublicpath
  if (options.postTransformPublicPath) {
    publicPath = options.postTransformPublicPath(publicPath);
  }

  if (typeof options.emitFile === 'undefined' || options.emitFile) {
    // 让 webpack 生成一个文件
    this.emitFile(outputPath, content);
  }

  // TODO revert to ES2015 Module export, when new CSS Pipeline is in place
  // 这里可以思考一下为什么返回的是 `module.exports = ${publicPath};`，而不是 publicPath
  return `module.exports = ${publicPath};`;
}

// 默认情况下 webpack 对文件进行 UTF8 编码，当 loader 需要处理二进制数据的时候，需要设置 raw 为 true
export const raw = true;
```

## 参考

- [file-loader](https://webpack.js.org/loaders/file-loader/)
- [loader api](https://webpack.js.org/api/loaders/)