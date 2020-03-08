
[官网文档](https://www.iconfont.cn/help/detail?spm=a313x.7781069.1998910419.d8d11a391&helptype=code)

## 1. icon 单个使用

可以下载 png 或者 svg  

使用 png 的情况，可以在下载的时候可以选择颜色与大小。  

而使用 svg 的情况，可以在代码中调整大小与颜色，如:  

```html
<svg t="1583669324443" class="icon"  width="200" height="200">
    <path d="xxxx" fill="#d81e06" p-id="1860">
    </path>
</svg>

// 改变大小和颜色
<svg t="1583669324443" class="icon"  width="100" height="100">
    <path d="xxxx" fill="green" p-id="1860">
    </path>
</svg>
```

使用 svg 时，通过 `svg` 标签上的 widht 和 height 可以设置大小，通过改变 `path` 标签上的 fill 来改变填充颜色.

**适合于引入图标比较少，不需要特别维护的场景**

## 2. unicode 使用

**缺点:**
- 因为是字体，所以不支持**多色(即一个图标多个颜色)**。只能使用平台里单色的图标，就算项目里有多色图标也会自动去色

**使用示例**: `<span class="iconfont">&#xe706;</span>`, 即通过文本 `&#xe706` 来显示图标

**步骤:**  

添加 icon 到购物车，选择下载代码, 下载下来的代码中，有 demo_index.html 文件，其中会看到所选图标的 unidcode， 比如:  

```
<span class="icon iconfont">&#xe706;</span>
<div class="name">巴黎</div>
```

即 `&#xe706;` 代表巴黎这个图标

于是构建如下样式代码:  
```css
// css 样式，用于引入字体文件: 
@font-face {
    font-family: "iconfont";
    src: url('./font/iconfont.eot?t=1583671571755');
    /* IE9 */
    src: url('./font/iconfont.eot?t=1583671571755#iefix') format('embedded-opentype'),
    /* IE6-IE8 */
    url('./font/iconfont.woff?t=1583671571755') format('woff'),
    url('./font/iconfont.ttf?t=1583671571755') format('truetype'),
    /* chrome, firefox, opera, Safari, Android, iOS 4.2+ */
    url('./font/iconfont.svg?t=1583671571755#iconfont') format('svg');
    /* iOS 4.1- */
}

.iconfont {
    font-family: "iconfont" !important;
    font-size: 16px;
    font-style: normal;
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
}

// 其中，设置 font-size 和 color 可以改变图标的大小和颜色
.iconfont {
    ...
    font-size: 100px;
    color: red;
    ...
}
```

html 中使用即可查看到效果:  
```html
<i class="iconfont">&#xe706;</i>
```

## 3. font-class 引用

**缺点:**
- 本质上还是 unicode，所以也不支持**多色(即一个图标多个颜色)**

**使用示例**: `<i class="iconfont icon-bali"></i>`

unicode 方式的问题在于，`&#xe706;` 这样的一段代码和巴黎图标对应，**语意非常不明确**，于是出现了 font-class 的方式:  

**步骤:**  

添加 icon 到购物车，选择下载代码, 下载下来的代码中，有 `iconfont.css` 文件，其内容示例如下:  

```
@font-face {
  font-family: "iconfont";
  src: url('iconfont.eot?t=1583671571755'); /* IE9 */
  src: url('iconfont.eot?t=1583671571755#iefix') format('embedded-opentype'), /* IE6-IE8 */
  // 这里有一段很长的代码啊，暂时删除
  url('iconfont.woff?t=1583671571755') format('woff'),
  url('iconfont.ttf?t=1583671571755') format('truetype'), /* chrome, firefox, opera, Safari, Android, iOS 4.2+ */
  url('iconfont.svg?t=1583671571755#iconfont') format('svg'); /* iOS 4.1- */
}

.iconfont {
  font-family: "iconfont" !important;
  font-size: 16px;
  font-style: normal;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}

.icon-bali:before {
  color: red;
  content: "\e706";
}

.icon-huashu:before {
  content: "\e70b";
}

.icon-limao:before {
  content: "\e70e";
}

// 其中，同样通过设置 font-size 和 color 可以改变图标的大小和颜色
.iconfont {
  color: "blue";
  font-size: 16px;
}
```
实际上，就是把 unicode 隐藏在了 `:before` 伪元素中，使用方式:  

```html
<i class="iconfont icon-bali"></i>
```

## 4. symbol 引用 (推荐)

原理就是一次性将图标 svg 打入 iconfont.js 中，使用时通过 `<use xlink:href="#icon-bali"></use>` 来使用特定的图标

**优点:**  
- 支持**多色图标**，不再受单色限制


**使用示例**:  
```html
<svg class="icon" aria-hidden="true">
    // 通过 xlink:href 设置图标
    <use xlink:href="#icon-bali"></use>
</svg>
```

**步骤:**  

添加 icon 到购物车，选择下载代码, 下载下来的代码中，有 `iconfont.js` 文件,将其引入页面，同时加入通用的 css 代码:  

```html
<script src="./font/iconfont.js"></script>
<style type="text/css">
    .icon {
        width: 1em;
        height: 1em;
        vertical-align: -0.15em;
        fill: currentColor;
        overflow: hidden;
    }
</style>

// 使用
<svg class="icon" aria-hidden="true">
    // 通过 xlink:href 设置图标
    <use xlink:href="#icon-bali"></use>
</svg>


// 其中，可以通过设置 font-size, 和 fill 来改变图标的大小和颜色
<style type="text/css">
    .icon {
        ...
        fill: red;         // 这里设置 fill 无效
        font-size: 200px;  // 设置 font-size 有效
        ...
    }
    #icon-bali path {
      fill: red;           // 通过这里设置颜色
    }
</style>
```
