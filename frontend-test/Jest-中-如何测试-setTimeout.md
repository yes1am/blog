[Jest Time Mocks](https://jestjs.io/docs/en/timer-mocks)  

原生的 time 函数 (比如，`setTimeout`, `setInterval`, `clearTimeout`, `clearInterval`) 不是很便于测试，因为需要依赖于真正的时间流逝，Jest 可以用函数来代替 timers, 从而让你来控制时间的流逝。  

假设我们需要测试一段异步的代码:  

```js
function fun(obj) {
  setTimeout(() => {
    obj.a = 1;
  }, 1000);
}


test('test', () => {
  const obj = { a: 2 };
  fun(obj);
  expect(obj).toEqual({ a: 1 });   // error
});
```

以上代码会失败，因为 `setTimeout` 是异步代码，需要一段时间之后才能得到结果.  

于是 Jest 提供了 `runAllTimers()` 这个函数来测试异步。  

```js
function fun(obj) {
  setTimeout(() => {
    obj.a = 1;
  }, 1000);
}


test('test', () => {
  const obj = { a: 2 };
  jest.useFakeTimers();        //  开启模拟定时器
  fun(obj);
  jest.runAllTimers();         // 加速，让所有的定时器都执行完毕
  expect(obj).toEqual({ a: 1 });
});
```

通过以上这种方式，异步代码就可以测试通过了。  

但是，如果存在 `timer 递归的时候`，`runAllTimer()` 就会失败，因为一直都存在定时器。  

```js
let a = 1;
let b = 2;

function fun() {
  setTimeout(() => {
    a = 2;
    setTimeout(() => {
      b = 3;
      fun();        // 定时器里递归调用定时器
    }, 2000);
  }, 1000);
}

// error, 不能使用 runAllTimers 方法
test('test', () => {
  jest.useFakeTimers();
  fun();
  jest.runAllTimers();
  expect(a).toBe(2);
  expect(b).toBe(3);
});
```

这个时候应该使用 `runOnlyPendingTimers`  

```
let a = 1;
let b = 2;

function fun() {
  setTimeout(() => {
    a = 2;
    setTimeout(() => {
      b = 3;
      fun();
    }, 2000);
  }, 1000);
}

test('test', () => {
  jest.useFakeTimers();
  expect(a).toBe(1);
  expect(b).toBe(2);
  fun();
  jest.runOnlyPendingTimers();  // 加速第一个定时器
  expect(a).toBe(2);            // 正确
  expect(b).toBe(3);            // 错误，第二个定时器还没执行
  jest.runOnlyPendingTimers();  // 加速第二个定时器
  expect(b).toBe(3);            // 正确
});
```

另外，我们可以通过 `advanceTimersByTime(time)` 来指定加速多久。  

还是这个例子:  

```js
let a = 1;
let b = 2;

function fun() {
  setTimeout(() => {
    a = 2;
    setTimeout(() => {
      b = 3;
      fun();
    }, 2000);
  }, 1000);
}


// 错误
// 即当我们加速 1s 的时候，只有第一个定时器会执行
test('test', () => {
  jest.useFakeTimers();
  fun();
  jest.advanceTimersByTime(1000);
  expect(a).toBe(2);
  expect(b).toBe(3);   // 应该是 2
});

// 正确
// 而当我们加速 3s 的时候，两个定时器都会执行
// 因此 a, b 的值都会发生变化
test('test', () => {
  jest.useFakeTimers();
  fun();
  jest.advanceTimersByTime(3000);
  expect(a).toBe(2);
  expect(b).toBe(3);
});
```
以上定时器的测试例子，同样适用于测试 `React` 组件，如某个操作发生，需要 `setTimeout` 一定时间之后才改变 `state` 的场景。  