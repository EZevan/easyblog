# <center>Python-搭建私有 pypi 仓库

> 背景：私有 pypi 仓库一方面可以提高下载速度，方便维护；另一方面针对非开源私有包可以更安全的使用、维护

## 使用 pypiserver 搭建私有仓库

### 安装 pypiserver

```bash
pip install pypiserver
```

### 准备私有 pypi 仓库目录

```bash
mkdir pypiserver
cd pyipserver
mkdir packages
```

### 启动 pypiserver

```bash
pypi-server run -p 8080 -P . -a . /pypiserver/packages --fallback-url https://mirrors.aliyun.com/pypi/simple/
```

* -p 8080 表示启动端口为 8080
* -P . -a . 表示使用当前目录下的 .htaccess 和 .htpasswd 文件进行认证, 即不需要认证
* /pypiserver/packages 表示私有仓库目录
* --fallback-url https://pypi.org/simple 表示当私有仓库中没有找到包时，会去 https://mirrors.aliyun.com/pypi/simple/ 中查找

### poetry 打包并上传到私有仓库

```bash
# 配置poetry私有仓库地址
poetry config repositories.private-pypi http://host:8080

# 打包并发布到私有仓库 private-pypi
poetry publish -r private-pypi

```

### 浏览器查看镜像仓库

* 访问 http://host:8080/simple/ 即可看到私有仓库中上传成功的包

### 从私有仓库安装 python 包

```bash
# 因为没有启用 https，所以需要加上 --trusted-host ip
pip install -i http://host:8080/simple/ 包名 --trusted-host ip

# 例如：
pip install -i http://testhub.taimei.com/simple/ domore-api --trusted-host testhub.taimei.com
```