## 1. 几种变量命名法

- 驼峰式 ( Camel Case )
    - 大驼峰式( Pascal Case ): `GetUserName`
    - 小驼峰式: `getUserName`
- 蛇式 ( Snake Case ): 
    - 小蛇式: `get_user_name`
    - 大蛇式: `GET_USER_NAME`
- 烤肉串式( Kebab Case ): `get-user-name`

## 2. 场景

一些情况下，后端的接口返回值对于前端并不友好，比如后端可能倾向于使用 **小蛇式**, 如:  

```js
{
    user_name: 'xxx',
    email_address: 'xxx'
}
```

而前端比较熟悉的是:  

```js
{
    userName: 'xxx',
    emailAddress: 'xxx'
}
```

因此就需要使用相应的函数进行转换:  

```js
const reduce = require('lodash/reduce')

function createTransform (transformKey) {
  function transformObject (value, depth = -1) {
    if (depth === 0 || value == null || typeof value !== 'object') {
      return value
    }

    if (Array.isArray(value)) {
      return value.map(item => transformObject(item, depth - 1))
    }

    return _reduce(value, (prev, val, key) => {
      // 将 key 进行转换
      prev[transformKey(key)] = transformObject(val, depth - 1)
      return prev
    }, {})
  }
  return transformObject
}
```

### 2.1 转换返回结果

将蛇式转为驼峰式，**将后端返回的结果进行转换**
```js
const camelCase = require('lodash/camelCase')
const deepCamcel = createTransform(camelCase)

deepCamcel({
  a_b: {
    c_d: {
      e_f: 'g_h'
    }
  }
}))

// 结果
{
  aB: {
    cD: {
      eF: 'g_h'  // 注意，只会转换 key，不会转换内容
    }
  }
}
```

其中 depth 参数可以用来控制转换的层数:  

```js
deepCamcel({
  a_b: {
    c_d: {
      e_f: 'g_h'
    }
  }
}, 1))

// 结果如下，即只转换对象的第一层 key
{
  aB: {
    c_d: {
      e_f: 'g_h'
    }
  }
}
```

### 2.2 转换请求参数

同时，在请求时，我们也要将我们参数转换为后端需要的参数格式:  

```js
const snakeCase = require('lodash/snakeCase')
const deepSnake = createTransform(snakeCase)

deepSnake({
  aB: {
    cD: {
      eF: 'gH'
    }
  }
})

// 结果
{
  a_b: {
    c_d: {
      e_f: 'gH'
    }
  }
}
```

## 3. 与 Axios 结合

[Axios 拦截器 文档](https://github.com/axios/axios#interceptors)  

Axios 可以通过设置拦截器，**在请求发出之前转换请求参数，在请求结果真正反应到页面之前转换返回结果**:   

1. 转换请求参数

```js
const snakeCase = require('lodash/snakeCase')
const deepSnake = createTransform(snakeCase)

// 使用拦截器
request.interceptors.request.use(transformKeysToSnakeCase);

function transformKeysToSnakeCase(config) {
  // POST
  if (config.data) {
    config.data = deepSnake(config.data);
  }

  // GET
  if (config.params) {
    config.params = deepSnake(config.params);
  }

  return config;
}
```

2. 转换返回结果

```js
const camelCase = require('lodash/camelCase')
const deepCamcel = createTransform(camelCase)

// 使用拦截器
request.interceptors.response.use(transformKeysToCamelCase);

function transformKeysToCamelCase(response) {
  if (response.data) {
    let caseOptions = null;
    if (response.config) {
      caseOptions = response.config.responseCaseOptions;
    }
    response.data = deepCamcel(response.data);
  }

  return response;
}
```