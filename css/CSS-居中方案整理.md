## 前言

认为自己的 CSS 停留在会用的阶段，遇到复杂一些的效果都是各种尝试，没有一些理性的知识做支撑。  

这次离职闲暇之余，便想着整理一下常见的 CSS 问题，下次遇到相应的场景，也能够有些参考。

## 1. 水平居中

### 1.1 行内元素或类行内元素水平居中  
利用 `text-align: center` 可以实现 **块级元素内部** 的 **内联元素** 水平居中

1. 文本

```html
<p style="text-align: center;background: #f06d06;">
  我要居中
</p>
```

2. display 值为 inline, inline-block, inline-table, inline-flex 之一

```html
<p style="text-align: center;background: #f06d06;">
  <div style="display: inline;">
    我要居中
  </div>
</p>
```

在测试以上代码的时候，发现并不能居中，而且最终渲染的结果是:  

```html
<p style="text-align: center;background: #f06d06;">
</p>
<div style="display: inline;">
  我要居中
</div>
<p>
</p>
```

也就是说， p 标签中不能嵌套 div 标签，否则会造成这种 ***bug***.  

于是换用以下代码测试:  

```html
<div style="text-align: center;background: #f06d06;">
  // 换成 inline-block, inlint-table, inline-flex 均可
  <div style="display: inline-flex;background: white;">
    我要居中
  </div>
</div>
```

### 1.2 单个块级元素水平居中


#### 1.2.1 margin: 0 auto;
通过 margin:0 auto; 使得块级元素水平居中，前提是 ***设置了居中元素的 width***

```html
<div style="background: #f06d06;">
  <div style="width: 200px;margin: 0 auto;background: white;">
    我要居中
  </div>
</div>
```

#### 1.2.2 width: fit-content

```html
<div style="background: blue; position: relative;">
  <div style="background: red; margin: 0 auto; width: fit-content;">
    我要居中 我要居中 我要居中 我要居中
  </div>
</div>
```

同上，使用 margin:0 auto; 设置要居中元素为 width: fit-content (相当于明确设置居中元素的宽度), 在 实现水平居中。  

缺点是, fit-content 的浏览器兼容略差

#### 1.2.3 绝对定位

通过设置要居中的元素向右移动父元素的 50%, 再向左移动自身元素的 -50%, 即可试下居中。

```html
<div style="background: blue; position: relative;">
  <div style="position: absolute; left: 50%; transform: translateX(-50%); background: red;">
    我要居中 我要居中 我要居中 我要居中
  </div>
</div>
```

如果已知子元素宽度，比如:  

```html
<div style="background: blue; position: relative;">
  <div style="position: absolute; width: 400px; left: 50%; transform: translateX(-50%); background: red;">
    我要居中 我要居中 我要居中 我要居中
  </div>
</div>
```

此时 tranxlateX(-50%) 还可以换成 margin-left: -200px.

另外，还可以利用 ***calc***， 设置准确的 left 值:  

```html
<div style="background: blue; position: relative;">
  <div style="position: absolute; width: 400px; left: calc(50% - 200px); background: red;">
      我要居中 我要居中 我要居中 我要居中
  </div>
</div>
```
    


### 1.3 多个块级元素水平居中

#### 1.3.1 text-align: center 方式

即通过让块级元素变为 ***inline-block*** 元素，再让父级 text-align: center 使得元素居中
```html
<div style="text-align: center;">
  <div style="background: red; display: inline-block;">
    a
  </div>
  <div style="background: red; display: inline-block;">
    b
  </div>
  <div style="background: red; display: inline-block;">
    c
  </div>
</div>
```


#### 1.3.2 flex 布局方式

通过父元素设置为 flex 布局，并设置 justify-content: center 实现居中

```html
<div style="display: flex; justify-content:center;">
  <div style="background: red;">
    a
  </div>
  <div style="background: red;">
    b
  </div>
  <div style="background: red;">
    c
  </div>
</div>
```

#### 1.3.3 使用 float 实现水平居中

原理是，设置多个元素的**共同父元素** float:left, 那么父元素的宽度即为其内容(即子元素)的宽度。子元素设置为 float: left, 保证多个子元素都在同一行。  

接着父元素 A 相对**父元素的父元素 B ** left: 50%。然后子元素 C 设置 right: 50% 相对与 A的，其实就类似 A 元素 transform: translateX(-50%), 从而实现居中。

```html
<div style="background: blue;">
  <div style="float: left; position: relative; left: 50%;">
    <div style="background: red;float: left;width: 200px;position: relative; right: 50%;">
      a
    </div>
    <div style="background: red;float: left; width: 200px;position: relative; right: 50%;">
      b
    </div>
    <div style="background: red;float: left; width: 200px;position: relative; right: 50%;">
      c
    </div>
  </div>
</div>
```

#### 1.3.4 绝对定位实现水平居中

原理同上，通过设置父元素为 position: absolute,使得父元素宽度取决于所包含的内容(即子元素)的宽度

```html
<div style="background: blue;">
  <div style="position: absolute; left: 50%;">
    <div style="background: red;float: left;width: 200px;position: relative; right: 50%;">
      a
    </div>
    <div style="background: red;float: left; width: 200px;position: relative; right: 50%;">
      b
    </div>
    <div style="background: red;float: left; width: 200px;position: relative; right: 50%;">
      c
    </div>
  </div>
</div>
```

## 2. 垂直居中

### 2.1 行内或类行内元素垂直居中

#### 2.1.1 padding-top === padding-bottom

即通过设置 padding-top 和 padding-bottom 相等，使得文字居中 (***适用于单行和多行文字***)

```html
<div style="background: blue; padding: 20px 0;">
  我要垂直居中
</div>
```

#### 2.1.2 line-height === height 垂直居中

即设置 line-height 的值与height值一样，但***只适用于单行文本***，如果有多行，则无效。

```html
<div style=" height: 40px;background: blue; line-height: 40px;">
  我要垂直居中
</div>
```

#### 2.1.3 table 垂直居中

***支持多行文本垂直居中***

设置父元素为 display: table，并设定高度。 子元素设为 display: table-cell; vertical-align: middle; 即可

```html
<div style="display: table; height: 100px;">
  <div style="background: blue; display: table-cell;vertical-align: middle;">
    我要垂直居中 我要垂直居中 我要垂直居中 我要垂直居中 我要垂直居中 我要垂直居中 我要垂直居中 我要垂直居中 
    我要垂直居中 我要垂直居中 我要垂直居中 我要垂直居中 我要垂直居中 我要垂直居中 我要垂直居中 我要垂直居中 
    我要垂直居中 我要垂直居中 我要垂直居中 我要垂直居中 我要垂直居中 我要垂直居中 我要垂直居中 我要垂直居中 
    我要垂直居中 我要垂直居中 我要垂直居中 我要垂直居中 我要垂直居中 我要垂直居中 我要垂直居中 我要垂直居中 
 </div>
</div>
```

同理，我们也可以直接用 table 标签:  

默认 tbody 的 vertivcal-align 即为 middle;
```html
<table style="background: red; height: 200px;">
  <tbody>
    <tr>
      <td>
        我要垂直居中 我要垂直居中 我要垂直居中 我要垂直居中 我要垂直居中 我要垂直居中 我要垂直居中 我要垂直居中
        我要垂直居中 我要垂直居中 我要垂直居中 我要垂直居中 我要垂直居中 我要垂直居中 我要垂直居中 我要垂直居中
        我要垂直居中 我要垂直居中 我要垂直居中 我要垂直居中 我要垂直居中 我要垂直居中 我要垂直居中 我要垂直居中
        我要垂直居中 我要垂直居中 我要垂直居中 我要垂直居中 我要垂直居中 我要垂直居中 我要垂直居中 我要垂直居中
      </td>
    </tr>
  </tbody>
</table>
```

#### 2.1.4 flex 布局垂直居中

```html
<div style="background: red; height: 200px; display: flex; flex-direction: column; justify-content: center;">
  <div>
    我要垂直居中 我要垂直居中 我要垂直居中 我要垂直居中 我要垂直居中 我要垂直居中 我要垂直居中 我要垂直居中 我要垂直居中
    我要垂直居中 我要垂直居中 我要垂直居中 我要垂直居中 我要垂直居中 我要垂直居中 我要垂直居中 我要垂直居中 我要垂直居中
    我要垂直居中 我要垂直居中 我要垂直居中 我要垂直居中 我要垂直居中 我要垂直居中 我要垂直居中 我要垂直居中 我要垂直居中
    我要垂直居中 我要垂直居中 我要垂直居中 我要垂直居中 我要垂直居中 我要垂直居中 我要垂直居中 我要垂直居中 我要垂直居中
  </div>
</div>
```

#### 2.1.5 幽灵元素实现垂直居中

即某个元素需要垂直居中，那么可以给它***加一个兄弟元素***(通过父元素为元素的方式)，其高度为父元素的 100%.  

同时设置两个元素都是 inline-block, 再通过 vertical-align：middle 实现垂直居中。  

```html
<style>
  .ghost-center:before {
    content: " ";
    display: inline-block;
    height: 100%;
    width: 1%;
    vertical-align: middle;
  }
</style>

<div style="background: red;height: 200px; position: relative;" class="ghost-center">
    // 注意 p 标签必须限制宽度
  <p style="display: inline-block;background: blue; vertical-align: middle; width: 200px;">
    我要垂直居中 我要垂直居中 我要垂直居中 我要垂直居中 我要垂直居中 我要垂直居中 我要垂直居中 我要垂直居中 我要垂直居中 我要垂直居中
    我要垂直居中 我要垂直居中 我要垂直居中 我要垂直居中 我要垂直居中 我要垂直居中 我要垂直居中 我要垂直居中 我要垂直居中 我要垂直居中
  </p>
</div>
```

### 2.2 块级元素垂直居中

#### 2.2.1 绝对定位实现垂直居中 

***元素高度已知***  

```html
<div style="background: red;height: 200px; position: relative;">
  <p style="background: blue; position: absolute; top: 50%; margin-top: -50px; height: 100px;">
    我要垂直居中 我要垂直居中 我要垂直居中 我要垂直居中 我要垂直居中 我要垂直居中 我要垂直居中 我要垂直居中 我要垂直居中 我要垂直居中
    我要垂直居中 我要垂直居中 我要垂直居中 我要垂直居中 我要垂直居中 我要垂直居中 我要垂直居中 我要垂直居中 我要垂直居中 我要垂直居中
  </p>
</div>
```

***元素高度未知***  

同水平居中一样，高度未知时可以采用 transform: translateY(-50%) 的方式：  

```html
<div style="background: red;height: 200px; position: relative;">
  <p style="margin: 0; background: blue; position: absolute; top: 50%; transform: translateY(-50%); height: 100px;">
    我要垂直居中 我要垂直居中 我要垂直居中 我要垂直居中 我要垂直居中 我要垂直居中 我要垂直居中 我要垂直居中 我要垂直居中 我要垂直居中
    我要垂直居中 我要垂直居中 我要垂直居中 我要垂直居中 我要垂直居中 我要垂直居中 我要垂直居中 我要垂直居中 我要垂直居中 我要垂直居中
  </p>
</div>
```

#### 2.2.2 flex 布局

```html
<div style="display: flex; height: 300px; background: red; align-items: center;">
  <div style="background: blue; height: 100px;">
    我要垂直居中 我要垂直居中 我要垂直居中 我要垂直居中 我要垂直居中 我要垂直居中 我要垂直居中 我要垂直居中
    我要垂直居中 我要垂直居中 我要垂直居中 我要垂直居中 我要垂直居中 我要垂直居中 我要垂直居中 我要垂直居中
  </div>
</div>
```

#### 2.2.3 table 布局

同行内元素垂直居中(***2.1.3部分内容***):  


```html
<table>
	<tbody>
    <tr>
      <td>
        <div style="background: green;">
          我要垂直居中 我要垂直居中 我要垂直居中 我要垂直居中 我要垂直居中 我要垂直居中 我要垂直居中 我要垂直居中
          我要垂直居中 我要垂直居中 我要垂直居中 我要垂直居中 我要垂直居中 我要垂直居中 我要垂直居中 我要垂直居中
          我要垂直居中 我要垂直居中 我要垂直居中 我要垂直居中 我要垂直居中 我要垂直居中 我要垂直居中 我要垂直居中
          我要垂直居中 我要垂直居中 我要垂直居中 我要垂直居中 我要垂直居中 我要垂直居中 我要垂直居中 我要垂直居中
        </div>
      </td>
    </tr>
  </tbody>
</table>
```

display: table-cell 也是一样的:  

```html
<div style="display: table; height: 300px; background: red;">
  <div style="display: table-cell;vertical-align: middle;">
    <div style="background: blue;">
      我要垂直居中 我要垂直居中 我要垂直居中 我要垂直居中 我要垂直居中 我要垂直居中 我要垂直居中 我要垂直居中
      我要垂直居中 我要垂直居中 我要垂直居中 我要垂直居中 我要垂直居中 我要垂直居中 我要垂直居中 我要垂直居中
      我要垂直居中 我要垂直居中 我要垂直居中 我要垂直居中 我要垂直居中 我要垂直居中 我要垂直居中 我要垂直居中
      我要垂直居中 我要垂直居中 我要垂直居中 我要垂直居中 我要垂直居中 我要垂直居中 我要垂直居中 我要垂直居中
    </div>
  </div>
</div>
```

## 3. 水平垂直居中

### 3.1 绝对定位

***元素宽高已知***  

```html
<div style="height: 400px; width: 400px; background: red; position: relative;">
  <div
    style="background: blue; height: 100px; width: 100px; position: absolute; left: 50%; top: 50%; margin-left: -50px; margin-top: -50px;">
    水平垂直居中
  </div>
</div>
```

absolute + calc 的方式适用于***元素宽高已知的情况***  

同理还有 transform 的方式适用于***元素宽高未知的情况***  


### 3.2 flex

略，同以上 flex 的方式  

```html
display: flex;
justify-content: center;
align-items: center;
```

### 3.3 absolute + margin: auto;

即通过绝对定位，同时设置 left,right,top,bottom 均为 0, 此时再设置 margin: auto; 即可实现水平垂直居中。前提是***已知元素的宽高***

```html
<div style="height: 400px; width: 400px; background: red; position: relative;">
  <div
    style="background: blue; height: 100px; width: 100px; position: absolute; left: 0; right: 0;top:0;bottom:0;margin: auto;">
    水平垂直居中
  </div>
</div>
```

### 3.4 使用 line-height

即通过 line-height 垂直居中， text-align: center; 实现水平居中。  

在子元素中， line-height: initial; text-align: left; 都是为了消除父元素样式对自己的影响。

```html
<div style="line-height: 200px; width: 200px; background: blue; text-align: center;">
  <div style="background: green; display: inline-block;line-height: initial; text-align: left;">
    我要垂直居中
  </div>
</div>
```

## 4. 最后说一说

本文只是对于网上答案的整理，没有原创部分，仅用于自己备忘，在实际工作中遇到相关问题能有资料参考。

该文章从动笔到现在已经两周时间了，忙于搬家，入职新公司等各种事情。现在是深夜 2019-12-09 00:52:00 ，整理完这个知识，接下来新的一周抓紧看 Vue 的文档，把 Entry Task 完成。尽管 Vue 相关技术栈和 TypeScript 都不是熟，一周的时间也觉得有点紧，但也要尽量去完成啊。  

另外，对于前端的很多的知识点，希望之后自己都能尽量弄明白，或者是自己的独立思考，或者是整理与理解网上的资料，充实自己也丰富自己的 blog。

## 参考资料
1. 核心参考: [CSS居中完整指南](https://www.w3cplus.com/css/centering-css-complete-guide.html)
2. [p标签里面不能嵌套div](https://www.cnblogs.com/lovelvxia/p/5726316.html)
3. [六种实现元素水平居中方法](https://www.cnblogs.com/chengxs/p/11231906.html)
4. 次要参考: [CSS实现水平垂直居中的1010种方式（史上最全）](https://segmentfault.com/a/1190000016389031)
5. 你能实现多少种水平垂直居中的布局（定宽高和不定宽高）:https://juejin.cn/post/6844903982960214029#heading-18