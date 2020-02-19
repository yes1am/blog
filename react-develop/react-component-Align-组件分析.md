[代码仓库](https://github.com/react-component/align)  

## 1. 正文

基于上次所分析的 [`dom-align` 源码](https://github.com/yes1am/blog/issues/19),现在看一下该代码如何被封装成`React`组件。 

### 1.1 调用方式

```
<Align
  ref={this.alignRef}
  target={this.getTarget}
  monitorWindowResize={this.state.monitor}
  align={this.state.align}
>
  <div
    style={{
      position: 'absolute',
      width: 50,
      height: 50,
      background: 'yellow',
    }}
  />
</Align>
```
其中`this.alignRef`是为了在当前父组件中调用`Align`组件的强制对齐方法,`this.$align.forceAlign();`。  

`target`即为`dom-align`中的`target`,可以是一个函数返回DOM节点。  

`monitorWindowResize`表示是否监听window的变化。  

### 1.2 实现

```
class Align extends Component {

  ...

  static defaultProps = {
    target: () => window,      // 默认target
    monitorBufferTime: 50,     // 防抖时间
    monitorWindowResize: false,
    disabled: false,        // 是否禁止对齐
  };

  componentDidMount() {
    const props = this.props;
    // if parent ref not attached .... use document.getElementById
    this.forceAlign();
    if (!props.disabled && props.monitorWindowResize) {
      this.startMonitorWindowResize();
    }
  }

  componentDidUpdate(prevProps) {
    let reAlign = false;
    const props = this.props;

    // 下面三种情况会发生重新对齐
    // 1.由disabled转为非disabled
    // 2. target改变
    // 3. source元素大小改变 
    if (!props.disabled) {
      const source = ReactDOM.findDOMNode(this);
      const sourceRect = source ? source.getBoundingClientRect() : null;

      if (prevProps.disabled) {
        // 之前是disabled
        reAlign = true;
      } else {
        const lastElement = getElement(prevProps.target);
        const currentElement = getElement(props.target);
        const lastPoint = getPoint(prevProps.target);
        const currentPoint = getPoint(props.target);

        if (isWindow(lastElement) && isWindow(currentElement)) {
          // Skip if is window
          reAlign = false;
        } else if (
          lastElement !== currentElement || // Element change
          (lastElement && !currentElement && currentPoint) || // Change from element to point
          (lastPoint && currentPoint && currentElement) || // Change from point to element
          (currentPoint && !isSamePoint(lastPoint, currentPoint))
        ) {
          reAlign = true;
        }

        // If source element size changed
        const preRect = this.sourceRect || {};
        if (
          !reAlign &&
          source &&
          (!isSimilarValue(preRect.width, sourceRect.width) || !isSimilarValue(preRect.height, sourceRect.height))
        ) {
          reAlign = true;
        }
      }

      this.sourceRect = sourceRect;
    }

    if (reAlign) {
      this.forceAlign();
    }

    if (props.monitorWindowResize && !props.disabled) {
      this.startMonitorWindowResize();
    } else {
      this.stopMonitorWindowResize();
    }
  }

  componentWillUnmount() {
    this.stopMonitorWindowResize();
  }

  startMonitorWindowResize() {
    if (!this.resizeHandler) {   // 防止重复添加监听
      this.bufferMonitor = buffer(this.forceAlign, this.props.monitorBufferTime);
      this.resizeHandler = addEventListener(window, 'resize', this.bufferMonitor);
    }
  }

  stopMonitorWindowResize() {
    if (this.resizeHandler) {
      this.bufferMonitor.clear();
      this.resizeHandler.remove();
      this.resizeHandler = null;
    }
  }

  forceAlign = () => {
    const { disabled, target, align, onAlign } = this.props;
    if (!disabled && target) {
      // 通过 ReactDOM.findDOMNode(this) 获取source
      // this其实最终获取的还是child component的DOM节点,因为render还是返回child
      const source = ReactDOM.findDOMNode(this);

      let result;
      const element = getElement(target);
      const point = getPoint(target);

      // IE lose focus after element realign
      // We should record activeElement and restore later
      const activeElement = document.activeElement;

      if (element) {
        result = alignElement(source, element, align);
      } else if (point) {
        result = alignPoint(source, point, align);
      }

      restoreFocus(activeElement, source);

      if (onAlign) {
        // 如果有onAlign方法，即在`对齐`之后触发
        onAlign(source, result);
      }
    }
  }

  render() {
    const { childrenProps, children } = this.props;
    const child = React.Children.only(children);
    if (childrenProps) {
      const newProps = {};
      const propList = Object.keys(childrenProps);
      propList.forEach((prop) => {
        newProps[prop] = this.props[childrenProps[prop]];
      });

      // 如果有childrenProps，需要通过cloneElement将childrenProps复制到该组件中
      return React.cloneElement(child, newProps);
    }
    return child;
  }
}

```
在初始化时会进行一次对齐，之后在 didUpdate 之后再根据情况对齐  

### 1.3 父组件中通过props.children渲染子组件时，如何获取子组件的ref

在刚刚的源码中，是通过`findDomNode`查找到的子元素DOM实例，然而官方文档又说`大多数情况下，你可以绑定一个 ref 到 DOM 节点上，可以完全避免使用 findDOMNode`. 那这种通过`this.props.children`的情况，能不能通过`ref`获取到实例呢？

https://stackoverflow.com/questions/52209204/is-it-possible-to-get-ref-of-props-children  

父组件：
```
class Parent extends Component {
  componentDidMount () {
    console.log(this.ref)   // 此时的 this.ref 与 findDomNode结果一致
  }
  render () {
    return React.cloneElement(
      this.props.children,
      // 此时的this为父组件，通过传递 ref 函数，使得子组件运行时会将ref绑定到父组件的this上
      { ref: el => { this.ref = el } }
    )
  }
}

调用
<Parent>
  <div>123</div>
</Parent>
```

### 1.4 buffer 与 addEventListener
刚刚源码中有这两个方法,`buffer`用于节流控制，即传统的多次触发则清除上一个定时器的做法，可以参考下:  
```js
export function buffer(fn, ms) {
  let timer;

  function clear() {
    if (timer) {
      clearTimeout(timer);
      timer = null;
    }
  }

  function bufferFn() {
    clear();  // 多次触发删除上次定时器
    timer = setTimeout(fn, ms);
  }

  bufferFn.clear = clear;    // 将clear挂载在返回的对象上
  // this.bufferMonitor = buffer(this.forceAlign, this.props.monitorBufferTime);
  //this.resizeHandler = addEventListener(window, 'resize', this.bufferMonitor);
  // 则可以通过 this.bufferMonitor.clear()清除定时器

  return bufferFn;
}
```

[addEventListener仓库地址](https://github.com/yiminghe/add-dom-event-listener)  

该方法屏蔽了`event`事件对象以及给DOM添加事件方法上，各浏览器实现不一致的问题:  

>在 IE6-8 中，事件模型与标准不同。使用非标准的 element.attachEvent() 方法绑定事件监听器。在该模型中，事件对象有一个 srcElement 属性，等价于target 属性。

即这个库即使在IE6-8中也可以使用`e.target`,其内部会处理成`srcElemnt`。  

```
function addEventListener (target, eventType, callback, option) {
  function wrapCallback (e) {
    const ne = new EventObject(e);
    callback.call(target, e)
  }

  if (target.addEventListener) {
    let useCapture = false
    if (typeof option === 'object') {
      useCapture = option.capture || false
    } else if (typeof option === 'boolean') {
      useCapture = option
    }

    target.addEventListener(eventType, wrapCallback, option || false)

    return {
      remove () {
        target.removeEventListener(eventType, wrapCallback, useCapture)
      }
    }
  } else if (target.attachEvent) {
    target.attachEvent(`on${eventType}`, wrapCallback)
    return {
      remove () {
        target.detachEvent(`on${eventType}`, wrapCallback)
      }
    }
  }
}
```

另外需要注意的是，该方法通过闭包的方式,返回一个`remove`方法，因此即使`callback`是匿名函数，也是能被移除的。