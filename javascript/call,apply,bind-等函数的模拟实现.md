## 前言

网上的面试题通常都会涉及各种函数的模拟实现，一开始的态度都是“嗤之以鼻”，要么觉得没意义(实际工作中都是用成熟开源的代码)，要么觉得这样做只是为了迎合面试。  

但是最近查看相关资料的时候，才意识到其实不单纯是应付面试，另一方面这也是对于 JavaScript 基础的考察。

## 1. call
指定 `this` 和 若干参数值的情况下，调用某个函数或者方法  

参考: https://github.com/mqyqingfeng/Blog/issues/11  

注意:
- 函数具有返回值

```js
Function.prototype.call = function(...args){
    const context = args[0] || window;
    const arg = args.slice(1);
    context.fn = this;
    const result = context.fn(...arg);
    delete context.fn;
    return result;
}
```

## 2. apply
指定 `this` 和 若干参数值的情况下，调用某个函数或者方法，区别于 call, 参数值以数组形式提供  

参考: https://github.com/mqyqingfeng/Blog/issues/11  

注意:
- 函数具有返回值

```js
Function.prototype.apply = function(context, args = []) {
    context = context || window;
    context.fn = this;
    const result = context.fn(...args);
    delete context.fn;
    return result;
}
```

## 3. bind

指定 `this` 以及若干参数值的情况下，返回一个新的函数  

参考: https://github.com/mqyqingfeng/Blog/issues/12  

注意：
- bind时接受的额外参数，应该作为实际调用时的 “前参数”
- 如果调用bind的不是函数，应该报错
- 如果bind之后的函数，别当作 Constructor 构造函数使用，那么应该使用构造函数返回的实例， 作为 this。
- 如果原来函数的 prototype 上具有方法或者属性，应该原样复制一份。同时，为了防止修改新的 prototype 会影响到原有函数的 prototype, 应该使用 NOOP 函数进行原型链的链接。

```js
Function.prototype.bind = function(...args){
    if(typeof this !== 'function') {
        throw new Error('Function.prototype.bind - what to be bound is not a function');
    }
    const fn = this;
    const context = args[0] || window;
    const arg = args.slice(1);
    const returnFun =  function(...innerArgs) {
        const newContext = context;
        if(this instanceof returnFun) {
            // it means, function is called as Constructor
            newContext = this;
        }
        return fn.apply(
            newContext,
            [].concat(arg, innerArgs)
        );
    }
    function NOOP() {}
    NOOP.prototype = this.prototype;
    returnFun.prototype = new NOOP();
    return returnFun;
}
```

## 4. new

参考: https://github.com/mqyqingfeng/Blog/issues/13

注意:  

- 返回对象的原型链需要连接到构造函数的prototype上
- 如果构造函数返回了对象，则直接返回该对象，否则返回新建的对象

```js
function objectFactory(Constructor, ...args) {
    const obj = {};
    const ret = Constructor.apply(obj,args)
    obj.__proto__ = Constructor.prototype;
    return typeof ret === 'object' ? ret : obj;
}
```

## 5. clone 

深拷贝浅拷贝  

注意:  
- 深拷贝的情况，如果属性值是对象，则进行递归

```js
function clone(obj, isDeep) {
    if(typeof obj !== 'object') {
        throw new Error('clone: the arguments[0] should be an object');
    }
    const newObj = obj instanceof Array ? [] : {};
    Object.keys(obj).forEach(key => {
        if(isDeep && typeof obj[key] === 'object') {
            newObj[key] = clone(obj[key])
        } else {
            newObj[key] = obj[key]
        }
    })
    return newObj;
}
```  

## 6. debounce

防抖，即 wait 时间之内多次触发函数，最终函数之后执行一次  

参考: https://github.com/mqyqingfeng/Blog/issues/22

注意:  
- setTimeout 中函数的 this 默认为 window 或者 undefined，因此需要先用 context 保存真正的 this 值。

```js
function debounce(fn, wait, isImmediate) {
    let timer = null;
    return function(...args) {
        const context = this;
        clearTimeout(timer);
        if(isImmediate) {
            const callNow = !timer;
            timer = setTimeout(function(){
                timer = null
            }, wait)
            if(callNow) {
                fn.apply(context, args)
            }
        } else {
            // this in setTimeout will be window or undefined
            time = setTimeout(function() {
                fn.apply(context, args)
            }, wait)    
        }
    }
}
```

## 7. throttle

节流，即频繁触发函数，每隔 wait 时间就会触发一次函数的执行  

参考: https://github.com/mqyqingfeng/Blog/issues/26  

实现方式:  

1. 时间戳法 

```js
function throttle(fn,wait) {
    let previous = 0;
    return function(...args) {
        const now = Date.now();
        const context = this;
        if(now - previous > wait) {
            fn.apply(context, args);
            previous = now;
        }
    } 
}
```

2. 定时器法

```js
function throttle(fn, wait) {
    let timer = null;
    return function(...args) {
        const context = this;
        if(!timer) {
            timer = setTimeout(() => {
                fn.apply(context, args);
                timer = null;
            }, wait)
        }
    }
}
```