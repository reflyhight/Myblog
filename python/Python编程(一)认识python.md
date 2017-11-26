
title: Python编程(一)认识python
tags: 
	- 简介
	- 注释

categories:
	- python
-------------------
>我本想花一两周快速把python编程这篇博客写完，但后来发现有点困难，python虽然说是一门简明优雅的语言，但要真正掌握它并不容易，如scala一样，它也有灵活的特性。希望这个学习笔记能写得更系统，有一定深度一些。学习过程借鉴很多博客，网络资源，还有书籍。
 
----------------------------------------------------------------------------------
特别鸣谢：
- 实验楼[https://www.shiyanlou.com/](https://www.shiyanlou.com/)
- 廖雪峰的官网[https://www.liaoxuefeng.com/](https://www.liaoxuefeng.com/)
 
----------------------------------------------------------------------------------
 
 
###    简介
python是一门解释执行的，面向对象的，动态语言。解释执行是指python不需要像java那样编译，而是通过解释器直接执行。python的特点是“优雅”、“明确”、“简单”。同时python也是强大的，它在算法领域和爬虫领域有着很明显的优势，它在web领域也相当流行。同时它也是完全开源的，有很庞大的开源爱好者为它的发展贡献。基于这些优点，python也成为了很流行的语言。
 
###    Guido van Rossum
python是由荷兰人`Guido van Rossum`（龟叔）发明，第一个公开发行版本1991年。为了打发圣诞节的无聊发明的（大牛都很任性，一不开心就发名一种语言），之所以选中Python（大蟒蛇的意思）作为该编程语言的名字，是因为他是一个叫Monty Python的喜剧团体的爱好者。
 
 
 
###    缩进
python是靠缩进来控制程序结构的，所以并没有像java中的大括号。缩进可以用tab制表符，或者空格，但两者不能混用。如下面代码中第2行和第4行代行首用tab制表符缩进，表示是一个代码块
```
>>> if(True):
...     print "true"
... else:
...     print "false"
...
true
 
```
 
###    标识符
所谓标识符，可以用来命名的符号，包括变量名，函数名，类名，常量名等等。
1.标识符可以由英文字符，数字，和下划线组成，且数字不能作为标识符的开头。
2.python语言中的关键字不能作为用户自定义的标识符使用。关键字在python的程序控制中有特殊意义
python中的关键字有如下：
```
'False', 'None', 'True', 'and', 'as', 'assert', 'break'
'class', 'continue', 'def', 'del', 'elif', 'else', 'except', 'finally'
'for', 'from', 'global', 'if', 'import', 'in', 'is', 'lambda'
'nonlocal', 'not', 'or', 'pass', 'raise'
'return', 'try', 'while', 'with', 'yield'
```
 
 
###    大小写
python语言对大小写敏感（好像除了sql类，语言基本都敏感），如布尔型True如果写成true就不合法，如变量名name和变量名Name是不同的两个变量
 
###    编写脚本
你可以使用命令行的方式学习python,但那并不是python编程的日常。将python程序写到纯文本文件中，然后授权，即完成了python脚本。第一个python脚本
在linux环境下vim一个文本文件 helloworld.py，内容如下
```
#!/usr/bin/python 
print("hello world")
```
授权执行
```
# chmod +x first.py
# ./helloworld.py
hello world
```
 
如果代码中没有`#!/usr/bin/python`这行，则可以使用python helloworld.py （同python2 helloworld.py）或者 python3 helloworld.py方式执行，python3 helloworld.py使用的版本为python3.x，而python2 helloworld.py使用的版本是python2.x。
 
 
###    注释
用`#`做单行注释，在前面我们讲过，三引号可以作为多行字符串的创建方法。因为单独的字符串在python脚本中不会执行，那么可以用三引号作为python的多行注释。
####    单行注释
使用vi编写如下文件，文件名为helloworld.py
```
#第一个python脚本
print("hello world")
```
执行效果 ，使用python3  helloworld.py 执行
```
hello world
```
####    多行注释
```
"""
这是多行注释
这是多行注释
"""
#第一个python脚本
print("hello world")
```
执行效果和单行注释相同
 
###    脚本编码
如上代码，如果使用`python helloworld.py` ，默认环境会使用python2.x执行，此时相同的代码会报错
```
# python helloworld.py
  File "helloworld.py", line 3
SyntaxError: Non-ASCII character '\xe5' in file helloworld.py on line 4, but no encoding declared; see http://www.python.org/peps/pep-0263.html for details
```
在脚本顶部使用`#coding=utf-8`或者 `# -*- coding: utf-8 -*-` 告诉解释器，使用utf-8读取脚本执行
```
#coding=utf-8
"""
多行注释
"""
#第一个python脚本
print("hello world")
```
在次使用`python helloworld.py`  时，不再报错
 
###    输入
vi编辑脚本inoutput.py ，内容如下
```
words = input("请输入你要说的话:")
print(words )
```
执行效果
```
# python3 inoutput.py
请输入你的身高:自定义输入，我是haima
自定义输入，我是haima
```
其中第2行冒号后面内容为键盘输入内容，接着打印出了刚刚输出的内容
**版本差异**，python2.x和python3.x的区别
使用python3的 input，输入任何字符串，将被当作字符串，而 python2中的input，会将输入的内容当作代码执行
分别使用python3和python2的命令行测试如下
 
python3
```
# python3
Python 3.6.0 (default, Aug 20 2017, 20:13:55)
[GCC 4.4.7 20120313 (Red Hat 4.4.7-4)] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> input("please input:")
please input:1+1
'1+1'
```
 
python2
```
python2
Python 2.6.6 (r266:84292, Nov 22 2013, 12:16:22)
[GCC 4.4.7 20120313 (Red Hat 4.4.7-4)] on linux2
Type "help", "copyright", "credits" or "license" for more information.
>>> input("please input:")
please input:1+1
2
>>> input("please input:")
please input:a+b
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "<string>", line 1, in <module>
NameError: name 'a' is not defined
>>> input("please input:")
please input:"a+b"
'a+b'
```
 
以上很明显看出python3中无论你输入什么，直接当作字符串处理了，而python2当代码处理
python2中使用raw_input()方法和python3中的input()方法效果相同，而python3如果想使得输入值当作代码执行，可以用eval函数转换
```
eval("1+1")
2
```
###    输出
输出就是刚刚的print()函数
```
print("www")
www
print("www",1)
www 1
```
print函数是[内建函数（Built-in Functions）](https://docs.python.org/3/library/functions.html)，print()函数 完整格式print(*objects, sep=’ ‘, end=’\n’, file=sys.stdout, flush=False)，print默认打印完成之后会换行，也就是带名参数end='\n'，如果要使得打印后不换行可使用如下语句
```
# print("hello",end='');print("hello2",end='');
hellohello2
# print("hello");print("hello2");
hello
hello2
```


