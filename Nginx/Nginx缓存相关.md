<img src="..\static\nginx缓存.png" alt="image-20211208235416542" style="zoom: 67%;" />

#### Nginx控制浏览器的缓存[expire]

```
location /static {
	expires 10s;
}
```

之后用户的响应header就有cache-control: max-age=101