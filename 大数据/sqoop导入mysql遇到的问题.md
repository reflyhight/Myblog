title: sqoop导入mysql遇到的问题
tags: 
	- sqoop
	- mysql
	- 报错

categories:
	- 大数据
------------





##     null值处理
事实上从hive到mysql并不是从hive表中的数据导入mysql，而是hive数据仓库文件中的hdfs文件导入到mysql。hive底层基于hdfs文件，导入与导出都是相对与hdfs来讲的，hive导数据到mysql即为hdfs文件导出到mysql，属于导出任务，使用export。

hive表生成的null字段用\N表示，导入到mysql就会显示为\N，如果mysql中数据为数字型，那么直接会jdbc无法转化\N，报错。为了解决这个问题，只能保证数值型字段不出现空值，可以先进行一个赛选，将null值转换为其他特殊值(这里我只能想到这种办法了），有时候转换后导入mysql仍然报错，那说明转换有问题，可以去hive中检查转换后是否还有null值存在。

在此翻看sqoop文档，发现sqoop-export 中有对NULL的特别处理，见[sqoop官方文档](http://sqoop.apache.org/docs/1.4.6/SqoopUserGuide.html#_syntax_4)。导出语法控制参数（Export control arguments）。

| Argument      |    Description |  翻译  |

| :-------- |: -------- ||: -------- | 
| --input-null-string <null-string>  | The string to be interpreted as null for string columns | 当字段为字符串类型时（varchar,char等），指定值会被转换为nulll导入|
| --input-null-non-string <null-string>  | The string to be interpreted as null for non-string columns | 当字段为非字符串类型时（Int,long等），指定值会被转换为nulll导入|

```
由于\N需要转义，所以需要写成    --input-null-string '\\N'     --input-null-non-string  '\\N'
```

同理，反过来从mysql导入数据到hdfs时，也有类似，详情参考[sqoop官方文档](http://sqoop.apache.org/docs/1.4.6/SqoopUserGuide.html#_selecting_the_data_to_import)。


##    特殊表情字符
很多时候要处理表情类字符，会报类似`“Caused by: java.sql.SQLException: Incorrect string value: '\xF0\x9F\x98\xAB\xF0\x9F...' for column 'content' at row 11”`。

查询到相关问题为字符集utf8mb4。参考[mysql官网说明](https://dev.mysql.com/doc/refman/5.7/en/charset-unicode-conversion.html)，mysql版本至少为5.5.3，设置字符集为utf8mb4（/etc/my.cnf 中配置`character-set-server=utf8mb4`和 `collation_server=utf8mb4_unicode_ci`。重启mysql）。修改表字段的字符集语句如官网说明,如下修改表结构操作。
```
ALTER TABLE t1 DEFAULT CHARACTER
SET utf8mb4,
 MODIFY col1 CHAR (10) CHARACTER
SET utf8mb4 COLLATE utf8mb4_unicode_ci NOT NULL;
```

当以建表中数据量太大时，修改表机构语句执行过程非常慢。此时考虑truncate表后再修改字符集，或者重建表，在具体创建表的时候，将该字段创建为utf8mb4字符编码。如下建表语句中的content字段。
```
CREATE TABLE test(
    id VARCHAR (200),
    content text CHARACTER
SET utf8mb4 COLLATE utf8mb4_unicode_ci
);
```


##    关键字
前两天在导入一张表的时候报如下错误。
```
jdbc4.MySQLSyntaxErrorException: You have an error in your SQL syntax; check the manual that corresponds to 
your MySQL server version for the right syntax to use near 'desc, area, reg_time,
```
sqoop的底层原理是jdbc，如上错误，在平时使用客户端（如navicat）执行sql时也能遇到。但sqoop底层的插入sql是自动生成。当目标表中有关键字作为字段名时（如```  `desc`  ```），生成的查询语句中就包含关键字，就会报错。所以请勿使用关键字作为目标表的字段名，虽然关键字在创建表时加上```  `` ```就能执行成功。
