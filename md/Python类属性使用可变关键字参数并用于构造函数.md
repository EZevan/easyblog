# <center>Python 类属性使用可变关键字参数并用于构造函数

1. python类属性中使用可变关键字参数

```py
class CustomException(Exception):
    def __init__(response_message_enum: ResponseEnum, **kargs: any):    # 构造函数中使用可变关键字参数 **kargs
        self response_code = response_message_enum
        self extra_message = {**kargs}      # 使用字典类型传递可关键字参数
```

这样在使用raise CustomException实例化的时候可以传入需要的可变关键字参数来定制内容，如：

```py
if not name:
    raise CustomException(ResponseEnum.code, message="参数不能为空"， args=name)
```