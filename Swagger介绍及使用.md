导语：
相信无论是前端还是后端开发，都或多或少地被接口文档折磨过。前端经常抱怨后端给的接口文档与实际情况不一致。后端又觉得编写及维护接口文档会耗费不少精力，经常来不及更新。其实无论是前端调用后端，还是后端调用后端，都期望有一个好的接口文档。但是这个接口文档对于程序员来说，就跟注释一样，经常会抱怨别人写的代码没有写注释，然而自己写起代码起来，最讨厌的，也是写注释。所以仅仅只通过强制来规范大家是不够的，随着时间推移，版本迭代，接口文档往往很容易就跟不上代码了。

Swagger是什么？它能干什么？
image.png
发现了痛点就要去找解决方案。解决方案用的人多了，就成了标准的规范，这就是Swagger的由来。通过这套规范，你只需要按照它的规范去定义接口及接口相关的信息。再通过Swagger衍生出来的一系列项目和工具，就可以做到生成各种格式的接口文档，生成多种语言的客户端和服务端的代码，以及在线接口调试页面等等。这样，如果按照新的开发模式，在开发新版本或者迭代版本的时候，只需要更新Swagger描述文件，就可以自动生成接口文档和客户端服务端代码，做到调用端代码、服务端代码以及接口文档的一致性。

但即便如此，对于许多开发来说，编写这个yml或json格式的描述文件，本身也是有一定负担的工作，特别是在后面持续迭代开发的时候，往往会忽略更新这个描述文件，直接更改代码。久而久之，这个描述文件也和实际项目渐行渐远，基于该描述文件生成的接口文档也失去了参考意义。所以作为Java届服务端的大一统框架Spring，迅速将Swagger规范纳入自身的标准，建立了Spring-swagger项目，后面改成了现在的Springfox。通过在项目中引入Springfox，可以扫描相关的代码，生成该描述文件，进而生成与代码一致的接口文档和客户端代码。这种通过代码生成接口文档的形式，在后面需求持续迭代的项目中，显得尤为重要和高效。

框架说明及使用
1.说明
现在SWAGGER官网主要提供了几种开源工具，提供相应的功能。可以通过配置甚至是修改源码以达到你想要的效果。

image.png
Swagger Codegen: 通过Codegen 可以将描述文件生成html格式和cwiki形式的接口文档，同时也能生成多钟语言的服务端和客户端的代码。支持通过jar包，docker，node等方式在本地化执行生成。也可以在后面的Swagger Editor中在线生成。

Swagger UI:提供了一个可视化的UI页面展示描述文件。接口的调用方、测试、项目经理等都可以在该页面中对相关接口进行查阅和做一些简单的接口请求。该项目支持在线导入描述文件和本地部署UI项目。

Swagger Editor: 类似于markendown编辑器的编辑Swagger描述文件的编辑器，该编辑支持实时预览描述文件的更新效果。也提供了在线编辑器和本地部署编辑器两种方式。

Swagger Inspector: 感觉和postman差不多，是一个可以对接口进行测试的在线版的postman。比在Swagger UI里面做接口请求，会返回更多的信息，也会保存你请求的实际请求参数等数据。

Swagger Hub：集成了上面所有项目的各个功能，你可以以项目和版本为单位，将你的描述文件上传到Swagger Hub中。在Swagger Hub中可以完成上面项目的所有工作，需要注册账号，分免费版和收费版。

PS：

Springfox Swagger: Spring 基于swagger规范，可以将基于SpringMVC和Spring Boot项目的项目代码，自动生成JSON格式的描述文件。本身不是属于Swagger官网提供的，在这里列出来做个说明，方便后面作一个使用的展开。

2.基于Spring框架的Swagger流程应用
这里不会介绍Swagger的工具具体如何使用，不会讲yml或者json格式描述文件的语法规范，也不会讲如何在SpringMVC或者Spring Boot中配置Springfox－swagger。这些都能从网上找到，而且配置起来都非常的简单。

这里想讲的是如何结合现有的工具和功能，设计一个流程，去保证一个项目从开始开发到后面持续迭代的时候，以最小代价去维护代码、接口文档以及Swagger描述文件。

2.1 项目开始阶段
一般来说，接口文档都是由服务端来编写的。在项目开发阶段的时候，服务端开发可以视情况来决定是直接编写服务端调用层代码，还是写Swagger描述文件。建议是如果项目启动阶段，就已经搭好了后台框架，那可以直接编写服务端被调用层的代码（即controller及其入参出参对象），然后通过Springfox－swagger 生成swagger json描述文件。如果项目启动阶段并没有相关后台框架，而前端对接口文档追得紧，那就建议先编写swagger描述文件，通过该描述文件生成接口文档。后续后台框架搭好了，也可以生成相关的服务端代码。

2.1 项目迭代阶段
到这个阶段，事情就简单很多了。后续后台人员，无需关注Swagger描述文件和接口文档，有需求变更导致接口变化，直接写代码就好了。把调用层的代码做个修改，然后生成新的描述文件和接口文档后，给到前端即可。真正做到了一劳永逸。

2.3流程
总结一下就是通过下面这两种流程中的一种，可以做到代码和接口文档的一致性，服务端开发再也不用花费精力去维护接口文档。

流程一
image.png
流程二
image.png
给Mock系统的正常请求及响应全流程数据
很多时候，如果你能在提供接口文档的同时，把所有接口的模拟请求响应数据也提供给前端。或者有Mock系统，直接将这些模拟数据录入到Mock系统中，那将会提高前端的开发效率，减少许多发生在联调时候才会发生的问题。

通过适当地在代码中加入swagger的注解，可以让你的接口文档描述信息更加详细，如果你把每个出入参数的示例值都配上，那前端就可以直接在接口文档中拿到模拟数据。相关的注解类及参数配置可以参考文末他人写的技术文章，这里也不作展开了。

相关示例注解代码和效果图如下：


#####Controller代码
@Override
    @ApiOperation(value = "post请求调用示例", notes = "invokePost说明", httpMethod = "POST")
    public FFResponseModel<DemoOutputDto> invokePost(@ApiParam(name="传入对象",value="传入json格式",required=true) @RequestBody @Valid DemoDto input) {
        log.info("/testPost is called. input=" + input.toString());
        return new FFResponseModel(Errcode.SUCCESS_CODE, Errcode.SUCCESS_MSG);
    }


#####接口请求入参对象   
@Data
@ApiModel(value="演示类",description="请求参数类" )
public class DemoDto implements Serializable {

    private static final long serialVersionUID = 1L;

    @NotNull
    @ApiModelProperty(value = "defaultStr",example="mockStrValue")
    private String strDemo;

    @NotNull
    @ApiModelProperty(example="1234343523",required = true)
    private Long longNum;

    @NotNull
    @ApiModelProperty(example="111111.111")
    private Double doubleNum;

    @NotNull
    @ApiModelProperty(example="2018-12-04T13:46:56.711Z")
    private Date date;
    
}

#####接口请求出参公共类
@ApiModel(value="基础返回类",description="基础返回类")
public class FFResponseModel<T> implements Serializable {

    private static final long serialVersionUID = -2215304260629038881L;
    // 状态码
    @ApiModelProperty(example="成功")
    private String code;
    // 业务提示语
    @ApiModelProperty(example="000000")
    private String msg;
    // 数据对象
    private T data;

...
}

#####接口请求出参实际数据对象
@Data
public class DemoOutputDto {

    private String res;

    @NotNull
    @ApiModelProperty(value = "defaultOutputStr",example="mockOutputStrValue")
    private String outputStrDemo;

    @NotNull
    @ApiModelProperty(example="6666666",required = true)
    private Long outputLongNum;

    @NotNull
    @ApiModelProperty(example="88888.888")
    private Double outputDoubleNum;

    @NotNull
    @ApiModelProperty(example="2018-12-12T11:11:11.111Z")
    private Date outputDate;
    
}


    
效果图
模拟请求数据报文：


image.png

image.png
模拟返回数据报文：


image.png
总结
其实归根到底，使用Swagger，就是把相关的信息存储在它定义的描述文件里面（yml或json格式），再通过维护这个描述文件可以去更新接口文档，以及生成各端代码。而Springfox-swagger,则可以通过扫描代码去生成这个描述文件，连描述文件都不需要再去维护了。所有的信息，都在代码里面了。代码即接口文档，接口文档即代码。

相关技术资料站
SWAGGER 官网
【工具使用】API表达工具----swagger
Swagger 常用注解使用详解



作者：wuqke
链接：https://www.jianshu.com/p/349e130e40d5
来源：简书
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。
