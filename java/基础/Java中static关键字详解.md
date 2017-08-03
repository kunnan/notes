Java中static关键字详解
[TOC]

static是Java的一个关键字，我们在Java开发中经常使用static，static主要用于下面五种情况：

# 静态变量

我们可以将类级别的变量声明为static，静态变量是属于类的，而不是属于类创建的对象或实例的。因为静态变量被类的所有实例共用，所以是线程不安全的。通常静态变量还和关键字final一起用，作为所有对象共用的资源或常量。如果静态变量不是私有的，那么可以通过ClassName.variableName来访问它。

```Java
private static int count;
public static String name;
public static final String DB_NAME = "contacts";
```

# 静态方法

静态方法也属于类，不属于实例。静态类只能访问类的静态变量或调用类的静态方法。通常静态方法作为工具方法，被其他类使用，而不需要创建类的实例。
静态方法如果没有使用静态变量，则是线程安全的。因为静态方法内部声明的变量，每个线程调用时都会重新创建一份，而不会共用同一个存储单元。

```Java
public class MathUtils {
	public static int add(int x,int y) {
		return x + y;
	}
}
```

# 静态块

静态块是类加载器加载对象时要执行的一组语句，它用于初始化静态变量，通常用于类加载的时候创建静态资源。我们在静态块中不能访问非静态变量。我们可以在一个类中有多个静态块，尽管这么做没什么意义，静态块只会在类加载到内存中的时候执行一次。

```Java
public class FinalDemo {
	
	public static String str;

	/**
	 * @param args
	 */
	public static void main(String[] args) {
		// TODO Auto-generated method stub
		new FinalDemo();
	}

	public static int add(int x,int y) {
		return x + y;
	}
	
	static {
		System.out.println("静态块3");
		//仅能访问静态变量和静态方法
		str = "Just";
		add(3,4);
	}
	
	static {
		System.out.println("静态块1");
	}
	

	public FinalDemo() {
		System.out.println("构造函数");
	}
}
```
运行结果：
```
静态块3
静态块1
构造函数
```
>由运行结果可知：
-	静态块最先加载，其次是构造函数；
-	有多个静态块时，按静态块顺序加载；
-	静态块内部代码按顺序运行;

# 静态内部类

一般我们对嵌套类使用static关键字，static不能用于最外层的类。静态的嵌套类和其他外层的类别无区别，嵌套只是方便打包。

下面我们来看一个使用static关键字的例子：

```Java
public class FinalDemo {
	
	/**
	 * 静态变量
	 */
	public static String str;

	/**
	 * @param args
	 */
	public static void main(String[] args) {
		// TODO Auto-generated method stub
		System.out.println("str: " + str);
		System.out.println("add(3,4):"+add(3, 4));
		
	}
	/**
	 * 静态方法
	 * @param x
	 * @param y
	 * @return
	 */
	public static int add(int x,int y) {
		return x + y;
	}
	
	/**
	 * 静态块
	 */
	static {
		System.out.println("静态块3");
		//仅能访问静态变量和静态方法
		str = "Test";
	}

	/**
	 * 静态类
	 * TODO
	 */
	public static class MyStaticClass {
		
		public String getTag() {
			//只能访问外部类的静态变量或方法
			return str;
		}
	}
}
```
运行结果是：
```
静态块3
str: Test
add(3,4):7
staticclass tag: Test
```

# 静态导包

静态导包就是java包的静态导入，用import static静态导入包是JDK1.5引入的特性。
一般我们导入一个类都用import com.java...className；而静态导入是这样：`import static com.java..className.* `。这里多了static关键字，还有就是类名后面多了` .* `，意思是导入这个类里所有的静态方法。当然也可以只导入某个静态方法，只要把`.*`替换成静态方法名就行了。然后在这个类中，就可以直接调用方法名来调用静态方法，而不必用`ClassName.`方法名的方式来调用。

优点：
这种方法可以简化一些操作，例如打印System.out.println()，就可以将其导入一个静态方法，在使用时直接println()就可以了。下面通过代码来看看两种方式的导包:

普通导包:

```Java
public class NormalImport {

    public static void main (String[] args) {

        System.out.println(Integer.MAX_VALUE);
        System.out.println(Integer.toHexString(42));

    }
    
}
```

静态导包:

```Java
public class StaticImport {


    public static void main (String[] args) {

        out.println(MAX_VALUE);
        out.println(toHexString(43));

    }
}
```
两个类的运行结果都是一样的。

静态导包的几条原则：
-	必须是import static而不是static import
-	提防含糊不清的static成员。例如，如果对Integer类和Long类执行了静态导入，引用MAX_VALUE时将导致一个编译器错误，因为Integer和Long都有一个MAX_VALUE常量，并且Java不会知道你在引用哪个MAX_VALUE。

