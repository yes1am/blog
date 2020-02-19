## 1. 类型断言

两种类型断言
```
1. 
var foo = <foo>bar  

2. 

var foo = bar as foo;
```

由于 `jsx` 语法的原因，只能使用 `as` 操作符  


## 2. 类型检查

对于原生dom标签，如 div， span 总是小写字母开头。 对于自定义组件都是大写字母开头  


## 3. 固有元素(dom标签)
如果你想在JSX中使用 <foo /> ，通常是会报错的，因为foo不是dom标签，通常是不会出现在 `JSX.IntrinsicElements`接口中的，如果要能够支持 foo ，则必须扩展该接口：  

```js
declare namespace JSX {
    interface IntrinsicElements {
        foo: any  // 扩展
    }
}

<foo />; // 正确
<bar />; // 错误

也可以使用以下代码，全部通过

declare namespace JSX {
    interface IntrinsicElements {
        [elemName: string]: any;
    }
}
```