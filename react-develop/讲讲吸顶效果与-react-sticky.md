## 前言

之前项目里的头部导航需要实现吸顶效果，一开始是自己实现，发现效果总是差那么一点，当时急着实现功能找来了[react-sticky](https://www.npmjs.com/package/react-sticky)这个库，现在有空便想着彻底琢磨透这个吸顶的问题。  

## 1. 粘性定位
吸顶效果自然会想到`position:sticky`, 这属性网上相关资料也很多，大家可以自行查阅。就提一点与我最初预想不一样的地方：  

*示例1. 符合我的预期,正常吸顶*
```
// html
<body>
    <div class="sticky">123</div>
</body>

// css  
body {
    height: 2000px;
}
div.sticky {
  position: sticky;
  top:0px;
}
```

*示例2. 不符合我的预期 不能吸顶*
```
// html
<body>
  <div class='sticky-container'>
      <div class="sticky">123</div>
  </div>
</body>

// css  
body {
    height: 2000px;
}
div.sticky-contaienr {
    height: 1000px;  // 除非加上这段代码才会有一定的吸顶效果
}
div.sticky {
  position: sticky;
  top:0px;
}
```
我以为只要加上了 `position:sticky`,设置了 `top` 的值就能吸顶，不管其他的元素如何，刚好也是我需要的效果，如示例1一样。  

但是其实对于 `position:sticky` 而言，它的活动范围只能在父元素内，滚动超过父元素的话，它一样不能吸顶。示例2中，`.sticky-container`的高度和 `.sticky` 的高度一致，滚动就没有吸顶效果。 给 `.sticky-container` 设置个 `1000px` 的高度，那 `.sticky` 就能在那 `1000px` 的滚动中吸顶。  

当然 `sticky` 这样设计是为了实现更为复杂的效果。  

附上一份参考资料 [CSS Position Sticky - How It Really Works!](https://medium.com/@elad/css-position-sticky-how-it-really-works-54cd01dc2d46)  

## 2. react-sticky  

### 2.1 使用
```
// React使用
<StickyContainer style={{height: 2000}}>
    <Sticky>
    {({style}) => {
        return <div style={style}>123 </div>         // 需要吸顶的元素
    }}
  </Sticky>
  其它内容
</StickyContainer>


// 对应生成的Dom
<div style='height: 2000px;'>                        // sticky-container
    <div>                                            //  parent
        <div style='padding-bottom: 0px;'></div>     //  placeholder
        <div>123 </div>                              // 吸顶元素
    </div>
   其它内容
</div>
```

### 2.2 疑惑
看上面的React代码及对应生成的dom结构，发现`Sticky`生成了一个嵌套div结构，把我们真正需要吸顶的元素给包裹了一层:  
```
<div>                                            //  parent
    <div style='padding-bottom: 0px;'></div>     //  placeholder
    <div>123 </div>                              // 吸顶元素
</div>
```  

一开始我是有些疑惑的，这个库为什么要这样实现，不能生成下面的结构嘛？减去`div1`,`div2`？  

```
<div style='height: 2000px;'>
    <div>123 </div>
   其它内容
</div>
```  

于是我先不管别人的代码，本地写demo，思考着如何实现吸顶效果，才慢慢理解到`react-sticky`的设计。  

### 2.3 解疑  
**吸顶**，即当 `页面滚动的距离` 超过 `吸顶元素距离文档(而非浏览器窗口)顶部的高度`时，则吸顶元素进行吸顶，否则吸顶元素变为正常的文档流定位。  

因此当然可以在`第一次`滚动前，通过吸顶元素(之后会用sticky代替)`sticky.getBoundingClientRect().top`获取元素距离html文档顶部的距离假设为`htmlTop`，之所以强调在第一次滚动前是因为，只有第一次滚动前代表的是距离html文档顶部的距离，之后有滚动了就只能代表距离浏览器窗口顶部的距离。

通过`document.documentElement.scrollTop`获取页面滚动距离假设为`scrollTop`，每次滚动事件触发时计算`scrollTop - htmlTop`,大于0则将`sticky元素的position`设为 `fixed`，否则恢复为原来的定位方式。  

这样是能正常吸顶的，但是会有个问题，由于`sticky`变为`fixed`脱离文档流，导致文档内容缺少一块。想象下：  
```
div1
div2
div3

1，2，3三个div，假如突然2变为fixed了，那么会变成：  

div1
div3 div2   
```  
即吸顶的之后，div3的内容会被div2遮挡住。

所以查看刚刚`react-sticky`生成的dom中，吸顶元素会有个兄弟元素`placeholder`。有了placeholder之后即使吸顶元素`fixed`了脱离文档流,也有placeholder占据它的位置：  

```
div1
placeholder div2
div3
```  

同时由于给吸顶元素添加了兄弟元素，那么最好的处理方式是再加个`parent
`把两个元素包裹起来，这样不容易被别的元素影响也不容易影响别的元素(我猜的)。  

### 2.4 源码

它的实现也很简单，就`Sticky.js`和`Container.js`两个文件，稍微讲下。代码不粘贴了点开这里看 [Container.js](https://github.com/captivationsoftware/react-sticky/blob/master/src/Container.js), [Sticky.js](https://github.com/captivationsoftware/react-sticky/blob/master/src/Sticky.js)。  

* 首先绑定一批事件：`resize`,`scroll`,`touchstart`,`touchmove`,`touchend` ,`pageshow`,`load`。  
* 通过观察者模式，当以上这些事件触发时，将`Container`的位置信息传递到`Sticky`组件上。    
* `Sticky`组件再通过计算位置信息判断是否需要`fixed`定位。  

其实也就是这样,当然它还支持了`relative`,`stacked`两种模式,因此代码更复杂些。看看从中我们能学到什么：  

* 用到了`raf`库控制动画，它是对`requestAnimationFrame`做了兼容性处理
* 使用到 `Context`，以及 观察者模式
* 居然需要监听那么多种事件(反正是我的话就只会加个scroll)
* 用`React.cloneElement`来创建元素，以致于最终使用`Sticky`组件用起来感觉有些不寻常。
* `disableHardwareAcceleration`属性用于关闭动画的硬件加速,实质上是决定是否设置`transform:"translateZ(0)";`  

对最后一个知识点感兴趣，老是说用`transform`能启动硬件加速，动画更流畅，真的假的？于是又去找了资料，[An Introduction to Hardware Acceleration with CSS Animations](https://www.sitepoint.com/introduction-to-hardware-acceleration-css-animations/)。本地测试chrome的`performance`发现用`left，top`，`fps,gpu,frames`等一片绿色柱状线条，用 `transform` 则只有零星几条绿色柱状线条。感觉有道理。  

## 3. 来个简易版  
理解完源码自己写(抄)一个,只实现最简单的吸顶功能:  
```
import React, { Component } from 'react'

const events = [
  'resize',
  'scroll',
  'touchstart',
  'touchmove',
  'touchend',
  'pageshow',
  'load'
]

const hardwareAcceleration = {transform: 'translateZ(0)'}

class EasyReactSticky extends Component {
  constructor (props) {
    super(props)
    this.placeholder = React.createRef()
    this.container = React.createRef()
    this.state = {
      style: {},
      placeholderHeight: 0
    }
    this.rafHandle = null
    this.handleEvent = this.handleEvent.bind(this)
  }

  componentDidMount () {
    events.forEach(event =>
      window.addEventListener(event, this.handleEvent)
    )
  }

  componentWillUnmount () {
    if (this.rafHandle) {
      raf.cancel(this.rafHandle)
      this.rafHandle = null
    }
    events.forEach(event =>
      window.removeEventListener(event, this.handleEvent)
    )
  }

  handleEvent () {
    this.rafHandle = raf(() => {
      const {top, height} = this.container.current.getBoundingClientRect()
      // 由于container只包裹着placeholder和吸顶元素，且container的定位属性不会改变
      // 因此container.getBoundingClientRect().top大于0则吸顶元素处于正常文档流
      // 小于0则吸顶元素进行fixed定位，同时placeholder撑开吸顶元素原有的空间
      const {width} = this.placeholder.current.getBoundingClientRect()
      if (top > 0) {
        this.setState({
          style: {
            ...hardwareAcceleration
          },
          placeholderHeight: 0
        })
      } else {
        this.setState({
          style: {
            position: 'fixed',
            top: '0',
            width,
            ...hardwareAcceleration
          },
          placeholderHeight: height
        })
      }
    })
  }

  render () {
    const {style, placeholderHeight} = this.state
    return (
      <div ref={this.container}>
        <div style={{height: placeholderHeight}} ref={this.placeholder} />
        {this.props.content(style)}
      </div>
    )
  }
}

//使用
<EasyReactSticky content={style => {
    return <div style={style}>this is EasyReactSticky</div>
}} />
```

显然，大部分代码借鉴 `react-sticky` ，减少了参数配置代码和对两种模式`stacked`和`relative`的支持。着实简易，同时改变了组件调用形式，采用了`render-props`。  

## 4. 总结  
本文源于之前工作急于完成任务而留下的一个小坑,所幸现在填上了。`react-sticky`在github上`1926`个star,本身却并不复杂,通过阅读这样一个经受住开源考验的小库也能学到不少东西。  

## 5. 讨论
* 如果给一个用`top,left`改变来做动画的元素，例如[An Introduction to Hardware Acceleration with CSS Animations](https://www.sitepoint.com/introduction-to-hardware-acceleration-css-animations/)中的第一个例子，添加`transform:translateZ(0)`，那样会有硬件加速嘛？(我测试的结果像是没有，依旧很多painting)

欢迎讨论~  

## 参考资料
[react-sticky](https://www.npmjs.com/package/react-sticky)  
[CSS Position Sticky - How It Really Works!](https://medium.com/@elad/css-position-sticky-how-it-really-works-54cd01dc2d46)  
[An Introduction to Hardware Acceleration with CSS Animations](https://www.sitepoint.com/introduction-to-hardware-acceleration-css-animations/)  
[render-props](https://reactjs.org/docs/render-props.html)  