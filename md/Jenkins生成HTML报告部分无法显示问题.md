# <center>Jenkins生成HTML报告部分无法显示问题
jenkins 配置使用html publisher查看jmeter html报告时，发现显示不全，很多东西显示不了。
 
项目配置：  
![project configuration](../assets/img/Jenkins%E7%94%9F%E6%88%90HTML%E6%8A%A5%E5%91%8A%E9%83%A8%E5%88%86%E6%97%A0%E6%B3%95%E6%98%BE%E7%A4%BA%E9%97%AE%E9%A2%98/jenkins-01.jpg)
 
查看html报告异常（很多资源无法加载）：  
![htlm report](../assets/img/Jenkins%E7%94%9F%E6%88%90HTML%E6%8A%A5%E5%91%8A%E9%83%A8%E5%88%86%E6%97%A0%E6%B3%95%E6%98%BE%E7%A4%BA%E9%97%AE%E9%A2%98/jenkins-02.jpg)
 

控制台查看加载日志（与安全机制有关）：   
![report console](../assets/img/Jenkins%E7%94%9F%E6%88%90HTML%E6%8A%A5%E5%91%8A%E9%83%A8%E5%88%86%E6%97%A0%E6%B3%95%E6%98%BE%E7%A4%BA%E9%97%AE%E9%A2%98/jenkins-03.jpg)


在查看官方文档后，这原来是安全问题所导致的。
Jenkins安全默认是将以下功能都关闭了
1、javascript
2、html上的内置插件
3、内置css或从其它站的css
4、从其它站的图处
5、AJAX
 
我的网页使用的是css和Js，所以显示不全。解决方法如下：
在jenkins系统管理中输入以下脚本运行：
System.setProperty("hudson.model.DirectoryBrowserSupport.CSP", "")
 
如下图：  
![script command line](../assets/img/Jenkins%E7%94%9F%E6%88%90HTML%E6%8A%A5%E5%91%8A%E9%83%A8%E5%88%86%E6%97%A0%E6%B3%95%E6%98%BE%E7%A4%BA%E9%97%AE%E9%A2%98/jenkins-04.jpg)

* 注：上述的解决方案只能是临时的，即当jenkins实例重启后，又需要重新执行该脚本；为了彻底解决该问题，可以新建一个job专门用来执行该脚本，触发时机为当jenkins实例启动时。

1 安装插件：Groovy；Startup Trigger  
2 全局工具配置：自动方式安装Groovy工具，版本使用默认即可  
3 新建一个自由风格的job  
4 配置job - 构建触发器：启用Build when job nodes start，Quiet period属性填0即可  
5 配置job - 增加构建步骤：Execute Groovy script  
6 在Execute Groovy script中填写如下命令即可：

    ```shell
    System.setProperty("hudson.model.DirectoryBrowserSupport.CSP", "")
    ```
 其中，Groovy Version选择“全局工具配置”中的Groovy名称即可。
 