## 缩进层级

### 使用制表符进行缩进
每一个缩进层级都用单独的制表符（tab character）表示.  

缺点：系统对制表符的解释不一致，某个系统用编辑器打开文件时看到的缩进和另一个系统用相同编辑器打开文件时看到的不一样。

### 使用空格符进行缩进
每个缩进层级由多个空格字符组成，在这种观点中有三种具体的做法：2个空格，4个空格和8个空格。  

好处：所有系统和编辑器中，对于空格的展示形式不会有任何差异，且可以在文本编辑器中配置敲击Tab键时插入几个空格。  
缺点：使用没有配置好的文本编辑器创建格式化的代码的方式非常原始。  

## 行的长度
很多语言的编程规范都提到一行代码最长不要超过80个字符。  
### 换行
通常在运算符后换行，下一行会增加两个层级的缩进。  
```js
// 好的做法，运算符后换行，第二行两个缩进
callFunction(document, element,
        window, "some string value");
        
// 不好的做法，一个缩进
callFunction(document, element,
    window, "some string value");

// 不好的做法，运算符之前换行
callFunction(document, element
        , window, "some string value");
```
在这个例子中，逗号是一个运算符，应当作为前一行的行尾。  


对于语句来说，同样可以采用以上的规则。  
```
if(isLeapYear && isFebruary && day === 29 &&
        noPlans) {
    waitAnotherFourYear();            
}
```
if语句被拆分成两行，断航在运算符&&之后，另外if语句的主题部分依然是`一个`缩进。  

这个规则有一个例外，当给变量赋值时，第二行的位置应该和赋值运算符的位置保持对齐,确保代码的可读性。
```
var result = a + b + c + d + 
             e + f;
```

## 空行
通常来讲，代码应该是一系列可读的段落，而不是一大段揉在一起的连续文本。有时候一段代码的语义和另一段代码不相关，这时就应该用空行将它们分割，确保语义有关联的代码展现在一起。  

最初的代码
```
if(w1 && w1.length){
    for(i =0; l = w1.length; i < l; i++){
        p = w[i];
        if(s.hasOwnProperty(p)){
            if(merge){
              Y.mix(r[p])  
            } else {
                r[p] = s[p]
            }
        }
    }
}
```
添加空行之后  
```
if(w1 && w1.length){

    for(i =0; l = w1.length; i < l; i++){
        p = w[i];
        
        if(s.hasOwnProperty(p)){
        
            if(merge){
              Y.mix(r[p])  
            } else {
                r[p] = s[p]
            }
        }
    }
}
```
这段示例代码展示的编程规范是在每个流程控制语句之前比如（if,for）添加空行。另外还有一些最佳实践：  
- 在方法之间
- 在方法的局部变量和第一条语句之间
- 在多行或单行注释之前
- 在方法内的逻辑片段之间添加空行，提高可读性  

## 命名
> “计算机科学只有两个难题，缓存失效和命名”  

Camel Case和Pascal Case都翻译成驼峰式大小写，Camel Case表示“小驼峰式大小写”即首字母小写命名法。Pascal Case指“大驼峰式大小写”即首字母大写命名法。  

此外还有一种`匈牙利命名法`，特点是名字之前已类型标识符作为前缀，比如sName表示字符串，iCount表示整数。  

### 变量和函数
变量名应当总是遵守驼峰式大小写命名法，并且命名前缀应当是名词，而函数名前缀应当是动词。

```js
// 好的写法
var count = 10;
var myName = "Nicholas";
var found = true;
function getName(){  // 动词
    return myName;
}

// 不好的写法
var getCount = 10;
var isFound = true;
function theName(){  // 名词
    return myName;
}
```

尽量在变量名中体现出值的数据类型。比如：count,length,size表明数据类型是数字。name,title,message表明数据类型是字符串。而单个字符命名的变量如i,j,k则通常在循环中使用。尽量不要使用诸如foo,bar和tmp之类的无意义命名。  

常见的函数和方法名动词约定：  
|动词|含义|
|---|---|
|can | 函数返回一个布尔值|
|has | 函数返回一个布尔值|
|is | 函数返回一个布尔值 |
|get|函数返回一个非布尔值|
|set|函数用来保存一个值|  

#### 常量
通用的命名约定常量，来自于C语言，以大写字母和下划线来命名。比如：  
```
var MAX_COUNT = 10;
VAR URL = "HTTP://www.baidu.com";
```

### 直接量
JavaScript中包含一些类型的原始值，字符串，数字，布尔值，null和undefined,同样有包含对象直接量和数组直接两，这其中，只有布尔值是自解释的(self-explanatory)的。

#### 多行字符串
```js
// 不好的写法
var longString = "here is the story of a \
man named bredy";

// 好的写法
var longString = "here is the story of a " + 
                 "man named bredy";
```

#### 数字
```
// 不推荐没有小数部分
var price = 10.;

// 不推荐没有整数部分
var price = .5;

// 不推荐八进制写法(已废弃)
var num = 010;
```

#### null
何种情况下使用null：  
- 用来初始化一个变量，这个变量可能赋值为一个对象。
- 用来和一个已经初始化的对象比较，这个变量可以是也可以不是一个对象。
- 当函数的参数期望是对象时，用作参数传入。
- 当函数的返回值期望是对象时，用作返回值传出。  

何种情况下不应该使用：
- 不要用null来检测是否传入了某个参数
- 不要用null来检测一个未初始化的变量  

理解null最好的方式是将它当做对象的占位符。  

#### undefined
那些没有被初始化的变量都有一个初始值，即undefined，表示这个变量等待被赋值。  
```js
// 不好的写法
var person;
console.log(person === undefined); //true
```
尽管这段代码能够正常工作，但建议在代码中使用undefined，这个值容易和返回“undefined”的typeof运算符混淆。  

不管是值为undefined或者变量未声明，typeof运算的结果都是undefined.  
```js
// foo 未声明
var person;
typeof person;   // "undefined"
typeof foo;   // "undefined"
```
通过禁止使用使用特殊值undefined，可以有效的确保只有在`变量未声明时`typeof才会返回"undefined",而声明变量时，建议将其赋值为null.  

#### 对象直接量和数组直接量
不赞成使用构造函数来创建数组或对象。  
```js
// 不好的写法
var a = new Array(1,2,3);
var b = new Object();
b.title = "js"
```

## 注释
### 单行注释
- 独占一行的注释，用来解释下一行代码，这行注释前总有一个空行，且缩减层级和下一行代码保持一致
- 在代码行的尾部的注释。代码结束到注释之间有至少一个缩进。注释+代码部分不应该超过单行最大字符数限制，如果超过了应该将注释放置在当前代码行的上方。
- 被注释掉的大段代码  

```js
// good
if(condition){
    
    // 如果代码执行到了这里，表示通过了所有安全性检查
    allowed()
}

// bad: 注释前没有空行
if(condition){
    // 如果代码执行到了这里，表示通过了所有安全性检查
    allowed()
}

// bad: 错误的缩进
if(condition){
// 如果代码执行到了这里，表示通过了所有安全性检查
    allowed()
}


// good
var a = b + c; // b不应该取值为null

// bad: 代码和注释之间没有间隔
var a = b + c;// b不应该取值为null

// good
// 当逻辑比较复杂时，需要使用多行注释
// 更好的描述逻辑关系
// 就像这样
if(condition){
    allowed()
}
```

### 多行注释
JAVA风格：  
```
/*
 * 另一段注释
 * 这段注释包含两行文本 
 */
```

多行注释出现在要描述的代码段之前，注释和代码之间没有空行间隔。且多行注释之前应当有一个空行，缩进层级应该和其描述的代码保持一致。  

```js
// good
if(condition){
    /*
     * 这是注释
     * 多行注释
     */
     allowed()
}

// bad 星号之后没有空格
if(condition){
    /*
     *这是注释
     *多行注释
     */
     allowed()
}

// bad 错误的缩进
if(condition){
/*
 *这是注释
 *多行注释
 */
     allowed()
}
```

#### 何时使用注释
- 难以理解的代码，当代码中隐藏某种特殊的逻辑时，比如 `YUI类库`中Y.mix(), `if(mode === 2)` 诸如此类简化的逻辑代码，作者自己对于数字有约定时，需要将约定内容用注释表示出来。
- 可能会认为是错误的代码

```js
while(element && element = element[axis]) { // 提示，赋值操作
    
}
```
- 浏览器特性hack,当针对特殊浏览器做兼容时，为兼容代码做好注释。  

#### 文档注释 JavaDoc

### 块语句间隔
主要有三种风格：  
- 在语句名，圆括号和左花括号之间没有空格间隔

```js
if(condition){
    doSomething();
}
```
- 左圆括号之前和右圆括号之后各添加一个空格  `推荐`

```js
if (condition) {
    doSomething();
}
```
- 括号前后都添加一个空格 `jquery`

```js
if ( condition ) {
    doSomething();
}
```

### switch
```js
switch('2'){
    case '1':
        console.log(1);
        break;
    case '2':
        console.log(2);
    default:
        console.log(3)
}

// 最终会打印 2和3，因为2之后没有break
```
#### default
```js
switch (condition){
    case 'fir':
        ...
    case 'sec':
        ...
    default:
        // 什么也没做
}
```
即使default中没有做任何逻辑，也建议在switch中加上default，同时给default加上注释。也可以如下：  
```js
switch (condition){
    case 'fir':
        ...
    case 'sec':
        ...
    // 没有default
}
```

### 使用条件语句来代替continue
```js
var a = [1, 2, 3, 4, 5], i, len;
for(i = 0; i < len; i++){
    if (i !== 2) {
        doSomething();
    }
}
```

### for in循环
for-in循环用来遍历对象属性存在一个问题，即不仅可以遍历对象的实例属性，也可以遍历从原型上继承而来的属性。因此，最好同时使用hasOwnProperty来过滤出实例属性。如果确实需要查询原型链属性时，需要额外添加注释。  
```js
var prop = null;
for (prop in object) {
    if (object.hasOwnProperty(prop) ){
        doSomething();
    }
}
```
#### 不要使用for in来遍历数组

## 变量声明提前
由于JS的声明提前，因此建议将局部变量作为函数内的第一条语句。  
```js
function doSomething() {
    var i, len;
    var value = 10;
    
    for(...){
        
    }
}
```
## 尽可能使用严格模式

## 相等
发生类型转换的场景就是，使用了判断相等运算符 `===` 和 `!==` 的时候，当要比较的两个值的数据类型不同时，就会发生强制类型转换。  

转换规则：  
1. 如果将数字和字符串进行比较，字符串会首先转换为数字。
2. 如果布尔值和数字比较，布尔值会转换为数字。false = 0，true = 1.
3. 如果其中一个值是对象，另一个值不是对象，则先调用对象的valueOf方法，得到原始值之后再进行比较。如果没有valueOf方法则调用toString方法。

## eval
eval会将传入的字符串当做代码执行，通过该函数来载入外部的js代码。  
```js
eval("alert('1')")
```
使用Function构造函数也可以做到这一点，setTImeout和setInterval也可以。  
```js
var myFun = new Function('alert(1)');
setTimeout("document.body.style.background='red'",50);
setInterval("document.body.style.background='red'",50);
```
原则：严禁使用Function，尽可能不用eval，定时器第一个参数不要用字符串形式，要用函数形式。  

## 原始包装类型
三种原始包装类型：String,Boolean,Number。原始包装类型的主要作用是让原始值具有对象般的行为。  
```js
var name = "michael";
console.log(name.toUpperCse());   // MICHAEL
name.author = true;    // 这行代码结束后，author属性就消失了
console.log(name.author);  // undefined, name.author创建的对象消失了
```
尽管name时一个字符串原始值不是对象，但是当调用toUpperCase方法的时候，JS引擎内部为其创建了String类型的新实例，紧接着被销毁了，当需要时会再创建另一个对象。  

> 原始值不具备对象特性，1.toString() 是报错的，必须 var a = 1; a.toString();

## 松耦合
如果两个组件耦合太紧，则说明一个组件和另一个组件直接相关，这样修改一个组件的逻辑，另一个组件的逻辑也必须修改。组件知道的越少，越有利于形成整个系统。

### 将js从css中抽离：css expression
```.css
.box{
    width: expression(document.body.offsetWidth + 'px')
}
```
浏览器会以高频率重复计算css表达式，严重影响性能。  

### 将css从js中抽离
```js
// bad
element.style.color = 'red';
element.style.left = '10px';
element.style.top = '100px';

// bad
element.style.cssText = "color:red;left:10px;top:10px";
```
最佳方法是操作css的className：  
```js
// good  原生方法
element.className += 'show'；
// good html5
element.classList.add("show");
```

### js从html中抽离
```
// bad
<button onclick="doSomething()">click me</button>
```

### addEventListener
DOM2级，IE8及之前不支持：  
```js
function addListener(target, type, handler){
    if (target.addEventListener) {
       // 第三个参数fals 表示非捕获模式 
       target.addEventListener(type, handler, false) 
    } else if (target.attachEvent) {
        target.attachEvent("on" + type, handler);
    } else {
        target["on" + type] = handler;
    }
}
```
###  将HTML从JS中抽离
在JS中使用HTML的场景通常时给innerHTML属性赋值时，这是非常不好的实践。解决办法：  
- 讲模板放置于远程服务器，使用XMLHttpRequest对象获取外部标签。  

```js
var xhr = new XMLHttpRequest();
// true代表异步
xhr.open('get','/js/dialog/',true)
xhr.send(null)
xhr.onreadystatechange = function() {
    if (xhr.readyState === 4 && xhr.status === 200){
        div.innerHTML = xhr.responseText;
    }
}

// jquery中的load
$('#dig-loader').load('/js/dialog/')
```
- 简单客户端模板 replace回调

```js
function sprintf(text) {
    var i = 1, args = arguments;
    return text.replace(/%s/g, function() {
        return (i<args.length) ? args[i++] :""
    })
}
var result = sprintf('<a href="%s">'</a>, "/item/4", "fourth item")
```

- 复杂客户端模板 handlebars

### 将字符串转为dom插入文档中
element.insertAdjacentHTML('beforeend', result);  

## 单全局变量
- YUI 定义全局YUI对象
- jQuery创建全局 $ 和 jQuery
- Dojo创建全局dojo对象
- CLosure创建全局 goog 对象

```js
var M = {};
M.Book = function(){}
M.book.prototype.turnPage = function(){}

M.book1 = new M.Book();
```
全局只有一个M对象。  

### 命名空间
在单全局对象之下创建很多的命名空间，YUI.DOM下的所有方法都是与DOM操作有关的，YUI.Event之下的所有方法都是和事件有关的。  

### 异步模块定义 AMD
```js
// AMD
define('module-name', ['dependency1','dependency2'], function(d1,d2) {
    // 模块正文
})
```
要想使用AMD模块，需要与之兼容的模块加载器如：requirejs。  

## 零全局变量
```js
(function(win) {
    var doc = win.document;
    // other code
}(window));
```

## 事件处理
```js
// bad 这段代码的逻辑应该是，只依赖x和y的数值进而改变dom的位置，不应该依赖于事件
function handleClick(event) {
    var popup = document.getElementById("popup");
    popup.style.left = event.clientX + 'px';
    popup.style.top = event.clientY + 'px';
}

element.addEventListener('click', handleClick)
```
### 隔离应用逻辑
```js
var myApplication = {
    handleClick: function(event) {
        this.showPopup(event);
    },
    showPopup(event){
        var popup = document.getElementById("popup");
        popup.style.left = event.clientX + 'px';
        popup.style.top = event.clientY + 'px';
    }
}

element.addEventListener('click', function(event) {
    myApplication.handleClick(event)
})
```
### 不要分发事件对象
即事件对象event从匿名事件处理函数传递到了handleClick中，之后传递到showPopup中。  

应用逻辑不应当依赖于event对象来完成正确的功能，原因如下：  
- 方法接口没有标明哪些数据是必要的。好的API一定是对于期望和依赖都是透明的，将event对象传入并不能告诉event对象的哪些属性是有用的，用来干什么？  
- 因此如果要测试这个方法，必须创建一个event对象作为参数传入，且需要明确知道使用了哪些信息才能写出测试代码。  

```js
// good
var myApplication = {
    handleClick: function(event) {
        this.showPopup(event.x, event.y);
    },
    showPopup(x, y){
        var popup = document.getElementById("popup");
        popup.style.left = x + 'px';
        popup.style.top = y + 'px';
    }
}

element.addEventListener('click', function(event) {
    myApplication.handleClick(event)
})
```

当处理事件时，最好让事件处理程序称为接触到event对象的唯一的函数，事件处理程序在进入应用逻辑之前针对event对象做必要的操作，包括阻止默认事件以及事件冒泡。  
```js
// best
var myApplication = {
    handleClick: function(event) {
        event.preventDefault();
        event.stopPropagation();

        this.showPopup(event.x, event.y);
    },
    showPopup(x, y){
        var popup = document.getElementById("popup");
        popup.style.left = x + 'px';
        popup.style.top = y + 'px';
    }
}

element.addEventListener('click', function(event) {
    myApplication.handleClick(event)
})
```

## 检测值
### 原始值
五种原始数据类型：字符串，数字，布尔值，null和undefined。  
- 对于字符串            typeof 返回 'string'
- 对于数字              typeof 返回 'number'
- 对于布尔值            typeof 返回 'boolean'
- 对于undefined         typeof 返回 'undefined'

对于null，可直接用 === 和 !== 判断， `document.getElementById("my-div") === null`

### 引用值
除了原始值之外都是引用，Object, Array, Date和Error，使用typeof检测都返回object: `typeof [] === 'object'`  

`typeof null === 'object'` 应杜绝使用typeof检测null的类型。  

检测引用值最好的方式是使用instance of方法： ` value instanceof constructor`  

```js
//检验日期
if(value instanceof Date) {}
// 检验正则
if(value instanceof RegExp) {}
// 检验error
if(value instanceof Error) {}
```
instanceof 不仅可以用来检测构造这个对象的构造器，还可以检测原型链。比如，默认所有对象都继承自Objct，因此每个对象的`value instanceof Object` 都返回 true.
```js
// 检验自定义类型
function Person() {
    
}
var me = new Person();
me instanceof Person;  // true
```
浏览器帧 frame A里的一个对象被传入到另一个帧 framee B中，两个帧里都定义了构造函数Person，来自帧A的对象是帧A的person的实例，则:  
```js
// true
frameAPersonInstance instance frameAPerson

// false
frameAPersonInstance instance frameBPerson
```
因此`instanceof` 不能跨帧使用。

### 检测函数
```js
myFun instanceof Funciton;  // true 但不能跨帧使用

// 更建议使用 typeof 方法,可以跨帧使用。
typeof myFun === 'function'      // true
```

### 检测数组
```js
// 鸭式辨型
function isArray(value) {
    return typeof value.sort === 'function'
}

// 流行方案
function isArray(value) {
    return Object.prototype.toString.call(value) === '[object Array]';
}

```

### 检测属性
```js
// bad 检测假值
if(object[propertyName]) {}

// bad 和null想比较
if(object[propertyName] != null) {}

// bad 和undefined比较
if(object[propertyName] != undefined) {}
```
以上代码均在通过给定名字检测对象上该属性的值，而非判断给定的名字对应的属性是否存在，因为当属性值就是 false，null，undefined时判断会出错。  

判断属性是否存在最好的方法是使用 `in` 运算符，只会简单的判断属性是否存在，而不会读属性的值。当实例对象的属性存在或者 `继承自对象的原型` ，则返回true。  
如果只想检测实例对象的属性是否存在，通过hasOwnProperty方法，判断原型对象上的属性时会返回 `false`。  

## 将配置数据从代码中分离出来
### 配置数据
- URL
- 需要展示给用户的字符串
- 重复的值
- 设置（每页的配置项）
- 任何可能发生变更的值

### 保存配置数据 <工具：Prop2Js>
配置数据最好放在单独的文件中，以便清晰的分割数据和应用逻辑。  


## 主动抛出自定义错误
使用throw操作符，提供一个`对象`作为错误抛出，任何类型的对象都可以作为错误抛出，最常用的是抛出Error对象：  
```js
// error类型对象如果没有被try - catch，则会在控制台显示错误
throw new Error('something bad happened');
```

### try-catch
能在浏览器处理抛出的错误之前解析它，把可能引发错误的代码放在try块中，把错误处理代码放在catch中：  
```js
try {
    doSomething();
} catch(e) {
    handleError(e);  // 处理错误
} finally {
    /**
     * 这里的代码无论如何是否有错误都会执行，
     */
}
```
不要将`catch块` 留空，总要写点什么来处理错误。  

### 七种错误类型
- Error，所有错误的基本类型，实际上引擎不会抛出给类型的错误
- EvalError，eval函数执行代码时发生错误
- RangeError，一个数字超出它的边界时抛出，如 `new Array(-20)` 试图创建长度为 -20的数组。
- ReferenceError，期望的对象不存在时抛出，比如在null对象上调用函数。
- SyntaxError，给eval()函数传递的代码有语法错误时抛出。
- TypeError，变量不是期望的类型时抛出，如 `new 10` , `'prop' in true`.
- URIError,给encodeURI(),encodeURIComponent()，decode... 等函数传递格式非法的URI字符串时抛出。

### 自定义错误类型
```js
function MyError(message) {
    this.message = message;
}
MyError.prototype = new myError();

throw new MyError("bug comes~");
```

## 不是你的对象不要动
JS独特之处在于没有任何东西是神圣不可侵犯的，可以修改任何可以触及的对象。  
### 原则
把已存在的JS对象当做一个实用工具函数库来对待：  
- 不覆盖方法
- 不新增方法
- 不删除方法

### delete 操作符
delete只能删除实例的属性和方法，不能删除原型上的实例或者方法。  

### 继承的限制
JS中，不能从DOM和BOM对象上继承，同样因为数组索引和length属性的关系，继承Array也是不能工作的。  

### 门面模式 jQuery
为已存在的对象创建一个新的接口，门面有时候称为包装器。  
```js
function DomWrapper(element) {
    this.element = element;
}
DomWrapper.prototype.addClass = function(className) {
    element.className += " " + className;
}

// 用法
var wrapper = new DomWrapper(document.getElementById("my-div"));
wrapper.addClass("selected")
```

### 阻止对象的修改
锁定对象，保证任何人不能有意或者无意的修改他们。有三种锁定的级别：  
```js
// 防止扩展 禁止为对象"添加"属性和方法，但已存在的属性和方法可以修改和删除
Object.preventExtension(obj);
Object.isExtensible(obj);     // false

// 密封，类似防止扩展，而且禁止为对象删除已存在的属性和方法
Object.seal(obj);
Object.isSealed(obj);     // true

// 冻结， 类似密封，而且禁止为对象修改已存在的属性和方法，所有字段只读。
Object.freeze(obj);
Object.isFrozen(obj);     // true
```

## requestAnimationFrame
```js
function setAnimation(callback) {
    if(window.requestAnimationFrame){               // 标准
        return requestAnimationFrame(callback)
    } else if (window.mozRequestAnimationFrame){    // firefox
        return mozRequestAnimationFrame(callback)
    } else if (window.webkitRequestAnimationFrame){    // webkit
        return webkitRequestAnimationFrame(callback)
    } else if (window.oRequestAnimationFrame){    // opera
        return oRequestAnimationFrame(callback)
    } else if (window.msRequestAnimationFrame){    // IR
        return msRequestAnimationFrame(callback)
    } else {
        return setTimeout(callback, 0)
    }
}
```

## 命题
原命题：   如果p, 那么q  
反命题：   如果q, 那么p             
逆命题：   如果!p, 那么!q  
逆反命题： 如果!q, 那么!p  

> 原命题 === 逆反命题,   反命题 === 逆命题

## UglifyJS
第一个基于Node.js的压缩工具，会去除注释和额外空格，替换变量名合并var表达式。  

## gzip压缩
大多数web服务器都支持运行时执行文件压缩，现代浏览器都支持http压缩。并且会发送http头作为请求的一部分，指明压缩类型: ` accept-encoding: gzip, deflate`。  
服务端接收到该请求头时，就明白浏览器支持通过 gzip 或 deflate 压缩的文件，因此发送响应时会把文件压缩，并设置响应头为为: ` content-encoding:gzip`.  

## 注释声明

- TODO     说明代码还未完成，应当包含下一步要做的事情
- HACK     说明代码走了一个捷径，应当包含为何使用hack的原因
- XXX      说明代码有问题，应当尽快修复
- FIXME    说明代码有问题应该尽快修复，重要性低于XXX
- REVIEW   说明代码的任何改动都需要进行评审

```js
// TODO: 希望有更快的方式
doSomething();

/**
 * HACK:以下代码为针对IE做的处理
 /
```

## 测试工具
- Jasmine
- google： JsTestDriver  
- PhantomJS
- QUnit
- Selenium
- Yeti
- YUI Test