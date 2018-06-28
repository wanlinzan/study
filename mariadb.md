参考
```angular2html
https://www.cnblogs.com/daixiang/p/5431639.html
```


进入目录
```angular2html
cd /usr/local/src
```

下载mariadb
```angular2html
wget https://downloads.mariadb.org/interstitial/mariadb-10.3.7/source/mariadb-10.3.7.tar.gz/from/https%3A//mirrors.shu.edu.cn/mariadb/
#下载后的文件名可能不对，重命名
mv distname mariadb-10.3.7.tar.gz

```

解压
```angular2html
tar xzvf mariadb-10.3.7.tar.gz
```

生成配置文件
```angular2html
cmake . -DCMAKE_INSTALL_PREFIX=/usr/local/mysql -DMYSQL_DATADIR=/usr/local/mysql/data -DSYSCONFDIR=/usr/local/mysql/etc  -DWITH_INNOBASE_STORAGE_ENGINE=1 -DWITH_ARCHIVE_STORAGE_ENGINE=1 -DWITH_BLACKHOLE_STORAGE_ENGINE=1 -DWITH_READLINE=1 -DWITH_SSL=system -DWITH_ZLIB=system -DWITH_LIBWRAP=0 -DMYSQL_UNIX_ADDR=/tmp/mysql.sock -DDEFAULT_CHARSET=utf8mb4 -DDEFAULT_COLLATION=utf8mb4_general_ci
```

如果失败了
```angular2html
make clean
rm -f CMakeCache.txt
```

安装
```angular2html
make && make install
```

创建MySQL用户
```angular2html
useradd -s /sbin/nologin -M mysql
```

初始化数据库
```angular2html
/usr/local/mysql/scripts/mysql_install_db --user=mysql --datadir=/usr/local/mysql/data
```


设置启动脚本：
```angular2html
cp /usr/local/mysql/support-files/mysql.server /etc/init.d/mysqld
chkconfig --add mysqld
chkconfig --list mysqld
```

启动服务：
```angular2html
service mysqld start 
```

初始化数据库
```angular2html
/usr/local/mysql/bin/mysql_secure_installation
```