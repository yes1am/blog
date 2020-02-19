## 1. 参考资料
1. [详解JavaScript中的Event Loop（事件循环）机制](https://zhuanlan.zhihu.com/p/33058983)  
2. [这一次，彻底弄懂 JavaScript 执行机制](https://juejin.im/post/59e85eebf265da430d571f89)  
3. [Event Loop的规范和实现](https://zhuanlan.zhihu.com/p/33087629)  


## 2. 整理
其中第二篇资料[这一次，彻底弄懂 JavaScript 执行机制](https://juejin.im/post/59e85eebf265da430d571f89),更具有说服力，能解释一部分的异步情况。  

### 2.1 [详解JavaScript中的Event Loop（事件循环）机制](https://zhuanlan.zhihu.com/p/33058983)

#### 2.1.1 知识点

js代码在执行时会将不同变量存放于内存中的不同位置，堆(heap)和栈(stack),其中堆存放着一些对象，栈存放一些基础类型变量以及对象的指针。  


执行栈 => 存放同步代码。  
事件队列 => 存放异步代码。  

当异步事件返回结果(比如定时器时间到期，ajax请求返回完数据)时，js会将异步事件的回调添加到事件队列中。而后在执行栈已经执行完毕情况之后，会查看事件队列是否有任务，如果有任务，则将任务放到执行栈中进行调用。

异步任务分为微任务(micro task)与宏任务(macro task)，对应会有微任务事件队列与宏任务事件队列:  
  宏任务：
  ```
  setInterval()
  setTimeout()
  ```
  微任务：  
  ```
  new Promise()
  new MutaionObserver()
  ```

>  当前执行栈执行完毕时会立刻处理所有微任务队列中的事件，然后再去宏任务队列取出一个事件，同一次事件循环中，微任务永远在宏任务之前执行  

#### 2.1.2 说明

文章后续还提到了 `node` 中 `libuv` 的事件循环模型，以及事件循环各个阶段的解释。  

### 2.2 [这一次，彻底弄懂 JavaScript 执行机制](https://juejin.im/post/59e85eebf265da430d571f89)

#### 2.2.1 知识点

1. 同步和异步任务进入不同的执行"场所"，同步任务进入主线程执行栈，异步任务进入 `Event Table` 并注册回调函数
2. 当异步任务指定的事件完成(定时器到时，请求 `success` )时，`Event Table` 会把回调函数转移到 `事件队列`(Event Queue)
3. 当主线程执行栈的任务执行完毕，会去事件队列读取函数，进入主线程执行
4. 上述过程不断重复，形成事件循环


setTimeout： 即执行到该代码时，会进入`Event Table`注册回调函数，当定时器到时之后，`Event Table`会将回调函数转移到`事件队列`。  

setInterval: 即执行到该代码时，会进入`Event Table`注册回调函数，之后每隔指定的时间，`Event Table`会将回调函数转移到`事件队列`。  


除了广义上的同步任务和异步任务，还会将任务划分为宏任务和微任务：  

1. macro-task(宏任务)：包括整体代码script，setTimeout，setInterval
2. micro-task(微任务)：Promise，process.nextTick

> 这个宏任务的划分比2.1资料中的划分多了一个整体代码script  

### 2.2.2 说明

这篇文章举例了一些异步代码的例子，便于理解。同时全文表达出 [执行一个宏任务,执行当前所有微任务]为一个 `event-loop`。  

## 2.3 [Event Loop的规范和实现](https://zhuanlan.zhihu.com/p/33087629)

### 2.3.1 知识点

~~`process.nextTick` 注册的函数优先级高于 `Promise`, 即微任务从 `Event Table` 转移到事件队列并不完全根据代码中出现的顺序，还会有优先级的区别。~~  
在评论区中发现以下说法：  
> process.nextTick 是放到当前同步执行栈的尾部，是一定比异步的任务队列早的。并不是因为优先级高于其他异步的任务。同步的执行栈到异步任务队列，process.nextTick 就是处于中间的那个。  
> 当然异步下，任务队列里就分宏任务 / 微任务。但是我也觉得这里不是由优先级决定的，而是宏任务内的执行栈清空后，再清空当前宏任务内的微任务。接着以此循环再去清空下一个宏任务的执行栈，再清空微任务。

这个说法就相当于说，`process.nextTick`是在同步执行栈尾部，在执行异步任务队列之前的。就是说它是一个在所有其它异步任务之前的异步任务。(有空再翻资料)


```
console.log(1)

setTimeout(() => {
    console.log(2)
    new Promise(resolve => {
        console.log(4)
        resolve()
    }).then(() => {
        console.log(5)
    })
    process.nextTick(() => {
        console.log(3)
    })
})


process.nextTick(() => {   // 位置1
    console.log(6)
})

new Promise(resolve => { 
    console.log(7)
    resolve()
}).then(() => {
    console.log(8)
})

process.nextTick(() => {   // 位置2
    console.log(6)
})
```

即process.nextTick这段代码不管是在位置1还是位置2，输出结果都是`1,7,6,8,2,4,3,5`  

同时以下这段代码的返回结果是不确定的：  
```
console.log(1)

setTimeout(() => {
    console.log(2)
    new Promise(resolve => {
        console.log(4)
        resolve()
    }).then(() => {
        console.log(5)
    })
    process.nextTick(() => {
        console.log(3)
    })
})

new Promise(resolve => {
    console.log(7)
    resolve()
}).then(() => {
    console.log(8)
})

process.nextTick(() => {
    console.log(6)
})

setTimeout(() => {
    console.log(9)
    process.nextTick(() => {
        console.log(10)
    })
    new Promise(resolve => {
        console.log(11)
        resolve()
    }).then(() => {
        console.log(12)
    })
})
```
有可能返回`1,7,6,8,2,4,3,5,9,11,10,12`,但也可能返回`1、7、6、8、2、4、9、11、3、10、5、12`,即两个setTimeout的执行可能混合。  

原因在于:  
```
setTimeout(() => { // 1

})

setTimeout(() => { // 2
  
})
```
这两段 `setTimeout` 都没有指定 `timeout`时间，默认是0,延时相同，被合并进同一个`expired timers queue`而一起执行。  

> 当做合并为同一个宏任务？,那为啥还会有两种可能的结果？一直返回`1、7、6、8、2、4、9、11、3、10、5、12` 才对吧，这一块还没有理解  

### 2.3.2 说明

这篇文章是在阅读 2.2[这一次，彻底弄懂 JavaScript 执行机制](https://juejin.im/post/59e85eebf265da430d571f89)之后写的，因此很适合和 `2.2` 连着一起看，同时还翻阅了 `blink` 与 `node`的源码解释了定时器`0ms和1ms为什么效果是一致的`，很具说服力。

### 2.4 代码解析

*示例一*
```
setTimeout(() => {
  console.log("clock1")
  setTimeout(() => {
    console.log('clock3')
  },200)
},1000)

setTimeout(() => {
  console.log("clock2")
},2000)
```

以上这段代码最终打印`clock1, clock3, clock2`.  

针对这种嵌套调用的情况，可以根据[这一次，彻底弄懂 JavaScript 执行机制](https://juejin.im/post/59e85eebf265da430d571f89)去理解: 

```
首先全局代码执行
1. 遇到异步任务，1000的定时器，加入到event table
2. 遇到异步任务，2000的定时器，加入到event table
3. 没有微任务，则进行事件循环
4. 进行到1000ms，将1000的定时器，加入到事件队列，且为宏任务
5. 此时宏任务中有1000ms这个事件
6. 没有微任务，执行事件循环
7. 有同步任务，则打印 clock1,
8. 有异步任务，将200ms的定时器加入到 event table
9. 没有微任务，则进行事件循环
10. 到了1200ms，将200的定时器，加入到事件队列，且为宏任务
11. 此时宏任务中有200ms这个事件
12. 没有微任务，执行事件循环
13. 有同步任务，则打印 clock3
14. 没有微任务，则进行事件循环
15. 到了2000ms，将2000的定时器，加入到事件队列，且为宏任务
16. 此时宏任务中有2000ms这个事件
17. 没有微任务，则进行事件循环
18. 有同步任务，则打印 clock2
```

*示例二*
```
setTimeout(() => {
  console.log("clock1")
  setTimeout(() => {
    console.log('clock3')
  },0)
},0)

setTimeout(() => {
  console.log("clock2")
},0);
```
以上这段代码始终打印`clock1, clock2, clock3`

```
首先全局代码执行
1. 遇到异步任务，0的定时器，加入到event table
2. 遇到异步任务，0的定时器，加入到event table
3. 没有微任务，则进行事件循环
4. 进行到0ms，将两个0的定时器，加入到事件队列，且为宏任务
5. 此时宏任务中有两个0ms的事件
6. 没有微任务，执行事件循环
7. 执行第一个宏任务，(第二个宏任务暂时不执行)，其中有同步任务，则打印 clock1,
8. 有异步任务，将0ms的定时器加入到 event table
9. 没有微任务，则进行事件循环
10. 到了0ms，将0ms的定时器，加入到事件队列，且为宏任务
11. 此时宏任务中有【之前剩余的宏任务二，当前】
12. 没有微任务，执行事件循环
13. 有同步任务，则打印 clock2
14. 没有微任务，执行事件循环
15. 有同步任务，则打印 clock3
```