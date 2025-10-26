---
title: vue-base
categories:
  - others
date: 2025-4-26 14:33:48
tags:
---

## vue简介

<!-- more -->

### 示例

```js
import { createApp, ref } from 'vue'

createApp({
  setup() {
    return {
      count: ref(0)
    }
  }
}).mount('#app')
```

```html
<div id="app">
  <button @click="count++">
    Count is: {{ count }}
  </button>
</div>
```

### 单文件组件

Vue 的单文件组件会将一个组件的逻辑 (JavaScript)，模板 (HTML) 和样式 (CSS) 封装在同一个文件里。

示例可以改写成一个vue文件：

```vue
<script setup>
import { ref } from 'vue'
const count = ref(0)
</script>

<template>
  <button @click="count++">Count is: {{ count }}</button>
</template>

<style scoped>
button {
  font-weight: bold;
}
</style>
```

### API 风格

选项式API 和 组合式API 

选项式：

```vue
<script>
export default {
  // data() 返回的属性将会成为响应式的状态
  // 并且暴露在 `this` 上
  data() {
    return {
      count: 0
    }
  },

  // methods 是一些用来更改状态与触发更新的函数
  // 它们可以在模板中作为事件处理器绑定
  methods: {
    increment() {
      this.count++
    }
  },

  // 生命周期钩子会在组件生命周期的各个不同阶段被调用
  // 例如这个函数就会在组件挂载完成后被调用
  mounted() {
    console.log(`The initial count is ${this.count}.`)
  }
}
</script>

<template>
  <button @click="increment">Count is: {{ count }}</button>
</template>
```

组合式：

标志：`<script setup>`

```vue
<script setup>
import { ref, onMounted } from 'vue'

// 响应式状态
const count = ref(0)

// 用来修改状态、触发更新的函数
function increment() {
  count.value++
}

// 生命周期钩子
onMounted(() => {
  console.log(`The initial count is ${count.value}.`)
})
</script>

<template>
  <button @click="increment">Count is: {{ count }}</button>
</template>
```

（选项式 API 是在组合式 API 的基础上实现的）

## 创建vue应用

每个 Vue 应用都是通过 createApp 函数创建一个新的 **应用实例**。

### 根组件

`createApp` 的对象实际上是一个组件。每个应用都需要一个“根组件”，其他组件将作为其子组件。

```js
import { createApp } from 'vue'
// 从一个单文件组件中导入根组件
import App from './App.vue'

const app = createApp(App)
```

### 挂载应用

应用实例必须在调用了 `.mount()` 方法后才会渲染出来。该方法接收一个“容器”参数，可以是一个实际的 DOM 元素或是一个 CSS 选择器字符串。

`.mount()` 方法应该始终在整个**应用配置**和**资源注册** **完成后被调用**。同时请注意，不同于其他资源注册方法，它的返回值是根组件实例而非应用实例。

```js
app.mount('#app')
```

### 应用配置

应用实例会暴露一个 `.config` 对象允许我们配置一些应用级的选项，例如定义一个应用级的错误处理器，用来捕获所有子组件上的错误。

```js
app.config.errorHandler = (err) => {  /* 处理错误 */ }
```

以上代码注册了一个组件，这使得 `TodoDeleteButton` 在应用的任何地方都是可用的。

```js
app.component('TodoDeleteButton', TodoDeleteButton)
```

