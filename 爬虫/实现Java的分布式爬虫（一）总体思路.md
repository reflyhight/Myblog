
title: 实现Java的分布式爬虫（一）总体设计
tags: 
	- java
	- spider
	- webmagic
	- 任务
	- 线程
	- 进程

categories:
	- 爬虫
------------



###    爬虫核心问题

目前我司爬虫存在的主要问题有以下，最核心的问题有：仅支持单节点任务，爬虫任务无可视化管理，没有对爬虫的状态进行监控。
<img src="http://ou9e0q35h.bkt.clouddn.com/2018-01-20_121019.png" style="width:600px">

###   目标
想设计一套爬虫系统，既然确定了要做，那先定一下目标，这就开始，出发
- 分布式爬虫。借鉴已有的webmagic和crapy的设计
- Java实现
- 爬虫任务管理可视化
- 爬虫日志管理可视化
- 数据管理可视化
 

###    爬虫任务的设计
####    Webmagic架构
简单爬由三个部分构成：下载，解析，持久化（保存数据落地）。复杂点儿的包括了URL调度器（实现了生产者-消费者模式和URL去重），多线程爬取。目前已经实现了以上功能的爬虫框架有Java界的Webmagic，Python界的scrapy （python多线程支持我木知？？）

<img src="http://ou9e0q35h.bkt.clouddn.com/2018-01-15_140752.png" style="width:600px">
 
####    分布式的爬虫进程
如果从头造轮子的话，短期内效果肯定不会太好，webmagic经历了五年时间的发展，两千多次代码的commit，在Downloader下载器，PageProcessor解析器等组件方面都做得比较完善了。也是目前我司使用的爬虫，爬虫队列默认采用QueueScheduler，内部LinkedBlockingQueue的实现，基于JDK的队列数据结构。而Python的scrapy有分布式的实现：scrapy-redis，采用redis做共享队列，实现了简单的分布式爬虫。webmagic-extension有RedisScheduler，可以实现简单的分布式爬取功能，真正使用的话，还需要修改一些细节。

<img src="http://ou9e0q35h.bkt.clouddn.com/2018-01-15_143940.png" style="width:600px">

####    监控
Webmagic实现了基于JMX的[简单监控](http://webmagic.io/docs/zh/posts/ch4-basic-page-processor/monitor.html)，可以监控正在执行的爬虫任务（爬取错误页面数，爬取正确页面数，爬取线程数，爬虫状态，一共爬取的次数，爬取的开始时间等），但这个监控功能相对较弱，作为一个后期监控体系的启发。
<img src="http://ou9e0q35h.bkt.clouddn.com/2018-01-15_150331.png" style="width:600px">


####    其他
- 设置代理，webmagic也做了相关封装，详见[配置代理](http://webmagic.io/docs/zh/posts/ch4-basic-page-processor/proxy.html)
- 登录，模拟登录选择htmlunit和PhantomJs，到具体网站实现再说
- 验证码识别（目前使用过打码平台，这块具体到实现再说）
- 采集规则可配置性，这个作为后期考虑
- 解析规则可配置性，这个需求不是特别急迫的需求，暂不考虑，后期实现的话，会大大增加爬虫的灵活性，和大大提高开发效率


###    任务管理与分布式
####   分布式构思
借鉴于很多经典的分布式架构，如hadoop，storm，基础都是主从式的任务调度。查看了一些轻量级的任务调度框架，其中[xxl-job](http://www.xuxueli.com/xxl-job/#/)是一款支持cron表达式的任务调度框架。爬虫本身常常有采集周期的问题，所以将xxl-job设计思路结合到一起，有种为爱而生的感觉（😂）。这里明确一下任务的定义，平时一个采集程序为一个jar，jar就是一个简单任务，决定任务的唯一性是这个jar的main方法入口类。加上时间纬度，一个特定的cron表达式cronExp，加上任务的执行的节点nodes[]，加上每个节点启动jar的进程数processNum确定了唯一一个分布式的爬虫任务。总结起来一个分布式爬虫任务由：  xxx.class ，cronExp，nodes[]，processNum定义。

<img src="http://ou9e0q35h.bkt.clouddn.com/2018-01-16_103453.png" style="width:600px">
####    任务管理
基本上使用webmagic和很多第三方解决办法解决细节问题就能够采集95%的页面了。除了爬虫任务本身，或许爬虫任务的管理才是最让人头痛的事，我们部门跑着N个爬虫的Jar，前后同事离职后，都不知道哪是哪了，反正大部分网站页面也不咋变，接受别人的代码修改有时候还不如自己重写，看不规范的代码是很痛苦的。除了爬虫任务，其他任务也一样，都需要合理管理的（脚本，离线任务等）。借鉴于xxl-job对任务的监控，实现了任务的增删改查，进程管理，这些功能在主节点实现会非常的不错。除此之外爬虫往往会有漏采数据的情况，即便不漏采，或许某个临时需求，让你采集某个人的信息，如果你也能通过这个平台补采单条数据，就很完善了。
 
**任务管理（增，删，改）**
- 增加一个Job，配置执行的周期，执行节点，每个节点启动的Job实例数
- 逻辑删除，暂停该任务
- 修改：修改执行周期，执行节点，Job实例数
- 任务调度结果（成功，失败）
 
**进程管理**
- 进程状态，spider进程是否正常运行
- 强制停止运行中的进程

###   数据管理
采集部门采集回来的数据不应该直接落地到应用库，直接落地到某个特定的应用库，对于做这个特定的应用是省事了，但如果又有新的应用加入，而加入的应用也会用到之前采集的数据，这样就繁琐了。采集的数据可以存放到一个统一规范的库中，按照一定的策略，建立存储体系，你可以把它叫做原始层数据仓库，当应用需要数据时，从原始层数据仓库提取，这里如果写个抽取工具，中间件，就很完善了。

###    日志管理
完善的项目都有日志这块，排错最快的方法就是查看日志。像爬虫基本上又是多线程的东西，仅仅只看进程的状态，都无法全面了解到爬虫的健康状况，所以做一个日志管理系统也是非常有必要的。可以监控到爬虫任务的运行日志，可以检索爬虫日志，就很完善了。