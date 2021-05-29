## 1. 分析React Router 打包
当前分析版本为 v5.2.0，最新 commit 为：[https://github.com/ReactTraining/react-router/tree/21a62e55c0d6196002bd4ab5b3350514976928cf](https://github.com/ReactTraining/react-router/tree/21a62e55c0d6196002bd4ab5b3350514976928cf)


项目使用 lerna 管理，在根目录上执行 `yarn` 可以安装项目的依赖。继续在根目录执行 `yarn build` 会进入 reaact-router, react-router-dom 等目录进行打包，打包工具是 rollup，会打包出 esm, cjs, umd 等结果。


react-router-dom 的一个目录结构：[https://github.com/ReactTraining/react-router/tree/21a62e55c0d6196002bd4ab5b3350514976928cf/packages/react-router-dom](https://github.com/ReactTraining/react-router/tree/21a62e55c0d6196002bd4ab5b3350514976928cf/packages/react-router-dom)
```javascript
/Users/songjp/fe/mime/react-router/packages/react-router-dom
├── BrowserRouter.js
├── HashRouter.js
├── Link.js
├── MemoryRouter.js
├── NavLink.js
├── Prompt.js
├── README.md
├── Redirect.js
├── Route.js
├── Router.js
├── StaticRouter.js
├── Switch.js
├── es
|  ├── BrowserRouter.js
|  ├── HashRouter.js
|  ├── Link.js
|  ├── MemoryRouter.js
|  ├── NavLink.js
|  ├── Prompt.js
|  ├── Redirect.js
|  ├── Route.js
|  ├── Router.js
|  ├── StaticRouter.js
|  ├── Switch.js
|  ├── generatePath.js
|  ├── matchPath.js
|  ├── warnAboutDeprecatedESMImport.js
|  └── withRouter.js
├── generatePath.js
├── index.js
├── jest.config.js
├── matchPath.js
├── modules
|  ├── BrowserRouter.js
|  ├── HashRouter.js
|  ├── Link.js
|  ├── NavLink.js
|  ├── index.js
|  └── utils
├── package.json
├── rollup.config.js
├── warnAboutDeprecatedCJSRequire.js
└── withRouter.js
```


可能你会有疑惑:

1. 为什么根目录下有 BrowserRouter.js, 同时有 es/BrowserRouter.js 以及 modules/BrowserRouter.js ?
1. 为什么根目录下有 Router.js，有 es/Router.js, 但是没有 modules/Router.js



直接说结论:

1. rollup 的打包都是根据 modules/index.js 进行打包的, 即源码是在 modules 目录里
1. react-router-dom 的源码是在 modules 目录，主要有 BrowserRouter，HashRouter, Link, NavLink 组件，至于 Router 组件，Switch 组件等，实际上是在 [react-router](https://github.com/ReactTraining/react-router/tree/21a62e55c0d6196002bd4ab5b3350514976928cf/packages/react-router) 这个仓库中。
1. 至于根目录下的 BrowserRouter.js 主要是为了兼容 `require("react-router-dom/%s")` 的引用方式，而实际上更应该使用 `require("react-router-dom").%s`
1. es 目录下的 BrowserRouter.js 主要是为了兼容 `import %s from "react-router-dom/es/%s` 这样的引用方式，而实际上，更建议使用 `import { %s } from "react-router-dom"` 的引用方式



即 **关注 modules 目录即可，es 目录下的文件以及根目录下的文件都是为了兼容一些组件引用方式**。
## 2. 分析 BrowserRouter
正常我们使用 react-router-dom 是这样的：
```javascript
import {
  BrowserRouter as Router,
  Switch,
  Route,
  Link
} from "./react-router-dom";

<Router>
  <Link to="/">Home</Link>
  <Link to="/about">About</Link>
  <Link to="/users">Users</Link>
  <Switch>
    <Route path="/about">
      <About />
    </Route>
    <Route path="/users">
      <Users />
    </Route>
    <Route path="/">
      <Home />
    </Route>
  </Switch>
</Router>
```
我们先看 BrowserRouter 组件。
```javascript
// react-router-dom/BrowserRouter.js
import { createBrowserHistory as createHistory } from "history";

class BrowserRouter extends React.Component {
  history = createHistory(this.props);
  render() {
    return <Router history={this.history} children={this.props.children} />;
  }
}
```
BrowserRouter 组件，实际上就是调用 [history](https://github.com/ReactTraining/history/tree/v4.6.2) 中的 [createBrowserHistory](https://github.com/ReactTraining/history/blob/v4.6.2/modules/createBrowserHistory.js) 方法，创建了一个 history 对象。


history 中有一个 createTransitionManager 文件，内部实现了一个[观察者模式](https://github.com/ReactTraining/history/blob/v4.6.2/modules/createTransitionManager.js#L47-L67):
```javascript
// history/createTransitionManager.js
let listeners = []

const appendListener = (fn) => {
  let isActive = true

  const listener = (...args) => {
    if (isActive)
      fn(...args)
  }

  listeners.push(listener)

  return () => {
    isActive = false
    listeners = listeners.filter(item => item !== listener)
  }
}

const notifyListeners = (...args) => {
  listeners.forEach(listener => listener(...args))
}
```
当本身 history 对象触发 popState 事件，或者调用 history.replace 或者 history.push 方法时，会触发 notifyListeners 通知所有的观察者。


而 react-router 中的 [Router 组件](https://github.com/ReactTraining/react-router/blob/v5.2.0/packages/react-router/modules/Router.js)其实**就是观察者**。它内部有个 location 的 state，并且 [订阅了 history 的变化](https://github.com/ReactTraining/react-router/blob/v5.2.0/packages/react-router/modules/Router.js#L31-L38):
```javascript
// react-router/Router.js
if (!props.staticContext) {
  this.unlisten = props.history.listen(location => {
    if (this._isMounted) {
      this.setState({ location });
    } else {
      this._pendingLocation = location;
    }
  });
}
```
当 history 发生变化时，会改变这个 location 的 state。这个 state 又作为 RouterContext.Provider 的值会继续传下去。
```javascript
// react-router/Router.js
<RouterContext.Provider
  value={{
    history: this.props.history,
    location: this.state.location,
    match: Router.computeRootMatch(this.state.location.pathname),
    staticContext: this.props.staticContext
  }}
>
<HistoryContext.Provider
  children={this.props.children || null}
  value={this.props.history}
/>
</RouterContext.Provider>
```
作为真正的 [Route 组件](https://github.com/ReactTraining/react-router/blob/v5.2.0/packages/react-router/modules/Route.js)(渲染组件)，内部判断 RouterContext 传下来的 location 值，和自身的的 Path ( 如 `<Route path="/"` ) 是否匹配，如果匹配则渲染 children。


[_判断是否匹配:_](https://github.com/ReactTraining/react-router/blob/v5.2.0/packages/react-router/modules/Route.js#L38-L42)
```javascript
// react-router/Route.js
const match = this.props.computedMatch
? this.props.computedMatch // <Switch> already computed the match for us
: this.props.path
? matchPath(location.pathname, this.props)
: context.match;
```
[_匹配则渲染 children_](https://github.com/ReactTraining/react-router/blob/v5.2.0/packages/react-router/modules/Route.js#L55-L73)_:_
```javascript
// react-router/Route.js
<RouterContext.Provider value={props}>
  {props.match
    ? children
      ? typeof children === "function"
        ? __DEV__
          ? evalChildrenDev(children, props, this.props.path)
          : children(props)
        : children
      : component
      ? React.createElement(component, props)
      : render
      ? render(props)
      : null
    : typeof children === "function"
    ? __DEV__
      ? evalChildrenDev(children, props, this.props.path)
      : children(props)
    : null}
</RouterContext.Provider>
```
以上代码可以看出，支持多种 children 的写法。分别对应 Route 组件的 component 参数，render 参数和 children 参数的写法。
## 3. 分析 HashRouter
对于 BrowserRouter，其实只是将 createBrowserHistory 换成了 [createHashHistory](https://github.com/ReactTraining/history/blob/v4.6.2/modules/createHashHistory.js), 所以最终逻辑的区别是在 [history](https://github.com/ReactTraining/history/tree/v4.6.2) 这个 npm 包里。
```javascript
// react-router-dom/HashRouter.js
import React from "react";
import { createHashHistory as createHistory } from "history";

class HashRouter extends React.Component {
  history = createHistory(this.props);

  render() {
    return <Router history={this.history} children={this.props.children} />;
  }
}

export default HashRouter;
```


**HashRouter 与 BrowserRouter 的共同点：**

1. 当我们点击 Link 组件进行跳转时，都是调用它们( BrowserRouter 或者 HashRouter ) 内部的 push 方法进行 url 的变更。对于 history 模式是通过 history.pushState, 对于 hash 模式是直接更改 window.location.hash。
1. 在各自的 push 方法中，都会调用内部的 setState 方法，这个方法会将最新的路由信息，通知到各个底层的 Route 组件，各个 Route 组件，通过和最新的路由信息做 match，来判断自己是不是需要进行展示。



**HashRouter 与 BrowserRouter 的不同点:**

1. 当**进行路由的回退/前进**时:
   1. history 模式主要是[监听 popState 事件](https://github.com/ReactTraining/history/blob/v4.6.2/modules/createBrowserHistory.js#L253)，~~来得到最新的路由信息~~ (不对，实际上在触发 popState 的时候，url 已经变化了，此时将 url 信息传递到 Route 就行)，再通过 setState 通知到 Route 组件。
      1. 注意 popState 事件在调用 history.pushState() 和 history.replaceState() 时不会执行，而只会在调用 history.back(), history.go()，history.forward() 时会执行（当点击浏览器回退按钮时，实际上相当于调用 history.back()，因此也会触发 popState 事件）。参考: popstate MDN：[https://developer.mozilla.org/en-US/docs/Web/API/Window/popstate_event](https://developer.mozilla.org/en-US/docs/Web/API/Window/popstate_event)
   2. 对于 hash 模式，主要是[监听 hashChange 事件](https://github.com/ReactTraining/history/blob/v4.6.2/modules/createHashHistory.js#L282)，~~来得到最新的路由信息~~（同上），通过 setState 通知到 Route 组件。
