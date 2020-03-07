为了测试新语法，可以打开参考资料中的 MDN 链接，其中有在线编辑器，或者使用 Chrome console

## 前言

公司的代码中已经开始有 `??` 和 `?.` 了，刚看到这些代码有点摸不着头脑，看完 MDN 也就了解个大概。  

担心细枝末节被我忽略了，得空来整理一下

## 1. 可选链(Optional chaining) ?. 

以前我们可能会写这样的代码:  

```

const a = {
    b: {
        c: {
            d: [1]
        }
    }
}

const data = (a && a.b && a.b.c && a.b.c.d) || []
```

即为了确保不会报 `cant read property from property`, 我们需要添加很长的判断条件  

有了，可选链之后，你可以这样:

示例一:
```js
a.e.f  // 报错，a.e 为 undefined，取 undefined.f 会报错

a.e?.f // undefined, 即，使用可选链之后，不会报错

a?.e.f
// 报错，即，a?.e 只能保证即使 a 为 undefined，a.e 也不会报错
// 但是不能阻止 a.e.f 报错

// 但是
console.log(a?.e.f.d.g)   // 会报错，a?.e 为 undefined, f 不是 undefined 的属性
console.log(a.e.f?.d.g)   // 会报错，a.e 为 undefined, f 不是 undefined 的属性
console.log(a.e?.f.d.g)  // 这样居然不会报错？？？为什么


// 针对以上会报错的两种情况，可以使用多个可选链叠加一起使用
console.log(a?.e?.f?.d?.g)   // 不会报错
```

示例二: 
```js
const a = null;
a.b;  // 报错，b 不是 null 的属性

a?.b  // 不会报错，打印 undefined

a?.b.c.d.e.f  // 打印 undefined, 不会报错, 为什么 ???
```

示例三, **可选链与函数调用**:  

语法 `func?.(args)`  

```js
const a = {
    b: {
        c: {
            d: [1]
        }
    }
}
console.log(a.e?.())  // 不会报错
console.log(a.e?.b())  // 同样不会报错


const a = {
    e: {
        c: {
            d: [1]
        }
    }
}
console.log(a.e?.())  // 会报错，a.e 不是函数


就是说，通过可选链，如果 a.e 为 null 或者 undefined，调用a.e?.() 不会报错。

但是，如果存在 a.e, 但是 a.e 不是函数，那么还是会正常报错
```  

**总结**  
可选链操作符使得:  
- **当对象为 null 或者 undefined 时，读取对象的属性不会报错。**
- **当函数为 null 或者 undefined 时, 调用该函数不会报错**
- **如果遇到连续的属性访问时，可以使用多个可选链(坏处可能是被  babel 编译之后，代码量变多)**

## 2. 双问号(Nullish coalescing operator) ??

`left ?? right`  

**当 left 为 null 或者 undefined 时，返回右边的值.**

**注意:**  
和 `||` 不一样的是，如果 left 为 '' 或者 0 或者 false, `??` 依然会返回 left 的值:  

```js
const foo = null ?? 'default string';
console.log(foo);   // 返回右边的值 default string

const foo = null || 'default string';
console.log(foo);   // 返回右边的值 default string

// 遇到 false,0,'', NaN 时不一样
const baz = 0 ?? 42;
console.log(baz);   // 返回左边的值，0

const baz = 0 || 42;
console.log(baz);   // 返回右边的值，42
```

**同样，它也有短路操作**

```
function A() { console.log('A was called'); return null;}
function B() { console.log('B was called');}

console.log( A() ?? B() );
// A 函数执行，返回 undefined，
// ?? 认为 undefined 是 "false" 值，此时会继续计算 B 函数
// 最终返回 undefined(B 函数没有返回值，则默认为 undefined)

// 短路
function A() { console.log('A was called'); return false;}
function B() { console.log('B was called');}

console.log( A() ?? B() );
// A 函数执行，返回 false，
// ?? 认为 false 是 "true" 值，不再计算 C 函数， 最终返回 false
```

## 3. 可选链与双问号

可选链允许使用 `undefined?.xxx()` 而不报错，当然不报错是一回事，它的返回结果还是 **undefined**.  

因此，这种时候可以再**配合双问号来设置默认值**:  

```
undefined?.xxx()  ?? '123'  // 返回 123
```

## 参考资料

1. [JS 新语法「可选链」「双问号」已进入 Stage 3](https://juejin.im/post/5d3a7603f265da1b9254209b)
2. [MDN 可选链](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/%E5%8F%AF%E9%80%89%E9%93%BE)
3. [MDN 双问号](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/Nullish_Coalescing_Operator)