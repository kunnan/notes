# 注解

## 概念

注解（Annotation），也叫元数据。一种代码级别的说明。它是JDK1.5及以后版本引入的一个特性，与类、接口、枚举是在同一个层次。它可以声明在包、类、字段、方法、局部变量、方法参数等的前面，用来对这些元素进行说明，注释。

## 作用

-	标记作用

用于告诉编译器一些信息让编译器能够实现基本的编译检查，如@Override、@Deprecated，

```Java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.SOURCE)
public @interface Override {
}

@Documented
@Retention(RetentionPolicy.RUNTIME)
public @interface Deprecated {
}
```
-	编译时动态处理

动态生成代码，如ButterKnife、Dagger2

-	运行时动态处理

获取注解信息，如Retrofit

## 分类

### 按功能分类

-	标准注解

是指Java自带的几个注解，Override、Deprecated、SuppressWarnings，分别表示重写方法、不推荐使用（过时的）、忽略某项Warning.

-	元注解

元注解是指用来定义注解的注解，JDK1.5定义了四种元注解：@Retention、@Target、@Inherit、@Documented。

-	自定义注解（meta-annotation）

自定义注解表示自己根据需要定义的注解，定义时需要用到上面的元注解。

### 按作用域分类

-	源码时注解（RetentionPolicy.SOURCE）
-	编译时注解（RetentionPolicy.CLASS）
-	运行时注解（RetentionPolicy.RUNTIME）

## 注解相关知识点

### 元注解知识点

-	@Target：指Annotation所修饰的对象范围，通过ElementType取值有8中，如下：

| 取值 | 含义 |
|--------|--------|
|  TYPE      |    类、接口（包括注解类型）或枚举   |
|  FILED      |   属性     |
|  METHOD      |  方法      |
|  PARAMETER      |  参数      |
|  CONSTRUCTOR      |   构造函数     |
|  LOCAL_VARIABLE      |     局部变量   |
|  ANNOTATION_TYPE      |      注解类型  |
|  PACKAGE      |    包    |

-	@Retention:指Annotation被保留的时间长短，通过RetentionPolicy取值有3种，如

| 值 | 含义 |
|--------|--------|
|   SOURCE     |   在源文件有效，编译时就会忽略|
|   CLASS     |    在Class文件中有效，但JVM将会忽略  如Override、SuppressWarnings等  |
|   RUNTIME     |  在运行时有效 ，所以它们能在运行时被JVM或其他使用反射机制的代码所读取和使用|

-	@Documented：是一个标记注解，表明这个注解应该被 javadoc工具记录. 默认情况下,javadoc是不包括注解的. 但如果声明注解时指定了 @Documented,则它会被 javadoc 之类的工具处理, 所以注解类型信息也会被包括在生成的文档中.

-	@Inherited：也是一个标记注解，@Inherited阐述了某个被标注的类型是被继承的，默认为false

### 注解定义格式

```Java
public @interface 注解名 {定义体}
```
### 注解支持的数据类型

1. 8种基本数据类型 int、float、boolean、byte、double、char、long、short  
2. String、Class、enum、Annotation
3. 以上所有类型的数组

### 注意

自定义注解如果只有一个参数成员，最好把定义体参数名称设为"value"，如@Target

```Java
@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.ANNOTATION_TYPE)
public @interface Target {
    ElementType[] value();
}
```

## Annotation自定义

### 调用

```Java
public class App {

    @MethodInfo(
        author = “trinea.cn+android@gmail.com”,
        date = "2014/02/14",
        version = 2)
    public String getAppName() {
        return "trinea";
    }
}
```

MethodInfo Annotation作用为给方法添加相关信息，包括author、date、version

### 定义

```Java
@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
@Inherited
public @interface MethodInfo {

    String author() default "trinea@gmail.com";

    String date();

    int version() default 1;
}
```
上面是MethondInfo的实现部分：
1. 通过@interface定义，注解名即为自定义注解名
2. 注解配置参数名为注解类的方法名且
	-	所有方法没有方法体，没有参数没有修饰符，实际只允许public&abstract修饰符，默认为public，不允许抛出异常
	-	方法返回值只能是基本类型、String、Class、Annotation、Enum或者是它们的一维数组
	-	若只有一个默认属性，可直接用value（）函数，一个属性都没有表示该 Annotation 为 Mark Annotation
3. 可以加default表示默认值

## Annotation解析

### 运行时Annotation解析

运行时Annotation指@Retention为RUNTIME的Annotation，可手动调用下面常用API解析：

-	method.getAnnotation(AnnotationName.class)

表示得到该Target某个Annotation的信息，因为一个Target可以被多个Annotation修饰

-	method.getAnnotations()

表示得到该Target所有Annotation

-	method.isAnnotationPresent(AnnotationName.class)

表示该Target是否被某个Annotation修饰

#### @Target为METHOND

解析示例:

```Java
public class App {
	
    @MethodInfo(
            author = "trinea.cn+android@gmail.com",
            date = "2014/02/14",
            version = 2)
	public String getAppName() {
		return "trinea";
	}
		
    @MethodInfo(author = "liu.cn+android@gmail.com",
            date = "2015/02/14",
            version = 3)
    public String getNewAppName() {
    	return "liu";
    }

	/**
	 * @param args
	 */
	public static void main(String[] args) {
		// TODO Auto-generated method stub
		
		try {
			Class<?> clz = Class.forName("com.michael.java.annotation.App");
			
			for(Method method : clz.getMethods()) {
				
				MethodInfo methodInfo = method.getAnnotation(MethodInfo.class);
				
				if (methodInfo != null) {
					System.out.println(method.getName());
					System.out.println(methodInfo.author());
					System.out.println(methodInfo.date());
					System.out.println(methodInfo.version());
					
					Annotation[] annotations = method.getAnnotations();
					for(Annotation annotation : annotations){
						System.out.println(annotation.toString());
					}
				}
				
			}
			
		} catch (ClassNotFoundException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}

	}
}

运行结果：

getAppName
trinea.cn+android@gmail.com
2014/02/14
2
@com.michael.java.annotation.MethodInfo(version=2, author=trinea.cn+android@gmail.com, date=2014/02/14)

getNewAppName
liu.cn+android@gmail.com
2015/02/14
3
interface com.michael.java.annotation.MethodInfo

```
#### @Target为FIELD

```Java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.FIELD)
public @interface Bind {
	
	int value();
}

public class FieldDemo {
	
	@Bind(2)
	public static int age;

	/**
	 * @param args
	 */
	public static void main(String[] args) {
		// TODO Auto-generated method stub
		
		Class<?> clz = new FieldDemo().getClass();
		
		for(Field field : clz.getFields()) {
			
			Bind bind = field.getAnnotation(Bind.class);
			
			if (bind != null) {
				System.out.println(field.getName());
				System.out.println(bind.value());
				age = bind.value();
				System.out.println(age);
			}
		}

	}

}

运行结果：

age
2
2

```
### 编译时Annotation解析

编译时Annotation指@Retention为CLASS的Annotation，由编译器自动解析。步骤为：

1. 自定义类继承自AbstractProcessor
2. 重写其中的process函数

编译器在编译时自动查找所有继承自AbstractProcessor的类，然后调用它们的process方法去处理

假设MethodInfo的@Retention为CLASS，解析示例如下：

```Java
@SupportedAnnotationTypes({ "com.michael.java.annotation.MethodInfo" })
public class MethodInfoProcessor extends AbstractProcessor {

    @Override
    public boolean process(Set<? extends TypeElement> annotations, RoundEnvironment env) {
        HashMap<String, String> map = new HashMap<String, String>();
        for (TypeElement te : annotations) {
            for (Element element : env.getElementsAnnotatedWith(te)) {
                MethodInfo methodInfo = element.getAnnotation(MethodInfo.class);
                map.put(element.getEnclosingElement().toString(), methodInfo.author());
            }
        }
        return false;
    }
}
```
SupportedAnnotationTypes 表示这个 Processor 要处理的 Annotation 名字。
process 函数中参数 annotations 表示待处理的 Annotations，参数 env 表示当前或是之前的运行环境
process 函数返回值表示这组 annotations 是否被这个 Processor 接受，如果接受后续子的 rocessor 不会再对这个 Annotations 进行处理。

# 依赖注入

## 依赖

如果在Class A中，有Class B的实例，则称Class A对Class B有一个依赖。例如下面类Human中用到一个Father对象，我们就说Human对类Father有一个依赖。

```Java
public class Human {
    ...
    Father father;
    ...
    public Human() {
        father = new Father();
    }
}
```
仔细看这段代码我们会发现存在一些问题：

1. 如果现在要改变 father 生成方式，如需要用new Father(String name)初始化 father，需要修改 Human 代码；
2. 如果想测试不同 Father 对象对 Human 的影响很困难，因为 father 的初始化被写死在了 Human 的构造函数中；
3. 如果new Father()过程非常缓慢，单测时我们希望用已经初始化好的 father 对象 Mock 掉这个过程也很困难。

## 依赖注入

上面将依赖在构造函数中直接初始化一种Hard init形式，弊端在于两个类不够独立，不方便测试。我们还有另外一种init方式，如下：

```Java
public class Human {
    ...
    Father father;
    ...
    public Human(Father father) {
        this.father = father;
    }
}
```
上面代码中，我们将 father 对象作为构造函数的一个参数传入。在调用 Human 的构造方法之前外部就已经初始化好了 Father 对象。**像这种非自己主动初始化依赖，而通过外部来传入依赖的方式，我们就称为依赖注入。**
现在我们发现上面 1 中存在的两个问题都很好解决了，简单的说依赖注入主要有两个好处：

1. 解耦，将依赖之间解耦。
2. 因为已经解耦，所以方便做单元测试，尤其是 Mock 测试。

## Java中的依赖注入

依赖注入的实现有很多途径，而在Java中，使用注解是最常用的。通过在字段的声明前添加@Inject注解进行标记，来实现对依赖对象的自动注入。

```Java
public class Human {
    ...
    @Inject Father father;
    ...
    public Human() {
    }
}
```

面这段代码看起来很神奇：只是增加了一个注解，Father 对象就能自动注入了？这个注入过程是怎么完成的？
实质上，如果你只是写了一个 @Inject 注解，Father 并不会被自动注入。你还需要使用一个依赖注入框架，并进行简单的配置。现在 Java 语言中较流行的依赖注入框架有 Google Guice、Spring 等，而在 Android 上比较流行的有 RoboGuice、Dagger、Dagger2 等
