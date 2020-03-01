通常我们都会看到或者听到这个概念，即 **Redux 只能有一个 store 对象**  

比如以下的问题:  
[Redux-react：关于redux只允许有一个store对象的问题](http://react-china.org/t/redux-react-redux-store/3089)

但根据对 Redux 目前的理解来看，Redux 就是创建一个 store 对象。然后通过 React-Redux 的 Provider 去将 store 的值作为 Context 的提供者，内部使用 connect() 来接收 Context 的值。  

而本身，**React 的 Context 是可以嵌套的，它会找到最新的 Provider  提供的值**:  

```js
const ThemeContext = React.createContext('name')

function A (props) {
  return <ThemeContext.Provider value='a'>
    <ThemeContext.Consumer>
      {name => {
        return <div>A: {name}                  // a
          <ThemeContext.Provider value='b'>
            <ThemeContext.Consumer>
              {name => {
                return <div>B: {name} </div>    // b
              }}
            </ThemeContext.Consumer>
          </ThemeContext.Provider>
        </div>
      }}
    </ThemeContext.Consumer>
  </ThemeContext.Provider>
}

function App () {
  return <div>
    app
    <A />
  </div>
}
```

即以上代码是行得通的，会各自读取最近的 Provider 的值，似乎只能有一个 store 是说不通的。

为了严谨性，我还尝试了 redux 的嵌套

```js
// 组件 A
const foo = (state = { foo: 0 }, action = {}) => {
  switch (action.type) {
    case 'ADD_FOO':
      return Object.assign({}, state, {
        foo: state.foo + action.payload
      })
    default:
      return state
  }
}

const mapStateToPropsA = state => ({
  state: state
})

const store1 = createStore(foo)

class A extends React.Component {
  render () {
    return <div>
      <div onClick={() => this.props.dispatch({ type: 'ADD_FOO', payload: 1 })}>A { JSON.stringify(this.props.state)}</div>   // { foo : 0 }
      { this.props.children}
    </div>
  }
}

const ACom = connect(mapStateToPropsA)(A)

// 组件 B
const bar = (state = { bar: 0 }, action = {}) => {
  switch (action.type) {
    case 'ADD_BAR':
      return Object.assign({}, state, {
        bar: state.bar + action.payload
      })
    default:
      return state
  }
}

const store2 = createStore(bar)

class B extends React.Component {
  render () {
    return <div onClick={() => this.props.dispatch({ type: 'ADD_BAR', payload: 2 })}>
    B { JSON.stringify(this.props.state)}    // { bar : 0 }
    </div>
  }
}

const mapStateToPropsB = state => ({
  state: state
})

const BCom = connect(mapStateToPropsB)(B)

// 最终渲染结果
ReactDOM.render(
  <Provider store={store1}>
    <ACom>
      <Provider store={store2}>
        <BCom />
      </Provider>
    </ACom>
  </Provider>,
  document.getElementById('root')
)
```

在以上这个例子中，进行了两个 store 的嵌套，最终结果证明，**他们是可以独立且正常工作的**。  

这时候我继续查询资料，想找到有说服力的文档，来证明我的观点，终于找到了[文档资料](https://cn.redux.js.org/docs/faq/StoreSetup.html)。  

>  “可能” 在一个页面中创建多个独立的 Redux store，但是预设模式中只会有一个 store。仅维持单个 store 不仅可以使用 Redux DevTools，还能简化数据的持久化及深加工、精简订阅的逻辑处理。
>   
> 在 Redux 中使用多个 store 的理由可能包括：
> - 对应用进行性能分析时，解决由于过于频繁更新部分 state 引起的性能问题。
> - 在更大的应用中 Redux 只是作为一个组件，这种情况下，你也许更倾向于为每个根组件创建单独的 store。  
>
> 然而，创建新的 store 不应成为你的第一反应，特别是当你从 Flux 背景迁移而来。首先尝试组合 reducer，只有当它无法解决你的问题时才使用多个 store。

即，**可以创建多个 store，但是通常不建议这样做**，在理由二中提到，**在更大的应用中 Redux 只是作为一个组件，这时候可以为每个组件创建一个单独的 store**。  

**因此，当你的组件是作为独立的组件时，是可以创建单独的一个 store，而不用担心调用方已经存在了 store 的问题。**
