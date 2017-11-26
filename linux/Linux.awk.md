title: Linux之强大的awk
tags: 
	- Linux
	- 数据分析

categories:
	- Linux
-------------------

# awk简介

awk是Linux中的一个命令，用来做文本处理与分析，功能简单强悍，同时它也是一门编程语言。
awk处理文本文件时，以行为单位，可以高效的对日志文件进行处理。


awk的man文档简介摘要：

	NAME

       gawk - pattern scanning and processing language	//awk其实是gawk，文本匹配查询和
	处理语言，

	SYNOPSIS

		gawk [ POSIX or GNU style options ] -f program-file [ -- ] file ...
		gawk [ POSIX or GNU style options ] [ -- ] program-text file ...

		pgawk [ POSIX or GNU style options ] -f program-file [ -- ] file ...
		pgawk [ POSIX or GNU style options ] [ -- ] program-text file ...

 	
		Gawk is the GNU Project?. implementation of the AWK programming language.  It
		conforms to the definition of the lan-
		guage in the POSIX 1003.1 Standard.  This version in turn is based on the 
	description in The  AWK  Programming  Lan-
		guage,  by  Aho,  Kernighan, and Weinberger, with the additional features
	 found in the System V Release 4 version of
		UNIX awk.  Gawk also provides more recent Bell Laboratories awk extensions, 
	and a number of GNU-specific extensions.


    

	【释】第一段英文大致说了，Gawk是GNU(通用许可开源)项目，源于unix awk，并支持更多的贝尔实验
	室awk的扩展。看起来Linux中的awk(gawk)是很牛叉的。废话不多说，马上就来认真学习一下。


# 使用方式
命令常见使用方式如下

	方式一：awk [options] 'script' [var=value] file(s) 
	方式二：awk [options] -f scriptfile [var=value] file(s)
	
	【解释】 其中 options是命令选项。方式一：'script'指awk的处理程序，执行程序。var=value是
	变量，file(s) 表示被处理的文本文件，可以一次性处理多个文本文件。方式二中-f scriptfile
	表示将处理程序放到一个文本中，当作脚本文件。


# 选项-F fs
指定分隔符，awk中的默认分隔符是空格 " "，默认以空格符分隔行文本。

选项在man文档中的摘要

	-F fs
       --field-separator fs	//指定分隔符
              Use fs for the input field separator (the value of the FS prede-
              fined variable).	
eg

	[案例1] //默认字段分隔符为" "
	# echo "hello hh"| awk '{print $1;print $2}'
	hello
	hh
	
	【解释】 案例中，并没有出现-F选项，此时默认分隔符为" " 将hello hh分隔为两段，再分别打印
	字段一：“hello”,再打印字段二：“hh”。$1和$2是位置变量，分别表示第一个字段和第二个字段。

	[案例2]
	# echo "hello:hh"| awk -F :  '{print $1;print $2}'
	hello
	hh
	
	【解释】通过-F指定分隔符为":" 分隔 hello:hh ，然后同 [案例1]



<!-- more -->

# 位置变量 $0,$1,$2..
$n 是位置参数，表示第n个字段或者说分隔后 第n位置上的字符串
$0表示整行
$NF：表示倒数第一个位置上的字符串。$NF-1表示倒数第二个位置，以此类推

eg
	[案例1]
	# cat pass.log //要处理的目标文件 pass.log  笔者截取了一段/etc/passwd 
	root:x:0:0:root:/root:/bin/bash
	bin:x:1:1:bin:/bin:/sbin/nologin
	daemon:x:2:2:daemon:/sbin:/sbin/nologin

	# awk -F : '{print ;}' pass.log 
	root:x:0:0:root:/root:/bin/bash
	bin:x:1:1:bin:/bin:/sbin/nologin
	daemon:x:2:2:daemon:/sbin:/sbin/nologin
	
	【解释】 "print;" 默认也是打印整行，同$0 

	
	[案例2]
	# awk -F : '{print $0;}' pass.log 
	root:x:0:0:root:/root:/bin/bash
	bin:x:1:1:bin:/bin:/sbin/nologin
	daemon:x:2:2:daemon:/sbin:/sbin/nologin

	【解释】 "print $0;" 打印整行


eg

	# awk -F : '{print $1 "   "  $NF;}' pass.log 
	root   /bin/bash
	bin   /sbin/nologin
	daemon   /sbin/nologin

	【解释】打印被 ":" 分隔后第一个位置上的字符串，然后再打印最后一个位置，中间输入空格方便查看


# 选项-f program-file	
将脚本写到文本文件中去,通过选项-f将awk命令的处理程序指定到某个脚本文件中，当处理程序复杂的时候通常采用这种方式。

man文档摘要

	-f program-file	
	    --file program-file	//--file 和-f效果相同
	         Read the AWK program source from the file program-file,  instead
	         of  from  the  first  command  line  argument.   Multiple -f (or
	         --file) options may be used.
	
eg

	[案例一]
	
	# vi sc.awk	//编辑脚本
	{print $1 "   "  $NF} //将执行文件写入 sc.awk中
	//vi编辑器，保存sc.awk文档退出。


	# awk -F : -f sc.awk  pass.log  //pass.log和前文中pass.log相同。
	root   /bin/bash
	bin   /sbin/nologin
	daemon   /sbin/nologin

	【解释】 将前文中案例 【#awk -F : '{print $1 "   "  $NF;}' pass.log】 中的
	'{print $1 "   " $NF;}'写到了 sc.awk中，此时，处理程序不用再用放在单引号''中了。

	【注意】如果对vi不熟悉，可以用echo "{print $1 "   "  $NF}" >sc.awk  来新建程序文件


# awk程序结构
awk处理程序由  BEGIN，主体，END 构成
BEGIN语句块 BEGIN{}

eg

	# vi sc.awk	//编辑脚本
	# awk program's Begin block  //【注意】这句话是脚本里的注释，不是命令
	BEGIN{print "here is the awk begin!!"} //BEGIN语句快
	{print $1 "   "  $NF}	


	# awk -F : -f sc.awk  pass.log 
	here is the awk begin!!	//BEGIN语句块 的输出效果 
	root   /bin/bash
	bin   /sbin/nologin
	daemon   /sbin/nologin

	【解释】 上面案例可以看出，BEGIN语句块中的代码会在程序最前面执行。且只执行一次。


主体部分，主题部分用 {} 表示，写在BEGIN之后，END之前。前文中使用的'{print}'都是程序主体。

eg

	# vi sc.awk	//编辑脚本
	# awk program's main block //【注意】这句话是脚本里的注释，不是命令
	BEGIN{print "here is the awk begin!!"}
	{
	        print "this is main"
	        print $1
	}

	# awk -F : -f sc.awk  pass.log 
	here is the awk begin!!
	this is main	// 执行 主体部分中的　print "this is main"
	root			// 执行 主体中的print $1
	this is main
	bin
	this is main
	daemon


	【解释】 主体中的代码，没处理一行就会被执行一次。

END语句块
和BEGIN语句块相似 END{} 命令最后会被执行，也仅被执行一次 

eg

	# vi sc.awk	//编辑脚本
	# awk program's end block //【注意】这句话是脚本里的注释，不是命令
	BEGIN{print "here is the awk begin!!"}
	{
		print $1
	}
	
	END{	
		print "ok,awk operate to the end"	//end模块打印语句
	}

	# awk -F : -f sc.awk  pass.log 
	here is the awk begin!!
	root
	bin
	daemon
	ok,awk operate to the end	//end模块的执行效果


# 变量
在awk的处理程序中可以申明变量，和使用变量。
申明方式 varName=varValue  

eg | 通过变量，计算文件的行数

	# vi sc.awk	//编辑脚本
	# awk program's var block //【注意】这句话是脚本里的注释，不是命令
	BEGIN{
			nu=0;			//申明一个变量表示行号
	}
	
	{
		nu ++;				//每处理一行数据，行号自动增加1
		print nu " " $0 	//输出行号和 第一个字段
	}
	
	END { 
		print "totall lines is " nu;	//最后打印总共的行数，即最后一行行号
	}

	# awk -F : -f sc.awk  pass.log 
	1 root:x:0:0:root:/root:/bin/bash
	2 bin:x:1:1:bin:/bin:/sbin/nologin
	3 daemon:x:2:2:daemon:/sbin:/sbin/nologin
	totall lines is 3

	【解释】 nu=0为变量申明，使用的时候，直接使用nu即可。不用像bash里用$打头


# 命令变量
我把它叫做命令变量，是因为变量是从命令中传入的，而并非awk命令中自定义的。
从命令外部引入变量，可以通过命令选项 [var=value] 把shell中的变量引入到awk处理程序中

	方式一： awk [options]  -v varname=varvalue 'script' file(s) 
	方式二： awk [options] 'script' varname=varvalue  file(s) 
	
	【解释】方式一是通过-v选项传入，方式二是 man手册里的选项[--]

eg | 方式一
	
	# awk -F : -v  aa="haima"  '{print aa " " $1}' pass.log 
	haima root
	haima bin
	haima daemon

	【解释】 {print aa " " $1} 打印了命令中传入的变量aa,aa的值是haima。然后再打印 $1

eg | 方式二

	# awk -F :  '{print aa " " $1}' aa="haima"  pass.log 
	haima root
	haima bin
	haima daemon


# 内置变量
内置变量是awk程序中系统自带的变量，分别都有各自的意义

	FS 字段分割符，在程序中【FS=":";】相当于 命令中使用 【-F :】
	IGNORECASE 是否忽略大小写，IGNORECASE=true则匹配时不区分大小写 
	NF 表示字段数，在执行过程中对应于当前的字段数 
	NR 表示记录数，在执行过程中对应于当前的行号
	OFS 输出字段分隔符（默认值是一个空格）
	ORS 输出行记录分隔符（默认值是一个换行符）

	IGNORECASE在笔者系统上没测试通过 centos6.5

	
eg	| FS 


	# vi sc.awk 
	# awk program's FS  //【注意】这句话是脚本里的注释，不是命令
	BEGIN{
	        FS=":" //用HAIMA 去分割，
	}
	
	{
	        print $1
	}

	# awk -f sc.awk  pass2.log 
	root
	bin
	daemon

	【解释】 FS=":" 需要在BEGIN中指定，效果与令行中使用 -F :相同


eg | NF NR

	# vi sc.awk 

	# awk program's NF NR //【注意】这句话是脚本里的注释，不是命令
	{
	        print  NR   " totall fileds" NF // 打印行号，打印切割后的段
	}
	
	END{
	        print "line numbers:" NR //END中能打印到行数
	
	 }

	# awk -F:  -f sc.awk pass.log 
	1 totall files:7
	2 totall files:7
	3 totall files:7
	line numbers:3

	【注意】 在END中不能打印到字段数NF


eg|OFS  ORS

	# awk -F : 'BEGIN{OFS="|";ORS=" LinEnd\n"}{print $1, $2;}'  pass.log 
	root|x LinEnd
	bin|x LinEnd
	daemon|x LinEnd


	【解释】 在BEGIN模块中申明了OFS="|"，输出字段分隔符是“|” ，而 ORS=" LinEnd\n" 申明了输出
	行分隔符为"linEnd\n"  其中"\n"是 换行符，如果没有“\n” ,将直接输出一行
	
	【注意】 “{print $1, $2;}” 时，必须用 "," 连接。如果 {print $1 $2} 字段分隔符将失效。因
	为空格在awk程序中可以连接两个字符串。

# awk的工作原理
通过前面的案例，已经大概了解awk的功能。下面再总结一下awk的工作流程。

	awk 'BEGIN{ commands } pattern{ commands } END{ commands }'  
	
	第一步：执行BEGIN{ commands }语句块中的语句；

	第二步：从文件或标准输入(stdin)读取一行，然后执行pattern{ commands }语句块，它逐行扫描
	文件，从第一行到最后一行重复这个过程，直到文件全部被读取完毕。

	第三步：当读至输入流末尾时，执行END{ commands }语句块。 

	BEGIN语句块在awk开始从输入流中读取行之前被执行，这是一个可选的语句块，比如变量初始化、打
	印输出表格的表头等语句通常可以写在BEGIN语句块中。

	END语句块在awk从输入流中读取完所有的行之后即被执行，比如打印所有行的分析结果这类信息汇总
	都是在END语句块中完成，它也是一个可选语句块。 

	pattern语句块中的通用命令是最重要的部分，它也是可选的。如果没有提供pattern语句块，则默认
	执行{ print }，即打印每一个读取到的行，awk读取的每一行都会执行该语句块。

	【注意】awk中的内置变量。不同的内置变量的作用域有所不同。使用的时候，需要注意在awk执行的某
	些阶段，该内置变量是否可用。


# 流程控制
if ... else if... else
awk可以用类似java中的if语句来做逻辑控制

语法结构如下：

	if(表达式)
		{语句1} 
	else if(表达式) 
		{语句2}
	else
		{语句3} 


eg 

	# vi nsc.awk  //新建一个awk程序
	# test for  "if.. else if.. else " //程序注释，非指令
	{
	 if(NR%2==0)
		{ print $2 " evenline"}	//偶数行就 输出第二个字段和 evenline
	else if (NR%3==0) 
		{print $3 " thrirdline"} //被3整除的行，就输出第三个字段 和 thrirdline
	else
		 {print $NF " othersLine"} //其他就输出 最后一个字段 和 othersLine

	# awk -F :  -f nsc.awk  /etc/passwd 
	/bin/bash othersLine
	x evenline
	2 thrirdline
	x evenline
	/sbin/nologin othersLine


# 循环语句
awk也支持for,while,do..while三种循环语句，语法与java中基本相同


## for 循环  

格式： for(变量;条件;表达式) {语句}

	# awk 'BEGIN{ total=0; i=0; do {total+=i;i++;} while(i<=100) print total; }'
	5050


## while循环

格式：while(表达式) {语句}

	# awk 'BEGIN{ test=100; total=0; while(i<=test){ total+=i; i++; } print total; }'
	5050

## do..while

格式：do {语句} while(条件)

	# awk 'BEGIN{ total=0; i=0; do {total+=i;i++;} while(i<=100) print total; }'
	5050

	# awk 'BEGIN{ total=0; i=1000; do {total+=i;i++;} while(i<=100) print total; }'
	1000	//虽然i一开始就满足i<=100  但do中还是执行了一次，所以打印total为 1000

	【注意】do.. while 循环语句至少一次循环，先执行do里面的内容，再判断，所以这样。


## break,continue
awk同样也支持break和continue控制。break和continue必须在循环体中使用。
break是直接跳出循环，不再执行循环
continue是跳到下次循环

# next控制
next 从单词的意思看起来和我continue相似。确实如此，next具有continue很相似的功能，只是next是跳出本行数据处理，处理下一行数据，而continue是跳出本次循环，执行下次循环。

eg | 跳过 行号为6-9行的数据

	# awk -F :  '{if(NR>5&&NR<10)next; print NR, $1}' /etc/passwd
	1 root
	2 bin
	3 daemon
	4 adm
	5 lp
	10 uucp
	11 operator
	12 games
	13 gopher
	14 ftp
	15 nobody

	【解释】 if(NR>5&&NR<10)next; 当行号在 6-10，直接跳过本行数据的处理。
	【注意】 因为next是跳过某个记录，或者说某行数据的处理，所以next只能出现在awk的主体中，不能
	出现在BEGIN 块，也不能出现在 END模块中，会报错。
	
# exit 控制
退出当前还没有执行的主体内容，如果END块中有内容，则执行END块中的内容。此时就会出现三种情况。

情况一：exit在BEGIN块中使用，从exit当前位置退出，直接执行END块
情况二：exit在主体中使用，从exit当前位置退出，直接执行END块
情况二：exit在END块中使用，从exit当前位置退出，已经退出END快，所以不再执行了。

eg | exit在主体中使用。

	# awk -F :  '{if(NR>5){exit};print NR, $1}END{print "end"}' /etc/passwd
	1 root
	2 bin
	3 daemon
	4 adm
	5 lp
	end
	
	【解释】当行号NR>5时，退出主体，再执行print "end"

eg | exit在BEGIN或END中使用

	# awk -F : '{print"begin";exit;print"hh"}{print NR, $1}END{print "end"}' /etc/passwd
	begin
	end

	【解释】print "begin" 执行完后，直接跳到了END模块执行 print "end"

	
# awk中的正则表达式
awk可以利用正则表达式来筛选数据，以达到更强大的处理能力


## 用于记录筛选
记录筛选，以一条记录作为筛选的单位

awk [options] 'script' [var=value] file(s) 在此主体格式不变的情况下，在script前面加入筛
选，此时，'script' 程序形式变成：'(/express/){awk处理程序}'，其中/express/是正则表达式

手动创建一个文件target.log 并写好内容如下

	# cat target.log //查看target.log
	27.19.74.143 - - GET /static/image/common/faq.gif HTTP/1.1" 200 1127
	110.52.250.126 -  GET /data/cache/style_1_widthauto.css?y7a HTTP/1.1" 200 1292
	27.19.74.143 - - GET /static/image/common/hot_1.gif HTTP/1.1" 200 680
	27.19.74.143 - - GET /static/image/common/hot_2.gif HTTP/1.1" 200 682
	27.19.74.143 - - GET /static/image/filetype/common.gif HTTP/1.1" 200 90
	110.52.250.126 -  GET /source/plugin/wsh_wx/img/wsh_zk.css HTTP/1.1" 200 1482
	110.52.250.126 -  GET /data/cache/style_1_forum_index.css?y7a HTTP/1.1" 200 2331
	110.52.250.126 -  GET /source/plugin/wsh_wx/img/wx_jqr.gif HTTP/1.1" 200 1770
	27.19.74.143 - - GET /static/image/common/recommend_1.gif HTTP/1.1" 200 1030
	110.52.250.126 -  GET /static/image/common/logo.png HTTP/1.1" 200 4542
	110.44 -  GET /static/image/common/logo.png HTTP/1.1" 200 4542
	
	【注意】 最后一行数据是  不正常的数据。ip 的格式不对
	

eg | 筛选出包含.gif的行
	
	# awk '(/gif/){print $0}' target.log 
	27.19.74.143 - - GET /static/image/common/faq.gif HTTP/1.1" 200 1127
	27.19.74.143 - - GET /static/image/common/hot_1.gif HTTP/1.1" 200 680
	27.19.74.143 - - GET /static/image/common/hot_2.gif HTTP/1.1" 200 682
	27.19.74.143 - - GET /static/image/filetype/common.gif HTTP/1.1" 200 90
	110.52.250.126 -  GET /source/plugin/wsh_wx/img/wx_jqr.gif HTTP/1.1" 200 1770
	27.19.74.143 - - GET /static/image/common/recommend_1.gif HTTP/1.1" 200 1030

eg | 筛选出数据不正常的行（target.log中最后一行数据为错误数据）

	#awk --posix '!/([1-9][0-9]{0,2}\.){3}[1-9][0-9]{0,2}/{print $0}' target.log
	110.44 -  GET /static/image/common/logo.png HTTP/1.1" 200 4542  

	【解释】 筛选行，首先选出行中含有IP格式，ip格式用/([1-9][0-9]{0,2}\.){3}[1-9][0-9]{0,2}/
	匹配，前面加 ！ 表示取非，也就是不含ip格式的行，被打印出来

	【注意】笔者查询了一下， 在gawk 4.0以下版本中使用{n,m}这样的正则，需要在在前面加上
	--re-interval或--posix选项。

## 用于字段筛选
awk也可通过筛选某个字段，来筛选某一行 
在正则表达式前面 $n ~ 表示 $n字段满足正则筛选  $n表示第n个字段
在正则表达式前面 $n !~ 表示 $n字段不满足正则筛选 $n表示第n个字段
使用$0~ 也可以对行进行筛选，$0表示整行的意思

eg| 用 ":" 切割/etc/passwd，筛选出用户名长度为4的用户

	# awk --posix -F :  '$1~/^.{4}$/{print $1}' /etc/passwd
	root
	sync
	halt
	mail
	uucp
	dbus
	vcsa
	abrt
	sshd

	【注意】笔者查询了一下， 在gawk 4.0以下版本中使用^$这样的正则，需要在在前面加上
	--re-interval或--posix选项。

eg| 用 ":" 切割/etc/passwd，筛选出用户名长度不为4的用户

	# awk --posix -F :  '$1!~/^.{4}$/{print $1}' /etc/passwd
	bin
	daemon
	adm
	lp
	shutdown
	operator
	games
	gopher
	ftp
	nobody
	usbmuxd
	rtkit
	//此处省略部分结果行

	【注意】笔者查询了一下， 在gawk 4.0以下版本中使用^$这样的正则，需要在在前面加上
	--re-interval或--posix选项。

## 用在表达式中
正则表达式用在返回值为true/false的表达式中，同样用~，!~匹配，匹配成功则返回true

eg | 处理/etc/passwd 匹配行数为二十多的 

	# awk --posix -F :  '{if(NR ~ /^[2][0-9]$/)print NR,$1}' /etc/passwd
	20 avahi-autoipd
	21 abrt
	22 haldaemon
	23 gdm
	24 saslauth
	25 postfix
	26 ntp
	27 apache
	28 pulse
	29 sshd

	【解释】 NR为内置变量行号，行号匹配正则表达式为/^[2][0-9]$/，则打印行号NR，第一个字段

