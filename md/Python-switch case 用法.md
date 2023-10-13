# <center>Python-switch case用法

> 背景：python在3.10版本之前如果想要实现`switch case`功能，只能通过if/elif等其他方式

## 手动方式实现 switch case

```python
def switch(lang):
    if lang == "JavaScript":
        return "You can become a frontend developer."
    elif lang == "Java":
        return "You can become a backend developer."
    elif lang == "Python":
        return "You can become a Data Scientist"
    elif lang == "Swift":
        return "You can become a mobile app developer"

print(switch("JavaScript"))   
print(switch("Java"))   
print(switch("Swift"))  

"""
Output: 
You can become a web developer.
You can become a backend developer.
You can become a mobile app developer
"""
```

## 新版switch case 语法糖 - match case

```python
# 语法如下

match term:
    case pattern-1:
         action-1
    case pattern-2:
         action-2
    case pattern-3:
         action-3
    case _:
         action-default
```

```python
# 示例：
match lang:
    case "JavaScript":
        return "You can become a web developer."
    case "Java":
        return "You can become a backend developer."
    case "Python":
        return "You can become a Data Scientist"
    case "Swift":
        return "You can become a mobile app developer"  
    case _:
        return "You can become a developer." 
```

* 注：`case _` 代表default，如果没有匹配到任何case，就会执行default内容