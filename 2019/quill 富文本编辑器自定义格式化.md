## quilljs
/Users/axuebin/Code/Github/articles/2019/quill 富文本编辑器自定义格式化.md
现在富文本编辑器轮子太多了，Github 上随便搜一下就有一堆，我需要实现的功能很简单，所以就佛系地选了 `quilljs`，quilljs 是一个轻量级的富文本编辑器。

链接：

- 官网：[https://quilljs.com/](https://quilljs.com/)
- Github： [https://github.com/quilljs/quill](https://github.com/quilljs/quill)
- React版本：[https://github.com/zenoamaro/react-quill](https://github.com/zenoamaro/react-quill)
- Vue版本：[https://github.com/surmon-china/vue-quill-editor](https://github.com/surmon-china/vue-quill-editor)

基础功能就不多说了，看文档就好。

主要是记录一下如何在 `toolbar` 上自定义一个按钮并实现自定义格式化。

## toolbar

`toolbar` 相关文档：[https://quilljs.com/docs/modules/toolbar/](https://quilljs.com/docs/modules/toolbar/)

### 基础用法

可以看到文档中有这么一段代码：

```javascript
var toolbarOptions = [
  ['bold', 'italic', 'underline', 'strike'],        // toggled buttons
  ['blockquote', 'code-block'],

  [{ 'header': 1 }, { 'header': 2 }],               // custom button values
  [{ 'list': 'ordered'}, { 'list': 'bullet' }],
  [{ 'script': 'sub'}, { 'script': 'super' }],      // superscript/subscript
  [{ 'indent': '-1'}, { 'indent': '+1' }],          // outdent/indent
  [{ 'direction': 'rtl' }],                         // text direction

  [{ 'size': ['small', false, 'large', 'huge'] }],  // custom dropdown
  [{ 'header': [1, 2, 3, 4, 5, 6, false] }],

  [{ 'color': [] }, { 'background': [] }],          // dropdown with defaults from theme
  [{ 'font': [] }],
  [{ 'align': [] }],

  ['clean']                                         // remove formatting button
];

var quill = new Quill('#editor', {
  modules: {
    toolbar: toolbarOptions
  },
  theme: 'snow'
});
```

这是 `toolbar` 上支持的一些格式化功能，比如 加粗、斜体、水平对齐等等常见的文档格式化。

可是我发现，貌似不太够用啊。

比如我想插入一些 `{{name}}` 这样的文本，并且是加粗的，总不能让我每次都输入 `{{}}` 吧，麻烦而且容易遗漏，得做成自动格式化的。

随手翻了一下文档，`quilljs` 支持本地 `module`，`toolbar` 上每一个格式化功能可以看作是一个 `module`，翻翻源码：

![](https://raw.githubusercontent.com/axuebin/articles/master/images/quill_format.png)

看一下 `link.js` 吧，简化了一下：

```javascript
import Inline from '../blots/inline';

class Link extends Inline {
  static create(value) {
    let node = super.create(value); // 创建一个节点
    node.setAttribute('href', value); // 将输入的 value 放到 href
    node.setAttribute('target', '_blank'); // target 设为空
    return node;
  }
  
  static formats(domNode) {
    return domNode.getAttribute('href'); // 获取放在 href 中的 value
  }
}
Link.blotName = 'link'; // bolt name
Link.tagName = 'A'; // 渲染成 html 标签

export { Link as default };

```

是不是实现一个简单的自定义格式化看上去很简单。

### 实现

以 vue 为例哈，大同小异。

#### 初始化

```html
<quill-editor ref="myTextEditor"
  class="editor-area"
  v-model="content"
  :options="editorOption"
  @blur="onEditorBlur($event)">
</quill-editor>
```

```javascript
const toolbarOptions = [
  [{ 'header': [1, 2, 3, 4, 5, 6, false] }],
  ['bold', 'italic', 'underline'],
  [{ 'color': [] }, { 'background': [] }],
  [{ 'align': [] }],
  ['formatParam'],
];
data() {
  return {
    editorOption: {
	   modules: {
	     toolbar: {
	       container: toolbarOptions,
	       handlers: {},
	     },
	   },
    }
  };
},
editor() {
  return this.$refs.myTextEditor.quill;
}
```

#### 创建

在项目组件的目录下创建一个 `formatParam.js`：

```javascript
import Quill from 'quill';

const Inline = Quill.import('blots/inline');

class formatParam extends Inline {
  static create() {
    const node = super.create();
    return node;
  }

  static formats(node) {
    return node;
  }
}
formatParam.blotName = 'formatParam';
formatParam.tagName = 'span';

export default formatParam;
```

我这里没有对 `value` 和 `node` 做任何处理，然后在 `handlers` 里做处理，貌似有点蠢。。

#### 注册

```javascript
import { quillEditor, Quill } from 'vue-quill-editor';
import FormatParam from './formatParam';

Quill.register(FormatParam);
```

#### 使用

首先在 `toolbar` 上放一个按钮是必须的，当然也可以放 `icon`，我就简单地处理一下：

```javascript
mounted() {
  const formatParamButton = document.querySelector('.ql-formatParam');
  formatParamButton.style.cssText = "width:80px; border:1px solid #ccc; border-radius:5px; padding: 0;";
  formatParamButton.innerText = "添加参数";
}
```

效果如图：

![](https://raw.githubusercontent.com/axuebin/articles/master/images/quill_toolbar.png)

然后就是要注册这个按钮的点击事件：

```javascript
handlers: {
  formatParam: () => {
    const range = this.editor.getSelection(true); // 获取光标位置
    const value = prompt('输入参数名（如：name）'); // 弹框输入返回值
    if (value) {
      this.editor.format('formatParam', value); // 格式化
      this.editor.insertText(range.index, `{{${value}}}`); // 显示在编辑器中
      this.editor.setSelection(range.index + value.length + 4, Quill.sources.SILENT); // 光标移到插入的文字后，并且让按钮失效
    }
  },
}
```

#### 渲染 html

我在自定义格式化的时候没直接渲染成 html，然后在保存的时候做了一下：

```javascript
watch: {
  content() {
    const { content } = this
    const result = content.replace(/\{\{(.*?)\}\}/g, (match, key) => `<span class="${key}">{{${key}}}</span>`);
    updateState({ content: result });
  },
}
```

这样就好了，想要插入一个参数的时候点击工具栏上的按钮，就会直接在编辑器里插入 `{{xxx}}` 的文本了。

效果如图：

![](https://raw.githubusercontent.com/axuebin/articles/master/images/quill_template.png)

## 总结

其实理论上应该可以在 `formatParam.js` 中把所有事情都做掉，也就不用最后的正则替换了。

按照这个思路，我们可以在富文本编辑器按照自己所需要的格式插入任何自定义内容。

参考链接：

- [https://github.com/quilljs/quill](https://github.com/quilljs/quill)
- [https://quilljs.com/guides/cloning-medium-with-parchment/](https://quilljs.com/guides/cloning-medium-with-parchment/)