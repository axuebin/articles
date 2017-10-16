---
layout: post
title:  "React的生命周期到底是怎么一回事？"
date:   2017-09-15 12:17:04
categories: 前端
tags: React
author: 薛彬
---

* content
{:toc}

尽量全面详细的整理一下React的生命周期中的知识点。





## 组件

组件是独立的封装的可以复用的一个小部件，它是React的核心思想之一。通过划分组件，可以将一个页面划分成独立的多个可复用的组件，各个组件通过嵌套、组合形成一个完整的页面。

在React中，组件基本由三个部分组成：属性（props）、状态（state）以及生命周期方法。可以将组件简单地看作一个“状态机”，根据不同的`state`和`props`呈现不同的UI，通过与用户的交互实现不同的状态，然后重新渲染组件，UI可以跟随数据变化而变化。

### 创建组件

组件常分为两种：`Class Component`和`Functional Component`。

#### 无状态组件

`Functional Component`也称为无状态组件，它多用于纯展示组件，这种组件只负责根据传入的`props`来渲染组件，而不涉及`state`状态管理。

> 在大部分React代码中，大多数组件被写成无状态的组件，通过简单组合可以构建成其他的组件等；这种通过多个简单然后合并成一个大应用的设计模式被提倡。

无状态组件可以通过函数形式或者ES6的箭头函数来创建：

```javascript
// 函数
function HelloFunctional(props){
  return <div>hello {props.name}</div>;
}

// ES6箭头函数
const HelloFunctional = (props) => (<div>hello {props.name}</div>);
```

无状态组件有以下几个特点：

1. 代码可读性更好
2. 组件不会被实例化，渲染性能提升
3. 无生命周期方法
4. 只能输入`props`，同样的输入一定会有同样的输出

所以，在项目中如果不需要进行状态管理，应该尽量写成无状态组件的形式。

#### 有状态组件

现在主流的创建有状态组件的形式是通过ES6的Class来创建，取代`React.createClass`：

```javascript
Class HelloClass extends React.Component{
  constructor(){
    this.state = {
      name:'axuebin'
    }
  }
  render(){
    return (<div>hello {this.state.name}</div>);
  }
}
```

这是最简洁的一个组件，它需要使用到内部状态`state`。

**当组件需要使用内部状态时或者需要使用生命周期方法时就需要使用有状态组件。**

## 组件的生命周期

React组件的生命周期可以分为挂载、渲染和卸载这几个阶段，当渲染后的组件更新后，会重新渲染组件，直到卸载。先分阶段来看看每个阶段有哪些生命周期函数。

### 挂载阶段（Mounting）

属于这个阶段的生命周期函数有：

1. constructor()
2. componentWillMount()
3. render()
4. componentDidMount()

#### constructor()

```javascript
constructor() {
  super();
  this.state = {name: 'axuebin'};
  this.handleClick = this.handleClick.bind(this); 
}
```

这个阶段就是组件的初始化，`constructor()`可以理解为组件的构造函数，从组件的类`class`实例化一个组件实例。这个函数是组件形成时就被调用的，是生命周期中最先执行的。

在`constructor()`函数内，首先必须执行`super()`，否则`this.props`将是未定义，会引发异常。

然后，如果有必要，可以进行：

- `state`的初始化
- 方法的绑定

如果不需要这两步，可以直接省略`constructor`函数。

有一点，在`constructor()`中，`this.props`返回`undefined`。

#### componentWillMount()

这个函数按照驼峰法的命名规则可以理解为“组件即将被挂载”，所以这个函数是组件首次渲染（render）前调用的。

在每次页面加载、刷新时，或者某个组件第一次展现时都会调用这个函数。通常地，我们推荐使用`constructor()`来替代。

**注意：在这个函数中，不可以调用`setState`来修改状态。**

#### render()

```javascript
render() {
  return(
    <div>hello {this.state.name} {this.props.age}</div>
  )
}
```

`render()`在生命周期中是必须的，是渲染组件用的。

当这个函数被调用时，需要检查`this.props`和`this.state`并且返回一个元素（有且只有一个元素），这个元素可能是一个原生DOM元素，也有可能是另一个React组件。

可以在`state`或`props`状态为空时试着返回一个`null`或者`false`来声明不想渲染任何东西。

在这个函数中，不应该改变组件的状态，也就是不执行`this.setState`，需要保持`render()`函数的纯净。

在这个函数中，可以对`props`进行调用并组合，但不可修改。

#### componentDidMount()

```javascript
componentDidMount() {
  this.setState({name:'xb'});
}
```

这个函数在组件加载渲染完成后立即调用，此时页面上已经渲染出真实的DOM了，可以在这个函数中访问到真实的DOM（可以通过`this.refs`来访问真实DOM）。

在这个阶段，还可以做一件事，可以修改`state`了！！！

而且，异步获取数据在这个阶段执行也是比较合理的，获取数据之后`setState`，然后重新渲染组件。

### 更新阶段（Updating）

属性或状态的改变会触发一次更新。当一个组件在被重新渲染时，这些方法将会被调用：

1. componentWillReceiveProps()
2. shouldComponentUpdate()
3. componentWillUpdate()
4. render()
5. componentDidUpdate()

#### componentWillReceiveProps()

已加载的组件在`props`发生变化时调用，若需要更新状态，可能需要对比`this.props`和`nextProps`然后在该方法中使用`this.setState`来处理状态的改变。

需要注意的是，有些情况下，即使`props`未改变也会触发该函数，所以一定要先比较`this.props`和`nextProps`再做操作。

该函数只监听`props`的改变，`this.setState`不会触发这个函数。

```javascript
componentWillReceiveProps(nextProps){
  if (this.props.color !== nextProps.color){
    this.setState({});
  }
}
```

#### shouldComponentUpdate()

这个函数只返回`true`或`false`，表示组件是否需要更新（重新渲染）。

1. 返回`true`就是紧接着以下的生命周期函数；
2. 返回`false`表示组件不需要重新渲染，不再执行任何生命周期函数（包括render）。

这个函数使用需谨慎，react官方文档中说道，在未来这个函数返回`false`可能仍然使得组件重新渲染。

#### componentWillUpdate()

这个函数看名字就和`componentWillMount`很像，它执行的阶段也很像。在接收到新的`props`或者`state`之后，这个函数就会在`render`前被调用。

同样的，在这个函数中不能使用`this.setState()`。如果需要更新状态，请在`componentWillReceiveProps`中调用`this.setState()`。

#### render()

又是一次的`render`。这和挂载阶段的`render`有什么区别呢？

在函数的性质上来说，两者毫无区别，只不过是在生命周期的不同阶段的调用。

- 前一个`render`是在组件第一次加载时调用的，也就是初次渲染，可以理解为`mount`；
- 后一个`render`是除去第一次之后调用的，也就是再渲染，`re-render`；

#### componentDidUpdate()

同样地，这个方法是在组件`re-render`之后调用的，该方法不会在初始化的时候调用。和`componentDidMount`一样，在这个函数中可以使用`this.refs`获取真实DOM。

还可以修改`state`哦，不过会导致组件再次`re-render`。

### 卸载阶段（Unmounting）

该方法将会在 component 从DOM中移除时调用

- componentWillUnmount()

#### componentWillUnmount()

卸载阶段就很简单了，就这一个生命周期函数，在组件被卸载和销毁之前立刻调用。

在这个函数中，应该处理任何必要的清理工作，比如销毁定时器、取消网络请求、清除之前创建的相关DOM节点等。

## 生命周期流程图

![](http://omufjr5bv.bkt.clouddn.com/react-lifecycle%E6%9C%AA%E5%91%BD%E5%90%8D%E6%96%87%E4%BB%B6.png)


