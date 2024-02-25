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
Nginx是一个优秀的静态Web、反向代理服务器，目前被广泛使用，其设计思路，尤其是模块支持能力非常强大，本文记录对该书的学习和实践。
<!--more-->
> 

## 概述
Nginx相比于其他Web服务器（Apache、Jetty、Tomcat、IIS），整体上有以下特点：
1. 高性能：一个显著的特点是，和Apache采用大量进程不同，Nginx推荐一个master+多个worker进程（worker数量和CPU数量相同），减少进程切换消耗，减少同步消耗。不过实际应用中，还要考虑worker进程是否会被阻塞，来最终决定进程数量。
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
Nginx的配置非常复杂，而且由于使用模块化的设计，不同模块内还有各自所支持的配置。在具体使用时，应当查询对应模块的内容。

### 基本配置
官网提供的一个[完整的典型Nginx配置示例](https://www.nginx.com/resources/wiki/start/topics/examples/full/)，如下所示

nginx.conf
```conf
user       www www;  ## Default: nobody
worker_processes  5;  ## Default: 1
error_log  logs/error.log;
pid        logs/nginx.pid;
worker_rlimit_nofile 8192;

events {
  worker_connections  4096;  ## Default: 1024
}

http {
  include    conf/mime.types;
  include    /etc/nginx/proxy.conf;
  include    /etc/nginx/fastcgi.conf;
  index    index.html index.htm index.php;

  default_type application/octet-stream;
  log_format   main '$remote_addr - $remote_user [$time_local]  $status '
    '"$request" $body_bytes_sent "$http_referer" '
    '"$http_user_agent" "$http_x_forwarded_for"';
  access_log   logs/access.log  main;
  sendfile     on;
  tcp_nopush   on;
  server_names_hash_bucket_size 128; # this seems to be required for some vhosts

  server { # php/fastcgi
    listen       80;
    server_name  domain1.com www.domain1.com;
    access_log   logs/domain1.access.log  main;
    root         html;

    location ~ \.php$ {
      fastcgi_pass   127.0.0.1:1025;
    }
  }

  server { # simple reverse-proxy
    listen       80;
    server_name  domain2.com www.domain2.com;
    access_log   logs/domain2.access.log  main;

    # serve static files
    location ~ ^/(images|javascript|js|css|flash|media|static)/  {
      root    /var/www/virtual/big.server.com/htdocs;
      expires 30d;
    }

    # pass requests for dynamic content to rails/turbogears/zope, et al
    location / {
      proxy_pass      http://127.0.0.1:8080;
    }
  }

  upstream big_server_com {
    server 127.0.0.3:8000 weight=5;
    server 127.0.0.3:8001 weight=5;
    server 192.168.0.1:8000;
    server 192.168.0.1:8001;
  }

  server { # simple load balancing
    listen          80;
    server_name     big.server.com;
    access_log      logs/big.server.access.log main;

    location / {
      proxy_pass      http://big_server_com;
    }
  }
}
```

<details>
<summary>proxy.conf</summary>


```conf
proxy_redirect          off;
proxy_set_header        Host            $host;
proxy_set_header        X-Real-IP       $remote_addr;
proxy_set_header        X-Forwarded-For $proxy_add_x_forwarded_for;
client_max_body_size    10m;
client_body_buffer_size 128k;
proxy_connect_timeout   90;
proxy_send_timeout      90;
proxy_read_timeout      90;
proxy_buffers           32 4k;
```

</details>

<details>
<summary>fastcgi.conf</summary>

```conf
fastcgi_param  SCRIPT_FILENAME    $document_root$fastcgi_script_name;
fastcgi_param  QUERY_STRING       $query_string;
fastcgi_param  REQUEST_METHOD     $request_method;
fastcgi_param  CONTENT_TYPE       $content_type;
fastcgi_param  CONTENT_LENGTH     $content_length;
fastcgi_param  SCRIPT_NAME        $fastcgi_script_name;
fastcgi_param  REQUEST_URI        $request_uri;
fastcgi_param  DOCUMENT_URI       $document_uri;
fastcgi_param  DOCUMENT_ROOT      $document_root;
fastcgi_param  SERVER_PROTOCOL    $server_protocol;
fastcgi_param  GATEWAY_INTERFACE  CGI/1.1;
fastcgi_param  SERVER_SOFTWARE    nginx/$nginx_version;
fastcgi_param  REMOTE_ADDR        $remote_addr;
fastcgi_param  REMOTE_PORT        $remote_port;
fastcgi_param  SERVER_ADDR        $server_addr;
fastcgi_param  SERVER_PORT        $server_port;
fastcgi_param  SERVER_NAME        $server_name;

fastcgi_index  index.php;

fastcgi_param  REDIRECT_STATUS    200;
```

</details>

<details>
<summary>mime.types</summary>

```conf
types {
  text/html                             html htm shtml;
  text/css                              css;
  text/xml                              xml rss;
  image/gif                             gif;
  image/jpeg                            jpeg jpg;
  application/x-javascript              js;
  text/plain                            txt;
  text/x-component                      htc;
  text/mathml                           mml;
  image/png                             png;
  image/x-icon                          ico;
  image/x-jng                           jng;
  image/vnd.wap.wbmp                    wbmp;
  application/java-archive              jar war ear;
  application/mac-binhex40              hqx;
  application/pdf                       pdf;
  application/x-cocoa                   cco;
  application/x-java-archive-diff       jardiff;
  application/x-java-jnlp-file          jnlp;
  application/x-makeself                run;
  application/x-perl                    pl pm;
  application/x-pilot                   prc pdb;
  application/x-rar-compressed          rar;
  application/x-redhat-package-manager  rpm;
  application/x-sea                     sea;
  application/x-shockwave-flash         swf;
  application/x-stuffit                 sit;
  application/x-tcl                     tcl tk;
  application/x-x509-ca-cert            der pem crt;
  application/x-xpinstall               xpi;
  application/zip                       zip;
  application/octet-stream              deb;
  application/octet-stream              bin exe dll;
  application/octet-stream              dmg;
  application/octet-stream              eot;
  application/octet-stream              iso img;
  application/octet-stream              msi msp msm;
  audio/mpeg                            mp3;
  audio/x-realaudio                     ra;
  video/mpeg                            mpeg mpg;
  video/quicktime                       mov;
  video/x-flv                           flv;
  video/x-msvideo                       avi;
  video/x-ms-wmv                        wmv;
  video/x-ms-asf                        asx asf;
  video/x-mng                           mng;
}
```

</details>

下面对配置进行讲解。Nginx的配置相对复杂，整体来说分为：块配置项、配置项、变量
1. 块配置项：如上文中的server/location/upstream/events/if，块配置项可以嵌套，内外层同名配置项关系由具体模块决定
2. 配置项：每一个配置项是一个配置名，加上若干个配置项值构成。以分号结尾。配置项值之间用空格隔开，如果某一个配置项内有语法符号，用单引号进行引用，例如
    ```conf
    # 配置项包含特殊字符，用单引号引用
    log_format main '$remote_addr - $remote_user [$time_local] "$request" ';
    # 配置项支持一些空间、时间单位写法
    expires 10y;
    gzip_buffers 4 8k;
    ```
3. 变量：使用形式例如```$xxx```，并不是所有模块都支持变量


Nginx在运行时，至少必须加载**几个核心模块**和**一个事件类模块**。这些模块运行时所支持的配置项称为基本配置：即所有其他模块执行时都依赖的配置项。即使是基本配置项，也有很多内容，整体上可以分为4类：
1. 用于调试和定位问题的配置，例如
    ```conf
    # 默认以守护进程方式启动，在调试时可以修改为off，便于gdb跟踪
    daemon on;
    # 默认master-worker模式，在调试时可以修改为off，则只创建master，并由master完成worker工作
    master_process on;
    # error日志，文件地址，等级
    error_log logs/error.log error;
    # 特殊调试点，在部分错误检测点，可以控制保存coredump
    debug_points [ stop | abort ];
    # 仅记录指定连接的debug级别日志
    debug_connection [ IP | CDIR ];
    # 限制coredump文件大小
    worker_rlimit_core 8k;
    # 设置coredump文件位置
    working_directory /your/path;
    ```
2. 正常运行的必备配置，例如
    ```conf
    # 定义环境变量
    env VAR
    env VAR=VALUE
    # 嵌入其他配置文件，支持相对路径
    include mime.types;
    # pid文件的路径
    pid logs/nginx.pid;
    # Nginx worker进程运行的用户及用户组
    user nobody nobody;
    # 指定Nginx worker进程可以打开的最大句柄描述符个数
    worker_rlimit_nofile 1024;
    # 限制信号队列，限制每个用户向Nginx发送的信号量的数量
    worker_rlimit_sigpending limit;
    ```
3. 优化性能的配置
    ```conf
    # Nginx worker进程个数
    # 如果不会出现阻塞式的调用，那么可以设置为CPU核数
    # 否则需要配置稍多一些的worker进程
    worker_processes 4;
    # 绑定Nginx worker进程到指定的CPU内核，仅有Linux支持
    # 本例子和上面的4个进程对应，指定了四个进程的绑定方式
    worker_cpu_affinity 1000 0100 0010 0001;
    # SSL硬件加速
    ssl_engine device;
    # Nginx worker进程优先级设置
    worker_priority 0;
    ```
4. 事件类配置项
    ```conf
    # 是否打开accept锁，即负载均衡锁
    accept_mutext on;
    # lock文件的路径，lock_file是为了兼容不支持原子锁的平台而设计的
    lock_file logs/nginx.lock;
    # 使用accept锁后到真正建立连接之间的延迟时间
    accept_mutex_delay 500ms;
    # 批量建立新连接
    multi_accept off;
    # 选择事件模型
    use [ kqueue | rtsig | epoll | /dev/poll | select | poll | eventport ];
    # 每个worker的最大连接数
    worker_connections 100;
    ```
> 很多配置项即使不出现在conf文件中，也是有默认值的。

### 静态Web服务器配置
使用Nginx最主要的功能是利用它的静态Web服务器能力，其核心模块是ngx_http_core_module，也有其他模块提供支持（如nginx_http_index_module等）。Nginx为静态服务器提供了非常多的功能，整体上可以分为8类：虚拟主机与请求的分发、文件路径的定义、内存及磁盘资源的分配、网络连接的设置、MIME类型的设置、对客户端请求的限制、文件操作的优化、对客户端请求的特殊处理。下面用表格的形式，列出部分配置项，有个印象就可以。
| 配置项 | 配置项值 | 配置含义说明 | 所属配置块 |
| --- | --- | --- | --- |
| listen | addr:port [ backlog \| rcvbuf \| ssl \| filter ... ] | 监听端口，支持多种监听参数 | server |
| server_name | name [ ... ] | 配置若干主机名称，用于支持虚拟主机 | server |
| server_names_hash_bucket_size | size | 当有大量虚拟主机时，使用散列函数进行存储 | http、server、location |
| server_name_in_redirect | on \| off | 重定向主机名称的处理 | http、server、location |
| location | [ =\|~\|~*\|^~\|@ ] /uri/ \{...\} | 根据用户URI匹配，并进入后面的配置块中 | server |
| root | path | 设置资源路径 | http、server、location、if |
| alias | path | 设置资源路径别名，和root略有不同 | location |
| index | file ... | 设置访问首页所展示的文件、可设置多个 | http、server、location |
| error_page | code [code ...] [=\|=anwer-code] uri\|@named_location | 当某个请求返回指定的错误码时，重定向到新的URI中，同时支持修改错误码 | http、server、location、if |
| recursive_error_pages | on\|off | 是否允许递归使用error_page | http、server、location |
| try_files | path ... uri | 尝试若干个路径下的文件用于返回，否则由uri路径负责进行返回 | server、location |
| client_body_in_file_only | on\|off\|clear | http包体是否存储到磁盘 | http、server、location |
| client_body_in_single_buffer | on\|off | http包体尽量写入到内存buffer中 | http、server、location |
| client_header_buffer_size | size | http头部的内存buffer大小 | http、server |
| client_body_buffer_size | size | http包体的内存buffer大小 | http、server、location |
| client_body_temp_path | dir-path [level1 ... ] | HTTP包体的临时存放目录 | http、server、location |
| connection_pool_size | size | 为已建立的TCP连接预留的内存池大小 | http、server |
| request_pool_size | size | 开始处理http请求后，为每个请求分配的内存池 | http、server |
| client_header_timeout | time | 读取HTTP头部的超时时间 | http、server、location |
| client_body_timeout | time | 读取HTTP包体的超时时间 | http、server、location |
| send_timeout | time | 向客户端发送响应，但没有反馈的超时时间 | http、server、location |
| lingering_close | off\|on\|always | 控制关闭连接时，对剩余接收到的用户数据的处理 | http、server、location |
| keepalive_disable | [msie6\|safari\|none] ... | 对某些浏览器禁用keepalive功能 | http、server、location |
| keepalive_timeout | time | keepalive超时时间 | http、server、location |
| keepalive_requests | n | 一个keepalive长连接上允许承载的请求最大数 | http、server、location |
| limit_except | method ... {...} | 按HTTP请求的方法名，限制用户请求 | location |
| client_max_body_size | size | HTTP请求包体的最大值 | http、server、location |
| limit_rate | speed | 对请求的限速 | http、server、location、if |
| sendfile | on\|off | 启用sendfile系统调用，减少内核和用户态的内存拷贝 | http、server、location |
| aio | on\|off | AIO系统调用 | http、server、location |
| open_file_cache | max=N\|off | 使用文件缓存 | http、server、location |
| open_file_cache_min_uses | number | 指定时间内不被淘汰，文件缓存所需最小访问次数 | http、server、location |
| ignore_invalid_headers | on\|off | 忽略不合法的HTTP头部 | http、server |
| underscores_in_headers | on\|off | HTTP头部名称是否允许带下划线 | http、server |
| if_modified_since | off\|exact\|before | 对客户端请求中的缓存时间处理方式 | http、server、location |
| merge_slashes | on\|off | 是否合并URI中相邻的斜线，如//合并为/ | http、server、location |
| resolver | address ... | 设置DNS服务器地址 | http、server、location |

mime类型的相关设置比较特别，它的格式都是
```conf
type {
    # 定义映射：mime类型 文件扩展名
    text/html html;
    # 多种文件可以映射到同一种mime类型
    text/html conf;
    image/jpeg jpg;
}
```
| 配置项 | 配置项值（含义或默认值） | 配置含义说明 | 所属配置块 |
| --- | --- | --- | --- |
| default_type | text/plain | 默认MIME类型 | http、server、location |
| types_hash_bucket_size | size | MIME类型映射的散列表，每个桶的内存大小 | http、server、location |
| types_hash_max_size | size | 散列表的总桶数，影响冲突率和内存 | http、server、location |

> 虚拟主机、以及location的匹配都有优先顺序，在使用时应当注意。

> 很多配置块、配置项支持正则表达式，如 alias、location

### 反向代理服务器的配置
反向代理，是将请求转发给合适的上有服务器，从而为客户端提供服务的一种机制。客户端只知道有反向代理服务器的存在，而不知道上游服务器的存在。

在作为反向代理服务器的时候，Nginx会**先将客户端请求缓存**到本地硬盘。再将请求转发给上游服务器（upstream）。而上游服务器返回的响应，则会**一边转发**给客户端，**一边缓存**到本地。这样做可以减少上游服务器的负载，把压力留到了Nginx服务器上。

![反向代理的转发流程](/images/book/understanding-nginx/basic-proxy.png)

这样做能减少上游服务器的原因主要是因为网络。Nginx和上游服务器之间的连接是内网，通常来说带宽大延迟低。先缓存再发送能减少连接的维持时间，减少并发的连接的数量。

反向代理服务器需要将请求转发给上游服务器，这个过程，一般会采用负载均衡的策略。Nginx也为负载均衡的实现提供了很多配置项。一个基本的配置形如
```conf
# 定义三个上游服务器，用于代理转发
upstream backend {
  server backend1.example.com;
  server backend2.example.com;
  server backend3.example.com;
}
server {
  location / {
    # 配置代理转发到上游
    proxy_pass http://backend;
  }
}
```

具体的部分常见配置项，如下表所示。
| 配置项 | 配置项值（含义或默认值） | 配置含义说明 | 所属配置块 |
| --- | --- | --- | --- |
| server | name [parameters] | 指定一个上游服务器的名字，并可以配置一些参数（如权重，超时时间） | upstream |
| ip_hash |  | 如果希望同一个用户的请求始终落在同一个服务器上，可以使用ip_hash，会利用ip做映射 | upstream |
| proxy_pass | URL | 将当前请求反向代理到对应的服务上 | location、if |
| proxy_set_header | Host $host | 反向代理转发过程中，保留请求中的Host头部 | |
| proxy_method | method | 反向代理转发时，转发后所使用的Http方法 | http、server、location |
| proxy_hide_header | header | 转发时丢弃的一些头部字段 | http、server、location |
| proxy_pass_request_body | on\|off | 转发时是否携带Http包体 | http、server、location |
| proxy_redirect | default\|off\|redirect | 当上游返回重定向或刷新时，是否重设头部的location、refresh等字段 | http、server、location |
| proxy_next_upstream | off\|error\|timeout\|http_500 ... | 当上游服务器转发出错时，继续处理还是报错，以及报什么错 | http、server、location |

> server配置块和server配置项是不同的内容

### 变量列表
| 变量名 | 有效范围 | 含义 |
| --- | --- | --- |
| $upstream_addr | 访问上游服务器期间 | 上游服务器地址 |
| $upstream_cache_status | 访问上游服务器期间 | 表示缓存命中情况 |
| $upstream_status | 访问上游服务器期间 | 上游服务器返回的响应码 |
| $upstream_response_time | 访问上游服务器期间 | 上游服务器响应时间（毫秒） |
| $upstream_http_$HEADER | 访问上游服务器期间 | HTTP头部，可选择具体头部字段 |


## 开发一个模块
### 请求处理流程
![一个简化的HTTP请求的处理流程](/images/book/understanding-nginx/simplified_http_process_flow.png)

如上图所示是一个简化的典型HTTP请求的处理流程。用语言来描述就是：
1. Nginx会先接收HTTP请求的所有的头部数据
2. 将请求的的URI和配置文件中的各种location配置块进行匹配
3. 根据最终匹配的location配置块中的HTTP模块配置，调用对应的模块。
> 非典型的HTTP请求的处理流程，则多半和模块所需要介入的阶段有关。比如需要根据IP地址决定某个客户端是否允许访问服务。具体将会在[核心原理](##核心原理)进行讲解。

### 数据结构和API速览
Nginx推荐使用封装的数据类型，而不是直接使用C的一些数据类型，例如
```c
NGX_OK;
NGX_ERROR;

// 有符号整型
typedef intptr_t ngx_int_t;
// 无符号整型
typedef uintptr_t ngx_uint_t;

// 字符串，data内不一定有\0，以len为准
typedef struct {
  size_t len;
  u_char *data;
} ngx_str_t;

// 链表元素（本质是数组）
typedef struct ngx_list_part_s ngx_list_part_t;
struct ngx_list_part_s {
  // 链表元素（数组）首地址
  void *elts;
  // 链表元素（数组）中存储对象总数
  ngx_uint_t nelts;
  // 单链表下一项
  ngx_list_part_t *next;
};

// 链表
typedef struct {
  // 链表尾
  ngx_list_part_t *last;
  // 链表头
  ngx_list_part_t part;
  // 当前链表中，每一个存储对象可用的最大字节数
  size_t size;
  // 每个ngx_list_part_t允许的最多对象数量
  ngx_uint_t nalloc;
  // 综上可知，一个ngx_list_part_t的elts，最大是size * nalloc个字节

  // 链表内存池对象
  ngx_pool_t *pool;
} ngx_list_t;
```

常见的库函数也有一些封装，例如

```c
// ngx_strncmp 字符串比较函数
#define ngx_strncmp(s1, s2, n) strncmp((const char *) s1, (const char *) s2, n)

// 创建链表
ngx_list_t ngx_list_create(ngx_pool_t pool, ngx_uint_t n, size_t size);
// 链表初始化
static ngx_inline ngx_int_t ngx_list_init(ngx_list_t list, ngx_pool_t pool, ngx_uint_t n, size_t size);
// 链表插入（先返回一个分配出来的地址，对地址赋值即可）
void* ngx_list_push(ngx_list_t list);

```


## 核心原理

## 术语
1. 虚拟主机：由于IP地址的数量有限，因此经常存在多个主机域名对应着同一个IP地址的情况，这时在nginx.conf中就可以按照server_name（对应用户请求中的主机域名）并通过server块来定义虚拟主机，每个server块就是一个虚拟主机，它只处理与之相对应的主机域名请求。

## 其他图表
ngx_list_t链表示意图

![ngx_list_t链表示意图](/images/book/understanding-nginx/ngx_list_t.png)


## 参考
1. [Nginx Docs](https://docs.nginx.com/nginx/)
2. [Nginx Wiki](https://www.nginx.com/resources/wiki/)
3. [Nginx 3rd Modules](https://www.nginx.com/resources/wiki/modules/index.html)