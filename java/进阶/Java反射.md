**Java反射**

[TOC]

# 什么是反射

Java反射是可以让我们在运行时获取类的函数、属性、父类、接口等Class内部信息的机制。通过反射还可以让我们在运行期实例化对象，调用方法，通过调用get/set方法获取和设置变量的值，即使方法或属性是类私有的也可以通过反射的形式调用，这种“看透Class“的能力被称为内省，这种能力在框架开发中尤为重要。有些情况下，我们要使用的类在运行时才会确定，这个时候我们不能在编译期就使用它，因此只能通过反射的形式来使用在运行时才存在的类（该类符合某种特定规范，例如JDBC），这是反射用得比较多的场景。

还有一个比较常见的场景就是编译时我们对于类的内部信息不可知，必须得到运行时才能获取类的具体信息。比如ORM框架，在运行时才能够获取类中的各个属性，然后通过反射的形式获取其属性名和值，存入数据库，这也是反射比较经典应用场景之一。

# Class类

既然反射是操作Class信息的，那么Class又是什么呢？

![](http://7xs7a3.com1.z0.glb.clouddn.com/reflection-class%E7%B1%BB.png)

当我们编写完一个Java项目之后，所有的Java文件都会被编译成一个.class文件，这些Class对象承载了这个类型的父类、接口、构造函数、方法、属性等原始信息，这些class文件在程序运行时会被ClassLoader加载到虚拟机中。当一个类被加载以后，Java虚拟机就会在内存中自动产生一个Class对象。我们通过new的形式创建对象实际上就是通过这些Class来创建，只是这个过程对于我们是不透明而已。

# 反射Class以及构造对象

## 获取Class对象

在你想检查一个类的信息之前，你首先需要获取类的Class对象。Java中的所有类型包括基本类型，即使是数组都有与之关联的Class类的对象。如果你在编译期知道一个类的名字的话，那么你可以使用如下的方式获取一个类的Class对象。

```Java
Class<String> myClass = String.class;
System.out.println(myClass);

打印：
class java.lang.String
```

如果你已经得到了某个对象，但是你想获取这个对象的Class对象，那么你可以通过下面的方法得到：

```Java
String str = new String();
Class<String> myClass = str.getClass();
System.out.println(myClass);

打印：
class java.lang.String
```

如果你在编译期获取不到目标类型，但是你知道它的完整类路径，那么你可以通过如下的形式来获取Class对象：

```Java
Class<?> myClass = null;
try {
	myClass = Class.forName("java.lang.String");
} catch (ClassNotFoundException e) {
	// TODO Auto-generated catch block
	e.printStackTrace();
}
System.out.println(myClass);

打印结果：

class java.lang.String
```
在使用Class.forName()方法时，你必须提供一个类的全名，这个全名包括类所在的包的名字。例如String类位于java.lang包中，那么它的完整路径就是java.lang.String。如果在调用Class.forName()方法时，没有在编译路径下（classpath）找到对象的类，那么将会抛出ClassNotFoundException。

**方法说明**

```Java
/**
* 加载指定的Class对象
*@param className 要加载的类的完整路径，例如java.lang.String。
*/
public static Class<?> forName(String className)throws ClassNotFoundException

/**
* 加载指定的Class对象
*@param className 要加载的类的完整路径，例如java.lang.String。
*@param initialize 是否要初始化该Class对象
*@param loader  指定加载该类的ClassLoader
*/
public static Class<?> forName(String name, boolean initialize,ClassLoader loader)throws ClassNotFoundException
```
## 通过Class对象构造目标类型的对象

一旦你拿到Class对象之后，你就可以为所欲为了，但获取Class对象只是第一步，我们需要在执行那些强大的行为之前通过Class对象构造出该类型的对象，然后才能通过该对象释放它的功能。我们知道，在java中要构造对象，必须通过该类的构造函数，那么其实反射也是一样的。但是它们确实是有区别的，通过反射构造对象，我们首先要获取类的Constructor（构造器）对象，然后通过Constructor来创建目标类的对象。

```Java
private static void classForName() {

	try {
		//1.获取Class对象
		Class<?> clz = Class.forName("com.michael.java.reflection.Student");

		//2.通过Class对象获取Constructor，Student构造函数中有一个String参数
		Constructor<?> constructor = clz.getConstructor(String.class);

		//3.通过Constructor创建Student对象
		Student object = (Student) constructor.newInstance("michael");

		System.out.println("obj: " + object.toString());
		object.showMyName();
		object.takeAnExamination();

	} catch (Exception e) {
		// TODO Auto-generated catch block
		e.printStackTrace();
	}
}

打印结果：

obj:  Student :  michael
My name is michael
 takeAnExamination

```
通过上述代码，我们就可以在运行时通过完整的类名来构建对象。

**获取构造函数方法**

```Java

//获取一个公有的构造函数，参数为可变参数，如果构造函数有参数，那么需要将参数的类型传递给getConstructor方法
public Constructor<T> getConstructor(Class<?>... parameterTypes)

//获取目标类所有的公有构造函数
public Constructor<?>[] getConstructors() throws SecurityException
```

>注意，当你通过反射获取到Constructor、Method、Filed后，在反射调用之前将此对象的accessible标志设置为true，以此来提升反射速度。值为true则指示反射的对象在使用时应该取消Java语言访问检查。值为false则指示反射的对象应该实施Java语言访问检查。例如：

```Java

//设置Constructor的Accessible
Constructor<?> constructor = clz.getConstructor(String.class);
constructor.setAccessible(true);

Method learnMethod = Student.class.getMethod("learn", String.class);
//设置Method的Accessible
learnMethod.setAccessible(true);
```
Student.java

```Java
package com.michael.java.reflection;

public class Student extends Person implements Examination{

    // 年级
    public int mGrade;
    private String age;

    public Student(String aName) {
        super(aName);
    }

    public Student(int grade, String aName) {
        super(aName);
        mGrade = grade;
    }

    private void learn(String course,int count) {
        System.out.println(mName + " learn " + course + "--"+ count);
    }

    public void takeAnExamination() {
        System.out.println(" takeAnExamination ");
    }

    public String toString() {
        return " Student :  " + mName;
    }

}

```

Person.java

```Java
package com.michael.java.reflection;

public class Person {

    String mName;

    public Person(String aName) {
        mName = aName;
    }

    private void sayHello(String friendName) {
        System.out.println(mName + " say hello to " + friendName);
    }

    protected void showMyName() {
        System.out.println("My name is " + mName);
    }

    public void breathe() {
        System.out.println(" take breathe ");
    }

}

```

Examination.java

```Java
package com.michael.java.reflection;

public interface Examination {

	public void takeAnExamination();
}
```
Breathe.java

```Java
package com.michael.java.reflection;

public interface Breathe {

	public void breathe();
}
```

# 反射获取类中函数

## 获取当前类中定义的方法

要获取当前类中定义的所有方法可以通过Class中的getDeclaredMethods函数，它会获取当前类中的public、default、protected、private的所有方法。`getDeclaredMethod(String name, Class...<?> parameterTypes)`则是获取某个指定的方法。代码示例如下：

```Java
private static void showDeclaredMethods() {

	Student student = new Student("Michael");

	// 获取类中所有方法
	Method[] methods = student.getClass().getDeclaredMethods();

	for (Method method : methods) {

		System.out.println("方法名: " + method.getName());
	}

	try {
		Method learnMethod = student.getClass().getDeclaredMethod("learn",
				String.class, Integer.TYPE);
		// 获取方法的参数类型列表
		Class<?>[] paramClasses = learnMethod.getParameterTypes();

		for (Class<?> clz : paramClasses) {
			System.out.println("learn 方法的参数类型: " + clz.getName());
		}

		// 是否是 private 函数，属性是否是 private 也可以使用这种方式判断
		System.out.println(learnMethod.getName() + " is private method: "
				+ Modifier.isPrivate(learnMethod.getModifiers()));

		//调用私有learn函数
		learnMethod.setAccessible(true);
		learnMethod.invoke(student, "java--->",2);

	} catch (Exception e) {
		// TODO Auto-generated catch block
		e.printStackTrace();
	}

}

运行结果:

方法名: toString
方法名: learn
方法名: takeAnExamination
learn 方法的参数类型: java.lang.String
learn 方法的参数类型: int
learn is private method: true
Michael learn java--->---2

```

## 获取当前类、父类中定义的公有方法

要获取当前类以及父类中的所有public方法可以通过Class中的getMethods函数，而getMethod则是获取某个指定的方法。代码示例如下：

```Java
private static void showMethods() {

	Student student = new Student("Michael");
	//获取所有公有方法
	Method[] methods = student.getClass().getMethods();

	for(Method method : methods){
		System.out.println("公有方法名： " + method.getName());
	}

	try {
		//通过 getMethod 只能获取公有方法，如果获取私有方法则会抛出异常
		Method method = student.getClass().getMethod("takeAnExamination");

		method.invoke(student);

	} catch (Exception e) {
		// TODO Auto-generated catch block
		e.printStackTrace();
	}
}

结果打印：

公有方法名： toString
公有方法名： takeAnExamination
公有方法名： breathe
公有方法名： wait
公有方法名： wait
公有方法名： wait
公有方法名： equals
公有方法名： hashCode
公有方法名： getClass
公有方法名： notify
公有方法名： notifyAll
 takeAnExamination 

```
**接口说明**

```Java
//获取Class对象中指定的所有函数，不包括从父类继承的函数
public Method[] getDeclaredMethods()

//获取Class对象中指定函数名和参数的函数，参数name表示函数名；参数parameterTypes表示参数类型列表
public Method getDeclaredMethod(String name, Class<?>... parameterTypes)

//获取Class对象中的所有公有函数（包括从父类和接口类继承的函数）
public Method[] getMethods()

//获取Class对象中指定函数名和参数的公有函数，参数name表示函数名；参数parameterTypes表示参数类型列表
public Method getMethod(String name, Class<?>... parameterTypes)
```
这里需要注意的是 getDeclaredMethod 和 getDeclaredMethods 包含 private、protected、default、public 的函数，并且通过这两个函数获取到的只是在自身中定义的函数，从父类中集成的函数不能够获取到。而 getMethod 和 getMethods 只包含 public 函数，父类中的公有函数也能够获取到。

# 反射类获取类中的属性

## 获取当前类中定义的属性

要获取当前类中定义的所有属性可以通过Class中的getDeclaredFields函数，它会获取到当前类中的public、protected、private的所有属性，而getDeclaredField则是获取指定的属性。

```Java
private static void showDeclaredFields() {

	Student student = new Student("Michael");

	//获取当前类的所有属性
	Field[] fields = student.getClass().getDeclaredFields();

	for(Field field : fields) {
		System.out.println("属性名： " + field.getName());
	}

	try {
		//获取当前类的指定属性
		Field gradeField = student.getClass().getDeclaredField("mGrade");
		//获取属性值
		System.out.println("grade is:" + gradeField.getInt(student));
		//设置属性值
		gradeField.set(student, 10);
		System.out.println("grade is:" + gradeField.getInt(student));

		Field ageField = student.getClass().getDeclaredField("age");
		ageField.setAccessible(true);
		System.out.println("age: " + ageField.get(student));

		ageField.set(student, "26");

		System.out.println("age: " + ageField.get(student));


	} catch (Exception e) {
		// TODO Auto-generated catch block
		e.printStackTrace();
	}
}

结果打印：

属性名： mGrade
属性名： age
grade is:0
grade is:10
age: null
age: 26

```
## 获取当前类、父类中定义的公有属性

要获取当前类以及父类中的所有 public 属性可以通过 Class 中的 getFields 函数，而 getField 则是获取某个指定的属性。代码示例如下 :

```Java
private static void showFields() {

	Student student = new Student("Michael");

	//获取当前类的所有属性
	Field[] fields = student.getClass().getFields();

	for(Field field : fields) {
		System.out.println("属性名： " + field.getName());
	}

	try {
		//获取当前类的指定属性
		Field gradeField = student.getClass().getField("mGrade");
		//获取属性值
		System.out.println("grade is:" + gradeField.getInt(student));
		//设置属性值
		gradeField.set(student, 10);
		System.out.println("grade is:" + gradeField.getInt(student));

		//getField()方法不能获取私有属性
		Field ageField = student.getClass().getField("age");
		//私有属性需调用次函数 JDK 1.8
		ageField.setAccessible(true);
		System.out.println("age: " + ageField.get(student));


	} catch (Exception e) {
		// TODO Auto-generated catch block
		e.printStackTrace();
	}
}

结果打印：

属性名： mGrade
grade is:0
grade is:10
java.lang.NoSuchFieldException: age  ##getField()方法不能获取私有属性
	at java.lang.Class.getField(Unknown Source)
	at com.michael.java.reflection.Main.showFields(Main.java:153)
	at com.michael.java.reflection.Main.main(Main.java:172)

```

**接口说明**

```Java
// 获取 Class 对象中指定属性名的属性，参数一为属性名
public Method getDeclaredField (String name)

// 获取该 Class 对象中的所有属性( 不包含从父类继承的属性 )
public Method[] getDeclaredFields ()

// 获取Class 对象中的指定的公有属性，参数一为属性名
public Method getField (String name)

// 获取该 Class 对象中的所有公有属性 ( 包含从父类和接口类集成下来的公有属性 )
public Method[] getFields ()
```
这里需要注意的是 getDeclaredField 和 getDeclaredFields 包含 private、protected、default、public 的属性，并且通过这两个函数获取到的只是在自身中定义的属性，从父类中集成的属性不能够获取到。而 getField 和 getFields 只包含 public 属性，父类中的公有属性也能够获取到。

# 反射获取父类与接口

## 获取父类

获取Class对象的父类

```Java
private static void showSuperClass() {

	Student student = new Student("Michael");
	Class<?> superClass = student.getClass().getSuperclass();

	while(superClass != null) {
		System.out.println("super class is : " + superClass.getName());

		//再获取父类的上一层父类，知道最后的Object类，Object的父类为null
		superClass = superClass.getSuperclass();
	}
}

打印结果:

super class is : com.michael.java.reflection.Person
super class is : java.lang.Object

```

## 获取接口

```Java
private static void showIntefaces() {
	Student student = new Student("Michael");
	Class<?>[] interfaces = student.getClass().getInterfaces();

	for(Class<?> clz : interfaces) {
		System.out.println("Student的接口有: " + clz.getName());
	}
}

结果打印：

Student的接口有: com.michael.java.reflection.Examination

```

# 获取注解信息

在框架开发中，注解加反射的组合使用是最为常见形式的。定义注解时我们会通过@Target 指定该注解能够作用的类型，看如下示例:

```Java
@Target({
		ElementType.METHOD, ElementType.FIELD, ElementType.TYPE
})
@Retention(RetentionPolicy.RUNTIME)
static @interface Test {

}
```
上述注解的@target 表示该注解只能用在函数上，还有 Type、Field、PARAMETER 等类型，可以参考上述给出的参考资料。通过反射 api 我们也能够获取一个 Class 对象获取类型、属性、函数等相关的对象，通过这些对象的 getAnnotation 接口获取到对应的注解信息。 首先我们需要在目标对象上添加上注解，例如 :

```Java
@Test(tag = "Student class Test Annoatation")
public class Student extends Person implements Examination {
    // 年级
    @Test(tag = "mGrade Test Annotation ")
    int mGrade;

    // ......
}
```

然后通过相关的注解函数得到注解信息，如下所示 :

```Java
private static void getAnnotationInfos() {
	Student student = new Student("mr.simple");
	Test classTest = student.getClass().getAnnotation(Test.class);
	System.out.println("class Annotatation tag = " + classTest.tag());

	Field field = null;
	try {
		field = student.getClass().getDeclaredField("mGrade");
		Test testAnnotation = field.getAnnotation(Test.class);
		System.out.println("属性的 Test 注解 tag : " + testAnnotation.tag());
	} catch (Exception e) {
		e.printStackTrace();
	}
}
```
输出结果为：
```Java
class Annotatation tag = Student class Test Annoatation
属性的 Test 注解 tag : mGrade Test Annotation
```
**接口说明**
```Java
// 获取指定类型的注解
public <A extends Annotation> A getAnnotation(Class<A> annotationClass) ;
// 获取 Class 对象中的所有注解
public Annotation[] getAnnotations() ;
```

# 后记

反射作为 Java 语言的重要特性，在开发中有着极为重要的作用。很多开发框架都是基于反射来实现对目标对象的操作，而反射配合注解更是设计开发框架的主流选择

