---
description: https://github.com/cpselvis/geektime-webpack-course
---

## 1. entry

webpack 根据入口文件，找到入口文件对应的依赖，入口文件的依赖再找到它们的依赖，最终形成了依赖树(依赖不仅仅是 js， 还可以是 css, 图片，字体等资源)。  

### 1.1 两种用法

单入口:  

```js
module.exports = {
    entry: './path/to/my/entry/file.js'
}
```  

多入口: 

```js
module.exports = {
    entry: {
        app: './src/app.js',
        adminApp: './src/adminApp.js'
    }
}
```

## 2. output

output 告诉 webpack 如何将编译后的文件输出到磁盘.  

```js
module.exports = {
    entry: {
        app: './src/app.js',
        search: './src/search,js'
    },
    output: {
        filename: '[name].js',   // 通过占位符来确保文件名称的唯一
        path: __dirname + '/dist'
    }
}
```

## 3. Loaders

webpack 开箱即用只支持 JS 和 JSON 两种文件类型，通过 Loaders 去支持其他文件类型(如 css, JSX 等)并把他们转化为有效的模块，并且可以添加到**依赖图**中。  

本身是一个函数，接受源文件作为参数，返回转换的结果。  

### 3.1 常用的 Loader  

| 名称 | 描述 |
|---|---|
| babel-loader | 转换 ES6, ES7 等新特性语法 |
| css-loader | 支持 css 文件的加载和解析 |
| less-loader | 将 less 文件转换为 css |
| ts-loader | 将 ts 转换为 js |
| file-loader | 将图片，字体进行打包 |
| raw-loader | 将文件以字符串的形式导入(文件内联) |
| thread-loader | 多进程打包 js 和 css |

### 3.2 用法

```js
module.exports = {
  entry: {
    app: './src/index.js',
    hello: './src/hello.js'
  },
  output: {
    path: path.join(__dirname, './dist'),
    filename: '[name].js'
  },
  mode: 'production',
  module: {
    rules: [
      test: /\.txt$/,                    // test 指定匹配规则
      use: 'raw-loader'                  // use 指定使用的 loader 名称
    ]
  }
}
```

## 4. Plugins

插件用于 bundle 文件的优化，资源管理，和环境变量的注入，等 Loaders 不能工作的领域。

作用于整个构建过程。  

### 4.1 常用的 Plugins

| 名称 | 描述 |
|---|---|
| CommonChunkPlugin | (多页面应用时) 将chunks 相同的模块代码提取成公共js |
| CleanWebpackPlugin | 清理构建目录 |
| ExtractTextWebpackPlugin | 将 css 从 bundle 文件中抽取成一个独立的 css |
| CopyWebpackPlugin | 将文件或文件夹拷贝到构建的输出目录 |
| HtmlWebpackPlugin | 创建 html 文件去承载输出的 bundle |
| UglifyjsWebpackPlugin | 压缩 JS |
| ZipWebpackPlugin | 将打包输出的资源生成一个 zip 包 |

### 4.2 plugins 的使用

```js
module.exports = {
  entry: {
    app: './src/index.js',
    hello: './src/hello.js'
  },
  output: {
    path: path.join(__dirname, './dist'),
    filename: '[name].js'
  },
  mode: 'production',
  module: {
    rules: [
      test: /\.txt$/,
      use: 'raw-loader'
    ]
  },
  plugins: [
    new HtmlWebpackPlugin({              // 插件
      template: './src/index.html'
    })
  ]
}
```
## 5. webpack 中的 mode

Mode 用于指定当前的构建环境: production, development 还是 none。  

设置 mode 可以使用 webpack 内置的一些函数，如代码是否压缩等，默认值为 production。  

### 5.1 mode 的内置函数功能

| 选项 | 描述 |
| --- | --- |
| development | 设置 process.env.NODE_ENV 为 development，开启 NamedChunksPlugin 和 NamedModulesPlugin **(用于代码热更新，可以知道热更新时是哪个模块更新了)** |
| production | 设置 process.env.NODE_ENV 为 production, 开启 FlagDependencyUsagePlugin，FlagIncludeChunksPlugin, OccurrenceOrderPlugin, SideEffectsFlagPlugin 和 TerserPlugin |
| none | 不开启任何优化选项 |  

## 6. 转换 ES6 和 React JSX

### 6.1 转换 ES6 

假设有以下的代码:  

```js
// 输入
const hello = () => {
  console.log('hello');
}

// 打包输出
const hello = () => {
  console.log('hello');
}
```

webpack 正常情况下，打包的结果和源代码一致。但是箭头函数在一些低版本的浏览器中并不能执行，对于低版本浏览器，我们需要将 ES6 语法转换为 ES5 的语法。  

于是我们添加上 babel-loader，而 babel-loader 依赖于 babel, 同时使用 .babelrc 作为配置文件.  

```
module.exports = {
  // ...
  module: {
    rules: [
      {
        test: /\.js$/,                  // 针对 .js 文件使用 babel-loader
        use: 'babel-loader'
      }
    ]
  },
  plugins: [
  ]
}
```

设置 .babelrc:  

```
{
  "presets": [
    "@babel/preset-env"              // 转换 es6 需要这一项
  ],
  "plugins": [
    "@babel/proposal-class-properties"
  ]
}

// 可以这么理解 plugins 和 presets, 即 plugins 对应一个功能，一个 presets 是一系列 plugins 的集合
```

安装依赖: `npm i @babel/core @babel/preset-env babel-loader -D`, (暂时可以先不用到 @babel/proposal-class-properties)  

之后，再次进行打包:  

```js
// 输入
const hello = () => {
  console.log('hello');
}

// 打包输出
var hello = function hello() {
  console.log('hello');
};
```  

于是，就能兼容低版本的浏览器了。  

### 6.2 转换 JSX

先安装 react 和 react-dom 以及转换 react 要用到的 `@babel/preset-react`:  

`npm i react react-dom @babel/preset-react -D`

在 `.babelrc` 中添加 `@babel/preset-react`  

```js
{
  "presets": [
    "@babel/preset-env",
    "@babel/preset-react"               // 新增对于 react 的解析
  ]
}
```

新建 index.js 文件:  

```js
import React from 'react'
import ReactDOM from 'react-dom';

function App() {
  return <div>hello world</div> 
}

ReactDOM.render(
  <App />,
  document.querySelector('#root')
)
```

打包该文件，引入到 html 文件中即可查看到效果。  

## 7. 转换 CSS, Less

### 7.1 解析 CSS

css-loader 用于加载 .css 文件，并且转化为 commonjs 对象，即当在文件中 `import './xxx.css'` 的时候，css-loader 会将 css 文件转换为 commonjs 对象  

`style-loader` 将样式通过 <style> 标签插入到 head 中  

先安装依赖:  

`npm i style-loader css-loader -D`  

配置一下 webpack.config.js  

```js
module: {
  ... 
  rules: [
    {
      test: /\.css$/,     // 新增对于 css 文件的解析，
      use: [              // use 支持链式调用，先执行 css-loader, 再执行 style-loader
        'style-loader',
        'css-loader'
      ]
    }
  ]
},
```

### 7.2 解析 Less

安装依赖:  

`npm i less less-loader -D`  

因此 less-loader 本身基于 less， 因此 less 也需要安装。  

接着，webpack.config.js 新增对于 .less 文件的支持:  

```js
module: {
    rules: [
      {
        test: /\.css$/,
        use: [
          'style-loader',
          'css-loader'
        ]
      },
      {
        test: /\.less$/,   // 新增该项
        use: [
          'style-loader',
          'css-loader',
          'less-loader'    // .less 文件先经过 less-loader 处理，再通过 css, style 等 loader 处理
        ]
      }
    ]
  },
```

## 8. 解析图片和字体

### 8.1 解析图片

file-loader 可以用来处理文件。  

安装依赖:  `npm i file-loader -D`  

配置 webpack.config.js：  

```js
module: {
  rules: [
    {
      test: /\.(png|jpg|jpeg|gif)$/,        // 针对这些图片文件使用 file-loader
      use: 'file-loader'
    }
  ]
}
```

使用
```js
import React from 'react'
import ReactDOM from 'react-dom';
import logo from './images/logo.jpg';   // 直接引入图片
import './index.less'

// 经过 file-loader 之后， logo.jpg 被编译为 [hash].jpg，存放在 dist 目录, hash 为计算所得的值
// 此时 logo 即为编译后的图片地址: [hash].jpg
console.log(logo);  

function App() {
  return <div className='hello'>
    <img src={logo} alt=""/>
  </div> 
}

ReactDOM.render(
  <App />,
  document.querySelector('#root')
)
```

### 8.2 解析字体

file-loader 也可以用来解析字体  

[字体下载](https://github.com/junmer/source-han-serif-ttf/blob/master/SubsetTTF/CN/SourceHanSerifCN-Bold.ttf)  

配置 webpack.config.js  

```js
{
    test: /\.(woff|woff2|eot|ttf|otf)$/,   // 针对字体文件使用 file-loader
    use: 'file-loader'
}
```

在 css 中引入字体，即可查看到效果

```
// ./images/SourceHanSerifCN-Bold.ttf 为字体存放的位置

@font-face {
  font-family: 'SourceHanSerifCN-Bold';
  src: url('./images/SourceHanSerifCN-Bold.ttf') format('truetype');
}
body {
  .hello {
    color: blue;
    font-family: 'SourceHanSerifCN-Bold';
  }
}
```

### 8.3 使用 url-loader 解析图片和字体

url-loader 和 file-loader 都可以处理图片和字体, 实质上 url-loader 是**基于 file-loader**的，url-loader 相对 file-loader 而言，可以设置**较小的资源**自动进行 base-64 的转换。  

安装依赖: `npm i url-loader -D`  

配置 webpack.config.js  

```js
{
  test: /\.(png|jpg|jpeg|gif)$/,
  use: [{
    loader: 'url-loader',
    options: {
      limit: 10240   // 10k, 如果图片小于 10k，那么就会被转为 base64
    }
  }]
},
```

测试图片可随意在 Google 搜索 **icon png**， 找到小于 10k 的图片  

```js
import React from 'react'
import ReactDOM from 'react-dom';
import logo from './images/logo.jpg';
import './index.less'


// 如果 logo.jpg 大于 10k， 会在 dist 目录生成一个 [hash].jpg 的文件， logo 变量为 [hash].jpg  的 url 路径

// 而如果 logo.jpg 小于 10k 时，则不会在 dist 目录生成文件，同时 logo 为 base64 的内容

console.log(logo); 

function App() {
  return <div className='hello'>
    hello
    <img src={logo} alt=""/>
  </div> 
}
```

## 9. webpack 设置文件监听

***缺点：每次变更文件，还是需要手动刷新浏览器才能够得到变化后的代码***

文件监听是在发现源码发生变化的时候，自动重新构建出新的输出文件。

两种方式:  

1. webpack 启动参数中添加 --watch, 例如在 package.json 中添加

`watch": "webpack --watch`  

此时修改源代码之后，**手动刷新浏览器**即可实现文件监听

2. 在 webpack.config.js 中设置 watch

```js
module.exports = {
  entry: {
    index: './src/index.js',
  },
  output: {
    path: path.join(__dirname, './dist'),
    filename: '[name].js'
  },
  watch: true            // 设置为监听文件变化
}
```

### 9.1 文件监听原理分析

webpack 通过 **轮询** 判断文件的最后编辑时间是否发生变化。如果某个文件发生了变化，**不会立刻告诉监听者**，而是先变化缓存起来，在 `aggregateTimeout` 时间(如300ms)内，如果有其他文件也进行了修改，则会同其他的文件一起进行编译。  

```js
module.export = {
    watch: true;  // 默认为false, 即不开启监听
    ignored: /node_modules/,  // 设置忽略监听的目录，设置 node_modules 之后会有性能的提升
    aggregateTimeout: 300,   // 监听到文件变化，先进行缓存，300ms 之后才会真正的进行编译
    poll: 1000   // 轮询的时间间隔，即每隔 1s 中去询问文件是否发生变化
}
```

## 10. webpack 中的热更新及原理分析

### 10.1 实现热更新的两种方式

1. 使用 webpack-dev-server

使用到 webpack-dev-server，使用 webpack-dev-server 之后，就可以实现 **文件改变之后，浏览器自动呈现最新的结果**  

webpack-dev-server 不像之前的方式一样，把文件输出到 dist 目录，而是先**放在内存中**，能够提升性能  

webpack-dev-server 需要和 `HotModuleReplacementPlugin` 插件一起使用。  

安装依赖:  `npm i webpack-dev-server -D`  

配置 webpack.config.js:  

```js
// HotModuleReplacementPlugin 是 webpack 内置的 plugin, 因此需要引入 webpack
const webpack = require('webpack')

module.exports = {
  watch: true,
  mode: 'development',   // webpack dev server 只在开发环境有效
  plugins: [
    new webpack.HotModuleReplacementPlugin()
  ],
  devServer: {           // 设置 webpack-dev-server 的配置
    contentBase: './dist',    // devServer 服务器的根目录
    hot: true
  }
}
```

修改 package.json  

```js
"scripts": {
    "build": "webpack",
    "dev": "webpack-dev-server --open"  // 新增这一项， --open 参数，会在编译完成之后自动打开默认的浏览器
},
```

此时修改文件，保存，浏览器即可看到最新的文件，而**无须手动刷新**  

2. 使用 webpack-dev-middleware

[参考 express-react-boilerplate ](https://github.com/yes1am/express-react-boilerplate/blob/master/app/server.js)  

由于 webpack-dev-server 使用配置的方式启动服务器，而日常开发中可能更需要的是自己实现 node 服务，从而更加灵活， 如使用 `express` 或者 `koa` 的形式，这时候就需要用到 `webpack-dev-middleware` 了  

`webpack-dev-middleware` 能够实现将 webpack 输出的文件，通过**内存**，传输给服务器。  

*以下是我的理解，通过查阅了一些资料，暂时没有进行测试*  
这个时候，浏览器是感知不到的，仍然需要手动刷新浏览器。 因此，通常还需要 `webpack-hot-middleware` 

### 10.2 原理分析

![webpack 热更新原理图](https://raw.githubusercontent.com/yes1am/PicBed/master/img/webpack%E7%83%AD%E6%9B%B4%E6%96%B0%E5%8E%9F%E7%90%86.png)  

**Webpack Compiler**: 将 js 源代码编译为 bundle.js  

**HMR Server**: 将热更新的文件，传输给浏览器端的 HMR Runtime  

**Bundle Server**:  作为服务器，提供文件给浏览器去访问，比如浏览器访问 `localhost:8080/bundle.js`， 那么就会请求到 `Bundle Server` 中去，而 `Bundle Server` 通过访问内存返回对应的资源  

**HMR Runtime**: 在 **开发阶段(development)** 的情况下，会被打包到 `bundle.js` 中, 然后通过 `websocket` 与 `HMR Server` 进行连接。 当 `HMR Server` 中得到到本地文件变化的消息之后，就会告诉 `HMR Runtime` 去更新 bundle.js.  


**热更新的两个过程:**  
**1. 启动阶段**：`webpack compiler` 将初始的代码，进行编译打包，然后传输给 `Bundle Server` 这个服务器，当客户端请求的时候，`Bundle Server` 就作为服务器返回对应的资源。 对应的路径是： `1 => 2 => A => B`  
**2. 更新阶段**:  当本地文件发生变化的时候，`webpack compiler` 将改变后的代码进行编译，编译好之后发送给 `HMR Server`, `HMR Server` 就可以知道是哪些 JS 模块发生了改变，之后再将这些变化通知给 `HMR Runtime`, 这样浏览器端就能进行对应模块的更新，而无需刷新浏览器。  

## 11. 文件指纹 (hash)

文件指纹，即打包后输出的文件名的后缀，通常可以用来做 **版本管理**。只有文件修改了之后，文件指纹才会修改，而没有修改的文件，指纹不会发生变化，这样浏览器能使用本地的缓存，优化访问速度。  

**Hash**: 和整个项目的构建相关，只要项目文件(任意一个文件)有修改，整个项目构建的 hash 值就会更改  

比如有 a.js 和 b.js 两个 entry ，如果 a.js 内容发生了变化，b.js 的hash最终也会改变，而这是没有必要的，因此引入了 **Chunkhash**。  

**Chunkhash**: 和 webpack 打包的 chunk 有关，不同的 entry 会生成不同的 chunkhash 值，此时如果一个 entry 中的某个 js 发生改变，并不会影响另一个 entry 的 chunkhash 值，因此 **对于 js 文件的指纹，采用 chunkhash**  

**Contenthash**: 根据文件内容来定义 hash，文件内容不变，则 contenthash 不变。 这样理解，`a.js` 引入了 `utils.js` 和 `index.css`, 如果 `utils.js` 改变了内容，a.js 是 chunkhash，那么 a.js 的指纹就会发生变化。 如果 index.css 也是 chunkhash, 那么 index.css 的指纹也会发生变化，但是 **index.css 的内容实际上并没有变化**，这样是不合理的。 因此**对于 css 类型的资源，一般都是设置为 contenthash** 只有文件内容发生变化才会改变指纹  

### 11.1 文件指纹设置

**JS的文件指纹设置**:  

设置 webpack.config.js
```js
module.exports = {
    output: {
        path: path.join(__dirname, './dist'),
        filename: '[name].[chunkhash:8].js'      // 表示取 hash 的前 8 位
    }
}
```

***注意***:  
chunkhash 不能和 `HotModuleReplacementPlugin` 一起使用，否则会报错  
> Cannot use [chunkhash] or [contenthash] for chunk in '[name].[chunkhash:8].js' (use [hash] instead)  

**css的文件指纹设置**:  

css 文件的解析，通过 `css-loader` 和 `style-loader` 之后，`style-loader` 会将 css 的内容插入到 html 的头部，**这时候并没有css文件生成**  

通常可以使用 `MiniCssExtractPluigin`, 将 css 抽取出来，成为独立的 css 文件。  

安装依赖: `npm i mini-css-extract-plugin -D`  

设置 webpack.config.js
```js
const MiniCssExtractPlugin = require('mini-css-extract-plugin');

module.exports = {
  module: {
    rules: [
      {
        test: /\.css$/,
        use: [
          // 由于 MiniCssExtractPlugin 是将 css 抽成单独的文件
          // 而 style-loader 是将 css 放到 html 头部形成单独的文件
          // 因此这两部分是冲突的，需要用 MiniCssExtractPlugin.loader 代替 style-loader  
          
          MiniCssExtractPlugin.loader,
          'css-loader'
        ]
      },
      {
        test: /\.less$/,
        use: [
          MiniCssExtractPlugin.loader,
          'css-loader',
          'less-loader'
        ]
      }
  },
  plugins: [
    new MiniCssExtractPlugin({
      filename: '[name].[contenthash:8].css'   // 在这里设置 css 文件的 hash
    })
  ]
}
```
**图片,字体等文件指纹设置**:  

图片，字体都是通过 file-loader 来设置的，设置 `file-loader` 的 name, 使用 `[hash]`, (**这个hash 不是与 chunkhash 和 contenthash 一起的 hash，而 file-loader 中 hash 占位符，用于表示文件内容，通常是由 md5 生成的**)  

此外 file-loader 还有其他的占位符:  

| 占位符 | 含义 |
| --- | --- |
| [ext] | 资源后缀名 |
| [name] | 文件名称 |
| [path] | 文件的相对路径 |
| [folder] | 文件所在的文件夹 |
| [contenthash] | 文件的内容 hash，默认是 md5 生成 |
| [hash] | 文件内容的 hash，默认是 md5 生成 |
| [emoij] | 一个随机的指代文件内容的 emoij |

设置 webpack.config.js
```js
module.exports = {
  module: {
    rules: [
        {
          test: /\.(png|jpg|jpeg|gif)$/,
          use: [{
            loader: 'file-loader',          // 不能使用 url-loader, 因为 url-loader 是采用 base64 的方式
            options: {
              name: '[name].[hash:8].[ext]' // 设置图片 hash
            }
          }]
        },
        {
          test: /\.(woff|woff2|eot|ttf|otf)$/,
          use: [{
            loader: 'file-loader',
            options: {
              name: '[name].[hash:8].[ext]'   // 设置字体 hash
            }
          }]
        }
    ] 
  }
}
```

## 12. HTML, CSS, JavaScript 的压缩

**JS 的压缩**  
webpack4 内置了 `uglifyjs-webpack-plugin` 参数来压缩 JS 代码，在 **production 环境默认是开启的** ，在 **development 环境默认是关闭的**，因此不需要额外的设置。当然，也可以手动安装该插件，从而设置更多的参数，比如让它支持 **并行压缩**。  

**CSS的压缩**  
通过 OptimizeCSSAssetsWebpackPlugin 插件来完成，同时使用 `cssnano`.

安装依赖: `npm i optimize-css-assets-webpack-plugin cssnano -D`  

设置 webpack.config.js  
```js
const OptimizeCssAssetsWebpackPlugin = require('optimize-css-assets-webpack-plugin');  

module.exports = {
  mode: 'production',
  plugins: [
    new OptimizeCssAssetsWebpackPlugin({
      assetNameRegExp: /\.css$/g,               // 匹配 css 文件
      cssProcessor: require('cssnano')          // 使用 cssnano
    })
  ]
}
```

这样，最终打包出来的 css 文件就是被压缩了的。  


**HTML文件的压缩**  
涉及 `html-webpack-plugin` 这个插件, 自动根据模板生成 html 文件，同时支持设置压缩参数(`minify`)。  

安装依赖:  `npm i html-webpack-plugin -D`  

设置 webpack.config.js:  
```js
const HtmlWebpackPlugin = require('html-webpack-plugin');

module.exports = {
    plugins: [
      // 第一个页面
      new HtmlWebpackPlugin({
        template: path.join(__dirname, 'src/index.html'),  // 使用的模板文件所在的位置
        filename: 'index.html',                            // 指定打包之后的 html 文件名称
        // 指定引入的 chunk 文件，因为最终会把 chunk 地址默认加到 html 中
        chunks: ['index'],
        inject: true,                                      // 设置 css, js 是否自动注入到页面中
        minify: {
          html5: true,
          collapseWhitespace: true,                        // 是否将 html 模板中的空白字符删除
          preserveLineBreaks: false,
          minifyCSS: true,
          minifyJS: true,
          removeComments: true                             //  是否移除 html 中的注释
        }
      }),
      // 多页面打包时，可以再新建一个配置
      // new HtmlWebpackPlugin({
      // })
    ],
}
```