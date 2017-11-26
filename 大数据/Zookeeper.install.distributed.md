title: 快速搭建zookeeper集群
tags: 
	- Linux
	- Hadoop
	- Zookeeper

categories:
	- 大数据
-------------------

zookeeper(简称zk)在安装的时候并没有主从之分，通过选举产生主节点角色，相对与需要配置主从关系的一些分布式软件来说，zk安装相对简单
官网下载zk安装包 [http://www.apache.org/dyn/closer.cgi/zookeeper/](http://www.apache.org/dyn/closer.cgi/zookeeper/)

### 解压
```
]# pwd
/opt
]# tar -zxvf zookeeper.tar.gz
..省略解压日志
]# mv /opt/zookeeper  /opt/zookeeper
```

###    单节点
重命名**zoo_sample.cfg**为**zoo.cfg**
```
]# cd /opt/zookeeper/conf/
]# mv zoo_sample.cfg  zoo.cfg
```
修改配置文件**zoo.cfg**，将原本的**dataDir=/tmp/zookeeper**注释掉，重新定义为**dataDir=/opt/zookeeper/data**
```
#dataDir=/tmp/zookeeper
dataDir=/opt/zookeeper/data
```
再在**zoo.cfg**配置文件中添加如下配置（集群节点信息配置），其中node1为当前节点的主机名
```
server.1=node1:2888:3888
server.2=node2:2888:3888
server.1=node3:2888:3888
```
在前面重新定义的**dataDir**目录写入节点号，与**server.1=node1:2888:3888**中的server.1对应为1写如
```
]# cd /opt/zookeeper/data
]# echo "1" > myid
```

###   其他节点

将已经配置好的zookeeper拷贝到其他节点的相同位置/opt
```
]# scp -r /opt/zookeeper root@node2:/opt/
...神略拷贝日志
]# scp -r /opt/zookeeper root@node3:/opt/
...神略拷贝日志
```

同理配置**dataDir**目录中的节点号，node2节点下:
```
]# cd /opt/zookeeper/data
]# echo "2" > myid
```
node3节点下:
```
]# cd /opt/zookeeper/data
]# echo "3" > myid
```

###   启动
分别启动每个节点的zkServer
zk没有启动整个集群的脚本，都是单节点启动，需要手动启动每个节点的服务（当然可以手写启动脚本，ssh远程启动其他节点的服务）
```
]# /opt/zookeeper/bin/zkServer.sh start
```

单节点查看进程
```
]# jps
3757 Jps
2963 QuorumPeerMain     #zk服务进程名
```

如果启动失败，可监控日志`tail -f /opt/zookeeper/zookeeper.out`

启动完成之后，查看节点角色
```
]# /opt/zookeeper/bin/zkServer.sh status
JMX enabled by default
Using config: /opt/zookeeper/bin/../conf/zoo.cfg
Mode: follower   #主节点显示为leader
```

