## 1. 使用 source map 之前

在 [express-react-boilerplate](https://github.com/yes1am/express-react-boilerplate) 中，一开始是没有 source map 的, 这导致的结果是, 开发环境下 chrome source 中看到的代码如下:  

![编译后的代码](https://raw.githubusercontent.com/yes1am/PicBed/master/img/20200314112717.png)  

而我们实际的代码是这样的:  

![实际代码](https://raw.githubusercontent.com/yes1am/PicBed/master/img/20200314112845.png)  

同时，我们还用到了 antd ，我们来看看 antd 的代码:  

![antd 的代码](https://raw.githubusercontent.com/yes1am/PicBed/master/img/20200314113106.png)

antd 默认使用的打包版本是 `antd/lib` 中的代码，我们在其中打断点，调试 antd 的代码。  

以上的两张 `chrome / source` 截图代码，虽然 **不是很易读**，但是勉强能够阅读调试。

## 2. 使用 source map 之后

我们通过给 webpack 添加 source map:  

```js
module.exports = Object.assign({
  mode: 'development',
  // 开启 source map，方便在 chrome source 中查看可阅读的代码
  devtool: 'source-map',
  entry: {
    //
  },
  output: {
    // 
  }
}, baseConfig.dev)
```

之后我们可以看到以下的代码, **我们的代码具备可阅读性并且可调试**:  

![添加source map 之后](https://raw.githubusercontent.com/yes1am/PicBed/master/img/20200314114340.png)

当然，此时 `antd/lib` 的代码还是和之前一样的。

## 3. 遇到的一次问题

在以上这些情况中，尽管 antd 的代码不是特别可读，但是起码 **能看懂一些**，所以也没有什么问题。  

直到有一天，我负责的组件库中某个组件出现了意料之外的情况，必须要调试这个组件库:  

![被压缩的组件库](https://raw.githubusercontent.com/yes1am/PicBed/master/img/20200314120038.png)

这是一份 **完全不可读** 的代码，同时由于被压缩，导致 **调试也极不方便**  

组件库的打包配置:  
```js
devtool: '#source-map',   // 打包出 source-map
externals: {
  react: 'react',
  'react-dom': 'react-dom',
},
optimization: {
  minimize: true,
  minimizer: [
    new TerserPlugin({
      cache: true,
      parallel: true,
      sourceMap: true,
    }),
  ],
},
```

所幸，组件库打包出了 source map 文件，因此我们可以通过**加载 source map** 来得到可阅读的代码。  

首先配置我们 `express-react-boilerplate` 项目的 webpack:  

先安装 source-map-loader, 然后配置如下:  

```js
module: {
  rules: [
    {
      test: /\.jsx?$/,
      use: ['source-map-loader', 'babel-loader'],
      enforce: 'pre'
    },
  ]
}
```

这样配置之后，我们就能看到 **可读性且可调试的代码**  

![添加 source-map-loader 之后](https://raw.githubusercontent.com/yes1am/PicBed/master/img/20200314121216.png)  

**注意，左侧的 webpack 新增了几行 webpack 目录, 并不在原来的 【webpack .】 中**，没有摸透什么原理，因此 **建议打开每个 webpack 目录看看是否能找到想看的组件代码**  

**注意**  
在 webpack.config.js 中:  

```js
module: {
  rules: [
    {
      test: /\.jsx?$/,
      use: ['source-map-loader', 'babel-loader'],
      enforce: 'pre'
      
      // 不能 exclude node_modules
      // 或者只 include src 下的代码
      // 以下代码都会导致组件的 source map 失败
      exclude: /node_modules/,
      include: [
        path.resolve(__dirname, '../app/views')
      ],
    },
  ]
}
```

通过以上的方式，我们就能够正常的调试组件代码了。
