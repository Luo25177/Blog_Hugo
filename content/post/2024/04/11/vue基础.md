---
title: "vue基础"
date: 2024-04-11 11:26:02
categories: "前端"
tags: ["vue3"]
summary: "vue3的一些基础知识"
---

[代码](https://github.com/Luo25177/Web.git)

## 简介

vue 是前端优秀框架，是一套用于构建用户界面的渐进式框架

### 特点

- 简单
- 易学
- ...

## 安装使用

### 使用命令行安装

`vue cli` 是一个基于 `vue.js` 进行快速开发的完整系统

```shell
npm install -g @vue/cli
```

测试，在终端输入以下指令测试是否安装成功

```shell
vue --version
```

### 一些功能

使用下列指令打开一个 gui 界面来引导项目创建等一系列操作

```shell
vue ui
```

### 创建项目

在终端使用下列指令创建一个项目

```shell
vue create projectname
```

输入之后需要使用上下按键来选择默认项目模板

可以选择手动选择，然后按照如下选择

![config](./config.png)

### 文件目录说明

| 目录/文件    | 说明                                                                               |
| ------------ | ---------------------------------------------------------------------------------- |
| build        | 项目构建(webpack)相关代码                                                          |
| config       | 配置目录，包括端口号等。初学可以使用默认的。                                       |
| node_modules | npm 加载的项目依赖模块                                                             |
| src          | 这里是我们要开发的目录，基本上要做的事情都在这个目录里。里面包含了几个目录及文件： |
| assets       | 放置一些图片，如 logo 等，还可以放全局使用的 css 文件                              |
| components   | 目录里面放了一个组件文件，可以不用。                                               |
| App.vue      | 项目入口文件，我们也可以直接将组件写这里，而不使用 components 目录。               |
| main.js      | 项目的核心文件。                                                                   |
| index.css    | 样式文件。                                                                         |
| static       | 静态资源目录，如图片、字体等。                                                     |
| public       | 公共资源目录。                                                                     |
| test         | 初始测试目录，可删除                                                               |
| .xxxx文件    | 这些是一些配置文件，包括语法配置，git配置等。                                      |
| index.html   | 首页入口文件，你可以添加一些 meta 信息或统计代码啥的。                             |
| package.json | 项目配置文件。                                                                     |
| README.md    | 项目的说明文档，markdown 格式                                                      |
| dist         | 使用 npm run build 命令打包后会生成该目录。                                        |

## 基础语法

### 文本

数据绑定常见的形式就是使用 `Mustache` 双大括号的语法文本插值

```VUE
<span>Message: {{msg}}</span>
```

一般配合 `js` 中的 `data()` 设置数据

```JavaScript
export default {
  name: '1',
  data() {
    return {
      msg: "msg"
    }
  }
}
```

### 原始HTML

双大括号会将数据解释为普通文本，而非 `html` 代码，为了输出 `html` 就需要使用 `v-html` 指令

### attribute属性

`Mustache` 语法不能再 `html` 属性中使用，但是可以使用 `v-bind` 指令。 `class` 与 `style` 是 `html` 元素的属性，用于设置元素的样式，我们可以用 `v-bind` 来设置样式属性。有两种写法，如下

```vue
<div v-bind:id="dynamicId">v-bind:</div>
<div :id="dynamicId">v-bind:</div>
```

**内联样式**

```vue
<div id="app">
  <div :style="{ color: activeColor, fontSize: fontSize + 'px' }">1</div>
</div>
<!--其中 activeColor:"red", fontSize: 20-->
```
上述会被渲染为

```html
<div :style="{ color: red, fontSize: 20px}">1</div>
```

内联样式还可指定多重值，提供一个包含多个值的数组，常用于提供多个带前缀的值，这样写只会渲染数组中最后一个被浏览器支持的值

```vue
<div :style="{ display: ['-webkit-box', '-ms-flexbox', 'flex'] }"></div>
```

**组件上使用class属性**

当你在带有单个根元素的自定义组件上使用 class 属性时，这些 class 将被添加到该元素中。此元素上的现有 class 将不会被覆盖

```vue
<template>
  <newComponentName class="classC classD"></newComponentName>
</template>

<script>
var app = Vue.createApp({})
app.component('newComponentName', {
  template: '<h1 class="classA classB">这是一个新组件标题</h1>'
})
</script>
```

最终渲染结果为

```html
<h1 class="classA classB classC classD">这是一个新组件标题</h1>
```

对于带数据绑定的 `class` 也可使用

```vue
<newComponentName :class="{ active: isActive }"></newComponentName>
```

当 `isActive` 为 `true` 时，最终渲染效果为

```html
<h1 class="classA classB active">这是一个新组件标题</h1>
```

如果新的组件有多个根元素，需要定义哪些部分将接收这个类。可以使用 `$attrs` 组件属性执行此操作，可以用于传递组件属性和事件。下列的操作实际上就是继承原来的类属性

```vue
<div id="app">
  <newComponentName class="classA"></newComponentName>
</div>
 
<script>
const app = Vue.createApp({})
app.component('newComponentName', {
  template: `
    <p :class="$attrs.class">这是一个段落</p>
    <span>这是一个子组件</span>
  `
})
app.mount('#app')
</script>
```

最终渲染结果为

```html
<div id="app">
  <p :class="classA">这是一个段落</p>
  <span>这是一个子组件</span>
</div>
```

### 使用Js表达式

`vue.js` 提供了完全的 js 表达式支持，使用双括号来表示使其被解析为 js 表达式。但是，双括号中只能包含单个表达式

```vue
// 有效
{{number + 1}}
{{ok? 'y' : 'n'}}

// 无效
{{var a = 1}} //语句
{{if(ok) {return msg;}}} // 流程控制语句也不会生效
```

### 条件渲染

- `v-if` 用于条件性的渲染一块内容，这块内容只会在指令的表达式返回 `true` 时渲染
  - 真正的条件渲染，会保证在切换过程中，条件块内的事件监听器和子组件适当的被销毁和重建
  - 也是惰性的，如果在初始渲染时条件为假，那什么也不做，直到变为真才会做渲染
- `v-else` 用于作为 `v-if` 的 `else` 块
- `v-show` 用于条件性展示元素的另一个选项
  - 不管初始条件是什么都会渲染元素，并且只是简单的基于 `css` 进行切换
  - 一般来说 `v-if` 有更高的切换开销，而 `v-show` 有更高的初始渲染开销，所以如果需要非常频繁的切换则使用 `v-show` ，如果在运行中很少改变就使用 `v-if`

### 列表渲染

- `v-for` 把一个数组映射为一组元素，基于一个数组来渲染一个列表，指令需要使用 `item in items` 语法来实现
- 当 `vue` 正在更新使用 `v-for` 渲染的元素列表时，它默认使用“就地更新”的策略，如果数据项的顺序被改变，vue 将不会移动 dom 元素来匹配数据项的顺序，而是就地更新每个元素，并且保证它们在每个索引位置正确渲染。为了给 vue 一个提示，以便它能够跟踪每个节点，从而重新排列现有元素，需要为每一项提供一个唯一的 `key` attribute

### 事件处理

- 监听事件，可以使用 `v-on` 指令（可缩写为 `@` 符号）来监听 dom 事件，并且在触发事件时执行一些 `js` 指令
- 事件处理方法，很多事件处理逻辑会比较复杂，所以可以写为一个调用的方法名称
- 内联处理器中的方法，也可以称作事件传递函数

### 表单输入绑定

- `v-model` 指令在表单 `<input>` ， `<textarea>` 以及 `<select>` 元素上创建双向数据绑定，它会根据空间类型自动选取正确的方法来更新元素。本质上不过是语法糖，负责监听用户输入事件来更新数据，并且完成一些特殊处理
- `.lazy` 修饰符， `v-model` 事件在每次 `input` 事件触发之后就进行数据同步，可以添加该修饰符，从而转为 `change` 事件之后进行同步
- `.trim` 修饰符，可以自动过滤用户输入的首尾空白字符

### 组件基础

- 单文件组件，单文件组件是一种特殊的文件格式，允许将 `vue` 组件中的模板，逻辑与样式封装在单个文件中
- 加载组件步骤
  - 引入组件 `import mycomponent from './components/mycomponent.vue'; './components/mycomponent'`
  - 挂载组件 `components: {mycomponent}`
  - 显示组件 `<mycomponent/>`
- 组件组织，通常一个应用会以一棵嵌套的组件树的形式来组织
  ![conponent_tree](./component_tree.png)

### props组件交互

组件与组件之间是需要存在交互的，否则完全没关系，组件的意义就很小了，而 `props` 是可以在组件上注册的一些自定义 `attribute` ，传递的方向就是父组件传递到子组件

`props` 传递的参数是没有类型限制的，但是传入数据为数组或者对象类型时，默认值时需要返回工厂模式，也就是定义一个返回一个空数组的函数

**使用方法**

1. 在调用组件的文件中声明需要传递的变量，写法与组件文件中一致

  ```js
  data() {
    return {
      title: "一个标题",
      age : 10,
    }
  }
  ```

  需要在加载组件时传入参数

  ```vue
  <props_test :title="title" :age="age"/>
  ```

2. 在组件中需要使用 `props` 声明变量

  ```vue
  <script>
  export default {
    name: "props_test",
    props: {
      title: {
        type:String,
        default:""
      },
      age: {
        type: Number,
        default:0
      }
    }
  }
  </script>
  ```

### 自定义事件组件交互

自定义事件可以在组件中反向传递数据， `prop` 可以将数据从父组件传递到子数组，对于反向操作，可以使用自定义事件实现

- 使用 `this.$emit()` 函数，其中第一个参数是字符串，理论上是随便的，但是需要与弗组件中对应，第二个参数就是传递的数据

**使用**

1. 首先在子组件中做定义一个按钮或者触发这个事件的元素

  ```HTMl
  <button @click="sendHandle">sendData</button>
  ```

  并且定义函数 `sendHandle`

  ```JavaScript
  sendHandle() {
    this.$emit("onEvent", "data");
  }
  ```

2. 在父组件中加载该组件时，需要做如下的定义

  ```HTML
  <props_test @onEvent="getDataHandle"/>
  ```

  需要注意，这里的 `onEvent` 与第一步中的 `$emit()` 的第一个参数是对应的，需要一致。然后自定义收到消息的回调函数，这里的 `data` 就是上述 `$emit()` 函数的第二个参数

  ```JavaScript
  getDataHandle(data) {
    console.log(data);
  }
  ```

### 组件的生命周期

每个组件在被创建时都要经过一系列初始化的过程，例如设置数据监听，编译模板，实例挂载到 dom 上等，这个过程会运行一些叫做生命周期钩子的函数，给了用户在不同阶段添加代码的功能

可以直接定义下面的函数，来实现在不同的阶段添加用户代码的功能

- 创建时 `beforeCreate` `created`
- 渲染时 `beforeMount` `mounted`
- 更新时 `beforeUpdate` `updated`
- 卸载时 `beforeUnmount` `unmounted`
