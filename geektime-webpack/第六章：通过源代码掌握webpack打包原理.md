---
description: https://github.com/cpselvis/geektime-webpack-course
---

略...  

## 简单记录

[yargs 处理命令行参数](https://github.com/yargs/yargs)  

[tapable](https://github.com/webpack/tapable) 类似 [EventEmitter](https://nodejs.org/api/events.html#events_class_eventemitter) 做事件处理

**将 ES6 代码转为 ES5 代码：**  
通过 babylon 将 ES6 代码生成 AST. 在通过 babel-core 将 AST 重新生成 ES5 代码。  

**分析模块之间的依赖关系:**  
通过 babel-traverse 的 ImportDeclaration 方法获取依赖属性。  