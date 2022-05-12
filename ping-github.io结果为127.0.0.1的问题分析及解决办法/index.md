# Ping Github.io结果为127.0.0 的问题分析及解决办法


### 定位问题

使用window命令`ping rowkey.cn`查看网络是否通，ping到的结果是：`127.0.0.1`。

大家都知道TCP/IP 在设计之初，被IP地址分为A类，B类，C类，D类及私有地址，而127.0.0.1就属于私有地址。众所周知，私有地址无论如何是不会映射到公网域名，如果ping某个域名返回的是私有地址，那么只有一种可能，要么DNS的解析出现问题，要么本地Host将域名映射不到某个私有地址。

那么接下来让咱们先看看Host。在自己的系统中找到host文件，然后打开后Host内容如下：

```
# Host Database
#
# localhost is used to configure the loopback interface
# when the system is booting.  Do not change this entry.

127.0.0.1  eureka7001.com
127.0.0.1  eureka7002.com
127.0.0.1  eureka7003.com


# Modified hosts end

185.199.108.153 superhj1987.github.io


127.0.0.1    activate.xxx.com
127.0.0.1    practivate.xxx.com
127.0.0.1    ereg.xxx.com
127.0.0.1    wip3.xxx.com
127.0.0.1    activate.wip3.xxx.com
127.0.0.1    3dns-3.xxx.com
127.0.0.1    3dns-2.xxx.com
127.0.0.1    adobe-dns.xxx.com
127.0.0.1    adobe-dns-2.adobe.com
127.0.0.1    adobe-dns-3.adobe.com
# Added by Docker Desktop
# To allow the same kube context to work on the host and the container:
127.0.0.1 kubernetes.docker.internal
# End of section
```

从上面的Host内容看到，我并没有在Host中硬配置该域名的IP。那么就剩下另一个可能，也就是DNS出现了问题。那么我们使用nslookup 命令来看看该域名到底在哪个环节出现问题。

```shell
landsnail@landsnail ~ % nslookup rowkey.cn           
Server:        116.116.116.116
Address:    116.116.116.116#53

Non-authoritative answer:
rowkey.cn    canonical name = superhj1987.github.io
Name:    superhj1987.github.io
Address: 127.0.0.1
```

从上面的返回我们能够清晰的看到，DNS服务器为`116.116.116.116`，而域名`rowkey.cn`通过cname跳到了`superhj1987.github.io`这个子域名上，接着返回的地址为：127.0.0.1，也就是上面ping命令返回的解释。

也就是`rowkey.cn`通过cname成功解析到子域名 `superhj1987.github.io`，而再解析域名`superhj1987.github.io` 的时候出现了该问题。

直接`ping superhj1987.github.io`返回的结果也是：127.0.0.1。那么为何该域名没有被正确解析呢？这里可能存在两个原因，第一个就是该域名在域名服务器：`116.116.116.116`被错误解析，第二个就是该域名被某种技术污染了，怎么换DNS服务器都返回这个结果。

接着排查问题。

我们更换DNS服务器为`114.114.144.144`，接着再使用`nslookup`命令查看结果。试了几次返回的结果依旧是`127.0.0.1`，到此我们可以得出结论：**github.io**的解析被某种神秘的力量污染了，所以不管更换到哪个DNS服务器，返回的结果都是这个IP地址。

### 解决问题

既然已经定位到问题，那么如何解决呢？其实答案很简单，就是在本地的Host中配置一条DNS解析，强制将**github.io**解析到正确的地址上即可，经过广大网友的友情帮助，我们得到如下配置：

```
185.199.110.153 superhj1987.github.io
185.199.111.153 superhj1987.github.io
185.199.108.153 superhj1987.github.io
185.199.109.153 superhj1987.github.io
```


