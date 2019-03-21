---
title: 将自己的站点升级成HTTPS的（疯狂踩坑）过程
tags: 技术宅
abbrlink: 45502
date: 2019-03-05 20:53:24
---

## 前言

又是一个阴雨天没有课的下午@(￣-￣)@
和舍友打完两把英雄联盟，又打开了掘金开始刷面经。（实习面试很慌啊！）
看到**HTTP 协议和 HTTPS 的区别**：

- HTTPS 就是身披 SSL( Secure Socket Layer )外壳的 HTTP，运行于 SSL 上，SSL 运行于 TCP 之上，是添加了加密和认证机制的 HTTP。就是在 HTTP(应用层）和 TCP（传输层）之间添加了一层 SSL 协议。
- HTTPS 默认运行在 443 端口，而 HTTP 默认运行在 80 端口。
- HTTPS 由于需要证书的加密机制，所以安全性更高，但同时 CPU 和内存的消耗也更多。
- HTTPS 使用共享密钥加密和公开密钥加密并用的混合加密机制
- 百度和谷歌也会优先收录 HTTPS 站点哦～

看到这里我不禁想起来，我每次打开自己的站点时候都会看到地址栏旁边有个不安全的标志，看着可是着实让人难受，不如来搞个 HTTPS 吧！

<!-- more -->

## 准备 SSL 证书

我购买的是阿里云的服务器，通过谷歌了解到阿里云是提供免费的 SSL 证书的，在如下的界面购买即可
![](http://ww1.sinaimg.cn/large/005ZR24Xly1g0s3csm5z3j31jg0y2ti7.jpg)

## 配置 Apache 支持 HTTPS

前面一顿操作，将获得的证书文件下载到了自己的电脑上
![](http://ww1.sinaimg.cn/large/005ZR24Xly1g0s3i2fsr1j316i0o4dpl.jpg)
下面开始重头戏，也是踩了不少坑的地方
也放一下[阿里云官方的配置文档](https://help.aliyun.com/document_detail/98727.html?spm=a2c4g.11186623.4.4.5b274c07Rzd4SE)
然而我们自己安装的 Apache 目录和官方的是不一样，许多配置文件都分散到了各个文件中。

### 开启 Ubuntu 的 OpenSSL 使 Apache 加载 SSL 模块

使用命令`sudo a2enmod ssl`加载 Apache 的 SSL 模块。Apache 加载 SSL 模块后，会在/etc/apache2/sites-available 下生成 default-ssl.conf 文件，我们在终端使用 sudo 权限，通过 vi 编辑器打开。
![](http://ww1.sinaimg.cn/large/005ZR24Xly1g0s6kvj9t7j314610yjzr.jpg)
这个文件需要做以下修改：

1. 第一个 VirtualHost 标签 改成 \*:443
2. ServerName 换成自己的域名
3. SSLCertificateFile、SSLCertificateKeyFile、SSLCertificateChainFile 对应的分别是刚刚下载下来的三个文件。
   完成后:wq 退出
   然后需要把 default-ssl.conf 映射至/etc/apache2/sites-enabled 文件夹
   使用命令`sudo ln -s /etc/apache2/sites-available/default-ssl.conf /etc/apache2/sites-enabled/001-ssl.conf`进行建立软链接操作。
   官方文档中

```
SSLProtocol all -SSLv2 -SSLv3
# 添加SSL协议支持协议，去掉不安全的协议。
SSLCipherSuite HIGH:!RC4:!MD5:!aNULL:!eNULL:!NULL:!DH:!EDH:!EXP:+MEDIUM
# 使用此加密套件。
SSLHonorCipherOrder on
```

在`/etc/apache2/mods-available/ssl.conf`中修改
最后重新加载 Apache 配置文件：`sudo /etc/init.d/apache2 force-reload`
重启 Apache 服务：`sudo /etc/init.d/apache2 restart`

### HTTP 重定向至 HTTPS

使用命令：`sudo a2enmod rewrite`加载 Apache 的 rewrite 模块
打开 `/etc/apache2/apache2.conf`修改如下代码

```
<Directory /var/www/>
        Options FollowSymLinks
        AllowOverride ALL
        Require all granted
</Directory>
```

然后进入你的网站根目录，使用命令 touch.htaccess 来创建.htaccess 文件
修改如下：

```
RewriteEngine on
RewriteCond  %{HTTPS} !=on
RewriteRule  ^(.*) https://%{SERVER_NAME}$1 [L,R]
```

然后重启服务器，测试我的博客网站咯～
![](http://ww1.sinaimg.cn/large/005ZR24Xly1g0s7csk8cmj31z0160aoq.jpg)
嘻嘻，大功告成！
