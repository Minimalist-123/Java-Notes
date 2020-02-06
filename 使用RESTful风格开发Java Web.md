什么是RESTful风格？
REST是REpresentational State Transfer的缩写（一般中文翻译为表述性状态转移），REST 是一种体系结构，而 HTTP 是一种包含了 REST 架构属性的协议，为了便于理解，我们把它的首字母拆分成不同的几个部分：

表述性（REpresentational）： REST 资源实际上可以用各种形式来进行表述，包括 XML、JSON 甚至 HTML——最适合资源使用者的任意形式；
状态（State）： 当使用 REST 的时候，我们更关注资源的状态而不是对资源采取的行为；
转义（Transfer）： REST 涉及到转移资源数据，它以某种表述性形式从一个应用转移到另一个应用。
简单地说，REST 就是将资源的状态以适合客户端或服务端的形式从服务端转移到客户端（或者反过来）。在 REST 中，资源通过 URL 进行识别和定位，然后通过行为(即 HTTP 方法)来定义 REST 来完成怎样的功能。

实例说明:
在平时的 Web 开发中，method 常用的值是 GET 和 POST，但是实际上，HTTP 方法还有 PATCH、DELETE、PUT 等其他值，这些方法又通常会匹配为如下的 CRUD 动作:

CRUD 动作	HTTP 方法
Create	POST
Read	GET
Update	PUT 或 PATCH
Delete	DELETE
尽管通常来讲，HTTP 方法会映射为 CRUD 动作，但这并不是严格的限制，有时候 PUT 也可以用来创建新的资源，POST 也可以用来更新资源。实际上，POST 请求非幂等的特性(即同一个 URL 可以得到不同的结果)使其成一个非常灵活地方法，对于无法适应其他 HTTP 方法语义的操作，它都能够胜任。

在使用 RESTful 风格之前，我们如果想要增加一条商品数据通常是这样的:

/addCategory?name=xxx
但是使用了 RESTful 风格之后就会变成:

/category
这就变成了使用同一个 URL ，通过约定不同的 HTTP 方法来实施不同的业务，这就是 RESTful 风格所做的事情了，为了有一个更加直观的理解，引用一下来自how2j.cn的图:


SpringBoot 中使用 RESTful
下面我使用 SpringBoot 结合文章：http://blog.didispace.com/springbootrestfulapi/ 来实例演示如何在 SpringBoot 中使用 RESTful 风格的编程并如何做单元测试

RESTful API 具体设计如下：


User实体定义：

public class User { 
 
    private Long id; 
    private String name; 
    private Integer age; 
 
    // 省略setter和getter 
     
}
实现对User对象的操作接口

@RestController 
@RequestMapping(value="/users")     // 通过这里配置使下面的映射都在/users下 
public class UserController { 
 
    // 创建线程安全的Map 
    static Map<Long, User> users = Collections.synchronizedMap(new HashMap<Long, User>()); 
 
    @RequestMapping(value="/", method=RequestMethod.GET) 
    public List<User> getUserList() { 
        // 处理"/users/"的GET请求，用来获取用户列表 
        // 还可以通过@RequestParam从页面中传递参数来进行查询条件或者翻页信息的传递 
        List<User> r = new ArrayList<User>(users.values()); 
        return r; 
    } 
 
    @RequestMapping(value="/", method=RequestMethod.POST) 
    public String postUser(@ModelAttribute User user) { 
        // 处理"/users/"的POST请求，用来创建User 
        // 除了@ModelAttribute绑定参数之外，还可以通过@RequestParam从页面中传递参数 
        users.put(user.getId(), user); 
        return "success"; 
    } 
 
    @RequestMapping(value="/{id}", method=RequestMethod.GET) 
    public User getUser(@PathVariable Long id) { 
        // 处理"/users/{id}"的GET请求，用来获取url中id值的User信息 
        // url中的id可通过@PathVariable绑定到函数的参数中 
        return users.get(id); 
    } 
 
    @RequestMapping(value="/{id}", method=RequestMethod.PUT) 
    public String putUser(@PathVariable Long id, @ModelAttribute User user) { 
        // 处理"/users/{id}"的PUT请求，用来更新User信息 
        User u = users.get(id); 
        u.setName(user.getName()); 
        u.setAge(user.getAge()); 
        users.put(id, u); 
        return "success"; 
    } 
 
    @RequestMapping(value="/{id}", method=RequestMethod.DELETE) 
    public String deleteUser(@PathVariable Long id) { 
        // 处理"/users/{id}"的DELETE请求，用来删除User 
        users.remove(id); 
        return "success"; 
    } 
 
}
编写测试单元
参考文章：http://tengj.top/2017/12/28/springboot12/#Controller单元测试
看过这几篇文章之后觉得好棒，还有这么方便的测试方法，这些以前都没有接触过...

下面针对该Controller编写测试用例验证正确性，具体如下。当然也可以通过浏览器插件等进行请求提交验证，因为涉及一些包的导入，这里给出全部代码：

package cn.wmyskxz.springboot;

import cn.wmyskxz.springboot.controller.UserController;
import org.junit.Before;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.mock.web.MockServletContext;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;
import org.springframework.test.context.web.WebAppConfiguration;
import org.springframework.test.web.servlet.MockMvc;
import org.springframework.test.web.servlet.RequestBuilder;
import org.springframework.test.web.servlet.setup.MockMvcBuilders;

import static org.hamcrest.Matchers.equalTo;
import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.*;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.content;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.status;


/**
 * @author: @我没有三颗心脏
 * @create: 2018-05-29-上午 8:39
 */
@RunWith(SpringJUnit4ClassRunner.class)
@SpringBootTest(classes = MockServletContext.class)
@WebAppConfiguration
public class ApplicationTests {

    private MockMvc mvc;

    @Before
    public void setUp() throws Exception {
        mvc = MockMvcBuilders.standaloneSetup(new UserController()).build();
    }

    @Test
    public void testUserController() throws Exception {
        // 测试UserController
        RequestBuilder request = null;

        // 1、get查一下user列表，应该为空
        request = get("/users/");
        mvc.perform(request)
                .andExpect(status().isOk())
                .andExpect(content().string(equalTo("[]")));

        // 2、post提交一个user
        request = post("/users/")
                .param("id", "1")
                .param("name", "测试大师")
                .param("age", "20");
        mvc.perform(request)
                .andExpect(content().string(equalTo("success")));

        // 3、get获取user列表，应该有刚才插入的数据
        request = get("/users/");
        mvc.perform(request)
                .andExpect(status().isOk())
                .andExpect(content().string(equalTo("[{\"id\":1,\"name\":\"测试大师\",\"age\":20}]")));

        // 4、put修改id为1的user
        request = put("/users/1")
                .param("name", "测试终极大师")
                .param("age", "30");
        mvc.perform(request)
                .andExpect(content().string(equalTo("success")));

        // 5、get一个id为1的user
        request = get("/users/1");
        mvc.perform(request)
                .andExpect(content().string(equalTo("{\"id\":1,\"name\":\"测试终极大师\",\"age\":30}")));

        // 6、del删除id为1的user
        request = delete("/users/1");
        mvc.perform(request)
                .andExpect(content().string(equalTo("success")));

        // 7、get查一下user列表，应该为空
        request = get("/users/");
        mvc.perform(request)
                .andExpect(status().isOk())
                .andExpect(content().string(equalTo("[]")));

    }

}
MockMvc实现了对HTTP请求的模拟，从示例的代码就能够看出MockMvc的简单用法，它能够直接使用网络的形式，转换到Controller的调用，这样使得测试速度快、不依赖网络环境，而且提供了一套验证的工具，这样可以使得请求的验证统一而且很方便。

需要注意的就是在MockMvc使用之前需要先用MockMvcBuilders构建MockMvc对象，如果对单元测试感兴趣的童鞋请戳上面的链接哦，这里就不细说了

测试信息
运行测试类，控制台返回的信息如下：

 __      __                               __
/\ \  __/\ \                             /\ \
\ \ \/\ \ \ \    ___ ___   __  __    ____\ \ \/'\    __  _  ____
 \ \ \ \ \ \ \ /' __` __`\/\ \/\ \  /',__\\ \ , <   /\ \/'\/\_ ,`\
  \ \ \_/ \_\ \/\ \/\ \/\ \ \ \_\ \/\__, `\\ \ \ \`\\/>  </\/_/  /_
   \ `\___x___/\ \_\ \_\ \_\/`____ \/\____/ \ \_\ \_\/\_/\_\ /\____\
    '\/__//__/  \/_/\/_/\/_/`/___/> \/___/   \/_/\/_/\//\/_/ \/____/
                               /\___/
                               \/__/
2018-05-29 09:28:18.730  INFO 5884 --- [           main] cn.wmyskxz.springboot.ApplicationTests   : Starting ApplicationTests on SC-201803262103 with PID 5884 (started by Administrator in E:\Java Projects\springboot)
2018-05-29 09:28:18.735  INFO 5884 --- [           main] cn.wmyskxz.springboot.ApplicationTests   : No active profile set, falling back to default profiles: default
2018-05-29 09:28:18.831  INFO 5884 --- [           main] o.s.w.c.s.GenericWebApplicationContext   : Refreshing org.springframework.web.context.support.GenericWebApplicationContext@7c37508a: startup date [Tue May 29 09:28:18 CST 2018]; root of context hierarchy
2018-05-29 09:28:19.200  INFO 5884 --- [           main] cn.wmyskxz.springboot.ApplicationTests   : Started ApplicationTests in 1.184 seconds (JVM running for 2.413)
2018-05-29 09:28:19.798  INFO 5884 --- [           main] s.w.s.m.m.a.RequestMappingHandlerMapping : Mapped "{[/users/{id}],methods=[PUT]}" onto public java.lang.String cn.wmyskxz.springboot.controller.UserController.putUser(java.lang.Long,cn.wmyskxz.springboot.pojo.User)
2018-05-29 09:28:19.800  INFO 5884 --- [           main] s.w.s.m.m.a.RequestMappingHandlerMapping : Mapped "{[/users/],methods=[GET]}" onto public java.util.List<cn.wmyskxz.springboot.pojo.User> cn.wmyskxz.springboot.controller.UserController.getUserList()
2018-05-29 09:28:19.800  INFO 5884 --- [           main] s.w.s.m.m.a.RequestMappingHandlerMapping : Mapped "{[/users/],methods=[POST]}" onto public java.lang.String cn.wmyskxz.springboot.controller.UserController.postUser(cn.wmyskxz.springboot.pojo.User)
2018-05-29 09:28:19.801  INFO 5884 --- [           main] s.w.s.m.m.a.RequestMappingHandlerMapping : Mapped "{[/users/{id}],methods=[DELETE]}" onto public java.lang.String cn.wmyskxz.springboot.controller.UserController.deleteUser(java.lang.Long)
2018-05-29 09:28:19.801  INFO 5884 --- [           main] s.w.s.m.m.a.RequestMappingHandlerMapping : Mapped "{[/users/{id}],methods=[GET]}" onto public cn.wmyskxz.springboot.pojo.User cn.wmyskxz.springboot.controller.UserController.getUser(java.lang.Long)
2018-05-29 09:28:19.850  INFO 5884 --- [           main] s.w.s.m.m.a.RequestMappingHandlerAdapter : Looking for @ControllerAdvice: org.springframework.test.web.servlet.setup.StubWebApplicationContext@42f8285e
2018-05-29 09:28:19.924  INFO 5884 --- [           main] o.s.mock.web.MockServletContext          : Initializing Spring FrameworkServlet ''
2018-05-29 09:28:19.925  INFO 5884 --- [           main] o.s.t.web.servlet.TestDispatcherServlet  : FrameworkServlet '': initialization started
2018-05-29 09:28:19.926  INFO 5884 --- [           main] o.s.t.web.servlet.TestDispatcherServlet  : FrameworkServlet '': initialization completed in 1 ms
通过控制台信息，我们得知通过 RESTful 风格能成功调用到正确的方法并且能获取到或者返回正确的参数，没有任何错误，则说明成功！

如果你想要看到更多的细节信息，可以在每次调用 perform() 方法后再跟上一句 .andDo(MockMvcResultHandlers.print()) ，例如：

        // 1、get查一下user列表，应该为空
        request = get("/users/");
        mvc.perform(request)
                .andExpect(status().isOk())
                .andExpect(content().string(equalTo("[]")))
                .andDo(MockMvcResultHandlers.print());
就能看到详细的信息，就像下面这样：

MockHttpServletRequest:
      HTTP Method = GET
      Request URI = /users/
       Parameters = {}
          Headers = {}
             Body = <no character encoding set>
    Session Attrs = {}

Handler:
             Type = cn.wmyskxz.springboot.controller.UserController
           Method = public java.util.List<cn.wmyskxz.springboot.pojo.User> cn.wmyskxz.springboot.controller.UserController.getUserList()

Async:
    Async started = false
     Async result = null

Resolved Exception:
             Type = null

ModelAndView:
        View name = null
             View = null
            Model = null

FlashMap:
       Attributes = null

MockHttpServletResponse:
           Status = 200
    Error message = null
          Headers = {Content-Type=[application/json;charset=UTF-8]}
     Content type = application/json;charset=UTF-8
             Body = []
    Forwarded URL = null
   Redirected URL = null
          Cookies = []
总结
我们仍然使用 @RequestMapping 注解，但不同的是，我们指定 method 属性来处理不同的 HTTP 方法，并且通过 @PathVariable 注解来将 HTTP 请求中的属性绑定到我们指定的形参上。

事实上，Spring 4.3 之后，为了更好的支持 RESTful 风格，增加了几个注解：@PutMapping、@GetMapping、@DeleteMapping、@PostMapping，从名字也能大概的看出，其实也就是将 method 属性的值与 @RequestMapping 进行了绑定而已，例如，我们对UserController中的deleteUser方法进行改造：

-----------改造前-----------
@RequestMapping(value="/{id}", method=RequestMethod.DELETE)
public String deleteUser(@PathVariable Long id) {
    // 处理"/users/{id}"的DELETE请求，用来删除User
    users.remove(id);
    return "success";
}

-----------改造后-----------
@DeleteMapping("/{id}")
public String deleteUser(@PathVariable Long id) {
    // 处理"/users/{id}"的DELETE请求，用来删除User
    users.remove(id);
    return "success";
}
使用Swagger2构造RESTful API文档
参考文章：http://blog.didispace.com/springbootswagger2/

RESTful 风格为后台与前台的交互提供了简洁的接口API，并且有利于减少与其他团队的沟通成本，通常情况下，我们会创建一份RESTful API文档来记录所有的接口细节，但是这样做有以下的几个问题：

由于接口众多，并且细节复杂（需要考虑不同的HTTP请求类型、HTTP头部信息、HTTP请求内容等），高质量地创建这份文档本身就是件非常吃力的事，下游的抱怨声不绝于耳。
随着时间推移，不断修改接口实现的时候都必须同步修改接口文档，而文档与代码又处于两个不同的媒介，除非有严格的管理机制，不然很容易导致不一致现象。
Swagger2的出现就是为了解决上述的这些问题，并且能够轻松的整合到我们的SpringBoot中去，它既可以减少我们创建文档的工作量，同时说明内容又可以整合到代码之中去，让维护文档和修改代码整合为一体，可以让我们在修改代码逻辑的同时方便的修改文档说明，这太酷了，另外Swagger2页提供了强大的页面测试功能来调试每个RESTful API，具体效果如下：


让我们赶紧来看看吧：

第一步：添加Swagger2依赖：
在 pom.xml 中加入Swagger2的依赖：

<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger2</artifactId>
    <version>2.2.2</version>
</dependency>
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger-ui</artifactId>
    <version>2.2.2</version>
</dependency>
第二步：创建Swagger2配置类
在SpringBoot启动类的同级目录下创建Swagger2的配置类 Swagger2：

@Configuration
@EnableSwagger2
public class Swagger2 {

    @Bean
    public Docket createRestApi() {
        return new Docket(DocumentationType.SWAGGER_2)
                .apiInfo(apiInfo())
                .select()
                .apis(RequestHandlerSelectors.basePackage("cn.wmyskxz.springboot"))
                .paths(PathSelectors.any())
                .build();
    }

    private ApiInfo apiInfo() {
        return new ApiInfoBuilder()
                .title("Spring Boot中使用Swagger2构建RESTful APIs")
                .description("原文地址链接：http://blog.didispace.com/springbootswagger2/")
                .termsOfServiceUrl("http://blog.didispace.com/")
                .contact("@我没有三颗心脏")
                .version("1.0")
                .build();
    }

}
如上面的代码所示，通过 @Configuration 注解让Spring来加载该配置类，再通过 @EnableSwagger2 注解来启动Swagger2；

再通过 createRestApi 函数创建 Docket 的Bean之后，apiInfo() 用来创建该API的基本信息（这些基本信息会展现在文档页面中），select() 函数返回一个 ApiSelectorBuilder 实例用来控制哪些接口暴露给Swagger来展现，本例采用指定扫描的包路径来定义，Swagger会扫描该包下所有的Controller定义的API，并产生文档内容（除了被 @ApiIgnore 指定的请求）

第三步：添加文档内容
在完成了上述配置后，其实已经可以生产文档内容，但是这样的文档主要针对请求本身，而描述主要来源于函数等命名产生，对用户并不友好，我们通常需要自己增加一些说明来丰富文档内容。如下所示，我们通过@ApiOperation注解来给API增加说明、通过@ApiImplicitParams、@ApiImplicitParam注解来给参数增加说明。

@RestController
@RequestMapping(value="/users")     // 通过这里配置使下面的映射都在/users下，可去除
public class UserController {

    static Map<Long, User> users = Collections.synchronizedMap(new HashMap<Long, User>());

    @ApiOperation(value="获取用户列表", notes="")
    @RequestMapping(value={""}, method=RequestMethod.GET)
    public List<User> getUserList() {
        List<User> r = new ArrayList<User>(users.values());
        return r;
    }

    @ApiOperation(value="创建用户", notes="根据User对象创建用户")
    @ApiImplicitParam(name = "user", value = "用户详细实体user", required = true, dataType = "User")
    @RequestMapping(value="", method=RequestMethod.POST)
    public String postUser(@RequestBody User user) {
        users.put(user.getId(), user);
        return "success";
    }

    @ApiOperation(value="获取用户详细信息", notes="根据url的id来获取用户详细信息")
    @ApiImplicitParam(name = "id", value = "用户ID", required = true, dataType = "Long")
    @RequestMapping(value="/{id}", method=RequestMethod.GET)
    public User getUser(@PathVariable Long id) {
        return users.get(id);
    }

    @ApiOperation(value="更新用户详细信息", notes="根据url的id来指定更新对象，并根据传过来的user信息来更新用户详细信息")
    @ApiImplicitParams({
            @ApiImplicitParam(name = "id", value = "用户ID", required = true, dataType = "Long"),
            @ApiImplicitParam(name = "user", value = "用户详细实体user", required = true, dataType = "User")
    })
    @RequestMapping(value="/{id}", method=RequestMethod.PUT)
    public String putUser(@PathVariable Long id, @RequestBody User user) {
        User u = users.get(id);
        u.setName(user.getName());
        u.setAge(user.getAge());
        users.put(id, u);
        return "success";
    }

    @ApiOperation(value="删除用户", notes="根据url的id来指定删除对象")
    @ApiImplicitParam(name = "id", value = "用户ID", required = true, dataType = "Long")
    @RequestMapping(value="/{id}", method=RequestMethod.DELETE)
    public String deleteUser(@PathVariable Long id) {
        users.remove(id);
        return "success";
    }

}
完成上述代码添加之后，启动Spring Boot程序，访问：http://localhost:8080/swagger-ui.html，就能看到前文展示的RESTful API的页面，我们可以点开具体的API请求，POST类型的/users请求为例，可找到上述代码中我们配置的Notes信息以及参数user的描述信息，如下图所示：


API文档访问与调试
在上图请求的页面中，我们可以看到一个Value的输入框，并且在右边的Model Schema中有示例的User对象模板，我们点击右边黄色的区域Value框中就会自动填好示例的模板数据，我们可以稍微修改修改，然后点击下方的 “Try it out!” 按钮，即可完成一次请求调用，这太酷了。


总结
对比之前用文档来记录RESTful API的方式，我们通过增加少量的配置内容，在原有代码的基础上侵入了忍受范围内的代码，就可以达到如此方便、直观的效果，可以说是使用Swagger2来对API文档进行管理，是个很不错的选择！

欢迎转载，转载请注明出处！
简书ID：@我没有三颗心脏
github：wmyskxz
欢迎关注公众微信号：wmyskxz_javaweb
分享自己的Java Web学习之路以及各种Java学习资料



作者：我没有三颗心脏
链接：https://www.jianshu.com/p/91600da4df95
来源：简书
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。
