# docker容器镜像常用操作命令   

* 容器相关  
```shell
# 列出所有容器（运行和停止状态的）
docker ps -a

# 列出所有容器ID
docker ps -aq

# 停止所有容器（删除镜像前必须停止容器）
docker stop $(docker ps -aq)

# 停止指定容器（指定容器名称等关键字，如：hello-world）
# 命令解释：awk '/hello-world/{print $1}'命令用来显示含有"hello-world"关键字的行的第1个字段；这里表示所有含有"hello-world"关键字的容器的第一个字段，即容器ID
docker stop $(docker ps -a | awk '/hello-world/{print $1}')

# 删除所有容器
docker rm $(docker ps -aq)

# 删除指定容器（指定容器名称等关键字，如：hello-world）
docker rm $(docker ps -a | awk '/hello-world/{print $1}')
```

* 镜像相关    
```shell
# 查看所有镜像
docker images

# 删除单个镜像（通过镜像ID指定）
docker rmi <image id>

# 删除所有镜像
docker rmi $(docker images -q)

# 强制删除所有镜像
docker rmi -f $(docker images -q)

# 删除指定镜像（如指定镜像仓库名或者标签等,如：hello-world）
docker rmi $(docker images | awk '/hello-world/{print $3}')

```