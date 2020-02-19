## 1. 简介

React Hooks 允许在不使用 class 语法的情况下，使用 state 及其他 react 的特性。  

*示例*
```js
function App () {
  const [count, setCount] = useState(0)  // 初始 state 为 0
  return <div onClick={() => setCount(count + 1)}>
    app count {count}
  </div>
}
```

## 2. 概览

useEffect 类似原来的 `componentDidMount`, `componentDidUpdate` 和 `componentWillUnmount` 的集合。  

```js
function App () {
  const [count, setCount] = useState(0)
  useEffect(() => {
    document.title = `count ${count}`
  })
  return <div onClick={() => setCount(count + 1)}>
    app count {count}
  </div>
}
```

react 在修改完 dom 之后(state 更新会触发dom修改)，会执行 useEffect 里的函数(副作用函数)。默认情况下， useEffect 在 react 每次修改 dom 之后都会执行，包括**第一次渲染**。  

副作用函数本身可以返回一个函数，用于 **清除副作用函数**，如事件移除。  

```js
function App () {
  const [count, setCount] = useState(0)

  function clickHandler () {
    console.log('click')
  }
  useEffect(() => {
    document.addEventListener('click', clickHandler)
    
    // 返回该函数，用于清理副作用函数
    return () => {
      console.log('unscribe')
      document.removeEventListener('click', clickHandler)
    }
  })
  return <div onClick={() => setCount(count + 1)}>
    app count {count}
  </div>
}

点击 div 之外，触发点击事件，打印 'click'。    // 多次点击打印: ['click'],['click'],['click']

点击 div，触发点击事件，打印 'click', 同时执行 `setCount` 触发 dom 更新，在更新 dom 之前会执行 **清理函数**， 打印 'unscribe'。
// 多次点击打印: ['click', 'unscribe'],['click','unscribe'],['click','unscribe']
```

### 2.1 Hooks 使用规则

***Hook 就是 JavaScript 函数***，可以进行**多次调用**。但是  


1. 只能在**函数最外层**调用 Hook。**不要在循环、条件判断或者子函数**中调用。
2. 只能在 **React 的函数组件** 中调用 Hook。不要在其他 JavaScript 函数中调用。（还有一个地方可以调用 Hook —— 就是**自定义的 Hook 中**，我们稍后会学习到。）

### 2.2 自定义 Hooks

在 Hooks 之前，我们通过[高阶组件](https://zh-hans.reactjs.org/docs/higher-order-components.html)或者[render props](https://zh-hans.reactjs.org/docs/render-props.html) 来在组件中复用状态，而自定义 Hook 可以在**不增加组件**的情况下，达到同样的目的。  

如，有一个自定义 Hooks `useTime`，可以每隔 1000s, 返回当前的时间，那么:  

```js
// useTime Hooks
function useTimer (defaultValue) {
  const [value, setValue] = useState(defaultValue)

  // 会改变 value 的函数
  setTimeout(() => {
    setValue(Date.now())
  }, 1000)

  return value
}


// App 组件

function App (props) {
  const now = useTimer(0)  // 这样使用，得到最新的返回值
  return <div>
    time now: {now}
  </div>
}
```

另外，Hooks 是复用**状态逻辑**，而不是复用 state 本身，事实上，Hook 的每一次调用，都有一个完全独立的 state。因此，你可以在一个组件中**多次调用同一个自定义 Hooks**  

```
function App (props) {
  const now = useTimer(0)
  const now1 = useTimer(0)
  return <div>
    // now 和 now1 并没有什么关系，是完全独立的两个值，所以不一定相等
    // 但又因为，都是表示当前时间，他们大部分时候值是相等的，但他们仍然是独立的 state
    time now: {now}, {now1}  
  </div>
}
```

自定义 Hooks 更像是一种约定，而不是功能.  

**如果函数名以 "use" 开头，且在函数内部调用了其它的 Hooks，我们就说这是一个自定义 Hook**。  

### 2.3. useContext

在之前，我们使用 Context.Consumer 来接受 Context.Provider 的值:  

```
const ThemeContext = React.createContext('light')

function App (props) {
  return <div>
    <ThemeContext.Provider value={'dark'}>
      <Child />
    </ThemeContext.Provider>
  </div>
}

function Child () {
  return <ThemeContext.Consumer>
    {theme => {
      return <div> theme: {theme}</div>
    }}
  </ThemeContext.Consumer>
}
```

而使用 useContext 之后，可以替换 Context.Consumer:  

```js
const ThemeContext = React.createContext('light')

function App (props) {
  return <div>
    <ThemeContext.Provider value={'dark'}>
      <Child />
    </ThemeContext.Provider>
  </div>
}

function Child () {
  const theme = useContext(ThemeContext)
  return <div>
      theme: {theme}
  </div>
}
```
即，useContext 接受一个 `React.createContext()` 的返回值 Context 对象作为参数，返回 Context 的值。  

**注意:**  
**useContext 只是用来替代 Context.Consumer 的，上层组件依然需要使用 Context.Provider**  

### 2.4 useReducer

注意:  
**useReducer 是 useState 的替代方案。也就是说它是涉及来替代 useState 的一些复杂场景(*比如 state 逻辑复杂，包含子 state 的场景*)，和 React Redux 没有关系** 

useReducer 接受一个 reducer 函数: `(state, action) => newState`, 返回当前的 state，以及 dispatch 方法。

我们来看看计数器的例子，分别使用 useState 和 useReducer:  

```js
// useState 的示例
function App (props) {
  const [count, setCount] = useState(0)
  return (
    <>
      Count: {count}
      <button onClick={() => setCount(count - 1)}>-</button>
      <button onClick={() => setCount(count + 1)}>+</button>
    </>
  )
}

// useReducer 的示例
const initialState = { count: 0 }

function reducer (state, action) {
  switch (action.type) {
    case 'increment':
      return { count: state.count + 1 }
    case 'decrement':
      return { count: state.count - 1 }
    default:
      throw new Error()
  }
}

function App (props) {
  const [state, dispatch] = useReducer(reducer, initialState)
  return (
    <>
      Count: {state.count}
      <button onClick={() => dispatch({ type: 'decrement' })}>-</button>
      <button onClick={() => dispatch({ type: 'increment' })}>+</button>
    </>
  )
}
```

看起来 useReducer 实现起来代码更多，但是，**在 state 较为复杂的情况下，useReducer 逻辑更为清晰**  

> **useReducer 是为了可以让你通过 reducer 来管理组件本地的复杂 state。**

useReducer 的简易源码:  

```js
function useReducer(reducer, initialState) {
  const [state, setState] = useState(initialState);

  function dispatch(action) {
    const nextState = reducer(state, action);
    setState(nextState);
  }

  return [state, dispatch];
}
```

## 3. 使用 State Hook

React 函数式组件的两种写法:  

```
const Example = (props) {
    // 你可以在这里使用 Hooks
    return <div />
}

或者

function Example(props) {
    // 你可以在这里使用 Hooks
    return <div />
}
```

以前我们把这些组件也称为**无状态组件**，因为他们没有内部的 state，但是现在，通过引入 **React Hooks**，我们可以在函数式组件内部使用 state。

因此更准确的说法应该是**函数式组件** 而不是**无状态组件**。  

**State Hooks 的目的就是给函数式组件添加 state**。  

## 4. 使用 Effect Hook

Effect Hook 可以让你在函数组件中执行**副作用**操作。  

数据获取，设置订阅以及手动更改 React 组件中的 DOM 都属于副作用。


如果你熟悉 React Class 中的生命周期，你可以把 Effect Hook 看作是 componentDidMount, componentDidUpdate, componetWillUnmout 这三个函数的组合  

传统 Class 组件: 

```js
class App extends React.Component {
  constructor (props) {
    super(props)
    this.state = {
      count: 0
    }
  }
  componentDidMount () {
    const { count } = this.state
    document.title = `You clicked ${count} times`
  }

  componentDidUpdate () {
    const { count } = this.state
    document.title = `You clicked ${count} times`
  }

  render () {
    const { count } = this.state
    return <div onClick={() => this.setState({
      count: count + 1
    })}>
    app
    </div>
  }
}
```

而使用 hook，我们可以重写该组件:  

```js
function App (props) {
  const [count, setCount] = useState(0)

  // 该函数在 didMount 和 didUpdate 的时候执行
  useEffect(() => {
    document.title = `You clicked ${count} times`
  })

  return (
    <div onClick={() => setCount(count + 1)}>
      app
    </div>
  )
}
```

那为什么 Effect 还会相当于 componentWillUnmount 呢？  

首先，副作用包含，**需要清除的副作用** 和 **无需清除的副作用**。  

需要清除的副作用: 比如事件监听，比如订阅的功能

无需清除的副作用:  比如发起请求，手动更改 DOM，或者记录日志，即执行完这些操作，我们就可以忽略它们了。  

Effect 的另一个功能，就是可以通过返回一个**清除副作用函数**来清除副作用。  

```js
// Class 组件清除副作用示例
class Example extends React.Component {
    componentDidMount() {
        document.addEventListener('click', this.handleClick)
    }
    
    componentDidMount() {
        document.removeEventListener('click', this.handleClick)
    }
}

// Effect 清除副作用示例
function Example() {
    useEffect(() => {
        document.addEventListener('click', this.handleClick)
        return () => {
            document.removeEventListener('click', this.handleClick)
        }
    })
}
```

**注意，返回清除副作用的函数是可选的，如果副作用不需要清除，那么可以不返回**  


### 4.1 使用 Effect 的提示

1. 使用多个 Effect 实现关注点分离

例如，一个副作用是改变 title，一个副作用是添加事件监听，那么**可以使用两个单独的 useEffect 来更好的管理代码。**  


```js
function App() {
    useEffect(() => {
        document.title = xxx;
    })
    
    useEffect(() => {
        document.addEventListener('click', this.handleClick)
        
        return () => {
            document.removeEventListener('click', this.handleClick)
        }
    })
}
```

2. [为什么每次更新的时候都要运行 Effect](https://zh-hans.reactjs.org/docs/hooks-effect.html#explanation-why-effects-run-on-each-update)


3. 如何不让 Effect 每次执行

前面提到，useEffect 每次都会执行，这在有些情况下，是没有必要的，那么如何才能限制 useEffect 的执行呢？  

在 Class 组件中，例如，我们通过判断 state 是否改变，来觉得是否执行副作用函数:  

```js
componentDidUpdate(prevProps, prevState) {
  if (prevState.count !== this.state.count) {
    document.title = `You clicked ${this.state.count} times`;
  }
}
```

而在 Effect Hook 中，我们也可以使用 useEffect 的**第二个参数来设置 effect 的依赖**，当依赖没有变化时，就不会执行 useEffect 里的函数.  

```js
function App (props) {
  const [count, setCount] = useState(0)
  const [count1] = useState(0)

  useEffect(() => {
    document.title = `You clicked ${count} times`
  })

  return (
    <div onClick={() => setCount(count + 1)}>
      app count: {count}, count1: {count1}
    </div>
  )
}
```

上面这个例子中，useEffect 没有依赖，因此每次 `setCount` 都会执行内部的函数。  

```js
function App (props) {
  const [count, setCount] = useState(0)
  const [count1] = useState(0)

  useEffect(() => {
    document.title = `You clicked ${count} times`
  }, [count1])        // 设置依赖，只有第一次会执行
  
  // 如果是这样
  useEffect(() => {
    document.title = `You clicked ${count} times`
  }, [count])        // 因为 count 一直变化，所以也是每次都会执行

  return (
    <div onClick={() => setCount(count + 1)}>
      app count: {count}, count1: {count1}
    </div>
  )
}
```
而通过设置 count1 作为依赖，只有第一次渲染的时候会执行该函数，之后只有在 count1 发生变化时，才会执行内部函数。

如果只想在第一次渲染时执行，不依赖任何外部的值，那么可以传递**空数组**。这样就相当于是 componentDidMount  

```js
function App (props) {
  const [count, setCount] = useState(0)

  useEffect(() => {
    document.title = `You clicked ${count} times`
  }, [])  // 空数组表示没有依赖，只会在第一次渲染时执行

  return (
    <div onClick={() => setCount(count + 1)}>
      app count: {count}
    </div>
  )
}
```

## 5. Hooks 规则

1. 只在最顶层使用 Hooks，如 useState，useEffect，**不要在循环，条件或者嵌套函数中调用 Hook**，因为 **React 依赖 Hook 调用的顺序**来保证正确的执行。  

不要这样:  

```js
if (name !== '') {
    useEffect(function persistForm() {
      localStorage.setItem('formData', name);
    });
}
```
因为该 Hook 的执行取决于 name 的值，当 name 发生变化时， Hook 的执行顺序发生变化，因此会导致 bug。  

如果我们想要有条件的执行 effect，我们**可以将判断逻辑转移到 useEffect 的内部**:  

```js
useEffect(function persistForm() {
    if (name !== '') {
      localStorage.setItem('formData', name);
    }
});
```

## 6. 自定义 Hook

例如，有一个 FrientStatue 组件:  

```js
function FriendStatus(props) {
  const [isOnline, setIsOnline] = useState(null);

  useEffect(() => {
    function handleStatusChange(status) {
      setIsOnline(status.isOnline);
    }

    ChatAPI.subscribeToFriendStatus(props.friend.id, handleStatusChange);
    return () => {
      ChatAPI.unsubscribeFromFriendStatus(props.friend.id, handleStatusChange);
    };
  });

  if (isOnline === null) {
    return 'Loading...';
  }
  return isOnline ? 'Online' : 'Offline';
}
```

和一个 FriendItem 组件:  

```js
function FriendListItem(props) {
  const [isOnline, setIsOnline] = useState(null);

  useEffect(() => {
    function handleStatusChange(status) {
      setIsOnline(status.isOnline);
    }

    ChatAPI.subscribeToFriendStatus(props.friend.id, handleStatusChange);
    return () => {
      ChatAPI.unsubscribeFromFriendStatus(props.friend.id, handleStatusChange);
    };
  });

  return (
    <li style={{ color: isOnline ? 'green' : 'black' }}>
      {props.friend.name}
    </li>
  );
}
```

以上两个组件都是接受 friend 作为 props, 内部通过查询 API 获取 friend
的状态。根据 friend 的状态 渲染不同的结果。

因此，我们可以使用 **自定义 Hook** 抽离出公共的 Hook 来。  

```
// 自定义 Hooks， useFriendStatus  
function useFriendStatus(friendID) {
  const [isOnline, setIsOnline] = useState(null);

  useEffect(() => {
    function handleStatusChange(status) {
      setIsOnline(status.isOnline);
    }

    ChatAPI.subscribeToFriendStatus(friendID, handleStatusChange);
    return () => {
      ChatAPI.unsubscribeFromFriendStatus(friendID, handleStatusChange);
    };
  });

  return isOnline;
}

// 改写 FriendStatus 组件
function FriendStatus(props) {
  if (isOnline === null) {
    return 'Loading...';
  }
  return isOnline ? 'Online' : 'Offline';
}

// 改写 FriendItem 组件
function FriendListItem(props) {
  const isOnline = useFriendStatus(props.friend.id);

  return (
    <li style={{ color: isOnline ? 'green' : 'black' }}>
      {props.friend.name}
    </li>
  );
}
```  

另外，与 React 组件不同的是，自定义 Hook 不需要具有特殊的标识。**我们可以自由的决定它的参数是什么，以及它应该返回什么（如果需要的话）**。  

换句话说，它就像一个正常的函数( **当然它内部使用 Hooks 也应该在顶层代码中** )。但是它的名字应该**始终以 use 开头，这样便于使用[插件](https://www.npmjs.com/package/eslint-plugin-react-hooks)判断其是否符合 Hook 的规则**.  

**同时，我们可以在多个 Hook 之前传递信息**：  

```js
function ChatRecipientPicker() {
  const [recipientID, setRecipientID] = useState(1);
  const isRecipientOnline = useFriendStatus(recipientID);

  return (
    <>
      <Circle color={isRecipientOnline ? 'green' : 'red'} />
      <select
        value={recipientID}
        onChange={e => setRecipientID(Number(e.target.value))}
      >
        {friendList.map(friend => (
          <option key={friend.id} value={friend.id}>
            {friend.name}
          </option>
        ))}
      </select>
    </>
  );
}
```

如上，**我们在 useState 与自定义 Hooks 组件传递信息**，当 通过下拉框，改变 recipientID 时，useFriendStatus **内部会重新订阅新的 recipientID**，来返回最新的 recipientID 对应的状态。  


**尽量避免过早地增加抽象逻辑。(优化都是在实现功能之后考虑的事)**  


## 7. Hook API 索引

### 7.1 useReducer

在前面的例子中，我们已经说过 useReducer 的用法了:  

```js
const initialState = { count: 0 }

function reducer (state, action) {
  switch (action.type) {
    case 'increment':
      return { count: state.count + 1 }
    case 'decrement':
      return { count: state.count - 1 }
    default:
      throw new Error()
  }
}

function App (props) {
  const [state, dispatch] = useReducer(reducer, initialState)
  return (
    <>
      Count: {state.count}
      <button onClick={() => dispatch({ type: 'decrement' })}>-</button>
      <button onClick={() => dispatch({ type: 'increment' })}>+</button>
    </>
  )
}
```
useReducer 接受 reducer， 以及初始的 state。而有时候，我们还可以选择**惰性**的初始化 state:  

```js
function init (initialCount) {
  return { count: initialCount }
}

function reducer (state, action) {
  switch (action.type) {
    case 'increment':
      return { count: state.count + 1 }
    case 'decrement':
      return { count: state.count - 1 }
    case 'reset':
      // 便于重置 state
      return init(action.payload)
    default:
      throw new Error()
  }
}

function App ({ initialCount = 0 }) {
  // 通过给 useReducer 传递第三个 init 函数
  // 这样，初始的 state 就是 init(initialCount) 的返回值
  const [state, dispatch] = useReducer(reducer, initialCount, init)
  return (
    <>
      Count: {state.count}
      <button
        // 便于重置 state
        onClick={() => dispatch({ type: 'reset', payload: initialCount })}>

        Reset
      </button>
      <button onClick={() => dispatch({ type: 'decrement' })}>-</button>
      <button onClick={() => dispatch({ type: 'increment' })}>+</button>
    </>
  )
}
```

这样做的好处:  
1. 可以将计算初始 state 的逻辑移到组件外面
否则的话:  

```js
function App ({ initialCount = 0 }) {
  const initialState = {count: initialCount}  // 计算 state 的逻辑混在组件内部
  const [state, dispatch] = useReducer(reducer, initialState)
  return (
    <>
      ..
    </>
  )
}
```

2. 便于重置 state (在 demo 中可见)

## 7.2 useMemo 和 useCallback

这两个函数都是用来实现**缓存**的

### 7.2.1 useMemo 

```js
function App ({ initialCount = 0 }) {
  const [count, setCount] = useState(1)
  const [val, setValue] = useState('')

  function expensive () {
    // 每次都会执行
    console.log('compute')
    return count
  }

  return <div>
    <h4>{count}-{val}-{expensive()}</h4>
    <div>
      <button onClick={() => setCount(count + 1)}>+c1</button>
      <input value={val} onChange={event => setValue(event.target.value)} />
    </div>
  </div>
}
```

在上面这个例子中，无论是 setValue 还是 setCount 执行，都会触发 expensive 函数的执行，而 expensive 本身只依赖于 count 的值，但目前就算 setValue 也会执行 expensive 函数。  

**如果 expensive 函数计算非常耗时，那么将是非常没有必要的**，因此可以使用 useMemo 进行优化:  

```js
function App ({ initialCount = 0 }) {
  const [count, setCount] = useState(1)
  const [val, setValue] = useState('')

  const expensive = useMemo(() => {
    // 只有 count 变化才会执行
    console.log('compute')
    return count
  }, [count])

  return <div>
    <h4>{count}-{val}-{expensive}</h4>
    <div>
      <button onClick={() => setCount(count + 1)}>+c1</button>
      <input value={val} onChange={event => setValue(event.target.value)} />
    </div>
  </div>
}
```

**useMemo 接受一个函数和依赖，返回一个值。第一次渲染时会执行，只有发生变化时，才会重新执行函数，返回新的值。** 

**如果依赖为空，那么内部函数只会在第一次渲染时执行**  

### 7.2.2 useCallback

与 useMemo 不同，useMemo 缓存的是值，而 useCallback 缓存函数。  

useCallback 接受一个**函数作为参数**和一个依赖数组，**缓存该函数**并进行返回。

当依赖发生变化时，会返回新的函数，否则，返回的还是原来的函数。  

```
// 利用 set 中，没有重复元素的特性，如果有重复，那么会删除多余的
const set = new Set()

function App ({ initialCount = 0 }) {
  const [count, setCount] = useState(1)
  const [val, setVal] = useState('')

  const expensive = useCallback(() => {
    console.log('compute')
  }, [count])
  set.add(expensive)
  
  // 一开始，打印 1 
  // 当改变 count 时，callback 执行，expensive 是新的函数，set.size 增加
  // 当改变 value 时，callback 不会执行，expensive 还是旧的函数，set.size 不会增加
  console.log(set.size)

  return <div>
    <div>
      count: {count}
      <button onClick={() => setCount(count + 1)}>+1</button>
      value: {val}
      <input value={val} onChange={event => setVal(event.target.value)} />
    </div>
  </div>
}
```

使用场景就是，将 useCallback 的返回值，作为参数传给子组件。子组件再将该参数，作为依赖，来决定是否执行其他逻辑。    

```
function Child (props) {

  // 除了第一次初始化之外，
  // 只有在父组件改变 count 的时候，才会执行 useEffect 里面的代码
  useEffect(() => {
    // do something with props.callback
  }, [props.callback])   // 
  return <div> child </div>
}

function App ({ initialCount = 0 }) {
  const [count, setCount] = useState(1)
  const [val, setVal] = useState('')

  // 只有改变 count 的时候，会改变传给 Child 的 callback 值
  const expensive = useCallback(() => {
    console.log('compute')
  }, [count])

  return <div>
    <div>
      count: {count}
      <button onClick={() => setCount(count + 1)}>+1</button>
      <input value={val} onChange={event => setVal(event.target.value)} />
      <Child callback={expensive} />
    </div>
  </div>
}
```

或者是，利用引用相等性，减少子组件的渲染：  

```js
class Child extends React.Component {
  shouldComponentUpdate (nextProps) {
    return nextProps.callback !== this.props.callback
  }
  render () {
    // 只有第一次渲染，以及父组件改变 count 时会执行 render
    // 父组件改变 val 不会导致重新渲染
    console.log('render')
    return <div> child </div>
  }
}

function App ({ initialCount = 0 }) {
  const [count, setCount] = useState(1)
  const [val, setVal] = useState('')

  const expensive = useCallback(() => {
    console.log('compute')
  }, [count])

  return <div>
    <div>
      count: {count}
      <button onClick={() => setCount(count + 1)}>+1</button>
      <input value={val} onChange={event => setVal(event.target.value)} />
      <Child callback={expensive} />
    </div>
  </div>
}
```

此外， `useCallback(fn, deps) 相当于 useMemo(() => fn, deps)`, 因为 useMemo 缓存值，而函数也是值，因此useMemo 也可以缓存函数。

**但反过来不行，因为值不一定是函数(有可能是数字，字符串等)。只有在值是函数时，可以改写。**

使用 useMemo 代替 useCallback，改写上面的例子, 实现一样的效果:  

```js
class Child extends React.Component {
  shouldComponentUpdate (nextProps) {
    return nextProps.callback !== this.props.callback
  }
  render () {
    console.log('render')
    return <div> child </div>
  }
}

function App ({ initialCount = 0 }) {
  const [count, setCount] = useState(1)
  const [val, setVal] = useState('')

  const expensive = useMemo(() => {
    return () => {
      console.log('compute')
    }
  }, [count])

  return <div>
    <div>
      count: {count}
      <button onClick={() => setCount(count + 1)}>+1</button>
      <input value={val} onChange={event => setVal(event.target.value)} />
      <Child callback={expensive} />
    </div>
  </div>
}
```

### 7.3 useRef

`const inputRef = useRef(initialValue)`  

useRef 返回一个可变的(.current 的值可变)  ref 对象，其 `.current` 属性会被初始化为 initialValue。返回的 **ref对象** 在整个生命周期中不会变。

```js
function App ({ initialCount = 0 }) {
  // inputRef.current 初始为 null
  const inputRef = useRef(null)

  return <div>
    <div>
      <input ref={inputRef} onChange={() =>
      // current 为 input dom
      console.log(inputRef.current)} />
    </div>
  </div>
}
```


- useImperativeHandle
- useLayoutEffect
- useDebugValue

略