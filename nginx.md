
进入源文件目录（一般把编译源代码放这里）
```
cd /usr/local/src
```

下载源代码
```
wget -O nginx-1.13.10.tar.gz http://nginx.org/download/nginx-1.13.10.tar.gz
```

解压源代码
```
tar xzvf nginx-1.13.10.tar.gz
```

进入解压后目录
```
cd nginx-1.13.10
```

安装依赖软件
```
yum -y install pcre-devel openssl-devel
```

创建用于NGINX的用户
```
useradd -s /sbin/nologin www
```

生成配置文件
```
./configure --user=nginx --group=nginx --with-threads --with-http_ssl_module --with-http_v2_module --with-http_realip_module --with-http_addition_module --with-http_sub_module --with-http_geoip_module --with-http_image_filter_module --with-http_xslt_module --with-http_dav_module --with-http_flv_module --with-http_mp4_module --with-http_gunzip_module --with-http_gzip_static_module --with-http_auth_request_module  --with-http_random_index_module --with-http_secure_link_module --with-http_degradation_module --with-http_slice_module --with-http_stub_status_module --without-http_charset_module  --with-mail  --with-stream  --with-stream_ssl_module  --with-stream_realip_module  --with-stream_geoip_module  --with-stream_ssl_preread_module  --with-google_perftools_module --with-compat
```


编译并且安装
```
make && make install
```


修改配置文件
```
vim /usr/local/nginx/conf/nginx.conf

# 修改用户为www 用户组为www
user www www
# 添加index/php
location / {
    root   /home/www;
    index  index.html index.htm index.php;

}

# 去掉这些代码钱的#号
location ~ \.php$ {
    root           /home/www;
    fastcgi_pass   127.0.0.1:9000;
    fastcgi_index  index.php;
    
    # 增加这两句支持PATH_INFO
    fastcgi_split_path_info ^(.+\.php)(.*)$; 
    fastcgi_param PATH_INFO $fastcgi_path_info;
    
    fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;
    include        fastcgi_params;
}


```


添加nginx到服务
```
vim /etc/init.d/nginx 

# 加入以下代码
#!/bin/bash
# nginx - this script starts and stops the nginx daemon
# chkconfig: - 85 15
# description: Nginx is an HTTP(S) server, HTTP(S) reverse \
# proxy and IMAP/POP3 proxy server
# processname: nginx
# config: /etc/nginx/nginx.conf
# config: /usr/local/nginx/conf/nginx.conf
# pidfile: /usr/local/nginx/logs/nginx.pid
# Source function library.
. /etc/rc.d/init.d/functions
# Source networking configuration.
. /etc/sysconfig/network
# Check that networking is up.
[ "$NETWORKING" = "no" ] && exit 0
nginx="/usr/local/nginx/sbin/nginx"
prog=$(basename $nginx)
NGINX_CONF_FILE="/usr/local/nginx/conf/nginx.conf"
[ -f /etc/sysconfig/nginx ] && . /etc/sysconfig/nginx
lockfile=/var/lock/subsys/nginx
make_dirs() {
# make required directories
user=`$nginx -V 2>&1 | grep "configure arguments:" | sed 's/[^*]*--user=\([^ ]*\).*/\1/g' -`
        if [ -z "`grep $user /etc/passwd`" ]; then
                useradd -M -s /bin/nologin $user
        fi
options=`$nginx -V 2>&1 | grep 'configure arguments:'`
for opt in $options; do
        if [ `echo $opt | grep '.*-temp-path'` ]; then
                value=`echo $opt | cut -d "=" -f 2`
                if [ ! -d "$value" ]; then
                        # echo "creating" $value
                        mkdir -p $value && chown -R $user $value
                fi
        fi
done
}
start() {
[ -x $nginx ] || exit 5
[ -f $NGINX_CONF_FILE ] || exit 6
make_dirs
echo -n $"Starting $prog: "
daemon $nginx -c $NGINX_CONF_FILE
retval=$?
echo
[ $retval -eq 0 ] && touch $lockfile
return $retval
}
stop() {
echo -n $"Stopping $prog: "
killproc $prog -QUIT
retval=$?
echo
[ $retval -eq 0 ] && rm -f $lockfile
return $retval
}
restart() {
#configtest || return $?
stop
sleep 1
start
}
reload() {
#configtest || return $?
echo -n $"Reloading $prog: "
killproc $nginx -HUP
RETVAL=$?
echo
}
force_reload() {
restart
}
configtest() {
$nginx -t -c $NGINX_CONF_FILE
}
rh_status() {
status $prog
}
rh_status_q() {
rh_status >/dev/null 2>&1
}
case "$1" in
start)
        rh_status_q && exit 0
        $1
        ;;
stop)
        rh_status_q || exit 0
        $1
        ;;
restart|configtest)
$1
;;
reload)
        rh_status_q || exit 7
        $1
        ;;
force-reload)
        force_reload
        ;;
status)
        rh_status
        ;;
condrestart|try-restart)
        rh_status_q || exit 0
        ;;
*)
echo $"Usage: $0 {start|stop|status|restart|condrestart|try-restart|reload|force-reload|configtest}"
exit 2
esac

```


修改权限/etc/init.d/nginx 权限
```
chmod 755 /etc/init.d/nginx
```

添加开机启动
```
systemctl enable nginx
```
启动 php-fpm 
```
systemctl start nginx
```


