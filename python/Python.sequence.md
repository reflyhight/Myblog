title: python 序列sequence
tags: 
	- python

categories:
	- python
-------------------

简介
---------------
有点像数组，可以用下标索引访问到容器中的数值，索引以0开始
序列中可以是不同类型的内容
字符串也能用索引脚标访问
逆序访问序列的元素，用负数表示索引

	>>> arr=["haima",1] 
	>>> arr[0]  
	'haima'
	>>> arr[1]
	1
	
	
	>>> name="haima" 
	>>> name[3]
	'm'
	
	>>> arr=["haima",1]  
	>>> arr[-1]
	1


基本操作
----------------
序列相乘，序列相加


	>>> arrbig=arr+["hailong",2]   //序列相加：简单的元素组合
	>>> arrbig
	['haima', 1, 'hailong', 2]


	>>> arrM=arr*5    //序列*数字   元素的简单重复
	>>> arrM
	['haima', 1, 'haima', 1, 'haima', 1, 'haima', 1, 'haima', 1]



<!-- more -->

序列分片
--------------
	arr[index1:index2:length] 
	     index1开始的元素，index2结束的元素，length 步长。
	     分片是，包含index1，不包含index2
	     步长默认为1
	     如果从首尾开始，则可以空index1，index2

	>>> arr=[1,2,3,4,5,6,7,8,9,10]  
	>>> arr[3:5]
	[4, 5]
                                                                                                                                                                                                                                                                              
   
	>>> arr=[1,2,3,4,5,6,7,8,9,10]   //有步长的
	>>> arr[3:8:2]
	[4, 6, 8]



	>>> arr=[1,2,3,4,5,6,7,8,9,10]   //逆序，脚标为负
	>>> arr[-5:-1]
	[6, 7, 8, 9]
	
	>>> arr=[1,2,3,4,5,6,7,8,9,10]
	>>> arr[0:-1]
	[1, 2, 3, 4, 5, 6, 7, 8, 9]
	
	>>> arr=[1,2,3,4,5,6,7,8,9,10]  //0脚标不在逆序脚标之内
	>>> arr[-1:0]
	[]
	
	>>> arr=[1,2,3,4,5,6,7,8,9,10]
	>>>arr[::]
	[1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
	
	>>> arr=[1,2,3,4,5,6,7,8,9,10]
	>>> arr[5::2]
	[6, 8, 10]
	
	>>> arr=[1,2,3,4,5,6,7,8,9,10]
	>>> arr[::2]
	[1, 3, 5, 7, 9]


空序列
----------
	>>> no=[]
	>>> no
	[]

	>>> no[0]   //length为0  ，没有元素，不能访问
	Traceback (most recent call last):
	  File "<stdin>", line 1, in <module>
	IndexError: list index out of range

	用None表示空元素，有点像null


	>>> no=[None]
	>>> print no[0]
	None




检验一个元素是否在序列里，用in
-----------------

	>>> user=["haima","hailong","long"]
	>>> "haima" in user
	True

	>>> str="haima is excellent"
	>>> "haima" in str
	True



内建函数len ,max ,min
----------------------

	>>> no=[]
	>>> len(no)
	0

	>>> no=[] //空序列会报错
	>>> max(no)
	Traceback (most recent call last):
	  File "<stdin>", line 1, in <module>
	ValueError: max() arg is an empty sequence
	
	>>> none=[None,None]   //最大的是空
	>>> max(none)


	>>> arr=[1,2,3,4,5,6,7,8,9,10]
	>>> max(arr)
	10
	
	>>> arr=[1,2,3,4,5,6,7,8,9,10]
	>>> min(arr)
	1
	
	>>> str="haima is excellent"  
	>>> len(str)
	18
	
	>>> str="haima is excellent"    //这个不知道是怎么比较的
	>>> max(str)
	'x'



list ：python的苦力。遍历。
-------------------------

	>>> list("hello")
	['h', 'e', 'l', 'l', 'o']
	
	list使用与所有序列和字符串。


赋值：和java中的数组赋值相同
--------------------

	>>> none=[None,None]
	>>> none[1]=0
	>>> none
	[None, 0]

	>>> none[4]=0  //序列也会下标越界
	Traceback (most recent call last):
	  File "<stdin>", line 1, in <module>
	IndexError: list assignment index out of range


分片赋值
------------

	>>> arr=[2,4,844,"dd"] //
	>>> arr[2:]="dd"
	>>> arr
	[2, 4, 'd', 'd']
	
	>>> arr[2:]=["dd"]*5
	>>> arr
	[2, 4, 'dd', 'dd', 'dd', 'dd', 'dd']
	
	
	>>> arr[2:]=["dd"]
	>>> arr
	[2, 4, 'dd']


删除元素
-----------

	>>> arr=[2,4,844,"dd"]
	>>> del arr[len(arr)-1]
	>>> arr
	[2, 4, 844]




index()
--------------
获取元素的索引

	>>> arr=[1,2,3]
	>>> arr.index(2)
	1


count()
------------
检查某个元素出现次数

	>>> arr=[1,2,3,2]
	>>> arr.count(2)
	2

	>>> arr=[1,[1,2],3]
	>>> arr.count(1)
	1
	>>> arr.count([1,2])
	1


append()
---------------
追加元素

	>>> arr=[1,2,3,4]
	>>> arr.append(1)
	>>> arr
	[1, 2, 3, 4, 1]


	>>> arr=[1,2,3,4]
	>>> arr.append([1])
	>>> arr
	[1, 2, 3, 4, [1]]



extend() 
---------------
把另外一个序列扩展到原序列中
	
	>>> arr=[1,2,3,4]
	>>> arr.extend([6,7])
	>>> arr
	[1, 2, 3, 4, 6, 7]



insert(index,a) 
---------------
在某个位置插入某个元素

	>>> arr=[1,2,3,4]
	>>> arr.insert(4,"ddd")
	>>> arr
	[1, 2, 3, 4, 'ddd']



pop()  
----------
移除  仅有该方法才能既移除元素，又返回元素

	>>> arr=[1,2,3,4]
	>>> arr.pop(0)
	1
	>>> arr
	[2, 3, 4]
	
	>>> arr=[1,2,3,4]
	>>> arr.pop()
	4
	>>> arr
	[1, 2, 3]

remove()  
--------------
移除匹配到的第一个元素，没有返回值

	>>> arr=[1,2,3,4,1]
	>>> arr.remove(1)
	>>> arr
	[2, 3, 4, 1]


	>>> arr=[1,2,3,4,1]
	>>> arr.remove(5)
	Traceback (most recent call last):
	  File "<stdin>", line 1, in <module>
	ValueError: list.remove(x): x not in list

	remove()  vs del
	del arr[0]  //删除某个元素



reverse()
---------
反转数组

	>>> arr=[1,2,3,4,1]
	>>> arr.reverse()
	>>> arr
	[1, 4, 3, 2, 1]

sort()
------------
在原位置对列表排序

	>>> arr=[1,2,3,4,1]
	>>> arr.sort()
	>>> arr
	[1, 1, 2, 3, 4]



	>>> arr=[1,2,3,4,1]
	>>> arrycopy=arr[:];   //分片赋值是拷贝副本的
	>>> arrycopy.sort();
	>>> arrycopy
	[1, 1, 2, 3, 4]
	>>> arr
	[1, 2, 3, 4, 1]  //arr的顺序没变


	>>> arr=[1,2,3,4,1]
	>>> arrycopy=arr;    //直接赋值，赋值引用。
	>>> arrycopy.sort();
	>>> arrycopy
	[1, 1, 2, 3, 4]
	>>> arr
	[1, 1, 2, 3, 4]  //arr的顺序变了


内置函数sorted()
--------------------
排序，并返回序列

	>>> sorted(arr)
	[1, 1, 2, 3, 4]
	>>> arr
	[1, 2, 3, 4, 1]
	
	>>> sorted("dsdfsdf")
	['d', 'd', 'd', 'f', 'f', 's', 's']


元组
-----------
元组与列表一样，也是一种序序列。唯一的不同是元组不能修改。



创建元组
-------------
	>>> 1,2,3  //创建元组方式，直接逗号分隔
	(1, 2, 3)
	
	>>> 1,   //一个元素的时候
	(1,)
	
	>>> (1,2,3)  //创建元组方式二
	(1, 2, 3)
	
	
	>>> (1)  //错误，
	1


	>>> (1,)  //一个元素时
	(1,)


元组加法
------------------
同样支持 加法和乘法

	>>> (1, 2, 3)*2
	(1, 2, 3, 1, 2, 3)
	
	
	>>> (1, 2, 3)+(1,)
	(1, 2, 3, 1)

tuple()函数
--------------
tuple()函数，将一个序列转化为元组

	>>> tuple([2,1,2])
	(2, 1, 2)
	
	>>> tuple((1,2,3))
	(1, 2, 3)
	
	
	>>> x=1,2,3
	>>> x[0]
	1
	>>> x[1:2]   //元组的分片还是元组
	(2,)


