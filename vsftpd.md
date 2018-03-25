vsftpd安装
```
yum -y install vsftpd
```

修改配置文件/etc/vsftpd/vsftpd.conf
```
# 禁用匿名用户登录
anonymous_enable=NO 

# 禁止用户切换到非ftp指定目录（自己的家目录）
chroot_local_user=YES

# 设置了禁止切换出家目录之后，不能具有写入权限，否则报错，所以最后面添加一行
allow_writeable_chroot=YES
```

创建ftp用户

```
# -s /sbin/nologin 限制用户登录系统  -d 指定用户ftp目录（家目录）
useradd -s /sbin/nologin -d /dir ftpuser
# 设置密码
passwd ftpuser 
```

启动vsftpd服务
```
systemctl start vsftpd
```

添加sftp支持
```
openssl req -x509 -nodes -days 365 -newkey rsa:1024 -keyout /etc/vsftpd/vsftpd.pem -out /etc/vsftpd/vsftpd.pem

# 在/etc/vsftpd/vsftpd.conf中添加
ssl_enable=YES
allow_anon_ssl=NO
force_local_data_ssl=YES
force_local_logins_ssl=YES
ssl_tlsv1=YES
ssl_sslv2=NO
ssl_sslv3=NO
rsa_cert_file=/etc/vsftpd/vsftpd.pem

# 如果出现类似“服务器发回了不可路由的地址。使用服务器地址代替。”
# “在网络安全组配置规则，放行ftp被动模式的高端端口1024/65535”
```

