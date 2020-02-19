## 前言

最早在看 `redux` 源码的时候，有个 applyMiddleware 函数，没能看明白(现在连它能干嘛的都忘了，衰)。只隐约记得，能够把一些 middleware 传入其中，然后它依次处理，顺序的执行一些逻辑(真忘了是干嘛的了)。  

再如 express 中，某个中间件的使用都是 `app.use(middleware)`, 而其内部具体干了什么还是不太清楚，最终呈现的效果是，当请求来的时候，依次的执行中间件，直到最后返回。  


那这两个知识盲点给我的感觉是，都有种 "**某个数组里存着函数,而里面的函数会依次执行**" 的感觉。之后，偶然看到这个[分享](https://www.yuque.com/online-sharing/activity/fm8or7)，说实话，觉得是能解决我的疑惑的，但还是绕不过来，理不清楚。  

这次抽空强行理解了一下，赶紧留住思路，免得跑了~

### 需求

想象我们有这样的一个数组:  

```js
const fun1 = (next) => {
  setTimeout(() => {
    console.log(1);
    next();
  }, 3000);
}

const fun2 = (next) => {
  setTimeout(() => {
    console.log(2);
    next();
  }, 2000);
}

const fun3 = (next) => {
  setTimeout(() => {
    console.log(3);
    next();
  }, 1000);
}

const funArr = [
  fun1,
  fun2,
  fun3
];


// 实现 run 方法，让数组中的函数依次执行  
function run() {
    
}
```

## 1. 自己的理解过程

### 1.1 正确分析
关键是 next 这个参数，我们需要的就是在 next 执行的时候，能够使得下一个函数执行，那么假如下一个函数就是 next, 会怎么样？   

先只有两个函数:  

`fun1(fun2)`  

当 fun1 执行完之后，fun2 作为 next 执行，看起来没有问题。  

但是！，当 fun2 执行的时候，fun2 的参数是 undefined，也就是说 fun2 的next 是 undefined ，那执行到 fun2 中的 next() 的时候就会报错。  

那我们就想，给 fun2 传一个空函数:  

```js
fun1(fun2(() => {}))
```

这样也会有问题，就是 `fun2(() => {}) ` 这是一个函数调用，当它作为参数的时候，首先会被计算出来。同时 `fun2(() => {}) `的返回值是 undefined, 又相当于 fun1 的参数是 undefined, 又会报错。

那么我们需要的就是，fun2 自身接受一个空函数，但是又不可以直接以 fun2() 的形式传给 fun1 ，那么我们考虑把 fun2 包裹在函数里:  

```js
fun1(() => fun2(() => {}))
```

那这样就顺利的实现了功能。  

那如果有三个函数的时候，我们自然可以推出以下的式子:  

```js
fun1(() => fun2(() => fun3(() => {})))
```

是的，但是，当函数个数不一定时，我们怎么来凑出这个式子呢？  

先直接上 reduce 吧(就是这么没有道理):  

先看看 reduce 的结构:

```js
arr.reduce((prev, curr) => {
    // code
}, initValue)
```

prev 为 initValue, 之后是 `code` 的返回值. curr 为当前迭代到的元素:  

回到我们的例子：  
当一个函数时:  

```js
fun1(() => {})
```

当两个函数时:  

```js
fun1(() => fun2(() => {}))
```

当三个函数时:  

```js
fun1(() => fun2(() => fun3(() => {})))
```

套用 reduce， 那就是:  

```js
function run(arr) {
    return arr.reduce((prev,curr) => {
        // 我们知道 curr 会依次为，fun1, fun2, fun3
        // 那 curr 为 fun1 时，我们为了凑出 fun1(() => {})
        // 因此让 prev 为 () => {}
        return () => curr(prev)
    }, () => {})
}

// 最终调用
run(funArr)()  // 因为 run 函数会返回一个函数
```

检验一下其余的:  
curr 为 fun1 时，run 函数最终返回:  

```js
() => fun1(() => {})     // prev
```
curr 为 fun2 时，prev 为 `() => fun1(() => {})`, run 函数最终返回:  

```js
() => fun2(() => fun1(() => {}))   // prev
```  

curr 为 fun3 时，prev 为 `() => fun2(() => fun1(() => {}))`, run 函数最终返回:  

```js
() => fun3(() => fun2(() => fun1(() => {})))
```

好像像那么回事，但是顺序好像反了，会先执行 fun3 而不是 fun1, 那我们换成 **reduceRight**:  

```js
function run(arr) {
    return arr.reduceRight((prev,curr) => {
        return () => curr(prev)
    }, () => {})
}
// 最终调用
run(funArr)()  // 因为 run 函数会返回一个函数
```

那结果应该是:  

```js
() => fun1(() => fun2(() => fun3(() => {})))
```

**最终实现了需求**  

### 1.2 易错  

在理解 1.1 上述过程中，我尝试了以下的代码:  

```js
function run(arr) {
    arr.reduce((prev,curr) => {
        return curr(prev)
    }, () => {})
}
```

以上代码(***在runJS***)会输出 3,2,1  于是，我以为我改成 reduceRight 就可以了，但是改成 reduceRight 之后，依旧是输出 3,2,1. 那是为什么呢？  

先转换以上代码:  

curr 为 fun1 时，prev 为 `() => {}`, 此时结果为:  

```js
fun1(() => {})   // prev
```

curr 为 fun2 时，prev 为 `fun1(() => {})`, 此时结果为:  

```js
fun2(fun1(() => {}))   // prev
```

curr 为 fun3 时，prev 为 `fun2(fun1(() => {}))`, 此时结果为:  

```js
fun3(fun2(fun1(() => {})))
```

**注意:**  

以上代码其实是会**报错**的！！！，如一开始分析的那样， fun1 会首先执行，返回值为 undefined, 当它作为 fun2 的参数时，其实是**报错了**，但是 runJS 居然没有报错。  

> 题外话，之后找到了 runjs 的 github，发现居然不开源，也发现一个有意思的讨论: [issue](https://github.com/lukehaas/RunJS/issues/30)  


以上的代码为什么还能输出结果呢？  

因为在不报错的情况下，以上代码相当于:  

```js
fun1()
fun2()
fun3()
```  

即相当于三个函数同时执行，因此最终的输出结果，**在 setTimtout 的时间不同的情况下，只和时间有关**。  

***因此也告诉我们，我们的这种场景, reduce 里面必须要返回函数***  

## 2. 更多方法

[参考](https://www.yuque.com/online-sharing/activity/fm8or7)

### 2.1 递归法

```js
function run(arr) {
  if(arr.length === 0) {
    return;
  }
  arr[0](() => run(arr.slice(1)))
}
```

其中 next 是个函数，相当于 `() => run(arr.slice(1))`， 每次 next 函数执行，都会执行数组的下一项。  

### 2.2 方法二

```js
function run(arr){
  const trigger = () => {
    if (arr.length === 0) {
      return;
    }
    arr.shift()();
  };
  arr = arr.map(fun => () => fun(trigger));
  trigger();
};
```

即 next 的功能就是执行数组的下一项，因此我们先定义 **trigger** 作为 next. 然后我们通过 map 函数，给每个函数外面包一层，当函数执行的时候，主动给它们传入 **trigger** 这个函数.

### 2.3 Promise 的方法

```js
function run(arr) {
  const trigger = () => {
    if (arr.length === 0) {
      return;
    }
    arr.shift()();
  };
  arr = arr.map(fun => () => new Promise((resolve) => {
    fun(resolve);
  }).then(trigger));
  trigger();
};
```

想一下，next 和 Promise 中的 resolve 是不是有点类似，就都是当执行的时候，会触发后续代码的执行。  

于是我们同样构造 **trigger** 函数，但这次，我们利用 Promise#resolve 之后，执行 .then 代码的特性，来触发 **trigger**  
*个人觉得这种方法和 2.2 其实是一样的，而且添加了 Promise, 增加了执行流程，不推荐*  

## 3. compose 函数

函数式编程中有个 compose 函数，这个函数组合几个函数，形成一个新的函数:  

```
const compose = ([f1, f2,f3]) => value => f1(f2(f3(value)));
```

即 compose 返回一个新的函数，该函数会被传入的函数,从右到左依次执行，同时右边函数的结果会作为左边函数的参数。  


```
function compose(funcs) {
  if (funcs.length === 0) {
    return arg => arg;
  }

  if (funcs.length === 1) {
    return funcs[0];
  }

  return funcs.reduce((a, b) => (...args) => a(b(...args)));
}
```

我们来看 reduce 的部分, 拆开是这样的:  

```js
funcs.reduce((prev,curr) => {
    return (...args) => {
        prev(curr(...args))
    }
})
```

*当 reduce 没有第二个参数 initValue 时，prev 即为 funcs[0], curr 即为 funcs[1]。*  

因此，当只有两个函数时, prev 为 `fun1`, curr 为 `fun2`, 最终返回的函数是:  

```js
(...args) => {              // prev
    fun1(fun2(...args))
}
```

当有三个函数时:  
prev 为:  
```js
(...args) => {
    fun1(fun2(...args))
}
```  
curr 为 `fun3`, 那么最终的结果就是:  

```js
(...args) => {
    (
        (...innerArgs) => {
            fun1(fun2(...innerArgs))
        }
    )
    (fun3(...args))
}

// 注意
以下是一个函数声明:
(
    (...innerArgs) => {
        fun1(fun2(...innerArgs))
    }
)

而 fun3(...args) 是上面函数的参数，因此最终替换过来就相当于:  

(...args) => {
    fun1(fun2(fun3(...args)))
}
```

### 3.1 示例

```js
function add1(number) {
    return number + 1
}

function multiply2(number) {
    return number * 2
}

function minus3(number) {
    return number - 3
}

const newFun = compose([add1,multiply2,minus3])
newFun(6)   // 输出 7 
// 7 = ((6 - 3) * 2) + 1
```

## 参考资料

1. 语雀: [来聊一道面试题吧](https://www.yuque.com/online-sharing/activity/fm8or7)
2. 知乎: [Node.js 的中间件里面的 next() 是干什么的？](https://www.zhihu.com/question/37693420/answer/176287446)