[英文原版](http://stritti.github.io/log4js/docu/users-guide.html)

## 1. 介绍

Log4js 是一个小型但有用的，用来在你的脚本中打印事件的JS库，在代码中通常是不会使用 `alert(debug message)` 的。另一方面像 `venkman` 的 debuger 工具也可以帮助你 debug，当然只是在你本地电脑上 debug ，并且是在你使用 FireFox 浏览器的时候。  

但是如果你是发布自己的web应用，就会出现大的问题。可能你有一个团队的测试人员测试你的网站，但是他们对于调试JS没有任何经验。  

Log4js可以解决这些问题，可能更多。你只需要简单的将 Log4js 添加到你的脚本中，这个库会打印你配置的事件。安装和配置会在下一个章节继续描述。  

使用 Log4js，你不需要修改完整的脚本如果你是在生产环境中使用 `alert-dialog`，你只需要配置你是否需要log并且在生产环境中应该怎么打印日志，没有更多的了。  

## 2. 下载和安装

为了使用 Log4js ，你需要从[下载部分](https://github.com/stritti/log4js/zipball/master)下载这个库，这个库有两种不同的格式，取决于你，你喜欢 `zip` 格式还是 `tar.gz` 格式，内容都是一样的。  

保持目录结构  
将库压缩包进行解压，但是保留库的目录结构，在解压之后，你会看到以下的目录结构:  

- log4js-{version}
    - docs
        - api
            - index.html (API文档)
        - users-giude.pdf(用户手册)
    - example
    - lib
        - lib4js-lib.js(压缩版的 Log4js 库)
    - src
        - js
            - log4js(未压缩版)

两个版本的 Log4js  
会有两个版本的 Log4js，一个文件在 `lib` 目录下的叫做 `lib4js-lib.js`, 是压缩版本的 `Log4js`, 这可以用于生产环境以降低带宽。log4js 文件在 `src/js` 目录的是 log4js 的源码，未压缩具有可读性。如果你想理解 log4js 内部是怎么工作的话，使用这个库。  

## 3. 使用
创建一个新的 JS 项目，或者用你已有的项目，把 log4js.js 文件拷贝进去，对于打印日志来说，不需要别的文件了。  

在添加了 log4js.js 文件之后，你需要将它引入到你的脚本中，使用以下的代码：  
`<script src="js/log4js.js" type="text/javascript"><script>`  

查看以下示例`HTML`文件：  

```
<!DOCTYPE html>
<html>
<head>
  <title>Log4js example</title>
  <script src="./js/log4js.combined.js"></script>
</head>
<body>
  
</body>
</html>
```
现在日志已经被初始化了，`Log4js` 提供了这样的一个方法: 你可以为你的日志传递一个目录,然后你得到一个日志的实例：  

`var myLogger = new Log4js.getLogger("myCategory")`  

注意:  
日志器被缓存起来了，如果你第二次请求一个 logger，它会返回第一次的那个日志器。  

现在日志器能够像下面的代码片段一样使用:  

```
myLogger.info('an info');
myLogger.warn('a warning');
myLogger.error('an error');
```

警告  
如果你测试以上的代码片段，什么都不会发生，你可能会疑惑为什么。其实问题在于 `logger` 并没有被配置正确，`Logging` 默认是被禁止的，因此你的脚本的行为和没有使用 `Log4js` 之前没有什么不一样。  

## 4. 配置
现在你的脚本中已经有日志了，但是这些时间并不会被显示出来，`Logger` 需要被正确配置。  

### 4.1 定义日志等级
首先日志的等级需要被定义，有很多种日志的等级，他们就像 `JavaTM的实现log4j`一样:  

Log4js 日志等级  

| Log4js.Level(等级) | Description(描述) |
|---|---|
| OFF | 不打印日志 |
| FATAL | 打印 `fatal` (重大的)错误|
| ERROR | 打印 `errors` (错误) |
| WARN | 打印 `warning` (警告) |
| INFO | 打印 `info` (提示信息) |
| DEBUG | 打印 `debug info` (debug信息)|
| TRACE | 打印 `trace` (跟踪信息) |
| ALL | 任何东西都会被打印 |

这些等级是包含的，因此如果你把日志等级设置为 `WARN`,那么所有的 `warning`,`errors` 和 `fatals` 都会被打印:  

`myLogger.setLevel(Log4js.Level.WARN)`

如果想要所有的东西都被打印，使用:  

`myLogger.setLevel(Log4js.Level.ALL)`

### 4.2 定义输出源

在定义了日志等级之后，我们需要设置输出源。输出源是用来定义 `logging event` 被怎样处理，怎样被展示以及怎样被储存。  

对于第一个使用 `ConsoleAppender` 的例子,(其它的输出源之后再说), `ConsoleAppender` 可以用于两种模式:  

- 当前页面行内打印
- 独立的打印窗口

行内的打印默认是被隐藏的，可以通过 `ALT+D` 来激活，`ConsoleAppender` 可以通过以下的代码被添加:  

`myLogger.addAppender(new ConsoleAppender(true))`

现在， `logger` 可以被使用了，再次尝试以下的代码:  

```
myLogger.info('an info');
myLogger.warn('a warning');
myLogger.error('an error');
```

示例的html代码应该像以下内容一样:  

```
<!DOCTYPE html>
<html>
<head>
  <title>Log4js example</title>
  <script src="./js/log4js.combined.js"></script>
</head>
<body>
  <script>
    var myLogger = new Log4js.getLogger("myCategory")
    myLogger.setLevel(Log4js.Level.WARN)
    myLogger.addAppender(new Log4js.ConsoleAppender(true))
    myLogger.info('an info');
    myLogger.warn('a warning');
    myLogger.error('an error');
</script>
</body>
</html>
```

如果你通过 `ALT+D` 打开控制台，你应该可以找到像下面图片一样的日志事件。  

恭喜，现在你已经可以在你的项目中使用 `Log4js` 了，接下来的章节只在你想了解更多如何配置 `Log4js` 的细节时需要阅读。  


## 5. 进阶

Log4js 是一个灵活的 `API` ，你可以在你的项目中定义几个不同的输出源。就像 `addAppender` 方法暗示的那样，是可以给一个 `logger` 添加多个输出源的。  

例如，你可以设置一个输出源打印到控制台上，另一个输出源将日志事件通过 `XmlHttpRequest` 发送到你的服务器。  

日志信息的格式也是可以配置的，但是这取决于使用怎样的输出源。例如，如果你使用 `AjaxAppender`, 你可以发送 `JSON` 或者 `XML` 格式的事件来让服务器去处理，更多的细节请查看[API 文档](http://stritti.github.io/log4js/apidocs/index.html) 以及下一章节。  

### 5.1 类图
- 略

### 5.2 使用 AjaxAppender

Log4js 也支持通过 `XmlHttpRequest` 异步的发送日志事件到服务器。这个输出源给予你在生产环境中更灵活的去打印 `JavaScript` 方法。因为在生产环境下你不能够访问到浏览器，你可以激活 `AjaxAppender` 来发送异步的日志信息到服务器中。在服务器中， logs 可以被例如，一个简单的 `JSP` 文件进行处理。就像在示例中的通过 log4js 来存储他们。  

`AjaxAppender` 需要一个 `URL` 来将日志发送出去，因为安全的原因，只能是同个服务器的相对`URL`:  

```
var ajaxLog = new Log4js.getLogger("ajaxTest");
ajaxLog.setLevel(Log4js.Level.ALL);
var ajaxAppender = new AjaxAppender("./log4j.jsp");
ajaxAppender.setThreshold(5);
ajaxLog.addAppender(ajaxAppender);
```

在以上的例子中, `AjaxAppender` 发送日志事件到相对的URL `./log4.JSP`, 定义了一个阈值`threshold`,输出源收集这些日志事件直到阈值到了，然后将所有收集的日志一次性发出去。  

默认的 `AjaxAppender` 使用 `XMLLayout` 来将日志事件转化为 `XML` 格式，但是如果你喜欢的话你也可以使用其他的排版比如 `JSONLayout` 或者 `HTMLLayout`  

注意  
这个布局非常像 `log4j` 的 `XML` 格式，`log4js.xsd` 定义了 `eventSet` 以及 服务器的返回。  
与 `log4j` 的 XML排版相比，`Log4js` 也提供了客户端浏览器的`id`,以及 `referer URL` 来标识请求源。另一方面，一些高级的特性比如 `NDC` 当前 `Log4js` 是不支持的。  

### 5.3 性能

为了改善你的软件的性能，更好的方式是用对应的 `isXXXEnabled()` 方法来包裹你的打印日志语句，在打印之前评估是否所要求的日志等级是开启的:  

```
if(myLogger.isDebugEnabled()) {
    myLogger.debug(...);
}
```

如果`debug`被关闭了，在构建可能的复杂信息之前发现 debug 等级被关闭了，系统将运行的更快。但是，这只在信息比较复杂的时候起作用，而不是一个简单的字符串。否则的话，源码就会因为条件判断语句变得冗余，影响阅读性。  

### 5.4 创建新的输出源和排版

已经有了一部分的输出源和排版类，但是可能你需要另外的，在`Log4js` 中没有实现的特性。对于这种情况，是有机会增强对应的API以支持特殊的特性。  

分享你的扩展  
如果一个特性能够被其他的开发者所使用，请通过补丁的形式分享你的扩展，详情查看[项目开发](http://stritti.github.io/project/index.html) 章节。  

## 6. 参考
- [Log4js GitHub project](https://github.com/stritti/log4js)
- [Log4js WIKI](https://github.com/stritti/log4js/wiki)
- [Apache Logging Website](http://logging.apache.org/)