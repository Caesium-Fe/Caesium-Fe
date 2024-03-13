# Django项目部署流程与Nginx安装配置

## 1. 什么是Nginx及其主要作用 

## nginx是一个高性能的HTTP和反向代理web服务器，同时也提供了IMAP/POP3/SMTP服务。

1）反向代理服务器：位于客户端用户与目标服务器之间，用户直接访问反向代理服务器就可以获取目标服务器的资源。

2）什么是负载均衡：讲工作任务分摊到多个操作单元上进行执行。

3）理解动静分离：加快网站的解析速度，将动态和静态页面由不同服务器进行解析。

## 2. 实现Nginx反向代理

1）项目部署原理分析：在项目中，nginx负责接收客户端的请求（一个nginx服务器能够同同一时刻支持5万的并发量），并将请求分为动态和静态，而uWSGI就将nginx的请求转变为django web框架能够理解的形式发送给django，根据客户端请求，django将响应返回交给uWSGI，依次传递到nginx，返回给客户端。

```shell
##############http请求##############
#################↓##################
###############nginx################
#######↓#################↓##########
####静态请求###########动态请求#######
#######↓#################↓##########
####静态文件############uWSGI########
#####*.css###############↓##########
#####*.js##############Django#######
#####*.png################↓#########
#####*.html#############app_1#######
#####......#############app_2#######
```

2）nginx安装与配置

nginx分两种版本安装，生产环境一般是linux，而学习环境为windows，两种都介绍下：

```nginx
#linux版本通过查看 runoob.com/linux/nginx-install-setup.html 即可

#windows版本
进到官网：https://nginx.org/en/download.html，找到自己需要的版本进行下载，得到压缩包后进行解压。
解压完成后：在nginx的配置文件是conf目录下的nginx.conf，修改文件。

'关于windows版本如何开启关闭nginx服务 遇到的问题...
'1.开启服务：使用cmd或双击 nginx.exe文件, 服务正常开启
'2.关闭服务：并不能通过直接关闭cmd，来进行关闭nginx服务，需同路径下新开一个cmd，输入nginx.exe -s stop指令进行关闭, 若关闭cmd后，需将nginx服务的所有进程关闭才算关闭。 
'注意：直接右键进程是关闭不掉进程的，这时需要将所有的nginx进程全部杀死，操作步骤如下：
'1.查看所有端口并找到使用nginx服务端口的进程号------ netstat -ano
'2.通过找到的进程号找到所有nginx.exe的进程--------- tasklist | findstr "****"(pid number) 1180  9280  9608  4416  11232
'3.结束所有进程：-------------------------------- taskkill /f /t /im nginx.exe
'4.最后验证下服务是否关闭就ok了。

《第一部分：全局块》
 worker_processes  1;
从配置文件开始到events块之间的内容，主要会设置一些影响Nginx服务器整体运行的配置指令，主要包括：配置运行Nginx服务器的用户（组）、允许生成的 worker process 数，进程PID存放路径、日志存放路径和类型以及配置文件的引入等。
上面这行 worker_processes 配置，是 Nginx 服务器并发处理服务的关键配置，该值越大，可以支持的并发处理量也越多，但是会受到硬件、软件等设备的约束。

《第二部分：events 块》
events {
	worker_connections  1024;
}
events 块涉及的指令主要影响Nginx服务器与用户的网络连接，常用的设置包括：是否开启对多 work process下的网络连接进行序列化，是否允许同时接收多个网络连接，选取哪种事件驱动模型来处理连接请求，每个 work process 可以同时支持的最大连接数等

上述例子就表示每个 work process 支持的最大连接数为 1024。这部分的配置对Nginx的性能影响较大，在实际中应该灵活配置。
《第三部分：http 块》
这部分是 Nginx 服务器配置中最频繁的部分，代理、缓存和日志定义等绝大多数功能和第三方模块的配置都在这里。需要注意的是：http 块也可以包括 http 全局块、server 块。下面的反向代理、动静分离、负载均衡都是在这部分中配置
▪ http 全局块：http 全局块配置的指令包括：文件引入、MIME-TYPE 定义、日志自定义、连接超时时间、单链接请求数上限等。
▪ server 块：这块和虚拟主机有密切关系，从用户角度看，虚拟主机和一台独立的硬件主机是完全一样的，该技术的产生是为了节省互联网服务器硬件成本。
每个http块可以包括多个server块，而每个server块就相当于一个虚拟主机。而每个server块也分为全局server块，以及可以同时包含多个locaton块。（☆☆☆☆☆）

《全局 server 块》
最常见的配置是本虚拟机主机的监听配置和本虚拟主机的名称或IP配置。

《location 块》
一个 server 块可以配置多个 location 块。
这块的主要作用是：基于 Nginx 服务器接收到的请求字符串（例如 server_name/uri-string），对虚拟主机名称（也可以是 IP 别名）之外的字符串（例如 前面的 /uri-string）进行匹配，对特定的请求进行处理。地址定向、数据缓存和应答控制等功能，还有许多第三方模块的配置也在这里进行。
```

配置反向代理：server部分就是客户端访问的目标地址及端口，location部分就是被代理服务器地址（location部分还可以写成匹配指令）

```nginx
location指令：

该指令用于匹配 URL， 语法如下：

location [ = | ~ | ~* | ^~] url {
}
= ：用于不含正则表达式的 url 前，要求请求字符串与 url 严格匹配，如果匹配成功，就停止继续向下搜索并立即处理该请求
~：用于表示 url 包含正则表达式，并且区分大小写
~*：用于表示 url 包含正则表达式，并且不区分大小写
^~：用于不含正则表达式的 url 前，要求 Nginx 服务器找到标识 url 和请求。字符串匹配度最高的 location 后，立即使用此 location 处理请求，而不再使用 location块中的正则 url 和请求字符串做匹配。
注意：如果 url 包含正则表达式，则必须要有 ~ 或者 ~* 标识
```

## 3.实现负载均衡

upstream：配置**负载均衡**，upstream 默认是以轮询的方式进行负载，另外还支持**四种模式**，分别是：

（1）weight：权重，指定轮询的概率，weight 与访问概率成正比

（2）ip_hash：按照访问 IP 的 hash 结果值分配

（3）fair：按后端服务器响应时间进行分配，响应时间越短优先级别越高

（4）url_hash：按照访问 URL 的 hash 结果值分配

如何实现：首先配置nginx.conf文件，在server的同一级新增upstream节点，设置对应的多个后端服务地址，这里设置的节点地址需要与代理跳转地址一致。

```nginx
upstream abc.com {  # 这里写的abc.com需要与下面location中proxy一致。
		server 127.0.0.1:9091; # 这里添加需要进行负载均衡的地址
		server 127.0.0.1:9092;
}
location / {
		proxy_pass http://abc.com;
}	
```

# Nginx参数详解

1、定义Nginx运行的用户和用户组

user www www;

2、nginx进程数,建议设置为等于CPU总核心数.

worker_processes 8;

3、全局错误日志定义类型,[ debug | info | notice | warn | error | crit ]
error_log /var/log/nginx/error.log info;

4、进程文件
pid /var/run/nginx.pid;

5、一个nginx进程打开的最多文件描述符数目,理论值应该是最多打开文件数（系统的值ulimit -n）与nginx进程数相除,但是nginx分配请求并不均匀,所以建议与ulimit -n的值保持一致.
worker_rlimit_nofile 65535;

## 工作模式与连接数上限

events
{
#参考事件模型,use [ kqueue | rtsig | epoll | /dev/poll | select | poll ]; epoll模型是Linux 2.6以上版本内核中的高性能网络I/O模型,如果跑在FreeBSD上面,就用kqueue模型.
use epoll;
#单个进程最大连接数（最大连接数=连接数*进程数）
worker_connections 65535;
}

## MySQL服务器，端口9998（集群环境）

stream{
    upstream mysql_socket{
        server 192.168.174.131:3306;
    }
    server{
        listen 9998;
        proxy_pass mysql_socket;
    }
}



## 设定http服务器

http
{
include mime.types; #文件扩展名与文件类型映射表
default_type application/octet-stream; #默认文件类型
#charset utf-8; #默认编码
server_names_hash_bucket_size 128; #服务器名字的hash表大小
client_header_buffer_size 32k; #上传文件大小限制
large_client_header_buffers 4 64k; #设定请求缓
client_max_body_size 8m; #设定请求缓

\#开启目录列表访问,合适下载服务器,默认关闭.
autoindex on; # 显示目录
autoindex_exact_size on; # 显示文件大小 默认为on,显示出文件的确切大小,单位是bytes 改为off后,显示出文件的大概大小,单位是kB或者MB或者GB
autoindex_localtime on; # 显示文件时间 默认为off,显示的文件时间为GMT时间 改为on后,显示的文件时间为文件的服务器时间

sendfile on; # 开启高效文件传输模式,sendfile指令指定nginx是否调用sendfile函数来输出文件,对于普通应用设为 on,如果用来进行下载等应用磁盘IO重负载应用,可设置为off,以平衡磁盘与网络I/O处理速度,降低系统的负载.注意：如果图片显示不正常把这个改成off.
tcp_nopush on; # 防止网络阻塞
tcp_nodelay on; # 防止网络阻塞

keepalive_timeout 120; # (单位s)设置客户端连接保持活动的超时时间,在超过这个时间后服务器会关闭该链接

\#FastCGI相关参数是为了改善网站的性能：减少资源占用,提高访问速度.下面参数看字面意思都能理解.
fastcgi_connect_timeout 300;
fastcgi_send_timeout 300;
fastcgi_read_timeout 300;
fastcgi_buffer_size 64k;
fastcgi_buffers 4 64k;
fastcgi_busy_buffers_size 128k;
fastcgi_temp_file_write_size 128k;

\# gzip模块设置
gzip on; #开启gzip压缩输出
gzip_min_length 1k; #允许压缩的页面的最小字节数,页面字节数从header偷得content-length中获取.默认是0,不管页面多大都进行压缩.建议设置成大于1k的字节数,小于1k可能会越压越大
gzip_buffers 4 16k; #表示申请4个单位为16k的内存作为压缩结果流缓存,默认值是申请与原始数据大小相同的内存空间来存储gzip压缩结果
gzip_http_version 1.1; #压缩版本（默认1.1,目前大部分浏览器已经支持gzip解压.前端如果是squid2.5请使用1.0）
gzip_comp_level 2; #压缩等级.1压缩比最小,处理速度快.9压缩比最大,比较消耗cpu资源,处理速度最慢,但是因为压缩比最大,所以包最小,传输速度快
gzip_types text/plain application/x-javascript text/css application/xml;
#压缩类型,默认就已经包含text/html,所以下面就不用再写了,写上去也不会有问题,但是会有一个warn.
gzip_vary on;#选项可以让前端的缓存服务器缓存经过gzip压缩的页面.例如:用squid缓存经过nginx压缩的数据

## 开启限制IP连接数的时候需要使用

#limit_zone crawler $binary_remote_addr 10m;

## 虚拟主机的配置

##upstream的负载均衡,四种调度算法(下例主讲)##
server
{
\#  监听端口
listen 80;
\#  域名可以有多个,用空格隔开
server_name ably.com;
\#  HTTP 自动跳转 HTTPS
rewrite ^(.*) https://$server_name$1 permanent;
}

#server_name的域名需要在host和hosts文件里进行添加     修改如下：

host：添加   www.XXXX.com  127.0.0.1

hosts： 添加  127.0.0.1   www.XXXX.com



server
{
\#  监听端口 HTTPS
listen 443 ssl;
server_name ably.com;

## 配置域名证书

ssl_certificate C:\WebServer\Certs\certificate.crt;
ssl_certificate_key C:\WebServer\Certs\private.key;
ssl_session_cache shared:SSL:1m;
ssl_session_timeout 5m;
ssl_protocols SSLv2 SSLv3 TLSv1;
ssl_ciphers ALL:!ADH:!EXPORT56:RC4+RSA:+HIGH:+MEDIUM:+LOW:+SSLv2:+EXP;
ssl_prefer_server_ciphers on;

index index.html index.htm index.php;
root /data/www/;
location ~ .*\.(php|php5)?$
{
fastcgi_pass 127.0.0.1:9000;
fastcgi_index index.php;
include fastcgi.conf;
}

##  配置地址拦截转发，解决跨域验证问题

location /oauth/{
proxy_pass https://localhost:13580/oauth/;
proxy_set_header HOST $host;
proxy_set_header X-Real-IP $remote_addr;
proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
}

## 图片缓存时间设置

location ~ .*\.(gif|jpg|jpeg|png|bmp|swf)$ {
expires 10d;
}

 

## JS和CSS缓存时间设置

location ~ .*\.(js|css)?$ {
expires 1h;
}

## 日志格式设定

log_format access '$remote_addr - $remote_user [$time_local] "$request" '
'$status $body_bytes_sent "$http_referer" '
'"$http_user_agent" $http_x_forwarded_for';
\#  定义本虚拟主机的访问日志
access_log /var/log/nginx/access.log access;



## 设定查看Nginx状态的地址：

StubStatus模块能够获取Nginx自上次启动以来的工作状态，此模块非核心模块，需要在Nginx编译安装时手工指定才能使用
location /NginxStatus {
stub_status on;
access_log on;
auth_basic "NginxStatus";
auth_basic_user_file conf/htpasswd;
#htpasswd文件的内容可以用apache提供的htpasswd工具来产生.
}

## Nginx多台服务器实现负载均衡：

1.Nginx负载均衡服务器：
IP：192.168.0.4（Nginx-Server）
2.Web服务器列表：
Web1:192.168.0.5（Nginx-Node1/Nginx-Web1） ；Web2:192.168.0.7（Nginx-Node2/Nginx-Web2）
3.实现目的：用户访问Nginx-Server（“http://mongo.demo.com:8888”）时，通过Nginx负载均衡到Web1和Web2服务器
Nginx负载均衡服务器的nginx.conf配置注释如下：
events
{
use epoll;
worker_connections 65535;
}
http
{
##upstream的负载均衡,四种调度算法##
#调度算法1:轮询.每个请求按时间顺序逐一分配到不同的后端服务器,如果后端某台服务器宕机,故障系统被自动剔除,使用户访问不受影响
upstream webhost {
server 192.168.0.5:6666 ;
server 192.168.0.7:6666 ;
}
#调度算法2:weight(权重).可以根据机器配置定义权重.权重越高被分配到的几率越大
upstream webhost {
server 192.168.0.5:6666 weight=2;
server 192.168.0.7:6666 weight=3;
}
#调度算法3:ip_hash. 每个请求按访问IP的hash结果分配,这样来自同一个IP的访客固定访问一个后端服务器,有效解决了动态网页存在的session共享问题
upstream webhost {
ip_hash;
server 192.168.0.5:6666 ;
server 192.168.0.7:6666 ;
}
#调度算法4:url_hash(需安装第三方插件).此方法按访问url的hash结果来分配请求,使每个url定向到同一个后端服务器,可以进一步提高后端缓存服务器的效率.Nginx本身是不支持url_hash的,如果需要使用这种调度算法,必须安装Nginx 的hash软件包
upstream webhost {
server 192.168.0.5:6666 ;
server 192.168.0.7:6666 ;
hash $request_uri;
}
#调度算法5:fair(需安装第三方插件).这是比上面两个更加智能的负载均衡算法.此种算法可以依据页面大小和加载时间长短智能地进行负载均衡,也就是根据后端服务器的响应时间来分配请求,响应时间短的优先分配.Nginx本身是不支持fair的,如果需要使用这种调度算法,必须下载Nginx的upstream_fair模块
#
#虚拟主机的配置(采用调度算法3:ip_hash)
server
{
listen 80;
server_name mongo.demo.com;
#对 "/" 启用反向代理
location / {
proxy_pass http://webhost;
proxy_redirect off;
proxy_set_header X-Real-IP $remote_addr;
#后端的Web服务器可以通过X-Forwarded-For获取用户真实IP
proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
#以下是一些反向代理的配置,可选.
proxy_set_header Host $host;
client_max_body_size 10m; #允许客户端请求的最大单文件字节数
client_body_buffer_size 128k; #缓冲区代理缓冲用户端请求的最大字节数,
proxy_connect_timeout 90; #nginx跟后端服务器连接超时时间(代理连接超时)
proxy_send_timeout 90; #后端服务器数据回传时间(代理发送超时)
proxy_read_timeout 90; #连接成功后,后端服务器响应时间(代理接收超时)
proxy_buffer_size 4k; #设置代理服务器（nginx）保存用户头信息的缓冲区大小
proxy_buffers 4 32k; #proxy_buffers缓冲区,网页平均在32k以下的设置
proxy_busy_buffers_size 64k; #高负荷下缓冲大小（proxy_buffers*2）
proxy_temp_file_write_size 64k;
#设定缓存文件夹大小,大于这个值,将从upstream服务器传
}
}
}

## 负载均衡操作示例：

操作对象：192.168.0.4（Nginx-Server）

#创建文件夹准备存放配置文件
$ mkdir -p /opt/confs
$ vim /opt/confs/nginx.conf

#编辑内容如下：
events
{use epoll;
worker_connections 65535;
}

http
{
upstream webhost {
ip_hash;
server 192.168.0.5:6666 ;
server 192.168.0.7:6666 ;
}

server
{
listen 80;
server_name mongo.demo.com;
location / {
proxy_pass http://webhost;
proxy_redirect off;
proxy_set_header X-Real-IP $remote_addr;
proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
proxy_set_header Host $host;
client_max_body_size 10m;
client_body_buffer_size 128k;
proxy_connect_timeout 90;
proxy_send_timeout 90;
proxy_read_timeout 90;
proxy_buffer_size 4k;
proxy_buffers 4 32k;
proxy_busy_buffers_size 64k;
proxy_temp_file_write_size 64k;
}
}
}

#然后保存并退出

#启动负载均衡服务器192.168.0.4（Nginx-Server）

docker run -d -p 8888:80 --name nginx-server -v /opt/confs/nginx.conf:/etc/nginx/nginx.conf --restart always nginx
操作对象：192.168.0.5（Nginx-Node1/Nginx-Web1）

#创建文件夹用于存放web页面

$ mkdir -p /opt/html
$ vim /opt/html/index.html

#编辑内容如下：

The host is 192.168.0.5(Docker02) - Node 1!

#然后保存并退出

#启动192.168.0.5（Nginx-Node1/Nginx-Web1）

$ docker run -d -p 6666:80 --name nginx-node1 -v /opt/html:/usr/share/nginx/html --restart always nginx
操作对象：192.168.0.7（Nginx-Node2/Nginx-Web2）

#创建文件夹用于存放web页面

$ mkdir -p /opt/html
$ vim /opt/html/index.html

#编辑内容如下：

The host is 192.168.0.7(Docker03) - Node 2!

#启动192.168.0.7（Nginx-Node2/Nginx-Web2）

$ docker run -d -p 6666:80 --name nginx-node2 -v $(pwd)/html:/usr/share/nginx/html --restart always nginx





# Nginx遇到的问题

```shell
1.修改Nginx.conf文件后启动Nginx，没有正常启动，查看出错日志： #问题没有解决
[alert] 19696#22104: the event "ngx_master_19696" was not signaled for 5s
遇到以上问题，解决办法：
	将修改全部恢复，仍没有正常启动，将nginx重装后任然没用；查看端口使用情况，并无发现有进程在占用；怀疑是机器资源不足。。。
	# 重启电脑后解决。
2
```

