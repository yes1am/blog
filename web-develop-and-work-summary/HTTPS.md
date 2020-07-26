# HTTPS
## 1. 加密算法
### 1.1 对称加密
即加密和解密都是使用同一个密钥，常见的对称加密算法：**DES, 3DES 和 AES**  

优点：

- **计算量小、加密速度快、加密效率高，适合加密比较大的数据。**

缺点：

- 发送方和接收方都需要知道密钥，因此存在密钥的传输，但传输的过程中，**无法保证密钥不被截获**。因此安全性得不到保证
- 每对用户每次使用对称加密算法时，都需要使用别人不知道的唯一密钥，使得收发双方所拥有的钥匙数量急剧增长，**密钥管理成为双方的负担**


<br />示例:<br />![对称加密](https://cdn.nlark.com/yuque/0/2020/png/286950/1595603690682-ae92abf8-c1be-4cf4-9846-a6280edb2e74.png#align=left&display=inline&height=401&margin=%5Bobject%20Object%5D&name=%E5%AF%B9%E7%A7%B0%E5%8A%A0%E5%AF%86&originHeight=589&originWidth=800&size=0&status=done&style=none&width=544)<br />

<a name="MNT5h"></a>
### 1.2 非对称加密

<br />加密和解密使用不同的密钥，公钥加密之后，只有用私钥才可以解开，私钥加密之后，只有用公钥才可以解开

常用算法:  RSA 算法

基本过程：

1. 甲方生成一对密钥，并将其中一把作为公钥对外公开
1. 得到该公钥的乙方，将自己的机密信息加密(如客户端的key)，再发送给甲方
1. 甲方再用自己的私钥，对乙方的加密信息解密出来



**优点:**

- 加密和解密使用**不同的钥匙**，私钥不需要通过网络进行传输，**安全性很高**。
- **加解密速度比对称加密慢**

**<br />示例：**<br />![非对称加密算法](https://cdn.nlark.com/yuque/0/2020/png/286950/1595636956838-abb59037-a1d1-4771-8ba8-880175ae257c.png#align=left&display=inline&height=472&margin=%5Bobject%20Object%5D&name=%E9%9D%9E%E5%AF%B9%E7%A7%B0%E5%8A%A0%E5%AF%86%E7%AE%97%E6%B3%95&originHeight=547&originWidth=800&size=0&status=done&style=none&width=690)<br />在对称加密中，传递的密钥如果被拦截，就可以用来解析或者伪造双方的信息，安全性不高。<br />而在非对称加密中：

1. 第2步中服务器的公钥 key 是公开的，无所谓拦截
1. 第4步中的加密后的 KEY，必须通过**服务端用私钥**才能解密，因此拦截了也是无用了。
1. 第5步中没有密钥key的传输，传输的都是加密后的内容，只有**客户端和服务器能使用密钥KEY解析信息**

**因此，保证了非对称加密的安全性**<br />

## 2. HTTPS 流程

<br />**HTTP 和 HTTPS 的关系：**HTTPS (Hypertext Transfer Protocol Secure) 是**基于 HTTP 的扩展**，在 HTTPS 中，原有的 HTTP 协议会得到 **TLS (安全传输层协议) 或其前辈 SSL (安全套接层) 的加密**。因此 HTTPS 也常指 HTTP over TLS 或 HTTP over SSL。<br />
<br />![http和https](https://cdn.nlark.com/yuque/0/2020/png/286950/1595671106857-d99f47cf-06f7-44ca-935e-fcedc7cb2af3.png#align=left&display=inline&height=523&margin=%5Bobject%20Object%5D&name=http%E5%92%8Chttps&originHeight=523&originWidth=800&size=0&status=done&style=none&width=800)<br />
<br />**流程：**<br />
<br />！![完整示例](https://cdn.nlark.com/yuque/0/2020/png/286950/1595637331346-468a0ce6-f9f9-4099-b706-0962d080ab42.png#align=left&display=inline&height=669&margin=%5Bobject%20Object%5D&name=%E5%AE%8C%E6%95%B4%E7%A4%BA%E4%BE%8B&originHeight=669&originWidth=800&size=0&status=done&style=none&width=800)<br />https 通信分为两大阶段：**证书验证阶段 + 数据传输阶段**，而数据传输阶段又分为：**非对称加密阶段 + 对称加密阶段**

1. 客户端请求 https 网址，连接到服务器的 **443 端口**(https的默认端口)
1. 采用 https 的服务器必须要用一套**数字 CA (Certification Authority)证书**
   1. 证书是需要申请的，由专门的数字证书认证机构通过严格的审核之后，所颁发的电子证书 **(当然了是要钱的，安全级别越高价格越贵)**
   1. 颁发证书的同时会**产生一个公钥和私钥，私钥由服务器端自己保存，不可泄露**
   1. 公钥附带在证书的信息中，可以公开，证书本身也附带一个证书电子签名，这个签名用来验证证书的完整性和真实性，可以防止证书被篡改。
3. 服务器响应客户端请求，**将证书传递给客户端，**证书包含公钥和大量其他信息：如证书颁发机构信息，公司信息和证书有效期信息等。(Chorme 浏览器通过点击地址栏的锁标志，再点击证书就能够看到证书详细信息)


<br />如下<br />![image.png](https://cdn.nlark.com/yuque/0/2020/png/286950/1595638729100-d5e6bc54-eb83-4f3e-b29e-f03df9ddee99.png#align=left&display=inline&height=467&margin=%5Bobject%20Object%5D&name=image.png&originHeight=934&originWidth=968&size=168856&status=done&style=none&width=484)<br />

4. 客户端解析证书比对其进行验证，如果证书不是由可信任机构颁发，或者证书中的域名和实际域名不一致，或者证书已经过期，就会向访问者发送警告，由其选择是否需要继续通信



![连接不是私密链接](https://cdn.nlark.com/yuque/0/2020/png/286950/1595639429501-ba977bd5-78c3-4179-b3fa-607bb1956cdd.png#align=left&display=inline&height=369&margin=%5Bobject%20Object%5D&name=%E8%BF%9E%E6%8E%A5%E4%B8%8D%E6%98%AF%E7%A7%81%E5%AF%86%E9%93%BE%E6%8E%A5&originHeight=514&originWidth=800&size=0&status=done&style=none&width=574)

5. 如果证书没有问题，客户端就会从服务器证书中取出**服务器的公钥 A，**然后客户端生成一个**随机码 KEY**，并使用**公钥A加密随机码 KEY**
5. 客户端将**加密后的随机码 KEY **发送给服务器，**作为后面对称加密的密钥**
5. 服务端收到**加密后的随机码 KEY** 之后，使用服务器自己的**私钥B对其进行解密**，**得到随机码KEY**
5. 经历以上的步骤，**服务器和客户端终于建立了安全连接，完美解决了对称加密的密钥泄露的问题**。后续服务器和客户端均使用**随机码 KEY (密钥)**对数据进行加密传输通信。

## 3. HTTP 与 HTTPS

**区别：**

1. 最最重要的区别就是安全性，HTTP 明文传输，不对数据进行加密安全性较差(但其实 HTTP 也能够自行加密内容进行传输，比如加密用户的密码，只是... **看参考资料2，用 HTTP 数据加密和 HTTPS 有什么区别**)。HTTPS (HTTP + SSL / TLS)的数据传输过程是加密的，安全性较好。
1. 使用 HTTPS 协议需要申请 CA 证书，一般免费证书较少，因而需要一定费用。证书颁发机构如：Symantec、Comodo、DigiCert 和 GlobalSign 等。
1. HTTP 页面响应速度比 HTTPS 快，这个很好理解，由于加了一层安全层，建立连接的过程更复杂，也要交换更多的数据，难免影响速度。
1. 由于 HTTPS 是建构在 SSL / TLS 之上的 HTTP 协议，所以，要比 HTTP 更耗费服务器资源。
1. HTTPS 和 HTTP 使用的是完全不同的连接方式，用的端口也不一样，前者是 443，后者是 80。



**HTTPS 的缺点：**

1. 在相同网络环境中，HTTPS 相比 HTTP 无论是响应时间还是耗电量都有大幅度上升。
1. HTTPS 的安全是有范围的，在黑客攻击、服务器劫持等情况下几乎起不到作用。
1. 在现有的证书机制下，中间人攻击依然有可能发生。(解决办法，看参考资料3)

## 参考资料

1. [HTTPS 详解一：附带最精美详尽的 HTTPS 原理图](https://segmentfault.com/a/1190000021494676)
1. [知乎：用 HTTP 数据加密和 HTTPS 有什么区别？](https://www.zhihu.com/question/52790301) 
1. [HTTPS详解二：SSL / TLS 工作原理和详细握手过程](https://segmentfault.com/a/1190000021559557?_ea=29659396)
