title: 快速搭建CDH集群
tags: 
	- Linux
	- Hadoop
	- Spark
	- CDH

categories:
	- 大数据
-------------------


简介：CDH是Cloudera Distributed Hadoop 的缩写。是Cloudera公司 出的Hadoop集成分布式系统，自带管理平台CM（Cloudera Manager）,可以方便的管理集群，并方便的添加大数据组件。不仅仅支持传统大数据框架Hadoop生态系统，还支持Spark相关，是目前最流行的大数据集成系统。

系统：centos 6.5 内存4G 磁盘20G

###    准备
在虚拟机中新建3台内存为4G的Centos 6.5的系统。主机名分别为cdha，cdhb，cdhc
分别设置静态IP，虚拟机采用NAT模式

####    设置主机映射    `【所有节点】`
```
# vim /etc/
192.168.33.13   cdha    
192.168.33.14   cdhb
192.168.33.15   cdhc
```
以cdha为cm安装节点，以下所指`cm节点`均为192.168.33.13 节点


####   设置主机名    `【所有节点】`
```
cdha主机
# vim /etc/sysconfig/network
NETWORKING=yes
HOSTNAME=cdha
# hostname cdhc  

其他主机同理
```

####   关闭防火墙   `【所有节点】`
正确操作是管理防火墙，这里为了简便，直接关闭，到时候再管理也行
```
# service iptables stop 
iptables: Setting chains to policy ACCEPT: nat mangle filte[  OK  ]
iptables: Flushing firewall rules:                         [  OK  ]
iptables: Unloading modules:                               [  OK  ]
# chkconfig iptables off
```
如果防火墙不能关闭，请选择防火墙管理策略 [相关链接1](http://www.cloudera.com/documentation/cdh/5-0-x/CDH5-Installation-Guide/cdh5ig_ports_cdh5.html)、[相关链接2](http://www.cloudera.com/documentation/manager/5-0-x/Cloudera-Manager-Installation-Guide/cm5ig_ports_cm.html)




####   关闭selinux    `【cm节点】`
只需在安装cm的主机（执行cloudera-manager-installer.bin）上执行关闭selinux的操作，不需要所有主机都执行。
编辑selinux文件，注释 SELINUX=enforcing，添加  SELINUX=disabled
编辑完成之后，重启该主机生效
```
# vi /etc/sysconfig/selinux
# This file controls the state of SELinux on the system.
# SELINUX= can take one of these three values:
#     enforcing - SELinux security policy is enforced.
#     permissive - SELinux prints warnings instead of enforcing.
#     disabled - No SELinux policy is loaded.
#SELINUX=enforcing
# SELINUXTYPE= can take one of these two values:
#     targeted - Targeted processes are protected,
#     mls - Multi Level Security protection.
SELINUXTYPE=targeted
SELINUX=disabled
```
####    安装mysql数据库     `【cm节点】`
安装部分略，以下为创建数据库，授权,设置登录密码，备用，用做CM的元数据库
```
mysql> create database cmuser;    
Query OK, 1 row affected (0.00 sec)

mysql> grant all privileges on cmuser.* to 'cmadmin'@'%' identified by 'cmadmin' ;               
Query OK, 0 rows affected (0.00 sec)

mysql> grant all privileges on cmuser.* to 'cmadmin'@'localhost' identified by 'cmadmin' ; 
Query OK, 0 rows affected (0.00 sec)

mysql> flush privileges;
Query OK, 0 rows affected (0.00 sec)
```
####   同步集群时间   

安装 ntp: yum install ntp 。centos6.5可能自带     `【所有节点】`
在cm节点配置ntp服务端，编辑 /etc/ntp.conf，在末尾加上如下两行   `【cm节点】`
```
# vim /etc/ntp.conf
server 127.127.1.0
Fudge 127.127.1.0 stratum 10
```
完成之后重启
```
# service ntpd restart
```

其他节点需要执行如下，和cm主机节点时间保持同步     `【除CM之外所有节点】`
```
# ntpdate cdha
17 May 07:49:33 ntpdate[3996]: adjust time server 192.168.33.13 offset -0.160912 sec
```
将同步操作加入到定时任务中     `【除CM之外所有节点】`
```
# crontab -e
#集群时间同步，没两小时同步一次
* */2 * * * /usr/sbin/ntpdate cdha  >>/root/logs/ntpdate/`date +%Y%m%d`.log  2>&1
```
####    安装JDK   `【所有节点】`
```
# mkdir /opt/zips
# mv jdk-7u67-linux-x64.tar.gz  /opt/zips
# cd /opt/zips/
# tar -zxvf jdk-7u67-linux-x64.tar.gz
# mv jdk1.7.0_67/ /opt/
# vim /etc/profile
export JAVA_HOME=/opt/jdk1.7.0_67
export PATH=$JAVA_HOME/bin:$PATH
# . /etc/profile
# java -version
java version "1.7.0_67"
```
####    ssh免登录    `【所有节点】`
首先生成pub ssh key
```
# ssh-keygen -t rsa
#一路回车
```
每个节点都生成pub ssh key后再将pub key追加到各个节点的~/.ssh/里，执行如下
```
# ssh-copy-id root@cdha 
# ssh-copy-id root@cdhb 
# ssh-copy-id root@cdhc 
```
每个节点都测试一下ssh登录，看是否能
```
# ssh cdha //登录后 ctrl+d退出登录
# ssh cdhb //登录后 ctrl+d退出登录
# ssh cdhc //登录后 ctrl+d退出登录
```
####   配置网络    `【所有节点】`
虽然说重要的大部分文件都是通过离线下载完成（迅雷下载），但安装过程中，有少部分依赖需要在线下载，所以需要保证所有节点能连接到外网。VMware虚拟机NAT模式下通常执行如下操作
```
# echo "nameserver 8.8.8.8" >>/etc/resolv.conf   //配置DNS解析主机
# ping www.baidu.com   //检测网络是否联通
```


###    开始安装
是不是已经被上面的繁琐步骤吓到，后面安装可能还会遇到各种乱七八糟的问题

####    安装列表
#####   CM  
下载连接[http://archive.cloudera.com/cm5/redhat/6/x86_64/cm/5.11.0/RPMS/x86_64/](http://archive.cloudera.com/cm5/redhat/6/x86_64/cm/5.11.0/RPMS/x86_64/) 建议使用迅雷，下载全部连接，再上传linux，直接下载略费劲
- cloudera-manager-agent-5.11.0-1.cm5110.p0.101.el6.x86_64.rpm    2017-04-12 22:14    9.1M    
- cloudera-manager-daemons-5.11.0-1.cm5110.p0.101.el6.x86_64.rpm    2017-04-12 22:14    635M    
- cloudera-manager-server-5.11.0-1.cm5110.p0.101.el6.x86_64.rpm    2017-04-12 22:14    8.5K    
- cloudera-manager-server-db-2-5.11.0-1.cm5110.p0.101.el6.x86_64.rpm    2017-04-12 22:14    10K    
- enterprise-debuginfo-5.11.0-1.cm5110.p0.101.el6.x86_64.rpm    2017-04-12 22:14    30M    
- jdk-6u31-linux-amd64.rpm    2017-04-12 22:05    68M    
- oracle-j2sdk1.7-1.7.0+update67-1.x86_64.rpm

#####   CDH    
下载连接[http://archive.cloudera.com/cdh5/parcels/5.11.0/](http://archive.cloudera.com/cdh5/parcels/5.11.0/) 也建议使用迅雷下载
- CDH-5.11.0-1.cdh5.11.0.p0.34-el6.parcel
- CDH-5.11.0-1.cdh5.11.0.p0.34-el6.parcel.sha1
- manifest.json

#####   installer     
较小，wget直接下载到cdha上
```
# wget    http://archive.cloudera.com/cm5/installer/5.11.0/cloudera-manager-installer.bin
```

下载完成之后，整理目录得到如下
```
# ll /opt
total 528
drwxr-xr-x  2 root root   4096 May 17 03:30 cdh5.11
-rw-r--r--  1 root root 519689 Mar 15 17:00 cloudera-manager-installer.bin
drwxr-xr-x  2 root root   4096 May 17 03:36 cm5.11
...省略更多
```
其中    /opt/cm5.11中存放2.1.1中下载列表中的所有下载的文件，/opt/cdh5.11中存放2.1.2中下载列表中的所有下载的文件
####   安装CM
#####   本地安装所需rpm
CM节点安装如下     `【CM节点】`
```
# cd /opt/cm5.11/
# yum localinstall --nogpgcheck *.rpm   //安装过程顺畅
```
其他节点安装如下     `【除CM之外所有节点】`
```
# yum localinstall --nogpgcheck cloudera-manager-daemons-5.11.0-1.cm5110.p0.101.el6.x86_64.rpm
# yum localinstall --nogpgcheck cloudera-manager-agent-5.11.0-1.cm5110.p0.101.el6.x86_64.rpm
# yum localinstall --nogpgcheck jdk-6u31-linux-amd64.rpm
# yum localinstall --nogpgcheck oracle-j2sdk1.7-1.7.0+update67-1.x86_64.rpm
```

#####   安装cloudera-manager-installer.bin    `【CM节点】`
```
# cd /opt/
# chmod u+x cloudera-manager-installer.bin
# ./cloudera-manager-installer.bin
```
此时提示你需要删除/etc/cloudera-scm-server/db.properties，那就删除之后继续执行cloudera-manager-installer.bin
```
# rm -rf /etc/cloudera-scm-server/db.properties 
# ./cloudera-manager-installer.bin
```
如果一切顺利，一路next，最后提示访问7180端口并用user:admin,pwd:admin登录，就安装完成了。
![](http://ou9e0q35h.bkt.clouddn.com/a3cbc6d852260bd61b1ceff94b3d0bed.png) 

如果报错，请查看`/var/log/cloudera-manager-installer`目录下的安装日志，`/var/log/cloudera-manager-installer`中有多个.log的安装日志，分别对应clouderamanager-installer.bin安装的不同阶段的日志

#####   启动   cloudera-scm-server
默认情况下cloudera-scm-server在installer安装完成之后就自动启动了。查看一下状态
```
# service cloudera-scm-server status
cloudera-scm-server (pid  3895) is running...
```
如果启动失败，可再次启动`service cloudera-scm-server start`，并同时开一个终端通过   
`tail -f /var/log/cloudera-scm-server/cloudera-scm-server.log`监控启动日志，排查。`service cloudera-scm-server`启动整个过程较慢，也可以通过监控日志查看是否已经启动完成。启动完成之后通过浏览器访问CM，地址`http://cdha:7180/`
![](http://ou9e0q35h.bkt.clouddn.com/690d8abe8531704e5965a665a26ed972.png) 

#####   修改数据库
下载mysql驱动
```
# wget http://central.maven.org/maven2/mysql/mysql-connector-java/5.1.35/mysql-connector-java-5.1.35.jar
# mv mysql-connector-java-5.1.35.jar  /usr/share/cmf/lib
```
修改db.properties，注释掉默认配置，加上以下关于mysql的配置。其中db.name,db.user,db.password对应前面1.5中所述
```
# vim /etc/cloudera-scm-server/db.properties
#com.cloudera.cmf.db.type=postgresql
#com.cloudera.cmf.db.host=localhost:7432
#com.cloudera.cmf.db.name=scm
#com.cloudera.cmf.db.user=scm
#com.cloudera.cmf.db.password=cyXzdA0mY0
#com.cloudera.cmf.db.setupType=EMBEDDED
com.cloudera.cmf.db.type=mysql
com.cloudera.cmf.db.host=localhost:3306
com.cloudera.cmf.db.name=cmuser
com.cloudera.cmf.db.user=cmadmin
com.cloudera.cmf.db.password=cmadmin
```
####   cdh安装准备
#####   移动parcel
将cdh中的的文件拷贝到/opt/cloudera/parcel-repo，并重命名CDH-5.11.0-1.cdh5.11.0.p0.34-el6.parcel.sha1 为CDH-5.11.0-1.cdh5.11.0.p0.34-el6.parcel.sha
```
# cp /opt/cdh5.11/*  /opt/cloudera/parcel-repo
# cd /opt/cloudera/parcel-repo
# ll
total 1485432
-rw-r--r-- 1 root root 1520997979 May 17 04:30 CDH-5.11.0-1.cdh5.11.0.p0.34-el6.parcel
-rw-r--r-- 1 root root         41 May 17 04:30 CDH-5.11.0-1.cdh5.11.0.p0.34-el6.parcel.sha1
-rw-r--r-- 1 root root      72602 May 17 04:30 manifest.json
# mv CDH-5.11.0-1.cdh5.11.0.p0.34-el6.parcel.sha1 CDH-5.11.0-1.cdh5.11.0.p0.34-el6.parcel.sha
```

#####   重启 cloudera-scm-server 
```
# service cloudera-scm-server restart
```
`你可能想问，为啥要重启cloudera-scm-server`，搞得这么麻烦。这里主要是促使mysql成为cm的元数据库，并使得/opt/cloudera/parcel-repo的cdh安装文件被cm识别。如果你有在启动过程中监控元数据库cmuser中的变化，你会发现CM会往元数据库中加入很多表，自动完成初始化功能。

####   安装CDH
整个安装过程，都是通过CM系统配置安装的，首先在此通过浏览器访问`http://cdha:7180/`，通过user:admin，pwd:admin登录到CM系统中

![](http://ou9e0q35h.bkt.clouddn.com/6e981e321e7a78651b516d81ab404bfd.png)
![](http://ou9e0q35h.bkt.clouddn.com/2b9504d14ea9659e8664bf3eeb95945a.png)
![](http://ou9e0q35h.bkt.clouddn.com/75c733a6c6272cef685574feab718549.png)
 #####   搜索主机
搜索局域网中的主机名或者IP，选择要安装CDH的主机
![](http://ou9e0q35h.bkt.clouddn.com/809c9fb89bee379315670694835d4732.png)
![](http://ou9e0q35h.bkt.clouddn.com/fa879092f869a1c20e263476d16b69bf.png)
#####   选择版本
选择已经在/opt/cloudera/parcel中放置的cdh的版本
![](http://ou9e0q35h.bkt.clouddn.com/27d097b535845549caf67fe8fc8e5649.png)
![](http://ou9e0q35h.bkt.clouddn.com/e9b0351d4daeb954167ed2186d37b45b.png)
![](http://ou9e0q35h.bkt.clouddn.com/9a5b8db6070b38ca55f19738fc823b77.png)
![](http://ou9e0q35h.bkt.clouddn.com/ca0f7c2574e835b3cfd72642ad209995.png)


#####   安装angent
前文说过的，手动在每个其他节点都已经安装过`cloudera-manager-daemons，cloudera-manager-agent，jdk-6u31-linux-amd64，oracle-j2sdk1.7-1.7.0+update67-1`,本地如果已经安装过这几个包，那么这个过程会比较快，如果没安装，默认会在线下载，再安装，这样就会比较慢。
![](http://ou9e0q35h.bkt.clouddn.com/97faba4c70a927abc64c89770b5063d6.png)
![](http://ou9e0q35h.bkt.clouddn.com/89fcd843d99b0014fb42a2b080777b7e.png)
![](http://ou9e0q35h.bkt.clouddn.com/bbc5e925453222c95d95447800508bf9.png)
![](http://ou9e0q35h.bkt.clouddn.com/cc56428960ee68cd5785d2064dd2fc53.png)
![](http://ou9e0q35h.bkt.clouddn.com/49e78bbd69b708ce289e33873549c3a4.png)
#####   CDH安装过程
这里才是cdh真正的安装过程，之前都是CM安装，和CDH准备。前文`2.3 cdh安装准备`中提到将parcel拷贝到指定目录的操作，如果上文中的操作无误，则这里下载将瞬间完成100%，如果没有瞬间完成，说明前文操作出了问题。`由于作者在装CDH时，机器配置有限，前文中安装的3个节点失败，后重新安装两个节点，固后面截图中显示的是2/2，大家在安装的时候也可以采用这种方式，先安装两个或者一个节点，等CM和CDH最简单的搭建完成后，再通过CM添像集群中添加更多节点，真实的服务器不用这样操作，这只是出于大部分学习者使用虚拟机的原因`
![](http://ou9e0q35h.bkt.clouddn.com/bbc5e925453222c95d95447800508bf9.png)
![](http://ou9e0q35h.bkt.clouddn.com/cc56428960ee68cd5785d2064dd2fc53.png)
![](http://ou9e0q35h.bkt.clouddn.com/49e78bbd69b708ce289e33873549c3a4.png)
#####   选择要装的服务
通常可以选择系统推荐的集中服务搭配，选择"核心 Hadoop“,这里考虑到本人用的电脑，虚拟机各方面性能，就自定义，先自定义仅安装Hadoop（yarn,hdfs）+Hive
![](http://ou9e0q35h.bkt.clouddn.com/cc0f712e9231961778aa235ced6132e7.png)
![](http://ou9e0q35h.bkt.clouddn.com/1fd7617a21a09204f17c9d0ef17f3ab5.png)

#####   分配服务
可以选择合理将服务分配到不同的节点上去。默认可以不安装的，这里选择先不安装。ZooKeeper和HiveServer2和DataNode选择安装到每个节点

![](http://ou9e0q35h.bkt.clouddn.com/40ea4b142133014e321bb5d6fdb1602c.png)
选择服务安装到哪台具体的机器上，如下图所示。
![](http://ou9e0q35h.bkt.clouddn.com/def0cb4ca1523dbd5e5b6dea10877763.png)

#####   元数据库设置
同前文    `1.5安装mysql数据库`段中讲的方法相同，先创建数据库，并授权用户。然后将数据库名，和用户填写到如下框中。测试连接ok后，再`继续`下一步。
```
mysql> create database hivewarehouse;
Query OK, 1 row affected (0.00 sec)

mysql> grant all privileges on hivewarehouse.* to 'hiveadmin'@'%' identified by 'hiveadmin' ;                           
Query OK, 0 rows affected (0.00 sec)

mysql> grant all privileges on hivewarehouse.* to 'hiveadmin'@'localhost' identified by 'hiveadmin' ; 
Query OK, 0 rows affected (0.00 sec)

mysql> flush privileges;                 
Query OK, 0 rows affected (0.01 sec)

mysql> create database rpwarehouse;
Query OK, 1 row affected (0.00 sec)

mysql> grant all privileges on rpwarehouse.* to 'rpadmin'@'localhost' identified by 'rpadmin' ;                     
Query OK, 0 rows affected (0.00 sec)

mysql> grant all privileges on rpwarehouse.* to 'rpadmin'@'%' identified by 'rpadmin' ;         
Query OK, 0 rows affected (0.00 sec)

mysql> flush privileges;                 
Query OK, 0 rows affected (0.00 sec)
```
![](http://ou9e0q35h.bkt.clouddn.com/575f98103484d033f3ae3e42f234ec0a.png)

#####    服务安装
具体的服务安装与运行，其中采用mysql作为元数据库的Hive需要将mysql驱动mysql-connector-java-5.1.35.jar（参考前文2.2.3）放入指定位置`/opt/cloudera/parcels/CDH-5.11.0-1.cdh5.11.0.p0.34/lib/hive/lib`，每个安装了HiveServer2的节点都需要加入驱动。
![](http://ou9e0q35h.bkt.clouddn.com/b617d900a2dffc032714706c983b52da.png)
![](http://ou9e0q35h.bkt.clouddn.com/06f2debb4b3e34cbc12ab240b2530105.png)
![](http://ou9e0q35h.bkt.clouddn.com/d18537051cb6781a1c9823551f13d906.png)
![](http://ou9e0q35h.bkt.clouddn.com/6ebb9b8364c9e8e28b90ee141c41bf29.png)



###   各种坑
####   低版本
#####   具体经过
因为我们公司用的是5.6.1的，于是我第一次也选择cm5.6.1，下载如下rpm
```
- cloudera-manager-agent-5.6.1-1.cm561.p0.3.el6.x86_64.rpm   
- cloudera-manager-daemons-5.6.1-1.cm561.p0.3.el6.x86_64.rpm  
- cloudera-manager-server-5.6.1-1.cm561.p0.3.el6.x86_64.rpm  
- cloudera-manager-server-db-2-5.6.1-1.cm561.p0.3.el6.x86_64.rpm   
- enterprise-debuginfo-5.6.1-1.cm561.p0.3.el6.x86_64.rpm    
- jdk-6u31-linux-amd64.rpm    2016-05-19 22:07  
- oracle-j2sdk1.7-1.7.0+update67-1.x86_64.rpm
```
通过wget下载cm5.6.1版本的installer.bin
```
http://archive.cloudera.com/cm5/installer/5.6.1/cloudera-manager-installer.bin
```
如上文中介绍的那种方式，先将rpm安装好，安装cloudera-manager-installer.bin,详见前文2.2段所述，此时安装到cloudera manager server直接卡住了。
![](http://ou9e0q35h.bkt.clouddn.com/12967a69566cd3c31e6c33609c5583e4.png)

 后来检查安装日志。如上文所提到的，安装cloudera-manager-installer.bin时，日志文件都存在于`/var/log/cloudera-manager-installer/`目录下。新建一个终端通过`tail -f /var/log/cloudera-manager-installer/3.install-cloudera-manager-server.log`监控安装日志

![](http://ou9e0q35h.bkt.clouddn.com/01ed5778b2c8cddab36b3eeacfb4e55c.png)

如上，发现installer程序没把安装的5.6.1版本的rpm放在眼里，直接去升级5.11.0版本了，此时又需要下载644兆，所以直接卡在这里了。此下载过程缓慢，并且经常超时。所以完全就放弃了5.6.1，装当前最高版本的cm5.11.0

#####   踩坑总结
如果按照前文`二、`段中的安装方式，直接安装cm5.11.0，不然cloudera-manager-installer.bin会自动在线升级rpm，缓慢并容易中断

####   高版本&主机名
#####   具体经过
改用最新版的cm后，自然是要装最高版本的cdh咯（第一想法），出于怕版本不匹配的想法，同样就用了cdh5.11.0。就在cdh快安装好的时候，出现`无法接收 Agent 发出的检测号`的错误。看到网上所说的解决方案基本都是以下两步骤`1.启动cloudera-scm-agent服务`，`2.检查网络配置`

![](http://ou9e0q35h.bkt.clouddn.com/7ccf1063cc1a953bba05a6d6db8345e2.png)
我任选了一个主机，启动cloudera-scm-agent服务。如下状况。
```
# service cloudera-scm-agent start
Starting cloudera-scm-agent:                               [  OK  ]
# service cloudera-scm-agent status
cloudera-scm-agent is stopped
```
这就很纠结了， cloudera-scm-agent服务启动瞬间就挂掉了。然后再通过`tail -f /var/log/cloudera-scm-agent/cloudera-scm-agent.out`监控。
```
# tail -f /var/log/cloudera-scm-agent/cloudera-scm-agent.out
[18/May/2017 07:00:37 +0000] 6801 MainThread agent        INFO     SCM Agent Version: 5.11.0
[18/May/2017 07:00:37 +0000] 6801 MainThread agent        WARNING  Expected mode 0751 for /var/run/cloudera-scm-agent but was 0755
[18/May/2017 07:00:37 +0000] 6801 MainThread agent        INFO     Re-using pre-existing directory: /var/run/cloudera-scm-agent
[18/May/2017 07:01:35 +0000] 6866 MainThread agent        INFO     SCM Agent Version: 5.11.0
[18/May/2017 07:01:35 +0000] 6866 MainThread agent        WARNING  Expected mode 0751 for /var/run/cloudera-scm-agent but was 0755
[18/May/2017 07:01:35 +0000] 6866 MainThread agent        INFO     Re-using pre-existing directory: /var/run/cloudera-scm-agent
```
完全找不到错误。网上有说查看`/var/log/cloudera-scm-agent/cloudera-scm-agent.log`，但此时目录`/var/log/cloudera-scm-agent/`下根本没有`cloudera-scm-agent.log`文件。还以为自己新建`cloudera-scm-agent.log`，就能看到里面有什么报错信息，结果里面硬是啥都没有。
通过` tail -f /var/log/messages`查看进程日志
```
May 18 07:09:02 cdh_a abrt: detected unhandled Python exception in '/usr/lib64/cmf/agent/build/env/bin/cmf-agent'
May 18 07:09:02 cdh_a abrtd: New client connected
May 18 07:09:02 cdh_a abrtd: Directory 'pyhook-2017-05-18-07:09:02-7004' creation detected
May 18 07:09:02 cdh_a abrt-server[7010]: Saved Python crash dump of pid 7004 to /var/spool/abrt/pyhook-2017-05-18-07:09:02-7004
May 18 07:09:02 cdh_a abrtd: Package 'cloudera-manager-agent' isn't signed with proper key
May 18 07:09:02 cdh_a abrtd: 'post-create' on '/var/spool/abrt/pyhook-2017-05-18-07:09:02-7004' exited with 1
May 18 07:09:02 cdh_a abrtd: Deleting problem directory '/var/spool/abrt/pyhook-2017-05-18-07:09:02-7004'
```
折腾了许久，人都是奔溃的。又想，是不是Python版本过低，又手动去升级Python到2.7，又去改abrtd的配置啥的，真是没招了。
**就想是不是cloudera-manager-agent版本太高**，重新卸载了稿本的cloudera-manager-agent，单独安装上了5.6.1的cloudera-manager-agent，此时再监控日志
```
# tail -f /var/log/cloudera-scm-agent/cloudera-scm-agent.out
Traceback (most recent call last):
  File "/usr/lib64/cmf/agent/src/cmf/agent.py", line 3512, in <module>
    main()
  File "/usr/lib64/cmf/agent/src/cmf/agent.py", line 3495, in main
    agent.configure()
  File "/usr/lib64/cmf/agent/src/cmf/agent.py", line 423, in configure
    raise Exception("Hostname is invalid; it contains an underscore character.")
```
`Hostname is invalid; it contains an underscore character.`报错信息是说主机名中含有特殊字符下划线，`underscore下划线的意思`。我原本用的主机名为cdh_a,cdh_b,cdh_c，修改了主机名，去掉下划线。再次启动agent,终于oh咯。但此时是5.6.1版的cloudera-manager-agent，怀疑5.11.0版的cloudera-manager-agent也是因为这个原因，又升级到5.11.0，此时5.11.0版的也启动正常了。
#####   踩坑总结
主机名最后用个普通的纯字母吧，反正是不能用下划线。这个坑，cm5.11.0真是完全找不出问题。


    




    