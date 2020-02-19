## 1. 前言

一直对于`antd`的源码耿耿于怀，日常开发中遇到要开发组件总是不清楚怎么去设计接口，抄`antd`也苦于其组件层层封装且代码分散在各个角落，无从抄起。  

在工作间隙之中，粗略的翻看了`dom-align`的源码，`antd` 内部许多定位的弹窗类组件都基于它，`dom`对齐在日常开发中也有需求，遂有了本文。  

## 2. 解读  

[ dom-align 仓库](https://github.com/yiminghe/dom-align)

### 2.1 API 

使用：  
```
import domAlign from 'dom-align';

const alignConfig = {
  points: ['tl', 'tr'], // 将source元素的 `top - center` 与 target 元素的 `top - right` 对齐
  offset: [10, 20],  // `source` 的 x偏移量是10， y的偏移量是 20
  targetOffset: ['30%','40%'], // `target` 的 x 偏移量是30%, y 的偏移量是 40%, 百分比是基于target
  overflow: { adjustX: true, adjustY: true }, // 当 source 被遮挡住的时候，是否进行自动调整
};

domAlign(sourceNode, targetNode, alignConfig);
```

### 2.2 source code

`dom-align` 暴露 `{alignElement, alignPoint}` 两个方法，默认暴露的是 `alignElement` 方法  


#### 2.2.1 alignElement方法
```
function alignElement(source, refNode, align) {
  // const target = align.target || refNode;  // 这里说明 target 也能够从 align,即方法的第三个参数进行传入  

  const refNodeRegion = getRegion(target);  // 获取 target 元素的 { width , height ,距离文档顶部的 top, left 等信息}

  const isTargetNotOutOfVisible = !isOutOfVisibleRect(target);

  return doAlign(source, refNodeRegion, align, isTargetNotOutOfVisible);
}
```

先暂时只看到 `getRegion` 部分,看看该方法做了什么:  

#### 2.2.2 getRegion方法
```
function getRegion(node) {
  let offset;
  let w;
  let h;
  if (!utils.isWindow(node) && node.nodeType !== 9) {
    // window 或者 document
    offset = utils.offset(node);   // offset 目标元素相对文档的左上距离
    w = utils.outerWidth(node);
    h = utils.outerHeight(node);
  } else {
    const win = utils.getWindow(node);
    offset = {
      left: utils.getWindowScrollLeft(win),
      top: utils.getWindowScrollTop(win),
    };
    w = utils.viewportWidth(win);
    h = utils.viewportHeight(win);
  }
  offset.width = w;
  offset.height = h;
  return offset;
}
```

如果该元素不是 `window` 元素或者 `document` 元素,则去取 该元素的 `offset`信息，`offset` 信息即为 `元素距离Document即整个文档的左上角的left值和top值`。  


#### 2.2.3 offset方法

`offset`信息会等于元素的`getBoundingClientRect()`中返回的`left, top`加上对应`window横向和竖向滚动的距离`。

由于`getBoundingClientRect()`返回的只是元素对应 **浏览器可视区域** 的`width,height,left,top`等值，则`该left+浏览器横向滚动距离` = `元素距离整个文档最左边的距离`。  

不过，除了这样简单计算之外,其代码中还会考虑到竖向滚动条的宽度：`document.documentElement.clientLeft || body.clientLeft`。  

然后`offset`还会携带元素的`width和height`信息,整体像是增强版的`getBoundingClientRect`,不够此时返回的`left,top`是相对于文档的，而不是相对于可视区域的。  

#### 2.2.4 getWHIgnoreDisplay方法
在获取元素的`width和height`信息时，该库封装了一个`getWHIgnoreDisplay`方法，看名字即为"获取元素的宽高信息而不管元素的display属性值"  

#### 2.2.5 getWHIgnoreDisplay方法
```
if (elem.offsetWidth !== 0) {
  val = getWH.apply(undefined, args);
} else {
  swap(elem, cssShow, () => {
    val = getWH.apply(undefined, args);
  });
}
```
该方法首先判断 `elem.offsetWidth`是否为0 ,因为当元素`display:none`的时候，宽度为0.  

而当元素`display:none`的时候，先通过 `swap(elem, cssShow)`,来看下`swap`的代码：  

#### 2.2.6 swap方法
```
const cssShow = {
  position: 'absolute',
  visibility: 'hidden',
  display: 'block',
};

function swap(elem, options, callback) {
  ... 

  // 记住旧的属性值
  for (name in options) {
    if (options.hasOwnProperty(name)) {
      old[name] = style[name];
      style[name] = options[name];
    }
  }

  callback.call(elem);

  // 还原旧的属性值
  for (name in options) {
    if (options.hasOwnProperty(name)) {
      style[name] = old[name];
    }
  }
}
```
就是当`display:none`的时候，先让元素`display:block`,通过`visibility`在页面上不可见，但是可以获取到该元素的样式信息，之后再将元素还原回`display:none`.  

#### 2.2.7 getWH

```js
 @param elem
 @param name
 @param {String} [extra]
 'padding' : (css width) + padding
 'border' : (css width) + padding + border
 'margin' : (css width) + padding + border + margin
 */
function getWH(elem, name, extra) {
  ...
}
```
该方法封装了获取一个元素各种宽高的方法，将`padding,border,margin`都考虑在内。  

最终 `getRegion` 方法返回了元素 相对文档的`left, top`信息，以及`width, height` 信息。  

####  2.2.8 isOutOfVisibleRect方法

*isOutOfVisibleRect方法*  
```
function isOutOfVisibleRect(target) {
  const visibleRect = getVisibleRectForElement(target);
  // 获取当前元素的可视区域。有点绕，其实是获取会影响该元素显示区域的，祖先元素内容区域的交集  
  // 返回值为相对整个当前文档的 { left , right , bottom , top }值，通过四个值（相当于四条线）能确定一个矩形区域

  const targetRegion = getRegion(target);
  // 获取目标元素 相对文档的`left, top`信息，以及`width, height` 信息

  return !visibleRect ||
    (targetRegion.left + targetRegion.width) <= visibleRect.left ||
    (targetRegion.top + targetRegion.height) <= visibleRect.top ||
    targetRegion.left >= visibleRect.right ||
    targetRegion.top >= visibleRect.bottom;
}
```

先来看`getVisibleRectForElement`：  

```
// 首先可视区域为整个文档
const visibleRect = {  
    left: 0,
    right: Infinity,
    top: 0,
    bottom: Infinity,
};

// 获取能影响该元素显示的祖先元素
// 对于定位元素来说，即使最近的父级定位元素
// 对于非定位元素来说，则返回直接父元素
let el = getOffsetParent(element);

...

// 即通过依次向 目标元素 的祖先元素递归,查找到会影响该元素显示的祖先元素
// 得到 目标元素 元素的相对文档的 { left,right,top,bottom }
// 简单理解为祖先中每一层父级都会对可视区域进行削减
// 因此所有`影响显示区域`的祖先元素的区域的交集即为最终的结果

while (el) {
  // 获取父级元素相对文档的距离
  const pos = utils.offset(el);     
  
  // 将border的宽度计算进来
  pos.left += el.clientLeft;
  pos.top += el.clientTop;
  
  // 以下代码即为 做区域的交集,不断的缩小区域, 不考虑滚动条
  visibleRect.top = Math.max(visibleRect.top, pos.top); 
  visibleRect.right = Math.min(visibleRect.right, pos.left + el.clientWidth);
  visibleRect.bottom = Math.min(visibleRect.bottom, pos.top + el.clientHeight);
  visibleRect.left = Math.max(visibleRect.left, pos.left);

  el = getOffsetParent(el); 
}

...

// 然后是一些根据 document，和window 元素对区域进行剪裁的代码

// ele.style.position 只能返回行内样式 position 中的值，不能返回 css 样式表中的值
// window.getComputedStyle(ele).position 会返回元素最终的 position 值，不管是style中还是css中的


// 最后确保该可视区域是 `正常的` 因为 top 小于 0 是不现实的。内容不可能在文档之外

return (
visibleRect.top >= 0 &&
  visibleRect.left >= 0 &&
  visibleRect.bottom > visibleRect.top &&
  visibleRect.right > visibleRect.left
) ? visibleRect : null;
```

#### 2.2.9 doAlign方法
该方法为最终的实现`dom`对齐的方法:  

*doAlign方法*  

其中参数依次为 `source` 元素，目标元素的 `width,height`,距离文档左上角的`left,top`值，然后是传入的 `align` 参数：  

```js
function doAlign(source, tgtRegion, align, isTgtRegionVisible) {

  // 处理 `align` 的参数
  ...

  let fail = 0;
  // getVisibleRectForElement方法之前讲过，就是根据元素的祖先节点中能够影响元素显示区域的元素  
  // 来计算元素的显示区域  
  const visibleRect = getVisibleRectForElement(source);
  
  // 计算元素当前在文档中所占的区域, left/top/width/height
  const elRegion = getRegion(source);
 

  // 计算出元素即将被放置的位置
  let elFuturePos = getElFuturePos(elRegion, tgtRegion, points, offset, targetOffset); 
  // 返回 { left , top }

  ...
```

先看到这里，即通过知道 `source` 元素和 `target` 元素 **相对文档** 的位置与大小，得到`source`即将被防止的位置:  

*getElFuturePos方法*
```js
/**
 * 
 * @param {*} elRegion: source 元素占据的区域 {left, top , width, top}
 * @param {*} refNodeRegion: target 元素占据的区域 {left, top , width, top}
 * @param {*} points  ['tr','cc'] align source t[op] r[ight] with target c[enter] c[enter]
 * @param {*} offset 
 * @param {*} targetOffset 
 */
function getElFuturePos(elRegion, refNodeRegion, points, offset, targetOffset) {
  // refNodeRegion 即为target元素，即将 `target元素与points[1]`做计算
  // 得到目标的对齐点的坐标
  const p1 = getAlignOffset(refNodeRegion, points[1]);
  
  // 得到source的对齐点左边
  const p2 = getAlignOffset(elRegion, points[0]);
  
  // 两者做差值，即为source元素需要进行移动的距离
  const diff = [p2.left - p1.left, p2.top - p1.top];

  // 再将offset的值考虑进来
  return {
    left: Math.round(elRegion.left - diff[0] + offset[0] - targetOffset[0]),
    top: Math.round(elRegion.top - diff[1] + offset[1] - targetOffset[1]),
  };
}
```

*getAlignOffset*方法的代码也很简单：  
```
/**
 * 获取 node 上的 align 对齐点 相对于页面的坐标
 * 比如说要得到target元素的对齐点位置,对齐在 target 的[br]
 * 首先传入的参数{left,top} 是 target的左上角位置
 * br 对应top-right即右下角，那么
 * right = left + width
 * bottom = top + width
 * 如果有 center , 则要除于 2
 */

function getAlignOffset(region, align) {
  const V = align.charAt(0);
  const H = align.charAt(1);
  
  const w = region.width;
  const h = region.height;

  let x = region.left;
  let y = region.top;

  if (V === 'c') {
    y += h / 2;
  } else if (V === 'b') {
    y += h;
  }

  if (H === 'c') {
    x += w / 2;
  } else if (H === 'r') {
    x += w;
  }

  return {
    left: x,
    top: y,
  };
}
```

继续回到 `doAlign` 方法:  

```js
  // 当前节点可以被放置的显示区域
  const visibleRect = getVisibleRectForElement(source);
  // 当前节点所占的区域, left/top/width/height
  const elRegion = getRegion(source);
  
  ...
  
  // 当前节点将要被放置的位置, 返回 { left , top } 值
  let elFuturePos = getElFuturePos(elRegion, tgtRegion, points, offset, targetOffset); 

  // 当前节点将要所处的区域
  // 即通过即将被放置的位置计算出即将所处的区域
  let newElRegion = utils.merge(elRegion, elFuturePos);

  // 如果不能完全展示，则进行反转
  // 反转之后，如果不是 【完全不能显示】 === 不能完全显示， 则记fail为1， 且保存为新的config

  // 下面这段代码用户当
  // 如果可视区域不能完全放置当前节点时允许调整
  if (visibleRect && (overflow.adjustX || overflow.adjustY) && isTgtRegionVisible) {
    if (overflow.adjustX) {
      // 如果横向不能放下
      if (isFailX(elFuturePos, elRegion, visibleRect)) {
      
        // 对齐的位置点 反一下,将 l => r , r => l
        const newPoints = flip(points, /[lr]/ig, {
          l: 'r',
          r: 'l',
        });
        // 偏移量也反转下
        const newOffset = flipOffset(offset, 0);
        const newTargetOffset = flipOffset(targetOffset, 0);
        const newElFuturePos = getElFuturePos(
          elRegion,
          tgtRegion,
          newPoints,
          newOffset,
          newTargetOffset
        );

        // 只要不是完全失败，就进行赋值
        if (!isCompleteFailX(newElFuturePos, elRegion, visibleRect)) {
          fail = 1;
          points = newPoints;
          offset = newOffset;
          targetOffset = newTargetOffset;
        }
      }
    }

    // 调整纵向的位置
    ...
```

上面的代码涉及到`isFailX`和`isCompleteFailX` 两个方法：  
```
/**
 * 
 * @param {*} elFuturePos source future { left , top }
 * @param {*} elRegion source now { left , top , width , height }
 * @param {*} visibleRect  { left , right, top , bottom }
 * 
 * 判断 x 方向是否 不能完全显示，即是否会有区域超过可视区，形成遮挡
 * 条件是 即将的left 小于 可视区域的left，也就是有一部分内容超过了可视区域的左边界
 * 即将的right + 宽度 大于 可视区域的right，也就是有一部分内容超过了可视区域的右边界
 */
function isFailX(elFuturePos, elRegion, visibleRect) {
  return elFuturePos.left < visibleRect.left ||
    elFuturePos.left + elRegion.width > visibleRect.right;
}


/**
 * 判断 x 方向是否完全不能显示， 即区域完全超出可视区，一点都看不到
 * 条件是 即将的left 大于 可视区域的right， 也就是整个元素在可视区域的右边
 * 即将的left + 宽度 小于 可视区域的left，也就是整个元素在可视区域的左边
 */
function isCompleteFailX(elFuturePos, elRegion, visibleRect) {
  return elFuturePos.left > visibleRect.right ||
    elFuturePos.left + elRegion.width < visibleRect.left;
}
```

整体看一下 *doAlign* 方法：  
1. 通过计算 source 元素当前位置，根据传入的参数，计算出即将所在的位置
2. 如果参数允许调整x和y方向的值，也即将所在的位置有内容会被遮挡，则进行位置的调整
3. 调整方法为 先反向调整，x方向交换 `lr`，y方向交换 `tb`
4. 反转之后检查
    1. 如果反转之后完全放不下，啥也不干
    2. 反转之后，不是完全放不下，则根据反转之后的位置重新计算将要被放置的位置
5. 判断是否任然放不下(感觉这段代码只适用于 **反转之后不是完全放不下的情况，因此此时才会重新计算将要被放置的位置，而确实完全放不下的情况，则这段代码与之前判断 isFail 没啥区别**)，如果仍然放不下，且确实需要调整，则可能会改变元素的宽度高度了  
6. 最后通过 utils.offset 方法设置最终的 `left,top` 样式

*setOffset方法*  
```
function setOffset(elem, offset, option) {
  if (option.ignoreShake) { 
    // 如果传参 忽视抖动，则不保留小数
    const oriOffset = getOffset(elem);

    const oLeft = oriOffset.left.toFixed(0);
    const oTop = oriOffset.top.toFixed(0);
    const tLeft = offset.left.toFixed(0);
    const tTop = offset.top.toFixed(0);

    if (oLeft === tLeft && oTop === tTop) {
      // 如果目标宽高和当前宽高一致，则直接返回
      return;
    }
  }


  if (option.useCssRight || option.useCssBottom) {
    setLeftTop(elem, offset, option);
  } else if (option.useCssTransform && getTransformName() in document.body.style) {
    setTransform(elem, offset, option);
  } else {
    setLeftTop(elem, offset, option);
  }
}
```

已知原来相对于文档的`left和top`，和最终相对文档的`left和top`值：  

1. *setTransform*方法，`translate`的值即为: 
```
// 即只需要移动差值部分即可
resultXY.x = originalXY.x + (offset.left - originalOffset.left);
resultXY.y = originalXY.y + offset.top - originalOffset.top;
```

2. *setLeftTop方法*稍微复杂些，实际上是`根据相对于文档的 {left, top} 值。设置元素样式上的left和top值`:  

```
// 根据相对于文档的 {left, top} 值。设置元素样式上的left和top值
function setLeftTop(elem, offset, option) {
  // set position first, in-case top/left are set even on static elem
  if (css(elem, 'position') === 'static') {
    elem.style.position = 'relative';
  }
  let presetH = -999;
  let presetV = -999;
  const horizontalProperty = getOffsetDirection('left', option);
  const verticalProperty = getOffsetDirection('top', option);
  const oppositeHorizontalProperty = oppositeOffsetDirection(horizontalProperty);
  const oppositeVerticalProperty = oppositeOffsetDirection(verticalProperty);

  if (horizontalProperty !== 'left') {
    presetH = 999;
  }

  if (verticalProperty !== 'top') {
    presetV = 999;
  }
  let originalTransition = '';
  // 记下一开始相对文档的偏移量
  const originalOffset = getOffset(elem);  
  
  if ('left' in offset || 'top' in offset) {
    originalTransition = getTransitionProperty(elem) || '';
    setTransitionProperty(elem, 'none');  
    // 保存 transition的值，即当前阶段做 left 和 top 的改变，是没有动画的
    // 因为当前阶段是在做计算，而非真正的设置样式
  }
  if ('left' in offset) {
    elem.style[oppositeHorizontalProperty] = '';
    elem.style[horizontalProperty] = `${presetH}px`;
  }
  if ('top' in offset) {
    elem.style[oppositeVerticalProperty] = '';
    elem.style[verticalProperty] = `${presetV}px`;
  }

  // 先设置为 -999 
  // force relayout
  forceRelayout(elem);
  const old = getOffset(elem);  // 记下变为预设之后的偏移量
  const originalStyle = {};
  for (const key in offset) {
    if (offset.hasOwnProperty(key)) {
      const dir = getOffsetDirection(key, option);  // 根据 useRight ，当传入left,useRight为true时，返回right
      const preset = key === 'left' ? presetH : presetV;
      const off = originalOffset[key] - old[key];   
      // 计算出由于 刚刚设置为预设{ 999, 999 }，导致元素实际移动的 距离
      
      // 比如元素最初实际的样式值为 { left: 20, top: 20 }
      // 但是 可能相对`文档`的 {left: 0, top: 0}, 因为一个元素最终相对文档的left，top还会受到父元素的影响
      // 后来样式设置为预设值 { left: -999, top: -999 }
      // 此时再计算出相对文档的 {left,top} 值，可以知道元素实际移动的距离
      // 通过预设值与实际移动的距离进行计算，得出最初元素的样式值
      // 大概类似 让你走到 999米的地方，但是你说你只走了990米，说明你一开始就在9米的位置处。进行反推
      // (为啥不通过getComputedStyle取呢？)
      if (dir === key) {
        originalStyle[dir] = preset + off;
      } else {
        originalStyle[dir] = preset - off;
      }
    }
  }
  // 这一步是为了复原回原来的样式值，取消掉原来预设的值。
  css(elem, originalStyle);
  
  // force relayout
  forceRelayout(elem);
  if ('left' in offset || 'top' in offset) {
    setTransitionProperty(elem, originalTransition);
    // 还原 transition的值 但是好像依然没有动画？
  }
  
  // 现在已知 元素相对文档的 {left,top}
  // 还知道 元素最终需要的 相对文档的 {left,top}值
  // 则只需要
  const ret = {};
  for (const key in offset) {
    if (offset.hasOwnProperty(key)) {
      const dir = getOffsetDirection(key, option);
      const off = offset[key] - originalOffset[key];
      if (key === dir) {
        ret[dir] = originalStyle[dir] + off;
      } else {
        ret[dir] = originalStyle[dir] - off;
      }
    }
  }
  css(elem, ret);
}
```

(现在有点怀疑这么一长段代码有没有必要，originalStyle直接通过getComputedStyle取不可以吗？嗯，经过测试，发现应该是可以的，和我理解的没差)  

## 3. 结语

通过粗略的分析，对`dom-align`有了更加清晰的认识，不再是完完全全的`黑盒子`，之后日常开发中如果需要实现`dom`对齐的需求也可以视情况引入。  

代码中的中文注释对我来说挺友好的，有一些小技巧比如`getWHIgnoreDisplay`,一些小代码片段`getPBMWidth`在日常开发中都可以使用上(这是最酷的，总有各种代码片段别人已经实现了且经受住了开源的考验，你理解完之后就可以放心的用了，如果不符合要求也能有把握去修改源码)。  

接下来，有空要从`dom-align`往上去阅读`antd`的源码了~