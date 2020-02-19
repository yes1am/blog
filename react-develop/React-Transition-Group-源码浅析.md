## 1. 前言

之前做日历组件的时候,移动端需要实现一个从底部弹出的一个弹层组件，涉及到React动画的相关知识。

当时了解到 `react-transition-group`的相关用法，但是不确定在实现`2C`组件的时候，引入外部组件是否恰当，毕竟原则上来说，最希望的是`2C`组件能足够轻量级，无其它无效代码。

在时间紧急情况下，当时使用回调的形式实现: 即当 componentDidMount时，给对应的DOM添加向上弹出动画class。在 visible 属性为false时，添加向下收缩动画的class, 等动画执行完通过回调告诉别的组件做别的操作。

也能实现动画的功能，但是总觉得这样的组件不够通用，于是在空闲之余查看了 `React-Transition-Group`组件的`Transition`部分源码, 特有此文，以做记录。

## 2. 流程图

[画流程图工具](https://marketplace.visualstudio.com/items?itemName=joaompinto.vscode-graphviz): Graphviz (dot) language support for Visual Studio Code


![react-transition-group](https://user-images.githubusercontent.com/25051945/64064373-0ae9be80-cc33-11e9-992a-3a71859162e1.png)

```
// 此时再次点击，使得in变为false，步骤简化版 
用户点击 in = false 开始 
=> getDerivedStateFromProps(不做处理)
=> render: 依旧渲染 entered 样式
=> didUpdate(), prevProps(in为true)不等于this.props(in为false)
=> status 等于 entered, 故 nextStatus 为 exiting,执行 this.updateStatus(false,exiting)
=> 执行 this.performExit
=> this.props.onExit
=> this.safeSetState({status: exiting})
=> getDerivedStateFromProps: nextIn为false, 不执行任何操作
=> render: 渲染 exiting 的样式
=> didUpdate: prevProps 等于 this.props
=> this.updateStatus(false, null)
=> 执行this.safeSetState({status: exiting})回调: this.props.onExiting
=> this.onTransitionEnd(node, timeouts.exit, callback): 在 timeout时间之后执行callback回调,即执行 this.safeSetState({status: exited})
=> this.safeSetState({ status: exited }): 触发 getDerivedStateFromProps, nextIn为false, 不执行任何操作
=> render: 渲染 exited 样式
=> didUpdate: prevProps 等于 this.props
=> this.updateStatus(false, null)
=> 执行this.safeSetState({ status: exited })回调: this.props.onExited(node)
```
## 3. 总结

经过阅读，即该组件首先根据参数，确定初始的status，比如如果`props.unmountOnExit`为`true`,那么初始`status`为`UNMOUNTED`,则初始的时候不进行渲染。  

之后，当接受到的`in` 属性发生变化的时候，如果属性值为true，则会执行`performEnter`,在 `performEnter` 中, 先`setState({status:'entering')`,之后过了`timeout`时间，执行`setState({status:'entered')`. 此外还会在适当的时机执行`this.props.onEnter`及`this.props.onEntering`等回调函数。

也就是, 如果接受到对应的属性，那么在不会一次性的`setState`到`entered`或者`exited`。而是在这中间穿插`entering`或者`exiting`的状态，将切换的状态传递给子组件去做动画。