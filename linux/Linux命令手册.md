

title: Linux命令手册
tags: 
	- Linux
	- 基础

categories:
	- Linux
-------------------


 
>本手册是对网站[http://man.linuxde.net/](http://man.linuxde.net/)中常见命令的摘要，也参考一些其他博客和Linux中man文档，linux中[man文档在线版](http://man.he.net/)。本文将持续更新。
 
##    cd
**作用**：用来切换工作目录
 
**格式**：cd [-L | -P] [directory]    `directory要切换到的目标目录`
 
**常见选项**：
- -p 如果要切换到的目标目录是一个符号连接，直接切换到符号连接指向的目标目录 ；
- -L 如果要切换的目标目录是一个符号的连接，直接切换到字符连接名代表的目录，而非符号连接所指向的目标目录；
-  \- 当仅实用"-"一个选项时，当前工作目录将被切换到环境变量"OLDPWD"所表示的目录；
 
**案例**：
```
cd 进入用户主目录；
cd ~ 进入用户主目录；
cd - 返回进入此目录之前所在的目录；
cd .. 返回上级目录（若当前目录为“/“，则执行完后还在“/"；".."为上级目录的意思）；
cd ../.. 返回上两级目录； cd !$ 把上个命令的参数作为cd参数使用。
 
//-P和-L的比较
$ cd -P /opt/cloudera/parcels/CDH
$ pwd
/opt/cloudera/parcels/CDH-5.11.0-1.cdh5.11.0.p0.34
$ cd -L /opt/cloudera/parcels/CDH
$ pwd
/opt/cloudera/parcels/CDH
```
 
**来源**：[http://man.linuxde.net/cd](http://man.linuxde.net/cd)
 
-------------------
 
##    ls
 
**作用**：用来显示目标列表，在Linux中是使用率较高的命令。ls命令的输出信息可以进行彩色加亮显示，以分区不同类型的文件
 
**格式**：ls [OPTION]... [FILE]   `FILE为要查看的目录或者文件`
 
**常见选项**：
- -l：以长格式显示目录下的内容列表。输出的信息从左到右依次包括文件名，文件类型、权限模式、硬连接数、所有者、组、文件大小和文件的最后修改时间等；
- --color[=WHEN]：使用不同的颜色高亮显示不同类型的;其中color的值[WHEN]是这样的 WHEN defaults to ‘always’ or can be ‘never’ or ‘auto’. ；
- -h：显示大小的时候，以易读形式显示，需要和-l配合使用才能生效；
- -a：显示全部文件，包括隐藏文件；
- -m：用“,”号区隔每个文件和目录的名称，-l和-m一起使用时，-l失效；
- -t：用文件和目录的更改时间排序；
- -r：以本来展现的顺序，反序展示；
 
**案例**：
```
$ ls -lh
total 293M
-rw-------. 1 root root 2.2K May  9 07:53 anaconda-ks.cfg
-rw-r--r--  1 root root  21K Jul 14 06:17 derby.log
--省略部分
 
```
 
**来源**：[http://man.linuxde.net/ls](http://man.linuxde.net/ls)
 
-------------------
 
##    echo
**作用**：用于在shell中打印shell变量的值，或者直接输出指定的字符串。
 
**格式**：echo [SHORT-OPTION]... [STRING]   `STRING为变量或者字符串`
 
**常见选项**：
- -e：激活转义字符；
 
**案例**：
```
$ echo "haima\thailong"
haima\thailong
 
$ echo -e  "haima\thailong"
haima   hailong
 
$ echo $HOSTNAME
cdha    //打印出了当前主机名
```
 
**来源**：[http://man.linuxde.net/echo](http://man.linuxde.net/echo)
 
-------------------
 
##    date
**作用**：用于显示或设置系统时间与日期。
 
**格式**：
- date [OPTION]... [+FORMAT]
- date [-u|--utc|--universal] [MMDDhhmm[[CC]YY][.ss]]
 
**常见选项**：
- -d<字符串>：显示字符串所指的日期与时间。字符串前后必须加上双引号；
- -s<字符串>：根据字符串来设置日期与时间。字符串前后必须加上双引号；
- -u：显示GMT；
- --help：在线帮助；
- --version：显示版本信息；
 
**FORMATE**：
```
%H 小时，24小时制（00~23）
%I 小时，12小时制（01~12）
%k 小时，24小时制（0~23）
%l 小时，12小时制（1~12）
%M 分钟（00~59）
%p 显示出AM或PM %r 显示时间，12小时制（hh:mm:ss %p）
%s 从1970年1月1日00:00:00到目前经历的秒数
%S 显示秒（00~59） %T 显示时间，24小时制（hh:mm:ss）
%X 显示时间的格式（%H:%M:%S）
%Z 显示时区，日期域（CST）
%a 星期的简称（Sun~Sat）
%A 星期的全称（Sunday~Saturday）
%h,%b 月的简称（Jan~Dec）
%B 月的全称（January~December）
%c 日期和时间（Tue Nov 20 14:12:58 2012）
%d 一个月的第几天（01~31）
%x,%D 日期（mm/dd/yy）
%j 一年的第几天（001~366）
%m 月份（01~12）
%w 一个星期的第几天（0代表星期天）
%W 一年的第几个星期（00~53，星期一为第一天）
%y 年的最后两个数字（1999则是99）
```
**案例**：
显示时间
```
date +%Y%m%d //显示前天年月日
date -d "+1 day" +%Y%m%d //显示前一天的日期
date -d "-1 month" +%Y%m%d //显示上一月的日期
date -d "+1 year" +%Y%m%d //显示下一年的日期
date -d "2 second" +"%Y-%m-%d %H:%M.%S"//显示两秒后
```
设定时间
```
date -s 20120523 //设置成20120523，这样会把具体时间设置成空00:00:00
date -s 01:01:01 //设置具体时间，不会对日期做更改
date -s "01:01:01 2012-05-23" //这样可以设置全部时间
date -s "01:01:01 20120523" //这样可以设置全部时间
date -s "2012-05-23 01:01:01" //这样可以设置全部时间
date -s "20120523 01:01:01" //这样可以设置全部时间
```
**来源**：[http://man.linuxde.net/date](http://man.linuxde.net/date)
 
-------------------
 
 

 
##   pwd
**作用**：查看当前工作目录的路径
 
**格式**：pwd [-LP]
 
**常见选项**：
- -L：L是logical的意思，表示逻辑路径。选项--logical已失效。打印当前工作目录，从$PWD中读取。不带任何选项的pwd和-L选项同意
- -P：P是physical，表示物理路径。选项--physical已经失效。如果$PWD中含有符号连接，将会打印连接指向的真实路径；
 
**命令行获取环境变量$PWD**
```
$ echo $PWD
```
 
**案例**：
```
$ pwd
/opt/cloudera/parcels/CDH
 
$ pwd -L
/opt/cloudera/parcels/CDH
 
$ pwd -P
/opt/cloudera/parcels/CDH-5.11.0-1.cdh5.11.0.p0.34
```
 
**来源**：
- [http://blog.csdn.net/yuyuntan/article/details/52577431](http://blog.csdn.net/yuyuntan/article/details/52577431)
- [http://www.cnblogs.com/kerrycode/p/3425309.html](http://www.cnblogs.com/kerrycode/p/3425309.html)
 
-------------------
 
##  cat
**作用**：如果有多个文本文件，将文本文件连接打印到标准输出
 
**格式**：cat [OPTION]... [FILE]...
 
**常见选项**：
- -n或-number：有1开始对所有输出的行数编号；
- -b或--number-nonblank：和-n相似，只不过对于空白行不编号；
- -s或--squeeze-blank：当遇到有连续两行以上的空白行，就代换为一行的空白行；
- -A：显示不可打印字符，行尾显示“$”。    equivalent to -vET；
- -e：等价于"-vE"选项；
- -t：等价于"-vT"选项；
- -v, --show-nonprinting：use ^ and M- notation, except for LFD and TAB；LFD应该代表的是行分隔符。（这个确实是博主根据上下文猜测，查了半天查到个long fucking day😆）
- -T, --show-tabs： display TAB characters as ^I；
- -E, --show-ends：display $ at end of each line；
 
**案例**：
```
cat m1 （在屏幕上显示文件ml的内容）
cat m1 m2 （同时显示文件ml和m2的内容）
cat m1 m2 > file （将文件ml和m2合并后放入文件file中）
```
 
**来源**：[http://man.linuxde.net/cat](http://man.linuxde.net/cat)
 
-------------------
 
##  more
**作用**：支持向下翻页的文本查看命令
 
**格式**：
- more [-dlfpcsu] [-num] [+/pattern] [+linenum] [file ...]
 
**更多使用说明**
```
按Space键：显示文本的下一屏内容；
按Enier键：只显示文本的下一行内容；
按斜线符/：接着输入一个模式，可以在文本中寻找下一个相匹配的模式；
按h键：显示帮助屏，该屏上有相关的帮助信息；
按b键,Ctrl+B ：显示上一屏内容。
按q键：退出more命令；
```
 
**常见选项**：
 
- -<数字>：指定每屏显示的行数；
- -d：显示“[press space to continue,'q' to quit.]”和“[Press 'h' for instructions]”；
- -c：不进行滚屏操作。每次刷新这个屏幕；
- -s：将多个空行压缩成一行显示；
- -u：禁止下划线；
- +<数字>：从指定数字的行开始显示；
 
**案例**：
```
more -c -10 file  //显示文件file的内容，每10行显示一次，而且在显示之前先清屏
```
 
**来源**：
- [http://man.linuxde.net/more](http://man.linuxde.net/more)
- [http://www.cnblogs.com/aijianshi/p/5750911.html](http://www.cnblogs.com/aijianshi/p/5750911.html)
 
-------------------


##   grep
 
**作用**：是一种强大的文本搜索工具，它能使用正则表达式搜索文本，并把匹配的行打印出来
 
**格式**：
- grep [OPTIONS] PATTERN [FILE...]
- grep [OPTIONS] [-e PATTERN | -f FILE] [FILE...]
 
**常见选项**：
- -c, --count：只显示匹配到的个数，个数不是行数，因为一行中可能匹配到多个
- -e：使用正则表达式
- -E --extended-regexp：使用扩展正则表达式
- -F：不使用正则表达式匹配，将要查找的串当作普通字符串
- -i：忽略大小写
- --color[=WHEN]：when可以取never,always,auto，显示是是否高亮
- -v  --invert-match：匹配不满足匹配条件（取反）
- -f<file>：将要多个要匹配的字符串写到文件中，每行一个，逐行匹配

```
关于-e,-E,-F选项，分布代表的三种模式分别对应egrep，Egrep，fgrep，正则，扩展正则，禁用正则。默认使用的模式是egrep。
这个在Linux的man文档的DESCRIPTION中就有直接提到
```
**案例**：
```
#先编辑test文本，其中第一行为空，第二行为数字
$ cat test1

00003234
$ grep -E '*' test1

00003234
$ grep -e '.' test1  
00003234
$ grep  '.' test1   
00003234

#先编辑一个文件test2,再编辑一个文件grepfile ,grepfile 中含有几个要匹配的字符串
$ cat test2
aaba
bbab
cacbc
$ cat grepfile 
a
ab
c
$ grep -f grepfile --color='auto' test2   
**aaba**
bb`ab`
`cac`b`c`
#特别注意，输出结果并没有`号，为了呈现'--color'的才彩色效果。手动加入，如果你再linux中测试，会看到正确的效果
```
 
**来源**：
- [http://man.linuxde.net/grep](http://man.linuxde.net/grep)
- [http://www.cnblogs.com/ggjucheng/archive/2013/01/13/2856896.html](http://www.cnblogs.com/ggjucheng/archive/2013/01/13/2856896.html)
 
-------------------
 
## less
**作用**：less 与 more 类似，但使用 less 可以随意浏览文件，而 more 仅能向前移动，却不能向后移动，而且 less 在查看之前不会加载整个文件。
 
**格式**：
```
less -?
       less --help
       less -V
       less --version
       less [-[+]aBcCdeEfFgGiIJKLmMnNqQrRsSuUVwWX~]
            [-b space] [-h lines] [-j line] [-k keyfile]
            [-{oO} logfile] [-p pattern] [-P prompt] [-t tag]
            [-T tagsfile] [-x tab,...] [-y lines] [-[z] lines]
            [-# shift] [+[+]cmd] [--] [filename]...
```
 
**更多使用说明**
```
[pageup]，b： 向上翻一页
[pagedown]，空格：向下翻一夜
d：向下翻半页
u：向上翻半页
/字符串：向下搜索“字符串”的功能
?字符串：向上搜索“字符串”的功能
n：重复前一个搜索（与 / 或 ? 有关）
N：反向重复前一个搜索（与 / 或 ? 有关）
q  退出less 命令
```
 
**常见选项**：
 
- -e  当文件显示结束后，自动离开
- -i  忽略搜索时的大小写
- -m  显示类似more命令的百分比
- -N  显示每行的行号
 
**案例**：
```
less -N  file 
```
 
**来源**：[http://www.cnblogs.com/aijianshi/p/5750911.html](http://www.cnblogs.com/aijianshi/p/5750911.htmll)
 
-------------------

##    cp
 
**作用**：
 
**格式**：
- cp [OPTION]... [-T] SOURCE DEST
- cp [OPTION]... SOURCE... DIRECTORY
- cp [OPTION]... -t DIRECTORY SOURCE...
 
**常见选项**：
- -T,--no-target-directory：将DEST当作普通文件处理,不会当作目录
- -R/r：递归拷贝
- -s：对源文件建立符号连接，而非复制文件；
- -b：如果目标有需要覆盖的文件，会先将覆盖文件备份;
- -p：保留源文件或目录的属性；
 
**案例**：
```
cp -r /usr/men /usr/zh    //将目录/usr/men及目录下的所有文件及其子目录复制到目录
cp -r -a aaa/* /bbb    //依然需要按Y来确认操作，并且把aaa目录以及子目录和文件属性也传递到了/bbb。
cp -T text.txt   a/b/c    //将text.txt 拷贝成文件名为'a/b/c'的文件
```
 
**来源**：
- [http://man.linuxde.net/cp](http://man.linuxde.net/cp)
- [https://linux.cn/article-2687-1.html](https://linux.cn/article-2687-1.html)
 
-------------------
 
##    mv
 
**作用**：用来对文件或目录重新命名，或者将文件从一个目录移到另一个目录中
 
**格式**：
- mv [OPTION]... [-T] SOURCE DEST
- mv [OPTION]... SOURCE... DIRECTORY
- mv [OPTION]... -t DIRECTORY SOURCE...
 
**常见选项**：
 
- -b：当文件存在时，覆盖前，为其创建一个备份；
- -u：当源文件比目标文件新或者目标文件不存在时，才执行移动操作。
- -T,--no-target-directory：将DEST当作普通文件处理,不会当作目录
 
**案例**：
```
mv test.txt test2.txt    //test.txt 改名为test2.txt
mv test.txt /opt/   //移动test.txt 到/opt/
mv -b test.txt /opt/   //移动test.txt 到/opt/ ，如果要覆盖的话，会先备份/opt/下之前的test.txt
```
 
**来源**：[http://man.linuxde.net/mv](http://man.linuxde.net/mv)
 
-------------------
 
##    rm
 
**作用**：命令可以删除一个目录中的一个或多个文件或目录，也可以将某个目录及其下属的所有文件及其子目录均删除掉。对于链接文件，只是删除整个链接文件，而原有文件保持不变。
 
**格式**：rm [OPTION]... FILE...
 
**常见选项**：
- -f：强制删除文件或目录； -i：删除已有文件或目录之前先询问用户；
- -r或-R：递归处理，将指定目录下的所有文件与子目录一并处理；
- --preserve-root：不对根目录进行递归操作；
- -v：显示指令的详细执行过程。
 
**案例**：
```
rm -f  test.txt
rm -rf  a/b
```
 
**来源**：[http://man.linuxde.net/rm](http://man.linuxde.net/rm)
 
-------------------
 
 
##  head
 
**作用**：显示文件的开头的内容。在默认情况下，head命令显示文件的头10行内容
 
**格式**：head [OPTION]... [FILE]...
 
**常见选项**：
- -n<数字>：指定显示头部内容的行数；
- -c<字符数>：指定显示头部内容的字符数；
- -v：总是显示文件名的头信息；
- -q：不显示文件名的头信息(感觉默认情况就啥也没显示的)；
 
**案例**：
```
head -3 install.log    //显示 install.log 前三行
head -n 3 -v install.log  //显示 install.log 前三行，并显示文件名到顶端
```
 
**来源**：[http://man.linuxde.net/head](http://man.linuxde.net/head)
 
-------------------
 
 
##  tail
**作用**：用于输入文件中的尾部内容
 
**格式**：tail [OPTION]... [FILE]...
 
**常见选项**：
- -f ，--follow，--follow=descriptor：动态持续显示文本追加内容；
- -F ，--follow=name --retry ：当文件不可访问时，会继续尝试；
- -n或--line=<行数N>：输出文件的尾部N（N位数字）行内容，默认情况下显示10行；
 
**案例**：
```
tail -5  file   //显示最后五行内容
tail -n 5 -f file    //显示文本最后五行内容，并别动态监控追加到尾部的内容。“tail -5 -f file” 会报错
tail -F file   //显示并监听文本尾部内容，如果失败，将重试
```
 
**来源**：[http://man.linuxde.net/tail](http://man.linuxde.net/tail)
 
-------------------
 

 
##  cut
**作用**：字面意思就是截取，截取的基本单位为行。用来显示行中的指定部分，删除文件中指定字段。
 
**格式**：cut OPTION... [FILE]...
 
**常见选项**：
- -b：仅显示行中指定字节范围的内容；
- -c：仅显示行中指定范围的字符；
- -d：指定字段的分隔符，默认的字段分隔符为“TAB”；
- -f：显示指定字段的内容；
- -n：与“-b”选项连用，不分割多字节字符；
- --complement：补足被选择的字节、字符或字段；
- --out-delimiter=<字段分隔符>：指定输出内容是的字段分割符；
 
**字节或者字符选项**
- N-：从第N个字节、字符、字段到结尾；
- N-M：从第N个字节、字符、字段到第M个（包括M在内）字节、字符、字段；
- -M：从第1个字节、字符、字段到第M个（包括M在内）字节、字符、字段；
 
**案例**：
```
//要处理的文本文件
$ cat test.txt
No Name Mark Percent 01
tom 69 91 02
jack 71 87 03
alex 68 98
 
$ cut -f 1 -d" "  test.txt    //显示第一列
$ cut -f 1,2 -d" "  test.txt    //显示第一列和第二列
 
```
 
**来源**：[http://man.linuxde.net/cut](http://man.linuxde.net/cut)
 
-------------------
 
 
##  touch
 
**作用**：一是用于把已存在文件的时间标签更新为系统当前的时间（默认方式），它们的数据将原封不动地保留下来。二是用来创建新的空文件
 
**格式**：touch [OPTION]... FILE...
 
**常见选项**：
- -a：修改atime
- -m：修改mtime
- -d：同时修改atime和 mtime
- -t：用格式化表示要修改成的时间戳 YYMMDDhhmm，而非当前时间
- --time=WORD  当WORD为access, atime, or use 时 同-a，当WORD 为  modify ，mtime 同-m
- -c：如果没有该文件，不执行创建操作
 
**Linux中的三种时间**
- 更改时间(mtime：Modify time):内容修改时间 ；
- 更改权限(ctime：Change time):只要文件发生改变，权限，属性如名称，内容发生改变，ctime都会发生更改；
- 读取时间(atime：Access time):读取文件内容的时间；
 
**查看Linux中的三种时间**
```
$ stat test2.txt
  File: `test2.txt'
  Size: 6               Blocks: 8          IO Block: 4096   regular file
Device: fd00h/64768d    Inode: 133850      Links: 1
Access: (0644/-rw-r--r--)  Uid: (    0/    root)   Gid: (    0/    root)
Access: 2017-07-16 08:42:49.036412390 -0400
Modify: 2017-07-16 08:31:18.038267641 -0400
Change: 2017-07-16 08:46:57.905270424 -0400
```
 
**案例**：
```
touch a.txt    //创建a.txt
touch a.txt    //atime,ctime,mtime都改为当前时间
touch -m a.txt   //mtime和ctime改为当前时间
touch -a a.txt   //atime和ctime改为当前时间
touch -d a.txt   //atime,ctime,mtime都改为当前时间
```
 
**来源**：
- [http://man.linuxde.net/touch](http://man.linuxde.net/touch)
- [http://www.letuknowit.com/post/50.html](http://www.letuknowit.com/post/50.html)
 
-------------------
 
## mkdir
 
**作用**：用来创建目录
 
 
**格式**：mkdir [OPTION]... DIRECTORY...
 
**常见选项**：
- -m<目标属性>或--mode<目标属性>建立目录的同时设置目录的权限；
- -p或--parents 若所要建立目录的上层目录目前尚未建立，则会一并建立上层目录；
 
 
**案例**：
```
$ mkdir -m 700 /usr/meng/     //在usr下创建meng目录，赋权限为700
$ mkdir    meng  //在当前目录下创建meng目录
$ mkdir  -p  a/b/c  //在当前目录下创建目录 a/b/c ，如果当前目录下没有a/b目录，则需要-p参数，否则创建失败
```
 
**来源**：[http://man.linuxde.net/mkdir](http://man.linuxde.net/mkdir)
 
-------------------

##    rmdir
 
**作用**：
 
**格式**：rmdir [OPTION]... DIRECTORY...
 
**常见选项**：
- -p或--parents：删除指定目录后，若该目录的上层目录已变成空目录，则将其一并删除；
 
**案例**：
```
rmdir -p  a/b/c
```
 
**来源**：[http://man.linuxde.net/rmdir](http://man.linuxde.net/rmdir)
 
-------------------

##  tr
**作用**：可以对来自标准输入的字符进行替换、压缩和删除。这个命令只能支持标准输入，不支持直接使用文本加工，通常可以通过将文本文件重定向到标准输入，传递给tr命令
 
**格式**：tr [OPTION]... SET1 [SET2]    其中SET1代表 一个字符集，SET2代表一个字符集，SET2是可选的，比如选项-d的时候就不需要SET2
 
**常见选项**：
- -c或--complement：取代所有不属于第一字符集的字符(相当于对 SET1 取‘非’操作后再作为 SET1 参数)；
- -d或--delete：删除所有属于第一字符集的字符；
- -s或--squeeze-repeats：把连续重复的字符以单独一个字符表示；
- -t或--truncate-set1：先删除第一字符集较第二字符集多出的字符；
 
**案例**：
```
$ echo "HELLO WORLD" | tr 'A-Z' 'a-z'
hello world    //大写转小写
 
 
$ echo "hello 123 world 456" | tr -d '0-9'
hello world    //删除数字
 
$ echo ABC123 | tr 'A-Z12' 'a-z'
abczz3    //大写转小写。但是多出来的1,2也会转化为z
 
$ echo ABC123 | tr -t 'A-Z12' 'a-z'
abc123    //将多出来的1，2先删除，在做转换操作
```
 
**来源**：[http://man.linuxde.net/tr](http://man.linuxde.net/tr)
 
-------------------


##  diff
**作用**：在最简单的情况下，比较给定的两个文件的不同。
 
**格式**：diff [OPTION]... FILE1 FILE2
 
**常见选项**：
- -i或--ignore-case：不检查大小写的不同；
- -b：忽略空格引起的差异；
- -B：忽略空行引起的差异；
- -c：默认情况会在标准输出的第一行表示出有差异的行号。使用-c,会显示每一行，在有差异的行首标记为!  ；
- --brief，-q：仅报告是否存在差异；
 
**案例**：
```
diff  test.txt test2.txt   //自行测试
diff -c test.txt test2.txt    //自行测试
diff -q test.txt test2.txt  //自行测试
 
//test.txt 和 test2.txt 如果内容相同，则上述案例都不会有结果输出
```
 
**来源**：[http://man.linuxde.net/diff](http://man.linuxde.net/diff)
 
-------------------

##    wget
**作用**：从指定的URL下载文件。wget默认会以最后一个符合/的后面的字符来命令，对于动态链接的下载通常文件名会不正确。
 
**格式**：wget [option]... [URL]...
 
**常见选项**：
- -r：递归下载方式；
- -i<FILE>：从指定文件获取要下载的URL地址；
- -O <FILE>   --output-document=FILE   ：把文档写到FILE文件中；
- -o<LOG>,  --output-file=LOG：    把下载日志写到FILE文件中；
- -A<后缀名>：指定要下载文件的后缀名，多个后缀名之间使用逗号进行分隔；
- -p，--page-requisites：下载所有为了html页面显示正常的文件；
- -P<prefix>，--directory-prefix=prefix：保存所有文件和目录到本地指定目录；
- -b ：   后台下载模式；
- -t<times>,--tries=times：设置下载失败时重试次数，默认情况重试次数是20次；
 
**案例**：
```
wget -i filelist.txt //下载filelist.txt 中的下载连接
wget -r -A.pdf url    //下载url中所有.pdf文件
wget -o download.log URL    //把下载日志写入download.log
wget -O wordpress.zip http://www.linuxde.net/download.aspx?id=1080   //将下载文件命名为wordpress.zip
```
 
**来源**：[http://man.linuxde.net/wget](http://man.linuxde.net/wget)
 
-------------------