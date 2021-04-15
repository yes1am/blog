## 1. 基础
output.library 的类型是: `string` | `string[]` | `object` 
## 2. 当 output.library 类型为 string
foo.js
```javascript
export function hello() {
  console.log(123);
}
```
webpack.config.js
```javascript
const path = require('path');

module.exports = {
  entry: path.join(__dirname, './src/foo.js'),
  output: {
    path: path.join(__dirname, "dist"),
    filename: 'dist.js',
    library: 'MyLib'  // 设置了 library
  },
  plugins: [],
  optimization: {
    minimize: false  // 不压缩，压缩后代码不具可读性
  }
}
```
打包结果:
```javascript
var MyLib;
/******/ (() => { // webpackBootstrap

... 省略省略省略省略省略省略省略省略省略省略省略....

var __webpack_exports__ = {};
__webpack_require__.r(__webpack_exports__);
/* harmony export */ __webpack_require__.d(__webpack_exports__, {
/* harmony export */   "hello": () => (/* binding */ hello)
/* harmony export */ });
function hello() {
  console.log(123);
}
MyLib = __webpack_exports__;  // 实际上就是给 MyLib 进行赋值
/******/ })()
;
```
之后，我们可以这样来使用:
```javascript
<script src="dist.js"></script>

MyLib.hello()  // 将会打印 123
```
### 2.1 示例 1
我们改变 foo.js 的内容:
```javascript
// 将
export function hello() {
  console.log(123);
}

// 改为
```
那么最终打包结果将是:
```javascript
var MyLib;
/******/ (() => { // webpackBootstrap
var __webpack_exports__ = {};
function hello() {
  console.log(123);
}
MyLib = __webpack_exports__;
/******/ })()
;
```
也就是说，我们的 entry 需要导出对应的内容


## 3. 多入口：当 output.library 类型为 string[]
在以上的示例中，我们只有一个 entry 配置，但是 webpack 可以接受很多种类型的入口，比如 array 或者 object。


如果我们提供 array 类型的 entry，那么最终只会有一个 entry 生效:
```javascript
// foo.js
export function hello() {
  console.log('foo');
}

// bar.js
export function hello() {
  console.log('bar');
}


// webpack.config.js
const path = require('path');

module.exports = {
  entry: [path.join(__dirname, './src/foo.js'), path.join(__dirname, './src/bar.js')],  // 最终只有 bar 导出的值会生效
  output: {
    path: path.join(__dirname, "dist"),
    filename: 'dist.js',
    library: 'MyLib'
  },
  plugins: [],
  optimization: {
    minimize: false  // 不压缩，压缩后代码不具可读性
  }
}

// 最终打包结果
var MyLib;
/******/ (() => { // webpackBootstrap

... 省略省略省略省略省略省略省略省略省略省略省略....

var __webpack_exports__ = {};
// This entry need to be wrapped in an IIFE because it need to be isolated against other entry modules.
(() => {
var __webpack_exports__ = {};
// foo.js 没有被用到
function hello() {
  console.log('foo');
}
})();

// 最终只有 bar 会被用到
(() => {
__webpack_require__.r(__webpack_exports__);
/* harmony export */ __webpack_require__.d(__webpack_exports__, {
/* harmony export */   "hello": () => (/* binding */ hello)
/* harmony export */ });
function hello() {
  console.log('bar');
}
})();

MyLib = __webpack_exports__;
/******/ })()
;
```
而如果是用 object 的 entry，那么两个 entry 都会生效
```javascript
const path = require('path');

module.exports = {
  entry: {
    foo: path.join(__dirname, './src/foo.js'),
    bar: path.join(__dirname, './src/bar.js')
  },
  output: {
    filename: '[name].js',  // 此时会打包出 foo.js 和 bar.js
    library: ['MyLib', '[name]'], // name 是一个占位符
  },
  plugins: [],
  optimization: {
    minimize: false  // 不压缩，压缩后代码不具可读性
  }
}
```
此时应该这样使用:
```javascript
<script src="foo.js"></script>
<script src="bar.js"></script>

MyLib.foo.hello()  // 将会打印 foo
MyLib.bar.hello()  // 将会打印 bar
```
## 4. 当 output.library 类型为 object
ouput.library 有以下这些属性:

- name
- type
- export
- auxiliaryComment
- umdNamedDefine



type 的默认值是 `var` , 同时有以下这么多类型: `module` , `assign` , `assign-properties` , `this` , `window` , `self` , `global` , `commonjs` , `commonjs2` , `commonjs-module` , `amd` , `amd-require` , `umd` , `umd2` , `jsonp`  and `system` 
```javascript
// webpack.config.js
const path = require('path');

module.exports = {
  entry: path.join(__dirname, './src/foo.js'),
  output: {
    filename: 'dist.js',
    library: {
      name: 'MyLib',
      type: 'var'
    },
    // 以上的 library 设置和 library: 'MyLib' 是一样的
  },
  plugins: [],
  optimization: {
    minimize: false  // 不压缩，压缩后代码不具可读性
  }
}
```


### 4.1 直接暴露一个变量
以下这些 type 会将 entry 入口导出的值，直接赋值给 output.library.name
#### 4.1.1 type: 'var'
```javascript
// webpack.config.js
const path = require('path');

module.exports = {
  entry: path.join(__dirname, './src/foo.js'),
  output: {
    filename: 'dist.js',
    library: {
      name: 'MyLib',
      type: 'var'
    },
  },
  plugins: [],
  optimization: {
    minimize: false  // 不压缩，压缩后代码不具可读性
  }
}
```


此时编译的结果，简单看来就是:
```javascript
var MyLib;
(() => {
	MyLib = __webpack_exports__;
})()
```
#### 4.1.2 type: assign
```javascript
// webpack.config.js

const path = require('path');

module.exports = {
  entry: path.join(__dirname, './src/foo.js'),
  output: {
    filename: 'dist.js',
    library: {
      name: 'MyLib',
      type: 'assign'
    },
  },
  plugins: [],
  optimization: {
    minimize: false  // 不压缩，压缩后代码不具可读性
  }
}
```
打包结果和 `var` 的情况类似，区别在于没有 var MyLib 这个变量声明，而是直接赋值(**那么得小心，前提是 MyLib 这个变量在其他地方已经被定义了，否则可能会报错**)：
```javascript
// var MyLib; 没有这一行代码
(() => {
	MyLib = __webpack_exports__;
})()
```
#### 4.1.3 type: 'assign-properties'
和 type： assign 类似，区别在于**如果别的地方也没有定义 MyLib，那么也不会报错**。因此更加**安全**
```javascript
// 打包结果
MyLib = typeof MyLib === "undefined" ? {} : MyLib

(() => {
	MyLib = __webpack_exports__;
})()
```
### 4.2 通过给对象赋值，来暴露
以下这些 type 会将 entry 入口导出的值，赋值给一个特殊的对象，其中属性名为 output.library.name
#### 4.2.1 type: this
将 MyLib 赋值给 this，**至于 this 究竟是什么，取决于你**。
```javascript
// 打包结果
var __webpack_exports__ = {};
__webpack_require__.r(__webpack_exports__);
/* harmony export */ __webpack_require__.d(__webpack_exports__, {
/* harmony export */   "hello": () => (/* binding */ hello)
/* harmony export */ });
function hello() {
  console.log('foo');
}
this.MyLib = __webpack_exports__;
```
#### 4.2.2 type: window
将 MyLib 赋值给 window
```javascript
// 打包结果
var __webpack_exports__ = {};
__webpack_require__.r(__webpack_exports__);
/* harmony export */ __webpack_require__.d(__webpack_exports__, {
/* harmony export */   "hello": () => (/* binding */ hello)
/* harmony export */ });
function hello() {
  console.log('foo');
}
window.MyLib = __webpack_exports__;
```


#### 4.2.3 type: global
此时，取决于 webpack 的[ target 配置](https://webpack.js.org/configuration/target/)，生成的结果可能是 `self`, `global` 或者是 `globalThis` 
```javascript
// 打包结果
var __webpack_exports__ = {};
__webpack_require__.r(__webpack_exports__);
/* harmony export */ __webpack_require__.d(__webpack_exports__, {
/* harmony export */   "hello": () => (/* binding */ hello)
/* harmony export */ });
function hello() {
  console.log('foo');
}

// 具体的值取决于 webpack target 的值
self.MyLib = __webpack_exports__;
global.MyLib = __webpack_exports__;
globalThis.MyLib = __webpack_exports__;
```
#### 4.2.4 type: 'commonjs'
此时可以设置 name 或者不设置 name


设置 name 的情况:
```javascript
// webpack.config.js
const path = require('path');

module.exports = {
  entry: path.join(__dirname, './src/foo.js'),
  output: {
    filename: 'dist.js',
    library: {
      name: 'MyLib',
      type: 'commonjs'
    },
  },
  plugins: [],
  optimization: {
    minimize: false  // 不压缩，压缩后代码不具可读性
  }
}

// 打包结果
var __webpack_exports__ = {};
__webpack_require__.r(__webpack_exports__);
/* harmony export */ __webpack_require__.d(__webpack_exports__, {
/* harmony export */   "hello": () => (/* binding */ hello)
/* harmony export */ });
function hello() {
  console.log('foo');
}
exports.MyLib = __webpack_exports__;

// 使用
const A = require('./dist.js')
A.MyLib.hello()
```
不设置 name 的情况:
```javascript
// webpack.config.js
const path = require('path');

module.exports = {
  entry: path.join(__dirname, './src/foo.js'),
  output: {
    filename: 'dist.js',
    library: {
      type: 'commonjs'
    },
  },
  plugins: [],
  optimization: {
    minimize: false  // 不压缩，压缩后代码不具可读性
  }
}

// 打包结果
var __webpack_exports__ = {};
__webpack_require__.r(__webpack_exports__);
/* harmony export */ __webpack_require__.d(__webpack_exports__, {
/* harmony export */   "hello": () => (/* binding */ hello)
/* harmony export */ });
function hello() {
  console.log('foo');
}
exports.MyLib = __webpack_exports__;

// 使用
const A = require('./dist.js')
A.hello()
```
### 4.3 模块系统
以下这些参数都是为了兼容不同类型的模块系统，不同的 type 之下，name 有不同的含义。


#### 4.3.1 type: module, (ESM 模块)
将代码转为 ESM 模块
```javascript
// webpack.config.js
const path = require('path');

module.exports = {
  entry: path.join(__dirname, './src/foo.js'),
  experiments: {  // 由于这个特性依然是试验阶段的，因此需要设置这个
    outputModule: true,
  },
  output: {
    filename: 'dist.js',
    library: {
      // 不需要设置 name
      type: 'module'
    },
  },
  plugins: [],
  optimization: {
    minimize: false  // 不压缩，压缩后代码不具可读性
  }
}
```
打包结果:
```javascript
var __webpack_exports__ = {};
/* harmony export */ __webpack_require__.d(__webpack_exports__, {
/* harmony export */   "o": () => (/* binding */ hello)
/* harmony export */ });
function hello() {
  console.log('foo');
}
var __webpack_exports__hello = __webpack_exports__.o;
export { __webpack_exports__hello as hello };
```
#### 4.3.2 type: commonjs2 (CommonJS 模块)
将值赋值给 `module.exports` , 这就是 NodeJS 中 CommonJS 的模块类型
```javascript
// 打包结果
var __webpack_exports__ = {};
__webpack_require__.r(__webpack_exports__);
/* harmony export */ __webpack_require__.d(__webpack_exports__, {
/* harmony export */   "hello": () => (/* binding */ hello)
/* harmony export */ });
function hello() {
  console.log('foo');
}
module.exports = __webpack_exports__;

// 使用
const a = require('./dist')
console.log(a.hello());
```
你也可以设置 output.library.name, 此时 entry 的返回值会赋值给 `module.exports[output.library.name]` 
```javascript
// webpack.config.js
const path = require('path');

module.exports = {
  entry: path.join(__dirname, './src/foo.js'),
  output: {
    filename: 'dist.js',
    library: {
      name: 'MyLib',  // 设置 name
      type: 'commonjs2'
    },
  },
  plugins: [],
  optimization: {
    minimize: false  // 不压缩，压缩后代码不具可读性
  }
}

// 打包结果
var __webpack_exports__ = {};
__webpack_require__.r(__webpack_exports__);
/* harmony export */ __webpack_require__.d(__webpack_exports__, {
/* harmony export */   "hello": () => (/* binding */ hello)
/* harmony export */ });
function hello() {
  console.log('foo');
}
module.exports.MyLib = __webpack_exports__;

// 使用
const a = require('./dist')
console.log(a.MyLib.hello());
```
#### 4.3.3 type: 'amd'
打包结果:
```javascript
define("MyLib", [], () => { return /******/ (() => { // webpackBootstrap
var __webpack_exports__ = {};
__webpack_require__.r(__webpack_exports__);
/* harmony export */ __webpack_require__.d(__webpack_exports__, {
/* harmony export */   "hello": () => (/* binding */ hello)
/* harmony export */ });
function hello() {
  console.log('foo');
}
/******/ 	return __webpack_exports__;
/******/ })()
;
});;
```
#### 4.3.4 type: 'umd'
UMD 模块类型在 CommonJS，AMD 和 全局变量的情况下，都可以使用。
```javascript
// 打包结果
(function webpackUniversalModuleDefinition(root, factory) {
	if(typeof exports === 'object' && typeof module === 'object')
		module.exports = factory();
	else if(typeof define === 'function' && define.amd)
		define([], factory);
	else if(typeof exports === 'object')
		exports["MyLib"] = factory();
	else
		root["MyLib"] = factory();
})(self, function() {
return /******/ (() => { // webpackBootstrap
/************************************************************************/
var __webpack_exports__ = {};
__webpack_require__.r(__webpack_exports__);
/* harmony export */ __webpack_require__.d(__webpack_exports__, {
/* harmony export */   "hello": () => (/* binding */ hello)
/* harmony export */ });
function hello() {
  console.log('foo');
}
/******/ 	return __webpack_exports__;
/******/ })()
;
});
```


如果忽略名字，那么 entry 返回的所有值，会循环的直接赋值给 root:
```javascript
(function webpackUniversalModuleDefinition(root, factory) {
	if(typeof exports === 'object' && typeof module === 'object')
		module.exports = factory();
	else if(typeof define === 'function' && define.amd)
		define([], factory);
	else {
		var a = factory();
    // 直接赋值给 root
		for(var i in a) (typeof exports === 'object' ? exports : root)[i] = a[i];
	}
})(self, function() {
});
```
你也可以给每个模块类型，都单独指定名字:
```javascript
// webpack.config.js
const path = require('path');

module.exports = {
  entry: path.join(__dirname, './src/foo.js'),
  output: {
    filename: 'dist.js',
    library: {
      name: {
        root: 'MyLibrary',
        amd: 'my-library',
        commonjs: 'my-common-library',
      },
      type: 'umd'
    },
  },
  plugins: [],
  optimization: {
    minimize: false  // 不压缩，压缩后代码不具可读性
  }
}

// 打包结果
(function webpackUniversalModuleDefinition(root, factory) {
	if(typeof exports === 'object' && typeof module === 'object')
		module.exports = factory();
	else if(typeof define === 'function' && define.amd)
		define([], factory);
	else if(typeof exports === 'object')
    // 各自独立的名字
		exports["my-common-library"] = factory();
	else
    // 各自独立的名字
		root["MyLibrary"] = factory();
})(self, function() {
return /******/ (() => { // webpackBootstrap
var __webpack_exports__ = {};
__webpack_require__.r(__webpack_exports__);
/* harmony export */ __webpack_require__.d(__webpack_exports__, {
/* harmony export */   "hello": () => (/* binding */ hello)
/* harmony export */ });
function hello() {
  console.log('foo');
}
/******/ 	return __webpack_exports__;
/******/ })()
;
});
```
#### 4.3.5 type: system 略
#### 4.3.6 type: jsonp
```javascript
// 打包结果
MyLib(/******/ (() => { // webpackBootstrap
/************************************************************************/
var __webpack_exports__ = {};
__webpack_require__.r(__webpack_exports__);
/* harmony export */ __webpack_require__.d(__webpack_exports__, {
/* harmony export */   "hello": () => (/* binding */ hello)
/* harmony export */ });
function hello() {
  console.log('foo');
}
/******/ 	return __webpack_exports__;
/******/ })()
);
```
此时 MyLib 是未定义的，因此我们需要这样使用:
```html
  <script>
    function MyLib (a) {
      // a 就是模块的结果
      a.hello()
    }
  </script>
  <script src="./dist/dist.js"></script>
```
## 5. output.library.export 的含义
在前面的示例中:
```javascript
// webpack.config.js
const path = require('path');

module.exports = {
  entry: path.join(__dirname, './src/foo.js'),
  output: {
    filename: 'dist.js',
    library: {
      name: 'MyLib',
      type: 'var',
    },
  },
  plugins: [],
  optimization: {
    minimize: false  // 不压缩，压缩后代码不具可读性
  }
}

// foo.js
export function hello() {
  console.log('foo');
}

// 打包结果为
var __webpack_exports__ = {};
__webpack_require__.r(__webpack_exports__);
/* harmony export */ __webpack_require__.d(__webpack_exports__, {
/* harmony export */   "hello": () => (/* binding */ hello)
/* harmony export */ });
function hello() {
  console.log('foo');
}
MyLib = __webpack_exports__;

// 使用
MyLib.hello()
```
而通过配置 output.library.export, 我们就可以:
```javascript
// webpack.config.js
const path = require('path');
module.exports = {
  entry: path.join(__dirname, './src/foo.js'),
  output: {
    filename: 'dist.js',
    library: {
      name: 'MyLib',
      type: 'var',
      export: 'hello'
    },
  }
}

// 打包结果为
var __webpack_exports__ = {};
/* harmony export */ __webpack_require__.d(__webpack_exports__, {
/* harmony export */   "hello": () => (/* binding */ hello)
/* harmony export */ });
function hello() {
  console.log('foo');
}
MyLib = __webpack_exports__.hello;

// 使用 
MyLib()
```


我们还可以看一下以下这种情况：
```javascript
// webpack.config.js
const path = require('path');

module.exports = {
  entry: path.join(__dirname, './src/foo.js'),
  output: {
    filename: 'dist.js',
    library: {
      name: 'MyLib',
      type: 'var',
      export: 'default'  // 使用
    },
  },
  plugins: [],
  optimization: {
    minimize: false  // 不压缩，压缩后代码不具可读性
  }
}
// foo.js
export default function hello() {
  console.log('foo');
}

// 打包结果为
var __webpack_exports__ = {};
__webpack_require__.r(__webpack_exports__);
/* harmony export */ __webpack_require__.d(__webpack_exports__, {
/* harmony export */   "default": () => (/* binding */ hello)   // 当 export default 时，这里的 key 就变成了 default
/* harmony export */ });
function hello() {
  console.log('foo');
}
MyLib = __webpack_exports__.default;

// 使用
MyLib()
```
我们可以注意到，当 foo.js 使用 export default 时，打包结果的 `__webpack_exports__`  对应函数的 key 就变成了 default。


另外，output.library.export 还可以是一个数组:
```javascript
// webpack.config.js
const path = require('path');

module.exports = {
  entry: path.join(__dirname, './src/foo.js'),
  output: {
    filename: 'dist.js',
    library: {
      name: 'MyLib',
      type: 'var',
      export: ['default', 'aa']  // 设置数组
    },
  },
  plugins: [],
  optimization: {
    minimize: false  // 不压缩，压缩后代码不具可读性
  }
}

// foo.js
export default {
  aa: function hello() {
    console.log('foo');
  }
}

// 打包结果:
var __webpack_exports__ = {};
/* harmony export */ __webpack_require__.d(__webpack_exports__, {
/* harmony export */   "default": () => (__WEBPACK_DEFAULT_EXPORT__)
/* harmony export */ });
/* harmony default export */ const __WEBPACK_DEFAULT_EXPORT__ = ({
  aa: function hello() {
    console.log('foo');
  }
});
MyLib = __webpack_exports__.default.aa;

// 使用方式
MyLib()
```
## 6. output.library.auxiliaryComment
在 UMD 的情况下，添加辅助注释:
```javascript
// webpack.config.js
const path = require('path');

module.exports = {
  entry: path.join(__dirname, './src/foo.js'),
  output: {
    filename: 'dist.js',
    library: {
      name: 'MyLib',
      type: 'umd',
      export: 'default',
      auxiliaryComment: '我是注释'
    },
  },
  plugins: [],
  optimization: {
    minimize: false  // 不压缩，压缩后代码不具可读性
  }
}

// 打包结果
(function webpackUniversalModuleDefinition(root, factory) {
	//我是注释
	if(typeof exports === 'object' && typeof module === 'object')
		module.exports = factory();
	//我是注释
	else if(typeof define === 'function' && define.amd)
		define([], factory);
	//我是注释
	else if(typeof exports === 'object')
		exports["MyLib"] = factory();
	//我是注释
	else
		root["MyLib"] = factory();
})(self, function() {
});
```
如果想更精确的控制，那么可以使用对象:
```javascript
// webpack.config.js
const path = require('path');
module.exports = {
  entry: path.join(__dirname, './src/foo.js'),
  output: {
    filename: 'dist.js',
    library: {
      name: 'MyLib',
      type: 'umd',
      export: 'default',
      auxiliaryComment: {
        root: '我是 root 注释',
        commonjs: '我是 CommonJS 注释',
        commonjs2: '我是 CommonJS2 注释',
        amd: '我是 AMD 注释',
      },
    },
  },
  plugins: [],
  optimization: {
    minimize: false  // 不压缩，压缩后代码不具可读性
  }
}

// 打包结果
(function webpackUniversalModuleDefinition(root, factory) {
	//我是 CommonJS2 注释
	if(typeof exports === 'object' && typeof module === 'object')
		module.exports = factory();
	//我是 AMD 注释
	else if(typeof define === 'function' && define.amd)
		define([], factory);
	//我是 CommonJS 注释
	else if(typeof exports === 'object')
		exports["MyLib"] = factory();
	//我是 root 注释
	else
		root["MyLib"] = factory();
})(self, function() {
});
```
## 7. output.library.umdNamedDefine
该配置用于给 AMD 添加名字。我们会注意到，当我们打包 UMD 时，似乎 AMD 是没有名字的: `define([], factory); ` 
```javascript
// 打包结果
(function webpackUniversalModuleDefinition(root, factory) {
	//我是 CommonJS2 注释
	if(typeof exports === 'object' && typeof module === 'object')
		module.exports = factory();
	//我是 AMD 注释
	else if(typeof define === 'function' && define.amd)
		define([], factory);                                    // 没有名字
	//我是 CommonJS 注释
	else if(typeof exports === 'object')
		exports["MyLib"] = factory();
	//我是 root 注释
	else
		root["MyLib"] = factory();
})(self, function() {
});
```
那是因为我们缺少了 `umdNamedDefine: true` 的配置:
```javascript
// webpack.config.js
const path = require('path');
module.exports = {
  entry: path.join(__dirname, './src/foo.js'),
  output: {
    filename: 'dist.js',
    library: {     // 分别设置不同的名字
      name: {
        root: 'MyLib-Root',
        amd: 'MyLib-Amd',
        commonjs: 'MyLib-CommonJS',
      },
      type: 'umd',
      umdNamedDefine: true,
    },
  },
  plugins: [],
  optimization: {
    minimize: false  // 不压缩，压缩后代码不具可读性
  }
}

// 打包结果
(function webpackUniversalModuleDefinition(root, factory) {
	if(typeof exports === 'object' && typeof module === 'object')
		module.exports = factory();
	else if(typeof define === 'function' && define.amd)
		define("MyLib-Amd", [], factory);                 // 此时就有名字了
	else if(typeof exports === 'object')
		exports["MyLib-CommonJS"] = factory();
	else
		root["MyLib-Root"] = factory();
})(self, function() {
});
```
## 8. 即将会过时的配置
以下两个配置都是即将不再支持的参数，可以用新的配置来替换

- output.libraryExport：可以使用 output.library.export 来替换它
- output.libraryTarget: 可以使用 output.library.type 来替换它



