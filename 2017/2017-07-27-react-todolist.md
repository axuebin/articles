---
layout: post
title:  "用React实现一个简易的TodoList"
date:   2017-07-27 14:29:10
categories: 前端
tags: React
author: 薛彬
---

* content
{:toc}

初学React，撸一个TodoList熟悉熟悉基本语法，只有最简单最简单的功能。





![](http://i.imgur.com/tT18EpC.png)



如上图所示，是一个最简单的TodoList的样子了，我们应该怎样把它拆成一个个的组件呢？

在之前看来，可能就是这样一个HTML结构：

```html
<div>
  <h1></h1>
  <div>
    <ul>
      <li></li>
      <li></li>
      <li></li>
    </ul>
  </div>
  <div>
    <input/>
    <button>保存</button>
  </div>
</div>
```

> React的核心思想是：封装组件。

我们也可以按照这个思路来进行组件设计

## 组件设计

![](http://i.imgur.com/bp6NaWf.png)

从小到大，从内到外 ~ 

我是这样进行设计的。

除去按钮，input这些之外，`<li></li>`是HTML中最小的元素，我们可以先每一个`<li></li>`当成是一个最小的组件，也就是图中橙色框的部分，它对应着每一条内容，我们先把它命名为`TodoItem`吧。

`<li></li>`的父级元素是`<ul></ul>`，那就把它看作一个组件呗，图中位于上方的蓝色部分，命名为`TodoList`。

恩，此时Todo内容的展示组件已经是够的了，我们再来加一个添加Todo内容的组件`AddTodoItem`吧，命名貌似有点丑- -，图中位于下方的蓝色部分。

最后就是最外层的红色部分了，它就是整个app的主体部分，包含着其它小组件，命名为`TodoBox`。

ok，暂时就这几个小组件 ~ 

然我们开始愉快的撸代码吧 ~

## 代码部分

### Index

先看看入口程序，很简单。

```javascript
var React = require('react');
var ReactDOM = require('react-dom');
import TodoBox from './components/todobox';
import './../css/index.css';

export default class Index extends React.Component {
  constructor(){
    super();
  };
  render() {
    return (
        <TodoBox />
    );
  }
}

ReactDOM.render(<Index/>,document.getElementById("example"))
```

### TodoItem

让我们想想啊，对于每一条内容来说，需要什么呢？

- 一个确认是否完成的`checkbox` [ ]
- 一条内容`text`
- 一个删除`button`
- zzzzzz.....其他的暂时先不加了~

那不是太简单了 ~

```html
<li>
  <input type="checkbox"/>找工作啊找工作啊
  <button>删除</button>
</li>
```

不不不，我们现在是在写`React`,要这样：

```javascript
import React from 'react';
import {Row, Col, Checkbox, Button} from 'antd';

export default class TodoItem extends React.Component {
  constructor(props) {
    super(props)
    this.toggleComplete = this.toggleComplete.bind(this)
    this.deleteTask = this.deleteTask.bind(this)
  }
  toggleComplete() {
    this.props.toggleComplete(this.props.taskId)
  }
  deleteTask() {
    this.props.deleteTask(this.props.taskId)
  }
  render() {
    let task = this.props.task
    let itemChecked
    if (this.props.complete === "true") {
      task = <del>{task}</del>
      itemChecked = true
    } else {
      itemChecked = false
    }
    return (
      <li className="list-group-item">
        <Row>
          <Col span={12}>
            <Checkbox checked={itemChecked} onChange={this.toggleComplete}/> {task}
          </Col>
          <Col span={12}>
            <Button type="danger" className="pull-right" onClick={this.deleteTask}>删除</Button>
          </Col>
        </Row>
      </li>
    )
  }
}
```

`import {Row, Col, Checkbox, Button} from 'antd'`是引入Ant Design。

> 我们采用 React 封装了一套 Ant Design 的组件库，也欢迎社区其他框架的实现版本。

引入这个之后，我们可以直接使用一些简单的UI组件，比如`Row`,`Col`,`Checkbox`,`Button`等，我们可以更加注重业务逻辑的实现。

### TodoList

接下来就是拿一个`<ul></ul>`把item包起来呗：

```javascript
import React from 'react';
import TodoItem from './todoitem';
export default class TodoList extends React.Component{
  constructor(props) {
    super(props);
  }
  render(){
    var taskList=this.props.data.map(listItem=>
      <TodoItem taskId={listItem.id}
                key={listItem.id}
                task={listItem.task}
                complete={listItem.complete}
                toggleComplete={this.props.toggleComplete}
                deleteTask={this.props.deleteTask}/>
    )
    return(
      <ul className="list-group">
        {taskList}
      </ul>
    )
  }
}
```

### AddTodoItem

添加内容这个组件也比较简单，就只需要一个`input`和一个`button`即可：

```javascript
import React from 'react';
import ReactDOM from 'react-dom';
import {Row, Col, Form, Input, Button,notification } from 'antd';
export default class AddTodoItem extends React.Component {
  constructor(props) {
    super(props)
    this.saveNewItem = this.saveNewItem.bind(this)
  }
  saveNewItem(e) {
    e.preventDefault()
    let element = ReactDOM.findDOMNode(this.refs.newItem)
    let task = element.value
    if (!task) {
      notification.open({
        description: 'Todo内容不得为空！',
    });
    } else {
      this.props.saveNewItem(task)
      element.value = ""
    }
  }
  render() {
    return (
      <div className="addtodoitem">
        <Form.Item>
          <label htmlFor="newItem"></label>
          <Input id="newItem" ref="newItem" type="text" placeholder="吃饭睡觉打豆豆~"></Input>
          <Button type="primary" className="pull-right" onClick={this.saveNewItem}>保存</Button>
        </Form.Item>
      </div>
    )
  }
}
```

### TodoBox

我们的小组件已经都实现了，拿一个大`box`包起来呗 ~ 

```javascript
import React from 'react';
import TodoList from './todolist';
import AddTodoItem from './addtodoitem';
import {Button, Icon, Row, Col} from 'antd';
export default class TodoBox extends React.Component {
  constructor(props) {
    super(props)
    this.state = {
      data: [
        {
          "id": "1",
          "task": "做一个TodoList Demo",
          "complete": "false"
        }, {
          "id": "2",
          "task": "学习ES6",
          "complete": "false"
        }, {
          "id": "3",
          "task": "Hello React",
          "complete": "true"
        }, {
          "id": "4",
          "task": "找工作",
          "complete": "false"
        }
      ]
    }
    this.handleToggleComplete = this.handleToggleComplete.bind(this);
    this.handleTaskDelete = this.handleTaskDelete.bind(this);
    this.handleAddTodoItem = this.handleAddTodoItem.bind(this);
  }
  generateGUID() {
    return 'xxxxxxxx-xxxx-4xxx-yxxx-xxxxxxxxxxxx'.replace(/[xy]/g, function(c) {
      var r = Math.random() * 16 | 0,
        v = c == 'x' ? r : (r & 0x3 | 0x8)
      return v.toString(16)
    })
  }
  handleToggleComplete(taskId) {
    let data = this.state.data;
    for (let item of data) {
      if (item.id === taskId) {
        item.complete = item.complete === "true" ? "false" : "true"
      }
    }
    this.setState({data})
  }
  handleTaskDelete(taskId) {
    let data = this.state.data
    data = data.filter(task => task.id !== taskId)
    this.setState({data})
  }
  handleAddTodoItem(task) {
    let newItem = {
      id: this.generateGUID(),
      task,
      complete: "false"
    }
    let data = this.state.data
    data = data.concat([newItem])
    this.setState({data})
  }
  render() {
    return (
      <div>
        <div className="well">
          <h1 className="text-center">React TodoList</h1>
          <TodoList data={this.state.data} toggleComplete={this.handleToggleComplete} deleteTask={this.handleTaskDelete}/>
          <AddTodoItem saveNewItem={this.handleAddTodoItem}/>
        </div>
        <Row>
          <Col span={12}></Col>
          <Col span={12}>
            <Button className="pull-left"><Icon type="user"/>
              <a href="http://axuebin.com">薛彬</a>
            </Button>
            <Button className="pull-right"><Icon type="github"/>
              <a href="https://github.com/axuebin">axuebin</a>
            </Button>
          </Col>
        </Row>
      </div>
    )
  }
}
```

注意：

- 通过props传递子组件需要的值和方法
- 传递方法时一定要bind(this)，不然内部this会指向不正确

## 源码

完整的Demo代码在这：[https://github.com/axuebin/react-todolist](https://github.com/axuebin/react-todolist "https://github.com/axuebin/react-todolist") 
