想象如下的组件结构，Trigger 是触发悬浮的按钮，Align 是**对齐与内容**组件，在 visible 为 true 的情况下，悬浮内容会显示出来，同时会对齐到 Trigger 。 

```js
<Trigger>
    点击我
    <Align>
        我是悬浮内容
    </Align>
</Trigger>
```

实际渲染的时候，Trigger 组件包裹着 "点击我"，而 Align 的内容通常是作为 Body 的直接子元素。  

在实现 Align 的时候，内部会有 **实现对齐的逻辑**，假设为 `makeAlign(sourceDom, targetDom)`，但前提是，我们能够拿到 "点击我" 最终渲染的 dom 结构以及 "我是悬浮内容" 的 dom。  

```js
this.triggerRef = React.createRef()

<Trigger ref={this.triggerRef}>
    点击我
    <Align triggerRef={this.triggerRef}>
        我是悬浮内容
    </Align>
</Trigger>


// Align 组件
componentDidMount() {
    // 在 didMount 的时候，makeAlign 内部将两个 dom 进行对齐
    const sourceDom = this.props.triggerRef.current;
    const targetDom = this.targetRef.current;
    makeAlign(sourceDom, targetDom)
}

render <div ref={this.targetRef}>
    // 将 children 渲染进来，这个例子的话，
    // children 是 "我是悬浮内容"
</div>
```

***问题一，current 可能为 null***  

```js
// Align 组件
componentDidMount() {
    // ========================================
    // 注意，this.props.triggerRef.current 可能为 null
    // 因为 triggerRef 属性是 父组件传下来的
    // 在 Align 这个子组件 didMount 的时候
    // 父组件中的 triggerRef.current 可能还是 null
    const sourceDom = this.props.triggerRef.current;
    const targetDom = this.targetRef.current;
    makeAlign(sourceDom, targetDom)
}
```

***问题一解决办法***  

1. 使用 ReactDOM.findDOMNode  或者 document.querySelectorXXX

```js
// 父组件
<Trigger ref={this.triggerRef}>
    点击我
    <Align triggerRef={this}>         // 即父组件往 align 传递自身 this
        我是悬浮内容
    </Align>
</Trigger>


// Align 组件
componentDidMount() {
    // ========================================
    // 通过 ReactDOM.findDomNode(this) 可以获得 dom 的引用
    // 因为 ReactDOM.findDomNode(this)， 最终结果等同于 "点击我" 这个 dom
    // 先前版本的 rc-align 也是这样实现的
    // 但是 React 官方不推荐使用 findDOMNode 这个方法
    
    const sourceDom = ReactDOM.findDomNode(this);
    const targetDom = this.targetRef.current;
    makeAlign(sourceDom, targetDom)
}


// 另外你可以使用 document.querySelectorXXX 这种 dom 选择的办法
// 也是可以获得值的
// 但是因为组件会被复用，因此唯一的 ID 或者 Class 需要维护一个池
// 确保 document.querySelectorXXX 获得对应的 dom 结构
```

2. 使用 setTimeout(0)

```js

<Trigger ref={this.triggerRef}>
    点击我
    <Align triggerRef={this.triggerRef}>
        我是悬浮内容
    </Align>
</Trigger>


// Align
componentDidMount() {
    setTimeout(() => {
        const sourceDom = this.props.triggerRef.current;
        const targetDom = this.targetRef.current;
        makeAlign(sourceDom, targetDom)
    }, 0)
}

// 通过 setTimeout(0)，确保里面的代码是在父组件 didMount 之后执行的 this.props.triggerRef.current 就是有值的
```

***问题二，应该传递 this.triggerRef 而不是 this.triggerRef.current***  

```js
<Trigger ref={this.triggerRef}>
    点击我
    <Align triggerRef={this.triggerRef}>
        我是悬浮内容
    </Align>
</Trigger>


// Align 组件
componentDidMount() {
    setTimeout(() => {
        const sourceDom = this.props.triggerRef.current;
        const targetDom = this.targetRef.current;
        makeAlign(sourceDom, targetDom)
    }, 0)
}


// 需要注意的是, 在这里我们传递是的 this.triggerRef 这是一个对象
// 而不是 this.triggerRef.current， 这样一个实际的 dom

<Align triggerRef={this.triggerRef}>
    我是悬浮内容
</Align>
```

经过测试发现，使用 ***this.triggerRef.current*** 还是会等于 `null`, 即使被 `setTimeout(0)` 包裹也是无效的。  