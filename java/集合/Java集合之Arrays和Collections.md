# Arrays

java.util.Arrays

[Arrays详细介绍](http://www.apihome.cn/api/java/Arrays.html)

Array是Java特有的数组。在你知道所要处理数据元素个数的情况下非常好用。java.util.Arrays 包含了许多处理数据的实用方法：

Arrays.asList：可以从 Array 转换成 List。可以作为其他集合类型构造器的参数。

Arrays.binarySearch：在一个已排序的或者其中一段中快速查找。

Arrays.copyOf：如果你想扩大数组容量又不想改变它的内容的时候可以使用这个方法。

Arrays.copyOfRange：可以复制整个数组或其中的一部分。

Arrays.deepEquals、Arrays.deepHashCode：Arrays.equals/hashCode的高级版本，支持子数组的操作。

Arrays.equals：如果你想要比较两个数组是否相等，应该调用这个方法而不是数组对象中的 equals方法（数组对象中没有重写equals()方法，所以这个方法之比较引用而不比较内容）。这个方法集合了Java 5的自动装箱和无参变量的特性，来实现将一个变量快速地传给 equals() 方法——所以这个方法在比较了对象的类型之后是直接传值进去比较的。

Arrays.fill：用一个给定的值填充整个数组或其中的一部分。

Arrays.hashCode：用来根据数组的内容计算其哈希值（数组对象的hashCode()不可用）。这个方法集合了Java 5的自动装箱和无参变量的特性，来实现将一个变量快速地传给 Arrays.hashcode方法——只是传值进去，不是对象。

Arrays.sort：对整个数组或者数组的一部分进行排序。也可以使用此方法用给定的比较器对对象数组进行排序。

Arrays.toString：打印数组的内容。

如果想要复制整个数组或其中一部分到另一个数组，可以调用 System.arraycopy方法。此方法从源数组中指定的位置复制指定个数的元素到目标数组里。这无疑是一个简便的方法。（有时候用 ByteBuffer bulk复制会更快。可以参考这篇文章）.

最后，所有的集合都可以用T[] Collection.toArray( T[] a ) 这个方法复制到数组中。通常会用这样的方式调用：
```
return coll.toArray( new T[ coll.size() ] );
```
这个方法会分配足够大的数组来储存所有的集合，这样 toArray 在返回值时就不必再分配空间了。

# Collections

java.util.Collections

[Collections详细介绍](http://www.apihome.cn/api/java/Collections.html)

就像有专门的java.util.Arrays来处理数组，Java中对集合也有java.util.Collections来处理。

第一组方法主要返回集合的各种数据：

Collections.checkedCollection / checkedList / checkedMap / checkedSet / checkedSortedMap / checkedSortedSet：检查要添加的元素的类型并返回结果。任何尝试添加非法类型的变量都会抛出一个ClassCastException异常。这个功能可以防止在运行的时候出错。//fixme

Collections.emptyList / emptyMap / emptySet ：返回一个固定的空集合，不能添加任何元素。

Collections.singleton / singletonList / singletonMap：返回一个只有一个入口的 set/list/map 集合。

Collections.synchronizedCollection / synchronizedList / synchronizedMap / synchronizedSet / synchronizedSortedMap / synchronizedSortedSet：获得集合的线程安全版本（多线程操作时开销低但不高效，而且不支持类似put或update这样的复合操作）

Collections.unmodifiableCollection / unmodifiableList / unmodifiableMap / unmodifiableSet / unmodifiableSortedMap / unmodifiableSortedSet：返回一个不可变的集合。当一个不可变对象中包含集合的时候，可以使用此方法。


第二组方法中，其中有一些方法因为某些原因没有加入到集合中：

Collections.addAll：添加一些元素或者一个数组的内容到集合中。

Collections.binarySearch：和数组的Arrays.binarySearch功能相同。

Collections.disjoint：检查两个集合是不是没有相同的元素。

Collections.fill：用一个指定的值代替集合中的所有元素。

Collections.frequency：集合中有多少元素是和给定元素相同的。

Collections.indexOfSubList / lastIndexOfSubList：和String.indexOf(String) / lastIndexOf(String)方法类似——找出给定的List中第一个出现或者最后一个出现的子表。

Collections.max / min：找出基于自然顺序或者比较器排序的集合中，最大的或者最小的元素。

Collections.replaceAll：将集合中的某一元素替换成另一个元素。

Collections.reverse：颠倒排列元素在集合中的顺序。如果你要在排序之后使用这个方法的话，在列表排序时，最好使用Collections.reverseOrder比较器。

Collections.rotate：根据给定的距离旋转元素。

Collections.shuffle：随机排放List集合中的节点，可以给定你自己的生成器——例如java.util.Random / java.util.ThreadLocalRandom or java.security.SecureRandom。

Collections.sort：将集合按照自然顺序或者给定的顺序排序。

Collections.swap：交换集合中两个元素的位置（多数开发者都是自己实现这个操作的）。

