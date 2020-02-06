JDBC使用8.0驱动包连接mysql设置时区serverTimezone
驱动包用的是新版 mysql-connector-java-8.0.16.jar
新版的驱动类改成了com.mysql.cj.jdbc.Driver
新版驱动连接url也有所改动
I、指定时区

如果不设置时区会相差13个小时
比如在java代码里面插入的时间为：2019-07-26 19:28:02
但是在数据库里面显示的时间却为：2019-07-26 06:28:02

所以使用上海时间（注意：没有asia/beijing时区）
serverTimezone=Asia/Shanghai

II、指定是否用ssl连接，true值还报错了
useSSL=false

完整代码：

url=jdbc:mysql://ip:port/xxx?useUnicode=true&characterEncoding=UTF-8&zeroDateTimeBehavior=convertToNull&serverTimezone=Asia/Shanghai&useSSL=false
driverClassName=com.mysql.cj.jdbc.Driver


原文链接：https://www.cnblogs.com/qdwyg2013/p/11275090.html
