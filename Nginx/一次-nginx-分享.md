Nginx 是一个**高性能**的 Web 和反向代理**服务器**。  

*首先，nginx 是一个服务器，和我们用 node 搭建的一样，不能因为它看起来是配置形式，就不把它当服务器。因此，它能够实现很多的功能。*

## 前言

迟到八个月的 nginx 分享整理，八个月的时间，换了工作，换了城市。  

```
过去十八岁，没戴表，不过有时间       --《陈奕迅 - 陀飞轮》
```

## 1. 预备知识

默认错误/访问日志地址: `/usr/local/var/log/nginx`  
默认配置文件地址: `/usr/local/etc/nginx`  

启动 nginx: `sudo nginx`  
关闭 nginx: `sudo nginx -s stop`  
重启 nginx: `sudo nginx -s reload`  
检测配置文件是否有语法错误: `sudo nginx -t`  
`include` 可用于引入配置文件，**便于配置文件的模块化**。  

**建议手动将涉及到的路径设置为绝对路径**  

*如果很明确相对路径是 **相对什么路径** 来说，或者只设置相对路径也能正常访问的话，也可以不设置*  
1. 比如日志路径设置为绝对地址，如 /usr/local/etc/nginx/logs/error.log, 即和配置文件在一个目录中，便于同时查看配置与日志。
2. 如设置 root 地址时，使用绝对路径确保能够正确访问。


### 1.1 host 文件

文件内容格式  

```
127.0.0.1	localhost
255.255.255.255	broadcasthost
::1             localhost
127.0.0.1 www.aaaa.com
127.0.0.1 www.bbbb.com
```

host 文件内容如上所示:  
1. 前面是ip，后面是域名，localhost 也是域名
2. ip 不能添加端口，**不能直接让某个域名指向某个IP+端口**, 又因为默认端口是 80，以上的配置相当于让 www.bbbb.com 指向 127.0.0.1:80

如配置 `127.0.0.1:8081 www.bbbb.com`,企图让访问 www.bbbb.com 直接访问 127.0.0.1:8081 是无效的。  

## 2. 常见问题及解决办法  

### 2.1  403 没有权限

[参考资料：chmod -R 777 报 403](https://www.v2ex.com/t/194046)  

```
location / {
    root  /Users/[username]/Desktop/express-react-boilerplate/dist;
}
```

假设 nginx 中的 location 设置如上，则即使在 Desktop 目录中执行`chmod -R 777 express-react-boilerplate`，也是没有效果的。  

原因是因为需要保证从 `/User/[username]/Desktop/express-react-boilerplate/dist` 都是有权限的，这样才能够访问 `dist` 目录。  

经过测试发现`Desktop`的权限默认是`700`，因此只要执行`chmod 701 Desktop`即可。[参考:linux中目录的x权限](http://cn.linux.vbird.org/linux_basic/0210filepermission.php#filepermission_dir)  

### 2.2 如何确认是否进入了某个 location

```nginx
log_format  locationLog  'locationLog: $remote_addr';
server {
    listen       80;
    server_name  localhost;

    location /aaa.html {
        access_log  /usr/local/etc/nginx/logs/aaa.log  locationLog;
        root /usr/local/etc/nginx/test/log;
    }

    location /bbb.html {
        access_log  /usr/local/etc/nginx/logs/bbb.log  locationLog;
        root /usr/local/etc/nginx/test/log;
    }
}
```

可能有些情况，我们不确定请求是到达了 `aaa.html` 还是 `bbb.html` 的 location，又或者可能根本没有进入 nginx，这时候应该怎么确认呢？  

其实，我们可以在里面定义 **当前 location** 的 `access_log` 文件及日志格式, 只有在进入当前 location 时，才会在对应的日志文件输出内容。

## 3. 配置文件

默认的配置文件:  

```nginx
#user  nobody;
worker_processes  1;

#error_log  logs/error.log;
#error_log  logs/error.log  notice;
#error_log  logs/error.log  info;

#pid        logs/nginx.pid;

events {
    worker_connections  1024;
}

http {
    include       mime.types;
    default_type  application/octet-stream;

    #log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
    #                  '$status $body_bytes_sent "$http_referer" '
    #                  '"$http_user_agent" "$http_x_forwarded_for"';

    #access_log  logs/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    #keepalive_timeout  0;
    keepalive_timeout  65;

    #gzip  on;

    server {
        listen       8080;
        server_name  localhost;

        #charset koi8-r;

        #access_log  logs/host.access.log  main;

        location / {
            root   html;
            index  index.html index.htm;
        }

        #error_page  404              /404.html;

        # redirect server error pages to the static page /50x.html
        #
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }

        # proxy the PHP scripts to Apache listening on 127.0.0.1:80
        #
        #location ~ \.php$ {
        #    proxy_pass   http://127.0.0.1;
        #}

        # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
        #
        #location ~ \.php$ {
        #    root           html;
        #    fastcgi_pass   127.0.0.1:9000;
        #    fastcgi_index  index.php;
        #    fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
        #    include        fastcgi_params;
        #}

        # deny access to .htaccess files, if Apache's document root
        # concurs with nginx's one
        #
        #location ~ /\.ht {
        #    deny  all;
        #}
    }

    # another virtual host using mix of IP-, name-, and port-based configuration
    #
    #server {
    #    listen       8000;
    #    listen       somename:8080;
    #    server_name  somename  alias  another.alias;

    #    location / {
    #        root   html;
    #        index  index.html index.htm;
    #    }
    #}

    # HTTPS server
    #
    #server {
    #    listen       443 ssl;
    #    server_name  localhost;

    #    ssl_certificate      cert.pem;
    #    ssl_certificate_key  cert.key;

    #    ssl_session_cache    shared:SSL:1m;
    #    ssl_session_timeout  5m;

    #    ssl_ciphers  HIGH:!aNULL:!MD5;
    #    ssl_prefer_server_ciphers  on;

    #    location / {
    #        root   html;
    #        index  index.html index.htm;
    #    }
    #}
    include servers/*;
}
```

按模块划分:  

```nginx
## 全局块

## events块
events {
}

## HTTP块
http {
    ## http全局

    ## server块
    server {
        ##server全局

        ## location块
        location {
        }

        location {
        }
    }

    server {
    }
}
```

## 4. 全局块

### 4.1 user

`user  nobody;`  

user 用于指定用户和组，nignx 将使用这个用户和组来启动 worker 进程  

### 4.2 work_processes
`worker_processes 1;`  

用于指定 nginx worker 的进程数。  

 [参考：master进程与worker进程](https://blog.csdn.net/mine_song/article/details/56678736)  

### 4.3 error_log

`error_log  logs/error.log;`  
`error_log  logs/error.log  notice;`  
`error_log  logs/error.log  info;`  


用于指定日志的存放路径及级别，常见的错误日志级别有:  
[debug | info | notice | warn | error | crit | alert | emerg]  

### 4.4 pid

`pid        logs/nginx.pid`  

用于指定 [master进程PID](http://nginx.org/en/docs/control.html) 的存放位置  

可在 nginx 关闭的时候，执行 `sudo nginx -s stop`， 即通过**相反**(关闭状态，继续执行关闭)的命令，主动使 nginx 报错:  
> nginx: [error] open() "/usr/local/var/run/nginx.pid" failed (2: No such file or directory)  

可以知道当前 pid 存放的位置。

## 5. events

### 5.1 accept_mutex

`accept_mutex on;`  

用于设置网路连接序列化，防止[惊群现象](https://www.cnblogs.com/Anker/p/7071849.html)发生。  

### 5.2 multi_accept

`multi_accept on;`  

用于设置一个进程是否同时接受多个网络连接  

### 5.3 use

`use epoll;`  

用于指定事件驱动模型，常见的事件驱动模型有 `select`, `poll`, `kqueue`, `epoll`, `resig`, `/dev/poll`, `eventport`  

### 5.4 worker_connections

`worker_connections  1024;`  

用于设定最大连接数，默认为512。  
[参考：max_client计算](https://blog.csdn.net/liupeifeng3514/article/details/79018236)  

## 6. http

### 6.1 MIME TYPE

`include       mime.types;`  

mime.types 示例:  

```
types {
    text/html                                        html htm shtml;
}
```

> MIME 配置用于告诉服务器，什么文件的后缀，应该返回怎样的 Content-Type。

以上配置表示 nginx 遇到以 html, htm, shtml 结尾的文件请求时，则设置 HTTP 请求的响应头的 `Content-Type` 为 `text/html`, 这样浏览器就能根据 Content-Type 正确处理文件。  

但有时候一些文件响应头中即使没有Content-Type，浏览器也能正常处理。[参考：浏览器嗅探](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/X-Content-Type-Options)。  

我们可以测试下, MIME-TYPE, 新建如下的 nginx 配置文件，并在 nginx.conf 中 include 进去:  

```
types {
    t                                    html htm shtml;
}

server {
    listen 80;
    server_name localhost;

    location / {
        // 添加请求头，关闭浏览器嗅探
        add_header X-Content-Type-Options nosniff;
        root /usr/local/etc/nginx/test/mimetype;
    }
}
```

我们通过设置一个错的 **Content-Type: t**, 再关闭浏览器的嗅探行为(错误 Content-Type 和关闭浏览器嗅探必须都设置好)，这样浏览器就不能够正确解析 html 标签，只能将 html 标签作为字符串渲染出来。  

关于 MIME-TYPE，express 中同样有相应的实现，具体的引用链为: [express-static](https://github.com/expressjs/express/blob/master/lib/express.js)
=>
[server-static send](https://github.com/expressjs/serve-static/blob/master/index.js)
=>
[send mime](https://github.com/pillarjs/send/blob/master/index.js) =>
[mime](https://github.com/broofa/node-mime/blob/master/types/standard.js)  

### 6.2 default_type

`default_type  application/octet-stream;`  

用于设置默认的 Content-Type, 当 mime-type 中找不到文件后缀对应的 Content-Type 时，便会使用的默认数据流格式。  

### 6.3 log_format

```
log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                     '$status $body_bytes_sent "$http_referer" '
                     '"$http_user_agent" "$http_x_forwarded_for"';
```

以上为定义了一个名为 `main` 的 **日志格式** 变量，变量的值为 `'$remote_addr...'`，其中 `$remote_addr` 等是内置变量。  

更多内置变量[可查看](http://nginx.org/en/docs/varindex.html)  

> 此时内置变量中的 remote_addr  是客户端的 IP，因为此时 nginx 是作为服务器来使用的。  
> 而当我们访问[百度](https://www.baidu.com/)页面时，Chrome Network 中的 remote address 对应是百度服务器的IP,即服务端的 IP。  
> 同时，如果我们本地使用正向代理，则 remote address 为代理服务器的地址(可查看网络代理中的 pac 文件)。  

我们可以通过加入其它变量，或者删除一些变量，来改变日志格式。  

### 6.4 access_log

用于指定 access.log 存放的路径，已经使用怎样的日志格式。

`access_log  logs/access.log  main;`  

如以上代码表明，access.log 将采用刚刚定义的 ***main*** 日志格式变量的日志格式  

### 6.5 sendfile

`sendfile        on;`  

用于指明是否开启 sendfile 系统调用。  

[参考资料](http://xiaorui.cc/2015/06/24/%E6%89%AF%E6%B7%A1nginx%E7%9A%84sendfile%E9%9B%B6%E6%8B%B7%E8%B4%9D%E7%9A%84%E6%A6%82%E5%BF%B5/)

> 结论: 当 Nginx 是一个静态文件服务器的时候，开启 SENDFILE 配置项能大大提高 Nginx 的性能。  
> 但是当 Nginx 是作为一个反向代理来使用的时候，SENDFILE 则没什么用了。

### 6.6 tcp_nopush

`tcp_nopush     on;`  

用于设置 socket 的参数 tcp_option, 仅在 sendfile 为 on 时有用。  

### 6.7 keepalive_timeout

`keepalive_timeout  65;`  

[参考资料](https://blog.csdn.net/huangjin0507/article/details/52396580)

设置心跳包的间隔时间，TCP 内嵌的一个心跳包，用于服务端探测客户端是否存活.  

### 6.8 gzip

`gzip  on;`  

用于指定是否开启 gzip 压缩。  

当开启时, Response Header 会新增以下项:  

```
Content-Encoding: gzip
```

同时传输的体积会变小。

## 7. server

建立一个虚拟的服务器(和我们的 node 服务器一样，也需要占用端口).  

1. 如果 node 已经有一个服务器监听 8080 端口，nginx server 再次监听 8080 端口会报错，nginx 会启动失败。  
2. 如果 nginx 先监听 8080 端口，再起 node 服务监听 8080 端口是无效的，访问 8080 依然是 nginx 服务器在起作用。(经过测试，似乎有时候会有意外，node 服务器可能会起效)  


### 7.1 listen

`listen 80;`  

指定当前服务器监听的端口。  

### 7.2 server_name

`server_name localhost`  

用于指定虚拟主机的域名  

```js
server {
    listen 8081;
    server_name www.bbbb.com;

    location / {
        root /usr/local/etc/nginx/test/server/serverB;
    }
}

server {
    listen 8081;
    server_name www.aaaa.com;

    location / {
        root /usr/local/etc/nginx/test/server/serverA;
    }
}
```

如上所示，我们可以针对 **一个端口添加两个 server**。前提是这两个服务的 server_name **不存在交集**，否则会失败。  

假设 host 文件如下，那么我们访问 www.bbbb.com:8081 即能访问到 `serverB`, **因为端口是 8081, 不是 80，因此不能省略端口**。  

```
127.0.0.1 www.aaaa.com
127.0.0.1 www.bbbb.com
```

### 7.3 如何实现负载均衡

[参考:负载均衡及其策略](http://nginx.org/en/docs/http/ngx_http_core_module.html#location)  
```
upstream balanceServer {
    server localhost:3004 weight=1;
    server localhost:3005 weight=2;
    server localhost:3006 weight=3;
}

server {
    listen 80;
    server_name localhost;

    location / {
        proxy_pass http://balanceServer;
    }
}
```

以上设置了一个负载均衡服务器，会将请求 80 端口的请求，转发到 3004,3005,3006 三个端口对应的服务器上。  

服务器权重分别为 1，2，3， 于是接收到请求的概率依次为 1/6, 2/6, 3/6。

### 7.4 location

location 用于处理具体的请求路径  

说到这个，我们需要了解下 location 的匹配规则 [参考资料：location匹配规则](http://nginx.org/en/docs/http/ngx_http_core_module.html#location)：  

1. location 可以使用前缀字符串，或者正则进行定义。   
2. 正则表达式 `~*` 代表不区分大小写，`~` 代表区分大小写。  
3. 当请求到来时，为了找到合适的 location，nginx 首先检查使用前缀字符串定义的 location，其中最长匹配的前缀字符串会**被选择，并记录下来**。
4. 然后使用正则表达式进行匹配，顺序是根据他们在配置文件中出现的顺序  
5. 只要一找到匹配的正则表达式，就会使用该正则对应的规则。
6. 如果没有找到匹配的正则，就会使用 **第3步** 记录下来的那一项规则。

location 块可以嵌套，以下是一些注意事项:  

对于不区分大小写的操作系统，如 macOS, Cygwin，匹配前缀字符串时不区分大小写(0.7.7 版本，*这里原文只有 0.7.7,我理解为 0.7.7 版本开始*)，然而字符的比较只限于 "一字节" 的语种 (*个人理解: 区分于中文一个字大于一个字节这种情况吧，难道还能用中文作为 location*)。  

正则表达式也可以包含捕获组(0.7.40 版本开始)，从而这些部或者可以在之后的指令中使用。  


如果最长匹配前缀是用 `^~` 修饰的，那么后续就不会再使用正则表达式进行检查了(前面提到，最长前缀字符串匹配之后，会被记录下来，而不是立即使用，还得通过正则表达式的检查)。  

另外，使用 `=` 修饰符可以定义一个 **精准匹配前缀字符串**. 如果精准匹配成功了，不会进行后续的搜索(将不会进行正则表达式的检查)。例如，如果对于 `/` 的请求非常频繁，定义 `location = /` 将会加速这些请求，因为只要一匹配到这一项就会停止继续搜索 location.这样的 loction 显然不能嵌套其它的 location。  

> 在 0.7.1 到 0.8.41 版本中，即使一个请求匹配到了没有 `=` 和 `^~` 修饰符的前缀字符串,也会停止搜索，不会检查正则表达式。  

让我们用例子来证明以上的论点:  

```nginx
location = / {
    [ configuration A ]
}

location / {
    [ configuration B ]
}

location /documents/ {
    [ configuration C ]
}

location ^~ /images/ {
    [ configuration D ]
}

location ~* \.(gif|jpg|jpeg)$ {
    [ configuration E ]
}
```

`/` 的请求会匹配到 configuration A  

`/index.html` 的请求会匹配到 configuration B  

`/documents/document.html` 的请求会匹配到 configuration C  

`/images/1.gif` 请求会匹配到 configuration D (**使用了 ^~ 前缀字符串匹配，将不会继续检查 configuration E**)  

`/documents/1.jpg` 请求会匹配到 configuration E (**首先最长前缀字符串匹配到配置 C，这时候会继续检查正则，之后正则匹配到配置 E**)  

> 小技巧: 如果 configuration D 和 configuration E 基本一致的情况下，我们可以通过 `add_header headerName headerD / headerE;`， 即添加自定义的 header，然后在 chrome 中检查响应头，来确认究竟是配置 D 生效，还是配置 E 生效。

`@` 前缀定义了一个**命名**的 location, 这样的 location 不用于常规的请求处理，而是用来请求重定向的。它们不能被嵌套，当然也不能包含其它的嵌套 location。  

如果一个 location 定义为一个前缀字符串，并且以 `/` 结尾，并且请求被 `proxy_pass`, `fastcgi_pass`, `uwsgi_pass`，`scgi_pass`, `memcached_pass` 或者 `grpc_pass` 其中之一处理时，那么就会执行特殊的处理流程。遇到字符串相同，但是没有末尾的 `/` 的请求时，会产生一个 301 重定向，并重定向至 **添加 / 之后的 url**  

假设有以下的配置
```
location /aaa/ {
    proxy_pass http://localhost:8080
}
```
此时请求 `/aaa`, 那么会先产生一个 301 重定向，重定向到 `localhost:/aaa/`, 之后经过 proxy_passs, 返回 `http://localhost:8080/aaa/ 的内容`  

如果这种行为不是你所期望的，那么可以定义一个更为精确的 URL location 规则，就像这样:  

```nginx
location /user/ {
    proxy_pass http://user.example.com;
}

location = /user {
    proxy_pass http://login.example.com;
}
```

此时遇到 `/user` 会直接转为 `login.example.com`, 而不是经过重定向，到 `user.example.com`

**总结:**  

* = 表示精确前缀字符串匹配，只有请求的url路径与后面的字符串完全相等时才会命中,命中后不再继续后续查找。
* ^~ 表示前缀字符串匹配，如果该符号后面的字符是当前请求的最佳匹配，那么会采用该规则，不再进行后续的查找。
* ~ 表示该规则是使用正则定义的，区分大小写。
* ~* 表示该规则是使用正则定义的，不区分大小写。  

#### 7.4.1 root

`root html/root;`  

用于设置 **响应请求资源时的根目录**，例如以上的设置，访问 `localhost/aa.html` 实际上访问的是服务器中 `html/root/aa.html`  

#### 7.4.2 index

`index  readme.html index.html;`  

用于指定默认首页的查找顺序，默认找 readme.html,找不到再找 index.html, 再找不到则可能报错。  

```nginx
server {
    listen       80;
    server_name  localhost;

    location / {
        root   /usr/local/etc/nginx/test/index;
        index  readme.html index.html index.html;
    }
}
```

设置以上配置，则当以 `localhost` 直接访问的时候，默认会找 `/usr/local/etc/nginx/test/index/readme.html` 文件，找不到再找 `/usr/local/etc/nginx/test/index/index.html` 文件。  

#### 7.4.3 autoindex

`autoindex on;`  

设置，当没找到默认首页时，是否显示目录索引。  

```nginx
server {
    listen       80;
    server_name  localhost;

    location / {
        autoindex on;
        root   /usr/local/etc/nginx/test/index;
        index  notexist.html;
    }hi 
}
```

设置以上配置，如果没有 `/usr/local/etc/nginx/test/index/notexist.html` 文件时，且 `autoindex` 为 `on`, 则会将 `/usr/local/etc/nginx/test/index` 目录下的所有文件以文件夹显示出来。  

而如果不存在默认文件，且 `autoindex` 不为 `on`，那么 nginx 会返回 403.  

#### 7.4.5 proxy_pass

`proxy_pass http://localhost:3005;`  

用于将请求反向代理到 http://localhost:3005  

```
server {
    listen       80;
    server_name  localhost;
    
    location / {
        // 反向代理到前端
        proxy_pass http://localhost:3004;
    }

    location /api {
        // 反向代理到后端
        proxy_pass http://localhost:3005;
    }
}
```

proxy_pass 的使用场景之一就是**解决跨域问题**，如上，我们通过**请求的前缀**区分前后端服务，对外统一是 `localhost`, 对内根据前缀转发到不同的服务去。从而避免跨域的问题。 

#### 7.4.6 rewrite

rewrite 用于重写 url：

```
location ^~ /aaa {
    root /usr/local/etc/nginx/test/location;
    rewrite /aaa /images break;
    proxy_pass http://127.0.0.1:8080;
}
```

如以上配置，则访问 `localhost/aaa` 实际会访问 `http://127.0.0.1:8080/images`  

#### 7.4.7 add_header 

add_header 用于添加响应头  

```
location ^~ /aaa {
    root /usr/local/etc/nginx/test/location;
    proxy_pass http://127.0.0.1:8080;
    add_header xxx images; 
}
```

以上配置，则返回的结果中，会包含响应头: `{ xxx: images }`


## 8. nginx模块

[前端可以用nginx做什么？](https://juejin.im/post/5bacbd395188255c8d0fd4b2#heading-2)  

[nginx module](http://nginx.org/en/docs/)  

[第三方模块](https://www.nginx.com/resources/wiki/modules/index.html)

[windows上编译nginx](http://nginx.org/en/docs/howto_build_on_win32.html)  

即 nginx 会默认内置一批常用模块，当有些功能内置模块不支持时可添加第三方模块并重新编译 nginx, 以支持新的功能，如 **动态切图服务** 等。

## 参考资料

1. [前端开发者必备的nginx知识](https://zhuanlan.zhihu.com/p/65393365)
2. [Nginx基本配置备忘](https://zhuanlan.zhihu.com/p/24524057)
3. [Nginx与前端开发](https://juejin.im/post/5bacbd395188255c8d0fd4b2)
4. [nginx中文官方文档](http://shouce.jb51.net/nginx-doc/Text/1.1_overview.html)
5. [反向代理与正向代理](https://juejin.im/post/5bacbd395188255c8d0fd4b2#heading-2)
6. [mac 如何安装 nginx](https://coderwall.com/p/dgwwuq/installing-nginx-in-mac-os-x-maverick-with-homebrew)