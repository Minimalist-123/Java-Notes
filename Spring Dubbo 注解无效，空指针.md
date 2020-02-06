今天采用Spring注解  Dubbo(注解)的方式整合服务

消费者

@Controller
public class HomeController {

private static final Logger logger = LoggerFactory.getLogger(HomeController.class);


    @Reference(version = "0.0.1")
    private IDemoService demoService;

@RequestMapping(value = "/say", method = RequestMethod.GET)
    @ResponseBody
public String sayHello(){
demoService.sayHello();
        return "1";
}

@RequestMapping(value = "/add", method = RequestMethod.GET)
public void addition(){
System.out.println(demoService.addition(5, 5));
}


}

application xml:

<dubbo:application name="consumer-of-helloworld-app" />
<dubbo:registry address="zookeeper://192.168.10.150:2181" />
    <dubbo:annotation package="com.demo.dubbo.controller"/>

提供者provider

import com.alibaba.dubbo.config.annotation.Service;

@Service(version = "0.0.1")
public class DemoServiceImpl implements IDemoService {


@Override
public void sayHello() {
System.out.println("hello dubbo!");
}


@Override
public int addition(int a, int b) {
int result = a + b;
System.out.println("addition:" + result);
return result;
}


}

application xml:

<dubbo:annotation package="com.winks.dubbo.demo.provider" />


其他配置都在dubbo.properties

消费者采用的是 bat启动，

消费者的 IDemoService demoService 始终是空指针。

有谁能否提供一个是dubbo注解实例，谢谢！



【解决方法】————————

【方法1】我成功了，出现空指针的原因是：spring mvc扫描的时候根本无法识别@Reference ，同一方面，dubbo的扫描也无法识别Spring @Controller ,所以两个扫描的顺序要排列好，如果先扫了controller，这时候把控制器都实例化好了，再扫dubbo的服务，就会出现空指针。

下面是我成功的代码：

<mvc:annotation-driven />

<!-- 查找xxx路径下所有@Controller 注释类,添加与项目相关的controller -->

<dubbo:annotation package="XXX.XXX.XXX.controller" />

<context:component-scan base-package="XXX.XXX.XXX.controller"/>

祝成功


【方法2】你的方法确实管用，调节顺序之后就可以。
如果其他同学试了这个方法不管用，可以试试加一个Bean。
@Component
public class DubboSupport
{
    @Reference(version="1.0.0")
    protected MemberService memberService;
    
    public MemberService getMemberService(){
        return this.memberService;
    }
}

然后在controller中：
  //方式二
    @Autowired
    private DubboSupport dubboSupport;
这种方法不受顺序影响
