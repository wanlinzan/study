安装mysql
```angular2html
yum -y install mariadb mariadb-server

```
启动mysql
```angular2html
systemctl start mariadb
```

初始化mysql(设置密码之类的)
```angular2html
mysql_secure_installation
```