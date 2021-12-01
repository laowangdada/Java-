#### 常用命令
./nginx -s stop/quit/reload
stop暴力关闭，强制关闭
./nginx -s quit 有请求则关闭，否则维持连接
./nginx -t 查看conf文件是否正确
./nginx -h 帮助help

常见bug

```apl
[root@10 conf]# ../sbin/nginx -s reload
nginx: [error] invalid PID number "" in "/usr/local/nginx/logs/nginx.pid"
[root@10 conf]# ../sbin/nginx -c /usr/local/nginx/conf/nginx.conf
```



#### 正向代理(代理客户端发起请求)

客户端是可以感知到代理的存在的. 因为在客户端需要一些配置.

![image-20210929104621233](static\正向代理.png)

#### 反向代理(代理服务器 Reverse Proxy)

客户端感知不到代理的存在, nginx代理的是服务器

![image-20210929141130837](static\反向代理.png)

#### Nginx反向代理

由于Nginx并发能力很强,因此一般用作前端的服务器直接向客户端提供静态文件服务. 同时也作为反向代理服务器来提供服务

![image-20211025213241634](static\nginx的使用.png)

注意: 当客户端发来HTTP请求时，Nginx并不会立刻转发到上游服务器，而是先把用户的请求包括HTTP包体）完整地接收到Nginx所在服务器的硬盘或者内存中，然后再向上游服务器发起连接，把缓存的客户端请求转发到上游服务器。

这样可以减缓Tomcat等服务器的压力.

##### 负载均衡的配置

1> upstream块  配置上游服务器    配置块http

```apl
upstream backend{
	server backend1.com;
	server backend2.com;
	server backend3.com;
}

server {
	location / {
		proxy_pass http://backend;
	}
}
```

**proxy_pass** 将当前请求反向代理到URL参数指定的服务器上,还可以把HTTP转换成HTTPS https\://backend

默认情况下反向代理是不会转发请求中的Host头部的。如果需要转发，那么必须加上配置：

```apl
proxy_set_header Host $host;
// other settings
proxy_method POST;5
```



2> server 指定上游服务器名称   可以是域名, ip , Unix句柄等 配置块upstream

| 可配置参数        | 效果                                                         |
| ----------------- | ------------------------------------------------------------ |
| weight=number     | 设置转发的权重                                               |
| fail_timeout=time | 表示该时间段内转发失败多少次后就认为上游服务器暂时不可用     |
| max_fails=number  | 该选项与fail_timeout配合使用，指在fail_timeout时间段内，如果向当前的上游服务器转发失败次数超过number，则认为在当前的fail_timeout时间段内这台上游服务器不可用 |
| down              | 表示所在的上游服务器永久下线，只在使用ip_hash配置项时才有用  |
| backup            | 在使用ip_hash配置项时它是无效的。它表示所在的上游服务器只是备份服务<br/>器，只有在所有的非备份上游服务器都失效后，才会向所在的上游服务器转发请求。 |

3> ip_hash  

根据用户访问的ip地址计算一个Key, 将key hash对应到一台特定的服务器







































