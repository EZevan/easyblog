# <center>Python解析Jmeter接口自动化脚本结果文件(.jtl文件)

1 分析.jtl文件结构  

    ```json
    timeStamp,elapsed,label,responseCode,responseMessage,threadName,dataType,success,failureMessage,bytes,sentBytes,grpThreads,allThreads,URL,Latency,IdleTime,Connect
    1593337383673,16,BeanShell Sampler - Empty downloads folder,200,OK,setUp Thread Group 1-1,text,true,,4,0,1,1,null,0,0,0
    1593337383885,1358,POST - /sso/sso/doLogin,200,OK,setUp Thread Group 1-1,text,true,,458,675,1,1,https://www.xxxxos.com/api/sso-web/sso/sso/doLogin,1355,0,1013
    1593337385315,180009,POST - /sso/sso/selectEnvironment,504,Gateway Time-out,setUp Thread Group 1-1,text,false,Expected to find an object with property ['success'] in path $ but found 'java.lang.String'. This is not a json object according to the JsonProvider: 'com.jayway.jsonpath.spi.json.JsonSmartJsonProvider'.,740,770,1,1,https://www.xxxxos.com/api/sso-web/sso/sso/selectEnvironment,180009,0,0
    ```
* 由17个属性构成：timeStamp，elapsed，label，responseCode等
* 每一个接口结果信息占一行（超出部分换行显示）
* 每一个接口固定以时间戳开头，每一个属性之间以逗号分隔 
* success属性：true表示接口执行成功，false表示接口执行失败 

2 python解析执行失败的接口

    ```python
    #coding=utf-8

    import json
    import requests
    import sys

    def getErrorMsg(file):
        sampleList = []
        contentArray = []

        fileContent = open(file,"r")
        for line in fileContent:
            if line[0:13].isdigit:
                line = line[14:]
                sampleList.append(line)
        print(sampleList)

        for sample in sampleList:
            contentArray = sample.split(",")
            if contentArray[6] == "false":
                errorMsg += "> Label: " + contentArray[1] + "  \n> ResponseCode: " + contentArray[2] + "  \n> ResponseMsg: " + contentArray[3] + "  \n> FailureMsg: " + contentArray[7] + "\n\n\n> "

        return errorMsg
    ```

注释：

    ```python  
    open(file,"r")                   # 内置函数读取.jtl文件，"r"表示默认的只读模式  
    for line in fileContent          # 遍历每行记录，这里 fileContent不需要使用fileContent.readlines(这样写会报错)   
    if line[0:13].isdigit:           # 判断是行的前13位均为数字（行是以13位时间戳开头）
        line = line[14:]             # 从第15位开始截取至末尾
        sampleList.append(line)      # 将截取的记录放进list

    for sample in sampleList         # 遍历sampleList，读取每一行的结果
    contentArray = sample.split(",") # 将每行记录使用逗号“,”拆分成数组
    if contentArray[6] == "false"    # 判断行记录数组中第7个属性success是否为false（即接口执行失败），失败则拼接出想要的失败信息
    ```

注：requests模块需要pip手动安装

    ```bash
    pip install requests
    ```