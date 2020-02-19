## 1. 项目地址 

[代码仓库](https://github.com/yes1am/express-react-boilerplate)  


## 2. 环境搭建

### 2.1 安装 webpack

`npm i webpack webpack-cli webpack-dev-middleware webpack-hot-middleware -D`  

[webpack-dev-middleware](https://www.npmjs.com/package/webpack-dev-middleware): 用于监听文件变化，编译进内存供服务器使用  

[webpack-hot-middleware](https://www.npmjs.com/package/webpack-hot-middleware): 连接浏览器与webpack，实现浏览器的热更新

### 2.2 安装 babel

`npm i @babel/core babel-loader @babel/preset-env @babel/preset-react -D`  

#### 2.2.1 相关依赖解释
`babel-loader`:  `babel`的`webpack`加载器  
`babel-preset-env`: 将 `ES6` 转为 `ES5`  
`babel-preset-react`: 用于转换 `react jsx` 语法  

#### 2.2.2 配置 .babelrc
```
{
    "presets": ["@babel/preset-env","@babel/preset-react"]
}
```  
presets是逆序的，意味着会先编译 `react` 再编译 `es6` 

### 2.3 React

`npm i react react-dom -D`  

### 2.4 Standard 代码格式化

`npm i standard -D`  

#### 2.4.1 配合vscode
在 `vscode` 安装 `standard` 插件，配置 `Standard : Auto Fix On Save`,重启 `vscode` 生效.  

#### 2.4.2 设置 script 脚本
```
"lint":"standard \"./app/**/*.js\"",
"fix":"standard \"./app/**/*.js\" --fix"
```  
其中 `**` 表示递归  

### 2.5 express

`npm i express`  

### 2.6 支持 css,less

`npm i style-loader css-loader less less-loader -D`

### 2.7 热加载  

#### 2.7.1 webpack.config.js  
```
const webpack = require('webpack')

...

entry: {
  app: ['./app/views/index.js', 'webpack-hot-middleware/client?path=/__webpack_hmr&reload=true']
},

...

plugins: [
  new webpack.HotModuleReplacementPlugin()
]
```  

#### 2.7.2 server.js  
```
const webpackDevMiddleware = require('webpack-dev-middleware')
const webpackHotMiddleware = require('webpack-hot-middleware')
const config = require('../webpack.config')

...

app.use(webpackDevMiddleware(compiler, {
  publicPath: config.output.publicPath,
  hot: true
}))

// Add hot middleware support, Check [HMR] connected in console
app.use(webpackHotMiddleware(compiler, {
  log: console.log,
  path: '/__webpack_hmr'
}))
```  

### 2.8 使用 antd

`npm i antd -D`  

#### 2.8.1 支持按需加载  

`npm i babel-plugin-import -D`  

* 配置 `.babelrc`  

```
"plugins": [
    [
        "import", {
            "libraryName": "antd",
            "style": true
        }
    ]
]
```  

* 配置 `webpack.config.js`  

```
{
    test: /\.(less|css)$/,
    use: [
        { loader: 'style-loader' },
        { loader: 'css-loader' },
        { loader: 'less-loader', options: { javascriptEnabled: true } }
    ]
}
```