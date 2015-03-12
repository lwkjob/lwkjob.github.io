 ---
layout: post
title: "nginx linux 安装 和 反向代理配置 "
description: " "
keywords: "nginx"
category: nginx
tags: [nginx]
---
#安装 nginx 
> 依次安装 pcre-8.35.tar.gz,zlib-1.2.7.tar.gz,openssl-fips-2.0.2.tar.gz,  nginx-1.2.6.tar.gz

1 安装pcre
[ftp://ftp.csx.cam.ac.uk/pub/software/programming/pcre/](ftp://ftp.csx.cam.ac.uk/pub/software/programming/pcre/) 下载最新的 PCRE 源码包，使用下面命令下载编译和安装 PCRE 包：

```
cd /usr/local/src
wget ftp://ftp.csx.cam.ac.uk/pub/software/programming/pcre/pcre-8.35.tar.gz 
tar -zxvf pcre-8.34.tar.gz
cd pcre-8.34
./configure
make
make install
```
2 安装zlib库

[http://zlib.net/zlib-1.2.8.tar.gz](http://zlib.net/zlib-1.2.8.tar.gz) 下载最新的 zlib 源码包，使用下面命令下载编译和安装 zlib包：
```
cd /usr/local/src

wget http://zlib.net/zlib-1.2.8.tar.gz
tar -zxvf zlib-1.2.8.tar.gz
cd zlib-1.2.8
./configure
make
make install
```

3 安装ssl（某些vps默认没装ssl)
```
cd /usr/local/src
wget http://www.openssl.org/source/openssl-1.0.1c.tar.gz
tar -zxvf openssl-1.0.1c.tar.gz
``` 

>  Nginx 一般有两个版本，分别是稳定版和开发版，您可以根据您的目的来选择这两个版本的其中一个，
> 下面是把 Nginx 安装到 /usr/local/nginx 目录下的详细步骤：
```
cd /usr/local/src
wget http://nginx.org/download/nginx-1.4.2.tar.gz
tar -zxvf nginx-1.4.2.tar.gz
cd nginx-1.4.2

./configure --sbin-path=/usr/local/nginx/nginx \
--conf-path=/usr/local/nginx/nginx.conf \
--pid-path=/usr/local/nginx/nginx.pid \
--with-http_ssl_module \
--with-pcre=/usr/local/src/pcre-8.34 \
--with-zlib=/usr/local/src/zlib-1.2.8 \
--with-openssl=/usr/local/src/openssl-1.0.1c

make
make install
```
> --with-pcre=/usr/src/pcre-8.34 指的是pcre-8.34 的源码路径。
> --with-zlib=/usr/src/zlib-1.2.7 指的是zlib-1.2.7 的源码路径。

4 启动
> 确保系统的 80 端口没被其他程序占用，运行 ` ./usr/local/nginx/nginx ` 命令来启动 Nginx，
> 如果你不是root用户 会报错
> 出现问题： nginx: [emerg] bind() to 0.0.0.0:80 failed (13: Permission denied) 
> 是因为端口号的问题，在Linux中1024以下的端口号都需要root用户才能使用
> 所以 你可以用root启动，或者修改配置文件`/usr/local/nginx/nginx.conf`

```

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
        listen       8080; #这里默认是80
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
#新加的2个反向代理
   server{
    listen 8081;
    server_name 192.168.10.217;
    location / {
        proxy_redirect off;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_pass http://192.168.10.209:8090; #需要被代理的服务器地址和端口
    }
    access_log logs/lwktest1.tk_access.log;
}
server{
    listen 8082;
    server_name 192.168.10.217;
    location / {
        proxy_redirect off;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_pass http://192.168.10.217:8090; #需要被代理的服务器地址和端口
    }
    access_log logs/lwktest2.tk_access.log;
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

}

``` 


'nginx -s stop '// 停止nginx
'nginx -s reload' // 重新加载配置文件
'nginx -s quit' // 退出nginx
查看是否启动：'netstat -ntlp'