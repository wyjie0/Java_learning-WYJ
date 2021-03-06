学习《深入理解Java虚拟机》和《Java虚拟机规范》的笔记

* [编译期](#编译期)
* [运行期](#运行期)
* [虚拟机类加载机制](#虚拟机类加载机制)
* [Java内存区域](#Java内存区域)

## 编译期

Java语言的编译期有三个可能的时期：
1. 前端编译器把.java文件转变成.class文件的过程（Javac、Eclipse JDT中的增量式编译器（ECJ））
2. 虚拟机的后端运行期编译器（JIT编译器、just in time compiler）把字节码转变成机器码的过程（HotSpot VM的C1、C2编译器）
3. 静态提前编译器（AOT编译器）直接把.java文件编译成本地机器代码的过程（GNU Compiler for the java（GCJ）、Excelsior JET）

### Javac编译器
编译过程大致分为三个过程：
1. 解析与填充符号表过程
2. 插入式注解处理器的注解处理过程
3. 语义分析与字节码生成过程

上述三个过程由源码中这几个方法来完成
```
initProcessAnnotations(processors);//准备过程：初始化插入式注解处理器

delegateCompiler =
    processAnnotations(//过程2:执行注解处理
        enterTrees(stopIfError(CompileState.PARSE, //过程1.2:输入到符号表
                parseFiles(sourceFileObjects))),//过程1.1:词法分析、语法分析
        classnames);

delegateCompiler.compile2();//过程3:分析及字节码生成

case BY_TODO:
    while (!todo.isEmpty())
        generate(desugar(flow(attribute(todo.remove()))));//标注->数据流分析->解语法糖->生成字节码
    break;
```

#### 解析与填充符号表
解析步骤由parseFiles()方法完成，包括词法分析与语法分析。词法分析将程序语句的关键字、变量名、字面量、运算符解析成一个一个的标记（Token），语法分析是根据
Token序列构造**抽象语法树**的过程。抽象语法树是一种用来描述程序代码语法结构的树形表示方式，语法树的每一个节点都代表着程序代码中的一个语法结构，例如包、类型、修饰符、运算符、接口、返回值等。构建好抽象语法树过后，编译器就基本不会再对源码文件进行操作了，后续的操作都建立在抽象语法树之上。

#### 语义分析
语义分析的主要任务是对结构上正确的源程序进行上下文有关性质的审查，如进行类型审查。
语义分析过程分为标注检查、数据及控制流分析以及解语法糖。

标注检查步骤检查的内容包括变量使用前是否已被声明、变量与赋值之间的数据类型是否能够匹配等。**在标注检查步骤中，会进行常量折叠**

Java中最常用的语法糖主要是泛型、变长参数、自动装箱/拆箱等，虚拟机运行时不支持这些语法，它们在编译阶段还原为简单的基础语法结构，这个过程称为**解语法糖
**

#### 字节码生成
在字节码生成阶段进行了少量的代码添加和转换工作

<init>()方法和<clinit>()方法就是在这个阶段添加的，这两个构造器的产生过程实际上是一个代码收敛的过程，编译器把语句块（对于实例构造器而言是“{}”块，对
  于类构造器而言是“static{}块”）、变量初始化（实例变量和类变量）、调用父类的实例构造器等操作收敛到<init>()和<clinit>()方法之中，并且保证一定是按
  **先执行父类的实例构造器，然后初始化变量，最后执行语句块的顺序进行**。还会做一些诸如把字符串的加操作替换为StringBuilder的append()操作等用于优化程
  序的实现逻辑的工作。

### Java中的语法糖

#### 泛型与类型擦除

Java中的泛型旨在程序源码中存在，在编译后的字节码文件中，就已经替换为原来额原生类型，并且在相应的地方插入了强制转型代码，因此，对于运行期的Java语言来
说，ArrayList<Integer>与ArrayList<String>就是同一个类，所以泛型技术实际上是Java语言的一颗语法糖，Java语言中的泛型实现方法称为类型擦除，基于这种方
法实现的泛型称为伪泛型。
    
使用擦除法的好处是实现简非常容易实现Backport，运行期能够节省一些类型所占的内存空间。但坏处是运行期无法像C#等有真泛型支持的语言那样，将泛型类型与用户定义的普通类型同等对待，例如运行期做反射时无法获得到泛型信息。
    
#### 自动装箱、拆箱与遍历循环

```
//自动装箱、拆箱与循环遍历
public static void main(String[] args) {
    List<Integer> list = Arrays.asList(1,2,3,4);
    int sum = 0;
    for (int i : list) {
        sum += i;
    }
    System.out.println(sum);
}
    
//自动装箱、拆箱与循环遍历编译后
public static void main(String[] args) {
    List list = Arrays.asList(
            new Integer[] {
                    Integer.valueOf(1),
                    Integer.valueOf(2),
                    Integer.valueOf(3),
                    Integer.valueOf(4)
            });
    int sum = 0;
    for (Iterator localIterator = list.iterator(); localIterator.hasNext();) {
        int i = ((Integer)localIterator.next()).intValue();
        sum += i;
    }
    System.out.println(sum);
}
```
* 自动装箱、拆箱在编译之后被转化成了对应的包装和还原方法，如上例中的Integer.valueOf()与Integer.intValue()方法
* 遍历循环则把代码还原成了迭代器的实现，这也是为何遍历需要被遍历的类实现Iterable接口的原因。
* 变长参数在调用的时候变成了一个数组类型的参数

```
//自动拆箱的陷阱
public static void main(String[] args) {
    Integer a = 1;
    Integer b = 2;
    Integer c = 3;
    Integer d = 3;
    Integer e = 321;
    Integer f = 321;
    Long g = 3L;
    System.out.println(c == d);
    System.out.println(e == f);
    System.out.println(c == (a + b));
    System.out.println(c.equals(a + b));
    System.out.println(g == (a + b));
    System.out.println(g.equals(a + b));
}

//Integer源码
public static Integer valueOf(int i) {
    if (i >= IntegerCache.low && i <= IntegerCache.high)
        return IntegerCache.cache[i + (-IntegerCache.low)];
    return new Integer(i);
}
```
默认IntegerCache.low为-128，IntegerCache.high为127，如果值在这个区间内，它就会把变量i当作一个变量放入内存中；如果不在这个范围内则会new一个Integer
对象。所以第一个为true。
又因为包装类的“==”运算在没有遇到算术运算的情况下不会自动拆箱，所以第二个创建了新的对象，所以e和f是不同的引用，输出false
第三、四、五为true是因为遇到了算数运算，所以自动拆箱为int类型
第六个输出false是因为equals()方法不处理数据类型的转换

#### 条件编译

在Java代码中，使用**条件为常量**的if语句就相当于实现了条件编译。if语句会在编译阶段就“执行”，编译过后的代码中只存在条件为真的语句。

## 运行期

即时编译器在运行期将“热点代码”（运行特别频繁的代码块）编译成本地平台相关的机器码，并进行各种层次的优化。不过即使编译器的实现没有具体规定，完全与虚拟机
的实现有关。

### HotSpot虚拟机内的即时编译器

HotSpot虚拟机采用解释器与编译器并存的架构
#### 解释器与编译器
当程序需要迅速启动和执行的时候，解释器可以首先发挥作用，省去编译的时间，立即执行。在程序运行后，编译器逐渐发挥作用，把越来越多的代码编译成本地代码之
后，可以获得更高的执行效率。

* 为何HotSpot虚拟机要使用解释器与编译器并存的架构?

解释器可以减少启动时间，而编译器可以提高运行时效率。当程序运行环境中内存资源限制较大，则可以使用解释器执行程序来节约内存，反之可以使用编译执行来提升效
率。同时，解释器还可以作为编译器激进优化时的一个“逃生门”，让编译器根据概率选择一些大多数时候都能提升运行速度的优化手段，当激进优化的假设不成立，如加载
了新类后类型继承结构出现变化、出现“罕见陷阱”时可以通过逆优化退回到解释状态继续执行。

* 为何HotSpot虚拟机要实现两个不同的即时编译器？

HotSpot虚拟机有Client Compiler和Server Compiler两个编译器。为了在程序启动响应速度与运行效率之间达到最佳平衡，HotSpot虚拟机采用**分层编译**的策略
分层编译根据编译器编译、优化的规模和耗时，划分出不同的编译层次，其中包括：

第0层：程序解释执行，解释器不开启性能监控功能，可触发第1层编译

第1层：也称为C1编译，将字节码编译成本地代码，进行简单、可靠的优化，如有必要将加入性能监控的逻辑

第2层：也称为C2编译，也是将字节码编译成本地代码，但是会启用一些编译耗时较长的优化，甚至会根据性能监控信息进行一些不可靠的激进优化

* 程序何时使用解释器执行？何时使用编译器执行？

“热点代码”会被即时编译器编译，而热点代码分为两类：被多次调用的方法；被多次执行的循环体。
对于后一种情况，编译动作是由循环体出发的，但编译器依然会以整个方法作为编译对象。这种编译方式因为编译发生在方法执行过程中，因此称之为栈上替换（On 
Stack Replacement,OSR编译，即方法栈帧还在栈上，方法就被替换了）

通过**热点探测**来判断一段代码是否是热点代码。有两种方式：基于采样的热点探测；基于计数器的热点探测。HotSpot虚拟机采用的是第二种方式。而计数器也分为
两种：方法调用计数器和回边计数器。

如果不做任何设置，方法调用计数器并不是方法被调用的绝对次数，而是一个相对的执行频率，即一段时间内方法被调用的次数。当超过一定的时间限度，如果方法的调用
次数仍然不足以让它提交给即时编译器编译，那这个方法的调用计数器就会被减少一半，这个过程称为**方法调用计数器热度的衰减**，这段时间被称为**半衰周期**。
可以使用-XX: -UserCounterDecay来关闭热度衰减，让方法计数器统计方法调用的绝对次数。

回边计数器统计一个方法中循环体代码执行的次数，在字节码中遇到控制流向后跳转的指令称为**回边**。

在默认设置下，无论是方法调用产生的即时编译请求，还是OSR编译请求，虚拟机在代码编译器还未完成之前，都仍然将按照解释方式继续执行，而编译动作则在后台的编
译线程中进行。

* 如何编译成本地代码

Client Compiler编译器：三段式编译，主要关注点在于局部性的优化

第一阶段：一个平台独立的前端将字节码构造成一种高级中间代码表示（HIR）。HIR使用静态单分配的形式来代表代码值，这可以使得一些在HIR的构造过程之中和之后进
行的优化动作更容易实现

第二阶段：一个平台相关的后端从HIR中产生低级中间代码表示（LIR）。在此之前会在HIR上完成一些优化，如空值检查消除、范围检查消除等。

第三阶段：在平台相关的后端使用线性扫描算法在LIR上分配寄存器，并在LIR上做窥孔优化，然后产生机器代码。

#### 编译器优化

语言无关的优化技术之一：公共子表达式消除
语言相关的优化技术之一：数组范围检查消除
最重要的优化技术：方法内联
最前沿的优化技术：逃逸分析

### Java与C/C++的编译器对比

Java虚拟机的即时编译器与C/C++的静态优化编译器相比，可能会由于下列原因导致输出的本地代码有一些劣势：

1. 即时编译器运行占用的是用户程序的运行时间，具有很大的时间压力，它能提供的优化手段也严重受制于编译成本。如果编译速度不能达到要求，那用户将在启动程序
或程序的某部分察觉到重大延迟，这点使得即时编译器不敢随便引入大规模的优化技术，而编译的时间成本在静态优化编译器中并不是主要的关注点。

2. Java语言是动态的类型安全语言，这就意味着需要由虚拟机来确保程序不会违反语言语义或访问非结构化内存。这就意味着，虚拟机必须频繁地进行动态检查，如实例
方法访问时检查空指针、数组元素访问时检查上下界范围、类型转化时检查继承关系等。

3. Java语言中虽然没有virtual关键字，但是使用虚方法的频率却远远大于C/C++语言，这意味着运行时对方法接受者进行多态选择的频率要远远大于C/C++语言，也意
味着即时编译器在进行一些优化（如方法内联）时的难度要远大于静态优化编译器

4. Java语言是可以动态扩展的语言，运行时加载新的类可能改变程序类型的继承关系，这使得许多全局的优化都难以进行，因为编译器无法看见程序的全貌，许多全局的
优化措施都只能以激进优化的方式来完成，编译器不得不时刻注意并随着类型的变化而在运行时撤销或重新进行一些优化

5. Java语言中对象的内存分配都是堆上进行的，只有方法中的局部变量才能在栈上分配。而C/C++的对象则有多种内存分配方式，既可能在堆上分配，又可能在栈上分
配，如果可以在栈上分配线程私有的对象，将减轻内存回收的压力。

不过Java语言的这些性能上的劣势都是为了换取开发效率上的优势而付出的代价。而且C/C++编译器所有优化都在编译期完成，以运行期性能监视为基础的优化它都无法完
成。

## 虚拟机类加载机制

虚拟机把描述类的数据从Class文件加载到内存，并对数据进行校验、转换解析和初始化，最终形成可以被虚拟机直接使用的Java类型，这就是虚拟机的类加载机制。在
Java语言里，类型的加载、连接和初始化过程都是在程序运行期间完成的，这种策略会零类加载时增加性能开销，但是会为Java应用程序提供高度的灵活性，Java里动态
扩展的语言特性就是依赖运行期动态加载和动态连接这个特点实现的。

### 类加载的时机

类加载的生命周期：加载、验证、准备、解析、初始化、使用和卸载。其中验证、准备、解析统称为连接。

![image](https://user-images.githubusercontent.com/25001763/70872599-4f337b80-1fe4-11ea-80b2-89d750e54187.png)

其中加载、验证、准备、初始化和卸载这5个阶段的顺U型是确定的，而解析阶段可能发生在初始化之后，这是为了支持Java语言的运行时绑定（动态绑定）。而且上述五
个阶段通常都是互相交互地混合式进行的，通常会在一个阶段执行过程中调用、激活另外一个阶段。

虚拟机规范严格规定有且仅有下列5中情况，必须立即对类进行“初始化”（加载、验证、准备在此之前开始）：

1. 遇到new、getstatic、putstatic或invokestatic这4条字节码指令时，如果类没有进行过初始化，则需要先触发其初始化。Java代码场景：使用new关键字实例化
对象的时候、读取或设置一个类的静态字段的时候（被final修饰、已在编译期把结果放入常量池的静态字段除外）、以及调用一个类的静态方法的时候。

2. 使用java.lang.reflect包的方法对类进行反射调用的时候，如果类没有进行过初始化，则需要先触发其初始化

3. 当初始化一个类的时候，如果发现其父类还没有进行过初始化，则先触发其**父类**的初始化。而一个接口在初始化时，并不要求其父接口全部都完成了初始化，只有
在真正使用到父接口的时候才会初始化。

4. 当虚拟机启动时，用户需要指定一个执行的主类（包含main()方法的那个类），虚拟机会先初始化这个主类

5. 当使用JDK 1.7 的动态语言支持时，如果一个java.lang.invoke.MethodHandle实例最后的解析结果为REF_getStatic、REF_putStatic、REF_invokeStatic的方
法句柄，并且这个方法句柄所对应的类没有进行过初始化时，先触发其初始化

这5种场景中的行为称为对一个类进行主动引用。除此之外，所有引用类的方式都不会触发初始化，称为被动引用。

被动引用的例子：

* 通过子类引用父类的静态字段，不会导致子类初始化
```
public class SuperClass {

    static {
        System.out.println("superclass init");
    }
    public static int value = 123;
}

public class SubClass extends SuperClass{

    static {
        System.out.println("subclass init");
    }

}

public class NotInitialization {

    public static void main(String[] args) {
        System.out.println(SubClass.value);
    }
}
```
上述代码不会输出“subclass init”。对于静态字段，只有直接定义这个字段的类才会被初始化，因此通过子类来引用父类中定义的静态字段，只会触发父类的初始化而
不会触发子类的初始化。

* 通过数组定义来引用类，不会触发此类的初始化

```
public class NotInitialization {

    public static void main(String[] args) {
        SuperClass[] superClasses = new SuperClass[10];
    }
}
```
这段代码不会输出任何结果。

* 常量在编译阶段会存入调用类的常量池中，本质上并没有直接引用到定义常量的类，因此不会触发定义常量的类的初始化
```
public class ConstClass {

    static {
        System.out.println("constclass init");
    }

    public final static int VALUE = 10;
}

public class NotInitialization {

    public static void main(String[] args) {
        System.out.println(ConstClass.VALUE);
    }
}
```
这段代码只会输出常量的值10，而不会输出“constclass init”。

## Java内存区域

### Java虚拟机运行时数据区
Java虚拟机在执行Java程序的过程中会把它所管理的内存划分为若干个不同的数据区域。

![image](https://user-images.githubusercontent.com/25001763/71047336-4d99bd00-2176-11ea-8245-03f8752ab6dd.png)

* 程序计数器

可看作当前线程所执行的字节码的行号指示器。在虚拟机的概念模型中，字节码解释器工作时就是通过改变这个计数器的值来选取下一条需要执行的字节码指令，分支、循
环、跳转、异常处理、线程恢复等基础功能都要依赖这个计数器来完成。

* Java虚拟机栈

每一个方法在执行的同时会创建一个栈帧用于存储局部变量表、操作数栈、动态链接、方法出口等信息。每一个方法从调用直至执行完成的过程，就对应着一个栈帧在虚拟
机栈中入栈到出栈的过程。

* 本地方法栈

本地方法栈为虚拟机使用到的Native方法服务。

* Java堆

Java堆用于存放对象实例，也是垃圾收集器管理的主要区域。Java堆可细分为新生代和老年代。Java堆可以处于物理上不连续的内存空间中，只要逻辑上连续。

* 方法区

方法区用于存储已被虚拟机加载的类信息、常量、静态变量、即时编译器编译后的代码等数据。

### 垃圾收集器与内存分配策略

* 哪些内存需要回收？
* 什么时候回收？
* 如何回收？

#### 判断对象死亡的方法

##### 1.引用计数算法
给对象中添加一个引用计数器，每当有一个地方引用它时，计数器值就加1；当引用失效时，计数器值就减1；任何时刻计数器为0的对象就是不可能再被使用的。这个方法
很难解决对象之间互相循环引用的问题。

##### 2.可达性分析算法
通过一系列的成为“GC Roots”的对象作为起始点，从这些节点开始向下搜索，搜索所走过的路径称为引用链，当一个对象到GC Roots没有任何引用链相连时，则证明此对
象是不可用的。

在Java中，可作为GC Roots的对象包括下面几种：
* 虚拟机栈（栈帧的本地变量表）中引用的对象
* 方法区中类静态属性引用的对象
* 方法区中常量引用的对象
* 本地方法栈中JNI（即一般说的Native方法）引用的对象

在可达性分析算法中不可达的对象，可以在finalize()方法中进行自我拯救，将自己赋值给某个类变量或对象的成员变量。但是最好不要这么做，因为这样运行代价高
昂，不确定性大，无法保证各个对象的调用顺序

#### 垃圾收集算法

##### 1.标记-清除算法
首先标记出所有需要回收的内存，在标记完成后统一回收所有被标记的对象。

不足：1. 效率问题，标记和清除两个过程的效率都不高；2. 空间问题，标记清除后会产生大量不连续的内存碎片

##### 2.复制算法
复制算法为了解决效率问题，它将可用内存按容量分为大小相等的两块，每次只使用其中的一块。当这一块的内存用完了，就将还存活的对象复制到另外一块上面，然后再
把已使用过的内存空间一次清理掉。

现在的商业虚拟机将内存分为一块较大的Eden空间和两块较小的Survivor空间，每次使用Eden和其中一块Survivor。当回收时，将Eden和Survivor中还存活着的对象一
次性地复制到另外一块Survivor空间上，最后清理掉Eden和刚才用过的Survivor空间。HotSpot虚拟机默认Eden和Survivor的大小比例是8:1。当Survivor空间不够用
时，需要依赖其他内存（老年代）进行分配担保

##### 3.标记-整理算法
复制手机算法在对象存活率较高时就要进行较多的复制操作，效率将会变低。在老年代里的算法一般使用标记-整理。

![image](https://user-images.githubusercontent.com/25001763/71051679-0c100e80-2184-11ea-88ed-19d9ff395bd1.png)

##### 4.分代收集算法
当前商业虚拟机的垃圾收集都采用“分代收集”算法，这种算法根据对象存活周期的不同将内存分为几块。一般是新生代和老年代，根据各个年代的特点采用最适当的收集算
法。在新生代中，每次垃圾收集时都发现有大批对象死去，只有少量存活，那就选用复制算法，只需要付出少量存活对象的复制成本就可以完成收集。而老年代中因为对象
存活率高、并没有额外空间对它进行分配担保，就必须使用标记-清理或标记-整理算法

#### 对象分配策略
##### 1.对象优先在Eden区分配，当Eden去没有足够空间进行分配时，虚拟机将发起一次Minor GC
##### 2.大对象直接进入老年代，避免在Eden区和两个Survivor区之间发生大量的内存复制
##### 3.长期存活的对象将进入老年代。
    * 虚拟机给每个对象定义了一个对象年龄计数器。如果对象在Eden出生并经过第一次Minor GC后仍然存活，并且能被Survivor容纳的话，将被移动到Survivor空间中，并且对象年龄设为1。对象在Survivor区中没熬过一次Minor GC，年龄就增加1，当它的年龄增加到一定程度（默认为15岁），就将会本晋升到老年代中。如果在Survivor空间中相同年龄所有对象大小的总和大于Survivor空间的一半，年龄大于或等于该年龄的对象就可以直接进入老年代。
