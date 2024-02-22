---
title: "深入理解Nginx：模块开发与架构解析"
date: 2024-02-22T10:01:25+08:00
categories:
- 读书笔记
- 技术书
tags:
- 中间件
- Web
thumbnailImagePosition: left
thumbnailImage: /images/thumbnail/book/understanding-nginx.png
draft: true
---
Nginx是一个优秀的服务器，目前被广泛使用，其设计思路，尤其是模块支持能力非常强大，本文记录对该书的学习和实践。
<!--more-->
> 

## 概述
Nginx相比于其他Web服务器（Apache、Jetty、Tomcat、IIS），整体上有以下特点：
1. 高性能：一个显著的特点是，和Apache采用大量进程不同，Nginx推荐一个master+多个worker进程（worker数量和CPU数量相同），减少进程切换消耗，减少同步消耗。
2. 高扩展性：模块化
3. 高可靠性：master进程 + worker子进程，worker出错时可以被快速替换
4. 低内存消耗：10000个非活跃的keep-alive连接只消耗2.5MB内存
5. 单机支持10w并发连接
6. 热部署，不间断升级，不间断修改配置
7. 自由的类似BSD-2 Clause的许可（只需要在引用时保留license）

对于这种高性能服务器的设计，Nginx在极端情况下的目标是：
1. 低并发压力下，用户获得高速访问的体验
2. 高并发压力下，尽量接入更多的用户，但每个用户的访问速度会有一定的下降（受制于处理器和网络带宽）

## 环境搭建
[Nginx官网](https://nginx.org/)提供源码，推荐使用编译的方式进行安装，它的依赖相对较少，主要有
1. gcc
2. pcre：Perl Compatible Regular Expressions（Perl兼容正则表达式）
3. zlib：对http内容压缩
4. openssl：加密，也提供了MD5、SHA1等散列函数
> Nginx提供了Windows下开箱即用的二进制程序，但本文只考虑其在Linux系统下的使用。

使用Nginx，推荐对Linux内核参数进行调整，以达到最佳效果。一个常用的配置是
```conf
# /etc/sysctl.conf内核参数
# 修改后使用sysctl -p使参数生效

# 单一进程最大可用的打开文件数（句柄数）
fs.file-max = 999999
# 允许将time_wait状态的tcp连接直接重用
net.ipv4.tcp_tw_reuse = 1
# keepalive消息发送间隔，间隔越短，无效连接被清理越快
net.ipv4.tcp_keepalive_time = 600
# tcp断开连接时在fin-2状态的最大等待时间
net.ipv4.tcp_fin_timeout = 30
# 允许的time_wait套接字数量的最大值，超过将会立刻清除并打印警告
net.ipv4.tcp_max_tw_buckets = 5000
# 接收syn的队列的最大长度，在来不及处理握手时，让TCP发起方稍等，而非直接丢弃
net.ipv4.tcp_max_syn.backlog = 1024
# UDP、TCP的本地端口取值范围
net.ipv4.ip_local_port_range = 1024 61000
# TCP接收缓存（用于TCP接收滑动窗口）的最小值、默认值、最大值
net.ipv4.tcp_rmem = 4096 32768 262142
# TCP发送缓存（用于TCP发送滑动窗口）的最小值、默认值、最大值
net.ipv4.tcp_wmem = 4096 32768 262142
# 网卡接收快于内核处理时，用于保存已接受但未处理的数据包
net.core.netdev_max_backlog = 8096
# 内核套接字接收缓存区默认的大小
net.core.rmem_default = 262144
# 内核套接字发送缓存区默认的大小
net.core.wmem_default = 262144
# 内核接收缓存最大尺寸
net.core.rmem_max = 2097152
# 内核发送缓存最大尺寸
net.core.wmem_max = 2097152
# 用于解决TCP的SYN攻击
net.ipv4.tcp_syncookies = 1
```
> 涉及到内存的配置，一定程度上会影响并发连接的数目，以及处理速度。因为每个连接都会消耗配置的内存量。需要做一定的权衡。

在解压缩源代码之后，一个基本的编译安装流程是
```bash
# 通过./configure --help 查看更多选项
./configure
make
make install

# 编译后查看编译选项，大写V
nginx -V

```

除了少量核心代码外，Nginx完全是由各种功能模块组成的。这些模块会根据配置参数决定自己的行为，因此，正确地使用各个模块非常关键。在configure的参数中，将其分为五大类。
1. 对事件模块，即处理事件驱动的模块。根据情况，可以选用rtsig、select、poll、aio。
1. 对默认即编译进入Nginx的HTTP模块，根据情况，可以选择性的去除一些HTTP模块，如charset、gzip、ssi、userid、auth、geo等。
1. 对默认不会编译进入Nginx的HTTP模块，可以显式加入，如ssl、realip、flv、mp4等。
1. 和邮件代理服务器相关的mail模块，如pop3、imap等。
1. 其他模块或控制参数，如debug、第三方模块、ipv6等。

> 对于编译时的更多默认模块信息，可以通过auto文件夹下的options文件进行查看。

**configure**其实就是一个bash脚本，在调用期间，主要完成的工作有
1. 合并默认参数和本次使用的参数，初始化目录结构，扫描整体源文件信息以便于后续生成Makefile。这些任务分别由auto文件夹下的options、init、sources脚本完成。
2. 判断是否有DEBUG标志，并写入DEBUG宏
3. 检查编译器环境是否符合要求，auto/cc/conf
4. 检查系统的一些必要头文件，auto/headers
5. 最重要的，构造运行期需要使用的各种modules，auto/modules。写入ngx_modules.c文件，主要是填充ngx_modules数组，以说明哪些模块会参与到一个请求的处理流程中。模块之间是有优先级和顺序的，必须正确的添加到数组中。
6. 编写Makefile、处理链接时依赖库处理、处理运行时属主属组，auto/make、auto/lib/make、auto/install
7. 总结本次configure，auto/summary

最终的中间文件夹obj下结构形如
```bash
|---obj
| |---ngx_auto_headers.h
| |---autoconf.err
| |---ngx_auto_config.h
| |---ngx_modules.c
| |---src
| | |---core
| | |---event
| | | |---modules
| | |---os
| | | |---unix
| | | |---win32
| | |---http
| | | |---modules
| | | | |---perl
| | |---mail
| | |---misc
| |---Makefile
```

## 基础使用
命令行控制
```bash
# 直接启动，读取/usr/local/nginx/conf/nginx.conf的配置
/usr/local/nginx/sbin/nginx

# 以下假定已将/usr/local/nginx/sbin加入PATH

# 另行指定配置文件
nginx -c /your/path/to/nginx.conf

# 指定安装目录
nginx -p /your/path/to/nginx

# 指定运行时全局配置
nginx -g "pid /your/path/to/your.pid" # 本例将会将pid写入your.pid

# 不启动，只检测配置是否正确
nginx -t

# 控制nginx启停
nginx -s stop   # 等价于kill SIGTERM/SIGINT
nginx -s quit   # 等待连接关闭再退出，等价于SIGQUIT
nginx -s reload
nginx -s reopen # 清空日志文件，开始重写

# 热升级
kill -s SIGUSR2 <old nginx master pid> # 向旧nginx发送SIGUSR2
/your/path/to/new/nginx # 启动新的
kill -s SIGQUIT <old nginx master pid> # 优雅的退出旧的
```

## Nginx的配置
一个典型的Nginx配置，如下所示
```conf
user nobody;
worker_processes 8;
error_log /var/log/nginx/error.log error;
# pid logs/nginx.pid;
events {
    use epoll;
    worker_connections 50000;
}
http {
    include mime.types;
    default_type application/octet-stream;
    log_format main 
        '$remote_addr [$time_local] "$request" '
        '$status $bytes_sent "$http_referer" '
        '"$http_user_agent" "$http_x_forwarded_for"';
    access_log logs/access.log main buffer=32k;
    # ...
    upstream backend {
        server 127.0.0.1:8080;
    }
    gzip on;
    server {
        location /webstatic {
            gzip off;
        }
    }
}

server {

}

location {

}

upstream {

}
```

Nginx的配置相对复杂，整体来说分为：块配置项、配置项、变量
1. 块配置项：如上文中的server/location/upstream/events，块配置项可以嵌套，内外层同名配置项关系由具体模块决定
2. 配置项：每一个配置项是一个配置名，加上若干个配置项值构成。以分号结尾。配置项值之间用空格隔开，如果某一个配置项内有语法符号，用单引号进行引用，例如
    ```conf
    # 配置项包含特殊字符，用单引号引用
    log_format main '$remote_addr - $remote_user [$time_local] "$request" ';
    # 配置项支持一些空间、时间单位写法
    expires 10y;
    gzip_buffers 4 8k;
    ```
3. 变量：使用形式例如```$xxx```，并不是所有模块都支持变量