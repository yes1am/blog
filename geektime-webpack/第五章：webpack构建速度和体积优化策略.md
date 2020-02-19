---
description: https://github.com/cpselvis/geektime-webpack-course
---

## 1. 初级分析，使用 webpack 内置的 stats

stats: 构建统计信息  

package.json

```js
scripts: {
    // 将构建的信息输出到当前的 stats.json 中
    "build:stats":"webpack --env production --json > stats.json",
}
```

## 2. 速度分析：使用 speed-measure-webpack-plugin

可以看到每个 loader 和 插件的执行耗时。  

安装依赖: `npm i speed-measure-webpack-plugin -D`  

webpack.config.js
```js
const SpeedMeasureWebpackPlugin = require('speed-measure-webpack-plugin')

const smp = new SpeedMeasureWebpackPlugin();

module.exports = smp.wrap({
 entry: ...,
 output: ...,
 plugins: [
    ...
 ]
})
```
通过将 webpack 配置，使用 smp 包裹一遍，最终可以得到每个插件，每个 loader 的时间。  

## 3. 体积分析: 使用 webpack-bundle-analyzer 分析体积

安装依赖: `npm i webpack-bundle-analyzer -D`  

设置 webpack.config.js  
```js
const { BundleAnalyzerPlugin } = require('webpack-bundle-analyzer')

plugins: [
    new BundleAnalyzerPlugin(),
]
```

1. 如某个包很大，想想是否有可能这个包用 CDN 进行加载，而不打入 bundle 中
2. 是否可以利用 js 懒加载，动态 import js

## 4. 使用高版本的 webpack 和 nodejs(优化构建时间)

webpack 4.0 对比 webpack 3.0 ，大概构建时间能降低 60% - 90%。 

使用 webpack 4 能带来优化的原因:  
1. V8 带来的优化(for of 替代 forEach, Map 和 Set 代替 Object，includes 代替 indexOf)
2. 默认使用更快的 md4 hash 算法
3. webpack AST 可以直接从 loader 传递给 AST, 减少解析时间
4. 使用字符串方法，替代正则表达式

## 5. 多进程/多实例构建  

可选方案：  
1. thread-loader (官方方案)
2. parallel-webpack
3. HappyPack  (webpack3 的方案，目前已不再维护，推荐使用 thread-loader)

HappyPack 原理，每次 webpack 解析一个模块，HappyPack 会将它和它的以来分配给 worker 线程中。**thread-loader 其实也是如此**。  


**happypack 配置**  

安装依赖: `npm install --save-dev happypack`  

设置 webpack.config.js  
```js
const HappyPack = require('happypack');

module.exports = {
    module: {
        rules: [
            {
                test: /\.js$/,
                use: [
                  // 'babel-loader',
                  'happypack/loader',  // 注释掉原有的loader，如 babel-loader, 添加 happypack/loader
                ],
            },
        ]
    },
    plugins: [
        new HappyPack({
          // 将注释掉的原有 loader 添加回来
          loaders: ['babel-loader'],
        }),
    ]
}
```

**thread-loader 配置**  

安装依赖: `npm install --save-dev thread-loader`  

设置 webpack.config.js  
```js
module.exports = {
    {
        test: /\.js$/,
        use: [
          {
            loader: 'thread-loader',
            options: {
              worker: 3,  // 启动三个 worker 进程
            },
          },
          'babel-loader',
        ],
      },
}
```

*不知道为啥，我的测试结果没有很大的提升~~*  

## 6. 多进程并行压缩代码


1. 使用 parallel-uglify-plugin 插件  


2. 使用 uglifyjs-webpack-plugin 开启 parallel 参数(不支持压缩 es6 语法)


3. terser-webpack-plugin 开启 parallel 参数 (**推荐使用**)

## 7. 进一步分包: 预编译资源模块

分包：即设置 extrnals 

思路： 将 react, react-dom 基础包通过 cdn 引入，不打入 bundle 中  

方法：使用 html-webpack-extrnals-plugin  

缺点：会引入很多的 script 标签，如 react 一个，react-dom 一个,react-redux 又是一个。  

**更好的方式** 是使用 DLLPlugin,DllReferencePlugin, 将 react, react-dom, redux, react-redux 基础包和业务基础包打包成一个文件。  

新建 webpack.dll.js  
```js
const path = require('path')
const webpack = require('webpack')

module.exports = {
  entry: {
    // 将 react 和 react-dom 进行打包，生成 output.filename 文件
    library: [
      'react',
      'react-dom',
    ],
  },
  output: {
    // 将生成的文件，存放于 build/library 目录中
    // 全局暴露的变量是 library
    filename: '[name]_[chunkhash:8].dll.js', // 生成 library_[xxx].dll.js
    path: path.join(__dirname, 'build/library'),
    library: '[name]',
  },
  plugins: [
    // =============================================
    // dll plugin 中的name 必须和 output.library 一致
    // =============================================
    new webpack.DllPlugin({
      name: '[name]',
      path: path.resolve(__dirname, './build/library/[name].json'),
    }),
  ],
}
```

设置 webpack.config.js
```js
plugins: [
    // manifest 的值与 DLLPlugin 的 path 输出的json路径需要一致
    new webpack.DllReferencePlugin({
      manifest: path.resolve(__dirname, './build/library/library.json'),
    }),
]
```

***最后，需要将 [name]_[chunkhash:8].dll.js 引入到 html 模板中。***

library.json 长这样:  
```
{
  "name": "library",
  "content": {
    "./node_modules/react/index.js": {
      "id": 0,
      "buildMeta": {
        "providedExports": true
      }
    },
    "./node_modules/react-dom/index.js": {
      "id": 0,
      "buildMeta": {
        "providedExports": true
      }
    },
  }
}
```

大致就是，DllPlugin 在打包 dll.js(内部暴露一个 library 的变量，内部包含了 react 和 react-dom 打包后的内容) 的时候，会生成以上的 .json 映射文件。  

在 DllReferencePlugin 中引用该映射文件，使得在遇到 react 和 react-dom 的时候，不会打入 bundle 中。  

**而最终，还是要在页面中引入 .dll.js 文件。**  

[参考资料](https://juejin.im/entry/5acbb6095188257cc20d98e1)  


## 8. 充分利用缓存提升二次构建速度

思路：  
1. babel-loader 开启缓存: `{cacheDirectory: ture}` [参考文档](https://webpack.js.org/loaders/babel-loader/#options)
2. terser-webpack-plugin 开启压缩缓存: `{cache: true}` [参考文档](https://webpack.js.org/plugins/terser-webpack-plugin/#cache)
3. 使用 cache-loader 或者 hard-source-wekpack-plugin  [参考文档](https://github.com/mzgoddard/hard-source-webpack-plugin#cachedirectory)

以上开启缓存之后，会在 node_modules 下生成 .cache 目录，用于存放缓存结果。  

## 9. 缩小构建目标

比如设置 babel-loader 不解析(`exclude`) node_modules 

1. 优化 resolve.modules 配置，减少模块搜索层级
2. 优化 resolve.mainFields 配置
3. 优化 resolve.extensions 配置
4. 合理使用 alias

[参考文档](https://webpack.js.org/configuration/resolve/#root)

```js
module.exports = {
    resolve: {
        alias: {
            // 让 import react 的时候，直接查找到该目录
            react: path.resolve(__dirname, './node_modules/react/dist/react.min.js')
            ...
        },
        // 只在当前的 node_modules 查找，不继续往父级查找
        modules: [path.resolve(__dirname, 'node_modules')],
        // 设置只查找什么后缀的文件
        extensions: ['.js'],
        // 默认 webpack 会查找 package.json 中的 main 字段
        // 之后会查找 index.js 文件等
        // 设置 mainFields 之后，只找 package.json 中的main 字段
        mainFields: ["main"]
    }
}
```

## 10. 使用 Tree Sharking 擦除无用的 CSS

无用的 CSS 代码如何删除掉?  

1. Purufy CSS: 遍历代码，识别已经用到的 CSS class
2. uncss: HTML 需要通过 jsdom 来加载，所有的样式通过 PostCSS 解析，通过 document.querySelector 来识别在 html 文件里面不存在的选择器。 即 document.querySelector 能获取到元素，则说明该样式有被用到。否则是无用的。  

做法: purgecss-webpack-plugin 与 mini-css-extract-plugin 一起使用  

[参考文档](https://github.com/FullHuman/purgecss/tree/master/packages/purgecss-webpack-plugin)  

安装依赖: `npm i purgecss-webpack-plugin -D`  

设置 webpack.config.js  

```js
const PurgecssPlugin = require('purgecss-webpack-plugin')

const PATHS = {
  src: path.join(__dirname, 'src'),
}

new PurgecssPlugin({
    paths: glob.sync(`${PATHS.src}/**/*`, { nodir: true }),
}),
```

## 11. 使用 webpack 进行图片压缩

[无版权图片网站 unsplash.com](https://unsplash.com/)  

基于 node 库的 imagemin (**推荐**) 或者 tinypng 的 API。  

安装依赖: `npm install image-webpack-loader --save-dev`  

设置 webpack.config.js  

```js
{
  test: /\.(png|jpg|jpeg|gif)$/,
  use: [{
    loader: 'file-loader', // 不能使用 url-loader, 因为 url-loader 是采用 base64 的方式
    options: {
      name: '[name].[hash:8].[ext]', // 设置图片 hash
    },
  },
  {
    loader: 'image-webpack-loader',
    options: {
      mozjpeg: {
        progressive: true,
        quality: 65,
      },
      // optipng.enabled: false will disable optipng
      optipng: {
        enabled: false,
      },
      pngquant: {
        quality: [0.65, 0.90],
        speed: 4,
      },
      gifsicle: {
        interlaced: false,
      },
      // the webp option will enable WEBP
      webp: {
        quality: 75,
      },
    },
  },
  ],
},
```

## 12. 使用动态 Polyfill 服务

例如 Promise 语法，Map, Set 语法。问题在于但部分手机浏览器能元素支持 Promise，有一些浏览器不能。  

如果统一返回 Promise，对于能支持 Promise 的机器来说，就是多余的。  


| 方案 | 优点 | 缺点 | 是否采用 |
| --- | --- | --- | --- |
| babel-polyfill | React 16 官方推荐 | 1. 包体积 200k, 难以单独抽离 Map 和 Set <br /> 2. 项目里 react 是单独引用的 cdn，如果要使用 polyfill, 需要单独构建一份在 react 之前加载 | 否 |
| babel-plugin-transform-runtime | 能够只引用某些用到的类或者方法，相对体积较小 | 不能 polyfill 原型上的方法，不适用于业务项目上的复杂环境 | 否 |
| 自己写 Map， Set，Promise 的polyfill [社区维护的 es6-shim](https://github.com/paulmillr/es6-shim) | 定制化高，体积小 | 1. 重复造轮子,日后年久失修成为坑 <br/> 2. 即使体积小，依然所有用户都要加载 | 否 |
| [polyfill-service](https://github.com/Financial-Times/polyfill-service) (后端服务，根据请求者的UA 返回相应的 polyfill) | 只给用户返回需要的 polyfill, 社区维护 | 部分国内奇葩浏览器UA可能无法识别，(此时可以降级返回所有的 polyfill) | 是 |  

polyfill-service 地址: https://polyfill.io/v3/polyfill.js  


体积优化总结:  
1. scope hoisting
2. tree-sharking
3. 公共资源分离
4. 图片压缩
5. 动态 polyfill