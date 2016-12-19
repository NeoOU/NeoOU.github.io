---
title: Zookeeper+ActiveMQ+Paho高可用消息推送
category: "zookeeper,android,消息中间件"
---

# 背景
真实线上环境，用来取代免费版极光推送。。
# 主要参考资料
我是勤劳的代码搬运工。感谢前人栽树！
- [ActiveMQ的简单使用](http://wosyingjun.iteye.com/blog/2314681)
- [ActiveMQ高可用集群方案](http://wosyingjun.iteye.com/blog/2314683)
- [ZooKeeper高可用集群的安装及配置](http://wosyingjun.iteye.com/blog/2312960)
- [使用 IBM MessageSight 和 MQTT 客户端应用程序实现高可用性](http://www.ibm.com/developerworks/cn/websphere/library/techarticles/1406_bakowski/1406_bakowski.html?ca=drs-&utm_source=tuicool&utm_medium=referral)

# 集群环境搭建
Zookeeper和ActiveMQ的安装配置步骤应君大人的文章讲解得很详细，只记录下自己的配置内容
## 线上环境
- 阿里云ECS服务器3台
- 服务器都预装[微柳网络V1.5镜像](https://market.aliyun.com/products/55528001/jxsc000238.html?spm=5176.ecsPrepay.image.selectFromMarketplace.52usUH)环境

## 我的Zookeeper集群配置
### 节点1的zoo.cgf
```
# The number of milliseconds of each tick
tickTime=2000

# The number of ticks that the initial
# synchronization phase can take
initLimit=10

# The number of ticks that can pass between
# sending a request and getting an acknowledgement
syncLimit=5

# the directory where the snapshot is stored.
# do not use /tmp for storage, /tmp here is just
# example sakes.
#dataDir=/usr/local/zookeeper/zookeeper-1/data
#dataLogDir=/usr/local/zookeeper/zookeeper-1/logs
dataDir=/tmp/zookeeper

# the port at which the clients will connect
clientPort=2181

server.1=0.0.0.0:2881:38811
server.2=120.77.***.***:2882:38822
server.3=120.77.***.***:2883:38833
```
### 节点2的zoo.cgf
主体与节点1相同
```
server.1=120.77.***.***:2881:38811
server.2=0.0.0.0:2882:38822
server.3=120.77.***.***:2883:38833
```
### 节点3的zoo.cgf
主体与节点1相同
```
server.1=120.77.***.***:2881:38811
server.2=120.77.***.***:2882:38822
server.3=0.0.0.0:2883:38833
```
## 我的ActiveMQ集群配置
### 各节点activemq.xml的broker配置
以下是节点1的配置，节点2、3的配置与1一致，只是把replicatedLevelDB的hostname属性值改为各自本机ip：
```
<persistenceAdapter>
<!--
    <kahaDB directory="${activemq.data}/kahadb"/>
    directory="activemq-data"
-->
    <replicatedLevelDB
         # The directory which the store will use to hold it's data files.
         # The store will create the directory if it does not already exist.
         directory="${activemq.data}/leveldb"

         # The number of nodes that will exist in the cluster.
         # At least (replicas/2)+1 nodes must be online to avoid service outage.
         replicas="3"

         # When this node becomes a master,
         # it will bind the configured address and port to service the replication protocol.
         bind="tcp://0.0.0.0:62621"

         # A comma separated list of ZooKeeper servers.
         zkAddress="120.77.***.***:2181,120.77.***.***:2182,120.77.***.***:2183"

         # The path to the ZooKeeper directory where Master/Slave election information will be exchanged.
         zkPath="/activemq/leveldb-stores"

         # The host name used to advertise the replication service when this node becomes the master.
         # If not set it will be automatically determined.
         # 本机ip
         hostname="120.77.***.***"/>
</persistenceAdapter>
<!-- username and password for connecting -->
<plugins>  
    <simpleAuthenticationPlugin>  
        <users>  
            <authenticationUser username="user1" password="user1" groups="users,admins"/>  
        </users>  
    </simpleAuthenticationPlugin>  
</plugins>
```

### 更改各节点控制台登陆密码
更改conf下jetty-realm.properties
```
# Defines users that can access the web (console, demo, etc.)
# username: password [,rolename ...]
admin: 123456, admin
user: 123456, user
```

# 逻辑实现
## 我的server端代码
## 我的client端代码
# 幺蛾子
各种各样幺蛾子满天飞。

## Zookeeper集群搭建
重点配置只有下面几行，可是，各种错误。。。
```
dataDir=/tmp/zookeeper
clientPort=2181
server.1=0.0.0.0:2881:38811
server.2=120.77.***.***:2882:38822
server.3=120.77.***.***:2883:38833
```
### 说明
在服务器上搭建完成后，想在本地虚拟机上重现错误，按应君大人的搭建步骤，结果一次通过。
## ActiveMQ集群搭建
## Server端代码
## Client端代码
