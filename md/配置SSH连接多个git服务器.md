# <center>配置SSH连接多个git服务器</center>  

1. 创建个人SSH key  

    ```shell
    # 查看本机是否存在ssh key

    # macOS/linux：
    cd ~/.ssh
    ls

    # Windows:
    cd "C:\Users\yourUsername\.ssh"

    # 新建SSH key(以macOS为例)：  
    cd ~/.ssh       
    ssh-keygen -t rsa -C "youremail@example.com"

    # 设置名称，如：id_rsa_gitlab 
    Enter file in which to save the key (/Users/evanzhang/.ssh/id_rsa): id_rsa_gitlab

    # 输入密码（个人机器工作环境的话建议可以无需密码，否则每次进行git提交，拉取操作都会要求校验密码；直接回车即可）
    Enter passphrase (empty for no passphrase):

    ```

2. 添加并配置config文件   
*注：若~/.ssh/目录下不存在config文件，则新建一个，内容如下：*   

```shell
#Host 服务器别名
Host git.taimei.com
#HostName 服务器ip地址或机器名
HostName git.taimei.com
#User连接服务器的用户名
User wenqiang.zhang
PreferredAuthentications publickey
#IdentityFile 密匙文件的具体路径
IdentityFile ~/.ssh/id_rsa_gitlab

Host github.com
HostName github.com
User evan.zhang
PreferredAuthentications publickey
IdentityFile ~/.ssh/id_rsa
```

3. 添加公钥至远程git服务器（github，gitlab等）  
* 把 ~/.ssh/id_rsa_gitlab.pub 文件中所有内容粘贴到GitLab的SSH keys中  

4. 测试
```shell
#使用 ssh -T git@Host进行测试，其中Host指上面配置的服务器别名
ssh -T git@git.taimei.com
ssh -T git@github.com
```

5. 可能遇到的问题   
出现Could not open a connection to your authentication agent；     
或者提交代码时依然需要验证远程仓库用户密码；

解决：
将ssh私钥添加到ssh agent中
```shell
ssh-add ~/.ssh/id_rsa
ssh-add ~/.ssh/id_rsa_gitlab

#此时如果提示类似"ermissions 0777 for 'id_rsa' are too open."，执行如下：
chmod 400 id_rsa id_rsa_gitlab
```
