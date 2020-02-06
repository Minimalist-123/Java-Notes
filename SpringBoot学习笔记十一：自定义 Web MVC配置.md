Spring MVC自动配置
当我们在项目中添加了spring-boot-starter-web依赖，Spring Boot会为Spring MVC提供自动配置，自动配置类为org.springframework.boot.autoconfigure.web.servlet包下的WebMvcAutoConfiguration

自动配置在Spring的默认值之上添加了以下功能：

配置InternalResourceViewResolver作为默认的视图解析器，包含ContentNegotiatingViewResolver和BeanNameViewResolver bean
支持提供静态资源，包括对WebJars的支持
自动注册Converter，GenericConverter和Formatter bean
支持HttpMessageConverters
自动注册MessageCodesResolver
静态index.html支持
自定义Favicon支持
自动使用ConfigurableWebBindingInitializer bean
如果要保留Spring Boot MVC功能并且想要添加其他 MVC配置（interceptors, formatters, view controllers以及其他功能），可以添加自己的类型为WebMvcConfigurer的配置类（以@Configuration注解标注），但不包含@EnableWebMvc。如果希望提供RequestMappingHandlerMapping，RequestMappingHandlerAdapter或ExceptionHandlerExceptionResolver的自定义实例，则可以声明WebMvcRegistrationsAdapter实例以提供此类组件。

如果想完全控制Spring MVC，可以使用@EnableWebMvc注解添加自己的配置类。

注意：Spring Boot 1.x时我们可以使用WebMvcConfigurerAdapter类来自定义Spring MVC配置，但Spring Boot 2.0已不推荐使用此类来进行自定义配置（已废弃），取而代之我们可以直接实现WebMvcConfigurer接口并重写相应方法来达到自定义Spring MVC配置的目的

大致原理就是WebMvcConfigurer接口基于java 8提供了默认方法，其实里面基本上也就是空实现

以下是WebMvcConfigurerAdapter的部分源码：

/**
 * An implementation of {@link WebMvcConfigurer} with empty methods allowing
 * subclasses to override only the methods they're interested in.
 *
 * @author Rossen Stoyanchev
 * @since 3.1
 * @deprecated as of 5.0 {@link WebMvcConfigurer} has default methods (made
 * possible by a Java 8 baseline) and can be implemented directly without the
 * need for this adapter
 */
@Deprecated
public abstract class WebMvcConfigurerAdapter implements WebMvcConfigurer {
    ...
}
HttpMessageConverters
Spring MVC使用HttpMessageConverter接口来转换HTTP请求和响应。消息转换器是在HttpMessageConvertersAutoConfiguration类中自动注册的。org.springframework.boot.autoconfigure.http包下HttpMessageConverters相关配置类如下：

消息转换器相关配置类
StringHttpMessageConverter是Spring Boot默认自动配置的HttpMessageConverter，除了默认的StringHttpMessageConverter，在HttpMessageConvertersAutoConfiguration配置类中还使用了@Import注解引入了JacksonHttpMessageConvertersConfiguration、GsonHttpMessageConvertersConfiguration、JsonbHttpMessageConvertersConfiguration，自动配置逻辑如下：

若jackson的相关jar包在类路径下，则通过JacksonHttpMessageConvertersConfiguration配置MappingJackson2HttpMessageConverter和MappingJackson2XmlHttpMessageConverter
若gson的相关jar包在类路径下且Jackson、Jsonb未依赖， 亦或gson的相关jar包在类路径下且设置了spring.http.converters.preferred-json-mapper属性值为gson，则通过GsonHttpMessageConvertersConfiguration配置GsonHttpMessageConverter
若JSON-B的相关jar包在类路径下且Jackson、Gson未依赖，亦或JSON-B的相关jar包在类路径下且设置了spring.http.converters.preferred-json-mapper属性值为jsonb，则通过JsonbHttpMessageConvertersConfiguration配置JsonbHttpMessageConverter
一般情况下，当我们在项目中添加了spring-boot-starter-web依赖，Spring Boot会默认引入jackson-core、jackson-databind、jackson-annotations依赖，但未引入jackson-dataformat-xml，因此会配置MappingJackson2HttpMessageConverter消息转换器。

最佳实践1：整合Gson
一般情况
排除Jackson依赖
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    <exclusions>
        <exclusion>
            <groupId>com.fasterxml.jackson.core</groupId>
            <artifactId>jackson-core</artifactId>
        </exclusion>
        <exclusion>
            <groupId>com.fasterxml.jackson.core</groupId>
            <artifactId>jackson-databind</artifactId>
        </exclusion>
        <exclusion>
            <groupId>com.fasterxml.jackson.core</groupId>
            <artifactId>jackson-annotations</artifactId>
        </exclusion>
        <exclusion>
            <groupId>com.fasterxml.jackson.datatype</groupId>
            <artifactId>jackson-datatype-jdk8</artifactId>
        </exclusion>
        <exclusion>
            <groupId>com.fasterxml.jackson.datatype</groupId>
            <artifactId>jackson-datatype-jsr310</artifactId>
        </exclusion>
        <exclusion>
            <groupId>com.fasterxml.jackson.module</groupId>
            <artifactId>jackson-module-parameter-names</artifactId>
        </exclusion>
    </exclusions>
</dependency>
这里完全排除了spring-boot-starter-web引入的jackson相关依赖，一般情况下排除jackson-databind依赖即可

添加Gson依赖
<dependency>
    <groupId>com.google.code.gson</groupId>
    <artifactId>gson</artifactId>
</dependency>
配置Gson相关属性
Spring Boot关于Gson的自动配置类为GsonAutoConfiguration，相关配置属性封装在GsonProperties类中
src/main/resources/application.yml

spring:
  gson:
    date-format: yyyy-MM-dd HH:mm:ss
这里贴出所有gson配置属性

# GSON GsonProperties
spring.gson.date-format= # Format to use when serializing Date objects.
spring.gson.disable-html-escaping= # Whether to disable the escaping of HTML characters such as '<', '>', etc.
spring.gson.disable-inner-class-serialization= # Whether to exclude inner classes during serialization.
spring.gson.enable-complex-map-key-serialization= # Whether to enable serialization of complex map keys (i.e. non-primitives).
spring.gson.exclude-fields-without-expose-annotation= # Whether to exclude all fields from consideration for serialization or deserialization that do not have the "Expose" annotation.
spring.gson.field-naming-policy= # Naming policy that should be applied to an object's field during serialization and deserialization.
spring.gson.generate-non-executable-json= # Whether to generate non executable JSON by prefixing the output with some special text.
spring.gson.lenient= # Whether to be lenient about parsing JSON that doesn't conform to RFC 4627.
spring.gson.long-serialization-policy= # Serialization policy for Long and long types.
spring.gson.pretty-printing= # Whether to output serialized JSON that fits in a page for pretty printing.
spring.gson.serialize-nulls= # Whether to serialize null fields.
这里有个问题就是用上述这种排除Jackson来配置Gson的方法在项目引入spring-boot-starter-jpa依赖后会报如下错误：

java.lang.IllegalStateException: Failed to introspect Class [org.springframework.data.web.config.SpringDataWebConfiguration] from ClassLoader [jdk.internal.loader.ClassLoaders$AppClassLoader@4459eb14]
    at org.springframework.util.ReflectionUtils.getDeclaredMethods(ReflectionUtils.java:659) ~[spring-core-5.0.8.RELEASE.jar:5.0.8.RELEASE]
    at org.springframework.util.ReflectionUtils.doWithMethods(ReflectionUtils.java:556) ~[spring-core-5.0.8.RELEASE.jar:5.0.8.RELEASE]
    at org.springframework.util.ReflectionUtils.doWithMethods(ReflectionUtils.java:541) ~[spring-core-5.0.8.RELEASE.jar:5.0.8.RELEASE]
    at org.springframework.util.ReflectionUtils.getUniqueDeclaredMethods(ReflectionUtils.java:599) ~[spring-core-5.0.8.RELEASE.jar:5.0.8.RELEASE]
    at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.getTypeForFactoryMethod(AbstractAutowireCapableBeanFactory.java:718) ~[spring-beans-5.0.8.RELEASE.jar:5.0.8.RELEASE]
    at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.determineTargetType(AbstractAutowireCapableBeanFactory.java:659) ~[spring-beans-5.0.8.RELEASE.jar:5.0.8.RELEASE]
    at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.predictBeanType(AbstractAutowireCapableBeanFactory.java:627) ~[spring-beans-5.0.8.RELEASE.jar:5.0.8.RELEASE]
    at org.springframework.beans.factory.support.AbstractBeanFactory.isFactoryBean(AbstractBeanFactory.java:1489) ~[spring-beans-5.0.8.RELEASE.jar:5.0.8.RELEASE]
    at org.springframework.beans.factory.support.DefaultListableBeanFactory.doGetBeanNamesForType(DefaultListableBeanFactory.java:419) ~[spring-beans-5.0.8.RELEASE.jar:5.0.8.RELEASE]
    at org.springframework.beans.factory.support.DefaultListableBeanFactory.getBeanNamesForType(DefaultListableBeanFactory.java:389) ~[spring-beans-5.0.8.RELEASE.jar:5.0.8.RELEASE]
    at org.springframework.beans.factory.support.DefaultListableBeanFactory.getBeansOfType(DefaultListableBeanFactory.java:510) ~[spring-beans-5.0.8.RELEASE.jar:5.0.8.RELEASE]
    at org.springframework.beans.factory.support.DefaultListableBeanFactory.getBeansOfType(DefaultListableBeanFactory.java:502) ~[spring-beans-5.0.8.RELEASE.jar:5.0.8.RELEASE]
    at org.springframework.context.support.AbstractApplicationContext.getBeansOfType(AbstractApplicationContext.java:1198) ~[spring-context-5.0.8.RELEASE.jar:5.0.8.RELEASE]
    at org.springframework.boot.SpringApplication.getExitCodeFromMappedException(SpringApplication.java:892) [spring-boot-2.0.4.RELEASE.jar:2.0.4.RELEASE]
    at org.springframework.boot.SpringApplication.getExitCodeFromException(SpringApplication.java:878) [spring-boot-2.0.4.RELEASE.jar:2.0.4.RELEASE]
    at org.springframework.boot.SpringApplication.handleExitCode(SpringApplication.java:864) [spring-boot-2.0.4.RELEASE.jar:2.0.4.RELEASE]
    at org.springframework.boot.SpringApplication.handleRunFailure(SpringApplication.java:813) [spring-boot-2.0.4.RELEASE.jar:2.0.4.RELEASE]
    at org.springframework.boot.SpringApplication.run(SpringApplication.java:341) [spring-boot-2.0.4.RELEASE.jar:2.0.4.RELEASE]
    at org.springframework.boot.SpringApplication.run(SpringApplication.java:1258) [spring-boot-2.0.4.RELEASE.jar:2.0.4.RELEASE]
    at org.springframework.boot.SpringApplication.run(SpringApplication.java:1246) [spring-boot-2.0.4.RELEASE.jar:2.0.4.RELEASE]
    at com.example.springbootmvc.SpringBootMvcApplication.main(SpringBootMvcApplication.java:10) [classes/:na]
    at java.base/jdk.internal.reflect.NativeMethodAccessorImpl.invoke0(Native Method) ~[na:na]
    at java.base/jdk.internal.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62) ~[na:na]
    at java.base/jdk.internal.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43) ~[na:na]
    at java.base/java.lang.reflect.Method.invoke(Method.java:564) ~[na:na]
    at org.springframework.boot.devtools.restart.RestartLauncher.run(RestartLauncher.java:49) [spring-boot-devtools-2.0.4.RELEASE.jar:2.0.4.RELEASE]
Caused by: java.lang.NoClassDefFoundError: com/fasterxml/jackson/databind/ObjectMapper
    at java.base/java.lang.Class.getDeclaredMethods0(Native Method) ~[na:na]
    at java.base/java.lang.Class.privateGetDeclaredMethods(Class.java:3119) ~[na:na]
    at java.base/java.lang.Class.getDeclaredMethods(Class.java:2268) ~[na:na]
    at org.springframework.util.ReflectionUtils.getDeclaredMethods(ReflectionUtils.java:641) ~[spring-core-5.0.8.RELEASE.jar:5.0.8.RELEASE]
    ... 25 common frames omitted
Caused by: java.lang.ClassNotFoundException: com.fasterxml.jackson.databind.ObjectMapper
    at java.base/jdk.internal.loader.BuiltinClassLoader.loadClass(BuiltinClassLoader.java:582) ~[na:na]
    at java.base/jdk.internal.loader.ClassLoaders$AppClassLoader.loadClass(ClassLoaders.java:190) ~[na:na]
    at java.base/java.lang.ClassLoader.loadClass(ClassLoader.java:499) ~[na:na]
    ... 29 common frames omitted
以上错误暂时未找到解决方法

特殊情况（无法排除jackson依赖）
添加Gson依赖（同上）
设置preferred-json-mapper属性
spring:
  http:
    converters:
      preferred-json-mapper: gson
设置preferred-json-mapper属性后依然可以使用spring.gson开头的属性在application.properties或application.yml文件中配置gson

最佳实践2：整合Fastjson
Fastjson简介
Fastjson是阿里巴巴旗下的一款开源json序列化与反序列化java类库，号称是java中最快的json类库
官方Github主页：https://github.com/alibaba/fastjson

Spring Boot整合Fastjson
添加依赖
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>fastjson</artifactId>
    <version>1.2.49</version>
</dependency>
集成Fastjson
参考自官方文档在 Spring 中集成 Fastjson
src/main/java/com/example/springbootmvc/config/WebMvcConfig.java

package com.example.springbootmvc.config;

import com.alibaba.fastjson.serializer.SerializerFeature;
import com.alibaba.fastjson.support.config.FastJsonConfig;
import com.alibaba.fastjson.support.spring.FastJsonHttpMessageConverter;
import org.springframework.context.annotation.Configuration;
import org.springframework.http.converter.HttpMessageConverter;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;

import java.util.List;

@Configuration
public class WebMvcConfig implements WebMvcConfigurer {

    @Override
    public void extendMessageConverters(List<HttpMessageConverter<?>> converters) {
        // 创建Fastjson消息转换器
        FastJsonHttpMessageConverter converter = new FastJsonHttpMessageConverter();
        // 创建Fastjson配置对象
        FastJsonConfig fastJsonConfig = new FastJsonConfig();
        fastJsonConfig.setSerializerFeatures(
                SerializerFeature.WriteNullBooleanAsFalse,
                SerializerFeature.WriteNullListAsEmpty,
                SerializerFeature.WriteNullNumberAsZero,
                SerializerFeature.WriteNullStringAsEmpty
        );
        converter.setFastJsonConfig(fastJsonConfig);
        converters.add(converter);
    }
}
Fastjson SerializerFeatures常用枚举值
枚举值	含义	备注
WriteNullListAsEmpty	List字段如果为null,输出为[],而非null	
WriteNullStringAsEmpty	字符类型字段如果为null,输出为"",而非null	
WriteNullBooleanAsFalse	Boolean字段如果为null,输出为false,而非null	
WriteNullNumberAsZero	数值字段如果为null,输出为0,而非null	
WriteMapNullValue	是否输出值为null的字段,默认为false	
自定义Jackson ObjectMapper
静态资源配置
Spring Boot中默认的静态资源配置是将类路径下的/static 、/public、/resources、/META-INF/resources文件夹中的静态资源直接映射为/**。这个默认行为是在WebMvcAutoConfiguration内部类WebMvcAutoConfigurationAdapter的addResourceHandlers方法中定义的，相关的属性配置类为ResourceProperties、WebMvcProperties
以下是部分源码：

@ConfigurationProperties(prefix = "spring.resources", ignoreUnknownFields = false)
public class ResourceProperties {

    private static final String[] CLASSPATH_RESOURCE_LOCATIONS = {
            "classpath:/META-INF/resources/", "classpath:/resources/",
            "classpath:/static/", "classpath:/public/" };

    /**
     * Locations of static resources. Defaults to classpath:[/META-INF/resources/,
     * /resources/, /static/, /public/].
     */
    private String[] staticLocations = CLASSPATH_RESOURCE_LOCATIONS;
}
@ConfigurationProperties(prefix = "spring.mvc")
public class WebMvcProperties {
    ...
    /**
     * Path pattern used for static resources.
     */
    private String staticPathPattern = "/**";
    ...
}
@Override
public void addResourceHandlers(ResourceHandlerRegistry registry) {
    if (!this.resourceProperties.isAddMappings()) {
        logger.debug("Default resource handling disabled");
        return;
    }
    Duration cachePeriod = this.resourceProperties.getCache().getPeriod();
    CacheControl cacheControl = this.resourceProperties.getCache()
            .getCachecontrol().toHttpCacheControl();
    // webjars支持
    if (!registry.hasMappingForPattern("/webjars/**")) {
        customizeResourceHandlerRegistration(registry
                .addResourceHandler("/webjars/**")
                .addResourceLocations("classpath:/META-INF/resources/webjars/")
                .setCachePeriod(getSeconds(cachePeriod))
                .setCacheControl(cacheControl));
    }
    String staticPathPattern = this.mvcProperties.getStaticPathPattern();
    // 默认静态资源处理
    if (!registry.hasMappingForPattern(staticPathPattern)) {
        customizeResourceHandlerRegistration(
                registry.addResourceHandler(staticPathPattern)
                        .addResourceLocations(getResourceLocations(
                                this.resourceProperties.getStaticLocations()))
                        .setCachePeriod(getSeconds(cachePeriod))
                        .setCacheControl(cacheControl));
    }
}
自定义静态资源配置
使用属性值配置
src/main/java/resources/application.yml

spring:
  mvc:
    static-path-pattern: /resources/**
  resources:
    static-locations: ["classpath:/META-INF/resources/", "classpath:/resources/",
                       "classpath:/static/", "classpath:/public/"]
重写addResourceHandlers进行配置
@Configuration
public class WebMvcConfig implements WebMvcConfigurer {

    @Override
    public void addResourceHandlers(ResourceHandlerRegistry registry) {
        registry.addResourceHandler("/**")
                .addResourceLocations("classpath:/META-INF/resources/", "classpath:/resources/",
                        "classpath:/static/", "classpath:/public/")
                .addResourceLocations("file:/Users/fulgens/Downloads/");
    }
    
}
以上代码相当于在默认配置基础上添加了虚拟目录

WebJars支持
WebJars简介
WebJars是将web前端资源（js，css等）打成jar包文件，然后借助Maven、Gradle等依赖管理及项目构建工具，以jar包形式对web前端资源进行统一依赖管理，保证这些Web资源版本唯一性。
官网地址 ： https://www.webjars.org/
WebJars官网
WebJars使用
引入依赖
官网首页，找到资源文件对应的maven依赖，写入项目pom.xml文件

<dependency>
    <groupId>org.webjars</groupId>
    <artifactId>webjars-locator</artifactId>
    <version>0.34</version>
</dependency>
<dependency>
    <groupId>org.webjars</groupId>
    <artifactId>jquery</artifactId>
    <version>3.3.1</version>
</dependency>
<dependency>
    <groupId>org.webjars</groupId>
    <artifactId>bootstrap</artifactId>
    <version>4.1.3</version>
</dependency>
<dependency>
    <groupId>org.webjars</groupId>
    <artifactId>layui</artifactId>
    <version>2.3.0</version>
</dependency>
这里引入了jquery、bootstrap、layui

页面引用
<link rel="stylesheet" href="/webjars/layui/css/layui.css"/>
<link rel="stylesheet" href="/webjars/bootstrap/css/bootstrap.min.css"/>
<script src="/webjars/jquery/jquery.js"></script>
<script src="/webjars/bootstrap/js/bootstrap.js"></script>
<script src="/webjars/layui/layui.all.js"></script>
注意：这里由于添加了webjars-locator依赖，在引入前端资源时省略了版本号，推荐使用
如果未添加webjars-locator依赖，在引入前端资源时你需要添加版本号，像下面这样：

<script src="/webjars/jquery/3.3.1/jquery.js"></script>
欢迎页Welcome Page
Spring Boot支持静态和模板化欢迎页面。 它首先在配置的静态资源目录中查找index.html文件。 如果找不到，则查找index模板。 如果找到任何一个，将自动用作应用程序的欢迎页面。
Spring Boot关于欢迎页的处理类为WelcomePageHandlerMapping

自定义Favicon
Spring Boot默认的提供的favicon是片小叶子，我们也可以自定义favicon，另Spring Boot关于Favicon的配置类为FaviconConfiguration，其默认会在配置的静态资源路径和类路径根目录下查找名为favicon.ico的Favicon图片，如果存在，则自动应用为应用的Favicon

关闭Favicon
设置spring.mvc.favicon.enabled属性值为false即可关闭Favicon，默认值为true
src/main/resources/application.yml

spring:
  mvc:
    favicon:
      enabled: false    # 禁用favicon
设置自己的Favicon
根据以上原理，我们只需要在静态资源路径或类路径根目录下放一张名为favicon.ico的Favicon图片即可

注意：需要注意浏览器缓存，导致favicon.ico没能及时更新，需浏览器清缓存。

模板引擎
整合Freemarker
添加依赖
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-freemarker</artifactId>
</dependency>
相关配置
src/main/resources/application.yml

spring:
  freemarker:
    suffix: .ftl
    cache: false
    charset: UTF-8
    content-type: text/html
    template-loader-path: "classpath:/templates/"
    expose-request-attributes: true
    expose-session-attributes: true
    expose-spring-macro-helpers: true
    request-context-attribute: request
这里贴出Freemarker所有配置属性

# FREEMARKER FreeMarkerProperties
spring.freemarker.allow-request-override=false # Whether HttpServletRequest attributes are allowed to override (hide) controller generated model attributes of the same name.
spring.freemarker.allow-session-override=false # Whether HttpSession attributes are allowed to override (hide) controller generated model attributes of the same name.
spring.freemarker.cache=false # Whether to enable template caching.
spring.freemarker.charset=UTF-8 # Template encoding.
spring.freemarker.check-template-location=true # Whether to check that the templates location exists.
spring.freemarker.content-type=text/html # Content-Type value.
spring.freemarker.enabled=true # Whether to enable MVC view resolution for this technology.
spring.freemarker.expose-request-attributes=false # Whether all request attributes should be added to the model prior to merging with the template.
spring.freemarker.expose-session-attributes=false # Whether all HttpSession attributes should be added to the model prior to merging with the template.
spring.freemarker.expose-spring-macro-helpers=true # Whether to expose a RequestContext for use by Spring's macro library, under the name "springMacroRequestContext".
spring.freemarker.prefer-file-system-access=true # Whether to prefer file system access for template loading. File system access enables hot detection of template changes.
spring.freemarker.prefix= # Prefix that gets prepended to view names when building a URL.
spring.freemarker.request-context-attribute= # Name of the RequestContext attribute for all views.
spring.freemarker.settings.*= # Well-known FreeMarker keys which are passed to FreeMarker's Configuration.
spring.freemarker.suffix=.ftl # Suffix that gets appended to view names when building a URL.
spring.freemarker.template-loader-path=classpath:/templates/ # Comma-separated list of template paths.
spring.freemarker.view-names= # White list of view names that can be resolved.</pre>
简单实践
src/main/resources/templates/index.ftl

<#import "app.ftl" as app>
<base id="basePath" href="${app.basePath}/">
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>后台首页</title>
    <link rel="stylesheet" href="/webjars/layui/css/layui.css"/>
    <link rel="stylesheet" href="/webjars/bootstrap/css/bootstrap.min.css"/>
    <script src="/webjars/jquery/jquery.js"></script>
    <script src="/webjars/bootstrap/js/bootstrap.js"></script>
</head>
<body class="layui-layout-body">
<div class="layui-layout layui-layout-admin">

    <#include "layout/header.ftl">

    <#include "layout/menu.ftl">

    <div class="layui-body">
        <div style="padding: 15px;">Hello World!</div>
    </div>

    <#include "layout/footer.ftl">

</div>
<script src="/webjars/layui/layui.all.js"></script>
<script>
    layui.use('element', function(){
        var element = layui.element;

    });
</script>
</body>
</html> 
src/main/resources/templates/app.ftl

<#assign basePath=request.contextPath >
src/main/resources/templates/layout/header.ftl

<#--header start-->
<div class="layui-header">
    <div class="layui-logo">Xxx后台管理系统</div>
    <ul class="layui-nav layui-layout-left">
        <li class="layui-nav-item"><a href="">控制台</a></li>
        <li class="layui-nav-item"><a href="">商品管理</a></li>
        <li class="layui-nav-item"><a href="">用户</a></li>
        <li class="layui-nav-item">
            <a href="javascript:;">其它系统</a>
            <dl class="layui-nav-child">
                <dd><a href="">邮件管理</a></dd>
                <dd><a href="">消息管理</a></dd>
                <dd><a href="">授权管理</a></dd>
            </dl>
        </li>
    </ul>
    <ul class="layui-nav layui-layout-right">
        <li class="layui-nav-item">
            <a href="javascript:;">
                <img src="http://t.cn/RCzsdCq" class="layui-nav-img">
                管理员
            </a>
            <dl class="layui-nav-child">
                <dd><a href="">基本资料</a></dd>
                <dd><a href="">安全设置</a></dd>
            </dl>
        </li>
        <li class="layui-nav-item"><a href="">退了</a></li>
    </ul>
</div>
<#--header end-->
src/main/resources/templates/layout/footer.ftl

<#--footer start-->
<div class="layui-footer">
    Copyright © 20xx - 2018  example.com 版权所有
</div>
<#--footer end-->
src/main/resources/templates/layout/menu.ftl

<#--menu start-->
<div class="layui-side layui-bg-black">
    <div class="layui-side-scroll">
        <ul class="layui-nav layui-nav-tree"  lay-filter="test">
            <li class="layui-nav-item layui-nav-itemed">
                <a class="" href="javascript:;">用户管理</a>
                <dl class="layui-nav-child">
                    <dd><a href="javascript:;">列表一</a></dd>
                    <dd><a href="javascript:;">列表二</a></dd>
                    <dd><a href="javascript:;">列表三</a></dd>
                    <dd><a href="">超链接</a></dd>
                </dl>
            </li>
            <li class="layui-nav-item">
                <a href="javascript:;">商品管理</a>
                <dl class="layui-nav-child">
                    <dd><a href="javascript:;">列表一</a></dd>
                    <dd><a href="javascript:;">列表二</a></dd>
                    <dd><a href="">超链接</a></dd>
                </dl>
            </li>
            <li class="layui-nav-item"><a href="">云市场</a></li>
            <li class="layui-nav-item"><a href="">发布商品</a></li>
        </ul>
    </div>
</div>
<#--menu end-->
视图控制器配置
对于一个传统的非前后端分离项目来说，视图控制器的配置是至关重要的，可以通过重写WebMvcConfigurer的addViewControllers(ViewControllerRegistry registry)方法来配置

@Override
public void addViewControllers(ViewControllerRegistry registry) {
    registry.addViewController("/").setViewName("/index");
    registry.addViewController("/index").setViewName("/index");
    registry.addViewController("/register").setViewName("/register");
    registry.addViewController("/login").setViewName("/login");
}
上面的代码与我们自己写一个Controller完成视图映射是一样的

@Controller
public class RouterController {

    @GetMapping(value = {"/", "/index"})
    public String toIndex() {
        return "/index";
    }

    @GetMapping("/register")
    public String toRegister() {
        return "/register";
    }

    @GetMapping("/login")
    public String toLogin() {
        return "/login";
    }

}
拦截器配置
这里是一个简单的登录校验拦截器
src/main/java/com/example/springbootmvc/web/interceptor/LoginInterceptor

package com.example.springbootmvc.web.interceptor;

import com.alibaba.fastjson.JSON;
import com.example.springbootmvc.common.constants.CommonContext;
import com.example.springbootmvc.common.utils.ServerResponse;
import com.example.springbootmvc.entity.User;
import org.apache.commons.lang3.StringUtils;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.web.method.HandlerMethod;
import org.springframework.web.servlet.HandlerInterceptor;
import org.springframework.web.servlet.ModelAndView;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
import java.io.OutputStream;
import java.util.Arrays;
import java.util.Iterator;
import java.util.Map;

/**
 * 登录校验拦截器
 */
public class LoginInterceptor implements HandlerInterceptor {

    private static final Logger log = LoggerFactory.getLogger(LoginInterceptor.class);

    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        if (handler instanceof HandlerMethod) {
            HandlerMethod handlerMethod = (HandlerMethod) handler;
            String methodName = handlerMethod.getMethod().getName();
            String className = handlerMethod.getBean().getClass().getSimpleName();
            log.info("拦截controller: {}， 拦截方法: {}", className, methodName);
        }

        // 获取请求url
        String toURL = request.getRequestURI();
        String queryString = request.getQueryString();
        if (StringUtils.isNotEmpty(queryString)) {
            toURL += "?" + queryString;
        }
        log.info("拦截请求URL: {}", toURL);

        // 获取拦截请求的请求参数
        StringBuffer sb = new StringBuffer();
        Map<String, String[]> parameterMap = request.getParameterMap();
        Iterator<Map.Entry<String, String[]>> iterator = parameterMap.entrySet().iterator();
        while (iterator.hasNext()) {
            Map.Entry<String, String[]> entry = iterator.next();
            String mapKey = entry.getKey();
            String mapValue = StringUtils.EMPTY;
            mapValue = Arrays.toString(entry.getValue());
            sb.append(mapKey).append("=").append(mapValue);
        }
        log.info("拦截请求入参: {}", sb.toString());

        User currenUser = (User) request.getSession().getAttribute(CommonContext.CURRENT_USER_CONTEXT);
        if (currenUser == null) {
            // 用户未登录跳转登录页面
            // response.sendRedirect("/login");
            request.getRequestDispatcher("/login").forward(request, response);
            // 保存用户请求url用于登录成功后跳转
            request.getSession().setAttribute(CommonContext.LOGIN_REDIRECT_URL, toURL);
            return false;
        }
        return true;
    }

    @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {

    }

    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {

    }
}
对于前后端分离的项目可能在拦截校验不通过时需要向前台返回一些信息

private void returnErrorMsg(HttpServletResponse response, ServerResponse serverResponse) {
    try (OutputStream os = response.getOutputStream()) {
        response.setCharacterEncoding("UTF-8");
        response.setContentType("application/json;charset=UTF-8");
        os.write(JSON.toJSONString(serverResponse).getBytes());
        os.flush();
    } catch (IOException e) {
        log.error("登录校验拦截器输出错误信息发生异常，异常信息： {}", e);
    }
}
这里ServerResponse是封装的一个通用服务端响应对象
通过重写WebMvcConfigurer的addInterceptors(InterceptorRegistry registry)方法配置拦截器

@Override
public void addInterceptors(InterceptorRegistry registry) {
    String[] excludePath = {"/login", "/doLogin", "/register", "/doRegister",
            "/error", "/**/*.js", "/**/*.css", "/**/*.jpg", "/**/*.jpeg",
            "/**/*.png", "/**/*.ico"};
    registry.addInterceptor(new LoginInterceptor())
            .addPathPatterns("/**")
            .excludePathPatterns(excludePath);
}
全局异常处理
Spring Boot默认的错误处理机制
默认情况下，Spring Boot提供/error错误映射，以合理的方式处理所有错误，并在servlet容器中注册为“全局”错误页面。对于机器客户端（machine clients），它会生成一个JSON响应，其中包含错误信、HTTP状态和异常消息的详细信息。对于浏览器客户端（browser clients），有一个“whitelabel”错误视图，以HTML格式呈现相同的数据（要自定义它，须添加一个解析错误的视图）。要完全替换默认行为，可以实现ErrorController接口并注册该类型的bean，或者添加ErrorAttributes类型的bean以使用现有机制但替换内容。
比如，我们自定义一个产生异常的映射：

@GetMapping("/test")
public void testException() {
    int i = 1/0;
}
浏览器访问http://localhost:8080/test我们会看到
Spring Boot默认whitelabel错误页面
再使用Restlet、Postman等接口测试工具访问


Spring Boot生成的JSON错误信息
其实，Spring Boot默认错误处理机制在BasicErrorController中定义，部分源码如下：

@Controller
@RequestMapping("${server.error.path:${error.path:/error}}")
public class BasicErrorController extends AbstractErrorController {

    private final ErrorProperties errorProperties;
    // ...
        
    @RequestMapping(produces = "text/html")
    public ModelAndView errorHtml(HttpServletRequest request,
            HttpServletResponse response) {
        HttpStatus status = getStatus(request);
        Map<String, Object> model = Collections.unmodifiableMap(getErrorAttributes(
                request, isIncludeStackTrace(request, MediaType.TEXT_HTML)));
        response.setStatus(status.value());
        ModelAndView modelAndView = resolveErrorView(request, response, status, model);
        return (modelAndView != null ? modelAndView : new ModelAndView("error", model));
    }

    @RequestMapping
    @ResponseBody
    public ResponseEntity<Map<String, Object>> error(HttpServletRequest request) {
        Map<String, Object> body = getErrorAttributes(request,
                isIncludeStackTrace(request, MediaType.ALL));
        HttpStatus status = getStatus(request);
        return new ResponseEntity<>(body, status);
    }
    // ...
}
BasicErrorController类图
自定义错误页面
如果要显示给定状态码的自定义HTML错误页面，可以将文件添加到/error目录。 错误页面可以是静态HTML（即，添加到任何静态资源文件夹下），也可以使用模板构建。 文件名应该是确切的状态代码或系列掩码。
例如，要映射404错误到静态HTML文件，目录结构将如下所示：

src/
 +- main/
  +- java/
   | + <source code>
  +- resources/
   +- public/
    +- error/
     | +- 404.html
    +- <other public assets>

同样映射500错误我们可以在/error目录下放一个500.html

使用FreeMarker模板映射所有5xx错误，目录结构如下：

src/
 +- main/
  +- java/
   | + <source code>
  +- resources/
   +- templates/
    +- error/
     | +- 5xx.ftl
    +- <other templates>

这里给出一个简单的5xx.ftl模板

<!DOCTYPE html>
<head>
    <meta charset="UTF-8"/>
    <title>5xx</title>
</head>
<body>
<div class="row border-bottom">
    <h1>Oh, There is something wrong</h1>
    <h3>timestamp: ${timestamp?datetime}</h3>
    <h3>status: ${status}</h3>
    <h3>error: ${error}</h3>
    <h3>message: ${message}</h3>
    <h3>path: ${path}</h3>
</div>
</body>
</html>
配置了以上5xx模板，我们再次访问http://localhost:8080/test

5xx模板页面错误信息展示
你可能会问timestamp、status...这些模型数据从哪来的呢？我们什么也没做不是吗？其实还是BasicErrorController在起作用，Spring默认提供了ErrorAttributes接口的实现类DefaultErrorAttributes，感兴趣可以去看一下其中的getErrorAttributes方法

对于更复杂的映射，还可以实现ErrorViewResolver接口，如以下示例所示：

public class MyErrorViewResolver implements ErrorViewResolver {

    @Override
    public ModelAndView resolveErrorView(HttpServletRequest request,
            HttpStatus status, Map<String, Object> model) {
        // Use the request or status to optionally return a ModelAndView
        return ...
    }

}
全局异常处理
Spring Boot默认的错误处理机制一般不会符合项目的要求，这个时候就需要我们自定义全局异常处理了

这里说一下，没有全局异常处理的系统，往往会使用像下面这样的笨办法，采用try-catch的方式，手动捕获来自service层的异常信息，然后返回对应的结果集，相信很多人都看到过类似的代码（如：封装成Result对象）；该方法虽然间接性的解决错误暴露的问题，同样的弊端也很明显，增加了代码量，当异常过多的情况下对应的catch层愈发的多了起来，很难管理这些业务异常和错误码之间的匹配，所以最好的方法就是通过简单配置全局掌控….

@GetMapping("/test2")
public Map<String, String> test2() {
    Map<String, Object> resultMap = new HashMap<>();
    // TODO 采用catch手动捕获，间接性的解决错误暴露的问题
    try {
        int i = 1 / 0;
        resultMap.put("code", "200");
        resultMap.put("data", "具体返回的结果集");
    } catch (Exception e) {
        resultMap.put("code", "500");
        resultMap.put("msg", "接口调用异常");
    }
    return resultMap;
}
自定义异常
src/main/java/com/example/springbootmvc/exception/CustomException.class

package com.example.springbootmvc.exception;

public class CustomException extends RuntimeException {

    private Integer code;

    public CustomException() {
        super();
    }

    public CustomException(Integer code, String message) {
        super(message);
        this.code = code;
    }

    public Integer getCode() {
        return code;
    }

}
通用服务端响应对象
src/main/java/com/example/springbootmvc/common/utils/ServerResponse.class

package com.example.springbootmvc.common.utils;

import com.alibaba.fastjson.annotation.JSONField;
import com.example.springbootmvc.common.enums.ResponseCode;
import com.fasterxml.jackson.annotation.JsonIgnore;
import com.fasterxml.jackson.annotation.JsonInclude;

import java.io.Serializable;

// @JsonInclude(JsonInclude.Include.NON_NULL)
public class ServerResponse<T> implements Serializable {

    private static final long serialVersionUID = -4577255781088498763L;

    // 响应状态
    private Integer status;

    // 响应消息
    private String msg;

    // 响应数据
    private T data;

    private ServerResponse() {

    }

    private ServerResponse(Integer status) {
        this.status = status;
    }

    private ServerResponse(Integer status, String msg) {
        this.status = status;
        this.msg = msg;
    }

    private ServerResponse(Integer status, T data) {
        this.status = status;
        this.data = data;
    }

    private ServerResponse(Integer status, String msg, T data) {
        this.status = status;
        this.msg = msg;
        this.data = data;
    }

    // @JsonIgnore  // jackson
    @JSONField(serialize = false)
    public boolean isSuccess() {
        return this.status == ResponseCode.SUCCESS.getCode();
    }

    public Integer getStatus() {
        return status;
    }

    public String getMsg() {
        return msg;
    }

    public T getData() {
        return data;
    }

    public static <T> ServerResponse<T> success() {
        return new ServerResponse<>(ResponseCode.SUCCESS.getCode());
    }

    public static <T> ServerResponse<T> successWithMsg(String msg) {
        return new ServerResponse<>(ResponseCode.SUCCESS.getCode(), msg);
    }

    public static <T> ServerResponse<T> successWithData(T data) {
        return new ServerResponse<>(ResponseCode.SUCCESS.getCode(), data);
    }

    public static <T> ServerResponse<T> successWithMsgAndData(String msg, T data) {
        return new ServerResponse<T>(ResponseCode.SUCCESS.getCode(), msg, data);
    }

    public static <T> ServerResponse<T> error() {
        return new ServerResponse<>(ResponseCode.ERROR.getCode());
    }

    public static <T> ServerResponse<T> errorWithMsg(String errorMsg) {
        return new ServerResponse<>(ResponseCode.ERROR.getCode(), errorMsg);
    }

    public static <T> ServerResponse<T> errorWithMsg(int errorCode, String errorMsg) {
        return new ServerResponse<>(errorCode, errorMsg);
    }

    public static <T> ServerResponse build(Integer status, String msg, T data) {
        return new ServerResponse(status, msg, data);
    }

}
全局异常处理
使用@ControllerAdvice及@ExceptionHandler
@ControllerAdvice 捕获 Controller 层抛出的异常，如果添加 @ResponseBody 返回信息则为JSON 格式。
@RestControllerAdvice 相当于 @ControllerAdvice 与 @ResponseBody 的结合体。
@ExceptionHandler 统一处理一种类的异常，减少代码冗余度。
@ResponseStatus 返回Http响应状态码
对于非前后端分离的传统项目（使用模板构建）往往需要同时支持自定义错误页面展示及Ajax请求返回错误信息
src/main/java/com/example/springbootmvc/aop/GlobalExceptionHandler.class

package com.example.springbootmvc.aop;

import com.alibaba.fastjson.JSON;
import com.example.springbootmvc.common.utils.ServerResponse;
import com.example.springbootmvc.exception.CustomException;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.http.HttpStatus;
import org.springframework.web.bind.annotation.ControllerAdvice;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.servlet.ModelAndView;
import org.springframework.web.servlet.mvc.method.annotation.ResponseEntityExceptionHandler;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
import java.io.OutputStream;

@ControllerAdvice
public class GlobalExceptionHandler extends ResponseEntityExceptionHandler {

    private static final Logger log = LoggerFactory.getLogger(GlobalExceptionHandler.class);

    @ExceptionHandler(CustomException.class)
    public Object handleCustomException(HttpServletRequest request, HttpServletResponse response, Exception e) {
        CustomException exception = (CustomException) e;
        if (isAjax(request)) {
            ServerResponse serverResponse = ServerResponse.build(exception.getCode(), exception.getMessage(), null);
            return serverResponse;
        } else {
            ModelAndView modelAndView = new ModelAndView();
            modelAndView.setStatus(getStatus(request));
            modelAndView.addObject("path", request.getRequestURI());
            modelAndView.addObject("exception", exception.getMessage());
            modelAndView.setViewName("error/error");
            return modelAndView;
        }
    }

    private boolean isAjax(HttpServletRequest request) {
        return request.getHeader("X-Requested-With") != null
                && "XMLHttpRequest".equals(request.getHeader("X-Requested-With"));
    }

    private void writeErrorMsg(HttpServletResponse response, ServerResponse serverResponse) {
        try (OutputStream os = response.getOutputStream()) {
            response.setCharacterEncoding("UTF-8");
            response.setContentType("application/json;charset=UTF-8");
            os.write(JSON.toJSONString(serverResponse).getBytes());
            os.flush();
        } catch (IOException e) {
            log.error("输出错误信息发生异常，异常信息： {}", e);
        }
    }

    private HttpStatus getStatus(HttpServletRequest request) {
        Integer statusCode = (Integer) request.getAttribute("javax.servlet.error.status_code");
        if (statusCode == null) {
            return HttpStatus.INTERNAL_SERVER_ERROR;
        }
        return HttpStatus.valueOf(statusCode);
    }

}
src/main/resources/templates/error/error.ftl

<!DOCTYPE html>
<head>
    <meta charset="UTF-8"/>
    <title>5xx</title>
</head>
<body>
<div class="row border-bottom">
    <h1>Oh, There is something wrong</h1>
    <h3>exception: ${exception}</h3>
    <h3>path: ${path}</h3>
</div>
</body>
</html>
对于前后端分离的项目只需要返回错误信息即可

package com.example.springbootmvc.aop;

import com.example.springbootmvc.common.utils.ServerResponse;
import com.example.springbootmvc.exception.CustomException;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.http.HttpHeaders;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.MethodArgumentNotValidException;
import org.springframework.web.bind.annotation.ControllerAdvice;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.ResponseStatus;
import org.springframework.web.bind.annotation.RestControllerAdvice;
import org.springframework.web.context.request.WebRequest;
import org.springframework.web.method.annotation.MethodArgumentTypeMismatchException;
import org.springframework.web.servlet.mvc.method.annotation.ResponseEntityExceptionHandler;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

@RestControllerAdvice
public class GlobalExceptionHandler2 extends ResponseEntityExceptionHandler {

    private static final Logger log = LoggerFactory.getLogger(GlobalExceptionHandler2.class);

    @ExceptionHandler(CustomException.class)
    @ResponseStatus(HttpStatus.BAD_REQUEST)
    public Object handleCustomException(HttpServletRequest request, HttpServletResponse response, Exception e) {
        CustomException exception = (CustomException) e;
        return ServerResponse.build(exception.getCode(), exception.getMessage(), null);
    }

    @ExceptionHandler(RuntimeException.class)
    @ResponseStatus(HttpStatus.BAD_REQUEST)
    public Object handleRuntimeException(HttpServletRequest request, HttpServletResponse response, Exception e) {
        RuntimeException exception = (RuntimeException) e;
        return ServerResponse.build(400, exception.getMessage(), null);
    }

    /**
     * 通用的接口映射异常处理方法
     */
    @Override
    protected ResponseEntity<Object> handleExceptionInternal(Exception e, Object body, HttpHeaders headers,
                                                             HttpStatus status, WebRequest request) {
        if (e instanceof MethodArgumentNotValidException) {
            MethodArgumentNotValidException exception = (MethodArgumentNotValidException) e;
            return new ResponseEntity(ServerResponse.build(Integer.valueOf(status.value()), exception.getBindingResult().getAllErrors().get(0).getDefaultMessage(), null), status);
        }
        if (e instanceof MethodArgumentTypeMismatchException) {
            MethodArgumentTypeMismatchException exception = (MethodArgumentTypeMismatchException) e;
            log.error("参数转换失败，方法：{}，参数：{}，信息：", exception.getParameter().getMethod().getName(),
                    exception.getParameter(), exception.getLocalizedMessage());
            return new ResponseEntity(ServerResponse.build(Integer.valueOf(status.value()), "参数转换失败", null), status);
        }
        return new ResponseEntity(ServerResponse.build(Integer.valueOf(status.value()), "参数转换失败", null), status);
    }

}
自定义异常使用
在Controller层直接向上抛出即可

@RequestMapping("/test")
public void testException(String param) {
    if (param == null) {
        throw new CustomException(400, "参数不能为空");
    }
    int i = 1/0;
}


作者：fulgens
链接：https://www.jianshu.com/p/d0bddbf729af
来源：简书
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。
