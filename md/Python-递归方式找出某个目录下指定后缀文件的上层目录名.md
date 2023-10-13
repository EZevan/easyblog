# <center>Python-递归方式获取某个目录下指定后缀文件的上层目录名

> 背景：跟 CICD 对接过程中，想要知道接口自动化测试脚本所在的目录名，以便告诉 CICD 哪些目录下的脚本是可以用的

## 需求描述

- 不同环境不同应用的自动化测试脚本都放在单独的目录下管理，结构如下：  
```
    test
    ├── app1              
    │   ├── app1-test.har
    │   ├── app1-test.json
    │   └── app1-test.py    
    ├── app2             
    │   ├── app2-test.har
    │   ├── app2-test.json
    │   └── app2-test.py
    ├── app3              
    |   ├── app3-test.har
    │   ├── app3-test.json
    │   └── app3-test.py
    uat
    ├── app1              
    │   ├── app1-test.har
    │   ├── app1-test.json
    │   └── app1-test.py    
    └── app2             
        ├── app2-test.har
        ├── app2-test.json
        └── app2-test.py
```
- 其中test、uat 表示不同环境，在其下面存在不同的应用目录，分别是 app1、app2、app3，每个应用目录下面都有一个特定的测试脚本文件app1-test.har，现在想要获取这个文件所在的`环境`和`应用`名称，即：{"test": ["app1", "app2", "app3"], "uat": ["app1", "app2"]}这样的字典结构

## 实现思路

- 递归的方式找到`.har`后缀文件，然后获取其上层的目录名即为应用名，再获取上层的目录名即为环境名，最后将其存储到字典中

## 代码实现

```python
import os

# 声明一个字典，用于存储环境和应用名
env_app = {}

# 递归方法实现

def get_app_names(path: str, app_names_out: dict)
"""
:param path: the path of specified directory
:param app_names_out: the dict to store the env and app names
:return: the dict of app names
"""
# 声明一个列表，用于存储应用名
app_names = []
