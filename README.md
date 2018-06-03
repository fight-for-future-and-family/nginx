# nginx
---
## nginx proxy_cache 缓存配置
>http{  
    ......  
    proxy_cache_path/data/nginx/tmp-test levels=1:2 keys_zone=tmp-test:100m inactive=7d max_size=1000g;  
} 
```
proxy_cache_path 缓存文件路径
levels 设置缓存文件目录层次；levels=1:2 表示两级目录
keys_zone 设置缓存名字和共享内存大小
inactive 在指定时间内没人访问则被删除
max_size 最大缓存空间，如果缓存空间满，默认覆盖掉缓存时间最长的资源。
```

- 如何使用proxy_cache
>配置项介绍：
Proxy_cache tmp-test 使用名为tmp-test的对应缓存配置
proxy_cache_valid  200 206 304 301 302 10d; 对httpcode为200…的缓存10天
proxy_cache_key $uri  定义缓存唯一key,通过唯一key来进行hash存取
proxy_set_header  自定义http header头，用于发送给后端真实服务器。
proxy_pass  指代理后转发的路径，注意是否需要最后的/
```
location /tmp-test/ {  
  proxy_cache tmp-test;  
  proxy_cache_valid  200 206 304 301 302 10d;  
  proxy_cache_key $uri;  
  proxy_set_header Host $host:$server_port;  
  proxy_set_header X-Real-IP $remote_addr;  
  proxy_set_header   X-Forwarded-For $proxy_add_x_forwarded_for;  
  proxy_passhttp://127.0.0.1:8081/media_store.php/tmp-test/;  
}
```  

- 问题一：主动清理缓存
>采用：nginx  proxy_cache_purge 模块 ，该模块与proxy_cache成对出现，功能正好相反。
设计方法：在nginx中，另启一个server，当需要清理响应资源的缓存时，在本机访问这个server。
例如:
访问 127.0.0.1:8083/tmp-test/TL39ef7ea6d8e8d48e87a30c43b8f75e30.txt 即可清理该资源的缓存文件。
配置方法：
```
location /tmp-test/ {  
                allow 127.0.0.1; //只允许本机访问  
                deny all; //禁止其他所有ip  
                proxy_cache_purge tmp-test $uri;  //清理缓存  
        }  
```