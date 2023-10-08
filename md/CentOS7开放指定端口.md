# <center>CentOS7开放指定端口

> 背景：CentOS启动指定端口的服务无法被访问

## 开放指定端口

```bash
firewall-cmd --zone=public --add-port=端口号/tcp --permanent 
```

## 查看防火墙状态

```bash
systemctl status firewalld.service
```

## 重启防火墙

```bash
systemctl restart firewalld.service
```

## 查看已开放端口

```bash
firewall-cmd --list-ports
```