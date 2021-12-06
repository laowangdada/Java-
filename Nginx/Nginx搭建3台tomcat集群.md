![image-20211206223922600](..\static\三台tomcat集群.png)

nginx配置方法

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

max_conns = num(int): 限制upstream中服务器中的最大连接数(该限制作用于每个worker process)

slow_start = time:  慢启动服务器,不能与hash和random策略使用

```c
upstream localhostTT {
    server 192.168.1.173:8080 weight=6 slow_sart=60s;  //60s内逐渐提升weight
    server 192.168.1.174:8080 weight=3;
    server 192.168.1.175:8080 weight=3;
}
```

down: 标识服务器当前不可用

backup: 标识当前服务器只有在其他服务器不可用时才使用

```c
upstream localhostTT {
    server 192.168.1.173:8080 weight=3 slow_sart=60s;  //60s内逐渐提升weight
    server 192.168.1.174:8080 backup;
    server 192.168.1.175:8080 weight=3;
}
```

max_fails: 最大失败次数n    fail_timeout: 失败的时间段t

**表示节点在t时间内n次出现不可用情况,则判断该节点不可用....然后再等t之后再尝试访问**





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
    server 192.168.1.173:8080 weight=1;
    server 192.168.1.174:8080 weight=3;
    server 192.168.1.175:8080 weight=5;
}
```

