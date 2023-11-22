# <center>sqlalchemy配置ORM模型基类

## 思考

- sqlalchemy 在配置ORM模型时，表名、通用字段名在每个模型中都需要重复定义，是否可以将这些通用字段定义在一个基类中，然后其他模型继承该基类，这样就可以减少代码量，提高开发效率

## 定义模型基类

```python
import re

from sqlalchemy.ext.declarative import declared_attr
from sqlalchemy import String, DateTime, func, Boolean, Integer, text
from sqlalchemy.orm import Mapped, mapped_column
from sqlalchemy_easy_softdelete.mixin import generate_soft_delete_mixin_class

from uuid import uuid1
from datetime import datetime

from app.db.session import Base  # 也可以换成直接从sqlalchemy.ext.declarative import declarative_base 导入Base


class BaseModel(Base, generate_soft_delete_mixin_class()):
    """
    database model base class that will contain
    such as table name, id, created_at, updated_at, is_deleted, etc.
    """
    # Generate __tablename__ automatically
    @declared_attr  # mark the class method as a mapping attribute or special declarative member.
    def __tablename__(cls):
        # convert class name to snake case
        class_name = cls.__name__
        class_name_snake = re.sub(r'(?!^)([A-Z]+)', r'_\1', class_name).lower()
        return f"t_{class_name_snake}"

    # only map subclass to individual table and leave the base model unmapped with "__abstract__" equal to True.
    __abstract__ = True

    id: Mapped[str] = mapped_column(
        String(50), primary_key=True, insert_default=uuid1().hex, doc="id: primary key", comment="id: primary key")
    version: Mapped[int] = mapped_column(
        Integer, nullable=False, insert_default=0, server_default=text("0"), doc="version: optimistic lock version", comment="version: optimistic lock version")
    created_at: Mapped[datetime] = mapped_column(
        DateTime, nullable=True, insert_default=func.now(), doc="created time", comment="created time")
    created_by: Mapped[str] = mapped_column(
        String(50), nullable=True, doc="created by", comment="created by")
    updated_at: Mapped[datetime] = mapped_column(
        DateTime, nullable=True, insert_default=func.now(), onupdate=func.now(), doc="updated time", comment="updated time")
    updated_by: Mapped[str] = mapped_column(
        String(50), nullable=True, doc="updated by", comment="updated by")
    is_deleted: Mapped[bool] = mapped_column(
        Boolean, nullable=False, insert_default=False, server_default=text("False"), doc="is deleted or not", comment="is deleted or not")

    __mapper_args__ = {"version_id_col": version}

```

## 定义__table__属性

    ```python
    @declared_attr  # mark the class method as a mapping attribute or special declarative member.
        def __tablename__(cls):
            # convert class name to snake case
            class_name = cls.__name__
            class_name_snake = re.sub(r'(?!^)([A-Z]+)', r'_\1', class_name).lower()
            return f"t_{class_name_snake}"
    ```

- 通过`@declared_attr`装饰器，将__tablename__类方法标记为ORM映射属性__tablename__，这样子类中就可以直接继承该属性作为表名，而不需要在子类中重复定义__tablename__属性

- `re.sub(r'(?!^)([A-Z]+)', r'_\1', class_name).lower()`将驼峰命令的ORM模型类名转换为蛇形命名的表名

## 定义__abstract__属性

    ```python
    __abstract__ = True
    ```

- 通过`__abstract__ = True`将基类设置为抽象类，这样子类继承基类的属性后，仅将对应属性映射到子类及其子类的数据库表中，而基类不会映射到数据库表中

## 配置version字段乐观锁

    ```python
    __mapper_args__ = {"version_id_col": version}
    ```

- 通过`__mapper_args__ = {"version_id_col": version}`配置version字段为乐观锁，这样在更新数据时，如果version字段的值与数据库中的值不一致，则会抛出`sqlalchemy.orm.exc.StaleDataError`异常，从而避免脏数据的更新，保证数据的一致性，具体可参考[sqlalchemy官方文档](https://docs.sqlalchemy.org/en/20/orm/versioning.html)

## 配置软删除

```python
from sqlalchemy_easy_softdelete.mixin import generate_soft_delete_mixin_class

class BaseModel(Base, generate_soft_delete_mixin_class()):
    ...
    is_deleted: Mapped[bool] = mapped_column(
        Boolean, nullable=False, insert_default=False, server_default=text("False"), doc="is deleted or not", comment="is deleted or not")
```

- 通过模型基类继承`generate_soft_delete_mixin_class()`生成软删除的基类，然后在模型基类中定义is_deleted字段，这样子类继承模型基类后，就可以使用软删除功能了，具体可参考[sqlalchemy-easy-softdelete](https://pypi.org/project/sqlalchemy-easy-softdelete/)