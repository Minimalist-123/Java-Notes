由 匿名 (未验证) 提交于 2019-12-03 00:18:01
Model和ModelAndView了，对于MVC框架，控制器Controller执行业务逻辑，用于产生模型数据Model，而视图View用于渲染模型数据。

使用Model和ModelAndView这两个类在spring的视图解析时作用以及区别。 

区别：

1、Model只是用来传输数据的，并不会进行业务的寻址。那如何设置返回地址，一般用controller的String返回值，把地址以字符串形式返回。

2、ModelAndView 却是可以进行业务寻址的，通过setViewName来设置跳转地址

3、两者还有一个最大的区别，Model是每一次请求可以自动创建，但是ModelAndView 是需要我们自己去new的。

个人理解

ModelAndView 就是将 servlet的跳转方式forward封装的一个方法。

RequestDispatcher dispatcher = request.getRequestDispatcher("/a.jsp");

dispatcher .forward(request, response);


文章来源: Model、ModelMap和ModelAndView使用区别理解

【原文链接】：https://www.e-learn.cn/content/qita/583953
