## 前言

一直以来自己的 commit message 都比较随意，之前有看到同事写: feat, chore等，同时还有相应的 emoij 表情，觉得很酷。便搜寻资料整理出此文，便于后续查询相关知识。

## 1. commit message 规范

```
<type>(<scope>): <subject>
// 空一行
<body>
// 空一行
<footer>


// 示例

docs(guide): updated fixed docs from Google Docs

Couple of typos fixed:
- indentation
- batchLogbatchLog -> batchLog
- start periodic checking
- missing brace
```

type 说明 commit 类型  

scope： 可选，用于说明 commit 影响的范围  

subject: commit 目的的**简单**描述

body: 可选，详细的 commit 描述  

footer: 可选，关闭的 issues 或者**不兼容的变更**  

## 2. Type 类型

1. feat: 新功能
2. fix: bug 修复
3. docs: 文档(documentation)
4. style: 代码格式化，添加分号等(不影响代码运行的变动)
5. refactor: 重构(不是新增功能也不是修改bug)
6. test: 增加或修改测试用例
7. chore: 构建过程或者辅助工具的变动
8. perf: 改善性能的修改

## 3. 如何书写多行 commit message

Git 每次提交代码，都需要写 Commit message，否则不允许提交  

当 Commit message 存在多行时，可以执行: **`git commit`**, 此时会进入 vim 编辑器，允许输入多行文字。 

## 4. 格式化输出 commit message

### 4.1 筛选出有效信息
通常我们使用 `git log` 查看 commit 信息，如下:  

![git log](https://raw.githubusercontent.com/yes1am/PicBed/master/img/e00826028e7c077daa4e32781042c18.png)

我们可以使用以下命令，得到更简洁的输出:  

`git log <last tag> HEAD --pretty=format:%s`  

![git log --pretty](https://raw.githubusercontent.com/yes1am/PicBed/master/img/1d4cd0f9c87640e77cee3d8bd1c4111.png)  

另外，我们可以使用 `--grep` 得到包含某些单词的提交，比如 `--grep feature` 来得到属于 feature 类型的提交:  

`git log <last release> HEAD --pretty=format:%s --grep feat`  


![git log --grep feature](https://raw.githubusercontent.com/yes1am/PicBed/master/img/0af1baf1dbdeb402139b362e30f8e64.png)  

## 5. [Commitizen](https://github.com/commitizen/cz-cli)  

通过与命令行的交互，生成符合 AngularJS 规范的 commit message  

具体使用参考[Commitizen 官方文档](https://github.com/commitizen/cz-cli)

## 6. commit message with emoij

如何在 commit 信息中添加 emoij 表情，以及不同 emoij 表情所对应的 type？  

参考 [git commit message emoji 使用指南](https://github.com/liuchengxu/git-commit-emoji-cn)，

## 参考资料
1. 阮一峰 [Commit message 和 Change log 编写指南](https://www.ruanyifeng.com/blog/2016/01/commit_message_change_log.html)
2. gold-miner: [AngularJS 规范中文翻译](https://github.com/yes1am/gold-miner/issues/5) 
3. [Augular 规范](https://docs.google.com/document/d/1QrDFcIiPjSLDn3EL15IJygNPiHORgU1_OOAqWjiDU5Y/edit#heading=h.uyo6cb12dt6w)
4. [Commitizen](https://github.com/commitizen/cz-cli)
5. [git commit 时使用 Emoji ?](https://zhuanlan.zhihu.com/p/29764863)
6. [git commit message emoji 使用指南](https://github.com/liuchengxu/git-commit-emoji-cn)
7. [An emoji guide for your commit messages](https://gitmoji.carloscuesta.me/)
8. [find github all emoij here](https://gist.github.com/rxaviers/7360908)