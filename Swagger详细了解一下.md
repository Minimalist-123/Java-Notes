简介
Swagger 是最流行的 API 开发工具，它遵循 OpenAPI Specification（OpenAPI 规范，也简称 OAS）。
Swagger 可以贯穿于整个 API 生态，如 API 的设计、编写 API 文档、测试和部署。
Swagger 是一种通用的，和编程语言无关的 API 描述规范。

应用场景
如果你的 RESTful API 接口都开发完成了，你可以用 Swagger-editor 来编写 API 文档（ yaml 文件 或 json 文件），然后通过 Swagger-ui 来渲染该文件，以非常美观的形式将你的 API 文档，展现给你的团队或者客户。
如果你的 RESTful API 还未开始，也可以使用 Swagger ，来设计和规范你的 API，以 Annotation （注解）的方式给你的源代码添加额外的数据。这样，Swagger 就可以检测到这些数据，自动生成对应的 API 文档。
规范
Swagger Specification（Swagger 规范），规定了如何对 API 的信息进行正确描述。
Swagger 规范，以前称作 Swagger Specification，现在称作 OpenAPI Specification（简称 OAS）。
Swagger 规范本身是与编程语言无关的，它支持两种语法风格：

YAML 语法
JSON 语法
这两种语法风格可以相互转换，都可以用来对我们的 RESTful API 接口的信息进行准确描述，便于人类和机器阅读。
在 Swagger 中，用于描述 API 信息的文档被称作 Swagger 文档。Swagger 的规范主要有两种：

Swagger 2.0
OpenAPI 3.0
关于 Swagger 规范的详细信息，请参考官方文档

Swagger文档
Swagger 文档（文件），指的是符合 Swagger 规范的文件，用于对 API 的信息进行完整地描述。
Swagger 文档是整个 Swagger 生态的核心。
Swagger 文档的类型有两种：yaml 文件和 json 文件。
yaml 文件用的是 YAML 语法风格；json 文件用的是 JSON 语法风格。这两种文件都可以用来描述 API 的信息，且可以相互转换。
简单的说，Swagger 文档就是 API 文档，只不过 Swagger 文档是用特定的语法来编写的。Swagger 文档本身看起来并不美观，这时，就需要一个好的 UI 工具将其渲染一番，这个工具就是 Swagger-ui。
我们可以用任何编辑器来编写 Swagger 文档，但为了方便在编辑的同时，检测 Swagger 文档是否符合规范，就有了 Swagger-editor 编辑器。


在这里插入图片描述
Swagger工具
Swagger提供了多种工具，帮助解决api的不同的情况下的问题


在这里插入图片描述
Swagger-editor
【功能】

编写 Swagger 文档
实时检测 Swagger 文档是否符合 Swagger 规范
调试 Swagger 文档里描述的 API 接口
转换 Swagger 文档（yaml 转 json，或 json 转 yaml）
【安装】

Web 版本的 Swagger-editor 直接运行在公网上，Swagger 已经给我们配置好了在线的 Swagger-editor。
也可以选择本地运行 Swagger-editor，需要 Node.js 环境支持。
本文使用docker部署，下载swagger-editor的容器

docker pull swaggerapi/swagger-editor
docker run -d -p 81:8080 swaggerapi/swagger-editor 
//启动，81:8080 将容器的8080端口暴露给localhost的81端口
在浏览中输入：localhost:81，就可以在容器中编辑api文档
在这里插入图片描述

【使用说明】：
Swagger-editor 分为菜单栏和主体界面两个部分。
主体界面分为左右两栏，左侧是编辑区，右侧是显示区。
编辑区里默认有一个 Swagger 文档的样例，你可以将其清空，编写自己的 API 描述。
显示区是对应编辑区中的Swagger 文档的 UI 渲染情况，也就是说，右侧显示区的结果和使用 Swagger-ui 渲染 Swagger 文档后的显示结果基本一致。
Swagger-editor 的菜单栏包含以下几个菜单：

File： 用于导入、导出、转换、清空 Swagger 文档
Edit： 用于转换为标准的 YAML 格式文件，比如删除空白行等
Generate Server： 用于构建服务器端 stub
Generate Client： 用于构建客户端 SDK
选择菜单栏【File】Save as YAML，保存为swagger.yaml文件，就是我们所说的swagger文档。

文档编辑参考swagger从入门到精通

Swagger-ui
Swagger-ui 是一套 HTML/CSS/JS 框架，用于渲染 Swagger 文档，以便提供美观的 API 文档界面。也就是说，Swagger-ui 是一个 UI 渲染工具。
【安装】
docker部署，下载swagger-ui的容器

docker pull swaggerapi/swagger-ui
【使用】

使用上面部署的Swagger-editor，在编辑框中完成文档编辑后在页面上上方点击 File -> Download JSON，将文件下载到本地（/Users/jiangsuyao/Downloads）命名为swagger.json
json文件挂在到容器中
//-e：执行容器中/foo/swagger.json
//-v：将/Users/fanfan/Downloads中的swagger.json挂在到 /foo中执行
docker run -p 82:8080 -e SWAGGER_JSON=/foo/swagger.json -v /Users/jiangsuyao/Downloads:/foo swaggerapi/swagger-ui
浏览器输入：localhost:82，即可看到与Swagger-editor的显示区同样的内容
在这里插入图片描述

【基于swagger-ui的接口测试】
1. 选择接口点击【try it out】
在这里插入图片描述

2. 修改“Example Value Model”里面参数，点击“Execute”发送请求
在这里插入图片描述

3. 点击发送后会出现下面视图，不管发送成功／失败。你可以通过下面视图来查看请求数据：
在这里插入图片描述

【springboot集成swagger-ui自动生成API文档】
添加依赖
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
编写配置文件
在application同级目录新建swagger2文件，添加swagger2配置类
package com.abel.example;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import springfox.documentation.builders.ApiInfoBuilder;
import springfox.documentation.builders.PathSelectors;
import springfox.documentation.builders.RequestHandlerSelectors;
import springfox.documentation.service.ApiInfo;
import springfox.documentation.spi.DocumentationType;
import springfox.documentation.spring.web.plugins.Docket;
import springfox.documentation.swagger2.annotations.EnableSwagger2;

@Configuration
@EnableSwagger2
public class Swagger2 {
    /**
     * 创建API应用
     * apiInfo() 增加API相关信息
     * 通过select()函数返回一个ApiSelectorBuilder实例,用来控制哪些接口暴露给Swagger来展现，
     * 本例采用指定扫描的包路径来定义指定要建立API的目录。
     *
     * @return
     */
    @Bean
    public Docket createRestApi() {
        return new Docket(DocumentationType.SWAGGER_2)
                .apiInfo(apiInfo())
                .select()
                .apis(RequestHandlerSelectors.basePackage("com.abel.example.controller"))
                .paths(PathSelectors.any())
                .build();
    }

    /**
     * 创建该API的基本信息（这些基本信息会展现在文档页面中）
     * 访问地址：http://项目实际地址/swagger-ui.html
     * @return
     */
    private ApiInfo apiInfo() {
        return new ApiInfoBuilder()
                .title("Spring Boot中使用Swagger2构建RESTful APIs")
                .description("更多请关注https://blog.csdn.net/u012373815")
                .termsOfServiceUrl("https://blog.csdn.net/u012373815")
                .contact("abel")
                .version("1.0")
                .build();
    }
}
在controller上添加注解，自动生成API
package com.abel.example.controller;

import javax.servlet.http.HttpServletRequest;
import java.util.Map;

import com.abel.example.bean.User;
import io.swagger.annotations.*;
import org.apache.log4j.Logger;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.*;


import com.abel.example.service.UserService;
import com.abel.example.util.CommonUtil;


@Controller
@RequestMapping(value = "/users")
@Api(value = "用户的增删改查")
public class UserController {

    @Autowired
    private UserService userService;


    /**
     * 查询所有的用户
     * api :localhost:8099/users
     * @return
     */
    @RequestMapping(method = RequestMethod.GET)
    @ResponseBody
    @ApiOperation(value = "获取用户列表，目前没有分页")
    public ResponseEntity<Object> findAll() {
        return new ResponseEntity<>(userService.listUsers(), HttpStatus.OK);
    }

    /**
     * 通过id 查找用户
     * api :localhost:8099/users/1
     * @param id
     * @return
     */
    @RequestMapping(value = "/{id}", method = RequestMethod.GET)
    @ResponseBody
    @ApiOperation(value = "通过id获取用户信息", notes="返回用户信息")
    public ResponseEntity<Object> getUserById(@PathVariable Integer id) {
        return new ResponseEntity<>(userService.getUserById(Long.valueOf(id)), HttpStatus.OK);
    }


    /**
     * 通过spring data jpa 调用方法
     * api :localhost:8099/users/byname?username=xxx
     * 通过用户名查找用户
     * @param request
     * @return
     */
    @RequestMapping(value = "/byname", method = RequestMethod.GET)
    @ResponseBody
    @ApiImplicitParam(paramType = "query",name= "username" ,value = "用户名",dataType = "string")
    @ApiOperation(value = "通过用户名获取用户信息", notes="返回用户信息")
    public ResponseEntity<Object> getUserByUserName(HttpServletRequest request) {
        Map<String, Object> map = CommonUtil.getParameterMap(request);
        String username = (String) map.get("username");
        return new ResponseEntity<>(userService.getUserByUserName(username), HttpStatus.OK);
    }

    /**
     * 通过spring data jpa 调用方法
     * api :localhost:8099/users/byUserNameContain?username=xxx
     * 通过用户名模糊查询
     * @param request
     * @return
     */
    @RequestMapping(value = "/byUserNameContain", method = RequestMethod.GET)
    @ResponseBody
    @ApiImplicitParam(paramType = "query",name= "username" ,value = "用户名",dataType = "string")
    @ApiOperation(value = "通过用户名模糊搜索用户信息", notes="返回用户信息")
    public ResponseEntity<Object> getUsers(HttpServletRequest request) {
        Map<String, Object> map = CommonUtil.getParameterMap(request);
        String username = (String) map.get("username");
        return new ResponseEntity<>(userService.getByUsernameContaining(username), HttpStatus.OK);
    }


    /**
     * 添加用户啊
     * api :localhost:8099/users
     *
     * @param user
     * @return
     */
    @RequestMapping(method = RequestMethod.POST)
    @ResponseBody
    @ApiModelProperty(value="user",notes = "用户信息的json串")
    @ApiOperation(value = "新增用户", notes="返回新增的用户信息")
    public ResponseEntity<Object> saveUser(@RequestBody User user) {
        return new ResponseEntity<>(userService.saveUser(user), HttpStatus.OK);
    }

    /**
     * 修改用户信息
     * api :localhost:8099/users
     * @param user
     * @return
     */
    @RequestMapping(method = RequestMethod.PUT)
    @ResponseBody
    @ApiModelProperty(value="user",notes = "修改后用户信息的json串")
    @ApiOperation(value = "新增用户", notes="返回新增的用户信息")
    public ResponseEntity<Object> updateUser(@RequestBody User user) {
        return new ResponseEntity<>(userService.updateUser(user), HttpStatus.OK);
    }

    /**
     * 通过ID删除用户
     * api :localhost:8099/users/2
     * @param id
     * @return
     */
    @RequestMapping(value = "/{id}", method = RequestMethod.DELETE)
    @ResponseBody
    @ApiOperation(value = "通过id删除用户信息", notes="返回删除状态1 成功 0 失败")
    public ResponseEntity<Object> deleteUser(@PathVariable Integer id) {
        return new ResponseEntity<>(userService.removeUser(id.longValue()), HttpStatus.OK);
    }
}
注解说明

@Api：用在类上，说明该类的作用。
@ApiOperation：注解来给API增加方法说明。
@ApiImplicitParams : 用在方法上包含一组参数说明。
@ApiImplicitParam：用来注解来给方法入参增加说明。
@ApiResponses：用于表示一组响应
@ApiResponse：用在@ApiResponses中，一般用于表达一个错误的响应信息
@ApiModel：描述一个Model的信息（一般用在请求参数无法使用@ApiImplicitParam注解进行描述的时候）
@ApiModelProperty：描述一个model的属性
其中
@ApiResponse参数：

code：数字，如400
message：信息，如“参数填写错误”
response：抛出异常的类
@ApiImplicitParam参数：
paramTpye：指定参数放在哪些地方（header/query/path/body/form）
name：参数名
dataTpye：参数类型
required：是否必输（true/false）
value：说明参数的意思
defaultValue：参数默认值
下载Swagger UI组件
去官网下载Zip包，或者在github上下载也可以，需要将dist文件夹下的所有文件的复制到webapp目录下
原理就是在系统加载的时候，Swagger配置类去扫描所有添加注释的接口，并且储存起来通过下面地址进行访问，返回JSON数据，在前端界面显示出来。
启动项目后，访问http://localhost:8099/swagger-ui.html，显示如下：
在这里插入图片描述
Swagger-Codegen
Swagger Codegen是一个开源的代码生成器，根据Swagger定义的RESTful API可以自动建立服务端和客户端的连接。Swagger Codegen的源码可以在Github上找到。
GitHub:https://github.com/swagger-api/swagger-codegen
【安装】
首先机器上需要有jdk，然后只要下载一个cli的文件就可以了

//下载
# wget https://oss.sonatype.org/content/repositories/releases/io/swagger/swagger-codegen-cli/2.2.1/swagger-codegen-cli-2.2.1.jar
//下载之后运行，返回结果可查看其支持的语言
# java -jar swagger-codegen-cli-2.2.1.jar
Available languages: [android, aspnet5, async-scala, cwiki, csharp, cpprest, dart, flash, python-flask, go, groovy, java, jaxrs, jaxrs-cxf, jaxrs-resteasy, jaxrs-spec, inflector, javascript, javascript-closure-angular, jmeter, nancyfx, nodejs-server, objc, perl, php, python, qt5cpp, ruby, scala, scalatra, silex-PHP, sinatra, rails5, slim, spring, dynamic-html, html, html2, swagger, swagger-yaml, swift, tizen, typescript-angular2, typescript-angular, typescript-node, typescript-fetch, akka-scala, CsharpDotNet2, clojure, haskell, lumen, go-server]
//查看支持某个语言的具体使用帮助，比如java
# java -jar swagger-codegen-cli-2.2.1.jar config-help -l java
【使用】
利用swagger-codegen根据服务生成客户端代码

//http://petstore.swagger.io/v2/swagger.json是官方的一个例子，我们可以改成自己的服务
# java -jar swagger-codegen-cli-2.2.1.jar generate -i http://petstore.swagger.io/v2/swagger.json -l java -o samples/client/pestore/java
在上面这段代码里，使用了三个参数，分别是-i和-l和-o。

-i，指定swagger描述文件的路径,url地址或路径文件;该参数为必须

-l，指定生成客户端代码的语言,该参数为必须

-o，指定生成文件的位置(默认当前目录)

除了可以指定上面三个参数，还有一些常用的：

-c ，json格式的配置文件的路径;文件为json格式,支持的配置项因语言的不同而不同

-a， 当获取远程swagger定义时,添加授权头信息;URL-encoded格式化的name,逗号隔开的多个值

--api-package， 指定生成的api类的包名

--artifact-id ，指定pom.xml的artifactId的值

--artifact-version ，指定pom.xml的artifact的版本

--group-id， 指定pom.xml的groupId的值

--model-package， 指定生成的model类的包名

-s ，指定该参数表示不覆盖已经存在的文件

-t ，指定模版文件所在目录
生成好的客户端代码：


在这里插入图片描述
参考文档：
https://blog.csdn.net/u012373815/article/details/82685962
https://www.cnblogs.com/shamo89/p/7680771.html


作者：LittleJessy
链接：https://www.jianshu.com/p/4fdac2a10c79
来源：简书
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。
