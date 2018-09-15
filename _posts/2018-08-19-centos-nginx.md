---
layout:       post
title:        "centos系统下安装Nginx"
subtitle:     "动手去做，很容易"
date:         2018-08-19 09:00:00
author:       "Alvin"
header-img:   "img/bathroom-escape.jpg"
header-mask:  0.3
catalog:      true
tags:
    - Centos
    - Nginx
    - 安装
    - alvin
---

**参考链接**

[CentOS 7 用 yum 安装 Nginx](http://chaishiwei.com/blog/1281.html)

[Nginx负载均衡配置](https://blog.csdn.net/xyang81/article/details/51702900)

#### 下载并安装
```
#使用以下命令
sudo yum install -y nginx
#sudo表示使用管理员权限运行命令
#yum是centos系统中下载安装程序的命令
#如果提示中发现yum资源库中没用Nginx的话，则使用以下命令进行添加
sudo rpm -Uvh http://nginx.org/packages/centos/7/noarch/RPMS/nginx-release-centos-7-0.el7.ngx.noarch.rpm
```
#### Nginx常用命令

```
#启动Nginx，使用默认配置文件启动，如果Nginx没有关闭，使用此种方式启动会出现端口被占用的情况
nginx
#停止nginx
nginx -s stop
#如果上面停止nginx的方式无效 可以强制停止
pkill -9 nginx
#重启nginx
nginx -s reload
#由于在Linux下写配置文件，容易丢个符号，导致启动失败，所以启动之前可以检查一下配置文件的正确性
nginx -t
#检查指定配置文件
nginx -t -c /etc/nginx/nginx.conf
```


#### 配置
安装成功之后，想要使用Nginx必须配置配置文件，默认配置文件的地址（/etc/nginx/nginx.conf）


```
user nginx;
worker_processes auto;
error_log /var/log/nginx/error.log;
pid /run/nginx.pid;

include /usr/share/nginx/modules/*.conf;

events {
    worker_connections 1024;
}

http {
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile            on;
    tcp_nopush          on;
    tcp_nodelay         on;
    keepalive_timeout   65;
    types_hash_max_size 2048;

    include             /etc/nginx/mime.types;
    default_type        application/octet-stream;

	include /etc/nginx/conf.d/*.conf;
	include /etc/nginx/sites-enabled/*;
    #以上配置均是默认值未曾修改，如果想搞懂上面的是什么意思，自己去慢慢学习吧
    #这个配置是负载均衡使用的
    #此处的app_nodejs是负载均衡的名字
	upstream app_nodejs {
	    #访问的实际地址是下面的，可以有多个，多个时就达到了负载均衡的作用，后面其实还有一个参数，但是此处写不写无区别。
		server 127.0.0.1:8082;
		keepalive 64;
	}
    	server {
    	#监听的是80端口，不建议换成其他端口，因为换成其他端口后，你访问时，域名也得加上加上端口，比如端口号改成8080，访问时则是：onloading.cn:8080
        listen	80	default;
        #访问的域名
		server_name onloading.cn; 
		#如果访问的是ip，则直接返回404，此处只允许通过域名访问
		if ($host ~ "\d+\.\d+\.\d+\.\d") {
    			return 404;
		}
		location / {
			proxy_set_header X-Real-IP $remote_addr;
        		proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        		proxy_set_header Host $http_host;
        		proxy_set_header X-Nginx-Proxy true;
        		proxy_set_header Connection "";
        		#指定使用哪个负载均衡，其他location的值均属于默认值
			proxy_pass http://app_nodejs;
			proxy_redirect off;
		}

    	}
}

```
如果想要进行反向代理设置，需要对http中的server节点进行设置，实现反向代理有两种方式，均是把下面的节点替换掉上面的默认文件的相关节点即可

**第一种、使用负载均衡的方式进行反向代理**
```
#app_nodejs名称是为了下面server找到对应的负载均衡
upstream app_nodejs {
	    #访问的实际地址
		server 127.0.0.1:8082;
	}
    	server {
    	#监听的是80端口，不建议换成其他端口，因为换成其他端口后，你访问时，域名也得加上加上端口，比如端口号改成8080，访问时则是：onloading.cn:8080
        listen	80	default;
        #访问的域名
		server_name onloading.cn; 
		location / {
			proxy_set_header X-Real-IP $remote_addr;
        		proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        		proxy_set_header Host $http_host;
        		proxy_set_header X-Nginx-Proxy true;
        		proxy_set_header Connection "";
        		#指定使用哪个负载均衡，其他location的值均属于默认值，即是上面的upstream的名称
			proxy_pass http://app_nodejs;
			proxy_redirect off;
		}

    	}
```

**第二种、不使用负载均衡，直接定义反向代理的地址**


```
#该种方式不需要使用upstream节点
    	server {
    	#监听的是80端口，不建议换成其他端口，因为换成其他端口后，你访问时，域名也得加上加上端口，比如端口号改成8080，访问时则是：onloading.cn:8080
        listen	80	default;
        #访问的域名
		server_name onloading.cn; 
		location / {
			proxy_set_header X-Real-IP $remote_addr;
        		proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        		proxy_set_header Host $http_host;
        		proxy_set_header X-Nginx-Proxy true;
        		proxy_set_header Connection "";
        		#与上面不同的就是，此处指定的是实际访问的地址
			proxy_pass 127.0.0.1:8082;
			proxy_redirect off;
		}

    	}
```

#### 负载均衡
**此处参考链接**

```
#此处的upstream表示平均分配给三台机器
upstream app_nodejs {
		server 192.168.0.100:8080;
		server 192.168.0.101:8080;
		server 192.168.0.101:8080;
	}
    	server {
        listen	80	default;
        #访问的域名
		server_name onloading.cn; 
		location / {
			proxy_set_header X-Real-IP $remote_addr;
        		proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        		proxy_set_header Host $http_host;
        		proxy_set_header X-Nginx-Proxy true;
        		proxy_set_header Connection "";
			proxy_pass http://app_nodejs;
			proxy_redirect off;
		}

    	}
```
* weight属性，默认为1，表示平均分配给每台机器。


```
upstream tomcats {
    server 192.168.0.100:8080 weight=2;  # 2/6次
    server 192.168.0.101:8080 weight=3;  # 3/6次
    server 192.168.0.102:8080 weight=1;  # 1/6次
}
```
* max_fails属性，默认为1，表示服务器失败的最多次数，如果超过该值，表示在fail_timeout时间内请求将不再分配到该服务器上。 如果设置为0，Nginx会将这台Server置为永久无效状态
* fail_timeout属性，默认为10秒，表示服务器的失败无效时长
* backup属性，备份机，所有服务器失效了之后，启用该服务器
* down属性，表示该台服务器无效


```
upstream tomcats {
    #表示100服务器的分配比例是2，失败最大次数为3，失败后重新失效时长为15秒
    server 192.168.0.100:8080 weight=2 max_fails=3 fail_timeout=15;
    #表示101服务器无效
    server 192.168.0.101:8080 down;
    #表示102服务器为备份服务器
    server 192.168.0.102:8080 backup;
}
```
* max_conns：表示该服务器的最大连接数量，默认为0，表示不限制。注意：1.5.9之后的版本才有这个配置


```
upstream tomcats {
    server 192.168.0.100:8080 max_conns=1000;
}
```

目前我只用过以上属性，当然Nginx还有很多其他的属性，有兴趣的可以从网上多找找。






