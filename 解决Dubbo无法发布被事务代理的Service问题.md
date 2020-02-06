原因：
在XxxServiceImpl类上加入事务注解后，Spring会为此类基于JDK动态代理技术创建代理对象，创建的代理对象完整类名为com.sun.proxy.$Proxyxx(xx为数字)，导致Dubbo在进行包匹配时没有成功（因为我们在发布服务时扫描的包为cn.itcast.dubbo.service），所以后面真正发布服务的代码没有执行。
解决方法：

第一步：指定使用cglib的动态代理
<tx:annotation-driven transaction-manager=“transactionManager” proxy-target-class=“true”/>
第二步：指定服务的接口类型
@Service(interfaceClass = XxxService.class)



原文链接：https://blog.csdn.net/qq_37252930/article/details/94721703
