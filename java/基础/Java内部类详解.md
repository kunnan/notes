**内部类**

[TOC]

# 定义

可以将一个类的定义放在另一个类的定义内部，这就是内部类。

内部类是一种非常有用的特性，因为它允许你把一些逻辑相关的类组织在一起，并控制位于内部的类的可视性。然而必须要了解的是内部类与组合是完全不同的概念。

# 创建局部内部类

创建内部类的方式就是把类的定义置于外部类的里面。

```Java
package com.michael.java.innerclass;

public class Parcel1 {

    class Content {

        private int i = 11;

        public int value() {
            return i;
        }
    }

    class Destination {

        private String label;

        public Destination(String where) {
            label = where;
        }

        public String readLabel() {
            return label;
        }
    }

    public void ship(String dest) {
        Content content = new Content();
        System.out.println(content.value());
        Destination destination = new Destination(dest);
        System.out.println(destination.readLabel());
    }

    /**
     * @param args
     */
    public static void main(String[] args) {
        // TODO Auto-generated method stub
        Parcel1 parcel = new Parcel1();
        parcel.ship("ShangHai");

    }

}

打印结果：
11
ShangHai
```

当我们在ship()方法里面使用内部类的时候，与使用普通类没有什么不同。在这里实际的区别只是内部类的名字是嵌套在Parcel里面的。不过这不是唯一的区别。

更典型的情况是，外部类将有一个方法，该方法返回一个指向内部类的引用，就像to()和contexts()看到的那样：

```Java
package com.michael.java.innerclass;

public class Parcel2 {

    class Content {

        private int i = 11;

        public int value() {
            return i;
        }
    }



    class Destination {

        private String label;

        public Destination(String where) {
            label = where;
        }

        public String readLabel() {
            return label;
        }
    }

    public Destination to(String dest) {
        return new Destination(dest);
    }

    public Content content() {
        return new Content();
    }

    public void ship(String dest) {
        Content content = content();
        System.out.println(content.value());
        Destination destination = to(dest);
        System.out.println(destination.readLabel());
    }

    /**
     * @param args
     */
    public static void main(String[] args) {
        // TODO Auto-generated method stub
        Parcel2 p = new Parcel2();
        p.ship("ShangHai");

        Parcel2 q = new Parcel2();
        Parcel2.Content content = q.content();
        System.out.println(content.value());
        Parcel2.Destination destination = q.to("Beijing");
        System.out.println(destination.readLabel());

    }

}
打印结果：
11
ShangHai
11
Beijing
```
如果想从外部类的非静态方法之外的任意位置创建某个内部类的对象，那么必须像在main()方法中那样，具体地指明这个对象的类型：OuterClassName.InnerClassName.

# 链接到外部类

当生产一个内部类的对象时，此对象与生产它的外部类对象之间就有了一种联系，所以它能访问其外部类对象的所有成员，而不需要任何特殊条件。此外，内部类还拥有其外部类的所有元素的访问权。看看下面的例子：

```Java
package com.michael.java.innerclass;


interface Selector {
	boolean end();
	Object cuurent();
	void next();
}

public class Sequence {

	private Object[] items;
	private int next = 0;

	public Sequence(int size) {
		items = new Object[size];
	}

	public void add(Object obj) {
		if (next < items.length) {
			items[next++] = obj;
		}
	}

	private class SequenceSelecotr implements Selector {

		private int i = 0;

		@Override
		public boolean end() {
			// TODO Auto-generated method stub
			return i == items.length; //直接使用外部类的对象
		}

		@Override
		public Object cuurent() {
			// TODO Auto-generated method stub
			return items[i];
		}

		@Override
		public void next() {
			// TODO Auto-generated method stub
			if (i < items.length) {
				i++;
			}
		}

	}
	/**
	 * @param args
	 */
	public static void main(String[] args) {
		// TODO Auto-generated method stub
		Sequence sequence = new Sequence(10);
		for(int i = 0;i < 10;i++) {
			sequence.add(Integer.toString(i));
		}

		SequenceSelecotr selector = sequence.new SequenceSelecotr();
		while(!selector.end()) {
			System.out.println("current: " + selector.cuurent());
			selector.next();
		}
	}

}

打印结果：

current: 0
current: 1
current: 2
current: 3
current: 4
current: 5
current: 6
current: 7
current: 8
current: 9
```
有结果可以得知，内部类可以直接访问其外部类的方法和字段，内部类自动拥有对其外部类所有成员的访问权。这是如何做到的呢？当某个外部类的对象创建一个内部类对象时，此内部类对象必定会秘密地捕获一个指向那个外部类对象的引用。然后，在你访问此外部类的成员时，就是用那个引用来选择外部类的成员。幸运的是，编译器会帮你处理所有的细节。但你现在可以看到的是：内部类的对象只能在与其外部类的对象相关联的情况下才能被创建（在内部类是非static类时）。构建内部类对象时，需要一个指向其外部类的引用，如果编译器访问不到这个引用就会报错。

# 使用.this与.new

如果你需要生产对象外部类对象的引用，可以使外部类的名字后面紧跟.this。这样产生的引用自动地具有正确的类型，这一点在编译期就被知晓并受到检查，因此没有任何运行时开销。下面例子展示如何使用.this:

```Java
public class DoThis {


	public class InnerClass {

		public DoThis outer() {
			return DoThis.this;
		}
	}

	public void f() {
		System.out.println("DoThis.f()");
	}
	/**
	 * @param args
	 */
	public static void main(String[] args) {
		// TODO Auto-generated method stub
		DoThis dt = new DoThis();
		InnerClass innerClass = dt.new InnerClass();
		innerClass.outer().f();

	}

}

打印结果:
DoThis.f()
```

有时候你肯想要告知某些其他对象，去创建其某个内部类的对象。要实现此目的，你必须在new表达式中提供对其他外部类对象的引用，这时需要使用.new语法，就像下面这样：

```Java
public class DotNew {

	public class Inner {

	}

	/**
	 * @param args
	 */
	public static void main(String[] args) {
		// TODO Auto-generated method stub
		DotNew dn = new DotNew();

		Inner inner = dn.new Inner();
	}

}
```
要想直接创建内部类的对象，不能去引用外部类的名字DotNew，而是必须使用外部类的对象来创建该内部类对象。这也解决了内部类名字作用域的问题，因此你不必声明也不能声明dn.new DotNew.Inner()。

在拥有外部类对象之前是不可能创建内部类对象的，这是因为内部类对象会暗地里连接到创建它的外部类引用对象上。但是如果你创建的是静态内部类，那么它就不需要对外部类对象的引用。

# 内部类与向上转型

```Java
public interface Destination {

	String readlabel();
}

public interface Contents {

	int value();
}


class Parcel4 {

	private class PContents implements Contents {

		private int i = 11;


		@Override
		public int value() {
			// TODO Auto-generated method stub
			return 11;
		}

	}

	protected class PDestination implements Destination {

		private String label;

		private PDestination(String dest) {
			label = dest;
		}

		@Override
		public String readlabel() {
			// TODO Auto-generated method stub
			return label;
		}

	}


	public Destination destination() {
		return new PDestination("Beijing");
	}

	public Contents contents() {
		return new PContents();
	}
}

public class TestParcel {

	public static void main(String[] args) {

		Parcel4 parcel4 = new Parcel4();
		Contents contents = parcel4.contents();
		System.out.println(contents.value());
		Destination destination = parcel4.destination();
		System.out.println(destination.readlabel());

        //不能向下转型
//		PContents contents1 = parcel4.contents();
//		PDestination destination1 = parcel4.destination();
		//不能访问Parcel4的私有成员
//		Contents contents2 = parcel4.new PContents();

	}
}
打印结果：
11
Beijing
```
在Parcel4类中，PDestination相当于Parcel4类的protected成员，所以只有Parcel4及其子类、还有与Parcel4在同一个包中的类能访问PDestination;PContents相当于Parcel4的私有成员，只有Parcel4类能够访问它。

# 在方法和作用域内的内部类

下面的例子展示在方法的作用域内（而不是在其他类的作用域内）创建一个完整的类，这称为局部内部类。

```Java
public class Parcel5 {

	public Destination destination(String dest) {
		
		class PDestination implements Destination{
			
			private String label;
			
			private PDestination(String dest) {
				label = dest;
			}

			@Override
			public String readlabel() {
				// TODO Auto-generated method stub
				return label;
			}
			
		}
		
		return new PDestination(dest);
	}
	
	public static void main(String[] args) {
		
		Parcel5 p = new Parcel5();
		Destination destination = p.destination("Beijing");
		System.out.println(destination.readlabel());
	}
}
```
注意：
在destination()中定义了内部类PDestination，并不意味着一旦dest()方法执行完毕，PDestination就不可用了。
你可以在同一个子目录下的任意类对某个内部类使用类标识符PDestination，这并不会有命名冲突。

下面的例子展示如何在任意的作用域内嵌入一个内部类：

```Java
public class Parcel6 {

	public void destination(boolean isCreate) {

		if (isCreate) {

			class PDestination implements Destination {

				private String label;

				private PDestination(String dest) {
					label = dest;
				}

				@Override
				public String readlabel() {
					// TODO Auto-generated method stub
					return label;
				}

			}
			
			PDestination destination = new PDestination("Beijing");
			System.out.println(destination.readlabel());

		}
		//error，超出作用域范围
//		PDestination destination = new PDestination("Beijing");
//		System.out.println(destination.readlabel());

	}

	public static void main(String[] args) {

		Parcel6 p = new Parcel6();
		p.destination(true);
	}
}

打印结果：
Beijing
```

# 匿名内部类

先看一个匿名内部类示例：

```Java
public class Parcel7 {

	public Contents contents() {
		//匿名内部类
		return new Contents() {
			
			@Override
			public int value() {
				// TODO Auto-generated method stub
				return 11;
			}
		};
	}
	
	public static void main(String[] args) {
		Parcel7 p = new Parcel7();
		System.out.println(p.contents().value());
	}
}
```
这种语句的意思是：创建一个继承自Contents的匿名类的对象。

上述匿名内部类的语法是下述形式的简化形式：

```Java
public class Parcel7 {
	
	class MyContents implements Contents {

		@Override
		public int value() {
			// TODO Auto-generated method stub
			return 11;
		}
		
	}

	public Contents contents() {
		
		return new MyContents();
	}
	
	public static void main(String[] args) {
		Parcel7 p = new Parcel7();
		System.out.println(p.contents().value());
	}
}
```
上面的匿名内部类中，使用了默认的构造函数来生成Contents。下面的代码展示了带参数的构造器的匿名内部类：

```Java
class Wrapper {
	
	private int i = 0;
	
	public Wrapper(int x) {
		this.i = x;
	}
	
	public int value() {
		return i;
	}
}

public class Parcel8 {
	
	public Wrapper wrapper(int x) {
		
		return new Wrapper(x) {
			
			@Override
			public int value() {
				// TODO Auto-generated method stub
				return super.value() * 22;
			}
		};
	}

	
	public static void main(String[] args) {
		Parcel8 p = new Parcel8();
		System.out.println(p.wrapper(3).value());
	}
}

```

实例初始化匿名内部类成员变量：

```Java
package com.michael.java.innerclass;



public class Parcel9 {
	
	public Destination destination(final String dest,final float price){
		
		return new Destination() {
			
			private int cost;
			
			//实例初始化成员变量
			{
				cost = Math.round(price);
				System.out.println("Price: " + cost);
			}
			
			private String label = dest;
			
			@Override
			public String readlabel() {
				// TODO Auto-generated method stub
				return label;
			}
		};
		
	}

	
	public static void main(String[] args) {
		Parcel9 p = new Parcel9();
		Destination destination = p.destination("Beijing", 25.6f);
		System.out.println(destination.readlabel());
	}
}

打印结果：

Price: 26
Beijing

```

对于匿名类而言，实例初始化的实际效果就是构造器，当然它受到了限制-你不能重载实例初始化方法。

# 嵌套类(静态内部类)

如果不需要内部类对象与其外部类对象之间有联系，那么将内部类声明为static，这就是静态内部类(嵌套类)。静态内部类与普通内部类的区别在于：

-	要创建静态内部的对象并不需要其外部类的对象
-	不能从静态内部类的对象中访问非静态的外部类对象
-	普通内部类的字段和方法，只能放在类的外部层次上，所以普通的内部类不能有static数据和static字段，也不能包含静态内部类，但是静态内部类可以包含这些特性。

```Java
package com.michael.java.innerclass;

import com.michael.java.innerclass.Parcel11.ParcelDestination.AnotherLevel;

public class Parcel11 {

	private static class ParcelContents implements Contents {

		private int i = 11;

		@Override
		public int value() {
			// TODO Auto-generated method stub
			return i;
		}

	}

	protected static class ParcelDestination implements Destination {

		private String label;

		private ParcelDestination(String dest) {
			label = dest;
		}

		@Override
		public String readlabel() {
			// TODO Auto-generated method stub
			return label;
		}

		//静态方法
		public static void f() {
			
			System.out.println("f()");
		}

		//静态变量
		static int x = 10;

		//静态内部类
		static class AnotherLevel {
			
			public static void f() {
				System.out.println("AnotherLevel f()");
			}

			static int x = 10;
		}

	}
	
	public static Destination destination(String dest) {
		return new ParcelDestination("Beijing");
	}
	
	public static Contents contents() {
		return new ParcelContents();
	}

	/**
	 * @param args
	 */
	public static void main(String[] args) {
		// TODO Auto-generated method stub
		Contents contents = contents();
		contents.value();
		
		Destination destination = destination("Beijing");
		destination.readlabel();
		
		System.out.println(ParcelDestination.x);
		ParcelDestination.f();
		AnotherLevel.f();
		System.out.println(AnotherLevel.x);

	}

}
```
在main方法中，不需要任何Parcel11对象，而是使用选取static成员的普通语法来调用方法。

# 接口的内部类

正常情况下，不能在接口内放置任何代码，但静态内部类可以作为接口的一部分，你放到接口中的任何类都自动地是public和static、的。因为类是static的，只是将嵌套类置于接口的命名空间内，这并不违反接口的规则。你甚至可以在内部类中实现外部接口，如下所示：

```Java
public interface ClassInInterface {
	
	void howdy();
	

	class Test implements ClassInInterface {

		@Override
		public void howdy() {
			// TODO Auto-generated method stub
			System.out.println("Howdy");
		}
	}

}

public class Main {

	public static void main(String[] args) {

		Test test = new Test();
		test.howdy();

	}
}
```

# 为什么需要内部类

内部类最吸引人的原因是：每个内部类都能独立地继承自一个（接口的）实现，所有无论外部类是否已经继承了某个（接口的）实现，对于内部类都没有影响。

下面让我们考虑这样一种情况：即必须在一个类中以某种方式实现两个接口。由于接口的灵活性，你有两种选择：使用单一类，或者使用内部类：

```Java
interface A {
	
}

interface B {
	
}

class X implements A,B {
	
}

class Y implements A {
	
	//内部类实现接口
	B makeB() {
		return new B() {
			
		};
	}
}

public class MultiInterface {
	
	static void takesA(A a){}
	static void takesB(B b){}

	/**
	 * @param args
	 */
	public static void main(String[] args) {
		// TODO Auto-generated method stub
		X x = new X();
		Y y = new Y();
		
		takesA(x);
		takesA(y);
		
		takesB(x);
		takesB(y.makeB());

	}

}
```

如果使用的是抽象的类或具体的类，而不是接口，那就只能使用内部类才能实现多重继承。

```Java
class D {}

abstract class E {}

class Z extends D {
	E makeE() {
		return new E() { };
	}
}

public class MultiImplementation {

	static void takesD(D d){}
	static void takesE(E e){}
	/**
	 * @param args
	 */
	public static void main(String[] args) {
		// TODO Auto-generated method stub
		Z z = new Z();
		
		takesD(z);
		takesE(z.makeE());

	}

}
```
使用内部类可以获得一些特性：

-	内部类可以有多个实例，每个实例都有自己的状态信息，并且与其外部类对象的信息相互独立
-	在单个外部类中，可以让对个内部类以不同的方式实现同一个接口，或继承同一个类
-	创建内部类对象的时刻并不依赖于外部类对象的创建
-	内部类就是一个独立的实体

# 内部类的继承

```Java
package com.michael.java.innerclass;

class WithInner {
	
	class Inner{
		
		public void print() {
			System.out.println("Inner");
		}
	}
}

public class InheritInner extends WithInner.Inner {
	
	// !public InheritInner() 不会被编译
	
	InheritInner(WithInner wi) {
		wi.super();
	}

	/**
	 * @param args
	 */
	public static void main(String[] args) {
		// TODO Auto-generated method stub

		WithInner wi = new WithInner();
		InheritInner inner = new InheritInner(wi);
		inner.print();
	}

}
```
可以看到，InheritInner只继承自内部类，而不是外部类，但是当要生成一个构造器时，默认的构造器并不能通过，并且不能只是传递一个指向外部类对象的引用，还必须在构造器中使用如下语句`OuterClassNameReference.super()`

# 内部类可以被覆盖吗

```Java
class Egg {
	
	private Yolk yolk;
	
	protected class Yolk {
		
		public Yolk() {
			System.out.println("Egg.Yolk()");
		}
	}
	
	public Egg() {
		System.out.println("New Egg");
		yolk = new Yolk();
	}
}

public class BigEgg extends Egg {
	
	public class Yolk {
		public Yolk() {
			System.out.println("BigEgg.Yolk()");
		}
	}

	/**
	 * @param args
	 */
	public static void main(String[] args) {
		// TODO Auto-generated method stub
		new BigEgg();

	}

}

打印结果是：
New Egg
Egg.Yolk()
```

结果说明，当BigEgg继承了Egg之后，BigEgg里面的内部类Yolk并没有覆盖Egg里面的内部类Yolk，这两个内部类是完全独立的两个实体，各自在自己的命名空间内。

# 内部类标识符

由于每个类都会产生一个。class文件，其中包含了如何创建该类型的对象的全部信息（此信息产生一个“meta-class”，叫做Class对象），内部类也必须生成一个.class文件以包含它们的Class对象信息。这些类文件的命名有严格的规则，加上“$”，再加上内部类的名字。

```Java
Counter.class
LocalInnerClass$1.class ## 匿名内部类 用数字1表示
LocalInnerClass$1LocalCounter.class ##局部内部类  LocalCounter在LocalInnerClass里面
LocalInnerClass.class
```







