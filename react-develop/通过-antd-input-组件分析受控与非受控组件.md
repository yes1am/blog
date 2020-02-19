## 前言
一直纠结于如何开发一个供别人使用的组件，定义哪些props，如何处理别人传参，而当别人不传参的时候，组件怎么运行。遂简单分析下`antd input`组件的代码  

## 1. 分析过程与结果
[当前版本链接](https://github.com/ant-design/ant-design/blob/ae26f76d945c462b6315ebd4a3740a9fd61ca610/components/input/Input.tsx)

无论是受控还是非受控组件，内部都有state,初始值先取决于props，否则取决于defaultValue  
```
const value = typeof props.value === 'undefined' ? props.defaultValue : props.value;
this.state = {
  value,
};
```  
当用户输入值之后，此时页面上显示的input值`已经发生改变!!`，然后分两种情况.    

1. 如果是非受控组件，即父组件没有传递value，则执行：  
```
if (!('value' in this.props)) {
    this.setState({
      value: value
    });
  }
```
因为最终渲染到页面中的是this.state:  
```
renderInput(prefixCls: string) {
    const { value } = this.state;
    return this.renderLabeledIcon(
      <input
        value={fixControlledValue(value)}
      />,
    );
  }
```
因此通过`setState`将value更新为用户输入值，可使得最终渲染的值与用户输入一致。  

2. 如果是受控组件，即父元素传递了`props.value`，则不会执行`setState`,`state.value`没有发生变化:  
    1. 假设父组件的onChange没有将输入值更新到`props.value`中，由于最终渲染的是`state.value`,而`state.value`没有发生变化，则即使input在用户输入的时候，界面上的值发生了改变，但下一次`render`的时候input的值还是会还原为初始`state.value`。  
    
    2. 而如果父组件的onChange将输入值更新到props.value中,此时input的`getDerivedStateFromProps`执行：  
    
    ```
    static getDerivedStateFromProps(nextProps: InputProps) {
        if ('value' in nextProps) {
          return {
            value: nextProps.value,
          };
        }
        return null;
    }
    ```  
    将新的`props.value`设置为新的`state.value`，因为最终渲染的是`state.value`,所以`render`的时候，`input`的值发生了变化。  
    
3. 无论是否为受控组件，只要父组件传递了`onChange`事件，就会执行，**因此,可以通过onChange事件得到组件内部的状态**，但是因为非受控组件没有`props.value`,即onChange没有实际效果，因此受控组件需要`value`和`onChange`一起工作。  

```js
handleChange(value) {
  if (!('value' in this.props)) {
    this.setState({
      currentValue: value,
    });
  }
  // 只要存在该函数，就会执行
  if (onChange) {
    onChange(value);
  }
}
```

4. 非受控组件也可以接受`onChange`但不接受`value`，即最终渲染的值还是内部组件自己控制，但是可以通过`onChange`告诉父元素，现在内部的值是什么。  


## 2. 查看历史版本
[历史版本链接](https://github.com/ant-design/ant-design/tree/30fe9918d88606990eec544a6b8a476b34d816a4)  

查看antd input的过去代码，试图找到在使用`getDerivedStateFromProps`之前，它是怎么处理的，发现在之前:  
```
const { value, className } = this.props;
if ('value' in this.props) {
  otherProps.value = fixControlledValue(value);
  // input要么是受控要么不受控，只能提供values或者defaultValue，不能两个都提供
  delete otherProps.defaultValue;
}
return this.renderLabeledIcon(
  <input
    {...otherProps}
  />,
);
```

也就是说，这个版本渲染的结果是依赖于`props.value`或者`props.defaultValue`的，没有内部的`state`，同时`input onChange`是直接调用父元素的`onChange`事件，内部没有做处理。 

## 3. 疑问

那么，使用内部state和不使用内部state而采取props的方式有什么更好的呢？为什么antd后来要换成使用内部state的形式？这点暂时没有理解。  