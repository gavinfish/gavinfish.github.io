---
layout: post
title:  "Zookeeper集群的部署"
date:   2015-12-25
categories: 大数据
tags: Zookeeper

---

# Zookeeper集群的部署

---

ZooKeeper是一个开源的分布式应用程序协调服务，是Google的Chubby一个开源的实现，是Hadoop、Hbase、Kafka等流行开源框架的重要组件。

以下实验环境为Ubuntu14.04，局域网内的三台普通计算机，虚拟机可以进行相同的配置。

## 配置IP映射

为了方便后续的操作，以及容易修改配置信息，我们先做个IP地址的映射。在/etc/hosts文件中添加如下内容：

```
192.168.0.100   zk1
192.168.0.101   zk2  
192.168.0.102   zk3 
```

注意每台机器都要进行相同的配置。

## 下载及配置

我们先在zk1的机器上进行配置，我下载的是 [zookeeper-3.4.7](http://www.eu.apache.org/dist/zookeeper/zookeeper-3.4.7/zookeeper-3.4.7.tar.gz)。解压完成后要复制该目录下的 `conf/zoo_sample.cfg` 为 `conf/zoo.cfg` ，并对 `zoo.cfg` 进行配置：

```
tickTime=2000  
# 修改成任意想要存放数据的位置，建议使用绝对路径
dataDir=/home/user/storage/zookeeper  
clientPort=2181  
initLimit=5  
syncLimit=2  
server.1=zk1:2888:3888  
server.2=zk2:2888:3888  
server.3=zk3:2888:3888  
```

在dataDir指定的目录下创建文件myid，里面添加一个数字1。

## 远程复制配置

现在把zk1上的文件复制到zk2和zk3上：

```
scp -r zookeeper-3.4.7/ user@zk2:/home/user/ 
scp -r zookeeper-3.4.7/ user@zk3:/home/user/
```

在这两台机器上也要有myid文件，内容改成相应的数字。

## 启动集群

在每台机器上zookeeper目录下执行以下命令来启动zookeeper服务：

```
bin/zkServer.sh start
```

启动zookeeper时每个节点都会试图去连接集群中的其他节点，所以在开启前面两台上的服务时会记录一些异常，等所有集群上的机器都启动完毕就恢复正常了。

## 日志查看

zookeeper的日志在目录下的zookeeper.out文件中，可以通过查看日志来了解启动的状况与节点运行情况，如下面的命令可以查看日志的最新部分：

```
tail -200f zookeeper.out
```

## 运行情况

我们还可以通过脚本来检测一下节点的运行情况：

```
bin/zkServer.sh status
```

下面是一个节点的输出，节点模式分为leader和follower：

```
ZooKeeper JMX enabled by default
Using config: /home/user/zookeeper-3.4.7/bin/../conf/zoo.cfg
Mode: follower
```