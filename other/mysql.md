<center>Mysql 5.7默认无密码</center>	
使用：

```shell
 sudo vim /etc/mysql/debian.cnf
 mysql -u debian-sys-maint -p //输入从上面文件得的密码
//进入后
>use mysql
>update mysql.user set authentication_string=password('12580') where user='root' and Host ='localhost';
>update user set plugin="mysql_native_password"; 
>flush privileges;
>quit
 sudo service mysql restart
 mysql -u root -p
 输入新密码
```

<center>内网穿透</center>

 简单来说实现不同局域网内的主机之间通过互联网进行通信的技术叫内网穿透 。

