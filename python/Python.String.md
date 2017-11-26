
title: python 字符串string
tags: 
	- python
	- string

categories:
	- python
-------------------


基本字符串操作
-----------------------------------------
索引、分片、乘法、判断成员资格、长度、最大值、最小值、加链

	>>> str="wo shi haima"
	>>> str[5]
	'i'
	>>> str[5:]
	'i haima'
	>>> str*2
	'wo shi haimawo shi haima'
	>>> "haima" in str
	True
	>>> len(str)
	12
	>>> max(str)
	'w'
	>>> min(str)
	''
	>>> str+" vick's brother"
	"wo shi haima vick's brother"


字符串格式化
--------------------------------------------
目标字符串以%为标记，格式化用%作为运算符 后面跟随要格式化替换的内容


	>>> "woshi %s"  %   "haima"   //被替换的目标字符串中%s  s表示替换者类型
	'woshi haima'

常见替换者类型符            类型
     
     i                              十进制整数
     f,F                           浮点型
     s                             字符串（使用str转化任意python对象）
     r                             字符串（使用repr转换任意python对象）
     C                             单字符
     

<!-- more -->


举例

	>>> from math import pi
	>>> "%10f" % pi        //10 可以指明替换宽度
	'  3.141593'

	>>> from math import pi
	>>> "%10.2f" % pi   // f前面点号后面可以指定一个保留小数位数
	'      3.14'

	>>> from math import pi
	>>> "%-10.2f" % pi   //符号（-）可以指明左对齐
	'3.14      '

find()  查找 
-------
返回查找到的第一个子字符串的索引，如果找不到，则返回-1

	>>> "hello I am haima".find("haima")
	11
	>>> "hello I am haima".find("haimd")
	-1



split() 切割方法
------------------------      
返回一个被分割后的序列

	>>>"hello I am haima".split(" ")
	['hello', 'I', 'am', 'haima']

	>>> "hello I    am haima".split(" ")     //*连续出现空格是，切割容易出错
	['hello', 'I', '', '', '', 'am', 'haima']


join()方法  
----------------
split() 的一个逆运算

	>>> " ".join(['hello', 'I', 'am', 'haima'])
	'hello I am haima'


lower()方法    
------------------
返回字符串的小写

	>>> "WO SHI HAIMA".lower();
	'wo shi haima'


upper()方法
-------------------
返回字符串的大写

	>>> "wo shi haima".upper();
	'WO SHI HAIMA'


title()方法
------------
可以将string中的单词首字母大写

	>>> "I'm haima,Vicky's brother".title();
	"I'M Haima,Vicky'S Brother"

capwords()方法
-------------------
可以看出title不算特别友好，在句子中有分隔符的时候

	>>> capwords("I'm haima,Vick's brother")
	"I'm Haima,vick's Brother"


replace()    
-------------
替换

	>>> "I am haima,I am a java developer.".replace("I am","He is")
	'He is haima,He is a java developer.'	//He is 替换 I am


strip() 
------------------
去除两端空格

	>>> "      I am rob or haima.    ".strip();
	'I am rob or haima.'


其他
-----------
单引号和双引号都能作为字符串的开始和结束标记。
字符串中间不能出现此字符串的开头或结束标记。如果需要，则用转移字符\

	>>> "I am 'haima' "
	"I am 'haima' "
	
	
	>>> 'I am "haima" '
	'I am "haima" '
	
	>>> 'I am \'haima\' '     //注意转移字符
	"I am 'haima' "




