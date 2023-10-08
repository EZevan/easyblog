# <center>CentOS7安装python11+最新openssl

> 背景：搭建 python 运行环境

## 准备工作

### 安装环境

```bash
# 准备安装环境
yum install -y gcc gcc-c++ autoconf automake zlib zlib-devel pcre-devel libffi-devel 
```

### 下载openssl&python11

```bash
# 浏览器访问https://www.openssl.org/source/ 确认最新的 openssl 版本
wget https://www.openssl.org/source/openssl-1.1.1u.tar.gz
wget https://www.python.org/ftp/python/3.11.4/Python-3.11.4.tgz
```

## 安装
### 安装 openssl

```bash
# 准备安装目录
mkdir /usr/local/openssl1.1.1

# 解压后编译、安装
tar -zxvf openssl-1.1.1u.tar.gz
cd openssl-1.1.1u
./config --prefix=/usr/local/openssl1.1.1
make && make install
```

### 安装 python
```bash
# 准备安装目录
mkdir /usr/local/python3.11.4

# 解压后编译、安装
tar -zxvf Python-3.11.4.tgz
cd Python-3.11.4
./configure --prefix=/usr/local/python3.11.4 --with-openssl=/usr/local/openssl1.1.1 --with-openssl-rpath=auto
make && make install
```

## 配置环境变量 - 软链

```bash
ln -f -s /usr/local/python3.11.4/bin/python3 /usr/bin/python3.11
ln -f -s /usr/local/python3.11.4/bin/pip3 /usr/bin/pip3.11
```

## 验证

```bash
python3 -V
```