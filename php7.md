
进入源文件目录（一般把编译源代码放这里）
```
cd /usr/local/src
```

下载源代码
```
wget -O php-7.2.3.tar.gz http://cn2.php.net/get/php-7.2.3.tar.gz/from/this/mirror
```

解压源代码
```
tar xzvf php-7.2.3.tar.gz
```

进入解压后目录
```
cd php-7.2.3
```

这里安装一些编译php需要用到的扩展
```
yum -y install libxml2-devel curl-devel
```


生成配置文件
```
./configure --prefix=/usr/local/php --enable-fpm --with-curl 
```


编译并且安装
```
make && make install
```


添加php-fpm配置文件
```
cp /usr/lcoal/php/etc/php-fpm.conf.default /usr/lcoal/php/etc/php-fpm.conf
cp /usr/lcoal/php/etc/php-fpm.d/www.conf.default /usr/lcoal/php/etc/php-fpm.d/www.conf
```

添加systemctl服务
```
cp /usr/local/src/php-7.2.3/sapi/fpm/init.d.php-fpm /etc/init.d/php-fpm
```

修改权限php-fpm权限
```
chmod 755 /etc/init.d/php-fpm
```

添加开机启动
```
systemctl enable php-fpm
```
启动 php-fpm 
```
systemctl start php-fpm
```