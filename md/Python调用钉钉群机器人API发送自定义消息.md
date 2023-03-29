# <center>Python调用钉钉群机器人API发送自定义消息

> 背景：  
> 1 Jenkins构建Jmeter接口自动化脚本，自动将报错接口信息自动发送至钉钉群；  
> 2 由于CI部署在内网，需要远程能在手机上直接查看钉钉机器人消息即可获取具体接口异常   
> 3 Jenkens自带的钉钉通知插件（DingTalk）只会发送构建成功/失败结果，详细的接口报错还需要通过链接跳转至Jenkins查看（远程移动端没有vpn的情况下无法访问内网）

### 解决方案（Workaround）  
使用python调用钉钉机器人API发送自定义消息

1 钉钉开放平台：自定义机器人开发  
https://ding-doc.dingtalk.com/doc#/serverapi3/iydd5h   
支持text，link，markdown等多种消息类型，这里以markdown为例进行简单介绍   

2 简单易用的python脚本  

```python
#coding=utf-8

import json
import requests
import sys

def sendMsg(url,file):
    errorMsg = getErrorMsg(file)
    headers = {'Content-Type': 'application/json;charset=utf-8'}
    data = {
        "msgtype": "markdown",
        "markdown": {
            "title": "测试",
            "text": "#### trial-api-test \n> " + errorMsg + " \n> ### [问题连接](https://www.dingtalk.com) \n",
        },
        "at": {
            "atMobiles":[]，
            "isAtAll": False
        }
    }

    if len(errorMsg) > 0:
        response = requests.post(url,data = json.dumps(data),headers=headers,verify=False)
    return response.text
```

注释：  
* errorMsg：通过getErrorMsg(file)方法获取jmeter自动化接口执行结果文件（.jtl文件）中所有报错接口的错误信息
* headers：请求头
* data：构造请求体，即将要发送至钉钉群的消息,其中:  
msgtype表示消息类型为markdown，仅支持几种常用的md语法；  
markdown为消息内容；   
at可以@指定的用户接受消息，或者@所有用户
* requests.post()：将构造的消息通过requests的post方法发送至指定钉钉，其中：  
url为钉钉群机器人的完整webhook（这里包含accessToken）；  
data为json格式请求参数   
headers为请求头  
verify表示是否验证ssl证书（有可能出现请求报错requests.exceptions.SSLError:HTTPSConnectionPool(host='xxx',443)，验证证书失败），这里直接使用False忽略证书验证，保证不会报错

_注：getErrorMsg(file)方法参考另一篇博文_