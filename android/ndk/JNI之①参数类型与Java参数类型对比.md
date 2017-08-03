**JNI之①参数类型与Java参数类型对比**

# 区别

*	Java中的返回值void和JNI中的void完全对应（仅仅一个）

*	Java中的基本数据类型（byte ,short ,int,long,float,double ,boolean,char－8种）在JNI中对应的数据类型只要在前面加上j就可以了（jbyte ,jshort ,jint,jlong,jfloat,jdouble ,jboolean,jchar）

*	Java中的对象，包括类库中定义的类、接口以及自定义类接口，都对应于JNI中的jobject

*	Java中的基本数据类型对应的数组对应于JNI中的j<type>Array类型（type就是8中基本数据类型）

*	Java中对象的数组对应于JNI中的jobjectArray类型

# 基本数据类型映射图

![ALT TEXT](http://hi.csdn.net/attachment/201111/10/0_13209061602sLG.gif)

# 引用数据类型的映射图

![ALT TEXT](http://hi.csdn.net/attachment/201111/10/0_1320906175mxw4.gif)

