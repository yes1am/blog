> redux-saga 是一个 redux 中间件

注意，redux-saga 是 redux 中间件，这意味着它应该配合 redux 一起使用，并且 react-redux 是将 redux 的 store 连接到 React 组件中，因此他们三者 **redux， react-redux, redux-saga** 是合作关系而非对立关系。  

## 1. 起步

[官方的新手示例](https://github.com/redux-saga/redux-saga-beginner-tutorial)

安装  

`npm install --save redux-saga`  

入口文件 index.js :  
```js
import { Provider } from 'react-redux'
import reducer from './reducer'
import { createStore, applyMiddleware, compose } from 'redux'
import createSagaMiddleware from 'redux-saga'
import rootSaga from './sagas'  // 引入 saga
import App from './app'

const sagaMiddleware = createSagaMiddleware()  // 创建 saga 中间件

// 使用 chrome redux 调试工具
const composeEnhancers = window.__REDUX_DEVTOOLS_EXTENSION_COMPOSE__ || compose


const store = createStore(reducer, composeEnhancers(
  applyMiddleware(sagaMiddleware)              // 使用中间件
))

sagaMiddleware.run(rootSaga)                   // 运行中间件

ReactDOM.render(
  <Provider store={store}>
    <App />
  </Provider>,
  document.getElementById('root')
)
```

**由以上例子可以看出，sagaMiddleware 应配合 redux, react-redux 一起使用**  

我们再来看以下 saga.js  

saga.js
```js
import { put, takeEvery, all, delay } from 'redux-saga/effects'

// Worker Saga: 工作 Saga 用于执行异步任务
export function * fetchUserInfo (action) {
  const { type, payload } = action
  yield delay(1000) // 模拟异步请求，暂停 1s
  const userInfoArr = [
    {
      name: 'user1',
      age: 'age1'
    },
    {
      name: 'user2',
      age: 'age2'
    }
  ]
  // 在这里调用 reducer
  yield put({ type: 'SET_USER_INFO', payload: userInfoArr[payload] })
}


// Watcher Saga: 监听器 saga，用于监听 action
export function * userInfo () {
  // 监听 type 为 FETCH_USER_INFO 的 action
  yield takeEvery('FETCH_USER_INFO', fetchUserInfo)
  /**
  * 可以监听多个 saga
  */
}


// 开启所有的监听器 saga
export default function * rootSaga () {
  yield all([
    userInfo()
  ])
}
```

在上面的代码中，分别有 **Worker Saga， Watcher Saga 和 最后的 rootSaga**  

Worker Saga 用于执行真正的任务，在这里可以使用 *call* 来发送请求，可以通过 *put* 来调用 reducer  

Watcher Saga 用于监听 action, 当 dispatch 到所监听的 action 时，就会调用对应的 Worker Saga  

rootSaga 则用于启动 Watcher Saga  


**通常对于异步请求的流程是:**  

页面 dispatch action => Watcher Saga 监听到 action，启动 Worker Saga => Worker Saga 发起异步请求，得到结果之后 put 一个 action => 这个 action 被 reducers 进行处理


reducer.js

```js
const reducer = (state = initialState, action = {}) => {
  switch (action.type) {
    case 'SET_USER_INFO':
      console.log(action)
      return {
        ...state,
        userInfo: { ...action.payload }
      }
    case 'CHANGE_YOU_COLOR':
      return {
        ...state,
        color: action.payload
      }
    default:
      return state
  }
}
```

**注意:**  

reducer 和 saga 如果监听到相同的 action, **他们两个都会执行**，为了避免状态流转混乱，应避免在 reducer 和 saga 中监听相同的 action。  


## 2. 实践

### 2.1 takeEvery 和 takeLatest  

```js
export function * userInfo () {
  yield takeEvery('FETCH_USER_INFO', fetchUserInfo)
  // 可以监听多个 action
  yield takeEvery('UPDATE_USER_INFO', updateUserInfo)
}
```

以上的例子中，每监听到 FETCH_USER_INFO， 就会调用 fetchUserInfo，尽管前一次的 fetchUserInfo 可能还没有执行完，这在某些情况下是不必要的。  

因此，有了 takeLatest 方法，这个方法 **如果监听到新的 FETCH_USER_INFO**，而前一次的 fetchUserInfo 还没有执行完，那么就会**取消前一次的 fetchUserInfo**。

### 2.2 使用 call 而不是直接请求

```js
function* watchFetchProducts() {
  yield takeEvery('PRODUCTS_REQUESTED', fetchProducts)
}


// 不推荐
function* fetchProducts() {
  // Api.fetch('/products') 为一个 promise
  const products = yield Api.fetch('/products')
}

// 推荐
function* fetchProducts() {
  const products = yield call(Api.fetch, '/products')
}
```

在上面的例子中，可以使用两种方式进行 api 的请求，但是更推荐使用第二种。**这会使得你的代码更容易被测试(暂时没理解)**  

### 2.3 使用 put 而不是直接 dispatch action

```js
function* fetchProducts(dispatch)
  const products = yield call(Api.fetch, '/products')
  
  // 不推荐
  dispatch({ type: 'PRODUCTS_RECEIVED', products })
  
  // 推荐
  yield put({ type: 'PRODUCTS_RECEIVED', products })
}
```

理由同样是便于测试

### 2.4 select 来获取当前 store 的 state

```js
// 假如 root state 为 { count : 0 }
// 则可以使用 
const count = yield select(state => state.count) 来获得 count 的值


// 使用 yield select() 来获取所有的 state
export function * fetchUserInfo (action) {
  const data = yield select()
  console.log('data', data)
}
```

## 常见错误

1. regeneratorRuntime is not defined  

问题在于当前不认识 saga 的 generator 语法, 查看 [issue](https://github.com/redux-saga/redux-saga/issues/280#issuecomment-593941552)

## 参考资料

1. [Redux Saga 中文文档](https://redux-saga-in-chinese.js.org/)
2. [Redux Saga 简单例子](https://github.com/yes1am/redux-demo/tree/redux-saga)