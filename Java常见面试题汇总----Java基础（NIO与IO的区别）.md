NIO与IO的区别
  NIO即New IO，这个库是在JDK1.4中才引入的。NIO和IO有相同的作用和目的，但实现方式不同，NIO主要用到的是块，所以NIO的效率要比IO高很多。在Java API中提供了两套NIO，一套是针对标准输入输出NIO，另一套就是网络编程NIO。
  NIO和IO的主要区别，下表总结了Java IO和NIO之间的主要区别：

IO	NIO
面向流	面向缓冲
阻塞IO	非阻塞IO
无	选择器


1、面向流与面向缓冲
  Java IO和NIO之间第一个最大的区别是，IO是面向流的，NIO是面向缓冲区的。 Java IO面向流意味着每次从流中读一个或多个字节，直至读取所有字节，它们没有被缓存在任何地方。此外，它不能前后移动流中的数据。如果需要前后移动从流中读取的数据，需要先将它缓存到一个缓冲区。Java NIO的缓冲导向方法略有不同。数据读取到一个它稍后处理的缓冲区，需要时可在缓冲区中前后移动。这就增加了处理过程中的灵活性。但是，还需要检查是否该缓冲区中包含所有您需要处理的数据。而且，需确保当更多的数据读入缓冲区时，不要覆盖缓冲区里尚未处理的数据。

2、阻塞与非阻塞IO
  Java IO的各种流是阻塞的。这意味着，当一个线程调用read() 或 write() 时，该线程被阻塞，直到有一些数据被读取，或数据完全写入。该线程在此期间不能再干任何事情了。Java NIO的非阻塞模式，使一个线程从某通道发送请求读取数据，但是它仅能得到目前可用的数据，如果目前没有数据可用时，就什么都不会获取，而不是保持线程阻塞，所以直至数据变的可以读取之前，该线程可以继续做其他的事情。非阻塞写也是如此。一个线程请求写入一些数据到某通道，但不需要等待它完全写入，这个线程同时可以去做别的事情。线程通常将非阻塞IO的空闲时间用于在其它通道上执行IO操作，所以一个单独的线程现在可以管理多个输入和输出通道（channel）。

3、选择器（Selectors）
  Java NIO的选择器允许一个单独的线程来监视多个输入通道，你可以注册多个通道使用一个选择器，然后使用一个单独的线程来“选择”通道：这些通道里已经有可以处理的输入，或者选择已准备写入的通道。这种选择机制，使得一个单独的线程很容易来管理多个通道。


18.1、NIO和IO适用场景
  NIO是为弥补传统IO的不足而诞生的，但是尺有所短寸有所长，NIO也有缺点，因为NIO是面向缓冲区的操作，每一次的数据处理都是对缓冲区进行的，那么就会有一个问题，在数据处理之前必须要判断缓冲区的数据是否完整或者已经读取完毕，如果没有，假设数据只读取了一部分，那么对不完整的数据处理没有任何意义。所以每次数据处理之前都要检测缓冲区数据。
  那么NIO和IO各适用的场景是什么呢？
  如果需要管理同时打开的成千上万个连接，这些连接每次只是发送少量的数据，例如聊天服务器，这时候用NIO处理数据可能是个很好的选择。
  而如果只有少量的连接，而这些连接每次要发送大量的数据，这时候传统的IO更合适。使用哪种处理数据，需要在数据的响应等待时间和检查缓冲区数据的时间上作比较来权衡选择。


18.2、Java NIO 总览
  Java NIO的三个核心基础组件，Channels、Buffers、Selectors。其余的诸如Pipe，FileLcok都是在使用以上三个核心组件时帮助更好使用的工具类。

一、Channels和Buffers的关系
  所有的IO操作在NIO中都是以Channel开始的。一个Channel就像一个流，NIO Channel和流很近似但是也有一些不同。
  1）、你既可以读取也可以写入到Channel，流只能读取或者写入，inputStream和outputStream。
  2）、Channel可以异步地读和写。
  3）、channel永远都是从一个buffer中读或者写入到一个buffer中去。
  

  基本的Channel实现有以下这些：
  1）、FileChannel：向文件当中读写数据；
  2）、DatagramChannel：通过UDP协议向网络读写数据；
  3）、SocketChannel：通过TCP协议向网络读写数据；
  4）、ServerSocketChannel：以一个web服务器的形式，监听到来的TCP连接，对每个连接建立一个SocketChannel。
  涵盖了UDP，TCP以及文件的IO操作。
  核心的buffer实现有这些：ByteBuffer、CharBuffer、DoubleBuffer、FloatBuffer、IntBuffer、LongBuffer、ShortBuffer，涵盖了所有的基本数据类型（4类8种，除了Boolean）。也有其他的buffer如MappedByteBuffer。
一个简单的channel例子：使用一个FileChannel将数据读入一个buffer。

RandomAccessFile aFile = new RandomAccessFile("data/nio-data.txt", "rw");
FileChannel inChannel = aFile.getChannel();

ByteBuffer buf = ByteBuffer.allocate(48);
int bytesRead = inChannel.read(buf);
while (bytesRead != -1) {
    System.out.println("Read " + bytesRead);
    buf.flip();

    while(buf.hasRemaining()){
        System.out.print((char) buf.get());
    }

    buf.clear();
    bytesRead = inChannel.read(buf);
}
aFile.close();
buf.flip()的意思是读写转换，首先你读入一个buffer，然后你flip，转换读写，然后再从buffer中读出。


二、NIO buffer
  NIO buffer在与NIO Channel交互时使用，数据从Channel中读取出来放入buffer，或者从buffer中读取出来写入Channel。
  buffer就是一块内存，你可以写入数据，并且在之后读取它。这块内存被包装成NIO buffer对象，它提供了一些方法来让你更简单地操作内存。
  buffer的基本使用，使用buffer读写数据基本上分为以下4部操作：
  1）、将数据写入buffer
  2）、调用buffer.flip()
  3）、将数据从buffer中读取出来
  4）、调用buffer.clear()或者buffer.compact()
  在写buffer的时候，buffer会跟踪写入了多少数据，需要读buffer的时候，需要调用flip()来将buffer从写模式切换成读模式，读模式中只能读取写入的数据，而非整个buffer。
  当数据都读完了，你需要清空buffer以供下次使用，可以有2种方法来操作：调用clear() 或者 调用compact()。
  区别：clear方法清空整个buffer，compact方法只清除你已经读取的数据，未读取的数据会被移到buffer的开头，此时写入数据会从当前数据的末尾开始。

// 创建一个容量为48的ByteBuffer
ByteBuffer buf = ByteBuffer.allocate(48);
// 从channel中读（取数据然后写）入buffer
int bytesRead = inChannel.read(buf); 
// 下面是读取buffer
while (bytesRead != -1) {
    buf.flip();  // 转换buffer为读模式
    System.out.print((char) buf.get()); // 一次读取一个byte
    buf.clear();  //清空buffer准备下一次写入
}
1、buffer的Capacity，Position和Limit
  buffer有3个属性需要熟悉以理解buffer的工作原理：
  容量（Capacity）：缓冲区能够容纳的数据元素的最大数量。容量在缓冲区创建时被设定，并且永远不能被改变。
  上界（Limit）：写模式中等价于buffer的大小，即capacity；读模式中为当前缓冲区中一共有多少数据，即可读的最大位置。这意味着当调用filp()方法切换成读模式时，limit的值变成position的值，而position重新指向0。
  位置（Position）：下一个要被读或写的元素的位置。初始化为0，buffer满时，position最大值为capacity-1。切换成读模式的时候，position指向0。Position会自动由相应的 get( )和 put( )函数更新。
  position和limit的值在读/写模式中是不一样的。capacity的值永远表示buffer的大小。
  下图解释了在读/写模式中Capacity，Position和Limit的意思。
  


2、创建一个buffer
  获得一个buffer 之前必须先分配一块内存，每个buffer类都有一个静态方法allocate() 来做这件事。
  下例为创建一个容量为48byte的ByteBuffer：
  ByteBuffer buf = ByteBuffer.allocate(48);
  创建一个1024个字符的CharBuffer
  CharBuffer buf = CharBuffer.allocate(1024);

3、将数据写入buffer
  写入buffer的方法有2种：
    1）、从一个Channel中写入buffer。
    2）、调用buffer的put()方法来自行写入数据。
  例：
  int bytesRead = inChannel.read(buf); // 从channel读入buffer
  buf.put(127); // 自行写入buffer
  put方法有很多的重载形式。以供你用各种不同的方法写入buffer中，比如从一个特定的position，或者写入一个array。

4、flip()
  flip方法将写模式切换成读模式，调用flip()方法会将limit设置为position，将position设置回0。
  换句话说，position标志着写模式中写到哪里，切换成读模式之后，limit标志着之前写到哪里，也就是现在能读到哪里。

5、从buffer中读取数据
  有2种方法可以从buffer中读取数据。
    1）、从buffer中读取数据到channel中。
    2）、使用buffer的get()方法自行从buffer中读出数据。
  例子：
  // 从buffer中读取数据到channel中
  int bytesWritten = inChannel.write(buf);
  // 使用buffer的get()方法自行从buffer中读出数据
  byte aByte = buf.get();
  get方法有很多的重载形式。以供你用各种不同的方法读取buffer中的数据。例如从特定位置读取数据，或者读一个数组出来。

6、rewind()
  rewind()方法将position设置为0，但是不会动buffer里的数据，这样可以从头开始重新读取数据，limit的值不会变，这意味着limit依旧标志着能读多少数据。

7、clear()和compact()
  当你读完所有的数据想要重新写入数据时，你可以调用clear或者compact方法。
  当你调用clear()方法的时候，position被设置为0，limit被设置为capacity，换句话说，buffer的数据虽然都还在，但是buffer被初始化了，处于可以被重写的状态。这也就意味着如果buffer中还有没被读取的数据，在执行clear之后，你无法知道数据读到哪儿了，剩下的数据还有多少。
  如果还有没有读完的数据，但是你想先写数据，可以用compact()方法，这样未读数据会放在buffer前端，可以在未读数据之后跟着写新的数据。compact()会复制未读数据到buffer前端，然后设置position为未读数据单位后面紧跟的位置。limit还是设置为capacity，这和clear是一样的。现在buffer处于可以写的状态，但是不会覆盖之前未读完的数据。

8、mark()和reset()
  你可以通过调用buffer.mark()来mark一个buffer中给定的位置。然后你就可以用buffer.reset()方法来将position设置回之前mark的位置。
  例子：
  buffer.mark();
  // 调用buffer.get()方法若干次，e.g. 比如在做parsing的时候
  buffer.reset(); //set position back to mark.

9、equals() 和 compareTo()
  使用这2种方法能够比较2个buffer。
  equals()方法：用于判断2个buffer是否相等，2个buffer是equal的，当它们：
  1）、是同一种数据类型的buffer。
  2）、buffer中未读取的bytes，chars等数据个数是一样的，即（limit-position）相等，capacity不需要相等，剩余数据的索引也不需要相等。
  3）、未读取的bytes，chars等内容是一模一样的，即各自[position，limit-1]索引的数据要完全相等。
  如你所见，equals()方法只比较buffer的部分内容，而不是buffer中所有的数据，事实上，它只比较buffer中剩余的元素是否一样。

compareTo()
  compareTo()方法：比较两个buffer的剩余元素（字节，字符等），用于例如： 排序。
  在下列情况下，缓冲区被认为比另一个缓冲区“小”：
  比较是针对每个缓冲区你剩余数据（从 position 到 limit）进行的，与它们在 equals() 中的方式相同，直到不相等的元素被发现或者到达缓冲区的上界。如果一个缓冲区在不相等元素发现前已经被耗尽，较短的缓冲区被认为是小于较长的缓冲区。


三、NIO Selectors
  Selector允许一个线程来监视多个Channel，这在当你的应用建立了多个连接，但是每个连接吞吐量都较小的时候是可行的。例如：一个聊天服务器。图为一个线程使用Selector处理三个Channel。



  要使用一个Selector，你要先注册这个Selector的Channels。然后你调用Selector的select()方法。这个方法会阻塞，直到它注册的Channels当中有一个准备好了的事件发生了。当select()方法返回的时候，线程可以处理这些事件，如新的连接的到来，数据收到了等。

作者：从菜鸟到老菜鸟
链接：https://www.jianshu.com/p/3ea69e682b2a
来源：简书
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。
