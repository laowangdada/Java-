![image-20211206223922600](..\static\三台tomcat集群.png)

##### nginx配置方法

```c
http {
    upstream localhostTT {
        server 192.168.1.173:8080;
        server 192.168.1.174:8080;
        server 192.168.1.175:8080;
    }
    server {
        listen       80;
        server_name  www.localhostTT.com;
        location / {
            proxy_pass http://localhostTT;
        }
       
    }
}
```

upstream 块下的指令参数

**max_conns = num(int):** 限制upstream中服务器中的最大连接数(该限制作用于每个worker process)

**slow_start = time:**  慢启动服务器,不能与hash和random策略使用

```c
upstream localhostTT {
    server 192.168.1.173:8080 weight=6 slow_sart=60s;  //60s内逐渐提升weight
    server 192.168.1.174:8080 weight=3;
    server 192.168.1.175:8080 weight=3;
}
```

**down:** 标识服务器当前不可用

**backup:** 标识当前服务器只有在其他服务器不可用时才使用

```c
upstream localhostTT {
    server 192.168.1.173:8080 weight=3 slow_sart=60s;  //60s内逐渐提升weight
    server 192.168.1.174:8080 backup;
    server 192.168.1.175:8080 weight=3;
}
```

**max_fails: 最大失败次数n    fail_timeout: 失败的时间段t**

**表示节点在t时间内n次出现不可用情况,则判断该节点不可用....然后再等t之后再尝试访问**

#### 使用keepalived提高吞吐量

在http早期 ，每个http请求都要求打开一个tpc socket连接，并且使用一次之后就断开这个tcp连接。

在nginx配置keep-alive可以改善这种状态，即在一次TCP连接中可以持续发送多份数据而不会断开连接。

通过使用keep-alive机制，可以减少tcp连接建立次数，也意味着可以减少TIME_WAIT状态连接，以此提高性能和提高http服务器的吞吐率(更少的tcp连接意味着更少的系统内核调用,socket的accept()和close()调用)。

但是keep-alive并不是免费的午餐,长时间的tcp连接容易导致系统资源无效占用。配置不当的keep-alive，有时比重复利用连接带来的损失还更大。所以，正确地设置 keep-alive timeout时间非常重要。

**keepalvie timeout
**Http的守护进程，一般都提供了keep-alive timeout时间设置参数。比如nginx的keepalive_timeout，和Apache的KeepAliveTimeout。这个 keepalive_timout时间值意味着：一个http产生的tcp连接在传送完最后一个响应后，还需要hold住 keepalive_timeout秒后，才开始关闭这个连接。当httpd守护进程发送完一个响应后，理应马上主动关闭相应的tcp连接，设置 keepalive_timeout后，httpd守护进程会想说：”再等等吧，看看浏览器还有没有请求过来”，这一等，便是 keepalive_timeout时间。如果守护进程在这个等待的时间里，一直没有收到浏览发过来http请求，则关闭这个http连接.

所以没有keepalive的情况下,

```
建立tcp连接 + 传送http请求 + php脚本执行 + 传送http响应 + 关闭tcp连接 + 2MSL 
```

配置keepalive之后

```
tcp建立 + (最后一个响应时间 – 第一个请求时间) + tcp关闭 + 2MSL
```



#### 负载均衡策略-轮询(default)

```c
upstream localhostTT {
        server 192.168.1.173:8080;
        server 192.168.1.174:8080;
        server 192.168.1.175:8080;
}
```

#### 负载均衡策略- 加权轮询

```c
upstream localhostTT {        
    server 192.168.1.173:8080 weight=1;
    server 192.168.1.174:8080 weight=3;
    server 192.168.1.175:8080 weight=5;
}
```

#### 负载均衡策略- ip_hash

```
upstream localhostTT {
	ip_hash;
    server 192.168.1.173:8080;
    server 192.168.1.174:8080;
    server 192.168.1.175:8080;
}
```

默认的ip_hash算法

[hash了192.168.1并没有hash主机号,hash的是网络号]

如果要临时移除服务器,不可以直接注释掉,要标记down位,否则hash和缓存等都会失效

带来的问题: 当服务器结点增加或者减少时候,hash要重新算,会话和缓存都会消失

一致性hash算法

满足均衡性, 单调性, 分散性

操作: 根据服务器的ip或者servername作为关键字进行hash.按照常用的hash算法来将对应的key哈希到一个具有2^32次方个节点的空间中，即0 ~ (2^32)-1的数字空间中。现在我们可以将这些数字头尾相连，想象成一个闭合的环形。

<img src="..\static\一致性hash.png" alt="image-20211208150918302" style="zoom: 50%;" />

当有用户来的时候,根据用户的ip哈希得到的结果,顺时针寻找第一台服务器

如果有服务器增加或者删除也可以保证尽量不影响其他用户

平衡性可以用虚拟节点来保证(因为当服务器数量比较少的时候,会导致这些服务器的负载不均衡)

比如A,B,C三台机器, 可以虚拟处A1,A2.....来平衡



#### 负载均衡策略- url_hash

<img src="..\static\urlHash.png" alt="image-20211208170351095" style="zoom:50%;" />

这样可以根据不同的url来进行集群

```c
upstream localhostTT {
	hash $request_uri;
    server 192.168.1.173:8080;
    server 192.168.1.174:8080;
    server 192.168.1.175:8080;
}
```

根据完整uri来进行hash: /account 和/account/是不同的字符串



#### 负载均衡策略- least_conn

哪个服务器的请求数量最少

```c
upstream localhostTT {
	least_conn;
    server 192.168.1.173:8080;
    server 192.168.1.174:8080;
    server 192.168.1.175:8080;
}
```









