
title: Python开发准备
tags: 
	- python
	- IDE

categories:
	- python
-------------------

>pydev主页 http://www.pydev.org/download.html
python主页 https://www.python.org/
python使用 https://docs.python.org/3/using/index.html
python的linux安装 https://docs.python.org/3/using/unix.html#on-linux


python通常有两种版本2.x和3.x，这两种版本是并行更新的，是两个分支，并不是3.x比2.x版本高的概念。而目前3.x更强大一些，但2.x使用者要多一些，更多代码维护在2.x版本。以安装2.7.10为例。

#### 1.Linux
进入官网[https://www.python.org/](https://www.python.org/ "python官网" )，Downloads>2.7.10>Download

<img src="http://ou9e0q35h.bkt.clouddn.com/e8cdd823e3f15957e14b473860d6aab3.png" style="width:800px">


选择源码安装版(Source release)下载。下载完成之后上传服务器环境。当然也可以用Linux  wget下载该下载连接地址。


安装过程如下
```
$ tar -zxvf Python-2.7.10.tgz
$ mv Python-2.7.10 python
$ cd python/

$ ./configure --prefix=/usr/local/python27    #配置python安装路径
$ make                                                           
$ make install

$ cd /usr/local/python27
$ bin/python
Python 2.7.10 (default, Feb  5 2017, 18:51:22) 
[GCC 4.4.7 20120313 (Red Hat 4.4.7-4)] on linux2
Type "help", "copyright", "credits" or "license" for more information.
>>> exit();
```

如上面执行安装目录（/usr/local/python27）下的bin/python就能看到python命令行了。
你可以将 /usr/local/python27放到环境变量中，如下：

```
$ vim /etc/profile    #添加如下两行到/etc/profile中

export PYTHON_HOME=/usr/local/python27
export PATH=$PATH:$PYTHON_HOME/bin

$ . /etc/profile        #重新加载一下/etc/profile,使环境变量生效

```

此时执行python，发现python版本或许还是linux自带版本。由于环境变量造成的。打印一下环境变量，可重新配置一下环境变量，将python的环境变量提前

```
$ vim /etc/profile    #添加如下两行到/etc/profile中

export PYTHON_HOME=/usr/local/python27
export PATH=$PYTHON_HOME/bin:$PATH

$ . /etc/profile        #重新加载一下/etc/profile,使环境变量生效

# python
Python 2.7.10 (default, Feb  5 2017, 18:51:22) 
[GCC 4.4.7 20120313 (Red Hat 4.4.7-4)] on linux2
Type "help", "copyright", "credits" or "license" for more information.
>>> exit();
```

#### 2.windows
下载环节同1.Linux 中讲到的，只是选择版本为windows版本，注意选择合适自己系统的版本，我这里是64位


<img src="http://ou9e0q35h.bkt.clouddn.com/ede59037101deeb8159302411243cee5.png" style="width:800px">

下载完成后，傻瓜式安装，这里我就不再多做阐述。不过请记录一下安装路径，因为配置IDE环境会用到。
然后就能在windows菜单栏里找到python命令行了，可以做python练习。

<img src="http://ou9e0q35h.bkt.clouddn.com/23fe25f0ea783381412f8f03ea040c11.png" style="width:250px">
<img src="http://ou9e0q35h.bkt.clouddn.com/019bce00b755fc3cd2b214288c52ce70.png" style="width:800px">

 
 
#### 3.pydev插件
开发Python的IDE特别多，更多参考[连接文章](http://www.oschina.net/news/57468/best-python-ide-for-developers)，因为本人最早是Java猿，所以目前还是选择Eclipse的插件pydev。具体很多说明见官网[http://www.pydev.org/download.html](http://www.pydev.org/download.html  "pydev官网" )，为了安装方便，这里直接用在线安装链接http://www.pydev.org/updates。

安装好后，重启eclipse后，能在preferences中找到PyDev，则说明插件安装成功。然后设置PyDev
<img src="http://ou9e0q35h.bkt.clouddn.com/c6afe762a33b9d0f3bd7559ef6c9e6d6.png" style="width:800px">



设置完成之后如下图所示。
<img src="http://ou9e0q35h.bkt.clouddn.com/92350f66dda446d2b737c5550f9dc89d.png" style="width:800px">



新建Python项目及python脚本。先新建PyDev Project。
<img src="http://ou9e0q35h.bkt.clouddn.com/959dc0aa6f0d53a3d268c141e20d6dac.png" style="width:800px">

然后新建PyDev package。
<img src="http://ou9e0q35h.bkt.clouddn.com/68a8ceb70bd6e11956b684dd25ea7b96.png" style="width:800px">


 
然后新建python脚本，在包下面右键，new->file 创建一个文件，文件后缀名为.py。写一段简单代码，再右键Python Run运行。
<img src="http://ou9e0q35h.bkt.clouddn.com/2eb2b97514fc302a619fe3ab25b7cedc.png" style="width:800px">

 

运行结果如下
<img src="http://ou9e0q35h.bkt.clouddn.com/71233ea6cb45aa515f21fd5bb834a12b.png" style="width:800px">







