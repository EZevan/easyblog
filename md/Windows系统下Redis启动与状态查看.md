# Windows系统下Redis启动与状态查看

* 查看 Redis服务是否启动  

1. 使用 telnet 命令查看 ip 端口是否可连接
```py
telnet 127.0.0.1 6379

# 若提示无法连接到主机则表示 redis 服务未启动
```

2. 使用 redis-cli 命令查看是否可连接 redis
```py
# 进入 Redis 安装目录后输入如下命令
redis-cli

# 如果提示无法连接则表示 redis 服务未启动
```

* 手动启动 Redis 服务
```py
# 进入 Redis 安装目录后，输入
redis-server redis.windows.conf
```

* 将 Redis 服务创建为 Windows 服务开机启动
```py
# 进入 Redis 安装目录后，输入：
redis-server --service-install redis.windows.conf
# 执行完成后打开 Windows 服务即可看到新创建的 Redis 服务
```

* 查看 Redis 最大连接数以及当前连接用户数
```py
# redis-cli连接到 redis 后
# 查看连接池设置的最大连接数：
config get maxclients

# 查看当前连接用户数
info clients
```