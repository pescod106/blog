## 1.类加载器
每个类编译后产生一个Class对象，存储为.class文件，JVM使用类加载器ClassLoader来加载类的字节码文件(.class)，一般的，只会用到一个原生的类加载器，它只加载Java API等可信类，通常只在本地磁盘中加载。如果需要从远程网络或数据库中下载.class文件。需要挂载额外的类加载器。

一般来说，类加载器是按照树形的层次结构组织的。每一个加载器都有一个父类加载器。另外每个类加载器都支持代理模式，既可以自己完成Java类的加载工作，也可以起代理给其他的类加载器。

类加载器的加载顺序有两种，父类有限策略、自己有限策略。父类有限策略是一般的情况(JDK)，在这种策略下，类在加载某个Java类之前，会尝试代理给其父类加载器，只有当父类加载器找不到时，才尝试自己加载。自己优先策略正好相反，它会尝试自己去加载，找不到的时候才要父类加载器去加载，这种在Web容器中比较常见(Tomcat)。

## 2.动态加载
不管是用什么的类加载器，类都是在第一次被使用时，动态的加载到JVM的。这句话有两层含义：
> 1. Java程序在运行时并不一定被完全加载，只有当发现该类还没有加载时，才去本地或者远程查找类的.clsss文件并验证和加载。
> 2. 当程序创建了第一个对类的静态成员(如类的静态变量、静态方法、构造方法-构造方法也是静态的)的引用时，才会加载该类。Java的这个特性叫做：**动态加载**

需要区分加载和初始化的区别，加载了一个类的.class文件，不意味着该Class对象被初始化，事实上，一个类的初始化包括三个步骤：

* 加载(loading)：由类加载器执行，查找字节码，并创建Class对象(只是创建)
* 链接(linking)：验证字节码，为静态域分配存储空间(只是分配，并不初始化该存储空间)，解析该类创建所需要的对其他类的引用
* 初始化(initialization)：首先执行静态初始化块static{}，初始化静态变量，执行静态方法(如构造方法)

### 链接

Java在加载了类之后，需要进行链接的步骤，链接简单地说，就是讲已经加载的Java二进制代码组合刀JVM运行状态中去。它包括三个步骤：

1. **验证**(verification)：验证是保证二进制字节码在结构上的正确性，具体来说，工作包括检测类型正确性，接入属性正确性(public、private)，检查final Class没有被继承，检查静态变量的正确性等
2. **准备**(perparation)：准备阶段主要是创建静态域，分配空间，给这些域设置默认值。需要注意的是两点：一个是在准备阶段不会执行任何代码，仅仅是设置默认值。二是这些默认值是这样分配的，原生类型全部设为0，如：float 0f，int 0，boolean 0，其他引用类型为NULL
3. **解析**(resolution)：解析的过程就是对类中的接口、类、方法、变量的符号引用进行解析并定位，解析成直接引用(符号引用就是编码使用字符串标识某个变量、接口的位置，直接引用就是根据符号引用翻译出来的地址),并保证这些类被正确得找到。解析的过程可能导致其他的类被加载。需要注意的是，根据不同的解析策略，这一步不一定是必须的，有些解析策略在解析时把所有引用解析，这是early relolution，要求所有引用都必须存在。还有一种策略是late relolution，这也是Oracle jdk所采取的策略，即在类只有被引用了，还没有被真正用到时，并不进行解析，只有当真正用到了，才去加载和解析这个类

### 初始化
static{}是在第一次初始化时执行的，且只执行一次，用下面的代码可以判定出来：

```
public class Toy {
    private String name;

    public static final int price = 10;

    static {
        System.out.println("initializing");
    }

    public Toy() {
        System.out.println("building");
    }

    public Toy(String name) {
        this.name = name;
    }
}
```
对上面的类执行下面的代码：

```
Class c = Class.forName("com.pescod.entity.Toy");
```
输出信息：
> initializing

可以看到，不实例化，只执行forName初始化时，仍然会执行static{}子句，但不执行构造方法，因此输出的只有initializing

根据Java虚拟机规范，所有Java虚拟机实现必须在每个类或接口被Java首次**主动使用**时才初始化。主要包括下面6中：

1. 创建类的实例
2. 访问某个类或者接口的静态变量，或者对静态变量赋值(如果访问的是静态编译时常量(即编译时可以确定值的常量)不会导致类的初始化)
3. 调用类中的静态方法
4. 反射(Class.forName("xxx.xxx"))
5. 初始化一个类的子类(相当于对父类的主动使用)，不过直接通过子类引用父类元素，不会引起子类的初始化([参考实例6](#example_6))
6. 被Java虚拟机标明为启动类的类(包含main方法)

#### 示例

##### 示例1
通过上面的讲解，将可以理解下面的程序

```
public class Toy {

    //静态子句，只在类第一次被加载并初始化时执行一次，而且只执行一次
    static {
        System.out.println("initializing");
    }

    //构造方法，在每次声明新对象时加载
    public Toy() {
        System.out.println("building");
    }
}
```
对上面的代码段，第一次调用Class.forName("Toy")，将执行static子句；如果在之后执行new Toy()都只执行构造方法。

##### 示例2
需要注意**newInstance**方法

```
//获得类（注意，需要使用含包名的全限定名）
Class cc = Class.forName("Toy");
//相当于new一个对象，但Gum类必须有默认构造方法（无参)
Toy toy=(Toy)cc.newInstance(); 
```

##### 示例3
用**类字面常量**.class和Class.forName都可以创建对类的引用，但是不同点在于，用.class创建Class对象的引用时，不会自动初始化该Class对象(static子句不会执行)

```
public class TestToy {
    public static void main(String[] args) {
        Class c = Toy.class; // 不会输出任何值
    }
}
```
使用Toy.class是在编译期运行的，因此在编译时必须已经有了Toy.Class文件，不然会编译失败，这与Class.forName("Toy")不同，后者是运行时动态加载。

但是，如果main方法直接写在Toy类中，那么调用Toy.class，会引起初始化，并输出initilizing,原因不是Toy.class引起的，而是该类中含有启动方法main，该方法会导致Toy类的初始化。

##### 示例4
**编译时常量**，回到完整的Toy类，如果直接输出：System.out.println(Toy.price)，会发现static子句和构造方法都没有被执行，这是因为在Toy中，常量price被static final限定，这样的常量叫做编译时常量，对于这种常量，不需要初始化就可以读取。

编译时常量必须满足三个条件：static、final、常量

下面几种都不是编译时常量，对他们的引用，都不会引起类的初始化：

```
static int a;
final int b;
static final int c = ClassInitialization.rand.nextInt(100);
static final int d;
static {
    d = 5;
}
```

##### 示例5
**static块的本质**，注意下面的代码

```
class StaticBlock {
    static final int c = 3;
    static final int d;
    static int e = 5;
    static {
        d = 5;
        e = 10;
        System.out.println("Initializing");
    }

    StaticBlock() {
        System.out.println("Building");
    }
}

public class StaticBlockTest {
    public static void main(String[] args) {
        System.out.println(StaticBlock.c);
        System.out.println(StaticBlock.d);
        System.out.println(StaticBlock.e);
    }
}

```
执行一下，结果为：
>3
>
>Initializing
>
>5
>
>10

原因是这样的：输出c时，由于c是编译时常量，不会引起类初始化，因此直接输出，输出d时，d不是编译时常量，所以会引起初始化操作，即static块的执行，于是d被赋值为5，e被赋值为10，然后输出Initializing，之后输出d=5,e=10

但e为什么是10呢？原来，JDK会自动为e的初始化创建一个static块，所以上面的代码等价于：

```
class StaticBlock {
    static final int d;
    static int e;
    static {
       e=5; 
    }

    static {
        d = 5;
        e = 10;
        System.out.println("Initializing");
    }

    StaticBlock() {
        System.out.println("Building");
    }
} 
```
可见，按**顺序执行**，e先被初始化为5，再被初始化为10，于是输出了10

类似的，容易想到下面的代码：

```
class StaticBlock {
    static {
        d = 5;
        e = 10;
        System.out.println("Initializing");
    }

    static final int d;

    static int e = 5;

    StaticBlock() {
        System.out.println("Building");
    }
}
```
在这段代码中，将e的声明放到了static块后面，于是，e先被初始化为10,在被初始化为5，所以这段代码中e会输出为5

##### <a name="example_6">示例6</a>
当访问一个Java类或接口的静态域时，只有真正声明这个域的类或接口才会被初始化

```
class B {
    static int value = 100;
    static {
        System.out.println("Class B is initialized");// 输出
    }
}

class A extends B {
    static {
        System.out.println("Class A is initialized"); // 不输出
    }
}

public class SuperClassTest {
    public static void main(String[] args) {
        System.out.println(A.value);// 输出100
    }
}
```
输出：
>Class B is initialized
>
>100

在该例子中，虽然通过A来引用了value,但value是在父类B中声明的，所以只会初始化B，而不会引起A的初始化。