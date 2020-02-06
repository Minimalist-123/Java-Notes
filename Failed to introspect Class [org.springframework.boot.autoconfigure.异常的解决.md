最近在看spring-boot，有看到利用redis，将session放到缓存中，实现分布式系统的session共享，引入下图的jar包



加入了启用redisHttpSesion的配置。



配置redis


java.lang.IllegalStateException: Failed to introspect Class [org.springframework.boot.autoconfigure.session.SessionAutoConfiguration$ServletSessionConfiguration] from ClassLoader [sun.misc.Launcher$AppClassLoader@18b4aac2]
    at org.springframework.util.ReflectionUtils.getDeclaredMethods(ReflectionUtils.java:507) ~[spring-core-5.1.7.RELEASE.jar:5.1.7.RELEASE]

配置完毕，启动却报了上述异常，无法映射这个sessionAUtoConfiguration注解对应的类，一直没有查到是什么原因

我的spring-boot版本是用的最新版的2.1.5.RELEASE。

尝试了各种方法，最后选择将springboot版本降到2.1.3.RELEASE,再启动发现就没有问题了。

后来查询相关资料，知道了spring-boot不同的starter有不同的版本要求支持。


————————————————
版权声明：本文为CSDN博主「m0_37837382」的原创文章，遵循 CC 4.0 BY-SA 版权协议，转载请附上原文出处链接及本声明。
原文链接：https://blog.csdn.net/m0_37837382/article/details/90727374
