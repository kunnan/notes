Java集合之并发容器LinkedBlockingQueue

LinkedBlockingQueue是一个基于已链接节点的、范围任意的阻塞队列的实现。 此队列按 FIFO（先进先出）排序元素。队列的头部 是在队列中时间最长的元素。队列的尾部 是在队列中时间最短的元素。新元素插入到队列的尾部，并且队列检索操作会获得位于队列头部的元素。链接队列的吞吐量通常要高于基于数组的队列， 但是在大多数并发应用程序中，其可预知的性能要低。

可选的容量范围构造方法参数作为防止队列过度扩展的一种方法。如果未指定容量，则它等于 Integer.MAX_VALUE。除非插入节点会使队列超出容量，否则每次插入后会动态地创建链接节点。

# 签名

```Java
public class LinkedBlockingQueue<E> extends AbstractQueue<E>
        implements BlockingQueue<E>, java.io.Serializable
```
LinkedBlockingQueue实现是线程安全的，实现了先进先出等特性，是作为生产者消费者的首选

# 构造函数

```Java
public LinkedBlockingQueue() {
	this(Integer.MAX_VALUE);
}

public LinkedBlockingQueue(int capacity) {
	if (capacity <= 0) throw new IllegalArgumentException();
	this.capacity = capacity;
	last = head = new Node<E>(null);
}
```

# 示例

```Java
public class LinkedBlockingQueueDemo {

	 /**
     * 
     * 定义装苹果的篮子
     * 
     */
    public class Basket {
        // 篮子，能够容纳3个苹果
        BlockingQueue<String> basket = new LinkedBlockingQueue<String>(3);

        // 生产苹果，放入篮子
        public void produce() throws InterruptedException {
            // put方法放入一个苹果，若basket满了，等到basket有位置
            basket.put("An apple"); //阻塞
 //           basket.offer("An Apple"); //非阻塞
         }

        // 消费苹果，从篮子中取走
        public String consume() throws InterruptedException {
            // take方法取出一个苹果，若basket为空，等到basket有苹果为止(获取并移除此队列的头部)
            return basket.take();  //阻塞
            //return basket.poll(); //非阻塞
        }
    }

    // 定义苹果生产者
    class Producer implements Runnable {
        private String instance;
        private Basket basket;

        public Producer(String instance, Basket basket) {
            this.instance = instance;
            this.basket = basket;
        }

        public void run() {
            try {
                while (true) {
                    // 生产苹果
                    System.out.println("生产者准备生产苹果：" + instance);
                    basket.produce();
                    // 休眠300ms
                    Thread.sleep(300);
                }
            } catch (InterruptedException ex) {
                System.out.println("Producer Interrupted");
            }
        }
    }

    // 定义苹果消费者
    class Consumer implements Runnable {
        private String instance;
        private Basket basket;

        public Consumer(String instance, Basket basket) {
            this.instance = instance;
            this.basket = basket;
        }

        public void run() {
            try {
                while (true) {
                    // 消费苹果
                    System.out.println("消费者准备消费苹果：" + instance);
                    System.out.println("consume: " + basket.consume());
                    // 休眠1000ms
//                    Thread.sleep(1000);
                }
            } catch (InterruptedException ex) {
                System.out.println("Consumer Interrupted");
            }
        }
    }

    public static void main(String[] args) {
    	
        LinkedBlockingQueueDemo test = new LinkedBlockingQueueDemo();

        // 建立一个装苹果的篮子
        Basket basket = test.new Basket();

        ExecutorService service = Executors.newCachedThreadPool();
        Producer producer = test.new Producer("生产者001", basket);
//        Producer producer2 = test.new Producer("生产者002", basket);
//        Consumer consumer = test.new Consumer("消费者001", basket);
        service.submit(producer);
//        service.submit(producer2);
//        service.submit(consumer);
        // 程序运行5s后，所有任务停止
//        try {
//            Thread.sleep(1000 * 1);
//        } catch (InterruptedException e) {
//            e.printStackTrace();
//        }
//        service.shutdownNow();
    }


}

```