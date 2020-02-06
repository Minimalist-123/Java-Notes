【Dubbo无法发布被事务代理的Service问题】


前言

在使用注解式dubbo开发的过程中，忽然发现Service上只要有@transactional注解或者是配置的事务切面时，该Service不能被dubbo发布。

问题详情

dubbo的配置：


[html] view plain copy
<span style="white-space:pre">    </span><!-- 定义注册中心，采用zookeeper -->  
    <dubbo:registry protocol="zookeeper" address="127.0.0.1:2181"/>  
    <!-- 定义协议为dubbo，在端口20880上暴露服务 -->  
    <dubbo:protocol name="dubbo" port="20880"/>  
    <dubbo:annotation package="com.rondo"/>  
<span style="white-space:pre">	</span><!-- 定义注册中心，采用zookeeper -->
	<dubbo:registry protocol="zookeeper" address="127.0.0.1:2181"/>
	<!-- 定义协议为dubbo，在端口20880上暴露服务 -->
	<dubbo:protocol name="dubbo" port="20880"/>
	<dubbo:annotation package="com.rondo"/>
事务的配置：
[html] view plain copy
<span style="white-space:pre">    </span><!-- 注解方式配置事务 -->  
    <bean id="txManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">  
        <property name="dataSource" ref="dataSource"/>  
    </bean>  
    <span style="white-space:pre">    </span><tx:annotation-driven transaction-manager="txManager" proxy-target-class="true"/>  
<span style="white-space:pre">	</span><!-- 注解方式配置事务 -->
	<bean id="txManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
		<property name="dataSource" ref="dataSource"/>
	</bean>
    <span style="white-space:pre">	</span><tx:annotation-driven transaction-manager="txManager" proxy-target-class="true"/>
[java] view plain copy
@Component  
@Service//这个service注解是dubbo的  
@Transactional  
public class UserServiceImpl implements UserService {  
  
    @Autowired  
    private UserMapper userMapper;  
  
    @Override  
    public Mapper<User> getMapper() {  
        return userMapper;  
    }  
}  
@Component
@Service//这个service注解是dubbo的
@Transactional
public class UserServiceImpl implements UserService {
 
	@Autowired
	private UserMapper userMapper;
 
	@Override
	public Mapper<User> getMapper() {
		return userMapper;
	}
}

这种情况下，UserService不能被dubbo发布。
解决思路

这个问题简直~~，后来发现，在使用事务的时候，配置文件中使用了cglib的方式（proxy-target-class="true"）为service生成代理，而当dubbo扫描注解的时候，这个被代理的UserService并没有dubbo的@service注解，因为dubbo定义这个注解的时候，没有允许子类集成父类的注解。

cglib生成代理的方式恰好是生成该类的子类，那么问题显而易见了，就是这个代理上缺少了@Service，怎么解决这个问题呢？------------修改dubbo源码是个好办法，让@service可以被子类继承。

解决方法

1.https://github.com/alibaba/dubbo上有dubbo的源码，找到dubbo-config/dubbo-config-api/src/main/java/com/alibaba/dubbo/config/annotation/Service.java，复制里面所有内容。

2.新建一个txt文件并修改名称为“Service.java”，打开粘贴所有内容，并添加注解@Inherited，别忘了导入Inherited的包，如图：

3.javac Service.java将该java文件编译为class，具体步骤就不用说了吧。。。

4.使用解压软件打开dubbo-x.x.x.jar，找到包com.alibaba.dubbo.config.annotation下的Service.class，用自己编译的替换掉原有的。

效果

做完上面这些，我觉得问题肯定解决了，于是重新启动了一下工程，结果消费端调用UserService中的方法的时候还是会报没有provider！word 天~，这怎么可能！于是我启动了dubbo-admin(关于这个玩意如果有疑问自己网上下载个dubbo-admin.war，你就明白了)，在查看发布的接口的时候发现悲剧了

我原本要发布的UserService，竟然是SpringProxy。发现dubbo的@Service注解中有一个属性“interfaceName”，这下问题得到彻底解决，将UserServiceImpl中的@Service注解修改为@Service(interfaceName="com.rondo.business.service.UserService")，即指定了接口名称，效果如图：



然后，在消费端调用了一下，调用成功，而且事务也是有效的。

【原文链接】：https://blog.csdn.net/benxiaohai529/article/details/78318810
