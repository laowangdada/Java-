Nginx上解决CORS问题

![image-20211206165310104](..\static\nginx跨域举例.png)

```c
server{
    #允许跨域请求的域，*代表所有
    add_header 'Access-Control-Allow-Origin' *;
    #允许带上cookie请求
    add_header 'Access-Control-Allow-Credentials' 'true';
    #允许请求的方法，比如 GET/POST/PUT/DELETE
    add_header 'Access-Control-Allow-Methods' *;
    #允许请求的header
    add_header 'Access-Control-Allow-Headers' *;

}
```

Nginx 静态资源防止盗链

```c
server{
    #对源站点验证
    valid_referers *.imooc.com;
    #非法引入会进入下方判断
    if ($invalid_referer) {
    	return 404;
    }
    
    location / {
        
    }
}
```

