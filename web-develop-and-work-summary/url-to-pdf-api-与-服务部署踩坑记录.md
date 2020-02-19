## 前言

最近有个“订阅”的需求，用户填写订阅时间，订阅某个dashboard之后，可以在邮件里定时收到dashboard的截图，不用点进系统进行查看，也方便做一些“周报”工作。  

## 1. url-to-pdf-api
实习期间做别的截图需求用过前端的方案`html2canvas`,那次效果不好所以就想寻找新的方案，随后发现一款`无头浏览器`的解决方案[url-to-pdf-api](https://github.com/alvarcarto/url-to-pdf-api)，里面的一些example,加上之前了解过[puppeteer](https://github.com/GoogleChrome/puppeteer) 顿时觉得这个方案很靠谱。  

## 2. 部署  
官网提供的代码很是方便，clone下来`npm instll`再`npm start`便齐活。测试了下要截图的页面，也是一切ok。以为一切结束，后来才知道这只是刚刚开始...  

### 2.1 Docker  
本地开发用的mac系统，服务器是linux系统，而且是docker环境，那先本地开始用Docker部署下吧。  
[Docker入门教程 阮一峰](http://www.ruanyifeng.com/blog/2018/02/docker-tutorial.html)  
[Docker入门到实践](https://yeasy.gitbooks.io/docker_practice/)  

基础命令:  
```
docker pull ubuntu : 拉取镜像

docker images ： 显示本地镜像

// 根据当前路径(.)下的Dockerfile打包出一个镜像，名字为 name  
docker image build -t name .

// 运行name镜像，启动容器,允许用户进行交互
// -t 选项让Docker分配一个伪终端（pseudo-tty）并绑定到容器的标准输入上
// -i 则让容器的标准输入保持打开。在交互模式下，用户可以通过所创建的终端来输入命令。
docker container run -p 8000:3000 -it name /bin/bash

// 进入已经启动的容器
docker container exec -it [containerID] /bin/bash

// 挂载 即将本机的某个目录挂载到容器中，两边文件会同步。
// 将本地的my/host/1目录挂载到容器的/container/1目录下
docker container run -p 8000:3000 -it -v /on/my/host/1:/on/the/container/1 name /bin/bash  

// 多次挂载
docker -v /on/my/host/1:/on/the/container/1 \
-v /on/my/host/2:/on/the/container/2 \
...  

// 如果启动容器时，容器由于一些原因报错(如容器的默认行为会做一些操作，比如读取某些文件但不存在)，但底层你依赖的容器具体做了什么你并不知道，此时可参考设置entrypoint。  
https://yeasy.gitbooks.io/docker_practice/image/dockerfile/entrypoint.html  
// 重置默认行为，进入bash
docker container exec -it [containerID] --entrypoint/bin/bash  

// 在基础镜像之上构建自己的镜像 commit
https://yeasy.gitbooks.io/docker_practice/image/commit.html  
```  

这样在本地起了个docker容器，测试了下是能成功运行服务的，遇到的问题根据 [trouble shooting](https://github.com/GoogleChrome/puppeteer/blob/master/docs/troubleshooting.md#chrome-headless-doesnt-launch-on-unix) 也基本能解决，主要是依赖的问题，chromium需要依赖一些字体，和其它的库，需要手动安装。  

```
#依赖库
yum install pango.x86_64 libXcomposite.x86_64 libXcursor.x86_64 libXdamage.x86_64 libXext.x86_64 libXi.x86_64 libXtst.x86_64 cups-libs.x86_64 libXScrnSaver.x86_64 libXrandr.x86_64 GConf2.x86_64 alsa-lib.x86_64 atk.x86_64 gtk3.x86_64 -y

#字体
yum install ipa-gothic-fonts xorg-x11-fonts-100dpi xorg-x11-fonts-75dpi xorg-x11-utils xorg-x11-fonts-cyrillic xorg-x11-fonts-Type1 xorg-x11-fonts-misc -y
```  

### 2.2 修改源
装依赖过程中，涉及到修改centos的默认源，采用国内下载源。否则下载速度太慢，参考以下：  

[网易开源镜像站](http://mirrors.163.com/.help/)  
centos，采用yum install命令安装依赖  
ubuntu，采用apt-get命令安装依赖  

### 2.3 puppeteer
执行 npm i 的时候，默认是会下载 chromium的，但是即使是墙外的环境，依然经常下载失败，所以就需要跳过下载：  
在 npm install 前设置环境变量(linux相关知识) `PUPPETEER_SKIP_CHROMIUM_DOWNLOAD=true`，比如在linux中可通过:  
```
env PUPPETEER_SKIP_CHROMIUM_DOWNLOAD=true
或者 PUPPETEER_SKIP_CHROMIUM_DOWNLOAD=true 设置
```  
但是这在服务器中不一定行，即使设置了环境变量有时候依然会报下载失败的错误。然后查到另一种方式 [puppeteer/install.js](https://github.com/GoogleChrome/puppeteer/issues/2270)： 

```
 if (process.env.PUPPETEER_SKIP_CHROMIUM_DOWNLOAD) { 
   console.log('**INFO** Skipping Chromium download. "PUPPETEER_SKIP_CHROMIUM_DOWNLOAD" environment variable was found.'); 
   return; 
 } 
 if (process.env.NPM_CONFIG_PUPPETEER_SKIP_CHROMIUM_DOWNLOAD || process.env.npm_config_puppeteer_skip_chromium_download) { 
   console.log('**INFO** Skipping Chromium download. "PUPPETEER_SKIP_CHROMIUM_DOWNLOAD" was set in npm config.'); 
   return; 
 } 
```  
原来代码里不仅判断env中 PUPPETEER_SKIP_CHROMIUM_DOWNLOAD 是否为true，也判断npm config中的该值,因此可以根目录新建 `.npmrc`文件，并设置该值：  

```
puppeteer_skip_chromium_download=1
```  

同时可以在该文件中看到：chromium的版本来自于`package.json`中的`puppeteer.chromium_revision`的值。即chromium的版本需要与puppeteer的版本保持一致。  
那既然不通过`npm install`下载了，就需要我们下载好，COPY到DOCKER镜像中。  
linux: https://commondatastorage.googleapis.com/chromium-browser-snapshots/index.html?prefix=Linux_x64/
mac: https://commondatastorage.googleapis.com/chromium-browser-snapshots/index.html?prefix=Mac/  

，找到对应之后下载zip压缩包解压，不同系统对应的executablePath也有区别。  

```
const browser = await puppeteer.launch({
  executablePath:"./chrome-linux/chrome"
  // executablePath:"./chrome-mac/Chromium.app/Contents/MacOS/Chromium"
});
```  

### 2.4 pm2
在本地开发的时候，可能开发环境通常都是 `npm start`来执行`index.js`，需要监听文件的话可能会使用`nodemon`，但是公司在服务器端统一采用的是更为强大的`pm2`进行应用进程的管理。找到这样[一篇博客](https://www.cnblogs.com/chyingp/p/pm2-documentation.html)，本质上都是在以`index.js`作为入口文件执行。  

## 3. 生产环境  
经过上诉的一些摸索之后，整个服务配置好就能部署到上线了。然后一切顺利本文结束。END...  

假的,Bug才是永恒  
1."Navigation Timeout ***"  
2.服务不稳定，多请求几次就挂了  

**第一个问题：请求超时**  
在本地是好的，怎么发到公司服务器上就坏了，于是开始进服务器查看。改服务器代码调试再重启服务(因为公司的一系列流程，本地 => gitlab => 发布平台发布很花时间)，吐槽下公司那个远程terminal真不好用。然后，并没有发现问题(debug能力太弱了)。无奈只能请教部门大神Y，(大神Y全程用vim，调试过程看到api不是看文档而是直接看源码，太强了)。Y还顺手展示了下远程node debug，打断点，慢慢得出结论是“可能是内外网环境问题，有些资源加载不出来导致服务最终超时”。然后也同时发现被截图的 `URL/index`首页能成功，`URL/detail`页会失败，因为这两个页面加载的资源不一致，所以针对不一致的资源进行排查(具体操作就是在服务器内 `curl` 各个资源域，查看能不能得到响应结果)。最终发现，多语言SDK加载的js文件出了问题，该js文件是外网地址，服务器环境下不能访问，本地开发的时候是可以的。。  
更换了多语言js加载方式都走内网之后，第一个问题就解决了。  

**第二个问题：服务不稳定**  
这个问题本地也能复现，多刷新几次页面就 500 错误。之前着急上这个服务，代码没有细看，现在处理了第一个问题，可以看看代码怎么写的了。  

```
render() {
    const browser = await puppeteer.launch();
    const page = await browser.newPage();
    ...
    await browser.close();
}
```  
原来每一次请求都会启动浏览器，打开页面，最后关闭。于是多刷新几次就开启多个浏览器消耗内存。  
基于此有了第一个方案：在第一个请求来临时开启一个Bowser，同时开启一个Page，之后每次来请求时判断有没有空闲的Page，没有则新增有则复用，请求结束后回收该Page用于处理新的请求。Browser只在服务关闭时进行销毁。  
然后在Code Review的时候Y大神又提了看法："最好是在页面初始化的时候开启Browser，同时初始化一批 page-pool"。解释是开启Brower为异步操作，多个请求时同时到来时需要保证唯一且第一个请求响应更慢。并且基于请求新建Page不可控，如果高并发依然会导致内存消耗请求卡死。  
确实说的在理，于是按照Y的建议，改写代码，服务启动的时候新建Browser同时新开10个Page，接下的请求会使用Page，如果高并发时没有多余Page则返回 `503` 表示服务不可用。  

处理好以上两个问题之后，系统现在算是平稳的在生产环境运行了。  

## 4. 总结  
几乎全程参与一次项目从开发到BUG解决到上线的过程，亲手写的代码不多，但过程不可谓之轻松。  

* 本地开发和生产环境是不一样的。生产环境相对更为"封闭"，稳定性要求也更高，自然代码质量逻辑不能疏忽。  
* 其实一开始还有个问题是"自维护容器"，因为 `chromium` 所依赖的那些字体库等，在公司标准的linux镜像里是不存在的。于是还尝试了申请机器进行"自维护",后来遇到发布错误找到公司发布团队，发现自维护其实是不被建议的，因为出了问题他们团队也不好排查错误。其次，最郁闷的是最后发现，之前也有团队做过这个"截图"的需求，同样用的这个库。他们当时因为依赖问题找到发布支持团队，团队给他们做了一个新的功能，只需要申请常规的容器，在发布的时候勾选“Chromium”环境即可，会自动安装那些依赖。(最终我们就是这样解决的)。  
* Debug及一些常规运维知识以及公司整体的规范约定(健康检查，NODE_ENV配置)也是需要掌握的,除了前端本身，服务器得了解，vim shell，网络知识得了解,不一定在你写代码的时候能用上，但是在排查错误的时候真的好用，而且高效。  



 
## 5. 其它  
### url-to-pdf-api 设置cookie 
```

page.setCookie({
   'value': 'light',
   'url': 'http:/***.com',
   'expires': Date.now() / 1000 + 100,
   'name': 'theme'
})


// url形式传入cookies=[{},{}]这种形式
&cookies[0][name]=theme&cookies[0][value]=light&cookies[0][url]=http://***.com&cookies[0][expires]=1554899743  
```

## 参考资料  
[1. centos安装使用puppeteer和headless chrome](https://segmentfault.com/a/1190000011382062)  
[2. centos安装puppeteer爬坑](https://luodao.me/post/puppeteer-pakeng.html)  
[3. linux速查手册](https://linuxtools-rst.readthedocs.io/zh_CN/latest/base/01_use_man.html)  
[4. node远程调试](https://qianduan.group/posts/59ee96e40119753d067b40e6)