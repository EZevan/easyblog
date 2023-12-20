# <center>sqlalchemy实体模型主键id使用uuid 作为默认值只生成一次

## 背景
> salalchemy 实体模型的主键 id 使用 uuid 作为默认值，fastapi 接口调用 orm 框架创建实体对象时，第二次以后的对象始终用的是第一次生成的 uuid 值，导致数据库写入异常，程序报错。

## 排错过程


```
打印日志如下：
sqlalchemy.engine.Engine [cached since 33.17s ago] ('850b0eca9f1611eebb93acbc329b8455',...)
```

1. 一开始从日志入手发现 sqlalchemy 会缓存上次生成的 uuid；经过一番搜索，各种 gpt，发现诸如禁用 sqlalchemy 缓存、seesion().close()、session().expire_all() 等方法都无效。 - -| 浪费了好多时间

2. 发现始终没有方案，突然想到主键 id 是通过在定义模型实体的时候用的是`uuid1().hex`生成默认值，代码如下:
    ```python
    class Project(BaseModel):
        id: Mapped[str] = mapped_column(
            String(50), primary_key=True, insert_default=uuid1().hex, doc="id: primary key", comment="id: primary key")
    ```

3.故而开始从 uuid 生成默认值开始查，最终从[stackoverflow](https://stackoverflow.com/questions/74014636/sqlalchemy-inserts-the-previous-default-value-into-the-next-one) 找到答案

> When SQLAlchemy sets up an ORM class it looks at the column default= entries. If they are a scalar value or a function call that returns a scalar value then they are evaluated once and that value is used for each new instance of the class. However, if the default= is a callable then it is called for each new instance.

- 也就是说，当 sqlalchemy 设置 orm 类时，它会查看列默认值。如果它们是标量值或返回标量值的函数调用，则只会对其进行一次评估，并将该值用于类的每个新实例。但是，如果默认值是可调用的，则会为每个新实例调用它。

- 而在我的例子种，uuid().hex会直接返回一个标量值（str 类型的值），所以只会被生成一次，并且将这个值用到每个新实例中

## 解决方案

- 按照上面提到的，只需要将主键 id 的默认值使用可调用对象即可，最简单的是使用 lambda 表达式，代码如下：

    ```python
    class Project(BaseModel):
        id: Mapped[str] = mapped_column(
            String(50), primary_key=True, insert_default=lambda: uuid1().hex, doc="id: primary key", comment="id: primary key")
    ```

- 重新运行程序，问题解决