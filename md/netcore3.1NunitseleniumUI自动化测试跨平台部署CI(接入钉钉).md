# netcore3.1 + Nunit + selenium UI自动化测试跨平台部署CI（接入钉钉)

1. 准备
* .netcore3.1+NUnit+selenium UI自动化测试框架 - Swat
* 自动化测试脚本项目，这里以trial-ui-auto-test为例
* jenkins 2.235.2 - 已安装
* Docker 19.03.13 - 已安装

2. 思路
* 将测试项目从安装依赖，构建到执行测试放在docker容器中运行；然后jenkins将测试报告和日志归档，构建完成后自动发送消息至钉钉群

3. 准备Dockerfile
```python
FROM mcr.microsoft.com/dotnet/core/sdk:3.1 AS builder

ENV DEBIAN_FRONTEND noninteractive

# Install Chrome
RUN sed -i s@/archive.ubuntu.com/@/mirrors.aliyun.com/@g /etc/apt/sources.list \
&& sed -i s@/deb.debian.org/@/mirrors.aliyun.com/@g /etc/apt/sources.list \
&& sed -i s@/security.debian.org/@/mirrors.aliyun.com/@g /etc/apt/sources.list \
&& apt-get clean \
&& apt-get update && apt-get install -y \
apt-transport-https \
ca-certificates \
curl \
gnupg \
hicolor-icon-theme \
libcanberra-gtk* \
libgl1-mesa-dri \
libgl1-mesa-glx \
libpango1.0-0 \
libpulse0 \
libv4l-0 \
fonts-symbola \
--no-install-recommends \
&& curl -sSL https://dl.google.com/linux/linux_signing_key.pub | apt-key add - \
&& echo "deb [arch=amd64] https://dl.google.com/linux/chrome/deb/ stable main" > /etc/apt/sources.list.d/google.list \
&& apt-get update && apt-get install -y \
google-chrome-stable \
--no-install-recommends \
&& apt-get purge --auto-remove -y curl \
&& rm -rf /var/lib/apt/lists/*

# install chinese fonts
RUN apt-get update \
&& apt-get -y install ttf-wqy-microhei ttf-wqy-zenhei \
&& apt-get clean

# copy full solution over  and restore package
# define the following working directory,so that the screenshot will be displayed properly in the jenkins report.
COPY . /app
WORKDIR /app
RUN dotnet restore \
&& dotnet build

WORKDIR /app/Swat.Test/bin/Debug/netcoreapp3.1
RUN ls -a

FROM builder AS testRunner
WORKDIR /job/trial-ui-auto-test/report
COPY --from=builder /app .
WORKDIR /job/trial-ui-auto-test/report/Swat.Test/
ENTRYPOINT  ["dotnet", "test", "--logger:trx"]
```
脚本解释：  
> 1. FROM mcr.microsoft.com/dotnet/core/sdk:3.1 AS builder 表示将dotnetcore3.1作为基础镜像
> 2. ENV DEBIAN_FRONTEND noninteractive 由于在更新apt-get命令相关依赖时，会涉及的选择时区，会终端自动化执行命令，该命令表示设置一个DEBIAN_FRONTEND变量，不使用交互模式
> 3. 紧接着两个RUN命令分别是下载安装headless chrome和下载安装中文字体（因为linux环境默认没有中文，在运行chrome时导致中文乱码）   
注：apt-get update，更新过程中由于是国外资源，异常慢甚至可能失败（踩坑了），将源换成国内源：使用该命令RUN sed -i s@/archive.ubuntu.com/@/mirrors.aliyun.com/@g /etc/apt/sources.list      
注：每一行命令末尾使用"\"进行换行，换行后行首使用"&&"连接符，以此来链接多条RUN命令；这样可以达到优化Dockerfile的目的  
> 4. COPY . /app 将上下文机器当前路径下的所有内容复制到容器中的/app目录下
> 5. WORKDIR /app 切换到容器的/app目录下
> 6. RUN dotnet restore \
    && dotnet build  
   这两个命令分别进行下载依赖，构建项目
> 7. WORKDIR /app/Swat.Test/bin/Debug/netcoreapp3.1  
   RUN ls -a   
   查看依赖是否构建完成
> 8. FROM builder AS testRunner 将builder作为基础镜像
> 9. WORKDIR /job/trial-ui-auto-test/report 切换到当前镜像的该路径下
   注：这里使用/job/trial-ui-auto-test/report的目的是为了构造自动化执行过程中，截图的保存路径，以便后续在jenkins服务器上归档测试报告中能正确显示截图
> 10. COPY --from=builder /app .  将builder镜像中编译构建的内容全部复制到当前镜像的/job/trial-ui-auto-test/report目录下
> 11. WORKDIR /job/trial-ui-auto-test/report/Swat.Test/   
    ENTRYPOINT  ["dotnet", "test", "--logger:trx"]   
    执行dotnet test，启动自动化测试   

4. jenkins构建docker
```
cd ${WORKSPACE}

# Remove the last executed docker container and image
sudo docker rm $(sudo docker ps -a | awk '/trial.ui.autotest/{print $1}')
sudo docker rmi $(sudo docker images | awk '/trial.ui.autotest/{print $3}')

# Copy the docker file to current directory
sudo cp ${JENKINS_HOME}/scripts/Dockerfile .

# Run the docker
sudo docker build -t trial.ui.autotest:latest .
sudo docker run --name trial.ui.autotest_container trial.ui.autotest
sudo mkdir -p ./log && sudo docker cp trial.ui.autotest_container:/job/trial-ui-auto-test/report/Swat.Test ./log
```
命令解释：   
sudo docker build -t trial.ui.autotest:latest . 构建镜像   
sudo docker run --name trial.ui.autotest_container trial.ui.autotest 运行docker容器，容器名称：trial.ui.autotest_container   
sudo mkdir -p ./log && sudo docker cp trial.ui.autotest_container:/job/trial-ui-auto-test/report/Swat.Test ./log  将容器中/job/trial-ui-auto-test/report/Swat.Test目录拷贝到机器的./log目录

5. 将测试生成的报告通过Html Report插件归档到jenkins服务器  
HTML directory to archive:log  表示将要归档的目录，log表示相对于当前${workspace}的目录，对应我这里的目录就是：/var/lib/jenkins/workspace/trial-ui-auto-test/log    
Index page[s]:Swat.Test/bin/Debug/netcoreapp3.1/Logs/reports/$  {date}/index.html 将要归档的报告的首页	    
Report title:report 报告的标题，既报告url的最后一层路由  
注：以我这个jenkins项目为例，报告归档的url：http://myhost/job/trial-ui-auto-test/report/ ,这样默认实际打开的是Swat.Test/bin/Debug/netcoreapp3.1/Logs/reports/${date}/index.html这个文件（${date}是当前日期）
