# <center>Python-typing中泛型的使用

> 背景：在使用 fastapi 构建 web 应用服务的时候，使用repository模式过程中， 借助 typing 的泛型类型可以很优雅的定义`BaseRepository`，以便于减少重复代码。

## 泛型

- 泛型, 就是将数据类型作为参数进行传递, 即在我们用的时候确定数据类型, 这是一种在面向对象语言中经常使用的特性, 例如 Java 中的泛型, C# 中的泛型, C++ 中的模板等等。

## Python中的泛型

- Python 中的泛型是通过 typing 模块来实现的, 该模块提供了很多泛型类型, 例如 TypeVar, Type, List, Dict, Tuple, Set, Callable 等等, 这些泛型类型有的可以用于函数的参数类型注解, 有的也可以用于类的属性类型注解

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

## 泛型类型
### TypeVar
- TypeVar 是一个泛型类型变量, 用于表示泛型类型与泛型函数定义的参数, 例如下面的例子中, T1 和 T2 都是 TypeVar 类型, 用于表示泛型类型, 但是 T1 和 T2 表示的类型是不同的, T1 表示的是 int 或 float类型, T2 表示的是 str 或 bytes类型, 通过这种方式, 我们可以在函数中使用泛型类型变量, 从而实现函数的泛型

    ```python
    from typing import TypeVar

    T1 = TypeVar('T1', int, float)
    T2 = TypeVar('T2', str, bytes)

    def demo(t1: T1, t2: T2) -> T2:
        return t2
    ```

- TypeVar 类型的参数说明
    - 第一个参数是变量名, 用于表示泛型类型变量
    - 第二个参数是可选的, 用于表示泛型类型变量的上界, 即泛型类型变量的类型必须是上界类型或者其子类, 如果没有指定上界, 则默认为 Any 类型

```python
T1 = TypeVar('T1', int, float)  # T1 表示的类型必须是 int 或 float 类型

T2 = TypeVar("T2", bound=T1)    # T2 表示的类型必须是 T1 类型或其子类
```

### Type
- Type 是一个泛型类型, 用于表示类型, 例如下面的例子中, T1 和 T2 都是 Type 类型, 用于表示类型, 但是 T1 和 T2 表示的类型是不同的, T1 表示的是 int 类型, T2 表示的是 str 类型, 通过这种方式, 我们可以在函数中使用泛型类型, 从而实现函数的泛型

    ```python
    from typing import Type

    T1 = Type[int]
    T2 = Type[str]

    def demo(t1: T1, t2: T2) -> T2:
        return t2
    ```

- Type[C]的形式为协变量, 表明C的所有子类都应使用与C相同的构造器签名 及类方法签名, 假如我们需要返回泛型类型的话, 需要用到

```python
from typing import Type

ModelType = TypeVar("ModelType", bound=BaseModel)

def demo(model: type[ModelType]) -> Type[ModelType]:
    return model
```