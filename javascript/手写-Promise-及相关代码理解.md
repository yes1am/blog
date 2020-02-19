## 前言
Promise 在日常开发中经常使用，但内部原理不太 理解，网上常看见一些源码的解析，得空了，便想看看相关的实现，对于之后的开发必然是有帮助的。  

**该理解为阅读他人博客之后的精简部分，用于本人理解，不一定适用于初学者。**

## 1. 分析

### 1.1 极精简版

首先，我们看下 Promise 是如何使用的  

```js
new Promise((resolve, reject) => {
    resolve('value')
})
.then(res => {
    // 处理 resolve
}, err => {
    // 处理 reject
})
```

我们知道 `Promise` 有三个状态，默认为 `pending`, 还有 `resolved` 和 `rejected`.  

由以上使用方式可知，Promise 是一个可以被 `new` 进行实例化的类，该类初始化接受一个函数，函数具有两个参数，一个函数将 `Promise` 状态变为 `resolved`, 一个将状态转变为 `rejected`。  

实例之后具有 `.then` 方法，该方法同样可以接受两个参数，分别处理`resolved`和 `rejected` 之后的 `Promise`。再依据 Promises/A+ 规范， ，我们可以写出以下代码：  

```js
// 常用字符串用变量代替
const PENDING = 'pending'
const RESOLVED = 'resolved'
const REJECTED = 'rejected'

class PPromise {
  constructor(executor){
    this.state = PENDING;
    this.value = null;
    this.error = null;
    this.resolve = this.resolve.bind(this);
    this.reject = this.reject.bind(this);
    executor(this.resolve,this.reject)
  }
  resolve(value) {
    if(this.state === PENDING) {
      this.state = RESOLVED;
      this.value = value;
    }
  }
  reject(error) {
    if(this.state === PENDING) {
      this.state = REJECTED;
      this.error = error;
    }
  }
  then(onFulFilled, onRejected) {
    onFulFilled = typeof onFulFilled === 'function' ? onFulFilled : v => v
    onRejected = typeof onRejected === 'function' ? onRejected : r => {
        throw r
    }
    if (this.state === RESOLVED) {
        onFulFilled(this.value)
    }
    if (this.state === REJECTED) {
        onRejected(this.error)
    }
  }
}

// 测试代码
new PPromise((resolve, reject) => {
  resolve(456)
})
.then(res => {
  console.log(res)
},err => {
  console.log(err)
})
```
以上代码，即简单的实现了 `Promise` 的功能。

但是，经常我们在 Promise 的构造函数中，并非是立即进行 `resolve` 或者 `reject`, 而是存在异步 `resolve` 的情况,于是我们继续改进.  

### 1.2 支持异步

其实支持异步的逻辑也很简单，即当 `.then` 执行的时候，Promise 的状态还是 `pending`。  

这时候，不能立即执行 onFulFilled或者onRejected方法，那我们可以将这两个函数保存起来，利用回调，在真正 resolve 的时候执行这两个函数:  

```js
const PENDING = 'pending'
const RESOLVED = 'resolved'
const REJECTED = 'rejected'

class PPromise {
  constructor(executor){
    this.state = PENDING;
    this.value = null;
    this.error = null;
    // 两个数组存放对应的回调
    this.onResolvedCallback = []
    this.onRejectedCallback = []
    this.resolve = this.resolve.bind(this);
    this.reject = this.reject.bind(this);
    executor(this.resolve,this.reject)
  }
  resolve(value) {
    if(this.state === PENDING) {
      this.state = RESOLVED;
      this.value = value;
      // 执行每个回调
      this.onResolvedCallback.forEach(fun => {
        fun(this.value)
      })
    }
  }
  reject(error) {
    if(this.state === PENDING) {
      this.state = REJECTED;
      this.error = error;
      // 执行每个回调
      this.onRejectedCallback.forEach(fun => {
        fun(this.error)
      })
    }
  }
  then(onFulFilled, onRejected) {
    onFulFilled = typeof onFulFilled === 'function' ? onFulFilled : v => v
    onRejected = typeof onRejected === 'function' ? onRejected : r => {
        throw r
    }
    if (this.state === RESOLVED) {
        onFulFilled(this.value)
    }
    if (this.state === REJECTED) {
        onRejected(this.error)
    }
    
    if (this.state === PENDING) {
        // 存回调函数
        this.onResolvedCallback.push(onFulFilled)
        this.onRejectedCallback.push(onRejected)
    }
  }
}

// 测试代码
new PPromise((resolve, reject) => {
  setTimeout(() => {
    resolve(456)
  }, 200);
})
.then(res => {
  console.log(res)
},err => {
  console.log(err)
})
```
此时即能兼容异步的情况  

### 1.3 .then方法异步
我们看下如下的例子:   

```js
console.log(1)
new Promise((resolve,rejected) => {
    console.log(2)
    resolve(4)
})
.then(res => {
    console.log(res)
})
console.log(3)
```
我们知道以上的代码会输出 `1,2,3,4`，而我们目前的代码则会输出`1,2,4,3`  


即 .then 方法是异步而非同步执行，因此`3`会在`4`之前打印，因此我们可以这样处理，将 `resolve` 和 `reject` 代码变为异步的:  

```js
const PENDING = 'pending'
const RESOLVED = 'resolved'
const REJECTED = 'rejected'

class PPromise {
  constructor(executor){
    this.state = PENDING;
    this.value = null;
    this.error = null;
    this.onResolvedCallback = []
    this.onRejectedCallback = []
    this.resolve = this.resolve.bind(this);
    this.reject = this.reject.bind(this);
    executor(this.resolve,this.reject)
  }
  resolve(value) {
    // 异步执行
    setTimeout(() => {
      if(this.state === PENDING) {
        this.state = RESOLVED;
        this.value = value;
        this.onResolvedCallback.forEach(fun => {
          fun(this.value)
        })
      }
    }, 0);
  }
  reject(error) {
    // 异步执行
    setTimeout(() => {
      if(this.state === PENDING) {
        this.state = REJECTED;
        this.error = error;
        this.onRejectedCallback.forEach(fun => {
          fun(this.error)
        })
      }
    }, 0);
  }
  then(onFulFilled, onRejected) {
    onFulFilled = typeof onFulFilled === 'function' ? onFulFilled : v => v
    onRejected = typeof onRejected === 'function' ? onRejected : r => {
        throw r
    }
    if (this.state === RESOLVED) {
        onFulFilled(this.value)
    }
    if (this.state === REJECTED) {
        onRejected(this.error)
    }
    
    if (this.state === PENDING) {
        // 存回调函数
        this.onResolvedCallback.push(onFulFilled)
        this.onRejectedCallback.push(onRejected)
    }
  }
}

// 测试代码
console.log(1)
new PPromise((resolve,rejected) => {
    console.log(2)
    resolve(4)
})
.then(res => {
    console.log(res)
})
console.log(3)
```  

### 1.4 链式调用
我们知道 Promise 中 `.then` 出的结果还可以继续调用 `.then` 方法，`Promises/A+` 规范规定 `Promise.then` 必须返回一个 Promise 对象,因此改写 `then` 方法:  

```js
then(onFulFilled, onRejected) {
// 返回一个新的 Promise
    return new PPromise((resolve, reject) => {
        onFulFilled = typeof onFulFilled === 'function' ? onFulFilled : v => v
        onRejected = typeof onRejected === 'function' ? onRejected : r => {
            throw r
        }
        if (this.state === RESOLVED) {
            onFulFilled(this.value)
        }
        if (this.state === REJECTED) {
            onRejected(this.error)
        }
        
        if (this.state === PENDING) {
            this.onResolvedCallback.push(onFulFilled)
            this.onRejectedCallback.push(onRejected)
        }
    }
})
```
由于 Promise 的构造函数会立即执行，因此不影响原有的代码  

1. `.then` 返回普通值

考虑以下的例子:  
```js
new Promise((resolve,reject) => {
    resolve(1)
})
.then(res => {
    return res + '2'
})
.then(res => {
    console.log(res)  // 打印 12
})

new Promise((resolve,reject) => {
    resolve(1)
})
.then(res => {
    console.log(res)  // 打印 1
})
.then(res => {
    console.log(res)   // 打印 undefined
})
```

即链式调用中，下一个的 `then` 会接收上一个 `then` 中 `return` 的结果，如果没有 `return`, 则接收 `undefined`  

2. `.then` 返回Promise

```js
new Promise((resolve,reject) => {
    resolve(1)
})
.then(res => {
    return new Promise((resolve, reject) => {
        resolve(2)
    })
})
.then(res => {
    console.log(res);   // 打印2
})
```
即在链式调用中，如果上一个 `then` 返回 Promise，那么下一个 `then` 会接收上一个 `Promise`中`resolve`或者`reject`的值。  


再次进行完善:  
```js
const PENDING = 'pending'
const RESOLVED = 'resolved'
const REJECTED = 'rejected'

const Promises = [];

class PPromise {
  constructor(executor) {
    this.state = PENDING;
    this.value = null;
    this.error = null;
    this.onResolvedCallback = []
    this.onRejectedCallback = []
    this.resolve = this.resolve.bind(this);
    this.reject = this.reject.bind(this);
    executor(this.resolve, this.reject)
  }
  resolve(value) {
    setTimeout(() => {
      if (this.state === PENDING) {
        this.state = RESOLVED;
        this.value = value;
        this.onResolvedCallback.forEach(fun => {
          fun(this.value)
        })
      }
    }, 0);
  }
  reject(error) {
    setTimeout(() => {
      if (this.state === PENDING) {
        this.state = REJECTED;
        this.error = error;
        this.onRejectedCallback.forEach(fun => {
          fun(this.error)
        })
      }
    }, 0);
  }
  then(onFulFilled, onRejected) {
    const newPromise = new PPromise((resolve, reject) => {
      onFulFilled = typeof onFulFilled === 'function' ? onFulFilled : v => v
      onRejected = typeof onRejected === 'function' ? onRejected : r => {
        throw r
      }
      if (this.state === RESOLVED) {
        try {
          const returnValue = onFulFilled(this.value)
          this.resolutionProcedure(newPromise,returnValue,resolve,reject)
        } catch (error) {
          reject(error)
        }
      }
      if (this.state === REJECTED) {
        try {
          const returnValue = onRejected(this.error)  
          this.resolutionProcedure(newPromise,returnValue,resolve,reject)
        } catch (error) {
          reject(error)
        }
      }

      if (this.state === PENDING) {
        this.onResolvedCallback.push(() => {
          try {
            const returnValue = onFulFilled(this.value)
            this.resolutionProcedure(newPromise,returnValue,resolve,reject)
          } catch (error) {
            reject(error)
          }
        })
        this.onRejectedCallback.push(() => {
          try {
            const returnValue = onRejected(this.error)
            this.resolutionProcedure(newPromise,returnValue,resolve,reject)
          } catch (error) {
            reject(error)
          } 
        })
      }
    })
    Promises.push(newPromise)
    return newPromise;
  }
  resolutionProcedure(promise,x,resolve,reject) {
    // 防止循环引用
    if(x === promise) {
      return reject(new TypeError('Error'))
    }

    // 如果 x 是 thenable 对象, 那么将 x.then 的结果，再进行处理
    if ((x !== null) && ((typeof x === 'object') || (typeof x === 'function'))) {
      try {
        then = x.then
        if (typeof then === 'function') {
          then.call(x, function rs(y) {
            if (!isCalled){
              isCalled = true;
              this.resolutionProcedure(promise, y, resolve, reject)
            }
          }, function rj(r) {
            if(!isCalled) {
              isCalled = true;
              reject(r)
            }
          })
        } else {
          resolve(x)
        }
      } catch(e) {
        if(!isCalled) {
          isCalled = true;
          reject(e)
        }
      }
    } else {
      resolve(x)
    }
  }
}

// 测试代码
new PPromise((resolve,reject) => {
  resolve(1)
})
.then(res => {
  console.log('res1',res)   // res1 1
  return new PPromise((resolve, reject) => {
      resolve(2)
  })
})
.then(res => {
  console.log('res2',res);  // res2 2
})
```

至此，一个比较完整的 Promise 就完成.

之后，我们可以添加一些常见的类方法和实例方法  

### 1.5 .catch 方法

`.catch` 方法接受一个函数作为参数，该函数会在之前的 Promise `reject` 时执行。

```js
class PPromise {
    ...
    catch(onRejected){
        return this.then(null, onRejected)
    }
}

// 测试代码
new PPromise((resolve,reject) => {
  resolve(1)
})
.then(res => {
  console.log('res1',res)
  return new PPromise((resolve, reject) => {
    reject(2)
  })
})
.then(res => {
  console.log('res2',res);
})
.catch(err => {
  console.log(err)
})
```

### 1.6 Promise.resolve()

该方法接收一个值作为参数，返回 resolve 该值的 Promise  

```js
PPromise.resolve = (value) => {
  return new PPromise((resolve, reject) => {
    resolve(value)
  })
}

// 测试代码
new PPromise((resolve,reject) => {
  resolve(1)
})
.then(res => {
  console.log('res1',res)
  return PPromise.resolve(2)
})
.then(res => {
  console.log('res2',res);   // 2
})
```

### 1.7 Promise.reject()
同 Promise.resolve(),代码如下:  

```js
PPromise.reject = (value) => {
  return new PPromise((resolve, reject) => {
    reject(value)
  })
}

// 测试代码
new PPromise((resolve,reject) => {
  resolve(1)
})
.then(res => {
  console.log('res1',res)
  return PPromise.reject(2)
})
.then(res => {
  console.log('res2',res);
})
.catch(err => {
  console.log('err',err)  // 2
})
```

### 1.8 Promise.all()

```js
PPromise.all = (promises) => {
  return new PPromise(function(resolve, reject) {
    const promiseLen = promises.length;
    let resolvedCount = 0;
    const result = [];

    for (let i = 0; i < promises.length; i++) {
      Promise.resolve(promises[i]).then(function(value) {
        result[i] = value;
        resolvedCount++;
        if(resolvedCount === promiseLen) {
          resolve(result)
        }
      }, function(error) {
        reject(error)
      })
    }
  })
}

// 测试代码
PPromise.all([
  new PPromise((resolve,reject) => {
    setTimeout(() => {
      resolve(1)
    }, 1000);
  }),
  new PPromise((resolve,reject) => {
    setTimeout(() => {
      resolve(2)
    }, 200);
  })
]).then(res => {
  console.log(res)  [1,2]
})
```

### 1.9 Promise.race()

```js
PPromise.race = (promises) => {
  return new PPromise(function(resolve, reject) {
    for (var i = 0; i < promises.length; i++) {
      Promise.resolve(promises[i]).then(function(value) {
        return resolve(value)
      }, function(error) {
        return reject(error)
      })
    }
  })
}

// 测试代码
PPromise.race([
  new PPromise((resolve,reject) => {
    setTimeout(() => {
      resolve(2)
    }, 1000);
  }),
  new PPromise((resolve,reject) => {
    setTimeout(() => {
      resolve(1)
    }, 200);
  })
]).then(res => {
  console.log(res)  // 2
})
```

## 2. 相关题

### 2.1 Promise 的异常捕获

首先回顾下 .catch 和 .then 的源码:  

```js
PPromise.prototype.catch = function(onRejected) {
  // .catch 是在当前的 Promise 实例上，调用 then 方法
  return this.then(null, onRejected)
}

PPromise.prototype.then = function(onFulFilled, onRejected) {
    // onRejected 默认代码会 throw error, 这个 error 会被 try catch 住 
    onRejected = onRejected ? onRejected : r => {
        throw r
    }
    return promise2 = new PPromise(function(resolve, reject) {
        ...
        try {
            var x = onResolved(value)
            resolvePromise(promise2, x, resolve, reject)
        } catch(e) {
            return reject(e)
        }
        ...
    }
    
}
```
然后我们查看示例一:  
```js
// 示例 1
PPromise.reject(1)
.then(res => {           // 第一个 then
  console.log('res',1)
})
.then(res => {  // 第二个 then
  console.log('res',2)
})
.catch(err => {
  // 直接执行到这，前面的 console.log 都不会执行
  console.log(err)  
})
```

1. 当 Promise.reject 之后，返回了一个 rejected 状态的 promise
2. 第一个 then 中，由于没有第二个参数 onRejected 函数，所以默认为 throw Error, 此时又会返回一个 rejected 状态的 promise。
3. 同理第二个 then 也是这样，返回一个 rejected 状态的 promise
4. 在最后的 .catch 中，返回的 rejected 的 promise，执行 `promise.then(null, err => { console.log(err) })`


即由于 Promise rejected之后，在前面没有被代码处理，导致一次次产生新的 rejected 状态的 Promise。  

再看看示例二:  
```js
// 示例二
PPromise.reject(1)
.then(res => {     // 第一个 then
  console.log('res',1) 
},err => {
    console.log(err);  //  1 
})
.then(res => {
  console.log('res',2)   // 2
})
.catch(err => {
  // 不会执行这里，因为 error 已经在前面捕获了
  console.log(err)  
})
```

1. 当 Promise.reject 之后，返回了一个 rejected 状态的 promise
2. 第一个 then 中，有第二个参数 onRejected 函数，且该函数没有报错或者返回 rejected 状态的 promise，因此返回一个新的 resolved 状态的 promise。

此时示例代码就可以简化如下:  
```js
Promise.resolve()
.then(res => {
  console.log('res',2)   // 2
})
.catch(err => {
  console.log(err)  
})
```
即是否执行 catch 里的代码，与最初的 Promise.reject 无关。  


### 2.2 如何串行执行 Promise

如下三个返回 Promise 的函数，如何让三个函数中的 Promise 依次执行  

```js
const promise1 = () => new Promise((resolve, reject) => {
  setTimeout(() => {
    console.log('promise1')
    resolve(1)
  }, 3000);
})

const promise2 = () => new Promise((resolve, reject) => {
  setTimeout(() => {
    console.log('promise2')
    resolve(2)
  }, 2000);
})

const promise3 = () => new Promise((resolve, reject) => {
  setTimeout(() => {
    console.log('promise3')
    resolve(3)
  }, 1000);
})
```

要完成以上的需求，我们可以使用 promise 链式调用的方式:  

```js
Promise.resolve()
.then(res => {
    return promise1(res)
})
.then(res => {
    return promise2(res)
})
.then(res => {
    return promise3(res)
})
```

然后转用 reduce，可以写出以下的代码:  

```js
function serialPromise(promises) {
    promises.reduce((prev, next) => {
        // 该 retunr 是 reduce 语法中的 return
        return prev.then(prevResult => {
            // 该 return 是 promise.then 语法中的 return
            return next(prevResult);
        })
    }, Promise.resolve())
}

// 调用
serialPromise([promise1, promise2, promise3]);  
// 3s   打印 promise1
// 5s   打印 promise2
// 6s   打印 promise3
```

但是，以上代码中，如果有一个 promise(比如 promise2) reject了，那么执行链就会断掉:  

```js
const promise2 = () => new Promise((resolve, reject) => {
  setTimeout(() => {
    console.log('promise2')
    reject(2)
  }, 2000);
})

...

// UnhandledPromiseRejectionWarning: 2 ... 报错
```
因此我们还需要捕获 promise 的异常,即使 reject 了也要继续执行之后的代码:  

```js
function serialPromise(promises) {
    promises.reduce((prev, next) => {
        return prev.then(prevResult => {
            return next(prevResult);
        }).catch(err => {
            return next(err)
        })
    }, Promise.resolve())
}

serialPromise([promise1, promise2, promise3]);  
// 3s   打印 promise1
// 5s   打印 promise2
// 7s   打印 promise2
// 8s   打印 promise3
```
即最终打印了两遍 promise2, 这是为什么呢？  

其实以上的代码，还原回正常的代码应该是:  
```js
Promise.resolve()
.then(res => {
    // 1
    return promise1(res)
})
.catch(err => {
    // 2
  return promise1(err)
})
.then(res => {    // 第一次
    // 3
    return promise2(res)
})
.catch(err => {   // 第二次
    // 4
  return promise2(err)
})
.then(res => {
    // 5
    return promise3(res)
})
.catch(err => {
    // 6
  return promise3(err)
})

执行顺序: 1 3 4 6
```

即以上代码有两个问题:  
1. 在第一次执行的时候，会执行一遍 promise2, 而此时返回的 Promise 是 reject 状态的, 会进入 .catch 执行第二次，这时候又会再执行一遍 promise2
2. 第二次的 promise 依旧是 reject 状态，所以最终执行的是 6， 而不是 5，***虽然没啥问题，但就是觉得怪怪的***。  

我们可以想到以下的代码，可以处理这个第一个问题：

```js
Promise.resolve()
.then(res => {
    // 1
    return promise1(res)
},err => {
    // 2
    return promise1(err)
})
.then(res => {
    // 3
    return promise2(res)
},err => {
    // 4
    return promise2(err)
})
.then(res => {
    // 5
    return promise3(res)
},err => {
    // 6
    return promise3(err)
})

执行顺序: 1 4 6
```

即依然会有第二个问题。  

另外一种解决方案是:  

```js
const serialPromises3=function(promises){
  const process=function(i,args){
    const curr=promises[i]
    // 由于 next 函数返回 undefined，不返回 rejected 的 promise
    // 所以不会存在执行两遍的情况
    // 而且之后的 promise 也是走的 onFulFilled 不是 onRejected
    const next=function(res){process(i+1,res)}  
    if(curr) {
      curr(args)
      .then(next)
      .catch(next)
    }
  }
  process(0)
}
```

### 2.3 如果 resolve 一个 Promise, 结果会如何？

面试题，以下代码会打印什么结果？

```js
new Promise(resolve => {
    resolve(Promise.reject())
})
.then(res => {
    console.log('success')
},err => {
    console.log('error')
})
```
放到控制台执行就知道了会打印 error， 为什么呢？按照我们的源码，我不管 resolve 的值是什么，只要是 resolve, 就应该执行 onFulFilled 才对。  

但是，其实 [有个 issue ](https://github.com/xieranmaya/blog/issues/3#issuecomment-546959287) 里讨论过这个:  

> Q: 为什么 Promise 构造函数里面的 resolve 不需要处理 value 有可能是 thenable 的逻辑?   

> A: 问的很好。实际上是需要的。ES6 Promise中就对此种情况做出了定义。  
Promise/A+标准并没有对构造函数的签名及行为做出定义，只定了then方法的行为。所以Promise/A+的测试中并不包含测试构造函数的用例。  

也就是说，我们平常使用到的 Promise 不仅仅实现了 Promise/A+ 的规范，同时对构造函数部分的代码也做了处理。**因此我们上面的源码不能反映 ES6 中 Promsie 的结果**  

于是我找到了 [ ES6 Promise ](https://github.com/stefanpenner/es6-promise), 且测试了结果确实是打印 error。  

于是我们想到，如果 resolve 中的结果是个 promise，我们应该将它的值作为 new Promise 的值,于是改进代码:  
```js
function resolve(value) {
  setTimeout(function () {
    if (self.status !== 'pending') {
      return
    }
    self.status = 'resolved'
    self.data = value

    if (value && value.then) {
        // 如果 resolve 一个 promise，则将 value.then 来决定当前 promise 的值
      value.then(res => {
        for (var i = 0; i < self.callbacks.length; i++) {
          self.callbacks[i].onResolved(value)
        }
      }, err => {
        for (var i = 0; i < self.callbacks.length; i++) {
          self.callbacks[i].onRejected(value)
        }
      })
    } else {
      for (var i = 0; i < self.callbacks.length; i++) {
        self.callbacks[i].onResolved(value)
      }
    }
  })
}
```

而查看 es6-promise, 确实在 resolve 的时候会有取 value.then 相关的代码(具体逻辑暂不研究):  

```js
function resolve(promise, value) {
  if (promise === value) {
    reject(promise, selfFulfillment());
  } else if (objectOrFunction(value)) {
    var then$$1 = void 0;
    try {
      then$$1 = value.then;  // 取 value.then
    } catch (error) {
      reject(promise, error);
      return;
    }
    handleMaybeThenable(promise, value, then$$1);
  } else {
    fulfill(promise, value);
  }
}
```  

于是，我又想，是不是在 reject 的时候，也会如此呢？:  
```js
new Promise((resolve,reject) => {
    reject(Promise.resolve())
})
.then(res => {
    console.log('success')
},err => {
    console.log('error')
})
```  

结果发现并不是，结果还是打印 error , 因为 es6-promise 中的 reject 并没有进行 value.then 的处理(粗略翻看代码，并没有 value.then 的操作)。  

## 参考资料
1. [js 真的是一步一步手写promise](https://juejin.im/user/59486117ac502e006bb01ec3)
2. yck 前端面试之道
3. [Promises/A+规范](https://www.ituring.com.cn/article/66566)
4. [剖析Promise内部结构，一步一步实现一个完整的、能通过所有Test case的Promise类](https://github.com/xieranmaya/blog/issues/3)
5. [Promise所有方法实现源码](https://github.com/xieranmaya/Promise3/blob/master/Promise3.js)
6. [精读《用 Reduce 实现 Promise 串行执行》](https://segmentfault.com/a/1190000016832285)
7. [串行执行promise](https://www.cnblogs.com/zhuxianguo/p/11445952.html)
8. [es6-promise](https://github.com/stefanpenner/es6-promise)