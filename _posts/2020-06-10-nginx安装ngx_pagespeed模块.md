---
layout: post
title:  "nginx安装ngx_pagespeed模块"
date:   2020-06-10 13：01：34
categories: nginx
tags: centos nginx
---

* content
{:toc}


###先安装基本依赖

```shell script
sudo yum install gcc-c++ pcre-devel zlib-devel make unzip libuuid-devel
```

###构建pagespeed
```shell script
cd /root
wget https://github.com/apache/incubator-pagespeed-ngx/archive/v1.13.35.2-stable.zip
unzip v1.13.35.2-stable.zip
cd incubator-pagespeed-ngx-1.13.35.2-stable
wget https://dl.google.com/dl/page-speed/psol/1.13.35.2-x64.tar.gz
tar -xzvf 1.13.35.2-x64.tar.gz

#注：psol 下载地址在 1.12.34 后发生变动了，如果是这版本之前，下载地址是：https://dl.google.com/dl/page-speed/psol/版本号.tar.gz。例如：：https://dl.google.com/dl/page-speed/psol/1.12.33.2.tar.gz。这个版本之后则是：https://dl.google.com/dl/page-speed/psol/版本号-x系统位数.tar.gz。例如：https://dl.google.com/dl/page-speed/psol/1.13.35.2-x64.tar.gz
```


###重新编译安装nginx

####查看原Nginx配置

```shell script
nginx -V

nginx version: nginx/1.4.4
built by gcc 4.8.5 20150623 (Red Hat 4.8.5-39) (GCC) 
TLS SNI support enabled
configure arguments: --user=www --group=www --prefix=/alidata/server/nginx --with-http_stub_status_module --without-http-cache --with-http_ssl_module --with-http_gzip_static_module --add-module=/root/incubator-pagespeed-ngx-1.13.35.2-stable

```

线上版本 
```shell script
nginx version: nginx/1.4.4
built by gcc 4.8.5 20150623 (Red Hat 4.8.5-16) (GCC) 
TLS SNI support enabled

configure arguments: --user=www --group=www --prefix=/alidata/server/nginx --with-http_stub_status_module --without-http-cache --with-http_ssl_module --with-http_gzip_static_module

```
   
下载对应的源码，知道yum 安装nginx 的版本为1.18.0,下载对应的源码

```shell script
wget wget http://nginx.org/download/nginx-1.18.0.tar.gz
```
解压
```shell script
tar -xzvf nginx-1.18.0.tar.gz
```

原配置基础上增加pagespeed模块、
```shell script
./configure --user=www --group=www --prefix=/alidata/server/nginx --with-http_stub_status_module --without-http-cache --with-http_ssl_module --with-http_gzip_static_module --add-dynamic-module=/root/incubator-pagespeed-ngx-1.13.35.2-stable --modules-path=/alidata/server/nginx/modules
```
线上版本
```shell script
./configure --user=www --group=www --prefix=/alidata/server/nginx --with-http_stub_status_module --without-http-cache --with-http_ssl_module --with-http_gzip_static_module --add-dynamic-module=/root/incubator-pagespeed-ngx-1.13.35.2-stable --modules-path=/alidata/server/nginx/modules
```


```shell script
make
```

同样，如果有需要，可对执行文件进行备份替换,(根据自己实际情况)

```shell script
cp /usr/sbin/nginx /usr/sbin/nginx.bak #备份
cp /opt/nginx-1.12.1/objs/nginx /usr/sbin/nginx #替换
    # 如果是动态模块，同时需要 cp /opt/nginx-1.12.1/objs/ngx_pagespeed.so /usr/share/nginx/modules/ 
```


配置

开启
```shell script
load_module "modules/ngx_pagespeed.so";
http {
    ...
}
server {
    listen       80;
    pagespeed on;
    pagespeed FileCachePath /tmp/ngx_pagespeed_cache;
    ...
}
```



nginx 升级平滑启动
```shell script
[root@nginx ~]# ps -ef|grep nginx

root       6324      1  0 09:06 ?        00:00:00 nginx: master process /usr/local/nginx-1.12.2/sbin/nginx


[root@nginx ~]# kill -USR2 6324


[root@nginx ~]# ps -ef|grep nginx
root       6324      1  0 09:06 ?        00:00:00 nginx: master process /usr/local/nginx-1.12.2/sbin/nginx
nobody     6325   6324  0 09:06 ?        00:00:00 nginx: worker process
root       6340   6324  0 09:12 ?        00:00:00 nginx: master process /usr/local/nginx-1.12.2/sbin/nginx
nobody     6341   6340  0 09:12 ?        00:00:00 nginx: worker process
root       6343   1244  0 09:12 pts/0    00:00:00 grep --color=auto nginx


```

这时新的master进程已经正常开启，但老的work进程也存在，所以我们使用下面的命令，将老的work进程发出平滑停止的信号，如下：

```shell script
[root@nginx ~]# kill -WINCH 6324

[root@nginx ~]# ps -ef|grep nginx
root       6324      1  0 09:06 ?        00:00:00 nginx: master process /usr/local/nginx-1.12.2/sbin/nginx
root       6340   6324  0 09:12 ?        00:00:00 nginx: master process /usr/local/nginx-1.12.2/sbin/nginx
nobody     6341   6340  0 09:12 ?        00:00:00 nginx: worker process
root       6346   1244  0 09:14 pts/0    00:00:00 grep --color=auto nginx

```

###注意

nginx 在 1.9.9后才添加了 load_module ，不然会报这个错
```shell script
nginx: [emerg] unknown directive "load_module" in /alidata/server/nginx/conf/nginx.conf:11
```