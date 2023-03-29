#  <center> Linux 后台运行任务

* 先了解下 nohup 和 & 两者的区别  
`nohup` 运行命令可以使命令永久执行下去，和用户终端无关，即使用户断开ssh连接也不会影响；注：nohup 没有后台运行的功能。   
`&` 是指后台运行任务，但当用户推出、挂起（hangup）的时候，任务也跟着退出。

* 于是，可以结合 `nohup` 和 `&`在后台永久运行某个任务/命令

```shell
# 推荐示例
nohup python test.py > out.log 2>&1 &
```

**参数说明**   

* `python test.py` 是需要后台运行的命令
* `>` 将运行的输出日志追加到`out.log`文件中；nohup 会默认将`stdout`输出到指定文件中
* `1` 表示stdout(standard output)，nohup command > out.log等价于 nohup command 1 > out.log；
* `2` 表示 stderr(standard error);
* `2>&1` 其中`&`是一个描述符，如果1或2前不加&，会被当成一个普通文件，故`2>&1`表示把标准错误重定向到标准输出文件中