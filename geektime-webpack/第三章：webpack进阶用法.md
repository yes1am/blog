---
description: https://github.com/cpselvis/geektime-webpack-course
---

## 1. 自动清理构建目录产物

在之前的示例中，每次构建之后，会在 `dist` 目录下新增文件，长期这样之后，会积攒许多的无用文件。  

1. 可以通过手动删除构建目录
2. 通过 npm scripts 删除构建目录(不够优雅)
    1. rm -rf ./dist && webpack
    2. rimraf ./dist && webpack
3. 使用 clean-webpack-plugin  

该插件**默认会删除 output 指定的输出目录**  

安装依赖: `npm i clean-webpack-plugin -D`  

配置 webpack.config.js:  
```js
const { CleanWebpackPlugin } = require('clean-webpack-plugin')

..
module.exports = {
    plugins: [
        new CleanWebpackPlugin()
    ]
}
```

## 2. 自动补全 CSS3 前缀

浏览器渲染引擎(内核) 划分:  
[参考资料](https://zhuanlan.zhihu.com/p/30236218)

| 浏览器 | 内核 | 前缀 |
| --- | --- | --- |
| IE | Trident | -ms |
| Firefox | Geko | -moz |
| Chrome | webkit | -webkit |
| Opera | Presto | -o |

使用 autoprefixer 插件，可以给某些属性(如 display: flex) 添加相应的浏览器前缀，使得浏览器支持某些属性。  

安装依赖: `npm i postcss-loader autoprefixer -D`  

配置 webpack.config.js  
```js
module.exports = {
    module: {
        rules:[
            {
            test: /\.less$/,
            use: [
              MiniCssExtractPlugin.loader,
              'css-loader',
              'less-loader',
              {
                loader: 'postcss-loader',
                options: {
                  plugins: () => [
                    require('autoprefixer')()
                    // 以下这种传入浏览器的方式被废弃，改用在 package.json 中定义
                    // require('autoprefixer')({ browsers: ['last 2 version'] })
                  ]
                }
              }
            ]
          },
      ]
    }
}

// 配置 package.json
"browserslist": [
    "last 2 version",
    ">1%",
    "ios 7"
]
```

## 3. 移动端 CSS px 转 rem

安装依赖: `npm i px2rem-loader -D`

配置 webpack.config.js  

```js
{
  test: /\.less$/,
  use: [
    MiniCssExtractPlugin.loader,
    'css-loader',
    'less-loader',
    {
      loader: 'postcss-loader',
      options: {
        plugins: () => [
          require('autoprefixer')()
        ]
      }
    },
    {
      loader: 'px2rem-loader',
      options: {
        remUnit: 75,   // 1rem = 75px
        remPrecision: 8  // 像素转换为rem之后，保留的小数点位数
      }
    }
  ]
}
```

此时，就能把所有引入的 css 中的像素值，转换为 rem 的值

*注意，在测试的过程中，发现好像不能解析 less 语法，会报代码错误，有机会可以再次测试*  

此刻在 html 页面中引入 [淘宝 flexible库](https://github.com/amfe/lib-flexible) 即可实现，移动端自适应问题。因为 屏幕大小变化时， flexible 库使得 rem 的值发生变化，而我们的样式经过 px2rem-loader 之后，所有的尺寸单位都是 rem 了，自然会做自适应。  

## 4. 静态资源内联

对于有一些资源，如埋点js，flexible js 等，我们希望这些 js 首先被加载，而不是夹杂在 bundle.js 中，从而更快的得到加载。  

另外，如果资源内联 html 模板中，那么会跟随 html 请求一并回来，减少 http 请求数量.  

**html片段(如 meta 信息) 和 js 内联**: 使用 `raw-loader`  

**raw-loader 实质上就是将文件内容读取出来，填充到 html 模板对应的位置。**  

**css 内联** 的两种方式:   

将 css 的样式内联到 html 中:  

1. style-loader  

```js
rules: [
    {
        test: /\.less$/,
        use: [
            loader: 'style-loader',
            options: {
                insertAt: 'top',  // 样式插入到 head
                singleton: true  // 将所有的 style 标签合成一个标签
            },
            "css-loader",
            "less-loader"
        ]
    }
]
```

2. 使用 html-inline-css-webpack-plugin (推荐)

将打包出来的 css 插入到 html 对应的位置。

<br/>

安装依赖: `npm i raw-loader@0.5.1 -D`  

在 html 模板中使用:  
```html
<head>
  // 以下代码 是 es6 中 ${} 的语法，读取文件内容填充到这个位置
  ${ require('raw-loader!./meta.html') }
  <script>
    // 针对 js 文件，可能内部使用了 es6 的语法，因此可以再用 babel-loader
    ${ require('raw-loader!babel-loader!../node_modules/xxx.js') }
  </script>
  <title>Docment</title>
</head>
```

## 5. 通用的多页面应用打包方案

每一次页面跳转的时候，后台服务器都会返回一个新的 html 文档，这种叫做多页面应用。  

思路就是每个页面对应一个 entry，打包出一个 js 文件。同时有一个 html-webpack-plugin 生成对应的页面。  

示例：
```js
entry: {
    index: './src/index/index.js',
    search: './src/search/index.js',
}
plugins: [
    new HtmlWebpackPlugin({
        template: path.join(__dirname,'src/index/index.html')
    }),
    new HtmlWebpackPlugin({
        template: path.join(__dirname,'src/search/index.html')
    }),
]
```

于是我们可以约定如下的数据结构:  

**每个目录对应一个页面，目录中的 index.js 对应 entry, index.html 作为页面模板**。
```
-src
----search              // 目录
------index.js
------index.html
----index               // 目录
------index.js
------index.html
```

利用 `glob.sync` 读取目录:  `entry: glob.sync(path.join(__dirname, './src/*/index.js'))`

安装依赖: `npm i glob -D`  

```js
// 设置 MPA, 产生 entry 和 htmlWebpackPlugin 的配置
const setMPA = () => {
  const entry = {};
  const htmlWebpackPlugins = [];

  const entryFiles = glob.sync(path.join(__dirname,'./src/*/index.js'))
  entryFiles.forEach(entryFile => {
    const match = entryFile.match(/src\/(.*)\/index\.js$/)
    const pageName = match && match[1];
    
    entry[pageName] = entryFile;
    htmlWebpackPlugins.push(
      new HtmlWebpackPlugin({
        template: path.join(__dirname, `src/${pageName}/index.html`),
        filename: `${pageName}.html`,
        chunks: [pageName],
        inject: true,
        minify: {
          html5: true,
          collapseWhitespace: true,
          preserveLineBreaks: false,
          minifyCSS: true,
          minifyJS: true,
          removeComments: true
        }
      })
    )
  })

  return {
    entry,
    htmlWebpackPlugins
  }
}

const { entry, htmlWebpackPlugins } = setMPA()

// webpack.config.js

module.exports = {
    entry: entry,
    plugins: [
        // 其余 plugins
    ].concat(htmlWebpackPlugins)
}
```

## 6. source map

[参考资料](https://www.ruanyifeng.com/blog/2013/01/javascript_source_map.html)  

作用: 将压缩后的代码，通过 source map 定位到源代码。  

开发环境开启，生产环境关闭，线上排查问题的时候可以将 source map 上传到错误监控系统。  

webpack 中的 source map 类型: 

| 类型 | 说明 |
| --- | --- |
| eval | 使用 eval 包括模块代码 |
| source-map | 产生 .map 文件 |
| cheap | 不包含列信息 |
| inline | 将 .map 作为 DataUrl 嵌入，不单独生成 .map 文件 |
| module | 包含 loader 的 source map |

设置 webpack.config.js:  

```js
module.exports = {
    devtool: 'source-map'
}
```

```js
// webpack.config.js
module.exports = {
    mode: 'development'
}
// app.js
function App() {
    debugger;
    return <div>app</div>
}
此时，development 模式下，不开启 sourcemap，那么在 chrome 调试是以下的代码：  
function App() {
  debugger;
  return react__WEBPACK_IMPORTED_MODULE_0___default.a.createElement("div", {
    className: "red"
  }, "hello world 3", react__WEBPACK_IMPORTED_MODULE_0___default.a.createElement("img", {
    src: _logo_png__WEBPACK_IMPORTED_MODULE_2__["default"],
    alt: ""
  }));
}

而开启 source map 之后，会显示以下的代码： 
function App() {
  debugger;
  return <div className='red'>
    hello world 3
    <img src={logo} alt=""/>
  </div> 
}
```
以上的测试在 mode 为 `production` 的时候不起效果，应该是 debugger 被移除了吧。  

经过测试发现:  

```js
import React from 'react'
import ReactDOM from 'react-dom';
import logo from './logo.png'
import './index.less'

function App() {
  debugger;
  console.log(1)
  return <div className='red'>
    hello world 3
    <img src={logo} alt=""/>
  </div> 
}

ReactDOM.render(
  <App />,
  document.querySelector('#root')
)
```

以上代码，在 development 情况下，打包出来的 index.js 为 900多k,在 production 的情况下，打包出来为 128k.  

## 7. 提取页面公共资源

思路: 将 react 和 react-dom 等基础包通过 cdn 引入，而不打入 bundle 中。  

### 7.1 使用 html-webpack-extrnals-plugin

安装依赖: `npm - html-webpack-extrnals-plugin`

设置 webpack.config.js  

```js
plugins: [
    new HtmlWebpackExternalsPlugin({
        externals: [
        {
          // 这样当我们在代码中引入 react 的时候，就不会将 react 打包进 bundle
          module: 'react',
          // 该 entry 看着没啥用，因为最终还是要在 html 模板中添加 script 标签，但是这一项还是不能删除
          entry: 'https://cdn.bootcss.com/react/16.10.2/umd/react.development.js',
          // 因为 React 会在全局暴露 React 这个变量，所以使用 React
          global: 'React'
        },
        {
          module: 'react-dom',
          entry: 'https://cdn.bootcss.com/react-dom/16.10.2/umd/react-dom.development.js',
          global: 'ReactDOM'
        }
        ]
    }),
]

配置 html( 必须添加 ):
<script src="https://cdn.bootcss.com/react/16.10.2/umd/react.development.js"></script>
<script src="https://cdn.bootcss.com/react-dom/16.10.2/umd/react-dom.development.js"></script>
```

通过设置以上代码，打包出来的结果就不会包含 react, react-dom，体积大大减小。

### 7.2 使用 SplitChunksPlugin 

通过该插件进行公共脚本分离，该插件是 webpack4 内置的，用于替代原来的 CommonChunksPlugin 插件。  

chunks 参数说明:  

- async 异步引入的库进行分离(例如 动态 import，引入 react)
- initial 同步引入的库进行分离
- all 所有引入的库都会进行分离 (推荐)

> minChunks: 某个公共模块被引用的次数，如为 2 ，则如果该模块被引用 >= 2, 则会被认为 minChunks.  


设置 webpack.config.js
```js
module.exports = {
    plugins: [
        new HtmlWebpackPlugin({
            template: path.join(__dirname, `src/${pageName}/index.html`),
            filename: `${pageName}.html`,
            // 将 vendor 引入到 html 中
            chunks: [pageName,'vendors'],
            inject: true,
            minify: {
              html5: true,
              collapseWhitespace: true,
              preserveLineBreaks: false,
              minifyCSS: true,
              minifyJS: true,
              removeComments: true
            }
        })
    ],
    
    // 分离基础包，如 react react-dom 外部代码
    optimization: {
        splitChunks: {
          cacheGroups: {
            commons: {
              test: /(react|react-dom)/,  // 将代码中引用的 react 和 react-dom, 抽取为一个公共的包，包名叫 vendors
              name: 'vendors',  
              chunks: 'all'
            }
          }
        }
    }
    
    // SplitChunksPlugin 还可以用于公共代码抽取(多个 entry 使用到了该文件,例如多页面的场景)的提取，例如的 helper, utils 等 :  
    // 例如多页面应用中有两个 entry，a.js 用于 a.html， b.js 用于 b.html 页面。  
    // a.js 和 b.js 最终都引入了 say.js，那么如果不抽取公共文件，a.bundle.js 和 b.bundle.js 最终都会有 say.js 的代码
    
    // 同样适用于 react, react-dom 等情况，打出 common 包之后
    
    optimization: {
        splitChunks: {
          // 不管包的大小，只要某个模块被引入，就会被提取。例如本例中的 say.js。 
          // 经过测试 say.js 打包出来假设为 1k，假设设置 minSize 为 2k,那么就不会生成 common.bundle.js   
          minSize: 0,
          cacheGroups: {
            commons: {
              name: 'common',    // 公共代码，被打包为 common.js
              chunks: 'all',
              minChunks: 2       // 该模块至少要被引入 2 次，才会被提取,在本例中, a.js 和 b.js 引入共两次. 假设只有 a.js 引入，或者设置 minChunks 为 3 ( 3 > 2) ，那么不会生成 common.bundle.js
            }
          }
        }
  }
}
```
**以上公共代码，要在页面中看到效果，也要添到 HtmlWebpackPlugin 中的 chunks 数组中。**  

## 8. Tree Sharking 

概念：一个模块可能有很多的方法，只要其中某个方法被使用到了(**甚至只是引入某个函数，而函数没有调用**)，则整个文件都会被打到 bundle 中，tree sharking 就是只把用到的方法打入 bundle, 没用到的方法会在 uglify 阶段被擦除掉。  

```js
import a from './a.js'

if(false) {
    a();  // 也会被 tree sharking 掉
}
```

要求:  

1. **必须使用 es6 的 import / export 语法，不能使用 cjs 中的 module.exports / require 语法**。  
2. 引入的模块代码不能有 *副作用*

因为 import 只能在代码顶层出现(不能在 if 中)，因此**在静态分析阶段，就能够知道那些方法被用到了**，而不需要在执行阶段才能判定。  

在 webpack 生产环境的情况下，(mode='production') 的情况下，会默认开启 tree sharking.  

## 9. Scope Hoisting

```js
// hello.js
import {a} from './tree-sharking'
console.log('this is hello');

// tree-sharking.js
export function a() {
  console.log('this is a')
}
export function b() {
  console.log("this is b")
}
```

编译之后的代码:  
```js
/******/ ([
/* 0 */
/***/ (function(module, __webpack_exports__, __webpack_require__) {

// 模块一: hello.js
console.log('this is hello');

/***/ }),
/* 1 */
/***/ (function(module, __webpack_exports__, __webpack_require__) {

"use strict";

// 模块二: tree-sharking
function a() {
  console.log('this is a');
}
function b() {
  console.log("this is b");
}

/***/ })
/******/ ]);
```

可以看到以上两个模块，都被函数包裹着:  

```js
(function(module, __webpack_exports__, __webpack_require__) {
// 模块代码
/***/ })
```

会导致: 
1. 大量闭包函数包裹着代码，导致代码体积增大(模块越多越明显)
2. 运行代码时创建的函数作用域变多，内存消耗变大

scope hoisting 原理: 将所有模块的代码按照 **引用顺序** 放在一个函数作用域里面，然后适当的重命名一些变量以防止变量名冲突。  
要求: 必须是 es6 语法， cjs 语法不支持。  

设置 webpack.config.js  

```js
// 默认 mode 为 production 是已经开启了 scope hoisting
// 但是 production 情况下，代码会被压缩，因此还是手动引入该插件
plugins: [
    new webpack.optimize.ModuleConcatenationPlugin()
]
```

编译结果：
```js
/***/ (function(module, __webpack_exports__, __webpack_require__) {
// 模块二: tree-sharking 
function a() {
  console.log('this is a');
}
function b() {
  console.log("this is b");
}

// 模块一: hello.js
console.log('this is hello');
/***/ })
```

结果同上，最终两个模块会在一个函数包裹里面，这样就减少了闭包数量。  

> 注意: 有些模块是经过 splitChunksPlugin 之后，可能依然会被函数包裹  

## 10. 代码分割和动态 import

对于大的 WEB 应用来说，将所有的代码都放在一个文件中，显然是不够有效的。特别是你的某些代码块是在某些特殊的时候才会被使用到。  

webpack 有一个功能就是将你的代码库分割成 chunks, 当代码运行到需要他们的时候再进行加载。  

适用场景:  
1. 抽离相同的代码到一个共享块
2. 脚本懒加载，使得初始化下载的代码更小

懒加载 JS 的两种方式:  
CommonJS: require.ensure  
ES6: 动态 import (没有原生支持，需要 babel 转换)  

安装依赖: `npm i @babel/plugin-syntax-dynamic-import -D`  

设置 .babelrc:  

```js
{
  "plugins": [
    "@babel/plugin-syntax-dynamic-import"
  ]
}
```

```js
// say.js
import React from 'react';
function Say () {
  return <div>动态 imoprt</div>
}
export default Say;

function App() {
  const [Text,setText] = useState(null)
  return <div className='red'>
    hello world 3 {Text}
    // 点击时，动态加载组件
    <img src={logo} alt="" onClick={() => {
      import('../../common/say').then(Text => {
        setText(Text.default)
      })
    }}/>
  </div> 
}
```

当 webpack 打包发现代码里有动态 import() 的语法，就会 **将被引入的组件单独打包出js** ，当点击时，才会加载该组件的js。  

## 11. Webpack 集成 Eslint

1. 在 CI/CD 中集成 eslint
2. 在 webpack 中集成 eslint

安装依赖：`npm i eslint eslint-plugin-import eslint-plugin-react eslint-plugin-jsx-a11y -D`  

`npm i eslint-loader -D`: 用于配合 babel-loader, 解析 js 文件

`npm i babel-eslint -D`: eslint 的解析器

`npm install eslint-config-airbnb -D`: eslint 的基础规则

设置 webpack.config.js  
```js
module.exports = {
    module: {
        {
            test: /\.js$/,
            use: [
              'babel-loader',
              'eslint-loader',  // 新增该项
            ],
        },
    }
}
```

设置 .eslintrc.js  
```js
module.exports = {
  "parser": "babel-eslint",  // 解析器
  "extends": "airbnb",       // 继承的基础规则
  "env": {
    "browser": true,  // 能识别浏览器环境的全局变量
    "node": true      // 能识别 node 环境的全局变量
  },
  "rules": {          // 可以添加自己的 rules
    "semi": 0
  }
}
```

## 12. webpack 打包组件和库

要求：  
1. 需要打包压缩版和非压缩版本
2. 支持 AMD/CMD/ESM 模块引入，同时支持 script 的方式引入

如何将库暴露出去:  

library: 指定库的全局变量，如 react 库在全局的变量是 React  
libraryTarget: 支持库的引入方式  
libraryExport: "default" // 设置为 default

```
// 如果不设置 libraryExport，那么
import a from './a.js'
a.default.add();

// 设置为 default 之后
import a from './a.js'
a.add();
```

webpack.config.js  
```js
module.exports = {
    entry: {
        'large-number': './src/index.js',
        'large-number.min': './src/index.js',
    },
    output: {
        filename: '[name].js',
        library: 'largeNumber',
        libraryTarget: 'umd',  // 支持 AMD/CMD/ESM 和 script 的方式引入
        libraryExport: 'default'
    }
}
```

如何针对 .min 进行压缩：  

1. 先将 mode 设置为 none
2. 设置 webpack.config.js  
```js
module.exports = {
    optimization: {
        minimize: true,
        minimizer: [
            new TerserPlugin({           // 使用该插件
                include: '/\.min\.js/'   // 针对该打包出来的文件
            })
        ]
    }
}
```

如何针对不同环境引入不同的包？  

设置 package.json 中 main 字段为 index.js, 同时该 index.js 的内容为:  

```js
if(process.env.NODE_ENV === 'production') {
    module.exports = require('./dist/xxx.min.js')
} else {
    module.exports = require('./dist/xxx.js')
}
```

## 13. webpack 实现 SSR 打包 (上)

SSR 优势：  
1. SEO 友好
2. 减少白屏时间

实现思路:  
服务端：  
- 使用 react-dom/server 的 renderToString 方法将 React 组件渲染成字符串
- 服务端路由返回对应的模板

客户端:  
- 打包出针对服务端的组件


服务端的代码: 
```js
// app.js

const express = require('express');
const { renderToString } = require('react-dom/server');
const SSR = require('../dist/search-server')  // 打包出来的search页面组件

const renderMarkup = (str) => {
    return `<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <meta http-equiv="X-UA-Compatible" content="ie=edge">
  <title>Document</title>
</head>
<body>
  <div id="root">${str}</div>
</body>
</html>
    `
}

const server = (port) => {
    const app = express();
    
    arr.get('/search',() => {
        res.status(200).send( renderMarkup(renderToString(SSR)) )
    })
    
    app.listen(port, () => {
        console.log('服务器启动成功')
    })
}
```

客户端打包入口文件的代码:  
```js
// index-server.js  // 该文件打包出来给服务端使用

//
注意， node 端 app.js 中使用的是 module.exports 和 require 的语法
因此以下代码的 import 都要改为 require 语法
//

import React, { useState } from 'react'
import ReactDOM from 'react-dom';  // 移除该行
import logo from './logo.png'
import './index.less'

function App() {
  const [Text, setText] = useState(null)
  return (
    <div className="red">
    hello world 3
      {Text}
      <img
        src={logo}
        alt=""
        onClick={() => {
          import('../../common/say').then((Text) => {
            setText(Text.default)
          })
        }}
      />
    </div>
  )
}

// 移除以下代码
ReactDOM.render(
  <App />,
  document.querySelector('#root'),
)

// 改为直接暴露
module.exports = <App />;
```

修改 webpack.ssr.js  
```
entry: {
    引入 index-server.js，服务端入口文件
}

output: {
    fileName: '[name]-sever.js'  // 打包输出 -server.js 文件
    libraryTarget: 'umd'         // 输出 umd 包
}

打包命令: webpack --config webpack.ssr.js
```

webpack ssr 打包存在的一些问题:  
1. 浏览器中的全局变量(nodejs 没有 window 和 document)
    1. 组件适配，将不兼容的组件根据打包环境进行适配
    2. 请求适配，将 fetch, ajax 请求改为 axios 或者 isomorphic
    -fetch  
2. 样式问题(nodejs 无法解析 css)
    1. 方案一： 服务端打包通过 ignore-loader 忽略 css 的解析
    2. 方案二：将 style-loader 替换为 isomorphic-style-loader(但是必须采用 css in js 的语法)


**报错：**  
1. window is undefined  

解决办法:  
在 app.js 添加以下代码:  

```
const express = require('express');

// 添加
if(typeof window === 'undefined') {
    global.window = {}
}

const server = () => {
    
}
```

## 14. webpack 实现 SSR 打包 (下)

13 节中，我们针对 css 可以采用忽略 css 的做法，但是应该怎么引入 css 呢？  

因为在打包出客户端模板时，css 被打包且内联进模板中了 ，因此我们需要的是将打包后的客户端模板作为基础的壳子，将服务端渲染的结果放进壳子中。  

```
最初的模板
<html>
    <!-- HTML_PLACEHOLDER -->
</html>

// 客户端打包之后的产物, 会将 css 引入到模板中
<html>
    <link href='./xxxx.css' >
    <!-- HTML_PLACEHOLDER -->
</html>

// 服务端读取客户端打包的产物，将 HTML_PLACEHOLDER 替换为 renderToString(SSR))
<html>
    <link href='./xxxx.css' >
    renderToString(SSR))  // 用 ssr 的结果替换原来的 HTML_PLACEHOLDER
</html>
```

同理，我们可以在初始的模板中，添加数据的 PLACEHOLDER:  
```html
<html>
    <!-- HTML_PLACEHOLDER -->
    <!-- HTML_DATA_PLACEHOLDER -->  // 添加数据入口，服务端渲染的时候用数据替换该模块
</html>
```

## 15. 优化构建时的命令行显示日志

统计信息 stats

| preset | alternative | Description |
| --- | --- | --- |
| errors-only | none | 只在发生错误时输出 |
| minimal | none | 只在发生错误或有新的编译时输出 |
| none | false | 没有输出 |
| normal | true | 标准输出 |
| verbose | none | 全部输出 |

设置 webpack.config.js  

```
module.exports = {
    devServer: {
        contentBase: './dist',
        hot: true,
        stats: 'errors-only',  // dev-server 的 stats
    },
    stats: 'errors-only'
}
```
以上设置，只会在 webpack 编译出错的时候才会打印错误信息，对于成功的情况，不会打印任何信息，因此也是不太友好。  

所以，可以使用  `friendly-errors-webpack-plugin` 插件，对于成功，警告，错误都有对应的错误提醒。  

安装依赖: `npm i friendly-errors-webpack-plugin -D`  

设置 webpack.config.js  
```js
const FriendlyErrorsWebpacjPlugin = require('friendly-errors-webpack-plugin')
module.exports = {
    plugins: [
        new FriendlyErrorsWebpacjPlugin()
    ],
    stats: 'errors-only'  // 可以配合该参数一块使用
}
```

## 16. 构建异常和中断处理
在每次构建完成之后，输入 `echo $?` 可以获取错误码的信息。如果错误码不为 0 , 则代表当前构建是失败的。  


Node.js 中的 process.exit 规范  
1. 0 表示成功完成，回调函数中,err 为 null
2. 非0 表示执行失败，回调函数中，err 不为 null，err.code 为传给 exit的数字  

compiler 在每次构建结束后会触发 done 这个 hook:  
```js
module.exports = {
    plugins: [
    function () {
      this.hooks.done.tap('done', (stats) => {
        if (stats.compilation.errors && stats.compilation.errors.length && process.argv.indexOf('--watch') === -1) {
          console.log('build error');
          process.exit(1)
        }
      })
    },
    ]
}
```
此时，如果你构建发生错误(例如引用了一个不存在的模块)，就会打印  `build error` 这个错误信息。  
