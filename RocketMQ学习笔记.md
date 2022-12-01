





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

- NameServer是一个几乎无状态节点，可集群部署，节点之间无任何信息同步。
- Broker部署相对复杂，Broker分为Master与Slave，一个Master可以对应多个Slave，但是一个Slave只能对应一个Master，Master与Slave的对应关系通过指定相同的BrokerName，不同的BrokerId来定义，BrokerId为0表示Master，非0表示Slave。Master也可以部署多个。每个Broker与NameServer集群中的所有节点建立长连接，定时注册Topic信息到所有NameServer。
- Producer与NameServer集群中的其中一个节点（随机选择）建立长连接，定期从NameServer取Topic路由信息，并向提供Topic服务的Master建立长连接，且定时向Master发送心跳。Producer完全无状态，可集群部署。
- Consumer与NameServer集群中的其中一个节点（随机选择）建立长连接，定期从NameServer取Topic路由信息，并向提供Topic服务的Master、Slave建立长连接，且定时向Master、Slave发送心跳。Consumer既可以从Master订阅消息，也可以从Slave订阅消息，订阅规则由Broker配置决定。







### 集群模式

#### 单Master模式

这种方式风险较大，一旦Broker重启或者宕机时，会导致整个服务不可用。不建议线上环境使用,可以用于本地测试



#### 多Master模式

一个集群无Slave，全是Master，例如2个Master或者3个Master，这种模式的优缺点如下：

- 优点：配置简单，单个Master宕机或重启维护对应用无影响，在磁盘配置为RAID10时，即使机器宕机不可恢复情况下，由于RAID10磁盘非常可靠，消息也不会丢（异步刷盘丢失少量消息，同步刷盘一条不丢），性能最高；
- 缺点：单台机器宕机期间，这台机器上未被消费的消息在机器恢复之前不可订阅，消息实时性会受到影响。





#### 多Master多Slave模式（异步）

每个Master配置一个Slave，有多对Master-Slave，HA采用异步复制方式，主备有短暂消息延迟（毫秒级），这种模式的优缺点如下：

- 优点：即使磁盘损坏，消息丢失的非常少，且消息实时性不会受影响，同时Master宕机后，消费者仍然可以从Slave消费，而且此过程对应用透明，不需要人工干预，性能同多Master模式几乎一样；
- 缺点：Master宕机，磁盘损坏情况下会丢失少量消息。





#### 多Master多Slave模式（同步）

每个Master配置一个Slave，有多对Master-Slave，HA采用同步双写方式，即只有主备都写成功，才向应用返回成功，这种模式的优缺点如下：

- 优点：数据与服务都无单点故障，Master宕机情况下，消息无延迟，服务可用性与数据可用性都非常高；
- 缺点：性能比异步复制模式略低（大约低10%左右），发送单个消息的RT会略高，且目前版本在主节点宕机后，备机不能自动切换为主机。







## 双主双从集群搭建

### 集群工作流程

1. 启动NameServer，NameServer起来后监听端口，等待Broker、Producer、Consumer连上来，相当于一个路由控制中心。
2. Broker启动，跟所有的NameServer保持长连接，定时发送心跳包。心跳包中包含当前Broker信息(IP+端口等)以及存储所有Topic信息。注册成功后，NameServer集群中就有Topic跟Broker的映射关系。
3. 收发消息前，先创建Topic，创建Topic时需要指定该Topic要存储在哪些Broker上，也可以在发送消息时自动创建Topic。
4. Producer发送消息，启动时先跟NameServer集群中的其中一台建立长连接，并从NameServer中获取当前发送的Topic存在哪些Broker上，轮询从队列列表中选择一个队列，然后与队列所在的Broker建立长连接从而向Broker发消息。
5. Consumer跟Producer类似，跟其中一台NameServer建立长连接，获取当前订阅Topic存在哪些Broker上，然后直接跟Broker建立连接通道，开始消费消息。







### 创建消息存储路径



```sh
mkdir ./data
mkdir ./data/master1/commitlog
mkdir ./data/master1/consumequeue
mkdir ./data/master1/index
mkdir ./data/master2/commitlog
mkdir ./data/master2/consumequeue
mkdir ./data/master2/index
mkdir ./data/slave1/commitlog
mkdir ./data/slave1/consumequeue
mkdir ./data/slave1/index
mkdir ./data/slave2/commitlog
mkdir ./data/slave2/consumequeue
mkdir ./data/slave2/index
```



```sh
PS H:\opensoft\rocketmq> mkdir ./data


    目录: H:\opensoft\rocketmq


Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
d-----         2022/12/1     20:42                data


PS H:\opensoft\rocketmq> mkdir ./data/master1/commitlog


    目录: H:\opensoft\rocketmq\data\master1


Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
d-----         2022/12/1     20:42                commitlog


PS H:\opensoft\rocketmq> mkdir ./data/master1/consumequeue


    目录: H:\opensoft\rocketmq\data\master1


Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
d-----         2022/12/1     20:42                consumequeue


PS H:\opensoft\rocketmq> mkdir ./data/master1/index


    目录: H:\opensoft\rocketmq\data\master1


Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
d-----         2022/12/1     20:42                index


PS H:\opensoft\rocketmq> mkdir ./data/master2/commitlog


    目录: H:\opensoft\rocketmq\data\master2


Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
d-----         2022/12/1     20:42                commitlog


PS H:\opensoft\rocketmq> mkdir ./data/master2/consumequeue


    目录: H:\opensoft\rocketmq\data\master2


Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
d-----         2022/12/1     20:42                consumequeue


PS H:\opensoft\rocketmq> mkdir ./data/master2/index


    目录: H:\opensoft\rocketmq\data\master2


Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
d-----         2022/12/1     20:42                index


PS H:\opensoft\rocketmq> mkdir ./data/slave1/commitlog


    目录: H:\opensoft\rocketmq\data\slave1


Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
d-----         2022/12/1     20:42                commitlog


PS H:\opensoft\rocketmq> mkdir ./data/slave1/consumequeue


    目录: H:\opensoft\rocketmq\data\slave1


Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
d-----         2022/12/1     20:42                consumequeue


PS H:\opensoft\rocketmq> mkdir ./data/slave1/index


    目录: H:\opensoft\rocketmq\data\slave1


Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
d-----         2022/12/1     20:42                index


PS H:\opensoft\rocketmq> mkdir ./data/slave2/commitlog


    目录: H:\opensoft\rocketmq\data\slave2


Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
d-----         2022/12/1     20:42                commitlog


PS H:\opensoft\rocketmq> mkdir ./data/slave2/consumequeue


    目录: H:\opensoft\rocketmq\data\slave2


Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
d-----         2022/12/1     20:42                consumequeue


PS H:\opensoft\rocketmq> mkdir ./data/slave2/index


    目录: H:\opensoft\rocketmq\data\slave2


Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
d-----         2022/12/1     20:42                index


PS H:\opensoft\rocketmq>
```

```sh
PS H:\opensoft\rocketmq> ls


    目录: H:\opensoft\rocketmq


Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
d-----        2022/11/29     21:21                benchmark
d-----        2022/11/29     21:21                bin
d-----        2022/11/29     21:21                conf
d-----         2022/12/1     20:42                data
d-----        2022/11/29     21:21                lib
-a----         2019/3/28     17:08          17336 LICENSE
-a----         2019/5/21     10:44           1337 NOTICE
-a----        2022/11/29     21:19           2523 README.md


PS H:\opensoft\rocketmq> cd .\data\
PS H:\opensoft\rocketmq\data> ls


    目录: H:\opensoft\rocketmq\data


Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
d-----         2022/12/1     20:42                master1
d-----         2022/12/1     20:42                master2
d-----         2022/12/1     20:42                slave1
d-----         2022/12/1     20:42                slave2


PS H:\opensoft\rocketmq\data> cd .\master1\
PS H:\opensoft\rocketmq\data\master1> ls


    目录: H:\opensoft\rocketmq\data\master1


Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
d-----         2022/12/1     20:42                commitlog
d-----         2022/12/1     20:42                consumequeue
d-----         2022/12/1     20:42                index


PS H:\opensoft\rocketmq\data\master1>
```







### broker配置文件

配置文件使用的是软件目录下\conf\2m-2s-sync的配置文件



```sh
PS H:\opensoft\rocketmq> ls


    目录: H:\opensoft\rocketmq


Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
d-----        2022/11/29     21:21                benchmark
d-----        2022/11/29     21:21                bin
d-----        2022/11/29     21:21                conf
d-----        2022/11/30     18:37                data
d-----        2022/11/29     21:21                lib
-a----         2019/3/28     17:08          17336 LICENSE
-a----         2019/5/21     10:44           1337 NOTICE
-a----        2022/11/29     21:19           2523 README.md


PS H:\opensoft\rocketmq> cd conf
PS H:\opensoft\rocketmq\conf> ls


    目录: H:\opensoft\rocketmq\conf


Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
d-----        2022/11/29     21:21                2m-2s-async
d-----        2022/11/29     21:21                2m-2s-sync
d-----        2022/11/29     21:21                2m-noslave
d-----        2022/11/29     21:21                dledger
-a----         2019/3/28     17:08            949 broker.conf
-a----         2019/3/28     17:08          14978 logback_broker.xml
-a----         2019/3/28     17:08           3836 logback_namesrv.xml
-a----         2019/3/28     17:08           3761 logback_tools.xml
-a----         2019/5/21     10:44           1305 plain_acl.yml
-a----         2019/5/21     10:44            834 tools.yml


PS H:\opensoft\rocketmq\conf> cd .\2m-2s-sync\
PS H:\opensoft\rocketmq\conf\2m-2s-sync> ls


    目录: H:\opensoft\rocketmq\conf\2m-2s-sync


Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
-a----         2019/3/28     17:08            922 broker-a-s.properties
-a----         2019/3/28     17:08            928 broker-a.properties
-a----         2019/3/28     17:08            922 broker-b-s.properties
-a----         2019/3/28     17:08            928 broker-b.properties


PS H:\opensoft\rocketmq\conf\2m-2s-sync> cat .\broker-a.properties
# Licensed to the Apache Software Foundation (ASF) under one or more
# contributor license agreements.  See the NOTICE file distributed with
# this work for additional information regarding copyright ownership.
# The ASF licenses this file to You under the Apache License, Version 2.0
# (the "License"); you may not use this file except in compliance with
# the License.  You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
brokerClusterName=DefaultCluster
brokerName=broker-a
brokerId=0
deleteWhen=04
fileReservedTime=48
brokerRole=SYNC_MASTER
flushDiskType=ASYNC_FLUSH
PS H:\opensoft\rocketmq\conf\2m-2s-sync>
```





#### master1

配置文件名称：broker-a.properties

修改配置如下：

```sh
#所属集群名字
brokerClusterName=rocketmq-cluster
#broker名字，注意此处不同的配置文件填写的不一样
brokerName=broker-a
#0 表示 Master，>0 表示 Slave
brokerId=0
#nameServer地址，分号分割
namesrvAddr=127.0.0.1:9876
#在发送消息时，自动创建服务器不存在的topic，默认创建的队列数
defaultTopicQueueNums=4
#是否允许 Broker 自动创建Topic，建议线下开启，线上关闭
autoCreateTopicEnable=true
#是否允许 Broker 自动创建订阅组，建议线下开启，线上关闭
autoCreateSubscriptionGroup=true
#Broker 对外服务的监听端口
listenPort=10911
#删除文件时间点，默认凌晨 4点
deleteWhen=04
#文件保留时间，默认 48 小时
fileReservedTime=120
#commitLog每个文件的大小默认1G
mapedFileSizeCommitLog=1073741824
#ConsumeQueue每个文件默认存30W条，根据业务情况调整
mapedFileSizeConsumeQueue=300000
#destroyMapedFileIntervalForcibly=120000
#redeleteHangedFileInterval=120000
#检测物理文件磁盘空间
diskMaxUsedSpaceRatio=88
#存储路径
storePathRootDir=./../data/master1
#commitLog 存储路径
storePathCommitLog=./../data/master1/commitlog
#消费队列存储路径存储路径
storePathConsumeQueue=./../data/master1/consumequeue
#消息索引存储路径
storePathIndex=./../data/master1/index
#checkpoint 文件存储路径
storeCheckpoint=./../data/master1/checkpoint
#abort 文件存储路径
abortFile=./../data/master1/abort
#限制的消息大小
maxMessageSize=65536
#flushCommitLogLeastPages=4
#flushConsumeQueueLeastPages=2
#flushCommitLogThoroughInterval=10000
#flushConsumeQueueThoroughInterval=60000
#Broker 的角色
#- ASYNC_MASTER 异步复制Master
#- SYNC_MASTER 同步双写Master
#- SLAVE
brokerRole=SYNC_MASTER
#刷盘方式
#- ASYNC_FLUSH 异步刷盘
#- SYNC_FLUSH 同步刷盘
flushDiskType=SYNC_FLUSH
#checkTransactionMessageEnable=false
#发消息线程池数量
#sendMessageThreadPoolNums=128
#拉消息线程池数量
#pullMessageThreadPoolNums=128
```





#### slave2

配置文件名称：broker-b-s.properties

```sh
#所属集群名字
brokerClusterName=rocketmq-cluster
#broker名字，注意此处不同的配置文件填写的不一样
brokerName=broker-b
#0 表示 Master，>0 表示 Slave
brokerId=1
#nameServer地址，分号分割
namesrvAddr=127.0.0.1:9876
#在发送消息时，自动创建服务器不存在的topic，默认创建的队列数
defaultTopicQueueNums=4
#是否允许 Broker 自动创建Topic，建议线下开启，线上关闭
autoCreateTopicEnable=true
#是否允许 Broker 自动创建订阅组，建议线下开启，线上关闭
autoCreateSubscriptionGroup=true
#Broker 对外服务的监听端口
listenPort=10932
#删除文件时间点，默认凌晨 4点
deleteWhen=04
#文件保留时间，默认 48 小时
fileReservedTime=120
#commitLog每个文件的大小默认1G
mapedFileSizeCommitLog=1073741824
#ConsumeQueue每个文件默认存30W条，根据业务情况调整
mapedFileSizeConsumeQueue=300000
#destroyMapedFileIntervalForcibly=120000
#redeleteHangedFileInterval=120000
#检测物理文件磁盘空间
diskMaxUsedSpaceRatio=88
#存储路径
storePathRootDir=./../data/slave2
#commitLog 存储路径
storePathCommitLog=./../data/slave2/commitlog
#消费队列存储路径存储路径
storePathConsumeQueue=./../data/slave2/consumequeue
#消息索引存储路径
storePathIndex=./../data/slave2/index
#checkpoint 文件存储路径
storeCheckpoint=./../data/slave2/checkpoint
#abort 文件存储路径
abortFile=./../data/slave2/abort
#限制的消息大小
maxMessageSize=65536
#flushCommitLogLeastPages=4
#flushConsumeQueueLeastPages=2
#flushCommitLogThoroughInterval=10000
#flushConsumeQueueThoroughInterval=60000
#Broker 的角色
#- ASYNC_MASTER 异步复制Master
#- SYNC_MASTER 同步双写Master
#- SLAVE
brokerRole=SLAVE
#刷盘方式
#- ASYNC_FLUSH 异步刷盘
#- SYNC_FLUSH 同步刷盘
flushDiskType=ASYNC_FLUSH
#checkTransactionMessageEnable=false
#发消息线程池数量
#sendMessageThreadPoolNums=128
#拉消息线程池数量
#pullMessageThreadPoolNums=128
```





#### master2

配置文件名称：broker-b.properties

```sh
#所属集群名字
brokerClusterName=rocketmq-cluster
#broker名字，注意此处不同的配置文件填写的不一样
brokerName=broker-b
#0 表示 Master，>0 表示 Slave
brokerId=0
#nameServer地址，分号分割
namesrvAddr=127.0.0.1:9876
#在发送消息时，自动创建服务器不存在的topic，默认创建的队列数
defaultTopicQueueNums=4
#是否允许 Broker 自动创建Topic，建议线下开启，线上关闭
autoCreateTopicEnable=true
#是否允许 Broker 自动创建订阅组，建议线下开启，线上关闭
autoCreateSubscriptionGroup=true
#Broker 对外服务的监听端口
listenPort=10912
#删除文件时间点，默认凌晨 4点
deleteWhen=04
#文件保留时间，默认 48 小时
fileReservedTime=120
#commitLog每个文件的大小默认1G
mapedFileSizeCommitLog=1073741824
#ConsumeQueue每个文件默认存30W条，根据业务情况调整
mapedFileSizeConsumeQueue=300000
#destroyMapedFileIntervalForcibly=120000
#redeleteHangedFileInterval=120000
#检测物理文件磁盘空间
diskMaxUsedSpaceRatio=88
#存储路径
storePathRootDir=./../data/master2
#commitLog 存储路径
storePathCommitLog=./../data/master2/commitlog
#消费队列存储路径存储路径
storePathConsumeQueue=./../data/master2/consumequeue
#消息索引存储路径
storePathIndex=./../data/master2/index
#checkpoint 文件存储路径
storeCheckpoint=./../data/master2/checkpoint
#abort 文件存储路径
abortFile=./../data/master2/abort
#限制的消息大小
maxMessageSize=65536
#flushCommitLogLeastPages=4
#flushConsumeQueueLeastPages=2
#flushCommitLogThoroughInterval=10000
#flushConsumeQueueThoroughInterval=60000
#Broker 的角色
#- ASYNC_MASTER 异步复制Master
#- SYNC_MASTER 同步双写Master
#- SLAVE
brokerRole=SYNC_MASTER
#刷盘方式
#- ASYNC_FLUSH 异步刷盘
#- SYNC_FLUSH 同步刷盘
flushDiskType=SYNC_FLUSH
#checkTransactionMessageEnable=false
#发消息线程池数量
#sendMessageThreadPoolNums=128
#拉消息线程池数量
#pullMessageThreadPoolNums=128
```





#### slave1

配置文件名称：broker-a-s.properties

```sh
#所属集群名字
brokerClusterName=rocketmq-cluster
#broker名字，注意此处不同的配置文件填写的不一样
brokerName=broker-a
#0 表示 Master，>0 表示 Slave
brokerId=1
#nameServer地址，分号分割
namesrvAddr=127.0.0.1:9876
#在发送消息时，自动创建服务器不存在的topic，默认创建的队列数
defaultTopicQueueNums=4
#是否允许 Broker 自动创建Topic，建议线下开启，线上关闭
autoCreateTopicEnable=true
#是否允许 Broker 自动创建订阅组，建议线下开启，线上关闭
autoCreateSubscriptionGroup=true
#Broker 对外服务的监听端口
listenPort=10931
#删除文件时间点，默认凌晨 4点
deleteWhen=04
#文件保留时间，默认 48 小时
fileReservedTime=120
#commitLog每个文件的大小默认1G
mapedFileSizeCommitLog=1073741824
#ConsumeQueue每个文件默认存30W条，根据业务情况调整
mapedFileSizeConsumeQueue=300000
#destroyMapedFileIntervalForcibly=120000
#redeleteHangedFileInterval=120000
#检测物理文件磁盘空间
diskMaxUsedSpaceRatio=88
#存储路径
storePathRootDir=./../data/slave1
#commitLog 存储路径
storePathCommitLog=./../data/slave1/commitlog
#消费队列存储路径存储路径
storePathConsumeQueue=./../data/slave1/consumequeue
#消息索引存储路径
storePathIndex=./../data/slave1/index
#checkpoint 文件存储路径
storeCheckpoint=./../data/slave1/checkpoint
#abort 文件存储路径
abortFile=./../data/slave1/abort
#限制的消息大小
maxMessageSize=65536
#flushCommitLogLeastPages=4
#flushConsumeQueueLeastPages=2
#flushCommitLogThoroughInterval=10000
#flushConsumeQueueThoroughInterval=60000
#Broker 的角色
#- ASYNC_MASTER 异步复制Master
#- SYNC_MASTER 同步双写Master
#- SLAVE
brokerRole=SLAVE
#刷盘方式
#- ASYNC_FLUSH 异步刷盘
#- SYNC_FLUSH 同步刷盘
flushDiskType=ASYNC_FLUSH
#checkTransactionMessageEnable=false
#发消息线程池数量
#sendMessageThreadPoolNums=128
#拉消息线程池数量
#pullMessageThreadPoolNums=128
```











### 服务启动

#### 启动NameServer

```sh
.\mqnamesrv
```



```sh
PS H:\opensoft\rocketmq\bin> .\mqnamesrv
OpenJDK 64-Bit Server VM warning: Using the DefNew young collector with the CMS collector is deprecated and will likely be removed in a future release
OpenJDK 64-Bit Server VM warning: UseCMSCompactAtFullCollection is deprecated and will likely be removed in a future release.
The Name Server boot success. serializeType=JSON
```





#### 启动Broker集群

master1：

```sh
.\mqbroker -c ./../conf/2m-2s-sync/broker-a.properties
```



slave1：

```sh
.\mqbroker -c ./../conf/2m-2s-sync/broker-a-s.properties
```



master2：

```sh
.\mqbroker -c ./../conf/2m-2s-sync/broker-b.properties
```



slave2：

```sh
.\mqbroker -c ./../conf/2m-2s-sync/broker-b-s.properties
```





```sh
PS H:\opensoft\rocketmq\bin> .\mqbroker -c ./../conf/2m-2s-sync/broker-a.properties
The broker[broker-a, 172.27.80.1:10911] boot success. serializeType=JSON and name server is 127.0.0.1:9876
```

```sh
```

