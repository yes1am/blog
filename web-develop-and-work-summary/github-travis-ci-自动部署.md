## 1. ci 能干嘛

能在push代码的时候执行某些操作，同时可以通过指定一些参数，使得只在特定的push，或者commit message时才会执行ci job。  

## 2. 配置示例  

```yml
language: node_js
node_js:
    - 10
cache:
  directories:
    - "node_modules"
install:
    - npm i
script:
    - npm run build
after_success:
    - cd .docz/dist
    - git init
    - git config --global user.name "${U_NAME}"
    - git config --global user.email "${U_EMAIL}"
    - git add -A
    - git commit -m 'deploy'
    - git push --quiet --force "https://${GH_TOKEN}@${GH_REF}" master:${P_BRANCH}
branches:
  except:
    - master
branches:
  only:
  - gh-pages
```  
分别是指定语言，指定node版本，指定缓存目录等，同时在after_success时，通过命令实现向某个特定URL和分支push代码。  

GH_TOKEN： 在github上[生成的token](https://github.com/settings/tokens)，我理解是只有携带该token，travis才能拥有权限执行push的操作(不确定新版本的travis是否还需要这样操作，好像可以直接操作github)。   

GH_REF: 类似github.com/[username]/[repository].git的地址，**无需 https 协议前缀**。  

P_BRANCH： 目标分支。  

## 3. 遇到的问题
### 3.1 travis.org 与 travis.com的区别
[资料](https://devops.stackexchange.com/questions/1201/whats-the-difference-between-travis-ci-org-and-travis-ci-com)，即原本 `travis-ci.com` 用于私人付费项目，`travis-ci.org` 用于公共免费项目。~~但18年5月之后都是推荐使用[travis-ci.com](https://travis-ci.com/).~~ 最近在部署 `react-story-book`的时候，发现 [travis-ci.com](https://travis-ci.com) 加载不到我的`react-story-book`仓库，而 [travis-ci.org](https://travis-ci.org) 可以，因此又使用了`travis-ci.org`。

### 3.2 报错 GitHub Pages branch not include  ...
[branches相关配置资料](https://docs.travis-ci.com/user/customizing-the-build#building-specific-branches),即默认gh-pages分支不会执行ci job,除非手动添加到白名单。  

```
branches:
  only:
  - gh-pages
```

## 4. 参考资料
[VuePress + Travis CI + Github Pages 自动线上生成文档](https://juejin.im/post/5d0715f6f265da1ba56b1e01#comment)