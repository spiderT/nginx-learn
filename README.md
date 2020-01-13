# nginx-learn

## 1. nginx 的优点

1. nginx 相对 apache 的优点：
- 轻量级，同样起web 服务，比apache 占用更少的内存及资源
- 抗并发，nginx 处理请求是异步非阻塞的，而apache 则是阻塞型的，在高并发下nginx 能保持低资源低消耗高性能
- 高度模块化的设计，编写模块相对简单
- 社区活跃，各种高性能模块出品迅速
- Nginx本身就是一个反向代理服务器
- Nginx支持7层负载均衡
- nginx 适合做静态，简单，效率高

2. 作为 Web 服务器：相比 Apache，Nginx 使用更少的资源，支持更多的并发连接，体现更高的效率，这点使 Nginx 尤其受到虚拟主机提供商的欢迎。在高连接并发的情况下，Nginx是Apache服务器不错的替代品: Nginx在美国是做虚拟主机生意的老板们经常选择的软件平台之一. 能够支持高达 50000 个并发连接数的响应, 感谢Nginx为我们选择了 epoll and kqueue 作为开发模型.
Nginx作为负载均衡服务器: Nginx 既可以在内部直接支持 Rails 和 PHP 程序对外进行服务, 也可以支持作为 HTTP代理服务器对外进行服务. Nginx采用C进行编写, 不论是系统资源开销还是CPU使用效率都比 Perlbal 要好很多.

3. Nginx 配置简洁, Apache 复杂 ，Nginx 启动特别容易, 并且几乎可以做到7*24不间断运行，即使运行数个月也不需要重新启动. 你还能够不间断服务的情况下进行软件版本的升级 . Nginx 静态处理性能比 Apache 高 3倍以上 ，Apache 对 PHP 支持比较简单，Nginx 需要配合其他后端来使用 ,Apache 的组件比 Nginx 多.

4. 最核心的区别在于apache是同步多进程模型，一个连接对应一个进程；nginx是异步的，多个连接（万级别）可以对应一个进程 .

 5. nginx的优势是处理静态请求，cpu内存使用率低，apache适合处理动态请求，所以现在一般前端用nginx作为反向代理抗住压力，apache作为后端处理动态请求。


 ## 2. nginx 的 编译安装

 下载页: http://nginx.org/en/download.html

 编译安装 Nginx

这里会将各依赖的源码编译进 Nginx, 所以 --with-zlib 和 --with-pare 后为对应的依赖源码目录路径。此外, 编译选项中还开启了 HTTPS 的协议支持 --with-http_ssl_module, 若不需要 HTTPS, 可取消该选项。

cd /Users/wid/Downloads/

tar zxvf nginx-1.8.0.tar.gz

cd nginx-1.8.0

./configure --prefix=/usr/local/nginx --with-zlib=../zlib-1.2.8 --with-pcre=../pcre-8.36 --with-http_ssl_module

make

sudo make install

编译安装完成, 测试启动、重启、停止:

cd /usr/local/nginx

#启动

sudo sbin/nginx     #浏览器访问 127.0.0.1 测试是否成功启动


#重启

sudo sbin/nginx -s reload


#停止

sudo sbin/nginx -s stop


## 3. nginx的 配置语法

1. 配置文件由指令和指令块构成

2. 每条指令由分号;结尾，指令与参数间以空格符号分隔

3. 指令块以{}打括号将多条指令组织 在一起

4. include语句允许组合多个配置文件以提升可维护性

5. 使用#符号添加注释，提高可读性

6. 使用$符号使用变量

7. 部分指令的参数支持正则表达式


```conf
user  nginx; #设置nginx服务的系统使用用户
worker_processes  1; #工作进程数

error_log  /var/log/nginx/error.log warn; #nginx的错误日志
pid        /var/run/nginx.pid; #nginx启动时候的pid

events {
    worker_connections  1024; #每个进程允许的最大连接数
}

http { #http请求配置，一个http可以包含多个server

    #定义 Content-Type
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

    #日志格式 此处main与access_log中的main对应
    #$remote_addr：客户端地址
    #$remote_user：http客户端请求nginx认证的用户名，默认不开启认证模块，不会记录
    #$timelocal：nginx的时间
    #$request：请求method + 路由 + http协议版本
    #status：http reponse 状态码
    #body_bytes_sent：response body的大小
    #$http_referer：referer头信息参数，表示上级页面
    #$http_user_agent：user-agent头信息参数，客户端信息
    #$http_x_forwarded_for：x-forwarded-for头信息参数
    log_format  main  '$http_user_agent' '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    #访问日志，后面的main表示使用log_format中的main格式记录到access.log中
    access_log  /var/log/nginx/access.log  main;

    #nginx的一大优势，高效率文件传输
    sendfile        on;
    #tcp_nopush     on;

    #客户端与服务端的超时时间，单位秒
    keepalive_timeout  65;

    #gzip  on;
    server { #http服务，一个server可以配置多个location
        listen       80; #服务监听端口
        server_name  localhost; #主机名、域名
    
        #charset koi8-r;
        #access_log  /var/log/nginx/host.access.log  main;
    
        location / {
            root   /usr/share/nginx/html; #页面存放目录
            index  index.html index.htm; #默认页面
        }
    
        #error_page  404              /404.html;
    
        # 将500 502 503 504的错误页面重定向到 /50x.html
        error_page   500 502 503 504  /50x.html;
        location = /50x.html { #匹配error_page指定的页面路径
            root   /usr/share/nginx/html; #页面存放的目录
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
    include /etc/nginx/conf.d/*.conf;
}
```

