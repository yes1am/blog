## 1. 关键概念

### 1.1 层叠顺序

层叠顺序，即在**同一层叠上下文**中的排序规则：  

1. 形成层叠上下文的背景和边框
2. `定位` 的带有 负值 `z-index` 的元素,该元素形成 `子的层叠上下文`
3. `非定位` 的块级元素
4. `非定位` 的 `浮动` 元素
5. `非定位` 的行内元素
6. `定位` 的 `z-index` 为 `0或者auto` 元素
7. `定位` 的 `z-index` 大于 `0` 元素

同一层叠顺序按照在HTML中出现的顺序层叠

### 1.2 形成层叠上下文的因素

1. 根元素html
2. 相对，绝对定位元素，且`z-index`不为`auto`
3. `position:fixed的元素`，不关心`z-index`的值
4. `z-index`不为`auto` 的flex项目，即父元素为`display:flex|inline-flex`
5. `opacity`小于1
6. `transform` 属性值不为`none`
7. `mix-blend-mode`属性值不为`normal`的元素
8. `filter`值不为`none`的元素
9. `perspective`值不为`none`的元素
10. `isolation`值为`isolate`的元素
11. `will-change`中指定任意css属性
12. `-webkit-overflow-scrolling`属性值为`touch`的元素

子元素的z-index值只在父级层叠上下文中有意义。没有创建自己层叠上下文的元素将被父级层叠上下文同化。  

## 2. MDN文档分析

### 2.1 [Stacking without z-index](https://developer.mozilla.org/zh-CN/docs/Web/Guide/CSS/Understanding_z_index/Stacking_without_z-index)  

#### 示例解释
因为`div1-div5` 祖先元素中只有`html`产生了层叠上下文，即`div1-div5`属于一个层叠上下文，符合以上规则。  

div1-div4属于【6】：`z-index`为`0或者auto`的定位元素。div5属于【3】： `非定位` 的块级元素。因此div5在div1-div4下面。  

div1-div4同属于【4】，因此按照html中的顺序依次堆叠。  

### 2.2 [堆叠与浮动](https://developer.mozilla.org/zh-CN/docs/Web/Guide/CSS/Understanding_z_index/Stacking_and_float)

#### 示例解释
同样，div1-div5都属于html这个层叠上下文。  

div1，div5 属于【6】：`z-index`为`0或者auto`的定位元素。  
div2,div3属于【4】：`非定位` 的 `浮动` 元素。  
div4属于【3】：`非定位` 的块级元素。  

因此顺序是：div4,div2,div3,div1,div5。  

**疑问**：当给div4设置opacity非1之后(产生层叠上下文)，顺序成了div2,div3,div1,div4,div5.像是理解为div4成了定位元素。因此在div1与div5之间，因为HTML中出现的顺序是div1,div4,div5。

### 2.3 [Adding z-index](https://developer.mozilla.org/zh-CN/docs/Web/Guide/CSS/Understanding_z_index/Adding_z-index)  

#### 示例解释

同样，div1-div5都属于html这个层叠上下文。  

div1-div4同属于【7】：`定位` 的 `z-index` 大于 `0` 元素。  
div5虽然有`z-index`但是对于该元素为`非定位`元素，所以无效,因此属于【3】：`非定位` 的块级元素。

在div-div4中,dom与z-index的关系:  
```
div1: 5
div2: 3
div3: 2
div4: 1
```  

因此最终顺序为：div5,div4,div3,div2,div1.  

### 2.4 [层叠上下文](https://developer.mozilla.org/zh-CN/docs/Web/Guide/CSS/Understanding_z_index/The_stacking_context)


#### 示例解释

div1-div6都形成了层叠上下文，属于【2】：相对，绝对定位元素，且`z-index`不为`auto`。  

div1-div3 父层叠上下文为html，即祖先元素中唯有html为层叠上下文。  
div4-div6 父层叠上下文为div3，即最近的层叠祖先元素为div3。  

```
div1: 5
div2: 2
div3: 4
    div4: 6  理解为 4.6
    div5: 1  4.1  
    div6: 3  4.3
```

因此层叠顺序为: div2, div3, div5, div6, div4, div1  

### 2.5 [Stacking context example 1](https://developer.mozilla.org/zh-TW/docs/Web/CSS/CSS_Positioning/Understanding_z_index/Stacking_context_example_1)

#### 示例解释

div1-div4都是定位元素，div1是div2父元素，div3是div4父元素。

此时因为div1-div4都没有生成层叠上下文，同属于html层叠上下文中，所以层叠顺序为: div1,div2,div3,div4。  

因为在同一堆叠上下文中，故此时给div2或者div4设置z-index时，即会使得div2或者div4在其它元素之上。  

### 2.6 [Stacking context example 2](https://developer.mozilla.org/zh-CN/docs/Web/Guide/CSS/Understanding_z_index/Stacking_context_example_2)

#### 示例解释

div1-div4都是定位元素，div1是div2父元素，div3是div4父元素。

div1-div3同属于html层叠上下文，div4属于div3层叠上下文：  

```
div1 : auto
div2 : 2
div3 : 1
    div4: 1.10
```  
因此层叠顺序为: div1,div3,div4,div2  

### 2.7 [Stacking context example 3](https://developer.mozilla.org/zh-CN/docs/Web/Guide/CSS/Understanding_z_index/Stacking_context_example_3)  

#### 示例解释

dom结构:  
```
level1
    container1
        level2
            container2
                level3
                level3
        level2
            container2
                level3
                level3
level1
```  

一级菜单level1相对定位，但没有产生层叠上下文。  

此时如果container1没有产生层叠上下文，则level2与level1同属html层叠上下文。此时根据HTML中出现的顺序，第一个level1下的level2会出现在第二个level1下面。

为使得所有level2出现在所有level1上面，则可以给level2设置z-index，(level2产生了层叠上下文)。同一层叠上下文下，z-index决定层级。  


### 2.8 [没人告诉你关于z-index的一些事](https://www.w3cplus.com/css/what-no-one-told-you-about-z-index.html)

#### 示例解释

```
<html>
<div class='div1'>
  <span class="red">Red</span>
</div>
<div class='div2'>
  <span class="green">Green</span>
</div>
<div class='div3'>
  <span class="blue">Blue</span>
</div>
</html>

.red, .green, .blue {
  position: absolute;
}
```

即 red,green,blue 都是绝对定位，且它们的父级div1-div3都没有形成层叠上下文，所以所有的元素同属于一个html的层叠上下文,因此给.red添加z-index会使得该元素显示在green和blue上面。

为了让red显示在green，blue下面，首先根据html出现顺序，div1层级在div2和div3下面，则只要保持层级不变，同时让div1变成层叠上下文，则.red的层级作用在div1上，即能让.red在div2,div3下面。  

### 2.9 antd中的弹窗类组件

antd中一些弹出类的组件，假如结构:  

```
<html>
    <div class='div1'>
        <Modal>
    </div>
    <div class='div2'>  // 疑问2，它必须是层叠上下文「opacity:0.99」时，123才会显示在modal上面. 为什么不是默认就在modal上面(按照HTML出现顺序排列)
        123
    </div>
</html>
```
即使你给modal很高的层级，可能仍然覆盖不了123： 比如这种情况:  
```
div.div1 {
    opacity: 0.99;   //任何能使它成为层叠上下文，又不影响原有层级的属性(即不要设置定位元素与z-index)
}
```
此时div1的层级必然小于div2,则Modal的层级z-index再大也没效果，都不能覆盖123。此时的解决办法是让div1的层级大于div2的层级。比如: 

```
div.div1 {
    position: relative;
    z-index:1;
}
```
但此时得确保div.div2的层级小于div1,即不能设置定位且z-index>=1. 否则123又会被覆盖。  

基于以上的所有情况，最好的办法就是antd那样，将Modal的实际DOM作为body的直接子元素，加在body的最后。  

**注意：只有z-index会影响层叠顺序，opacity只是创造层叠上下文，不会改变层叠顺序，因此:**
```
div.div1 {
    opacity: 0.99;   // 此时div1任然在div2下面，但是由于创造了层叠上下文，导致.red只是在div1中起效果
}
```

### 3. 参考资料及疑问

*在2.2和2.9中分别有一处暂时不能解释的疑问*  

[MDN：理解css的z-index属性](https://developer.mozilla.org/zh-CN/docs/Web/Guide/CSS/Understanding_z_index)  
[antd中弹窗类组件默认作为body子组件的参考](https://github.com/react-component/select/issues/404)  
[没人告诉你关于z-index的一些事](https://www.w3cplus.com/css/what-no-one-told-you-about-z-index.html)  
[z-context chrome插件，分析元素的层叠上下文以及z-index](https://github.com/gwwar/z-context/blob/master/devtools/index.js)  