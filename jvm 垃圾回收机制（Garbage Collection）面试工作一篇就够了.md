版权声明：转载必须注明本文转自晓_晨的博客：http://blog.csdn.net/niunai112

目录
目录
1.如何判定是否需要回收
引用计数算法
可达性算法
2.内存回收算法
标记清除算法
复制
标记压缩
分代收集算法
3.垃圾回收器
年轻代垃圾回收器
serial
parallel new
parallel sacvenge
老年代垃圾回收器
serial old
parallel old
concurrent mark sweep
G1
各收集器间的搭配关系
4.常用JVM调优参数
5.finalized
6.强软弱虚引用
强引用
软引用（SoftReference）
弱引用（WeakReference）
虚引用（PhantomReference）
7.参考引用
　 自动垃圾回收机制是java的一个特性，相较于c/c++程序员需要自己分配内存，在使用结束后自己回收内存而言，Java实在对程序员太友好了。Java的垃圾回收全部都是由虚拟机自动完成的，不需要程序员额外写啥代码。作为一个Java程序猿，学习GC是非常有必要的，根据项目特性，优化GC也是一个优秀程序猿的基本能力之一。下面就让我们来系统学习一下JVM 的GC吧。

1.如何判定是否需要回收
　JVM要实现自动回收垃圾，那么它就需要判断，哪些内存可以回收，哪些不行。下面两个算法是虚拟机用来判断的方法

引用计数算法
正如算法名，这个算法就是给对象增加一个引用计数，每当对象被别的对象引用时，就将该对象的引用计数加一。所以当一个对象的引用计数为0的话，那么就说明这个对象没有被任何对象使用，那么JVM就可以认为这个对象是可以回收的对象啦。 
　这个算法的优点是它的效率非常高，能非常直观的判断一个对象是否能被回收。 
　 
　这图上的E对象就要被回收了（没有被任何对象引用） 
　缺点也很明显，1.无法区分循环引用的对象（A引用了B，B引用了A），这2个对象的引用计数永远不可能为0，这2个对象无法被JVM回收。2.需要维护对象引用计数的值。

可达性算法
　为了解决引用计数算法的缺点，所以就有了可达性算法，这个算法就是通过 GC Roots 的对象作为起始点，然后通过这个节点往下找他引用的对象，直到最外层的叶子节点。当一个对象无法被 GC Roots 找到时，那么它就是可回收对象。 
　 
　　这图上的E对象就要被回收了（E对象不可达） 
GC Roots对象的包括如下几种： 
1).虚拟机栈中的本地变量表中的引用的对象 
2).方法区中的类静态属性引用的对象 
3).方法区中的常量引用的对象 
4).本地方法栈中JNI的引用的对象

2.内存回收算法
　当JVM已经找到要回收的对象的时候就要开始回收对象了，JVM回收对象有以下三大回收算法

标记清除算法
　这是最基础的垃圾回收算法，之所以说它是最基础的是因为它最容易实现，思想也是最简单的。标记-清除算法分为两个阶段：标记阶段和清除阶段。标记阶段的任务是标记出所有需要被回收的对象，清除阶段就是回收被标记的对象所占用的空间。 
 
　该算法会产生大量的内存碎片，可能会导致当JVM要分配大对象内存时，不能找到可用的内存空间的问题。

复制
　将内存划分为等大小的两块。每次只使用其中一块，当这一块内存满后将尚存活的对象复制到另一块上去，然后把满的那块内存清掉。 
 
　不会产生内存碎片了，但是可利用的内存将变成原来内存的一半，而且需要付出复制内存对象带来的消耗。

标记压缩
　结合了以上两个算法，为了避免缺陷而提出。先找出存活对象，然后把存活的对象移向内存的一端。然后清除端边界外的对象。 
　

分代收集算法
　分代收集法是目前大部分JVM所采用的方法，其核心思想是根据对象存活的不同生命周期将内存划分为不同的域，一般情况下将GC堆划分为新生代，老生代，永久代（元数据区）。因为不同的区域，其存储对象的特点不同，因此可以根据不同区域选择不同的算法。新生代的特点是每次垃圾回收时都有大量垃圾需要被回收，回收频率很高，老生代的特点是每次垃圾回收时只有少量对象需要被回收，回收频率很低。

3.垃圾回收器
年轻代垃圾回收器
serial
　单线程回收垃圾，它在进行垃圾收集时，必须暂停其他所有的工作线程，直到它收集完成。 
Serial收集器依然是虚拟机运行在Client模式下默认新生代收集器，对于运行在Client模式下的虚拟机来说是一个很好的选择。 


parallel new
　ParNew收集器其实就是Serial收集器的多线程版本，除了使用多线程进行垃圾收集之外，其余行为包括Serial收集器可用的所有控制参数、收集算法、Stop The Worl、对象分配规则、回收策略等都与Serial 收集器完全一样。 
　

　ParNew收集器是许多运行在Server模式下的虚拟机中首选新生代收集器，其中有一个与性能无关但很重要的原因是，除Serial收集器之外，目前只有ParNew它能与CMS收集器配合工作。

parallel sacvenge
　Parallel Scavenge收集器是一个新生代收集器，它也是使用复制算法的收集器，是并行的多线程收集器

　该收集器的目标是达到一个可控制的吞吐量（Throughput）。所谓吞吐量就是CPU用于运行用户代码的时间与CPU总消耗时间的比值，即 吞吐量=运行用户代码时间/（运行用户代码时间+垃圾收集时间）

　停顿时间越短就越适合需要与用户交互的程序，良好的响应速度能提升用户体验，而高吞吐量则可用高效率地利用CPU时间，尽快完成程序的运算任务，主要适合在后台运算而不需要太多交互的任务。

　Parallel Scavenge收集器提供两个参数用于精确控制吞吐量，分别是控制最大垃圾收起停顿时间的

　-XX:MaxGCPauseMillis参数以及直接设置吞吐量大小的-XX:GCTimeRatio参数

　Parallel Scavenge收集器还有一个参数：-XX:+UseAdaptiveSizePolicy。这是一个开关参数，当这个参数打开后，就不需要手工指定新生代的大小（-Xmn）、Eden与Survivor区的比例（-XX:SurvivorRatio）、晋升老年代对象年龄（-XX:PretenureSizeThreshold）等细节参数，只需要把基本的内存数据设置好（如-Xmx设置最大堆），然后使用MaxGCPauseMillis参数或GCTimeRation参数给虚拟机设立一个优化目标。

　自适应调节策略也是Parallel Scavenge收集器与ParNew收集器的一个重要区别

老年代垃圾回收器
serial old
老年代的垃圾回收器，它是一个单线程收集器，使用标记整理算法。


主要两大用途：

（1）在JDK1.5以及之前的版本中与Parallel Scavenge收集器搭配使用

（2）作为CMS收集器的后备预案，在并发收集发生Concurrent Mode Failure时使用

Serial Old收集器的工作工程

parallel old
Parallel Old 是Parallel Scavenge收集器的老年代版本，使用多线程和“标记-整理”算法。 


concurrent mark sweep
CMS(Concurrent Mark Sweep)收集器是一种以获取最短回收停顿时间为目标的收集器。目前很大一部分的Java应用集中在互联网站或者B/S系统的服务端上，这类应用尤其重视服务器的响应速度，希望系统停顿时间最短，以给用户带来较好的体验。CMS收集器就非常符合这类应用的需求

CMS收集器是基于“标记-清除”算法实现的。它的运作过程相对前面几种收集器来说更复杂一些，整个过程分为4个步骤：

1.初始标记，2.并发标记,3.重新标记，4.并发清除

其中，初始标记、重新标记这两个步骤仍然需要“Stop The World”.



CMS收集器主要优点：并发收集，低停顿。

CMS的缺点：

（1）CMS回收器采用的基础算法是Mark-Sweep。所有CMS不会整理、压缩堆空间。这样就会有一个问题：经过CMS收集的堆会产生空间碎片。 CMS不对堆空间整理压缩节约了垃圾回收的停顿时间，但也带来的堆空间的浪费。为了解决堆空间浪费问题，CMS回收器不再采用简单的指针指向一块可用堆空 间来为下次对象分配使用。而是把一些未分配的空间汇总成一个列表，当JVM分配对象空间的时候，会搜索这个列表找到足够大的空间来存下这个对象。

（2）CMS的另一个缺点是它需要更大的堆空间。因为CMS标记阶段应用程序的线程还是在执行的，那么就会有堆空间继续分配的情况，为了保证在CMS回 收完堆之前还有空间分配给正在运行的应用程序，必须预留一部分空间。也就是说，CMS不会在老年代满的时候才开始收集。相反，它会尝试更早的开始收集，已 避免上面提到的情况：在回收完成之前，堆没有足够空间分配！在JDK1.6中，CMS收集器的启动阀值已经提升至92%。CMS就开始行动了。 – XX:CMSInitiatingOccupancyFraction =n 来设置这个阀值。

（3）需要更多的CPU资源。从上面的图可以看到，为了让应用程序不停顿，CMS线程和应用程序线程并发执行，这样就需要有更多的CPU，单纯靠线程切 换是不靠谱的。并且，重新标记阶段，为空保证STW快速完成，也要用到更多的甚至所有的CPU资源。

G1
JAVA8之后广泛使用，G1 将整个对区域划分为若干个Region，每个Region的大小是2的倍数（1M,2M,4M,8M,16M,32M，通过设置堆的大小和Region数量计算得出。 
Region区域划分与其他收集类似，不同的是单独将大对象分配到了单独的region中，会分配一组连续的Region区域（Humongous start 和 humonous Contoinue 组成），所以一共有四类Region（Eden，Survior，Humongous和Old）， 
G1 作用于整个堆内存区域，设计的目的就是减少Full GC的产生。在Full GC过程中由于G1 是单线程进行，会产生较长时间的停顿。 
G1的OldGc标记过程可以和yongGc并行执行，但是OldGc一定在YongGc之后执行，即MixedGc在yongGC之后执行。

各收集器间的搭配关系
有连线则说明可以一起使用 


4.常用JVM调优参数
-XX:+PrintGC：每次GC时打印相关信息。 
-XX:+PrintGCDetails：每次GC时打印详细信息。 
-XX:+PrintGCTimeStamps：打印每次GC的时间戳。 
-Xms:设置JVM初始内存。此值可以设置与-Xmx相同，以避免每次垃圾回收完成后JVM重新分配内存。 
-Xmx:设置JVM最大堆内存 
-Xss:设置每个线程的栈大小。JDK5.0以后每个线程栈大小为1M，之前每个线程栈大小为256K。应当根据应用的线程所需内存大小进行调整。在相同物理内存下，减小这个值能生成更多的线程。但是操作系统对一个进程内的线程数还是有限制的，不能无限生成，经验值在3000~5000左右。需要注意的是：当这个值被设置的较大（例如>2MB）时将会在很大程度上降低系统的性能。 
-Xmn:设置年轻代大小。（NewSize+MaxNewSize） 
-XX:NewSize=1024m：设置年轻代初始值为1024M。 
-XX:MaxNewSize=1024m：设置年轻代最大值为1024M。 
-XX:PermSize=256m：设置持久代初始值为256M。 
-XX:MaxPermSize=256m：设置持久代最大值为256M。 
-XX:NewRatio=4：设置年轻代（包括1个Eden和2个Survivor区）与年老代的比值。表示年轻代比年老代为1:4。 
-XX:SurvivorRatio=4：设置年轻代中Eden区与Survivor区的比值。表示2个Survivor区（JVM堆内存年轻代中默认有2个大小相等的Survivor区）与1个Eden区的比值为2:4，即1个Survivor区占整个年轻代大小的1/6。 
-XX:MaxTenuringThreshold=7：表示一个对象如果在Survivor区（救助空间）移动了7次还没有被垃圾回收就进入年老代。如果设置为0的话，则年轻代对象不经过Survivor区，直接进入年老代，对于需要大量常驻内存的应用，这样做可以提高效率。如果将此值设置为一个较大值，则年轻代对象会在Survivor区进行多次复制，这样可以增加对象在年轻代存活时间，增加对象在年轻代被垃圾回收的概率，减少Full GC的频率，这样做可以在某种程度上提高服务稳定性。 
-XX:PretenureSizeThreshold:当对象的大小超过这个设置值时，直接进入老年代。 
-XX:+UseSerialGC：设置串行收集器。

ParallelGC relation 
-XX:+UseParallelGC：设置为并行收集器。此配置仅对年轻代有效。即年轻代使用并行收集，而年老代仍使用串行收集。 
-XX:+UseParallelOldGC：配置年老代垃圾收集方式为并行收集。JDK6.0才开始支持对年老代并行收集。 
-XX:+UseParNewGC：设置年轻代为并行收集。可与CMS收集同时使用。JDK5.0以上，JVM会根据系统配置自行设置，所以无需再设置此值。 
-XX:ParallelGCThreads：配置并行收集器的线程数，即：同时多少个线程一起进行垃圾回收。此值最好配置与处理器数目相等。 
-XX:MaxGCPauseMillis：设置每次年轻代垃圾回收的最长时间，如果无法满足此时间，JVM会自动调整年轻代大小，以满足此值。 
-XX:GCTimeRadio：可接受GC时间占比（目标吞吐量） 吞吐量=1-1/(1+N) 
MaxGCPauseMillis优先级高于GCTimeRadio

CMS GC relation 
-XX:+UseConcMarkSweepGC：设置年老代为并发收集。 
-XX:+CMSFullGCsBeforeCompaction：由于并发收集器不对内存空间进行压缩、整理，所以运行一段时间以后会产生“碎片”，使得运行效率降低。此值设置运行多少次GC以后对内存空间进行压缩、整理。 
-XX:+CMSParallelRemarkEnable 
-XX:+UseCMSCompactAtFullCollection：：打开对年老代的压缩。可能会影响性能，但是可以消除碎片

5.finalized
　finalize()的功能 : 一旦垃圾回收器准备释放对象所占的内存空间, 如果对象覆盖了finalize()并且函数体内不能是空的, 就会首先调用对象的finalize(), 然后在下一次垃圾回收动作发生的时候真正收回对象所占的空间.

　finalize()有一个特点就是: JVM始终只调用一次. 无论这个对象被垃圾回收器标记为什么状态, finalize()始终只调用一次. 但是程序员在代码中主动调用的不记录在这之内.

finalize()主要使用的方面:

　根据垃圾回收器的第2点可知, java垃圾回收器只能回收创建在堆中的java对象, 而对于不是这种方式创建的对象则没有方法处理, 这就需要使用finalize()对这部分对象所占的资源进行释放. 使用到这一点的就是JNI本地对象, 通过JNI来调用本地方法创建的对象只能通过finalize()保证使用之后进行销毁,释放内存 
充当保证使用之后释放资源的最后一道屏障, 比如使用数据库连接之后未断开,并且由于程序员的个人原因忘记了释放连接, 这时就只能依靠finalize()函数来释放资源.

尽量避免使用finalize(): 
　finalize()不一定会被调用, 因为java的垃圾回收器的特性就决定了它不一定会被调用 
就算finalize()函数被调用, 它被调用的时间充满了不确定性, 因为程序中其他线程的优先级远远高于执行finalize（）函数线程的优先级。也许等到finalize()被调用, 数据库的连接池或者文件句柄早就耗尽了. 
如果一种未被捕获的异常在使用finalize方法时被抛出，这个异常不会被捕获，finalize方法的终结过程也会终止，造成对象出于破坏的状态。被破坏的对象又很可能导致部分资源无法被回收, 造成浪费. 
finalize()和垃圾回收器的运行本身就要耗费资源, 也许会导致程序的暂时停止.

6.强软弱虚引用
强引用
　最普遍的一种引用方式，如String s = “123”，变量s就是字符串“123”的强引用，只要强引用存在，则垃圾回收器就不会回收这个对象。我们平时的引用基本都是强引用。（永不回收）

软引用（SoftReference）
　用于描述还有用但非必须的对象，如果内存不足，就回收。一般用于实现内存敏感的高速缓存，软引用可以和引用队列ReferenceQueue联合使用，如果软引用的对象被垃圾回收，JVM就会把这个软引用加入到与之关联的引用队列中。（不够才回收） 
　

    public static void main(String[] args) {
        int[] ints = new int[300 * 1024 * 1024];
        SoftReference softReference = new SoftReference(ints);
        ints = null;
        System.out.println(softReference.get());
        ints = new int[300 * 1024 * 1024];
        System.out.println(softReference.get());

    }
1
2
3
4
5
6
7
8
9
弱引用（WeakReference）
　在垃圾回收器线程扫描它所管辖的内存区域的过程中，一旦发现了只具有弱引用的对象，不管当前内存空间足够与否，都会回收它的内存。（遇到就回收）

    public static void main(String[] args) {

        String s = new String("123");

        ReferenceQueue<Object> objectReferenceQueue = new ReferenceQueue<>();
        WeakReference weakReference = new WeakReference<>(s, objectReferenceQueue);
        s = null;
        System.out.println(weakReference.get());
        System.gc();
        System.out.println(weakReference.get());

    }
1
2
3
4
5
6
7
8
9
10
11
12
虚引用（PhantomReference）
　就是形同虚设，与其他几种引用都不同，虚引用并不会决定对象的生命周期。如果一个对象仅持有虚引用，那么它就和没有任何引用一样，在任何时候都可能被垃圾回收器回收。 虚引用主要用来跟踪对象被垃圾回收器回收的活动。（遇到就回收，必须与ReferenceQueue一起使用） 
　虚引用与软引用和弱引用的一个区别在于：虚引用必须和引用队列 （ReferenceQueue）联合使用。当垃圾回收器准备回收一个对象时，如果发现它还有虚引用，就会在回收对象的内存之前，把这个虚引用加入到与之关联的引用队列中。

    public static void main(String[] args) {
        String s = new String("123");
        ReferenceQueue<String> objectReferenceQueue = new ReferenceQueue<>();
        PhantomReference phantomReference = new PhantomReference<>(s, objectReferenceQueue);
        System.out.println(phantomReference.get());
        s = null;
        System.gc();
        System.out.println(objectReferenceQueue.poll() == phantomReference);
        System.out.println(phantomReference.get());

    }
1
2
3
4
5
6
7
8
9
10
11
12
7.参考引用
Java Memory Model 
Java Garbage Collection Basics 
从实际案例聊聊Java应用的GC优化 
Understanding the Java Memory Model and Garbage Collection 
很不错的GC资料

最后感谢一下前人们的付出，所以我们现在才能如此快速的获取知识！！！
————————————————
版权声明：本文为CSDN博主「晓_晨」的原创文章，遵循 CC 4.0 BY-SA 版权协议，转载请附上原文出处链接及本声明。
原文链接：https://blog.csdn.net/niunai112/article/details/81071438
