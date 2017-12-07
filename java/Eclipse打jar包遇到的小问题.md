title: Eclipse打jar包遇到的小问题
tags: 
	- Eclipse
	- Java
	- Jar
	- Maven

categories:
	- Java
---------------------




>最近实在是太忙，没更新博客的时间。。。😭

打jar包，一个很简单的工作
通常我们使用export Runnable JAR就行了


 <img src="http://ou9e0q35h.bkt.clouddn.com/14c507e3-76a1-42f9-90fe-bfc238b3f14c2.png" style="width:800px">
打完包直接`java -jar ebotappdata-server.jar` 就能执行了，完全没问题。
过了两天我修改了一下该任务的代码，重新打包，执行测试，报错：
```
Exception in thread "main" java.lang.ExceptionInInitializerError
        at cn.entgroup.ebotappdata.App.main(App.java:30)
Caused by: java.lang.NullPointerException
        at java.util.Properties$LineReader.readLine(Properties.java:434)
        at java.util.Properties.load0(Properties.java:353)
        at java.util.Properties.load(Properties.java:341)
        at cn.entgroup.ebotappdata.service.ClientService.<clinit>(ClientService.java:41)
```
这个错误在如下代码处报错的
```
properties.load(DataSourceUtil.class.getClassLoader().getResourceAsStream("dbcp2.properties"));
```
然而我并没改动过这个代码，心里突然就不开心了，我觉得不是我代码的问题，毕竟本地也能正常测试，前两天还好好的。还好之前那个jar包我并没删掉，把两份jar包拿来比较（使用某解压工具查看）

 <img src="http://ou9e0q35h.bkt.clouddn.com/14c507e3-76a1-42f9-90fe-bfc238b3f14c3.png" style="width:800px">
 <img src="http://ou9e0q35h.bkt.clouddn.com/14c507e3-76a1-42f9-90fe-bfc238b3f14c4.png" style="width:800px">

这尼玛结构都不一样了。要报错的结构把配置文件放到了resources里面，思索许久，觉得是项目的build path的问题，也去google了不少时间，google上面的方法大致是说把resources放到包结构下面去[连接](https://stackoverflow.com/questions/25718327/filenotfoundexception-starting-jar-cant-see-file-in-resources-folder)，我不想用新的方法去解决这个问题，我就想搞明白我前两天是怎么得到结构正确的jar包的。在我反复倒腾build path的时候，正确的结构终于被我倒腾出来了，什么情况下正确的结构会出现呢，当我把src/main/resources目录先移除然后在次添加上打jar包，得到的jar结构正确。当我执行maven update语句后打出来的包结构包含resources。(感觉到maven深深的恶意)，我平时会喜欢配置一下maven的build标签，再update，这样使用可以不用手动调编译jdk版本啥的，但也是因为build标签里面的配置造成这jar的问题
```
<build>
        <!-- 源目录 -->
        <sourceDirectory>src/main/java</sourceDirectory>
        <testSourceDirectory>src/test/java</testSourceDirectory>
        <resources>
            <resource>
                <directory>src/main/resources</directory>
            </resource>
        </resources>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.7.0</version>
                <configuration>
                    <source>1.7</source>
                    <target>1.7</target>
                </configuration>
            </plugin>
        </plugins>
    </build>
```
不说maven这，来看看手动新添加src/main/resources和maven update两者的比较：
 <img src="http://ou9e0q35h.bkt.clouddn.com/14c507e3-76a1-42f9-90fe-bfc238b3f14c7.png" style="width:800px">
 <img src="http://ou9e0q35h.bkt.clouddn.com/14c507e3-76a1-42f9-90fe-bfc238b3f14c8.png" style="width:800px">
是的，你没看错，就是因为Excluded:**，你可以直接修改Excluded，然后也ok
 <img src="http://ou9e0q35h.bkt.clouddn.com/14c507e3-76a1-42f9-90fe-bfc238b3f14c11.png" style="width:800px">
 
其实maven也可以直接打jar包的，用起来还很方便的。用eclipse打jar真的坑，并且你可以用解压工具观察一下结构，eclipse的打包方式相对来说，那叫个乱哟，，，，那为啥我还要用eclipse打包，或许只是因为方便，不想去贴maven打包的配置。
maven打包的配置如下，配置好打包配置后，在项目上右键 Run as -> Maven install，然后就能在maven对应库里找到一个以jar-with-dependencies.jar结尾的jar包
```xml
<build>
        <!-- 源目录 -->
        <sourceDirectory>src/main/java</sourceDirectory>
        <testSourceDirectory>src/test/java</testSourceDirectory>
        <resources>
            <resource>
                <directory>src/main/resources</directory>
            </resource>
        </resources>
        <plugins>
         <plugin>
                <artifactId>maven-assembly-plugin</artifactId>
                <configuration>
                    <descriptorRefs>
                        <descriptorRef>jar-with-dependencies</descriptorRef>
                    </descriptorRefs>
                    <archive>
                        <manifest>
                            <mainClass>cn.entgroup.ebotappdata.App</mainClass>
                        </manifest>
                    </archive>
                </configuration>
                <executions>
                    <execution>
                        <id>make-assembly</id>
                        <phase>package</phase>
                        <goals>
                            <goal>single</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.7.0</version>
                <configuration>
                    <source>1.7</source>
                    <target>1.7</target>
                </configuration>
            </plugin>
        </plugins>
    </build>
```