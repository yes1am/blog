## 前言

继承(百度百科):  
继承可以使得子类具有父类的属性和方法，或者重新定义，追加属性和方法。  

## 1. 原型链继承

即设置子类的原型为父类的实例(继承了属性与方法)

```js
function Parent(name) {
    this.name = 'parent';
}

Parent.prototype.getName = function() {
    return this.name;
}

function Child() {
    
}

Child.prototype = new Parent();
let child1 = new Child();
child.name === 'parent'  // true;
child.getName();        // 'parent'
```

我们知道 `new Parent()` 这种方式构建的实例 `parent`, 本身有 `name` 属性，同时通过 `__proto__` 原型链可以访问到 `getName` 方法。  


`Child.prototype = new Parent()` ，则 `Child` 实例也能访问到 `name` 属性和 `getName` 方法。  

缺点:  
1. 由于在 `Child.prototype` 上的属性是被所有 `Child` 实例所共享的，对于基础类型，这没有什么问题，而对于复杂类型，会存在被某个实例修改，而影响别的实例的情况:  

```js
function Parent(name) {
    this.name = 'parent';
    this.toys = ['foo', 'bar'];
}

Parent.prototype.getName = function() {
    return this.name;
}

function Child() {}

Child.prototype = new Parent();
let child1 = new Child();
let child2 = new Child();

child1.name = 'parent1'
child1.name   // parent1
child2.name   // parent2

// 直接赋值的情况，不会改变原型
child1.toys = ['foo1', 'bar1'];
child1.toys // ['foo1', 'bar1']; 
child2.toys // ['foo', 'bar'];

// 会改变原型的情况
child1.toys.push('baz')
child1.toys  // [ 'foo', 'bar', 'baz' ]
child2.toys  // [ 'foo', 'bar', 'baz' ]
```

## 2. 借助构造函数(经典继承)

即在子类中调用父类的构造函数(以此来继承属性)  

```js
function Parent() {
    this.name = 'parent';
}

function Child() {
    Parent.call(this);
}

var child1 = new Child()
```

优点:  
解决了原型链继承，原型上引用类型共享的问题


缺点：  
1. 由于是在 `Child` 构造函数内调用 `Parent` 函数来初始化 `Child` 实例的。那么 `Parent.prototype` 上的属性方法并不能访问到:  

```js
function Parent() {
    this.name = 'parent';
}

Parent.prototype.say = function() {}

function Child() {
    Parent.call(this);
}

var child1 = new Child()
child1.say();   // error, 无 say 方法
```

2. 同时, 如果想从 `Parent` 类中继承方法,那么:    

```js
function Parent() {
    this.name = 'parent';
    this.sayName = function() {
        console.log(this.name)
    }
}

Parent.prototype.say = function() {}

function Child() {
    Parent.call(this);
}

var child1 = new Child()
var child2 = new Child()
child1.sayName === child2.sayName;  // false
```

即每个实例的方法都是新创建的(占用内存)，尽管它们的代码逻辑是一样的。

## 3. 组合继承(最常用)

设置子类的原型为父类的实例，同时在子类中调用父类的构造函数。 

设置原型来继承方法，防止多次创建。子类中调用父类构造函数来继承属性，防止引用类型数据的共享。  

```js
function Parent(name) {
    this.name = name;
}

Parent.prototype.say = function() {
  console.log(this.name)
}

function Child(name) {
    Parent.call(this,name);
}

Child.prototype = new Parent();
// 以上 Child.prototype = ... 这段代码，只是想让 Child 的实例能访问得到 Parent.prototype 上的方法，那为什么不可以 Child.prototype = Parent.prototype  
// 我想是为了防止改变 Child.prototype 时，如 Child.prototype.*** = *** ，会改变 Parent.prototype 吧  
// 因此选择让 Child.prototype 指向 Parent 实例

Child.prototype.constructor = Child;  
// Child.prototype = new Parent(); 导致 child1 的 constructor 为 Parent
// 所以需要我们手动设置回去

var child1 = new Child('child1')
var child2 = new Child('child2')

child1.say === child2.say;  // say
```

## 4. 原型式继承

区别与原型链继承，原型式继承父类其实是一个对象，而不像原型链继承中父类其实是一个函数(当然函数也是对象)  

```js
function createObj(obj) {
    function F(){}
    F.prototype = obj;
    return new F();
}
```  

内部创建一个空的构造函数 `F`， 将 `F.prototype = obj`, 即将传入的对象参数，作为空函数的原型对象，最后返回 `new F()` 的实例，则最终返回的实例通过原型链能够访问到 `obj` 上的属性和方法。  

缺点:  
同原型链继承一样，由于它们的原型对象都是 `obj`, 如果存在引用类型的数据，则一个实例对属性的修改会影响另外的实例:  

```js
function createObj(obj) {
  function F() {};
  F.prototype = obj;
  return new F();
}

const origin = {
  name: 'origin name',
  toys: [1]
}
const child1 = createObj(origin)
const child2 = createObj(origin)
child1.toys.push(2)
child1.toys  // [1,2]  // 对于引用类型的修改，会影响另外的实例
child2.toys  // [1,2]
```

## 5. 寄生式继承

```js
function createObj(obj) {
  const o = Object.create(obj);
  o.say = function() {}
  return o;
}

其中 Object.create() 原理就是
Object.create = function(obj) {
    function F() {}
    F.prototype = obj;
    return new F();
}
```

缺点:  
每次实例上的方法都是新创建的方法，互不相等(尽管逻辑是一致的)。

*偷偷吐槽: 这段代码和上面那段原型式继承好像没什么区别...*  

## 6. 寄生组合式继承

在组合继承中:  

```js
function Parent(name) {
    this.name = name;
}

Parent.prototype.say = function(){}

function Child(name) {
    Parent.call(this, name);  // 调用 Parent 构造函数
}

Child.prototype = new Parent();  // 调用 Parent 构造函数

const child1 = new Child();
```

即会存在调用两次父类构造函数的情况，这导致:  `child1` 实例本身有 `name` 属性，同时 `Child.prototype` 上也有 `name` 属性。  

那能不能减少一次调用呢?  

我们知道，子类中调用父类构造函数这一次调用父类构造函数(1),是为了继承或者说初始化实例的属性。  

而将子类原型指向父类实例导致的父类构造函数调用(2)，其实只是为了继承父类原型上的方法或属性。  

因此，我们能不能将指向父类原型这次父类构造函数调用省去，只能访问到父类的原型，即 `Child.prototype` 能够访问到 `Parent.prototype`？  

这时候，想到前面的原型式继承，就是传入一个对象 `obj`，最终返回的对象，能够访问到 `obj` 对象上的属性。因此我们进行改造:  

```js
function Parent(name) {
    this.name = name;
}

Parent.prototype.say = function(){}

function Child(name) {
    Parent.call(this, name);  // 调用 Parent 构造函数
}

// 原型式继承
function createObj(obj) {
    function F() {}
    F.prototype = obj;
    return new F();
    // 即其实还是调用了某个构造函数，只是该构造函数是个空函数，所以性能更好吧
}

Child.prototype = createObj(Parent.prototype)  
// 不直接 Child.prototype = Parent.prototype 的原因前面说过
// 防止修改 Child.prototype 会影响 Parent.prototype

Child.prototype.constructor = Child;

const child1 = new Child();
```  

最后封装一下继承的方法:  

```js
function createObj(obj) {
    function F() {}
    F.prototype = obj;
    return new F();
}

function extends(Child, Parent) {
    Child.prototype = createObj(Parent.prototype);
    Child.prototype.constructor = Child;
}
```  

优点:  
1. 这种方法的高效率体现在只调用一次 Parent 构造函数
2. 避免了在 `Parent.prototype 上创建不必要的属性(应该是 Child.prototype上吧，毕竟 Child.prototype = new Parent() 会创建多余属性)
3. 原型链还能保持不变  

开发人员普遍认为寄生组合式继承式引用类型最理想的继承范式  


## 参考资料

1. [JavaScript深入之继承的多种方式和优缺点](https://github.com/mqyqingfeng/Blog/issues/16)
