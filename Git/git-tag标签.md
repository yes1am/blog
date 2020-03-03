## 1. git tag
git 可以给历史中的某一次提交打上标签  

- 列出标签: `git tag`

Git 中有两种标签，**轻量标签** 和 **附注标签**，**推荐使用附注标签**  

- 轻量标签很像一个不会改变的分支，它只是一个特定提交的引用。
- 附注标签是存储在 Git 数据库中的一个完整对象，包含更多的信息。

添加轻量标签:  
```shell
git tag <标签名>

// git tag v1.0
```

添加附注标签:  
```shell
git tag -a  <标签名> -m <备注信息>

// git tag -a v1.0 v1.1 -m "add tag v1.1"
备注信息是必须的
```

**注意:**  
- 已有轻量标签 v1.0， 则不能新增附注标签 v1.0 了,即轻量标签和附注标签是同一个命名空间
- 轻量标签，和附注标签都可以显示在 github 上，前提是已经被 **push**


展示 tag 的信息，以及 tag 对应的提交信息:  

```shell
git show <标签名>

// git show v1.0
```

给之前的提交添加 tag:  

```shell
git tag <标签名> <commit 对应的 hash>

// git tag v1.3 e4c21ccf0fdc6e39577c70db6a52b8df5cd92829
```

推送 tag:  

git push 不会自动将 tag 信息 push 到远程，因此你必须手动的 push tag

```shell
git push prigin <标签名>

// git push origin v1.6
```
如果想一次性推送所有的 tag， 则可以:  

```shell
git push origin --tags
```

删除本地标签:  

```shell
git tag -d <标签名>  

// git tag -d v1.1
```
但是这样操作并不会删除远程的 tag， 因此你需要:  

```shell
git push origin :refs/tags/<已经被删除的，需要更新的tag>

// git push origin :refs/tags/v1.4
```

## 2. npm version

如果你要升级项目的版本  

一种方式是通过手动修改 package.json 的 version 字段，然后**手动 add, commit**  

另一种方式是直接使用 `npm verison [ major(大版本) | minor(feature) | patch(修复 bug) ]` 

执行类似 `npm version patch` 的命令，会触发两个操作:  
- 修改 paclage.json 中的 version，将 v1.0.0 改为  v1.0.1
- 保存修改，并产生新的 commit，例如 `git add package.json && git commit`

此时新的 commit 信息，默认为新版本号，准确的说，执行的 commit 应该是: `git commit -m "v1.0.1"`  

如果想自定义信息，可以:  

```shell
npm version patch -m "Upgrade to %s"  

// 其中 %s 会自动替换为新的版本号，如 v1.0.2
```

**同样，新的 tag 信息必须手动执行 `git push prigin <标签名>` 才会推送给远程**


## 参考资料
1. [Git 基础 - 打标签](https://git-scm.com/book/zh/v2/Git-%E5%9F%BA%E7%A1%80-%E6%89%93%E6%A0%87%E7%AD%BE)
2. [Git tag 的使用与 npm version](https://www.jianshu.com/p/9e64bdf1e8f9)