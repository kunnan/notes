Java中final关键字详解

[TOC]

Java开发中，我们常常见到final关键字，经常在使用匿名内部类的时候可能会经常用到final关键字。Java中的String类就是一个final类，下面我们就来详细了解一下final这个关键字的用法。

# final关键字的基本用法

Java中，final关键字可以用来修饰类、方法和变量（包括成员变量和局部变量）。下面就从这三个方面来学习下final关键字的基本用法。

## 修饰类

当final关键字修饰一个类时，表明这个类不能被继承。final类中的成员变量可以根据需要设为final，但是要注意final类中的所有成员方法都会被隐式地指定为final方法。

![](http://7xs7a3.com1.z0.glb.clouddn.com/final-final%E4%BF%AE%E9%A5%B0%E7%B1%BB.png)

>在使用final修饰类的时候，要注意谨慎选择，除非这个类真的在以后不会用来继承或者是出于安全的考虑，尽量不要讲类设计为final类。

## 修饰方法

使用final关键字修饰方法的原因有两个：第一个原因是把方法锁定，以防止任何子类修改它的含义；第二个原因是效率。在早期的Java实现版本中，会将final方法转为内嵌调用，但是如果方法过于庞大，可能看不到内嵌调用带来的任何性能提升，不需要使用final方法进行这些优化。因此，如果想要禁止该方法在子类中被覆盖，那么久可以将该方法用final关键字修饰。

注：类的private方法会隐式地被指定为final方法。

## 修饰变量

final修饰变量的基本语法：

-	如果final修饰的是基本数据类型的变量，则其数值一旦在初始化之后便不能更改；
-	如果final修饰的是引用类型的变量，则其初始化之后便不能再指向另一个对象；

示例如下：

![](http://7xs7a3.com1.z0.glb.clouddn.com/final-final%E4%BF%AE%E9%A5%B0%E5%8F%98%E9%87%8F.png)

上面代码中，对变量i和obj的重新赋值都报错了。

# 深入理解final关键字

在了解final关键字的基本用法后，我们来看看final关键字容易混淆的地方。

## 类的final变量和普通变量有什么区别？

当用final作用于类的成员变量时，成员变量（注意是类的成员变量，局部变量只需要保证在使用之前被初始化赋值即可）必须在定义时或者构造器中进行初始化赋值，而且final变量一旦被初始化赋值之后，就不能再被赋值了。

final变量和普通变量到底有什么区别呢？先看下面的例子：

```Java
	public static void main(String[] args) {
		// TODO Auto-generated method stub
		
		String a = "helloworld";
		final String b = "hello";
		String c = "hello";
		String d = b + "world";
		String e = c + "world";
		
		System.out.println((a == d));
		System.out.println((a == e));

	}
```

运行结果:
```
true
false
```
为什么第一个比较结果是true，而第二个比较结果为false？
这就是final变量和普变量的区别了，当final变量是基本数据类型以及String类型时，如果在编译期间能知道它的确切值，则编译器会把它当做编译器常量使用，也就是说用到该final变量的地方相当于直接访问这个常量，不需要在运行时确定。因此在上面的代码中，由于b被final修饰，因此会被当做编译器常量，所以在使用到b的地方会直接将变量b替换为它的值，而对于变量c的访问则在运行时通过链接来运行。要注意，只有在编译期间确切知道final变量值的情况下，编译器才会进行这样的优化，下面的这段代码就不会进行这样的优化：

```Java
	public static void main(String[] args) {
		// TODO Auto-generated method stub
		
		String a = "helloworld";
		final String b = getStr();
		String c = "hello";
		String d = b + "world";
		String e = c + "world";
		
		System.out.println((a == d));
		System.out.println((a == e));

	}
	
	public static String getStr() {
		return "hell0";
	}
```
结果是：
```
false
false
```

## 被final修饰的引用变量指向的对象内容可变吗？

final修饰的引用变量一旦初始化赋值之后就不能再指向其他的对象，那么该引用变量指向的对象的内容可变吗？先看下面的例子：

```Java
public class FinalDemo {

	/**
	 * @param args
	 */
	public static void main(String[] args) {
		// TODO Auto-generated method stub
		final Person person = new Person();
		person.name = "liu";
		person.age = 27;
		System.out.println(person.toString());

	}


}

class Person {
	public String name = "zhang";
	public int age = 0;
	@Override
	public String toString() {
		return "Person [name=" + name + ", age=" + age + "]";
	}
	
}
```
结果是：

```
Person [name=liu, age=27]
```
>由运行结果可知，引用变量被final修饰之后，虽然不能再指向其他的对象，但是它指向的对象的内容是可变的。

## final和static

我们经常容易把final和static关键字混淆，static作用于成员变量用来表示只保存一份副本，而final的作用是用来保证变量不可变。

```Java
public class FinalDemo {

	/**
	 * @param args
	 */
	public static void main(String[] args) {
		// TODO Auto-generated method stub
		MyClass class1 = new MyClass();
		System.out.println("i: " + class1.i);
		System.out.println("j: " + class1.j);

		MyClass class2 = new MyClass();
		System.out.println("i: " + class2.i);
		System.out.println("j: " + class2.j);
	}


}

class MyClass {
	
	public final double i = Math.random();
	public static double j = Math.random();
		
}
```
运行结果:

```
i: 0.6230804434703052
j: 0.8707443438720905

i: 0.6451393896255735
j: 0.8707443438720905

```
从结果可知，每次打印的i值不同，而j值都是一样的。从这里口可以知道final和static的区别了。

## 匿名内部类中使用的外部局部变量为什么只能是final变量？

保证数据的一致性。Java编译器限定必须将外部局部变量限制为final变量，不允许对外部局部变量进行更改（对应引用类型的变量，是不允许指向新的对象），这样就保证了数据的一致性。

## 关于final参数的问题

关于网上流传的“当你在方法中不需要改变作为参数的对象变量时，明确使用final进行声明，会防止你无意的修改而影响到调用方法外的变量”，这句话其实是不恰当的。
因为无论参数是基本数据类型的变量还是引用类型的变量，使用final声明都不会达到上面所说的效果。

下面看例子：

![](http://7xs7a3.com1.z0.glb.clouddn.com/final-final%E4%BF%AE%E9%A5%B0%E5%8F%82%E6%95%B0.png)

上面的代码像是让人觉得final修饰之后，就不能在方法中更改变量i的值了。殊不知，方法changValue和main方法中的变量i根本就不是一个变量，因为对于基本类型的变量，Java参数传递采用的是值传递，相当于直接将变量进行了拷贝，所以即使没有final修饰的情况下，在方法内部改变了变量i的值也不会影响方法外的i。

再看下面的代码：

```Java
public class FinalDemo {

	/**
	 * @param args
	 */
	public static void main(String[] args) {
		// TODO Auto-generated method stub
		MyClass myClass = new MyClass();
		StringBuffer buffer = new StringBuffer("hello");
		myClass.changValue(buffer);
		System.out.println(buffer.toString());
		myClass.changValue2(buffer);
		System.out.println(buffer.toString());
		
	}


}

class MyClass {
		
	public void changValue(final StringBuffer buffer) {
		buffer.append("world");
	}
	
	public void changValue2(StringBuffer buffer) {
		StringBuffer buffer2 = buffer;
		buffer2.append("你好");
	}
}
```
运行结果是：

```
helloworld
helloworld你好
```
由结果可知，很显然，用final进行修饰参数和不修饰参数都没有阻止在changeValue中改变buffer指向的对象的内容。原因在对于引用数据类型，Java采用的是按引用传递，传递的是引用的地址，也就是变量所对象的内存空间的地址。在这里形参和实参指向的是同一个对象，因此让形参重新指向另一个对象对实参并没有任何影响。

