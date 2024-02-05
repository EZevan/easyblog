# <center>Python-判断key或value是否在枚举中

> 背景：获取 jira issues 信息时，只想要期望的指定字段，而不是所有字段，所以需要指定自定义字段范围。考虑使用枚举类，以实现字段的可扩展性。

## 使用枚举类的好处

1. 代码更加清晰，可读性更高
    - 通过枚举类，可以清晰的看到所有的字段，而不是直接使用字符串，需要查看源码才能知道有哪些字段
2. 代码更加健壮，不容易出错
    - 通过枚举类，可以限定字段的范围，不容易出现拼写错误，或者字段不存在的情况
    - 通过枚举类，可以限定字段的类型，不容易出现类型错误的情况
    - 通过枚举类，可以限定字段的取值范围，不容易出现取值错误的情况
    - 通过枚举类，可以限定字段的默认值，不容易出现默认值错误的情况
3. 代码更加易于维护，可扩展性更强
    - 通过枚举类，可以方便的扩展字段，而不需要修改原有代码
    - 通过枚举类，可以方便的修改字段的默认值，而不需要修改原有代码
    - 通过枚举类，可以方便的修改字段的取值范围，而不需要修改原有代码
    - 通过枚举类，可以方便的修改字段的类型，而不需要修改原有代码

## 使用枚举类取值

如：有如下枚举类

```python
from enum import Enum
class EnumDemo(Enum):
    RED_COLOR = 'red'
    BLUE_COLOR = 'blue'
    GREEN_COLOR = 'green'
```
1. 获取枚举类的字段名
    - `EnumDemo.__members__.keys()`

2. 获取枚举类指定字段名
    - `EnumDemo.RED_COLOR.name`

3. 获取枚举类的字段值
    - `EnumDemo.__members__.values()`

4. 获取枚举类指定字段值
    - `EnumDemo.RED_COLOR.value`

## 判断key或value是否在枚举中(某个指定值是否在枚举类中)

    ```python
    from enum import Enum
    class testEnum(Enum):
        KEY1 = 0
        KEY2 = "yes"

    "key1" in testEnum.__members__
    0 in testEnum._value2member_map_
    ```

1. 判断key是否在枚举类中
    - `"key1" in testEnum.__members__`

2. 判断value是否在枚举类中
    - `0 in testEnum._value2member_map_`
    - `'yes' in testEnum._value2member_map_`


