最近一个活动页面中有一个小需求，用户点击或者长按就可以复制内容到剪贴板，记录一下实现过程和遇到的坑。

## 常见方法

查了一下万能的Google，现在常见的方法主要是以下两种：

- 第三方库：clipboard.js
- 原生方法：document.execCommand()

分别来看看这两种方法是如何使用的。

## clipboard.js

这是clipboard的官网：[https://clipboardjs.com/](https://clipboardjs.com/)，看起来就是这么的简单。

### 引用

直接引用： `<script src="dist/clipboard.min.js"></script>`

包： `npm install clipboard --save` ，然后 `import Clipboard from 'clipboard';`

### 使用

#### 从输入框复制

现在页面上有一个 `<input>` 标签，我们需要复制其中的内容，我们可以这样做：

```html
<input id="demoInput" value="hello world">
<button class="btn" data-clipboard-target="#demoInput">点我复制</button>
```

```javascript
import Clipboard from 'clipboard';
const btnCopy = new Clipboard('btn');
```

注意到，在 `<button>` 标签中添加了一个 `data-clipboard-target` 属性，它的值是需要复制的 `<input>` 的 `id`，顾名思义是从整个标签中复制内容。

#### 直接复制

有的时候，我们并不希望从 `<input>` 中复制内容，仅仅是直接从变量中取值。如果在 `Vue` 中我们可以这样做：

```html
<button class="btn" :data-clipboard-text="copyValue">点我复制</button>
``` 

```javascript
import Clipboard from 'clipboard';
const btnCopy = new Clipboard('btn');
this.copyValue = 'hello world';
```

#### 事件

有的时候我们需要在复制后做一些事情，这时候就需要回调函数的支持。

在处理函数中加入以下代码：

```javascript
// 复制成功后执行的回调函数
clipboard.on('success', function(e) {
    console.info('Action:', e.action); // 动作名称，比如：Action: copy
    console.info('Text:', e.text); // 内容，比如：Text：hello word
    console.info('Trigger:', e.trigger); // 触发元素：比如：<button class="btn" :data-clipboard-text="copyValue">点我复制</button>
    e.clearSelection(); // 清除选中内容
});

// 复制失败后执行的回调函数
clipboard.on('error', function(e) {
    console.error('Action:', e.action);
    console.error('Trigger:', e.trigger);
});
```

### 小结

文档中还提到，如果在单页面中使用 `clipboard` ，为了使得生命周期管理更加的优雅，在使用完之后记得 `btn.destroy()` 销毁一下。

`clipboard` 使用起来是不是很简单。但是，就为了一个 `copy` 功能就使用额外的第三方库是不是不够优雅，这时候该怎么办？那就用原生方法实现呗。

## document.execCommand()方法

先看看这个方法在 `MDN` 上是怎么定义的：

> which allows one to run commands to manipulate the contents of the editable region.

意思就是可以允许运行命令来操作可编辑区域的内容，注意，是**可编辑区域**。

### 定义

> bool = document.execCommand(aCommandName, aShowDefaultUI, aValueArgument)

方法返回一个 `Boolean` 值，表示操作是否成功。

- `aCommandName` ：表示命令名称，比如： `copy`, `cut` 等（更多命令见[命令](https://developer.mozilla.org/en-US/docs/Web/API/Document/execCommand#%E5%91%BD%E4%BB%A4)）；
- `aShowDefaultUI`：是否展示用户界面，一般情况下都是 `false`；
- `aValueArgument`：有些命令需要额外的参数，一般用不到；

### 兼容性

这个方法在之前的兼容性其实是不太好的，但是好在现在已经基本兼容所有主流浏览器了，在移动端也可以使用。

![兼容性](http://omufjr5bv.bkt.clouddn.com/execCommand.png)

### 使用

#### 从输入框复制

现在页面上有一个 `<input>` 标签，我们想要复制其中的内容，我们可以这样做：

```html
<input id="demoInput" value="hello world">
<button id="btn">点我复制</button>
```

```javascript
const btn = document.querySelector('#btn');
btn.addEventListener('click', () => {
	const input = document.querySelector('#demoInput');
	input.select();
	if (document.execCommand('copy')) {
		document.execCommand('copy');
		console.log('复制成功');
	}
})
```

#### 其它地方复制

有的时候页面上并没有 `<input>` 标签，我们可能需要从一个 `<div>` 中复制内容，或者直接复制变量。

还记得在 `execCommand()` 方法的定义中提到，它只能操作**可编辑区域**，也就是意味着除了 `<input>`、`<textarea>` 这样的输入域以外，是无法使用这个方法的。

这时候我们需要曲线救国。

```html
<button id="btn">点我复制</button>
```

```javascript
const btn = document.querySelector('#btn');
btn.addEventListener('click',() => {
	const input = document.createElement('input');
	document.body.appendChild(input);
 	input.setAttribute('value', '听说你想复制我');
	input.select();
	if (document.execCommand('copy')) {
		document.execCommand('copy');
		console.log('复制成功');
	}
    document.body.removeChild(input);
})
```

算是曲线救国成功了吧。在使用这个方法时，遇到了几个坑。

#### 遇到的坑

在Chrome下调试的时候，这个方法时完美运行的。然后到了移动端调试的时候，坑就出来了。

对，没错，就是你，ios。。。

1. 点击复制时屏幕下方会出现白屏抖动，仔细看是拉起键盘又瞬间收起

	知道了抖动是由于什么产生的就比较好解决了。既然是拉起键盘，那就是聚焦到了输入域，那只要让输入域不可输入就好了，在代码中添加 `input.setAttribute('readonly', 'readonly');` 使这个 `<input>` 是只读的，就不会拉起键盘了。

2. 无法复制

	这个问题是由于 `input.select()` 在ios下并没有选中全部内容，我们需要使用另一个方法来选中内容，这个方法就是 `input.setSelectionRange(0, input.value.length);`。
	
完整代码如下：

```javascript
const btn = document.querySelector('#btn');
btn.addEventListener('click',() => {
	const input = document.createElement('input');
    input.setAttribute('readonly', 'readonly');
    input.setAttribute('value', 'hello world');
    document.body.appendChild(input);
	input.setSelectionRange(0, 9999);
	if (document.execCommand('copy')) {
		document.execCommand('copy');
		console.log('复制成功');
	}
    document.body.removeChild(input);
})
```

## 总结

以上就是关于JavaScript如何实现复制内容到剪贴板，附上几个链接：

[execCommand MDN](https://developer.mozilla.org/en-US/docs/Web/API/Document/execCommand)

[execCommand兼容性](https://caniuse.com/#search=execCommand)

[clipboard.js](https://github.com/zenorocha/clipboard.js)