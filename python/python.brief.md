
title: python入门
tags: 
	- python
	- 入门

categories:
	- python
-------------------


python的使用
------
	Linux中基本都自带安装了python编译器，其他安装方式待查询。
	
	使用python命令，
	     -V 查看版本
	     安装python 通常linux 自带安装了python 
	     直接在linux环境中使用命令python即可进入python交互式解释器
	
	man python 一下，查看python的基本参数


运算符
-----------------------------------------
	>>> 1/2   //整除
	0
	
	>>> 1.0/3   //浮点型
	0.33333333333333331
	>>> 1/3.0
	0.33333333333333331
	>>> 1/2.0
	0.5


	>>> 1.0//2  //浮点型强制整除
	0.0


	>>> 10%3     //取余
	1


	>>> 2**5  //幂运算     **表示 幂运算
	32

<!-- more -->


类型
-----------

	>>> 100000000000000000000000000000000     //自动转型为Long型
	100000000000000000000000000000000L  
	
	>>> x=3        //变量
	>>>u x
	3


函数
----------
	input() 函数  提供用户标砖输入
	
	>>> x=input()
	"dd"
	>>> x
	'dd'
	
	
	>>> x=input("input a word number: \n")
	input a word number:
	50
	>>> print x
	50
	
	
	pow(a,b) 函数 也是幂运算，和**效果相同
	
	>>> pow(2,5)
	32




	>>> print 4
	4
	>>> print (3)   //新版python中print变成函数，不仅是表达式
	3


模块引入
-------------

	>>> import math    //引入模块，有点像java中包，类引入
	>>> math.floor(3.9)
	3.0
	
	
	>>> import cmath   //复数运算   引入cmath模块
	>>> cmath.sqrt(-1)
	1j



python 脚本
----------------

	编写python脚本，用纯文本文档
	和bash shell相似，在文本的开头有一句声明#!/usr/bin/env python(bash 的声明为#!/bin/bash)
	
	和bash一样，用文本编辑器编辑好脚本后，chmod u+x  test.py   授权，使得test.py拥有 执行权限
	执行test.py和 bash执行相同。./test.py执行。
	
	脚本内容--------------------------------------
	#!/usr/bin/env python
	#My first python shell
	print "I LOVE YOU"
	print "Do you love me too?"
	--------------------------------------------------
	------------------------------------执行效果
	# ./test.py    
	I LOVE YOU
	Do you love me too?
	--------------------------------------------------

repr()  和``
--------------------------------------
将表达式 转为结果，字符串

	# ./test.py----------------
	#!/usr/bin/env python
	#my first python shell
	print "I LOVE YOU"
	print "Do you love me too?"
	print repr("Hello world")
	print repr(1000L)
	print "hello" + repr(100)
	print "hello"+ repr(1234*4321) # repr can convert express to str
	print "hello"+ `123*321`  # `` function as same as repr
	#print 123+ `22*441` # str can't  convert to int  
	
	# ./test1.py ------------------执行效果
	I LOVE YOU
	Do you love me too?
	'Hello world'
	1000L
	hello100
	hello5332114
	hello39483        


input 和  raw_input 
---------------------
区别：raw_input 接受用户标准输入流时，不需要使用双引号作为起始标记

	#!/usr/bin/env python
	#input vs raw_input
	x=input("input your name:\n")
	y=raw_input("raw_input your name:\n")
	
	print x
	print y
	
	# ./test3.py
	input your name:
	"wo shi haima"	//imput 用户输入时，需要双引号
	raw_input your name:
	woshi haima	//raw_input	用户直接标准输入
	
	wo shi haima
	woshi haima



print 和  print r  
----------------------
区别：print r  会保持原字符输出。print  会识别转义字符

	>>> print "hello \n hallo"
	hello 
	 hallo	//转义后，换行输出了

	>>> print r"hello \n hallo";
	hello \n hallo	//“\n未转义，直接输出”
