## 正文

实现一个日历选择器，其中日历部分是一个弹层，有个常见的需求时，当点击日历框之外的部分，则日历框消失。而其中日历框是处于一个`React.createPortal()`创造出来的结构中,而日历框因为兼容移动端和online端，又会有两个组件`CalendarMobile`和`CalendarPC`.  

大致的嵌套结构是:  
```
CheckDate
  DateInput
  CalendarMobile       -- Portal
    or
  CalendarPC           -- Portal
```
其中`DateInput`点击之后，`Portal`组件出现，点击`Portal`组件内部不做处理，点击`Portal`之外关闭`Portal`。

基于[React事件处理](https://github.com/yes1am/blog/issues/11)的知识,之前的实现是:  
1. 在window上添加`click`事件监听，在监听到`click`之后触发关闭弹层。  
2. `CalendarMobile`和`CalendarPC`组件上最顶层使用`e.nativeEvent.stopImmediatePropagation`阻止事件冒泡至`window`,也就不会触发到`window`上的事件了。  
3. `DateInput`上也添加`e.nativeEvent.stopImmediatePropagation`阻止冒泡。  

但是后来发现可以在`CheckDate`组件中统一处理，查看[相关文档](https://reactjs.org/docs/portals.html#event-bubbling-through-portals)  
> 尽管portal可以在DOM的任意位置，但是它就表现的跟正常Reach child一样，像 Context 还是正常的工作，因为 portal依然存在于 React Tree，而不在意实际在DOM中的位置  
> 同样 Event 冒泡也是一样的，portal 内部触发的事件依然会传递到React Tree中的祖先组件中，尽管祖先组件在DOM tree中并不是 portal 组件的祖先元素。

[示例代码](https://codepen.io/gaearon/pen/jGBWpE)