# <center>alembic通过配置env.py实现异步迁移

## 背景

> fastapi项目开发中使用的是 sqlalchemy 异步数据库 ORM 框架，alembic 是 sqlalchemy 的数据库迁移工具，alembic 本身不支持异步数据库迁移，但是可以通过修改 env.py 文件实现异步数据库迁移。

## 实现

### 1. env.py 文件中默认的 run_migrations_online 方法

```python
def run_migrations_online() -> None:
    """Run migrations in 'online' mode.

    In this scenario we need to create an Engine
    and associate a connection with the context.

    """
    connectable = engine_from_config(
        config.get_section(config.config_ini_section),
        prefix="sqlalchemy.",
        poolclass=pool.NullPool,
    )

    with connectable.connect() as connection:
        context.configure(
            connection=connection,
            target_metadata=target_metadata
        )

        with context.begin_transaction():
            context.run_migrations()
```

### 2. 修改 run_migrations_online 方法为异步实现

```python
from sqlalchemy import pool

import asyncio
from sqlalchemy.engine import Connection
from sqlalchemy.ext.asyncio import async_engine_from_config

async def run_migrations_online() -> None:
    """Run migrations in 'online' mode with async way.

    In this scenario we need proceed following steps:
    1. create an asynchronous Engine
    2. then associate an asynchronous connection with the context
    3. finally run migrations synchronized.

    """
    def do_migrations(conn: Connection) -> None:
        """do migrations in synchronized way."""
        context.configure(connection=conn, target_metadata=target_metadata)
        with context.begin_transaction():
            context.run_migrations()

    # step1: create an asynchronous Engine
    connectable = async_engine_from_config(
        config.get_section(config.config_ini_section, {}),
        prefix="sqlalchemy.",
        poolclass=pool.NullPool,
    )

    # step2: associate an asynchronous database connection with the asynchronous context
    async with connectable.connect() as connection:
        await connection.run_sync(do_migrations)    # step3: run migrations synchronized
    await connectable.dispose()

asyncio.run(run_migrations_online())    #at the last: run migrations in async mode!
```