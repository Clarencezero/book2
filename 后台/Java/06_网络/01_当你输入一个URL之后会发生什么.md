# 当你输入一个 URL 之后会发生什么

## 1. DNS 域名解析

所有浏览器首先要确认的是域名所对应的服务器在哪里。将域名解析成对应的服务器 IP 地址这项工作，是由 DNS 服务器来完成的。客户端收到你输入的域名地址后，它首先去找本地的 hosts 文件，检查在该文件中是否有相应的域名、IP 对应关系，如果有，则向其 IP 地址发送请求，如果没有，再去找 DNS 服务器。一般用户很少去编辑修改 hosts 文件。

![DNS 根 ](https://cdn.jsdelivr.net/gh/Clarencezero/poi/DNS root.png)

![DNS 递归搜索 ](https://cdn.jsdelivr.net/gh/Clarencezero/poi/DNS search.png)

- 浏览器客户端向本地 DNS 服务器发送一个含有域名 www.cnblogs.com 的 DNS 查询报文。`本地 DNS 服务器`把查询报文转发到`根 DNS 服务器`，

  `根 DNS 服务器`注意到其 com 后缀，于是向本地 DNS 服务器返回 comDNS 服务器的 IP 地址。

- `本地 DNS 服务器`再次向 comDNS 服务器发送查询请求，

- `comDNS 服务器`注意到其 www.cnblogs.com 后缀并用负责该域名的权威 DNS 服务器的 IP 地址作为回应。最后，本地 DNS 服务器将含有 www.cnblogs.com 的 IP 地址的响应报文发送给客户端。

- **从客户端到本地服务器属于递归查询，而 DNS 服务器之间的交互属于迭代查询。**

### 2. 建立 TCP 连接

#### 2.1 三次握手

![三次握手 ](https://cdn.jsdelivr.net/gh/Clarencezero/poi/tcp three hand.png)

客户端发送一个带有 SYN 标志的数据包给服务端，服务端收到后，回传一个带有 SYN/ACK 标志的数据包以示传达确认信息，最后客户端再回传一个带 ACK 标志的数据包，代表握手结束，连接成功。

上图也可以这么理解：

客户端：“你好，在家不，有你快递。”

服务端：“在的，送来就行。”

客户端：“好嘞。”

TCP 三次握手之后,浏览器将自己的打包好的包裹不是直接给 TCP, 而是委托 TLS 保安全权负责。里面的内容保密, 不能被修改。TLS 也会经过三次握手:

TLS 大叔先发言：你好，我支持 TLS 版本 1.2，以及我的认证算法、加密算法、数据校验算法，此外还有我的随机码，收到请回复。
TLS 服务器回复：你好，我也支持 1.2 版本，那我们就使用 xx 认证算法、xx 加密算法、xx 数据校验算法，我的随机码是 xx，来实现安保措施，你看好吗？
TLS 大叔：没问题啊，能出示一下你的证件（数字证书）吗？
TLS 服务器：okay，这是我的证件，请过目。
TLS 大叔发现对方发过来两个证书：
证书 1：“*.zhihu.com”，由 GeoTrust RSA CA2018 签名并颁发

证书 2：“GeoTrust RSA CA2018°，由 Digicert Global Root CA 签名并颁发

验证过程如下：

1. 用 Digicert Global Root CA 的公钥解密证书 2 的签名 Digicert Global Root CA 作为一个权威 CA，已经被浏览器预先安装在可信任根证书列表，那么我们信任该 CA 的一切，当然包括其公钥，在该证书里包含了明文的公钥，如下图所示：

2. 解开了，证明是该 CA 私钥加密的，由于 CA 私钥只有 CA 知道，证书有效，并信任 GeoTrust RSA CA2018 公钥。
3. 解不开，证明不是 CA 私钥加密，无效证书。



### 3 发送 HTTP 请求

![报文](https://cdn.jsdelivr.net/gh/Clarencezero/poi/baowen.png)

### 4 服务器处理请求

![服务器处理 ](https://cdn.jsdelivr.net/gh/Clarencezero/poi/server handle.png)

### 5 返回响应结果

![HTTP 返回 ](https://cdn.jsdelivr.net/gh/Clarencezero/poi/http response.png)

![返回码](https://cdn.jsdelivr.net/gh/Clarencezero/poi/return code.png)

### 10.6 关闭 TCP 连接

![4 次挥手 ](https://cdn.jsdelivr.net/gh/Clarencezero/poi/4 hand byby.png)
上图可以这么理解：

客户端：“兄弟，我这边没数据要传了，咱关闭连接吧。”

服务端：“收到，我看看我这边有木有数据了。”

服务端：“兄弟，我这边也没数据要传你了，咱可以关闭连接了。”

客户端：“好嘞。”

### 10.7 浏览器解析 HTML

浏览器通过解析 HTML，生成 DOM 树，解析 CSS，生成 CSS 规则树，然后通过 DOM 树和 CSS 规则树生成渲染树。渲染树与 DOM 树不同，渲染树中并没有 head、display 为 none 等不必显示的节点。

要注意的是，浏览器的解析过程并非是串连进行的，比如在解析 CSS 的同时，可以继续加载解析 HTML，但在解析执行 JS 脚本时，会停止解析后续 HTML，这就会出现阻塞问题，关于 JS 阻塞相关问题，这里不过多阐述,后面会单独开篇讲解。

### 10.8 浏览器布局渲染

根据渲染树布局，计算 CSS 样式，即每个节点在页面中的大小和位置等几何信息。HTML 默认是流式布局的，CSS 和 js 会打破这种布局，改变 DOM 的外观样式以及大小和位置。这时就要提到两个重要概念：replaint 和 reflow。

