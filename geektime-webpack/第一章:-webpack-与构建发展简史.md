---
description: https://github.com/cpselvis/geektime-webpack-course
---

## 1. 为什么需要构建工具

1. 转换 ES6 语法
2. 转换 JSX，Vue 指令， Angular 指令
3. CSS 前缀补全/预处理器
4. 压缩混淆
5. 图片压缩

## 2. 前端构建演变之路

ant + YUI Tool -> grunt -> fis3 / glulp -> rollup / webpack / parcel  

grunt: 打包结果放在本地磁盘，存在 IO 的操作，速度慢  
gulp: 打包结果放在内存中，优化打包速度  

## 3. 初识 webpack

默认配置文件为: webpack.config.js, 可以通过 `webpack --config 指定配置文件`。  

配置文件示例:  
```js
module.exports = {
  entry: './src/index.js',    // 打包入口文件
  output: {                   // 打包的输出
    path: path.join(__dirname, './dist'),
    filename: 'bundle.js'
  },
  mode: 'production',         // 环境
  module: {
    rules: [                  // loader 配置
      {
        test: /\.txt$/,
        use: 'raw-loader'
      }
    ]
  },
  plugins: [                  // 插件配置
    new HtmlWebpackPlugin({
      template: './src/index.html'
    })
  ]
}
```

### 3.1 零配置的 webpack 

webpack 4.0 宣称包含零配置的webpack，即不需要任何的配置，即可以使用 webpack 进行打包。实际上，零配置的 webpack 为:  

```js
module.exports = {
  entry: './src/index.js',
  output: {
    path: path.join(__dirname, './dist'),
    filename: 'bundle.js'
  }
}
```

即默认 entry 和 output  

## 4. 安装 webpack

`npm i webpack webpack-cli -D`  

因为 webpack 4 将 webpack 内核(webpack)与 webpack-cli 分离了。  

可以通过 `node_modules/.bin/webpack -v` 查看 webpack 的版本  
```js
// hello.js
export function hello() {
    console.log('hello')
}

// src/index.js
import { hello } from './hello.js'

hello();
```

以上代码被编译之后，再被引入 html 中，是能正确打印 'hello' 的，所以不需要什么 loader ，webpack 就能解析 es6 module 的语法？  

*或者其实是，浏览器能识别 import 的语法？*  