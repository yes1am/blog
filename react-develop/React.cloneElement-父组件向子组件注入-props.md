## 1. 分析

在实现 Dropdown 组件的时候有一个场景：  

```js
<Dropdown
  overlay={
    <DropdownMenu>
      <DropdownItem>DropdownItem1</DropdownItem>
      <DropdownItem>DropdownItem2</DropdownItem>
      <DropdownItem>DropdownItem3</DropdownItem>
    </DropdownMenu>
  }
>
  <Button>Dropdown</Button>
</Dropdown>
```

出现在页面中的是 `<Button>Dropdown</Button>` 这个组件，overlay 参数中的组件在点击 Button 时会显示。  

因此我们需要给 Button 添加点击事件，而这一切是在 `Dropdwn` 内部处理的。  

当然， Button 很好处理，比如我们可以这样:  

```js
// Dropdown 组件 render 函数返回

const triggerEleProps = {}

if (trigger === 'click') {
  triggerEleProps.onClick = this.toggle;
}
    
<div
  className="dropdown"
  {...triggerEleProps}
>
  {children}
</div>
}
```

即我们可以通过在 children (实际上就是 `<Button>Dropdown</Button>`) 外面包一层元素，通过在外层元素上添加事件即可。  

但是，如果我们要给 `overlay` 添加事件怎么处理:  

```js
const { overlay } = this.props;

<div
  className="dropdown"
  {...triggerEleProps}
>
  {children}
  {overlay}
</div>

这时候给 `overlay` 外层添加一层 dom 是不合理的
会新增 **多余且无用** 的 dom

<div
  className="dropdown"
  {...triggerEleProps}
>
  {children}
  <div>              // 为了添加事件而添加dom
    {overlay}
  </div>
</div>
```

那应该怎么办呢？  

**答案是 React.cloneElement()**  

```js
const newProps = { onMouseEnter: this.show }
const newOverlay = React.cloneElement(overlay, {...newProps});

<div
  className="dropdown"
  {...triggerEleProps}
>
  {children}
  {newOverlay}
</div>
```

通过 `cloneElement` 即可将为组件添加新的属性。  

此外，看以下的 demo:  

```js
// child 组件
function Child (props) {
  console.log(props)            // 最终会打印 {a: 1}
  return <div>child</div>
}

// parent 组件
function Parent (props) {
  const child = React.cloneElement(props.overlay, { a: 1 })
  return child
}

// App 组件
export default function App () {
  return (
    <Parent>
      <Child />
    </Parent>
  )
}
```

最终 Child 组件会打印出 `{ a: 1 }`, 也就是说 `cloneElement(childElement，newProps)` 可以在 `childElement` 的 `props` 中拿到 `newProps`. 

*一开始测试的时候，没有拿到 newProps，然后就很疑惑，要怎样把 props 注入 Child 呢?* 好像就只能用 render props 的方法，只有这样才能**将 Parent 组件内部的数据传递给 Child 组件**:  

```js
// Child 组件
function Child (props) {
  console.log(props)
  return <div>child</div>
}

// Parent 组件
function Parent (props) {
  return props.children({ a: 1 })
}

// App 组件
export default function App () {
  return (
    <Parent>
      {(props) => {
        return <Child {...props} />
      }}
    </Parent>
  )
}
```

但是这样不够优雅，***调用方能够感知到 Parent 和 Child 组件通信的过程，需要调用方显式的 解构 props， 传入到 Child 组件里***  

而通过 `React.cloneElement()` 的方式, 可以实现调用方**无感**的使用。***而内部的数据传递，应该被我们隐藏了***  

## 2. 应用场景

*以下都是个人理解*  

其实第一节说完，应用场景就很明显了，关键在于 **调用方无感知数据流**。  

注意到 antd 的 Table 组件有两种调用方式了:  

```js
// 普通方式
<Table
    columns={columns}
    dataSource={data}
/>

// JSX 方式
<Table dataSource={data}>
    <ColumnGroup title="Name">
      <Column title="First Name" dataIndex="firstName" key="firstName" />
      <Column title="Last Name" dataIndex="lastName" key="lastName" />
    </ColumnGroup>
    <Column title="Age" dataIndex="age" key="age" />
    <Column title="Address" dataIndex="address" key="address" />
</Table>
```  

对于普通方式，Table 组件将所有的逻辑，结构都隐藏了。对于用户来说，不是很直观。  

对于 JSX 方式而言，用户会更加清楚最终显示的结构会是怎么样的。换言之，**可阅读性** 较好。  

以上两种方式用户也都是**无感知数据流**，因为 Table 还是比较纯粹的显示型组件。而如果是存在交互的组件，如 `Table` 和 `Column` 存在相互的调用，普通方式 和 JSX 方式都能实现，JSX方式可能就很依赖于 `React.cloneElement()`。