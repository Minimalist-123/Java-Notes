什么是枚举？
我们学习过单例模式，即一个类只有一个实例。而枚举其实就是多例，一个类有多个实例，但实例的个数不是无穷的，是有限个数的。例如word文档的对齐方式有几种：左对齐、居中对齐、右对齐。开车的方向有几种：前、后、左、右！
　　我们称呼枚举类中实例为枚举项！一般一个枚举类的枚举项的个数不应该太多，如果一个枚举类有30个枚举项就太多了！
定义枚举类型
定义枚举类型需要使用enum关键字，例如：
public enum Direction {
    FRONT, BEHIND, LEFT, RIGHT;
}
Direction d = Direction.FRONT;
注意，定义枚举类的关键字是enum，而不是Enum，所有关键字都是小写的！
其中FRONT、BEHIND、LEFT、RIGHT都是枚举项，它们都是本类的实例，本类一共就只有四个实例对象。
在定义枚举项时，多个枚举项之间使用逗号分隔，最后一个枚举项后需要给出分号！但如果枚举类中只有枚举项（没有构造器、方法、实例变量），那么可以省略分号！建议不要省略分号！
不能使用new来创建枚举类的对象，因为枚举类中的实例就是类中的枚举项，所以在类外只能使用类名.枚举项。

枚举与switch
枚举类型可以在switch中使用
Direction d = Direction.FRONT;
  switch(d) {
    case FRONT: System.out.println("前面");break;
    case BEHIND:System.out.println("后面");break;
    case LEFT:  System.out.println("左面");break;
    case RIGHT: System.out.println("右面");break;
    default:System.out.println("错误的方向");
}
Direction d1 = d;
System.out.println(d1);
注意，在switch中，不能使用枚举类名称，例如：“case Direction.FRONT：”这是错误的，因为编译器会根据switch中d的类型来判定每个枚举类型，在case中必须直接给出与d相同类型的枚举选项，而不能再有类型。

所有枚举类都是Enum的子类
所有枚举类都默认是Enum类的子类，无需我们使用extends来继承。这说明Enum中的方法所有枚举类都拥有。
int compareTo(E e)：比较两个枚举常量谁大谁小，其实比较的就是枚举常量在枚举类中声明的顺序，例如FRONT的下标为0，BEHIND下标为1，那么FRONT小于BEHIND；
boolean equals(Object o)：比较两个枚举常量是否相等；
int hashCode()：返回枚举常量的hashCode；
String name()：返回枚举常量的名字；
int ordinal()：返回枚举常量在枚举类中声明的序号，第一个枚举常量序号为0；
String toString()：把枚举常量转换成字符串；
static T valueOf(Class enumType, String name)：把字符串转换成枚举常量。
枚举类的构造器
枚举类也可以有构造器，构造器默认都是private修饰，而且只能是private。因为枚举类的实例不能让外界来创建！
enum Direction {
    FRONT, BEHIND, LEFT, RIGHT;//[在枚举常量后面必须添加分号，因为在枚举常量后面还有其他成员时，分号是必须的。枚举常量必须在枚举类中所有成员的上方声明。]
    
    Direction()//[枚举类的构造器不可以添加访问修饰符，枚举类的构造器默认是private的。但你自己不能添加private来修饰构造器。] {
        System.out.println("hello");
    }
}
其实创建枚举项就等同于调用本类的无参构造器，所以FRONT、BEHIND、LEFT、RIGHT四个枚举项等同于调用了四次无参构造器，所以你会看到四个hello输出。

枚举类可以有成员
其实枚举类和正常的类一样，可以有实例变量，实例方法，静态方法等等，只不过它的实例个数是有限的，不能再创建实例而已。
enum Direction {
    FRONT("front"), BEHIND("behind"), LEFT("left"), RIGHT("right");
    
    private String name;
    
    Direction(String name) {
        this.name = name;
    }
    
    public String getName() {
        return name;
    }
}
Direction d = Direction.FRONT;
System.out.println(d.getName());
因为Direction类只有唯一的构造器，并且是有参的构造器，所以在创建枚举项时，必须为构造器赋值：FRONT(“front”)，其中”front”就是传递给构造器的参数。你不要鄙视这种语法，你应该做的是接受这种语法！
Direction类中还有一个实例域：String name，我们在构造器中为其赋值，而且本类还提供了getName()这个实例方法，它会返回name的值。

枚举类中还可以有抽象方法
还可以在枚举类中给出抽象方法，然后在创建每个枚举项时使用“特殊”的语法来重复抽象方法。所谓“特殊”语法就是匿名内部类！也就是说每个枚举项都是一个匿名类的子类对象！
通常fun()方法应该定义为抽象的方法，因为每个枚举常量都会去重写它。
你无法把Direction声明为抽象类，但需要声明fun()方法为抽象方法。


enum Direction {
    FRONT() {
        public void fun() {
            System.out.println("FROND：重写了fun()方法");
        }
    }, 
    BEHIND() {
        public void fun() {
            System.out.println("BEHIND：重写了fun()方法");
        }
    }, 
    LEFT() {
        public void fun() {
            System.out.println("LEFT：重写了fun()方法");
        }
    },
    RIGHT() {
        public void fun() {
            System.out.println("RIGHT：重写了fun()方法");
        }
    };
    
    public abstract void fun()[只需要把fun()方法修改为抽象方法，但不可以把Direction类声明为抽象类。];
}
每个枚举类都有两个特殊方法
每个枚举类都有两个不用声明就可以调用的static方法，而且这两个方法不是父类中的方法。这又是枚举类特殊的地方，下面是Direction类的特殊方法。
static Direction[] values()：返回本类所有枚举常量；
static Direction valueOf(String name)：通过枚举常量的名字返回Direction常量，注意，这个方法与Enum类中的valueOf()方法的参数个数不同。

作者：煮黑豆
链接：https://www.jianshu.com/p/7d3e3f6695a5
来源：简书
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。
