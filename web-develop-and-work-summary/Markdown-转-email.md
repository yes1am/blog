## 前言

组里在搞 `code-review`,最初每次 `review` 结束都是发一个当天 `review` 结果的 `markdown` 链接,后来同事觉得邮件里只有一个链接太单一了，希望能在邮件内容中加上`review`结果,慢慢的希望邮件中包含`review出来的markdown`内容。  

## 1. shell方式

y同事先手起了一个应用，通过`gitlab-ci`的方式，在每次push的时候，执行`shell`脚本,读取`markdown`文件并通过`pandoc`将`markown`转为`html`，最后通过公司邮件服务发送出去，一切顺利。  

直到有一天，组长觉得程序员应该有基本的审美，原因是邮件格式很丑..  

## 2. 为什么丑

首先是因为，公司邮件服务是调用前需要申请邮件模板，由于偷懒(申请邮件服务比较麻烦)，我们使用了之前另外一个应用的邮件模板。该模板最外层被`<center>`标签包裹，且被限制了宽度(700px),导致邮件内容挤在一起。  

## 3. 遇到的问题及做的尝试

### 3.1 编码问题
最初不知道为什么内容会挤在一起，因为并不知道邮件模板的内容，基于前端的思维，我们希望能查看到`审查邮件的元素`。  

但，mac自带邮件查看不到邮件源码。outlook能看到，但是看到的是被`Quoted-printable`编码之后的内容，编码方式可通过邮件中的源码 `Content-transfer-encoding: quoted-printable`得到。  

在网上找到[邮件编码解码](http://www.nicetool.net/app/quoted_encode.html)工具，解析出了熟悉的html标签。  

### 3.2 样式及客户端兼容问题

对于样式问题，理所当然想到的是加`<style>`样式覆盖，权重不够就加`!important`。但还是会遇到样式与预期不符的情况，以及mac自带邮件与outlook表现不一致。甚至最终我们决定`markdown`文件全局使用`table`语法来写，以此解决对齐等问题。  

## 4. 思路

按理说上面的方案已经能用了，虽说不好看但是也不会太丑，直到轮到我`code review`了,发现：  

全局使用`table`语法写 `markdown` 太反人类了！  

于是决定正儿八经处理这个问题，陆续查到一些资料表明，行内样式的`html`对邮件兼容性是最为友好的。同时了解到一个周刊类的[邮件格式化工具](https://github.com/CtripFE/format-weekly)也是通过解析 `markdown` 添加行内样式的方式实现的。  

后来了解到 [inline-css](https://www.npmjs.com/package/inline-css) 可以帮我们做到将样式抽取变为行内样式。  

## 5. 最终实现

于是决定重写发送邮件的代码,采用 `node` 实现。  

### 5.1 gen.js

```js
const fs = require('fs')
const path = require('path')
const inlineCss = require('inline-css');
const request = require('request');

const showdown = require('showdown');
showdown.setOption('tables', true);  // true to convert table
const converter = new showdown.Converter();

const wrapContentWithHtml = str => `<!DOCTYPE html>
<html lang='en'>
<head>
  <style>
    // any style you want
  </style>
</head>
<body>
  ${str}
</body>
</html>`


const main = () => {
  return new Promise((res,rej) => {
    const mdContent = fs.readFileSync(path.resolve(__dirname, `./**.md`), {
      encoding: 'utf-8'
    });

    const wrappedHtml = wrapContentWithHtml(converter.makeHtml(mdContent));
  
    inlineCss(wrappedHtml, {
        url: '/'
      })
      .then(function (html) {
        res(html)
      }).catch(err => {
        rej(err)
      })
    })
}

main().then(html => {
    request({
      url:'邮件服务API',
      headers: {
        'Content-Type': 'application/json'
      },
      method:'post',
      body: JSON.stringify({
        content: html,
      })
    },(err,response,body) => {
      if(err){
        console.log("Error: ",err);
      } else {
        console.log("Send Mail Success!")
      }
    })
})
```

### 5.2 gitlab.yml  

```yaml
# one image with node environment， and container the npm packages like inline-css, showdown ..
image: image-***

test:
  script:
    - node ./gen.js
  only:
    variables:
      # do this job when commit message contains EMAIL, case sensitive
      - $CI_COMMIT_MESSAGE =~ /EMAIL/  
```

### 5.3 流程

即当push代码之后，执行ci，ci镜像中执行`node gen.js`,因此要求镜像拥有node环境，以及相关的`inline-css`等`npm package`, 所以在镜像中有`package.json`文件，并已经执行了`npm install`。  

### 5.4 注意node与shell路径的差别

```
第一种方式
node gen.js
/Users/jpsong/Desktop/code-review-sender/gen.js  // 相关路径

第二种方式
node code-review-sender/gen.js 
/Users/jpsong/Desktop/code-review-sender/gen.js  // 相关路径
```

第一种方式和第二种方式执行结果一样，即 `node` 只关心需要运行的js文件的路径。而shell，如ls， 是与在什么路径下执行该命令的有关。  

因此对于shell脚本，可以放置在根目录，然后在 `/build/project` 下执行 `/shell.sh`,依然是可以获取到 `/build/project`目录的内容。  

但如果`gen.js`文件在根目录下，在 `/build/project` 下执行 `node /gen.js` , 由于`/build/project 与 /gen.js`不是同一目录，所以会出现问题。  
所以需要在 `/build/project` 中执行 `node ./gen.js`,才能访问到 `/build/project` 的内容。  

## 参考资料

1. [邮件编码解码工具](http://www.nicetool.net/app/quoted_encode.html)  
2.  [inline-css](https://www.npmjs.com/package/inline-css)