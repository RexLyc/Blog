---
title: "深入理解Nginx：模块开发与架构解析"
date: 2024-02-22T10:01:25+08:00
categories:
- 读书笔记
- 技术书
tags:
- 中间件
- Web
- 暂停更新
thumbnailImagePosition: left
thumbnailImage: /images/thumbnail/book/understanding-nginx.png
---
Nginx是一个优秀的静态Web、反向代理服务器，目前被广泛使用，其设计思路，尤其是模块支持能力非常强大，本文记录对该书的学习和实践。
<!--more-->

> Nginx作为Web服务器、反向代理服务器，编写模块所能带来的扩展性主要有三种，handler类型（处理请求，给反馈）、过滤器类型（filter）、负载均衡类型（upstream、load-balance）。在编写代码时，一定要注意避免同步阻塞。

> 书中的Nginx版本较低，在实际实践时，使用的是1.24.0版本。

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
# 可能的依赖项(不同包管理器可能不一致)
sudo apt install libpcre3 libpcre3-dev \
  libperl-dev zlib1g-dev openssl libssl-dev

# 如果不是源代码安装的openssl，需要使用参数
# ./configure --with-http_ssl_module
# ./configure --with-debug
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
| proxy_set_header | Host $host | 反向代理转发过程中，保留请求中的Host头部 | http、server、location |
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
// Nginx全局错误码
// 一般来说，框架在不同的场景下，对不同的错误码都有专门的响应
#define NGX_OK 0
#define NGX_ERROR -1
#define NGX_AGAIN -2
#define NGX_BUSY -3
#define NGX_DONE -4
#define NGX_DECLINED -5
#define NGX_ABORT -6

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

// 键值
typedef struct {
  // 散列值
  ngx_uint_t hash;
  // 键
  ngx_str_t key;
  // 值
  ngx_str_t value;
  // 全小写化的key
  u_char *lowcase_key;
} ngx_table_elt_t;

// 缓冲区，是Nginx最重要的数据结构，也是一个较为复杂的结构
// 缓冲区设计的好坏，将会直接影响到内存的使用，以及响应速度
typedef struct ngx_buf_s ngx_buf_t;
typedef void* ngx_buf_tag_t; 
struct ngx_buf_s {
  /* pos通常是用来告诉使用者本次应该从
  pos这个位置开始处理内存中的数据，这样设置是因为同一个
  ngx_buf_t可能被多次反复处理。当然，
  pos的含义是由使用它的模块定义的 */
  u_char *pos;
  /* last通常表示有效的内容到此为止，注意，
  pos与last之间的内存是希望nginx处理的内容 */
  u_char *last;
  /* 处理文件时用，和pos与last类似，
  file_pos表示将要处理的文件位置，
  file_last表示截止的文件位置 */
  off_t file_pos;
  off_t file_last;
  // 如果ngx_buf_t缓冲区用于内存，那么start指向这段内存的起始地址
  u_char *start;
  // 与start成员对应，指向缓冲区内存的末尾
  u_char *end;
  /* 表示当前缓冲区的类型，例如由哪个模块使用就指向这个模块
  ngx_module_t变量的地址 */
  ngx_buf_tag_t tag;
  // 引用的文件
  ngx_file_t *file;
  /* 当前缓冲区的影子缓冲区（共享缓冲区），该成员很少用到，
  仅仅在使用缓冲区转发上游服务器的响应时才使用了shadow成员，
  这是因为Nginx希望尽可能节约内存了，分配一块内存并使用
  ngx_buf_t表示接收到的上游服务器响应后，在向下游客户端转发时，
  可能会把这块内存存储到文件中，也可能直接向下游发送，
  此时Nginx绝不会重新复制一份内存用于新的目的，
  而是再次建立一个ngx_buf_t结构体指向原内存，
  这样多个ngx_buf_t结构体指向了同一块内存，
  它们之间的关系就通过shadow成员来引用。
  这种设计过于复杂，通常不建议使用 */
  ngx_buf_t *shadow;
  // 临时内存标志位，为1时表示数据在内存中且这段内存可以修改
  unsigned temporary:1;
  // 标志位，为1时表示数据在内存中且这段内存不可以被修改
  unsigned memory:1;
  // 标志位，为1时表示这段内存是用mmap系统调用映射过来的，不可以被修改
  unsigned mmap:1;
  // 标志位，为1时表示可回收
  unsigned recycled:1;
  // 标志位，为1时表示这段缓冲区处理的是文件而不是内存
  unsigned in_file:1;
  // 标志位，为1时表示需要执行flush操作
  unsigned flush:1;
  /* 标志位，对于操作这块缓冲区时是否使用同步方式，需谨慎考虑，
  这可能会阻塞Nginx进程，Nginx中所有操作几乎都是异步的，
  这是它支持高并发的关键。有些框架代码在sync为1时，
  可能会有阻塞的方式进行I/O操作，它的意义视使用它的Nginx模块而定 */
  unsigned sync:1;
  /* 标志位，表示是否是最后一块缓冲区，因为ngx_buf_t可以由
  ngx_chain_t链表串联起来，因此，当last_buf为1时，
  表示当前是最后一块待处理的缓冲区 */
  unsigned last_buf:1;
  // 标志位，表示是否是ngx_chain_t中的最后一块缓冲区
  unsigned last_in_chain:1;
  /* 标志位，表示是否是最后一个影子缓冲区，与
  shadow域配合使用。通常不建议使用它 */
  unsigned last_shadow:1;
  // 标志位，表示当前缓冲区是否属于临时文件
  unsigned temp_file:1;
};

// 缓冲区链，常用于发送HTTP包体
typedef struct ngx_chain_s ngx_chain_t;
struct ngx_chain_s {
  // 当前缓冲区
  ngx_buf_t *buf;
  // 链中的下一个
  ngx_chain_t *next;
};

// 上文中的ngx_file_t
typedef struct ngx_file_s ngx_file_t;
struct ngx_file_s {
  // 文件句柄描述符
  ngx_fd_t fd;
  // 文件名称
  ngx_str_t name;
  // 文件大小等资源信息，实际就是Linux系统定义的stat结构
  ngx_file_info_t info;
  /* 该偏移量告诉Nginx现在处理到文件何处了，
  一般不用设置它，Nginx框架会根据当前发送状态设置它 */
  off_t offset;
  // 当前文件系统偏移量，一般不用设置它，同样由Nginx框架设置
  off_t sys_offset;
  // 日志对象，相关的日志会输出到log指定的日志文件中
  ngx_log_t *log;
  // 目前未使用
  unsigned valid_info:1;
  // 与配置文件中的directio配置项相对应，在发送大文件时可以设为1
  unsigned directio:1;
};

// Nginx内的文件信息是Linux系统文件信息结构体stat的别名
typedef struct stat ngx_file_info_t;

// 只要使用了文件，就需要对文件句柄进行清理，清理过程所需的结构体如下
typedef struct {
  // 文件句柄
  ngx_fd_t fd;
  // 文件名称
  u_char *name;
  // 日志对象
  ngx_log_t *log;
} ngx_pool_cleanup_file_t;

// 清理工作链表，执行清理的方法，以及待清理数据
typedef struct ngx_pool_cleanup_s ngx_pool_cleanup_t;
struct ngx_pool_cleanup_s {
  // 执行实际清理资源工作的回调方法
  ngx_pool_cleanup_pt handler;
  // handler回调方法需要的参数
  void *data;
  // 下一个ngx_pool_cleanup_t清理对象，如果没有，需置为NULL
  ngx_pool_cleanup_t *next;
};

// 用于存储配置项解析出来的路径参数
typedef struct {
  ngx_str_t name;
  size_t len;
  size_t level[3];
  ngx_path_manager_pt manager;
  ngx_path_loader_pt loader;
  void data;
  u_char conf_file;
  ngx_uint_t line;
} ngx_path_t;

// ngx提供的解析http解析结果存储结构
typedef struct {
  ngx_uint_t code;
  ngx_uint_t count;
  u_char *start;
  u_char *end;
} ngx_http_status_t;
```

常见的库函数、工具函数也有一些封装，例如

```c
// ngx_strncmp 字符串比较函数
#define ngx_strncmp(s1, s2, n) strncmp((const char *) s1, (const char *) s2, n)

// 创建链表
ngx_list_t ngx_list_create(ngx_pool_t pool, ngx_uint_t n, size_t size);
// 链表初始化
static ngx_inline ngx_int_t ngx_list_init(ngx_list_t list, ngx_pool_t pool, ngx_uint_t n, size_t size);
// 链表插入（先返回一个分配出来的地址，对地址赋值即可）
void* ngx_list_push(ngx_list_t list);

// 打开文件
#define ngx_open_file(name, mode, create, access) \
  open((const char *) name, mode|create, access)

// 获取文件信息
#define ngx_file_info(file, sb) stat((const char *) file, sb)

// offsetof的可能实现，用于在解析配置时，计算成员的偏移
#define offsetof(type,member)(size_t)&(((type*)0)->member)

// 字符串转数字
ngx_int_t ngx_atoi(u_char *line, size_t n);

// 配置项合并宏（所有可以用=号赋值的内容的合并）
#define ngx_conf_merge_value(conf, prev, default)                            \
    if (conf == NGX_CONF_UNSET) {                                            \
        conf = (prev == NGX_CONF_UNSET) ? default : prev;                    \
    }


```

### 加入编译
Nginx的模块是以源码形式加入项目的，框架提供的加入方式，需要编写一个config文件，核心是修改三种变量，其内容如下
```conf
# 定义模块名称。可以将ngx_http_hello_module替换为你想用的任意名称
ngx_addon_name=ngx_http_hello_module
# 添加到模块数组，编译目标。HTTP_MODULES NGX_ADDON_SRCS必须以追加的形式进行修改
HTTP_MODULES="$HTTP_MODULES ngx_http_hello_module"
NGX_ADDON_SRCS="$NGX_ADDON_SRCS $ngx_addon_dir/ngx_http_hello_module.c"
```

在实际开发中，config文件中提供的变量，以及可以修改的内容都更多。而且可供增加的模块种类也更多。事实上，包括$CORE_MODULES、$EVENT_MODULES、$HTTP_MODULES、$HTTP_FILTER_MODULES、$HTTP_HEADERS_FILTER_MODULE等模块变量都可以重定义，它们分别对应着Nginx的核心模块、事件模块、HTTP模块、HTTP过滤模块、HTTP头部过滤模块。

一般将config文件和自定义模块的源文件放在同一目录中，此后，可以调用configure尝试编译，如下
```bash
# 需指明添加的模块路径
./configure --add-module=./path/to/ngx_http_hello_module
```

### HTTP模块模板
和其他框架一样，Nginx为模块开发提供了一个模板，用户只需要对其中的回调部分进行开发。模块定义的入口类型是```ngx_module_t```，他的定义如下

```c
typedef struct ngx_module_s ngx_module_t;
struct ngx_module_s {
  /* 下面的ctx_index、index、spare0、
  spare1、spare2、spare3、version变量，
  不需要在定义时赋值，可以用Nginx准备好的宏来定义，
  #define NGX_MODULE_V1 0, 0, 0, 0, 0, 0, 1
  对于一类模块（由下面的type成员决定类别）而言，
  ctx_index表示当前模块在这类模块中的序号。
  这个成员常常是由管理这类模块的一个Nginx核心模块设置的。
  对于所有的HTTP模块而言，ctx_index是由核心模块ngx_http_module设置的。
  ctx_index非常重要，Nginx的模块化设计非常依赖于各个模块的顺序，
  它们既用于表达优先级，也用于表明每个模块的位置，借以帮助
  Nginx框架快速获得某个模块的数据 */
  ngx_uint_t ctx_index;
  /* index表示当前模块在ngx_modules数组中的序号。
  注意，ctx_index表示的是当前模块在一类模块中的序号，
  而index表示当前模块在所有模块中的序号，它同样关键。
  Nginx启动时会根据ngx_modules数组设置各模块的
  index值。初始化方式例如：
    ngx_max_module = 0;
    for (i = 0; ngx_modules[i]; i++) {
      ngx_modules[i]->index = ngx_max_module++; 
    }
  */
  ngx_uint_t index;
  // spare系列的保留变量，暂未使用
  ngx_uint_t spare0;
  ngx_uint_t spare1;
  ngx_uint_t spare2;
  ngx_uint_t spare3;
  // 模块的版本，便于将来的扩展。目前只有一种，默认为1
  ngx_uint_t version;
  /* ctx用于指向一类模块的上下文结构体，
  Nginx模块有许多种类，不同类模块之间的功能差别很大。
  例如，事件类型的模块主要处理I/O事件相关的功能，
  HTTP类型的模块主要处理HTTP应用层的功能。
  每个模块都有了自己的特性，而ctx将会指向特定类型模块的公共接口。
  例如，在HTTP模块中，ctx需要指向ngx_http_module_t结构体 */
  void *ctx;
  // commands将处理nginx.conf中的配置项，详见第4章
  ngx_command_t *commands;
  /* type表示该模块的类型，它与ctx指针是紧密相关的。
  在官方Nginx中，它的取值范围是以下5种：
    NGX_HTTP_MODULE、NGX_CORE_MODULE、
    NGX_CONF_MODULE、NGX_EVENT_MODULE、
    NGX_MAIL_MODULE。
  实际上，还可以自定义新的模块类型 */
  ngx_uint_t type;
  
  /* 在Nginx的启动、停止过程中，
  以下7个函数指针表示有7个执行点会分别调用这7种方法。
  对于任一个方法而言，如果不需要Nginx在某个时刻执行它，
  那么简单地把它设为NULL空指针即可 */

  /* 虽然从字面上理解应当在master进程启动时回调init_master，
  但到目前为止，框架代码从来不会调用它，因此，可将init_master设为NULL */
  ngx_int_t (*init_master)(ngx_log_t *log); 
  
  /* init_module回调方法在初始化所有模块时被调用。在master/worker模式下，
  这个阶段将在启动worker子进程前完成 */
  ngx_int_t (*init_module)(ngx_cycle_t *cycle); 
  
  /* init_process回调方法在正常服务前被调用。
  在master/worker模式下，多个worker子进程已经产生，
  在每个worker进程的初始化过程会调用所有模块的init_process函数 */
  ngx_int_t (*init_process)(ngx_cycle_t *cycle); 
  
  /* 由于Nginx暂不支持多线程模式，所以init_thread在框架代码中没有被调用过，设为NULL*/
  ngx_int_t (*init_thread)(ngx_cycle_t *cycle); 
  
  // 同上，exit_thread也不支持，设为NULL
  void (*exit_thread)(ngx_cycle_t *cycle); 
  
  /* exit_process回调方法在服务停止前调用。
  在master/worker模式下，worker进程会在退出前调用它 */
  void (*exit_process)(ngx_cycle_t *cycle);
  
  // exit_master回调方法将在master进程退出前被调用
  void (*exit_master)(ngx_cycle_t *cycle); 
  
  /* 以下8个spare_hook变量也是保留字段，目前没有使用，
  但可用Nginx提供的NGX_MODULE_V1_PADDING宏来填充。
  #define NGX_MODULE_V1_PADDING 0, 0, 0, 0, 0, 0, 0, 0*/
  uintptr_t spare_hook0;
  uintptr_t spare_hook1;
  uintptr_t spare_hook2;
  uintptr_t spare_hook3;
  uintptr_t spare_hook4;
  uintptr_t spare_hook5;
  uintptr_t spare_hook6;
  uintptr_t spare_hook7;
};
```

虽然有了注释，但是这个类型仍然有一些地方值得再展开。
1. 定义一个HTTP模块时，务必把type字段设为NGX_HTTP_MODULE。
2. 对于下列回调方法：init_module、init_process、exit_process、exit_master，调用它们的是Nginx的框架代码。而和Http核心模块无关，即使配置中没有配置Http配置块，仍然会被调用。因此，通常开发HTTP模块时都把它们设为NULL空指针。这样，当Nginx不作为Web服务器使用时，不会执行HTTP模块的任何代码。因此可见，```ngx_module_t```内的所有的回调，其实都不是必要的。

因此定义HTTP模块时，最重要的实际是**设置ctx和commands**这两个成员。对于HTTP类型的模块来说，ngx_module_t中的ctx指针必须指向ngx_http_module_t接口（这是HTTP核心模块框架的要求）。下面先来分析ngx_http_module_t结构体的成员。
```c
typedef struct {
  // 解析配置文件前调用
  ngx_int_t (*preconfiguration)(ngx_conf_t *cf);
  // 完成配置文件的解析后调用
  ngx_int_t (*postconfiguration)(ngx_conf_t *cf); 

  /* 当需要创建数据结构用于存储main级别
  （直属于http{...}块的配置项）的全局配置项时，
  可以通过create_main_conf回调方法创建存储全局配置项的结构体 */
  void* (*create_main_conf)(ngx_conf_t *cf); 
  
  // 常用于初始化main级别配置项
  char* (*init_main_conf)(ngx_conf_t *cf, void *conf);
  
  /* 当需要创建数据结构用于存储srv级别
  （直属于虚拟主机server{...}块的配置项）的配置项时，
  可以通过实现create_srv_conf回调方法创建存储srv级别配置项的结构体 */
  void* (*create_srv_conf)(ngx_conf_t *cf);
  
  // merge_srv_conf回调方法主要用于合并main级别和srv级别下的同名配置项
  char* (*merge_srv_conf)(ngx_conf_t *cf, void *prev, void *conf);
  
  /* 当需要创建数据结构用于存储loc级别
  （直属于location{...}块的配置项）的配置项时，
  可以实现create_loc_conf回调方法 */
  void* (*create_loc_conf)(ngx_conf_t *cf);
  
  // merge_loc_conf回调方法主要用于合并srv级别和loc级别下的同名配置项
  char* (*merge_loc_conf)(ngx_conf_t *cf, void *prev, void *conf); 
} ngx_http_module_t;
```

不过，这8个阶段的调用顺序与上述定义的顺序是不同的。在Nginx启动过程中，HTTP框架调用这些回调方法的实际顺序，和具体的nginx.conf配置文件有关。其运行时顺序有可能是这样的：
1. create_main_conf
2. create_srv_conf
3. create_loc_conf
4. preconfiguration
5. init_main_conf
6. merge_srv_conf
7. merge_loc_conf
8. postconfiguration
由于可以不对配置解析过程进行干涉（即不解析配置项的值），因此这8个回调，也都可以设置成NULL。但如果需要接收配置项的值作为控制模块运行的参数，或者想详细控制在HTTP框架的某个阶段，插入一个handler，则一般需要实现某个回调。比如在postconfiguration中进行添加handler，形如
```c
// 这种方式插入的handler们，称为content phase handlers
static ngx_int_t ngx_http_hello_post_conf(ngx_conf_t *cf)
{
  ngx_http_handler_pt        *h;
  ngx_http_core_main_conf_t  *cmcf;

  cmcf = ngx_http_conf_get_module_main_conf(cf, ngx_http_core_module);
  // 选择在CONTENT_PHASE阶段插入处理器
  h = ngx_array_push(&cmcf->phases[NGX_HTTP_CONTENT_PHASE].handlers);
  if (h == NULL) {
    return NGX_ERROR;
  }

  *h = ngx_http_hello_handler;

  return NGX_OK;
}
```

commands数组用于定义模块的配置文件参数，每一个数组元素都是ngx_command_t类型，数组的结尾用ngx_null_command表示。Nginx在解析配置文件中的一个配置项时首先会遍历所有的模块，对于每一个模块而言，即通过遍历commands数组进行，另外，在数组中检查到ngx_null_command时，会停止使用当前模块解析该配置项。每一个ngx_command_t结构体定义了自己感兴趣的一个配置项：

```c
// 每一个command_s，定义了一个配置项
typedef struct ngx_command_s ngx_command_t;
struct ngx_command_s {
  // 配置项名称，如"gzip"
  ngx_str_t name;
  
  /* 配置项类型，type将指定配置项可以出现的位置。
  例如，出现在server{}或location{}中，以及它可以携带的参数个数 */
  ngx_uint_t type;

  // 出现了name中指定的配置项后，将会调用set方法处理配置项的参数
  char* (*set)(ngx_conf_t *cf, ngx_command_t cmd, void *conf);
  
  // 在配置文件中的偏移量
  ngx_uint_t conf;

  /* 通常用于使用预设的解析方法解析配置项，
  这是配置模块的一个优秀设计。它需要与conf配合使用 */
  ngx_uint_t offset;

  // 配置项读取后的处理方法，必须是ngx_conf_post_t结构的指针
  void *post;
};

// 结尾的ngx_null_command只是一个空的ngx_command_t宏，如下所示：
#define ngx_null_command { ngx_null_string, 0, NULL, 0, 0, NULL }
```


### 完整Demo
```c
#include <ngx_config.h>
#include <ngx_core.h>
#include <ngx_http.h>
 
static ngx_int_t ngx_http_hello_handler(ngx_http_request_t *r);
 
static char* ngx_http_hello(ngx_conf_t *cf, ngx_command_t *cmd, void *conf);

/**
 * 处理nginx.conf中的配置命令解析
 * 例如：
 * location /hello {
 *  	hello
 * }
 * 当用户请求:http://127.0.0.1/hello的时候，请求会跳转到hello这个配置上
 * hello的命令行解析回调函数：ngx_http_hello
 */
static ngx_command_t ngx_http_hello_commands[] = {
  {
    ngx_string("hello"),
    NGX_HTTP_MAIN_CONF | NGX_HTTP_SRV_CONF | NGX_HTTP_LOC_CONF | NGX_HTTP_LMT_CONF | NGX_CONF_NOARGS,
    ngx_http_hello,
    NGX_HTTP_LOC_CONF_OFFSET,
    0,
    NULL
  },
  ngx_null_command
};
 
 
/**
 * 模块上下文
 */
static ngx_http_module_t ngx_http_hello_module_ctx = { NULL, NULL, NULL, NULL,
  NULL, NULL, NULL, NULL };
 
/**
 * 模块的定义
 */
ngx_module_t ngx_http_hello_module = {
  NGX_MODULE_V1,
  &ngx_http_hello_module_ctx,
  ngx_http_hello_commands,
  NGX_HTTP_MODULE,
  NULL,
  NULL,
  NULL,
  NULL,
  NULL,
  NULL,
  NULL,
  NGX_MODULE_V1_PADDING
};
 
/**
 * 命令解析的回调函数
 * 该函数中，主要获取loc的配置，并且设置location中的回调函数handler
 */
static char *
ngx_http_hello(ngx_conf_t *cf, ngx_command_t *cmd, void *conf) {
	ngx_http_core_loc_conf_t *clcf;
 
	clcf = ngx_http_conf_get_module_loc_conf(cf, ngx_http_core_module);
	/* 设置回调函数。当请求http://127.0.0.1/hello的时候，会调用此回调函数 */
	clcf->handler = ngx_http_hello_handler;
 
	return NGX_CONF_OK;
}
 
/**
 * 模块回调函数，输出hello world
 */
static ngx_int_t ngx_http_hello_handler(ngx_http_request_t *r) {
  // HTTP方法名检查
	if (!(r->method & (NGX_HTTP_GET | NGX_HTTP_HEAD))) {
		return NGX_HTTP_NOT_ALLOWED;
	}
  // 包体处理，丢弃
	ngx_int_t rc = ngx_http_discard_request_body(r);
	if (rc != NGX_OK) {
		return rc;
	}
  
	ngx_str_t type = ngx_string("text/plain");
	ngx_str_t response = ngx_string("Hello World");
	r->headers_out.status = NGX_HTTP_OK;
	r->headers_out.content_length_n = response.len;
	r->headers_out.content_type = type;
 
	rc = ngx_http_send_header(r);
	if (rc == NGX_ERROR || rc > NGX_OK || r->header_only) {
		return rc;
	}
 
	ngx_buf_t *b;
	b = ngx_create_temp_buf(r->pool, response.len);
	if (b == NULL) {
		return NGX_HTTP_INTERNAL_SERVER_ERROR;
	}
 
	ngx_memcpy(b->pos, response.data, response.len);
	b->last = b->pos + response.len;
	b->last_buf = 1;
 
	ngx_chain_t out;
	out.buf = b;
	out.next = NULL;
 
	return ngx_http_output_filter(r, &out);
}
```

本Demo相对简单，是一个只有无参配置项的模块。在本Demo代码中，并没有显式的指定，处理函数在HTTP框架支持的11个阶段中的哪个阶段生效。以这种方式挂载的handler也被称为content handler。这是因为在该模块的command中，定义的set阶段里，有对该location设置了handler，当框架在NGX_HTTP_CONTENT_PHASE阶段，执行到此处时，会明白该模块对当前location进行了处理以后，不再需要遍历所有的content phase handlers，而是直接执行一个handler，并返回。这部分内容将会在后续讲解原理时再详细展开。

## 核心原理
### 完整配置
在前文的Demo中，所使用的配置项只是一个无参数的```hello```。其作用也非常简单，就是通知Nginx的HTTP框架，在出现该配置项时，启用对应模块。显然HTTP框架需要处理更为复杂的配置项。

配置项是用户和Nginx的接口，对于模块开发者而言，使用```ngx_http_module_t```和```ngx_command_t```对配置项处理，是模块自定义处理业务的入口。总体上来说，对配置项的处理可以分为4个步骤：
1. 创建数据结构用于存储配置项对应的参数。
2. 设定配置项在nginx.conf中出现时的限制条件与回调方法。
3. 实现第2步中的回调方法，或者使用Nginx框架预设的14个回调方法。
4. 合并不同级别的配置块中出现的同名配置项。

```c
// 配置项所能支持的所有数据结构
typedef struct {
  ngx_str_t my_str;
  ngx_int_t my_num;
  ngx_flag_t my_flag;
  size_t my_size;
  ngx_array_t* my_str_array;
  ngx_array_t* my_keyval;
  off_t my_off;
  ngx_msec_t my_msec;
  time_t my_sec;
  ngx_bufs_t my_bufs;
  ngx_uint_t my_enum_seq;
  ngx_uint_t my_bitmask;
  ngx_uint_t my_access;
  ngx_path_t* my_path;
} ngx_http_mytest_conf_t;
```

和前面的简单Demo不同，因为我们要开始真正存储配置项，所以需要创建结构体。HTTP模块一般只需要实现module种的create_loc_conf方法，因为我们只对location在某种URL匹配下的请求处理感兴趣。一个示例的写法如下，
```c
static void* ngx_http_mytest_create_loc_conf(ngx_conf_t cf)
{
  ngx_http_mytest_conf_t mycf;
  mycf = (ngx_http_mytest_conf_t *)ngx_pcalloc(cf->pool, sizeof(ngx_http_mytest_conf_t));
  if (mycf == NULL) {
    return NULL;
  }
  // 值必须进行初始化，否则后面使用预制解析方法时，会抛出异常（duplicate）
  mycf->my_flag = NGX_CONF_UNSET;
  mycf->my_num = NGX_CONF_UNSET;
  mycf->my_str_array = NGX_CONF_UNSET_PTR;
  mycf->my_keyval = NULL;
  mycf->my_off = NGX_CONF_UNSET;
  mycf->my_msec = NGX_CONF_UNSET_MSEC;
  mycf->my_sec = NGX_CONF_UNSET;
  mycf->my_size = NGX_CONF_UNSET_SIZE;
  return mycf;
}
```

为了实现嵌套，或者同级的配置项，Nginx在进行配置解析时，需要从外到内逐层处理。

<span id="Nginx配置文件处理时序图"/>

![Nginx配置文件处理时序图](/images/book/understanding-nginx/config_process_flow.png)

> 图中省略了解析location的过程，实际上和server块的解析非常类似。

自定义模块首先应当设置的就是ctx和command，先看一下command。

command内的type取值，标志了模块配置项的生效方式和参数量等内容。部分内容，列举如下。
| type类型 | type取值 | 意义 |
| --- | --- | --- |
| 处理配置项时获取当前{}配置块的方式 | NGX_DIRECT_CONF | 和NGX_MAIN_CONF一同使用，表示该模块需要解析不在任何{}块内的全局配置，应当仅由NGX_CORE_MODULE使用 |
| 配置项可以在哪些{}块中出现 | NGX_MAIN_CONF | 可以出现在全局，不属于任何{}块 |
| 配置项可以在哪些{}块中出现 | NGX_EVENT_CONF | 配置项可以出现在events块内 |
| 配置项可以在哪些{}块中出现 | NGX_MAIL_MAIN_CONF | 配置项可以出现在mail块或者imap块内 |
| 配置项可以在哪些{}块中出现 | NGX_MALL_SRV_CONF | 配置项可以出现在server{}块内，但该server块必须属于mail{}块或者imap{}块 |
| 配置项可以在哪些{}块中出现 | NGX_HTTP_MAIN_CONF | 配置项可以出现在http{}块内 |
| 配置项可以在哪些{}块中出现 | NGX_HTTP_SRV_CONF | 配置项可以出现在server{}块内，然而该server{}块必须属于http{}块 |
| 配置项可以在哪些{}块中出现 | NGX_HTTP_LOC_CONF | 配置项可以出现在location{}块内，然而该location{}块必须属于http{}块 |
| 配置项可以在哪些{}块中出现 | NGX_HTTP_UPS_CONF | 配置项可以出现在upstream{}块内，然而该upstream{}块必须属于http{}块 |
| 配置项可以在哪些{}块中出现 | NGX_HTTP_SIF_CONF | 配置项可以出现在server{}块内的if{}块中。目前有rewrite模块会使用，该if{}块必须属于http{}块 |
| 配置项可以在哪些{}块中出现 | NGX_HTTP_LIF_CONF | 配置项可以出现在location{}块内的if{}块中。目前仅有rewrite模块会使用，该if{}块必须属于http{}块 |
| 配置项可以在哪些{}块中出现 | NGX_HTTP_LMT_CONF | 配置项可以出现在limit_except{}块内，然而该limit_except{}块必须属于http{}块 |
| 限制配置项的参数个数 | NGX_CONF_NOARGS | 配置项不携带任何参数 |
| 限制配置项的参数个数 | NGX CONF TAKEI | 必须携带1个参数（省略其他类似的） |
| 限制参数的形式 | NGX_CONF_BLOCK | 配置项定义了一种新的配置块，和http、server、location类似 |
| 限制参数的形式 | NGX_CONF_FLAG | 配置项携带的参数只能是1个，且是on或off |

每个进程中都有一个唯一的ngx_cycle_t核心结构体，它有一个成员conf_ctx维护着所有模块的配置结构体，其类型是void****。conf_ctx意义为首先指向一个成员皆为指针的数组，其中每个成员指针又指向另外一个成员皆为指针的数组，第2个子数组中的成员指针才会指向各模块生成的配置结构体。

对配置的处理有两种方式，一种是自己手动去写command中的set函数，另一种是利用nginx提供的自动解析函数。其中HTTP提供的一共有14种。如下表所示
| 方法名 | 行为 |
| --- | --- |
| ngx_conf_set_flag_slot | 处理on、off类配置项参数，且用ngx_flat_t存储 |
| ngx_conf_set_str_slot | 只有一个参数，且用ngx_str_t存储 |
| ngx_conf_set_str_array_slot | 用ngx_str_t为元素的数组，存储该配置项每一次出现后的1个参数 |
| ngx_conf_set_keyval_slot | 用键值存储该配置项每一次出现后的2个参数 |
| ngx_conf_set_num_slot | 存储1个整形参数 |
| ngx_conf_set_size_slot | 存储1个代表空间值的参数 |
| ngx_conf_set_off_slot | 存储1个代表偏移量的参数 |
| ngx_conf_set_msec_slot | 存储1个毫秒级时间参数 |
| ngx_conf_set_sec_slot | 存储2个秒级时间参数 |
| ngx_conf_set_bufs_slot | 存储2个参数，分别是缓冲区数量，缓冲区大小 |
| ngx_conf_set_enum_slot | 存储1个参数，是设定好的枚举字符串 |
| ngx_conf_set_bitmask_slot | 存储1个参数，将设定好的字符串映射到bitmask |
| ngx_conf_set_access_slot | 存储1~3个参数，用于设置文件的读写权限 |
| ngx_conf_set_path_slot | 存储1个参数，用于设置路径 |

如果使用官方提供的配置解析方法，因为这种函数内无法确定配置应当属于MAIN/SERVER/LOCATION的哪一级，所以需要给出conf变量、offset变量，分别用于指示配置项所处的内存偏移位置，以及所解析参数在配置项内的成员的偏移。前者即三种取值之一：```NGX_HTTP_MAIN_CONF_OFFSET```、```NGX_HTTP_SRV_CONF_OFFSET```、```NGX_HTTP_LOC_CONF_OFFSET```。这三个级别也分别对应前面讲述的create_main_conf、create_srv_conf、create_loc_conf方法创建的结构体。默认情况下，父子层级的同名配置项会进行合并，如果不希望直接合并，保留不同级别的配置项，就需要自定义这三个方法。

post在每个配置项解析完成后调用，但其过于灵活，编写起来并不统一。一般模块开发直接使用module种的init_main_conf等方法统一处理解析完的配置项。

上面这段内容可能比较抽象，用一个对```ngx_http_mytest_conf_t```处理的例子说明。
```c
// 注意每一项都需要在create_loc_conf的实现中，给出UNSET含义的初值
static ngx_command_t ngx_http_mytest_commands[] = {
// …
  {
    // 当出现test_flag on/off时，进行解析，解析结果写入my_flag字段
    ngx_string("test_flag"),
    NGX_HTTP_LOC_CONF| NGX_CONF_FLAG,
    ngx_conf_set_flag_slot,
    NGX_HTTP_LOC_CONF_OFFSET,
    offsetof(ngx_http_mytest_conf_t, my_flag),
    NULL
  },
  {
    // 当出现test_str xxxx时解析，写入my_str字段
    ngx_string("test_str"),
    NGX_HTTP_MAIN_CONF|NGX_HTTP_SRV_CONF|NGX_HTTP_LOC_CONF| NGX_CONF_TAKE1,
    ngx_conf_set_str_slot,
    NGX_HTTP_LOC_CONF_OFFSET,
    offsetof(ngx_http_mytest_conf_t, my_str),
    NULL
  },
  ngx_null_command
};
```

实际上offsetof的使用非常灵活，如果我们在```ngx_http_mytest_conf_t```内再使用一层结构体，比如```ngx_http_upstream_conf_t```类型的upstream变量。那么可以用以下的方式，直接将子结构体的配置项数据也填充进去
```c
/*
struct {
  ngx_http_upstream_conf_t upstream;
  ...
} ngx_http_mytest_conf_t;
*/
offsetof(ngx_http_mytest_conf_t, upstream.connect_timeout)
```

如果要完全自行处理配置项及其参数，则需要自行编写set函数，例如
```c
// 假定此次要处理的配置项，是由两个参数构成
typedef struct {
  ngx_str_t my_config_str;
  ngx_int_t my_config_num;
} ngx_http_mytest_conf_t;

static char* ngx_conf_set_myconfig(ngx_conf_t* cf, ngx_command_t cmd, void* conf)
{
  /* 注意，参数conf就是HTTP框架传给用户的在
  ngx_http_mytest_create_loc_conf回调方法中分配的结构体
  ngx_http_mytest_conf_t */
  ngx_http_mytest_conf_t mycf = conf;
  /* cf->args是1个ngx_array_t队列，它的成员都是
  ngx_str_t结构。我们用value指向ngx_array_t的
  elts内容，其中value[1]就是第1个参数，同理，
  value[2]是第2个参数 */
  ngx_str_t value = cf->args->elts;
  // ngx_array_t的nelts表示参数的个数
  if (cf->args->nelts > 1)
  {
    // 直接赋值即可，ngx_str_t结构只是指针的传递
    mycf->my_config_str = value[1];
  }
  if (cf->args->nelts > 2)
  {
    // 将字符串形式的第2个参数转为整型
    mycf->my_config_num = ngx_atoi(value[2].data, value[2].len);
    /* 如果字符串转化整型失败，将报“invalid number”错误，
    Nginx启动失败 */
    if (mycf->my_config_num == NGX_ERROR) {
      return "invalid number";
    }
  }
  // 返回成功
  return NGX_CONF_OK;
}
```

在任意层级的create_xxx_conf中，都会创建对应层级的配置项。但如果需要对配置进行合并，则需要在对应层级之间，实现merge_xxx_conf回调。比如将server和location进行合并。
```c
static char* ngx_http_mytest_merge_loc_conf(ngx_conf_t *cf, void *parent, void *child)
{
  ngx_http_mytest_conf_t *prev = (ngx_http_mytest_conf_t *)parent;
  ngx_http_mytest_conf_t *conf = (ngx_http_mytest_conf_t *)child;
  // 如果父级优先级更高，用父覆盖子级
  ngx_conf_merge_str_value(conf->my_str,
    prev->my_str, "defaultstr");
  return NGX_CONF_OK;
}
```

### 配置模型
前文讲了一个完整的配置项应该如何进行处理。本章节从HTTP框架的视角，描述配置项的解析流程。

HTTP的配置解析流程是从发现HTTP配置块开始的。框架会建立ngx_http_conf_ctx_t结构，定义如下
```c
typedef struct {
  /* 指针数组，数组中的每个元素指向所有HTTP模块
  create_main_conf方法产生的结构体 */
  void *main_conf;
  /* 指针数组，数组中的每个元素指向所有
  HTTP模块create_srv_conf方法产生的结构体 */
  void *srv_conf;
  /* 指针数组，数组中的每个元素指向所有
  HTTP模块create_loc_conf方法产生的结构体 */
  void *loc_conf;
} ngx_http_conf_ctx_t;
```
如定义所示，HTTP框架会为所有的HTTP模块建立3个数组，分别存放代表main级、server级、location级的HTTP模块相关配置。如果在module的设置内，用户放弃了create方法的实现，设置为NULL，则对应模块不会产生结构体存储在这里的ctx中。可以回看一下前面的[Nginx配置文件处理时序图](#Nginx配置文件处理时序图)，处理流程如下。
1. 主循环启动配置文件解析
2. 发现http配置块，启动http框架（ngx_http_module）：
  1. 初始化所有http模块，为其填充index，创建ngx_http_conf_ctx_t
  2. 调用所有模块的create_xxx_conf方法，将指针保存到ngx_http_conf_ctx_t
  3. 调用每个模块的preconfiguration
  4. 循环解析http内的所有配置项
    1. 每次检测到一个配置项后，会遍历所有的http模块，检查它们的ngx_command_t数组中的name项是否与配置项名相同
    2. 调用相关模块command中的set方法
    3. 如果检测到server配置项，将会交给ngx_http_core_module处理
      1. ngx_http_core_module也会建立ngx_http_conf_ctx_t结构，但它只需要调用每个http模块的create_srv_conf、create_loc_conf并保存指针。而它的main_conf指针，**直接指向**创建自己的父http的ngx_http_conf_ctx_t中的main_conf。
      2. 循环解析server块内的配置项
      3. 如果有location配置块，同理

> 注意区分NGX_HTTP_MODULE、NGX_HTTP_CORE_MODULE。

![Nginx配置上下文内存模型示意1](/images/book/understanding-nginx/ngx_conf_ctx.png)

配置上下文内存模型示意图1

![Nginx配置上下文内存模型示意2](/images/book/understanding-nginx/ngx_conf_ctx_2.png)

配置上下文内存模型示意图2

显然这种结构带来了大量冗余。在配置解析过程中，http、server、location的数量之和，是create_loc_conf的调用次数，也是它产生的loc_conf的个数，同理http、server的数量之和，是create_srv_conf的次数。这都是**为了解决同名配置项合并**的问题。

这也是Nginx的一个设计心思，在Http块内的main级别，可以配置server、location级别的配置项的值，这些值就需要分别存储在Http块内的main_conf/srv_conf/loc_conf内，这样才能对后续的server、location子节点起作用。

当你理解了上面的内存结构，接下来需要解决的配置项的合并问题，思路也很显然了，就是从根部的ngx_http_conf_ctx_t开始向下合并。对于每一个HTTP模块，如果他
1. 实现了merge_srv_conf，则将它所属的http块的srv_conf和当前srv级别的srv_conf进行合并
2. 实现了merge_loc_conf，则需要将所属的http块、所属的server块，的loc_conf，分别合并。而且由于location允许嵌套，实际上也会出现location之间的合并。
> Nginx给出了一系列预设的合并方法，可以直接使用。

### 请求的上下文
因为Nginx使用了异步的编程模型，所以需要使用一种方式，为每一个参与请求处理的模块，保留每一个请求所关联的上下文内容。这种上下文在一个模块参与请求处理时创建（在内存池中），当请求响应处理结束后释放。不同模块之间的上下文相互独立。

在设计上，并不是由进程来维护这个上下文，而是由请求自己保管这个上下文。上下文的生命周期和请求的生命周期一致。上下文的设置和使用从两个宏开始
```c
// 此处的r就是ngx_http_request_t，module是各模块定义的ngx_module_t类型变量
// 从这里也可以看出，任何对请求上下文的读写操作，都是从请求处开始的
#define ngx_http_get_module_ctx(r, module) (r)->ctx[module.ctx_index]
#define ngx_http_set_ctx(r, c, module) r->ctx[module.ctx_index] = c;
```

在前面的完整Demo中，我们在handler内直接处理了请求，如果一个请求更为复杂，可能会涉及到多个handler的调用，那就可以使用本章节所说的请求上下文，例如
```c
// 模块自定义handler
static ngx_int_t ngx_http_mytest_handler(ngx_http_request_t *r)
{
  // 首先调用ngx_http_get_module_ctx宏来获取上下文结构体
  ngx_http_mytest_ctx_t* myctx = ngx_http_get_module_ctx(r,ngx_http_mytest_module);
  // 如果之前没有设置过上下文，那么应当返回NULL
  if (myctx == NULL)
  {
    /* 必须在当前请求的内存池r->pool中分配上下文结构体，这样请求结束时结构体占用的内存才会释放 */
    myctx = ngx_palloc(r->pool, sizeof(ngx_http_mytest_ctx_t));
    if (myctx == NULL)
    {
      return NGX_ERROR;
    }
    // 将刚分配的结构体设置到当前请求的上下文中
    ngx_http_set_ctx(r,myctx,ngx_http_mytest_module);
  }
  // 之后可以任意使用myctx这个上下文结构体
  // ...
}
```

### 访问第三方服务
访问第三方服务是Nginx能够作为反向代理所需的核心能力。也为模块开发提供了一个强大的对外接口。Nginx提供了两种全异步方式来与第三方服务器通信：upstream与subrequest。其中subrequest实际上也是基于upstream模块实现。区别主要在于对请求的业务需要
1. upstream的重点在于透传，高效率的作为代理，透传http数据
2. subrequest的重点正如其名字：子请求。Nginx会根据上游第三方服务的响应情况，组建响应结果，并反馈给原始客户端。

#### upstream
upstream的使用方式并不复杂，它提供了8个回调方法，用户只需要视自己的需要实现其中几个回调方法就可以了。而upstream在```ngx_http_request_s```结构体中存储，用户在处理时可以进行操作。发起upstream的处理流程可以概括为四个步骤

![启动upstream的流程](/images/book/understanding-nginx/upstream_in_handler.png)

> 注1：上游服务器的IP地址可以从配置文件中，也可以从HTTP请求头部中，或者任意自定义

> 注2：ngx_http_upstream_init方法启用upstream机制也是一个异步操作，且调用后handler应当返回NGX_DONE（表示后续模块不需要再处理），为了保证nginx不销毁请求，需要将引用计数+1，即```r->main->count++;```。

![upstream的处理流程](/images/book/understanding-nginx/upstream_process_flow.png)

如上图所示的是upstream的一般执行流程，这个流程在ngx_http_upstream_init执行之后，由HTTP框架负责进行处理。为了进一步理解upstream，这里贴出ngx_http_upstream_t的部分结构
```c
typedef struct ngx_http_upstream_s ngx_http_upstream_t; struct ngx_http_upstream_s {
  // …
  /* request_bufs决定发送什么样的请求给上游服务器，在实现
  create_request方法时需要设置它*/
  ngx_chain_t *request_bufs;
  // upstream访问时的所有限制性参数，包括连接、发送和接收超时时间等
  ngx_http_upstream_conf_t *conf;
  // 通过resolved可以直接指定上游服务器地址，地址个数、地址数组
  ngx_http_upstream_resolved_t *resolved;
  /* buffer成员存储接收自上游服务器发来的响应内容，由于它会被复用，所以具有下列多种意义：
    a)在使用process_header方法解析上游响应的包头时，buffer中将会保存完整的响应包头；
    b)当下面的buffering成员为1，而且此时upstream是向下游转发上游的包体时，buffer没有意义；
    c)当buffering标志位为0时，buffer缓冲区会被用于反复地接收上游的包体，进而向下游转发；
    d)当upstream并不用于转发上游包体时，buffer会被用于反复接收上游的包体，HTTP模块实现的
      input_filter方法需要关注它 */
  ngx_buf_t buffer;
  // 构造发往上游服务器的请求内容
  ngx_int_t (*create_request)(ngx_http_request_t *r);
  /* 收到上游服务器的响应后就会回调process_header方法。
  如果process_header返回NGX_AGAIN，那么是在告诉
  upstream还没有收到完整的响应包头，此时，对于本次
  upstream请求来说，再次接收到上游服务器发来的TCP流时，
  还会调用process_header方法处理，直到process_header函数返回非
  NGX_AGAIN值这一阶段才会停止 */
  ngx_int_t (*process_header)(ngx_http_request_t *r);
  // 销毁upstream请求时调用
  void (*finalize_request)(ngx_http_request_t *r, ngx_int_t rc); 
  
  // 5个可选的回调方法
  // 自定义接收包体前的初始化阶段（内存等），不定义的话，upstream将会使用默认实现
  ngx_int_t (*input_filter_init)(void *data);
  // 自定义接收包体，不定义也有默认实现
  ngx_int_t (*input_filter)(void *data, ssize_t bytes);
  // upstream时，如果发现已向上游发送请求，但连接断开，会尝试重连，并调用该函数
  ngx_int_t (*reinit_request)(ngx_http_request_t *r);
  void (*abort_request)(ngx_http_request_t *r);
  ngx_int_t (*rewrite_redirect)(ngx_http_request_t *r, ngx_table_elt_t *h, size_t prefix);
  // SSL协议访问上游服务器
  unsigned ssl:1;
  /* 在向客户端转发上游服务器的包体时才有用。当buffering为1时，
  表示使用多个缓冲区以及磁盘文件来转发上游的响应包体。
  当Nginx与上游间的网速远大于Nginx与下游客户端间的网速时，
  让Nginx开辟更多的内存甚至使用磁盘文件来缓存上游的响应包体，
  这是有意义的，它可以减轻上游服务器的并发压力。
  当buffering为0时，表示只使用上面的这一个buffer缓冲区来向下游转发响应包体 */
  unsigned buffering:1;
  // …
};
```

从结构中可知，所提供的8个回调方法种，三个是必须实现的，其余5个则可以根据需要进行实现。另外需要明白的一点是，当我们在自己的模块中主动使用upstream时，upstream配置块和我们没有关系。我们需要自己为upstream的相关配置引入配置项数据。

另外在nginx做反向代理时，upstream内部直接提供了所谓的pass header、hide header的字段，用于透传、隐藏某型http头部字段。这样做的目的有很多，比如
1. 安全性提高：通过隐藏或修改服务器的一些敏感信息（如Server头部、X-Powered-By头部等），可以减少潜在攻击者获取的信息量，降低被攻击的风险。例如，如果攻击者不知道后端使用的是哪个版本的Apache或PHP，那么他们利用这些软件已知漏洞的难度会增加。
1. 统一响应头: 在使用Nginx作为反向代理服务多个应用时，不同的应用可能会生成不同的响应头。通过Nginx统一修改或移除某些响应头，可以对外提供更一致的响应头信息，提高整体的专业度和美观度。
1. 性能优化：虽然不是主要原因，但通过移除一些不必要的头信息，可以减少HTTP响应的大小，从而微小提升网络传输效率。

遵守策略或法规：在某些情况下，为了符合公司策略或者遵守某些法律法规的要求，可能需要隐藏或修改响应头中的信息。

![nginx处理upstream流程](/images/book/understanding-nginx/nginx-upstream-event.png)

如上图所示的是一次create_request回调的序列图。对应的finalize_request在请求被销毁前一定会被调用（无论upstream请求成功与否）。

![ngxin处理upstream-processheader流程](/images/book/understanding-nginx/upstream-process-header.png)

上图所示的是请求第三方服务中，process_header的流程。如果buffer缓冲区全满却还没有解析到完整的响应头部（也就是说，process_header一直在返回NGX_AGAIN）。

由于upstream的例子较长，所以本文在最后给出了一个[upstream示例](#upstream)

#### subrequest
相比于使用upstream实现访问第三方模块，subrequest提供了更好的封装，使用也更简单。主要有4步
1. 在nginx.conf文件中配置好子请求的处理方式。
1. 在父请求的处理中启动subrequest子请求。
1. 实现子请求执行结束时的回调方法。
1. 实现父请求被激活时的回调方法。

子请求subrequest的生命周期，是从父请求中创建子请求开始的。在父请求的处理方法返回NGX_DONE后，框架开始执行子请求。启动执行时的时序图如下图所示。
![子请求的创建](/images/book/understanding-nginx/subrequest-create-seq.png)

子请求的发送顺序有两种方式，一种是不控制的异步发送，一种是postpone模块提供按照链表中顺序的发送方式。后者还可以通过ngx_http_postpone_filter进一步在每个子请求响应中控制、调整转发下一个子请求的方式。

```c
// 本例子针对新浪财经股票板块，注意2024年3月6日，此时需要为请求添加Referer才能不被服务器拒绝，配置如下
/*
location /list {
  # 子请求依然可以从配置中寻找对应的location，这是一个递归的过程
  proxy_pass http://hq.sinajs.cn;
  # 为了防止被新浪拒绝
  proxy_set_header Referer http://finance.sina.com.cn;
  # 不进行压缩
  proxy_set_header Accept-Encoding "";
}

location /query {
  # 用于启动自定义模块
  mysubrequest;
}

*/

#include <ngx_config.h>
#include <ngx_core.h>
#include <ngx_http.h>
static ngx_int_t ngx_http_mysubrequest_handler(ngx_http_request_t *r);
 
static char* ngx_http_mysubrequest(ngx_conf_t *cf, ngx_command_t *cmd, void *conf);

static void mysubrequest_post_handler(ngx_http_request_t *r);
 
static ngx_command_t ngx_http_mysubrequest_commands[] = {
		{
				ngx_string("mysubrequest"),
				NGX_HTTP_MAIN_CONF | NGX_HTTP_SRV_CONF | NGX_HTTP_LOC_CONF | NGX_HTTP_LMT_CONF | NGX_CONF_NOARGS,
				ngx_http_mysubrequest,
				NGX_HTTP_LOC_CONF_OFFSET,
				0,
				NULL
		},
		ngx_null_command
};
 
 
/**
 * 模块上下文
 */
static ngx_http_module_t ngx_http_mysubrequest_module_ctx = { NULL, NULL, NULL, NULL,
		NULL, NULL, NULL, NULL };
 
/**
 * 模块的定义
 */
ngx_module_t ngx_http_mysubrequest_module = {
    NGX_MODULE_V1,
    &ngx_http_mysubrequest_module_ctx,
    ngx_http_mysubrequest_commands,
    NGX_HTTP_MODULE,
    NULL,
    NULL,
    NULL,
    NULL,
    NULL,
    NULL,
    NULL,
    NGX_MODULE_V1_PADDING
};

static char* ngx_http_mysubrequest(ngx_conf_t *cf, ngx_command_t *cmd, void *conf) {
	ngx_http_core_loc_conf_t *clcf;
 
	clcf = ngx_http_conf_get_module_loc_conf(cf, ngx_http_core_module);
	clcf->handler = ngx_http_mysubrequest_handler;
 
	return NGX_CONF_OK;
}

typedef struct {
    ngx_str_t stock[6];
} ngx_http_mysubrequest_ctx_t;

// 子请求的回调
static ngx_int_t mysubrequest_subrequest_post_handler(ngx_http_request_t *r, void *data, ngx_int_t rc) {
  // 当前请求r是子请求，它的parent成员指向父请求
  ngx_http_request_t *pr = r->parent;
  /* 注意，由于上下文是保存在父请求中的，所以要由pr取上下文。其实有更简单的方法，即参数
  data就是上下文，初始化subrequest时就对其进行设置。这里仅为了说明如何获取到父请求的上下文 */
  ngx_http_mysubrequest_ctx_t* myctx = ngx_http_get_module_ctx(pr,ngx_http_mysubrequest_module);
  pr->headers_out.status = r->headers_out.status;
  /* NGX_HTTP_OK（也就是200），则意味着访问新浪服务器成功，接着将开始解析HTTP包体 */
  if (r->headers_out.status == NGX_HTTP_OK) {
    int flag = 0;
    /* 在不转发响应时，buffer中会保存上游服务器的响应。特别是在使用反向代理模块访问上游服务器时，如果它使用
    upstream机制时没有重定义input_filter方法，upstream机制默认的input_filter方法会试图把所有的上游响应全部保存到buffer缓冲区中 */
    ngx_buf_t* pRecvBuf = &r->upstream->buffer;
    /* 以下开始解析上游服务器的响应，并将解析出的值赋到上下文结构体myctx->stock数组中 */
    // 这里的解析是因为我们的访问是使用了对新浪财经板块的引用，已知他的返回结构，做了对应的解析
    for (;pRecvBuf->pos != pRecvBuf->last; pRecvBuf->pos++) {
      if (*pRecvBuf->pos == ',' || *pRecvBuf->pos == '\"') {
        if (flag > 0)
        {
          myctx->stock[flag-1].len = pRecvBuf->pos-myctx->stock[flag-1].data;
        }
        flag++;
        myctx->stock[flag-1].data = pRecvBuf->pos+1;
      }
      if (flag > 6)
        break;
    }
  }
  // 设置接下来父请求的回调方法，这一步很重要
  pr->write_event_handler = mysubrequest_post_handler;
  return NGX_OK;
}

// 父请求的回调
static void mysubrequest_post_handler(ngx_http_request_t *r) {
  // 如果没有返回200，则直接把错误码发回用户
  if (r->headers_out.status != NGX_HTTP_OK) {
    ngx_http_finalize_request(r, r->headers_out.status); return;
  }
  // 当前请求是父请求，直接取其上下文
  ngx_http_mysubrequest_ctx_t* myctx = ngx_http_get_module_ctx(r,ngx_http_mysubrequest_module);
  /* 定义发给用户的HTTP包体内容，格式为：stock[…],Today current price: …, volumn: … */
  ngx_str_t output_format = ngx_string("stock[%V],Today current price: %V, volumn: %V");
  // 计算待发送包体的长度
  int bodylen = output_format.len + myctx->stock[0].len +myctx->stock[1].len+myctx->stock[4].len-6;
  r->headers_out.content_length_n = bodylen;
  //
  ngx_buf_t* b = ngx_create_temp_buf(r->pool, bodylen);
  ngx_snprintf(b->pos, bodylen, (char*)output_format.data, &myctx->stock[0],&myctx->stock[1],&myctx->stock[4]);
  b->last = b->pos + bodylen;
  b->last_buf = 1;
  ngx_chain_t out;
  out.buf = b;
  out.next = NULL;
  // 设置Content-Type，注意，在汉字编码方面，新浪服务器使用了GBK
  static ngx_str_t type = ngx_string("text/plain; charset=GBK");
  r->headers_out.content_type = type;
  r->headers_out.status = NGX_HTTP_OK;
  r->connection->buffered |= NGX_HTTP_WRITE_BUFFERED;
  ngx_int_t ret = ngx_http_send_header(r);
  ret = ngx_http_output_filter(r, &out);
  /* ngx_http_finalize_request结束请求，因为这时HTTP框架不会再帮忙调用它 */
  ngx_http_finalize_request(r, ret);
}

static ngx_int_t ngx_http_mysubrequest_handler(ngx_http_request_t *r) {
  // 创建HTTP上下文
  ngx_http_mysubrequest_ctx_t* myctx = ngx_http_get_module_ctx(r,ngx_http_mysubrequest_module);
  if (myctx == NULL)
  {
    myctx = ngx_palloc(r->pool, sizeof(ngx_http_mysubrequest_ctx_t));
    if (myctx == NULL)
    {
      return NGX_ERROR;
    }
    // 将上下文设置到原始请求r中
    ngx_http_set_ctx(r,myctx,ngx_http_mysubrequest_module);
  }
  // ngx_http_post_subrequest_t结构体会决定子请求的回调方法，参见
  ngx_http_post_subrequest_t *psr = ngx_palloc(r->pool, sizeof(ngx_http_post_subrequest_t));
  if (psr == NULL) {
    return NGX_HTTP_INTERNAL_SERVER_ERROR;
  }
  // 设置子请求回调方法为mysubrequest_subrequest_post_handler
  psr->handler = mysubrequest_subrequest_post_handler;

  /* 将data设为myctx上下文，这样回调mysubrequest_subrequest_post_handler时传入的data参数就是myctx*/
  psr->data = myctx;
  /* 子请求的URI前缀是/list，这是因为访问新浪服务器的请求必须是类似/list=s_sh000001的URI，
  这与在nginx.conf中配置的子请求location的URI是一致的 */
  ngx_str_t sub_prefix = ngx_string("/list=");
  ngx_str_t sub_location;
  sub_location.len = sub_prefix.len + r->args.len;
  sub_location.data = ngx_palloc(r->pool, sub_location.len);
  ngx_snprintf(sub_location.data, sub_location.len, "%V%V",&sub_prefix,&r->args);
  // 作为subrequest，将多个子请求合并为最终响应结果返回给父请求的发起方
  ngx_log_error(NGX_LOG_DEBUG, r->connection->log, 0, "subrequest url request content====> %V", &sub_location);
  // sr
  ngx_http_request_t *sr;
  /* 调用ngx_http_subrequest创建子请求，它只会返回NGX_OK或者NGX_ERROR。返回NGX_OK时，sr已经是合法的子请求。
  注意，这里的NGX_HTTP_SUBREQUEST_IN_MEMORY参数将告诉upstream模块把上游服务器的响应全部保存在子请求的
  sr->upstream->buffer内存缓冲区中 */
  ngx_int_t rc = ngx_http_subrequest(r, &sub_location, NULL, &sr, psr, NGX_HTTP_SUBREQUEST_IN_MEMORY);
  if (rc != NGX_OK) {
    return NGX_ERROR;
  }
  // 必须返回NGX_DONE，原因同upstream
  return NGX_DONE;
}
```

### HTTP过滤器
前面的所有NGinx模块都有一个类似的特点，他们会对所有的用户请求，或者对某些匹配了location的请求产生作用。但是有一个明显限制，如果希望多个模块都对一个请求产生作用，则往往需要用subrequest来完成，将一个请求拆解成多个子请求交给对应的不同模块，再进行处理，这很麻烦。

为了解决这个问题，Nginx支持HTTP过滤器，可以同时由多个过滤器对同一个HTTP请求产生作用。HTTP过滤模块主要是来处理一些附加的功能，如gzip过滤模块可以把发送给用户的静态文件进行gzip压缩处理后再发出去，image_filter这个第三方过滤模块可以将图片类的静态文件制作成缩略图。而且，这些过滤模块的效果是可以根据需要叠加的，比如先由not_modify过滤模块处理请求中的浏览器缓存信息，再交给range过滤模块处理HTTP range协议（支持断点续传），然后交由gzip过滤模块进行压缩，可以看到，一个请求经由各HTTP过滤模块流水线般地依次进行处理了。

值得注意的是，HTTP过滤模块只处理从服务端发往客户端的数据（入口是ngx_http_send_header、ngx_http_output_filter）。而不处理客户端发往服务端的。

过滤模块实现起来相对简单，只需要根据需要实现如下两个方法
```c
// 对头部过滤
typedef ngx_int_t (*ngx_http_output_header_filter_pt)(ngx_http_request_t r);
// 对包体过滤
typedef ngx_int_t (ngx_http_output_body_filter_pt)(ngx_http_request_t r, ngx_chain_t chain);
```

HTTP框架对过滤模块的使用也很简单，就是在提交向客户端响应数据时，遍历如下两个链表，并依次执行
```c
// 过滤器链表头部
extern ngx_http_output_header_filter_pt ngx_http_top_header_filter;
// 过滤器链表包体
extern ngx_http_output_body_filter_pt ngx_http_top_body_filter;

// 在自定义编码实际编写时，ngx_http_next_header_filter和ngx_http_next_body_filter都必须是static静态变量
```

但当我们再往下ngx_http_output_header_filter_pt的结构式，可能会令人很疑惑，这并不是常规的链表。实际上在编译Nginx源代码时，已经定义了一个由所有HTTP过滤模块组成的单链表，这个单链表与一般的链表是不一样的，它有另类的风格：链表的每一个元素都是一个独立的C源代码文件，而这个C源代码文件会通过两个static静态指针（分别用于处理HTTP头部和HTTP包体）再指向下一个文件中的过滤方法。因此自定义过滤器，在添加到链表这一步的**写法必须是固定**的。在每一次HTTP过滤模块的初始化过程中，用户将原始头部保存到next，并将当前模块的过滤器方法保存到头部。整体上的效果形如

```c
static ngx_int_t ngx_http_myfilter_init(ngx_conf_t *cf) {
    // 插入到头部处理方法链表的首部
    ngx_http_next_header_filter = ngx_http_top_header_filter;
    ngx_http_top_header_filter = ngx_http_myfilter_header_filter;
    //
    ngx_http_next_body_filter = ngx_http_top_body_filter;
    ngx_http_top_body_filter = ngx_http_myfilter_body_filter;
    return NGX_OK;
}

/*
其他各种模块
static ngx_int_t ngx_xxx_filter_init(ngx_conf_t *cf) {  ... }
*/

// 查看代码可知write_filter是最初的链表头，write_filter也是最早进行初始化的
static ngx_int_t ngx_http_write_filter_init(ngx_conf_t *cf)
{
    ngx_http_top_body_filter = ngx_http_write_filter;
    return NGX_OK;
}

// 实际使用中
static ngx_int_t ngx_http_myfilter_header_filter(ngx_http_request_t *r) {
  // ...
  // 调用下一个
  return ngx_http_next_header_filter(r);
}
```

每一个过滤器模块在初始化的时候，都是将自己插入到链表的首部（而且NGinx的链表加入就是从首部开始的），因此实际上执行的顺序是初始化的顺序的逆序。而过滤模块的初始化顺序则和普通模块一样，受ngx_modules数组控制。由于官方有一些过滤模块，因此自定义的过滤模块会在数组中处在一个特定的位置。其执行顺序是
1. ngx_http_not_modified_filter_module：只处理头部，根据缓存判断用户响应是否未修改，决定是否发送304响应
2. ngx_http_range_body_filter_module：处理请求中的range信息，返回包体中的指定部分给用户
3. ngx_http_copy_filter_module：仅对包体做处理，处理ngx_chain_t结构体
4. ngx_http_headers_filter_module：仅对包头做处理，在nginx.conf中修改相关配置，最终回在本过滤器中用于增删改http头部字段
5. 第三方HTTP过滤模块
6. ngx_http_userid_filter_module：仅对头部，基于cookie做认证管理
7. ngx_http_charset_filter_module：按nginx.conf的配置，改动编码返回给用户
8. ngx_http_ssi_filter_module：支持SSI，将文件内容包含在网页中返回
9. ngx_http_postpone_filter_module：仅对HTTP包体处理，将子请求有序返回响应
10. ngx_http_gzip_filter_module：对包体压缩
11. ngx_http_range_header_filter_module：支持range协议
12. ngx_http_chunked_filter_module：支持chunk编码，注意chunk和range的方式不同，chunk更接近于流式，是不知道Content-Length的长度的
13. ngx_http_header_filter_module：仅对头部，用r->headers_out填写头部
14. ngx_http_write_filter_module：仅对包体，负责发送响应

链表结构虽然带来了简单的连续调用方案，但是有一个问题就是无法对请求进行直接的过滤。因此需要在处理过程中，根据请求的上下文，由用户来计算当前请求**是否满足使用该http过滤器**。这一步不像之前的http模块，能在初始化时将回调嵌入http框架中。

和其他HTTP模块一样，HTTP过滤器的主要区别是在config、源文件中修改响应的模块类型，比如config中modules变量要更改为HTTP_FILTER_MODULES。下面提供一个完整的demo。
```c
#include <ngx_config.h>
#include <ngx_core.h>
#include <ngx_http.h>
#include <ngx_log.h>

typedef struct {
    ngx_flag_t enable;
} ngx_http_myfilter_conf_t;

static void* ngx_http_myfilter_create_conf(ngx_conf_t *cf) {
    ngx_http_myfilter_conf_t *mycf;
    // 创建存储配置项的结构体
    mycf = (ngx_http_myfilter_conf_t *)ngx_pcalloc(cf->pool, sizeof(ngx_http_myfilter_conf_t));
    if (mycf == NULL) {
        return NULL;
    }
    // ngx_flat_t类型的变量。如果使用预设函数ngx_conf_set_flag_slot解析配置项参数，那么必须初始化为NGX_CONF_UNSET
    mycf->enable= NGX_CONF_UNSET;
    return mycf;
}

static char* ngx_http_myfilter_merge_conf(ngx_conf_t *cf, void *parent, void *child) {
    ngx_http_myfilter_conf_t *prev = (ngx_http_myfilter_conf_t *)parent;
    ngx_http_myfilter_conf_t *conf = (ngx_http_myfilter_conf_t *)child; 
    //ngx_flat_t类型的配置项enable
    ngx_conf_merge_value(conf->enable, prev->enable, 0);
    return NGX_CONF_OK;
}

typedef struct {
    ngx_int_t add_prefix;
} ngx_http_myfilter_ctx_t;

static ngx_command_t ngx_http_myfilter_commands[] = {
    {
        ngx_string("add_prefix"),
        NGX_HTTP_MAIN_CONF|NGX_HTTP_SRV_CONF|NGX_HTTP_LOC_CONF|NGX_HTTP_LMT_CONF|NGX_CONF_FLAG,
        ngx_conf_set_flag_slot,
        NGX_HTTP_LOC_CONF_OFFSET,
        offsetof(ngx_http_myfilter_conf_t, enable),
        NULL
    },
    ngx_null_command
};

// 前置声明
static ngx_int_t ngx_http_myfilter_init(ngx_conf_t *cf);

static ngx_http_module_t ngx_http_myfilter_module_ctx = {
    NULL, /* preconfiguration方法 */
    ngx_http_myfilter_init, /* postconfiguration方法 */
    NULL, /* create_main_conf方法 */
    NULL, /* init_main_conf方法 */
    NULL, /* create_srv_conf方法 */
    NULL, /* merge_srv_conf方法 */
    ngx_http_myfilter_create_conf,/* create_loc_conf方法 */
    ngx_http_myfilter_merge_conf /* merge_loc_conf方法 */
};

ngx_module_t ngx_http_myfilter_module = {
    NGX_MODULE_V1,
    &ngx_http_myfilter_module_ctx, /* module context */
    ngx_http_myfilter_commands, /* module directives */
    NGX_HTTP_MODULE, /* module type */
    NULL, /* init master */
    NULL, /* init module */
    NULL, /* init process */
    NULL, /* init thread */
    NULL, /* exit thread */
    NULL, /* exit process */
    NULL, /* exit master */
    NGX_MODULE_V1_PADDING
};

static ngx_http_output_header_filter_pt ngx_http_next_header_filter;
static ngx_http_output_body_filter_pt ngx_http_next_body_filter;

// 前置声明
static ngx_int_t ngx_http_myfilter_header_filter(ngx_http_request_t *r);
static ngx_int_t ngx_http_myfilter_body_filter(ngx_http_request_t *r, ngx_chain_t *in);

static ngx_int_t ngx_http_myfilter_init(ngx_conf_t *cf) {
    // 插入到头部处理方法链表的首部
    ngx_http_next_header_filter = ngx_http_top_header_filter;
    ngx_http_top_header_filter = ngx_http_myfilter_header_filter;
    //
    ngx_http_next_body_filter = ngx_http_top_body_filter;
    ngx_http_top_body_filter = ngx_http_myfilter_body_filter;
    return NGX_OK;
}

static ngx_str_t filter_prefix = ngx_string("[my filter prefix]");

static ngx_int_t ngx_http_myfilter_header_filter(ngx_http_request_t *r) {
    ngx_http_myfilter_ctx_t *ctx;
    ngx_http_myfilter_conf_t *conf;
    /* 如果不是返回成功，那么这时是不需要理会是否加前缀的，直接交由下一个过滤模块处理响应码非200的情况 */
    if (r->headers_out.status != NGX_HTTP_OK) {
        return ngx_http_next_header_filter(r);
    }
    // 获取HTTP上下文
    ctx = ngx_http_get_module_ctx(r, ngx_http_myfilter_module);
    if (ctx) {
        /* 该请求的上下文已经存在，这说明ngx_http_myfilter_header_filter已经被调用过1次，直接交由下一个过滤模块处理 */
        return ngx_http_next_header_filter(r);
    }
    // 获取存储配置项的ngx_http_myfilter_conf_t结构体
    conf = ngx_http_get_module_loc_conf(r, ngx_http_myfilter_module);
    /* 如果enable成员为0，也就是配置文件中没有配置add_prefix配置项，或者add_prefix配置项的参数值是off，那么这时直接交由下一个过滤模块处理 */
    if (conf->enable == 0) {
        return ngx_http_next_header_filter(r);
    }
    // 构造HTTP上下文结构体ngx_http_myfilter_ctx_t
    ctx = ngx_pcalloc(r->pool, sizeof(ngx_http_myfilter_ctx_t));
    if (ctx == NULL) {
        return NGX_ERROR;
    }
    // add_prefix为0表示不加前缀
    ctx->add_prefix = 0;
    // 将构造的上下文设置到当前请求中
    ngx_http_set_ctx(r, ctx, ngx_http_myfilter_module);
    // myfilter过滤模块只处理Content-Type是“text/plain”类型的HTTP响应
    if (r->headers_out.content_type.len >= sizeof("text/plain") - 1
        && ngx_strncasecmp(r->headers_out.content_type.data, (u_char *) "text/plain",sizeof("text/plain") - 1) == 0) {
        // 设置为1表示需要在HTTP包体前加入前缀
        ctx->add_prefix = 1;
        /* 当处理模块已经在Content-Length中写入了HTTP包体的长度时，由于我们加入了前缀字符串
        ，所以需要把这个字符串的长度也加入到Content-Length中*/
        if (r->headers_out.content_length_n > 0)
            r->headers_out.content_length_n += filter_prefix.len;
    }
    // 交由下一个过滤模块继续处理
    return ngx_http_next_header_filter(r);
}

static ngx_int_t ngx_http_myfilter_body_filter(ngx_http_request_t *r, ngx_chain_t *in) {
    ngx_http_myfilter_ctx_t *ctx;
    ctx = ngx_http_get_module_ctx(r, ngx_http_myfilter_module);
    /* 如果获取不到上下文，或者上下文结构体中的add_prefix为0或者2时，都不会添加前缀，这时直接交给下一个HTTP过滤模块处理 */
    if (ctx == NULL || ctx->add_prefix != 1) {
        return ngx_http_next_body_filter(r, in);
    }
    /* 将add_prefix设置为2，这样即使ngx_http_myfilter_body_filter再次回调时，也不会重复添加前缀 */
    ctx->add_prefix = 2;
    // 从请求的内存池中分配内存，用于存储字符串前缀
    ngx_buf_t* b = ngx_create_temp_buf(r->pool, filter_prefix.len);
    // 将ngx_buf_t中的指针正确地指向filter_prefix字符串
    b->start = b->pos = filter_prefix.data; b->last = b->pos + filter_prefix.len;
    /* 从请求的内存池中生成ngx_chain_t链表，将刚分配的ngx_buf_t设置到buf成员中，并将它添加到原先待发送的HTTP包体前面 */
    ngx_chain_t *cl = ngx_alloc_chain_link(r->pool);
    cl->buf = b;
    cl->next = in;
    // 调用下一个模块的HTTP包体处理方法，注意，这时传入的是新生成的cl链表
    return ngx_http_next_body_filter(r, cl);
}
```

### 进程

### HTTP框架
HTTP框架中最重要的就是11个处理阶段，而作为第三方模块，一般只介入其中的7个阶段处理。这11个阶段如下所示
```c
typedef enum {
  // 在接收到完整的HTTP头部后处理的HTTP阶段
  NGX_HTTP_POST_READ_PHASE = 0,
  /* 在还没有查询到URI匹配的location前，
  这时rewrite重写URL也作为一个独立的HTTP阶段 */
  NGX_HTTP_SERVER_REWRITE_PHASE,

  /* 根据URI寻找匹配的location，
  这个阶段通常由ngx_http_core_module模块实现，
  不建议其他HTTP模块重新定义这一阶段的行为 */
  NGX_HTTP_FIND_CONFIG_PHASE,

  /* 在NGX_HTTP_FIND_CONFIG_PHASE阶段之后重写URL,
  与NGX_HTTP_SERVER_REWRITE_PHASE阶段显然是不同的，
  因为这两者会导致查找到不同的location块（location是与URI进行匹配的） */
  NGX_HTTP_REWRITE_PHASE,

  /* 这一阶段是用于在rewrite重写URL后重新跳到
  NGX_HTTP_FIND_CONFIG_PHASE阶段，找到与新的
  URI匹配的location。所以，这一阶段是无法由第三方
  HTTP模块处理的，而仅由ngx_http_core_module模块使用 */
  NGX_HTTP_POST_REWRITE_PHASE,

  /* 处理NGX_HTTP_ACCESS_PHASE阶段前，
  HTTP模块可以介入的处理阶段  */
  NGX_HTTP_PREACCESS_PHASE,

  /* 这个阶段用于让HTTP模块判断是否允许这个请求访问
  Nginx服务器 */
  NGX_HTTP_ACCESS_PHASE,

  /*当NGX_HTTP_ACCESS_PHASE阶段中HTTP模块的
  handler处理方法返回不允许访问的错误码时（实际是
  NGX_HTTP_FORBIDDEN或者NGX_HTTP_UNAUTHORIZED），
  这个阶段将负责构造拒绝服务的用户响应。
  所以，这个阶段实际上用于给NGX_HTTP_ACCESS_PHASE阶段收尾 */
  NGX_HTTP_POST_ACCESS_PHASE,
  
  /* 这个阶段完全是为了try_files配置项而设立的。
  当HTTP请求访问静态文件资源时，
  try_files配置项可以使这个请求顺序地访问多个静态文件资源，
  如果某一次访问失败，则继续访问try_files中指定的下一个静态资源。
  另外，这个功能完全是在NGX_HTTP_TRY_FILES_PHASE阶段中实现的 */
  NGX_HTTP_TRY_FILES_PHASE,

  // 用于处理HTTP请求内容的阶段，这是大部分HTTP模块最喜欢介入的阶段
  NGX_HTTP_CONTENT_PHASE,

  /* 处理完请求后记录日志的阶段。例如，
  ngx_http_log_module模块就在这个阶段中加入了一个
  handler处理方法，使得每个HTTP请求处理完毕后会记录access_log日志 */
  NGX_HTTP_LOG_PHASE
} ngx_http_phases;
```

常用的HTTP处理器handler，其函数签名形如
```c
typedef ngx_int_t (*ngx_http_handler_pt)(ngx_http_request_t *r);
```

### HTTP数据结构
对HTTP请求的封装结构，主要内容如下
```c
typedef struct ngx_http_request_s ngx_http_request_t;
struct ngx_http_request_s {
  ngx_uint_t method;
  ngx_uint_t http_version;
  ngx_str_t request_line;
  ngx_str_t uri;
  ngx_str_t args;
  ngx_str_t exten;
  ngx_str_t unparsed_uri;
  ngx_str_t method_name;
  ngx_str_t http_protocol;
  u_char *uri_start;
  u_char *uri_end;
  u_char *uri_ext;
  u_char *args_start;
  u_char *request_start;
  u_char *request_end;
  u_char *method_end;
  u_char *schema_start;
  u_char *schema_end;
  // 未经解析的头部
  ngx_buf_t *header_in;
  // 解析后的头部
  ngx_http_headers_in_t headers_in;
  // 请求响应的头部
  ngx_http_headers_out_t headers_out; 
  // 在请求处理过程中使用的内存池对象
  ngx_pool_t *pool;
  // 支持range断点续传
  unsigned allow_ranges:1;
  // 指向所有参与模块所使用的请求上下文的数组
  void **ctx;
  // upstream成员
  ngx_http_upstream_t *upstream;
  // 子请求postpone
  ngx_http_postpone_request_t *postponed;

  // ... 其他内容
};

// HTTP请求的头部
typedef struct {
  /* 所有解析过的HTTP头部都在headers链表中，
  每一个元素都是ngx_table_elt_t成员，这里是所有的头部字段，
  其中有一些标准定义的，会在结构体的后续字段中直接给出，
  对于不常见的/自定义的字段，需要自行遍历 */
  ngx_list_t headers; 

  /* 以下每个ngx_table_elt_t成员都是RFC2616规范中定义的HTTP头部，
  它们实际都指向headers链表中的相应成员。
  注意，当它们为NULL空指针时，表示没有解析到相应的HTTP头部 */
  ngx_table_elt_t *host; ngx_table_elt_t *connection; ngx_table_elt_t *if_modified_since;
  ngx_table_elt_t *if_unmodified_since; ngx_table_elt_t *user_agent; ngx_table_elt_t *referer;
  ngx_table_elt_t *content_length; ngx_table_elt_t *content_type; ngx_table_elt_t *range;
  ngx_table_elt_t *if_range; ngx_table_elt_t *transfer_encoding; ngx_table_elt_t *expect; 
#if (NGX_HTTP_GZIP)
  ngx_table_elt_t *accept_encoding;
  ngx_table_elt_t *via;
#endif
  ngx_table_elt_t *authorization; ngx_table_elt_t *keep_alive; 
#if (NGX_HTTP_PROXY || NGX_HTTP_REALIP || NGX_HTTP_GEO)
  ngx_table_elt_t *x_forwarded_for;
#endif
#if (NGX_HTTP_REALIP)
  ngx_table_elt_t *x_real_ip;
#endif
#if (NGX_HTTP_HEADERS)
  ngx_table_elt_t *accept; ngx_table_elt_t *accept_language;
#endif
#if (NGX_HTTP_DAV)
  ngx_table_elt_t *depth; ngx_table_elt_t *destination;
  ngx_table_elt_t *overwrite; ngx_table_elt_t *date;
#endif
  /* user和passwd是只有ngx_http_auth_basic_module才会用到的成员，这里可以忽略 */
  ngx_str_t user; ngx_str_t passwd; 
  /* cookies是以ngx_array_t数组存储的 */
  ngx_array_t cookies;
  // server名称
  ngx_str_t server;
  // 根据ngx_table_elt_t *content_length计算出的HTTP包体大小
  off_t content_length_n;
  time_t keep_alive_n; 
  /* HTTP连接类型，它的取值范围是0、
  NGX_http_CONNECTION_CLOSE或者
  NGX_HTTP_CONNECTION_KEEP_ALIVE */
  unsigned connection_type:2;
  /* 以下7个标志位是HTTP框架根据浏览器传来的useragent头部，
  它们可用来判断浏览器的类型，值为1时表示是相应的浏览器发来的请求，
  值为0时则相反 */
  unsigned msie:1;
  unsigned msie6:1;
  unsigned opera:1;
  unsigned gecko:1;
  unsigned chrome:1;
  unsigned safari:1;
  unsigned konqueror:1;
} ngx_http_headers_in_t;

// HTTP请求响应的头部
typedef struct {
  // 待发送的HTTP头部链表，与headers_in中的headers成员类似
  // 和headers_in中的一样，可以用ngx_list_push向其中添加自定义头部
  ngx_list_t headers;

  /* 响应中的状态值，如200表示成功。
  这里可以使用各个宏，如NGX_HTTP_OK */
  ngx_uint_t status;
  
  // 响应的状态行，如“HTTP/1.1 201 CREATED”
  ngx_str_t status_line;
  
  /* 以下成员（包括ngx_table_elt_t）都是
  RFC1616规范中定义的HTTP头部，设置后，
  ngx_http_header_filter_module过滤模块可以把它们加到待发送的网络包中 */
  ngx_table_elt_t *server; ngx_table_elt_t *date; ngx_table_elt_t *content_length;
  ngx_table_elt_t *content_encoding; ngx_table_elt_t *location;
  ngx_table_elt_t *refresh; ngx_table_elt_t *last_modified;
  ngx_table_elt_t *content_range; ngx_table_elt_t *accept_ranges;
  ngx_table_elt_t *www_authenticate; ngx_table_elt_t *expires; ngx_table_elt_t *etag;
  ngx_str_t *override_charset; 
  
  /* ngx_http_set_content_type(r)方法帮助我们设置
  Content-Type头部，这个方法会根据URI中的文件扩展名并对应着
  mime.type来设置Content-Type值 */
  size_t content_type_len;
  ngx_str_t content_type;
  ngx_str_t charset;
  u_char *content_type_lowcase;
  ngx_uint_t content_type_hash;
  ngx_array_t cache_control;

  /* content_length_n后，
  不用再次到ngx_table_elt_t *content_ length中设置响应长度 */
  off_t content_length_n;
  time_t date_time;
  time_t last_modified_time;
} ngx_http_headers_out_t;
```

在获取HTTP包体方面，首先要明确，HTTP的包体大小可以非常大，因此同步的处理是不现实的，Nginx提供的接收HTTP数据包的方式也是异步的，签名如下
```c
// 调用该函数，要求nginx开始异步接收包体，接收结束的回调为post_handler
ngx_int_t ngx_http_read_client_request_body(ngx_http_request_t *r, ngx_http_client_body_handler_pt post_handler);
// 回调函数的类型定义为
typedef void (*ngx_http_client_body_handler_pt)(ngx_http_request_t *r);
// 在该函数内，可以使用request_body来获取到缓存的临时文件，路径/名称等信息
// r->request_body->temp_file->file.name

```

在实际使用中，根据情况，可能读取包体，或舍弃包体，示例代码如下
```c
// 接收包体时的通用写法
ngx_int_t rc = ngx_http_read_client_request_body(r, ngx_http_mytest_body_handler); 
if (rc >= NGX_HTTP_SPECIAL_RESPONSE) {
  return rc;
}
return NGX_DONE;
// 舍弃包体
ngx_int_t rc = ngx_http_discard_request_body(r);
if (rc != NGX_OK) {
  return rc;
}
```

### HTTP响应
和请求一样，反馈给调用者的响应也是分为头部、包体两部分进行处理。使用的分别API如下
```c
// 将准备好的request发送出去
ngx_int_t ngx_http_send_header(ngx_http_request_t *r);

// 发送包体，在这一步，会自动调用ngx_http_finalize_request，结束这次请求
ngx_int_t ngx_http_output_filter(ngx_http_request_t r, ngx_chain_t in);

// 如果不发送包体，也需要调用，提示请求结束处理
void ngx_http_finalize_request(ngx_http_request_t *r, ngx_int_t rc)
```

包体相比于包头的处理要复杂一些，主要的难点在于对内存缓冲区的处理。在Nginx处理过程中，对于包体的内存使用，是要用内存池去申请内存，填充并发送的。而且还要在写入数据之后，对相关的```pos```、```last```指针进行修改，否则发送的长度会出现错误。在这个过程中，主要使用的API如下
```c
// 申请一个128字节大小的内存缓冲区
ngx_buf_t *b = ngx_create_temp_buf(r->pool, 128);
```

在实际使用中的调用示例如下
```c
// 对发送情况进行判断
ngx_int_t rc = ngx_http_send_header(r); 
if (rc == NGX_ERROR || rc > NGX_OK || r->header_only) {
  return rc;
}
// 如果有包体需要发送，则执行后续，否则可以直接return NGX_DONE;
ngx_buf_t *b = ngx_create_temp_buf(r->pool, 128);
/
ngx_chain_t out;
out.buf = b;
out.next = NULL;
return ngx_http_output_filter(r, &out);
```

另外由于包体中还可以是文件数据，为了提高效率，避免同步阻塞，以及利用系统上可能存在的高效的文件API。直接发送磁盘文件是最佳选择。在这个过程中，分别使用到了```ngx_file_t```，```ngx_open_file```，```ngx_pool_cleanup_add```等内容，示例如下。
```c
ngx_buf_t *b;
b = ngx_palloc(r->pool, sizeof(ngx_buf_t));
// 例如计划读取文件test.txt
u_char* filename = (u_char*)"/tmp/test.txt";
// 标记为文件类buf
b->in_file = 1;
// 对ngx_file_t进行申请和填写
b->file = ngx_pcalloc(r->pool, sizeof(ngx_file_t));
b->file->fd = ngx_open_file(filename, NGX_FILE_RDONLY|NGX_FILE_NONBLOCK, NGX_FILE_OPEN, 0);
b->file->log = r->connection->log;
b->file->name.data = filename;
b->file->name.len = strlen(filename);
if (b->file->fd <= 0)
{
  return NGX_HTTP_NOT_FOUND;
}
// 对HTTP响应信息的填写，需要先获取文件信息
if (ngx_file_info(filename, &b->file->info) == NGX_FILE_ERROR)
{
  return NGX_HTTP_INTERNAL_SERVER_ERROR;
}
r->headers_out.content_length_n = b->file->info.st_size;
b->file_pos = 0;
b->file_last = b->file->info.st_size;
// 设置文件句柄清理
ngx_pool_cleanup_t* cln = ngx_pool_cleanup_add(r->pool, sizeof(ngx_pool_cleanup_file_t));
if (cln == NULL) {
  return NGX_ERROR;
}
// 文件固定用ngx_pool_cleanup_file
cln->handler = ngx_pool_cleanup_file;
// 文件的清理类型是ngx_pool_cleanup_file_t
ngx_pool_cleanup_file_t *clnf = cln->data;
clnf->fd = b->file->fd;
clnf->name = b->file->name.data;
clnf->log = r->pool->log;
```

RFC2616规范中定义了range协议，它给出了一种规则使得客户端可以在一次请求中只下载完整文件的某一部分，这样就可支持客户端在开启多个线程的同时下载一份文件，其中每个线程仅下载文件的一部分，最后组成一个完整的文件。Nginx也支持range协议，而且是框架级别的支持，只需要在发送时将相关变量置为1，例如
```c
// r为ngx_http_request_t*
r->allow_ranges=1;
```

## 扩展
### 对HTTPS的处理

## 内置工具
### 日志
ngx_errlog_module模块。是所有模块的通用日志模块。由于编译平台不一定支持可变参数，因此接口有两种。核心代码如下
```c
// 提升易用性的宏
#define ngx_log_error(level, log, args...) \
  if ((log)->log_level >= level) ngx_log_error_core(level, log, args)
#define ngx_log_debug(level, log, args...) \
  if ((log)->log_level & level) \
    ngx_log_error_core(NGX_LOG_DEBUG, log, args)

// 对于不可变参数平台，Nginx提供的接口形如
// ngx_log_debug0、ngx_log_debug1 ... 代表无参数、1个参数等

// 日志函数原型：本次日志级别、log参数（配置级别、日志文件等）、错误码、C风格格式字符串(不完全相同，请查文档)
void ngx_log_error_core(ngx_uint_t level, ngx_log_t log, ngx_err_t err, const char fmt, ...);

// 用例
ngx_log_error(NGX_LOG_ALERT, r->connection->log,0,
  "test_flag=%d,test_str=%V,path=%*s,mycf addr=%p",
  mycf->my_flag,
  &mycf->my_str,
  mycf->my_path->name.len,
  mycf->my_path->name.data,
  mycf);
```

### 高级数据结构
Nginx为了保证跨平台性，对于一些必要的高级数据结构进行了实现。在这里直接以样例代码的方式，给出这些数据结构的用法。有时间也可以学习一下相关实现。

(有序)双向链表，但排序是插入排序，另外链表只负责连接，不负责数据部分元素分配（看代码就明白了）
```c
// ngx_queue.h，API是若干宏
struct ngx_queue_s {
    ngx_queue_t  *prev;
    ngx_queue_t  *next;
};

// 使用方式：在自定义结构体内，嵌入链表元素
typedef struct {
  u_char* str;
  int num;
  ngx_queue_t qEle;
} TestNode;

// 初始化
ngx_queue_t queueContainer;
ngx_queue_init(&queueContainer);

// 从ngx_queue_t* a，获取TestNode
TestNode *aNode = ngx_queue_data(a, TestNode, qEle);

// 加入链表
ngx_queue_insert_tail(&queueContainer, &(aNode->qEle));

// 遍历
ngx_queue_t* q;
for (q = ngx_queue_head(&queueContainer);q != ngx_queue_sentinel(&queueContainer);q = ngx_queue_next(q))
{
  TestNode* eleNode = ngx_queue_data(q, TestNode, qEle);
  // ...
}

// 排序
ngx_queue_sort(&queueContainer, compTestNode);

// 其中compTestNode是比较函数，形如
ngx_int_t compTestNode(const ngx_queue_t* a, const ngx_queue_t* b) {
  return ngx_queue_data(a, TestNode, qEle)->num > ngx_queue_data(b, TestNode, qEle)->num;
}
```

动态数组，类似于C++的vector，能自动扩容（2倍）
```c
typedef struct ngx_array_s ngx_array_t;
struct ngx_array_s {
  // elts指向数组的首地址
  void *elts;
  // nelts是数组中已经使用的元素个数
  ngx_uint_t nelts;
  // 每个数组元素占用的内存大小
  size_t size;
  // 当前数组中能够容纳元素个数的总大小
  ngx_uint_t nalloc;
  // 内存池对象
  ngx_pool_t *pool;
};


typedef struct TestNode {
  u_char* str;
  int num;
}
// 一组使用样例，cf为ngx_conf_t*，这里主要是为了使用以下内存池对象
// 创建动态数组
ngx_array_t* dynamicArray = ngx_array_create(cf->pool, 1, sizeof(TestNode));
// 创建元素
TestNode* a = ngx_array_push(dynamicArray);
a->num = 1;

TestNode* b = ngx_array_push_n(dynamicArray, 3);
b->num = 3;
(b+1)->num = 4;
(b+2)->num = 5;

TestNode* nodeArray = dynamicArray->elts;
ngx_uint_t arraySeq = 0;
for (; arraySeq < dynamicArray->nelts; arraySeq++)
{
  a = nodeArray + arraySeq;
  // 处理
}
ngx_array_destroy(dynamicArray);
```

单向链表ngx_list_t，已在前文描述。

红黑树
```c
typedef ngx_uint_t ngx_rbtree_key_t;
typedef struct ngx_rbtree_node_s ngx_rbtree_node_t;
// 红黑树节点
struct ngx_rbtree_node_s {
  // 无符号整型的关键字
  ngx_rbtree_key_t key;
  // 左子节点
  ngx_rbtree_node_t *left;
  // 右子节点
  ngx_rbtree_node_t *right;
  // 父节点
  ngx_rbtree_node_t *parent;
  // 节点的颜色，0表示黑色，1表示红色
  u_char color;
  // 仅1个字节的节点数据。由于表示的空间太小，所以一般很少使用
  u_char data;
};

// 需要使用红黑树时，将元素嵌入自定义数据结构体
typedef struct {
/* 一般都将ngx_rbtree_node_t节点结构体放在自定义数据类型的第1位，以方便类型的强制转换 */
  ngx_rbtree_node_t node;
  ngx_uint_t num;
} TestRBTreeNode;

// 真正的树结构
typedef struct ngx_rbtree_s ngx_rbtree_t;
/*为解决不同节点含有相同关键字的元素冲突问题，红黑树设置了
ngx_rbtree_insert_pt指针，这样可灵活地添加冲突元素*/
typedef void (*ngx_rbtree_insert_pt) (ngx_rbtree_node_t root,ngx_rbtree_node_t node, ngx_rbtree_node_t *sentinel);
struct ngx_rbtree_s {
  // 指向树的根节点。注意，根节点也是数据元素
  ngx_rbtree_node_t *root;
  // 指向NIL哨兵节点
  ngx_rbtree_node_t *sentinel;
  // 表示红黑树添加元素的函数指针，它决定在添加新节点时的行为究竟是替换还是新增
  ngx_rbtree_insert_pt insert;
};

// 操作示例

```

基数树

支持通配符的散列表

### 进程间通信

## 头文件速览
本章为了直观，保留了头文件的路径，实际在代码中引用时并不需要给出前缀，直接使用头文件名即可。

| 文件名 | 文件内容 |
| --- | --- |
| /src/http/ngx_http.h | 包含HTTP框架的整体头文件 |
| /src/core/ngx_config.h | Nginx核心，配置处理 |
| /src/core/ngx_conf_file.h | Nginx配置处理工具 |
| /src/core/ngx_core.h | Nginx框架核心 |
| /src/core/ngx_string.h | 字符串工具 |


## 术语
1. 虚拟主机：由于IP地址的数量有限，因此经常存在多个主机域名对应着同一个IP地址的情况，这时在nginx.conf中就可以按照server_name（对应用户请求中的主机域名）并通过server块来定义虚拟主机，每个server块就是一个虚拟主机，它只处理与之相对应的主机域名请求。

## 其他图表
ngx_list_t链表示意图

![ngx_list_t链表示意图](/images/book/understanding-nginx/ngx_list_t.png)

## 其他Demo
### upstream
本Demo基本来自书中，稍微修改了上游服务器网址（使用百度），以及模块名称。目标是通过在自定义模块中使用upstream模块，完成支持客户端传入URL，Nginx用其URL中的查询参数，向外部搜索引擎网站，查询数据，并透传返回查询结果。

但是由于百度现在对请求有安全验证，因此发送后会被跳转到验证页面。如果想验证，也可以自己搭一个普通的HTTP回显服务器。

配置参数，配置项处理
```c
#include <ngx_config.h>
#include <ngx_core.h>
#include <ngx_http.h>
#include <ngx_log.h>
 
static ngx_int_t ngx_http_myupstream_handler(ngx_http_request_t *r);
 
static char* ngx_http_myupstream(ngx_conf_t *cf, ngx_command_t *cmd, void *conf);

static ngx_int_t myupstream_upstream_process_header(ngx_http_request_t *r);

static void* ngx_http_myupstream_create_loc_conf(ngx_conf_t *cf);
static char* ngx_http_myupstream_merge_loc_conf(ngx_conf_t *cf, void *parent, void *child) ;
 
 
static ngx_command_t ngx_http_myupstream_commands[] = {
	{
			ngx_string("myupstream"),
			NGX_HTTP_MAIN_CONF | NGX_HTTP_SRV_CONF | NGX_HTTP_LOC_CONF | NGX_HTTP_LMT_CONF | NGX_CONF_NOARGS,
			ngx_http_myupstream,
			NGX_HTTP_LOC_CONF_OFFSET,
			0,
			NULL
	},
	ngx_null_command
};
 
 
/**
 * 模块上下文
 */
static ngx_http_module_t ngx_http_myupstream_module_ctx = { NULL, NULL, NULL, NULL,
		NULL, NULL, ngx_http_myupstream_create_loc_conf, ngx_http_myupstream_merge_loc_conf };
 
/**
 * 模块的定义
 */
ngx_module_t ngx_http_myupstream_module = {
		NGX_MODULE_V1,
		&ngx_http_myupstream_module_ctx,
		ngx_http_myupstream_commands,
		NGX_HTTP_MODULE,
		NULL,
		NULL,
		NULL,
		NULL,
		NULL,
		NULL,
		NULL,
		NGX_MODULE_V1_PADDING
};
 
/**
 * 命令解析的回调函数
 * 该函数中，主要获取loc的配置，并且设置location中的回调函数handler
 */
static char* ngx_http_myupstream(ngx_conf_t *cf, ngx_command_t *cmd, void *conf) {
	ngx_http_core_loc_conf_t *clcf;
 
	clcf = ngx_http_conf_get_module_loc_conf(cf, ngx_http_core_module);
	/* 设置回调函数。当请求/myupstream的时候，会调用此回调函数 */
	clcf->handler = ngx_http_myupstream_handler;
 
	return NGX_CONF_OK;
}

// 在自定义模块配置中，加入所需要的upstream参数
typedef struct {
  ngx_http_upstream_conf_t upstream;
} ngx_http_myupstream_conf_t;

// 模块create_loc_conf的实现
static void* ngx_http_myupstream_create_loc_conf(ngx_conf_t *cf) {
  ngx_http_myupstream_conf_t *mycf;
  mycf = (ngx_http_myupstream_conf_t *)ngx_pcalloc(cf->pool, sizeof(ngx_http_myupstream_conf_t));
  if (mycf == NULL) {
    return NULL;
  }
  /* 以下简单的硬编码ngx_http_upstream_conf_t结构中的各成员，
  如超时时间，都设为1分钟，这也是HTTP反向代理模块的默认值 */
  mycf->upstream.connect_timeout = 60000;
  mycf->upstream.send_timeout = 60000;
  mycf->upstream.read_timeout = 60000;
  mycf->upstream.store_access = 0600;
  /* buffering已经决定了将以固定大小的内存作为缓冲区来转发上游的响应包体，
  这块固定缓冲区的大小就是buffer_size。
  如果buffering为1，就会使用更多的内存缓存来不及发往下游的响应。
  例如，最多使用bufs.num个缓冲区且每个缓冲区大小为bufs.size。
  另外，还会使用临时文件，临时文件的最大长度为max_temp_file_size */
  mycf->upstream.buffering = 0;
  mycf->upstream.bufs.num = 8;
  mycf->upstream.bufs.size = ngx_pagesize;
  mycf->upstream.buffer_size = ngx_pagesize;
  mycf->upstream.busy_buffers_size = 2 * ngx_pagesize;
  mycf->upstream.temp_file_write_size = 2 * ngx_pagesize;
  mycf->upstream.max_temp_file_size = 1024 * 1024 * 1024;
  /* upstream hide_headers成员必须要初始化（upstream在解析完上游服务器返回的包头时，
  会调用ngx_http_upstream_process_headers方法按照hide_headers成员将本应转发给下游的一些HTTP头部隐藏），
  这里将它赋为NGX_CONF_UNSET_PTR ，这是为了在merge合并配置项方法中使用
  upstream模块提供的ngx_http_upstream_hide_headers_hash方法初始化hide_headers成员 */
  mycf->upstream.hide_headers = NGX_CONF_UNSET_PTR;
  mycf->upstream.pass_headers = NGX_CONF_UNSET_PTR;
  return mycf;
}

static ngx_str_t  ngx_http_proxy_hide_headers[] = {
  ngx_string("Date"),
  ngx_string("Server"),
  ngx_string("X-Pad"),
  ngx_string("X-Accel-Expires"),
  ngx_string("X-Accel-Redirect"),
  ngx_string("X-Accel-Limit-Rate"),
  ngx_string("X-Accel-Buffering"),
  ngx_string("X-Accel-Charset"),
  ngx_null_string
};

// merge_loc_conf
static char *ngx_http_myupstream_merge_loc_conf(ngx_conf_t *cf, void *parent, void *child) {
  ngx_http_myupstream_conf_t *prev = (ngx_http_myupstream_conf_t *)parent;
  ngx_http_myupstream_conf_t *conf = (ngx_http_myupstream_conf_t *)child;
  ngx_hash_init_t hash;
  hash.max_size = 100;
  hash.bucket_size = 1024;
  hash.name = "proxy_headers_hash";
  // ngx_http_upstream_hide_headers_hash具备合并功能，ngx_http_proxy_hide_headers是存储默认会隐藏的包头的数组
  if (ngx_http_upstream_hide_headers_hash(cf, &conf->upstream, &prev->upstream
    , ngx_http_proxy_hide_headers, &hash) != NGX_OK)
  {
    return NGX_CONF_ERROR;
  }
  return NGX_CONF_OK;
}

// 将status加入到ctx中
typedef struct {
  // ...
  ngx_http_status_t status;
  ngx_str_t backendServer;
} ngx_http_myupstream_ctx_t;

// 必须实现的回调之一：构造请求
static ngx_int_t myupstream_upstream_create_request(ngx_http_request_t *r) {
  /* 发往baidu上游服务器的请求很简单，就是模仿正常的搜索请求，
  以/searchq=…的URL来发起搜索请求 */
  static ngx_str_t backendQueryLine =
  // 注意这里的内容非常重要，如果有错误，会导致接到400系错误码
  ngx_string("GET /s?wd=%V HTTP/1.1\r\nHost: www.baidu.com\r\nConnection: close\r\n\r\n"); 
  ngx_int_t queryLineLen = backendQueryLine.len + r->args.len - 2;
  /* epoll多次调度send才能发送完成，这时必须保证这段内存不会被释放；
  另一个好处是，在请求结束时，这段内存会被自动释放，降低内存泄漏的可能 */
  ngx_buf_t* b = ngx_create_temp_buf(r->pool, queryLineLen);
  if (b == NULL)
    return NGX_ERROR;
  // last要指向请求的末尾
  b->last = b->pos + queryLineLen;
  // 作用相当于snprintf
  ngx_snprintf(b->pos, queryLineLen ,
    (char*)backendQueryLine.data,&r->args);
  /* r->upstream->request_bufs是一个ngx_chain_t结构，它包含着要发送给上游服务器的请求 */
  r->upstream->request_bufs = ngx_alloc_chain_link(r->pool);
  if (r->upstream->request_bufs == NULL)
    return NGX_ERROR;
  // request_bufs在这里只包含1个ngx_buf_t缓冲区
  r->upstream->request_bufs->buf = b;
  r->upstream->request_bufs->next = NULL;
  r->upstream->request_sent = 0;
  r->upstream->header_sent = 0;
  // header_hash不可以为0
  r->header_hash = 1;
  return NGX_OK;
}

// 对返回数据的状态行进行解析
static ngx_int_t myupstream_process_status_line(ngx_http_request_t *r) {
  size_t len;
  ngx_int_t rc;
  ngx_http_upstream_t *u;
  // 上下文中才会保存多次解析HTTP响应行的状态，下面首先取出请求的上下文
  ngx_http_myupstream_ctx_t* ctx = ngx_http_get_module_ctx(r,ngx_http_myupstream_module);
  if (ctx == NULL) {
    return NGX_ERROR;
  }
  u = r->upstream;
  /* HTTP框架提供的ngx_http_parse_status_line方法可以解析HTTP响应行，
  它的输入就是收到的字符流和上下文中的ngx_http_status_t结构 */
  rc = ngx_http_parse_status_line(r, &u->buffer, &ctx->status);
  // 返回NGX_AGAIN时，表示还没有解析出完整的HTTP响应行，需要接收更多的字符流再进行解析
  if (rc == NGX_AGAIN) {
    return rc;
  }
  // 返回 NGX_ERROR时，表示没有接收到合法的HTTP响应行
  if (rc == NGX_ERROR) {
    ngx_log_error(NGX_LOG_ERR, r->connection->log, 0, "upstream sent no valid HTTP/1.0 header");
    r->http_version = NGX_HTTP_VERSION_9;
    u->state->status = NGX_HTTP_OK;
    return NGX_OK;
  }
  /* 以下表示在解析到完整的HTTP响应行时，会做一些简单的赋值操作，
  将解析出的信息设置到r->upstream->headers_in结构体中。
  当upstream解析完所有的包头时，会把headers_in中的成员设置到将要向下游发送的
  r->headers_out结构体中，也就是说，现在用户向headers_in中设置的信息，
  最终都会发往下游客户端。为什么不直接设置r->headers_out而要多此一举呢？
  因为upstream希望能够按照ngx_http_upstream_conf_t配置结构体中的
  hide_headers等成员对发往下游的响应头部做统一处理 */
  if (u->state) {
    u->state->status = ctx->status.code;
  }
  u->headers_in.status_n = ctx->status.code;
  len = ctx->status.end - ctx->status.start;
  u->headers_in.status_line.len = len;
  u->headers_in.status_line.data = ngx_pnalloc(r->pool, len);
  if (u->headers_in.status_line.data == NULL) {
    return NGX_ERROR;
  }
  ngx_memcpy(u->headers_in.status_line.data, ctx->status.start, len);
  /* 下一步将开始解析HTTP头部。设置process_header回调方法为
  myupstream_upstream_process_header，之后再收到的新字符流将由
  myupstream_upstream_process_header解析 */
  u->process_header = myupstream_upstream_process_header;
  /* 如果本次收到的字符流除了HTTP响应行外，还有多余的字符，
  那么将由myupstream_upstream_process_header方法解析 */
  return myupstream_upstream_process_header(r);
}

// 处理响应包头，process_header
static ngx_int_t myupstream_upstream_process_header(ngx_http_request_t *r) {
  ngx_int_t rc;
  ngx_table_elt_t *h;
  ngx_http_upstream_header_t *hh;
  ngx_http_upstream_main_conf_t *umcf;
  /* 这里将upstream模块配置项ngx_http_upstream_main_conf_t取出来，
  目的只有一个，就是对将要转发给下游客户端的HTTP响应头部进行统一处理。
  该结构体中存储了需要进行统一处理的HTTP头部名称和回调方法 */
  umcf = ngx_http_get_module_main_conf(r, ngx_http_upstream_module);
  // 循环地解析所有的HTTP头部
  for ( ;; ) {
    /* HTTP框架提供了基础性的ngx_http_parse_header_line方法，它用于解析HTTP头部 */
    rc = ngx_http_parse_header_line(r, &r->upstream->buffer, 1);
    // 返回NGX_OK时，表示解析出一行HTTP头部
    if (rc == NGX_OK) {
      // 向headers_in.headers这个ngx_list_t链表中添加HTTP头部
      h = ngx_list_push(&r->upstream->headers_in.headers); if (h == NULL) {
        return NGX_ERROR;
      }
      // 下面开始构造刚刚添加到headers链表中的HTTP头部
      h->hash = r->header_hash;
      h->key.len = r->header_name_end - r->header_name_start;
      h->value.len = r->header_end - r->header_start;
      //HTTP头部的内存空间
      h->key.data = ngx_pnalloc(r->pool, h->key.len + 1 + h->value.len + 1 + h->key.len);
      if (h->key.data == NULL) {
        return NGX_ERROR;
      }
      h->value.data = h->key.data + h->key.len + 1;
      h->lowcase_key = h->key.data + h->key.len + 1 + h->value.len + 1;
      ngx_memcpy(h->key.data, r->header_name_start, h->key.len);
      h->key.data[h->key.len] = '\0';
      ngx_memcpy(h->value.data, r->header_start, h->value.len);
      h->value.data[h->value.len] = '\0';
      if (h->key.len == r->lowcase_index) {
        ngx_memcpy(h->lowcase_key, r->lowcase_header, h->key.len);
      } else {
        ngx_strlow(h->lowcase_key, h->key.data, h->key.len);
      }
      // upstream模块会对一些HTTP头部做特殊处理
      hh = ngx_hash_find(&umcf->headers_in_hash, h->hash, h->lowcase_key, h->key.len);
      if (hh && hh->handler(r, h, hh->offset) != NGX_OK) {
        return NGX_ERROR;
      }
      continue;
    }
    /* 返回NGX_HTTP_PARSE_HEADER_DONE时，表示响应中所有的
    HTTP头部都解析完毕，接下来再接收到的都将是HTTP包体 */
    if (rc == NGX_HTTP_PARSE_HEADER_DONE) {
    /* 如果之前解析HTTP头部时没有发现server和date头部，
    那么下面会根据HTTP协议规范添加这两个头部 */
    if (r->upstream->headers_in.server == NULL) {
      h = ngx_list_push(&r->upstream->headers_in.headers);
      if (h == NULL) {
        return NGX_ERROR;
      }
      h->hash = ngx_hash(ngx_hash(ngx_hash(ngx_hash(
        ngx_hash('s', 'e'), 'r'), 'v'), 'e'), 'r');
      ngx_str_set(&h->key, "Server");
      ngx_str_null(&h->value);
      h->lowcase_key = (u_char *) "server";
      }
      if (r->upstream->headers_in.date == NULL) {
        h = ngx_list_push(&r->upstream->headers_in.headers);
        if (h == NULL) {
          return NGX_ERROR;
        }
        h->hash = ngx_hash(ngx_hash(ngx_hash('d', 'a'), 't'), 'e');
        ngx_str_set(&h->key, "Date");
        ngx_str_null(&h->value);
        h->lowcase_key = (u_char *) "date"; 
      }
      return NGX_OK;
    }
    /*如果返回
    NGX_AGAIN，则表示状态机还没有解析到完整的
    HTTP头部，此时要求
    upstream模块继续接收新的字符流，然后交由
    process_header回调方法解析
    */
    if (rc == NGX_AGAIN) {
      return NGX_AGAIN;
    }
    // 其他返回值都是非法的
    ngx_log_error(NGX_LOG_ERR, r->connection->log, 0, "upstream sent invalid header"); 
    return NGX_HTTP_UPSTREAM_INVALID_HEADER;
  }
}

// 虽然finalize_request必须实现，但在本例中没有什么可做的
static void myupstream_upstream_finalize_request(ngx_http_request_t *r, ngx_int_t rc) {
  ngx_log_error(NGX_LOG_DEBUG, r->connection->log,0, "myupstream_upstream_finalize_request");
}

static ngx_int_t ngx_http_myupstream_handler(ngx_http_request_t *r) {
  // 首先建立HTTP上下文结构体ngx_http_myupstream_ctx_t
  ngx_http_myupstream_ctx_t* myctx = ngx_http_get_module_ctx(r,ngx_http_myupstream_module);
  if (myctx == NULL)
  {
    myctx = ngx_palloc(r->pool, sizeof(ngx_http_myupstream_ctx_t));
    if (myctx == NULL)
    {
      return NGX_ERROR;
    }
    // 将新建的上下文与请求关联起来
    ngx_http_set_ctx(r,myctx,ngx_http_myupstream_module);
  }
  /* 对每1个要使用upstream的请求，必须调用且只能调用1次ngx_http_upstream_create方法，
  它会初始化r->upstream成员 */
  if (ngx_http_upstream_create(r) != NGX_OK) {
    ngx_log_error(NGX_LOG_ERR, r->connection->log, 0,"ngx_http_upstream_create() failed");
    return NGX_ERROR;
  }
  // 得到配置结构体ngx_http_myupstream_conf_t
  ngx_http_myupstream_conf_t *mycf = 
    (ngx_http_myupstream_conf_t *) ngx_http_get_module_loc_conf(r, ngx_http_myupstream_module);
  ngx_http_upstream_t *u = r->upstream;
  // r->upstream->conf成员
  u->conf = &mycf->upstream;
  // 决定转发包体时使用的缓冲区
  u->buffering = mycf->upstream.buffering;
  // 以下代码开始初始化resolved结构体，用来保存上游服务器的地址
  u->resolved = 
    (ngx_http_upstream_resolved_t*) ngx_pcalloc(r->pool, sizeof(ngx_http_upstream_resolved_t));
  if (u->resolved == NULL) {
    ngx_log_error(NGX_LOG_ERR, r->connection->log, 0
      , "ngx_pcalloc resolved error. %s.", strerror(errno));
    return NGX_ERROR;
  }
  // 这里的上游服务器就是 www.baidu.com
  static struct sockaddr_in backendSockAddr; 
  struct hostent* pHost = gethostbyname((char*)"www.baidu.com");
  if (pHost == NULL)
  {
    ngx_log_error(NGX_LOG_ERR, r->connection->log, 0, "gethostbyname fail. %s", strerror(errno));
    return NGX_ERROR;
  }
  // 访问上游服务器的80端口
  backendSockAddr.sin_family = AF_INET;
  backendSockAddr.sin_port = htons((in_port_t) 80);
  char* pDmsIP = inet_ntoa(*(struct in_addr*) (pHost->h_addr_list[0]));
  backendSockAddr.sin_addr.s_addr = inet_addr(pDmsIP);
  myctx->backendServer.data = (u_char*)pDmsIP;
  myctx->backendServer.len = strlen(pDmsIP);
  // resolved成员中
  u->resolved->sockaddr = (struct sockaddr *)&backendSockAddr;
  u->resolved->socklen = sizeof(struct sockaddr_in);
  u->resolved->naddrs = 1;
  // 少数和书上不同的位置，这里如果不设置port、host，会直接报错
  u->resolved->port = htons((in_port_t) 80);
  ngx_str_set(&u->resolved->host,"www.baidu.com");
  
  // 设置3个必须实现的回调方法，也就是5.3.3节~5.3.5节中实现的3个方法
  u->create_request = myupstream_upstream_create_request;
  u->process_header = myupstream_process_status_line;
  u->finalize_request = myupstream_upstream_finalize_request;
  // count成员加1，参见5.1.5节
  r->main->count++;
  // 启动upstream
  ngx_http_upstream_init(r);
  // 必须返回NGX_DONE
  return NGX_DONE;
}


```

### subrequest
### filter


## 参考
1. [Nginx Docs](https://docs.nginx.com/nginx/)
2. [Nginx Wiki](https://www.nginx.com/resources/wiki/)
3. [Nginx 3rd Modules](https://www.nginx.com/resources/wiki/modules/index.html)
4. [Nginx安装介绍-树莓派](https://blog.csdn.net/Hallo_ween/article/details/107836013)
5. [Nginx源码分析](https://blog.csdn.net/initphp/category_9265172.html)
6. [Nginx源码分析-实战篇](https://initphp.blog.csdn.net/article/details/72912128)
7. [Nginx开发：从入门到精通](https://tengine.taobao.org/book/index.html)
8. [Web Framework Benchmark Round 22 2023-10-17](https://www.techempower.com/benchmarks/#hw=ph&test=plaintext&section=data-r22)
    > 这个排名看个乐呵就行，很多框架并不具备生产环境应用价值