---
title: https的配置
date: 2017-02-27 14:01:09
tags: [https]
---
这两天在准备前端面试题，看到了很多关于https的问题，于是乎，打算给自己的博客弄上https，并且写下关于https的心得<!--more-->
## HTTPS
### 何为https？
https就是基于http的安全的传输协议。从表面看，当访问一个网站时：如果配置了https，地址栏处则会有绿色的锁，(SSL),换句话说可以通过https进行访问。http是利用了对称加密算法，非对称加密算法，以及hash算法对数据传输进行加密的传输协议。
### https的握手过程
当在地址栏处通过https://url 访问一个网站的时候，建立https的过程如下所示：
1. 浏览器将自己支持的一套加密规则在http的基础上发送给网站
2. 网站收到浏览器的消息之后，把这个网站的证书(包括 url，公钥，hash算法，加密规则)等发送给浏览器。
3. 浏览器收到证书之后，对该证书进行验证，如果此证书得到信任，则随机生成一串数字作为密码。然后根据hash算法对握手消息进行计算，并用随机数对其进行加密，然后用公钥对加密过的握手消息及随机数进行加密，发送给网站。
4. 网站收到加密的握手消息，首先用私钥进行解密，得到加密的握手消息和随机数，用随机数对握手消息进行解密，得到握手消息。并用hash算法进行计算，与浏览器发送过来的进行比较。若一致，然后用密码对一段握手消息进行加密，发送给浏览器.
5. 浏览器解密握手消息，用hash进行计算判断是否一致，若一致则完成此次握手过程。
### https与http的区别
http是超文本链接协议。https:安全超文本链接协议，它是http的安全版。
最大的区别在于:http传输是明文传输，端口号是80。https传输是密文传输，端口号是443,且可以加密传输，身份认证(ssl)
### https在个人博客的配置
一般申请CA证书都需要收费，这里使用了Cloudflare(有免费的).
其原理是：用户访问url的时候，需要进行DNS解析，然后在该域名解析的地方，将该域名解析服务器更改为CF提供的域名服务器，如果在该域名服务器已经添加了解析记录，则CF会起到中转的作用，可以成功的获得该url映射的ip地址，然后就可以访问该url了。
步骤：
1. github page 是有自带的https。如果使用了自定义域名的话，则需要自己对其进行配置。以我的博客为例:yooki.top 。 我是在阿里云购买的域名，这里的域名解析设置2个A记录，相应的值为github的ip地址.![](http://of8m1pnnt.bkt.clouddn.com/%E8%A7%A3%E6%9E%90.png)
2. [CLOUDFLARE](https://www.cloudflare.com/a/add-site)传送.按照引导设置即可
 ![](http://of8m1pnnt.bkt.clouddn.com/nameserver.png)
3. 更改域名nameserver为CLOUDFLARE的nameserver 等待status从pending转变为active，大约5分钟的等待时间。
4. 对Crypto进行SSL设置，为flexible，如图所示.![](http://of8m1pnnt.bkt.clouddn.com/flexbile.png)
5. 对Page Rules进行设置，如图所示
![](http://of8m1pnnt.bkt.clouddn.com/page%20rules.png)
![](http://of8m1pnnt.bkt.clouddn.com/page%20rule1.png)
![](http://of8m1pnnt.bkt.clouddn.com/page%20rule2.png)
这就结束了，等待一段时间之后就可以通过https进行访问url了.
