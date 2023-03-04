# Linux快速安装python3.8（踩坑指南）

> 安装环境：centos7.5  
> 安装版本：python3.8.5  

1. python依赖包安装  
**注：在安装过程中会校验python所有依赖，如果缺失会安装失败，相对于指定某些必要依赖的安装方式，最好是直接安装所有依赖的开发工具包**

```c
yum -y groupinstall "Development Tools"
```

2. 下载python安装包  
方式1：科学上网   
```c
wget https://www.python.org/ftp/python/3.8.0/Python-3.8.5.tgz 
```

方式2：下载到本地手动上传  
https://www.python.org/ftp/python/3.8.0/Python-3.8.5.tgz

3. 解压   
```
tar tar -zxvf Python-3.8.5.tgz
```

4. 安装
```c
cd Python-3.8.5
./configure --prefix=/usr/local/python3
make && make install
```
**注意：如果没有root权限，上述安装可能会不成功，改成如下命令即可**
```c
cd Python-3.8.5
sudo ./configure --prefix=/usr/local/python3
sudo make
sudo make instalcl
```

5. 建立软链接
```
ln -s /usr/local/python3/bin/python3.8 /usr/bin/python3
ln -s /usr/local/python3/bin/pip3.8 /usr/bin/pip3
```

6. 设置环境变量
```shell
macOS：
vim ~/.bash_profile
linux：
vim etc/profile
# 追加上如下内容
PATH=$PATH:$HOME/bin:/usr/local/python3/bin
export PATH

#生效
source ~/.bash_profile
```

7.pip3安装第三方库超时报错的workaround - 切换安装镜像源   
国内主流的几个源：
```
阿里云 http://mirrors.aliyun.com/pypi/simple/
中国科技大学 https://pypi.mirrors.ustc.edu.cn/simple/
豆瓣(douban) http://pypi.douban.com/simple/
清华大学 https://pypi.tuna.tsinghua.edu.cn/simple/
中国科学技术大学 http://pypi.mirrors.ustc.edu.cn/simple/
```
以阿里云为例：
```
pip install web.py -i http://mirrors.aliyun.com/pypi/simple/
```
**注：如果报错提示不支持http协议，可以换成其他https源或者使用如下命令**
```
pip install web.py -i http://mirrors.aliyun.com/pypi/simple/ --trusted-host mirrors.aliyun.com
```
