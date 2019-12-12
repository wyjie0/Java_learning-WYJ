学习《深入理解Java虚拟机》的笔记

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
