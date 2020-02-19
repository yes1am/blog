[英文原版](https://docs.gitlab.com/ee/user/project/pages/getting_started_part_four.html)

# 为GitLab Pages创建并调整GitLab CI CD

为了开始GitLab Pages, 你应该使用一个项目模板，一个 `.gitlab-ci.yml` 模板，或者 fork 一个已经存在的示例项目。  

因此，你没必要完全理解 GitLab CI/CD 就能部署你的网站。但是，有些情况下你需要编写你自己的或者调整现有的配置文件。这篇文章会带领你经历这个过程。  

这篇文章也提供了一个全面而清晰的介绍来熟悉 `.gitlab-ci.yml` 文件并且来编写它。  

GitLab CI/CD 有许多的用途，从你的GitLab中通过持续集成，持续交付，和持续部署等方法来构建，测试，部署你的应用。你需要用GitLab Pages 来构建你的网站并且部署到静态页面服务器。  

为了使用GitLab CI/CD， 你要做的第一件事就是一份叫做 `.gitlab-ci.yml` 的配置文件放置在你的网站的根目录下。  

这个文件的作用是告诉 `GitLab Runner` 去执行那些你会在命令行中执行的脚本。Runner就像你的终端一样工作，GitLab CI/CD 告诉 Runner 去执行哪些命令。CI/CD 和 Runner 都内置于GitLab中，你不需要设置额外的事情他们就能工作。  

解析 [GitLab CI/CD 的每个细节](https://docs.gitlab.com/ee/ci/yaml/README.html) 以及 GitLab Runner 不在本篇文章的范围内，但是为了能够编写我们自己的或者修改调整已经存在的`.gitlab-ci.yml`,我们需要理解一些基本的事情。 这是一个 `YAML` 文件，有着自己的语法，你可以利用 [GitLab CI Lint Tool](https://gitlab.com/ci/lint)工具来检验你的CI语法。  

## 1. 实际例子

想象下你有一个 `Jekyll` 网站，为了本地构建它，你需要打开你的终端，执行 `jekyll build`。 当然，在构建之前，你需要在你的电脑上安装 `Jekyll`。 为了安装，你需要打开你的终端执行 `gem install jekyll`。是这样吧？ GitLab CI + GitLab Runner 也是这样做，但是你只需要在`.gitlab-ci.yml`中编写你需要执行的脚本，这样GitLab Runner就会帮你做好这些事情。这看起来比实际上复杂一些.  

你需要告诉Runner:  
```
gem install jekyll
jekyll build
```

### 1.1 脚本 Script
一个 YAML 格式的脚本就像下面那样：  
```
script:
 - gem install jekyll
 - jekyll build
```

### 1.2 任务 Job

到目前还好，每个脚本，在Gitlab中被一个job管理着， job 是一系列你需要应用到指定Task上的脚本和设置  

```
job:
  script:
  - gem install jekyll
  - jekyll build
```

对于GitLab Page 来说， job 有一个特殊的名字，叫做 `pages`, 它告诉 Runner 你想要用 GitLab Pages 部署你的网站：  

```
pages:
  script:
  - gem install jekyll
  - jekyll build
```

### 1.3 Public 目录

我们也需要告诉 Jekyll 你想要把你的网站构建在哪里，并且GitLab Pages 只会考虑在 `public` 目录中的文件。对于 Jekyll 来说，我们需要指定一个标志位 flag 来指明构建网站的 `destination (-d)`： `jekyll build -d public`， 当然，我们需要告诉我们的 Runner： 

```
pages:
  script:
  - gem install jekyll
  - jekyll build -d public
```

### 1.4 Artifacts (有道翻译：史前古器物；人工产品？)
我们也需要告诉 Runner ，这个 job 生成了 artifacts ,artifacts 就是 Jekyll 构建的网站。 artifacts存放在哪里呢？在 `public` 目录：  

```
pages:
  script:
  - gem install jekyll
  - jekyll build -d public
  artifacts:
    paths:
    - public
```

以上的脚本足够你用 GitLab Pages 创建 Jekyll 网站了，但是 从 Jekyll 3.4.0 开始，它的默认模板创建命令 `jekyll new project` 需要`Bundler` 来安装 Jekyll 的依赖和默认主题。为了调整我们的脚本来适应新的需求，我们只需要用Bundler来安装和构建Jekyll:  

```
pages:
  script:
  - bundle install
  - bundle exec jekyll build -d public
  artifacts:
    paths:
    - public
```

就是这样，一个拥有以上内容的 `.gitlab-ci.yml` 会将你的 Jekyll 3.4.0 版本的网站部署到 GitLab Pages，这是我们示例的最小配置。在接下来的步骤中，我们会添加额外的选项，提炼出新的脚本。  

Artifacts 会在 GitLab Pages 页面部署之后被自动删除，你可以指定过期时间来保留 artifacts 一段时间。  

### 1.5 镜像 Image

到了这一步，你可能会问你自己："好吧，但是我需要Ruby来安装Jekyll，脚本中的Ruby在哪？"，这个问题很简单，在 `.gitlab-ci.yml`中 GitLab Runner 找的第一件事就是 Docker 镜像，它制定了你的容器中需要什么环境来执行脚本:  

```
image: ruby:2.3

pages:
  script:
  - bundle install
  - bundle exec jekyll build -d public
  artifacts:
    paths:
    - public
```

在这个例子中，你告诉 Runner 去把包含 Ruby 2.3 的镜像拉下来作为它文件系统的一部分。当你不指定镜像时，Runner 会使用一个默认的镜像， 是 Ruby 2.1.  

如果你的SSG 需要 NodeJS 来构建，你需要指定你想用哪个镜像，这个镜像应该 包含NodeJS。 例如，对于 Hexo 站点，你可以使用 `image: node：4.2.2`  

让我们继续

### 1.6 分支 Branching

如果你使用GitLab作为版本控制平台，你项目中会有一些分支的的策略。意味着，你的项目中会有其他的分支，但是你想只有提交到默认分支(通常是master)才部署你的网站。为了实现这个，你需要在CI中添加另外一行，告诉 Runner 只在 master 分支执行叫做 `pages` 的 job：  

```
image: ruby:2.3

pages:
  script:
  - bundle install
  - bundle exec jekyll build -d public
  artifacts:
    paths:
    - public
  only:
  - master
```

### 1.7 阶段 Stages

另一个有意思的需要记住的概念是构建阶段 `stage`. 你的应用在部署到 `staging` 或者 生产环境之前可以通过一系列的测试和其他任务。GitLab 默认有三个 stages: 构建，测试和部署。为了指定你的job是在哪个 stage 执行，可以简单的添加一行：  

```
image: ruby:2.3

pages:
  stage: deploy
  script:
  - bundle install
  - bundle exec jekyll build -d public
  artifacts:
    paths:
    - public
  only:
  - master
```

你可能会有疑问："为什么我需要关心stage"，那是为了让你能够在部署你的网站到生产环境之前，去测试你的脚本并且检查构建出来的网站。你想要在你push到master的时候去执行测试脚本。很简单，在CI里添加一个 job ，让它在master分支之外的分支提交的时候，去执行测试：  

```
image: ruby:2.3

pages:
  stage: deploy
  script:
  - bundle install
  - bundle exec jekyll build -d public
  artifacts:
    paths:
    - public
  only:
  - master

test:
  stage: test
  script:
  - bundle install
  - bundle exec jekyll build -d test
  artifacts:
    paths:
    - test
  except:
  - master
```

`test`的job在`test`阶段执行，Jekyll 会在 test 目录构建站点，这个 job 会影响除了`master`之外的其它分支。  

给不同的 `jobs` 分配 `stage` 最大的好处是，同一个 `stage` 的不同 `job` 会并行的执行。如果你的应用再部署之前需要多次的测试，你可以同时执行所有的测试，没必要等待一个测试跑完再执行下一个。当然这是 GitLab CI 和 GitLab Runner 的简短介绍，他们实际上更为强大。这就是为什么有能力为 GitLab Pages 创建和调整你的构建过程。

### 1.8 脚本执行前 Before Script

为了避免在多个job中执行多次相同的脚本，你可以添加 `before_script` 参数，在这你可以定义哪些命令你想让每一个job都执行。在我们的例子中，我们在 `pages` 和 `test` 的 `jobs` 中都执行了 `bundle install`, 没必要重复:  

```
image: ruby:2.3

before_script:
  - bundle install

pages:
  stage: deploy
  script:
  - bundle exec jekyll build -d public
  artifacts:
    paths:
    - public
  only:
  - master

test:
  stage: test
  script:
  - bundle exec jekyll build -d test
  artifacts:
    paths:
    - test
  except:
  - master
```

### 1.9 缓存依赖 Cache Dependencies

如果你需要加速构建，缓存你项目中的依赖，那么可以使用 `cache` 参数，在我们的例子中，在执行 `bundle install` 时，我们缓存 Jekyll 这个依赖到 `vendor` 目录中：  

```
image: ruby:2.3

cache:
  paths:
  - vendor/

before_script:
  - bundle install --path vendor

pages:
  stage: deploy
  script:
  - bundle exec jekyll build -d public
  artifacts:
    paths:
    - public
  only:
  - master

test:
  stage: test
  script:
  - bundle exec jekyll build -d test
  artifacts:
    paths:
    - test
  except:
  - master
```

这个例子中，我们需要在 `_config.yml` 文件中忽略掉 `/vendor` 目录，否则 Jekyll 会把它当做一个常规的目录在构建网站时进行编译：  

```
exclude:
  - vendor
```

现在我们的 GitLab CI 不仅仅是构建我们的网站，而且会持续测试我们的功能分支的提交，缓存 Bunder 安装的依赖，并且根据每次`master`分支的提交，持续的部署我们的网站。  

## 2. GitLab Pages 中的 GitLab CI 高阶指引

你能用 GitLab CI 做的事情取决于你的创造力。一旦你习惯了，你可以开始创建极好的脚本来自动化的完成过去需要你手动完成的任务。 阅读 [GitLab CI 文档](https://docs.gitlab.com/ee/ci/yaml/README.html) 来了解怎么进一步的完善你的脚本。  

- [了解如何使用GitLab CI environment 来部署你的网站到 staging 和生产](https://about.gitlab.com/2016/08/26/ci-deployment-and-environments/)
- [了解如何串行的，并行的执行任务，或者创建一个自定义的pipeline](https://about.gitlab.com/2016/07/29/the-basics-of-gitlab-ci/)
- [拉取不同项目的指定目录来部署一个网站](https://about.gitlab.com/2016/12/07/building-a-new-gitlab-docs-site-with-nanoc-gitlab-ci-and-gitlab-pages/)
- [如何使用GitLab Pages产出代码覆盖率报告](https://about.gitlab.com/2016/11/03/publish-code-coverage-report-with-gitlab-pages/)