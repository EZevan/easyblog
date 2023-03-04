# Kafka查看 topic消费情况

1. 查询消费数据，必须要指定组
```c
cd /opt/kafka/bin/

# 查找topic组
./kafka-consumer-groups.sh --bootstrap-server localhost:9092 --list

# 查看etrial组
./kafka-consumer-groups.sh --describe --bootstrap-server localhost:9092 --group etrial
```
2. 查询消费情况

|TOPIC|PARTITION|CURRENT-OFFSET|LOG-END-OFFSET|LAG|CONSUMER-ID|HOST|CLIENT-ID|
|----|----|----|----|----|----|----|----|
|topic名字|分区id|当前已消费的条数|总条数|未消费的条数|消费id|主机ip|客户端id|