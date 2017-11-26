title: Linux之shell脚本入门
tags: 
	- Linux
	- 脚本

categories:
	- Linux
-------------------

shell简介
--------
shell脚本是一组命令的组合，通常写在一个文本中。
shell是壳子的意思，在Linux中，可以把Linux内核执行命令看作是一个模型，而命令就像内核的壳。
shell包括sh,csh,ksh,bash,tcsh,zsh。现在最广泛的是bash，本文也是对bash的入门介绍。


第一个脚本
-----------------------
bash 的申明是在行首申明为#!/bin/bash
除了首行中申明脚本外，#表示注释，以#开头的行将不被执行。

**普通的方式**
步骤1：编辑一个纯文本文档
	# vi bash1.sh
	
	#!/bin/bash			//文本首行申明为bash
	#my first bashell!	//#号注释整行
	echo "I am Haima";	//脚本的具体内容
	echo "I am Vicky";

步骤2：授予文本执行权限
	# chmod u+x bash1.sh // 第一种方式，授权当前用户拥有执行选项

步骤3：执行脚本
	# ./bash1.sh 	//执行脚本
	I am Haima
	I am Vicky

**bash命令方式**
步骤1：编辑一个纯文本文档
	# vi bash2.sh
	
	#no need declare	//普通注释，不用申明bash
	#my first bashell!	//#号注释整行
	echo "I am Haima";	//脚本的具体内容
	echo "I am Vicky";

步骤2：直接执行命令bash启动shell脚本
	# bash bash2.sh 
	I am Haima
	I am Vicky


**关于调试**

	显示执行步骤	//没什么用
	 bash -x bash1.sh
	bash语法检查
	 bash -n bash2.sh

	【释】shell脚本执行时，如果中间有错误，脚本不会停止，所以  语法检查时非常有必要的。

<!-- more -->

变量简介
----

1：只能使用数字，字母和下划线，且不能以数字开头
2：变量名区分大小写
3：建议命令要通俗易懂
4：变量分为：本地变量、环境变量、局部变量、位置变量、

本地变量
----------
声明：VAR_NAME=VAR_NAME  

使用：$VAR_NAME 或者 $(VAR_NAME) 

撤销变量：unset VAR_NAME

eg|本地变量

	# val="vicky"
	# echo $val
	vicky
	# echo $valhhh
						//什么也没打印，因为没有找到 $valhhh
	# echo ${val}hhh
	vickyhhh
	# unset val			//撤销变量
	# echo $val
						//撤销后什么也不打印
	


环境变量
--------
声明：export VAR_NAME=VALUE 使用和普通变量相同
该变量对当前会话shell和子shell有效

eg1|环境变量

	# export eveVal="hello"	//申明一个环境变量
	# evalVal1="vicky"
	# bash
	# echo $eveVal 		//打印环境变量
	hello
	# echo  $eveVal1	//打印普通变量
		//普通变量无法在子shell会话中使用，什么也没打出来

	
eg2|环境变量

	#vi	/etc/profile	//编辑linux环境变量配置文件
	JAVA_HOME="Jdk_Path"
	export PATH=$PATH:$JAVA_HOME	//拼接环境变量JAVA_HOME，Linux中用":"，windows中配置
	JAVA用的 ";"

	#source /etc/profile	//将/etc/profile在当前shell会话执行一次
	# echo  $PATH
	.:.:/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin:/root/bin:Jdk_Path

	【释】配置在/etc/profile中的环境变量，每次会话开启，Linux会默认被执行一次，所以对所有的会
	话有效


source和pstree
---------

source：在当前shell会话中执行文件中的所有指令。

	source filename [arguments]	//man文档摘要
	        Read  and  execute  commands   from filename in the current shell envi-
			ronment and return the exit  status of  the  last command executed from 
			filename. 

eg:source

	案例详情见环境变量案例  【eg2|环境变量】

pstree:打印当前Linux中运行的所有进程的进程树

	NAME	//man文档摘要
	       pstree - display a tree of processes

eg|pstree

	# bash 	//新建一个子shell会话
	# pstree 
	├─gnome-terminal──bash───bash───pstree	
	# exit	//退出子shell会话，回到原shell会话
	# pstree
	├─gnome-terminal─┬─bash───pstree

	【释】案例中只截取pstree中部分，用于展示
	



局部变量
---------
在函数体内的变量是局部变量。局部变量的

申明：local VAR_NAME=VALUE   使用时，在函数内使用

eg|局部变量
	# local hah="halo"	//局部变量只能在函数中使用
	bash: local: can only be used in a function

	1 #!/bin/bash
    2 
    3 function main(){
    4         local val="hello" //局部变量的申明
    5         echo $val	//打印
    6 }
    7 
    8 main
    9 
    10 echo $val

	# ./test3.sh 
	hello	//调用main时打印出来
			//第10行代码中，并没有找到变量$val


位置变量
-------------

	$1,$2,.....${10}....
	test.sh 3 89
	$0：脚本自身
	$1：脚本的第一个参数
	$2：脚本的第二个参数
	..

	$?：接收上一条命令的返回状态码
	返回状态码在0-255之间
	$#：参数个数
	$*：或者$@：所有的参数
	$$：获取当前shell的进程号（PID）(可以实现脚本自杀)(或者使用exit命令直接退出也可以使用
		exit [num])

eg|位置变量
	
	# vi fun.sh		//编辑脚本

	  1 #position argument

      2 function main(){
      3         echo $1
      4         echo $2
      5         echo $0
      6 
      7 }
      8 
      9 main 				
     10         echo $0
     11         echo $1
     12         echo $2

	# bash fun.sh  aa bb	//执行脚本
							//第3行未能打出$1
							//第4行未能打出$2
	fun.sh					//第5行能打出$0		
	fun.sh					//第10行执行结果
	aa						//第11行执行结果
	bb         				//第12行执行结果

	【释】方法中无法接收到shell外面穿的参数，只能接受方法参数。但是方法体内也能引用$0.


三种引号
----------

	''单引号不解析变量
	echo '$crxy'
	""双引号会解析变量
	echo "$crxy"
	``反引号是执行并引用一个命令的执行结果，类似于$(...)
	echo `$crxy`
	
	# crxy="hello"
	# echo "$crxy"
	hello
	
	#echo '$crxy'	//你可以理解成为原样输出
	$crxy	

	# command="1+1"
	# `command`
	# echo `$command`
	bash: 1+1: command not found	//执行时出错，会将变量当命令执行

	mydate="date +%Y_%d"
	# echo `$mydate`
	2016_18


for循环
---------
	格式1
	for ((i=0;i<10;i++))
	do
	  ...
	done

	格式2
	for i in 0 1 2 3 4 5 6 7 8 9
	do 
	...
	done

	格式三
	for i in {0..9}
	do
	  ...
	done

笔者试了一下，以下java的循环格式，也可以执行

	for((i=0;i<10;i++))
	{
	        echo  this is for ${i}
	}

if elif else分支语句
-----------------
**if	[条件]	then	..	elif	[条件]	then	..	else	.. 	fi**


分支语句格式

 	if 条件1; 
		then
	分支1
	elif 条件2
		then
	分支2
		else 
	分支n
          fi

case语句
-----------
相当于JAVA中的swich语句

	case 变量引用 in
	PATTERN1)
		分支1
	;;
	PATTERN2)
		分支2
	;;
	...
		*)
	分支n
	;;
	esac

eg|case

	# tail -20 case.sh 
	var=10
	case $var in 
	5)
	 echo "5"
	;;
	10)
	echo "10"
	;;
	*)
	echo "don't know"
	;;
	esac
	
	# bash -x  case.sh //-x可以打印执行过程
	+ var=10
	+ case $var in
	+ echo 10
	10

break，continue
--------------------
和java中的break和continue相同
break，跳出循环体，不再执行循环
continue提过本次循环，执行下次循环

算数表达式
-------------------
let varName=算术表达式

varName=$[算术表达式]

varName=$((算术表达式))

eg|算数表达式

	# let var=1+1	//注意赋值时，变量名(var)和赋值号(=)之间不能有空格，有空格会被认为
	# echo $var
	2

	# var=$[1+1]
	# echo $var
	2	
	
	# var=$((1+1))
	# echo $var
	2


方法
----------
申明一个方法的方式function methodName

调用的时候：methodName

	# tail -20  function.sh //查看脚本内容
	
	function main(){
		echo $1 firstmeber
		echo $0 proceName
		echo $@ all members
		echo $$ proceId
		echo $* all mebbers
		echo $# count
	}
	
	echo $0
	
	main var1 var2 var3	//调用函数
	
	
	# bash function.sh	//执行脚本
	function.sh
	var1 firstmeber
	function.sh proceName
	var1 var2 var3 all members
	4981 proceId
	var1 var2 var3 all mebbers
	3 count
	
date命令
-------------------
显示当前时间

格式化输出 +%Y-%m-%d

格式%s表示自1970-01-01 00:00:00以来的秒数

指定时间输出  --date='2009-01-01 11:11:11'

指定时间输出  --date='3 days ago'

	# echo $(date  +%Y-%m-%d)
	2016-04-18
	
	# echo $(date  --date='1 days ago')		//一天前
	Sun Apr 17 07:00:49 PDT 2016

	# echo $(date  --date='-1 days ago')	//一天后
	Tue Apr 19 07:01:45 PDT 2016	


未完待续
----------