> Hive官网的手册很详尽，基本上参考就能轻松运用hive [Wiki](https://cwiki.apache.org/confluence/display/Hive/Home)
 
[toc]
 
###    Hive CLI（命令行）
hive cli指的是`$HIVE_HOME/bin/hive`命令
 
####    Hive Command Line Options（命令行选项）
使用    `$HIVE_HOME/bin/hive -H`   或者 $HIVE_HOME/bin/hive --help查看hive cli的命令选项      详见[Wiki](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+Cli#LanguageManualCli-HiveCommandLineOptions)
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
另外查询版本可以用hive --version（至少我这边cdh5.6.1可以这样用）
 
 
###   LanguageManual Commands（支持命令）
这些命令都并非sql语句，可以使用在HiveQL或者直接使用在CLI 、 Beeline中
 
- **quit、exit**    退出交互式shell   (CLI 、 Beeline)
- **reset**    将配置重置为默认值，任意配置参数无论是使用set命令或者-hiveconf 设置的参数都会被重置为默认值。注意使用hiveconf:为前缀的参数不会被重置（历史原因造成）
- **set** <key>=<value>    设置配置。如果没忘记写配置名，那么不会报错，只会警告
- **set**    打印配置，包括用户定义覆盖和Hive自带的
- **set -v**    打印所有hadoop和hive的配置变量
- **add FILE[S] <filepath> <filepath>*  **        添加文件到分布式缓存中，可以参考[Resources](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+Cli#LanguageManualCli-HiveResources)的相关说明
- **add JAR[S] <filepath> <filepath>*    **    添加jar文件到分布式缓存中，可以参考[Resources](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+Cli#LanguageManualCli-HiveResources)的相关说明
- **add ARCHIVE[S] <filepath> <filepath>*   **  添加压缩文件到分布式缓存中，可以参考[Resources](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+Cli#LanguageManualCli-HiveResources)的相关说明
- **add  FILE[S]**   |  **add JAR[S]**  | **add ARCHIVE[S]**   在Version 1.2.0后支持   ivyurl的形式我，见[LanguageManual](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+Cli#LanguageManualCli-TooltoClearDanglingScratchDirectories)
 
- **list FILE[S]**    列出加入分布式缓存中的文件
- **list JAR[S] **   列出加入分布式缓存中的Jar
- **list ARCHIVE[S] **   列出加入分布式缓存中的ARCHIVE
 
- **delete FILE[S] <filepath>*  **  删除分布式缓存的文件
- **delete JAR[S] <filepath>*  **  删除分布式缓存的JAR
- **delete ARCHIVE[S] <filepath>*  **   删除分布式缓存的ARCHIVE
- **delete FILE[S] <ivyurl> <ivyurl>*  **  | **  delete JAR[S] <ivyurl> <ivyurl>* **  | ** delete ARCHIVE[S] <ivyurl> <ivyurl>*  **  在Hive 1.2.0后被支持
- **! <command>  **  执行linux命令
- **dfs <dfs command> **   执行dfs命令
- **<query string>  **  执行hive sql 
- **source FILE  <filepath> **     执行文件里的hive sql，(cdh5.6.1-hive1.1.0版本没有关键词FILE，使用source  filepath）
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
	StringT	STRING comment 'type String',
	VarcharT	VARCHAR(100)	comment 'type Varchar',
	CharT	CHAR(50)	comment 'type Char',
	BooleanT BOOLEAN comment 'type Boolean',
	BinaryT  BINARY comment 'type Binary',
	ArraysT	ARRAY<STRING> comment ' type ARRAY',
	MapT MAP<STRING,INT> comment 'type	MAP',
	StructT STRUCT<name:STRING,age:INT>	comment ' type STRUCT'
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
数据库定义语句主要包含    `create/drop/alter/truncate/show/describe`


###    HiveServer2 and Beeline
hive 2.x 以后引入了HiveServer2，HiveServer2拥有自己CLI，叫Beeline，Hive CLI会在不久的将来过时，而建议使用Beeline替代它      详见[Wiki](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+Cli#LanguageManualCli-DeprecationinfavorofBeelineCLI)
 
 
