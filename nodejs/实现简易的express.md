# 简单版本 express 服务器

## 1. 基础
最简单的服务器
```javascript
const http = require('http');
const server = http.createServer(function (request, response) {
    // 在这里处理请求
  
    // 发送 HTTP 头部 
    // HTTP 状态值: 200 : OK
    // 内容类型: text/plain
    response.writeHead(200, {'Content-Type': 'text/plain'});
    // 发送响应数据 "Hello World"
    response.end('Hello World\n');
});

server.listen(8888);

console.log("端口: 8888");
```


<a name="tfB8V"></a>
## 2. 实现


<a name="ys9Nx"></a>
### 2.1 第一版，基础

<br />首先 express 最简单的服务器代码如下:<br />

```javascript
const express = require('express');
const app = express()
const port = 3000

app.get('/', function(req, res) {
  res.end('You send GET request')
})

app.listen(port, () => console.log(`Example app listening at http://localhost:${port}`))
```

<br />即 express 是一个函数，函数执行能生成一个 app 对象，对象上具有 listen，于是我们可以写出以下的代码：<br />
<br />_express/index.js_
```javascript
const http = require("http");

const createServer = () => {
  // 这里取名为 app, 只是为了方便，实际上就是个处理请求的函数
  const app = function (req, res) {
    res.end('Response From Server')
  }

  app.listen = (...args) => {
    const server = http.createServer(app)
    server.listen(...args)
  }
  return app
}

module.exports = createServer;
```


<a name="UPbFq"></a>
### 2.2 第二版，请求方法和路径

<br />在 express 中，能够针对请求的方法和路径，做对应的处理，比如:
```javascript
const express = require('express');
const app = express()
const port = 3000

app.get('/name', function(req, res) {
  res.end('name')
})

app.get('/age', function(req, res) {
  res.end('9')
})

// 统配所有方法和路径
app.all('*', function(req, res) {
  res.end('all response')
})

app.listen(port, () => console.log(`Example app listening at http://localhost:${port}`))
```
此时访问 localhost:3000/name 返回 name，访问 localhost:3000/age 返回 9，如果没有对应的路由，则会匹配到 all 这个兜底路由:<br />
<br />为了实现以上效果，我们需要扩展我们的 express/index.js :
```javascript
const http = require("http");
const url = require('url');
const { METHODS } = http;

// METHODS 为
// [
//   'ACL',         'BIND',       'CHECKOUT',
//   'CONNECT',     'COPY',       'DELETE',
//   'GET',         'HEAD',       'LINK',
//   'LOCK',        'M-SEARCH',   'MERGE',
//   'MKACTIVITY',  'MKCALENDAR', 'MKCOL',
//   'MOVE',        'NOTIFY',     'OPTIONS',
//   'PATCH',       'POST',       'PROPFIND',
//   'PROPPATCH',   'PURGE',      'PUT',
//   'REBIND',      'REPORT',     'SEARCH',
//   'SOURCE',      'SUBSCRIBE',  'TRACE',
//   'UNBIND',      'UNLINK',     'UNLOCK',
//   'UNSUBSCRIBE'
// ]

const createServer = () => {
  // 这里取名为 app, 只是为了方便，方便在函数中使用 this，因为 listen 方法也在 app 身上，实际上就是个处理请求的函数
  const app = function (req, res) {
    // 当请求真正到来的时候，调用先前存在的 handles，一一进行对比参数和方法，如果匹配则执行
    for (let i = 0; i < app.handles.length; i++) {
      const { method, path, handler } = app.handles[i];
      // 取出请求的方法
      const reqMethod = req.method.toLowerCase();
      // 取出请求的路径
      const { pathname } = url.parse(req.url, true)
      // 如果方法,路径匹配，则进行处理， 同时处理 all
      if( (method === reqMethod || method === 'all') && (pathname === path || path === '*')) {
        handler(req, res)
      }
    }
    // 如果前面没有 res.end， 就会执行这个默认返回
    res.end('Default Response')
  }

  app.handles = [];  // 存放 layer: (路由和中间件)

  app.listen = function (...args) {
    const server = http.createServer(app)
    server.listen(...args)
  }

  // 给 app 添加 get, post 等 http 请求方法
  // 每个方法其实给 handles 推送一个 layer
  METHODS.forEach(method => {
    method = method.toLowerCase();
    app[method] = function (path, handler) {
      app.handles.push({
        method,
        path,
        handler
      })
    }
  })

  // 处理 all
  app.all = function (path, handler) {
    app.handles.push({
      method: 'all',
      path,
      handler
    })
  }

  return app
}

module.exports = createServer;
```
<a name="I7Gd5"></a>
### 
<a name="zaLQo"></a>
### 2.3 第三版，中间件


在 express 中，可以使用中间件，中间件可以在执行路由之前，对请求进行处理
```javascript
const express = require('express');
const app = express()
const port = 3000

// 中间件
const middle1 = function (req, res, next) {
  console.log('middle1')
  // 必须使用 next 才会进入下一个中间件
  next()
}

const middle2 = function (req, res, next) {
  console.log('middle2')
  next()
}

// 以下两种方式是一样的，即 use 没有第一个参数，则默认是 /， 即针对某个 url 使用中间件
app.use('/', middle1);  // 等价于 app.use(middle1)
app.use('/', middle2);

app.get('/name', function(req, res) {
  res.end('name')
})

app.get('/age', function(req, res) {
  res.end('9')
})

app.all('*', function(req, res) {
  res.end('all response')
})

app.listen(port, () => console.log(`Example app listening at http://localhost:${port}`))
```
那么继续修改我们的代码:

```javascript
const http = require("http");
const url = require('url');
const { METHODS } = http;

// METHODS 为
// [
//   'ACL',         'BIND',       'CHECKOUT',
//   'CONNECT',     'COPY',       'DELETE',
//   'GET',         'HEAD',       'LINK',
//   'LOCK',        'M-SEARCH',   'MERGE',
//   'MKACTIVITY',  'MKCALENDAR', 'MKCOL',
//   'MOVE',        'NOTIFY',     'OPTIONS',
//   'PATCH',       'POST',       'PROPFIND',
//   'PROPPATCH',   'PURGE',      'PUT',
//   'REBIND',      'REPORT',     'SEARCH',
//   'SOURCE',      'SUBSCRIBE',  'TRACE',
//   'UNBIND',      'UNLINK',     'UNLOCK',
//   'UNSUBSCRIBE'
// ]

const createServer = () => {
  // 这里取名为 app, 只是为了方便，实际上就是个处理请求的函数
  const app = function (req, res) {
    // 取出请求的方法
    const reqMethod = req.method.toLowerCase();
    // 取出请求的路径
    const { pathname } = url.parse(req.url, true)

    // 中间件中的 next 函数，迭代 handles
    let index = 0;
    function next() {
      if(index === app.handles.length) {
        // 如果最终没有匹配到任何路由或中间件，那么返回 not found
        res.end('not found');
        return;
      }
      // 每次调用 next，就取下一个 layer
      const { method, path, handler } = app.handles[index++];
      if (method === 'middleware') {
        // 如果是中间件，且匹配到了
        if(path === '/' || path === pathname || pathname.startsWidth(path + '/')) {
          // 传入 next 函数
          console.log('匹配到了');
          handler(req, res, next);
        } else {
          // 没有匹配到，则调用下一个 layer
          next()
        }
      } else {
        // 如果方法,路径匹配，则进行处理， 同时处理 all
        if( (method === reqMethod || method === 'all') && (pathname === path || path === '*')) {
          handler(req, res)
        } else {
          // 如果没有找到，则调用下一个 layer
          next();
        }
      }
    }

    next();
  }

  app.handles = [];  // 存放 layer: (路由和中间件)

  app.listen = function (...args) {
    const server = http.createServer(app)
    server.listen(...args)
  }

  // 给 app 添加 get, post 等 http 请求方法
  // 每个方法其实给 handles 推送一个 layer
  METHODS.forEach(method => {
    method = method.toLowerCase();
    app[method] = function (path, handler) {
      app.handles.push({
        method,
        path,
        handler
      })
    }
  })

  app.all = function (path, handler) {
    app.handles.push({
      method: 'all',
      path,
      handler
    })
  }

  // use 方法
  app.use = function(path, handler) {
    // 如果不传 path，那么默认 path 为 '/'
    if(typeof handler !== 'function') {
      handler = path
      path = '/'
    }

    app.handles.push({
      method: 'middleware',  // 表示这是一个中间件
      path,
      handler
    })
  }

  return app
}

module.exports = createServer;
```
<br />
<a name="mth3V"></a>
### 2.4 第四版，错误中间件


错误中间件，即当中间件使用**参数**调用 next() 函数时，该参数会被当做是 error，此时后续的中间件和路由都不会执行，而是直接执行错误中间件

```javascript
const express = require('express');
const app = express()
const port = 3000

// 中间件
const middle1 = function (req, res, next) {
  console.log('middle1')
  // 必须使用 next 才会进入下一个中间件
  next('12312')
}

const middle2 = function (req, res, next) {
  console.log('middle2')
  next()
}

// 以下两种方式是一样的，即 use 没有第一个参数，则默认是 /， 即针对某个 url 使用中间件
app.use('/', middle1);  // 等价于 app.use(middle1)
app.use('/', middle2);

app.get('/name', function(req, res) {
  res.end('name')
})

app.get('/age', function(req, res) {
  res.end('9')
})

app.all('*', function(req, res) {
  res.end('all response')
})

// 错误中间件, 约定该中间件参数为 4 个参数
app.use(function(err, req, res, next) {
  console.log('error', err);  // 打印 error 12312
})

app.listen(port, () => console.log(`Example app listening at http://localhost:${port}`))
```
<a name="0ruwq"></a>
## 
那么继续改造我们的代码：<br />

```javascript
const http = require("http");
const url = require('url');
const { METHODS } = http;

// METHODS 为
// [
//   'ACL',         'BIND',       'CHECKOUT',
//   'CONNECT',     'COPY',       'DELETE',
//   'GET',         'HEAD',       'LINK',
//   'LOCK',        'M-SEARCH',   'MERGE',
//   'MKACTIVITY',  'MKCALENDAR', 'MKCOL',
//   'MOVE',        'NOTIFY',     'OPTIONS',
//   'PATCH',       'POST',       'PROPFIND',
//   'PROPPATCH',   'PURGE',      'PUT',
//   'REBIND',      'REPORT',     'SEARCH',
//   'SOURCE',      'SUBSCRIBE',  'TRACE',
//   'UNBIND',      'UNLINK',     'UNLOCK',
//   'UNSUBSCRIBE'
// ]

const createServer = () => {
  // 这里取名为 app, 只是为了方便，实际上就是个处理请求的函数
  const app = function (req, res) {
    // 取出请求的方法
    const reqMethod = req.method.toLowerCase();
    // 取出请求的路径
    const { pathname } = url.parse(req.url, true)

    // 中间件中的 next 函数，迭代 handles
    let index = 0;
    function next(err) {
      if(index === app.handles.length) {
        // 如果最终没有匹配到任何路由或中间件，那么返回 not found
        res.end('not found');
        return;
      }
      // 每次调用 next，就取下一个 layer
      const { method, path, handler } = app.handles[index++];

      if(err) {
        // next 有参数，则认为是错误, 那么跳过中间的中间件和路由，去到错误中间件（函数参数为4）
        if(handler.length === 4) {
          handler(err, req, res, next);
        } else {
          // 如果没有匹配到，则继续抛到下一个中间件或路由
          next(err);
        }
      } else {
        if (method === 'middleware') {
          // 如果是中间件，且匹配到了
          if(path === '/' || path === pathname || pathname.startsWidth(path + '/')) {
            // 传入 next 函数
            console.log('匹配到了');
            handler(req, res, next);
          } else {
            // 没有匹配到，则调用下一个 layer
            next()
          }
        } else {
          // 如果方法,路径匹配，则进行处理， 同时处理 all
          if( (method === reqMethod || method === 'all') && (pathname === path || path === '*')) {
            handler(req, res)
          } else {
            // 如果没有找到，则调用下一个 layer
            next();
          }
        }
      }
    }
    next();
  }

  app.handles = [];  // 存放 layer: (路由和中间件)

  app.listen = function (...args) {
    const server = http.createServer(app)
    server.listen(...args)
  }

  // 给 app 添加 get, post 等 http 请求方法
  // 每个方法其实给 handles 推送一个 layer
  METHODS.forEach(method => {
    method = method.toLowerCase();
    app[method] = function (path, handler) {
      app.handles.push({
        method,
        path,
        handler
      })
    }
  })

  app.all = function (path, handler) {
    app.handles.push({
      method: 'all',
      path,
      handler
    })
  }

  // 中间件
  app.use = function(path, handler) {
    // 如果不传 path，那么默认 path 为 '/'
    if(typeof handler !== 'function') {
      handler = path
      path = '/'
    }

    app.handles.push({
      method: 'middleware',  // 表示这是一个中间件
      path,
      handler
    })
  }

  return app
}

module.exports = createServer;
```


<a name="JWXny"></a>
### 2.5 第五版，处理 req 的参数

<br />express 中，对 req, res 对象进行了封装，可以便捷的得到一些参数：<br />

```javascript
const express = require('express');
const app = express()
const port = 3000

// 中间件
const middle1 = function (req, res, next) {
  console.log(req.path)  // 请求 http://localhost:3000/name 时，打印 /name
  console.log(req.hostname)  // 打印 localhost
  console.log(req.query)  // 打印 {}
  next()
}

const middle2 = function (req, res, next) {
  console.log('middle2')
  next()
}

app.use('/', middle1);
app.use('/', middle2);

app.get('/name', function(req, res) {
  res.end('name')
})

app.get('/age', function(req, res) {
  res.end('9')
})

app.all('*', function(req, res) {
  res.end('all response')
})

// 错误中间件, 约定该中间件参数为 4 个参数
app.use(function(err, req, res, next) {
  console.log('error', err);  // 打印 error 12312
  res.end('error')
})

app.listen(port, () => console.log(`Example app listening at http://localhost:${port}`))
```

<br />这是因为 express 内部有内置的中间件，通过该中间件给 req 设置了属性, 代码如下：
```javascript
const http = require("http");
const url = require('url');
const { METHODS } = http;

const createServer = () => {
  // 这里取名为 app, 只是为了方便，实际上就是个处理请求的函数
  const app = function (req, res) {
    // 取出请求的方法
    const reqMethod = req.method.toLowerCase();
    // 取出请求的路径
    const { pathname } = url.parse(req.url, true)

    // 中间件中的 next 函数，迭代 handles
    let index = 0;
    function next(err) {
      if(index === app.handles.length) {
        // 如果最终没有匹配到任何路由或中间件，那么返回 not found
        res.end('not found');
        return;
      }
      // 每次调用 next，就取下一个 layer
      const { method, path, handler } = app.handles[index++];

      if(err) {
        // next 有参数，则认为是错误, 那么跳过中间的中间件和路由，去到错误中间件（函数参数为4）
        if(handler.length === 4) {
          handler(err, req, res, next);
        } else {
          // 如果没有匹配到，则继续抛到下一个中间件或路由
          next(err);
        }
      } else {
        if (method === 'middleware') {
          // 如果是中间件，且匹配到了
          if(path === '/' || path === pathname || pathname.startsWidth(path + '/')) {
            // 传入 next 函数
            handler(req, res, next);
          } else {
            // 没有匹配到，则调用下一个 layer
            next()
          }
        } else {
          // 如果方法,路径匹配，则进行处理， 同时处理 all
          if( (method === reqMethod || method === 'all') && (pathname === path || path === '*')) {
            handler(req, res)
          } else {
            // 如果没有找到，则调用下一个 layer
            next();
          }
        }
      }
    }
    next();
  }

  app.handles = [];  // 存放 layer: (路由和中间件)

  app.listen = function (...args) {
    const server = http.createServer(app)
    server.listen(...args)
  }

  // 给 app 添加 get, post 等 http 请求方法
  // 每个方法其实给 handles 推送一个 layer
  METHODS.forEach(method => {
    method = method.toLowerCase();
    app[method] = function (path, handler) {
      app.handles.push({
        method,
        path,
        handler
      })
    }
  })

  app.all = function (path, handler) {
    app.handles.push({
      method: 'all',
      path,
      handler
    })
  }

  // 中间件
  app.use = function(path, handler) {
    // 如果不传 path，那么默认 path 为 '/'
    if(typeof handler !== 'function') {
      handler = path
      path = '/'
    }

    app.handles.push({
      method: 'middleware',  // 表示这是一个中间件
      path,
      handler
    })
  }

  // 内置中间件，设置 req
  app.use(function(req, res, next) {
    let { pathname, query } = url.parse(req.url, true);
    let hostname = req.headers.host.split(":")[0];

    req.hostname = hostname;
    req.path = pathname;
    req.query = query;
    next();
  })

  return app
}

module.exports = createServer;
```
<a name="liMs8"></a>
## 
<a name="2tjzx"></a>
## 3. TODO


1. 路径参数 /user/:id
1. 子路由
1. res 的封装
1. 模板的渲染
<a name="SEvPf"></a>
## 参考资料

1. 视频：[从零实现简易express框架](https://www.bilibili.com/video/BV1sJ411N7Kn/?spm_id_from=333.788.videocard.0)
1. [渐进式Express源码学习](https://github.com/sunkuo/grow-to-express) 
