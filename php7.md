
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
./configure --prefix=/usr/local/php --sysconfdir=/user/local/php/conf --enable-fpm --with-fpm-user=www --with-fpm-group=www --with-fpm-systemd  --with-curl --with-pdo-mysql --with-openssl --with-kerberos --with-system-ciphers --with-pcre-regex --with-pcre-jit --with-zlib --enable-bcmath --with-bz2 --enable-calendar --enable-ftp --with-config-file-path=/usr/local/php/conf --with-config-file-scan-dir=/usr/local/php/conf --disable-short-tags --enable-exif --enable-libgcc  --enable-ftp --with-imap --with-interbase --enable-intl --with-gd --with-gettext --with-gmp --with-ldap --with-ldap-sasl --with-libmbfl --with-onig --with-mhash --enable-gd-jis-conv --enable-mbstring --with-libzip --enable-embedded-mysqli --with-mysqli --enable-pcntl --with-pdo-dblib --with-pdo-firebird --with-pdo-mysql --with-zlib-dir --with-pdo-pgsql --with-pgsql --enable-sockets --with-iconv-dir --with-xsl --with-xmlrpc --with-sodium --with-imap-ssl --with-password-argon2 --enable-sysvmsg --enable-sysvsem --enable-sysvshm --with-tidy --with-libexpat-dir --enable-wddx --with-pspell --with-libedit --with-readline --with-recode --with-pear --with-mm --enable-shmop --with-libxml-dir --with-snmp --with-openssl-dir --enable-soap --enable-zip --enable-mysqlnd
```


编译并且安装
```
make && make install
```


复制php.ini配置文件
```
cp /usr/local/src/php-7.2.3/php.ini-production /usr/local/php/php.ini
```
 

添加php-fpm配置文件
```
cp /usr/lcoal/src/php-7.2.3/etc/php-fpm.conf.default /usr/lcoal/php/etc/php-fpm.conf
cp /usr/lcoal/src/php-7.2.3/etc/php-fpm.d/www.conf.default /usr/lcoal/php/etc/php-fpm.d/www.conf
```

修改php-fpm子进程用户为www
```
vim /usr/local/php/etc/php-fpm.d/www.conf

#修改
user = www
group = www
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


附录:编译动态扩展
```
# 进入源代码目录
cd /usr/local/src/php/php-7.2.3/ext/openssl

# 复制编译配置
cp config0.m4 config.m4

# 执行安装命令
/usr/local/php/bin/phpize

# 检测并生成配置
./configure --with-openssl --with-php-config=/usr/local/php/bin/php-config

# 编译并安装
make && make install

# 进入配置文件打开动态扩展，去掉前面的分号
vi /usr/local/php/lib/php.ini 
extension=openssl

# 重启php-fpm
systemctl restart php-fpm
```