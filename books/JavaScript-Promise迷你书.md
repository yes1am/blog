**对象方法与实例方法的表示**  

实例方法：`Promise#then`，表示为Promise的实例对象的then方法  
对象方法：`Promise.all`，表示为Promise类的静态方法  

## 1. 什么是Promise

### 1.1 构造器

```
var promise = new Promise((resolve, reject) {
    // 异步处理
    // 处理结束之后调用resolve或者reject
})
```

### 1.2 实例方法

`promise.then(onFulfilled, onRejected)`  
当resolve(成功)时，onFulfilled会被调用, 当reject(失败)时，onRejected会被调用。  

`promise.then` 在成功和失败时都可以调用，如果只想处理异常情况，可以使用 `promise.then(undefined, onRejected)`,即只指定reject的回调函数。  

不过这种情况下`promise.catch(onRejected)`应该是更好的选择，即推荐使用`.catch`来将 `resolve` 和 `reject` 处理分开来写，而不是使用 `.then(onResolve,onReject)`  

```
new Promise((resolve,reject) => {
  return (2)
  resolve(1);
}).then(res => {
  console.log(res)  //  不打印，promise一直是pending
})

new Promise((resolve,reject) => {
  resolve(1);
  return (2)
}).then(res => {
  console.log(res) //  打印1， 与return的值无关
})

new Promise((resolve,reject) => {
  reject(1);
  return (2)
}).then(res => {
  console.log(res)
},(ee) => {
  console.log("@@",ee)         // 打印 1，即错误由这里捕获
}).catch((err) => {
  console.log(err)             // 不打印
})
```  

### 1.3 静态方法

`Promise.all()`，`Promise.resolve()`，`Promise.reject()`等  

### 1.4 promise的状态

promise的三种状态：`Fulfilled`, `Rejected`, `Pending` .暂时没有查询其内部状态的方法。  

promise对象的状态从 `pending` 转为 `fulfilled` 或者 `rejected` 之后，这个对象的状态就不会再发生任何变化，即 `.then` 后执行的函数之后被调用一次。  

## 2. 实战 Promise

### 2.1 创建promise对象

一般情况下可以使用 `new Promise()`来创建promise对象，同时还可以使用`Promise.resolve()`和`Promise.reject()`这两个方法。  

`Promise.resolve(42)` 可以认为是以下代码的语法糖：  
```
new Promise(resolve => {
    resolve(42)
})
```  
即 `Promise.resolve(value)` 返回值也是一个 `promise对象` ，因此可以使用`.then`进行调用。  

```
Promise.resolve(42).then(value => {
    console.log(value);     // 42
})
```  

`Promise.resolve` 作为 `new Promise()` 的快捷方式，在进行`promise对象的初始化`或者编写测试代码时都非常方便。  

### 2.2 thenable

`Promise.resolve` 的另一个作用就是将 `thenable（即，拥有.then方法）` 的对象转换为 `promise` 对象。  

这种将 `thenable对象` 转换为 `promise对象` 的机制要求 `thenable对象` 所拥有的 `then` 方法与 `Promise` 对象的 `then` 方法具有相同的功能和处理过程。  

[jQuery.ajax()](https://api.jquery.com/jQuery.ajax/)返回值就是 `thenable` 的，因为该返回值是 `jqXHR Object` 对象，具有 `.then` 方法。  

```
Promise.resolve($.ajax('/json/comment.json')).then(value => {
    console.log(value)
}).catch(err => {
    console.log(err)
})
```  
即转换为 `promise` 对象之后，就能直接使用 `then` 或者 `catch` 等方法了。  

### 2.3 Promise.reject()

`Promise.reject(error)` 也是 `new Promise()` 方法的快捷方式。`Promise.reject(new Error('error'))` 是下面代码的语法糖：  
```
new Promise((resolve,reject) => {
    reject('error')
})


// 将错误对象传递给onReject函数处理
Promise.reject('error').catch(function(error){
    console.error(error);
});
```

### 2.4 Promise只能进行异步操作

```
var promise = new Promise(function (resolve){
    console.log("inner promise"); // 1
    resolve(42);  // 即使是立即resolve，也是异步的调用.then方法
});
promise.then(function(value){
    console.log(value); // 3
});
console.log("outer promise"); // 2
```  

### 2.5 try catch 不能捕获异步的error

```
let a = null;
try {
    a = setTimeout(() => {
    throw new Error('123')  // try catch 不能捕获异步的异常，这里无打印
  }, 2000)
} catch (e){
  console.log(e)        
}

console.log(a)
```  

### 2.6 方法链 Promise#then


```
new Promise((resolve,reject) => {
  resolve(1);
}).then(res => {
  new Promise((resolve,reject) => {  // 此时这段代码返回值为undefined
    console.log('res from', res)
    setTimeout(()=>{
      console.log('timeout')
      resolve(res+2)
    }, 100)
  })
}).then(res => {
  console.log('last', res)          // 则此时的res为undefined
}).catch(e => {
  console.log('err',e);
})



new Promise((resolve,reject) => {
  resolve(1);
}).then(res => {
  return new Promise((resolve,reject) => {  // 此时这段代码返回值为promise,即必须用return返回
    console.log('res from', res)
    setTimeout(()=>{
      console.log('timeout')
      resolve('i am promise')
    }, 100)
  })
}).then(res => {
  console.log('last', res)          // 则此时的res为 i am promise
}).catch(e => {
  console.log('err',e);
})
```

### 2.7 异常处理

```
function taskA() {
    console.log("Task A");
}
function taskB() {
    console.log("Task B");
}
function onRejected(error) {
    console.log("Catch Error: A or B", error);
}
function finalTask() {
    console.log("Final Task");
}

Promise.resolve()
    .then(taskA)
    .then(taskB)
    .catch(onRejected)
    .then(finalTask);
```

即当 `taskA` 或者 `taskB` 中出现异常(这种异常可以是 `throw一个error` ，或者返回一个 `rejected状态的promise对象` )，都会进入 `onRejected` 处理。  

```
function taskA() {
    console.log("Task A");
    new Promise((res,rej) => {  // 这种方式虽然有reject的promise对象，但是没有用return返回，因此并不会进入catch
      rej(123);
    })
}
function taskB() {
    console.log("Task B");
}
function onRejected(error) {
    console.log("Catch Error: A or B", error);
}
function finalTask() {
    console.log("Final Task");
}

Promise.resolve()
    .then(taskA)
    .then(taskB)
    .catch(onRejected)
    .then(finalTask);
```

由于在 `onRejected` 和 `finalTask` 两个任务后面没有 `catch` 处理，因此如果在这两个任务出现异常，将不会被捕获。  

```
function taskA() {
    console.log("Task A");
    return new Promise((res,rej) => {  // 用return返回，则不会执行taskB，而是进入catch中
      rej(123);
    })
}

function taskB() {
    console.log("Task B");
}

function onRejected(error) {
    console.log("Catch Error: A or B", error);
}

function finalTask() {
    console.log("Final Task");
}

Promise.resolve()
    .then(taskA)
    .then(taskB)
    .catch(onRejected)
    .then(finalTask);
```

### 2.8 Promise#then 参数传递

如果两个任务之间要进行值的传递，则需要在上一个任务中将值 return 出来

```
function doubleUp(value) {
    return value * 2;
}
function increment(value) {
    // return的值会作为下一个任务的参数，return的值会被Promise.resolve()进行包装，即每次调用then都是返回一个promise对象
    return value + 1;  
    
}
function output(value) {
    console.log(value);// => (1 + 1) * 2
}

var promise = Promise.resolve(1);
promise
    .then(increment)
    .then(doubleUp)
    .then(output)
    .catch(function(error){
        // promise chain中出现异常的时候会被调用
        console.error(error);
    });
```

### 2.9 Promise#catch

`Promise#catch` 只是 `Promise#then(undefined,onRejected)` 方法的别名，即当 `promise对象` 状态变为 `Rejected` 时的回调。  

### 2.10 then 和 catch 都是返回新的 promise 对象

```
var aPromise = new Promise(function (resolve) {
    resolve(100);
});
var thenPromise = aPromise.then(function (value) {
    console.log(value);
});
var catchPromise = aPromise.catch(function (value) {
    console.log(value);
});
console.log(aPromise !== thenPromise);     // true
console.log(aPromise !== catchPromise);   // true
```

### 2.11 Promise.all 与 Promise.race

`Promise.all`: 当数组里所有的 `promise对象` 全部变为 `resolve或reject状态` 的时候，才会调用 `.then` 方法。如下，即最长时间 `128ms` 之后，才会执行 `.then`   

```
// `delay`毫秒后执行resolve
function timerPromisefy(delay) {
    return new Promise(function (resolve) {
        setTimeout(function () {
            resolve(delay);
        }, delay);
    });
}
var startDate = Date.now();
Promise.all([
    timerPromisefy(1),
    timerPromisefy(32),
    timerPromisefy(64),
    timerPromisefy(128)
]).then(function (values) {
    console.log(Date.now() - startDate + 'ms');
    // 約128ms
    console.log(values);    // [1,32,64,128]
});
```

`Promise.race()`: 只要有 `一个promise对象`  变为 `fulfilled` 或者 `rejected` 状态，就会调用后续的 `.then` 或者 `.catch` 处理。如下：  

4ms之后 `第一个promise` 进行 `resolve` ，因此 `Promise.race` 进行后续的处理。但是 `loserPromise` 这个 `promise` 依然会执行，并不会因为 `race` 而被取消。  

```
var winnerPromise = new Promise(function (resolve) {
        setTimeout(function () {
            console.log('this is winner');
            resolve('this is winner');
        }, 4);
    });
var loserPromise = new Promise(function (resolve) {
        setTimeout(function () {
            console.log('this is loser');      // 依然会打印
            resolve('this is loser');
        }, 1000);
    });
Promise.race([winnerPromise, loserPromise]).then(function (value) {
    console.log(value);    // => 'this is winner'
});
```  

### 2.12 使用 .then(undefined, onRejected) 还是 .catch(onRejected)

```
Promise.resolve(1).then(() => {
  return Promise.reject('bad')
}, (err) => {
    // 不打印，即使用.then(第一个参数,第二个参数),则第二个参数不能捕获第一个参数的异常。只能继续在后续添加.then去捕获
    console.log(err)
}).then(undefined,(err) => {
  console.log('err',err)  //  这里能打印 bad
})


// 使用.catch能捕获.then的第一个参数中所抛出的异常
Promise.resolve(1).then(() => {
  return Promise.reject('good')
}).catch((err) => {
  console.log(err)
})
```

即如果使用 `then` 的方式，如果 `onFulfilled` 中抛出异常，是不能被当前的 `onRejected` 捕获的，因此使用 `.catch` 是更好的处理错误方式。  

## 3. Promise Advanced

`Promises/A+` 是 `ES6 Promises` 的前身，如果一个类库兼容 `Promise/A+` 的话,那么说明除了 `then` 方法之外，还支持 `Promise.all和catch`。  

### 3.1 类库 bluebird

这个类库除了兼容 `Promise` 规范之外，还扩展了 `取消promise` 对象的运行，取得 `promise` 的运行进度，以及 `错误处理的扩展检测` 等非常丰富的功能。  

### 3.2 使用 reject 还是 throw error

`Promise`的构造函数，以及then调用的函数都可以认为是在`try catch`代码块中执行的，所以这些代码内部使用`throw`，程序不会因为异常而终止。  

```
new Promise(() => {
  throw new Error('123')
})
console.log(456)     // 依旧会打印456


throw new Error('123')
console.log(456)     // 因为异常，不打印
```

在 `promise` 中使用 `throw`，会被`try catch`住，最终 `promise` 变为 `rejected` 状态。  
```
var promise = new Promise(function(resolve, reject){
    throw new Error("message");
});
promise.catch(function(error){
    console.error(error);// => "message"
});
```

### 3.3 在then中使用reject

```
Promise.resolve(1).then(val => {
  return new Promise((res,rej) => {
     setTimeout(() => {
      rej(12);
    }, 1000)  
  })
}).catch(e => {
  console.log(e)          // 12
})



Promise.resolve(1).then(val => {
     setTimeout(() => {
        Promise.reject(12);   // 无效,即 promise 不会等待定时器触发
    }, 1000)  
}).catch(e => {
  console.log(e)
})



Promise.resolve(1).then(val => {
  return Promise.reject(123)  // 直接使用reject
}).catch(e => {
  console.log(e)          // 123
})
```

### 3.4 处理超时

```
function delayPromise(ms) {
    return new Promise(function (resolve) {
        setTimeout(resolve, ms);
    });
}

function timeoutPromise(promise, ms) {
    var timeout = delayPromise(ms).then(function () {
            throw new Error('Operation timed out after ' + ms + ' ms');
        });
    return Promise.race([promise, timeout]);
}

// 运行示例
var taskPromise = new Promise(function(resolve){
    // 随便一些什么处理
    var delay = Math.random() * 2000;
    setTimeout(function(){
        resolve(delay + "ms");
    }, delay);
});

timeoutPromise(taskPromise, 1000).then(function(value){
    console.log("taskPromise在规定时间内结束 : " + value);
}).catch(function(error){
    console.log("发生超时", error);
});

```

#### 3.4.1 超时取消xhr请求

```
function copyOwnFrom(target, source) {
    Object.getOwnPropertyNames(source).forEach(function (propName) {
        Object.defineProperty(target, propName, Object.getOwnPropertyDescriptor(source, propName));
    });
    return target;
}

function TimeoutError() {
    var superInstance = Error.apply(null, arguments);
    copyOwnFrom(this, superInstance);
}

TimeoutError.prototype = Object.create(Error.prototype);
TimeoutError.prototype.constructor = TimeoutError;

function delayPromise(ms) {
    return new Promise(function (resolve) {
        setTimeout(resolve, ms);
    });
}

function timeoutPromise(promise, ms) {
    var timeout = delayPromise(ms).then(function () {
            return Promise.reject(new TimeoutError('Operation timed out after ' + ms + ' ms'));
        });
    return Promise.race([promise, timeout]);
}

function cancelableXHR(URL) {
    var req = new XMLHttpRequest();
    var promise = new Promise(function (resolve, reject) {
            req.open('GET', URL, true);
            req.onload = function () {
                if (req.status === 200) {
                    resolve(req.responseText);
                } else {
                    reject(new Error(req.statusText));
                }
            };
            req.onerror = function () {
                reject(new Error(req.statusText));
            };
            req.onabort = function () {
                reject(new Error('abort this request'));
            };
            req.send();
        });
    var abort = function () {
        // 如果request还没有结束的话就执行abort
        // https://developer.mozilla.org/en/docs/Web/API/XMLHttpRequest/Using_XMLHttpRequest
        if (req.readyState !== XMLHttpRequest.UNSENT) {
            req.abort();
        }
    };
    return {
        promise: promise,
        abort: abort
    };
}
var object = cancelableXHR('http://httpbin.org/get');
// main
timeoutPromise(object.promise, 1000).then(function (contents) {
    console.log('Contents', contents);
}).catch(function (error) {
    if (error instanceof TimeoutError) {
        object.abort();
        return console.log(error);
    }
    console.log('XHR Error :', error);
});
```

### 3.5 Promise#done

```
Promise.resolve(1).then(()=> {
  console.lg(1)     // 不会报错，因为promise代码类似在一个大的try catch中运行，所以即使console.log拼错了，也不会被报出来，除非手动加上catch 
})


Promise.resolve(1).then(()=> {
  console.lg(1)
}).catch(ee => {
  console.log(ee)  // 报错 console.lg is not a function
})
```

这种被内部消化的问题叫做 `unhandled rejection`，即rejected时没有找到响应的处理程序。  
`promise.prototype.done`的源码  

```
if (typeof Promise.prototype.done === 'undefined') {
    Promise.prototype.done = function (onFulfilled, onRejected) {
        this.then(onFulfilled, onRejected).catch(function (error) {
            setTimeout(function () {
                throw error;
            }, 0);
        });
    };
}
```

使用起来和 `.then` 一样，但是不会返回一个 `promise` 值，因此不能链式调用,即只能出现在方法链的最后面。因此：  

```
if (typeof Promise.prototype.done === 'undefined') {
    Promise.prototype.done = function (onFulfilled, onRejected) {
        this.then(onFulfilled, onRejected).catch(function (error) {
            setTimeout(function () {
                throw error;
            }, 0);
        });
    };
}
Promise.resolve(1).done(()=> {
  console.lg(1)     //此时抛出异常
})
```

### 3.6 More

**4.7,4.8 两节应该多看！关于用Promise包装API以及顺序处理Promise的问题**

## 4. 参考资料

1. [Promise迷你书](http://liubin.org/promises-book/#chapter4-advanced-promise)