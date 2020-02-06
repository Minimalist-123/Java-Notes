 1、这两种请求是HTTP协议中常用的请求,Get请求把表单的数据显式地放在URI中,并且对长度和数据值编码有所限制.Post请求把表单数据放在HTTP请求体中,并且没有长度限制.

 2、get吧数据放在网址中，例如：http://www.abc.com/index.php?a=1&b=2 其中?a=1&b=2就是get数据，并且连http://www.abc.com/index.php长度限制在1024个字。

post则是把数据放到http请求中，例如还是传输a=1&b=2，可是网址还是http://www.abc.com/index.php，想看到post数据GET在浏览器回退时是无害的，而POST会再次提交请求。可以用一些抓包工具，如Wireshark等



 3、GET产生的URL地址可以被Bookmark，而POST不可以。



 4、GET请求会被浏览器主动cache，而POST不会，除非手动设置。



 5、GET请求只能进行url编码，而POST支持多种编码方式。



 6、GET请求参数会被完整保留在浏览器历史记录里，而POST中的参数不会被保留。



 7、GET请求在URL中传送的参数是有长度限制的，而POST么有。



 8、对参数的数据类型，GET只接受ASCII字符，而POST没有限制。



 9、GET比POST更不安全，因为参数直接暴露在URL上，所以不能用来传递敏感信息。



 10、GET参数通过URL传递，POST放在Request body中。





GET和POST还有一个重大区别，简单的说：

GET产生一个TCP数据包;POST产生两个TCP数据包。

长的说：

对于GET方式的请求，浏览器会把http header和data一并发送出去，服务器响应200(返回数据);

而对于POST，浏览器先发送header，服务器响应100 continue，浏览器再发送data，服务器响应200 ok(返回数据)。

0人点赞
日记本


作者：han2019
链接：https://www.jianshu.com/p/d547884315fe
来源：简书
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。
