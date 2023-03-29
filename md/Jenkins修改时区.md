# <center>Jenkins修改时区  

1. 访问http://jenkins_url/systemInfo,查看user.timezone变量值   

* 默认是美国/纽约时间   

2. 修改时区   
对于常规使用jenkins安装包安装的jenkins，通过修改启动配置文件修改：  

```bash
# 对于centos系统，修改 /ect/sysconfig/jenkins
sudo vim /etc/sysconfig/jenkins

# 修改JENKINS_JAVA_OPTIONS
JENKINS_JAVA_OPTIONS="-Djava.awt.headless=true -Dorg.apache.commons.jelly.tags.fmt.timeZone=Asia/Shanghai"

# 重启jenkins
service jenkins restart
```