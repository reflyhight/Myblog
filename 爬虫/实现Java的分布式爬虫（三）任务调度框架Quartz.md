
title: 实现Java的分布式爬虫（三）任务调度框架Quartz.md
tags: 
	- java
	- 任务
	- 调度器
	- 触发器

categories:
	- 爬虫
------------


>本文参考学习了博客[路边飞~](https://www.cnblogs.com/drift-ice/p/3817269.html)、《精通spring4.x企业级开发》中的任务调度和异步执行器和[官方文档](http://www.quartz-scheduler.org/documentation/quartz-2.2.x/tutorials/)。

 
###    相关概念
Quartz把任务调度的过程抽象成了一些概念，最基本的三个元素是**Scheduler调度器**，**Trigger触发器**，**Job任务**。简单理解就是：**触发器**决定了任务的执行时间点，**调度器**在任务的执行时间点执行相应的**任务**（调度器内部维护了线程池去执行任务的）

**Scheduler    调度器**
Quartz也有调度器的概念，这里的调度器和webmagic中的调度器是两个东西。这里的Scheduler是任务的管理中心，具体也就是执行任务的容器，真正任务的发起者。
 
**Trigger   触发器**
如果从中文的语义来理解的话，可能容易让人误解。Trigger决定了执行的时间点，或者说任务的生命周期信息，它定义了任务的执行时间点。
 
**Job   任务**
任务就是具体的执行内容，需要用户实现Quartz中org.quartz.Job接口中的execute方法，方法的具体内容便是任务的具体内容。

**JobDetail    任务元信息**
JobDetail定义了任务的元信息。初学者会有点儿难以理解，有了Job，又来个JobDetail。Quartz在执行任务（Job）的时候都会新建一个Job实例，JobDetail承担job实例的信息的保存。

###   入门案例
使用2.3.0，加入pom依赖
```xml
<dependency>
    <groupId>org.quartz-scheduler</groupId>
    <artifactId>quartz</artifactId>
    <version>2.3.0</version>
</dependency>
<dependency>
    <groupId>org.quartz-scheduler</groupId>
    <artifactId>quartz-jobs</artifactId>
    <version>2.3.0</version>
</dependency>
```

Job的实现类
```java
import org.quartz.Job;
import org.quartz.JobDetail;
import org.quartz.JobExecutionContext;
import org.quartz.JobExecutionException;
 
public class SimpleJob implements Job{
    @Override
    public void execute(JobExecutionContext context) throws JobExecutionException {
        // TODO Auto-generated method stub
        System.out.println(context.getTrigger().getKey().getName());
    }
}
```
构造一个简单的调度过程
```java
import org.quartz.JobBuilder;
import org.quartz.JobDetail;
import org.quartz.Scheduler;
import org.quartz.SchedulerException;
import org.quartz.SimpleScheduleBuilder;
import org.quartz.Trigger;
import org.quartz.TriggerBuilder;
import org.quartz.impl.StdSchedulerFactory;
 
public class SimpleQuartzTest {
 
    public static void main(String[] args) {
 
        try {
            //调度器容器
            Scheduler scheduler = StdSchedulerFactory.getDefaultScheduler();
 
            //构建一个JobDetail
            JobDetail job  = JobBuilder.newJob(SimpleJob.class)
                    .withIdentity("simple", "g1")
                    .build();
 
            //构造schedulebuilder
            SimpleScheduleBuilder schedulebuilder = SimpleScheduleBuilder.simpleSchedule()
            .withIntervalInSeconds(1)
            .repeatForever();
 
            //构建一个触发器
            Trigger trigger = TriggerBuilder.newTrigger().withIdentity("simple", "g1")
                    .withSchedule(schedulebuilder)
                    .startNow()
                    .build();
 
             //加入这个调度
            scheduler.scheduleJob(job, trigger);
 
            //启动这个调度器
            scheduler.start();
 
        } catch (SchedulerException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        }
    }
}
```
该案例就是每隔一秒打印一下触发器的名字。接着来分析屡一下入门案例。

####   SchedulerFactory
通过SchedulerFactory构造Scheduler，SchedulerFactory是接口，有org.quartz.impl.StdSchedulerFactory，和org.quartz.impl.DirectSchedulerFactory两种实现
StdSchedulerFactory构造Scheduler的方法如下
```java
public static Scheduler getDefaultScheduler() throws SchedulerException {
        StdSchedulerFactory fact = new StdSchedulerFactory();
 
        return fact.getScheduler();
    }

    public Scheduler getScheduler() throws SchedulerException {
        if (cfg == null) {
            initialize();
        }
 
        SchedulerRepository schedRep = SchedulerRepository.getInstance();
 
        Scheduler sched = schedRep.lookup(getSchedulerName());
 
        if (sched != null) {
            if (sched.isShutdown()) {
                schedRep.remove(getSchedulerName());
            } else {
                return sched;
            }
        }
 
        sched = instantiate();
 
        return sched;
    }
```
####   Scheduler
由前面代码可以看到StdSchedulerFactory构造出Scheduler。而StdSchedulerFactory构造的Scheduler为StdScheduler，代码太多，不具体分析代码了，但你通过debug透视看出Scheduler的具体类型为StdScheduler。org.quartz.Scheduler作为接口，一共有三个具体的实现类
- org.quartz.impl.StdScheduler   标准
- org.quartz.impl.RemoteScheduler   实现远程方法调用的
- org.quartz.impl.RemoteMBeanScheduler  
    - org.quartz.ee.jmx.jboss.JBoss4RMIRemoteMBeanScheduler 



####   JobBuilder
org.quartz.JobBuilder类用于构造JobDetail实例的，来看看核心方法build()
```java
    public JobDetail build() {
 
        JobDetailImpl job = new JobDetailImpl();
 
        job.setJobClass(jobClass);
        job.setDescription(description);
        if(key == null)
            key = new JobKey(Key.createUniqueName(null), null);
        job.setKey(key);
        job.setDurability(durability);
        job.setRequestsRecovery(shouldRecover);
 
 
        if(!jobDataMap.isEmpty())
            job.setJobDataMap(jobDataMap);
 
        return job;
    }
```
从上面构造JobDetail实例时，需要指定job的class等信息

####   TriggerBuilder
再看看 org.quartz.TriggerBuilder 的对Trigger构造，方法名同样为build：
```
    public T build() {
 
        if(scheduleBuilder == null)
            scheduleBuilder = SimpleScheduleBuilder.simpleSchedule();
        MutableTrigger trig = scheduleBuilder.build();
 
        trig.setCalendarName(calendarName);
        trig.setDescription(description);
        trig.setStartTime(startTime);
        trig.setEndTime(endTime);
        if(key == null)
            key = new TriggerKey(Key.createUniqueName(null), null);
        trig.setKey(key);
        if(jobKey != null)
            trig.setJobKey(jobKey);
        trig.setPriority(priority);
 
        if(!jobDataMap.isEmpty())
            trig.setJobDataMap(jobDataMap);
 
        return (T) trig;
    }
```
TriggerBuilder有属性scheduleBuilder，用户如果不设置scheduleBuilder，那么默认会被设置为SimpleScheduleBuilder，其中**MutableTrigger trig = scheduleBuilder.build()**，我们可以发现，构造Trigger其实是scheduleBuilder😵。
####   ScheduleBuilder
org.quartz.ScheduleBuilder<T extends Trigger>作为接口，有四个实现类分别为
- org.quartz.CalendarIntervalScheduleBuilder
- org.quartz.CronScheduleBuilder
- org.quartz.DailyTimeIntervalScheduleBuilder
- org.quartz.SimpleScheduleBuilder


###   构造触发器Trigger
通过上面的简单案例的思路，不难看出Quartz的复杂性在于Trigger，Trigger实际上是由ScheduleBuilder构造出来的，分别有四种ScheduleBuilder构造出对应的四种Trigger

####   SimpleScheduleBuilder && SimpleTrigger
入门案例中我们已经使用过，再来分析一下，SimpleScheduleBuilder中拥有三个属性
- interval:long    执行间隔
- repeatCount:int    重复次数
- misfireInstruction:int     MisFire策略（错过触发策略）

再看看SimpleScheduleBuilder的build方法
```
    public MutableTrigger build() {
 
        SimpleTriggerImpl st = new SimpleTriggerImpl();
        st.setRepeatInterval(interval);
        st.setRepeatCount(repeatCount);
        st.setMisfireInstruction(misfireInstruction);
 
        return st;
    }
```
SimpleScheduleBuilder能构造的属性也就只有三个了，其他的构造其实是TriggerBuilder做的，但最终构造的对象还是org.quartz.impl.triggers.SimpleTriggerImpl，SimpleTriggerImpl中的repeatInterval，repeatCount，misfireInstruction由SimpleScheduleBuilder中的三个属性构造而来。SimpleTriggerImpl实现了SimpleTrigger


 
####   CronScheduleBuilder    &&    CronTrigger
CronScheduleBuilder是最常用的ScheduleBuilder了，看名字就知道了，它支持强大的Cron表达式，Cron表达式参考[cron表达式](http://blog.mxjhaima.com/2017/07/03/%E7%BB%BC%E5%90%88/cron%E8%A1%A8%E8%BE%BE%E5%BC%8F/)，CronScheduleBuilder中有两个可以定义的属性
- cronExpression:org.quartz.CronExpression    cron表达式对象
- misfireInstruction:int    MisFire策略（错过触发策略）

创建CronScheduleBuilder对象有如下：
- protected CronScheduleBuilder(CronExpression cronExpression) 构造方法
- public static CronScheduleBuilder cronSchedule(String cronExpression)
- public static CronScheduleBuilder cronScheduleNonvalidatedExpression
- public static CronScheduleBuilder cronSchedule(CronExpression cronExpression)

看看CronScheduleBuilder的build方法
```
public MutableTrigger build() {
 
        CronTriggerImpl ct = new CronTriggerImpl();
 
        ct.setCronExpression(cronExpression);
        ct.setTimeZone(cronExpression.getTimeZone());
        ct.setMisfireInstruction(misfireInstruction);
 
        return ct;
    }
```
构造的CronTriggerImpl为CronTrigger的实现类

####   CalendarIntervalScheduleBuilder && CalendarIntervalTrigger
最重要的两个属性为
- int interval 间隔数量  
- IntervalUnit intervalUnit    间隔时间单位
这使得构造的Trigger可以指定间隔的单位，比起SimpleScheduleBuilder更方便，间隔时间单位默认为IntervalUnit.DAY，间隔数量为1也就表示默认执行周期为一天。CalendarIntervalScheduleBuilder提供了许多控制时间间隔的方法以withInterval开头，轻松控制间隔为：秒，分钟，小时，天，月，年，星期

看看CronScheduleBuilder的build方法
```
    public MutableTrigger build() {
 
        CalendarIntervalTriggerImpl st = new CalendarIntervalTriggerImpl();
        st.setRepeatInterval(interval);
        st.setRepeatIntervalUnit(intervalUnit);
        st.setMisfireInstruction(misfireInstruction);
        st.setTimeZone(timeZone);
        st.setPreserveHourOfDayAcrossDaylightSavings(preserveHourOfDayAcrossDaylightSavings);
        st.setSkipDayIfHourDoesNotExist(skipDayIfHourDoesNotExist);
 
        return st;
    }
```
构造的CalendarIntervalTriggerImpl为CalendarIntervalTrigger的实现类

####    DailyTimeIntervalScheduleBuilder    &&    DailyTimeIntervalTrigger
DailyTimeIntervalScheduleBuilder中包含的属性列表如下
- int interval = 1;    间隔数量，默认为1
- IntervalUnit intervalUnit = IntervalUnit.MINUTE;    间隔单位默认为分钟
- Set<Integer> daysOfWeek;    指定星期号    
- TimeOfDay startTimeOfDay;    
- TimeOfDay endTimeOfDay;
- int repeatCount = DailyTimeIntervalTrigger.REPEAT_INDEFINITELY;   指定后可以限制执行次数

通过DailyTimeIntervalTrigger可以轻松指定每天在哪个时间段内间隔的任务，并且可以指定星期

###    任务信息存储    Job Stores
任务信息存储，分为基于内存的存储（RAMJobStore）和 数据库的存储(JDBCJobStore)。你可以理解为任务持久化，只是基于内存的存储说是持久化有点容易让人误解。



####   RAMJobStore
关于基于内存的，官方文档有这样[说明](http://www.quartz-scheduler.org/documentation/quartz-2.2.x/tutorials/tutorial-lesson-09)，你只需要配置quartz配置文件的如下配置项，不用做其他
```
org.quartz.jobStore.class = org.quartz.simpl.RAMJobStore
```

而搜索org.quartz.simpl.RAMJobStore的引用（eclipse   中ctrl+shift+g），我们可以看到，在类StdSchedulerFactory中的private Scheduler instantiate()的方法中有如下
```
 String jsClass = cfg.getStringProperty(PROP_JOB_STORE_CLASS,
                RAMJobStore.class.getName());
 
        if (jsClass == null) {
            initException = new SchedulerException(
                    "JobStore class not specified. ");
            throw initException;
        }
```
追踪**cfg.getStringProperty**方法，如下
```
 public String getStringProperty(String name, String def) {
        String val = props.getProperty(name, def);
        if (val == null) {
            return def;
        }
 
        val = val.trim();
 
        return (val.length() == 0) ? def : val;
    }
```
整体看下来，也就是默认情况下使用的**org.quartz.simpl.RAMJobStore**，如果你不配置**Job Stores**默认也是**org.quartz.simpl.RAMJobStore**
