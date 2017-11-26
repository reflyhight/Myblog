
title: Hadoop 之hdfs（一）
tags: 
	- dfs
	- Hadoop

categories:
	- 大数据

-------------------


# hdfs 命令行shell


hdfs 命令行shell 是在Linux操作 hdfs 的接口，或者说工具
>shell列表

	[-appendToFile <localsrc> ... <dst>]
	[-cat [-ignoreCrc] <src> ...]
	[-checksum <src> ...]
	[-chgrp [-R] GROUP PATH...]
	[-chmod [-R] <MODE[,MODE]... | OCTALMODE> PATH...]
	[-chown [-R] [OWNER][:[GROUP]] PATH...]
	[-copyFromLocal [-f] [-p] [-l] <localsrc> ... <dst>]
	[-copyToLocal [-p] [-ignoreCrc] [-crc] <src> ... <localdst>]
	[-count [-q] [-h] <path> ...]
	[-cp [-f] [-p | -p[topax]] <src> ... <dst>]
	[-createSnapshot <snapshotDir> [<snapshotName>]]
	[-deleteSnapshot <snapshotDir> <snapshotName>]
	[-df [-h] [<path> ...]]
	[-du [-s] [-h] <path> ...]
	[-expunge]
	[-get [-p] [-ignoreCrc] [-crc] <src> ... <localdst>]
	[-getfacl [-R] <path>]
	[-getfattr [-R] {-n name | -d} [-e en] <path>]
	[-getmerge [-nl] <src> <localdst>]
	[-help [cmd ...]]
	[-ls [-d] [-h] [-R] [<path> ...]]
	[-mkdir [-p] <path> ...]
	[-moveFromLocal <localsrc> ... <dst>]
	[-moveToLocal <src> <localdst>]
	[-mv <src> ... <dst>]
	[-put [-f] [-p] [-l] <localsrc> ... <dst>]
	[-renameSnapshot <snapshotDir> <oldName> <newName>]
	[-rm [-f] [-r|-R] [-skipTrash] <src> ...]
	[-rmdir [--ignore-fail-on-non-empty] <dir> ...]
	[-setfacl [-R] [{-b|-k} {-m|-x <acl_spec>} <path>]|[--set <acl_spec> <path>]]
	[-setfattr {-n name [-v value] | -x name} <path>]
	[-setrep [-R] [-w] <rep> <path> ...]
	[-stat [format] <path> ...]
	[-tail [-f] <file>]
	[-test -[defsz] <path>]
	[-text [-ignoreCrc] <src> ...]
	[-touchz <path> ...]
	[-usage [cmd ...]]


<!-- more -->

## hdfs dfs -ls
> 格式	【[-ls [-d] [-h] [-R] [<path> ...]]】
> 查看 hdfs 上的文件列表，与Linux中的ls命令相似，只是查询的是hdfs中的文件

	# dfs -ls /
	Found 5 items
	-rw-r--r--   1 root supergroup      87027 2016-05-06 05:37 /hadoop.log
	drwxr-xr-x   - root supergroup          0 2016-05-06 06:08 /hhh
	drwx------   - root supergroup          0 2016-05-06 06:07 /history
	-rw-r--r--   1 root supergroup  142376665 2016-05-08 05:17 /jdk
	drwxr-xr-x   - root supergroup          0 2016-05-06 06:07 /tmp

> -R 递归查看。如果是路劲，则显示路劲下的文件
> -d 仅显示路径
> -h 格式化显示大小，人易读的格式

## hdfs dfs -put
> 格式 【[-put [-f] [-p] [-l] <localsrc> ... <dst>]】
> 上传文件到hdfs文件系统中

	# dfs -put  /usr/local/log/redis.log  /
	# dfs -ls /
	Found 6 items
	-rw-r--r--   1 root supergroup      87027 2016-05-06 05:37 /hadoop.log
	drwxr-xr-x   - root supergroup          0 2016-05-06 06:08 /hhh
	drwx------   - root supergroup          0 2016-05-06 06:07 /history
	-rw-r--r--   1 root supergroup  142376665 2016-05-08 05:17 /jdk
	-rw-r--r--   1 root supergroup      21187 2016-05-08 05:29 /redis.log
	drwxr-xr-x   - root supergroup          0 2016-05-06 06:07 /tmp

> -f 不询问而直接覆盖
> -l 上传时策略改为延迟加载的模式

	

## hdfs dfs -get
> 格式[-get [-p] [-ignoreCrc] [-crc] <src> ... <localdst>]
> 从hdfs上下载文件文件



	# dfs -get /redis.log   /usr/local/
	# tail redis.log 
	[3812] 28 Apr 03:00:27.486 # WARNING overcommit_memory is set tthis to take effect.
	[3812] 28 Apr 03:00:27.4


## hdfs dfs -text
> 格式 [-text [-ignoreCrc] <src> ...]
> 查看文本文件内容
	
	# dfs -text /redis.log
	[9190] 26 Apr 23:12:02.789 * Increased maximum number of open files to 10032 (it was originally set to 1024).
	                _._                                                  
	           _.-``__ ''-._                                             
	      _.-``    `.  `_.  ''-._           Redis 2.8.24 (00000000/0) 64 bit


## hdfs dfs -tail
> 格式 [-tail [-f] <file>]
> 显示文本文件末尾内容，默认显示1k内容
	
	# dfs -tail /redis.log
	is to take effect.
	[3812] 28 Apr 03:00:27.486 # WARNING you have Transparent Huge Pages 
	(THP) support enabled in your kernel. This will create latency and memory
	 usage issues with Redis. To fix this issue run the command 'echo never >
	 /sys/kernel/mm/transparent_hugepage/enabled' as root, and add it to your
	 /etc/rc.local in order to retain the setting after a r

> -f 表示动态显示，会随时显示文档当前被追加进来的数据


## hdfs dfs -mkdir
> 格式 【[-mkdir [-p] <path> ...]】
> 创建文件夹
> -p 如果有多级文件夹，则使用该参数，创建多级文件夹

## hdfs dfs -tail
> 格式 【[-cp [-f] [-p | -p[topax]] <src> ... <dst>]】
> 拷贝文件

## hdfs dfs -count
> 格式 【[-count [-q] [-h] <path> ...]】
> 查看路劲的信息
> -h 格式化文件大小，使得易读性
> 默认顺序 DIR_COUNT FILE_COUNT CONTENT_SIZE FILE_NAME


## hdfs dfs -help
> 格式【[-help [cmd ...]]】
> 查看其他命令的使用方式

	# dfs -help count
	-count [-q] [-h] <path> ... :
	  Count the number of directories, files and bytes under the paths
	  that match the specified file pattern.  The output columns are:
	  DIR_COUNT FILE_COUNT CONTENT_SIZE FILE_NAME or
	  QUOTA REMAINING_QUOTA SPACE_QUOTA REMAINING_SPACE_QUOTA 
	        DIR_COUNT FILE_COUNT CONTENT_SIZE FILE_NAME
	  The -h option shows file sizes in human readable format.