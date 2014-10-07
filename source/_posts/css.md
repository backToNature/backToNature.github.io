title: Mac OSX 10.9搭建nginx+mysql+php-fpm环境
date: 2014-09-29 15:55:24
tags:
---

### 安装homebrew
homebrew是mac下非常好用的包管理器，会自动安装相关的依赖包，将你从繁琐的软件依赖安装中解放出来。 
安装homebrew也非常简单，只要在终端中输入:

``` bash
$ ruby -e "$(curl -fsSL https://raw.github.com/Homebrew/homebrew/go/install)"
```
homebrew的常用命令:

``` bash
$ brew update #更新可安装包的最新信息，建议每次安装前都运行下
$ brew search pkg_name #搜索相关的包信息
$ brew install pkg_name #安装包
```
想了解更多地信息，请参看[homebrew](http://brew.sh/)

### 安装nginx
安装

``` bash
$ brew search nginx
$ brew install nginx
```
配置

``` bash
$ cd /usr/local/etc/nginx/
$ mkdir conf.d
$ vim nginx.conf
$ vim ./conf.d/default.conf
```

nginx.conf内容,

```
worker_processes  1;  
 
error_log       /usr/local/var/log/nginx/error.log warn;
 
pid        /usr/local/var/run/nginx.pid;
 
events {
    worker_connections  256;
}
 
http {
    include       mime.types;
    default_type  application/octet-stream;
 
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';
 
    access_log      /usr/local/var/log/nginx/access.log main;
    port_in_redirect off;
    sendfile        on; 
    keepalive_timeout  65; 
 
    include /usr/local/etc/nginx/conf.d/*.conf;
}
```
default.conf文件内容,

```
server {
    listen       8080;
    server_name  localhost;
 
    root /Users/user_name/nginx_sites/; # 该项要修改为你准备存放相关网页的路径
 
    location / { 
        index index.php;
        autoindex on; 
    }   
 
    #proxy the php scripts to php-fpm  
    location ~ \.php$ {
        include /usr/local/etc/nginx/fastcgi.conf;
        fastcgi_intercept_errors on; 
        fastcgi_pass   127.0.0.1:9000; 
    }   
 
}
```
### 安装php-fpm
Mac OSX 10.9的系统自带了PHP、php-fpm，省去了安装php-fpm的麻烦。<br>
这里需要简单地修改下php-fpm的配置，否则运行php-fpm会报错。

``` bash
$ sudo cp /private/etc/php-fpm.conf.default /private/etc/php-fpm.conf
$ vim /private/etc/php-fpm.conf
```

修改php-fpm.conf文件中的`error_log`项，默认该项被注释掉，这里需要去注释并且修改为`error_log = /usr/local/var/log/php-fpm.log`。如果不修改该值，运行php-fpm的时候会提示log文件输出路径不存在的错误。

### 安装mysql
安装

``` bash
$ brew install mysql
```

常用命令
```bash
$ mysql.server start #启动mysql服务
$ mysql.server stop #关闭mysql服务
```

配置 
在终端运行`mysql_secure_installation`脚本，该脚本会一步步提示你设置一系列安全性相关的参数，包括：`设置root密码`，`关闭匿名访问`，`不允许root用户远程访问`，`移除test数据库`。当然运行该脚本前记得先启动mysql服务。

### 测试nginx服务
在之前nginx配置文件default.conf中设置的root项对应的文件夹下创建测试文件index.php:
```
<!-- ~/nginx_sites/index.php -->
<?php phpinfo(); ?>
```
启动nginx服务，sudo nginx；<br>
修改配置文件，重启nginx服务，sudo nginx -s reload <br>
启动php服务，sudo php-fpm； <br>
在浏览器地址栏中输入localhost:8080，如果配置正确地话，应该能看到PHP相关信息的页面。



