## 正文

在日常开发中会有这样的需求，对点击事件进行防抖处理，比如点击按钮会进行请求，那假如多次点击则不希望每次都触发请求，对后端接口不利。  

通常的做法是，每次在 `constructor` 中将点击事件处理函数变为防抖函数:  
```js
constructor(props){
  ...
  this.handleClick = this.handleClick.bind(this)
  this.handleClick = _.debounce(this.handleClick, debounceMillis)
}

render(){
  <div onClick={this.handleClick}>
  ...
  </div>
}
```

这样不好的是每次需要防抖都要写一遍重复的代码，因此基于高阶组件的思路，来写一个点击防抖组件:  

```js
import React, { Component } from 'react';
import debounce from 'lodash.debounce';
import PropTypes from 'prop-types';

class DebounceClick extends Component {
  constructor(props) {
    super(props);
    const { children, debounceMillis, persist } = props;
    const { onClick } = children.props;
    const debounceHandle = debounce(onClick, debounceMillis);
    if (persist) {
      // in debounce source code, there is async code `setTimeout`
      // so if we want to call e.persist(), we can only do here, or change debounce source code
      this.handle = e => {
        e.persist();
        debounceHandle(e);
      };
    } else {
      this.handle = debounceHandle;
    }
  }

  noop() {}

  render() {
    const { children, active } = this.props;
    return React.cloneElement(children, { onClick: active ? this.handle : this.noop });
  }
}

DebounceClick.propTypes = {
  debounceMillis: PropTypes.number,
  persist: PropTypes.bool,
  active: PropTypes.bool,
};

DebounceClick.defaultProps = {
  debounceMillis: 500, // 500 is better, 200ms for most user, this will appear almost instantly
  persist: false,
  active: true, // false means not trigger onClick
};

export default DebounceClick;
```

为了让该高阶组件尽量不与需要防抖的组件耦合，因此 `onClick` 直接从 `children.props` 中获取。  

另外需要注意的是，`React` 事件采用事件池的方式，意味着事件对象上的一些属性会在事件处理完之后 `nullify` , 而 `debounce` 内部由于存在 `setTimtout`, 导致最终的处理函数 `handleClick` 即使使用 `e.persist()` 也不能保留该事件对象。  

因此要么改动 `debounce` 源码，不过改高阶组件一样可以实现:  
```js
if (persist) {
  // in debounce source code, there is async code `setTimeout`
  // so if we want to call e.persist(), we can only do here, or change debounce source code
  this.handle = e => {
    e.persist();
    debounceHandle(e);
  };
} else {
  this.handle = debounceHandle;
}
```

另外有时候不仅是需要防抖，而且能够控制是否触发点击处理函数，因此还接受`active` 参数。  

调用示例:  
```js
<DebounceClick>
  <div onClick={this.handleClick}>
  ...
  </div>
</DebounceClick>
```

暂时先这样，若有坑再填 ~