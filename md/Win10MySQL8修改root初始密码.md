# <center>Win10 MySQL8 修改root初始密码</center>  

1. 使用管理员方式打开cmd，cd至mysql安装bin目录下  
```
cd "C:\Program Files\Mysql\Mysql Server 8.0\bin"
```

2. 关闭mysql服务，跳过密码验证打开mysql数据库   
```
net stop mysql

mysqld --console --skip-grant-tables --shared-memory
```

3. 使用管理员身份重新打开一个cmd窗口，即可使用无密码方式进入mysql修改密码
```
mysql
use mysql
update user set authentication_string = '' where user = 'root';
flush privileges;
alter user 'root'@'localhost' identified by 'your password';
```

4. 重新启动服务
```
net start mysql
```