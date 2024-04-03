# <center>sqlalchemy连接数据库探活配置

> python web 异步编程中，在使用`sqlalchemy`作为数据库ORM框架时，配合`aiomysql`作为 MySQL 异步数据库引擎；如果没有对数据库连接池配置管理，在 mysql 连接默认8小时超时后，我们在访问数据库的时候会失去连接，必须重试才能再次与数据库建立连接，这在客户端实际使用中很不友好

## 探活实现

1. `sqlalchemy`同步引擎`create_engine`中通过下面两个参数配合使用即可实现探活
    
    - `pool_recycle`
    - `pool_pre_ping`

    ```py
    engin = create_engine(
        mysql_connection, 
        pool_recycle=3600,  # 表示1小时后数据库连接会被回收，及超时断开，以节省不必要的开销
        pool_pre_ping=True  # 表示启用连接池「pre-ping」特性，以测试连接是否处于存活状态，即探活
        )
    ```

2. 对于`sqlalchemy`异步引擎`create_ async_engine`在处理探活时，主要是通过异步连接池管理类`AsyncAdaptedQueuePool`配合`pool_recycle`实现

 - 根据官方介绍，使用`create_async_engine`生成异步引擎时，会默认使用连接池管理类 - `AsyncAdaptedQueuePool`，所以在配置的时候不需要显示配置，即

    ```py
    engine = create_async_engine(
        aiomysql_connection,
        pool_recycle=3600
        # poolclass=AsyncAdaptedQueuePool  无需显示配置 poolclass 参数
    )
    ```

- 关于`AsyncAdaptedQueuePool`，做一下补充
    - AsyncAdaptedQueuePool 是SQLAlchemy中的异步连接池实现，用于管理异步数据库连接的池化。它是SQLAlchemy 1.4版本中引入的新特性，专门为异步操作提供支持。
    - AsyncAdaptedQueuePool继承自同步连接池管理类`QuenePool`，AsyncAdaptedQueuePool的参数和操作在其他方面与QueuePool相同。
    - 异步操作与传统的同步操作不同，需要使用异步连接池来管理数据库连接，以确保在异步环境中能够高效地处理数据库连接的获取和释放。
    - 主要功能体现在以下
        1. 异步连接池管理：允许在异步操作中使用池化的数据库连接，以提高性能和效率。 
        2. 异步连接获取：能够异步地获取数据库连接，避免在异步环境中出现阻塞。 
        3. 异步连接释放：能够异步地释放数据库连接，确保在异步操作完成后正确释放资源

- 基于上述特性，`create_async_engine`创建的引擎在数据库连接超时（1小时后被连接池连接被回收）后，`AsyncAdaptedQueuePool`会重新创建一个新的线程保持住数据库连接，以实现`数据库连接探活`