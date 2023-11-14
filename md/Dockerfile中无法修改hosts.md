# <center>Dockerfile中无法修改hosts

## 背景描述
- 构建 python 应用镜像时，需要修改 hosts 文件，以便能够访问内网的私有仓库，在 Dockerfile 中尝试直配置 hosts 均无法生效

    1. 在 Dockerfile 中使用 `RUN cp hosts /etc/hosts` 命令替换 hosts 文件，但是在构建镜像时，发现 hosts 文件并没有被修改
    2. 在 Dockerfile 中使用 `RUN echo "192.168.xxx.xxx xxx.com" >> /etc/hosts` 命令追加 hosts 文件，但是在构建镜像时，会报错，提示 hosts 文件只读

## 问题分析
- Dockerfile 中的每一条指令都会创建一个新的镜像层，而每一层都是只读的，所以在 Dockerfile 中修改 hosts 文件是无效的

    > Dockerfile 中的每一条指令都会创建一个新的镜像层，而每一层都是只读的。因此，如果你在 Dockerfile 中使用 `RUN echo "192.168.xxx.xxx xxx.com" >> /etc/hosts` 命令追加 hosts 文件，那么在构建镜像时，会报错，提示 hosts 文件只读

## 解决方案

1. 在 Dockerfile 中使用 `COPY` 命令将 hosts 文件拷贝到镜像中，然后在容器启动时，使用 `CMD` 命令将 hosts 文件拷贝到 `/etc/hosts` 中，再从内网仓库安装依赖包

    Dockerfile 文件内容如下：

    ```dockerfile
    FROM python:3.11-slim

    COPY hosts /etc/hosts

    CMD cp /hosts /etc/hosts && pip install -i http://xxx.com/pypi/simple/ package_name
    ```
2. 如果是通过 docker-compose 编排并启动容器，可以在 docker-compose.yml 文件中配置 `command` 命令，将 hosts 文件拷贝到 `/etc/hosts` 中，再从内网仓库安装依赖包

    docker-compose.yml 文件内容如下：

    ```yaml
    version: '3.7'
    services:
      app:
        build:
          context: .
          dockerfile: Dockerfile
        command: /bin/bash -c "cp /hosts /etc/hosts && pip install -i http://xxx.com/pypi/simple/ package_name"
        volumes:
          - ./hosts:/hosts
    ```
3. 如果是通过 docker-compose 编排并启动容器，可以在 docker-compose.yml 文件中配置 `volumes` 参数，将 hosts 文件挂载到容器中，再从内网仓库安装依赖包

    docker-compose.yml 文件内容如下：

    ```yaml
    version: '3.7'
    services:
      app:
        build:
          context: .
          dockerfile: Dockerfile
        volumes:
          - ./hosts:/etc/hosts
        command: /bin/bash -c "pip install -i http://xxx.com/pypi/simple/ package_name && python app.py"
    ```
4. 如果是通过 docker run 命令启动容器，可以在启动容器时，使用 `-v` 参数将 hosts 文件挂载到容器中，再从内网仓库安装依赖包

    ```shell
    docker run -it -v /path/to/hosts:/hosts python:3.11-slim /bin/bash -c "cp /hosts /etc/hosts && pip install -i http://xxx.com/pypi/simple/ package_name"
    ```