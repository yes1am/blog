## 1. Snapshot Test

```js
import React from 'react';
import { render } from 'enzyme';
import XXXComponent from './XXXComponent';

// 可以使用 render 或者 shallow
// 测试了一下，mount 偶尔会报 timeout 的错误
// 具体使用什么 api， 可以参考 antd
describe('<XXXComponent /> Component', () => {
  it('should match snapshot', () => {
    const component = render( <XXXComponent />);
    expect(component).toMatchSnapshot();
  });
})
```

### 1.1 更新快照结果
`jest -u` 或者 `npm run test -- -u`

## 2. 模拟事件

### 2.1 click 事件
```js
const component = mount(<XXXComponent />);
// before click, expect
component.find('.XXX').at(1).simulate('click');
// after click, expect
```

### 2.2 change 事件

```js
// 找到 input type=checkbox 的元素
// 注意, 第二个参数需要模拟 event 所需的参数
component.find('XXX').at(0).simulate('change',{target:{checked:true}})
```

### 2.3 keydown 键盘事件

```js
// 40 为按键码
component.find('XXX').at(0).simulate('keydown',{keyCode: 40 })
```

## 3. 更新 props

如果想测试组件 props 的变更，可以使用 setProps

```js
import XXXComponent from './XXXComponent'

test('XXX', () => {
    // 新建一个组件，包裹被测组件
    const Demo = (props) => {
      return (
        <XXXComponent value={props.value} />
      );
    };
    const component = mount(<Demo value='tab1' />);
    // before set props, expect
    component.setProps({value:"tab2"})
    // after set props, expect
});
```

## 4. 测试函数执行

```js
// 新建模拟函数
const checkedFun = jest.fn();
// 找到几个元素，且元素 .checked 为 true
// 则执行一遍函数
component.find('.XXX').forEach(node => {
  if(node.props().checked) {
    checkedFun()
  }
})
// 检验函数执行的次数
expect(checkedFun).toBeCalledTimes(0)

// 当 props 改变时，
const newSelectedRowKeys = [0,1,2,3,4,5];
component.setProps({selectedRowKeys:newSelectedRowKeys})

// 再次校验函数执行次数
component.find('.XXX').forEach(node => {
  if(node.props().checked) {
    checkedFun()
  }
})
expect(checkedFun).toBeCalledTimes(newSelectedRowKeys.length)
```

## 5. 通过 .props() 获得组件的 props

> 只能用于 Class 组件

如例4，其中 node 为 <input type='checkbox' /> ,我们通过判断 `node.props().checked` 来判断该元素是否被选中。  

## 6. 通过 .state() 获得组件的 state

> 只能用于 Class 组件

```js
// 如 change 事件触发之后，我们可以检验 state 是否发生变化
component.find('XXX').at(0).simulate('change',{target:{checked:true}})

expect(component.state()[ state的key ]).toBe(预期的值)
```

## 7. 模拟 mouseenter 

```js

html结构
<dropdown onMouseEnter={xxx}>
    <button />
</dropdown>


component.find('dropdown').at(0).simulate('mouseenter')  // 符合预期
component.find('button').at(0).simulate('mouseenter')   // 不符合预期
```

即在模拟 mouseenter 的时候，最好是给**原绑定事件的元素去模拟事件**(而不是想着事件会冒泡，而click 事件确实能利用冒泡)，不然可能会出现异常。


## 8. 获取原生 dom 对象

*getDOMNode()*  通过该方法可以获得原生的dom 对象
```js
expect([...component.getDOMNode().classList].includes('shopee-table')).toBe(true)
```

## 注意事项

1. 千万不要传入 this 作为 props

```js
<A>
    <B parent={this} />
</A>
```

在平常写代码中，如果 B 组件中要用到许多 A 组件中的值(如 props 或者 state ), 为了偷懒，我们可能会将 `this` 作为参数传递下去。然后在 B 组件中通过 `this.parent.xxx` 来获取对应的属性。  

千万不要这样做，在配合 enzyme mount 方法做测试的时候，会形成递归。造成内存不够用的情况。  

> FATAL ERROR: Ineffective mark-compacts near heap limit Allocation failed - JavaScript heap out of memory