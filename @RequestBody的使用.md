提示：建议一定要看后面的@RequestBody的核心逻辑源码以及六个重要结论！本文前半部分的内容都是一些基
           本知识常识，可选择性跳过。

声明：本文是基于SpringBoot，进行的演示说明。

基础知识介绍：
        @RequestBody主要用来接收前端传递给后端的json字符串中的数据的(请求体中的数据的)；GET方式无请求体，所以使用@RequestBody接收数据时，前端不能使用GET方式提交数据，而是用POST方式进行提交。在后端的同一个接收方法里，@RequestBody与@RequestParam()可以同时使用，@RequestBody最多只能有一个，而@RequestParam()可以有多个。

注：一个请求，只有一个RequestBody；一个请求，可以有多个RequestParam。

注：当同时使用@RequestParam（）和@RequestBody时，@RequestParam（）指定的参数可以是普通元素、
       数组、集合、对象等等(即:当，@RequestBody 与@RequestParam()可以同时使用时，原SpringMVC接收
       参数的机制不变，只不过RequestBody 接收的是请求体里面的数据；而RequestParam接收的是key-value
       里面的参数，所以它会被切面进行处理从而可以用普通元素、数组、集合、对象等接收)。
       即：如果参数时放在请求体中，传入后台的话，那么后台要用@RequestBody才能接收到；如果不是放在
              请求体中的话，那么后台接收前台传过来的参数时，要用@RequestParam来接收，或则形参前
              什么也不写也能接收。

注：如果参数前写了@RequestParam(xxx)，那么前端必须有对应的xxx名字才行(不管其是否有值，当然可以通
       过设置该注解的required属性来调节是否必须传)，如果没有xxx名的话，那么请求会出错，报400。

注：如果参数前不写@RequestParam(xxx)的话，那么就前端可以有可以没有对应的xxx名字才行，如果有xxx名
       的话，那么就会自动匹配；没有的话，请求也能正确发送。
       追注：这里与feign消费服务时不同；feign消费服务时，如果参数前什么也不写，那么会被默认是
                  @RequestBody的。

如果后端参数是一个对象，且该参数前是以@RequestBody修饰的，那么前端传递json参数时，必须满足以下要求：

后端@RequestBody注解对应的类在将HTTP的输入流(含请求体)装配到目标类(即：@RequestBody后面的类)时，会根据json字符串中的key来匹配对应实体类的属性，如果匹配一致且json中的该key对应的值符合(或可转换为)，这一条我会在下面详细分析，其他的都可简单略过，但是本文末的核心逻辑代码以及几个结论一定要看！ 实体类的对应属性的类型要求时,会调用实体类的setter方法将值赋给该属性。

json字符串中，如果value为""的话，后端对应属性如果是String类型的，那么接受到的就是""，如果是后端属性的类型是Integer、Double等类型，那么接收到的就是null。

json字符串中，如果value为null的话，后端对应收到的就是null。

如果某个参数没有value的话，在传json字符串给后端时，要么干脆就不把该字段写到json字符串中；要么写value时， 必须有值，null  或""都行。千万不能有类似"stature":，这样的写法，如:



注：关于@RequestParam()的用法，这里就不再一一说明了，可详见 《程序员成长笔记(一)》中的相关章节。

示例详细说明：
先给出两个等下要用到的实体类

User实体类：



Team实体类：



@RequestBody直接以String接收前端传过来的json数据：

后端对应的Controller：



使用PostMan测试：



@RequestBody以简单对象接收前端传过来的json数据：

后端对应的Controller：



使用PostMan测试：



@RequestBody以复杂对象接收前端传过来的json数据：

后端对应的Controller：



使用PostMan测试：



@RequestBody与简单的@RequestParam()同时使用：

后端对应的Controller：



使用PostMan测试：



@RequestBody与复杂的@RequestParam()同时使用：

后端对应的Controller：



使用PostMan测试：



@RequestBody接收请求体中的json数据；不加注解接收URL中的数据并组装为对象：

后端对应的Controller：



使用PostMan测试：



注：如果在后端方法参数前，指定了@RequestParam()的话，那么前端必须要有对应字段才行(当然可以通过设置
       该注解的required属性来调节是否必须传)，否者会报错；如果参数前没有任何该注解，那么前端可以传，也可
       以不传，如：



上图中，如果我们传参中没有指定token，那么请求能正常进去，但是token为null；如果在String token前指定了@RequestParam(“token”)，那么前端必须要有token这个键时，请求才能正常进去，否者报400错误。

@RequestBody与前端传过来的json数据的匹配规则
声明：根据不同的Content-Type等情况,Spring-MVC会采取不同的HttpMessageConverter实现来进行信息转换解析。
          下面介绍的是最常用的：前端以Content-Type 为application/json,传递json字符串数据;后端以@RequestBody
          模型接收数据的情况。

解析json数据大体流程概述：
        Http传递请求体信息，最终会被封装进com.fasterxml.jackson.core.json.UTF8StreamJsonParser中(提示：Spring采用CharacterEncodingFilter设置了默认编码为UTF-8)，然后在public class BeanDeserializer extends BeanDeserializerBase implements java.io.Serializable中，通过 public Object deserializeFromObject(JsonParser p, DeserializationContext ctxt) throws IOException方法进行解析。

核心逻辑分析示例：
        假设前端传的json串是这样的： {"name1":"邓沙利文","age":123,"mot":"我是一只小小小小鸟~"} 后端的模型只有name和age属性，以及对应的setter/getter方法；给出一般用到的deserializeFromObject(JsonParser p, DeserializationContext ctxt)方法的核心逻辑：



小技巧之指定模型中的属性对应什么key
这里简单介绍，更多的可参考：

           public class BeanPropertyMap implements Iterable<SettableBeanProperty>,java.io.Serializable

给出Controller中的测试类:



给出模型中的属性(setter/getter方法没截出来)：



使用postman测试一下，示例：



上图简单测试了一下，但是测得并不全面,这里就不带大家一起测试了，直接给出。

全面的结论：
结论①：@JsonAlias注解，实现:json转模型时，使json中的特定key能转化为特定的模型属性;但是模型转json时，
               对应的转换后的key仍然与属性名一致，见：上图示例中的name字段的请求与响应。
               以下图进一步说明：



                  此时，json字符串转换为模型时，json中key为Name或为name123或为name的都能识别。

结论②：@JsonProperty注解，实现：json转模型时，使json中的特定key能转化为指定的模型属性；同样的，模
               型转json时，对应的转换后的key为指定的key，见：示例中的motto字段的请求与响应。
               以下图进一步说明：



               此时，json字符串转换为模型时，key为MOTTO的能识别，但key为motto的不能识别。

结论③：@JsonAlias注解需要依赖于setter、getter，而@JsonProperty注解不需要。

结论④：在不考虑上述两个注解的一般情况下，key与属性匹配时,默认大小写敏感。

结论⑤：有多个相同的key的json字符串中，转换为模型时，会以相同的几个key中，排在最后的那个key的值给模
               型属性复制，因为setter会覆盖原来的值。见示例中的gender属性。

结论⑥：后端@RequestBody注解对应的类在将HTTP的输入流(含请求体)装配到目标类(即:@RequestBody后面
               的类)时，会根据json字符串中的key来匹配对应实体类的属性，如果匹配一致且json中的该key对应的值
               符合(或可转换为)实体类的对应属性的类型要求时，会调用实体类的setter方法将值赋给该属性。

 

^_^ 如有不当之处，欢迎指正
^_^ 代码托管链接
               https://github.com/JustryDeng...RequestBody...
^_^ 本文已经被收录进《程序员成长笔记(二)》，笔者JustryDeng
————————————————
版权声明：本文为CSDN博主「justry_deng」的原创文章，遵循 CC 4.0 BY-SA 版权协议，转载请附上原文出处链接及本声明。
原文链接：https://blog.csdn.net/justry_deng/article/details/80972817
