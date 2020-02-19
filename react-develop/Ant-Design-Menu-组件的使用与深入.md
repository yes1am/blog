## 1. 需求

最近项目中要修改原有的菜单,项目UI为antd,antd的导航菜单长这样:  

![menu](https://user-images.githubusercontent.com/25051945/55289795-c8ab1f80-53fd-11e9-8786-8b14b7cd4907.png)  

看着挺好的,完美对齐,但是当把我的菜单文案填入之后发现:  

![my-menu](https://user-images.githubusercontent.com/25051945/55289798-ce086a00-53fd-11e9-8c43-d46cc6c9b227.png)  

左右不对齐,这也太丑了吧,这要是放任不管要被怼的。  
## 2. 排查问题
开始审查元素,先查看官方demo正常能对齐的样式:  

![submenu](https://user-images.githubusercontent.com/25051945/55289802-d95b9580-53fd-11e9-8e92-d1e0d6fa8752.png)    

再看下我的demo不能对齐的样式:  

![my-submenu](https://user-images.githubusercontent.com/25051945/55289808-de204980-53fd-11e9-9745-d2107a7ecf3d.png)    

发现我的菜单里少了一个 `min-width` ,也就是说antd在某一步给官方的demo添加了style属性,而没有给我的菜单添加。  

为什么不给我的加！！！?  

直接来吧,先来一个[MutationObserver](https://developer.mozilla.org/zh-CN/docs/Web/API/MutationObserver),详情看MDN文档。  

你是不是想在 `antd` 给那个 `ul` 标签添加 `style="min-width"` 的时候告诉你一下？甚至能打个断点来调试下,但不知道怎么操作？直接上代码:  
```
var ele = document.getElementById('item_2$Menu')   // 先找出该元素
var config = {attributes: true, attributeFilter: ['style']}
var callback = function (mutationsList) {
  console.log(mutationsList)
}
var observer = new window.MutationObserver(callback)
observer.observe(ele, config)
```  
上面这段代码就是说,在 `item_2$Menu` 的 `style` 属性发生变化的时候,打印下 `mutationsList` 。于是在我将鼠标移入菜单的时候,打印了以下内容:  

![mutation](https://user-images.githubusercontent.com/25051945/55289815-e7a9b180-53fd-11e9-81e0-608a5853893d.png)    

这有什么用呢？别急,说明鼠标移入,会执行到这里的代码,那么,不如打个断点？  

![chrome-breakpoint](https://user-images.githubusercontent.com/25051945/55289817-ed06fc00-53fd-11e9-96c2-a863716dd09a.png)    

鼠标移入时,浏览器停在了断点上,右边的 `call stack` 调用栈显示正在执行 `callback` 函数,看它的下面 `adjustWith` ,也就是说代码先执行 `adjustWith` ,然后触发了我们的断点。我们接着点开 `adjustWith` ,发现以下代码:  

![adjust-width](https://user-images.githubusercontent.com/25051945/55289819-f5f7cd80-53fd-11e9-8367-594843355f27.png)    

看来就是这段代码导致的。在浏览器中查看不方便,都是编译之后的代码,转战 `Vscode` ,查看我们的node_modules目录,先找到这个文件 `node-modules/rc-menu/es/SubMenu.js` 的 `adjustWidth` 方法:  
```
this.adjustWidth = function () {
  /* istanbul ignore if */
  if (!_this3.subMenuTitle || !_this3.menuInstance) {
    return;
  }
  var popupMenu = ReactDOM.findDOMNode(_this3.menuInstance);
  if (popupMenu.offsetWidth >= _this3.subMenuTitle.offsetWidth) {
    return;
  }

  /* istanbul ignore next */
  popupMenu.style.minWidth = _this3.subMenuTitle.offsetWidth + 'px';
};
```  
原来它会先判断宽度,如果 `popupMenu` 的宽度大于父级的 `Title` 宽度,就会直接返回,小于的时候才会加上 `min-width` 属性。  

这么费劲总算找到了！  

但是,找到了然后呢？问题是我咋去对齐？既然文案长度不一致,那么居中对齐好了。  

## 3. 解决

继续查看 `antd` 的文档,看看有没有什么参数方法遗漏了,发现个Menu文档小角落有个 `More options in rc-menu` ,[点进去](https://github.com/react-component/menu#api)之后发现了一片更广阔的世界 ( 其实我之前就问过同事知道antd还依赖于 `react-component` ,这个库才是`antd`组件具体的实现 ) 。  

经过一番查找,找到一个props叫做 `builtinPlacements` 很可疑,描述是 `Describes how the popup menus should be positioned(描述popup的菜单如何被定位)` ,参数为 [dom-align](https://github.com/yiminghe/dom-align) 的配置对象,继续查看 `dom-align` 的 [介绍](https://github.com/yiminghe/dom-align) 发现这就是一个处理定位的小库,处理 domA(sourceNode) 和 domB(targetNode) 的位置关系:  
```
const alignConfig = {
  points: ['tl', 'tr'],        // align top left point of sourceNode with top right point of targetNode
  offset: [10, 20],            // the offset sourceNode by 10px in x and 20px in y,
  targetOffset: ['30%','40%'], // the offset targetNode by 30% of targetNode width in x and 40% of targetNode height in y,
  overflow: { adjustX: true, adjustY: true }, // auto adjust position when sourceNode is overflowed
};

domAlign(domA, domB, alignConfig);
```
这样,就能让domA的 `左上角(tl)` 和domB的 `右上角(tr)` 对齐。直觉告诉我 `builtinPlacements` 属性能解决我的对齐问题。  


接下来继续调试,先介绍个调试工具 (同事告诉我的) 。想想,之前我要调试前端代码都是一堆的 `console.log` ,要么在浏览器 `source` 中打断点调试,也可以调试 `node_modules` 里面的代码(也是从同事那里学来的),但是缺点是代码都是被编译打包过的,可读性不好。于是有 `Vscode` 的插件 [Debugger for Chrome](https://github.com/Microsoft/vscode-chrome-debug),有了这个插件之后,可以直接在 `Vscode` 里面给前端js代码**打断点**！。  

### 接下来可能比较跳跃:  

直接找到node_modules下的 `rc-menu/es/submenu` 目录,就是react-component下的menu组件。先搜索文件夹内搜索 `builtinPlacements` 这个词,看下哪几个地方用到了。发现如下:  

1. *rc-menu/es/submenu.js*
```
...
var builtinPlacements = props.builtinPlacements;
...
React.createElement(
    Trigger,
    {
        ...
        builtinPlacements: _extends({}, placements, builtinPlacements),
        ...
    }
```  
原来Submenu拿到传入的 `builtinPlacements` 用来创建 `Trigger` 了,继续找 `Trigger` 发现:  

2. *rc-trigger/es/index.js*
```
Trigger.prototype.getPopupAlign = function getPopupAlign() {
    ...
    var builtinPlacements = props.builtinPlacements;
    ...
    return getAlignFromPlacement(builtinPlacements, popupPlacement, popupAlign)
};

...
var align = _this5.getPopupAlign();
...
return React.createElement(
      Popup,
      _extends({
        align: align,
    })
)
```
得,又拿来创建 `Popup` 了,不过先记住这个函数 
`getAlignFromPlacement(builtinPlacements, prefixCls, align, alignPoint)`:  

3. *rc-trigger/es/Popup.js*
```
import Align from 'rc-align';
...
React.createElement(
  Align,
  {
    ...
    align: align
    ...
  },
}
```

拿去创建 `Align` 了:  

4. *rc-align/es/Align.js*
```
import { alignElement, alignPoint } from 'dom-align';  // 你总算出来了
...

var align = _this$props.align
...
if (element) {
  result = alignElement(source, element, align);
} else if (point) {
  result = alignPoint(source, point, align);
}
...
```

所以大概关系是 `Antd Menu` => `rc-menu` => `rc-trigger` => `rc-align` => `dom-align`  ...  

其中你在 `Menu` 传入的 `builtinPlacements` 参数,会在rc-trigger中被当做参数传入 `getAlignPopupClassName(builtinPlacements, prefixCls, align, alignPoint)` ,得到的结果最终会被传入到 `dom-align` 的 `alignElement` 或者 `alignPoint` 中。  

但是 `builtinPlacements` 这个参数怎么传值呢？  

5. *function getAlignFromPlacement()*  

![](static/align-placement.png)  

也就是说我们需要传入类似这样的对象:  
```
builtinPlacements: {
    bottomLeft: {
        // alignConfig对象
        points: ['tl', 'tr'],
        offset: [10, 20],
        ...
    },
    leftTop: {
        ...
    }
}
```
并且可知: getAlignFromPlacement(builtinPlacements, placementStr, align) 中的 `placementStr` 此时为 `bottomLeft` ,所以我们的Menu变成了:  
```
<Menu builtinPlacements={
    {
        bottomLeft: 
        {
            points: ['tc', 'bc'], // 子菜单的 "上中" 和 对应菜单的title "下中" 对齐。
            overflow: {
              adjustX: 1,
              adjustY: 1
            },
            offset: [0, 5]
        }
    }
}>
{this.renderMenuItems(menuItems)}
</Menu>
```  
至于 `placementStr` 的值 `bottomLeft` ,其实是:  

*rc-menu/es/SubMenu.js*
```
var popupPlacementMap = {
  horizontal: 'bottomLeft',
  vertical: 'rightTop',
  'vertical-left': 'rightTop',
  'vertical-right': 'leftTop'
};

var popupPlacement = popupPlacementMap[props.mode];
// 这个值最终作为"placementStr"的值,水平菜单Menu的"mode"为"horizontal"时,"placementStr"即为"bottomLeft"。  
```
`bottomLeft` , `rightTop` 的具体位置图我猜可以参考Antd的 [Popconfirm](https://ant-design.gitee.io/components/popconfirm-cn/)。  

### 最终效果图:  

![iamwinner](https://user-images.githubusercontent.com/25051945/55289823-060fad00-53fe-11e9-898a-d619df642c22.png) 

## 4. 总结  
本文针对工作中的遇到的一个菜单组件对齐问题,粗略讲到了`调试思路`,`antd组件结构`,`dom-align`, `MutationObserver 和 Debuggr for Chrome插件`,涉及代码并不复杂, 希望读者看完能有些许收获。  

感谢我那些无所不知的同事们。  