## 1. 什么是 Redux 中间件

在 Redux 基础中，用户 dispatch(action), reducer 计算出新的 state，并执行重新渲染。  

但是，异步操作怎么办  

Action 发出以后，Reducer 立即算出 State，这叫做同步  
Action 发出以后，**过一段时间再执行 Reduce**r，这就是异步。

```js
// redux dispatch 方法，在执行 dispatch 之后，会立即执行 reducer
function dispatch (action) {
    currentState = currentReducer(currentState, action)
    
    // 在 dispatch 之后，通知所有的 listener
    const listeners = (currentListeners = nextListeners)
    for (let i = 0; i < listeners.length; i++) {
      const listener = listeners[i]
      listener()
    }
    
    return action
}
```

因此引入了中间件的概念:  

中间件示例
```js
import React from 'react'
import ReactDOM from 'react-dom'
import { Provider } from 'react-redux'
import rootReducer from './reducers'
import { createStore } from 'redux'
import App from './app'

const store = createStore(
  rootReducer,
  window.__REDUX_DEVTOOLS_EXTENSION__ && window.__REDUX_DEVTOOLS_EXTENSION__()
)

## 中间件
let next = store.dispatch   // 保存原来的 dispatch 方法
store.dispatch = function dispatchAndLog (action) {
  console.log('dispatching', action)
  next(action)   // 执行真正的 reducer 
  console.log('next state', store.getState())
}

ReactDOM.render(
  <Provider store={store}>
    <App />
  </Provider>,
  document.getElementById('root')
)
```

以上的中间件，在执行真正的 reducers 之前，**打印了 action 的信息**，在执行之后，打印 `最新state` 的信息   

> 即中间件就是一个函数，**对 store.dispatch 方法进行了改造**，在发出 Action 和执行 Reducer 这两步之间，添加了其他功能。

## 2. 使用 redux-logger 中间件

以上那个打印信息的中间件，已有现成的 [redux-logger](https://github.com/LogRocket/redux-logger)，我们引入即可使用.

`npm i redux-logger -S`  

```js
import React from 'react'
import ReactDOM from 'react-dom'
import { Provider } from 'react-redux'
import rootReducer from './reducers'
import { createStore, applyMiddleware, compose } from 'redux'
import { createLogger } from 'redux-logger'
import App from './app'
const logger = createLogger()

const composeEnhancers = window.__REDUX_DEVTOOLS_EXTENSION_COMPOSE__ || compose
const store = createStore(rootReducer, composeEnhancers(
  applyMiddleware(logger)
))

ReactDOM.render(
  <Provider store={store}>
    <App />
  </Provider>,
  document.getElementById('root')
)
```

### 2.1 中间件的顺序

如果有多个中间件，可以给 applyMiddleware 传入多个参数:  

```js
const store = createStore(
  reducer,
  applyMiddleware(thunk, promise, logger)
);
```
其中**中间件的顺序是有要求的(具体查看中间件的文档)**，因为 `applyMiddleware` 内部使用了 `compose` 函数，会按照某种顺序依次执行中间件。

applyMiddleware 代码解析  
略

## 3. 异步操作的思路

同步操作只需要 dispatch(action)， 而异步操作则需要三种 action
```
1. 开始请求接口
2. 请求接口成功
3. 请求接口失败
```

第一个 action 可以认为是同步的，但是，**如何在请求成功或者请求失败时，自动 dispatch 成功或失败的 action** 呢？  

前面我们提到了 Action Creator，用于创建 action:  

```
// 我们的组件，didMount 时 dispatch 一个 action
class AsyncApp extends Component {
  componentDidMount() {
    const { dispatch, selectedPost } = this.props
    dispatch(fetchPosts(selectedPost))
  }
}

// action creator
// fetchPosts 方法接受一个 title，返回一个函数
// 这个函数执行的时候，会先 dispatch 一个 requestPosts(postTitle) 的 action
// 在异步请求请求成功之后，会 dispatch 一个 receivePosts(postTitle, json) 的 action

const fetchPosts = postTitle => (dispatch, getState) => {
  dispatch(requestPosts(postTitle));
  return fetch(`/some/API/${postTitle}.json`)
    .then(response => response.json())
    .then(json => dispatch(receivePosts(postTitle, json)));
  };
};
```

所以，通过改造 action creator 就能实现，触发多个 action。  

可有一个问题，之前我们已知， dispatch 的参数必须是对象，而这里 dispatch 的参数是一个函数 `(dispatch, getState) => {}`.  

**因此我们引入 `redux-thunk` 中间件，使得 dispatch 可以接受函数作为参数**  

因此，异步操作的第一种解决方案就是，写出一个返回函数的 Action Creator，然后使用redux-thunk中间件改造store.dispatch。

此外，还有 redux-promise 中间件这种解决方案。 略 



## 参考资料

1. [Redux 入门教程（二）：中间件与异步操作](http://www.ruanyifeng.com/blog/2016/09/redux_tutorial_part_two_async_operations.html)