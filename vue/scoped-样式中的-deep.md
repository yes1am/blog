
[参考资料](https://vue-loader.vuejs.org/zh/guide/scoped-css.html) 

现在有以下两个组件, Home.vue 和 HelloWorld.vue

Home.vue  
```html
<template>
  <div class="home">
    <HelloWorld />
  </div>
</template>

<style scoped>
</style>
```

HelloWorld.vue  
```html
<template>
  <div class="hello">
    hello
    <div class="world">
      world
    </div>
  </div>
</template>
```

由于 Home 组件的样式是 scoped，所以以下代码是无效的：  

Home.vue
```html
<style scoped>
.home .world {
    color: red;  
}
</style>
```
***以上代码并不能让 world 变为红色***  

但是，如果我们使用以下代码:  

Home.vue
```html
<style scoped>
.home .hello {
    color: red;  
}
</style>
```

那么 hello 和 world 都会变红色，hello 作为**子组件的根元素**，能够在父级的 scoped 中被访问到。[参考](https://vue-loader.vuejs.org/zh/guide/scoped-css.html#%E5%AD%90%E7%BB%84%E4%BB%B6%E7%9A%84%E6%A0%B9%E5%85%83%E7%B4%A0)  

而 world 显示为红色，单纯是因为从 hello 中通过 css 继承而来。  


为了实现在 **scoped 的 Home 组件** 中控制 .world 的样式，Vue 中引入了 **深度作用选择器**, 如下:  

```html
// 方式1，兼容 scss
<style scoped>
.home /deep/ .world {
  color: red;
}
</style>


// 方式二
<style scoped>
.home ::v-deep .world {
  color: red;
}
</style>

// 方式三  不能用于 scss 中
<style scoped>
.home >>> .world {
  color: red;
}
</style>
```