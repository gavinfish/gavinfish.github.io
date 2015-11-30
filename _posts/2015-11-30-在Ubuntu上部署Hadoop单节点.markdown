---
layout: post
title:  "在Ubuntu上部署Hadoop单节点"
date:   2015-11-30
categories: Hadoop 
tags: Hadoop Ubuntu

---

# 在Ubuntu上部署Hadoop单节点

## 安装Java环境

Hadoop框架是依赖Java环境的，直接用最简单的方式,直接安装OpenJDK。

```
# 更新一下源列表
fish@hadoop:~$ sudo apt-get update

# 安装OpenJDK
fish@hadoop:~$ sudo apt-get install default-jdk

# 测试Java环境
fish@hadoop:~$ java -version
java version "1.7.0_65"
OpenJDK Runtime Environment (IcedTea 2.5.3) (7u71-2.5.3-0ubuntu0.14.04.1)
OpenJDK 64-Bit Server VM (build 24.65-b04, mixed mode)
```

## 安装SSH

Hadoop要依赖于SSH来管理节点，下面安装SSH。

```
# 安装SSH
fish@hadoop:~$ sudo apt-get install ssh

# 测试SSH
fish@hadoop:~$ which ssh
/usr/bin/ssh
fish@hadoop:~$ which sshd
/usr/sbin/sshd
```

## 添加Hadoop用户

为Hadoop添加专门的用户。

```
# 添加用户
fish@hadoop:~$ sudo addgroup hadoop
Adding group `hadoop' (GID 1002) ...
Done.

fish@hadoop:~$ sudo adduser --ingroup hadoop hduser
Adding user `hduser' ...
Adding new user `hduser' (1001) with group `hadoop' ...
Creating home directory `/home/hduser' ...
Copying files from `/etc/skel' ...
Enter new UNIX password: 
Retype new UNIX password: 
passwd: password updated successfully
Changing the user information for hduser
Enter the new value, or press ENTER for the default
	Full Name []: 
	Room Number []: 
	Work Phone []: 
	Home Phone []: 
	Other []: 
Is the information correct? [Y/n] Y

# 添加root权限
fish@hadoop:~$ sudo adduser hduser sudo
[sudo] password for fish: 
Adding user `hduser' to group `sudo' ...
Adding user hduser to group sudo
Done.
```

## 设置SSH密钥

```
# 切换用户
fish@hadoop:/home$ su hduser
Password: 
hduser@hadoop:/home$ cd ~

# 生成密钥
hduser@hadoop:~$ ssh-keygen -t rsa -P ""
Generating public/private rsa key pair.
Enter file in which to save the key (/home/hduser/.ssh/id_rsa): 
Created directory '/home/hduser/.ssh'.
Your identification has been saved in /home/hduser/.ssh/id_rsa.
Your public key has been saved in /home/hduser/.ssh/id_rsa.pub.
The key fingerprint is:
93:94:8f:3b:8e:c8:2b:c0:52:96:6c:8c:61:54:46:03 hduser@hadoop-7260
The key's randomart image is:
+--[ RSA 2048]----+
|.E+=             |
|... .    .       |
|.= .    o        |
|. B    . +       |
|.+      S .      |
|o.       o       |
|..      o        |
|  .. . o .       |
|   .+.. .        |
+-----------------+
'

# 添加到认证密钥中，避免每次都要输密码
hduser@hadoop:~$ cat $HOME/.ssh/id_rsa.pub >> $HOME/.ssh/authorized_keys
```

## 安装Hadoop

```
# 下载解压Hadoop
hduser@hadoop:~$ wget http://mirrors.sonic.net/apache/hadoop/common/hadoop-2.7.1/hadoop-2.7.1.tar.gz
hduser@hadoop:~$ tar xvzf hadoop-2.7.1.tar.gz

# 创建存放目录
hduser@hadoop:~$ sudo mkdir /usr/local/hadoop
hduser@hadoop:~$ cd hadoop-2.7.1/
hduser@hadoop:~/hadoop-2.7.1$ sudo mv * /usr/local/hadoop
hduser@hadoop:~/hadoop-2.7.1$ sudo chown -R hduser:hadoop /usr/local/hadoop
```

## 设置配置文件

Hadoop还有很多配置文件要设置。

##### 1. ~/.bashrc

在bashrc文件中添加如下内容：

```
#HADOOP VARIABLES START
export JAVA_HOME=/usr/lib/jvm/java-7-openjdk-amd64
export HADOOP_INSTALL=/usr/local/hadoop
export PATH=$PATH:$HADOOP_INSTALL/bin
export PATH=$PATH:$HADOOP_INSTALL/sbin
export HADOOP_MAPRED_HOME=$HADOOP_INSTALL
export HADOOP_COMMON_HOME=$HADOOP_INSTALL
export HADOOP_HDFS_HOME=$HADOOP_INSTALL
export YARN_HOME=$HADOOP_INSTALL
export HADOOP_COMMON_LIB_NATIVE_DIR=$HADOOP_INSTALL/lib/native
export HADOOP_OPTS="-Djava.library.path=$HADOOP_INSTALL/lib"
#HADOOP VARIABLES END
```

```
# 查看Java路径
hduser@hadoop:~$ update-alternatives --config java
There is only one alternative in link group java (providing /usr/bin/java): /usr/lib/jvm/java-7-openjdk-amd64/jre/bin/java
Nothing to configure.

# 编辑~/.bashrc
hduser@hadoop:~$ vi ~/.bashrc
hduser@hadoop:~$ source ~/.bashrc
```

##### 2. /usr/local/hadoop/etc/hadoop/hadoop-env.sh

在hadoop-env.sh文件中添加如下内容：

```
export JAVA_HOME=/usr/lib/jvm/java-7-openjdk-amd64
```

```
hduser@hadoop:~$ vi /usr/local/hadoop/etc/hadoop/hadoop-env.sh
```

##### 3. /usr/local/hadoop/etc/hadoop/core-site.xml

将以下内容添加到core-site.xml文件中的`<configuration>`标签中：

```xml
<property>
  <name>hadoop.tmp.dir</name>
  <value>/app/hadoop/tmp</value>
  <description>A base for other temporary directories.</description>
 </property>

 <property>
  <name>fs.default.name</name>
  <value>hdfs://localhost:54310</value>
  <description>The name of the default file system.  A URI whose
  scheme and authority determine the FileSystem implementation.  The
  uri's scheme determines the config property (fs.SCHEME.impl) naming
  the FileSystem implementation class.  The uri's authority is used to
  determine the host, port, etc. for a filesystem.</description>
 </property>
```

```
hduser@hadoop:~$ sudo mkdir -p /app/hadoop/tmp
hduser@hadoop:~$ sudo chown hduser:hadoop /app/hadoop/tmp
hduser@hadoop:~$ vi /usr/local/hadoop/etc/hadoop/core-site.xml
```

##### 4. /usr/local/hadoop/etc/hadoop/mapred-site.xml

将以下内容添加到mapred-site.xml文件中的`<configuration>`标签中：

```xml
<property>
  <name>mapred.job.tracker</name>
  <value>localhost:54311</value>
  <description>The host and port that the MapReduce job tracker runs
  at.  If "local", then jobs are run in-process as a single map
  and reduce task.
  </description>
 </property>
```

```
hduser@hadoop:~$ cp /usr/local/hadoop/etc/hadoop/mapred-site.xml.template /usr/local/hadoop/etc/hadoop/mapred-site.xml
hduser@hadoop:~$ vi /usr/local/hadoop/etc/hadoop/mapred-site.xml
```

##### 5. /usr/local/hadoop/etc/hadoop/hdfs-site.xml

将以下内容添加到hdfs-site.xml文件中的`<configuration>`标签中：

```xml
 <property>
  <name>dfs.replication</name>
  <value>1</value>
  <description>Default block replication.
  The actual number of replications can be specified when the file is created.
  The default is used if replication is not specified in create time.
  </description>
 </property>
 <property>
   <name>dfs.namenode.name.dir</name>
   <value>file:/usr/local/hadoop_store/hdfs/namenode</value>
 </property>
 <property>
   <name>dfs.datanode.data.dir</name>
   <value>file:/usr/local/hadoop_store/hdfs/datanode</value>
 </property>
```

```
hduser@hadoop:~$ sudo mkdir -p /usr/local/hadoop_store/hdfs/namenode
hduser@hadoop:~$ sudo mkdir -p /usr/local/hadoop_store/hdfs/datanode
hduser@hadoop:~$ sudo chown -R hduser:hadoop /usr/local/hadoop_store
hduser@hadoop:~$ vi /usr/local/hadoop/etc/hadoop/hdfs-site.xml
```

## 格式化Hadoop文件系统

在开始使用之前，我们要格式化Hadoop文件系统。

```
hduser@hadoop:~$ hadoop namenode -format
DEPRECATED: Use of this script to execute hdfs command is deprecated.
Instead use the hdfs command for it.

15/11/29 19:49:27 INFO namenode.NameNode: STARTUP_MSG: 
/************************************************************
STARTUP_MSG: Starting NameNode
STARTUP_MSG:   host = hadoop/10.64.253.22
STARTUP_MSG:   args = [-format]
STARTUP_MSG:   version = 2.7.1
......
15/11/29 19:49:29 INFO util.ExitUtil: Exiting with status 0
15/11/29 19:49:29 INFO namenode.NameNode: SHUTDOWN_MSG: 
/************************************************************
SHUTDOWN_MSG: Shutting down NameNode at hadoop-7260.lvs01.dev.ebayc3.com/10.64.253.22
************************************************************/
```

## 启动Hadoop

```
hduser@hadoop-7260:~$ cd /usr/local/hadoop/sbin/
// 查看所有的命令集
hduser@hadoop-7260:/usr/local/hadoop/sbin$ ls
distribute-exclude.sh    start-all.cmd        stop-balancer.sh
hadoop-daemon.sh         start-all.sh         stop-dfs.cmd
hadoop-daemons.sh        start-balancer.sh    stop-dfs.sh
hdfs-config.cmd          start-dfs.cmd        stop-secure-dns.sh
hdfs-config.sh           start-dfs.sh         stop-yarn.cmd
httpfs.sh                start-secure-dns.sh  stop-yarn.sh
kms.sh                   start-yarn.cmd       yarn-daemon.sh
mr-jobhistory-daemon.sh  start-yarn.sh        yarn-daemons.sh
refresh-namenodes.sh     stop-all.cmd
slaves.sh                stop-all.sh
// 启动Hadoop
hduser@hadoop:/usr/local/hadoop/sbin$ start-all.sh 
This script is Deprecated. Instead use start-dfs.sh and start-yarn.sh
15/11/29 19:56:23 WARN util.NativeCodeLoader: Unable to load native-hadoop library for your platform... using builtin-java classes where applicable
Starting namenodes on [localhost]
......
# 检查启动情况
hduser@hadoop-7260:/usr/local/hadoop/sbin$ jps
5814 ResourceManager
5620 SecondaryNameNode
5401 DataNode
5951 NodeManager
5239 NameNode
6837 Jps
```

至此Hadoop在Ubuntu中的搭建工作完成。