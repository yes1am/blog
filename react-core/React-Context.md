> Context 提供了一个**无需为每层组件手动添加 props, 就能在组件树间传递数据**的方法

## 1. 何时使用 Context

通常，在一颗组件树中，如果最上层的状态，要传递到子孙组件中，那么需要中间的父级节点都将 props 进行传递:  

```js
export default function App () {
  const [theme, setTheme] = useState('dark')
  const clickHandle = () => {
    if (theme === 'dark') {
      setTheme('light')
    } else {
      setTheme('dark')
    }
  }
  return <div>
    <div onClick={clickHandle}>change theme</div>
    <Parent theme={theme} />
  </div>
}

function Parent ({ theme }) {
  return <Child theme={theme} />
}

function Child ({ theme }) {
  return <div>{theme}</div>
}
```

上面这个例子，Parent 本身并不需要 `theme` 这个 props，只是为了传递给子组件，所以进行一个**透传**。  

而使用 context, 就能避免**透传**的问题，中间元素不需要传递 props 。  

```js
// 创建一个 context，默认值为 'light'
const ThemeContext = React.createContext('light')

export default function App () {
  const [theme, setTheme] = useState('dark')
  const clickHandle = () => {
    if (theme === 'dark') {
      setTheme('light')
    } else {
      setTheme('dark')
    }
  }
  // 使用一个 Provider  来将当前的 theme 传递给以下的组件树
  // 无论多深，任何组件都能读取这个值
  // 同时 theme 是当前组件的 state，值可以被改变
  return <div>
    <div onClick={clickHandle}>change theme</div>
    <ThemeContext.Provider value={theme}>
      <Parent />
    </ThemeContext.Provider>
  </div>
}

// 中间组件，不用再手动传递 theme 了
function Parent () {
  return <Child />
}

class Child extends Component {
  // 指定 contextType 
  // React 会往父组件往上找，找到最近的 ThemeContext.Provider 所提供的值
  // 如果没有找到 ThemeContext.Provider， 那么就会使用 React.createContext 的默认值
  
  static contextType = ThemeContext;
  render () {
    return <div>{this.context}</div>
  }
}
```

## 2. 注意事项

- 一个订阅了 Context 对象的组件(即 `static contextType 等于某个 Context 对象`)，那么这个组件就会 **从父级组件中，匹配离自己最近** 的 Context.Provider 提供的值  


- 当没有从父级组件中，匹配到 Context.Provider 时，将会使用 `React.createContext(默认值)` 的默认值，Context.Provider 提供 ` undefined` 时, `React.createContext(默认值)` 的默认值 **不会生效**。  


- Provider 接受 value 属性，传递给消费组件，**一个 Provider 可以有多个消费组件，多个 Provider 也可以嵌套使用，消费组件会使用最近的 Provider
 数据**  
 
 
- 当 Provider 的 value 发生变化时，**内部所有消费组件都会重新渲染**，重新渲染不受`shouldComponentUpdate` 影响  


-  检测 Provider 的 value 发生变化是根据 `Object.is` 函数

## 3. Context.Consumer

在刚刚的例子中，Child 组件获取读取 context 的值，是通过指定 `static contextType` 来实现的

```js
class Child extends Component {
  // 通过这种方式
  static contextType = ThemeContext;
  render () {
    return <div>{this.context}</div>
  }
}
```

而实际上，我们还可以通过 `Context.Consumer` 的方式，这是一种 `render props` 的实现:  

```js
class Child extends Component {
  render () {
    return <ThemeContext.Consumer>
      {value => <div>{value}</div>}
    </ThemeContext.Consumer>
  }
}
```

## 4. 示例

### 4.1 动态 Context

其实我们刚刚的例子都是 动态 Context 的例子:  

```js
export default function App () {
  const [theme, setTheme] = useState('dark')
  const clickHandle = () => {
    if (theme === 'dark') {
      setTheme('light')
    } else {
      setTheme('dark')
    }
  }
  return <div>
    <div onClick={clickHandle}>change theme</div>
    <ThemeContext.Provider value={theme}>
      <Parent />
    </ThemeContext.Provider>
  </div>
}

// 静态 Context
<ThemeContext.Provider value='dark'>
    <Parent />
</ThemeContext.Provider>
```

即 Provider 提供的值，是一个变量，它取决于 **父组件的state**, 而父组件的 state 是可以随时变的。  

与之对应的是 **静态 Context**

## 4.2 在嵌套组件中更新 Context

```js
// 创建一个 context 对象，该对象默认包含一个值，和一个函数
const ThemeContext = React.createContext({
  theme: 'light',
  toggleTheme: () => {}
})

export default function App () {
  const [theme, setTheme] = useState('dark')
  const clickHandle = () => {
    if (theme === 'dark') {
      setTheme('light')
    } else {
      setTheme('dark')
    }
  }
  return <div>
    <div>change theme</div>
    // 在 Provider 中，将值和改变该值的函数传入
    <ThemeContext.Provider value={{
      theme,
      toggleTheme: clickHandle
    }}>
      <Parent />
    </ThemeContext.Provider>
  </div>
}

function Parent () {
  return <Child />
}

function Child () {
  // 子组件中就能拿到值，和改变值的函数
  return <ThemeContext.Consumer>
    {({ theme, toggleTheme }) => <div onClick={() => toggleTheme()}>{theme}</div>}
  </ThemeContext.Consumer>
}
```

之前的例子，改变 `Provider.value` 的值 都是**发生在顶层组件的**，但是我们会有在嵌套组件中，改变 `Provider.value` 的需求，于是就有这个示例。  

## 4.3 嵌套的 Context

```js
const ThemeContext = React.createContext('light')
const SizeContext = React.createContext('normal')

export default function App () {
  return <ThemeContext.Provider value='dark'>
    <SizeContext.Provider value='small'>
      <Parent />
    </SizeContext.Provider>
  </ThemeContext.Provider>
}

function Parent () {
  return <Child />
}

function Child () {
  return <ThemeContext.Consumer>
    {theme => {
      return <SizeContext.Consumer>
        {size => <div>{size} {theme}</div>}
      </SizeContext.Consumer>
    }}
  </ThemeContext.Consumer>
}
```
刚刚提到多个 Provier 可以嵌套使用，以上就是嵌套使用的例子。  

## 5. 过时的 API

[官方文档](https://zh-hans.reactjs.org/docs/legacy-context.html)  

```js
import PropTypes from 'prop-types'        // 必须 (1)

export default class App extends Component {
  constructor (props) {
    super(props)
    this.state = {
      theme: 'dark'
    }
  }

  getChildContext () {                     // 必须 (2)
    return { theme: this.state.theme }     // 动态 context
    // return { theme: 'dark' }            // 静态 context
  }

  static childContextTypes = {             // 必须 (3)
    theme: PropTypes.string
  }

  handleClick = () => {
    this.setState({
      theme: this.state.theme === 'dark' ? 'light' : 'dark'
    })
  }

  render () {
    return <div>
      <div onClick={this.handleClick}>change theme</div>
      <Parent />
    </div>
  }
}

function Parent () {
  return <Child />
}

class Child extends Component {
  static contextTypes = {                 // 必须 (4)
    theme: PropTypes.string
  }

  render () {
    return <div>{this.context.theme}</div>
  }
}

//  如果 Child 是函数式组件，可以用以下的方式

const Child = (props, context) => <div>{context.theme}</div>

Child.contextTypes = { theme: PropTypes.string }
```

在 过时的 API 中，以上 **四个必须** 是必须要有的。  

## 参考文档

1. React 官方文档 [Context](https://zh-hans.reactjs.org/docs/context.html#when-to-use-context)