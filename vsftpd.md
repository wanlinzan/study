######vsftpd安装
```
yum -y install vsftpd

```

######修改配置文件/etc/vsftpd/vsftpd.conf

```
# 禁用匿名用户登录
anonymous_enable=NO 

# 禁止用户切换到非ftp指定目录（自己的家目录）
chroot_local_user=YES

```

######创建ftp用户

```
# -s /sbin/nologin 限制用户登录系统  -d 指定用户ftp目录（家目录）
useradd -s /sbin/nologin -d /dir ftpuser
# 设置密码
passwd ftpuser 

```

###启动vsftpd服务
```
systemctl start vsftpd
```

