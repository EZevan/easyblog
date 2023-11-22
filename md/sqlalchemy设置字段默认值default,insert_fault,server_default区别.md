# <center>sqlalchemy设置字段默认值 default/insert_fault/server_default 区别

## 背景

> fastapi项目开发中使用sqlalchemy来配置对象关系映射时，实体模型的字段默认值设置有三种方式，分别是default、insert_fault、server_default，这三种方式的区别是什么？

## 先说结论

- default：用于设置ORM模型在创建表时字段的默认值, 但是在操作ORM模型插入数据时不会使用该默认值
- insert_fault：用于设置ORM模型在插入数据时字段的默认值, 即操作ORM模型的insert方法时会使用该默认值
- server_default：用于设置数据库在创建表时字段的默认值, 即通过在 CREATE TABLE 语句中添加 DEFAULT 语句来设置默认值

## 举例说明
...