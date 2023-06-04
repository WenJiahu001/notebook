#RocketMq安装以后注意事项
```shell
export NAMESRV_ADDR=localhost:9876
./mqadmin clusterList -n 127.0.0.1:9876
./mqadmin updateTopic -c rocketmq-cluster -t TopicLicosEvent
tail -f ~/logs/rocketmqlogs/namesrv.log
```
```shell
# 1.设置环境变量
export NAMESRV_ADDR=localhost:9876
# 2.使用安装包的Demo发送消息
sh tools.sh org.apache.rocketmq.example.quickstart.Producer
# 3.接收消息
sh tools.sh org.apache.rocketmq.example.quickstart.Consumer
```