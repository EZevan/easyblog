# <center>CentOS7 安装docker及docker-compose 

## 配置要求
系统：Centos7
Linux 内核：官方建议 3.10 以上

注意：本文的命令使用的是 root 用户登录执行，不是 root 的话所有命令前面要加 sudo

1. 查看当前的内核版本

    ```bash
    uname -r
    ```

2. 更新 yum 包（使用 root 权限，生产环境中此步操作需慎重，耗时较长，该步骤可以跳过）

    ```bash
    # yum -y update 升级所有包同时也升级软件和系统内核；
    # yum -y upgrade 只升级所有包，不升级软件和系统内核
    yum -y update
    ```

3. 卸载旧版本（如果之前安装过的话）

    ```bash
    yum remove docker  docker-common docker-selinux docker-engine
    ```

## 安装Docker

1. 1.安装软件包
- 安装需要的软件包，yum-util 提供yum-config-manager功能，另两个是devicemapper驱动依赖

    ```bash
    yum install -y yum-utils device-mapper-persistent-data lvm2
    ```

2. 设置 yum 源

    ```bash
    #（中央仓库）
    yum-config-manager --add-repo http://download.docker.com/linux/centos/docker-ce.repo

    # （阿里仓库）
    yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
    ```

3. 选择docker版本
- 查看可用版本有哪些

    ```bash
    yum list docker-ce --showduplicates | sort -r
    ```

4. 安装docker
- 选择一个版本并安装：yum install docker-ce-版本号

    ```bash
    yum -y install docker-ce-18.03.1.ce
    ```

5. 启动 Docker

    ```bash
    systemctl start docker

    # 设置开机自启
    systemctl enable docker
    ```  

6. 查看docker版本

    ```bash
    docker version
    ```

## 安装 Docker Compose

1. 下载Docker Compose二进制文件：使用以下命令下载Docker Compose的最新稳定版本

    ```bash
    sudo curl -L "https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
    ```
- 注：大概率会出现下载失败的情况，可直接前往 github 上下载 release 版本的二进制文件

2. 手动下载Docker Compose二进制文件

    ```bash
    # 使用以下命令查看适合的版本信息
    uname -s   # 查看系统内核版本，可能是 Linux 或者 Darwin
    uname -m   # 查看系统位数，可能是 x86_64 或者 arm64

    # 如下载文件：docker-compose-linux-x86_64
    ```

3. 为二进制文件添加可执行权限
- 将 docker-compose 文件上传服务器后，为二进制文件添加可执行权限

    ```bash
    chmod +x /usr/local/bin/docker-compose
    ```

4. 建立软链

    ```bash
    ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose
    ```

5. 查看docker-compose版本

    ```bash
    docker-compose version
    ``` 