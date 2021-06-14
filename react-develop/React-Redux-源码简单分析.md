# React Redux 原理

简单的使用示例:  
```js
import { Provider, connect } from './react-redux/src'
import { createStore } from './redux/src'
import rootReducer from './reducers'

const store = createStore(rootReducer)

function Child(props) {
  return <div>
    <div>name: {props.name}</div>
    <div>age: {props.age}</div>
    <button onClick={() => props.dispatch({
        type: 'CHANGE_NAME',
        data: Math.random()
      })}
    >
      change name
    </button>
    <button onClick={() => props.dispatch({
        type: 'CHANGE_AGE',
        data: Math.random()
      })}
    >
      change age
    </button>
  </div>
}

const mapStateToProps = state => {
  return {
    name: state.name,
    age: state.age
  }
}

const mapDispatchToProps = dispatch => ({
  onClick: () => dispatch()
})

const ConnectChild = connect(
  mapStateToProps,
  // mapDispatchToProps
)(Child)

function Demo() {
  return <Provider store={store}>
    <ConnectChild />
  </Provider>
}
export default Demo;
```

## 1. Redux 源码

源码：https://github.com/reduxjs/redux/tree/v4.1.0

```js
/Users/songjp/fe/mime/workshop/packages/react-basic/src/react-redux-demo/redux/src
├── applyMiddleware.js
├── bindActionCreators.js
├── combineReducers.js
├── compose.js
├── createStore.js
├── index.js
└── utils
   ├── actionTypes.js
   ├── formatProdErrorMessage.js
   ├── isPlainObject.js
   ├── kindOf.js
   ├── symbol-observable.js
   └── warning.js
```

*index.js* 导出了以下内容:  
```js
export {
  createStore,
  combineReducers,
  bindActionCreators,
  applyMiddleware,
  compose
}
```

### 1.1 createStore

源码简化版:

```js
import ActionTypes from './utils/actionTypes'

export default function createStore(reducer, preloadedState, enhancer) {
  let currentReducer = reducer
  let currentState = preloadedState
  let currentListeners = []
  let nextListeners = currentListeners

  function getState() {
    return currentState
  }

  function subscribe(listener) {
    nextListeners.push(listener)

    return function unsubscribe() {
      const index = nextListeners.indexOf(listener)
      nextListeners.splice(index, 1)
    }
  }

  function dispatch(action) {
    currentState = currentReducer(currentState, action)
    const listeners = (currentListeners = nextListeners)
    for (let i = 0; i < listeners.length; i++) {
      const listener = listeners[i]
      listener()
    }

    return action
  }

  dispatch({ type: ActionTypes.INIT })

  return {
    dispatch,
    subscribe,
    getState,
  }
}
```

1. createStore 创建了一个 store 对象，含有 `dispatch`, `subscribe`, `getState` 等方法
2. 大致原理是，内部通过闭包维护一个 `currentState` 属性，`nextListeners` 属性。`nextListeners` 对应 **观察者模式** 中的观察者列表。
3. `subscribe` 方法可以往 `nextListeners` 中添加观察者
4. 当调用 `dispatch` 方法是，会执行 `currentState = currentReducer(currentState, action)` 改变当前的 `currentState` 属性。之后会通知到所有的观察者去执行。
5. 而 `getState` 方法可以获取最新的 `currentState` 方法。

## 2. React-Redux 源码

源码地址: https://github.com/reduxjs/react-redux/tree/v6.0.0

*在 React-Redux 7.1 版本之后，引入 hooks 写法，老实说，源码可阅读性有点下降(是的，我没看懂...)，因此，我们还是看旧版本的代码吧，大致思想应该是一致的*

### 2.1 Provider 源码

*源码简化版*:  
```js
class Provider extends Component {
  constructor(props) {
    const { store } = props
    this.state = {
      storeState: store.getState(),
      store
    }
  }

  componentDidMount() {
    this.subscribe()
  }

  subscribe() {
    const { store } = this.props

    store.subscribe(() => {
      const newStoreState = store.getState()
      this.setState(providerState => {
        // If the value is the same, skip the unnecessary state update.
        if (providerState.storeState === newStoreState) {
          return null
        }
        return { storeState: newStoreState }
      })
    })
  }

  render() {
    const Context = this.props.context || ReactReduxContext

    return (
      <Context.Provider value={this.state}>
        {this.props.children}
      </Context.Provider>
    )
  }
}
```

1. Provider 内部有 `storeState` 和 `store` 两个 state
2. 在 componentDidMount 时调用 `store.subscribe()`, 内部会重新 `setState({ storeState })`. 即当我们在子组件调用 dispatch 方法时, 会更新 Provider 组件的 storeState 状态。
3. Provider 再通过 Context 将自己的 state 传递到子组件中


### 2.2 connectAdvanced.js 源码

```js
class Connect extends OuterBaseComponent {
  constructor(props) {
    super(props)
    invariant(
      forwardRef ? !props.wrapperProps[storeKey] : !props[storeKey],
      'Passing redux store in props has been removed and does not do anything. ' +
        customStoreWarningMessage
    )
    this.selectDerivedProps = makeDerivedPropsSelector()
    this.selectChildElement = makeChildElementSelector()
    this.renderWrappedComponent = this.renderWrappedComponent.bind(this)
  }

  renderWrappedComponent(value) {
    const { storeState, store } = value

    let wrapperProps = this.props
    let forwardedRef

    if (forwardRef) {
      wrapperProps = this.props.wrapperProps
      forwardedRef = this.props.forwardedRef
    }

    // 在这里，会根据 matchStateToProps 以及 matchDispatchToProps
    // 计算出最终需要传递给子组件的 props, 比如在我们的示例中是 { name, age, dispatch }
    // 不过 selectDerivedProps 这个方法确实有点绕，代码看晕了
    let derivedProps = this.selectDerivedProps(
      storeState,
      wrapperProps,
      store
    )
    return this.selectChildElement(derivedProps, forwardedRef)
  }

  render() {
    const ContextToUse = this.props.context || Context

    return (
      <ContextToUse.Consumer>
        {this.renderWrappedComponent}
      </ContextToUse.Consumer>
    )
  }
}
```

- 被 `connect()(Component)` 包裹的组件, 会通过 `selectDerivedProps(storeState, store)` 从 storeState 中得到你需要的属性 derivedProps (通过计算你的 **mapStateTopProps** 和 **mapDispatchToState** 得出)

## 3. 原理总结

1. 在 Redux 库中，createStore() 返回 store 对象，对象上有 `{ getState, dispatch, getState }` 等属性 ，该对象传给 React-Redux 的 Provider 组件
2. React-Redux 中的 Provider 组件中通过 store.subscribe 监听 store 的**状态数据**变化。每次**数据**变化后都会把**状态数据**用 Context API 传递到子组件
3. 当调用 dispatch 方法时，在 Redux 中会通过 `currentState = currentReducer(currentState, action)` 也就是"过一遍" reducer 方法，得到最新的**状态数据**
4. 此时子组件会得到最新的 state 信息，通过 `selectDerivedProps(storeState, store)` 来得到最终的 props，进行渲染。另外，如果发现从 store 中接受到的 props 没有发生变化，则不会重新渲染。


### 3.1 流程

Redux 中的三个概念:  

1. dispatch
2. reducer
3. state

至于 Action Creator, 实际上就是为了便捷的创建 action，**有更好，没有也行**。

```js
用户的交互，触发 dispatch(aciton)  => 通过 reducer(currentState, action) 进行计算，得到最新的 state => 当 state 发生变化时，connect 中会重新计算需要赋值给子组件的 props，并重新渲染子组件
```

## 4. TODO

整理 compose，applyMiddleware，combineReducers，以及中间件的一个原理

