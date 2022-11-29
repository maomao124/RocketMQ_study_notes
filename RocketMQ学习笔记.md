





# MQ介绍

## 为什么要用MQ

消息队列是一种“先进先出”的数据结构



其应用场景主要包含以下3个方面



### 应用解耦

系统的耦合性越高，容错性就越低。以电商应用为例，用户创建订单后，如果耦合调用库存系统、物流系统、支付系统，任何一个子系统出了故障或者因为升级等原因暂时不可用，都会造成下单操作异常，影响用户使用体验。

使用消息队列解耦合，系统的耦合性就会提高了。比如物流系统发生故障，需要几分钟才能来修复，在这段时间内，物流系统要处理的数据被缓存到消息队列中，用户的下单操作正常完成。当物流系统回复后，补充处理存在消息队列中的订单消息即可，终端系统感知不到物流系统发生过几分钟故障。





### 流量削峰

应用系统如果遇到系统请求流量的瞬间猛增，有可能会将系统压垮。有了消息队列可以将大量请求缓存起来，分散到很长一段时间处理，这样可以大大提到系统的稳定性和用户体验。

一般情况，为了保证系统的稳定性，如果系统负载超过阈值，就会阻止用户请求，这会影响用户体验，而如果使用消息队列将请求缓存起来，等待系统处理完毕后通知用户下单完毕，这样总不能下单体验要好。

业务系统正常时段的QPS如果是1000，流量最高峰是10000，为了应对流量高峰配置高性能的服务器显然不划算，这时可以使用消息队列对峰值流量削峰





### 数据分发

通过消息队列可以让数据在多个系统更加之间进行流通。数据的产生方不需要关心谁来使用数据，只需要将数据发送到消息队列，数据使用方直接在消息队列中直接获取数据即可



![image-20221128184448193](img/RocketMQ学习笔记/image-20221128184448193.png)



![image-20221128184456189](img/RocketMQ学习笔记/image-20221128184456189.png)













## MQ的优点和缺点

优点：

* 应用解耦
* 流量削峰
* 数据分发



缺点：

* 系统可用性降低：系统引入的外部依赖越多，系统稳定性越差。一旦MQ宕机，就会对业务造成影响
* 系统复杂度提高：MQ的加入大大增加了系统的复杂度，以前系统间是同步的远程调用，现在是通过MQ进行异步调用
* 一致性问题：A系统处理完业务，通过MQ给B、C、D三个系统发消息数据，如果B系统、C系统处理成功，D系统处理失败













# RocketMQ快速入门

## 下载RocketMQ



[下载地址](https://www.apache.org/dyn/closer.cgi?path=rocketmq/4.5.1/rocketmq-all-4.5.1-bin-release.zip)



## 安装

解压版，需要解压到某一个文件夹里



```sh
PS H:\opensoft> ls


    目录: H:\opensoft


Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
d-----          2022/5/5     18:34                apache-jmeter-5.4.3
d-----          2022/6/2     13:25                cerebro-0.9.4
d-----         2022/5/26     20:37                elasticsearch-analysis-ik-8.2.0
d-----          2022/6/1     23:19                elasticsearch-cluster
d-----          2022/5/3     10:55                kibana-8.1.3
d-----         2022/5/30     23:02                logstash-8.1.3
d-----        2022/11/22     14:58                MongoDB
d-----         2022/7/20     12:25                mycat
d-----         2022/6/15     12:23                mycat-1.6
d-----         2022/7/19     15:35                nacos
d-----         2022/7/16     14:30                naocs-cluster
d-----         2022/7/16     21:38                nginx-1.21.6
d-----          2022/5/6     23:16                pvzpak-master.git
d-----        2022/11/29     21:21                rocketmq
d-----         2021/4/25     16:01                seata-server-1.4.2
d-----         2022/7/28     20:20                seata-server-cluster
d-----         2022/7/23     16:36                Sentinel
d-----         2022/8/11     20:14                浏览器主页


PS H:\opensoft> cd .\rocketmq\
PS H:\opensoft\rocketmq> ls


    目录: H:\opensoft\rocketmq


Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
d-----        2022/11/29     21:21                benchmark
d-----        2022/11/29     21:21                bin
d-----        2022/11/29     21:21                conf
d-----        2022/11/29     21:21                lib
-a----         2019/3/28     17:08          17336 LICENSE
-a----         2019/5/21     10:44           1337 NOTICE
-a----        2022/11/29     21:19           2523 README.md


PS H:\opensoft\rocketmq> cd .\bin\
PS H:\opensoft\rocketmq\bin> ls


    目录: H:\opensoft\rocketmq\bin


Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
d-----        2022/11/29     21:21                dledger
-a----         2019/3/28     17:08           1654 cachedog.sh
-a----         2019/3/28     17:08            845 cleancache.sh
-a----         2019/3/28     17:08           1116 cleancache.v1.sh
-a----         2019/3/28     17:08           1398 mqadmin
-a----         2019/3/28     17:08           1029 mqadmin.cmd
-a----         2019/3/28     17:08           1422 mqadmin.xml
-a----         2019/3/28     17:08           1394 mqbroker
-a----         2019/3/28     17:08           1084 mqbroker.cmd
-a----         2019/3/28     17:08           1373 mqbroker.numanode0
-a----         2019/3/28     17:08           1373 mqbroker.numanode1
-a----         2019/3/28     17:08           1373 mqbroker.numanode2
-a----         2019/3/28     17:08           1373 mqbroker.numanode3
-a----         2019/3/28     17:08           1388 mqbroker.xml
-a----         2019/3/28     17:08           1396 mqnamesrv
-a----         2019/3/28     17:08           1088 mqnamesrv.cmd
-a----         2019/3/28     17:08           1390 mqnamesrv.xml
-a----         2019/3/28     17:08           1571 mqshutdown
-a----         2019/3/28     17:08           1398 mqshutdown.cmd
-a----         2019/3/28     17:08           2222 os.sh
-a----         2019/3/28     17:08           1148 play.cmd
-a----         2019/3/28     17:08           1008 play.sh
-a----         2019/3/28     17:08            772 README.md
-a----         2019/3/28     17:08           2206 runbroker.cmd
-a----         2019/3/28     17:08           2849 runbroker.sh
-a----         2019/3/28     17:08           1816 runserver.cmd
-a----         2019/3/28     17:08           2443 runserver.sh
-a----         2019/3/28     17:08           1156 setcache.sh
-a----         2019/3/28     17:08           1408 startfsrv.sh
-a----         2019/3/28     17:08           1601 tools.cmd
-a----         2019/3/28     17:08           1870 tools.sh


PS H:\opensoft\rocketmq\bin>
```





## 目录介绍

* bin：启动脚本，包括shell脚本和CMD脚本
* conf：实例配置文件 ，包括broker配置文件、logback配置文件等
* lib：依赖jar包，包括Netty、commons-lang、FastJSON等





## 配置环境变量



![image-20221129212853486](img/RocketMQ学习笔记/image-20221129212853486.png)



![image-20221129212829736](img/RocketMQ学习笔记/image-20221129212829736.png)













## 启动RocketMQ

### 启动NameServer

```sh
PS H:\opensoft\rocketmq\bin> .\mqnamesrv
OpenJDK 64-Bit Server VM warning: Using the DefNew young collector with the CMS collector is deprecated and will likely be removed in a future release
OpenJDK 64-Bit Server VM warning: UseCMSCompactAtFullCollection is deprecated and will likely be removed in a future release.
The Name Server boot success. serializeType=JSON
```





### 启动Broker

配置环境变量

![image-20221129214853936](img/RocketMQ学习笔记/image-20221129214853936.png)



```sh
PS H:\opensoft\rocketmq\bin> .\mqbroker
The broker[mao, 172.27.80.1:10911] boot success. serializeType=JSON and name server is 127.0.0.1:9876
```







## 测试RocketMQ

### 发送消息





```sh
.\tools org.apache.rocketmq.example.quickstart.Producer
```



```sh
PS H:\opensoft\rocketmq\bin> .\tools org.apache.rocketmq.example.quickstart.Producer
OpenJDK 64-Bit Server VM warning: ignoring option PermSize=128m; support was removed in 8.0
OpenJDK 64-Bit Server VM warning: ignoring option MaxPermSize=128m; support was removed in 8.0
21:52:00.739 [main] DEBUG i.n.u.i.l.InternalLoggerFactory - Using SLF4J as the default logging framework
SendResult [sendStatus=SEND_OK, msgId=76FBC10A4DDC60E53B9394E342180000, offsetMsgId=AC1B500100002A9F0000000000000000, messageQueue=MessageQueue [topic=TopicTest, brokerName=mao, queueId=3], queueOffset=0]
SendResult [sendStatus=SEND_OK, msgId=76FBC10A4DDC60E53B9394E342380001, offsetMsgId=AC1B500100002A9F00000000000000B2, messageQueue=MessageQueue [topic=TopicTest, brokerName=mao, queueId=0], queueOffset=0]
SendResult [sendStatus=SEND_OK, msgId=76FBC10A4DDC60E53B9394E3423B0002, offsetMsgId=AC1B500100002A9F0000000000000164, messageQueue=MessageQueue [topic=TopicTest, brokerName=mao, queueId=1], queueOffset=0]
......
......
......
SendResult [sendStatus=SEND_OK, msgId=76FBC10A4DDC60E53B9394E3482103D6, offsetMsgId=AC1B500100002A9F000000000002B20A, messageQueue=MessageQueue [topic=TopicTest, brokerName=mao, queueId=1], queueOffset=245]
SendResult [sendStatus=SEND_OK, msgId=76FBC10A4DDC60E53B9394E3482203D7, offsetMsgId=AC1B500100002A9F000000000002B2BE, messageQueue=MessageQueue [topic=TopicTest, brokerName=mao, queueId=2], queueOffset=245]
SendResult [sendStatus=SEND_OK, msgId=76FBC10A4DDC60E53B9394E3482303D8, offsetMsgId=AC1B500100002A9F000000000002B372, messageQueue=MessageQueue [topic=TopicTest, brokerName=mao, queueId=3], queueOffset=246]
SendResult [sendStatus=SEND_OK, msgId=76FBC10A4DDC60E53B9394E3482303D9, offsetMsgId=AC1B500100002A9F000000000002B426, messageQueue=MessageQueue [topic=TopicTest, brokerName=mao, queueId=0], queueOffset=246]
SendResult [sendStatus=SEND_OK, msgId=76FBC10A4DDC60E53B9394E3482803DA, offsetMsgId=AC1B500100002A9F000000000002B4DA, messageQueue=MessageQueue [topic=TopicTest, brokerName=mao, queueId=1], queueOffset=246]
SendResult [sendStatus=SEND_OK, msgId=76FBC10A4DDC60E53B9394E3482903DB, offsetMsgId=AC1B500100002A9F000000000002B58E, messageQueue=MessageQueue [topic=TopicTest, brokerName=mao, queueId=2], queueOffset=246]
SendResult [sendStatus=SEND_OK, msgId=76FBC10A4DDC60E53B9394E3482A03DC, offsetMsgId=AC1B500100002A9F000000000002B642, messageQueue=MessageQueue [topic=TopicTest, brokerName=mao, queueId=3], queueOffset=247]
SendResult [sendStatus=SEND_OK, msgId=76FBC10A4DDC60E53B9394E3482B03DD, offsetMsgId=AC1B500100002A9F000000000002B6F6, messageQueue=MessageQueue [topic=TopicTest, brokerName=mao, queueId=0], queueOffset=247]
SendResult [sendStatus=SEND_OK, msgId=76FBC10A4DDC60E53B9394E3482C03DE, offsetMsgId=AC1B500100002A9F000000000002B7AA, messageQueue=MessageQueue [topic=TopicTest, brokerName=mao, queueId=1], queueOffset=247]
SendResult [sendStatus=SEND_OK, msgId=76FBC10A4DDC60E53B9394E3482C03DF, offsetMsgId=AC1B500100002A9F000000000002B85E, messageQueue=MessageQueue [topic=TopicTest, brokerName=mao, queueId=2], queueOffset=247]
SendResult [sendStatus=SEND_OK, msgId=76FBC10A4DDC60E53B9394E3482D03E0, offsetMsgId=AC1B500100002A9F000000000002B912, messageQueue=MessageQueue [topic=TopicTest, brokerName=mao, queueId=3], queueOffset=248]
SendResult [sendStatus=SEND_OK, msgId=76FBC10A4DDC60E53B9394E3482E03E1, offsetMsgId=AC1B500100002A9F000000000002B9C6, messageQueue=MessageQueue [topic=TopicTest, brokerName=mao, queueId=0], queueOffset=248]
SendResult [sendStatus=SEND_OK, msgId=76FBC10A4DDC60E53B9394E3482F03E2, offsetMsgId=AC1B500100002A9F000000000002BA7A, messageQueue=MessageQueue [topic=TopicTest, brokerName=mao, queueId=1], queueOffset=248]
SendResult [sendStatus=SEND_OK, msgId=76FBC10A4DDC60E53B9394E3483003E3, offsetMsgId=AC1B500100002A9F000000000002BB2E, messageQueue=MessageQueue [topic=TopicTest, brokerName=mao, queueId=2], queueOffset=248]
SendResult [sendStatus=SEND_OK, msgId=76FBC10A4DDC60E53B9394E3483103E4, offsetMsgId=AC1B500100002A9F000000000002BBE2, messageQueue=MessageQueue [topic=TopicTest, brokerName=mao, queueId=3], queueOffset=249]
SendResult [sendStatus=SEND_OK, msgId=76FBC10A4DDC60E53B9394E3483203E5, offsetMsgId=AC1B500100002A9F000000000002BC96, messageQueue=MessageQueue [topic=TopicTest, brokerName=mao, queueId=0], queueOffset=249]
SendResult [sendStatus=SEND_OK, msgId=76FBC10A4DDC60E53B9394E3483203E6, offsetMsgId=AC1B500100002A9F000000000002BD4A, messageQueue=MessageQueue [topic=TopicTest, brokerName=mao, queueId=1], queueOffset=249]
SendResult [sendStatus=SEND_OK, msgId=76FBC10A4DDC60E53B9394E3483303E7, offsetMsgId=AC1B500100002A9F000000000002BDFE, messageQueue=MessageQueue [topic=TopicTest, brokerName=mao, queueId=2], queueOffset=249]
21:52:03.132 [NettyClientSelector_1] INFO  RocketmqRemoting - closeChannel: close the connection to remote address[127.0.0.1:9876] result: true
21:52:03.133 [NettyClientSelector_1] INFO  RocketmqRemoting - closeChannel: close the connection to remote address[172.27.80.1:10911] result: true
PS H:\opensoft\rocketmq\bin>
```







### 接收消息

```sh
.\tools org.apache.rocketmq.example.quickstart.Consumer
```



```sh
......
......
......
ConsumeMessageThread_13 Receive New Messages: [MessageExt [queueId=0, storeSize=180, queueOffset=113, sysFlag=0, bornTimestamp=1669729922426, bornHost=/172.27.80.1:61263, storeTimestamp=1669729922426, storeHost=/172.27.80.1:10911, msgId=AC1B500100002A9F0000000000013E16, commitLogOffset=81430, bodyCRC=1655802458, reconsumeTimes=0, preparedTransactionOffset=0, toString()=Message{topic='TopicTest', flag=0, properties={MIN_OFFSET=0, MAX_OFFSET=250, CONSUME_START_TIME=1669730050849, UNIQ_KEY=76FBC10A4DDC60E53B9394E3457A01C5, WAIT=true, TAGS=TagA}, body=[72, 101, 108, 108, 111, 32, 82, 111, 99, 107, 101, 116, 77, 81, 32, 52, 53, 51], transactionId='null'}]]
ConsumeMessageThread_10 Receive New Messages: [MessageExt [queueId=0, storeSize=180, queueOffset=112, sysFlag=0, bornTimestamp=1669729922418, bornHost=/172.27.80.1:61263, storeTimestamp=1669729922418, storeHost=/172.27.80.1:10911, msgId=AC1B500100002A9F0000000000013B46, commitLogOffset=80710, bodyCRC=461328901, reconsumeTimes=0, preparedTransactionOffset=0, toString()=Message{topic='TopicTest', flag=0, properties={MIN_OFFSET=0, MAX_OFFSET=250, CONSUME_START_TIME=1669730050849, UNIQ_KEY=76FBC10A4DDC60E53B9394E3457201C1, WAIT=true, TAGS=TagA}, body=[72, 101, 108, 108, 111, 32, 82, 111, 99, 107, 101, 116, 77, 81, 32, 52, 52, 57], transactionId='null'}]]
ConsumeMessageThread_5 Receive New Messages: [MessageExt [queueId=0, storeSize=180, queueOffset=111, sysFlag=0, bornTimestamp=1669729922412, bornHost=/172.27.80.1:61263, storeTimestamp=1669729922412, storeHost=/172.27.80.1:10911, msgId=AC1B500100002A9F0000000000013876, commitLogOffset=79990, bodyCRC=315170350, reconsumeTimes=0, preparedTransactionOffset=0, toString()=Message{topic='TopicTest', flag=0, properties={MIN_OFFSET=0, MAX_OFFSET=250, CONSUME_START_TIME=1669730050848, UNIQ_KEY=76FBC10A4DDC60E53B9394E3456C01BD, WAIT=true, TAGS=TagA}, body=[72, 101, 108, 108, 111, 32, 82, 111, 99, 107, 101, 116, 77, 81, 32, 52, 52, 53], transactionId='null'}]]
ConsumeMessageThread_2 Receive New Messages: [MessageExt [queueId=0, storeSize=180, queueOffset=110, sysFlag=0, bornTimestamp=1669729922404, bornHost=/172.27.80.1:61263, storeTimestamp=1669729922405, storeHost=/172.27.80.1:10911, msgId=AC1B500100002A9F00000000000135A6, commitLogOffset=79270, bodyCRC=363125303, reconsumeTimes=0, preparedTransactionOffset=0, toString()=Message{topic='TopicTest', flag=0, properties={MIN_OFFSET=0, MAX_OFFSET=250, CONSUME_START_TIME=1669730050848, UNIQ_KEY=76FBC10A4DDC60E53B9394E3456401B9, WAIT=true, TAGS=TagA}, body=[72, 101, 108, 108, 111, 32, 82, 111, 99, 107, 101, 116, 77, 81, 32, 52, 52, 49], transactionId='null'}]]
ConsumeMessageThread_9 Receive New Messages: [MessageExt [queueId=0, storeSize=180, queueOffset=109, sysFlag=0, bornTimestamp=1669729922399, bornHost=/172.27.80.1:61263, storeTimestamp=1669729922399, storeHost=/172.27.80.1:10911, msgId=AC1B500100002A9F00000000000132D6, commitLogOffset=78550, bodyCRC=864479685, reconsumeTimes=0, preparedTransactionOffset=0, toString()=Message{topic='TopicTest', flag=0, properties={MIN_OFFSET=0, MAX_OFFSET=250, CONSUME_START_TIME=1669730050848, UNIQ_KEY=76FBC10A4DDC60E53B9394E3455F01B5, WAIT=true, TAGS=TagA}, body=[72, 101, 108, 108, 111, 32, 82, 111, 99, 107, 101, 116, 77, 81, 32, 52, 51, 55], transactionId='null'}]]
......
......
......
```







## 关闭RocketMQ

```sh
.\mqshutdown namesrv
```

```sh
.\mqshutdown broker
```



```sh
PS H:\opensoft\rocketmq\bin> .\mqshutdown namesrv
killing name server
成功: 已终止 PID 为 9444 的进程。
Done!
PS H:\opensoft\rocketmq\bin> .\mqshutdown broker
killing broker
成功: 已终止 PID 为 21088 的进程。
Done!
PS H:\opensoft\rocketmq\bin>
```



















# RocketMQ集群搭建

## 各角色介绍

* Producer：消息的发送者；举例：发信者
* Consumer：消息接收者；举例：收信者
* Broker：暂存和传输消息；举例：邮局
* NameServer：管理Broker；举例：各个邮局的管理机构
* Topic：区分消息的种类；一个发送者可以发送消息给一个或者多个Topic；一个消息的接收者可以订阅一个或者多个Topic消息
* Message Queue：相当于是Topic的分区；用于并行发送和接收消息





![image-20221129220034773](img/RocketMQ学习笔记/image-20221129220034773.png)









## 集群搭建方式

### 集群特点
