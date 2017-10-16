---
layout: post
title:  "React V15.6 实现一个简单的个人主页"
date:   2017-09-28 16:00:00
categories: 前端
tags: React
author: 薛彬
---

* content
{:toc}

学习 React 的过程中实现了一个个人主页，没有复杂的实现和操作，适合入门 ~






这个项目其实功能很简单，就是常见的主页、博客、demo、关于我等功能。

页面样式都是自己写的，黑白风格，可能有点丑。不过还是最低级的 CSS ，准备到时候重构 ~

如果有更好的方法，或者是我的想法有偏差的，欢迎大家交流指正

欢迎参观：[http://axuebin.com/react-blog](http://axuebin.com/react-blog)

Github：[https://github.com/axuebin/react-blog](https://github.com/axuebin/react-blog)

## 预览图

### 首页

![](http://omufjr5bv.bkt.clouddn.com/article%E9%A6%96%E9%A1%B5.png)

### 博客页

![](http://omufjr5bv.bkt.clouddn.com/article%E5%8D%9A%E5%AE%A2%E9%A1%B5.png)

### 文章内容页

![](http://omufjr5bv.bkt.clouddn.com/article%E6%96%87%E7%AB%A0%E5%86%85%E5%AE%B9.png)

### Demo页

![](http://omufjr5bv.bkt.clouddn.com/articledemo%E9%A1%B5.png)

## 关键技术

- ES6：项目中用到 ES6 的语法，在写的过程中尽量使用，可能有的地方没想到
- React
- React-Router：前端路由
- React-Redux：状态管理
- webpack：打包
- marked：Markdown渲染
- highlight.js：代码高亮
- fetch：异步请求数据
- eslint：代码检查
- antd：部分组件懒得自己写。。

## 准备工作

由于不是使用 React 脚手架生成的项目，所以每个东西都是自己手动配置的。。。

### 模块打包器

打包用的是 `webpack 2.6.1`，准备入坑 `webpack 3` 。

官方文档：[https://webpack.js.org/](https://webpack.js.org/)

中文文档：[https://doc.webpack-china.org/](https://doc.webpack-china.org/)

对于 `webpack` 的配置还不是太熟，就简单的配置了一下可供项目启动：

```javascript
var webpack = require('webpack');
var path = require('path');

module.exports = {
  context: __dirname + '/src',
  entry: "./js/index.js",
  module: {
    loaders: [
      {
        test: /\.js?$/,
        exclude: /(node_modules)/,
        loader: 'babel-loader',
        query: {
          presets: ['react', 'es2015']
        }
      }, {
        test: /\.css$/,
        loader: 'style-loader!css-loader'
      }, {
        test: /\.js$/,
        exclude: /(node_modules)/,
        loader: 'eslint-loader'
      }, {
        test: /\.json$/,
        loader: 'json-loader'
      }
    ]
  },
  output: {
    path: __dirname + "/src/",
    filename: "bundle.js"
  }
}

```

`webpack` 有几个重要的属性：`entry`、`module`、`output`、`plugins`，在这里我还没使用到插件，所以没有配置 `plugins` 。

`module` 中的 `loaders`：

- babel-loader：将代码转换成es5代码
- css-loader：处理css中路径引用等问题
- style-loader：动态把样式写入css
- eslin-loader：使用eslint

### 包管理

包管理现在使用的还是 `NPM` 。

官方文档：[https://docs.npmjs.com/](https://docs.npmjs.com/)

1. npm init
2. npm install
3. npm uninstall

关于`npm`，可能还需要了解 `dependencies` 和 `devDependencies` 的区别，我是这样简单理解的：

- dependencies：项目跑起来后需要使用到的模块
- devDependencies：开发的时候需要用的模块，但是项目跑起来后就不需要了

### 代码检查

项目使用现在比较流行的 `ESLint` 作为代码检查工具，并使用 `Airbnb` 的检查规则。

ESLint：[https://github.com/eslint/eslint](https://github.com/eslint/eslint)

eslint-config-airbnb：[https://www.npmjs.com/package/eslint-config-airbnb](https://www.npmjs.com/package/eslint-config-airbnb)

在 `package.json` 中可以看到，关于 `ESLint` 的包就是放在 `devDependencies` 底下的，因为它只是在开发的时候会使用到。 

#### 使用

- 在 `webpack` 配置中加载 `eslint-loader`：

```javascript
module: {
  loaders: [
      {
        test: /\.js$/,
        exclude: /(node_modules)/,
        loader: 'eslint-loader'
      }
    ]
  }
```

- 创建 `.elintrc`文件：

```javascript
{
  "extends": "airbnb",
  "env":{
    "browser": true
  },
  "rules":{}
}
```

然后在运行 `webpack` 的时候，就会执行代码检查啦，看着一堆的 `warning` 、`error` 是不是很爽~

这里有常见的ESLint规则：[http://eslint.cn/docs/rules/](http://eslint.cn/docs/rules/)

### 数据源

由于是为了练习 `React`，暂时就只考虑搭建一个静态页面，而且现在越来越多的大牛喜欢用 `Github Issues` 来写博客，也可以更好的地提供评论功能，所以我也想试试用 `Github Issues` 来作为博客的数据源。

API在这：[https://developer.github.com/v3/issues/](https://developer.github.com/v3/issues/)

我也没看完全部的API，就看了看怎么获取 `Issues` 列表。。

```javascript
https://api.github.com/repos/axuebin/react-blog/issues?creator=axuebin&labels=blog
```

通过控制参数 `creator` 和 `labels`，可以筛选出作为展示的 `Issues`。它会返回一个带有 `issue` 格式对象的数组。每一个 `issue` 有很多属性，我们可能不需要那么多，先了解了解底下这几种：

```javascript
// 为了方便，我把注释写在json中了。。
[{
  "url": ,  // issue 的 url
  "id": ,  // issue id ， 是一个随机生成的不重复的数字串 
  "number": ,  // issue number ， 根据创建 issue 的顺序从1开始累加
  "title": ,  // issue 的标题
  "labels": [], // issue 的所有 label，它是一个数组
  "created_at": , // 创建 issue 的时间
  "updated_at": , // 最后修改 issue 的时间
  "body": , // issue 的内容
}]
```

#### 异步请求数据

项目中使用的异步请求数据的方法时 `fetch`。

关于 `fetch` ：[https://segmentfault.com/a/1190000003810652](https://segmentfault.com/a/1190000003810652)

使用起来很简单：

```javascript
fetch(url).then(response => response.json())
      .then(json => console.log(json))
      .catch(e => console.log(e));
```

### markdown 渲染

在 `Github` 上查找关于如何在 `React` 实现 `markdown` 的渲染，查到了这两种库：

- react-markdown：[https://github.com/rexxars/react-markdown](https://github.com/rexxars/react-markdown)
- marked：[https://github.com/chjj/marked](https://github.com/chjj/marked)

使用起来都很简单。

如果是 `react-markdown`,只需要这样做：

```javascript
import ReactMarkdown from 'react-markdown';

const input = '# This is a header\n\nAnd this is a paragraph';
ReactDOM.render(
    <ReactMarkdown source={input} />,
    document.getElementById('container')
);
```

如果是`marked`，这样做：

```javascript
import marked from 'marked';

const input = '# This is a header\n\nAnd this is a paragraph';
const output = marked(input);
```

这里有点不太一样，我们获取到了一个字符串 `output`，注意，是一个字符串，所以我们得将它插入到 `dom`中，在 `React` 中，我们可以这样做：

```html
<div dangerouslySetInnerHTML={{ __html: output }} />
```

由于我们的项目是基于 `React` 的，所以想着用 `react-markdown`会更好，而且由于安全问题 `React` 也不提倡直接往 `dom` 里插入字符串，然而在使用过程中发现，`react-markdown` 对表格的支持不友好，所以只好弃用，改用 `marked`。

### 代码高亮

代码高亮用的是`highlight.js`：[https://github.com/isagalaev/highlight.js](https://github.com/isagalaev/highlight.js)

它和`marked`可以无缝衔接~

只需要这样既可：

```javascript
import hljs from 'highlight.js';

marked.setOptions({
  highlight: code => hljs.highlightAuto(code).value,
});
```

`highlight.js`是支持多种代码配色风格的，可以在`css`文件中进行切换：

```css
@import '~highlight.js/styles/atom-one-dark.css';
```

在这可以看到每种语言的高亮效果和配色风格：[https://highlightjs.org/](https://highlightjs.org/)

## React

### state 和 props 是什么

可以看之前的一篇文章：[https://github.com/axuebin/react-blog/issues/8](https://github.com/axuebin/react-blog/issues/8)

### 关于React组件的生命周期

可以看之前的一篇文章：[https://github.com/axuebin/react-blog/issues/9](https://github.com/axuebin/react-blog/issues/9)

## 前端路由

项目中前端路由用的是 `React-Router V4`。

官方文档：[https://reacttraining.com/react-router/web/guides/quick-start](https://reacttraining.com/react-router/web/guides/quick-start)

中文文档：[http://reacttraining.cn/](http://reacttraining.cn/)

### 基本使用

```javascript
<Link to="/blog">Blog</Link>
```

```javascript
<Router>
  <Route exact path="/" component={Home} />
  <Route path="/blog" component={Blog} />
  <Route path="/demo" component={Demo} />
</Router>
```

注意：一定要在根目录的 `Route` 中声明 `exact`，要不然点击任何链接都无法跳转。

### 2级目录跳转

比如我现在要在博客页面上点击跳转，此时的 `url` 是 `localhost:8080/blog`,需要变成 `localhost:8080/blog/article`，可以这样做：

```javascript
<Route path={`${this.props.match.url}/article/:number`} component={Article} />
```

这样就可以跳转到 `localhost:8080/blog/article` 了，而且还传递了一个 `number` 参数，在 `article` 中可以通过 `this.props.params.number`获取。

### HashRouter

当我把项目托管到 `Github Page` 后，出现了这样一个问题。

> 刷新页面出现 `Cannot GET /` 提示，路由未生效。

通过了解，知道了原因是这样，并且可以解决：

- 由于刷新之后，会根据URL对服务器发送请求，而不是处理路由，导致出现 `Cannot GET /` 错误。
- 通过修改 `<Router>` → `<HashRouter>` 。
- `<HashRouter>` 借助URL上的哈希值（hash）来实现路由。可以在不需要全屏刷新的情况下，达到切换页面的目的。

### 路由跳转后不会自动回到顶部

当前一个页面滚动到一定区域后，点击跳转后，页面虽然跳转了，但是会停留在滚动的区域，不会自动回到页面顶部。

可以通过这样来解决：

```javascript
componentDidMount() {
    this.node.scrollIntoView();
}

render() {
  return (
    <div ref={node => this.node = node} ></div>
  );
}
```

## 状态管理

项目中多次需要用到从 `Github Issues` 请求来的数据，因为之前就知道 `Redux` 这个东西的存在，虽然有点大材小用，为了学习还是将它用于项目的状态管理，只需要请求一次数据即可。

官方文档：[http://redux.js.org/](http://redux.js.org/)

中文文档：[http://cn.redux.js.org/](http://cn.redux.js.org/)

简单的来说，每一次的修改状态都需要触发 `action` ，然而其实项目中我现在还没用到修改数据2333。。。

关于状态管理这一块，由于还不是太了解，就不误人子弟了~

## 主要组件

React是基于组件构建的，所以在搭建页面的开始，我们要先考虑一下我们需要一些什么样的组件，这些组件之间有什么关系，哪些组件是可以复用的等等等。

### 首页

![](http://omufjr5bv.bkt.clouddn.com/article%E9%A6%96%E9%A1%B5.gif)

可以看到，我主要将首页分成了四个部分：

- header：网站标题，副标题，导航栏
- banner：about me ~，准备用自己的照片换个背景，但是还没有合适的照片
- card area：暂时是三个卡片
  - blog card：最近的几篇博文
  - demo card：几个小demo类别
  - me card：算是我放飞自我的地方吧 
- footer：版权信息、备案信息、浏览量

### 博客页

![](http://omufjr5bv.bkt.clouddn.com/article%E5%8D%9A%E5%AE%A2%E9%A1%B5.gif)

博客页就是很中规中矩的一个页面吧，这部分是整个项目中代码量最多的部分，包括以下几部分：

- 文章列表组件
- 翻页组件
- 归档按钮组件
- 类别组件
- 标签组件

#### 文章列表

文章列表其实就是一个 `list`，里面有一个个的 `item`:

```html
<div class="archive-list">
  <div class="blog-article-item">文章1</div>
  <div class="blog-article-item">文章2</div>
<div>
```

对于每一个 `item`，其实是这样的：

![](http://omufjr5bv.bkt.clouddn.com/article%E6%96%87%E7%AB%A0item.png)

一个文章item组件它可能需要包括：

- 文章标题
- 文章发布的时间、类别、标签等
- 文章摘要
- ...

如果用 `DOM` 来描述，它应该是这样的：

```html
<div class="blog-article-item">
  <div class="blog-article-item-title">文章标题</div>
  <div class="blog-article-item-time">时间</div>
  <div class="blog-article-item-label">类别</div>
  <div class="blog-article-item-label">标签</div>
  <div class="blog-article-item-desc">摘要</div>
</div>
```

所以，我们可以有很多个组件：

- 文章列表组件 `<ArticleList />`
- 文章item组件 `<ArticleItem />`
- 类别标签组件 `<ArticleLabel />`

它们可能是这样一个关系：

```javascript
<ArticleList>
  <ArticleItem>
    <ArticleTitle />
    <ArticleTime />
    <ArticleLabel />
    <ArticleDesc />
  </ArticleItem>
  <ArticleItem></ArticleItem>
  <ArticleItem></ArticleItem>
</ArticleList>
```

#### 分页

对于分页功能，传统的实现方法是在后端完成分页然后分批返回到前端的，比如可能会返回一段这样的数据：

```javascript
{
  total:500,
  page:1,
  data:[]
}
```

也就是后端会返回分好页的数据，含有表示总数据量的`total`、当前页数的`page`，以及属于该页的数据`data`。

然而，我这个页面只是个静态页面，数据是放在Github Issues上的通过API获取的。（Github Issues的分页貌似不能自定义数量...），所以没法直接返回分好的数据，所以只能在前端强行分页~

分页功能这一块我偷懒了...用的是 `antd` 的翻页组件 `<Pagination />`。

官方文档：[https://ant.design/components/pagination-cn/](https://ant.design/components/pagination-cn/)

文档很清晰，使用起来也特别简单。

前端渲染的逻辑（有点蠢）：将数据存放到一个数组中，根据当前页数和每页显示条数来计算该显示的索引值，取出相应的数据即可。

翻页组件中：

```javascript
constructor() {
  super();
  this.onChangePage = this.onChangePage.bind(this);
}

onChangePage(pageNumber) {
  this.props.handlePageChange(pageNumber);
}

render() {
  return (
    <div className="blog-article-paging">
      <Pagination onChange={this.onChangePage} defaultPageSize={this.props.defaultPageSize} total={this.props.total} />
    </div>
  );
}
```

当页数发生改变后，会触发从父组件传进 `<ArticlePaging />` 的方法 `handlePageChange`，从而将页数传递到父组件中，然后传递到 `<ArticleList />` 中。 

父组件中：

```javascript
handlePageChange(pageNumber) {
  this.setState({ currentPage: pageNumber });
}

render() {
  return (
    <div className="archive-list-area">
      <ArticleList issues={this.props.issues} defaultPageSize={this.state.defaultPageSize} pageNumber={this.state.currentPage} />
      <ArticlePaging handlePageChange={this.handlePageChange} total={this.props.issues.length} defaultPageSize={this.state.defaultPageSize} />
    </div>
  );
}
```

列表中：

```javascript
render() {
  const articlelist = [];
  const issues = this.props.issues;
  const currentPage = this.props.pageNumber;
  const defaultPageSize = this.props.defaultPageSize;
  const start = currentPage === 1 ? 0 : (currentPage - 1) * defaultPageSize;
  const end = start + defaultPageSize < issues.length ? start + defaultPageSize : issues.length;
  for (let i = start; i < end; i += 1) {
    const item = issues[i];
    articlelist.push(<ArticleItem />);
  }
}
```

#### label

在 `Github Issues` 中，可以为一个 `issue` 添加很多个 `label`，我将这些对于博客内容有用的 `label` 分为三类，分别用不同颜色来表示。

这里说明一下， `label` 创建后会随机生成一个 `id`，虽然说 `id` 是不重复的，但是文章的类别、标签会一直在增加，当新加一个 `label` 时，程序中可能也要进行对应的修改，当作区分 `label` 的标准可能就不太合适，所以我采用颜色来区分它们。

- 表示这是一篇文章的blog：只有有 `blog` 的 `issue` 才能显示在页面上，过滤 `bug` 、`help` 等
- 表示文章类别的：用来表示文章的类别，比如“前端”、“摄影”等
- 表示文章标签的：用来表示文章的标签，比如“JavaScript”、“React”等

即使有新的 `label` ，也只要根据颜色区分是属于哪一类就好了。

##### 类别

![](http://omufjr5bv.bkt.clouddn.com/article%E7%B1%BB%E5%88%AB.gif)

在这里的思路主要就是：遍历所有 `issues`，然后再遍历每个 `issue`的 `labels`，找出属于类别的 `label`，然后计数。

```javascript
const categoryList = [];
const categoryHash = {};
for (let i = 0; i < issues.length; i += 1) {
  const labels = issues[i].labels;
  for (let j = 0; j < labels.length; j += 1) {
    if (labels[j].color === COLOR_LABEL_CATEGORY) {
      const category = labels[j].name;
      if (categoryHash[category] === undefined) {
        categoryHash[category] = true;
        const categoryTemp = { category, sum: 1 };
        categoryList.push(categoryTemp);
      } else {
        for (let k = 0; k < categoryList.length; k += 1) {
          if (categoryList[k].category === category) {
            categoryList[k].sum += 1;
          }
        }
      }
    }
  }
}
```

这样实现得要经历三次循环，复杂度有点高，感觉有点蠢，有待改进，如果有更好的方法，请多多指教~

##### 标签

![](http://omufjr5bv.bkt.clouddn.com/article%E6%A0%87%E7%AD%BE.gif)

这里的思路和类别的思路基本一样，只不过不同的显示方式而已。

本来这里是想通过字体大小来体现每个标签的权重，后来觉得可能对于我来说，暂时只有那几个标签会很频繁，其它标签可能会很少，用字体大小来区分就没有什么意义，还是改成排序的方式。

### 文章页

![](http://omufjr5bv.bkt.clouddn.com/article%E6%96%87%E7%AB%A0%E9%A1%B5.gif)

文章页主要分为两部分：

- 文章内容区域：显示文章内容，显示在页面的主体区域
- 章节目录：文章的章节目录，显示在文章的右侧区域

#### 文章内容

有两种方式获取文章具体内容：

- 从之前已经请求过的数组中去遍历查找所需的文章内容
- 通过 `issue number` 重新发一次请求直接获取内容

最后我选择了后者。

文章是用 `markdown` 语法写的，所以要先转成 `html` 然后插入页面中，这里用了一个 `React` 不提倡的属性：`dangerouslySetInnerHTML`。 

除了渲染`markdown`，我们还得对文章中的代码进行高亮显示，还有就是定制文章中不同标签的样式。

#### 章节目录

首先，这里有一个 `issue`，希望大家可以给一些建议~

文章内容是通过 `markdown` 渲染后插入 `dom` 中的，由于 `React` 不建议通过 `document.getElementById` 的形式获取 `dom` 元素，所以只能想办法通过字符串匹配的方式获取文章的各个章节标题。

由于我不太熟悉正则表达式，曾经还在sf上咨询过，就采用了其中一个答案：

```javascript
const issues = content;
const menu = [];
const patt = /(#+)\s+?(.+)/g;
let result = null;
while ((result = patt.exec(issues))) {
  menu.push({ level: result[1].length, title: result[2] });
}
```

这样可以获取到所有的 `#` 的字符串，也就是 `markdown` 中的标题， `result[1].length` 表示有几个 `#`，其实就是几级标题的意思，`title` 就是标题内容了。

这里还有一个问题，本来通过 `<a target="" />` 的方式可以实现点击跳转，但是现在渲染出来的 `html` 中对于每一个标题没有独一无二的标识。。。

### 归档页

按年份归档：

![](http://omufjr5bv.bkt.clouddn.com/article%E5%BD%92%E6%A1%A31.png)

按类别归档：

![](http://omufjr5bv.bkt.clouddn.com/article%E5%BD%92%E6%A1%A3.png)

按标签归档：

![](http://omufjr5bv.bkt.clouddn.com/article%E5%BD%92%E6%A1%A32.png)

## 问题

基本功能是已经基本实现了，现在还存在着以下几个问题，也算是一个 `TodoList` 吧

- 评论功能。拟利用 `Github Issues API` 实现评论，得实现 `Github` 授权登录
- 回到顶部。拟利用 `antd` 的组件，但是 `state` 中 `visibility` 一直是 `false`
- 首页渲染。现在打包完的js文件还是太大了，导致首页渲染太慢，这个是接下来工作的重点，也了解过关于这方面的优化：
  - `webpack` 按需加载。这可能是目前最方便的方式
  - 服务端渲染。这就麻烦了，但是好处也多，不仅解决渲染问题，还有利于SEO，所以也是 `todo` 之一
- 代码混乱，逻辑不对。这是我自己的问题，需要再修炼。


原文地址：[https://github.com/axuebin/react-blog/issues/17](https://github.com/axuebin/react-blog/issues/17)