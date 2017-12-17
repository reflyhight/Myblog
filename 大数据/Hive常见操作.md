> Hive官网的手册很详尽，基本上参考就能轻松运用hive [Wiki](https://cwiki.apache.org/confluence/display/Hive/Home)
 

 
###    Hive CLI（命令行）
hive cli指的是`$HIVE_HOME/bin/hive`命令
 
####    Hive Command Line Options（命令行选项）
使用    `$HIVE_HOME/bin/hive -H`   或者 $HIVE_HOME/bin/hive --help查看hive cli的命令选项      详见    [Wiki](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+Cli#LanguageManualCli-HiveCommandLineOptions)
```
 -d,--define <key=value>         申明变量    -d date='20171207'
 -e <quoted-query-string>        执行指定脚本    -e "select * from tb1"
 -f <filename>                   执行指定文件中的sql    -f  f1.sql
 -H,--help                            使用帮助
 -h <hostname>                    指定主机/IP
    --hiveconf <property=value>   Use value for given property
    --hivevar <key=value>         Variable substitution to apply to hive commands. e.g. --hivevar A=B
 -i <filename>                   指定文件中的sql初始化执行
 -p <port>                       指定端口
 -S,--silent                      安静模式，不打印执行的sql
 -v,--verbose                   和-S相反，也是默认的模式
```
另外查询版本可以用`hive --version`（至少我这边cdh5.6.1可以这样用）
 
 
###   LanguageManual Commands（支持命令）
这些命令都并非sql语句，可以使用在HiveQL或者直接使用在CLI 、 Beeline中    详见    [Wiki](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+Commands)
 
- `quit、exit`    退出交互式shell   (CLI 、 Beeline)
- `reset`    将配置重置为默认值，任意配置参数无论是使用set命令或者-hiveconf 设置的参数都会被重置为默认值。注意使用hiveconf:为前缀的参数不会被重置（历史原因造成）
- `set <key>=<value>`     设置配置。特别注意，如果你把配置的名字写错，系统并不会报错提醒你😆
- `set`    打印配置，包括用户定义覆盖和Hive自带的
- `set -v`    打印所有hadoop和hive的配置变量
- `add FILE[S] <filepath> <filepath>*  `        添加文件到分布式缓存中，可以参考[Resources](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+Cli#LanguageManualCli-HiveResources)的相关说明
- `add JAR[S] <filepath> <filepath>*    `    添加jar文件到分布式缓存中，可以参考[Resources](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+Cli#LanguageManualCli-HiveResources)的相关说明
- `add ARCHIVE[S] <filepath> <filepath>*   `  添加压缩文件到分布式缓存中，可以参考[Resources](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+Cli#LanguageManualCli-HiveResources)的相关说明
- `add  FILE[S]`   |  `add JAR[S]`  | `add ARCHIVE[S]`   在Version 1.2.0后支持   ivyurl的形式我，见[LanguageManual](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+Cli#LanguageManualCli-TooltoClearDanglingScratchDirectories)
 
- `list FILE[S]`    列出加入分布式缓存中的文件
- `list JAR[S] `   列出加入分布式缓存中的Jar
- `list ARCHIVE[S] `   列出加入分布式缓存中的ARCHIVE
 
- `delete FILE[S] <filepath>*  `  删除分布式缓存的文件
- `delete JAR[S] <filepath>*  `  删除分布式缓存的JAR
- `delete ARCHIVE[S] <filepath>*  `   删除分布式缓存的ARCHIVE
- `delete FILE[S] <ivyurl> <ivyurl>*  `  | `  delete JAR[S] <ivyurl> <ivyurl>* `  | ` delete ARCHIVE[S] <ivyurl> <ivyurl>*  `  在Hive 1.2.0后被支持
- `! <command>  `  执行linux命令
- `dfs <dfs command> `   执行dfs命令
- `<query string>  `  执行hive sql
- `source FILE  <filepath> `     执行文件里的hive sql，(cdh5.6.1-hive1.1.0版本没有关键词FILE，使用source  filepath）
- compile  ` <groovy string> `  AS GROOVY NAMED <name>   没用过：This allows inline Groovy code to be compiled and be used as a UDF (as of Hive 0.13.0). For a usage example, see Nov. 2013 Hive Contributors Meetup Presentations – Using Dynamic Compilation with Hive.
 
使用案例:
```
hive> set mapred.reduce.tasks=32;
hive> set;
hive> select a.* from tab1;
hive> !ls;
hive> dfs -ls;
```
 
###    LanguageManual Types（支持类型）
以下是Hive支持的常见类型，详见    [WIKI](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+Types)
####    Numeric Types（数值型）
```
TINYINT (1-byte signed integer, from -128 to 127)
SMALLINT (2-byte signed integer, from -32,768 to 32,767)
INT/INTEGER (4-byte signed integer, from -2,147,483,648 to 2,147,483,647)    INTEGER 在hive2.2.0后和INT同义
BIGINT (8-byte signed integer, from -9,223,372,036,854,775,808 to 9,223,372,036,854,775,807)
FLOAT (4-byte single precision floating point number)
DOUBLE (8-byte double precision floating point number)
DOUBLE PRECISION (alias for DOUBLE, only available starting with Hive 2.2.0)
DECIMAL    Introduced in Hive 0.11.0 with a precision of 38 digits   Hive 0.13.0 introduced user-definable precision and scale   对应java中的BigDecimal ，在精准计算中使用，避免float或者double类型造成的精度问题（计算机2进制造成浮点型计算结果问题）
NUMERIC (same as DECIMAL, starting with Hive 3.0.0)
```
####    Date/Time Types（日期类型）
```
TIMESTAMP (Note: Only available starting with Hive 0.8.0)    可以转换为数值型（浮点型和整型）使用UNIX timestamp(距离1970年的秒数)，转换为String格式可为`YYYY-MM-DD HH:MM:SS.fffffffff`,TIMESTAMP在text files中的格式必须为yyyy-mm-dd hh:mm:ss[.f...]
DATE (Note: Only available starting with Hive 0.12.0)    YYYY-­MM-­DD表示的日期，date类型text files的格式必须为YYYY-­MM-­DD，支持string和Timestamp转换（需要mr计算，比较慢）
INTERVAL (Note: Only available starting with Hive 1.2.0)    时间间隔，应该是做时间计算的，沒找到相关用法
```
####   String Types   (字符串类型)
```
STRING    不限制长度
VARCHAR (Note: Only available starting with Hive 0.12.0)    (between 1 and 65535)
CHAR (Note: Only available starting with Hive 0.13.0)    (between 1 and 255)，固定长度，长度不够，会用空格补齐，不怎么用
```
####   Misc Types   （其他杂类）
```
BOOLEAN
BINARY (Note: Only available starting with Hive 0.8.0)    并不知道和普通的字符串有什么区别
```
####    Complex Types    (复合类型)
```
arrays: ARRAY<data_type> (Note: negative values and non-constant expressions are allowed as of Hive 0.14.)
maps: MAP<primitive_type, data_type> (Note: negative values and non-constant expressions are allowed as of Hive 0.14.)
structs: STRUCT<col_name : data_type [COMMENT col_comment], ...>
union: UNIONTYPE<data_type, data_type, ...> (Note: Only available starting with Hive 0.7.0.)    Hive支持这种类型比较弱，不支持Join,where和group by 语句
```
 
####    综合案例
创建表：
```sql
-- 不知道union怎么用，暂时没加入
drop table if EXISTS typedemo;
CREATE table typedemo(
    NumericT INT comment 'Numeric type INT',
    DateT TIMESTAMP comment 'DateT type TIMESTAMP',
    DateT2 DATE comment 'DateT type date',
    StringT    STRING comment 'type String',
    VarcharT    VARCHAR(100)    comment 'type Varchar',
    CharT    CHAR(50)    comment 'type Char',
    BooleanT BOOLEAN comment 'type Boolean',
    BinaryT  BINARY comment 'type Binary',
    ArraysT    ARRAY<STRING> comment ' type ARRAY',
    MapT MAP<STRING,INT> comment 'type    MAP',
    StructT STRUCT<name:STRING,age:INT>    comment ' type STRUCT'
)
row format delimited
fields terminated by '\t' -- 字段分割符
collection items terminated by ',' -- 集合元素分隔符，array元素，Map的entry之间，struct的元素之间
map keys terminated by ':'    -- map键值对之间
lines terminated by '\n'
stored as textfile;
```
**注意**：从 `row format delimited`开始为指定行格式，`row format delimited`是必须的，其他`terminated`语句可以缺少，但需要保持几个语句的先后顺序
vim 编辑数据    type.test.data：
```
#注意字段分割符为制表符
1       2017-12-09 21:41:00     2017-12-09      str     vchar   char    true    binary  rob,mxj weight:136,age:18       rob,18
```
加入到表中：
```
load data local inpath './type.test.data'   into table typedemo;
```
 
测试数据：
```
hive (test)> select numerict,datet,datet2,stringt,varchart,chart from typedemo;
OK
numerict        datet   datet2  stringt varchart        chart
1       2017-12-09 21:41:00     2017-12-09      str     vchar   char                                            
Time taken: 0.105 seconds, Fetched: 1 row(s)
 
hive (test)> select booleant,binaryt,arrayst,mapt,structt from typedemo;
OK
booleant        binaryt arrayst mapt    structt
true    n)گ     ["rob","mxj"]   {"weight":136,"age":18} {"name":"rob","age":18}
Time taken: 0.085 seconds, Fetched: 1 row(s)
 
hive (test)> select arrayst[0]  from typedemo;
....MapReduce运行日志
OK
_c0
rob
 
hive (test)> select structt.name as name from typedemo;
....MapReduce运行日志
OK
name
rob
 
hive (test)> select MapT['age'] as age from typedemo;
....MapReduce运行日志
OK
age
18
```
 
###   DDL(Data Definition Language) 数据库数据定义语句
数据库定义语句主要包含    `create/drop/alter/truncate/show/describe`    详见    [Wiki](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+DDL)
 
####    Overview    （梗概）
DDL的语句的关键词都包含如下：
- CREATE DATABASE/SCHEMA, TABLE, VIEW, FUNCTION, INDEX
- DROP DATABASE/SCHEMA, TABLE, VIEW, INDEX
- TRUNCATE TABLE
- ALTER DATABASE/SCHEMA, TABLE, VIEW
- MSCK REPAIR TABLE (or ALTER TABLE RECOVER PARTITIONS)
- SHOW DATABASES/SCHEMAS, TABLES, TBLPROPERTIES, VIEWS, PARTITIONS, FUNCTIONS, INDEX[ES], COLUMNS, CREATE TABLE
- DESCRIBE DATABASE/SCHEMA, table_name, view_name
 
PARTITION (分区)语句通常是表的限定，出来SHOW PARTITIONS语句
 
####   Create/Drop/Alter/Use Database     （数据库相关语句）
**Create Database**
```
CREATE (DATABASE|SCHEMA) [IF NOT EXISTS] database_name
[COMMENT database_comment]    -- 添加数据库说名
[LOCATION hdfs_path]    -- 指定hive默认存储路径
[WITH DBPROPERTIES (property_name=property_value, ...)];    -- 添加一些属性
```
**特别说明：**  SCHEMA和DATABASE同义，原文说the same，are interchangeable。CREATE DATABASE was added in Hive 0.6   The WITH DBPROPERTIES clause was added in Hive 0.7 。
 
**Drop Database**
```
DROP (DATABASE|SCHEMA) [IF EXISTS] database_name [RESTRICT|CASCADE];
```
 
**Alter Database**
```
ALTER (DATABASE|SCHEMA) database_name SET DBPROPERTIES (property_name=property_value, ...);   -- (Note: SCHEMA added in Hive 0.14.0)
 
ALTER (DATABASE|SCHEMA) database_name SET OWNER [USER|ROLE] user_or_role;   -- (Note: Hive 0.13.0 and later; SCHEMA added in Hive 0.14.0)
 
ALTER (DATABASE|SCHEMA) database_name SET LOCATION hdfs_path; -- (Note: Hive 2.2.1, 2.4.0 and later)
```
**特别说明：**    除了DBPROPERTIES，OWNER ，LOCATION，其他的元数据信息是不可更改的，其中LOCATION不会更改现在已有的数据存放位置，LOCATION只是数据库默认数据存放位置，只是新创建表的时候，数据存储位置会默认放到设置的位置。这种操作和修改表默认位置而不影响已有分区是一个道理。ALTER SCHEMA was added in Hive 0.14 (HIVE-6601).
 
**Use Database**
```
USE database_name;
USE DEFAULT;
```
**特别说明：**    使用USE语句切换数据库，切换至默认数据库使用USE DEFAULT。USE database_name was added in Hive 0.6 (HIVE-675).
 
####    Create/Drop/Truncate Table
**Create Table**
这个就有点儿长
```
CREATE [TEMPORARY] [EXTERNAL] TABLE [IF NOT EXISTS] [db_name.]table_name    -- (Note: TEMPORARY available in Hive 0.14.0 and later)
  [(col_name data_type [COMMENT col_comment], ... [constraint_specification])]
  [COMMENT table_comment]
  [PARTITIONED BY (col_name data_type [COMMENT col_comment], ...)]
  [CLUSTERED BY (col_name, col_name, ...) [SORTED BY (col_name [ASC|DESC], ...)] INTO num_buckets BUCKETS]
  [SKEWED BY (col_name, col_name, ...)                  -- (Note: Available in Hive 0.10.0 and later)]
     ON ((col_value, col_value, ...), (col_value, col_value, ...), ...)
     [STORED AS DIRECTORIES]
  [
   [ROW FORMAT row_format]
   [STORED AS file_format]
     | STORED BY 'storage.handler.class.name' [WITH SERDEPROPERTIES (...)]  -- (Note: Available in Hive 0.6.0 and later)
  ]
  [LOCATION hdfs_path]
  [TBLPROPERTIES (property_name=property_value, ...)]   -- (Note: Available in Hive 0.6.0 and later)
  [AS select_statement];   -- (Note: Available in Hive 0.5.0 and later; not supported for external tables)
 
CREATE [TEMPORARY] [EXTERNAL] TABLE [IF NOT EXISTS] [db_name.]table_name
  LIKE existing_table_or_view_name
  [LOCATION hdfs_path];
 
data_type
  : primitive_type
  | array_type
  | map_type
  | struct_type
  | union_type  -- (Note: Available in Hive 0.7.0 and later)
 
primitive_type
  : TINYINT
  | SMALLINT
  | INT
  | BIGINT
  | BOOLEAN
  | FLOAT
  | DOUBLE
  | DOUBLE PRECISION -- (Note: Available in Hive 2.2.0 and later)
  | STRING
  | BINARY      -- (Note: Available in Hive 0.8.0 and later)
  | TIMESTAMP   -- (Note: Available in Hive 0.8.0 and later)
  | DECIMAL     -- (Note: Available in Hive 0.11.0 and later)
  | DECIMAL(precision, scale)  -- (Note: Available in Hive 0.13.0 and later)
  | DATE        -- (Note: Available in Hive 0.12.0 and later)
  | VARCHAR     -- (Note: Available in Hive 0.12.0 and later)
  | CHAR        -- (Note: Available in Hive 0.13.0 and later)
 
array_type
  : ARRAY < data_type >
 
map_type
  : MAP < primitive_type, data_type >
 
struct_type
  : STRUCT < col_name : data_type [COMMENT col_comment], ...>
 
union_type
   : UNIONTYPE < data_type, data_type, ... >  -- (Note: Available in Hive 0.7.0 and later)
 
row_format
  : DELIMITED [FIELDS TERMINATED BY char [ESCAPED BY char]] [COLLECTION ITEMS TERMINATED BY char]
        [MAP KEYS TERMINATED BY char] [LINES TERMINATED BY char]
        [NULL DEFINED AS char]   -- (Note: Available in Hive 0.13 and later)
  | SERDE serde_name [WITH SERDEPROPERTIES (property_name=property_value, property_name=property_value, ...)]
 
file_format:
  : SEQUENCEFILE
  | TEXTFILE    -- (Default, depending on hive.default.fileformat configuration)
  | RCFILE      -- (Note: Available in Hive 0.6.0 and later)
  | ORC         -- (Note: Available in Hive 0.11.0 and later)
  | PARQUET     -- (Note: Available in Hive 0.13.0 and later)
  | AVRO        -- (Note: Available in Hive 0.14.0 and later)
  | INPUTFORMAT input_format_classname OUTPUTFORMAT output_format_classname
 
constraint_specification:
  : [, PRIMARY KEY (col_name, ...) DISABLE NOVALIDATE ]
    [, CONSTRAINT constraint_name FOREIGN KEY (col_name, ...) REFERENCES table_name(col_name, ...) DISABLE NOVALIDATE
```
 
**创建表：**
- 创建给定定义表明的表，存在同名表会抛出错误，可以使用IF NOT EXISTS 语句来防止报错。
- 表名和列名对大小写都是不敏感的（定义的大写都会转换为小写），不过SerDe 序列化定义和property 属性名都是大小写敏感的
- 0.12版本和0.12之前，只有字母和下划线才能被定义为表名或者列名
- 0.13后支持所有的unicode
- 表和列的`注释`都是字符串
- 不加EXTERNAL定义的表为`管理表`（managed table）加EXTERNAL定义的表叫做`外部表`（external table），查看表是外部表还是管理表可以使用[ DESCRIBE EXTENDED table_name 语句](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+DDL#LanguageManualDDL-DescribeTable/View/Column) 
- `表属性`（TBLPROPERTIES）定义语句允许你定义自己k-v形式的元数据，一些预定义的表属性已经存在，如last_modified_user ，last_modified_time会自动被Hive加上，其他的预定义的属性包括
   - TBLPROPERTIES ("comment"="table_comment")
   - TBLPROPERTIES ("hbase.table.name"="table_name") – see HBase Integration.
   - TBLPROPERTIES ("immutable"="true") or ("immutable"="false") in release 0.13.0+ (HIVE-6406) – see Inserting Data into Hive Tables from Queries.
   - TBLPROPERTIES ("orc.compress"="ZLIB") or ("orc.compress"="SNAPPY") or ("orc.compress"="NONE") and other ORC properties – see ORC Files.
   - TBLPROPERTIES ("transactional"="true") or ("transactional"="false") in release 0.14.0+, the default is "false" – see Hive Transactions.
   - TBLPROPERTIES ("NO_AUTO_COMPACTION"="true") or ("NO_AUTO_COMPACTION"="false"), the default is "false" – see Hive Transactions.
   - TBLPROPERTIES ("compactor.mapreduce.map.memory.mb"="mapper_memory") – see Hive Transactions.
   - TBLPROPERTIES ("compactorthreshold.hive.compactor.delta.num.threshold"="threshold_num") – see Hive Transactions.
   - TBLPROPERTIES ("compactorthreshold.hive.compactor.delta.pct.threshold"="threshold_pct") – see Hive Transactions.
   -  TBLPROPERTIES ("auto.purge"="true") or ("auto.purge"="false") in release 1.2.0+ (HIVE-9118) – see Drop Table, Drop Partitions, Truncate Table, and Insert Overwrite.
   -  TBLPROPERTIES ("EXTERNAL"="TRUE") in release 0.6.0+ (HIVE-1329) – Change a managed table to an external table and vice versa for "FALSE". 2.4.0后
"EXTERNAL"属性由str型改为boolean形

- 更多关于 `注释`，`表属性`，`序列化`，参考[Alter Table](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+DDL#LanguageManualDDL-AlterTable)

- 更多数据类型查看[Type System](https://cwiki.apache.org/confluence/display/Hive/Tutorial#Tutorial-TypeSystem) [ Hive Data Types](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+Types)

**管理表和外部表**：
- 管理表，默认创建的表都是管理表（不加External申明），管理表的`数据文件`，`元数据`，`计算statistics`都有Hive自身管理。管理表的数据被保管在默认[hive.metastore.warehouse.dir](https://cwiki.apache.org/confluence/display/Hive/Configuration+Properties#ConfigurationProperties-hive.metastore.warehouse.dir) 路劲下,默认路径可以被 location 定义覆盖
- 外部表，An external table describes the metadata / schema on external files. External table files can be accessed and managed by processes outside of Hive.(这句话的意思就是说:一个外部表的元数据信息可以在外部文件中描述，外部表可以被Hive以外程序管理和使用。【这句话我不是很明白】）外部表的分区结构如果被修改了，[MSCK REPAIR TABLE table_name ](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+DDL#LanguageManualDDL-RecoverPartitions(MSCKREPAIRTABLE))，[相关博客](http://blog.csdn.net/opensure/article/details/51323220)
- [Statistics](https://cwiki.apache.org/confluence/display/Hive/StatsDev) can be managed on internal and external tables and partitions for query optimization. (这句的意思是，内部表（管理表）和外部表中一些统计可以对查询做优化【暂时不明白它的真正意思】）

**存储格式**
- Hive支持内置格式和自定义的格式，压缩存储详见[CompressedStorage](https://cwiki.apache.org/confluence/display/Hive/CompressedStorage)
- 内置格式如下列表：


未完待续。。
 
