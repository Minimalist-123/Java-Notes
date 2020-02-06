 通过前面两章对Spring Cloud Ribbon和Spring Cloud Hystrix的介绍，我们已经掌握了开发微服务应用时，两个重要武器，学会了如何在微服务架构中实现客户端负载均衡的服务调用以及如何通过断路器来保护我们的微服务应用。这两者将被作为基础工具类框架广泛地应用在各个微服务的实现中，不仅包括我们自身的业务类微服务，也包括一些基础设施类微服务（比如网关）。此外，在实践过程中，我们会发现对这两个框架的使用几乎是同时出现的。既然如此，那么是否有更高层次的封装来整合这两个基础工具以简化开发呢？本章我们即将介绍的Spring Cloud Ribbon与Spring Cloud Hystrix，除了提供这两者的强大功能之外，它还提供了一种声明式的Web服务客户端定义方式。

 我们在使用Spring Cloud Ribbon时，通常都会利用它对RestTemplate的请求拦截来实现对依赖服务的接口调用，而RestTemplate已经实现了对HTTP请求的封装处理，形成了一套模版化的调用方法。在之前的例子中，我们只是简单介绍了RestTemplate调用对实现，但是在实际开发中，由于对服务依赖对调用可能不止于一处，往往一个接口会被多处调用，所以我们通常都会针对各个微服务自行封装一些客户端累来包装这些依赖服务的调用。这个时候我们会发现，由于RestTemplate的封装，几乎每一个调用都是简单的模版化内容。综合上述这些情况，Spring Cloud Fegin在此基础上做了进一步封装，由它来帮助我们定义和实现依赖服务接口的定义。在Spring Cloud Feign的实现下，我们只需创建一个接口并用注解的方式来配置它，即可完成对服务提供方的接口绑定，简化了在使用Spring Cloud Ribbon时自行封装服务调用客户端的开发量。Spring Cloud Feign具备可插拔的注解支持，包括Feign注解和JAX-RS注解。同时，为了适应Spring的广大用户，它在Netflix Feign的基础上扩展了对Spring MVC的注解支持。这对于习惯于Spring MVC的开发者来说，无疑是一个好消息，你我这样可以大大减少学习适应它的成本。另外，对于Feign自身的一些主要组件，比如编码器和解码器等，它也以可插拔的方式提供，在有需求等时候我们以方便扩张和替换它们。

快速入门
 在本节中，我们将通过一个简单示例来展示Spring Cloud Feign在服务客户端定义所带来的便利。下面等示例将继续使用之前我们实现等hello-service服务，这里我们会通过Spring Cloud Feign提供的声明式服务绑定功能来实现对该服务接口的调用。

▪️首先，创建一个Spring Boot基础工程，取名为kyle-service-feign，并在pom.xml中引入spring-cloud-starter-eureka和spring-cloud-starter-feign依赖，具体内容如下所示。

<parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>1.4.5.RELEASE</version>
        <relativePath/>
    </parent>

    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
        <java.version>1.8</java.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-eureka</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-feign</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
            <exclusions>
                <exclusion>
                    <groupId>org.springframework.boot</groupId>
                    <artifactId>spring-boot-starter-tomcat</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-undertow</artifactId>
            <scope>provided</scope>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>Camden.SR7</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

    <repositories>
        <repository>
            <id>spring-milestones</id>
            <name>Spring Milestones</name>
            <url>https://repo.spring.io/milestone</url>
            <snapshots>
                <enabled>false</enabled>
            </snapshots>
        </repository>
    </repositories>
▪️创建应用主类Application，并通过@EnableFeignClients注解开启Spring Cloud Feign的支持功能。

@EnableEurekaClient
@SpringBootApplication
@EnableFeignClients(basePackages = { "com.kyle.client.feign.inter" })
public class Application {
    
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
▪️定义HelloServiceFeign，接口@FeignClient注解指定服务名来绑定服务，然后再使用Spring MVC的注解来绑定具体该服务提供的REST接口。

@FeignClient(value = "hello-service-provider")
public interface HelloServiceFeign {

    @RequestMapping(value = "/demo/getHost", method = RequestMethod.GET)
    public String getHost(String name);

    @RequestMapping(value = "/demo/postPerson", method = RequestMethod.POST, produces = "application/json; charset=UTF-8")
    public Person postPerson(String name);
}
注意：这里服务名不区分大小写，所以使用hello-service-provider和HELLO-SERVICE-PROVIDER都是可以的。另外，在Brixton.SR5版本中，原有的serviceId属性已经被废弃，若要写属性名，可以使用name或value。

▪️接着，创建一个RestClientController来实现对Feign客户端的调用。使用@Autowired直接注入上面定义的HelloServiceFeign实例，并在postPerson函数中调用这个绑定了hello-service服务接口的客户端来向该服务发起/hello接口的调用。

@RestController
public class RestClientController {

    @Autowired
    private HelloServiceFeign client;

    /**
     * @param name
     * @return Person
     * @Description: 测试服务提供者post接口
     * @create date 2018年5月19日上午9:44:08
     */
    @RequestMapping(value = "/client/postPerson", method = RequestMethod.POST, produces = "application/json; charset=UTF-8")
    public Person postPerson(String name) {
        return client.postPerson(name);
    }

    /**
     * @param name
     * @return String
     * @Description: 测试服务提供者get接口
     * @create date 2018年5月19日上午9:46:34
     */
    @RequestMapping(value = "/client/getHost", method = RequestMethod.GET)
    public String getHost(String name) {
        return client.getHost(name);
    }
}
▪️最后，同Ribbon实现的服务消费者一样，需要在application.properties中指定服务注册中心，并定义自身的服务名为feign-service-provider，为了方便本地调试与之前的Ribbon消费者区分，端口使用8868。

#spring.application.name=ribbon-service-provider
eureka.instance.appname=feign-service-provider
eureka.instance.virtualHostName=feign-service-provider
eureka.instance.secureVirtualHostName=feign-service-provider

server.port=8868
eureka.instance.instance-id=${spring.cloud.client.ipAddress}:ribbon-service-provider-peer:${server.port}
#注册到另外两个节点，实现集群
eureka.client.serviceUrl.defaultZone=http://localhost:8887/eureka/,http://localhost:8888/eureka/,http://localhost:8889/eureka/
测试验证
 如之前验证Ribbon客户端负载均衡一样，我们先启动服务注册中心以及两个HELLO-SERVICE-PROVIDER，然后启动FEIGN-SERVICE-PROVIDER，此时我们在Eureka信息面板中可以看到如下内容：
注册中心的服务注册
 发送几次GET请求到http://localhost:8868/client/getHost?name=kyle，可以得到如之前Ribbon实现时一样到效果，正确返回hi, kyle! i from 10.166.37.142:8877。依然是利用Ribbon维护了针对HELLO-SERVICE-PROVIDER的服务列表信息，并且通过轮询实现了客户端负载均衡。而与Ribbon不同到是，通过Feign只需定义服务绑定接口，以声明式的方法，优雅而简单地实现了服务调用。

测试结果补充
8877节点
8878节点
参数绑定
 现实系统中的各种业务接口要比上一节复杂得多，我们会再HTTP的各个位置传入各种不同类型的参数，并且再返回响应的时候也可能是一个复杂的对象结构。再本节中，我们将详细介绍Feign中的不同形式参数的绑定方法。

 再开始介绍Spring Cloud Feign的参数绑定之前，我们先扩张以下服务提供者hello-service-provider。增加下面这些接口，其中包含带有Request参数的请求、带有Header信息的请求、带有RequestBody的请求以及请求响应体中是一个对象的请求。

/**
     * @param name
     * @return Person
     * @Description: post接口
     * @create date 2018年5月19日上午9:44:08
     */
    @RequestMapping(value = "/demo/postPerson", method = RequestMethod.POST, produces = "application/json; charset=UTF-8")
    public Person postPerson(@RequestParam("name") String name) {
        Person person = new Person();
        person.setName(name);
        person.setAge("10");
        person.setSex("man");
        return person;
    }

    /**
     * @param person
     * @return Person
     * @Description: post接口
     * @create date 2018年6月27日下午5:50:56
     */
    @RequestMapping(value = "/demo/postPerson", method = RequestMethod.POST, produces = "application/json; charset=UTF-8")
    public Person postPerson(@RequestBody Person person) {
        person.setAge("10");
        person.setSex("man");
        return person;
    }

    /**
     * @param name
     * @return String
     * @Description: get接口
     * @create date 2018年5月19日上午9:46:34
     */
    @RequestMapping(value = "/demo/getHost", method = RequestMethod.GET)
    public String getHost(@RequestParam("name") String name) {
        return "hi, " + name + "! i from " + ipAddress + ":" + port;
    }

    /**
     * @param name
     * @param age
     * @return String
     * @Description: get接口,包含header信息
     * @create date 2018年6月27日下午5:43:29
     */
    @RequestMapping(value = "/demo/getHost", method = RequestMethod.GET)
    public String getHost(@RequestParam("name") String name, @RequestHeader Integer age) {
        return "hi, " + name + ", your age is " + age + "! i from " + ipAddress + ":" + port;
    }
 在完成了对hello-service-provider的改造之后，下面我们开始在快速入门示例的kyle-service-feign应用中实现这些新增的绑定。

首先，在kyle-service-feign中创建Person类。
然后，在HelloServiceFeign接口中增加对上述三个新增接口的绑定声明，修改后，完成的HelloServiceFeign如下所示：
@FeignClient(value = "hello-service-provider")
public interface HelloServiceFeign {

    @RequestMapping(value = "/demo/getHost", method = RequestMethod.GET, produces = "application/json")
    public String getHost(@RequestParam("name") String name);

    @RequestMapping(value = "/demo/postPerson", method = RequestMethod.POST, produces = "application/json; charset=UTF-8")
    public Person postPerson(@RequestParam("name") String name);

    @RequestMapping(value = "/body/postPerson", method = RequestMethod.POST, produces = "application/json; charset=UTF-8")
    public Person postPerson(@RequestBody Person person);

    @RequestMapping(value = "/head/getHost", method = RequestMethod.GET, produces = "application/json")
    public String getHost(@RequestParam("name") String name, @RequestHeader("age") Integer age);
}
 这里一定要注意，再定义各参数绑定时，@RequestParam、@RequestHeader等可以指定参数名称的主角，它们的value千万不能少。在Spring MVC程序中，这些注解会根据参数名来作为默认值，但是在Feign中绑定参数必须通过value属性来指明具体的参数名，不然会抛出==IllegalStateException==异常，value属性不能为空。

最后，在RestClientController中新增两个接口，来对本节新增的声明接口调用，修改后的完整代码如下所示：
    /**
     * @return Person
     * @Description: post接口
     * @create date 2018年6月27日下午5:50:56
     */
    @RequestMapping(value = "/feign/project/postPerson", method = RequestMethod.POST, produces = "application/json; charset=UTF-8")
    public Person postPerson() {
        Person person = new Person();
        person.setName("kyle");
        return client.postPerson(person);
    }
    /**
     * @param name
     * @param age
     * @return String
     * @Description: get接口,包含header信息
     * @create date 2018年6月27日下午5:43:29
     */
    @RequestMapping(value = "/feign/head/getHost", method = RequestMethod.GET)
    public String getHost(@RequestParam("name") String name, @RequestParam("name") Integer age) {
        return client.getHost(name, age);
    }
测试验证
 在完成上述改造之后，启动服务注册中心、两个hello-service-privider服务以及我们改造的kyle-service-feign。通过发送GET请求到==http://localhost:8868/feign/head/getHost?name=kyle&age=18==，通过发送POST请求到==http://localhost:8868/feign/project/postPerson==，请求触发HelloServiceFeign对新增接口的调用。最终，我们会获得如下图的结果，代表接口绑定和调试成功。

新接口测试
新接口测试
Ribbon使用
 由于Spring Cloud Feign的客户端负载均衡是通过Spring Cloud Ribbon实现的，所以我们可以直接配置Ribbon客户端的方式来自定义各个服务客户端调用参数。那么我们如何使用Spring Cloud Feign的工程中使用Ribbon的配置呢？

全局配置
 全局配置的方法非常简单，我们可以直接使用ribbon.<key>=<value>的方式来设置ribbon的各项默认参数。如下：

#以下配置全局有效
ribbon.eureka.enabled=true
#建立连接超时时间，原1000
ribbon.ConnectTimeout=60000
#请求处理的超时时间，5分钟
ribbon.ReadTimeout=60000
#所有操作都重试
ribbon.OkToRetryOnAllOperations=true
#重试发生，更换节点数最大值
ribbon.MaxAutoRetriesNextServer=10
#单个节点重试最大值
ribbon.MaxAutoRetries=1
指定服务配置
 大多数情况下，我们对于服务调用的超时时间可能会根据实际服务的特性做一些调整，所以仅仅进行个性化配置的方式与使用Spring Cloud Ribbon时的配置方式是意义的，都采用<client>.ribbon.key=value的格式进行设置。但是，这里就有一个疑问了，<cleint>所指代的Ribbon客户端在那里呢？

 回想一下，在定义Feign客户端的时候，我们使用了@FeignClient注解。在初始化过程中，Spring Cloud Feign会根据该注解的name属性或value属性指定的服务名，自动创建一个同名的Ribbon客户端。如下：

#以下配置对服务hello-service-provider有效
hello-service-provider.ribbon.eureka.enabled=true
#建立连接超时时间，原1000
hello-service-provider.ribbon.ConnectTimeout=60000
#请求处理的超时时间，5分钟
hello-service-provider.ribbon.ReadTimeout=60000
#所有操作都重试
hello-service-provider.ribbon.OkToRetryOnAllOperations=true
#重试发生，更换节点数最大值
hello-service-provider.ribbon.MaxAutoRetriesNextServer=10
#单个节点重试最大值
hello-service-provider.ribbon.MaxAutoRetries=1
负载均衡策略
 Spring Cloud Ribbon默认负载均衡策略是轮询策略，不过该不一定满足我们的需要。Ribbon一共提供了7种负载均衡策略，如果我们需要ZoneAvoidanceRule，首先要在application.properties文件中添加配置，如下所示：

ribbon.NFLoadBalancerRuleClassName=com.netflix.loadbalancer.ZoneAvoidanceRule
 不过，只是添加了如上配置，还无法实现负载均衡策略的更改。我们还需要实例化该策略，可以在应用主类中直接加入IRule实例的创建，如下：

/**
 * 服务调用者，，eureka客户端 feign调用
 *
 * @version
 * @author kyle 2017年7月9日下午6:39:15
 * @since 1.8
 */
@EnableEurekaClient
@SpringBootApplication
@EnableFeignClients(basePackages = { "com.kyle.client.feign.inter" })
public class Application {

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }

    @Bean
    public IRule feignRule() {
        return new ZoneAvoidanceRule();
    }
}
 想要深入了解Ribbon的原理，或者想详细了解7种负载均衡策略的，可以参考我另一篇博客《Ribbon详解》，我会在博客最下面给出链接。

非Spring Boot工程使用Feign
 从前两节来看在Spring Boot工程中使用Feign，非常的便利。不过实际生产中，在微服务的初期只能从次要系统开始进行改造，可能很多系统由于历史原因仍然是非Spring Boot的工程，然后这些系统如何使用微服务？如何使用注册中心？如何进行负载均衡呢？

 ▪️首先我们在kyle-service-feign创建调用接口OldSystemPostFeign和OldSystemGetFeign，然后使用feign注解提供的相关注解，包含@RequestLine、@Param、@HeaderParam、@Headers等，主要提供了请求方法、请求参数、头信息参数等操作。

/**
 * 非Spring Boot工程使用feign组件,post请求
 *
 * @version
 * @author kyle 2018年6月28日下午2:05:39
 * @since 1.8
 */
public interface OldSystemPostFeign {

    /**
     * @param person
     * @return Person
     * @Description:
     * @create date 2018年6月28日下午2:08:56
     */
    @RequestLine("POST /body/postPerson") // post 提交
    @Headers({ "Content-Type: application/json; charset=UTF-8", "Accept: application/json; charset=UTF-8" })
    public Person postPerson(Person person);

}
/**
 * 非Spring Boot工程使用feign组件,get请求
 *
 * @version
 * @author kyle 2018年6月28日下午3:06:34
 * @since 1.8
 */
public interface OldSystemGetFeign {
    /**
     * @param name
     * @return String
     * @Description:
     * @create date 2018年6月28日下午2:08:43
     */
    @RequestLine("GET /demo/getHost?name={name}")
    public String getHost(@Param("name") String name);

    /**
     * @param name
     * @param age
     * @return String
     * @Description:
     * @create date 2018年6月28日下午2:14:38
     */
    @RequestLine("GET /head/getHost?name={name}")
    @Headers({ "age: {age}" })
    public String getHost(@Param("name") String name, @Param("age") String age);
}
 ▪️我们需要脱离Spring Boot和Spring Cloud的支持，使用feign原生的一些东西。在进行Feign封装之前我们需要一些额外的组件，比如编码器。新增组件依赖如下所示：

<dependency>
            <groupId>com.netflix.feign</groupId>
            <artifactId>feign-core</artifactId>
            <version>8.18.0</version>
        </dependency>
        <dependency>
            <groupId>com.netflix.feign</groupId>
            <artifactId>feign-ribbon</artifactId>
            <version>8.18.0</version>
        </dependency>
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>fastjson</artifactId>
            <version>1.2.33</version>
        </dependency>
        <dependency>
            <groupId>io.github.openfeign</groupId>
            <artifactId>feign-jackson</artifactId>
            <version>9.3.1</version>
        </dependency>
        <dependency>
            <groupId>com.fasterxml.jackson.core</groupId>
            <artifactId>jackson-core</artifactId>
        </dependency>
 ▪️我们需要一个feign-clientproperties文件，来进行ribbon相关的参数配置，配置如下：

#对当前实例的重试次数
hello-service-provider.ribbon.MaxAutoRetries=1
#切换实例的重试次数
hello-service-provider.ribbon.MaxAutoRetriesNextServer=2
#对所有操作请求都进行重试
hello-service-provider.ribbon.OkToRetryOnAllOperations=true
#
hello-service-provider.ribbon.ServerListRefreshInterval=2000
#请求连接的超时时间
hello-service-provider.ribbon.ConnectTimeout=3000
#请求处理的超时时间
hello-service-provider.ribbon.ReadTimeout=3000

hello-service-provider.ribbon.listOfServers=localhost:8877,localhost:8878

hello-service-provider.ribbon.EnablePrimeConnections=false
 ▪️到目前为止，相关要素已经准备好了，接下来需要feign和ribbon的封装了。我们需要创建OldSystemFeignClientConfiguration类，作用是加载feign-client.properties文件，并创建一个附带负载均衡器的RibbonClient，然后封装出一个附带Jackson编解码器的FeignClient，如下所示：

/**
 * FeignClient创建类
 *
 * @version
 * @author kyle 2017年8月28日下午2:59:49
 * @since 1.8
 */
public class OldSystemFeignClientConfiguration {

    private static void loadProperties() {
        try {
            // 加载配置文件
            ConfigurationManager.loadPropertiesFromResources("feign-client.properties");
        } catch (final IOException e) {
            e.printStackTrace();
        }
    }

        private static IRule zoneAvoidanceRule() {
        return new ZoneAvoidanceRule();
    }

    private static RibbonClient getRibbonClient() {
        loadProperties();
        // 创建附带负载均衡器的RibbonClient
        final RibbonClient client = RibbonClient.builder().lbClientFactory(new LBClientFactory() {
            @Override
            public LBClient create(String clientName) {
                final IClientConfig config = ClientFactory.getNamedConfig(clientName);
                final ILoadBalancer lb = ClientFactory.getNamedLoadBalancer(clientName);
                final ZoneAwareLoadBalancer zb = (ZoneAwareLoadBalancer) lb;
                zb.setRule(zoneAvoidanceRule());
                return LBClient.create(lb, config);
            }
        }).build();
        return client;
    }

    /**
     * @return OldSystemPostFeign
     * @Description: 实现ribbon负载均衡，使用Jackson进行编解码
     * @create date 2018年6月28日下午2:28:56
     */
    public static OldSystemPostFeign remotePostService() {
        // 封装一个使用Jackson编解码器的FeignClient客户端
        final OldSystemPostFeign computeService = Feign.builder().client(getRibbonClient())
                .encoder(new JacksonEncoder()).decoder(new JacksonDecoder())
                .target(OldSystemPostFeign.class, "http://hello-service-provider/");
        return computeService;
    }

    /**
     * @return OldSystemGetFeign
     * @Description: 实现ribbon负载均衡，get请求
     * @create date 2018年6月28日下午3:11:55
     */
    public static OldSystemGetFeign remoteGetService() {
        // 封装一个使用Jackson编解码器的FeignClient客户端
        final OldSystemGetFeign computeService = Feign.builder().client(getRibbonClient())
                .target(OldSystemGetFeign.class, "http://hello-service-provider/");
        return computeService;
    }

}
 ▪️然后我需要一个测试类FeignClientTest，测试以上3个接口，然后将结果输出到控台如下所示：

public class FeignClientTest {
    public static void main(String[] args) {
        OldSystemPostFeign feignPostClient = OldSystemFeignClientConfiguration.remotePostService();
        Person person = new Person();
        person.setName("kyle");
        System.out.println(feignPostClient.postPerson(person).toString());
        OldSystemGetFeign feignGetClient = OldSystemFeignClientConfiguration.remoteGetService();
        System.out.println(feignGetClient.getHost("kyle"));
        System.out.println(feignGetClient.getHost("kyle", "18"));
    }
}
 ▪️在完成上述改造之后，启动测试类FeignClientTest，获得如下的结果，说明调用使用了负载均衡。

15:21:45.595 [main] INFO com.netflix.loadbalancer.DynamicServerListLoadBalancer - DynamicServerListLoadBalancer for client hello-service-provider initialized: DynamicServerListLoadBalancer:{NFLoadBalancer:name=hello-service-provider,current list of Servers=[localhost:8877, localhost:8878],Load balancer stats=Zone stats: {unknown=[Zone:unknown;   Instance count:2;   Active connections count: 0;    Circuit breaker tripped count: 0;   Active connections per server: 0.0;]
},Server stats: [[Server:localhost:8878;    Zone:UNKNOWN;   Total Requests:0;   Successive connection failure:0;    Total blackout seconds:0;   Last connection made:Thu Jan 01 08:00:00 CST 1970;  First connection made: Thu Jan 01 08:00:00 CST 1970;    Active Connections:0;   total failure count in last (1000) msecs:0; average resp time:0.0;  90 percentile resp time:0.0;    95 percentile resp time:0.0;    min resp time:0.0;  max resp time:0.0;  stddev resp time:0.0]
, [Server:localhost:8877;   Zone:UNKNOWN;   Total Requests:0;   Successive connection failure:0;    Total blackout seconds:0;   Last connection made:Thu Jan 01 08:00:00 CST 1970;  First connection made: Thu Jan 01 08:00:00 CST 1970;    Active Connections:0;   total failure count in last (1000) msecs:0; average resp time:0.0;  90 percentile resp time:0.0;    95 percentile resp time:0.0;    min resp time:0.0;  max resp time:0.0;  stddev resp time:0.0]
]}ServerList:com.netflix.loadbalancer.ConfigurationBasedServerList@489115ef
15:21:45.595 [main] INFO com.netflix.client.ClientFactory - Client:hello-service-provider instantiated a LoadBalancer:DynamicServerListLoadBalancer:{NFLoadBalancer:name=hello-service-provider,current list of Servers=[localhost:8877, localhost:8878],Load balancer stats=Zone stats: {unknown=[Zone:unknown;    Instance count:2;   Active connections count: 0;    Circuit breaker tripped count: 0;   Active connections per server: 0.0;]
},Server stats: [[Server:localhost:8878;    Zone:UNKNOWN;   Total Requests:0;   Successive connection failure:0;    Total blackout seconds:0;   Last connection made:Thu Jan 01 08:00:00 CST 1970;  First connection made: Thu Jan 01 08:00:00 CST 1970;    Active Connections:0;   total failure count in last (1000) msecs:0; average resp time:0.0;  90 percentile resp time:0.0;    95 percentile resp time:0.0;    min resp time:0.0;  max resp time:0.0;  stddev resp time:0.0]
, [Server:localhost:8877;   Zone:UNKNOWN;   Total Requests:0;   Successive connection failure:0;    Total blackout seconds:0;   Last connection made:Thu Jan 01 08:00:00 CST 1970;  First connection made: Thu Jan 01 08:00:00 CST 1970;    Active Connections:0;   total failure count in last (1000) msecs:0; average resp time:0.0;  90 percentile resp time:0.0;    95 percentile resp time:0.0;    min resp time:0.0;  max resp time:0.0;  stddev resp time:0.0]
]}ServerList:com.netflix.loadbalancer.ConfigurationBasedServerList@489115ef
15:21:45.598 [main] INFO com.netflix.config.ChainedDynamicProperty - Flipping property: hello-service-provider.ribbon.ActiveConnectionsLimit to use NEXT property: niws.loadbalancer.availabilityFilteringRule.activeConnectionsLimit = 2147483647
15:21:45.639 [main] DEBUG com.netflix.loadbalancer.ZoneAwareLoadBalancer - Zone aware logic disabled or there is only one zone
15:21:45.647 [main] DEBUG com.netflix.loadbalancer.LoadBalancerContext - hello-service-provider using LB returned Server: localhost:8877 for request http:///body/postPerson
Person [name=kyle, age=10, sex=man]
15:21:45.756 [main] INFO com.netflix.config.ChainedDynamicProperty - Flipping property: hello-service-provider.ribbon.ActiveConnectionsLimit to use NEXT property: niws.loadbalancer.availabilityFilteringRule.activeConnectionsLimit = 2147483647
15:21:45.757 [main] DEBUG com.netflix.loadbalancer.ZoneAwareLoadBalancer - Zone aware logic disabled or there is only one zone
15:21:45.757 [main] DEBUG com.netflix.loadbalancer.LoadBalancerContext - hello-service-provider using LB returned Server: localhost:8877 for request http:///demo/getHost?name=kyle
hi, kyle! i from 10.166.37.142:8877
15:21:45.762 [main] INFO com.netflix.config.ChainedDynamicProperty - Flipping property: hello-service-provider.ribbon.ActiveConnectionsLimit to use NEXT property: niws.loadbalancer.availabilityFilteringRule.activeConnectionsLimit = 2147483647
15:21:45.763 [main] DEBUG com.netflix.loadbalancer.ZoneAwareLoadBalancer - Zone aware logic disabled or there is only one zone
15:21:45.763 [main] DEBUG com.netflix.loadbalancer.LoadBalancerContext - hello-service-provider using LB returned Server: localhost:8877 for request http:///head/getHost?name=kyle
hi, kyle, your age is 18! i from 10.166.37.142:8877
15:21:45.770 [Thread-1] INFO com.netflix.loadbalancer.PollingServerListUpdater - Shutting down the Executor Pool for PollingServerListUpdater

 细心的同学会发现，非Spring Boot使用feign调用根本没有使用到注册中心的服务发现。在此我提供一个思路，我们可以调用代理微服务，再由代理进行服务发现。那么这个代理服务应该具备哪些功能和作用呢？我将会在下一篇博客详细讲述Netflix公司的API网关组件zuul，它承担路由转发，拦截过滤，流量控制等功能。

feign使用遇到的一些重要point
▪️第一次请求失败

 原因：由于spring的懒加载机制导致大量的类只有在真正使用的才会真正创建，由于默认的熔断超时时间（1秒）过短，导致第一次请求很容易失败，特别互相依赖复杂的时候。

 解决方法：提升熔断超时时间和ribbon超时时间，配置如下：

#设置hystrix超时时间
hystrix.command.default.execution.isolation.thread.timeoutInMilliseconds=60000
#请求处理的超时时间
ribbon.ReadTimeout=10000
▪️Feign的Http Client

 Feign在默认情况下使用的是JDK原生URLConnection发送HTTP请求，没有连接池，但是对每个地址会保持一个长连接，即利用HTTP的persistence connection。我们可以用Apache的HTTP Client替换Feign原始的http client，从而获取连接池、超时时间等与性能息息相关的控制能力。Spring Cloud从Brixtion.SR5版本开始支持这种替换，首先在项目中声明Apcahe HTTP Client和feign-httpclient依赖,然后在application.properties中添加：

feign.httpclient.enabled=true
▪️如何实现在feign请求之前进行操作

 feign组件提供了请求操作接口RequestInterceptor，实现之后对apply函数进行重写就能对request进行修改，包括header和body操作。

/**
 * 使用自定义的RequestInterceptor，在request发送之前，将信息放入请求
 *
 * @version
 * @author kyle 2017年8月31日上午10:23:01
 * @since 1.8
 */
@Component
public class TokenRequestInterceptor implements RequestInterceptor {
    @Override
    public void apply(RequestTemplate template) {
        String method = template.method();
        String url = template.url();
    }
}
▪️请求压缩
 Spring Cloud Feign支持对请求和响应进行GZIP压缩，以减少通信过程中的性能损耗。我们只需通过下面两个参数设置，就能开启请求与响应的压缩功能：

feign.compression.request.enabled=true
feign.compression.response.enabled=true
 同时，我们还能对请求压缩做一些更细致的设置，比如下面的配置内容指定了压缩的请求数据类型，并设置了压缩的大小下限，只有超过这个大小的请求才会对其进行压缩。

feign.compression.request.enabled=true
feign.compression.request.nime-types=text/xml,application/xml,application/json
feign.compression.requestmin-request-size=2048
 上述配置的feign.compression.request.nime-types和feign.compression.requestmin-request-size均为默认值。

▪️日志配置

 Spring Cloud Feign在构建被@FeignClient注解修饰的服务客户端时，会为每一个客户端都创建一个feign的请求细节。可以在application.properties文件中使用logging.level.<FeignClient>的参数配置格式来开启指定Feign客户端的DEBUG日志，其中<FeignClient>为Feign客户端定义捷克队完整路径，比如针对本博文中我们实现的HelloServiceFeign可以如下配置开启：

logging.level.com.kyle.client.feign.inter.HelloServiceFeign=DEBUG
 但是，只是添加了如上配置，还无法实现对DEBUG日志的输出。这时由于Feign客户端默认对Logger.Level对象定义为NONE级别，该界别不会记录任何Feign调用过程中对信息，所以我们需要调整它对级别，针对全局对日志级别，可以在应用主类中直接假如Logger.Level的Bean创建，具体如下：

/**
 * 服务调用者，，eureka客户端 feign调用
 *
 * @version
 * @author kyle 2017年7月9日下午6:39:15
 * @since 1.8
 */
@EnableEurekaClient
@SpringBootApplication
@EnableFeignClients(basePackages = { "com.kyle.client.feign.inter" })
public class Application {

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }

    @Bean
    public Logger.Level feignLoggerLevel() {
        return Logger.Level.FULL;
    }
}

 在调整日志级别为FULL之后，我们可以再访问第一节的http://localhost:8868/feign/postPerson?name=kyle接口，这是我们在kyle-service-feign的控制台中可以看到类似下面的请求详细的日志：

2018-06-28 16:19:58.393 DEBUG 4140 --- [vice-provider-1] c.k.c.feign.inter.HelloServiceFeign      : [HelloServiceFeign#postPerson] <--- HTTP/1.1 200 OK (302ms)
2018-06-28 16:19:58.393 DEBUG 4140 --- [vice-provider-1] c.k.c.feign.inter.HelloServiceFeign      : [HelloServiceFeign#postPerson] connection: keep-alive
2018-06-28 16:19:58.394 DEBUG 4140 --- [vice-provider-1] c.k.c.feign.inter.HelloServiceFeign      : [HelloServiceFeign#postPerson] content-type: application/json;charset=UTF-8
2018-06-28 16:19:58.394 DEBUG 4140 --- [vice-provider-1] c.k.c.feign.inter.HelloServiceFeign      : [HelloServiceFeign#postPerson] date: Thu, 28 Jun 2018 08:19:58 GMT
2018-06-28 16:19:58.394 DEBUG 4140 --- [vice-provider-1] c.k.c.feign.inter.HelloServiceFeign      : [HelloServiceFeign#postPerson] transfer-encoding: chunked
2018-06-28 16:19:58.394 DEBUG 4140 --- [vice-provider-1] c.k.c.feign.inter.HelloServiceFeign      : [HelloServiceFeign#postPerson] 
2018-06-28 16:19:58.396 DEBUG 4140 --- [vice-provider-1] c.k.c.feign.inter.HelloServiceFeign      : [HelloServiceFeign#postPerson] {"name":"kyle","age":"10","sex":"man"}
2018-06-28 16:19:58.396 DEBUG 4140 --- [vice-provider-1] c.k.c.feign.inter.HelloServiceFeign      : [HelloServiceFeign#postPerson] <--- END HTTP (38-byte body)
 对于Feign的Logger级别主要有下面4类，可根据实际需要进行调整使用。

NONE：不记录任何信息。
BASIC：仅记录请求方法、URL以及响应状态码和执行时间。
HEADERS：出了记录BASIC级别的信息之外，还会记录请求和响应的头信息。
FULL：记录所有请求与响应的细节，包括头信息、请求体、元数据等。
▪️负载均衡异常

 当我们只是对一个微服务进行调用的时候，Ribbon提供的支持好像没什么问题。不过在我们进行多个微服务调用时会产生异常，这也是大多数人忽略的。

 情景描述：2个应用B和C,在A中使用feign client调用B和C；测试结果，假如先调用B，再调用C都是有效的，但是再调用B就是无效的；（B,C先后顺序改变，都会产生这个bug）
 解决方法：在主启动类使用注解@RibbonClient，进行RibbonClient配置，如下所示：

/**
 * 服务调用者，，eureka客户端 feign调用
 *
 * @version
 * @author kyle 2017年7月9日下午6:39:15
 * @since 1.8
 */
@EnableEurekaClient
@SpringBootApplication
@RibbonClient(value = "hello-service-provider")
@EnableFeignClients(basePackages = { "com.kyle.client.feign.inter" })
public class Application {

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }

    @Bean
    public IRule feignRule() {
        return new ZoneAvoidanceRule();
    }
}
RSA加解签
 一位博友提醒我应该补充一下encoder和decoder，让我想到了我公司之前一个需求，保证请求响应过程时body中的数据处于加密状态（互联网金融安全性要求）？我当初是自己下载源码，然后对内部代码进行梳理，最后找到介入点，改造成功。现在我带大家认识一下我是如何接入的，如下是我是梳理的时序图：
feign组件.png
内部类Builder
 看不懂是吗？不要紧，我下面详细讲解一下，先看一下我们之前的非Spring Boot工程中封装FeignClient：

// 封装一个使用Jackson编解码器的FeignClient客户端
final OldSystemPostFeign computeService = Feign.builder().client(getRibbonClient())
        .encoder(new JacksonEncoder()).decoder(new JacksonDecoder())
        .target(OldSystemPostFeign.class, "http://hello-service-provider/");
 我们先来看一下Feign内部类Builder，我们所有可以进行配置的要素都在下图中:
Feign内部类Builder
JDK动态代理
 OldSystemPostFeign只是一个接口，Feign为什么需要使用接口来调用远程接口？原因就是使用JDK动态代理，我们可以去看Feign是如何进行处理。

首先，我们看一下内部类Builder的builder函数：
builder函数
如上图所示，返回都是Feign的子类ReflectiveFeign，我们去看看ReflectiveFeign里面做了什么，很明显使用了JDK动态代理：
eflectiveFeign的newInstance函数
遍历目标接口的所有方法
添加默认实现
创建动态代理处理器
进行代理
如上图所示，我们已经知道Feign使用动态代理，这就是为什么我们只要接口封装远程接口就可以实现调用了，因为Feign给我们都每个调用接口创建了对应的代理类进行请求处理和响应处理。注意上图的第三步，以下是我顺便贴出代码实现逻辑，找出代理类：
//接口InvocationHandlerFactory的create的函数
/**
 * Controls reflective method dispatch.
 */
public interface InvocationHandlerFactory {

  InvocationHandler create(Target target, Map<Method, MethodHandler> dispatch);

  /**
   * Like {@link InvocationHandler#invoke(Object, java.lang.reflect.Method, Object[])}, except for a
   * single method.
   */
  interface MethodHandler {

    Object invoke(Object[] argv) throws Throwable;
  }

  static final class Default implements InvocationHandlerFactory {

    @Override
    public InvocationHandler create(Target target, Map<Method, MethodHandler> dispatch) {
      return new ReflectiveFeign.FeignInvocationHandler(target, dispatch);
    }
  }
}

//其实create函数返回是FeignInvocationHandler，它就是动态代理处理器
static class FeignInvocationHandler implements InvocationHandler {

    private final Target target;
    private final Map<Method, MethodHandler> dispatch;
    ...
    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
    ...
    return dispatch.get(method).invoke(args);
    }
    ...
}
接下来，我们开始去找寻目标接口的每个方法的执行者，我们先要看接口的MethodHandler实现类：
MethodHandler实现类
如上图所示，默认实现类是SynchronousMethodHandler，当你看它的时候你就知道你找对了。
final class SynchronousMethodHandler implements MethodHandler {

  private static final long MAX_RESPONSE_BUFFER_SIZE = 8192L;

  private final MethodMetadata metadata;
  private final Target<?> target;
  private final Client client;
  private final Retryer retryer;
  private final List<RequestInterceptor> requestInterceptors;
  private final Logger logger;
  private final Logger.Level logLevel;
  private final RequestTemplate.Factory buildTemplateFromArgs;
  private final Options options;
  private final Decoder decoder;
  private final ErrorDecoder errorDecoder;
  private final boolean decode404;

  ...

  @Override
  public Object invoke(Object[] argv) throws Throwable {
    RequestTemplate template = buildTemplateFromArgs.create(argv);
    //feign的重试机制
    Retryer retryer = this.retryer.clone();
    while (true) {
      try {
        return executeAndDecode(template);
      } catch (RetryableException e) {
        retryer.continueOrPropagate(e);
        if (logLevel != Logger.Level.NONE) {
          logger.logRetry(metadata.configKey(), logLevel);
        }
        continue;
      }
    }
  }

  Object executeAndDecode(RequestTemplate template) throws Throwable {
    Request request = targetRequest(template);

    if (logLevel != Logger.Level.NONE) {
      logger.logRequest(metadata.configKey(), logLevel, request);
    }

    Response response;
    long start = System.nanoTime();
    try {
        //HttpClient调用，返回response
      response = client.execute(request, options);
    } catch (IOException e) {
      if (logLevel != Logger.Level.NONE) {
        logger.logIOException(metadata.configKey(), logLevel, e, elapsedTime(start));
      }
      throw errorExecuting(request, e);
    }
    //连接超时时间处理
    long elapsedTime = TimeUnit.NANOSECONDS.toMillis(System.nanoTime() - start);

    boolean shouldClose = true;
    try {
      ...
      if (response.status() >= 200 && response.status() < 300) {
        if (void.class == metadata.returnType()) {
          return null;
        } else {
        //响应成功，进行解码
          return decode(response);
        }
      } else if (decode404 && response.status() == 404) {
      //响应失败，进行解码
        return decoder.decode(response, metadata.returnType());
      } else {
      //响应失败，使用异常解码器解码
        throw errorDecoder.decode(metadata.configKey(), response);
      }
    } catch (IOException e) {
      ...
  }
  ...
  Object decode(Response response) throws Throwable {
    try {
    //使用默认解码器解码，如果你设置了解码器，使用设置的进行解码
      return decoder.decode(response, metadata.returnType());
    } catch (FeignException e) {
      throw e;
    } catch (RuntimeException e) {
      throw new DecodeException(e.getMessage(), e);
    }
  }
  
如上源码所示，我们可以清晰的看待Feign调用响应之后对Response进行解码的过程，不过怎么没有看到Request进行编码呢，其实在创建RestTemplate的时候就已经进行编码了，我们来看看ReflectiveFeign的内部类BuildEncodedTemplateFromArgs和BuildFormEncodedTemplateFromArgs：
    private static class BuildFormEncodedTemplateFromArgs extends BuildTemplateByResolvingArgs {

    private final Encoder encoder;

    private BuildFormEncodedTemplateFromArgs(MethodMetadata metadata, Encoder encoder) {
      super(metadata);
      this.encoder = encoder;
    }

    @Override
    protected RequestTemplate resolve(Object[] argv, RequestTemplate mutable,
                                      Map<String, Object> variables) {
      Map<String, Object> formVariables = new LinkedHashMap<String, Object>();
      for (Entry<String, Object> entry : variables.entrySet()) {
        if (metadata.formParams().contains(entry.getKey())) {
          formVariables.put(entry.getKey(), entry.getValue());
        }
      }
      try {
        encoder.encode(formVariables, Encoder.MAP_STRING_WILDCARD, mutable);
      } catch (EncodeException e) {
        throw e;
      } catch (RuntimeException e) {
        throw new EncodeException(e.getMessage(), e);
      }
      return super.resolve(argv, mutable, variables);
    }
  }
 private static class BuildEncodedTemplateFromArgs extends BuildTemplateByResolvingArgs {

    private final Encoder encoder;

    private BuildEncodedTemplateFromArgs(MethodMetadata metadata, Encoder encoder) {
      super(metadata);
      this.encoder = encoder;
    }

    @Override
    protected RequestTemplate resolve(Object[] argv, RequestTemplate mutable,
                                      Map<String, Object> variables) {
      Object body = argv[metadata.bodyIndex()];
      checkArgument(body != null, "Body parameter %s was null", metadata.bodyIndex());
      try {
        encoder.encode(body, metadata.bodyType(), mutable);
      } catch (EncodeException e) {
        throw e;
      } catch (RuntimeException e) {
        throw new EncodeException(e.getMessage(), e);
      }
      return super.resolve(argv, mutable, variables);
    }
  }
源码改造
 至此我们已经熟悉了，Feign整个调用过程以及编码器和解码器的使用，接下来看看我是如何进行RSA加解签。

▪️改造方法

request之前：使用RequestIntercept拦截请求，将body数据进行解密，然后再放入body。
reponse之前：SynchronousMethodHandler的invoke函数执行具体的调用，在响应body进行json转换之前，将body数据进行解密，然后转换成对象返回。
▪️加签

/**
 * Feign请求之前，进行RSA加密
 * 
 * @version
 * @author kyle 2018年5月23日下午1:59:40
 * @since 1.8
 */
public class RSAEncryptRequestInterceptor implements RequestInterceptor {

    @Override
    public void apply(RequestTemplate template) {
        // RSA加密
        if (!getPrivateKeyCache().isEmpty()) {
            String key = getPrivateKeyCache().get(template.url().split("/")[1]);
            if (null != key && !"".equals(key) && "POST".equals(template.method())) {
                byte[] body = template.body();
                try {
                    String bodyContext = new String(body, "UTF-8");
                    String encryptData = RSAUtil.encrypt(key.trim(), bodyContext);
                    template.body(encryptData);
                } catch (UnsupportedEncodingException unsupportedE) {
                    unsupportedE.printStackTrace();
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }
        }

    }

}
▪️解签

 由于Feign并没有提供对Response操作对接口，所以我只能改动源码，切入点是SynchronousMethodHandler的decode函数

Object decode(Response response) throws Throwable {
        Request request = response.request();
        String serviceId = request.url().split("/")[3];
        String publicKey = getPublicKeyCache().get(serviceId);
        try {
            // RSA解密
            if (null != publicKey && !"".equals(publicKey)) {
                byte[] bodyData = Util.toByteArray(response.body().asInputStream());
                String bodyContext = new String(bodyData, "UTF-8");
                String decryptData = RSAUtil.decrypt(publicKey.trim(), bodyContext);
                response = response.toBuilder().body(decryptData.getBytes(UTF_8)).build();
            }
            return decoder.decode(response, metadata.returnType());
        } catch (FeignException e) {
            throw e;
        } catch (RuntimeException e) {
            throw new DecodeException(e.getMessage(), e);
        } finally {
            ensureClosed(response.body());
        }
    }
 至此一个通用的基于Feign加解签的组件就开发完成了，不需要业务开发者再去考虑加解签的事情，让他们可以专注于业务。

《Ribbon详解》
如果需要給我修改意见的发送邮箱：erghjmncq6643981@163.com

本博客的代码示例已上传GitHub：Spring Cloud Netflix组件入门

资料参考：《Spring Cloud 微服务实战》

作者：Chandler_珏瑜
链接：https://www.jianshu.com/p/59295c91dde7
来源：简书
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。
