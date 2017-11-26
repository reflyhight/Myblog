
title: python 字典dictionary
tags: 
	- python
	- json

categories:
	- python
-------------------



简介
-------------------
字典有点像其他语言中的json。以大括号作为起始标记，以键值对的数据结构存在。

	>>> worker={"name":"haima","age":25}
	>>> worker["name"]
	'haima'


基本操作
-----------------------------------------
字典长度，取值，重新赋值，删除键值对，权限检查（检查键是否存在）

	>>> worker={"name":"haima","age":25}
	>>> len(worker)     //长度
	2
	>>> worker["name"]     //取值
	'haima'
	>>> worker["name"]="vicky"     //重新赋值
	>>> worker
	{'age': 25, 'name': 'vicky'}
	>>> del worker["name"]     //删除键值对
	>>> worker
	{'age': 25}
	>>> "age" in worker     //检查键是否存在
	True

键类型
-----------------------------
键的类型可以是任意的不可变类型，比如浮点型，字符串或者元组

	>>> x={1:3}
	>>> x
	{1: 3}
	>>> x[1]
	3

<!-- more -->

格式化替换
--------------------------

	>>> temp="wo shi %(name)s"
	>>> stu={"name":"haima"}
	>>> temp % stu
	'wo shi haima'



clear()
---------------------------
clear方法清楚字典中所有项。没有返回值

	>>> stu={"name":"haima"}
	>>> out=stu.clear()
	>>> print out
	None

	>>> x={"name":"haima"}
	>>> y=x
	>>> y.clear()     //清空，是清空指向对象中的内容
	>>> x
	{}


	>>> x={"name":"haima"}
	>>> y=x
	>>> y={}     //赋值，直接改变了y的引用
	>>> x
	{'name': 'haima'}


copy()
-----------------------------
拷贝字典，实现的是浅拷贝，拷贝后，两者同键的值相同。

	>>> x={"name":"haima"}     //拷贝后"haima"和"haima"相同
	>>> y=x.copy();
	>>> x["name"]="vicky"     //重新赋值，值发生改变
	>>> y
	{'name': 'haima'}
	
	
	>>> x={"scores":[1,3,4]}
	>>> y=x.copy()
	>>> y["scores"][0]=5;     //改变值得内容
	>>> x
	{'scores': [5, 3, 4]}     //因为x中的值和y中相同，所以x中也改变了
	
	

get()
--------------
通过键取值，取不到值时，会返回None。而普通取值，会报错。

	>>> x={"name":"haima"}   
	>>> print x.get("age")
	None
	
	>>> x["age"]
	Traceback (most recent call last):
	  File "<stdin>", line 1, in <module>
	KeyError: 'age'
	
	

has_key()
-------------
相当与基本操作in，判断成员资格

	>>> x={"name":"haima"}   
	>>> x.has_key("age")
	False

items()
-----------------
返回所有键值对，的序列

	>>> x={"name":"haima","age":25}   
	>>> x.items();
	[('age', 25), ('name', 'haima')]
	

keys()
----------------
返回所有键的序列

	>>> x={"name":"haima","age":25}   
	>>> x.keys();
	['age', 'name']
	

pop()
--------------
移除键值对，并且返回，该键值对中的值

	>>> x={"name":"haima","age":25}   
	>>> x.pop("name")
	'haima'
	>>> x
	{'age': 25}

popitem()
-------------------------
顺序弹出字典中的一个键值对

	>>> x={"name":"haima","age":25}   
	>>> x.popitem();
	('age', 25)
	>>> x
	{'name': 'haima'}


setdefault()
--------------------
设置某个键的默认值

	>>> x={}
	>>> x.setdefault("name","haima")
	'haima'
	>>> x.get("name")
	'haima'


update()
------------------
update方法可以利用一个字典项更新另外一个字典
更新时，目标字典中多出来的键值对，直接加入到原字典中

	>>> x={"name":"haima","age":25}  
	>>> x.update(y);
	>>> x
	{'job': 'teacher', 'age': 25, 'name': 'vicky'} //可以看出，更新时候，"job"项直接被加入
	到原字典中了

values()
-----------------
获取所有的值，返回一个序列

	>>> x={"name":"haima","age":25}   
	>>> x.values();
	[25, 'haima']

