推荐WebSocket的三大理由：

1、采用全双工通信，摆脱传统HTTP轮询的窘境。
2、采用W3C国际标准，完美支持HTML5。
3、简单高效，容易上手。
学习目标
快速学会通过WebSocket编写简单聊天功能。

快速查阅
专题阅读：《SpringBoot 布道系列》

源码下载：SpringBoot-WebSocket-Chat

温馨提示：
1、WebSocket是HTML5开始提供的一种在单个 TCP 连接上进行全双工通讯的协议。在WebSocket API中，浏览器和服务器只需要做一个握手的动作，然后，浏览器和服务器之间就形成了一条快速通道。两者之间就直接可以数据互相传送。
2、浏览器通过 JavaScript 向服务器发出建立 WebSocket 连接的请求，连接建立以后，客户端和服务器端就可以通过 TCP 连接直接交换数据。
3、当你获取 Web Socket 连接后，你可以通过 send() 方法来向服务器发送数据，并通过 onmessage 事件来接收服务器返回的数据。

使用教程
一、打造 WebSocket 聊天客户端
温馨提示：得益于W3C国际标准的实现，我们在浏览器JS就能直接创建WebSocket对象，再通过简单的回调函数就能完成WebSocket客户端的编写，非常简单！接下来让我们一探究竟。

使用说明：
使用步骤：1、获取WebSocket客户端对象。
例如： var webSocket = new WebSocket(url);

使用步骤：2、获取WebSocket回调函数。
例如：webSocket.onmessage = function (event) {console.log('WebSocket收到消息：' + event.data);

事件类型	WebSocket回调函数	事件描述
open	webSocket.onopen	当打开连接后触发
message	webSocket.onmessage	当客户端接收服务端数据时触发
error	webSocket.onerror	当通信异常时触发
close	webSocket.onclose	当连接关闭时触发
使用步骤：3、发送消息给服务端
例如：webSokcet.send(jsonStr) 结合实际场景 本案例采用JSON字符串进行消息通信。

具体实现：
下面是本案例在线聊天的客户端实现的JS代码，附带详细注释。

<script>

    /**
     * WebSocket客户端
     *
     * 使用说明：
     * 1、WebSocket客户端通过回调函数来接收服务端消息。例如：webSocket.onmessage
     * 2、WebSocket客户端通过send方法来发送消息给服务端。例如：webSocket.send();
     */
    function getWebSocket() {
        /**
         * WebSocket客户端 PS：URL开头表示WebSocket协议 中间是域名端口 结尾是服务端映射地址
         */
        var webSocket = new WebSocket('ws://localhost:8080/chat');
        /**
         * 当服务端打开连接
         */
        webSocket.onopen = function (event) {
            console.log('WebSocket打开连接');
        };

        /**
         * 当服务端发来消息：1.广播消息 2.更新在线人数
         */
        webSocket.onmessage = function (event) {
            console.log('WebSocket收到消息：%c' + event.data, 'color:green');
            //获取服务端消息
            var message = JSON.parse(event.data) || {};
            var $messageContainer = $('.message-container');
            //喉咙发炎
            if (message.type === 'SPEAK') {
                $messageContainer.append(
                    '<div class="mdui-card" style="margin: 10px 0;">' +
                    '<div class="mdui-card-primary">' +
                    '<div class="mdui-card-content message-content">' + message.username + "：" + message.msg + '</div>' +
                    '</div></div>');
            }
            $('.chat-num').text(message.onlineCount);
            //防止刷屏
            var $cards = $messageContainer.children('.mdui-card:visible').toArray();
            if ($cards.length > 5) {
                $cards.forEach(function (item, index) {
                    index < $cards.length - 5 && $(item).slideUp('fast');
                });
            }
        };

        /**
         * 关闭连接
         */
        webSocket.onclose = function (event) {
            console.log('WebSocket关闭连接');
        };

        /**
         * 通信失败
         */
        webSocket.onerror = function (event) {
            console.log('WebSocket发生异常');

        };
        return webSocket;
    }

    var webSocket = getWebSocket();


    /**
     * 通过WebSocket对象发送消息给服务端
     */
    function sendMsgToServer() {
        var $message = $('#msg');
        if ($message.val()) {
            webSocket.send(JSON.stringify({username: $('#username').text(), msg: $message.val()}));
            $message.val(null);
        }

    }
    /**
     * 清屏
     */
    function clearMsg(){
      $(".message-container").empty();
    }

    /**
     * 使用ENTER发送消息
     */
    document.onkeydown = function (event) {
        var e = event || window.event || arguments.callee.caller.arguments[0];
        e.keyCode === 13 && sendMsgToServer();
    };
</script>
========================================================================

二、打造 WebSocket 聊天服务端
温馨提示：得益于SpringBoot提供的自动配置，我们只需要通过简单注解@ServerEndpoint就就能创建WebSocket服务端，再通过简单的回调函数就能完成WebSocket服务端的编写，比起客户端的使用同样非常简单！

使用说明：
首先在POM文件引入spring-boot-starter-websocket 、thymeleaf 、FastJson等依赖。

使用步骤：1、开启WebSocket服务端的自动注册。
【这里需要特别提醒：ServerEndpointExporter 是由Spring官方提供的标准实现，用于扫描ServerEndpointConfig配置类和@ServerEndpoint注解实例。使用规则也很简单：1.如果使用默认的嵌入式容器 比如Tomcat 则必须手工在上下文提供ServerEndpointExporter。2. 如果使用外部容器部署war包，则不要提供提供ServerEndpointExporter，因为此时SpringBoot默认将扫描服务端的行为交给外部容器处理。】

@Configuration
public class WebSocketConfig {
    @Bean
    public ServerEndpointExporter serverEndpointExporter() {

        return new ServerEndpointExporter();
    }
}
使用步骤：2、创建WebSocket服务端。
核心思路：

① 通过注解@ServerEndpoint来声明实例化WebSocket服务端。
② 通过注解@OnOpen、@OnMessage、@OnClose、@OnError 来声明回调函数。
事件类型	WebSocket服务端注解	事件描述
open	@OnOpen	当打开连接后触发
message	@OnMessage	当接收客户端信息时触发
error	@OnError	当通信异常时触发
close	@OnClose	当连接关闭时触发
③ 通过ConcurrentHashMap保存全部在线会话对象。
@Component
@ServerEndpoint("/chat")//标记此类为服务端
public class WebSocketChatServer {

    /**
     * 全部在线会话  PS: 基于场景考虑 这里使用线程安全的Map存储会话对象。
     */
    private static Map<String, Session> onlineSessions = new ConcurrentHashMap<>();


    /**
     * 当客户端打开连接：1.添加会话对象 2.更新在线人数
     */
    @OnOpen
    public void onOpen(Session session) {
        onlineSessions.put(session.getId(), session);
        sendMessageToAll(Message.jsonStr(Message.ENTER, "", "", onlineSessions.size()));
    }

    /**
     * 当客户端发送消息：1.获取它的用户名和消息 2.发送消息给所有人
     * <p>
     * PS: 这里约定传递的消息为JSON字符串 方便传递更多参数！
     */
    @OnMessage
    public void onMessage(Session session, String jsonStr) {
        Message message = JSON.parseObject(jsonStr, Message.class);
        sendMessageToAll(Message.jsonStr(Message.SPEAK, message.getUsername(), message.getMsg(), onlineSessions.size()));
    }

    /**
     * 当关闭连接：1.移除会话对象 2.更新在线人数
     */
    @OnClose
    public void onClose(Session session) {
        onlineSessions.remove(session.getId());
        sendMessageToAll(Message.jsonStr(Message.QUIT, "", "下线了！", onlineSessions.size()));
    }

    /**
     * 当通信发生异常：打印错误日志
     */
    @OnError
    public void onError(Session session, Throwable error) {
        error.printStackTrace();
    }

    /**
     * 公共方法：发送信息给所有人
     */
    private static void sendMessageToAll(String msg) {
        onlineSessions.forEach((id, session) -> {
            try {
                session.getBasicRemote().sendText(msg);
            } catch (IOException e) {
                e.printStackTrace();
            }
        });
    }

}
④ 通过会话对象 javax.websocket.Session 来发消息给客户端。
/**
 * WebSocket 聊天消息类
 */
package com.hehe.chat;

import com.alibaba.fastjson.JSON;

/**
 * WebSocket 聊天消息类
 */
public class Message {

    public static final String ENTER = "ENTER";
    public static final String SPEAK = "SPEAK";
    public static final String QUIT = "QUIT";

    private String type;//消息类型

    private String username; //发送人

    private String msg; //发送消息

    private int onlineCount; //在线用户数

    public static String jsonStr(String type, String username, String msg, int onlineTotal) {
        return JSON.toJSONString(new Message(type, username, msg, onlineTotal));
    }

    public Message(String type, String username, String msg, int onlineCount) {
        this.type = type;
        this.username = username;
        this.msg = msg;
        this.onlineCount = onlineCount;
    }

    //这里省略get/set方法 请自行补充
}

三、WebSocket在线聊天案例的视频演示
1、源码下载

至此，我们完成了客户端和服务端的编码，由于篇幅有限，本教程的页面代码并未完整贴上，想要完整的体验效果请在Github下载源码。传送门：springboot-websocket-chat

2、视频演示

上面一顿操作猛如虎，实际到底是啥样子呢，接下来由哈士奇童鞋为我们演示最终版的在线聊天案例：


ws-gif.gif
四、全文总结
1、使用WebSocket用于实时双向通讯的场景，常见的如聊天室、跨系统消息推送等。

2、创建WebSocket客户端使用JS内置对象+回调函数+send方法发送消息。

3、创建WebSocket服务端使用注解声明实例+使用注解声明回调方法+使用Session发送消息。

作者：yizhiwazi
链接：https://www.jianshu.com/p/55cfc9fcb69e
来源：简书
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。
