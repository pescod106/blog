## 问题描述
JAVA中String类以形参传递到函数里面，修改后外面引用不能获取到更改后的值

## 示例代码
```
public class Temp {
    String str = "good";
    Integer integerParam = 1000;
    Double doubleParam = 2D;
    Long longParam = 1L;
    char[] chars = {'a', 'b', 'c'};

    public void change(Long longParam,
                       Double doubleParam,
                       Integer integerParam,
                       String str,
                       char[] chars) {
        str = "test";
        integerParam = 200000;
        longParam = 10L;
        doubleParam = 3D;
        chars[0] = 'd';
    }

    public static void main(String[] args) {
        Temp temp = new Temp();
        System.out.println("执行方法之前");
        System.out.println("temp.str：" + temp.str);
        System.out.println("temp.integerParam：" + temp.integerParam);
        System.out.println("temp.doubleParam：" + temp.doubleParam);
        System.out.println("temp.longParam：" + temp.longParam);
        System.out.println("temp.chars："+new String(temp.chars));
        System.out.println();

        temp.change(temp.longParam, temp.doubleParam, temp.integerParam, temp.str, temp.chars);
        System.out.println("执行方法之后");
        System.out.println("temp.str：" + temp.str);
        System.out.println("temp.integerParam：" + temp.integerParam);
        System.out.println("temp.doubleParam：" + temp.doubleParam);
        System.out.println("temp.longParam：" + temp.longParam);
        System.out.println("temp.chars："+new String(temp.chars));
    }
}
```

输出结果如下
>
> 执行方法之前
> 
> temp.str：good
> 
>temp.integerParam：1000
>
>temp.doubleParam：2.0
>
>temp.longParam：1
>
>temp.chars：abc
>
>
>执行方法之后
>
>temp.str：good
>
>temp.integerParam：1000
>
>temp.doubleParam：2.0
>
>temp.longParam：1
>
>temp.chars：dbc
>

## 原因

String类的存储是通过final修饰的char[]数组来存储数据的，不可更改。所以当每次外部一个String类型的引用传递到方法内部时候，只是把外部String类型变量的引用传递给了方法参数变量。外部String变量和方法参数变量都是实际char[]数组的引用而已。所以当在方法内部改变这个参数的引用时候，因为char[]数组不可改变，所以每次新建变量都是新建一个新的String实例。很明显外部String类型变量没有指向新的String实例。所以也就不会获取到新的改变。

下面程序例程假定tString指向A内存空间，A内存空间存放了”hello”这个字符串，然后调用modst函数将tString引用赋值给了text引用，注意是引用。确实是传址，我们知道String是不可变的，任何进行更改的操作都会产生新的String实例。所以在方法里面text指向了B空间，B空间存放了”sdf” 字符串，但是这个时候tString还是指向A空间，并没有指向B空间。

```
public static String modst(String text) {
    return text = "sdf";
}
    String tString = "hello";
    System.out.println(tString);
    //改变text指向
    modst(tString);
    System.out.println(tString);

```
![]()

**总结一下三句话**

1. 对象是传引用
2. 原始类型是传值
3. String、Integer、Double、Short、Byte等immutable类型因为没有提供自身修改的函数，每次操作都是新生成一个对象，所以要特殊对待。可以认为是传值。

Integer和String一样，保存value的类变量是final属性，无法被修改，只能被重新赋值/生成新的对象。当Integer作为方法参数传递进方法内部时，对其的赋值将会导致原Integer的引用被指向了方法内的栈地址，失去了对原类变量地址的指向。对赋值后的Integer对象做的任何操作，都不会影响原来对象。
