[英文原版](https://docs.google.com/document/d/1QrDFcIiPjSLDn3EL15IJygNPiHORgU1_OOAqWjiDU5Y/edit#)  

# AngularJS Git 提交信息约定

## 1. 目的
1. 允许通过脚本自动生成 CHANGELOG.md
2. 允许通过 git bisect 忽视不重要的提交(例如代码格式化)
3. 当查看提交历史时，提供更好的信息

### 1.1. 生成 CHANGELOG.md
在 changelog 中，我们会有三个部分：**新的特性，bug 修复，不兼容的改动**。这样的清单可以在发布的时候通过脚本生成，并且会有相关的提交的链接。当然，你可以在真正发布之前修改 changelog，但是脚本依旧能够生成一个骨架。  

1. 从最近一次发布开始,列出所有的主题(指 commit message 信息的第一行):  

`git log <last tag> HEAD --pretty=format:%s`


2. 列出这一次发布的新特性

`git log <last release> HEAD --grep feature`

### 1.2 区分出不重要的提交

这些是代码格式化导致的变化(添加/删除 空格/空行， 缩进)，缺少分号，注释。因此，当你在寻找一个真正的修改时，你可以忽略 这些没有逻辑代码修改的提交。  

当使用二分查找法时，你可以这样来忽略:  

`git bisect skip $(git rev-list --grep irrelevant <good place> HEAD)`  
### 1.3. 在查看提交历史的时候，提供更多的信息

这像是添加了 "上下文" 信息。  

查看这些提交信息(从最近的一些 angular的提交中获得的)  

1. Fix small typo in docs widget (tutorial instructions)
2. Fix test for scenario.Application - should remove old iframe
3. docs - various doc fixes
4. docs - stripping extra new lines
5. Replaced double line break with single when text is fetched from Google
6. Added support for properties in documentation

对应翻译： 
1. 修复了 widget 文档中的小错误(教程说明)
2. 修复了 scenario.Application 的测试 - 应该移除老的 iFrame
3. 文档 - 多处文档修改
4. 文档 - 删除多余的空行
5. 当文本从 Google 返回的时候，使用一个空白行代替两行
6. 添加 documentation 属性的支持

所有的这些信息都尝试说明修改了哪里，但是他们没有任何的约定...  

再查看一下这些信息：  

1. fix comment stripping
2. fixing broken links
3. Bit of refactoring
4. Check whether links do exist and throw exception
5. Fix sitemap include (to work on case sensitive linux)

对应翻译:  
1. 删除注释
2. 删除坏链接
3. 很多重构
4. 检查链接是否存在并抛出异常
5. 修改 sitemap include (为了在大小写敏感的linux上正常允许)

你能猜出里面修改了什么嘛？这些信息没有说明地点... 

还有可能就是一些代码片段： docs, docs-parser, compiler, scenario-runner...  

我知道，你可以通过查看什么文件改变了来得到一些信息，但是，那样效率太低。并且，当查看 git 记录的时候，我能知道我们大家都尝试说明文件修改的地点，只是缺少了约定。  

## 2. commit message 的格式

```
<type>(<scope>): <subject>
<BLANK LINE>  // 空白行
<body>
<BLANK LINE>  // 空白行
<footer>
```

commit message 的一行不要超过 100 个字符，这样会使得提交信息在 github 上和其他 git 工具上，更具有可读性。  

一个 commit message 包含一个*header*，*body*，和*footer*,两者之间通过**空白行**分隔  

### 2.1 撤销，重做 (Revert)

如果一个提交时对上一个提交的重做，那么 header 应该以 `revert` 开始，紧接着是**撤销提交的header**。在 body 中应该说明：`这是对 [hash] commit 的重做` 其中 hash 是被重做的提交的 SHA  

### 2.2 commit 信息头部 (Message header)

commit 信息头部应该是 **单行**，包含了这次修改的简洁描述，包含一个**类型**，一个可选的**作用域**以及一个**主体**  

#### 2.2.1 允许的类型
描述了这次 commit 提交属于的类型:  

1. feat (feature) 新增特性
2. fix (bug fix) 修复bug
3. docs (documentation) 文档方面的工作
4. style (代码格式化，缺少分号，冒号等)
5. refactor 重构
6. test (增加测试代码)
7. chore (辅助工具的维护)

#### 2.2.2 允许的域

域可以是任何指出这次修改发生地的名词，比如 **$location**,**$browser**,**$compile**, **$rootScropt**，**ngHref**，**ngClick**，**ngView** 等等  

如果没有适当的域的话，你也可以使用 * 号  

#### 2.2.3 主体内容 (subject)

一个非常简短的对修改的描述

1. 使用命令的语气，一般现在时："change" 而不是 "changed" 或者 "changes"
2. 第一个字母不需要大写
3. 末尾不要使用 .

### 2.3 message body (信息主体)
1. 和 <subject> 主体内容一样，使用命令语气，一般现代时
2. 包括修改的动机，以及之前版本的对比

https://365git.tumblr.com/post/3308646748/writing-git-commit-messages  

https://tbaggery.com/2008/04/19/a-note-about-git-commit-messages.html


### 2.4 message footer (信息尾部)

#### 2.4.1 不兼容的修改

所有的不兼容的修改都应该作为 breaking change 在 footer 里提到，并且应该以 `BREAKING CHANGEL:` 和 一个空格或者一个两空白行开始。剩下的内容就是用来描述这次修改，理由和迁移说明。  

例如：  

```
BREAKING CHANGE: isolate scope bindings definition has changed and
    the inject option for the directive controller injection was removed.
    
    To migrate the code follow the example below:
    
    Before:
    
    scope: {
      myAttr: 'attribute',
      myBind: 'bind',
      myExpression: 'expression',
      myEval: 'evaluate',
      myAccessor: 'accessor'
    }
    
    After:
    
    scope: {
      myAttr: '@',
      myBind: '@',
      myExpression: '&',
      // myEval - usually not useful, but in cases where the expression is assignable, you can use '='
      myAccessor: '=' // in directive's template change myAccessor() to myAccessor
    }
    
    The removed `inject` wasn't generaly useful for directives so there should be no code using it.
```

#### 2.4.2 相关的 issues
关闭的 bugs 应该在 footer 中单独用一行来展示，并且以 **Closes** 这个前缀开始，就像：  

```
Closes #234
```

或者是多个 issues：  

```
Closes #123, #245, #992
```  

## 3. Commit message 示例

**示例1**  

```
feat $browser): onUrlChange event (popstate/hashchange/polling)

Added new event to $browser:
- forward popstate event if available
- forward hashchange event if popstate not available
- do polling when neither popstate nor hashchange available

Breaks $browser.onHashChange, which was removed (use onUrlChange instead)
```

**示例2**  

```
fix($compile): couple of unit tests for IE9

Older IEs serialize html uppercased, but IE9 does not...
Would be better to expect case insensitive, unfortunately jasmine does
not allow to user regexps for throw expectations.

Closes #392
Breaks foo.bar api, foo.baz should be used instead
```

**示例3**  
```
feat(directive): ng:disabled, ng:checked, ng:multiple, ng:readonly, ng:selected

New directives for proper binding these attributes in older browsers (IE).
Added coresponding description, live examples and e2e tests.

Closes #351
```

**示例4**  
```
style($location): add couple of missing semi colons
```

**示例5**  
```
docs(guide): updated fixed docs from Google Docs

Couple of typos fixed:
- indentation
- batchLogbatchLog -> batchLog
- start periodic checking
- missing brace
```  

**示例6**  
```
feat($compile): simplify isolate scope bindings

Changed the isolate scope binding options to:
  - @attr - attribute binding (including interpolation)
  - =model - by-directional model binding
  - &expr - expression execution binding

This change simplifies the terminology as well as
number of choices available to the developer. It
also supports local name aliasing from the parent.

BREAKING CHANGE: isolate scope bindings definition has changed and
the inject option for the directive controller injection was removed.

To migrate the code follow the example below:

Before:

scope: {
  myAttr: 'attribute',
  myBind: 'bind',
  myExpression: 'expression',
  myEval: 'evaluate',
  myAccessor: 'accessor'
}

After:

scope: {
  myAttr: '@',
  myBind: '@',
  myExpression: '&',
  // myEval - usually not useful, but in cases where the expression is assignable, you can use '='
  myAccessor: '=' // in directive's template change myAccessor() to myAccessor
}

The removed `inject` wasn't generaly useful for directives so there should be no code using it.

```