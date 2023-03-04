# Linux环境快速安装Jmeter  

1 检查是否安装java环境

```
java -version
```

2 创建jmeter目录,进入该目录

```vim
mkdir /usr/local/jmeter
cd /usr/local/jmeter
```

3 下载安装Jmeter  

* 科学上网：直接下载安装  
```yml
wget https://archive.apache.org/dist/jmeter/binaries/apache-jmeter-5.3.tgz
```

* 手动方式：前往手动下载并上传 http://jmeter.apache.org/download_jmeter.cgi#binaries  

4 解压

```
tar -zxvf apache-jmeter-5.3.tgz
```

5 配置环境变量

```vim
vim /etc/profile

在文件最后添加
export JMETER_HOME=/usr/local/jmeter/apache-jmeter-5.1.1
export PATH=$JMETER_HOME/bin:$PATH
export CLASSPATH=$JMETER_HOME/lib/ext/ApacheJMeter_core.jar:$JMETER_HOME/lib/jorphan.jar:$JMETER_HOME/lib/logkit-2.0.jar:$CLASSPATH
```

6 配置生效

```vim
source /etc/profile
```

7 检查是否安装成功

```

jmeter -v
```

8 启动运行jmeter   
* Linux下使用非GUI方式启动运行jmeter脚本参考另一边博文
* MacOS下可以使用GUI方式启动jmeter，命令如下：

```vim
cd /usr/local/jmeter/apache-jmeter-5.1.1/bin/

sh jmeter
```