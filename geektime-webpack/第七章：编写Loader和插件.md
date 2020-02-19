---
description: https://github.com/cpselvis/geektime-webpack-course
---

## 1. loader

loader 只是一个 **导出函数** 的 JavaScript 模块  

该函数将输入经过转换，得到输出。

```js
module.exports = function(source) {
    return source;
};
```

`loader-runner`：loader-runner 允许你在 **不安装 webpack 的情况下运行 loaders**。  


**loader 的参数获取:**  
[loader-utils 中的 getOptions 方法](https://github.com/webpack/loader-utils#getoptions)  

**loader 异常处理:**  

两种方式：  
1. loader 内直接通过 throw new Error() 抛出异常
2. 通过 this.callback 传递错误

```js
this.callback(
 err: Error | null,
 content: string | Buffer,
 sourceMap?: SourceMap,
 meta?: any
);
```

**loader 的异步处理：**  

通过 this.async 来返回一个异步函数，该异步函数的*第一个参数是 Error，第二个参数是处理的结果*
```js
module.exports = function(input) {
    const callback = this.async();
    setTimeout(() => {
      // 在异步的 loader 中通过回调函数，将结果返回
      callback(null, input + input);
    })
```

**在 loader 中使用缓存：**  

webpack 中默认已经开启了 loader 缓存，可以在 loader 内使用 `this.cacheable(false)` 关掉缓存.

缓存条件: 
1. loader 的结果在相同的输入下有确定的输出
2. 有依赖的 loader 无法使用缓存


**loader 如何进行文件输出?**  

通过 this.emitFile 写出文件  

```js
const loaderUtils = require("loader-utils");
module.exports = function(content) {
    // 得到要输出的文件的路径
    const url = loaderUtils.interpolateName(this, "[hash].[ext]", {
content, });
    // 输出文件
    this.emitFile(url, content);
    const path = `__webpack_public_path__ + ${JSON.stringify(url)};`;
    return `export default ${path}`;
};
```

## 2. 插件

插件没有像 `loader-runner` 一样的运行环境，只能在 webpack 里运行。  

**插件的基本结构**  

```js
// 基本结构, 就是一个类，拥有一个 apply 方法, 该方法的参数接受一个 compiler 对象
// 在插件内部，监听 webpack 内部的 hook，然后做出对应的处理
class MyPlugin {
    apply(compiler) {
        compiler.hooks.done.tap(' My Plugin', () => {
            console.log('Hello World!');
        });
    }
}

module.exports = MyPlugin;

// webpack.config.js
module.exports = {
    plugins: [ new MyPlugin() ]
}
```

**插件中如何获取传递的参数?**  

通过插件的构造函数进行获取  
```js
module.exports = class MyPlugin {
    constructor(options) {
        // 获取参数
        this.options = options;
    } 
    apply() {
        console.log("apply", this.options);
    }
};
```

**插件的错误处理**  

1. 参数校验阶段可以直接 throw 的方式抛出  

`throw new Error(“ Error Message”)`  

2. 通过 compilation 对象的 warnings 和 errors 接收

```js
// 最终错误和 warning 会被输出到控制台
compilation.warnings.push("warning"); compilation.errors.push("error");
```

**通过 Compilation 进行文件输出:**  

```js
// 需要输出文件，需要 RawSource 库
const { RawSource } = require("webpack-sources");

module.exports = class DemoPlugin {
    constructor(options) {
        this.options = options;
    }
    apply(compiler) {
        const { name } = this.options;
        compiler.plugin("emit", (compilation, cb) => {
            // 将某个文件，设置为compilation.assets[fileName]
            // 即可进行文件的输出
            compilation.assets[name] = new RawSource("demo");
            cb();
        });
    }
};
```

另外，有一个别的插件，如 `html-webpack-plugin`，也会暴露一些 hooks，这时候可以让插件，监听 `html-webpack-plugin` 插件的一些 hooks 进行一些操作。  