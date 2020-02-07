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

## Vue 实例

每个 Vue 应用都是通过用 new Vue 创建一个新的 Vue 实例开始的

### el 挂载点

el 是用来设置 Vue 实例挂载的元素，Vue 会管理 el 选项命中的元素及其内部的后代元素

- 建议使用 ID 选择器
- 不能使用 html 和 body 标签

### data 数据对象

Vue 中用到的数据定义在 data 中，可以写复杂的数据类型，渲染复杂类型数据时，遵守 JS 的语法即可

### 数据与方法

当一个 Vue 实例被创建时，它将 data 对象中的所有的属性加入到 Vue 的 **响应式系统** 中。当这些属性的值发生改变时，视图会进行重渲染。值得注意的是，只有 **当实例被创建时就已经存在于 data 中的属性** 才是响应式的，如果添加一个新的属性，将不会触发任何视图的更新，可以选择为之后需要的属性设置一个初始值

- 唯一的例外是使用 `Object.freeze()`，可以冻结一个对象，被冻结对象自身的所有属性都不可能以任何方式被修改

### 实例生命周期钩子

每个 Vue 实例在被创建时都要经过一系列的初始化过程，例如，需要设置数据监听、编译模板、将实例挂载到 DOM 并在数据变化时更新 DOM 等。同时在这个过程中也会运行一些叫做 **生命周期钩子** 的函数，这给了用户在不同阶段添加自己的代码的机会

```javascript
beforeCreate: function () {
	console.log('beforeCreate')
}
```

### 生命周期

![lifecycle](../md.assets/lifecycle.png)

## 模版语法

Vue.js 使用了基于 HTML 的模板语法，允许开发者声明式地将 DOM 绑定至底层 Vue 实例的数据。所有 Vue.js 的模板都是合法的 HTML ，所以能被遵循规范的浏览器和 HTML 解析器解析

- 在底层的实现上，Vue 将模板编译成虚拟 DOM 渲染函数。结合响应系统，Vue 能够 智能地计算出最少需要重新渲染多少组件，并 **把 DOM 操作次数减到最少**

### 插值

#### 文本

数据绑定最常见的形式就是使用 Mustache 语法（双大括号）的文本插值，Mustache 标签将会被替代为对应数据对象上属性的值。无论何时，绑定的数据对象上属性发生了改变，插值处的内容都会更新

```html
<span>{{ message }}</span>
```

通过使用 `v-once` 指令，可以执行一次性地插值，当数据改变时，插值处的内容不会更新。但可能会影响到该节点上的其它数据绑定

```html
<!-- 使用v-once指令，修改message属性值后不会改变 -->
<div id="app">
    <span v-once>{{ message }}</span>
    <span>{{ message }}</span>
    <button v-on:click="message = 'Vue No.1!'">改变</button>
</div>
```

#### 原始 HTML

双大括号会将数据解释为普通文本，而非 HTML 代码。为了输出真正的 HTML，可以使用 `v-html` 指令，**但会忽略解析属性值中的数据绑定**

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

#### Attribute

Mustache 语法不能作用在 HTML attribute 上，遇到这种情况应该使用 `v-bind` 指令

```html
<div v-bind:id="dynamicId"></div>
```

对于布尔 attribute（它们只要存在就意味着值为 true），`v-bind` 工作起来略有不同，如果值是 null、undefined 或 false，则不会被包含在渲染出来的元素中

```html
<!-- isButtonDisabled不为true的话，button不会渲染出disabled属性-->
<button v-bind:disabled="isButtonDisabled">Button</button>
```

#### 使用 JavaScript 表达式

除了绑定简单的属性键值外，对于所有的数据绑定，Vue.js 都提供了完全的 JavaScript 表达式支持，每个绑定都只能包含 **单个表达式**

```html
{{ number + 1 }}
{{ ok ? 'YES' : 'NO' }}
{{ message.split('').reverse().join('') }}
<div v-bind:id="'list-' + id"></div>
```

```html
<!-- 错误的示例 -->
<!-- 这是语句，不是表达式 -->
{{ var a = 1 }}
<!-- 流控制也不会生效，可以使用三元表达式 -->
{{ if (ok) { return message } }}
```

### 指令

指令带有前缀 `v-`，以表示它们是 Vue 提供的特殊 attribute。指令 attribute 的值预期是单个 JavaScript 表达式。指令的职责是，当表达式的值改变时，将其产生的连带影响，**响应式** 地作用于 DOM

#### 参数

一些指令能够接收一个参数，在指令名称之后以冒号表示

- `v-bind` 指令可以用于响应式地更新 HTML attribute

```html
<a v-bind:href="url">Click me!</a>
```

- `v-on` 指令，可以用于监听 DOM 事件

```html
<a v-on:click="doSomething">Click me!</a>
```

`v-` 前缀作为一种视觉提示，用来识别模板中 Vue 特定的 attribute。在使用 Vue.js 为现有标签添加动态行为时，`v-` 前缀很有帮助，然而，对于一些频繁用到的指令来说，就会感到使用繁琐。同时，在构建由 Vue 管理所有模板的单页面应用程序（SPA）时，`v-` 前缀也变得没那么重要了。因此，**Vue 为 `v-bind` 和 `v-on` 这两个最常用的指令，提供了特定简写**

- `v-bind` 缩写

```html
<a v-bind:href="url">Click me!</a>
<!-- 缩写 -->
<a :href="url">Click me!</a>
```

- `v-on` 的缩写

```html
<a v-on:click="doSomething">Click me!</a>
<!-- 缩写 -->
<a @click="doSomething">Click me!</a>
```

#### 动态参数

从 2.6.0 开始，可以用方括号括起来的 JavaScript 表达式作为一个指令的参数

```html
<!-- 可以在data中添加attribute_name属性，赋值为'href' -->
<a v-bind:[attribute_name]="url">Click me!</a>
```

- 对动态参数的值的约束：动态参数预期会求出一个字符串，异常情况下值为 null。这个特殊的 null 值可以被显性地用于移除绑定。任何其它非字符串类型的值都将会触发一个警告
- 对动态参数表达式的约束：动态参数表达式有一些语法约束，因为某些字符，如空格和引号，放在 HTML attribute 名里是无效的，例如 `v-bind:['foo' + bar]="value"`，可以使用没有空格或引号的表达式，或用计算属性替代这种复杂表达式
- 在 DOM 中使用模板时（直接在一个 HTML 文件里撰写模板），还需要 **避免使用大写字符来命名键名**，因为浏览器会把 attribute 名全部强制转为小写，例如 `v-bind:[someAttr]` 会被转换为 `v-bind:[someattr]`

#### 修饰符

修饰符是以半角句号 `.` 指明的特殊后缀，用于指出一个指令应该以特殊方式绑定

```html
<!-- .prevent修饰符告诉v-on指令对于触发的事件调用event.preventDefault() -->
<form v-on:submit.prevent="onSubmit">...</form>
```

## 计算属性

模板内的表达式非常便利，但是设计它们的初衷是用于简单运算的。在模板中放入太多的逻辑会让模板过重且难以维护，所以，对于任何复杂逻辑，都应当使用计算属性

```html
<div>
  {{ message.split('').reverse().join('') }}
</div>

<!-- 修改为 -->
<div>
  <p>Original message: "{{ message }}"</p>
  <p>Computed reversed message: "{{ reversedMessage }}"</p>
</div>
<script>
var vm = new Vue({
  data: {
    message: 'Hello'
  },
  computed: {
    reversedMessage: function () {
      return this.message.split('').reverse().join('')
    }
  }
})
</script>
```

可以像绑定普通属性一样在模板中绑定计算属性。Vue 知道 `vm.reversedMessage` 依赖于 `vm.message`，因此当 `vm.message` 发生改变时，所有依赖 `vm.reversedMessage`的绑定也会更新。而且最妙的是已经以声明的方式创建了这种依赖关系：计算属性的 getter 函数是没有副作用的，这使它更易于测试和理解

### 计算属性缓存 vs 方法

可以将同一函数定义为一个方法而不是一个计算属性。两种方式的最终结果是完全相同的。然而，不同的是计算属性是基于它们的响应式依赖进行 **缓存** 的。只在相关响应式依赖发生改变时它们才会重新求值。这就意味着只要 message 还没有发生改变，多次访问 reversedMessage 计算属性会立即返回之前的计算结果，而不必再次执行函数。而调用方法的话，每当触发重新渲染时，都会再次执行函数

```javascript
methods: {
  reversedMessage: function () {
    return this.message.split('').reverse().join('')
  }
}

computed: {
  reversedMessage: function () {
    return this.message.split('').reverse().join('')
  }
}
```

### 计算属性 vs 侦听属性

Vue 提供了一种更通用的方式来观察和响应 Vue 实例上的数据变动：**侦听属性**。当有一些数据需要随着其它数据变动而变动时，就很容易滥用 watch。然而，通常更好的做法是使用计算属性而不是命令式的 watch 回调

```html
<div>{{ fullName }}</div>

<script>
var vm = new Vue({
  data: {
    firstName: 'Foo',
    lastName: 'Bar',
    fullName: 'Foo Bar'
  },
  watch: {
    firstName: function (val) {
      this.fullName = val + ' ' + this.lastName
    },
    lastName: function (val) {
      this.fullName = this.firstName + ' ' + val
    }
  }
})

// 可修改为
var vm = new Vue({
  data: {
    firstName: 'Foo',
    lastName: 'Bar'
  },
  computed: {
    fullName: function () {
      return this.firstName + ' ' + this.lastName
    }
  }
})
</script>
```

#### 计算属性的 setter

计算属性默认只有 getter ，不过在需要时也可以提供一个 setter 

```javascript
computed: {
  fullName: {
    get: function () {
      return this.firstName + ' ' + this.lastName
    },
    set: function (newValue) {
      var names = newValue.split(' ')
      this.firstName = names[0]
      this.lastName = names[names.length - 1]
    }
  }
}
```

### 侦听器

虽然计算属性在大多数情况下更合适，但有时也需要一个自定义的侦听器。Vue 通过 watch 选项提供了一个更通用的方法，来响应数据的变化。当需要在数据变化时执行异步或开销较大的操作时，这个方式是最有用的

> 
>
> 施工中
>
> 

## Class 与 Style 绑定

操作元素的 class 列表和内联样式是数据绑定的一个常见需求。因为它们都是属性，所以可以用 `v-bind` 处理它们：只需要通过表达式计算出字符串结果即可。不过，字符串拼接麻烦且易错。因此，在将 `v-bind` 用于 class 和 style 时，Vue.js 做了专门的增强。表达式结果的类型除了字符串之外，还可以是对象或数组

### 绑定 HTML Class

#### 对象语法

```html
<div v-bind:class="{ active: isActive }"></div>
<script>
var vm = new Vue({
    data: {
        isActive: true
    }
})
</script>
```

也可以在对象中传入更多属性来动态切换多个 class。此外，`v-bind:class` 指令也可以与普通的 class 属性共存

```html
<!-- text-danger加上引号是因为有- -->
<div class="static"
  v-bind:class="{ active: isActive, 'text-danger': hasError }"></div>

<script>
var vm = new Vue({
    data: {
        isActive: true,
        hasError: true
    }
})
</script>
```

#### 数组语法

- 可以把一个数组传给 `v-bind:class`，以应用一个 class 列表

```html
<div v-bind:class="[activeClass, errorClass]"></div>

<script>
var vm = new Vue({
    data: {
        activeClass: 'active',
  		errorClass: 'text-danger'
    }
})
</script>
```

- 也可以根据条件切换列表中的 class，可以用三元表达式等

```html
<div v-bind:class="[isActive ? activeClass : '', errorClass]"></div>
```

- 当有多个条件 class 时这样写有些繁琐，所以在数组语法中也可以使用对象语法

```html
<div v-bind:class="[{ active: isActive }, errorClass]"></div>
```

#### 用在组件上

> 
>
> 施工中
>
> 

### 绑定内联样式

#### 对象语法

`v-bind:style` 的对象语法十分直观，看着非常像 CSS，但其实是一个 JavaScript 对象。CSS 属性名可以用驼峰式或短横线分隔（记得用引号括起来）来命名

```html
<!-- 驼峰式 -->
<div v-bind:style="{ color: activeColor, fontSize: fontSize + 'px' }"></div>
<!-- 短横线分隔 -->
<div v-bind:style="{ color: activeColor, 'font-size': fontSize + 'px' }"></div>
```

直接绑定到一个样式对象通常更好，这会让模板更清晰

```html
<div v-bind:style="styleObject"></div>
<script>
var vm = new Vue({
    data: {
        styleObject: {
            color: 'red',
            fontSize: '13px'
        }
    }
})
</script>
```

#### 数组语法

`v-bind:style` 的数组语法可以将多个样式对象应用到同一个元素上

```html
<div v-bind:style="[baseStyles, overridingStyles]"></div>
```

#### 自动添加前缀

当 `v-bind:style` 使用需要添加浏览器引擎前缀的 CSS 属性时，如 `transform`，Vue.js 会 **自动侦测并添加相应的前缀**

- `-webkit-`：谷歌、Safari、新版 Opera 浏览器，以及几乎所有 iOS 系统中的浏览器（包括 iOS 系统中的火狐浏览器）；简单的说，所有基于 WebKit 内核的浏览器
- `-moz- `：火狐浏览器
- `-o-`：旧版 Opera 浏览器
- `-ms-`：IE 浏览器 和 Edge 浏览器

#### 多重值

从 2.3.0 起可以为 style 绑定中的属性提供一个包含多个值的数组，常用于提供多个带前缀的值

```html
<!-- 会渲染数组中最后一个被浏览器支持的值 -->
<div :style="{ display: ['-webkit-box', '-ms-flexbox', 'flex'] }"></div>
```

## 条件渲染

### v-if

`v-if` 指令用于条件性地渲染一块内容。这块内容只会在指令的表达式返回 truthy 值的时候被渲染

- truthy（真值）指的是在布尔值上下文中，转换后的值为真的值
  - 所有值都是真值，除非它们被定义为假值（即除 false、0、""、null、undefined 和 NaN 以外皆为真值）

```html
<div v-if="type === 'A'"> A </div>
<div v-else-if="type === 'B'"> B </div>
<div v-else-if="type === 'C'"> C </div>
<div v-else> Not A/B/C </div>
```

- `v-else`，`v-else-if` 必须紧跟在带 `v-if` 或者 `v-else-if` 的元素之后

#### 在 `<template>` 元素上使用 v-if 条件渲染分组

> 
>
> 施工中
>
> 

#### 用 key 管理可复用的元素

> 
>
> 施工中
>
> 

### v-show

`v-show` 的用法与 `v-if` 的用法大致一样，不同的是带有 `v-show` 的元素始终会被渲染并保留在 DOM 中。`v-show` 只是简单地切换元素的 CSS 属性 display，而 `v-if` 操纵的是 dom 元素

- `v-show` 不支持 `<template>` 元素，也不支持 `v-else`

### v-if 与 v-show

- `v-if` 是真正的条件渲染，因为它会确保在切换过程中条件块内的事件监听器和子组件适当地被销毁和重建

- `v-if` 是 **惰性的**，如果在初始渲染时条件为假，则什么也不做，**直到条件第一次变为真时，才会开始渲染条件块**。而 `v-show` 就简单得多，**不管初始条件是什么，元素总是会被渲染**，并且只是简单地基于 CSS 进行切换

- 一般来说，`v-if` 有 **更高的切换开销**，而 `v-show` 有 **更高的初始渲染开销**
  - 如果需要非常频繁地切换，则使用 `v-show` 较好
  - 如果在运行时条件很少改变，则使用 `v-if` 较好

- 当 `v-if` 与 `v-for` 一起使用时，`v-for` 具有比 `v-if` 更高的优先级，**但不推荐一起使用**

## 列表渲染

### 用 v-for 把一个数组对应为一组元素

可以用 `v-for` 指令基于一个数组来渲染一个列表。`v-for` 指令需要使用 `item in items` 形式的特殊语法，其中 `items` 是源数据数组，而 `item` 则是被迭代的数组元素的 **别名**

```html
<ul id="app">
  <li v-for="item in items">
    {{ item.message }}
  </li>
</ul>

<script>
var vm = new Vue({
  data: {
    items: [
      { message: 'Foo' },
      { message: 'Bar' }
    ]
  }
})
</script>
```

在 `v-for` 块中，我们可以访问所有父作用域的属性。`v-for` 还支持一个可选的第二个参数，即当前项的索引

```html
<ul id="app">
  <li v-for="(item, index) in items">
    {{ index }} - {{ item.message }}
  </li>
</ul>
```

### 在 v-for 里使用对象

```html
<ul id="app">
  <li v-for="value in user">
    {{ value }}
  </li>
</ul>

<script>
var vm = new Vue({
  data: {
    user: {
      name: '习近平',
      age: 18
    }
  }
})
</script>
```

可以提供第二个的参数为 property 名称（即键名）

```html
<div v-for="(value, name) in user">
  {{ name }}: {{ value }}
</div>
```

可以用第三个参数作为索引

```html
<div v-for="(value, name, index) in user">
  {{ index }}. {{ name }}: {{ value }}
</div>
```

### 维护状态

当 Vue 正在更新使用 `v-for` 渲染的元素列表时，它默认使用 **就地更新** 的策略。如果数据项的顺序被改变，Vue 将不会移动 DOM 元素来匹配数据项的顺序，而是就地更新每个元素，并且确保它们在每个索引位置正确渲染

- 这个默认的模式是高效的，但是 **只适用于不依赖子组件状态或临时 DOM 状态（ 例如表单输入值)）的列表渲染输出**

为了给 Vue 一个提示，以便它能跟踪每个节点的身份，从而重用和重新排序现有元素，需要为每项提供一个唯一 的 `key` 属性

```html
<div v-for="item in items" v-bind:key="item.id"></div>
```

- 建议尽可能在使用 `v-for` 时提供 key attribute，除非遍历输出的 DOM 内容非常简单，或者是刻意依赖默认行为以获取性能上的提升
- 不要使用对象或数组之类的非基本类型值作为 `v-for` 的 key，请用字符串或数值类型的值

### 数组更新检测

#### 变异方法

Vue 将被侦听的数组的变异方法进行了包裹，所以它们也将会触发视图更新。可以打开控制台，对数组尝试调用变异方法。这些被包裹过的方法包括

- `push()`：向数组的末尾添加一个或更多元素，并返回新的长度
- `pop()`：删除并返回数组的最后一个元素
- `shift()`：删除并返回数组的第一个元素
- `unshift()`：向数组的开头添加一个或更多元素，并返回新的长度
- `splice()`：删除元素，并向数组添加新元素
- `sort()`：对数组的元素进行排序
- `reverse()`：颠倒数组中元素的顺序

#### 替换数组

非变异方法，例如 `filter()`、`concat()` 和 `slice()` 。它们不会改变原始数组，而 **总是返回一个新数组**。当使用非变异方法时，可以用新数组替换旧数组

- 这并不会导致 Vue 丢弃现有 DOM 并重新渲染整个列表。Vue 为了使得 DOM 元素得到最大范围的重用而实现了一些智能的启发式方法，所以用一个含有相同元素的数组去替换原来的数组是非常高效的操作

#### 注意事项

由于 JavaScript 的限制，Vue 不能检测以下数组的变动

- 利用索引直接设置一个数组项时，例如：`vm.items[indexOfItem] = newValue`，可以使用以下两种方式解决
  - `Vue.set(vm.items, indexOfItem, newValue)`
  - `vm.items.splice(indexOfItem, 1, newValue)`

- 修改数组的长度时，例如：`vm.items.length = newLength`，可以使用以下方式解决
  - `vm.items.splice(newLength)`

### 对象变更检测注意事项

由于 JavaScript 的限制，Vue 不能检测对象属性的添加或删除。对于已经创建的实例，Vue 不允许动态添加根级别的响应式属性。但是，可以使用 `Vue.set(object, propertyName, value)` 方法向嵌套对象添加响应式属性

```javascript
var vm = new Vue({
  data: {
    userProfile: {
      name: '习近平'
    }
  }
})

// 可以使用
Vue.set(vm.userProfile, 'age', 27)
```

为已有对象赋值多个新属性，比如使用 `Object.assign()` 或 `_.extend()`。在这种情况下，应该用两个对象的属性创建一个新的对象

```javascript
Object.assign(vm.userProfile, {
  age: 27,
  favoriteColor: 'Vue Green'
})

// 正确写法
vm.userProfile = Object.assign({}, vm.userProfile, {
  age: 27,
  favoriteColor: 'Vue Green'
})
```

### 显示过滤 / 排序后的结果

若想要显示一个数组经过过滤或排序后的版本，而不实际改变或重置原始数据。在这种情况下，可以创建一个计算属性，来返回过滤或排序后的数组

```html
<li v-for="n in evenNumbers">{{ n }}</li>

<script>
var vm = new Vue({
    data: {
        numbers: [1, 2, 3, 4, 5]
    },
    computed: {
        evenNumbers: function () {
            return this.numbers.filter(function (number) {
                return number % 2 === 0
            })
        }
    }
})
</script>
```

### 在 v-for 里使用值范围

`v-for` 也可以接受整数。在这种情况下，它会把模板重复对应次数

```html
<div>
  <span v-for="n in 10">{{ n }} </span>
</div>
```

### 在 `<template>` 上使用 v-for

> 
>
> 施工中
>
> 

### v-for 与 v-if 一同使用

不推荐在同一元素上使用 `v-if` 和 `v-for`，当它们处于同一节点，`v-for` 的优先级比 `v-if` 更高，这意味着 `v-if` 将分别重复运行于每个 `v-for` 循环中。若只想为部分项渲染节点时，这种优先级的机制会十分有用

```html
<!-- 将只渲染未完成的todo -->
<li v-for="todo in todos" v-if="!todo.isComplete">
  {{ todo }}
</li>
```

若想有条件地跳过循环的执行，那么可以将 `v-if` 置于外层元素（或 `template`）上

```html
<ul v-if="todos.length">
  <li v-for="todo in todos">
    {{ todo }}
  </li>
</ul>
<p v-else>No todos left!</p>
```

### 在组件上使用 v-for

> 
>
> 施工中
>
> 

## 事件处理

### 监听事件

可以用 `v-on` 指令监听 DOM 事件，并在触发时运行一些 JavaScript 代码

```html
<div id="app">
  <button v-on:click="counter += 1">Add 1</button>
  <p>The button above has been clicked {{ counter }} times.</p>
</div>

<script>
var vm = new Vue({
  el: '#app',
  data: {
    counter: 0
  }
})
</script>
```

### 事件处理方法

许多事件处理逻辑会更为复杂，所以直接把 JavaScript 代码写在 `v-on` 指令中是不可行的。因此 `v-on` 还可以接收一个需要调用的方法名称

```html
<div id="app">
  <button v-on:click="greet">Greet</button>
</div>

<script>
var vm = new Vue({
  el: '#app',
  data: {
    name: 'Vue.js'
  },
  methods: {
    greet: function (event) {
      alert('Hey ' + this.name + 'Fuck You!')
      // `event` 是原生 DOM 事件
      if (event) {
        alert(event.target.tagName)
      }
    }
  }
})
</script>
```

### 内联处理器中的方法

除了直接绑定到一个方法，也可以在内联 JavaScript 语句中调用方法

```html
<div id="app">
  <button v-on:click="say('lm')">辣妹</button>
  <button v-on:click="say('fk')">法克</button>
</div>

<script>
var vm = new Vue({
  methods: {
    say: function (message) {
      alert(message)
    }
  }
})
</script>
```

### 事件修饰符

在事件处理程序中调用 `event.preventDefault()` 或 `event.stopPropagation()` 是非常常见的需求。尽管可以在方法中轻松实现这点，但更好的方式是：方法只有纯粹的数据逻辑，而不是去处理 DOM 事件细节。为了解决这个问题，Vue.js 为 `v-on` 提供了**事件修饰符**

- `.stop`
- `.prevent`
- `.capture`
- `.self`
- `.once`
- `.passive`

```html
<!-- 阻止单击事件继续传播 -->
<a v-on:click.stop="doThis"></a>

<!-- 提交事件不再重载页面 -->
<form v-on:submit.prevent="onSubmit"></form>

<!-- 修饰符可以串联 -->
<a v-on:click.stop.prevent="doThat"></a>

<!-- 添加事件监听器时使用事件捕获模式 -->
<!-- 即内部元素触发的事件先在此处理，然后才交由内部元素进行处理 -->
<div v-on:click.capture="doThis">...</div>

<!-- 只当在event.target是当前元素自身时触发处理函数 -->
<!-- 即事件不是从内部元素触发的 -->
<div v-on:click.self="doThat">...</div>

<!-- 点击事件将只会触发一次 -->
<a v-on:click.once="doThis"></a>

<!-- 滚动事件的默认行为（即滚动行为）将会立即触发 -->
<!-- 而不会等待onScroll完成  -->
<!-- 这其中包含event.preventDefault()的情况 -->
<div v-on:scroll.passive="onScroll">...</div>
```

- 使用修饰符时，顺序很重要；**相应的代码会以同样的顺序产生**

### 按键修饰符

在监听键盘事件时，经常需要检查详细的按键。Vue 允许为 `v-on` 在监听键盘事件时添加按键修饰符

```html
<!-- 只有在key是Enter时调用vm.submit() -->
<input v-on:keyup.enter="submit">
```

#### 按键码

keyCode 的事件用法已经被废弃了，并可能不会被最新的浏览器支持。为了在必要的情况下支持旧浏览器，Vue 提供了绝大多数常用的按键码的别名

- `.enter`
- `.tab`
- `.delete`：捕获“删除”和“退格”键
- `.esc`
- `.space`
- `.up`
- `.down`
- `.left`
- `.right`

### 系统修饰符

可以用如下修饰符来实现仅在按下相应按键时才触发鼠标或键盘事件的监听器。

- `.ctrl`
- `.alt`
- `.shift`
- `.meta`：在 Mac 系统键盘上，对应 command 键；在 Windows 系统键盘，对应 Windows 徽标键

#### .exact 修饰符

允许控制由精确的系统修饰符组合触发的事件

```html
<!-- 即使Alt或Shift被一同按下时也会触发 -->
<button @click.ctrl="onClick">A</button>

<!-- 有且只有Ctrl被按下的时候才触发 -->
<button @click.ctrl.exact="onCtrlClick">A</button>

<!-- 没有任何系统修饰符被按下的时候才触发 -->
<button @click.exact="onClick">A</button>
```

#### 鼠标按钮修饰符

`.left`、`.right`、`.middle`：这些修饰符会限制处理函数仅响应特定的鼠标按钮

### 为什么在 HTML 中监听事件

这种事件监听的方式违背了关注点分离这个长期以来的优良传统。但因为所有的 Vue.js 事件处理方法和表达式都严格绑定在当前视图的 ViewModel 上，它不会导致任何维护上的困难。实际上，使用 `v-on` 有几个好处

- 扫一眼 HTML 模板便能轻松定位在 JavaScript 代码里对应的方法
- 因为无须在 JavaScript 里手动绑定事件，ViewModel 代码可以是非常纯粹的逻辑，和 DOM 完全解耦，更易于测试
- 当一个 ViewModel 被销毁时，所有的事件处理器都会自动被删除，无须担心如何清理它们

## 表单输入绑定

### 基础用法

可以用 `v-model` 指令在表单 `input`、`textarea` 及 `select` 元素上创建双向数据绑定。它会根据控件类型自动选取正确的方法来更新元素。尽管有些神奇，但 `v-model` 本质上不过是语法糖。它负责监听用户的输入事件以更新数据，并对一些极端场景进行一些特殊处理

- `v-model` 会忽略所有表单元素的 `value`、`checked`、`selected` 的初始值而总是将 Vue 实例的数据作为数据来源。应该通过 JavaScript 在组件的 data 选项中声明初始值

`v-model` 在内部为不同的输入元素使用不同的属性并抛出不同的事件

- text 和 textarea 元素使用 value 属性和 input 事件
- checkbox 和 radio 使用 checked 属性和 change 事件
- select 字段将 value 作为 prop 并将 change 作为事件

#### 文本

```html
<!-- 文本 -->
<input v-model="message" placeholder="edit me">
<p>Message is: {{ message }}</p>

<!-- 多行文本 -->
<span>Multiline message is:</span>
<p style="white-space: pre-line;">{{ message }}</p>
<br>
<textarea v-model="message" placeholder="add multiple lines"></textarea>
```

#### 复选框

```html
<div id='app'>
  <!-- 单个复选框 -->
  <input type="checkbox" id="checkbox" v-model="checked">
  <label for="checkbox">{{ checked }}</label>
  <br>
  <!-- 多个复选框，绑定到同一个数组 -->
  <input type="checkbox" value="Jack" v-model="checkedNames">
  <label for="jack">Jack</label>
  <input type="checkbox" value="John" v-model="checkedNames">
  <label for="john">John</label>
  <input type="checkbox" value="Mike" v-model="checkedNames">
  <label for="mike">Mike</label>
  <br>
  <span>Checked names: {{ checkedNames }}</span>
</div>

<script>
var vm = new Vue({
  el: '#app',
  data: {
    checked:false,
    checkedNames: []
  }
})
</script>
```

#### 单选按钮

```html
<div id="app">
  <input type="radio" value="One" v-model="picked">
  <label for="one">One</label>
  <br>
  <input type="radio" value="Two" v-model="picked">
  <label for="two">Two</label>
  <br>
  <span>Picked: {{ picked }}</span>
</div>

<script>
var vm = new Vue({
  el: '#app',
  data: {
    picked: ''
  }
})
</script>
```

#### 选择框

```html
<div id="app">
    <!-- 单选 -->
    <select v-model="picked">
        <option disabled value="">请选择</option>
        <option>A</option>
        <option>B</option>
        <option>C</option>
    </select>
    <span>Picked: {{ picked }}</span>
    <br>
    <!-- 多选 -->
    <select v-model="selected" multiple style="width: 50px;">
        <option>A</option>
        <option>B</option>
        <option>C</option>
    </select>
    <br>
    <span>Selected: {{ selected }}</span>
</div>

<script>
var vm = new Vue({
    el: '#app',
    data: {
        picked: '',
        selected: []
    }
})
</script>
```

### 值绑定

对于单选按钮，复选框及选择框的选项，`v-model` 绑定的值通常是静态字符串（对于复选框也可以是布尔值）。但是有时可能需要把值绑定到 Vue 实例的一个动态属性上，这时可以用 `v-bind` 实现，并且这个属性的值可以不是字符串

#### 复选框

```html
<input type="checkbox" v-model="toggle" true-value="yes" false-value="no">
```

这里的 `true-value` 和 `false-value` 并不会影响输入控件的 value，因为浏览器在提交表单时并不会包含未被选中的复选框。如果要确保表单中这两个值中的一个能够被提交，请换用单选按钮

#### 单选按钮

```html
<input type="radio" v-model="pick" v-bind:value="a">
```

#### 选择框的选项

```html
<select v-model="selected">
  <!-- 内联对象字面量 -->
  <option v-bind:value="{ number: 123 }">123</option>
</select>
```

### 修饰符

#### .lazy

在默认情况下，`v-model` 在每次 input 事件触发后将输入框的值与数据进行同步（除了输入法来组合文字时）。可以添加 lazy 修饰符，从而转变为使用 change 事件进行同步

```html
<!-- 在change时而非input时更新 -->
<input v-model.lazy="msg" >
```

#### .number

如果想自动将用户的输入值转为数值类型，可以给 `v-model` 添加 number 修饰符。这通常很有用，因为即使在 `type="number"` 时，HTML 输入元素的值也总会返回字符串。如果这个值无法被 `parseFloat()` 解析，则会返回原始的值

```html
<input v-model.number="age" type="number">
```

#### .trim

如果要自动过滤用户输入的首尾空白字符，可以给 `v-model` 添加 `trim` 修饰符

```html
<input v-model.trim="msg">
```

### 在组件上使用 v-model

>
>
>施工中
>
>

## 组件基础

> 
>
> 施工中
>
> 

## 更多

- [Vue 官方教程](https://cn.vuejs.org/v2/guide/index.html)