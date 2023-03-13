# <center>Mac终端添加ll，la，l快捷命令

**Step 1**  
打开终端，打开.bash_profile配置文件

    ```
    vim ~/.bash_profile
    ```
注：如果.bash_profile文件不存在，则先创建

    ```
    touch ~/.bash_profile
    ```

**Step 2**   
编辑.bash_profile,添加如下代码

    ```
    alias ll='ls -alF'
    alias la='ls -A'
    alias l='ls -CF'
    ```
**Step 3**  
配置文件生效

    ```
    source ~/.bash_profile
    ```