#  <center> Git同时提交代码到多个仓库

> 背景：公司内部有私有部署的gitlab仓库，同时我自己也有github仓库，下面介绍两种思路来解决这个问题。

## 方式1：

- 使用github 和 gitlag 的 webhook 功能，配置后，提交代码到github后，自动触发gitlab的代码同步。

- 注：此方式需要 gitlab 拥有管理员权限才可启用 webhook 功能。

## 方式2：

- 使用git remote 来实现，同一份本地库关联多个远程库。

- 实现方式：
    
    1. 在本地库中添加多个远程库地址

        ```bash
        # 默认情况下我们会使用origin作为远程库的别名，这样提交代码的时候直接使用git push origin master 或 git push 就自动提交到默认仓库的指定分支了
        git remote add origin <your-repository-address>

        # 添加额外的远程库
        git remote add github <your-repository-address>
        ```

    2. 查看本地仓库关联的远程库

        ```bash
        git remote -v

        # 输出如下
        gitlab  <your-gitlab-repository-address> (fetch)
        gitlab  <your-gitlab-repository-address> (push)
        origin  <your-github-repository-address> (fetch)
        origin  <your-github-repository-address> (push)
        ``` 
    3. 提交代码到多个远程库

        ```bash
        # git push 默认会提交到 github 仓库
        git push

        # 提交到 gitlab 仓库
        git push gitlab master 
        # 或
        git push gitlab
        ```

- 注：此方式需要在本地库中添加多个远程库地址，提交代码的时候需要指定提交到哪个远程库。