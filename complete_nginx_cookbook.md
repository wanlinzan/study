#高性能平衡负载
### HTTP 负载平衡
#### 问题
 
 您需要在两个或更多HTTP服务器之间分配负载。
#### 解决方案
 通过使用NGINX HTTP的`upstream`模块进行HTTP服务器的负载平衡：
```nginx
upstream backend {
    server 10.10.12.45:80 weight=1;
    server app.example.com:80 weight=2;
}
server {
    location / {
        proxy_pass http://backend;
    }
}

```
 此配置在端口80上的两个HTTP服务器之间平衡负载。weight参数指示NGINX将两倍的连接传递给第二个服务器，并且weight参数默认为1


### TCP 负载平衡
#### 问题
 
 您需要在两台或更多台TCP服务器之间分配负载。

#### 解决方案
 
使用NGINX的`stream`模块在TCP服务器上进行负载平衡
```
stream {
    upstream mysql_read {
        server read1.example.com:3306 weight=5;
        server read2.example.com:3306;
        server 10.10.12.34:3306 backup;
    }
    server {
        listen 3306;
        proxy_pass mysql_read;
    }
}

```


### 负载平衡方式
#### 问题
 循环法负载平衡不适合您的用例，因为您具有异构的工作负载或服务器池。
#### 解决方案
 使用NGINX的一种负载平衡方法，循环法，最少链接，最少时间，通用散列或IP散列：
```
upstream backend {
 least_conn;
 server backend.example.com;
 server backend1.example.com;
}

```
`weight`因为每台服务器的承载能力不同
- 循环法：默认的分配方式,考虑`weight`
- `least_conn`：连接数最少的优先分配,考虑`weight`
- `least_time`:响应最快的优先分配（少量的连接并不一定意味着最快的响应）
- `hash` ：通用hash方式，如果`upstream`中的服务器增加或减少，哈希请求将被重新分配，consistent,参考一致性hash
- `ip_hash` :只支持http,考虑`weight`


### 连接限制
#### 问题
 负载平衡服务器压力过重

#### 解决方案
 使用NGINX Plus的max_conns参数限制与上游服务器的连接：
```
upstream backend {
    zone backends 64k;
    queue 750 timeout=30s;
    server webserver1.example.com max_conns=25;
    server webserver2.example.com max_conns=15;
}
配置每台服务器同时处理最大连接数max_conns
共享内存大小zone backends 64k指定worker间共享每个服务器处理多少个连接以及多少个请求排队数据

等待队列的大小，等待最长时间
```

# 智能session会话持久
### Sticky Cookie
#### 问题
 让同一个用户访问同一个服务器
#### 解决方案
```
upstream backend {
    server backend1.example.com;
    server backend2.example.com;
    sticky cookie
           affinity //cookie名称
           expires=1h //有效期
           domain=.example.com //使用域名
           httponly 
           secure
           path=/; //有效路径
}

```


### Sticky Learn
#### 问题
 您需要使用现有的Cookie将下游客户端绑定到上游服务器。
#### 解决方案
 使用 `sticky learn` 指令来发现和跟踪由上游应用程序创建的Cookie
```
upstream backend {
    server backend1.example.com:8080;
    server backend2.example.com:8081;
    sticky learn
           create=$upstream_cookie_cookiename
           lookup=$cookie_cookiename
           zone=client_sessions:2m;//2m的共享内存，大约16000个会话
}

```

### Sticky Routing
#### 问题
 您需要精确控制持久会话如何路由到上游服务器
#### 解决方案
```
map $cookie_jsessionid $route_cookie {
    ~.+\.(?P<route>\w+)$ $route;
}
map $request_uri $route_uri {
    ~jsessionid=.+\.(?P<route>\w+)$ $route;
}
upstream backend {
    server backend1.example.com route=a;
    server backend2.example.com route=b;
    sticky route $route_cookie $route_uri;
}

```


### Connection Draining
#### 问题
 在服务会话期间，您需要适当地移除服务器以进行维护或其他原因。
#### 解决方案
 通过NGINX Plus API使用drain参数指示NGINX停止发送尚未被跟踪的新连接：
```
curl 'http://localhost/upstream_conf?upstream=backend&id=1&drain=1'

```
# 应用程序健康检查
### 如何检查
#### 问题
 你需要检查你的申请健康，但不知道要检查什么。
#### 解决方案
 使用简单但直接的应用程序健康指示。 例如，简单地返回HTTP 200响应的处理程序会告诉负载均衡器应用程序进程正在运行。



### 慢启动
#### 问题
 您的应用程序需要在完全生产负载之前升级。
#### 解决方案
 在服务器重新引入到上游负载平衡池时，使用server指令中的`slow_start`参数逐渐增加指定时间内的连接数
```
upstream {
    zone backend 64k;
    server server1.example.com slow_start=20s;
    server server2.example.com slow_start=15s;
}

```
 服务器指令配置将在重新引入池后缓慢增加到上游服务器的流量。
 server1将在20秒内缓慢增加连接数，server2将在15秒内缓慢增加连接数。
 慢启动是一段时间内缓慢提高代理服务器请求数量的概念。 慢启动允许应用程序通过填充缓存进行预热，启动数据库连接，而不会在连接启动时立即被连接所淹没。 当运行状况检查失败的服务器开始再次通过并重新进入负载平衡池时，此功能会生效。




### TCP 健康检查
#### 问题
 您需要检查上游TCP服务器的运行状况，并从池中删除不健康的服务器。
#### 解决方案
```
stream {
    server {
        listen 3306;
        proxy_pass read_backend;
        health_check interval=10 passes=2 fails=3;
    }
}

```
 该示例主动监视上游服务器。 如果上游服务器无法响应由NGINX发起的三个或更多TCP连接，则认为上游服务器不健康。 NGINX每10秒执行一次检查。 服务器在通过两次健康检查后才会被认为是健康的。

### HTTP 健康检查
#### 问题
 您需要主动检查上游HTTP服务器的健康状况
#### 解决方案
```
http {
    server {
        ...
        location / {
            proxy_pass http://backend;
            health_check interval=2s
                         fails=2
                         passes=5
                         uri=/
                         match=welcome;
        }
    }
    # status is 200, content type is "text/html",
    # and body contains "Welcome to nginx!"
    match welcome {
        status 200;
        header Content-Type = text/html;
        body ~ "Welcome to nginx!";
    }
}

```


### 高可用性部署模式
#### 问题

#### 解决方案





### 高可用性部署模式
#### 问题

#### 解决方案






### 高可用性部署模式
#### 问题

#### 解决方案




### 高可用性部署模式
#### 问题

#### 解决方案