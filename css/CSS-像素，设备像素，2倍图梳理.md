## 前言

一直对于移动端网页开发都不咋熟悉，之前也写过移动端的页面，但是似乎并没有遇到什么问题，就正常写 css。对于一些概念也不熟悉，什么两倍图，移动端兼容问题等。因为后续会进入 react-native 的开发，想着整理一些概念也是有必要的。  
## 1. [前端不止：Retina屏幕下两倍图](https://juejin.im/entry/5a2a01126fb9a0451543ca05) 概括

**屏幕分辨率:**  
屏幕有多少个像素(也称为设备像素，物理属性，不可变)，`width` * `height`。比如 iPhone7, 分辨率是 `1334 * 750`。  

*疑问: 电脑的调整分辨率是干嘛用的？*  

**图片大小:**  
图片长宽的像素个数: `width` * `height`，如某张图片是 `100` * `100`  

> 假如一张 1334 * 750 的图片在 iPhone7 中进行完全展示，那么一个图片像素对应一个设备像素，因此是完全保真的显示。  

**像素密度(PPI):**  
每英寸所拥有的的像素**数目**, PPI 数值越大，代表显示屏能够**以更高的密度显示图像**，画面的细节会更丰富。  

> 以Retina屏幕为例，它并不是像普通显示器那样通过增大尺寸来增加分辨率，而是靠提升屏幕单位面积内的像素数量，即 PPI 来提升分辨率，这样就有了高像素密度屏幕

### 1.1 web 中的 CSS 像素

> CSS像素是一个抽象概念，设备无关像素，简称 "DIPS"，device-independent像素，主要使用在浏览器上，用来精确的度量（确定）Web页面上的内容。  

> 同样的 css 大小，在 mobile，pc 上显示的大小是一样的，(如用尺子测量都是 2cm)。  

**devicePixelRatio设备像素比:**  

即 **设备物理像素(屏幕分辨率)**与**设备独立像素(css像素)** 的比例:  `window.devicePixelRatio = 物理像素 / 设备独立像素`  

- 普通密度桌面显示屏的devicePixelRatio = 1
- 高密度桌面显示屏(Mac Retina)的devicePixelRatio = 2
- 主流手机显示屏的devicePixelRatio = 2 或 3


```css
.box {
    width: 200px;
    height: 200px;
}
```

以上代码在设备上绘制一个 200*300 像素的盒子，(**该大小在所有物理设备上看起来都一致**)  

在 devicePixelRatio 为 1 时，真实设备会用 200 * 300 个设备像素去渲染它。但是在 devicePixelRatio 为 2 时，真实设备会用 400 * 600 个设备像素去渲染。  

对于图片来说，一张照片的大小就是 200 * 300 像素，用 200 * 300 的设备像素，能保真的显示。而如果使用 400 * 600 的设备像素去渲染，就会模糊 (**可以理解为被拉伸,被放大**) 。  


因此，引入了 **2倍图，3倍图的概念**。  

也就是同一张图片，做成 1倍图，像素是: 200 * 300，做成 2倍图，像素是 400 * 600.  

图片容器 css 设置为 200 * 300:  
```
// css 样式
.img {
    width: 200px;
    height: 300px;
}
```
在 devicePixelRatio 为 1 的屏幕中, 设置的 css 像素对应的真实物理像素为 200 * 300，引入一倍图，像素刚好是 200 * 300, **一个物理像素对应一个图片像素,可以保真显示**。  

在 devicePixelRatio 为 2 的屏幕中, css 像素任然是 200 * 300, 对应的真实物理像素为 400 * 600 (width * 2, height * 2), 这时候**引入二倍图**, 像素刚好是 400 * 600, **一个物理像素对应一个图片像素,可以保真显示**。

使用媒体查询更换图片:  

```
#element { background-image: url('hires.png'); }

@media only screen and (min-device-pixel-ratio: 2) {
    #element { background-image: url('hires@2x.png'); }
}

@media only screen and (min-device-pixel-ratio: 3) {
    #element { background-image: url('hires@3x.png'); }
}
```

## 参考资料

1. [前端不止：Retina屏幕下两倍图](https://juejin.im/entry/5a2a01126fb9a0451543ca05)
2. [什么是三倍图？——移动端尺寸知识入门](https://zhuanlan.zhihu.com/p/34988701)
3. [使用Flexible实现手淘H5页面的终端适配](https://github.com/amfe/article/issues/17)