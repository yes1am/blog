## 前言

一直以来，登录都是 web 系统很基础却很重要的功能，但是其原理，却很少进行梳理。  

在做入职测试的时候，再一次遇到了登录的问题，碍于时间紧张，只是简单的进行了实现，而过程中遇到的一些疑惑，自己也没有进行解答。  

有空之余便查阅了一些相关文档，结合自己的理解，整理出本文，便于自己翻阅参考, 同时如果之后工作中有更多的领悟，也会更新到本文中。

## 1. 简单实用的登录

1. 前端输入账号密码，发送请求到服务端
2. 服务端端查询数据库，账号密码匹配则登录成功，服务器端生成并设置 Session `Session[SessionID] = 用户信息`,同时将 SessionID 通过 `set-cookie: SessionID` 头设置到客户端的 cookie 中，这样后续客户端发起的请求，都会带上这个cookie。Session 对象存在于服务端内存中，当服务重启时，Session 失效。
3. 当后续请求携带 Cookie 时，服务端验证该 SessionID 是否有效(是否存在且没有过期)，如果有效则从 Session 中读取先前保存的 用户信息，以该身份进行之后的请求处理。

流程图如下：  
![简单实用的登录](http://insights.thoughtworkers.org/wp-content/uploads/2016/12/3-traditional-cookie.jpg)

**缺点：**  
1. 前面也提到了，Session 对象存在服务器内存中，如果服务器出现异常导致重启，那么用户就可能突然登出。解决的办法是：**引入新的存储系统如 redis 等来存储会话信息**  

**问题：**  

*那如果 redis 也出错了怎么办，是不是 redis 要进行灾备，保证两个以上 redis 存在，通过增加数量保证一定有一个服务是好的*  

**注意点：**  

服务端 Session 和 客户端 Cookie 都可以设置过期时间


## 2. 传统 Web 应用中身份验证的最佳实践

由于第一种方案需要在服务端保存 Session，那么就会遇到需要 Session 共享的问题，比如负载均衡中，有 A,B,C,D 四个服务器，用户第一次在 A 服务器登录，Session 存在于 A 服务器中，下次如果用户访问到 B 服务器，那么就会被认为是未登录的。  

**解决办法是**:  将登录信息之后的 Session 放在第三方的存储服务中，A.B.C.D 服务器都去访问第三方存储服务校验是否登录。  


**最佳实践**: 参考 [JSON Web Token ](https://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html)  

其实我们可以不把登录信息放在 Session 中，而是**直接加密返回给客户端**，(通过使用自包含的，含有加密内容的 Cookie 作为替代凭据)。  

1. 客户端使用账号密码登录，服务端验证账号密码匹配，则使用**加密算法**，加密**用户信息**得到一个字符串，并 `set-cookie=该加密字符串`
2. 客户端得到该 cookie 之后，以后每次请求带上 cookie，服务端使用**解密算法**解析出用户的身份信息，进行下一步的请求处理
3. 由于校验的过程是通过加解密算法的配合，只要算法一致，不同的服务器也能得到同样的结果，因此不存在考虑**如何共享登录态**的问题。

## 3. 单点登录

用户在一个站点登录之后，不需要在其它兄弟站点中再次登录。比如，假设 **在支付宝登录之后，再进入淘宝或者阿里云，不需要再次登录**。

流程图如下：  

![](https://gss3.bdstatic.com/7Po3dSag_xI4khGkpoWK1HF6hhy/baike/c0%3Dbaike80%2C5%2C5%2C80%2C26/sign=58240a0eba8f8c54f7decd7d5b404690/b219ebc4b74543a97553362c10178a82b8011482.jpg)  

1. 客户端请求应用系统 1，没有登录，于是让客户端会访问独立的鉴权站点： IDB 身份验证系统
2. 客户端在 IDB 身份验证系统登录，且 IDB 身份验证系统返回 ticket 给客户端
3. 之后，客户端访问应用系统 3 的时候，会携带 ticket，而应用系统 3 则拿着 ticket 去 IDB 身份验证系统进行校验。由于客户端已经在 IDB 系统中登录，因此该请求会被允许。从而进行接下来的请求处理流程。  


由于没有实践过，单点登录的知识简单整理至此，有必要可以查找相关资料深入学习。回忆下之前上家公司内网系统的 SSO 运用，第一次登录 A 系统会跳转到一个独立的页面进行登录，之后再进入 B 系统，不需要再次输入账号密码，原理估计就是如此吧。  

## 4. 登录实践

在真实项目中，除了登录成功，前端获得 cookie ，再后续请求携带 cookie 之外。还会涉及到路由跳转拦截(比如想直接到一个需要登录的页面应该被拒绝)，或者**已经登录之后**想主动回到登录页也应该是被拒绝，除非主动退出登录删除 cookie。  

本章用 Vue-Router + Vuex 举例：  

首先，对于我们开发者来说，我们是知道哪些页面需要登录态才能访问，而哪些页面不需要登录也能访问。因此我们可以在路由中添加字段以作区分:  

[Vue-router 官方文档示例](https://router.vuejs.org/zh/guide/advanced/meta.html)  

比如我们定义如下几个路由:  

```js
[
    {
      //登录页面
      path: '/login',
      component: login,
      meta: {
        requiresAuth: false
      }
    },
    {
      // 下载页面
      path: '/download',
      component: download,
      meta: {
        requiresAuth: false
      }
    },
    {
      // faq 页面
      path: '/faq',
      component: faq,
      meta: {
        requiresAuth: false
      }
    },
    {
      // 列表页
      path: '/list',
      component: list,
      meta: {
        requiresAuth: true
      }
    },
    {
      // 详情页
      path: '/detail',
      component: detail,
      meta: {
        requiresAuth: true
      }
    },
],
```
如上，有登录页，下载页和 FAQ 页面不需要登录态即可访问，而详情页和列表页则需要登录态。  

### 4.1 客户端如何保存登录态信息

客户端通过账号密码登录成功之后，服务端有可能直接`set-cookie=token` 设置 cookie 到了前端。也可能是将 token 信息作为返回体进行返回。  

无论哪一种方式，针对 vuex 这种方案，我们可以将 token 信息存入到 vuex 的 state 中，通过内存访问 state 比每次通过 API 访问 cookie 或者 locaStorage 应该是更为高效吧(我猜的)。  

**为了实现关闭浏览器或者刷新浏览器之后，网站不需要再次登录**，我们可以使用 cookie 或者 localStorage 来保存 token 信息，因为 vuex 是 基于内存的，刷新或者关闭浏览器都会失效。  

```js
伪代码

fetch('/login')
.then(res => {
    // 如果账号密码正确
    // 设置 vuex 的 token
    vuex.token = token
    // 设置 cookie 或 localStorage
    storage.token = token
})
```

这样我们就保存了登录态信息。

### 4.2 保存登录态信息之后还要做什么

在客户端保存登录态信息之后，在每一次**进行页面跳转(或者请求接口之前)**，都可以进行登录态的校验(**只是单纯校验该 token 是否有效，而无法真正校验 token 是否有效，真正的校验还是需要后端**)。  

```js
伪代码

// 检验客户端是否存在登录态信息
function checkLogged() {
  const token = vuex.token;
  if (!token) {
    // 如果 vuex 中没有，则从 storage 中获取
    // 这个场景是在，第一次打开页面，或者刷新页面时，这时候 vuex 还没有 token 信息
    if (!getLocalToken()) {
      // vuex 没有, storage 没有，那么就是没有登录
      return false;
    }
    
    // 如果 vuex 没有, 但是 storage 里有
    // 那用户是登录的状态，同时为了以后可以直接校验 vuex 里的 token
    // 我们需要把 storage 中的信息设置回 vuex 里
    vuex.token = token
    return true;
  }
  return true;
}


// 导航守卫， 在每一次要跳转页面时，都会进入这段校验  
router.beforeEach((to, from, next) => {
    // 针对 from(从哪来)，to(到哪去)， checkLogged(当前是否登录) 等信息来得到结果
    // 决定是否进行跳转，或者应该跳转到什么页面
    
    // 要去的页面是否需要登录
    const isToPageNeedLogin = to.matched.some(record => record.meta.requiresAuth)
    
    // 当前是否有登录态
    const isLogged = checkLogged()
});
```
如上所说，我们可以拿到 from, to, checkLogged 等信息，来进行判断，那我们来进行穷举吧。  

### 4.3 页面之间跳转的 18 种情况

先规定几种标识：  

**from:**  
f1 => 当前页面需要登录(如 /list, /detail)  
f2 => 当前页面不需要登录，但也不是登录页(如 /download, /faq): **之后称为普通页**  
f3 => 当前页面不需要登录，且是登录页

**to:**  
t1 => 想去的页面需要登录(如 /list, /detail)  
t2 => 想去的页面不需要登录，但也不是登录页(如 /download, /faq)：**之后称为普通页**  
t3 => 想去的页面不需要登录，且是登录页

**checkLogged():**  
token0 => 用户已登录  
token1 => 用户未登录  


| 情况 | checkLogged | from | to | 描述 | 结果 |
|---|---|---|---|---|---|
| 1| toekn0 | f1 | t1 | 已登录用户，从需要登录页面，跳转到需要登录的页面 | ***next()*** <br>不做处理，放行该跳转 |
| 2 | token0 | f1 | t2 | 已登录用户，从需要登录的页面，跳转到普通页面 | ***next()***<br>即不做处理，放行该跳转 |
| 3| token0 | f1 | t3 | 已登录用户，从需要登录的页面，跳转到登录页 | ***next(特定页面)***<br>不允许跳转，提醒“跳转失败，需要先退出登录” |
| 4 | token0 | f2 | t1 | 已登录用户，从普通页面，跳转到需要登录的页面 | ***next()***<br> 即不作处理，放行该跳转|
| 5 | token0 | f2 | t2 | 已登录用户，从普通页面，跳转普通页面 | ***next()***<br> 即不作处理，放行该跳转|
| 6 | token0 | f2 | t3 | 已登录用户，从普通页面，跳转登录页面 | ***next(特定页面)***<br>不允许跳转，提醒“跳转失败，需要先退出登录”|
| 7 | token0 | f3 | t1 | 已登录用户，从登录页面，跳转到需要登录的页面 | ***不考虑这种情况***<br>一个已登录用户，不可能在登录页|
| 8 | token0 | f3 | t2 | 已登录用户，从登录页面，跳转到普通页面 | ***不考虑这种情况***<br>一个已登录用户，不可能在登录页|
| 9 | token0 | f3 | t3 | 已登录用户，从登录页面，跳转登录页面 | ***不考虑这种情况***<br>一个已登录用户，不可能在登录页|
| 10 | toekn1 | f1 | t1 | 未登录用户，从需要登录页面，跳转到需要登录的页面 | ***next(登录页)*** <br>一开始用户登录了，出现在已登录页面 <br>后来由于登陆态过期，变成未登录用户 <br> 此时跳转到需要登录的页面，则需要先跳转到登录页 |
| 11 | token1 | f1 | t2 | 未登录用户，从需要登录的页面，跳转到普通页面 | ***next()***<br>即不做处理，放行该跳转，因为普通页面不需要登录态 |
| 12 | token1 | f1 | t3 | 未登录用户，从需要登录的页面，跳转到登录页 | ***next()***<br> 放行该跳转 |
| 13 | token1 | f2 | t1 | 未登录用户，从普通页面，跳转到需要登录的页面 | ***next(登录页)***<br> 需要先跳转到登录页面 |
| 14 | token1 | f2 | t2 | 未登录用户，从普通页面，跳转普通页面 | ***next()***<br> 直接放行|
| 15 | token1 | f2 | t3 | 未登录用户，从普通页面，跳转登录页面 | ***next()***<br>放行”|
| 16 | token1 | f3 | t1 | 未登录用户，从登录页面，跳转到需要登录的页面 | ***next(login)***<br>需要先跳转到登录页面|
| 17 | token1 | f3 | t2 | 未登录用户，从登录页面，跳转到普通页面 | ***next()***<br>放行|
| 18 | token1 | f3 | t3 | 未登录用户，从登录页面，跳转登录页面 | ***next()***<br>放行|

由以上信息可知，虽然一共有18种情况，但是实际可以总结为**四种情况** (因为有些信息，比如用户当前页面的信息，其实是没有用到的):  

1. 用户已登录
    1. 用户想跳转去登录页 =>  提醒跳转失败，需要先退出登录才能到登录页面，并**跳转到指定页面或不进行跳转**。
    2. 否则， 直接放行
2. 用户未登录
    1. 用户想跳转到需要登录的页面 => 跳转到提醒用户先要进行登录，并跳转到 **登录页**
    2. 否则，直接放行

**导航守卫**  
```js
router.beforeEach((to, from, next) => {
    // 用户是否已登录
    const isLogged = checkLogged();
    
    // 要去的页面是 需要登录页
    const isToPageNeedLogin = to.matched.some(record => record.meta.requiresAuth)
    // 要去的页面是 登录页
    const isToLoginPage = to.path === '/login';

    if(isLogged) {
        if(isToLoginPage) {
            // 提醒用户跳转失败，要想回到登录页需要先退出登录
            // 同时可以跳转到 === 指定页面=== ，如首页
            // 如果只是保持页面不跳转，会有点奇怪，用户没有操作反馈)
            next(指定页面)
        } else {
            next();
        }
    } else {
        if(isToPageNeedLogin) {
            // 提醒用户需要先进行登录
            next('login')
        } else {
            next();
        }
    }
});
```

## 5. 某公司内部某系统的登录流程梳理
权限系统：  
用户，角色，权限(API接口)三者之间互相多对多的关系。  


### 5.1 登录

1. 前端输入账号密码进行登录
2. node 端得到账号密码之后，用账号密码请求 `sso` 单点登录接口，返回 `sso` 相关信息(包含用户唯一标识)之后，`md5` 加密该信息得到 `token`
3. 通过用户唯一标识，查询数据库，得到**该用户的详细信息，以及该用户的权限列表**。
4. `redis` 缓存该用户 `token`，值为**用户详细信息及权限列表**。设置一定的**有效期**。并返回到前端，(如果有需要也可以将不敏感的用户信息进行返回，一般系统都有个人信息的展示)。
5. 前端将 `token` 保存到 `localStorage` 中，封装 `ajax` 方法，每次请求都会带上 `token`
6. 之后每次请求，后端将前端带入的 `token` 拿来校验，判断是否有效。有效则取出 `token` 中的用户信息和权限列表。再判断用户是否拥有当前请求的权限，有则继续下一步，没有则返回。

### 5.2 用户退出登录

前端发送退出登录请求，后端执行 `redis.remove(token)` 清除 `token` 信息，同时前端清除 `localStorage` 中的 `token`，跳转到登录页。  

这样，请求后端接口时，由于没有 `token`，后端不会返回数据。  
而前端路由方面，可以通过全局的导航守卫设置，`localStorage` 没有 `token` 信息，就会跳转到登录页。  

### 5.3 系统管理员强制他人退出登录

系统管理员可以查看到所有用户，可以清楚别人登录的 `token`。

原理是，用户在登录的时候，`token`( `md5` 加密的用户信息和时间信息) 除了设置进 `redis`， 还会更新到数据库里(如 `user` 表的 `access_token` 字段)。  

那系统管理员可以知道所有用户的 `id`，通过 `id` 查询到用户的 `access_token` 值，再执行 `redis.remove( access_token )` 进行用户 `token` 的删除。  

用户那边下一次请求时，后端校验 `token` 失败，则前端进行 `localStorage token` 的删除以及跳转到登录页。

## 6. 登录与安全

刚刚听了部门安全的分享，发现在记录这部分知识的时候忽略了安全，如果完全依赖于 cookie 判断登录, 那么就需要防范 CSRF 的攻击。  

依赖 localStorage, 也可能被 xss 攻击，暴露 localStorage。  

这样，登录问题又变得复杂了起来。

## 参考资料
1. ThoughtWorks: [登录工程：传统 Web 应用中的身份验证技术](http://insights.thoughtworkers.org/traditional-web-app-authentication/)
2. 阮一峰: [JSON Web Token 入门教程](https://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html)
3. [Vue登录注册，并保持登录状态](https://segmentfault.com/a/1190000016040068)