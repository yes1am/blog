symbol类型的值是通过Symbol构造函数创建的。

```js
let sym1 = Symbol();
let sym2 = Symbol('key');   // 字符串参数
```

Symbols是不可改变且唯一的  

```
let sym1 = Symbol("key");
let sym2 = Symbol("key");

sym1 === sym2; // false, symbols是唯一的
```

像字符串一样，symbols也可以被用做对象属性的键。  

```js
let sym = Symbol();

let obj = {
    [sym]: "value"
};

console.log(obj[sym]); // "value"
```  

**内置的Symbol**:  

内置symbols用来表示语言内部的行为

| Symbol | 描述 |
|---|---|
| Symbol.hasInstance | 方法，会被instanceof运算符调用。构造器对象用来识别一个对象是否是其实例。|
|Symbol.isConcatSpreadable | 布尔值，表示当在一个对象上调用Array.prototype.concat时，这个对象的数组元素是否可展开。|
| Symbol.iterator | 方法，被for-of语句调用。返回对象的默认迭代器。 |
|Symbol.match |方法，被String.prototype.match调用。正则表达式用来匹配字符串。 |
|Symbol.replace |方法，被String.prototype.replace调用。正则表达式用来替换字符串中匹配的子串。 |
|Symbol.search |方法，被String.prototype.search调用。正则表达式返回被匹配部分在字符串中的索引。 |
|Symbol.species |函数值，为一个构造函数。用来创建派生对象。 |
|Symbol.split |方法，被String.prototype.split调用。正则表达式来用分割字符串。 |
|Symbol.toPrimitive |方法，被ToPrimitive抽象操作调用。把对象转换为相应的原始值。 |
|Symbol.toStringTag |方法，被内置方法Object.prototype.toString调用。返回创建对象时默认的字符串描述。 |
| Symbol.unscopables |对象，它自己拥有的属性会被with作用域排除在外。|


例如，你可以这样来自定义 MyArray 的 instanceof 行为:  
```js
class MyArray {  
  static [Symbol.hasInstance](instance) {
    return Array.isArray(instance);
  }
}
console.log([] instanceof MyArray); // true
```