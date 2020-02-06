【Spring的注解@Qualifier用法】
在Controller中需要注入service那么我的这个server有两个实现类如何区分开这两个impl呢？

根据注入资源的注解不同实现的方式有一点小小的区别

下面上铺垫图

请忽略我的红线

##在Controller中使用 @Autowired注入时


Qualifier的意思是合格者，通过这个标示，表明了哪个实现类才是我们所需要的，添加@Qualifier注解，需要注意的是@Qualifier的参数名称为我们之前定义@Service注解的名称之一。

##使用@Resource注入时

使用@resource注入时比较简单了注解自带了“name”的val就是@Service注解的名称之一。


原文链接：https://blog.csdn.net/qq_36567005/article/details/80611139
