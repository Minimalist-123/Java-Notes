1. 概述
垃圾回收（Garbage Collection，GC），顾名思义就是释放垃圾占用的空间，防止内存泄露。有效的使用可以使用的内存，对内存堆中已经死亡的或者长时间没有使用的对象进行清除和回收。

2. 垃圾判断算法
2.1 引用计数法
给每个对象添加一个计数器，当有地方引用该对象时计数器加1，当引用失效时计数器减1。用对象计数器是否为0来判断对象是否可被回收。缺点：无法解决循环引用的问题。


先创建一个字符串，String m = new String("jack");，这时候 "jack" 有一个引用，就是m。然后将m设置为null，这时候 "jack" 的引用次数就等于 0 了，在引用计数算法中，意味着这块内容就需要被回收了。


引用计数算法是将垃圾回收分摊到整个应用程序的运行当中了，而不是在进行垃圾收集时，要挂起整个应用的运行，直到对堆中所有对象的处理都结束。因此，采用引用计数的垃圾收集不属于严格意义上的Stop-The-World的垃圾收集机制。

看似很美好，但我们知道JVM的垃圾回收就是Stop-The-World的，那是什么原因导致我们最终放弃了引用计数算法呢？看下面的例子。

public class ReferenceCountingGC {

  public Object instance;

  public ReferenceCountingGC(String name) {
  }

  public static void testGC(){

    ReferenceCountingGC a = new ReferenceCountingGC("objA");
    ReferenceCountingGC b = new ReferenceCountingGC("objB");

    a.instance = b;
    b.instance = a;

    a = null;
    b = null;
  }
}
我们可以看到，最后这2个对象已经不可能再被访问了，但由于他们相互引用着对方，导致它们的引用计数永远都不会为0，通过引用计数算法，也就永远无法通知GC收集器回收它们。

2.2 可达性分析算法
通过GC ROOT的对象作为搜索起始点，通过引用向下搜索，所走过的路径称为引用链。通过对象是否有到达引用链的路径来判断对象是否可被回收（可作为GC ROOT的对象：虚拟机栈中引用的对象，方法区中类静态属性引用的对象，方法区中常量引用的对象，本地方法栈中JNI引用的对象）


通过可达性算法，成功解决了引用计数所无法解决的循环依赖问题，只要你无法与GC Root建立直接或间接的连接，系统就会判定你为可回收对象。那这样就引申出了另一个问题，哪些属于GC Root。

Java内存区域中可以作为GC ROOT的对象：

虚拟机栈中引用的对象
public class StackLocalParameter {

  public StackLocalParameter(String name) {}

  public static void testGC() {
    StackLocalParameter s = new StackLocalParameter("localParameter");
    s = null;
  }
}
此时的s，即为GC Root，当s置空时，localParameter对象也断掉了与GC Root的引用链，将被回收。

方法区中类静态属性引用的对象
public class MethodAreaStaicProperties {

  public static MethodAreaStaicProperties m;

  public MethodAreaStaicProperties(String name) {}

  public static void testGC(){
    MethodAreaStaicProperties s = new MethodAreaStaicProperties("properties");
    s.m = new MethodAreaStaicProperties("parameter");
    s = null;
  }
}
此时的s，即为GC Root，s置为null，经过GC后，s所指向的properties对象由于无法与GC Root建立关系被回收。而m作为类的静态属性，也属于GC Root，parameter 对象依然与GC root建立着连接，所以此时parameter对象并不会被回收。

方法区中常量引用的对象
public class MethodAreaStaicProperties {

  public static final MethodAreaStaicProperties m = MethodAreaStaicProperties("final");

  public MethodAreaStaicProperties(String name) {}

  public static void testGC() {
    MethodAreaStaicProperties s = new MethodAreaStaicProperties("staticProperties");
    s = null;
  }
}
m即为方法区中的常量引用，也为GC Root，s置为null后，final对象也不会因没有与GC Root建立联系而被回收。

本地方法栈中引用的对象

任何native接口都会使用某种本地方法栈，实现的本地方法接口是使用C连接模型的话，那么它的本地方法栈就是C栈。当线程调用Java方法时，虚拟机会创建一个新的栈帧并压入Java栈。然而当它调用的是本地方法时，虚拟机会保持Java栈不变，不再在线程的Java栈中压入新的帧，虚拟机只是简单地动态连接并直接调用指定的本地方法。


3. 垃圾回收算法
在确定了哪些垃圾可以被回收后，垃圾收集器要做的事情就是开始进行垃圾回收，但是这里面涉及到一个问题是：如何高效地进行垃圾回收。这里我们讨论几种常见的垃圾收集算法的核心思想。

3.1 标记-清除算法

标记清除算法（Mark-Sweep）是最基础的一种垃圾回收算法，它分为2部分，先把内存区域中的这些对象进行标记，哪些属于可回收标记出来，然后把这些垃圾拎出来清理掉。就像上图一样，清理掉的垃圾就变成未使用的内存区域，等待被再次使用。但它存在一个很大的问题，那就是内存碎片。

上图中等方块的假设是2M，小一些的是1M，大一些的是4M。等我们回收完，内存就会切成了很多段。我们知道开辟内存空间时，需要的是连续的内存区域，这时候我们需要一个2M的内存区域，其中有2个1M是没法用的。这样就导致，其实我们本身还有这么多的内存的，但却用不了。

3.2 复制算法

复制算法（Copying）是在标记清除算法基础上演化而来，解决标记清除算法的内存碎片问题。它将可用内存按容量划分为大小相等的两块，每次只使用其中的一块。当这一块的内存用完了，就将还存活着的对象复制到另外一块上面，然后再把已使用过的内存空间一次清理掉。保证了内存的连续可用，内存分配时也就不用考虑内存碎片等复杂情况。复制算法暴露了另一个问题，例如硬盘本来有500G，但却只能用200G，代价实在太高。

3.3 标记-整理算法

标记-整理算法标记过程仍然与标记-清除算法一样，但后续步骤不是直接对可回收对象进行清理，而是让所有存活的对象都向一端移动，再清理掉端边界以外的内存区域。

标记整理算法解决了内存碎片的问题，也规避了复制算法只能利用一半内存区域的弊端。标记整理算法对内存变动更频繁，需要整理所有存活对象的引用地址，在效率上比复制算法要差很多。一般是把Java堆分为新生代和老年代，这样就可以根据各个年代的特点采用最适当的收集算法。

3.4 分代收集算法
分代收集算法分代收集算法严格来说并不是一种思想或理论，而是融合上述3种基础的算法思想，而产生的针对不同情况所采用不同算法的一套组合拳，根据对象存活周期的不同将内存划分为几块。

在新生代中，每次垃圾收集时都发现有大批对象死去，只有少量存活，那就选用复制算法，只需要付出少量存活对象的复制成本就可以完成收集。

在老年代中，因为对象存活率高、没有额外空间对它进行分配担保，就必须使用标记-清理算法或者标记-整理算法来进行回收。

4. 内存区域与回收策略
对象的内存分配，往大方向讲，就是在堆上分配（但也可能经过JIT编译后被拆散为标量类型并间接地栈上分配），对象主要分配在新生代的Eden区上，如果启动了本地线程分配缓冲，将按线程优先在TLAB上分配。少数情况下也可能会直接分配在老年代中（大对象直接分到老年代），分配的规则并不是百分百固定的，其细节取决于当前使用的是哪一种垃圾收集器组合，还有虚拟机中与内存相关的参数的设置。

4.1 对象优先在Eden分配
大多数情况下，对象会在新生代Eden区中分配。当Eden区没有足够空间进行分配时，虚拟机会发起一次 Minor GC。Minor GC相比Major GC更频繁，回收速度也更快。通过Minor GC之后，Eden区中绝大部分对象会被回收，而那些存活对象，将会送到Survivor的From区（若From区空间不够，则直接进入Old区） 。

4.2 Survivor区
Survivor区相当于是Eden区和Old区的一个缓冲，类似于我们交通灯中的黄灯。Survivor又分为2个区，一个是From区，一个是To区。每次执行Minor GC，会将Eden区中存活的对象放到Survivor的From区，而在From区中，仍存活的对象会根据他们的年龄值来决定去向。（From Survivor和To Survivor的逻辑关系会发生颠倒： From变To ， To变From，目的是保证有连续的空间存放对方，避免碎片化的发生）

4.2.1 Survivor区存在的意义
如果没有Survivor区，Eden区每进行一次Minor GC，存活的对象就会被送到老年代，老年代很快就会被填满。而有很多对象虽然一次Minor GC没有消灭，但其实也并不会蹦跶多久，或许第二次，第三次就需要被清除。这时候移入老年区，很明显不是一个明智的决定。所以，Survivor的存在意义就是减少被送到老年代的对象，进而减少Major GC的发生。Survivor的预筛选保证，只有经历16次Minor GC还能在新生代中存活的对象，才会被送到老年代。

4.3 大对象直接进入老年代
所谓大对象是指，需要大量连续内存空间的Java对象，最典型的大对象就是那种很长的字符串以及数组。大对象对虚拟机的内存分配来说就是一个坏消息，经常出现大对象容易导致内存还有不少空间时就提前触发垃圾收集以获取足够的连续空间来 “安置” 它们。

虚拟机提供了一个XX:PretenureSizeThreshold参数，令大于这个设置值的对象直接在老年代分配，这样做的目的是避免在Eden区及两个Survivor区之间发生大量的内存复制（新生代采用的是复制算法）。

4.4 长期存活的对象将进入老年代
虚拟机给每个对象定义了一个对象年龄（Age）计数器，如果对象在Eden出生并经过第一次Minor GC后仍然存活，并且能被Survivor容纳的话，将被移动到Survivor空间中（正常情况下对象会不断的在Survivor的From与To区之间移动），并且对象年龄设为1。对象在Survivor区中每经历一次Minor GC，年龄就增加1岁，当它的年龄增加到一定程度（默认15岁），就将会晋升到老年代中。对象晋升老年代的年龄阈值，可以通过参数 XX:MaxPretenuringThreshold 设置。

4.5 动态对象年龄判定
为了能更好地适应不同程度的内存状况，虚拟机并不是永远地要求对象的年龄必须达到 MaxPretenuringThreshold才能晋升老年代，如果Survivor空间中相同年龄所有对象大小的总和大于Survivor空间的一半，年龄大于或等于改年龄的对象就可以直接进入老年代，无需等到MaxPretenuringThreshold中要求的年龄。

这其实有点类似于负载均衡，轮询是负载均衡的一种，保证每台机器都分得同样的请求。看似很均衡，但每台机的硬件不通，健康状况不同，我们还可以基于每台机接受的请求数，或每台机的响应时间等，来调整我们的负载均衡算法。

作者：窜天猴Plus
链接：https://www.jianshu.com/p/23f8249886c6
来源：简书
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。
