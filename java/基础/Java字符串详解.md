**Java字符串**

[TOC]

## 不可变String

String对象时不可变的。查看JDK文档你就会发现，String类中每一个看起来会修改String值的方法，实际上都是创建了一个全新的String对象，这个新的String对象来包含修改后的字符串内容，而最初的String对象则没有变化。

看看下面的代码：

```Java
public class Immutable {

    /**
     * @param args
     */
    public static void main(String[] args) {

        String str = "hello";
        System.out.println(str); //hello
        String str1 = upCase(str);
        System.out.println(str1); //HELLO
        System.out.println(str); //hello
    }

    public static String upCase(String s) {
        return s.toUpperCase();
    }
}
```

结果打印如下：

```
hello
HELLO
hello
```

当把str传给upCase()方法时，实际上传递的是引用的一个拷贝。其实，每当把String对象作为方法的参数时，都会复制一份对象的引用，而该引用所指向的对象其实一直指向同一个内存地址，从未改变。 回到upCase方法的定义，传入其中的引用有了名字s，只有upCase方法运行的时候，局部引用s才回存在，一旦upCase运行结束，s就消失了。当然upCase的返回值，其实只是最终结果的引用。而这个引用已经指向了一个新的对象，而原本的str对象还在原始的位置。

## 重载"+"与StringBuilder

String的不可变性会带来一定的效率问题。为String对象重载的"+"操作符就是一个例子。重载的意思是，一个操作符在应用特定的类时，被赋予特殊的意义（用于String的"+"与"+="是Java中仅有的两个重载过的操作符，而Java并不允许程序员重载任何操作符）。

操作符"+"可以用来连接String：

```Java
public class Concatention {

    /**
     * @param args
     */
    public static void main(String[] args) {
        // TODO Auto-generated method stub

        String str = "hello";
        String result = str + " welcom" + " to beijing";
        System.out.println(result);
    }

}

结果是:
hello welcom to beijing
```

这段代码可能是这样工作的：String有一个append（）方法，它会生成一个新的String对象，以包含"欢迎来"与str连接后的字符串。然后，该对象再与"到北京"相连，生成一个新的String对象result。 这种工作方式当然是可行的，但是为了生成最终的String，此方式会产生一大堆需要垃圾回收的中间对象。当达到一定的数量之后，性能表现会相当糟糕。

下面来看下以上代码到底是如何工作的，可以用JDK自带的工具javap命令来反编译以上代码。命令如下：

```
javap -c Concatention  //-c 表示生产JVM字节码
```

编译后的字节码为：

```
Compiled from "Concatention.java"
class Concatention {
  Concatention();
    Code:
       0: aload_0
       1: invokespecial #1                  // Method java/lang/Object."<init>":()V
       4: return

  public static void main(java.lang.String[]);
    Code:
       0: ldc           #2                  // String hello
       2: astore_1
       3: new           #3                  // class java/lang/StringBuilder
       6: dup
       7: invokespecial #4                  // Method java/lang/StringBuilder."<init>":()V
      10: aload_1
      11: invokevirtual #5                  // Method java/lang/StringBuilder.append:(Ljava/lang/String;)Ljava/lang/StringBuilder;
      14: ldc           #6                  // String  welcom
      16: invokevirtual #5                  // Method java/lang/StringBuilder.append:(Ljava/lang/String;)Ljava/lang/StringBuilder;
      19: ldc           #7                  // String  to beijing
      21: invokevirtual #5                  // Method java/lang/StringBuilder.append:(Ljava/lang/String;)Ljava/lang/StringBuilder;
      24: invokevirtual #8                  // Method java/lang/StringBuilder.toString:()Ljava/lang/String;
      27: astore_2
      28: getstatic     #9                  // Field java/lang/System.out:Ljava/io/PrintStream;
      31: aload_2
      32: invokevirtual #10                 // Method java/io/PrintStream.println:(Ljava/lang/String;)V
      35: return
}
```

需要注意的重点是：编译器自动引入了java.lang.StringBuilder类。虽然我们在源代码中并没有使用StringBuilder类，但是编译器却自作主张地使用了它，因为它更高效。

现在，你也行会觉得可以随意使用String对象，反正编译器会自动优化性能。可是在这之前，让我们更深入地看看编译器能为我们优化到什么程序。下面的程序采用两种方式生成一个String：方法一使用说个String对象，方法二使用StringBuilder

```Java
public class WhitherStringBuilder {

    public String implicit(String[] strs) {

        String result = "";

        for(int i = 0;i < strs.length;i++) {
            result += strs[i];
        }

        return result;
    }

    public String explicit(String[] strs) {

        StringBuilder builder = new StringBuilder();
        for(int i = 0;i < strs.length;i++) {
            builder.append(strs[i]);
        }

        return builder.toString();
    }
}
```

现在运行`javap -c WitherStringBuilder`可以看到两个方法对应的字节码，首先是implicit()方法：

```
  public java.lang.String implicit(java.lang.String[]);
    Code:
       0: ldc           #2                  // String
       2: astore_2
       3: iconst_0
       4: istore_3
       5: iload_3
       6: aload_1
       7: arraylength
       8: if_icmpge     38
      11: new           #3                  // class java/lang/StringBuilder
      14: dup
      15: invokespecial #4                  // Method java/lang/StringBuilder."<init>":()V
      18: aload_2
      19: invokevirtual #5                  // Method java/lang/StringBuilder.append:(Ljava/lang/String;)Ljava/lang/StringBuilder;
      22: aload_1
      23: iload_3
      24: aaload
      25: invokevirtual #5                  // Method java/lang/StringBuilder.append:(Ljava/lang/String;)Ljava/lang/StringBuilder;
      28: invokevirtual #6                  // Method java/lang/StringBuilder.toString:()Ljava/lang/String;
      31: astore_2
      32: iinc          3, 1
      35: goto          5
      38: aload_2
      39: areturn
```

从第8行到第35行构成了一个循环体。要注意的重点是：StringBuilder是在循环之内构造的，这意味着每经过循环一次，就会创建一个新的StringBuilder对象。

下面是explicit()方法对应的字节码：

```
 public java.lang.String explicit(java.lang.String[]);
    Code:
       0: new           #3                  // class java/lang/StringBuilder
       3: dup
       4: invokespecial #4                  // Method java/lang/StringBuilder."<init>":()V
       7: astore_2
       8: iconst_0
       9: istore_3
      10: iload_3
      11: aload_1
      12: arraylength
      13: if_icmpge     30
      16: aload_2
      17: aload_1
      18: iload_3
      19: aaload
      20: invokevirtual #5                  // Method java/lang/StringBuilder.append:(Ljava/lang/String;)Ljava/lang/StringBuilder;
      23: pop
      24: iinc          3, 1
      27: goto          10
      30: aload_2
      31: invokevirtual #6                  // Method java/lang/StringBuilder.toString:()Ljava/lang/String;
      34: areturn
```

可以看到，不仅循环部分的代码更简短、更简单，而且它只生成一个StringBuilder对象。显示地创建StringBuilder还允许你预先为其指定大小。如果你已经知道最终的字符串大概有多长，那预先指定StringBuilder的大小可以避免多次重新分配缓冲。

因此，当你为一个类编写toString()方法时，如果字符串操作比较简单，那就可以信赖编译器,它会为你合理地构造最终的字符串结果。但是，如果你要在toString方法中使用循环时，最好还是自己创建一个StringBuilder对象，用它来构造最终的结果。

## 无意识的递归

Java中的每个类从根本上都是继承自Object，标准容器类自然也不例外。因此容器类都有toString方法，并复写了该方法，使得它生成的String结果能够表达容器自身，以及容器所包含的对象。例如ArrayList.toString()，它会遍历ArrayList中包含的所有对象，调用每个元素上的toString方法：

如果你希望toString()方法打印出对象的内存地址，也许你会考虑使用this关键字：

```Java
public class InfiniteRecursion {

    public String toString() {
        return "Recursion address: " + this + "\n";
    }

    public static void main(String[] args) {

        List<InfiniteRecursion> list = new ArrayList<InfiniteRecursion>();

        for (int i = 0; i < 10; i++)
            list.add(new InfiniteRecursion());

        System.out.println(list);

    }
}

打印结果
Exception in thread "main" java.lang.StackOverflowError
    at java.lang.String.getChars(String.java:826)
    at java.lang.AbstractStringBuilder.append(AbstractStringBuilder.java:416)
    at java.lang.StringBuilder.append(StringBuilder.java:132)
    at com.micheal.java.staticdemo.InfiniteRecursion.toString(InfiniteRecursion.java:12)
    at com.micheal.java.staticdemo.InfiniteRecursion.toString(InfiniteRecursion.java:12)
    at com.micheal.java.staticdemo.InfiniteRecursion.toString(InfiniteRecursion.java:12)
```

这里发生了自动类型转换，由InfiniteRecursion类型转换成String类型。因为编译器看到一个String对象后面跟着"+"，而在后面的对象不是String而是this，于是编译器试着将this转换成一个String。它怎么转换的呢，正是通过调用this上的toString方法，于是就发生了递归调用，造成栈溢出。

如果你真的想要打印出对象的内存地址，应该调用Objec.toString()方法，所有，你应该调用super.toString()方法.

## String上的操作

以下是String对象具备的一些基本方法，重载的方法归纳在同一行中：

![](http://7xs7a3.com1.z0.glb.clouddn.com/string-String%E4%B8%8A%E7%9A%84%E6%93%8D%E4%BD%9C1.png) ![](http://7xs7a3.com1.z0.glb.clouddn.com/string-String%E4%B8%8A%E7%9A%84%E6%93%8D%E4%BD%9C2.png)

上上图表中可以看出，当需要改变字符串的内容时，String类的方法都会返回一个新的String对象。同时，如果内容没有发生改变，String的方法只是返回指向原对象的引用而已，这可以节约存储空间以及避免额外的开销。

## 格式化输出

### System.out.printf()

```Java
System.out.printf("Row: %d %s\n",12,"just");
```

### System.out.format()

```Java
System.out.format("Row: %d %s\n",12,"just");
```


