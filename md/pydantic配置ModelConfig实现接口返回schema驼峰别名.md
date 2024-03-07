# <center>pydantic配置ModelConfig实现接口返回schema驼峰别名

> 说明：

> Python变量、属性命名规范以蛇形命名法为主，而在 Java、C#、前端语言中，变量、属性命名规范以驼峰命名法为主。在Python Web实际开发中，前后端交互时，前端需要的数据格式一般是驼峰命名法，而后端返回的数据格式一般是蛇形命名法。因此，需要在后端返回数据时，将数据格式转换为驼峰命名法。

## 别名转换

- 使用 Pydantic 的 ModelConfig 类的 alias_generator 属性，可以实现接口返回 schema 驼峰别名。

    ```python
    from pydantic import BaseModel
    from pydantic.alias_generator import to_camel

    class User(BaseModel):
        id: str
        user_name: str = None
        user_age: int = None

        class Config:
            alias_generator = to_camel  # 指定别名生成器为驼峰命名法
    ```

- 配合使用`populated_by_name`属性，可以同时支持原本属性名以及驼峰命名。

    ```python
    from pydantic import BaseModel
    from pydantic.alias_generator import to_camel

    class SchemaBaseModel(BaseModel):

        class Config:
            alias_generator = to_camel  # 指定别名生成器为驼峰命名法
            populated_by_name = True    # 支持原本属性名以及驼峰命名
    ```

- 完成以上配置后，接口实际返回的数据格式依然是蛇形命名；原因是在处理接口返回数据时使用 pytdantic 的 `model_dump` 方法将 python 对象转换为 json 字符串时，by_alias属性默认为 False，需要指定参数`by_alias=True`，才能实现接口返回 schema 驼峰别名。

- 基于此，通过重写 pytdantic 的 `model_dump` 方法，修改默认的`by_alias=True`即可实现全局接口返回的 schema 为驼峰别名。

    ```python
    from pydantic import BaseModel
    from pydantic.alias_generator import to_camel
    from pydantic.main import ModelMetaclass

    class SchemaBaseModel(BaseModel):

        class Config:
            alias_generator = to_camel  # 指定别名生成器为驼峰命名法
            populated_by_name = True    # 支持原本属性名以及驼峰命名

        def model_dump(self, *, by_alias: bool = True, **kwargs) -> str:
            return super().model_dump(by_alias=by_alias, **kwargs)
    ```

- 最后，所有 schema 类继承 SchemaBaseModel，即可实现全局接口返回的 schema 为驼峰别名。

    ```python
    from pydantic import BaseModel
    from pydantic.alias_generator import to_camel

    class User(SchemaBaseModel):
        id: str
        user_name: str = None
        user_age: int = None
    ```