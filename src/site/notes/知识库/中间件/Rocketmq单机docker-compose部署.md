---
{"dg-publish":true,"permalink":"/知识库/中间件/Rocketmq单机docker-compose部署/"}
---

创建一个rocketmq目录用作部署资料目录
``` shell
mkdir rocketmq
```

#### Nameserver节点

创建nameserver所需目录
``` shell
cd rocketmq
mkdir nameserver/logs -p
mkdir nameserver/bin -p
```

设置权限
``` shell
# 777 文件所属者、文件所属组和其他人有读取 & 写入 & 执行全部权限。rwxrwxrwx
chmod 777 -R nameserver/*
```

创建一个临时容器，从容器中拷贝需要挂载到宿主机的配置文件
``` shell
docker run -d --privileged=true --name rmqnamesrv apache/rocketmq:5.1.0 sh mqnamesrv
```

``` shell
docker cp rmqnamesrv:/home/rocketmq/rocketmq-5.1.0/bin/runserver.sh ./nameserver/bin/runserver.sh
```

修改`runserver.sh`，找到调用`calculate_heap_sizes`函数的位置注释掉保存即可，拉到脚本最底部就能找到

``` shell
vim ./nameserver/bin/runserver.sh
```

![image.png](https://cdn.jsdelivr.net/gh/CtrlAndZ/imgded@main/blog/20240325100211.png)

删除临时容器
``` shell
docker rm -f rmqnamesrv
```

#### Broker节点
创建broker所需目录
``` shell
mkdir broker/logs -p
mkdir broker/data -p
mkdir broker/conf -p
mkdir broker/bin -p
```

设置权限
``` shell
# 777 文件所属者、文件所属组和其他人有读取 & 写入 & 执行全部权限。rwxrwxrwx
chmod 777 -R broker/*
```

创建broker配置文件
``` shell
vim ./broker/conf/broker.conf
```

``` properties
# 集群名称
brokerClusterName = DefaultCluster
# 节点名称
brokerName = broker-a
# broker id节点ID， 0 表示 master, 其他的正整数表示 slave，不能小于0 
brokerId = 0
# Broker服务地址        String  内部使用填内网ip，如果是需要给外部使用填公网ip
brokerIP1 = 172.19.150.1
# Broker角色
brokerRole = ASYNC_MASTER
# 刷盘方式
flushDiskType = ASYNC_FLUSH
# 在每天的什么时间删除已经超过文件保留时间的 commit log，默认值04
deleteWhen = 04
# 以小时计算的文件保留时间 默认值72小时
fileReservedTime = 72
# 是否允许Broker 自动创建Topic，建议线下开启，线上关闭
autoCreateTopicEnable=true
# 是否允许Broker自动创建订阅组，建议线下开启，线上关闭
autoCreateSubscriptionGroup=true
```

创建一个临时容器，从容器中拷贝需要挂载到宿主机的配置文件

``` shell
docker run -d --name mqbroker --privileged=true apache/rocketmq:5.1.0 sh mqbroker
```

``` shell
docker cp mqbroker:/home/rocketmq/rocketmq-5.1.0/bin/runbroker.sh ./broker/bin/runbroker.sh
```

修改`runserver.sh`，找到调用`calculate_heap_sizes`函数的位置注释掉保存即可，拉到脚本最底部就能找到（步骤与上面Nameserver节点一样）

删除broker临时容器
``` shell
docker rm -f mqbroker
```

#### 创建docker-compose.yml

``` shell 
vim docker-compose.yml
```

``` yml
version: '3.8'
services:
  rmqnamesrv:
    image: apache/rocketmq:5.1.0
    container_name: rmqnamesrv
    ports:
      - 9876:9876
    restart: always
    privileged: true
    volumes:
      - ./nameserver/logs:/home/rocketmq/logs
      - ./nameserver/bin/runserver.sh:/home/rocketmq/rocketmq-5.1.0/bin/runserver.sh
    environment:
      - MAX_HEAP_SIZE=256M
      - HEAP_NEWSIZE=128M
    command: ["sh","mqnamesrv"]
  broker:
    image: apache/rocketmq:5.1.0
    container_name: rmqbroker
    ports:
      - 10909:10909
      - 10911:10911
    restart: always
    privileged: true
    volumes:
      - ./broker/logs:/home/rocketmq/logs
      - ./broker/store:/home/rocketmq/logs
      - ./broker/conf/broker.conf:/home/rocketmq/broker.conf
      - ./broker/bin/runbroker.sh:/home/rocketmq/rocketmq-5.1.0/bin/runbroker.sh
    depends_on:
      - 'rmqnamesrv'
    environment:
      - NAMESRV_ADDR=rmqnamesrv:9876
      - MAX_HEAP_SIZE=512M
      - HEAP_NEWSIZE=256M
    command: ["sh","mqbroker","-c","/home/rocketmq/broker.conf"]
  # rmqdashboard容器为可选项
  rmqdashboard:
    image: apacherocketmq/rocketmq-dashboard:latest
    container_name: rocketmq-dashboard
    ports:
      - 8080:8080
    restart: always
    privileged: true
    depends_on:
      - 'rmqnamesrv'
    environment:
      - JAVA_OPTS= -Xmx256M -Xms256M -Xmn128M -Drocketmq.namesrv.addr=rmqnamesrv:9876 -Dcom.rocketmq.sendMessageWithVIPChannel=false
```

其中rmqdashboard为可选项，是rockermq的可视化工具，不用可以不部署

启动容器
``` shell
docker-compose up -d
```
