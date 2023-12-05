# <center>Python-typing使用

> 官方文档: [typing](https://docs.python.org/zh-cn/3/library/typing.html)  
> 注: typing是python3.5(PEP484)开始的  
> 可用于: 类型检查器、集成开发环境、静态检查器等工作, 但是不强制使用

## 使用方式

- 两种使用方式
1. 别名：先用`=`定义别名，使用的时候用`:`

    ```python
    from typing import List

    T = List[str]

    def demo(t1: T) -> T:
        return t1
    ```
2. 直接使用：使用的时候直接用`:`
    ```python
    from typing import List

    def demo(t1: List[str]) -> List[str]:
        return t1
    ```

## 创建新类型 NewType
> 用于创建新类型，但是不会创建新的类，只是创建了一个别名
> `NewType(name, tp)` 返回其原本的值  
> 静态类型检查器会将新类型看作是原始类型的一个子类; 即: `name`是新的类型 `tp`是原始类型   
> 注: 在python3.10之前NewType是普通的函数, 之后是一个类, 会产生一个类, 返回的原来的值, 故不会对常规函数产生影响

```python
from typing import NewType, NoReturn
 
# 创建int的子类, 名为UserId
UserIdType = NewType('UserId', int)
 
# 使用
def name_by_id(user_id: UserIdType) -> NoReturn:
    print(user_id)
 
 
# 类型检测错误
UserIdType('user')
 
# 类型检测成功, 返回原类型 "int"
num = UserIdType(5)
 
# 类型检测错误, 需要传入UserIdType, 而不是int
name_by_id(42)
# 类型检测成功
name_by_id(UserIdType(42))  # OK
```

## 元组 Tuple
> 使用场景  
> 有限元素: Tuple[X, Y], 即第一个元素类型为X, 第二个元素类型为Y  
> 空元组: Tuple[()]  
> 可用...指定任意个同类: Tuple[int, ...]即多个int元素的元组  
> 不指定时(Tuple)等价于Tuple[Any, ...]  

```python
from typing import Tuple
 
 
# 任意类型元组
def x(t: Tuple):
    pass
 
 
# 全为字符串的元组
def y(t: Tuple[str, ...]):
    pass
 
 
# 多个类型有 "..." 时为任意类型
def z(t: Tuple[str, int, ...]):
    pass
```

## 列表 List
> 使用场景同 `Tuple`

```python
from typing import List
 
 
# 任意类型
def x(t: List):
    pass
 
 
# 全为字符串
def y(t: List[str, ...]):
    pass
 
 
# 多个类型有 "..." 时为任意类型
def z(t: List[str, int, ...]):
    pass
```

## 字典 Dict
> 使用：Dict[key类型, val类型]

```python
from typing import Dict
 
 
# key为字符串 value为int
def x(t: Dict[str, int]):
    pass
```

## 集合 Set
> 使用：Set[元素类型]  
> 注: 集合中的元素类型必须一致, 最多只能有一个参数, 即只能有一种类型  
> 不能使用...

```python
from typing import Set
 
 
# 任意类型
def x(t: Set):
    pass
 
 
# 字符串
def y(t: Set[str]):
    pass
```

## 序列类型 Sequence
> 使用注意点与Set相同

```python
from typing import Union, Sequence
 
 
# int
def x(t: Sequence[int]):
    pass
```

## 可调用类型 Callable
> 即可调用对象  
> 定义形如: Callable[[Arg1Type, Arg2Type], ReturnType],
第一个参数是列表 值为可调用对象的参数类型, 第二个参数是返回值

```python
from typing import Callable
 
 
def demo(c: Callable[[int, str], str]):
    res = c(1, "a")
    # 1a <class 'str'>
    print(res, type(res))
 
 
def func(a: int, b: str) -> str:
    return str(a) + b
 
 
demo(func)
```

## 任意类型 Any
> 任意类型, 与不使用typing时的效果一致

```python
from typing import Any
 
 
# 任意类型
def x(a: Any, b: ...):
    pass
 
 
x(1, 2)
x("a", "b")
```

## 无返回值 NoReturn
> 标记没有返回值的函数, 与不使用typing时的效果一致

```python
from typing import NoReturn
 
def stop() -> NoReturn:
    raise RuntimeError('no way')
```

## 联合类型 Union
> 用于多种类型中的一种, 即必须是指定的类型   
> 联合中的联合会被展平: Union[Union[int, str], float] == Union[int, str, float]  
> 单参数即本身: Union[int] == int  
> 冗余参数跳过: Union[int, str, int] == Union[int, str] == int | str  
> 无顺序: Union[int, str] == Union[str, int]  

```python
from typing import Union
 
 
# str或int
def x(t: Union[str, int]):
    pass
```

- python3.10之后可以写成X | Y, 不需要额外导入Union  

## 可选类型 Optional
> 可以是指定的类型  
> 含默认值的可选参数不需要在类型注解上添加Optional限定符  
> 默认值为None时, 可以用Optional

```python
from typing import Optional
 
 
# t的类型可以是int
def x(t: Optional[int] = None):
    pass
```

- 注：`Optional[X]`等价于`Union[X, None]`，等价于 `X | None`