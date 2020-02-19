---
description: https://github.com/cpselvis/geektime-webpack-course
---

## 1. 构建配置包的设计

1. 通用性
    1. 业务开发者无需关注构建配置
    2. 同一团队构建脚本
2. 可维护性
    1. 构建配置合理的拆分
    2. README 文档， ChangeLog 文档
3. 质量
    1. 冒烟测试，单元测试，测试覆盖率
    2. 持续集成

冒烟测试(**确保每次测试能生成我们需要的文件，即确保生成的产物能用于发布**)。  

通过多个配置文件，管理不同环境的 webpack 配置:  
1. 基础配置 webpack.base.js
2. 开发环境 webpack.dev.js
3. 生产环境 webpack.prod.js
4. SSR 环境 webpack.ssr.js

> 最后通过 webpack-merge，例如在 webpack.dev.js 中  

```
// webpack.dev.js
const merge = require('webpack-merge');
const baseConfig = require('./webpack.base.js');

const devConfig = {
    
}

module.exports = merge(baseConfig, devConfig);  // 将当前配置和 baseConfig 进行合并
```

抽离成一个 npm 包统一管理:  
1. 规范：Git commit 规范，README, Eslint 规范，Semver 规范
2. 质量：冒烟测试，单元测试，测试覆盖率和 CI

## 2. 功能模块设计

- 基础配置： webpack.base.js
    - 资源解析
        - 解析 es6
        - 解析 react
        - 解析 css
        - 解析 less
        - 解析图片
        - 解析字体
    - 样式增强
        - css3 前缀补齐
        - css px2rem
    - 目录清理
    - 多页面打包
    - 命令行信息显示优化
    - 错误捕获和处理
    - css 抽取成单独的文件
- 开发环境: webpack.dev.js
    - 代码热更新
        - css 热更新
        - js 热更新
    - source map
- 生产环境: webpack.prod.js
    - 代码压缩
    - 文件指纹
    - tree sharking
    - scope hoisting
    - 速度优化: 基础包用 cdn 
    - 体积优化: 代码分割
- ssr: webpack.ssr.js
    - css 解析 ignore
    - output libraryTarget 设置


## 3. 使用 eslint 规范构建脚本

```js
module.exports = {
  "parser": "babel-eslint",
  // eslint-config-airbnb 用于 react，但是对于基础构建包，不一定需要支持 react，因此可以使用以下的基础包
  "extends": "eslint-config-airbnb",
  "extends": "eslint-config-airbnb-base"
}
```

## 4. 冒烟测试

冒烟测试是指对提交测试的软件在进行详细深入的测试功能之前，而进行的预测试。这种预测试的主要目的是保证**基本功能可用**，如 当前例子中，需要确保比如 **能打包出 bundle.js**, **能生成 html 文件**。  

测试文件
```js
// 通过给 webpack 传入配置，进行手动打包
webpack(prodConfig, () => {
    // 校验打包结果
})
```

## 5. 单元测试和测试覆盖率

测试覆盖率: istanbul  

## 6. 持续集成和 Travis CI

核心措施: 代码集成到主干分支之前，**必须通过自动化测试**，只要有一个测试用例失败，就不能集成。  

## 7. 发布构建包到 npm 社区

发布到 npm:  
添加用户: npm adduser  

升级版本:  

1. 升级补丁版本号: npm version patch  (修复bug)
2. 升级小版本号: npm version minor    (添加 feature)
3. 升级大版本号: npm version major    (重大变更，或者 api 不兼容)

***以上命令会自动修改 package.json 中的 version，同时会自动打上 tag 标签***

[git tag相关资料](https://git-scm.com/book/zh/v2/Git-%E5%9F%BA%E7%A1%80-%E6%89%93%E6%A0%87%E7%AD%BE)  

发布版本: npm publish 

## 8. Git Commit 规范 和 ChangeLog 生成

**校验 commit message 相关包**: validate-commit-msg  

**生成 changelog 相关包**: conventional-changelog-cli

## 9. 语义化版本 Semantic Versioning 规范格式

软件的版本通常由三位组成,如 X.Y.Z。  

版本是 **严格递增** 的，16.2.0 -> 16.3.0 -> 16.3.1  

在发行重要版本时，可以先发行 alpha(内部测试), beta(外部小范围测试), rc (release candidate) 发布先行版本(公测)。  

遵循规范的优势:  
1. 避免出现循环依赖
2. 依赖冲突减少

主版本号: 当你做了不兼容的 API 修改  
次版本号: 当你做了向下兼容的功能性新增,新增 feature  
修订版本号: 当你做了向下兼容的问题修正，修复 bug  

格式： 主版本号.次版本号.修订版本号-(一连串以 . 分割的标识符)  **标识符可以由英文，数字和连接号([0-9A-Za-z])组成**  

如: `16.0.0-rc.1`, `16.0.0-beta.1`,`16.0.0-beta.6`  

- alpha 版本: 内部测试版，一般不对外发布，会有很多 bug，只有测试人员使用  
- beta 版本: 也是测试版，这个阶段的版本会一直加入新的功能，在 Alpha 版本之后推出
- rc: Release Candidate 系统平台上就是发行候选版本，RC 版本不会再加入新的功能了，主要着重于除错。  