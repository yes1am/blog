[codesandbox 链接](https://codesandbox.io/s/competent-mendeleev-kl41l)

## 1. mouseenter, mouseleave

分别在鼠标移入元素时，和移出元素时触发，**不会受子元素的影响**

1. 从外界移入父元素，**触发父元素的 mouseenter**
2. 从父元素移入子元素时
    - 子元素 ***会*** 触发 mouseenter
    - 父元素 ***不会*** 触发 mouseleave
3. 从子元素移入父元素时
    - 子元素 ***会*** 触发 mouseleave
    - 父元素 ***不会*** 触发 mouseenter

## 2. mouseover,mouseout

和 mouseenter, mouseleave 类似，**会受子元素的影响**

1. 从外界移入父元素，**触发父元素的 mouseoevr**
2. 从父元素移入子元素时,按照以下顺序执行
    1. 父元素触发 **mouseout**
    2. 子元素触发 **mouseover**
    3. 父元素触发 **mouseover**
3. 从子元素移入父元素时,按照以下顺序执行
    1. 子元素触发 **mouseout**
    2. 父元素触发 **mouseout**
    3. 父元素触发 **mouseover**

## 3. mousemove

鼠标在元素上移动时触发，**即使移入子元素，仍然会继续触发**