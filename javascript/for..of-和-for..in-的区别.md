## 1. Symbol.iterator

如果一个对象实现了 Symbol.iterator 属性，那么这个对象就被认为是可迭代的。一些内置的类型，如 Array, Map, Set, String, Int32Array, Unit32Array 等等，它们的 Symbol.iterator 属性已经被实现了。 对象上的 Symbol.iterator 函数负责迭代，返回得到对象的值数组。  

**只有实现了 Symbol.iterator 属性，才能使用 for of 语法与展开运算符**  

1. 数组

数组内部原生实现了 Symbol.iterator

```js
const arr = [1,2,3]

for(let i of arr) {
    console.log(arr)  // 1, 2, 3
}
```

2. 对象

初始化的对象没有实现 Symbol.interator, 因此在对象上使用 for of 会导致报错

```js
const obj = {
  foo: 1,
  bar: 2
}

// 报错， obj is not iterable
for(let i of obj) {
  console.log(i,typeof i)
}

const obj1= [...obj];   // 报错 obj is not iterable


=============== 疑问 ===============
// 没有报错, 不知道 {...obj} 和 [...obj] 有什么区别
const obj2 = {...obj}  
```

为了使得对象能够使用 for of 语法，可以主动实现 Symbol.iterator 属性:  

```js
let length = 2;
let arr = ['a','b'];
let index = 0;
const obj = {
  a: 1,
  b: 2,
  [Symbol.iterator]: function() {
    let next = () => {
      return {
        value: this[arr[index]],
        done: length === index++
      }
    }
    return {
      next
    }
  }
}

console.log([...obj]) // [1,2]
console.log('start');
for(let i of obj) {
    console.log(i)  // 1,2
}
```

如上所示，可以主动实现 Symbol.iterator 属性。**注意以上示例中的两段代码不能同时存在**，否则，[...obj] 将迭代器的 index 增加到了 2, 再次运行 for of 的语法，done 永远不能为 true, 因此会陷入无限循环。  

## 2. for..of vs for..in

for..of 和 for.. in 都能够迭代数组，但是迭代的值是不一样的。 for..in 返回的结果是**被迭代对象的key组成的数组，key均为字符串类型**。而 for..of 返回的是**迭代对象的属性值的数组**。  

```js
const a = [4,5,6]

for(let i in a) {
  console.log(i)  // '0','1','2' 且为字符串
}

for(let i of a) {
  console.log(i)  // 4，5，6 
}
```

另外一个区别是， for..in 可以用在任何对象上，用来检查对象的属性。for..of 则主要关心可迭代对象的值，内置的对象如 `Map`, `Set` 实现了 `Symbol.iterator` 属性可以被访问到值。  

```js
let pets = new Set(['cat','dog','hamster']);

pets['specials'] = 'mammals';

for(let pet in pets) {
  console.log(pet)   // 只打印 specials
}

for (let pet of pets) {
    console.log(pet); // 打印出 "cat", "dog", "hamster"
}
```

当处于 ES5 或者 ES3 的编译环境时，迭代器只能用于迭代 Array 的值。将 for..of 用于非数组则会报错，即使这些非数组实现了 Symbol.iterator 属性.  

对于数组的情况，编译器会生成一个简单的 for 循环，来代替 for..of 循环，比如:  

```js
let numbers = [1,2,3];
for(let num of numbers) {
    console.log(num);  // 1, 2, 3
}
```
将会被转换为:  
```js
var numbers = [1,2,3];
for(var _i - 0; _i < numbers.length; _i++) {
    var number = numbers[_i];
    console.log(num);
}
```
在 ES6(ECMAScript 2015) 和更高级的编译环境中，编译器会生成 for..of 循环来执行内置的迭代器。

## 3. 更多示例

### 3.1 for in 能得到自身及原型上的属性

数组原型
```js
Array.prototype.sayHello = function() {}

Array.prototype.str = 'i am array'

const arr = [1,2,3]

arr.name = 'arr name'

for(let i in arr) {
  console.log(i);  // '0','1','2','name','str','sayHello'
}
```

对象原型
```js
Object.prototype.str = 'obj str';
Object.prototype.sayHello = function() {}

var obj = {name:'obj name', age: 'obj age'}

for(let i in obj) {
  console.log(i);  // 'name',' age', 'str', 'sayHello'
}
```

即 for..in 能够得到对象(以及数组)原型上的属性(**那为啥数组的 forEach, map，对象的 toString  等方法循环出来？？？**)  

由于可以遍历原型上的属性，如有需要，可以使用 hasOwnProperty 来进行属性的过滤:  

过滤出自身的属性
```js
for(let i in obj) {
    if(obj.hasOwnProperty(i)) {
        
    }
  console.log(i);  // 'name',' age'
}
```

### 3.2 for..of 只能迭代数组自身的值
1. 如第一部分所讲，将 for..of 直接用于对象上会报错，因为对象默认没有实现 Symbol.iterator 属性
2. 用于数组
```js
Array.prototype.say = function() {}

const arr = ['1',2];

for(let i of arr) {
    // '1' string , 2 number
  console.log(i,typeof i)
}
```

即 for..of 循环不能迭代到数组的原型上  

## 4. 总结

1. for..of 只能用在数组，或者实现了 Symbol.iterator 属性的对象上，**得到的是 "值"**
2. for..in 能用到对象和数组上，都会查找原型对象上，**得到的是 "属性"**

## 参考资料
1. [ TypeScript 中的描述](https://www.typescriptlang.org/docs/handbook/iterators-and-generators.html)
2. [JS 中 for..in 和 for..of 的区别](https://juejin.im/post/5aea83c86fb9a07aae15013b)