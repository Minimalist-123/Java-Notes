java类加载器分类详解
  1、Bootstrap ClassLoader：启动类加载器，也叫根类加载器，负责加载java的核心类库，例如(%JAVA_HOME%/lib)目录下的rt.jar（包含System，String这样的核心类），根类加载器非常特殊，它不是java.lang.ClassLoader的子类，它是JVM自身内部由C/C++实现的，并不是java实现的

  2、Extension ClassLoader：扩展类加载器，负责加载扩展目录(%JAVA_HOME%/jre/lib/ext)下的jar包，用户可以把自己开发的类打包成jar包放在这个目录下即可扩展核心类以外的功能

  3、System ClassLoader\APP ClassLoader，系统类加载器，又称为应用程序类加载器，是加载CLASSPATH环境变量下所指定的jar包与类路径，一般来说，用户自定义的就是由APP ClassLoader加载的

各类加载器之间的关系
   以组合关系复用父类加载器的父子关系，注意，这里的父子关系并不是以继承关系实现的

类加载器的双亲委派加载机制(重点)
    当一个类收到了类加载请求，他首先不会尝试自己去加载这个类，而是把这个请求委派给父类去完成，每一个层次类加载都是如此，因此所有的加载请求都应该传送到启动类加载器中，只有当父类加载器反馈自己无法完成这个请求的时候（在它的加载路径里找不到这个所需要加载的类），子类加载器才会尝试自己去加载

过程如下图所示：



双亲委派模型的源码实现
   主要体现在ClassLoader的loadClass()方法，思路很简单：先检查是否已经被加载，若没有被加载则调用父类的LoadClass()方法，若父类加载器为空，则默认使用启动类加载器作为父类加载器，如果父类加载器加载失败，抛出ClassNotFoundException异常后，调用自己的findClass()方法进行加载
————————————————
版权声明：本文为CSDN博主「笙南」的原创文章，遵循 CC 4.0 BY-SA 版权协议，转载请附上原文出处链接及本声明。
原文链接：https://blog.csdn.net/weixin_38118016/article/details/79579657
