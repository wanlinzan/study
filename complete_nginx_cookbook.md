# 高性能平衡负载
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

```
配置每台服务器同时处理最大连接数max_conns
共享内存大小zone backends 64k指定worker间共享每个服务器处理多少个连接以及多少个请求排队数据

等待队列的大小，等待最长时间

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

# 高可用性部署模式

### NGINX HA模式
#### 问题
您需要高度可用的负载平衡解决方案。
#### 解决方案
通过安装NGINX Plus存储库中的nginx-ha-keepalived软件包，使用NGINX Plus的HA模式。
NGINX Plus存储库包含一个名为nginx-hakeepalived的软件包。 这个基于keepalived的软件包管理一个暴露给客户端的虚拟IP地址。 另一个过程在NGINX服务器上运行，确保NGINX Plus和keepalived进程正在运行。 Keepalived是利用虚拟路由器冗余协议（VRRP）的一个过程，通常将称为心跳的小型消息发送到备份服务器。 如果备份服务器连续三个周期未收到心跳，备份服务器将启动故障切换，将虚拟IP地址移到自身并成为主服务器。 nginxha-keepalived的故障转移功能可以配置为识别定制故障情况。



### 使用DNS负载均衡负载平衡器
#### 问题
您需要在两台或更多台NGINX服务器之间分配负载。
#### 解决方案
通过将多个IP地址添加到DNS A记录，使用DNS循环跨越NGINX服务器。类似nginx默认循环负载模式






### 在EC2上进行负载平衡
#### 问题
您在AWS中使用NGINX，而NGINX Plus HA不支持Amazon IP。
#### 解决方案
通过配置NGINX服务器的Auto Scaling组并将Auto Scaling组连接到弹性负载平衡器，将NGINX置于弹性负载平衡器之后。 或者，您可以通过Amazon Web Services控制台，命令行界面或API手动将NGINX服务器手动放入弹性负载均衡器。



# 大规模可扩展的内容缓存

### 缓存区域
#### 问题
您需要缓存内容并需要定义缓存的存储位置。
#### 解决方案
使用proxy_cache_path指令定义共享内存缓存区域和内容的位置：
```
proxy_cache_path /var/nginx/cache //缓存目录，存储缓存信息
                 keys_zone=CACHE:60m //60m的共享内存空间，用于存储活动密钥和响应元数据的共享内存空间
                 levels=1:2 //目录结构的级别
                 inactive=3h //缓存有效期3小时
                 max_size=20g;//最大20GB
proxy_cache CACHE; // 通知特定的上下文以使用缓存区域

```
proxy_cache_path在HTTP上下文中有效，
 
proxy_cache指令在HTTP，服务器和位置上下文中有效。


### 缓存哈希键
#### 问题
你需要控制你的内容被缓存和查找的方式。
#### 解决方案
使用proxy_cache_key指令以及变量来定义构成缓存命中或未命中的内容：
```
proxy_cache_key "$host$request_uri $cookie_user";
```
这个缓存散列键将指示NGINX根据请求的主机和URI来缓存页面，以及定义用户的cookie。 有了这个，您可以缓存动态页面，而无需提供为其他用户生成的内容。





### 绕过缓存
#### 问题
您需要绕过缓存的能力
#### 解决方案
使用具有非空值或非零值的proxy_cache_bypass指令。 一种方法是通过在不希望缓存的位置块中设置一个等于1的变量：
```
proxy_cache_bypass $ http_cache_bypass;
```
如果名为cache_bypass的HTTP请求标头设置为任何非0值，该配置将告诉NGINX绕过缓存。



### 缓存性能
#### 问题
您需要通过在客户端进行缓存来提高性能。
#### 解决方案
使用客户端缓存控制标题：
```
location ~* \.(css|js)$ {
    expires 1y;
    add_header Cache-Control "public";
}
```
该位置块指定客户端可以缓存CSS和JavaScript文件的内容。 expires指令指示客户端，他们的缓存资源在一年后将不再有效。
add_header指令将HTTP响应头CacheControl添加到响应中，
其值为public，它允许沿途的任何缓存服务器缓存资源。 如果我们指定私有，则只允许客户端缓存该值。

### 清除缓存
#### 问题
您需要使缓存中的对象无效。
#### 解决方案
使用NGINX Plus的清除功能，proxy_cache_purge指令和一个非空值或零值变量：
```
map $request_method $purge_method {
    PURGE 1;
    default 0;
}
server {
    ...
    location / {
        ...
        proxy_cache_purge $purge_method;
    }
}

```

# 先进的媒体流

### 为MP4和FLV提供服务
#### 问题
您需要传输源自MPEG-4（MP4）或Flash视频（FLV）的数字媒体。
#### 解决方案
指定一个HTTP位置块为.mp4或.flv。 NGINX将使用渐进式下载或HTTP伪码流式传输媒体，并寻求支持：
```
http {
    server {
        ...
        location /videos/ {
            mp4;
        }
        location ~ \.flv$ {
            flv;
        }
    }
}

```
示例位置块告诉NGINX视频目录中的文件是MP4格式类型，并且可以通过渐进式下载支持进行流式传输。 第二个位置块指示NGINX以.flv结尾的任何文件都是Flash Video格式，并且可以通过HTTP伪流控支持进行流式传输。

### 用HLS流媒体
#### 问题
您需要支持在MP4文件中打包的H.264 / AAC编码内容的HTTP实时流（HLS）。
#### 解决方案
利用NGINX Plus的HLS模块进行实时分段，打包和复用，并控制碎片缓冲等，如转发HLS参数：
```
location /hls/ {
    hls; # Use the HLS handler to manage requests
    # Serve content from the following location
    alias /var/www/video;
    # HLS parameters
    hls_fragment 4s;
    hls_buffers 10 10m;//缓冲的数量和大小
    hls_mp4_buffer_size 1m;//缓冲了1m后允许客户端播放
    hls_mp4_max_buffer_size 5m;//有些视频的元数据以较大，所以允许最大缓冲区
}

```


### 使用HDS进行流式传输
#### 问题
您需要支持Adobe的HTTP动态数据流（HDS），该数据已经被碎片化并与元数据分离。
#### 解决方案
使用NGINX Plus支持碎片化FLV文件（F4F）模块为您的用户提供Adobe Adaptive Streaming：
```
location /video/ {
    alias /var/www/transformed_video;
    f4f;
    f4f_buffer_size 512k;//索引文件的缓冲区大小。
}

```
该示例指示NGINX Plus使用NGINX Plus F4F模块将先前分散的介质从磁盘上的位置提供给客户端。 索引文件（.f4x）的缓冲区大小设置为512千字节。

### 带宽限制
#### 问题
您需要将带宽限制到下游媒体流客户端，而不会影响观看体验。
#### 解决方案
```
location /video/ {
    mp4;
    mp4_limit_rate_after 15s;//15秒后开始限制
    mp4_limit_rate 1.2;//下载速度是播放速度的1.2倍
}

```
此配置允许下游客户端在应用比特率限制之前下载15秒。 15秒后，客户端可以以比特率120％的速率下载媒体，这使得客户端的下载速度总是比他们播放的速度快。


# 高级活动监视


### 高级活动监视
#### 问题

#### 解决方案



### 高可用性部署模式
#### 问题

#### 解决方案