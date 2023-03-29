# <center>Linux环境快速安装Jenkins

* Prerequisite   
1. 获取安装列表  

```bash
yum -y list java*
``` 

2. 安装jdk环境   

```bash
yum -y install java
```

3. 测试jdk是否安装成功

```bash
java -version
```

4. 安装wget命令

```bash
yum install wget
```

5.1  第一种安装 - rpm方式：下载Jenkins安装包（成功后，jenkins可能会报错）

```bash
查看版本 http://pkg.jenkins-ci.org/redhat-stable/

wget http://repos.fedorapeople.org/repos/dchen/apache-maven/epel-apache-maven.repo -O /etc/yum.repos.d/epel-apache-maven.repo

wget http://pkg.jenkins-ci.org/redhat-stable/jenkins-2.7.3-1.1.noarch.rpm   
```

5.2 第二种安装：

```bash
sudo wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat-stable/jenkins.repo
sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io.key

yum install jenkins
```

6. 安装Jenkins

```bash
进入当前目录进行安装
rpm -ivh jenkins-2.7.3-1.1.noarch.rpm  

查看是否安装
rpm -qa|grep jenkins
```

7. 修改启动端口（默认8080）

```bash
vi /etc/sysconfig/jenkins
```

8. 重启jenkins服务

```bash
service jenkins restart
```

9. 查看初始管理员账户admin的登陆密码

```bash
cat /var/lib/jenkins/secrets/initialAdminPassword
```

10. 访问jenkins

```bash
浏览器访问：ip:port；如：192.168.100.114:8080
```

11. 初始配置，安装插件（推荐配置或者自定义插件配置）

```bash
注：在内网情况下，可能会出现一直卡在初始安装插件的Getting Startd界面；

jenkins在安装插件前总是尝试连接www.google.com,来判断网络是否连通。谷歌的网站在大陆是连不上的，所以会出现这个问题。

解决方案：将 /var/lib/jenkins/updates/default.json 文件中的http://www.google.com/ 改为 http://www.baidu.com，重启服务重新访问jenkins

注：如果此时该目录下还没有updates目录，可以先直接访问ip:port/pluginManager/advanced,进入插件更新配置界面，提交保存，即可看到/root/.Jenkins/updates/default.json文件
```

12. 安装完成以后service jenkins restart启动Jenkins的时候提示“Starting jenkins (via systemctl): Job for jenkins.service failed because the control process exited with error code. See "systemctl status jenkins.service" and "journalctl -xe" for details.”，此时无法启动Jenkins，需要执行systemctl status jenkins.service来检查下其状态；

通过检查Jenkins的运行状态我们可以捕捉到启动的时候报错“Starting Jenkins bash: /usr/bin/java: No such file or directory”，没有找到jdk

解决方案：命令 which java 找到jdk安装路径；打开“Jenkins的启动脚本路径/etc/rc.d/init.d/jenkins”，将实际的jdk路径添加到该文件中。

13. 卸载jenkins

```bash
service jenkins stop
yum clean all  //清理yum缓存，不执行也可以重新安装
yum -y remove jenkins
rm -rf /var/cache/jenkins
rm -rf /var/lib/jenkins/
```