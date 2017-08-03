Java中的构造函数调用顺序
=====

面试题中经常这样的题目，分析子类父类构造函数的调用顺序，下面开始分情况讨论父类子类构造函数的调用顺序：

先创建两个类A和B，其中B继承A

```Java
public class A {
	
	public A() {
		System.out.println("A的无参构造函数");
	}
	
	public A(String msg) {
		System.out.println("A的有参构造函数: " + msg);
	}

	public void print() {
		System.out.println("A的print方法");
	}
}

public class B extends A{

	public B() {
		System.out.println("B的无参构造函数");
	}
	
	public B(String msg){
	//	super(msg); //此句代码表示调用父类A的有参构造函数，不加此句代码则表示调用父类A的默认无参构造函数
		System.out.println("B的有参构造函数: " + msg);
	}

	public void print() {
		System.out.println("B的print方法");
	}

	public void add(){
		System.out.println("A的add方法");
	}
}
```
调用上述类：

```Java
	public static void main(String[] args) {
		// TODO Auto-generated method stub
		B b = new B();
		B b1 = new B("hello");
		b.print();
		b.add();
	}
```

运行结果：

```
A的无参构造函数
B的无参构造函数
A的无参构造函数: hello
B的有参构造函数: hello
B的print方法
A的add方法
```
从运行结果可得出以下结论:
-	实例化无参子类B时，先调用父类A的无参构造函数，再调用B的无参构造函数；
-	实例化有参子类B时，先调用父类A的默认无参构造函数，再调用B的有参构造函数；若要调用A的有参构造函数，则在类B的有参构造函数的第一行加上super(msg)这句代码。
-	子类B调用方法时有两种情况：一是如果子类B中的方法和父类A中的print方法一致时，则调用子类B中print的方法，但是可通过super.print()调用父类中的print方法;二是如果子类B中无add方法而父类A中有add方法时，则子类B的示例直接调用父类A中的add方法；
-	父类A不能调用子类B中的方法和变量