[英文原版](https://docs.gitlab.com/ee/ci/introduction/index.html#introduction-to-cicd-methodologies)  

## 1. 介绍GitLab上的CI/CD

在这篇文章中我们会描述下 持续集成， 持续交付， 持续部署的概念，以及介绍 GitLab CI/CD.  

### 1.1 CI/CD方法论
软件开发的持续方法论是基于脚本的自动化执行，以减少应用程序开发时引入错误的机会。从项目开发到部署很少需要甚至不需要人工的接入。  

它包含持续的构建，测试，以及每一次版本小迭代时的部署代码，降低了在有bug的，失败的上一个版本中去开发新的代码。  

这种方法论主要有三种处理方式，每一种都应该根据你的战略来应用。  


#### 1.1.1 持续集成

想象下一个应用，它的源码在 GitLab 的仓库里，开发者每天提交很多新的代码，对于仓库的每一个提交，你可以创建一系列的脚本来自动的构建，测试你的应用，减少向你的应用引入错误的机会。  

这种实践就是持续集成，对于应用的每一次提交，甚至是开发分支，它都应该持续且自动的被构建，和测试，确保新提交的代码通过了所有的测试，参考，并且代码符合你给你应用设定的规范。  

GitLab自身就是使用持续集成作为软件开发方法，对于项目的每一次提交，都会有一系列脚本的代码检查。  

#### 1.1.2 持续交付

持续交付是持续集成之外的一个步骤，你的应用不仅仅是构建，测试提交上来的每一行代码变化，并且，它也是持续的被部署，尽管部署是手动的。  

这种方式确保你的代码被自动检查，但是需要人类介入来手动的触发部署。

#### 1.1.3 持续部署

持续部署是持续集成之外更进的一步，和持续交付一样。不同的是，持续交付是手动的部署你的应用，持续部署是自动部署，它的部署不需要人类的介入。  

### 1.2 GitLab上的CI/CD

GitLab CI/CD 是一款强大的内置于GitLab的工具，它允许你将所有的持续方法(持续集成，交付，部署)应用到你的软件当中，而无需第三方应用或者整合。

#### 1.2.1 GitLab CI/CD 是怎么工作的

为了使用GitLab CI/CD,你需要做的是，你的应用代码库是在一个Git仓库中，并且你的构建，测试和部署脚本都在一个叫做`.gitlab-ci.yml`的文件中指定，并且在你应用仓库的根目录中。  

在这个文件中，你可以定义你想要执行的脚本，定义包含和缓存的依赖，选择哪些命令你想要串行的执行而哪些需要并行，定义你应用想要部署的位置，并且指定是自动执行这些脚本还是手动的执行其中的一部分。一旦你熟悉了 GitLab CI/CD，你可以在配置文件中添加更多的高级步骤。  

为了向那个文件添加脚本，你需要将他们序列化的组织好，以适应你的应用并且与你希望的测试保持一致。为了可视化这个过程，想象下你配置文件中添加的所有脚本都像是在你自己电脑上的终端上执行命令一样。  

一旦你向你的仓库添加了 `.gitlab.yml` 配置文件，GitLab会发现它并且使用一个叫做 `GitLab Runner` 的工具去执行你的脚本，这个工具就像你的终端命令行一样。  

脚本都被划分为 `jobs` ，并且所有的 `jobs` 组合在一起构成了 `pipeline`， 一个最简单的 `.gitlab-ci.yml` 的例子可以包含以下内容：  

```shell
before_script:
  - apt-get install rubygems ruby-dev -y

run-test:
  script:
    - ruby --version
```

`before_script` 属性会在执行其它所有事情之前去安装你的应用所需要的依赖，一个叫做 `run-test` 的 `job` 会打印出当前系统 ruby 的版本，他们两个组成了一个 `pipeline`, 会在这个仓库任何分支的每一次提交时执行。  

GitLab CI/CD 不仅执行你定义的一些 `jobs`，并且会展示在执行过程中发生了什么，这些你可以在`terminal` 中看到。  

你为你的应用创建了策略，GitLab 根据你定义的配置文件执行 `pipeline`，你的 `pipeline` 的状态也会在 `GitLab` 中展示出来。  

最后，如果有任何一个步骤出了错，你都可以轻易的 `回滚` 所有的变化。  

#### 1.2.2 基本的 CI/CD 工作流

根据以下的例子，想象下 GitLab CI/CD 是怎么适应大多数开发工作流的。  

假设你已经在issue中讨论了一段代码的实现，想要在本地去实现它。一旦你将该"功能"分支提交到了远程的GitLab仓库， 你设定的CI/CD 的 `pipeline`会被触发，之后GitLab CI/CD 会：
- 自动化的执行脚本(串行或者并行的)
    - 构建，测试你的应用
    - 使用 预览版应用 来预览你的提交，你会在 localhost 中预览到

一旦你对自己的实现比较满意：  
- 修订你的代码并且通过
- 将功能分支合并到默认分支
    - GitLab CI/CD 会将你的代码变动，自动的部署到生产环境
- 最终，你和你的团队可以轻易的在出错的时候进行回滚

GitLab CI/CD 有能力做的更多，但是这个工作流的示例了GitLab整个过程的能力，没必要使用任何外部的工具来交付你的应用。并且，最有用的是，你可以通过GitLab UI可视化整个过程。  

##### 1.2.2.1 深入了解一下 CI/CD 的基本工作流

如果我们深入的了解一下基本工作流，我们可以看到GitLab在 DevOps 生命周期中每个阶段的功能特性，像下面的插图示例的那样：  

如果你从左向右的查看这张图片，你会看到GitLab在每个阶段的一些功能特性(校验，打包，发布)：  
1. 校验  
- 通过持续集成自动的构建并测试你的应用
- 通过 [GitLab Code Quality](https://docs.gitlab.com/ee/user/project/merge_requests/code_quality.html) 分析你的代码质量
- 通过[Browser Performance Testing](https://docs.gitlab.com/ee/user/project/merge_requests/browser_performance_testing.html) 确定代码改动的性能的影响
- 进行一系列的测试，比如 `Container Scanning`，`Dependency Scanning` 和 `JUnit tests`
- 通过 `Review App` 部署你的代码改动来预览你的应用的各个分支改动

2. 打包
- 通过 `Container Registry` 来存储 Docker 镜像
- 通过 `NPM Registry` 来存储 NPM 包
- 通过 `Maven Registry` 来存储  Maven artifacts

3. 发布

- 持续部署，自动化的将你的应用部署到生产环境
- 持续交付，手动的操作来部署你的应用到生产环境
- 使用 `GitLab Pages` 来部署静态页面
- 只在一部分的应用"出口"中带有新的功能特性，让一部分用户可以通过`Canary Deployments` 暂时的预览你的应用部署的特性
- 在 `Feature Flags` 前部署你的功能分支
- 使用 `GitLab Releases` 添加发布记录
- 使用 `Deploy Boards` 查看当前Kubernetes上每个CI环境的健康状况
- 使用 `Auto Deploy` 向Kubernetes的生产环境部署你的应用

使用GitLab CI/CD 你还可以：  

- 使用 `Auto DevOps` 简单的设置你的应用的整个生命周期
- 部署你的应用到不同的环境
- 安装你自己的 GitLab Runner
- 安排好 `pipelines`
- 通过 `Security Test reports` 检查应用的安全缺陷

#### 1.2.3 首先设置 GitLab CI/CD

为了开始使用 GitLab CI/CD，你需要熟悉 `.gitlab-ci.yml` 配置文件的语法以及它的属性。  

这篇文章[介绍GitLab CI/CD中GitLab Pages的相关概念](https://docs.gitlab.com/ee/user/project/pages/getting_started_part_four.html)，介绍了如何部署静态网站。尽管它是用来拯救那些想要编写他们自己的页面脚本的用户，它也起到了介绍如何给GitLab CI/CD设置一些程序的作用。它包含了很基本通用的编写 CI/CD 配置文件的步骤，所以我们推荐你通过阅读它来理解 GitLab的CI/CD逻辑，并且学会怎样为你的应用去编写你自己的脚本(或者调整当前已有的配置文件)。  

为了深入 GitLab CI/CD 配置文件，请查看 [.gitlab-ci.yml完全参考](https://docs.gitlab.com/ee/ci/yaml/README.html)