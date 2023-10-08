# <center>Python 理解可变位置参数中的"*"的作用

> 背景：在使用异步`asyncio.create_subprocess_exe()`方法执行外部命令的时候，其中`*args`作为可变的位置参数只能接收 str，bytes 类型；而构造准备的`sys.argv`是一个 `list`类型，所以需要将其转换为`*args`可接收的`str`类型。

* `*`的作用是将可迭代对象拆分为单个元素，即"解包"；如下示例中，它将`list`类型的`b`拆分为单个元素，作为位置参数传递给`print()`方法，`print(*b)` 等效于 `print("a","b","c")`。

```py
>>> a = [1, 2, 3]
>>> print(*a)
1 2 3

>>> b = ["a", "b", "c"]
>>> print(*b)
a b c
```

