get和post请求的区别 
你轻轻松松的给出了一个“标准答案”：

GET在浏览器回退时是无害的，而POST会再次提交请求。

GET产生的URL地址可以被Bookmark，而POST不可以。

GET请求会被浏览器主动cache，而POST不会，除非手动设置。

GET请求只能进行url编码，而POST支持多种编码方式。

GET请求参数会被完整保留在浏览器历史记录里，而POST中的参数不会被保留。

GET请求在URL中传送的参数是有长度限制的，而POST么有。

对参数的数据类型，GET只接受ASCII字符，而POST没有限制。

GET比POST更不安全，因为参数直接暴露在URL上，所以不能用来传递敏感信息。

GET参数通过URL传递，POST放在Request body中。

如果面试官问你，你这样回答的话

“很遗憾这不是我们想要的答案！”

如果想要知道get和post的区别，首先你要知道get和post是什么？

get和post是http协议中的两种发送请求的方法。

那http又是什么呢？

http是基于TCP/IP的关于数据如何在万维网中如何通信的协议。

http的底层是TCP/IP。所有get和post的底层也是TCP/IP，也就是说，get/post都是TCP链接。get和post能做的事情是一样的。你要给get加上request body，给post带上url参数，技术上是完全行得通的。

get和post还有一个重大的区别，假肚腩来说：

get产生一个TCP数据包；post产生两个TCP数据包。

复杂来说就是：

对于get方式的请求，浏览器会把http header和data一并发送出去，服务器响应那个200(返回数据)；

而对于post，浏览器先发送header，服务器响应100 continue，浏览器再发送data，服务器响应200ok( 返回数据)/

也就是谁，get只需要汽车跑一趟就把货送到了，二post得跑两趟，第一趟，先去和服务器打个招呼，第二趟才把货送过来。

因为post需要两步，时间上消耗得多一点，看起来get比post更有效。

1.get与post都有自己得语义，不饿能随便混用。

2.据研究，再网络环境好的情况下，发一次包得事件和发两次包得事件差别基本可以无视。而在网络环境差的情况下，两次包得TCP在验证数据包完整性下，有非常大的有点。

3.并不是所有浏览器都会在post发送两次包，Firefox就只发送一次。

作者：Dumbass_
链接：https://www.jianshu.com/p/c4732708bfd0
来源：简书
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。
