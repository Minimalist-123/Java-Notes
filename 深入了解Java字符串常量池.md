java中有几种不同的常量池，以下的内容是对java中几种常量池的介绍以及重点研究一下字符串常量池。

class常量池
我们写的每一个Java类被编译后，就会形成一份class文件；class文件中除了包含类的版本、字段、方法、接口等描述信息外，还有一项信息就是常量池(constant pool table)，用于存放编译器生成的各种 字面量 (Literal)和 符号引用 (Symbolic References)，每个class文件都有一个class常量池。

其中 字面量 包括：1.文本字符串 2.八种基本类型的值 3.被声明为final的常量等; 符号引用 包括：1.类和方法的全限定名 2.字段的名称和描述符 3.方法的名称和描述符。

运行时常量池
运行时常量池存在于内存中，也就是class常量池被加载到内存之后的版本，是方法区的一部分。不同之处是：它的字面量可以动态的添加(String类的intern()),符号引用可以被解析为直接引用。

JVM在执行某个类的时候，必须经过加载、连接、初始化，而连接又包括验证、准备、解析三个阶段。而当类加载到内存中后，jvm就会将class常量池中的内容存放到运行时常量池中，由此可知，运行时常量池也是每个类都有一个。在解析阶段，会把符号引用替换为直接引用，解析的过程会去查询字符串常量池，也就是我们下面要说的StringTable，以保证运行时常量池所引用的字符串与字符串常量池中是一致的。

字符串常量池
在JDK6.0及之前版本，字符串常量池存放在方法区中在JDK7.0版本以后，字符串常量池被移到了堆中了。至于为什么移到堆内，大概是由于方法区的内存空间太小了。

在HotSpot VM里实现的string pool功能的是一个StringTable类，它是一个Hash表，默认值大小长度是1009；这个StringTable在每个HotSpot VM的实例只有一份，被所有的类共享。字符串常量由一个一个字符组成，放在了StringTable上。

在JDK6.0中，StringTable的长度是固定的，长度就是1009，因此如果放入String Pool中的String非常多，就会造成hash冲突，导致链表过长，当调用String#intern()时会需要到链表上一个一个找，从而导致性能大幅度下降；在JDK7.0中，StringTable的长度可以通过参数指定。

下面看一下实例：

String s = new String("abc")
这条语句创建了几个对象?

答案：共2个。第一个对象是”abc”字符串存储在常量池中，第二个对象在JAVA Heap中的 String 对象。这里不要混淆了s是放在栈里面的指向了Heap堆中的String对象。

比较下列两种创建字符串的方法：

String str1 = new String("abc");
String str2 = "abc";
答案：第一种是用new()来新建对象的，它会在存放于堆中。每调用一次就会创建一个新的对象。 运行时期创建 。

第二种是先在栈中创建一个对String类的对象引用变量str2，然后通过符号引用去字符串常量池里找有没有”abc”,如果没有，则将”abc”存放进字符串常量池，并令str2指向”abc”，如果已经有”abc” 则直接令str2指向“abc”。“abc”存于常量池在 编译期间完成 。

String s1 = new String("s1") ;
String s1 = new String("s1") ;
上面一共创建了几个对象？

答案：答案:3个 ,编译期Constant Pool中创建1个,运行期heap中创建2个.（用new创建的每new一次就在堆上创建一个对象，用引号创建的如果在常量池中已有就直接指向，不用创建）

比较字符串的‘==’和‘equals()’区别？

答案：

‘==’ 比较的是变量(栈)内存中存放的对象的(堆)内存地址，用来判断两个对象的地址是否相同，即是否是指相同一个对象。比较的是真正意义上的指针操作。注意：

1、比较的是操作符两端的操作数是否是同一个对象。

2、两边的操作数必须是同一类型的（可以是父子类之间）才能编译通过。

3、引用类型比较的是地址（即是否指向同一个对象），基本数据类型比较的是值，值相等则为true，如：int a=10 与 long b=10L 与 double c=10.0都是相同的（为true），因为他们都指向地址为10的堆。

‘equals()’用来比较的是两个对象是否相等，由于所有的类都是继承自java.lang.Object类的，在Object中的基类中定义了一个equals的方法，这个方法的初始行为是比较对象的内存地址，但String类中重写了equals方法， 比较的是字符串的内容 ，而不再是比较类在堆内存中的存放地址了。

总结：在没有重写equals方法的情况下，他们之间的比较还是基于他们在内存中的存放位置的地址值的，因为Object的equals方法也是用双等号（==）进行比较的，所以比较后的结果跟双等号（==）的结果相同。String类中重写了equals方法，变成了字符串内容的比较。

　　String s1 = "sss111";
　　String s2 = "sss111";
　　System.out.println(s1 == s2); //结果为true
　　String s1 = new String("sss111");
　　String s2 = "sss111";
　　System.out.println(s1 == s2); //结果为false
String s0 = "111";              //pool
String s1 = new String("111");  //heap
final String s2 = "111";        //pool
String s3 = "sss111";           //pool
String s4 = "sss" + "111";      //pool
String s5 = "sss" + s0;         //heap 
String s6 = "sss" + s1;         //heap
String s7 = "sss" + s2;         //pool
String s8 = "sss" + s0;         //heap
 
System.out.println(s3 == s4);   //true
System.out.println(s3 == s5);   //false
System.out.println(s3 == s6);   //false
System.out.println(s3 == s7);   //true
System.out.println(s5 == s6);   //false
System.out.println(s5 == s8);   //false
结合上面分析,总结如下:向大家推荐一个架构学习交流裙。交流学习裙号：687810532，里面会分享一些资深架构师录制的视频录像

1.单独使用””引号创建的字符串都是常量,编译期就已经确定存储到Constant Pool中；

2.使用new String(“”)创建的对象会存储到heap中,是运行期新创建的；

3.使用只包含常量的字符串连接符如”aa” + “aa”创建的也是常量,编译期就能确定,已经确定存储到String Pool中,String pool中存有“aaaa”；但不会存有“aa”。

4.使用包含变量的字符串连接符如”aa” + s1创建的对象是运行期才创建的,存储在heap中；只要s1是变量，不论s1指向池中的字符串对象还是堆中的字符串对象，运行期s1 + “aa”操作实际上是编译器创建了StringBuilder对象进行了append操作后通过toString()返回了一个字符串对象存在heap上。

5.String s2 = “aa” + s1; String s3 = “aa” + s1; 这种情况，虽然s2,s3都是指向了使用包含变量的字符串连接符如”aa” + s1创建的存在堆上的对象，并且都是s1 + “aa”。但是却指向两个不同的对象，两行代码实际上在堆上new出了两个StringBuilder对象来进行append操作。在Thinking in java一书中285页的例子也可以说明。

6.对于final String s2 = “111”。s2是一个用final修饰的变量，在编译期已知，在运行s2+”aa”时直接用常量“111”来代替s2。所以s2+”aa”等效于“111”+ “aa”。在编译期就已经生成的字符串对象“111aa”存放在常量池中。

作者：小刀爱编程
链接：https://www.jianshu.com/p/12c20ec338a4
来源：简书
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。
