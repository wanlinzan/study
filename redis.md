进入目录
```angular2html
cd /usr/local/src
```

下载
```angular2html
wget http://download.redis.io/releases/redis-4.0.9.tar.gz
```

解压
```angular2html
tar xzvf redis-4.0.9.tar.gz
```

安装
```angular2html
cd redis-4.0.9
make 
cd src
make PREFIX=/usr/local/redis install

#复制配置文件到redis安装目录中
mkdir /usr/local/redis/conf
cp /usr/local/src/redis-4.0.9/redis.conf /usr/local/redis/conf/redis.conf
cp /usr/local/src/redis-4.0.9/sentinel.conf /usr/local/redis/conf/sentinel.conf
```


修改redis为常驻内存服务
```angular2html
vim /usr/local/redis/conf/redis.conf 
#里面修改一下参数
daemonize yes 
```

redis 设置为服务
```angular2html
cp /usr/local/src/redis-4.0.9/utils/redis_init_script /etc/rc.d/init.d/redis


#编辑一些配置，安装自己的目录设置
vim /etc/rc.d/init.d/redis
#里面最前面添加两行
# chkconfig:   2345 90 10
# description:  Redis is a persistent key-value database

chkconfig --add redis

```

启动redis服务
```angular2html
systemctl start redis
```