看到NIO一堆繁杂的API瞬间就不想学这个NIO了，但是我们可以先从模糊的认识上先理解下NIO和IO的区别。



初识IO
特点：采用面向流的操作。IO流是没有缓存的概念，所以就需要每次从流中一个一个字节的去读或者每次从流中一个一个字节的去写。而且我们知道流本身是单工的数据通信。IO可以分为文件IO和网络IO。文件IO是中的read和write都是阻塞的，网络IO中的read、write、accept等都是阻塞的。由于IO流是阻塞的又叫做BIO。
IO文件流阻塞例子：

public static void writeByBIO(String path, String context) {
        FileOutputStream fos = null;
        
        try {
            fos = new FileOutputStream(path, true);
            
            for (int i = 0 ; i < 3 ; i++) {
                fos.write((context + "\n").getBytes());
            }
        } catch (FileNotFoundException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            try {
                if (fos != null) {
                    fos.close();
                }
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
}

public static void readByBIO(String path) {
        FileInputStream fis = null;
        
        try {
            fis = new FileInputStream(path);
            int length;
            
            while ((length = fis.read()) != -1) {
                System.out.print((char) length);
            }
        } catch (FileNotFoundException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            try {
                if (fis != null) {
                    fis.close();
                }
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
}
测试：

public static void main(String[] args) {
        new Thread(new Runnable() {

            @Override
            public void run() {
                writeByBIO("F:\\Hello.log", "1");
            }
        }).start();
        
        new Thread(new Runnable() {

            @Override
            public void run() {
                writeByBIO("F:\\Hello.log", "2");
            }
        }).start();
        
        new Thread(new Runnable() {

            @Override
            public void run() {
                readByBIO("F:\\Hello.log");
            }
        }).start();
}
BIO.png

说明：可以发现当第一个线程写入的时候，第二个线程和第三个线程虽然已经启动，但是处于线程阻塞挂起的状态。虽然多线程调度具有随机性，CPU按照时间分片，但是由于其本身的阻塞的特点，不会出现脏数据；虽然简单的顺序执行得到保证，但是扩展性不高、效率不高，很容易成为系统瓶颈。

初识NIO
     采用面向缓冲区的操作。NIO采用缓冲区的概念，先将数据都缓存在缓冲区中，缓冲区采用开始读写的位置position、读写结束位置limit、缓冲区最大容量capacity等标记来设计。NIO还采用了通道的概念，可以进行全双工数据通信。同样NIO分为文件NIO和网络NIO。
     文件NIO中的通道是一种阻塞式。
文件NIO阻塞例子：

public static void writeByNIO(String path, String context) {
        FileOutputStream fos = null;
        FileChannel fc = null;
        
        try {
            fos = new FileOutputStream(path, true);
            fc = fos.getChannel();
            ByteBuffer bb = ByteBuffer.allocate(1024 * 1024 * 5);
            
            for (int i = 0 ; i < 3 ; i++) {
                bb.put((context + "\n").getBytes());
            }
            
            bb.flip();
            fc.write(bb);
            bb.clear();
        } catch (FileNotFoundException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            try {
                if (fc != null) {
                    fc.close();
                }
            } catch (IOException e) {
                e.printStackTrace();
            }
            
            try {
                if (fos != null) {
                    fos.close();
                }
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
}
    
public static void readByNIO(String path) {
        FileInputStream fis = null;
        FileChannel fc = null;
        
        try {
            fis = new FileInputStream(path);
            fc = fis.getChannel();
            ByteBuffer bb = ByteBuffer.allocate(1024 * 1024 * 5);
            int length;
            
            while ((length = fc.read(bb)) != -1) {
                bb.flip();
                System.out.println(new String(bb.array(), 0 , length));
                bb.clear();
            }
        } catch (FileNotFoundException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            try {
                if (fc != null) {
                    fc.close();
                }
            } catch (IOException e) {
                e.printStackTrace();
            }
            
            try {
                if (fis != null) {
                    fis.close();
                }
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
}
测试：

public static void main(String[] args) {
        new Thread(new Runnable() {

            @Override
            public void run() {
                writeByNIO("F:\\Hello.log", "1");
            }
        }).start();
        
        new Thread(new Runnable() {

            @Override
            public void run() {
                writeByNIO("F:\\Hello.log", "2");
            }
        }).start();
        
        new Thread(new Runnable() {

            @Override
            public void run() {
                readByNIO("F:\\Hello.log");
            }
        }).start();
}
NIO.png
网络NIO
     网络NIO的ServerSocketChannel可以通过configureBlocking方法来设置非阻塞。NIO读写分为准备读写和真正读写，而在准备读写的阶段：将通道中SelectionKey中接收请求事件、读事件、写事件、连接事件这些注册到Selector选择器上，此时每个键对应的通道已经做好了I/O操作的准备（注册这个操作本身是非阻塞的）。
NIO一个重要的特点是：socket主要的读、写、注册和接收函数，在等待就绪阶段都是非阻塞的，真正的I/O操作是同步阻塞的（消耗CPU但性能非常高）。 （这段话来源于https://zhuanlan.zhihu.com/p/23488863）

     NIO是采用了Selector选择器的概念，对于前面准备好I/O操作的通道的键。选择器使用select方法进行死循环轮询使用的键SelectionKey找对应的事件和通道。因为select方法是阻塞的并且进行死循环，所以只能是单线程的操作。然后通过SelectionKey中的方法测试通道是否可读可写等操作再进行对缓冲区读写。

     流和缓冲区对比：流就是每次去购物，一个东西一个东西去拿到收银台付款。而缓冲区就是将每个东西都放到购物车，然后一次拿到收银台付款。

区别
IO流的特点就是建立连接，然后去读写就行了。对于是否可读可写通过阻塞等方式来处理。NIO的特点就是将读写拆分为准备读写和真正读写两个方面，在准备读写阶段采用非阻塞的方式来处理，而到真正读写的时候实际上还是阻塞的方式。



作者：sunpy
链接：https://www.jianshu.com/p/948f60a8e64e
来源：简书
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。
