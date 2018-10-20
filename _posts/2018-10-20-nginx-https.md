---
layout:       post
title:        "Nginx配置项目为Https"
subtitle:     "动手去做，很容易"
date:         2018-10-20 13:00:00
author:       "Alvin"
header-img:   "img/centos-nginx.jpg"
header-mask:  0.3
catalog:      true
tags:
    - Https
    - Nginx
    - 安装
    - 技术
---

## 前言
>一般公司的项目，访问 www.domain.com 与domain.com都会跳转到同意个域名下，并且都是https的访问模式。所以今天这篇博客主要说一下，如何实现的。再聊聊为什么大家都用https。再深入聊聊https的原理

## 生成https证书

想要使用https访问，必须要在服务器上有https证书，https分为付费的与免费的。付费的到阿里云上买就可以了，免费的阿里云也有，但是比较麻烦，个人不喜欢在阿里云申请免费https证书，此处借鉴了[Let's Encrypt - 靠谱又免费的SSL证书](https://www.jianshu.com/p/ae407556b13d),这上面有很详细的步骤，跟着走，便能得到证书了。

## 配置Nginx
* 找到配置文件，如果别人安装的你找不到，则使用下面的命令
```
nginx -t
```
* 上配置：

```
    #该配置表示非https的监控
    server {
                listen 80;
                server_name  www.domain.com domain.com;
 		        rewrite ".*" https://domain.com;#如果域名为上述，则直接跳转到 https://domain.com
    }
    #该配置表示https的监控
	server {
		listen 443;
		server_name domain.com;
		ssl on;
        ssl_certificate   /alidata/www-domain-com/domain-crt.crt;
        ssl_certificate_key  /alidata/www-domain-com/domain-key.key;
		ssl_session_timeout 5m;
	    ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
	    ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:HIGH:!aNULL:!MD5:!RC4:!DHE;
	    ssl_prefer_server_ciphers on;		
		location / {
			proxy_pass http://127.0.0.1:3000;
		}
	}

```
* 解释配置：
    * rewrite:表示跳转，如果监控匹配到了，则跳转到相应的域名，跳转位置可以加https或http
    * ssl_certificate：表示证书的存放路径
    * ssl_certificate_key：表示秘钥的存放路径
    * ssl_session_timeout: 表示会话缓存的过期时间，默认为5m（分钟）
    * ssl_ciphers：表示启用指定的加密算法
    * ssl_protocols：表示启用指定的协议


* 配置完成之后，重启Nginx
```
nginx -s reload
```

## 什么是https

>HTTPS (Secure Hypertext Transfer Protocol)安全超文本传输协议，是一个安全通信通道，它基于HTTP开发用于在客户计算机和服务器之间交换信息。它使用安全套接字层(SSL)进行信息交换，简单来说它是HTTP的安全版,是使用TLS/SSL加密的HTTP协议

## http 与https的区别

出自[详解 HTTPS、TLS、SSL、HTTP区别和关系](https://www.wosign.com/INFO/https_tls_ssl_http.htm)

* HTTPS协议需要到证书颁发机构(Certificate Authority，简称CA)申请证书，一般免费证书很少，需要交费。
* HTTP是超文本传输协议，信息是明文传输，HTTPS则是具有安全性的SSL加密传输协议。
* HTTP和HTTPS使用的是完全不同的连接方式，使用的端口也不一样,前者是80,后者是443。
* HTTP的连接很简单,是无状态的。
* HTTPS协议是由SSL+HTTP协议构建的可进行加密传输、身份认证的网络协议，要比HTTP协议安全。

从上面可看出，HTTPS和HTTP协议相比提供了

* 数据完整性：内容传输经过完整性校验
* 数据隐私性：内容经过对称加密，每个连接生成一个唯一的加密密钥
* 身份认证：第三方无法伪造服务端(客户端)身份

其中，数据完整性和隐私性由TLS Record Protocol保证，身份认证由TLS Handshaking Protocols实现