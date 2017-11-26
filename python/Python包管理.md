
title: Python包管理 
tags: 
	- python
	- package
	- module

categories:
	- python
-------------------


>python官网 https://www.python.org/
pypi主页 https://pypi.python.org/pypi
pypi教程页面 https://wiki.python.org/moin/CheeseShopTutorial
pip主页 https://pip.pypa.io/en/stable/   https://pip.pypa.io/en/latest/
解决Mysql-python安装失败帖子 http://www.cnblogs.com/fnng/p/4115607.html#3312372
windows下python包非官方下载地址 http://www.lfd.uci.edu/~gohlke/pythonlibs/#mysql-python
发布模块 https://docs.python.org/3/distutils/index.html

###    包管理简介

python中包的概念，具体到python代码中便是具体的模块，或者模块集，如Mysql所需要的包。但又类似于Java中的maven依赖。在官网中的菜单pypi中能找到相关说明（https://pypi.python.org/pypi）。

<img src="http://ou9e0q35h.bkt.clouddn.com/a704dd11a4ca675b4e15950822fbb72f.png" style="width:895px;border:2px solid #C6C6C6;" >
 
进入PyPI主页的介绍。PyPI是python包索引的简写(PyPI - the Python Package Index)，这个在PyPI右边菜单->PyPI Tutorial有很具体的说明如下图。你可以认为，PyPI类似于java中的maven包管理，但这些包需要通过install才能在Python中使用。

<img src="http://ou9e0q35h.bkt.clouddn.com/7ae68b48f33f2822dbf00063de3606ae.png"  style="width:895px;border:2px solid #C6C6C6;" >

在[https://pypi.python.org/pypi](https://pypi.python.org/pypi)中可以看到  “Get Packages” 的说明，说了两种方式安装python包，一是通过pip安装，二是通过下载，解压后使用“python setup.py install” 来安装。

<img src="http://ou9e0q35h.bkt.clouddn.com/7fb53ce803026eaa6c5366b827b184f9.png"  style="width:895px;border:2px solid #C6C6C6;" >


###    PIP简介
来到pip主页，可以看到对pip的介绍。pip也是python的一个包，只不过这个包比较特殊，它是一个工具，并且可以用于安装其他的包。“The PyPA recommended tool for installing Python packages”。PyPA又是什么呢。是维护相关python包的权威机构。PyPA(Python Packaging Authority) The Python Packaging Authority (PyPA) is a working group that maintains many of the relevant projects in Python packaging.更多见[PyPA相关](https://www.pypa.io/en/latest/) 

<img src="http://ou9e0q35h.bkt.clouddn.com/9c8a415d02f80abe96ae22ea1986f40f.png"  style="width:895px;border:2px solid #C6C6C6;" >
 
 
在pip的[https://pip.pypa.io/en/stable/installing/#do-i-need-to-install-pip](https://pip.pypa.io/en/stable/installing/#do-i-need-to-install-pip)页面中，我们可以看到相关安装pip的说明，两种版本的Python的高版本已经自带了pip，分别为 Python 2 >=2.7.9 or Python 3 >=3.4，使用前经过简单的升级就可以用了。

<img src="http://ou9e0q35h.bkt.clouddn.com/4d55a5f3bfce866f53a67a0ee168f4a3.png"  style="width:895px;border:2px solid #C6C6C6;" >
 
 
在installing安装章节,介绍了两种安装pip的方法，通过get-pip.py或者Using Linux Package Managers。这两种方式在后文[Linux安装PIP](/2017/08/09/python/Python包管理/#Linux安装PIP)中再说明。

<img src="http://ou9e0q35h.bkt.clouddn.com/692badaeedb33e62a066b6e6fa4bffc9.png"  style="width:895px;border:2px solid #C6C6C6;" >
 

###    windows安装PIP
上面已经说过了，高版本python自带pip。低版本python需要自行安装。由于实际上pip也是python的包，所以它也能用“python setup.py install”的方式安装。那就通过  下载，解压，python setup.py install 的方式安装。通过以下[连接](https://pypi.python.org/pypi/pip/)找到pip包主页下载，同样你也可以从pypi主页搜索pip包。进入到pip包的主页后，下载后缀名为tar.gz 的压缩包。

<img src="http://ou9e0q35h.bkt.clouddn.com/cad1db3b20fd315bd67434cb014b952b.png"  style="width:895px;border:2px solid #C6C6C6;" >

<img src="http://ou9e0q35h.bkt.clouddn.com/12149c82d1276194fe94d5712826a068.png"  style="width:895px;border:2px solid #C6C6C6;" >
 
 
下载完成之后，解压到文件夹，在cmd命令行进入到文件夹，然后执行 “python setup.py install”。此时如果windows环境变量没有配置python安装路径，会报“python 不是内部或外部命令”。安装好Python的前提条件下，如下设置python的环境变量。安装Python见博客<a href="/2017/08/07/python/Python开发准备/" target="_blank">【Python开发准备】</a>  
 
1.新建PYTHON_HOME

<img src="http://ou9e0q35h.bkt.clouddn.com/1fb75f430adf7d1e807e121a14668b48.png"  style="width:895px;border:2px solid #C6C6C6;" >
 
 
2.将PYTHON_HOME加入到path中

<img src="http://ou9e0q35h.bkt.clouddn.com/ee1dd19dfef0db1748033310607c8857.png"  style="width:895px;border:2px solid #C6C6C6;" >
 
 
确定保存之后打开cmd命令行，输入python测试一下，如果成功，会有进入python命令行。

<img src="http://ou9e0q35h.bkt.clouddn.com/6ae417196a0ffa00b646f33b1bc60116.png"  style="width:895px;border:2px solid #C6C6C6;" >
 
 
接着回到安装pip。在cmd命令行进入到pip解压后的文件夹中，然后执行 “python setup.py install”

<img src="http://ou9e0q35h.bkt.clouddn.com/5109f5451c902a5581ee4af64d2a17b2.png"  style="width:895px;border:2px solid #C6C6C6;" >

<img src="http://ou9e0q35h.bkt.clouddn.com/023a8cd6097a445488957ba0fad5c7d6.png"  style="width:895px;border:2px solid #C6C6C6;" >
 
 
###    pip的使用
根据官网的说明https://pip.pypa.io/en/stable/user_guide/#installing-packages  安装python包可以采用以下命令就能轻松安装了。
```
$ pip install SomePackage            # latest version
$ pip install SomePackage==1.0.4     # specific version
$ pip install 'SomePackage>=1.0.4'     # minimum version
```
 
###    windows安装mysql-python模块
 
 
使用python连接mysql时，需要用到mysql-python包，在cmd命令行适用命令 “pip install mysql-python” 安装。此时cmd命令行也会提示 "pip 不是内部或外部命令"。此时也需要将pip配置到windows的环境变量中。由于pip被默认安装到python安装目录下的script文件夹下面，所以将该路径配置到windows环境变量中。具体参考将python命令加入环境变量。

<img src="http://ou9e0q35h.bkt.clouddn.com/e31e56b9960e709334ace3d205123790.png" style="width:895px;border:2px solid #C6C6C6;" >
 
打开CMD命令行安装mysql-python。pip install mysql-python

<img src="http://ou9e0q35h.bkt.clouddn.com/d15f7aa6eed18c1a0d4f3299a270138e.png"  style="width:895px;border:2px solid #C6C6C6;" >

<img src="http://ou9e0q35h.bkt.clouddn.com/bec954d3d9feeaa1bae01e1f1f46065c.png"  style="width:895px;border:2px solid #C6C6C6;" >
 
 
 
结果安装失败了。这个问题，我也在网上搜了许多资料。有说直接下载 MySQL-python-1.x.xwin-amd64-py2.7.exe。https://sourceforge.net/projects/mysql-python/?source=typ_redirect。
 
 
目前作为一个初学者也不用纠结这个问题了。因为安装python包可以用“python setup.py install” 来安装。按照安装pip的方法，去pypi官网搜索mysql-python，然后下载zip包，然后解压安装。

<img src="http://ou9e0q35h.bkt.clouddn.com/7b1d94a867e9f0260e031e3abe2e15a6.png"  style="width:895px;border:2px solid #C6C6C6;" >

<img src="http://ou9e0q35h.bkt.clouddn.com/16f28cf871eb3e322f5f88a185686bd0.png"  style="width:895px;border:2px solid #C6C6C6;" >
 
 
 
 
 
结果一看，又安装失败了。（当场打脸）
我又去网上搜索了一圈，找到如下方法。到 www.lfd.uci.edu/~gohlke/pythonlibs/#mysql-python 下载MySQL_python‑1.2.5‑cp27‑none‑win_amd64.whl，（这里应该是下载相应的版本）然后用 pip 进行安装。
 
网络回答：

<img src="http://ou9e0q35h.bkt.clouddn.com/2b236cf79334a0fb8d5b2b3075483bf1.png"  style="width:895px;border:2px solid #C6C6C6;" >
 
 
亲手验证：

<img src="http://ou9e0q35h.bkt.clouddn.com/38f9b08f9577ff84d55ee1381827da7a.png"  style="width:895px;border:2px solid #C6C6C6;" >
 
 
重启eclipse后，引用mysql模块不会出现编译错误，且能正常执行。

<img src="http://ou9e0q35h.bkt.clouddn.com/74bbb2f02b7035df2a470d3090c61814.png"  style="width:895px;border:2px solid #C6C6C6;" >
 
 
 
 

 

###    Linux安装PIP
在安装好Python后，再安装PIP，可以通过像Windows的方式。使用python setup.py install 的方式安装。
```
] tar -zxvf pip-9.0.1.tar.gz
] cd pip-9.0.1
] python setup.py install
##安装日志省略
Installed /usr/lib/python2.6/site-packages/pip-9.0.1-py2.6.egg
Processing dependencies for pip==9.0.1
Finished processing dependencies for pip==9.0.1
```

如[https://pip.pypa.io/en/latest/installing/](https://pip.pypa.io/en/latest/installing/)所说。Linux系统还可以使用**get-pip.py**和**Linux Package Managers**两种方式来安装。而**Linux Package Managers**的方式也就是linux包管理方式，如Centos系统的yum安装
```
]curl https://bootstrap.pypa.io/get-pip.py
]python get-pip.py
#按官方文档操作，并没成功😪
```

```
sudo yum install python-pip
```



###    Linux使用PIP
使用方面就和windows一样了，这个并没有区别。当然还可以使用python -m pip install SomePackage,这个在[https://docs.python.org/3/installing/index.html](https://docs.python.org/3/installing/index.html)有详细介绍。





###    卸载PIP
通过PIP可以直接卸载已经安装的包，使用**pip uninstall SomePackage**，但如何卸载PIP呢。发现**pip uninstall  pip**也能做到，厉害了，自己卸载自己。



###    关键备忘
 
```
python setup.py install   #包的通用安装方法
pip install SomePackage    #pip方式安装包
pip install SomePackage==1.0    #pip方式安装特别版本的包
pip install SomePackage>=1.0
python -m pip install -U pip   #升级pip
pip uninstall SomePackage  #pip卸载包
pip list    #查询已经安装的包
```

###    总结
PIP可以方便的管理包，这种安装方式很方便，因为你不用关心安装包的依赖，PIP有自动安装依赖的功能。而 python setup.py install 方式是不会自动安装依赖的。这个见[https://wiki.python.org/moin/CheeseShopTutorial](https://wiki.python.org/moin/CheeseShopTutorial)说明


