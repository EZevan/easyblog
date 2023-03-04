# MySql8 修改root用户密码   
> 背景：  
1  忘记mysql root用户登录连接密码
2  MySql8移除了password密码加密函数
3  User表中的密码字段换成了authentication_string   
***

**较低版本的MySql中，如5.7及一下；修改root用户密码方法如下**
1. 通过修改/etc/my.cnf文件设置跳过验证
```C
vim /etc/my.cnf
# 在[mysql]后添加skip-grant-tables
# 重启服务
service mysqld restart 
```  
2. mysql -uroot -p 免密直接进入mysql库，修改user表密码  
```
mysql -uroot -p
use mysql

update user set password=password('your password') where user='root';
```  
3. 将/etc/my.cnf还原，重启mysql

**MySql8后，修改root用户密码方法如下**  

1. 通过修改/etc/my.cnf文件设置跳过验证
```C
vim /etc/my.cnf
# 在[mysql]后添加skip-grant-tables
# 重启服务
service mysqld restart 
```    
2. mysql -uroot -p 免密直接进入mysql库，修改root用户密码 
```python
mysql -uroot -p
use mysql
# 修改root用户密码为空
update user set authentication_string='' where user = 'root';

# 如果想要设置简单密码，需要修改密码策略
# 先查看密码策略
mysql> show variables like 'validate_password%';
+--------------------------------------+--------+
| Variable_name                        | Value  |
+--------------------------------------+--------+
| validate_password_check_user_name    | OFF    |
| validate_password_dictionary_file    |        |
| validate_password_length             | 8      |
| validate_password_mixed_case_count   | 1      |
| validate_password_number_count       | 1      |
| validate_password_policy             | MEDIUM |
| validate_password_special_char_count | 1      |
+--------------------------------------+--------+
# 修改密码长度
set global validate_password_length=6;

# 修改密码策略为LOW
set global validate_password_policy=0;

# 修改大小写个数和特殊字符个数为0
set global validate_password_mixed_case_count=0;
set global validate_password_special_char_count=0;

```   
3. 修改root用户密码
```
# 将/etc/my.cnf还原
# 使用ALTER修改root用户密码（由于已经root密码设置为空，故无需密码即可访问mysql）
mysql -uroot -p
use mysql
ALTER user 'root'@'localhost' IDENTIFIED BY 'new password';
# 或者
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'new passowrd';

# 注：不需要flush privileges来刷新权限。
```  
4. 重启mysql服务