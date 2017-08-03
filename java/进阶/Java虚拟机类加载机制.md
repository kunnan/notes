**Java虚拟机类加载机制**

[TOC]

# Java 程序的执行过程

Java 程序的执行过程：Java 源代码文件（.Java文件）-> Java Compiler（Java编译器）->Java 字节码文件（.class文件）->类加载器（Class Loader）->Runtime Data Area（运行时数据）-> Execution Engine（执行引擎）。

虚拟机把描述类的数据从 Class 文件中加载到内存，并对数据进行校验、转换解析和初始化，最终形成可以被虚拟机直接使用的Java类型，这就是虚拟机的类加载机制。

# 类的生命周期

![](http://images0.cnblogs.com/blog2015/544748/201505/251937420687138.jpg)

# 类文件来源

* 从JAR包加载class文件
* 从本地文件系统加载class文件
* 从网络加载class文件
* 把一个Java源文件动态编译，并执行加载

# 类加载过程

类加载的全过程包括加载、连接（包括验证、准备、解析）和初始化五个阶段。

## 加载阶段

在加载阶段虚拟机需要完成3件事情：

1. 通过一个类的全限定名来获取定义此类的二进制字节流。
2. 将这个字节流所代表的静态存储结构转化为方法区的运行时数据结构。
3. 在内存中生成一个代表这个类的java.lang.Class对象，作为方法区这个类的各种数据的访问入口。

## 连接阶段

* 验证：确保Class文件的字节流中包含的信息符合当前虚拟机的要求
    * 文件格式验证
    * 元数据验证
    * 字节码验证
    * 符号引用验证
* 准备：正式为类变量分配内存，并根据类变量类型设置默认值（通常情况下是数据类型的零值）；
* 解析：虚拟机将常量池内的符号引用直接替换为直接引用的过程；
    * 类或接口的解析
    * 字段解析
    * 类方法解析
    * 接口方法解析

## 初始化阶段

初始化阶段是执行类构造器 `<clinit>`方法的过程，给类的所有变量赋值。

# 类什么时候才被初始化

只有下列情况会导致类的初始化：

* new 关键字实例化对象
* 访问某个类或接口的静态变量（除final修饰的静态字段），或者对该静态变量赋值
* 调用类的静态方法
* 反射
* 初始化一个类的子类（首先会初始化父类）
* JVM启动时标明的启动类即包含 main()方法的类

这些情况称为对一个类的主动引用，除此之外，所有引用类的方式都不会触发初始化，称为被动引用。

# 类加载器

## 分类

* 启动类加载器（Bootstrap ClassLoader）：c++编写，负责加载Java核心类，如String、System等；
* 扩展类加载器（Extension ClassLoade）：Java编写，负责加载JRE的扩展类库；
* 系统类加载器（System ClassLoade）：Java编写，负责加载CLASSPATH环境变量所指定的类库；一般情况下就是程序中默认的类加载器；
* 用户类加载器：自定义类加载器，继承ClassLoader

## 双亲委派模型

![类加载器双亲委派模型](http://img.blog.csdn.net/20160102154038185)

双亲委派模型要求除了顶层的启动类加载器外，其余的类加载器都应当有自己的父类加载器。这里类加载之间的父子关系一般不会以继承的关系来实现，而是都使用组合关系来复用父加载器的代码。

双亲委派模型的**工作过程**：如果一个类加载器收到了类加载的请求，它首先不会尝试自己去加载这个类，而是把这个请求委派给父类加载器去完成，每一个层次的类加载器都是如此，因此所有的加载请求最终都应该传送到顶层的启动类加载器中，只有当父类加载器反馈自己无法完成这个加载请求（它的搜索范围没有找到所需的类）时，子加载器才回尝试自己去加载。这样就保证了**加载类在程序的各种类加载器环境中都是同一个类，相反如果没有使用,各个类加载器自行加载的话。那么系统将出现多个不同的加载类,那么java类型体系中最基本的行为也无法保证**。

```Java
protected synchronized Class<?> loadClass(String name, boolean resolve)
	throws ClassNotFoundException
    {
	// 首先，检查请求的类是否已经被加载过了
	Class c = findLoadedClass(name);
	if (c == null) {
	    try {
		if (parent != null) {
		    // 优先让 parent 加载器去加载
		    c = parent.loadClass(name, false);
		} else {
		    /// 如无 parent，表示当前是 BootstrapClassLoader，调用 native 方法去 JVM 加载
		    c = findBootstrapClassOrNull(name);
		}
	    } catch (ClassNotFoundException e) {
                //父类和启动类加载器均无法完成加载请求
            }
            if (c == null) {
	        // 如果 parent 均没有加载到目标 class，调用自身的 findClass() 方法去搜索
	        c = findClass(name);
	    }
	}
	if (resolve) {
	    resolveClass(c);
	}
	return c;
}
```

# 类加载的三种方式

1. 通过JVM初始化加载，new 关键字创建一个类的实例
2. 通过Class.forName()方法动态加载，通过反射加载类型，并创建实例
3. 通过ClassLoader.loadClass()方法动态加载，应用程序可以通过继承 ClassLoader 实现自己的类装载器

三者区别：

* 1和2使用的类加载器是相同的，都是当前类加载器。（即：this.getClass.getClassLoader）。
* 3由用户指定类加载器。如果需要在当前类路径以外寻找类，则只能采用第3种方式。第3种方式加载的类与当前类分属不同的命名空间。

对于相同的类，JVM最多会载入一次。但如果同一个class文件被不同的ClassLoader载入，那么载入后的两个类是完全不同的。

# 参考资料

《深入理解Java虚拟机》

