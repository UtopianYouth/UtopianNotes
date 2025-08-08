# Nginx 入门

## Nginx 的使用场景

> - HTTP 反向代理服务器；
>   - 反向代理：服务器通过反向代理服务器，进行网络资源的传输；
>   - 负载均衡：通过反向代理服务器，可以实现将**多个待处理的请求业务**均发给多个业务服务器，避免一个服务器高负载，而其他服务器资源空闲浪费的情况；
> - 动态资源和静态资源的分离，例子，Tomcat 实现动态资源的解析，Nginx 服务器直接通过配置文件，转发指定路径下的静态资源。
>
> - 优点；
>   - 高并发，高性能；
>   - 可扩展、高可靠；
>   - 热部署，开源。

## 正向代理和反向代理

<center>
    <img src = "./images/inet_rinet.png" width = "80%">
</center>



## Nginx 常用命令

```shell
# 1. 启动 nginx 服务
/usr/sbin/nginx	
# 2. 帮助文档
nginx -h
# 3. 测试配置文件语法正确性
nginx -t
# 4. 版本
nginx -v
# 5. 查看正在运行的 nginx 路径
ps -aux |grep nginx
# 6. -s 向 nginx 服务器发送信号
nginx -s stop		# 强制停止 nginx 服务器
nginx -s quit		# 处理完所有请求后停止
nginx -s reload		# 重启 nginx
nginx -s reopen		# nginx 主进程重新开启一个新的日志文件
```

## Nginx 配置文件

> - 基于指令和指令块；
> - 每条语句以 `;` 结尾；
> - `{}` 组织多条指令。
>
> 语法：
>
> ```nginx
> # 表示注释
> # 引入
> include 
> # 变量
> $var
> ```
>
> `nginx.conf` 配置文件
>
> 所在目录是 `etc/nginx/`，可以使用 `cat` 命令显示其内容
>
> ```nginx
> # 运行用户，默认是nginx
> user  nginx;
> 
> # nginx进程数,一般设置为和cpu核数一样
> worker_processes  1;
> 
> # 全局错误日志路径
> error_log  /var/log/nginx/error.log warn;
> 
> # 进程pid路径
> pid        /var/run/nginx.pid;
> 
> 
> events {
> # 最大连接数
>     worker_connections  1024;
> }
> 
> # 设置http服务器
> http {
>     include       /etc/nginx/mime.types;
>     default_type  application/octet-stream;
>     
>     # 设置日志的格式
>     log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
>                       '$status $body_bytes_sent "$http_referer" '
>                       '"$http_user_agent" "$http_x_forwarded_for"';
>     # 访问日志的路径
>     access_log  /var/log/nginx/access.log;
> 
> 	# 开启高效传输模式
>     sendfile        on;
>     #tcp_nopush     on;
>     
> 	# 长连接超时时间，单位是秒
>     keepalive_timeout  65;
>     
> 	#传输时是否压缩，压缩的话需要解压，但是传的大小就小了
>     #gzip  on;
>     
> 	#加载其他的配置文件，一带多
>     include /etc/nginx/conf.d/*.conf;
> }
> ```
>
> `default` 配置文件
>
> Ubuntu 下载的 Nginx 版本，对于单个应用配置文件所在目录是 `/etc/nginx/sites-available/`，`default` 文件为 Nginx 默认应用的一个 Demo。

**`nginx.conf`、`/etc/nginx/sites-available/`、`conf.d/` 三者之间的关系**

> - `/etc/nginx/nginx.conf`：主配置文件，**Nginx 启动时加载的**首要配置文件，它定义了其运行的**全局环境**；
>
> - `/etc/nginx/conf.d/`：通用配置目录，存放**不属于特定虚拟主机（站点），但适用于所有或部分 HTTP/HTTPS 流量** 的辅助性或通用性**配置文件片段**；
>
> - `/etc/nginx/sites-available/`：可用站点配置目录，存放**定义单个虚拟主机（一个网站或应用）** 的完整配置文件。
>
> 拓展：
>
> - `/etc/nginx/sites-enabled/`：启用的站点配置目录，存放指向 `.../sites-available/` 中希望 Nginx 激活的站点配置文件的**符号链接（Symbolic Links）**。
>
> Nginx 通过这种分层的、模块化的配置结构，能有效地管理和扩展你 Nginx Web 服务器，无论是运行单个站点还是托管数十个不同的应用。

## Nginx 资源文件

在 Nginx 可用站点配置目录 `/etc/nginx/sites-available/` 下，可以在站点配置文件中，编写 `root /your_path` 指令，来确定该站点资源文件所在文件系统的位置。

在编写站点配置文件时，默认推荐 `/var/www/example.com` 规范指定资源的路径，可以理解为是 Linux 服务器的一种规范。

