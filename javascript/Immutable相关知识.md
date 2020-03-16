## 1. JS 与 Immutable 的对应关系

- JS.Array => Immutable.List
- JS.Map => Immutable.Map
- JS.Set => Immutable.Set

## 2. 比较 immutable 值

应该使用 is 或 equals 来比较 immutable 的值

```js
import { is, Map } from 'immutable'

const a = Map({ a: 1, b: 2 })
const b = Map({ a: 1, b: 2 })
console.log(a.equals(b))     // true
console.log(is(a, b))        // true
console.log(a === b)         // false
```

如果一个操作，返回了一个没有变化的对象，那么可以使用 === 来进行比较:  

```js
import { Map } from 'immutable'

const a = Map({ a: 1, b: 2 })
const b = a.set('b', 2)         // 实际上并没有改变 a
console.log(a === b)            // true
console.log(a.equals(b))        // true
console.log(is(a, b))           // true
```

## 3. Immutable List 具有和 JS Array 一样的 API

```js
import { is, List } from 'immutable'

const list1 = List([ 1, 2 ])         // List [1,2]
const list2 = list1.push(3, 4, 5)    // List [1,2,3,4,5]

console.log(list1 === list2)         // false，都是返回新的 List
console.log(list1.equals(list2))     // false
console.log(is(list1, list2))        // false

const list3 = list2.unshift(0)       // List [0,1,2,3,4,5]
list3.forEach(v => {
  console.log(v)                     // 0,1,2,3,4,5
})
```

## 4. 原生的 JS 对象或数组作为 Immutable 的函数参数

在任何能接受 Collection 的地方，都能传入原生的 JS 对象或数组作为参数

```js
import { Map } from 'immutable'

const map1 = Map({ a: 1 })
const map2 = Map({ b: 2 })
const obj = { c: 2 }
console.log(map1.merge(map2, obj)) // Map {a:1,b:2,c:3}

const list1 = List([1])
const list2 = List([2])
const arr = [3]
console.log(list1.concat(list2, arr)) // List [1,2,3]
```

## 5. 原生 JS 对象与 Immutable 对象互相转换

**fromJS**: 将原生 JS 对象和数组转成 Immutable:  

```js
import { Map, List, fromJS } from 'immutable'

// 转换对象
const a = { 1: 'one' }
console.log(a['1'], a[1])    // 'one', 'one'
const map = fromJS(a)        // Map {'1' : 'one'}

// 当将原生 js 对象转为 immutable 时，所有的 key 都会转为字符串类型。
// 因此 map.get(1) 返回 undefined
console.log(map.get('1'), map.get(1)) // 'one', undefined


// 转换数组
const b = [1]
const list = fromJS(b)
console.log(list) // List [1]
```

将 immutable 转换为原生 JS 对象分为浅转换和深转换:  

- 浅转换: toArray(), toObject(), 即只转换一层
- 深转换: toJS()

```js

// 1. immutable 转对象
const map = Map({ a: 1, b: Map({ c: 2 }) })
console.log(map.toObject())   // { a: 1, b: Map {c: 2} }  其中 b 还是 Map 类型
console.log(map.toJS())       // { a: 1, b: { c: 2 } }  完全转化为 JS 对象类型

// 同时 immutable 对象都实现了 toJSON 方法，当调用 JSON.stringify 时会调用
console.log(JSON.stringify(map)) 


// 2. immutable 转数组
const arr = List([1, List([2])])
console.log(arr.toArray())   // [1, List[2]]  其中第二个元素还是 List 类型
console.log(arr.toJS())      // [1,[2]]  完全转化为 JS 数组类型

// 同时 immutable 对象都实现了 toJSON 方法，当调用 JSON.stringify 时会调用
console.log(JSON.stringify(arr)) 
```

## 6. 嵌套结构的读取和操作

```js
import { fromJS } from 'immutable'

const map1 = fromJS({ a: { b: { c: [3, 4, 5] } } })


// 1. 合并
const map2 = map1.mergeDeep({ a: { b: { d: 6 } } })   
console.log(map2.toJS())                     // { a: { b: { c: [3, 4, 5] }, d: 6 } }

// 2. 根据路径读取
console.log(map2.getIn(['a', 'b', 'd']))     // 6

// 3. 更新属性值
const map3 = map2.updateIn([ 'a', 'b', 'd' ], value => value + 1)
console.log(map3.getIn(['a', 'b', 'd']))     // 7

// 4. 更新 List
const map4 = map2.updateIn([ 'a', 'b', 'c' ], list => list.push(6))
console.log(map4.getIn(['a', 'b', 'c']))     // List [3,4,5,6]
```

## 7. 自返回的优化和 withMutations 优化

自返回的优化: 
```js
const map1 = Map({ a: 1, b: 2 })
const map2 = map1.set('b', 2)
console.log(map1 === map2)    // true, 当生成的值不变时，immutable 会返回原来的对象


const map3 = map1.set('b', 3)
const map4 = map1.set('b', 3)
console.log(map3 === map4)    // false, 当生成的值变化时，immutable 每次都会返回新的对象，新对象之间是相互独立的
console.log(map3.equals(map4)) // true, 但是使用 is 或者 equals 检测是相等的
```
可以使用 withMutations 来合并一些变化, 因为【没有临时的中间对象生成】，因此能够优化性能

```js
const list1 = List([ 1, 2, 3 ])
const list2 = list1.withMutations(function (list) {
  list.push(4).push(5).push(6)
})
console.log(list2)  // List [1, 2, 3, 4, 5, 6]
```
## 8. 性能优化举例

```js
import { updateIn, Map } from 'immutable'

const Child = React.memo((props) => {
  console.log('render child')  // 每次点击都会打印
  return <div>{JSON.stringify(props)}</div>
})

export default class App extends Component {
  constructor (props) {
    super(props)
    this.state = {
      store: {
        a: {
          b: 1
        }
      }
    }
  }
  handleClick = () => {
    this.setState({ store: { a: { b: 1 } } })
  }
  render = () => {
    return (
      <div className='title' onClick={this.handleClick}>
        Hello, express-react-dev-template
        <Child store={this.state.store} />
      </div>
    )
  }
}
```

在平常写组件的时候，我们可能会有以上的代码，尽管实际上 `store` 的值没有发生变化，但是还是每次都是打印 **render child**  

React.memo 没有生效，当然这种场景下我们可以通过**自定义 React.memo 的比较函数**，来保证这种情况下不渲染。  

**但是当 props 数据结构比较复杂时，自定义函数也会很麻烦**  

而使用 immutable 就没有这个问题:  

```js
const Child = React.memo((props) => {
  console.log('render h1')    // 只会打印一次
  return <div>{JSON.stringify(props)}</div>
})

export default class App extends Component {
  constructor (props) {
    super(props)
    this.state = {
      store: Map({ a: { b: 1 } })
    }
  }
  handleClick = () => {
    this.setState({ store: updateIn(this.state.store, ['a', 'b'], value => value) })
  }
  render = () => {
    return (
      <div className='title' onClick={this.handleClick}>
            Hello, express-react-dev-template
        <Child store={this.state.store} />
      </div>
    )
  }
}
```

通过定义 immutable 类型的数据作为 state 的值，React.memo 就会认为两次的 props 相等，从而避免无谓的渲染，

## 9. List

### 9.1 构造函数

**List 是函数而不是类，因此不需要使用 new 关键字**  

List 接受实现了 Iterable 接口的值(如数组，Set 等)作为参数，返回 List 结构。

```js
const a = List()            // 空数组
console.log(a)

const b = List([1, 2, 3])   // 接受数组作为参数
console.log(b)

const c = List(new Set([1, 2, 3]))   // 接受 set 作为参数

console.log(c.equals(b))     // true，equals 会进行值的比较，而不会严格要求引用相等
```

### 9.2 静态方法

```js
// List.isList: 判断是否是 List
List.isList([])              // false
List.isList(List([]))        // true

// List.of: 根据参数，生成 List 结构
List.isList(List.of(1, 2)) // true
```

### 9.3 size

```js
// 返回元素数量
List.of(1, 2).size        // 2
```

### 9.4 改变数据的方法

```js
// set(index, value),  给 index 位置设置 value 值
const a = List([1, 2])
console.log(a.set(2, 3))              // [1, 2, 3]

// delete(index)  删除 index 所在的元素
// 别名: alias
const a = List([1, 2, 3])
console.log(a.delete(2))              // [1, 2]

// insert(index, value) 在 index 插入值 value
const a = List([1, 2])
console.log(a.insert(2, 3))           // [1，2，3]

// clear()     清空 list
const a = List([1, 2])
console.log(a.clear())           // []

// push(value)   添加元素
const a = List([1, 2])
console.log(a.push(3))           // [1,2,3]

// pop(), shift(), unshift(value)  略

// update(index, (value) => value)  更新该值
const a = List([1, 2])
console.log(a.update(0, value => value + 1)) // [2,2]
```

### 9.5 深度改变数据的方法

```js
// setIn(keyPath: [], value ) 根据路径设置值
const a = List([1, 2, List([3, 4])])
// 相当于 a[2][0] = 4
console.log(a.setIn([2, 0], 4)) // [1,2, List[4,4]]

// deleteIn(keyPath: [] )  删除指定 keypath 的值
const a = List([1, 2, List([3, 4])])
console.log(a.deleteIn([2, 0])) // [1,2, List[ undefined, 4]]

// updateIn(), mergeIn(). mergeDeepIn() 略，查看 Map
```

## 10. Map

和 JS 中的 Map 类似，可以把 **对象** 作为 key。  

### 10.1 构造函数

```js
const a = Map()  // 空 Map

const a = Map({ key: 'value' })  // Map {key: 'value' }
// 或者
const a = Map([ ['key', 'value'] ])  // Map { 'key': 'value' }
```

### 10.2 静态方法

```
// Map.isMap(value)  // 判断 value 是否为 Map 类型
const a = Map([ ['key', 'value'] ])
console.log(Map.isMap(a))          // true
console.log(Map.isMap('1'))        // false
```

### 10.3 size

```js
const a = Map([ ['key', 'value'] ])
console.log(a.size)          // 1
```

### 10.4 改变数据的方法

```js
// set(key, value)  设置属性和值
const a = Map()
a.set('key', 'value')    // Map { key : value }

// delete(key)  删除某个属性
const a = Map({'key': 'value'})
a.delete('key')         // Map {}

// deleteAll(keys: string[])  删除一批属性
const a = Map({'key': 'value', 'foo': 'bar'})
a.deleteAll(['key', 'foo'])         // Map {}

// clear()  清空 Map
const a = Map({'key': 'value', 'foo': 'bar'})
a.clear()         // Map {}

// update(key, (value) => value)
const a = Map({'key': 'value'})
a.update('key', value => value + value)   // Map {'key': 'valuevalue'}
// 设置数组
const a = Map({'key': List([])})
a.update('key', list => list.push(1))   // Map {'key': [1]}

// merge, 合并对象
const map1 = Map({ a: 1, b: 2 })
const map2 = Map({ a: 2, c: 3 })
map1.merge(map2)         // {a:2,b:2,c3}
map2.merge(map1)         // {a:1,b:2,c3}

// mergeWith()， 和 merge 类似，可以提供一个函数，用于处理当map1 和 map2 相同 key 的情况
// mergeDeep(), mergeDeepWith()  略
```

### 10.5 深度改变数据的方法

```js
// setIn(keyPath, value) 根据 path 设置值
const map = Map({ a: { b: { c: { d: 1 } } } })
console.log(map.setIn(['a', 'b'], 1))  // Map {a: {b: 1}}

// deleteIn(keyPath)  根据 path 删除值
const map = Map({ a: { b: { c: { d: 1 } } } })
console.log(map.deleteIn(['a', 'b'], 1))  // Map {a:{}}

// updateIn(keyPath,(value) => value)  根据 keyPath 更新值
const map = Map({ a: { b: { c: { d: 1 } } } })
console.log(map.updateIn(['a', 'b'], value => 2))  // Map {a: {b: 2}}
```

## 11. 别的类库

在日常开发中，我们还可以使用诸如 [immutability-helper](https://github.com/kolodny/immutability-helper) 这样的简单一些的库。

比如刚刚那个性能优化的例子，如果使用 immutability-helper 那就是这样:  

```js
import update from 'immutability-helper'

const Child = React.memo((props) => {
  console.log('render h1')   // 同样只会触发一次
  return <div>{JSON.stringify(props)}</div>
})

export default class App extends Component {
  constructor (props) {
    super(props)
    this.state = {
      store: {
        a: {
          b: 1
        }
      }
    }
  }
  handleClick = () => {
    console.log('click')
    this.setState({ store: update(this.state.store, { a: { b: { $set: 1 } } }) })
  }
  render = () => {
    return (
      <div className='title' onClick={this.handleClick}>
        Hello, express-react-dev-template
        <Child store={this.state.store} />
      </div>
    )
  }
}
```


## 参考资料

1. [Immutable 官网](https://immutable-js.github.io/immutable-js/)
2. [API 文档](https://immutable-js.github.io/immutable-js/docs/#/List)
3. [immutability-helper 官网](https://github.com/kolodny/immutability-helper)