## 前言

所在的前端小组要求组内成员每周轮流分享,眼看这周就轮到我了,便思考如何能顺利得"混过"这次分享。  

得和工作搭点边，一点不搭"ga"的话意义不大。得通俗易懂，源码啥的太难了担心组员们听不懂(其实是自己看不懂源码)。希望以后...  

面试官：“你简历上说你看过React源码?”  
我：“没错随便你提问哪一行,正数倒数从中间往两边数都行”   

"咳咳，停止YY"  

想起了之前遇到过的,有些疑惑的 `React的事件`问题,当时在一篇博客的指导下顺利解决了,这次有机会不如再整理一下，也好将知识"据为己有"。  

## 1. React事件绑定
> React事件绑定的本质是将事件代理到 `document` 上。  

我们都知道可以通过 `控制台 > Elements > Event Listeners`可以查看当前页面所绑定的事件。  
在`react + react-router-dom` 的环境下,以下代码:  
```
import React from 'react'
import ReactDOM from 'react-dom'
import { BrowserRouter as Router, Route, Link } from 'react-router-dom'

ReactDOM.render(
  <Router>
    <div>
      <Link to='/Event'>Event</Link>
      <Route path='/Event' component={null} />
    </div>
  </Router>,
  document.getElementById('root')
)
```
`Event Listener` 显示:  

![event listener](https://user-images.githubusercontent.com/25051945/56863659-d8615800-69eb-11e9-9037-a4cb7922933d.png)

即此时最简单的代码已经分别在`document`和`window`上绑定了`click`和`popstate`有事件了,同时后面的行号表示触发这次事件绑定的源码所在位置(调试的话就点进去打断点)。这是什么情况，`react`自带事件监听器？于是我:  
```
...
ReactDOM.render(
  null,
  document.getElementById('root')
)
```
再查看发现已经没有事件监听器了。顿时就好猜了，`react-router-dom`中，`Link`标签能响应点击,那么 `click` 事件便有可能是它绑定的，同时路由切换对应'history'的事件，所以两个事件都是`react-router-dom` 绑定的。  

于是我:
```
ReactDOM.render(
  <Router>
    <div>
      <Route path='/Event' component={null} />
    </div>
  </Router>,
  document.getElementById('root')
)
```
然后 `click`监听器没了，还剩下`popstate`监听器，有兴趣也可试试删除 `Link` 组件 `node_modules/react-router-dom/es/Link.js` render 方法中的 `onClick: this.handleClick` 看看事件监听器的变化。  
至于`popstate`监听器可查看 `Router` 组件 `node_modules/react-router/es/Router.js`的 `componentWillMount` 中的`history.listen`  

还有个测试结果: 每一种类型(click，onmouseover等)的事件，由于代理到 `document` 的原因,只会在`Event Listener`中出现一遍。  

## 2. React 事件池  
其实官方文档已经说的很清楚了:  
> 这是出于性能因素考虑，合成事件(SyntheticEvent)是被池化的。这意味着SyntheticEvent对象将会被重用，在调用事件回调之后所有属性将会被废弃。 因此，你不能以异步的方式访问SyntheticEvent对象。  

```
function handleClick(event) {
  console.log(event.type) // => "click", 同步是能访问到值的
  setTimeout(function() {
    console.log(event.type); // => null  异步的方式读取,由于event对象的属性都被废弃了(便于循环利用event对象),所以访问不到值
  }, 0);
}
```

关于池化,可以这么理解,你用浏览器打开了两个标签页，一个看`juejin`一个查`google`。此时`juejin`上的一篇文章的某个概念又不懂,要查`google`,你是再开一个标签页进`google`查呢？还是利用已经存在的`google`标签页查？  
再开一个的好处是原来`google`标签页的内容还保留着，你还可以切换查看，但是新开标签页是要消耗更多资源的。  
利用原来的`google`意味着省去新开标签页的带来的资源和时间成本，坏处是你回头想看原来`google`的内容就不是简单切换标签页了(得回退，找历史记录)。  

`react`提供了`"新开标签页，保留原标签的方式"`:  
```
function handleClick(event) {
  console.log(event.type) // => "click", 同步是能访问到值的
  event.persist();        // "允许代码保留对事件的引用(新开标签页)"
  setTimeout(function() {
    console.log(event.type); // => "click"  该对象不会被回收重用
  }, 0);
}
```  
## 3. React事件对象 e 与原生事件对象 e  
React事件对象e，自身有`e.nativeEvent` 可以访问`原生事件对象e`,沿着原型链找到 `SyntheticEvent`原型上有`stopPropagation` 和 `preventDefault`方法。  

原生事件对象e,沿着原型链找到 `Event`，在它原型上除了以上两个方法之外，还有`stopImmediatePropagation`方法,关于stopImmediatePropagation查看 [MDN文档](https://developer.mozilla.org/zh-CN/docs/Web/API/Event/stopImmediatePropagation#Notes)。  

```
domA.addEventListener("click", (event) => {
    console.log('doma click 1');
    event.stopImmediatePropagation();
});

domA.addEventListener("click", (event) => {
    console.log('doma click 2')
});
```  
即`stopImmediatePropagation`除了能做到像`stopPropagation`一样阻止事件向父级冒泡之外，也能阻止当前元素**剩余的,同类型事件的**执行(第一个 `click` 触发时，调用 `e.stopImmediatePropagtion` 阻止当前元素第二个 `click` 事件的触发)。  

## 4. 详解
*index.js*
```
import React from 'react'
import ReactDOM from 'react-dom'
import { BrowserRouter as Router, Route, Link } from 'react-router-dom'
import Event from './event.js'

ReactDOM.render(
  <Router>
    <div>
      <Link to='/Event'>Event</Link>
      <Route path='/Event' component={Event} />
    </div>
  </Router>,
  document.getElementById('root')
)
```


*以下代码为event.js代码,且均省去无关代码*  
#### 4.1 示例:  
```
// componentDidMount
document.addEventListener('click', (e) => {
    console.log('document click')
})

// 方法
handleClick (e) {
    console.log('div click')
    e.stopPropagation()  // e为React事件对象
}

// render
<div className='div-1' onClick={this.handleClick}>
    div-1
</div>
```  
当点击`div-1`时,打印`div click`,`document click`,即`e.stopPropagtion`只能阻止同为`React事件类型`的冒泡。结合`React事件本质上是绑定到document上`,于是以上代码相当于在`componentDidMount中添加两个事件监听`:  
```
// componentDidMount
document.addEventListener('click', (e) => {        // 事件监听 1
    console.log('div click')
    e.stopPropagation()
})

document.addEventListener('click', (e) => {       // 事件监听 2
    console.log('document click')
})

// 为什么是事件监听1在前，即为什么handleClick转化之后的事件绑定先于componentDidMount中的事件绑定。待会会说到事件绑定顺序的问题。  
```  
也就是在document对象上绑定两次事件,e.stopPropagation阻止不了当前元素**剩余的,同类型事件的**执行,刚刚说到`e.stopImmediatePropagation`可以，就是说下面的代码能阻止`document click2`的打印:  
```
document.addEventListener('click', (e) => {
    console.log('document click1')
    e.stopImmediatePropagation()
})

document.addEventListener('click', (e) => {
    console.log('document click2')  // e.stopImmediatePropagation阻止事件冒泡到这
})
```  
于是可以这样改动：  
```
handleClick (e) {
    console.log('div click')
    // e.stopImmediatePropagation() // 错误, 此时e为React事件对象，需要e.nativeEvent访问原生事件对象
    e.nativeEvent.stopImmediatePropagation()
}
```  

于是就顺利阻止了事件冒泡到 `document` 上。  

#### 4.2 事件绑定顺序  
刚刚说的绑定顺序，为什么知道是:
```
document.addEventListener('click', (e) => {
    console.log('div click')
})

document.addEventListener('click', (e) => {
    console.log('document click')
})
```  
而不是:  
```
document.addEventListener('click', (e) => {
    console.log('document click')
})

document.addEventListener('click', (e) => {
    console.log('div click')
})
```  
其实很简单，因为`render方法先于componentDidMount`执行,所以`handleClick`转化之后的事件绑定先于`原本componentDidMount`中的事件绑定。打断点也很容易得出结论(会定位到这个位置`/node_modules/react-dom/cjs/react-dom.development.js的trapBubbledEvent方法`)。那假如我这样做，即在`render`时给 `document` 添加事件监听:  

```
// 方法
handleClick (e) {
    console.log('div click')
}

// render
document.addEventListener('click', (e) => {
    console.log('document click')
})
<div className='div-1' onClick={this.handleClick}>
    div-1
</div>
```  
那 `document click` 先于 `div click` 绑定，所以先打印 `document click`再打印`div click`.  错！ 

我们忽略了一个问题，在`event.js`中是`document click`先于React组件的事件绑定，但是在`index.js`中,React的`click`事件最早在`react-router-dom`中的`Link`组件中绑定过。即,还是`React事件绑定`先于`document事件绑定`。所以打印结果依然是`div click, document click`。为此我们可以做个测试验证下,改动index.js:  

*index.js*  
```
class Test extends Component {
  componentDidMount () {
    // 位置2
    document.addEventListener('click', (e) => {
        console.log('document clicked')
    })
  }
  handleClick() {
      console.log('div clicked')
  }
  render () {
    // 位置1
    document.addEventListener('click', (e) => {
        console.log('document clicked')
    })
    return <div onClick={this.handleClick}>123</div>
  }
}

ReactDOM.render(
  <Test />,
  document.getElementById('root')
)
```  

在位置1添加事件时，则先打印`document clicked`，后打印`div clicked`  
在位置2添加事件时，则先打印`div clicked`，后打印`document clicked`  
符合我们的预期。  

#### 4.3 执行顺序与阻止冒泡总结
```
子组件dom原生事件           --A 执行顺序与添加该事件的位置无关 
父组件dom原生事件           --B 执行顺序与添加该事件的位置无关
document原生事件            --C 执行顺序与添加该事件位置有关!! 位置 1
document代理事件            --D react内部对所有子组件事件进行代理
    子组件React事件         --D1 合成事件，代理在document上
    父组件React事件         --D2 合成事件，代理在document上
document原生事件            --C  位置2
```  
如上,页面有子组件和父组件，同时给它们绑定React事件和dom原生事件。document对象也可在`位置1`或者`位置2`,即代理事件进行事件绑定(即 `react` 第一次 `render` 带有事件属性的组件，比如 `index.js` 中的 `render` `Link组件` )之前或者之后(如`componentDidMount`)中绑定事件(事件绑定顺序会影响最终执行顺序)。  

**在没有任何阻止冒泡情况下，点击子组件:**  
原生事件从子组件开始冒泡，执行事件A  
事件冒泡到父组件，执行事件B  
此时冒泡到document上，执行C还是执行D取决于 `document` 原生事件是在位置1还是位置2. 在位置1则执行C。否则去执行D  
React内部分发事件，执行子组件React事件D1  
React内部冒泡，执行父组件React事件D2  
如果`document`原生事件在位置2绑定，则在此执行事件C  

**在事件A或者事件B阶段:**  
可以通过 `e.stopPropagation`或者(`e.stopImmediatePropagation，因为e.stopImmediatePropagation包含e.stopPropagation`) 阻止冒泡，此时document原生事件`C`及所有React事件(D1,D2)均不会执行。  

**在事件C阶段:**  
如果该事件在位置1,可以通过`e.stopImmediatePropagation`阻止React事件(D1,D2)的执行。但是用 `e.stopPropagation` 的话则不能阻止React事件执行。  
如果事件在位置2，则无论如何也**不能**阻止React事件执行。  

**在合成事件D1阶段:**  
可通过 `e.stopPropagation` 来阻止事件 `D2` 的执行(`e.nativeEvent.stopImmediatePropagation`不行，因为不管是 D1 还是 D2，其实真正的 document 事件监听是同一个，只是监听函数内部区分了 D1 还是 D2 ),此时`e` 为React事件对象。  

如果document原生事件C在位置2，无论在D1还是D2阶段，则都能通过`e.nativeEvent.stopImmediatePropagation`阻止原生事件C的执行。  

## 5. 总结  
本文核心知识来源于参考资料`React 事件代理...`,感谢作者`youngwind`。几乎不涉及源码(目前看源码还是比较吃力)，结合自己的理解加测试连蒙带猜凑合而成,希望对读者有所帮助。  

## 参考资料
[React 事件代理与 stopImmediatePropagation](https://github.com/youngwind/blog/issues/107)  
[React源码解读系列 – 事件机制](http://zhenhua-lee.github.io/react/react-event.html)