### Nginx配置语法

```apl
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
    keepalive_timeout  65; //单位s
    #gzip  on; // 内容传输css，js等会经过压缩再传输，提高传输效率
    server {
        listen       80;
        server_name  localhost;
        #charset koi8-r;
        #access_log  logs/host.access.log  main;
        location / {
            root   html;
            index  index.html index.htm;
        }
       
    }
}
```

##### Server块

一个http块中可以有多个server块, 一个server块代表着一个虚拟主机

###### listen参数表示该虚拟主机如何监听端口

```apl
listen 443 default_server ssl;
listen 127.0.0.1 default_server accept_filter=dataready backlog=1024;
```

后面可以接其他参数

| listen ip+port 后可以加的参数 |                                                              |
| ----------------------------- | ------------------------------------------------------------ |
| default                       | 当一个请求无法匹配配置文件中的所有主机域名时，就会选用默认的虚拟主机 |
| backlog=num                   | 在TCP建立三次握手过程中，进程还没有开始处理监听句柄，这时backlog队列将会放置这些新连接 |
| deferred                      | 在设置该参数后，若用户发起建立连接请求，并且完成了TCP的三次握手，内核也不会为了这次的连接调度worker进程来处理，只有用户真的发送请求数据时（内核已经在网卡中收到请求数据包），内核才会唤醒worker进程处理这个连接。这个参数适用于大并发的情况下，它减轻了worker进程的负担。当请求数据来临时，worker进程才会开始处理这个连接 |

###### server_name 主机名称

server_name后可以跟多个主机名称，如server_name  www.testweb.com www.download.testweb.com; 在开始处理一个HTTP请求时，Nginx会取出header头中的Host，与每个server中的server_name进行匹配，以此决定到底由哪一个server块来处理这个请求。有可能一个Host与多个server块中的server_name都匹配，这时就会根据匹配优先级来选择实际处理的server块。

###### server_names_hash_bucket_size  (32 or 64 or)

为了提高快速寻找到相应server name的能力，Nginx使用散列表来存储server name。
server_names_hash_bucket_size设置了每个散列桶占用的内存大小

###### server_name_in_redirect on|off

重定向主机名称

###### location

location会尝试根据用户请求的URI来匹配,如果匹配就选择location{}来处理用户请求.

匹配规则

1. location = /{}        只有当用户请求的是/时
2. location ~              表示匹配是大小写敏感
3. location ~*             表示匹配大小写不敏感
4. location ^~  images{}       表示前半部分与images匹配即可
5. location /{}              都没匹配上,则它处理,相当于都default

##### 文件路径的定义

1.root 定义资源文件相对于HTTP请求的根目录

```apl
location /download {
	root optwebhtml;
}
```

如果请求是/download/inde

x/test.html, 那么web服务器就会返回optwebhtml/download/index/test.html

2.alias 相当于conf=usr/local/nginx/conf/   **[只能配置在location块]**

```apl
location conf {
	alias usr/local/nginx/conf/;
}
```

如果用户请求/conf/nginx.conf,  那么映射到服务器上的位置就是usr/local/nginx/conf/nginx.conf

3.index  访问的URI是/ 用来 定义主页位置,

```apl
location {
	root path;
	index index.html htmlindex.php /index.php;
}
```

nginx接收到请求后先尝试访问path/index.html, 失败再访问htmlindex.php

4.error_page 重定向配置

```apl
error_page 404 404.html
error_page 502 503 504 50x.html
error_page 403 http://example.com/forbidden.html
```

如果不想修改URI让其他location]处理

```apl
location /{
	error_page 404 @fallback;
}
location @fallback {
	proxy_pass http://backend;
}
```





















