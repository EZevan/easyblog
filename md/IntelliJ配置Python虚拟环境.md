# <center>IntelliJ配置Python虚拟环境

> 当计算机已经安装过IntelliJ IDEA的情况下，可以在IntelliJ IDEA中安装插件的方式扩展Python开发功能，就能实现同Pycharm一样的功能了（Pycharm相当于IntelliJ IDEA的一个子集）

## 安装Python插件   
打开IntelliJ IDEA，进入`File - Settings - Plugins`，搜索插件`Python`，点击安装即可
![IntelliJ-01](../assets/img/IntelliJ%E9%85%8D%E7%BD%AEPython%E8%99%9A%E6%8B%9F%E7%8E%AF%E5%A2%83/IntelliJ-01.png "")

## 配置虚拟环境  
1. 安装`virtualenv` 
```py
# cmd或者shell执行
pip install virtualenv
# 若下载速度慢，可临时切换国内源
pip install 要安装的包名 -i http://mirrors.aliyun.com/pypi/simple/ --trusted-host mirrors.aliyun.com
``` 
2. 创建Python项目，进入项目根目录
3. 创建虚拟环境
```py
# 在项目根目录下创建名为`.venv`的虚拟环境
virtualenv .venv
```
4. 配置项目依赖的环境为虚拟环境
进入`File - Project Structure..`，或者快捷键`ctrl+alt+shift+s`打开项目结构配置下的`SDKS`，配置项目依赖的环境为刚刚创建的虚拟环境
![IntelliJ-02](../assets/img/IntelliJ%E9%85%8D%E7%BD%AEPython%E8%99%9A%E6%8B%9F%E7%8E%AF%E5%A2%83/IntelliJ-02.png "")
![IntelliJ-03](../assets/img/IntelliJ%E9%85%8D%E7%BD%AEPython%E8%99%9A%E6%8B%9F%E7%8E%AF%E5%A2%83/IntelliJ-03.png "")
5. 激活虚拟环境
```shell
# windows
.venv\Scripts\activate.bat
# linux/macOS
source venv/bin/activate
```
## 下载依赖库文件
虚拟环境配置完成激活后，此时即进入虚拟环境中，可进行依赖的安装
```py
pip install -r requirements.txt
```
配置完成后重启项目

