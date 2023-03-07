# <center>Jenkins快速配置Jmeter接口自动话并输出报告</center>   

1 jenkins安装插件
* Performance Plugin：基于Jmeter脚本执行结果，跟踪各项性能KPI指标  
* HTML Publisher：用于发布HTML报告

2 配置执行jmeter的job  
* 在构建环境下启用：Delete workspace before build starts
* 新增构建步骤：执行Windows批处理命令（linux下选择执行shell命令即可）
```vim
"D:\Program Files\apach-jmeter-5.3\bin\jmeter" -n -t "D:\Program File\apache-jmeter-5.3\test script\api\trial_api_test.jmx" -l apireport.jtl -e -o report
```
命令释义：  
"D:\Program Files\apach-jmeter-5.3\bin\jmeter"   
表示调用机器上安装的jmeter（这里以Windows系统为例，使用双引号是因为路径格式问题；linux格式相同，使用不同的目录即可）

"D:\Program File\apache-jmeter-5.3\test script\api\trial_api_test.jmx"
表示待执行的jmeter脚本

参数说明：  
    -n : 非GUI模式执行JMeter  
    -t : 执行测试文件所在的位置及文件名  
    -r : 远程将所有agent启动用在分布式测试场景下，不是分布式测试只是单点就不需要-r  
    -l : 指定生成测试结果的保存文件， jtl 文件格式  
    -e : 测试结束后，生成测试报告  
    -o : 指定测试报告的存放位置  
    -o指定的文件及文件夹，必须不存在 ，否则执行会失败；对应上面的命令就是report文件夹必须不存在，否则报错（其实在jenkins任务构建开始启用Delete workspace before build starts后，就保证了report文件夹每次执行job的时候都不存在）

3 在“增加构建后操作步骤”下，选择“Publish HTML reports”
* 在HTML directory to archive中填写上述命令中的报告文件夹即可，即：report；其他保持默认配置

4 在“增加构建后操作步骤”下，选择“Publish Performacen test result report”
* 在Source data files（autodetects format）中填写：**/*.jtl；其他保持默认配置

5 至此，构建job后，就能看到jmeter脚本执行的结果报告了

**6 特别注意：**  
linux环境执行jmeter脚本报错：Error in NonGUIDriver java.lang.IllegalArgumentException: Problem loading XML from:'/usr/local/jmeter/apache-jmeter-5.1.1/testScripts/eTrial_api_test_env.jmx'.  Cause: CannotResolveClassException: kg.apc.jmeter.vizualizers.CorrectedResultCollector

解决方案：   
* 保证脚本运行机器和编写脚本的机器使用相同版本的jmeter
* jmeter脚本测试计划中所有的“名称”不能出现中文，如：控制器名称，取样器名称，断言名称等；其他诸如请求参数，beanshell代码注释，取样器备注等可以包含中文字符
* 脚本中存在类似于jp@gc - Active Threads Over Time 第三方的监听器，去除监听器（查看结果树和聚合报告可以保留）



