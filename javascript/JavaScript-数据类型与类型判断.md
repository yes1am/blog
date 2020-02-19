## 前言

JS 中的数据经常打交道，与之相关的数据类型，在平常开发中却不是很在意，只有在开发中遇到问题才会去搜索相关知识，而本身没有相关的知识储备。  

上周面试中，面试官问了几个问题，慢慢意识到自己基础并不扎实。  

Q1: JS 有哪些数据类型?  
A: `number`,`boolean`,`string`,`null`,`undefined`,`symbol`, `object`

Q2: 有哪些方法判断数据类型?  
A: `typeof`, `instanceof`  

Q3: 还有别的方法?  
A: ??? 还有嘛(疑问： Array.isArray 对，这个方法可以判断数组，那还有别的方法可以判断别的数据类型嘛？)  

Q4: 哪些是原始类型?  
A: `number`,`boolean`,`string`,`undefined`,`symbol`。`object` 不是原始类型。(疑问：那 `null` 呢? typeof `null` 也是 `object`, 函数呢，Date呢，正则呢,  那怎么判断某个类型是不是原始类型？)  

同时，最后面试官举了个数据类型的使用场景，在写组件的时候，写公共函数的时候，需要去校验参数的类型，如果参数不对，需要抛出对应的错误。  

的确，健壮性足够强的代码，应该要考虑到很多的情况，而这些的基础都要明确数据类型。  

## 1. 数据类型

*解决 Q1, 即以下八种数据类型，其中对象类型又包含函数对象，日期对象等*  

[MDN](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Data_structures) 中已经说明了，JS 中有 8 种数据类型(七种原始类型和对象类型):  

*解决 Q4, 即以下七种类型是原始类型，其他对象类型都不是原始类型*  

> 七种原始类型:  
1. number
2. string
3. boolean
4. null
5. undefined
6. bigint
7. symbol  

原始类型存储的都是值，是没有属性或者方法的。 如 `1.toString()` 会报错，因为 1 这个基础类型没有 `toString` 方法。  

注意:  `'1'.toString()` 是可以的, 是因为在调用 `toString` 方法的时候，字符串 '1' 已经转为 **字符串对象** 了，而对象是有方法的。  

> 对象类型: 除了原始类型，都是对象类型

1. 标准对象，键值对 {}
2. 函数对象
3. 日期，Date 对象
4. 数组，Array 对象
5. 正则, RegExp 对象
6. 错误，Error 对象

## 2. 数据类型

| typeof 表达式 | 结果 |
| --- | --- |
| typeof 1 | number |
| typeof '1' | string |
| typeof true | boolean |
| typeof undefined | undefined |
| typeof function() {} | function |
| typeof Symbol('a') | symbol |
| typeof null | object |
| typeof {} | object |
| typeof new Date() | object |
| typeof /1/ | object |
| typeof [] | object |  
| typeof new Error() | object |  

也就是说，typeof 可以检验出, `number`,`string`,`boolean`,`undefined`,`function`, `symbol` 类型，而对于 `null` 和 其他的对象类型数据，返回值都是 `object`，不能区分。  

那如何区分, null, 标准对象，函数对象，日期对象，数组对象,错误对象和正则对象呢？  

*解决 Q2,Q3, 即还可以使用 Object.prototype.toString() 方法来判断类型*  

答案是 `Object.prototype.toString()`  

| Object.prototype.toString.call() | 结果 |
| --- | --- |
| Object.prototype.toString.call(1) | [object Number] |
| Object.prototype.toString.call('1') | [object String] |
| Object.prototype.toString.call(true) | [object Boolean] |
| Object.prototype.toString.call(undefined) | [object Undefined] |
| Object.prototype.toString.call(function() {}) | [object Function] |
| Object.prototype.toString.call(Symbol(1)) | [object Symbol] |
| Object.prototype.toString.call(null) | [object Null] |
| Object.prototype.toString.call({}) | [object Object] |
| Object.prototype.toString.call(new Date()) | [object Date] |
| Object.prototype.toString.call(/1/) | [object RegExp] |
| Object.prototype.toString.call([]) | [object Array] |
| Object.prototype.toString.call(new Error()) | [object Error] |

接下来写一个 `type` 函数，该函数返回任意数据的数据类型。  

```js
var class2type = {};
// 生成class2type映射
"Number String Boolean Undefined Function Symbol Null Object Date RegExp Array Error".split(" ").map(function(item, index) {
    class2type["[object " + item + "]"] = item.toLowerCase();
})
function type(){
    return class2type[Object.prototype.toString.call(obj)]
}
```

至此，我们能判断原始类型以及一些对象类型。  

## 3. 额外的 Utils 方法

### 3.1 isWindow 
依据:  window 对象有 window 属性指向自身。  
```js
function isWindow(obj) {
    return obj !== null && obj === obj.window;
}
```

### 3.2 isEmptyObject

依据: 空对象上没有属性  
```js
function isEmptyObject( obj ) {
  return typeof obj === 'object' && obj !== null && !Object.keys(obj).length
}
```

### 3.3 isElement

依据: DOM 元素的 nodeType 为 1

```js
 function isElement(obj) {
    return !!(obj && obj.nodeType === 1);
};
```  

### 3.4 isValidDate (是否为有效的日期对象)

依据: `isNaN(new Date('foo')) === true`

```js
function isValidDate(d) {
  return d instanceof Date && !isNaN(d);
}
```

此外，我们可以再关注一下 Date 的一下奇怪行为:  

```js
new Date('a')   // Invalid Date  此时，返回值是个"不合法日期"对象
new Date('a') === 'Invalid Date'   // false
new Date('a').toString() === 'Invalid Date'   // true

new Date('a').valueOf()  // NaN  即，这个 "不合法日期对象" 转换为数字是为 NaN  

isNaN(new Date('a'))     // true  "不合法日期对象" 为 NaN
new Date('a') instanceof Date    // true，即 "不合法日期对象" 依然是 Date 的实例


const date = new Date('a')
date.getFullYear()   // NaN  // 即 "不合法日期对象" 上调用方法一律返回 NaN
date.getMonth()      // NaN
date.getDate()       // NaN

new Date(NaN,NaN,NaN);   // Invalid Date，即返回 "不合法日期对象" 
```

## 4. 数据类型与内存

在 JS 中，每一个数据都需要一个内存空间，而内存空间又被分为:  栈内存(stack)与堆内存(heap)。

`number`, `boolean`, `string`, `null`, `undefined`, `symbol` 这几种基础类型和**函数**，它们的值会存放在栈内存中。  

而引用类型数据，如 `Array` (所需内存大小不一样), 标准对象 `{}`, 它们的值存放在堆内存中。  

```js
var a = null; // 栈
// 变量 a 存放于栈中，它的值是 null, 也是存放在栈中

var b = { m: 20 }; 
// 变量 b 存放于栈中,它的值是对象的地址，地址也存放在栈中
// 而地址指向的 {m: 20} 作为对象存在于堆内存中
```

## 参考资料

1. [JavaScript专题之类型判断](https://github.com/mqyqingfeng/Blog/issues/28)
2. [10分钟了解JS堆、栈以及事件循环的概念](https://juejin.im/post/5b1deac06fb9a01e643e2a95)