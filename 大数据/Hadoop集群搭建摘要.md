title: Hadoop集群搭建摘要

tags: 
	- Hadoop

categories:
	- 大数据

-----------------------------

 
>本文摘取自hadoop官方文档。版本Apache Hadoop 2.6.5
1.[http://hadoop.apache.org/docs/r2.6.5/hadoop-project-dist/hadoop-common/SingleCluster.html](http://hadoop.apache.org/docs/r2.6.5/hadoop-project-dist/hadoop-common/SingleCluster.html)
2.[http://hadoop.apache.org/docs/r2.6.5/hadoop-project-dist/hadoop-common/ClusterSetup.html](http://hadoop.apache.org/docs/r2.6.5/hadoop-project-dist/hadoop-common/ClusterSetup.html)
 
 
 
##    前提
 
###    系统
vmware新建两台虚拟机：**hado88**，**hado87**
 
主机映射
```
##在/etc/hosts最后追加如下映射
192.168.33.88   hado88
192.168.33.87   hado87
```
 
修改主机名
```
##hado88修改/etc/sysconfig/network
HOSTNAME=hado88
 
##hado87修改/etc/sysconfig/network
HOSTNAME=hado87
```
 
 
###    安装    java （略）
###    安装ssh && rsync    （略）
###   ssh免密

hado88和hado87都执行以下命令:
```
$ ssh-keygen -t rsa 
//一路回车
$ ssh-copy-id root@hado88
$ ssh-copy-id root@hado87
```




 
 
##   下载
[https://mirrors.tuna.tsinghua.edu.cn/apache/hadoop/common/](https://mirrors.tuna.tsinghua.edu.cn/apache/hadoop/common/)
下载hadoop-2.6.5.tar.gz    解压后放到/opt/hadoop-2.6.5
 
 
 
##    环境变量配置文件文件
`etc/hadoop/yarn-env.sh`和`etc/hadoop/hadoop-env.sh`分別是hdfs和yarn的环境变量。其中以变量名HADOOP打头的是hadoop-env.sh配置的,以YARN打头的是yarn-env.sh配置的
 
###    必须配置的
在   `etc/hadoop/hadoop-env.sh`中需要配置如下两行
```
export JAVA_HOME=path/to/java
export HADOOP_PREFIX=path/to/hadoop
```

其中**HADOOP_PREFIX**也不是必须配置的。hadoop-env.sh文件中明确说明，必须配置的是**JAVA_HOME**。hadoop-env.sh原文注释如下：
```
# The only required environment variable is JAVA_HOME.  All others are
# optional.  When running a distributed configuration it is best to
# set JAVA_HOME in this file, so that it is correctly defined on
# remote nodes.
```
 
###    建议配置的

```
export   HADOOP_PID_DIR=path/to/HADOOP_PID_DIR
export   HADOOP_SECURE_DN_PID_DIR=path/to/HADOOP_SECURE_DN_PID_DIR
```


官网原文：`In most cases you should also specify HADOOP_PID_DIR and HADOOP_SECURE_DN_PID_DIR to point to directories 
that can only be written to by the users that are going to run the hadoop daemons. Otherwise there is the 
potential for a symlink attack.`

 

 
###    其他
 
####   代表JVM参数的环境变量
| 进程|    环境变量 |
| :-------- |: -------- |
| Daemon  |   Environment Variable |
| NameNode  |   HADOOP_NAMENODE_OPTS |
| DataNode  |  HADOOP_DATANODE_OPTS |
| Secondary NameNode  |   HADOOP_SECONDARYNAMENODE_OPTS |
| ResourceManager   |  YARN_RESOURCEMANAGER_OPTS |
| NodeManager   |  YARN_NODEMANAGER_OPTS |
| WebAppProxy   |  YARN_PROXYSERVER_OPTS |
| Map Reduce Job History Server    | HADOOP_JOB_HISTORYSERVER_OPTS |
 
####   其他有用的环境变量
| 进程|    环境变量 |
| :-------- |: -------- |
| ResourceManager   |   YARN_RESOURCEMANAGER_HEAPSIZE  |
| NodeManager   |   YARN_NODEMANAGER_HEAPSIZE  |
| WebAppProxy  |    YARN_PROXYSERVER_HEAPSIZE  |
| Map Reduce Job History Server   |   HADOOP_JOB_HISTORYSERVER_HEAPSIZE  |
 
##    xml配置文件
核心xml配置文件包括`core-site.xml`,`hdfs-site.xml`,`yarn-site.xml`,`mapred-site.xml`，均在目录 etc/hadoop/下面。格式如下：
 
```
<configuration>
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://hado88:9000</value>
    </property>
</configuration>
```
 
###    core-site.xml配置文件
| Parameter  |   Value    | Notes |
| :-------- |: -------- |: -------- |
| fs.defaultFS  |   NameNode URI  |   hdfs://host:port/          Namenode 的节点设置以及hdfs端口|
|  io.file.buffer.size    |   131072   |    Size of read/write buffer used in SequenceFiles. |
 
 
 
 
###    conf/hdfs-site.xml配置文件
####  Configurations for NameNode
| Parameter  |   Value  |   Notes |
| :-------- |: -------- |: -------- |
|  `dfs.namenode.name.dir`  |   Path on the local filesystem where the NameNode stores the namespace and transactions logs persistently.   |   If this is a comma-delimited list of directories then the name table is replicated in all of the directories, for redundancy. |
|  dfs.namenode.hosts / dfs.namenode.hosts.exclude   |  List of permitted/excluded DataNodes.   |   If necessary, use these files to control the list of allowable datanodes.
|  dfs.blocksize    |  268435456   |   HDFS blocksize of 256MB for large file-systems. |
|  dfs.namenode.handler.count  |     100 | More NameNode server threads to handle RPCs from large number of DataNodes.
 
####  Configurations for DataNode
| Parameter  |   Value  |   Notes |
| :-------- |: -------- |: -------- |
|  `dfs.datanode.data.dir`   |   Comma separated list of paths on the local filesystem of a DataNode where it should store its blocks.  |    If this is a comma-delimited list of directories, then data will be stored in all named directories, typically on different devices. |
| dfs.replication | 3 |Default block replication. The actual number of replications can be specified when the file is created. The default is used if replication is not specified in create time. |
 
###   yarn-site.xml配置文件
####   Configurations for ResourceManager and NodeManager:
 
| Parameter  |   Value  |   Notes |
| :-------- |: -------- |: -------- |
|  yarn.acl.enable   |   true / false    |  Enable ACLs? Defaults to false.
|  yarn.admin.acl    |  Admin ACL  |    ACL to set admins on the cluster. ACLs are of for comma-separated-usersspacecomma-separated-groups. Defaults to special value of * which means anyone. Special value of just space means no one has access.
|  yarn.log-aggregation-enable    |  false   |   Configuration to enable or disable log aggregation
 
####    Configurations for ResourceManager:
 
 
| Parameter  |   Value  |   Notes |
| :-------- |: -------- |: -------- |
    |  `yarn.resourcemanager.address`      |    ResourceManager host:port for clients to submit jobs.        |  host:port    。 If set    overrides the hostname set in yarn.resourcemanager.hostname.        |
    |  yarn.resourcemanager.scheduler.address       |   ResourceManager host:port for ApplicationMasters to talk to Scheduler to obtain resources.       |   host:port。If set, overrides the hostname set in yarn.resourcemanager.hostname.    |
    |  yarn.resourcemanager.resource-tracker.address       |   ResourceManager host:port for NodeManagers.        |  host:port。If set, overrides the hostname set in yarn.resourcemanager.hostname.    |
    |  yarn.resourcemanager.admin.address      |    ResourceManager host:port for administrative commands.      |    host:port。If set, overrides the hostname set in yarn.resourcemanager.hostname.    |
    |  yarn.resourcemanager.webapp.address      |    ResourceManager web-ui host:port.       |   host:port。If set, overrides the hostname set in yarn.resourcemanager.hostname.    |
    |  yarn.resourcemanager.hostname     |     ResourceManager host.        |  host。Single hostname that can be set in place of setting all yarn.resourcemanager*address resources. Results in default ports for ResourceManager components.    |
    |  yarn.resourcemanager.scheduler.class      |    ResourceManager Scheduler class.       |   CapacityScheduler (recommended), FairScheduler (also recommended), or FifoScheduler    |
    |  yarn.scheduler.minimum-allocation-mb      |    Minimum limit of memory to allocate to each container request at the Resource Manager.       |   In MBs    |
    |  yarn.scheduler.maximum-allocation-mb     |     Maximum limit of memory to allocate to each container request at the Resource Manager.      |    In MBs    |
    |  yarn.resourcemanager.nodes.include-path / yarn.resourcemanager.nodes.exclude-path      |    List of permitted/excluded NodeManagers.       |   If necessary, use these files to control the list of allowable NodeManagers.    |
 
 
####    Configurations for NodeManager:
 
| Parameter  |   Value  |   Notes |
| :-------- |: -------- |: -------- |
   |  yarn.nodemanager.resource.memory-mb       |  Resource i.e. available physical memory, in MB, for given NodeManager     |    Defines total available resources on the NodeManager to be made available to running containers   |
   |  yarn.nodemanager.vmem-pmem-ratio     |    Maximum ratio by which virtual memory usage of tasks may exceed physical memory       |  The virtual memory usage of each task may exceed its physical memory limit by this ratio. The total amount of virtual memory used by tasks on the NodeManager may exceed its physical memory usage by this ratio.
   |  yarn.nodemanager.local-dirs      |   Comma-separated list of paths on the local filesystem where intermediate data is written.      |   Multiple paths help spread disk i/o.
   |  yarn.nodemanager.log-dirs      |   Comma-separated list of paths on the local filesystem where logs are written.     |    Multiple paths help spread disk i/o.
   |  yarn.nodemanager.log.retain-seconds       |  10800     |    Default time (in seconds) to retain log files on the NodeManager Only applicable if log-aggregation is disabled.
   |  yarn.nodemanager.remote-app-log-dir       |  /logs      |   HDFS directory where the application logs are moved on application completion. Need to set appropriate permissions. Only applicable if log-aggregation is enabled.
   |  yarn.nodemanager.remote-app-log-dir-suffix    |     logs       |  Suffix appended to the remote log dir. Logs will be aggregated to ${yarn.nodemanager.remote-app-log-dir}/${user}/${thisParam} Only applicable if log-aggregation is enabled.
   |  `yarn.nodemanager.aux-services`      |   mapreduce_shuffle      |   Shuffle service that needs to be set for Map Reduce applications.
 
####    Configurations for History Server (Needs to be moved elsewhere):
 
 
| Parameter  |   Value  |   Notes |
| :-------- |: -------- |: -------- |
 |  yarn.log-aggregation.retain-seconds      |           -1           |    How long to keep aggregation logs before deleting them. -1 disables. Be careful, set this too small and you will spam the name node.      |
   |  yarn.log-aggregation.retain-check-interval-seconds      |            -1     |    Time between checks for aggregated log retention. If set to 0 or a negative value then the value is computed as one-tenth of the aggregated log retention time. Be careful, set this too small and you will spam the name node.  | 
 
 
###    conf/mapred-site.xml
####   Configurations for MapReduce Applications:
| Parameter  |   Value  |   Notes |
| :-------- |: -------- |: -------- |
 |  `mapreduce.framework.name`     |  yarn    |   Execution framework set to Hadoop YARN.
 |  mapreduce.map.memory.mb     |  1536    |   Larger resource limit for maps.
 |  mapreduce.map.java.opts    |   -Xmx1024M    |   Larger heap-size for child jvms of maps.
 |  mapreduce.reduce.memory.mb   |    3072 |      Larger resource limit for reduces.
 |  mapreduce.reduce.java.opts    |   -Xmx2560M   |   Larger heap-size for child jvms of reduces.
 |  mapreduce.task.io.sort.mb    |   512  |     Higher memory-limit while sorting data for efficiency.
 |  mapreduce.task.io.sort.factor    |   100    |   More streams merged at once while sorting files.
 |  mapreduce.reduce.shuffle.parallelcopies   |   50   |    Higher number of parallel copies run by reduces to fetch outputs from very large number of maps.
 
####   Configurations for MapReduce JobHistory Server:
| Parameter  |   Value  |   Notes |
| :-------- |: -------- |: -------- |
   |    mapreduce.jobhistory.address     |      MapReduce JobHistory Server host:port   |        Default port is 10020.
   |    mapreduce.jobhistory.webapp.address    |       MapReduce JobHistory Server Web UI host:port     |      Default port is 19888.
   |    mapreduce.jobhistory.intermediate-done-dir   |        /mr-history/tmp     |      Directory where history files are written by MapReduce jobs.
   |    mapreduce.jobhistory.done-dir   |      /mr-history/done        |     Directory where history files are managed by the MR JobHistory Server.
 
###    XML建议配置项
所有的Xml配置文件项太多，很多都是默认有值的，更多见[core-default.xml](http://hadoop.apache.org/docs/r2.6.5/hadoop-project-dist/hadoop-common/core-default.xml)，[hdfs-default.xml](http://hadoop.apache.org/docs/r2.6.5/hadoop-project-dist/hadoop-hdfs/hdfs-default.xml)，[mapred-default.xml](http://hadoop.apache.org/docs/r2.6.5/hadoop-mapreduce-client/hadoop-mapreduce-client-core/mapred-default.xml)，[yarn-default.xml](http://hadoop.apache.org/docs/r2.6.5/hadoop-yarn/hadoop-yarn-common/yarn-default.xml)。搭建集群建议配置项如下：
 
- core-site.xml
   - fs.defaultFS
- hdfs-site.xml
   - dfs.replication
   - dfs.namenode.name.dir
   - dfs.datanode.data.dir
- yarn-site.xml
    - yarn.resourcemanager.address
    - yarn.nodemanager.aux-services
- mapred-site.xml
    - mapreduce.framework.name

**本次搭建时**：`dfs.replication配置副本数量为2，因为就两台虚拟机hado87和hado88，副本数多于2并没什么意义；dfs.namenode.name.dir设置为/opt/hadoop-2.6.3/data/namenode，dfs.datanode.data.dir设置为/opt/hadoop-2.6.3/data/datanode，提前新建这两个目录；fs.defaultFS值为hdfs://hado88:9000；yarn.resourcemanager.address为hado88，yarn.nodemanager.aux-services和mapreduce.framework.name为前文表格中的对应值。`


##    slaves配置文件
 etc/hadoop目录下名为slaves的文件配置DataNode节点。将所有需要启动DataNode的节点IP或者主机名写到slaves文件中。编辑slaves如下:
```
hado88
hado87
```

##   部署
将所有配置文件配置好之后，将hadoop-2.6.5目录拷贝到需要安装的节点上。**保证hadoop-2.6.5在每个节点路劲相同**。如在hado87上配置好/opt/hadoop-2.6.5，将hadoop-2.6.5拷贝到hado88节点的/opt/。
```
$ scp -r /opt/hadoop-2.6.5 root@hado88:/opt/
```

##   启动集群
###   格式化
首先格式化hdfs文件系统，相当于初始化。执行命令`./bin/hdfs namenode -format`。**此步骤为必须的步骤，不执行则无法启动datanode进程**
###   启动服务
启动集群的服务。执行命令`./sbin/start-all.sh`。翻阅start-all.sh代码，发现其实是先后执行`./sbin/start-dfs.sh` 和`./sbin/start-yarn.sh`

##   完善
此时，集群搭建好了。将hadoop的命令加入到用户的环境变量中，方便使用。编辑    /etc/profile 顶部添加：
```
export HADOOP_HOME=/opt/hadoop-2.6.5
export PATH=$PATH:$HADOOP_HOME/bin:$HADOOP_HOME/sbin
```
再重新刷新一下用户环境变量
```
$ source /etc/profile
```

测试hdfs
```
$ hdfs dfs -put test.txt  /
17/07/21 15:48:24 WARN util.NativeCodeLoader: Unable to load native-hadoop library for your platform... using builtin-java classes where applicable
$ hdfs dfs -ls /
17/07/21 15:48:47 WARN util.NativeCodeLoader: Unable to load native-hadoop library for your platform... using builtin-java classes where applicable
Found 1 items
-rw-r--r--   2 root supergroup          0 2017-07-21 15:48 /test.txt
```















 
 