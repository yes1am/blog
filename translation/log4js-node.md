[英文原版](https://log4js-node.github.io/log4js-node/index.html)  

## 1. API

### 1.1 configuration - log4js.configure(object || string)

有一个配置 `log4js` 的入口点，字符串的参数会被当做一个文件用来加载配置。配置文件应该是`JSON`格式的，并且包含了一个配置对象，你也可以直接给`configure`传递一个配置对象。  

配置应该在你的应用程序第一次加载 `log4js` 之后尽快出现。如果你不调用 `configure`, `log4js` 会使用 `LOG4JS_CONFIG`(如果有的话)作为默认的配置。默认的配置定义了一个输出源，该输出源会将有颜色的排版打印到标准输出中，但是定义的默认日志等级是 `OFF`, 这意味着没有日志将会被输出。  


如果你在使用集群，在worker进程和master进程中都调用了 `configure`，worker进程会为你的(日志)分类，以及任何你定义的自定义的等级选择适当的日志等级，输出源只在master进程中被定义，所以没有多个进程尝试向一个输出源写入的危险。不像之前的版本，现在在集群中使用 `log4js` 无需特殊的配置。  

配置对象至少需要定义一个输出源，以及一个默认的分类，如果配置文件不合法 `Log4js` 将会抛出错误。  

`configure` 方法必须返回一个 `log4js` 的配置对象。  

### 1.2 Configuration Object

属性:  

- levels (可选的，对象) - 用来定义日志等级，或者是重新定义用来覆盖已有的。这是一个 map 对象，使用 level 的名字作为 key(字符串，不区分大小写)，使用 `object` 作为值。这个 `object` 应该有两个属性，等级的值(整数)，以及颜色。日志等级用于表示日志消息的重要性，使用整数是为了能够排序。如果在配置文件中不特地指定，就会使用默认值。  
`ALL < TRACE < DEBUG < INFO < WARN < ERROR < FATAL < MARK < OFF` OFF 等级是用来关闭日志的，而不是实际的日志级别。比如，你永远不会 `logger.off('some log message')`. 除了默认的等级之外，还使用此处定义的等级，使用整数值用来决定于默认等级的关系。如果你定义了一个和默认等级相同名字的等级，在 `config` 中的整数值优先级更高。等级的名字必须以字母开头，并且只能有字母，数字，下划线。  

- appender(对象) - 输出源名字到输出源对象的映射。输出源必须要有一个 `type` 的属性，其它属性取决于输出源的类型。  
- categories(对象) - 目录名字到目录对象的映射，你必须定义 `default` 这个目录，当日志事件不匹配任何一个目录时，就会使用到 `default` 目录。目录的定义有两个属性:  
    - appenders(字符串数组)- 这个目录会使用到的输出源数组，一个目录至少需要一个输出源
    - level(字符串，大小写不敏感)-定义这个目录会发送给输出源的最小日志等级。如果设置成了`error`, 那么输出源只会接受 `error`,`fatal`,`mark`,其它的`info`,`warn`,`debug`，或者 `trace` 都会被忽略。
    - enableCallStack(布尔值，默认为false)- 将它设置为 true，那么这个目录的日志事件将会使用调用栈来生成行号和文件名。查看[排版模式](https://log4js-node.github.io/log4js-node/layouts.html)学习如何在你的输出源中输出这些值

- pm2(布尔值，可选)- 如果你在使用 `pm2` 的话，将它设置为true，否则的话日志将不会工作(你也需要安装 `pm2-intercom` 作为 pm2 的模块 `pm2 install pm2-intercom`)。

...

### Loggers- log4js.getLogger([category])

略

## 2. 示例代码

```
const log4js = require('./lib/log4js');
log4js.configure({
  appenders: {       // 输出源
    cheese: {
      type: 'file',      // 输出类型 [ console | stdout | ... ]
      filename: 'cheese.log'     // 输出到该文件
    }
  },
  categories: {
    default: {
      appenders: ['cheese'],
      level: 'error'
    }
  }
});

const logger = log4js.getLogger('cheese');
// 指定 cheese 目录，没有该目录，于是转向 default 目录。
// default目录指定最小输出的等级是Level,因此只有 error 和 fatal 会输出
logger.trace('Entering cheese testing');
logger.debug('Got cheese.');
logger.info('Cheese is Comté.');
logger.warn('Cheese is quite smelly.');
logger.error('Cheese is too ripe!');
logger.fatal('Cheese was breeding ground for listeria.');
```