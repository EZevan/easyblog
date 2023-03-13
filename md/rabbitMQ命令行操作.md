# <center>rabbitMQ 命令行队列操作
1. 创建队列

    ```
    # 创建名为file_es_add的队列
    rabbitmqctl eval 'rabbit_amqqueue:declare({resource, <<"/">>, queue, <<"file_es_add">>}, true, false, [], none).'
    ```

2. 创建 exchange

    ```
    # 创建名为fs_file_es的 exchange, 其中类型为topic
    rabbitmqctl eval 'rabbit_exchange:declare({resource, <<"/">>, exchange, <<"fs_file_es">>}, topic, true, false, false, []).'
    ```

3. 绑定队列

    ```
    # 绑定队列file_es_delete到fs_file_es上，指定了 routingKey为 delete
    rabbitmqctl eval 'rabbit_binding:add({binding, {resource, <<"/">>, exchange, <<"fs_file_es">>}, <<"delete">>, {resource, <<"/">>, queue, <<"file_es_delete">>}, []}).'
    ```

4. 查看队列列表

    ```
    # 结果中第一列为队列名称，第二列为当前队列存在未消费的消息数量
    rabbitmqctl list_queues
    ```

5. 查看 exchange 列表

    ```
    # 结果中第一列为 exchange 名称，第二列为当前 exchange 类型
    rabbitmqctl list_exchanges
    ```

6. 查看绑定列表

    ```
    # 结果中依次显示为：exchange名称 exchange queue名称 queue routingKey名称
    rabbitmqctl list_bindings
    ```

7. 清除指定队列消息

    ```
    rabbitmqctl purge_queue 指定队列名称
    ```