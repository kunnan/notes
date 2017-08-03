Java IO详解
==

[TOC]

# 概述

IO流用来处理设备之间的数据传输，Java对数据的操作是通过流的方式，Java用于操作流的对象都在IO包中。

流按操作数据分为两种：字节流与字符流
流按流向分为：输入流、输出流

# IO流常用基类

## 字符流的抽象基类
Reader
Writer

>注：由这四个类派生出来的子类名称都是以其父类名作为子类名的后缀
如：InputStream的子类FileInputStream
如：Reader的子类FileReader

## 字节流的抽象基类
InputStream
OutputStream

# 字符流操作

## FileWriter字符流写入

FileWriter 后缀是父类名 前缀名是功能

FileWriter写文件步骤:

1. 创建一个FileWrite对象，该对象一初始化就必须要明确被操作的文件，而且该文件会被创建到指定目录，如果该目录下已有同名文件则被覆盖`FileWrite fw = new FileWrite("demo.txt")`；

2. 调用write方法将字符串写入到流中`fw.write("abcde")`;

3. 刷新流对象中的缓冲中的数据,将数据刷到目的地中
`fw.flush()`;//flush刷新后，流可以继续使用，close刷新后，流将会关闭
`fw.close()`;//关闭流资源，但关闭之前会刷新一次内部的缓冲中的数据，最后必须要关闭流

代码示例

```
/*
需求:在硬盘上，创建一个文件并写入一些文字数据。
找到一个专门用于操作文件的Writer子类对象。FileWriter。  后缀名是父类名。 前缀名是该流对象的功能。
*/
import java.io.*;
class  FileWriterDemo
{
	public static void main(String[] args) throws IOException
	{
		//创建一个FileWriter对象。该对象一被初始化就必须要明确被操作的文件。
		//而且该文件会被创建到指定目录下。如果该目录下已有同名文件，将被覆盖。
		//其实该步就是在明确数据要存放的目的地。
		FileWriter fw = new FileWriter("demo.txt");
		//调用write方法，将字符串写入到流中。
		fw.write("abcde");
		//刷新流对象中的缓冲中的数据。
		//将数据刷到目的地中。
		//fw.flush();
		//关闭流资源，但是关闭之前会刷新一次内部的缓冲中的数据。
		//将数据刷到目的地中。
		//和flush区别：flush刷新后，流可以继续使用，close刷新后，会将流关闭。
		fw.close();
	}
}

/*
演示对已有文件的数据续写。
*/
import java.io.*;
class  FileWriterDemo3
{
	public static void main(String[] args) throws IOException  //不能抛，要处理异常//传递一个true参数，代表不覆盖已有的文件。并在已有文件的末尾处进行数据续写。
		FileWriter fw = new FileWriter("demo.txt",true);
		fw.write("nihao\r\nxiexie");  //换行续写
		fw.close();
	}
}
```

## FileReader字符流读取

```
import java.io.*;
class  FileReaderDemo
{
	public static void main(String[] args) throws IOException
	{
		//创建一个文件读取流对象，和指定名称的文件相关联。
		//要保证该文件是已经存在的，如果不存在，会发生异常FileNotFoundException
		FileReader fr = new FileReader("demo.txt");
		//调用读取流对象的read方法。
		//read():一次读一个字符。而且会自动往下读。		
		int ch = 0;
		while((ch=fr.read())!=-1)
		{
			System.out.println("ch="+(char)ch);
		}
		/*
		while(true)
		{
			int ch = fr.read();
			if(ch==-1)
				break;
			System.out.println("ch="+(char)ch);
		}
		*/
		fr.close();
	}
}
```

```
/*
第二种方式：通过字符数组进行读取。 
  常用这种
*/
import java.io.*;
class FileReaderDemo2 
{
	public static void main(String[] args) throws IOException
	{
		FileReader fr = new FileReader("demo.txt");
		
		//定义一个字符数组。用于存储读到字符。
		//该read(char[])返回的是读到字符个数。
		char[] buf = new char[1024];
		int num = 0;
		while((num=fr.read(buf))!=-1)
		{
			System.out.println(new String(buf,0,num));
		}
		
		fr.close();
	}
}

```

## 文件拷贝

```

//将C盘一个文本文件复制到D盘。
/*
复制的原理：
其实就是将C盘下的文件数据存储到D盘的一个文件中。
步骤：
1，在D盘创建一个文件。用于存储C盘文件中的数据。
2，定义读取流和C盘文件关联。
3，通过不断的读写完成数据存储。
4，关闭资源。
*/
import java.io.*;
class CopyText 
{
	public static void main(String[] args) throws IOException
	{
		copy_2();
	}
	public static void copy_2()
	{
		FileWriter fw = null;
		FileReader fr = null;
		try
		{
			fw = new FileWriter("SystemDemo_copy.txt");
			fr = new FileReader("SystemDemo.java");
			char[] buf = new char[1024];
			int len = 0;
			while((len=fr.read(buf))!=-1)
			{
				fw.write(buf,0,len);
			}
		}
		catch (IOException e)
		{
			throw new RuntimeException("读写失败");
		}
		finally
		{
			if(fr!=null)
				try
				{
					fr.close();
				}
				catch (IOException e)
				{
				}
			if(fw!=null)
				try
				{
					fw.close();
				}
				catch (IOException e)
				{
				}
		}
	}
	//从C盘读一个字符，就往D盘写一个字符。
	public static void copy_1()throws IOException
	{
		//创建目的地。
		FileWriter fw = new FileWriter("RuntimeDemo_copy.txt");
		//与已有文件关联。
		FileReader fr = new FileReader("RuntimeDemo.java");
		int ch = 0;
		while((ch=fr.read())!=-1)
		{
			fw.write(ch);
		}
		
		fw.close();
		fr.close();
	}
}
```

## BufferedReader字符流缓冲区读

从字符输入流中读取文本，缓冲各个字符，从而实现字符、数组和行的高效读取
可以指定缓冲区的大小，或者可以使用默认大小。大多数情况下，默认值就足够大了

```
/*
字符读取流缓冲区：
该缓冲区提供了一个一次读一行的方法 readLine，方便于对文本数据的获取。
当返回null时，表示读到文件末尾。
readLine方法返回的时候只返回回车符之前的数据内容。并不返回回车符。
*/
import java.io.*;
class  BufferedReaderDemo
{
	public static void main(String[] args) throws IOException
	{
		//创建一个读取流对象和文件相关联。
		FileReader fr = new FileReader("buf.txt");
		//为了提高效率。加入缓冲技术。将字符读取流对象作为参数传递给缓冲对象的构造函数。
		BufferedReader bufr = new BufferedReader(fr);
		
		String line = null;
		while((line=bufr.readLine())!=null)
		{
			System.out.print(line);
		}
		bufr.close();
	}
}

```

## BufferedWriter字符流缓冲区读

将文本写入字符输出流，缓冲各个字符，从而提供单个字符、数组和字符串的高效写入
可以指定缓冲区的大小，或者接受默认的大小。在大多数情况下，默认值就足够大了
通常 Writer 将其输出立即发送到底层字符或字节流。除非要求提示输出，否则建议用 BufferedWriter 包装所有其 write() 操作可能开销很高的 Writer（如 FileWriters 和 OutputStreamWriters）。

```
/*
缓冲区的出现是为了提高流的操作效率而出现的。
所以在创建缓冲区之前，必须要先有流对象。
该缓冲区中提供了一个跨平台的换行符。
newLine();
*/
import java.io.*;
class  BufferedWriterDemo
{
	public static void main(String[] args) throws IOException
	{
		//创建一个字符写入流对象。
		FileWriter fw = new FileWriter("buf.txt");
		//为了提高字符写入流效率。加入了缓冲技术。
		//只要将需要被提高效率的流对象作为参数传递给缓冲区的构造函数即可。
		BufferedWriter bufw = new BufferedWriter(fw);
		for(int x=1; x<5; x++)
		{
			bufw.write("abcd"+x);
			bufw.newLine(); //写入一个行分隔符
			bufw.flush();
		}
		//记住，只要用到缓冲区，就要记得刷新。
		//bufw.flush();
		//其实关闭缓冲区，就是在关闭缓冲区中的流对象。
		bufw.close();
	}
}
```

## 通过缓冲区复制文本

```


/*
通过缓冲区复制一个.java文件。
*/
import java.io.*;
class  CopyTextByBuf
{
	public static void main(String[] args) 
	{
		BufferedReader bufr = null;
		BufferedWriter bufw = null;
		try
		{
			bufr = new BufferedReader(new FileReader("BufferedWriterDemo.java"));
			bufw = new BufferedWriter(new FileWriter("bufWriter_Copy.txt"));
			String line = null;
			while((line=bufr.readLine())!=null)
			{
				bufw.write(line);
				bufw.newLine();
				bufw.flush();
			}
		}
		catch (IOException e)
		{
			throw new RuntimeException("读写失败");
		}
		finally
		{
			try
			{
				if(bufr!=null)
					bufr.close();
			}
			catch (IOException e)
			{
				throw new RuntimeException("读取关闭失败");
			}
			try
			{
				if(bufw!=null)
					bufw.close();
			}
			catch (IOException e)
			{
				throw new RuntimeException("写入关闭失败");
			}
		}
	}
}
}
```

# 字节流操作

FileInputStream 从文件系统中的某个文件中获得输入字节。哪些文件可用取决于主机环境。
FileInputStream 用于读取诸如图像数据之类的原始字节流。要读取字符流，请考虑使用 FileReader。

FileOutputStream文件输出流是用于将数据写入 File 或 FileDescriptor 的输出流。文件是否可用或能否可以被创建取决于基础平台。特别是某些平台一次只允许一个 FileOutputStream（或其他文件写入对象）打开文件进行写入。在这种情况下，如果所涉及的文件已经打开，则此类中的构造方法将失败。
FileOutputStream 用于写入诸如图像数据之类的原始字节的流。要写入字符流，请考虑使用 FileWriter。

## FileInputStream和FileOutputStream

```
字节流：
InputStream  OutputStream
需求，想要操作图片数据。这时就要用到字节流。
复制一个图片.
*/
import java.io.*;
class  FileStream
{
	public static void main(String[] args) throws IOException
	{
		readFile_3();
	}
	public static void readFile_3()throws IOException
	{
		FileInputStream fis = new FileInputStream("fos.txt");
		
//		int num = fis.available();
		byte[] buf = new byte[fis.available()];//定义一个刚刚好的缓冲区。不用在循环了。
		fis.read(buf);
		System.out.println(new String(buf));
		fis.close();
	}
	public static void readFile_2()throws IOException
	{
		FileInputStream fis = new FileInputStream("fos.txt");
		byte[] buf = new byte[1024];
		int len = 0;
		while((len=fis.read(buf))!=-1)
		{
			System.out.println(new String(buf,0,len));
		}
		fis.close();
		
	}
	public static void readFile_1()throws IOException
	{
		FileInputStream fis = new FileInputStream("fos.txt");
		int ch = 0;
		while((ch=fis.read())!=-1)
		{
			System.out.println((char)ch);
		}
		fis.close();
	}
	public static void writeFile()throws IOException
	{
		FileOutputStream fos = new FileOutputStream("fos.txt");
		
		fos.write("abcde".getBytes());
		fos.close();
		
	}
}
```

## 复制图片

```



/*
复制一个图片
思路：
1，用字节读取流对象和图片关联。
2，用字节写入流对象创建一个图片文件。用于存储获取到的图片数据。
3，通过循环读写，完成数据的存储。
4，关闭资源。
*/
import java.io.*;
class  CopyPic
{
	public static void main(String[] args) 
	{
		FileOutputStream fos = null;
		FileInputStream fis = null;
		try
		{
			fos = new FileOutputStream("c:\\2.bmp");
			fis = new FileInputStream("c:\\1.bmp");
			byte[] buf = new byte[1024];
			int len = 0;
			while((len=fis.read(buf))!=-1)
			{
				fos.write(buf,0,len);
			}
		}
		catch (IOException e)
		{
			throw new RuntimeException("复制文件失败");
		}
		finally
		{
			try
			{
				if(fis!=null)
					fis.close();
			}
			catch (IOException e)
			{
				throw new RuntimeException("读取关闭失败");
			}
			try
			{
				if(fos!=null)
					fos.close();
			}
			catch (IOException e)
			{
				throw new RuntimeException("写入关闭失败");
			}
		}
	}
}

```

## 字节流的缓冲区

```



/*
演示mp3的复制。通过缓冲区。
BufferedOutputStream
BufferedInputStream
*/
import java.io.*;
class  CopyMp3
{
	public static void main(String[] args) throws IOException
	{
		long start = System.currentTimeMillis();
		copy_2();
		long end = System.currentTimeMillis();
		System.out.println((end-start)+"毫秒");
	}
	public static void copy_2()throws IOException
	{
		MyBufferedInputStream bufis = new MyBufferedInputStream(new FileInputStream("c:\\9.mp3"));
		BufferedOutputStream bufos = new BufferedOutputStream(new FileOutputStream("c:\\3.mp3"));
		
		int by = 0;
		//System.out.println("第一个字节:"+bufis.myRead());
		while((by=bufis.myRead())!=-1)
		{
			bufos.write(by);
		}
		bufos.close();
		bufis.myClose();
	}
	//通过字节流的缓冲区完成复制。
	public static void copy_1()throws IOException
	{
		BufferedInputStream bufis = new BufferedInputStream(new FileInputStream("c:\\0.mp3"));
		BufferedOutputStream bufos = new BufferedOutputStream(new FileOutputStream("c:\\1.mp3"));
		
		int by = 0;
		while((by=bufis.read())!=-1)
		{
			bufos.write(by);
		}
		bufos.close();
		bufis.close();
		
	}
}

```

# 打印流

## 字节打印流PrintStream

构造函数可以接收的参数类型：
1. File对象。File
2. 字符串路径。String
3. 字节输出流。OutputStream

## 字符打印流PrintWriter

构造函数可以接收的参数类型：
1. File对象。File
2. 字符串路径。String
3. 字节输出流。OutputStream
4. 字符输出流，Writer。

## 示例

```
/*
在没有刷新前，你写入的数据并没有真正写入文件，只是保存在内存中。刷新后才会写入文件，如果程序中没有调用刷新方法，当程序执行完时会自动刷新，也就是只有到数据全部执行完才会一次性写入，大数据量时对运行效率有影响。
创建不具有自动行刷新的对象，就是用这个对象写入数据时不会自动刷新。
*/
import java.io.*;
class  PrintStreamDemo
{
	public static void main(String[] args) throws IOException
	{
		BufferedReader bufr = 
			new BufferedReader(new InputStreamReader(System.in));
		PrintWriter out = new PrintWriter(new FileWriter("a.txt"),true);
		String line = null;
		while((line=bufr.readLine())!=null)
		{
			if("over".equals(line))
				break;
			out.println(line.toUpperCase());
			//out.flush();
		}
		out.close();
		bufr.close();
	}	
}
```

