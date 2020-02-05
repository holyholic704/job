## 简介

Vue 是一套用于构建用户界面的 **渐进式框架**。与其它大型框架不同的是，Vue 被设计为可以 **自底向上逐层应用**。Vue 的核心库只关注视图层，不仅易于上手，还便于与第三方库或既有项目整合。另一方面，当与现代化的工具链以及各种支持类库结合使用时，Vue 也完全能够为复杂的单页应用提供驱动

### 渐进式框架

主张最少，只做职责之内该做的事，不做职责之外的事，不需要一次把所有的东西都用上

### 自底而上逐层应用

作为渐进式框架要实现的目标就是方便项目增量开发

### 兼容性

Vue 不支持 IE8 及以下版本，因为 Vue 使用了 IE8 无法模拟的 ECMAScript 5 特性。但它支持所有兼容 ECMAScript 5 的浏览器

### 声明式渲染

Vue.js 的核心是一个允许采用简洁的模板语法来声明式地将数据渲染进 DOM 的系统，所有东西都是响应式的

## 使用

### el 挂载点

el 是用来设置 Vue 实例挂载的元素，Vue 会管理 el 选项命中的元素及其内部的后代元素

- 建议使用 ID 选择器
- 不能使用 HTML 和 BODY

### data 数据对象

Vue 中用到的数据定义在 data 中，可以写复杂的数据类型，渲染复杂类型数据时，遵守 JS 的语法即可

### 指令

指令带有前缀 `v-`，以表示它们是 Vue 提供的特殊 attribute，它们会在渲染的 DOM 上应用特殊的响应式行为

#### v-text

设置标签的文本值

```html
<div id="app">
    <p v-text="message"></p>
    <!-- 插值表达式：只有括号中的内容会被替换，推荐 -->
    <p>{{message}}</p>
</div>

<script>
	var app = new Vue({
        el: '#app',
        data: {
            message: 'Fuck You!'
        }
    })
</script>
```

#### v-html

设置标签的 innerHTML

```html
<div id="app">
    <p v-html="content"></p>
</div>

<script>
	var app = new Vue({
        el: '#app',
        data: {
            content: '<a href="http://www.baidu.com">Fuck You!</a>'
        }
    })
</script>
```

#### v-on

为元素绑定事件

```html
<div id="app">
  <p>{{ message }}</p>
  <!-- 事件名不需要写on，v-on:click可以简写为@click -->
  <button v-on:click="reverse">反转</button>
  <span v-on:mouseenter="changeWord">花Q!</span>
</div>

<script>
var app = new Vue({
  el: '#app',
  data: {
    message: 'Fuck You!'
  },
  methods: {
    reverse: function () {
      // 通过this关键字可以访问定义在data中的数据
      this.message = this.message.split('').reverse().join('')
    },
    changeWord: function () {
      this.message = '花Q!'
    }
  }
})
</script>
```

#### v-show

根据表达式的真假，切换元素的显示和隐藏，操纵的是 display 属性

```html
<div id="app">
  <span v-show="true">Fuck You!</span>
  <span v-show="isShow">花Q!</span>
  <span v-show="num>=1">艹!</span>
</div>

<script>
var app = new Vue({
  el: '#app',
  data: {
    isShow: false,
    num: 0
  }
})
</script>
```

#### v-if

根据表达式的真假来切换元素的显示和隐藏，操纵的是 dom 元素

```html
<div id="app">
  <p v-if="seen">you can see</p>
</div>

<script>
var app = new Vue({
  el: '#app',
  data: {
    seen: true
  }
})
</script>
```

#### v-bind

设置元素的属性

```html
<div id="app">
  <a v-bind:href="url">Click me!</a>
</div>

<script>
var app = new Vue({
  el: '#app',
  data: {
    url:'http://www.baidu.com'
  }
})
</script>
```

#### v-for

根据数据生成列表结构

```html
<div id="app">
    <ul>
        <li v-for="todo in todos">{{todo.text}}</li>
    </ul>
    <ul>
        <li v-for="(item,index) in arr">{{index+1}}: {{item}}</li>
    </ul>
</div>
<script>
var app = new Vue({
  el: '#app',
  data: {
    todos: [
        {text: '早饭'},
        {text: '午饭'},
        {text: '晚饭'}
    ],
    arr: ['one', 'two', 'three']
  }
})
</script>
```

#### v-model

双向数据绑定，获取和设置表单元素的值

```html
<div id="app">
    <input type="text" v-model="count">
    <button v-on:click="minus"> - </button>
    <button v-on:click="plus"> + </button>
</div>

<script>
var app = new Vue({
    el: '#app',
    data: {
        count:1
    },
    methods: {
        minus: function () {
            this.count = Number.parseInt(this.count) - 1
        },
        plus: function () {
            this.count = Number.parseInt(this.count) + 1
        }
    }
})
</script>
```







































```html
<div id="app">
  <p v-if="seen">you can see</p>
  <ul>
	<li v-for="todo in todos">{{todo.text}}</li>
  </ul>
</div>

<script>
// 实例化Vue对象
var app = new Vue({
  // 挂载点，表示当前Vue对象接管app的div区域
  el: '#app',
  // 数据对象
  data: {
    seen: true,
    todos: [
        {text: '早饭'},
        {text: '午饭'},
        {text: '晚饭'}
    ]
  }
})
</script>
```

### 处理用户输入

为了让用户和应用之间进行交互，可以用 `v-on` 指令添加一个事件监听器，通过它调用在 Vue 实例中定义的方法

```html
<div id="app">
  <p>{{ message }}</p>
  <button v-on:click="reverseMessage">反转</button>
</div>

<script>
var app = new Vue({
  el: '#app',
  data: {
    message: 'Hello World!'
  },
  methods: {
    reverseMessage: function () {
      this.message = this.message.split('').reverse().join('')
    }
  }
})
</script>
```

Vue 还提供了 `v-model` 指令，它能轻松实现表单输入和应用状态之间的双向绑定

```html
<div id="app">
  <p>{{ message }}</p>
  <input v-model="message">
</div>

<script>
var app = new Vue({
  el: '#app',
  data: {
    message: 'Hello World!'
  }
})
</script>
```

















