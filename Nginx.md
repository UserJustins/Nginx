# Nginx基本概念	

​	Nginx是一个高性能的HTTP和反向代理服务器，特点就是内存占用率低、并发能力强；事实上Nginx的并发能力在同类型的网页服务器中表现较好，中国大陆使用Nginx网站的用户有：百度、京东、新浪等。

Nginx可以作为静态页面的WEB服务器，同时还支持CGI协议的动态语言，比如perl、php等，但是不支持java，Java程序只能通过与Tomcat配合完成。Nginx专为性能优化而开发，性能是它最重要的考量，实现上非常注重效率，能经受高负载的考验，有报告表明能支持高达5000个并发连接数。

https://lnmp.org/nginx.html

## 	反向代理

**正向代理**：如果把局域网外的Internet想象成巨大的资源库，如果局域网中的客户端要访问Internet,则需要通过代理服务器来访问，这种代理服务器就成为正向代理。浏览器中配置代理服务器。

**反向代理**：其实客户端对代理是无感知的，因为客户端不需要任何配置就可以访问，我们只需要将请求发送到反向代理服务器，由反向代理服务器选择目标服务器获取数据并响应给客户端，此时反向代理服务器和目标服务器对外就是同一台服务器，暴露的是代理服务器地址，隐藏了真实服务器IP地址。==可以简单的理解==：反向代理服务器和Tomcat在同一台电脑上，但客户端只能够访问反向代理服务器

## 	负载均衡

在高并发的情况下，增加服务器的数量，然后将请求 分发到各个服务器上，将原先请求集中到单一服务器上的情况改为将请求分发到多个服务器上，分发到不同的服务器上的这种行为可以理解成负载均衡。

## 	动静分离

为了加快网站的解析速度，可以吧动态页面和静态页面由不同的服务器来解析，加快解析速度，降低原来单个服务器的压力。

# Nginx安装

## 安装依赖

### pcre-8.39.tar.gz	

```shell
tar -zxvf pcre-8.39.tar.gz
```

进入解压之后目录执行命令，进行检查的操作和安装

```shell
./configure #检查
#编译并安装
make && make install
#检查安装情况
rpm -qa pcre或者pcre-config --version
```

### openssl-fips-2.0.9.tar.gz

1、tar -zxvf openssl-fips-2.0.9.tar.gz

2、进入解压文件夹目录执行 ./config

3、make && make install 编译安装

### zlib-1.2.11.tar.gz

1、tar -zxvf zlib-1.2.11.tar.gz

2、进入解压文件夹目录执行 ./configure

3、make && make install 编译安装

## Nginx安装

1、tar -zxvf nginx-1.8.0.tar.gz

2、进入解压文件夹目录执行 ./configure

3、make && make install 编译安装

安装完成以后Linux系统的 /usr/local路径下就会有一个nginx的文件夹下有个sbin目录。

# Nginx启动

```shell
cd /usr/local/nginx/sbin

./nginx
```

查看进程  ==ps -ef |grep nginx==

```shell
[root@localhost sbin]# ps -ef |grep nginx
root      14150      1  0 19:02 ?        00:00:00 nginx: master process ./nginx
nobody    14152  14150  0 19:02 ?        00:00:00 nginx: worker process
root      14164   3223  0 19:03 pts/0    00:00:00 grep --color=auto nginx

```

查看Nginx的端口号，进行启动

```shell
cd  /usr/local/nginx/conf 

vi nginx.conf
```

一般情况下默认监听的是80端口，因此在浏览器中直接使用IP就可以访问

# FireWall开放端口

==查看Linux开放的端口号==

```shell
#命令 ----->      firewall-cmd --list-all
[root@localhost conf]# firewall-cmd --list-all
public (default, active)
  interfaces: eno16777736 virbr0
  sources: 
  services: dhcpv6-client ssh
  ports: 61616/tcp 8161/tcp 80/tcp #开放的端口
  masquerade: no
  forward-ports: 
  icmp-blocks: 
  rich rules: 
```



1、开放指定端口

```shell
      firewall-cmd --zone=public --add-port=80/tcp --permanent
```

 命令含义：
--zone #作用域
--add-port=1935/tcp  #添加端口，格式为：端口/通讯协议
--permanent  #永久生效，没有此参数重启后失效

2、重启防火墙

```shell
 firewall-cmd --reload
```

3、查看端口号
netstat -ntlp   //查看当前所有tcp端口·

```shell
netstat -ntulp |grep 80   
```

# Nginx的常用命令

命令生效的前提是要在该目录下/usr/local/nginx/sbin

## 查看版本号

```shell
[root@localhost sbin]# ./nginx -v
nginx version: nginx/1.8.0
```

## 关闭

./nginx -s stop

```shell
[root@localhost sbin]# ./nginx -s stop
[root@localhost sbin]# ps -ef|grep nginx
root      15281   3223  0 19:26 pts/0    00:00:00 grep --color=auto nginx

```

## 启动

```shell
[root@localhost sbin]# ./nginx 
[root@localhost sbin]# ps -ef|grep nginx
root      15298      1  0 19:27 ?        00:00:00 nginx: master process ./nginx
nobody    15300  15298  0 19:27 ?        00:00:00 nginx: worker process
root      15304   3223  0 19:27 pts/0    00:00:00 grep --color=auto nginx

```

## 修改配置文件重新加载

配置文件的路径/usr/local/nginx/conf下的nginx.conf文件,修改了此文件就使用重新加载命令

```shell
#当然该命令也是在/usr/local/nginx/sbin执行的
./nginx -s reload
```

# Nginx的配置文件

**位置**：/usr/local/nginx/conf下的nginx.conf文件

文件分为三个主要的部分

==第一部分：全局块==

从配置文件开始到events块之间的内容，主要设置一些影响服务器整体运行的配置指令，主要包括配置运行Nginx服务器的用户（组）、允许生成的worker process数、进程PID存放路径、日志存放路径和类型以及配置文件的引入等。比如

```shell
worker_processes  1;
```

这是Nginx服务器并发处理服务的关键配置，该值越大，可以支持的并发处理量也越多，但是会受到硬件和软件等设备的制约

==第二部分：events块==

events块涉及的指令主要影响nginx服务器与用户的网络连接

==第三部分：http块==

http块是配置最频繁的部分，**http块也可以包括http全局块、server块**。http块中除server块的部分就是http全局块

每个http块可以包含多个server块，而每个server块就相当于一个虚拟主机，而每一个server块也分为全局server块，以及可同时包含多个location块。

**service--->全局server块**：

本虚拟主机的监听配置和本虚拟主机的名称或IP配置

**service--->location块**：

一个server块可以配置多个location块，这块的主要作用就是基于Nginx的服务器接收到的请求字符串（例如 server_name/uri-string），对虚拟主机名称（也可以是IP别名）之外的字符串（例如 前面的、uri-string )进行匹配，对特定的请求进行处理，地址定向、数据缓存和应答控制等功能，还有许多第三方模块配置也在这里进行。



```shell

#user  nobody;
worker_processes  1;

#error_log  logs/error.log;
#error_log  logs/error.log  notice;
#error_log  logs/error.log  info;

#pid        logs/nginx.pid;


events {
	#支持的最大连接数
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
        listen       80;
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

# Location块匹配URI

1、==**=**== ：用于不含正则表达式的uri前，要求请求字符串与uri严格匹配，如果匹配成功，就停止继续向下搜索并立即处理该请求

2、==**~**==：用于表示uri包含正则表达式，并且区分大小写

3、==**~***==：用于表示uri包含的正则表达式，并且不区分大小写

注意：如果uri包含正则表达式，则必须要有  ~  或者  ~*   标识。  

# Tomcat安装启动

```shell
#解压即可完成安装
tar -zxvf apache-tomcat-7.0.104
#进入bin目录下启动
./startup.sh
#查看启动是否成功进入logs目录，tail命令执行完使用Ctrl+c退出
[root@localhost logs]# tail -f catalina.out
七月 01, 2020 10:48:26 上午 org.apache.catalina.startup.HostConfig deployDirectory
信息: Web应用程序目录[/opt/tomcat/apache-tomcat-7.0.104/webapps/host-manager]的部署已在[421]毫秒内完成
七月 01, 2020 10:48:26 上午 org.apache.catalina.startup.HostConfig deployDirectory
信息: 把web 应用程序部署到目录 [/opt/tomcat/apache-tomcat-7.0.104/webapps/manager]
七月 01, 2020 10:48:26 上午 org.apache.catalina.startup.HostConfig deployDirectory
信息: Web应用程序目录[/opt/tomcat/apache-tomcat-7.0.104/webapps/manager]的部署已在[83]毫秒内完成
七月 01, 2020 10:48:26 上午 org.apache.coyote.AbstractProtocol start
信息: 开始协议处理句柄["http-bio-8080"]
七月 01, 2020 10:48:26 上午 org.apache.catalina.startup.Catalina start
信息: Server startup in 3429 ms #启动成功

```

# Nginx配置实例---反向代理

## Simple

实现的效果：Tomcat默认访问的首页是  IP:Port   比如(http://192.168.246.135:8080/)，现将该地址设置成http://192.168.246.135也可以正常访问。即请求反向代理服务器Nginx，由Nginx跳转到对应的Tomcat上，因此需要在Nginx中配置来告诉它跳转到何处

```shell
   server {
        listen       80;
        server_name  192.168.246.135;

        location / {
            root   html;
            proxy_pass http://127.0.0.1:8080;
            index  index.html index.htm;
        }

```

配置完成记得     ./nginx -s reload

## Advanced 

==两个不同的应用程序==，根据各自的上下文路径来跳转到对应的应用程序中去。

根据URL请求的地址不同，Nginx将不同的请求跳转到不同的Tomcat端口中。

使Nginx监听9001端口，如果请求是http://192.168.246.135:9001/app/上下文，就将请求跳转到Tomcat的8081端口，如果请求是http://192.168.246.135:9001/web/上下文，就将请求跳转到Tomcat的8080端口。

当然你先得有两个Tomcat并开放的是不同的端口，并在其中简单的部署一个index.html用于页面是否真的跳转的依据。

## 配置Nginx.conf

```shell
   #http块中配置一个server
   server {
    	#监听9001端口
        listen       9001;
        server_name  192.168.246.135;
		#请求是http://192.168.246.135:9001/web/上下文，将跳转到Tomcat的8080端口
        location ~/web/ {
                proxy_pass http://127.0.0.1:8080;
        }
        #请求是http://192.168.246.135:9001/app/上下文，将跳转到Tomcat的8081端口
        location ~/app/ {
                proxy_pass http://127.0.0.1:8081;
        }

    }
```

# Nginx配置实例---负载均衡

## 默认轮询

==相同的应用程序用多个服务器进行部署==，应用上下文名称 **edu**。

### nginx.conf配置

```shell
http {
    include       mime.types;
    default_type  application/octet-stream;
    sendfile        on;
    keepalive_timeout  65;

	#===============================================================
	#配置参与负载均衡的都有哪些服务器
	#eduservice是自定义标识
    upstream eduservice{
        server 192.168.246.135:8080;
        server 192.168.246.135:8081;
    }
    #===============================================================
	#===============================================================
    server {
        listen       80;
        server_name  192.168.246.135;

        location / {
            root   html;
            #映射到要跳转的服务器集群上
            proxy_pass http://edyservice;
            index  index.html index.htm;
     }
     #===============================================================

```

访问该地址http://192.168.246.135/edu/index.html将默认轮询的访问upstream eduservice中配置的每一个端口号，如果其中一个down掉，那么down掉的就不能正常访问了。

## weight权重

weight表示权重默认是1，权重越高被分配的客户端就越多

```shell
 upstream eduservice{
        server 192.168.246.135:8080 weight=5;
        server 192.168.246.135:8081 weight=10;
    }
```

## ip_hash

每个请求按访问的IP的hash结果来进行分配，这样每个请求将固定的访问同一个服务器，可以解决session的问题

```shell
 upstream eduservice{
 		ip_hash;
        server 192.168.246.135:8080;
        server 192.168.246.135:8081;
    }
```

## fair

按后端服务器的响应时间来分配请求，响应时间短的优先分配。

```shell
    upstream eduservice{
        server 192.168.246.135:8080;
        server 192.168.246.135:8081;
        fair;
    }
```

# Nginx配置实例---动静分离

动静分离：nginx将动态请求和静态请求分发到不同的服务器中，此时只是简单的使静态资源的请求跳转到指定的服务器上指定的路径中去。

不同的静态资源放在：/opt/static/下对应的的img和html文件夹中

## Nginx配置

访问http://192.168.246.135/html/index.html就请求该路径下/opt/static/html的资源。

访问http://192.168.246.135/img/01.png就请求该路径下/opt/static/img的资源。

需要注意：URL中的img或html需要和/opt/static/下的文件名字对应起来，如果够仔细的话，你会发现

location ==**/html/** =={==


​            root ==**/opt/static/**==;

​            index  index.html index.htm;
​        }

标记的地方组合起来就是静态资源所放的路径/opt/static/ /html/

```shell
   server {
        listen       80;
        server_name  192.168.246.135;

        location /html/ {
            root  /opt/static/;
            index  index.html index.htm;
        }
        location /img/ {
            root  /opt/static/;
        }
	}
```

