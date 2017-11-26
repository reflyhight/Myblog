title: java中的时区设置
tags: 
	- Java
	- date

categories:
	- Java
-------------------





java 进程放到服务器上后。发现进程获取到的时间比本地时间晚8小时。


测试代码
```

import java.text.SimpleDateFormat;
import java.util.Date;
import java.util.TimeZone;

public class TimeTest {

        public static void main(String[] args) {
                Date date= new Date();
                SimpleDateFormat formate=new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
                System.out.println(formate.format(date));

        }
}
```

服务器运行如下效果
```
]$ javac TimeTest.java
]$ java TimeTest
2016-12-15 02:41:31
]$ date
Thu Dec 15 10:41:35 CST 2016
```

网上查询，说得不是特别清楚。这个问题是因为JAVA虚拟机中的默认时区的问题。以下两种解决方案

## 通过JAVA_OPTION配置java  虚拟机参数

```
]$ java -Duser.timezone=GMT+8 TimeTest
2016-12-15 10:45:37
]$ date
Thu Dec 15 10:45:42 CST 2016
```

## 通过java代码设置
vim TimeTest.java  
```
import java.text.SimpleDateFormat;
import java.util.Date;
import java.util.TimeZone;

public class TimeTest {

        public static void main(String[] args) {
                TimeZone.setDefault(TimeZone.getTimeZone("GMT+08"));
                Date date= new Date();
                SimpleDateFormat formate=new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
                System.out.println(formate.format(date));

        }
}
```
执行效果：

```
]$ javac TimeTest.java         
]$ java TimeTest
2016-12-15 10:48:03
]$ date
Thu Dec 15 10:48:07 CST 2016
```