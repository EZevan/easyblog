# <center>CentOS 7安装MySQL8.x  

1. 删除旧版本mysql

    ```py
    # 查看是否已经安装过旧版本 
    rpm -qa | grep mysql

    # 结果为空表示未安装任何版本的mysql，否则需要手动删除结果中的所有安装包,如：
    mysql-community-common-8.0.25-1.el7.x86_64
    mysql-community-libs-compat-8.0.25-1.el7.x86_64
    mysql80-community-release-el7-3.noarch
    mysql-community-client-plugins-8.0.25-1.el7.x86_64
    mysql-community-libs-8.0.25-1.el7.x86_64
    mysql-community-server-8.0.25-1.el7.x86_64
    mysql-community-client-8.0.25-1.el7.x86_64 

    # 关闭mysqld服务
    systemctl stop mysqld

    # 使用以下命令删除上述安装包
    yum remove mysql-community-common mysql-community-libs-compat mysql-community-client-plugins mysql-community-libs mysql-community-server mysql-community-client

    # 最后清理残留的mysql相关文件
    whereis mysql

    # 使用find全盘扫描
    find / -name mysql

    # 使用rm清楚查出的所有目录文件
    rm -rf xxx/mysql
    ```   

2. 安装MySQL  
> 注：centos的yum 源中默认是没有mysql的，所以我们需要先去官网下载mysql的repo源并安装；mysql官网下载链接：mysql repo下载地址 如下：https://dev.mysql.com/downloads/repo/yum/  

    ```py
    # 使用任意版本的rpm包均可，如：mysql80-community-release-el7-3.noarch.rpm

    # 下载
    wget -i -c https://dev.mysql.com/get/mysql80-community-release-el7-3.noarch.rpm

    # 执行结果
    会在/etc/yum.repos.d/目录下生成两个repo文件mysql-community.repo mysql-community-source.repo
    ```  

3. 更新yum命令（相关依赖）

    ```
    yum clean all
    yum makecache
    ```

4. 安装MySQL

    ```py
    yum install mysql-community-server -y
    ```

5. 开启MySQL服务

    ```
    systemctl start msqld.service
    ```

6. 初始化

    ```py
    #mysql在安装后会创建一个root@localhost账户，并且把初始的密码放到了/var/log/mysqld.log文件中

    cat /var/log/mysqld.log | grep password

    # 使用初始密码登录
    mysql -u root -p

    # 此时任何数据库操作都会提示需要使用ALTER USER语句修改密码
    ALTER USER 'root'@'localhost' IDENTIFIED BY 'youpassword';
    ```

7. 防火墙开放3306端口

    ```py
    # 查看所有已开放的端口
    firewall-cmd --zone=public --list-ports

    # #开启一个端口，添加--permanent永久生效，没有此参数重启后失效
    firewall-cmd --zone=public --add-port=80/tcp --permanent

    # 开放端口3306
    firewall-cmd --permanent --add-port=3306/tcp

    # 移除端口3306
    firewall-cmd --permanent --remove-port=3306/tcp

    # 重启防火墙，修改后重启防火墙生效
    firewall-cmd --reload
    ```

8. 将MySQL添加到开机启动项,并启动

    ```
    systemctl enable mysqld.service
    systemctl start mysqld.service
    ```

9. 开启远程访问权限

    ```
    # 命令行访问进入mysql，修改user表中root用户的host为『%』
    mysql -u root -p
    use mysql
    update user set host='%' where user='root'
    ```





