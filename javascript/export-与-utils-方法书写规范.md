- [use es6 import export in node](https://stackoverflow.com/questions/45854169/how-can-i-use-an-es6-import-in-node)

- [阮一峰 es6 module](http://es6.ruanyifeng.com/#docs/module)


## 1. export 命令

```js
export const firstName = 'Michael';
export const lastName = 'Jackson';
export const year = 1958;

// 等价于

const firstName = 'Michael';
const lastName = 'Jackson';
const year = 1958;

export { firstName, lastName, year };  // 优先考虑这种方式，写在文件尾部可以清晰看到输出了哪些接口
```
但是以上两种方式，外部都不能用 `import Utils from '***'` 的方式访问，因为文件没有用 `export default` 方式输出接口。因而只能用 `import {firstName, lastName, year}` 的方式访问。  


`export` 除了输出变量，还可以输出函数或类：  
```js
export function multiply(x, y) {
  return x * y;
};
```

`export` 命令规定对外的接口，如果暴露的是常量而不是接口,则会报错:  
```js
// error
const a = 1;
export a; 

// ok
export const a;

// error
function a () {
  console.log(123)
}
export a;

// ok
export function a () {
  console.log(123)
}
```

## 2. import 命令

`import` 命令接受一对"大括号",指定要从其它模块导入的变量名。该大括号不允许解构：  

```js
// a.js
export const A = {a:1}

// b.js
import { A:{a} } from './a.js'     // error
```

`import` 会执行所加载的模块,但是只会执行一次:  

```js
// main.js
console.log('i am main')
const A = 'main'
export default A

// utils1.js
import A from './main.mjs'
console.log('i am utils1')
export default () => {}

// utils2.js
import A from './main.mjs'
console.log('i am utils2')
export default () => {}

// index.js
import u1 from './utils1.mjs'
import u2 from './utils2.mjs'


// 打印
// i am main  // 即该模块被引用两次，但只会执行一次
// i ma utils1
// i ma utils2
```

## 3. 整体加载
```js
// utils.js
export const A = { a: 1 }
export const B = { b: 1 }

// index.js
import Utils from './utils.mjs';  // error, utils 没有 export default

import * as Utils from './utils.mjs';
// { A: { a: 1 }, B: { b: 1 } }, 即可以通过 * 指定一个对象，将所有值加载到该对象上
```

## 4. export default 命令
```js
// utils.js
export const A = { a: 1 }

// index.js  
import {A} from './utils.mjs';
// {a:1}

import * as A from './utils.mjs';
// {A:{a:1}}
```
即对于以上的形式，使用`import`加载模块需要知道模块内部的属性方法名，或者使用整体加载的形式。

而使用 `export default` 可以使得在 `import` 时指定任意名字:  

```js
// utils.js
const A = { a: 1 }
export default A

import A from './utils.mjs';
// {a:1}  即通过 export default 的形式，可以在不知道接口名的情况下，引入默认的接口
```

`export default` 用于指定模块的默认输出，因此一个模块只能有一个默认输出。  

`export default` 命令的本质是将后面的值，赋给 `default` 变量,所以:  

```js
// ok
export default 1
// error
export 1

// ok
const a = 1;
export default a;
// error
export default const a = 1;

// ok
export default function a(){}
// ok
function a(){}
export default a; 
```

## 5. Utils 推荐写法

```js
export default {
  a () {
    console.log('a')
  },
  b () {
    this.a()      // console.log('a') 
    console.log('b')
  }
}
```
即能在 `{}` 中通过 `this` 获取到当前的对象的别的 `Util` 方法.  

外部模块需要调用时只需 `import Utils from './utils.mjs'`