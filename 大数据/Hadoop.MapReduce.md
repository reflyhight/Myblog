title: Hadoop 之MapReduce（一）
tags: 
	- MapReduce
	- Hadoop

categories:
	- 大数据

-----------------------------

#	简介
MapReduce是一种编程模型。是一种分布式计算模型。MapReduce计算模型是基于Hadoop 文件系统Hdfs的文件系统。

##	Map 
> 映射（Mapping）对集合里的每个目标应用同一个操作。即，如果你想把表单里每个单元格乘以二，那么把这个函数单独地应用在每个单元格上的操作就属于mapping。

##	分组排列
> 在Reduce任务开始之前，会将Map中的键值对进行分组排序，相同的键将被分为同一组，而值将被保存到一个队列中。然后再按值排序。


##	Reduce
> 化简（Reducing ）遍历集合中的元素来返回一个综合的结果。即，输出表单里一列数字的和这个任务属reducing。


![MapReduce原理](/images/hadoop.maperduce_1.png)


##	理解
> 我自己关于mapreduce的理解是，好比炒菜，将食材切片则像是map，而下过烹饪则像是reduce的过程，而只是map和reduce的过程是分布式的。

<!-- more -->

# JAVA API
Hadoop 提供JAVA API.下面以统计某个文本单词出现次数为列。

## maven依赖
>通过maven构建mapreduce 所需要的依赖

```xml
<dependencies>
<dependency>
	<groupId>org.apache.hadoop</groupId>
	<artifactId>hadoop-common</artifactId>
	<version>2.6.0</version>
</dependency>
<dependency>
	<groupId>org.apache.hadoop</groupId>
	<artifactId>hadoop-client</artifactId>
	<version>2.6.0</version>
</dependency>
<dependency>
	<groupId>org.apache.hadoop</groupId>
	<artifactId>hadoop-hdfs</artifactId>
	<version>2.6.0</version>
</dependency>
<dependency>
	<groupId>org.apache.hadoop</groupId>
	<artifactId>hadoop-yarn-common</artifactId>
	<version>2.6.0</version>
</dependency>
<dependency>
	<groupId>org.apache.hadoop</groupId>
	<artifactId>hadoop-yarn-client</artifactId>
	<version>2.6.0</version>
</dependency>
<dependency>
	<groupId>org.apache.hadoop</groupId>
	<artifactId>hadoop-yarn-server-common</artifactId>
	<version>2.6.0</version>
</dependency>
<dependency>
	<groupId>org.apache.hadoop</groupId>
	<artifactId>hadoop-yarn-server-resourcemanager</artifactId>
	<version>2.6.0</version>
</dependency>
<dependency>
	<groupId>org.apache.hadoop</groupId>
	<artifactId>hadoop-yarn-server-nodemanager</artifactId>
	<version>2.6.0</version>
</dependency>
<dependency>
	<groupId>org.apache.hadoop</groupId>
	<artifactId>hadoop-yarn-server-applicationhistoryservice</artifactId>
	<version>2.6.0</version>
</dependency>
<dependency>
	<groupId>org.apache.hadoop</groupId>
	<artifactId>hadoop-mapreduce-client-core</artifactId>
	<version>2.6.0</version>
</dependency>
<dependency>
	<groupId>org.apache.hadoop</groupId>
	<artifactId>hadoop-mapreduce-client-shuffle</artifactId>
	<version>2.6.0</version>
</dependency>



<dependency>
	<groupId>jdk.tools</groupId>
	<artifactId>jdk.tools</artifactId>
	<version>1.7</version>
	<scope>system</scope>
	<systemPath>${JAVA_HOME}/lib/tools.jar</systemPath>
</dependency>
</dependencies>
```

## Mapper类
**org.apache.hadoop.mapreduce.Mapper**
> hadoop 提供** *[org.apache.hadoop.mapreduce.Mapper]* ** <KEYIN, VALUEIN, KEYOUT, VALUEOUT>类，提供给用户实现Map,也就是分布式运算的第一步，将数据做粗处理，将数据（主要是日志文件）做键值对的形式。（此过程就叫Map，映射）

> 其中泛型<KEYIN, VALUEIN, KEYOUT, VALUEOUT> 中VALUEIN是Map task进程中要处理的数据对象，以行为单位，你可以把VALUEIN看作日志的一行。而KEYOUT则是map处理后的键值对中的键，VALUEOUT则是map处理


```java

//MyMap.java
package it.haima.mapreduce;

import java.io.IOException;

import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Mapper;

/* Mapper<KEYIN, VALUEIN, KEYOUT, VALUEOUT> */
class MyMap extends Mapper<LongWritable, Text, Text, LongWritable> {

	@Override
	protected void map(LongWritable key, Text value,
		Mapper<LongWritable, Text, Text, LongWritable>.Context context)
			throws IOException, InterruptedException {
		// TODO Auto-generated method stub
//		String[] split = value.toString().split("[\t ]+");

		//简单用空格切割一行数据
		String[] split = value.toString().split(" ");

		for (String s : split) {
			Text keyout = new Text();
			keyout.set(s);
			LongWritable valout = new LongWritable();
			valout.set(1);

			context.write(keyout, valout); //写入上下文，

		}
	}
}
```

## 排序分组
> map执行完成后，会将键值分组并排序，分组和排序皆按照Key为标准。

> 排序分组，此过程是hadoop内部实现

	-----目标文件内容------
	haima vicky
	haima xuandong


	-----Maptask处理过后----
	haima 1
	vicky 1
	haima 1
	xuandong 1
	

	-----hadoop分组排序后------
	haima <1,1>		//key 分别是  1，1
	xuandong <1>	//key 是 1
	vicky <1>		//key 是 1






## Reducer类
**org.apache.hadoop.mapreduce.Reducer**
> 而类** *[org.apache.hadoop.mapreduce.Reducer]* ** <KEYIN,VALUEIN,KEYOUT,VALUEOUT>类，实现对maper处理后的数据再加工。

> 其中泛型<KEYIN, VALUEIN, KEYOUT, VALUEOUT> KEYIN是MyMap.java 146行 *context.write(keyout, valout)* 中的keyout，VALUEIN则是valout

```java

//MyReduce.java
package it.haima.mapreduce;

import java.io.IOException;

import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Reducer;

/* Reducer<KEYIN, VALUEIN, KEYOUT, VALUEOUT> */
class MyReduce extends Reducer<Text, LongWritable, Text, LongWritable> {
	@Override
	protected void reduce(Text key, Iterable<LongWritable> valuesIn,
		Reducer<Text, LongWritable, Text, LongWritable>.Context context)
			throws IOException, InterruptedException {
		// TODO Auto-generated method stub

		int sum = 0;

		//以前文分组后的，haima <1,1>为例，则valuesIn是  <1,1>的一个集合
		for (LongWritable valWrite : valuesIn) {

			long l = valWrite.get();

			//简单统计 haima出现次数
			sum += l; 

		}

		LongWritable valout = new LongWritable();
		valout.set(sum);

		context.write(key, valout);//

	}
}

```

## 入口指定
> 新建mapreduce任务(job),并设置各种配置
> 将MyWordCount.java，MyReduce.java，MyMap.java打成jar包，并制定Main入口


```java
//MyWordCount.java
package it.haima.mapreduce;

import java.io.IOException;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;

public class MyWordCount {

	public static void main(String[] args) throws IOException, 
	ClassNotFoundException, InterruptedException {
		
	Configuration conf = new Configuration();
	Job job = Job.getInstance(conf, MyWordCount.class.getSimpleName());
	job.setJarByClass(MyWordCount.class);
	
	//设置处理的路径
	//hdfs路径，要处理的文件路径
	FileInputFormat.setInputPaths(job, new Path(args[0])); 

	//hdfs路径，处理后文件存放路径
	FileOutputFormat.setOutputPath(job, new Path(args[1]));	
	job.setNumReduceTasks(Integer.parseInt(args[2]));
	
	
	FileInputFormat.setInputDirRecursive(job, true);
	
	
	

	job.setMapperClass(MyMap.class);//指定maper
	job.setReducerClass(MyReduce.class);//指定reducer
	
	job.setMapOutputKeyClass(Text.class);//指定reduce key 类型
	job.setMapOutputValueClass(LongWritable.class); //指定  reduce value 类型
	
	job.setOutputKeyClass(Text.class);	//指定处理后的key类型
	job.setOutputValueClass(LongWritable.class);//指定处理后的val类型
	
	
	job.waitForCompletion(true);
		
	}

}

```