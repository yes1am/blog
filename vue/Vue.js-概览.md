# 基础

## 1. 关键语法

```js
v-bind：**指令**，在 dom 上应用特殊的响应式行为，前缀 v- 表示它们是 vue 提供的特殊特性  

v-if: 控制元素是否显示  

v-for: 循环

v-on: 添加事件绑定  

v-model: 表单输入与应用状态的双向绑定  

Vue.component(componentName, componentOptions)   // 注册组件  
```

## 2. Vue 实例

```js
带有前缀 $ 的实例属性与方法： $data, $el, $watch
```

生命周期钩子  

1. created: 实例被创建之后
2. mounted, uodated, destroyed 等，生命周期中的 this 指向 Vue 实例

> 注意: 不要在钩子函数，选项属性或者回调上使用箭头函数，比如 `created: () => console.log(this.a)` 或者 `vm.$watch('a', newValue => this.myMethod())`, 因为箭头函数没有 this，this 就不等于 Vue 实例了  


生命周期示例图

![](https://cn.vuejs.org/images/lifecycle.png)  

## 3. 模板语法

### 3.1 插值

**文本**  

{{}} 双大括号语法用于绑定实例上 data 对象的属性，使用 v-once 执行一次性插值(绑定一次)。  


**原始HTML**  
{{}} 会将数据解释为普通文本，而非 HTML，为了输出真正的 HTML， 需要使用 v-html 

**HTML特性**  

{{}} 语法不能用在 HTML 特性上，这种情况需要使用 v-bind  指令  

** JavaScript 表达式**  

模板中可以使用完全的 JavaScript 表达式支持，但是只支持 ***单个表达式***  

而且模板表达式只能访问 ***全局变量的一个白名单***，不能够在其中试图访问用户定义的全局变量。  

### 3.2 指令

指令是带有 `v-` 前缀的特殊属性，指令的预期值是 ***单个 JavaScript表达式***  

**参数**  

一些指令能够接受 ***参数***，在指令后面以冒号显示。

如以下的 href 即为参数，表示将元素的 hred 特性与表达式 url 的值绑定。  

```html
<a v-bind:href="url">...</a>
```

以下的 click 即为参数，用于监听 DOM 事件
```html
<a v-on:click="doSomething">...</a>
```

**动态参数**  (存在一些语法约束)

即可以使用 JavaScript 表达式作为指令的参数  

```html
<a v-bind:[attributeName]="url"> ... </a>
```

当  Vue 实例的 data 属性中，attributeName 为 href, 则以上代码等同于:  

```html
<a v-bind:href="url"> ... </a>
```

**修饰符**  

修饰符是以 . 指明的特殊后缀，用于支持一个指令以特殊的方式绑定。 如 .prevent 告诉 v-on 指令对于触发的事件调用 event.preventDefault():    

```html
<form v-on:submit.prevent="onSubmit">...</form>
```

### 3.3 缩写

**v-bind**  

```html
<!-- 完整语法 -->
<a v-bind:href="url">...</a>

<!-- 缩写 -->
<a :href="url">...</a>
```

**v-on**  

```html
<!-- 完整语法 -->
<a v-on:click="doSomething">...</a>

<!-- 缩写 -->
<a @click="doSomething">...</a>
```

## 4. 计算属性与侦听器

### 4.1 计算属性

为了防止在模板中使用过多的逻辑，因此产生了***计算属性***  

***Q&A: 使用计算属性还是方法？***  

**计算属性基于响应式的依赖进行缓存**，如果依赖没变化，那么访问**计算属性**会立即返回之前的结果。而使用**方法**的化，无论响应式依赖有没有变化，只要重新渲染都是执行**方法**。  


**watch 表示侦听属器**  

略

**计算属性的setter**  

计算属性默认只有 getter，不过在需要时可以提供一个 ***setter***  

### 4.2 侦听器

使用 watch 的语法，可以在数据变化会导致 ***异步请求 Ajax 或开销较大的操作时*** 使用。


## 5. Class 与 Style 绑定

### 5.1 绑定 Class

通过 v-bind:class 实现，同时 v-bind:class 可以与普通的 class 属性共存。  

```html
<div
  class="static"
  v-bind:class="{ active: isActive, 'text-danger': hasError }"
></div>
```

**对象语法**  

```html
<div v-bind:class="{ active: isActive }"></div>
```

同时绑定的数据对象可以不必内联在模板里，可以来自于 **data 属性**

```js
<div v-bind:class="classObject"></div>

data: {
  classObject: {
    active: true,
    'text-danger': false
  }
}
```  

或者来自**计算属性**  

```js
data: {
  isActive: true,
  error: null
},
computed: {
  classObject: function () {
    return {
      active: this.isActive && !this.error,
      'text-danger': this.error && this.error.type === 'fatal'
    }
  }
}
```

**数组语法**  

```html
<div v-bind:class="[activeClass, errorClass]"></div>
```

### 5.2 绑定 Style

注意绑定 style 里的 css 属性需要使用驼峰式的写法，或者短横线用引号包裹

**驼峰式 fontSize**
```html
<div v-bind:style="{ color: activeColor, fontSize: fontSize + 'px' }"></div>
```

**短横线用引号包裹**  

```html
<div id="app">
  <div v-bind:style="{'font-size': '20px'}">你好</div>
</div>
```

## 6. 条件渲染

### 6.1 v-if

vi-if, v-else, v-else-if 等用于控制是否渲染某个单一元素.  

当需要控制是否渲染一组元素时可以使用 <template> 进行包裹  

```html
<template v-if="ok">
  <h1>Title</h1>
  <p>Paragraph 1</p>
  <p>Paragraph 2</p>
</template>
```

**使用 key 管理可复用的元素**  

### 6.2 v-show

v-show 只是切换元素的 css, display 的值。 v-if 是直接影响 DOM 的渲染。  


### 6.3 v-if vs v-show

略

### 6.4 v-if 不要与 v-for 一起使用

## 7. 列表渲染 v-for

v-for 可用于***遍历数组与对象***，

### 7.1 数组更新检测

**变异方法**  

Vue 对**会改变原数组**的数组方法进行了包裹，确保数组的变化会触发视图更新。变异方法包括如下:  

```
push()
pop()
shift()
unshift()
splice()
sort()
reverse()
```

**非变异方法**  

即这些方法不会改变原数组，而是产生了新的数组，如 filter(), concat(), slice()，当使用非变异方法时，可以用新数组替换就数组：  

```js
example1.items = example1.items.filter(function (item) {
  return item.message.match(/Foo/)
})
```

***注意事项***  

由于 JavaScript 的限制，Vue **不能**检测以下数组的变化：  

1. 通过索引直接设置数组元素时: `vm.items[index] = newValue`;  (**可以通过 Vue.$set() 方法, 通过 splice 方法解决**)
2. 修改数组的长度是: vm.items.length = newLength;(**可以通过 splice(newLength) **)

同样由于JavaScript 的限制，Vue 不能检测对象属性的添加和删除，这意味着添加新的属性并不能有响应式的效果。(因此可以使用 **Vue.set** 来设置新的响应式属性)。  

## 8. 事件处理

### 8.1 事件修饰符

```js
.stop: 阻止单击事件继续传播
.prevent
.capture: 添加事件监听器时使用事件捕获模式
.self:  只当在 event.target 是当前元素自身时触发处理函数
.once
.passive: (移动端滚动相关)
```

### 8.2 按键修饰符

```html
// 只有 enter 键的 keyup 事件才调用 vm.submit() 
<input v-on:keyup.enter="submit">
```
## 9. 表单输入绑定

https://cn.vuejs.org/v2/guide/forms.html  
略


## 10. 组件基础

组件分为**根实例组件**与**普通组件**，

根实例组件需要 el 属性指定挂载的 DOM 节点，普通组件没有 el 属性。 其他的属性如 data, computed, watch，以及生命周期钩子等都是一样的。  

区别于根实例组件，***一个普通组件的 data 选项必须是一个函数***，这样各个组件实例之间才能互相独立。  

### 10.1 组件注册类型

全局注册与局部注册  

### 10.2 父子通信

*** $emit 与 on ***

```html
不传参
// 父组件
<blog-post
  ...
  v-on:enlarge-text="postFontSize += 0.1"
></blog-post>

// 子组件 blog-post
<button v-on:click="$emit('enlarge-text')">
  Enlarge text
</button>


传参
// 父组件，使用 $event 访问参数值
<blog-post
  ...
  v-on:enlarge-text="postFontSize += $event"
></blog-post>

// 子组件
<button v-on:click="$emit('enlarge-text', 0.1)">
  Enlarge text
</button>
```

### 10.3 slot

### 10.4 动态组件

特殊的 `is`  

# 深入了解 组件

## 1. 组件注册

两张组件取名方式:  

kebab-case 短横线连线: `my-component-name` (推荐方式)  
PascalCase 大驼峰式: `MyComponentName`  

### 1.1 [全局注册](https://cn.vuejs.org/v2/guide/components-registration.html#%E5%B1%80%E9%83%A8%E6%B3%A8%E5%86%8C)  



```js
Vue.component('component-a', {
    
})
```

### 1.2 局部注册

```js
// 通过普通的 JavaScript 对象来定义组件
var ComponentA = { /* ... */ }
var ComponentB = { /* ... */ }
var ComponentC = { /* ... */ }

// 在根组件中进行注册
new Vue({
  el: '#app',
  components: {
    'component-a': ComponentA,
    'component-b': ComponentB
  }
})
```
即通过 组件(无论是根组件或者是普通组件) 的 components 属性进行注册。  

## 2. Prop

给 prop 传递静态的值  

```html
<blog-post title="My journey with Vue"></blog-post>
```

给 prop 动态赋值  

```html
<blog-post v-bind:title="post.title"></blog-post>
```

### 2.1 [Props 类型验证](https://cn.vuejs.org/v2/guide/components-props.html#Prop-%E9%AA%8C%E8%AF%81)

略  

## 3. 自定义事件

略

## 4. 插槽 slot

略