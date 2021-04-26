## 1. 如何调试
[father](https://github.com/umijs/father) 是个由 [lerna](https://github.com/lerna/lerna) 管理的库，组件打包功能主要是 packages/[father-build](https://github.com/umijs/father/tree/master/packages/father-build) 实现的。安装完依赖后，执行 `yarn build` 后会将 father-build/src 目录下的 ts 代码编译成 lib 目录下的 js 代码，最终执行的是 lib 目录下的 js 代码。


`yarn build` 是由[ umi-tools](https://github.com/umijs/umi-tools/blob/master/src/build.js) 来编译的，里面用 babel 来实现的将 ts 代码 变成 js 代码，如果有需要可以在调整 umi-tools 内部的 babel 配置(比如把一些 plugin 注释掉，从而避免代码过度编译，让我们更方便的将编译后的代码与原来的 ts 代码进行对应) ，再重新执行 `yarn build` 。后面再用 VSCode 调试 Node 的那一套方法就可以进行断点调试了。
## 2. 简单分析源码
在 [father-build.js](https://github.com/umijs/father/blob/8ad068379263cdf3bf91c41644672a2e159c5ce2/packages/father-build/bin/father-build.js) 中，支持通过命令行传递参数，比如 `node ./bin/father-build.js --esm --cjs --umd --file bar ./demo/foo/index.js` ，然后通过 [yargs-parser](https://www.npmjs.com/package/yargs-parser) 进行解析，得到结果为: 
```json
{
  _: ['./demo/foo/index.js'],
  cjs: true,
  esm: true,
  file: 'bar',
  umd: true
}
```
然后进入到 [build.ts](https://github.com/umijs/father/blob/8ad068379263cdf3bf91c41644672a2e159c5ce2/packages/father-build/src/build.ts#L237-L241) 中判断你是否使用了 lerna，来决定是否根据 lerna 来调整打包逻辑。核心是 [build](https://github.com/umijs/father/blob/8ad068379263cdf3bf91c41644672a2e159c5ce2/packages/father-build/src/build.ts#L97) 方法。


文档中提到 **配置文件支持 es6 和 TypeScript**，实际上是因为[这段代码](https://github.com/umijs/father/blob/8ad068379263cdf3bf91c41644672a2e159c5ce2/packages/father-build/src/build.ts#L104-L107) 使用了 [babel-register](https://babeljs.io/docs/en/babel-register)。使用 babel-register 之后，所有后续被 node 使用 require 语法引用的文件，都会被 babel 进行代码转换。这个可以简单测试下，有 foo.js 和 index.js 两个文件，源码如下:
```javascript
// foo.js
import React from 'react'
export default <div>foo</div>

// index.js
// require("@babel/register");
const b = require('./b');
console.log(b)
```
如果注释 `require("@babel/register")` , 那么执行 `node index.js` 就会报错，因为 node 默认不支持 esm 的语法。而加上 `require("@babel/register")` 之后，就能正常打印结果了。因为此时 foo.js 已经被 babel 进行转换了。


同时会通过 [getUserConfig](https://github.com/umijs/father/blob/8ad068379263cdf3bf91c41644672a2e159c5ce2/packages/father-build/src/build.ts#L24) 读取用户的配置文件，比如读取 `.fatherrc.js` , `.fatherrc.ts` 等等，同时 [schema.ts](https://github.com/umijs/father/blob/8ad068379263cdf3bf91c41644672a2e159c5ce2/packages/father-build/src/schema.ts) 中定义配置文件应该遵循的格式，读取配置文件时会进行校验。


之后就会根据你配置中选择的打包方式，是 rollup 还是 babel 来进行相应的处理
### 2.1 rollup 模式
如果选择使用 rollup 进行打包，那么代码就会先经过 [rollup.ts](https://github.com/umijs/father/blob/8ad068379263cdf3bf91c41644672a2e159c5ce2/packages/father-build/src/rollup.ts) 进入到 [getRollupConfig.ts](https://github.com/umijs/father/blob/8ad068379263cdf3bf91c41644672a2e159c5ce2/packages/father-build/src/getRollupConfig.ts) 中来，且在进入到 getRollupConfig 之前，会经过 [normalizeBundleOpts](https://github.com/umijs/father/blob/8ad068379263cdf3bf91c41644672a2e159c5ce2/packages/father-build/src/normalizeBundleOpts.ts) 处理一些入参，比如处理 **overridesByEntry **参数。


到了 getRollupConfig.ts 中，就根据 type 来拼装 rollup 的参数, 包括组合 plugins，externals 来进行编译。
```javascript
// 部分代码
switch (type) {
  case 'esm':
    return [
      {
        input,
        output: {
          format,
          file: join(cwd, `dist/${(esm && (esm as any).file) || `${name}.esm`}.js`),
        },
        plugins: [...getPlugins(), ...(esm && (esm as any).minify ? [terser(terserOpts)] : [])],
        external: testExternal.bind(null, external, externalsExclude),
      }
    ];

  case 'cjs':
    return [
      {
        input,
        output: {
          format,
          file: join(cwd, `dist/${(cjs && (cjs as any).file) || name}.js`),
        },
        plugins: [...getPlugins(), ...(cjs && (cjs as any).minify ? [terser(terserOpts)] : [])],
        external: testExternal.bind(null, external, externalsExclude),
      },
    ];

  case 'umd':
    // Add umd related plugins
    const extraUmdPlugins = [
      commonjs({
        include,
        // namedExports options has been remove from https://github.com/rollup/plugins/pull/149
      }),
    ];

    return [
      {
        input,
        output: {
          format,
          sourcemap: umd && umd.sourcemap,
          file: join(cwd, `dist/${(umd && umd.file) || `${name}.umd`}.js`),
          globals: umd && umd.globals,
          name: (umd && umd.name) || (pkg.name && camelCase(basename(pkg.name))),
        },
        plugins: [
          ...getPlugins(),
          ...extraUmdPlugins,
          replace({
            'process.env.NODE_ENV': JSON.stringify('development'),
          }),
        ],
        external: testExternal.bind(null, externalPeerDeps, externalsExclude),
      }
    ];

  default:
    throw new Error(`Unsupported type ${type}`);
}
```
father-build 采用的是 rollup JavaScript API 的方式，通过 "循环" 多个 entry 依次进行打包。我理解是因为 rollup 默认的配置方式无法打包多 entry，而只支持单个 entry。
### 2.2 Babel 模式
如果选择 babel 的方式，则会进入到 [babel.ts](https://github.com/umijs/father/blob/8ad068379263cdf3bf91c41644672a2e159c5ce2/packages/father-build/src/babel.ts)。**注意**，代码中硬编码了[读取 src 目录](https://github.com/umijs/father/blob/8ad068379263cdf3bf91c41644672a2e159c5ce2/packages/father-build/src/babel.ts#L62)，因此此时的 entry 配置是无效的。然后通过 [pattern](https://github.com/umijs/father/blob/8ad068379263cdf3bf91c41644672a2e159c5ce2/packages/father-build/src/babel.ts#L188-L196) 找出需要编译的文件，进入到 [createStream 方法](https://github.com/umijs/father/blob/8ad068379263cdf3bf91c41644672a2e159c5ce2/packages/father-build/src/babel.ts#L135)。


[核心代码](https://github.com/umijs/father/blob/8ad068379263cdf3bf91c41644672a2e159c5ce2/packages/father-build/src/babel.ts#L147-L184):
```javascript
return vfs
// 读取源文件
.src(src, {
 allowEmpty: true,
 base: srcPath
})
// 先处理 ts
.pipe(
 gulpIf(f => !disableTypeCheck && isTsFile(f.path), gulpTs(tsConfig))
)
.pipe(
 // 处理 less 文件
 gulpIf(
   f => lessInBabelMode && /\.less$/.test(f.path),
   gulpLess(lessInBabelMode || {})
 )
)
.pipe(
 gulpIf(
   f => isTransform(f.path),
   through.obj((file, env, cb) => {
     try {
       file.contents = Buffer.from(
         // 遇到 tsx, jsx 就用 babel 去处理
         // transform 方法也就是根据 babel 配置来编译文件
         transform({
           file,
           type
         })
       );
       // .jsx -> .js
       file.path = file.path.replace(extname(file.path), ".js");
       cb(null, file);
     } catch (e) {
       signale.error(`Compiled faild: ${file.path}`);
       console.log(e);
       cb(null);
     }
   })
 )
)
.pipe(vfs.dest(targetPath));
```
其中 [vinyl-fs](https://github.com/gulpjs/vinyl-fs) 是在 gulpjs 这个组织下的一个仓库，看起来也是和 gulp 有点关系。因此在里面也使用了 `gulp-less`  和 `gulp-typescript` 对代码进行处理。


简单来说就是根据用户的配置，选择使用 babel 或者 rollup 进行处理，当然在此过程中涉及了许多的第三方依赖库，有需要可以继续查看这些库的功能。其实我只是对 rollup 流程中一些库的文档看的多一些，对于 babel 流程中涉及的第三方依赖，我也就猜测了下功能，没具体测试。毕竟每个库都测试下还挺耗费时间的，有些时候我们不需要了解的那么细。
## 3. rollup 和 babel 模式的一些区别
先简单区分下 rollup 和 babel：

- rollup 用来打包
- babel 用来转换代码。



那有什么区别呢？我们来做个测试，有 foo.tsx, bar.tsx 和 index.tsx 三个文件，源码如下:
```javascript
// foo.tsx
import React from 'react';
import lodash from 'lodash';
import "./index.less";

export default () => {
  console.log(lodash.VERSION)
  return <div className="foo">foo</div>
}

// bar.tsx
import React from 'react';
import "./index.less";

export default () => {
  return <div className="bar">bar</div>
}

// index.tsx
import Foo from './foo'
import Bar from './bar'

export {
  Foo,
  Bar
}
```
看一下两种模式下，index.tsx 的打包结果:


- babel 模式的结果
```javascript
import Foo from './foo';
import Bar from './bar';
export { Foo, Bar };
```

- rollup 模式的结果:
```javascript
import React from 'react';
import lodash from 'lodash';

var index = (function () {
  console.log(lodash.VERSION);
  return /*#__PURE__*/React.createElement("div", {
    className: "foo"
  }, "foo");
});

var index$1 = (function () {
  return /*#__PURE__*/React.createElement("div", {
    className: "bar"
  }, "bar");
});

export { index$1 as Bar, index as Foo };

```
因此我们可以这样理解：使用 rollup 打包 index.tsx 是需要知道原来 foo.tsx 和 bar.tsx 的内容，将他们的内容合并到一起称为打包。而使用 babel 转换 index.tsx 的代码不需要知道原来 foo.tsx 和 bar.tsx 的内容，只需要把在 index.tsx 中遇到的不认识的代码(比如遇到 TS 语法，JSX 语法) 就按规则进行转换就可以了。这些规则就是 babel 的 plugins 和 presets。
另外，如果不设置 external，由于 foo.tsx 或者 bar.tsx 中使用了 react，那么 rollup 模式**甚至会把 react 源码打包进来**。**


其实在 father 的[文档中](https://github.com/umijs/father#bonus)，也写清楚了。只是当你还不熟悉 babel，rollup 的时候，可能不是很理解。
> rollup 是跟进 entry 把项目依赖打包在一起输出一个文件，babel 是把 src 目录转化成 lib（cjs） 或 es（esm）

rollup 会把所有用到的代码打包进来，babel 只是转换代码。并且在 rollup 模式时，如果要转换 JSX 代码，或者将箭头函数变成普通函数，同样会用到 babel 来转换代码，其中依赖了 @rollup/plugin-babel 这个插件。

再来看我使用过程中遇到的两个小问题:


1. 为什么 rollup 模式下会生成类型文件，babel 模式没有

因为 rollup 模式使用了 [rollup-plugin-typescript2](https://github.com/ezolenko/rollup-plugin-typescript2) 插件，并且默认 `declaration: true` 会生成类型文件, 而 babel 模式使用了 gulp 的 [gulp-typescript ](https://www.npmjs.com/package/gulp-typescript)插件，要生成类型文件似乎更复杂一些, gulp-typescript 文档中提到了: 
> - `declaration` (boolean) - Generates corresponding .d.ts files. You need to pipe the `dts` streams to save these files.

而 father-build 默认是没有处理的 dts 的。


2. 为什么 babel 模式下处理不了 sass，而 rollup 模式可以

因为在 babel 模式中，就是硬编码了 [相关逻辑](https://github.com/umijs/father/blob/master/packages/father-build/src/babel.ts#L157-L160)：遇到 .less 文件使用 gulp-less 进行处理，却没有对 sass 的支持。而在 rollup 中使用了 rollup-plugin-postcss 对 sass 和 less [进行处理](https://github.com/umijs/father/blob/master/packages/father-build/src/getRollupConfig.ts#L160-L167)。
## 4. 简单总结
在我没看源码前，这个 father-build 用起来是有点懵的。我不知道某个参数具体是干嘛用的，仿佛打包结果完全不可控(_毕竟里面有一些硬编码的代码，比如 babel 默认转换 src 目录，不允许传参。_)，又因为混杂了 babel 和 rollup 两个东西，容易使人头晕。


后来在看了 rollup 以及 babel 相关文档之后，自己边调试边阅读代码，基本就清楚了整个流程。遇到问题也能够知道问题出现在哪了。


个人感觉，打包的复杂点是在配置，需要配置如何处理样式，ts，npm 包。配置过程会涉及非常多的第三方依赖包，需要了解每个包所实现的功能，需要的配置，如何和其它的包一起工作。另外还有个坑，就是第三方依赖包的版本需要确定，有时候你装的依赖包名称都对了，但就是打包失败，可能就是因为依赖包版本的问题。

father-build 能帮助我们很方便的进行打包，省去配置 rollup 或者 babel 的麻烦 (不得不说有些_配置挺花时间的_)。但是也有小缺点，比如刚刚提到的硬编码问题。建议有空的话可以先阅读其代码(代码很清晰也不多，主要是配置复杂些，也需要先知道一些 babel 和 rollup 的知识)，做一些 demo 看看打包结果，能更好的掌握它，如果发现某些地方可以优化也可以去给开源贡献下代码。

## 结语
最近一直在看打包相关的知识，上一篇文章 [使用 rollup 打造自己的 npm 包 (全流程)](https://juejin.cn/post/6950557086916804645) 主要讲了自己如何配置 rollup 来打包一个纯 js 的 npm 包，以及其中涉及到的文档，测试，以及发布相关的内容。


在分析 father-build 源码的过程中，我也根据 father-build 中 rollup 的相关流程和配置，打包了一个 React 组件库，[项目地址](https://github.com/yes1am/mie-ui)。其中大部分内容上面提到的文章：[使用 rollup 打造自己的 npm 包 (全流程)](https://juejin.cn/post/6950557086916804645) 是一样的，无非是多了样式的处理，文档工具换成了 storybook，有兴趣也可以看看。


最后，如果本文有帮助到你，欢迎 [点个 star](https://github.com/yes1am/mie-ui) 支持，有错误烦请指出，有疑问也欢迎评论交流。