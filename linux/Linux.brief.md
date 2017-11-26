
title: Linux入门
tags: 
	- Linux
	- 入门

categories:
	- Linux
-------------------


Linux简介
---------------
Linux就是一个操作系统，就像你多少已经了解的Windows（xp,7,8）和Max OS,至于操作系统是什么，就不用过多解释了,Linux是基于unix内核的操作系统。1991年由 Linus Torvalds 发布原始版。

linux重要人物
------------------
     Ken Thompson：C语言之父和Unix之父
     Dennis Ritchie：C语言之父和Unix之父
     Stallman：著名黑客，GNU创始人，开发了Emacs、gcc、bash shell
     Linus Torvalds：Linux之父，芬兰赫尔辛基大学


Linux原则
----------------
linux系统的基本原则

     1.由目的的小程序组成     组合小程序完成复杂任务
     2.一切皆文件
     3.尽量避免捕获用户接口	//笔者不是特别明白
     4.配置文件，保存为纯文本
     5.严格区分大小写

Linux Vs windows
-----------------
	windows没有的
			稳定的系统
			安全性和漏洞的快速修补
			多用户	//貌似都一样
			用户和用户组的规划
			相对较少的系统资源占用
			可定制裁剪，移植到嵌入式平台（如安卓设备）
			可选择的多种图形用户界面（如GNOME,KDE）
			为开源而生，Linux系统本生开源的同时，在Linux平台的软件也大多开源	 
		
	
	Linux没有的
		没有特定的支持厂商
		游戏娱乐支持度不足
		专业软件支持度不足

<!-- more -->

Linux应用范畴
-------------
	或许你之前不知道Linux,要知道，你之前在windows使用百度，谷歌，上淘宝，聊QQ时，支撑这些软件
	和服务的，是后台成千上万的Linux服务器主机，它们时时刻刻都在进行着忙碌的数据处理和运算，可以
	说世界上大部分软件和服务都是运行在Linux之上的，什么云计算，大数据，移动互联网，说得风起云涌
	，要没用Linux,全都得瘫（除了微软自家的），相比这应用范畴大家也懂了哈，要没有Linux，Windows
	可干不了几个事

Linux学习攻略
-------------
	明确目的：为开源而生，为开发而学，为搭建强大的服务器。
	面对现实：Linux大都在命令行下操作，能否接受不用或少用图形界面
	学习技巧:熟练掌握查询命令方法，装个Linux系统练习，实践性的东西都交给实践（光是看书是完全不
	够的）。
	
	掌握学习技巧，学会使用Linux自带的使用说明书，如man,info,--help


阅读须知
------------

	【释】：对前文做更多的补充
	eg:代表案例
	案例中的代码都是笔者在Linux检验过，所以命令都用#开头
	本文案例用的是Centos6.5，可能各个平台略有差异。


	
---------------------------------------------------------

命令
----------------------
Linux中交互主要的手段是通过命令

	命令格式 ：【命令名】   【选项】   【参数】
	
	选项：
		短选项用 “-”	表示
		长选项用 “--”表示 

	参数： 命令的作用对象
          

	eg:
	   ls     -a     /root	//查看/root目录下的所有文件
	  【命令】  【选项】  【参数】

	【释】多个短选项是可以连写。如-r -f 可以连写为 -rf 。长选项本来较少，不做讨论

------------------------------------------------------


man命令
------------------
manual的意思，查询某个命令的使用说明。
我觉得这个命令是所有命令中最重要的。可以查询所有命令怎么使用。

man|eg

	#man  man      //查看man命令的使用说明书

	NAME		
	       man - format and display the on-line manual pages
	
	SYNOPSIS  [-acdfFhkKtwW]  [--path]  [-m system] [-p string] [-C config_file]
	       [-M pathlist] [-P pager] [-B browser] [-H htmlpager] [-S  section_list]
	       [section] name 

	OPTIONS
       -C  config_file
              Specify   the   configuration   file  to  use;  the  default  is
              /etc/man.config.  (See man.config(5).)

	**-----------------------**

	释[1]Name：被查询命令的命令名和命令功能说明

	释[2]SYNOPSIS：被查询命令的使用格式。
	使用格式中的符号：
		<>: 必须参数
		[]:可选参数
		...:可以出现多次
		name:命令名

	释[3]OPTIONS：被查询命令的选项



**阅读man文档时可以用到的技巧**

	命令模式 space/空格 向下翻页
	命令模式 b健   向上翻页
	命令模式 q健   推出
	          
	/【搜索的关键字】向后搜索关键字
	？【搜索的关键字】向前搜索关键字
	n:下一个搜索到的关键字
	N:上一个搜索到的关键字

---------------------------------------

type,which,whatis命令
--------------------------------
这三个命令都和man一样，可以查询命令的信息。换句话说，这三个命令的参数也是一个命令

type     【目标命令】：查看目标命令是否是内部命令。（查看命令的类型）如果不是，将输出命令所在位置的路径

which     【目标命令】：查看目标命令所在位置的路径

	which - shows the full path of (shell) commands.

whatis     【目标命令】：查看命令在系统中使用说明的收录情况

	whatis - search the whatis database for complete words.


type|eg

	# type man
	man is hashed (/usr/bin/man)     //man是一个内部命令

	# type python
	python is /usr/bin/python

which|eg

	# which man     
	/usr/bin/man     //man命令的路径

whatis|eg

	# whatis rm
	rm                   (1)  - remove files or directories
	rm                   (1p)  - remove directory entries	//（1p）章介绍删除目录的
	所有说明

	释[1]所有命令的说明书都记录在Linux中的database中，分章节。不同章节的说明书说明不同的用途。
	上面执行命令whatis rm时，查看rm命令的使用说明说的章节。分别为（1）章 删除文件或文件夹，
	（1p）章 删除文件夹映射。此时再结合man命令查询该章节的使用说明。

man|eg

	# man 1p rm	//查询（1p）章节的使用说明书

	RM(1P)                     POSIX Programmer?. Manual                    RM(1P)

	PROLOG
	       This  manual page is part of the POSIX Programmer?. Manual.
	       The Linux implementation of this interface may differ (con-
	       sult  the  corresponding  Linux  manual page for details of
	       Linux behavior), or the interface may not be implemented on
	       Linux.
	
	NAME
	       rm - remove directory entries
	
	SYNOPSIS
	       rm [-fiRr] file...

-----------------------------------------

ls命令
------------
（list的意思）查看目录下的内容

	# man ls     //查看ls的使用说明书
	
	NAME
	       ls - list directory contents	//ls命令的作用列出目录下的内容
	
	SYNOPSIS
	       ls [OPTION]... [FILE]...

字段摘要

	-a, --all	//列出所有，包括隐藏文件（Linux中以点开头的文件）
              do not ignore entries starting with .


  	-l     use a long listing format	//以长格式显示（显示列表元素的更多信息）

 	-d, --directory	
              list directory entries instead of contents, and do not  derefer-
              ence symbolic links  	//显示当前目录的属性，其他的就不显示了

 	-L, --dereference
              when showing file information for a symbolic link, show informa-
              tion for the file the link references rather than for  the  link
              itself //		读不懂，自己查去

	--color[=WHEN]	//用不同颜色显示不同类型的文件
      

	# type ls
	ls is aliased to `ls --color=auto' //表明本ls命令起始是  ls --color=auto 的别名



ls|eg  
  
	# ls -la /root/	
	total 200	//列表200项，案例中有省略
	dr-xr-x---. 25 root root  4096 Apr 14 09:02 .
	dr-xr-xr-x. 21 root root  4096 Apr 14 08:24 ..
	
	# ls -dl	//-d，只显示目录的属性，其他的都不显示
	dr-xr-x---. 25 root root 4096 Apr 16 01:57 .

-----------------------------------------------

cd命令
-------------------
切换目录

	# man cd     //查看cd命令的使用说明书，发现cd命令是bash。在下方查到相关说明

	
	cd [-L|-P] [dir]    //cd命令仅有-P和-L两种选项
	              Change  the  current directory to dir.  //改变当前目录，到[dir] 下面
	
	the -P	//-p 表示用的是物理路径
              option  says  to use the physical directory structure instead of
              following symbolic links 

	the -L	//-L表示后面是符号链接，也就后面讲到的软连接。
			 option forces symbolic links to be fol-lowed.



cd|eg

	# cd /usr/local/	//切换到/usr/local/ 目录下
	# ls -l
	total 36	//列表36项，案例中有省略
	drwxr-xr-x. 2 root root 4096 Sep 23  2011 bin

	【释】虽然说cd有两个参数，-P,-L，但笔者尝试了一下，这两个参数好像却是没起作用。
	甚至可以这样用cd -PL d.ln	  d.ln是一个软连接。笔者使用的是centos 6.5

-----------------------------------------

. .. ~ 路径
-------------------------

.或者./表示当前路径
..或者../表示当前路径的父目录
~或者~/表示当前用户的家目录。家目录将在Linux权限单元介绍
绝对路径：以/开头，如/root/local
相对路径：以文件名，或者.开头，或者..开头


eg

	#cd	/root/local	//绝对路劲，
	#cd	test	//相对路径，切换到当前路径的test文件夹下
	#cd	./test  //等同于	cd	test  切换到当前路径的test文件夹下
	#cd	../test	//切换到父目录下的test目录下，相当于于是当前目录的兄弟关系目录
	#echo "alas cls='clear'" >> ~/.bashrc 	//追加一个别名到【家目录下】的.bashrc
	【释】当前登陆用户为root用户，家目录是/root

--------------------------------------------

mkdir,rmdir命令
------------
mkdir 创建一个空的文件夹/目录
rmdir 删除一个空的文件夹/目录

mkdir的man文档

	NAME
	       mkdir - make directories
	
	SYNOPSIS
	       mkdir [OPTION]... DIRECTORY...
	

	-p, --parents	//如果有多级目录，自动创建父目录
              no error if existing, make parent directories as needed

	-v, --verbose	//创建时，显示创建过程信息
	              print a message for each created directory


rmdir的man文档

	NAME
	       rmdir - remove empty directories
	
	SYNOPSIS
	       rmdir [OPTION]... DIRECTORY...


	-p, --parents	//如果有多级目录，删除祖先目录
              remove DIRECTORY and its ancestors;

	 -v, --verbose	//删除时，显示删除过程信息
              output a diagnostic for every directory processed


mkdir|rmdir|eg
	
	# mkdir -pv a/b/c	//创建多级目录，创建时显示创建过程
	mkdir: created directory `a'
	mkdir: created directory `a/b'
	mkdir: created directory `a/b/c'
	
	# rmdir -pv a/b/c	//删除多级目录，删除时显示删除过程
	rmdir: removing directory, `a/b/c'
	rmdir: removing directory, `a/b'
	rmdir: removing directory, `a'

	# mkdir -p a/b/c	//创建多级目录a/b/c
	# ls
	# mkdir a/d    		//再创建 a/d
	# rmdir -p a/b/c	//删除多级目录，
	rmdir: failed to remove directory `a': Directory not empty	//提示删除a失败
	# ls -l				//发现，并没真正删除文件夹a,因为a中还有d文件夹
	drwxr-xr-x. 3 root root  4096 Apr 17 08:09 a
	# ls a
	d

	【释】**-p并不是递归删除**，可能很多人都觉得效果和递归一样，就直接下一个递归删除文件夹的定
	义。从最后一个案例看出，并不是递归删除。只要父目录中还有文件，父级目录将删除失败。Linux的
	man文档还是比较考究的，很准确，反而很多做技术的不喜欢阅读英文文档，总拿自己感觉作定论，
	最后以讹传讹了。
	  
-------------------------------------------------

pwd 命令
--------------
查看当前目录或位置

	NAME
	       pwd  - print name of current/working direc-
	       tory
	

pwd|eg

	# cd /usr/local/
	# pwd
	/usr/local	//显示当前位置

---------------------------------

touch命令
----------
更改文件的时间戳
创建空文件：在没有该文件的情况下，默认会直接创建一个空文件。


	NAME
       touch - change file timestamps

	SYNOPSIS
	       touch [OPTION]... FILE...

	-c, --no-create	//如果没有该文件，不会创建该文件。
              do not create any files
	
	-a     change only the access time	//仅改变(access time)使用时间
	
	-m     change only the modification time	//仅仅改变修改时间（modify time）

	--time=WORD		//--time=atime等同于 -a  --time=mtime等同于 -m	

	              change the specified time: WORD is access, atime, or use: equiv-
	              alent to -a WORD is modify or mtime: equivalent to -m	



touch|eg
	
	# stat anaconda-ks.cfg //查看anaconda-ks.cfg的时间戳信息
	Access: 2016-04-14 05:05:13.435999672 -0700
	Modify: 2016-04-14 05:05:13.437999672 -0700
	Change: 2016-04-14 05:05:17.910999668 -0700
	
	# date	
	Fri Apr 15 20:48:50 PDT 2016 //查看系统现在时间（如果系统时间不准确，可能会让你误以为
	touch没生效）
	
	//修改access time
	# touch -a  anaconda-ks.cfg 	//更改最后使用时间(access time)
	# stat anaconda-ks.cfg 
	Access: 2016-04-15 20:49:08.056005657 -0700	//access time更改了
	Modify: 2016-04-14 05:05:13.437999672 -0700
	Change: 2016-04-15 20:49:08.056005657 -0700	//Access或Modify更改，这个字段就会更改

	//修改Modify time
	# touch -m  anaconda-ks.cfg	//更改  最后修改时间（Modify time）
	# stat anaconda-ks.cfg 
	Access: 2016-04-15 20:49:08.056005657 -0700
	Modify: 2016-04-15 20:49:23.867005845 -0700
	Change: 2016-04-15 20:49:23.867005845 -0700
	
	//默认touch修改时间戳
	# touch anaconda-ks.cfg //不加参数，两种时间都会更改
	# stat anaconda-ks.cfg 
	Access: 2016-04-15 20:50:31.895005698 -0700
	Modify: 2016-04-15 20:50:31.895005698 -0700
	Change: 2016-04-15 20:50:31.895005698 -0700

	//创建文件
	#touch haima.txt	//当前没有文件hello.txt,touch命令会默认创建hello.txt的空文件
	#touch -c  vicky.txt	//-c，touch命令不会创建vicky.txt文件
	# ls
	anaconda-ks.cfg  Downloads    install.log.syslog  Public     Videos
	Desktop          hello.txt    Music               Templates
	Documents        install.log  Pictures            test

	【释】stat命令可以查看文件的属性，本案例中只选取了文件时间戳的属性，其他属性此处省略

-----------------------------------------------------------------
	
mv命令
----
移动文件或文件夹，相当于windows中的剪切功能
重命名文件或文件夹

	NAME
	       mv - move (rename) files
	
	SYNOPSIS
	       mv [OPTION]... [-T] SOURCE DEST
	       mv [OPTION]... SOURCE... DIRECTORY

	Rename SOURCE to DEST, or move SOURCE(s) to DIRECTORY.


	--backup[=CONTROL]	//复制时，目标文件如果存在，将备份已经存在的文件
	          make a backup of each existing destination file

	-n, --no-clobber	//不覆盖已存在文件
              do not overwrite an existing file	

	-i, --interactive	//如果需要覆盖，会征求用户的同意。
              prompt before overwrite

	-f, --force	//如果会覆盖，不给用户提示信息，和-i相反
              do not prompt before overwriting

	-b     //和--backup相同功能，但是不需要再跟其他参数
		like --backup but does not accept an argument

	【解】--backup[=CONTROL]中CONTROL可以有多种，如simple，numbered，off.更多请查询man文档
		所以可以组合成：mv --backup=simple haima.txt ./test
					   mv --backup=numbered haima.txt ./test	


mv|移动|eg

	# mv vicky.txt  ./test/	//移动vicky.txt到 ./test目录下

	# touch vicky.txt	//再次创建vicky.txt
	# mv --backup=numbered  vicky.txt ./test/	//如果需要覆盖，采用numbered的方式备份
	mv: overwrite `./test/vicky.txt'? y	//想想为什么会冒出一行提示？

	# ls ./test/
	vicky.txt  vicky.txt.~1~	//第一次移动进来的vicky.txt备份为“vicky.txt.~1~”了

	# type mv
	mv is aliased to `mv -i'	//原来mv是 mv -i的别名。执行mv时相当于已经加入了-i选项了。

	# touch vicky.txt
	
	# mv -b vicky.txt ./test/ 	//使用-b代替--backup  -b会使用上次设置的备份方式
	mv: overwrite `./test/vicky.txt'? y
	# ls ./test/
	vicky.txt  vicky.txt.~1~  vicky.txt.~2~

mv|重命名|eg

	# touch vicky.txt	再次创建vicky.txt
	# mv vicky.txt  haima.txt	//vicky.txt 重命名为haima.txt

	# mv -b  haima.txt  ./test/vicky.txt	//重命名和移动一步到位
	mv: overwrite `./test/vicky.txt'? y		//有备份参数 -b，放心的输入y，确认操作
	# ls ./test/
	vicky.txt  vicky.txt.~1~  vicky.txt.~2~  vicky.txt.~3~

-----------------------------------------------------

cp命令
-----------
拷贝文件或文件夹

	NAME
	       cp - copy files and directories
	
	SYNOPSIS
	       cp [OPTION]... [-T] SOURCE DEST
	       cp [OPTION]... SOURCE... DIRECTORY
	
	Copy SOURCE to DEST, or multiple SOURCE(s) to DIRECTORY.
	

	-R, -r, --recursive	//递归拷贝。目标文件夹中如果还有子文件，需要用到该参数
              copy directories recursively

	-l, --link	//创建该文件的链接，而不再拷贝文件本身
              link files instead of copying


	-s, --symbolic-link	//创建软链接，而不再拷贝文件，相当于创建了一个快捷方式
              make symbolic links instead of copying

	-i, --interactive	和mv指令相同，如果目的地已经有文件了，提示用户
              prompt before overwrite (overrides a previous -n option)

	-f, --force	//和mv指令相同，如果目的地已经有文件了，不给用户提示，直接覆盖
              if an existing destination file cannot be opened, remove it  and
              try again (redundant if the -n option is used)

	--backup[=CONTROL]	//和rm命令相同，备份要覆盖的文件
              make a backup of each existing destination file


cp|eg

	# cp -r test/  test2/	//递归拷贝test文件夹，拷贝到当前文件夹下，并命名为test2
	# ls -l
	total 100	//列表有所省略
	drwxr-xr-x. 3 root root  4096 Apr 16 01:24 test
	drwxr-xr-x. 3 root root  4096 Apr 16 01:24 test2
	
	# cp -l vicky.txt  viky.lin
	# cp -s vicky.txt  viky.sy
	# ls -l
	-rw-r--r--. 2 root root     0 Apr 16 01:27 vicky.txt
	drwxr-xr-x. 2 root root  4096 Apr 13 21:07 Videos
	-rw-r--r--. 2 root root     0 Apr 16 01:27 viky.lin
	lrwxrwxrwx. 1 root root     9 Apr 16 01:29 viky.sy -> vicky.txt //

	【释】软连接就是windows中的快捷方式，大小比较小。删除文件本身时，软连接失效。
		硬链接相当于对于源文件的复制，删除时互不影响。但编辑时，任意操纵一个，另外一个也会有
		相同改变。硬链接不能是目录。
-------------------------------------------------


rm指令
---------
删除文件，文件夹

	NAME
	       rm - remove files or directories//删除文件，路径，
	
	SYNOPSIS
	       rm [OPTION]... FILE...	



	-f, --force	//和mv文件中的-f相似，不询问用户
              ignore nonexistent files, never prompt

 	-i     prompt before every removal	//和mv文件中的-f相似，询问用户

 	-r, -R, --recursive	//递归删除，文件夹包含子文件夹的情况
              remove directories and their contents recursively

rm|eg

	# rm vicky.txt	//删除文件
	rm: remove regular empty file `vicky.txt'? y

	# type rm
	rm is aliased to `rm -i'	//rm相当于rm -i
	
	# rm -f vicky.txt.~ vicky.txt.~2~ vicky.txt.~3	//一次删除三个文件，并且不询问

	# cd ..
	# rm test	//删除文件夹失败
	rm: cannot remove `test': Is a directory
	
	
	# rm -r test	//删除文件夹。如果使用  rm -rf test 则下面不会出现我询问提示
	rm: descend into directory `test'? y
	rm: remove regular file `test/.hello.txt.swp'? y
	rm: remove regular empty file `test/vicky.txt.~1~'? y
	rm: remove directory `test'? y

------------------------------------


总结
-------
本文主要介绍了使用特别频繁的一些命令。更多的是以阅读命令使用说明书的形式来阐述每个命令的。更多关于Linux的学习文章，将陆续更新。笔者也是初学，记录学习Linux的学习过程，不足之处，见谅。