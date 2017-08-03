**Otto源码解析**


首先来看看构造函数：

# 构造函数

```Java

private final String identifier;

private final ThreadEnforcer enforcer;

private final HandlerFinder handlerFinder;

public Bus() {
    this(DEFAULT_IDENTIFIER);
}

public Bus(String identifier) {
    this(ThreadEnforcer.MAIN, identifier);
}

public Bus(ThreadEnforcer enforcer, String identifier) {
    this(enforcer, identifier, HandlerFinder.ANNOTATED);
}

Bus(ThreadEnforcer enforcer, String identifier, HandlerFinder handlerFinder) {
    this.enforcer =  enforcer;
    this.identifier = identifier;
    this.handlerFinder = handlerFinder;
}
```
默认参数enforcer=ThreadEnforcer.MAIN，identifier=DEFAULT_IDENTIFIER，handlerFinder=HandlerFinder.ANNOTATED。下面来看看这些参数是什么意思：

## ThreadEnforce

ThreadEnforce是一个接口，enforce()方法用于检查当前的线程是否为指定的线程类型

```Java
public interface ThreadEnforcer {

    ThreadEnforcer ANY = new ThreadEnforcer() {
            @Override
            public void enforce(Bus bus) {
                // Allow any thread.
            }
        };

    ThreadEnforcer MAIN = new ThreadEnforcer() {
            @Override
            public void enforce(Bus bus) {
                if (Looper.myLooper() != Looper.getMainLooper()) {
                    throw new IllegalStateException("Event bus " + bus +
                        " accessed from non-main thread " + Looper.myLooper());
                }
            }
        };

    void enforce(Bus bus);
}
```
不带参数的构造函数默认使用ThreadEnforcer.MAIN，表示enforce()方法必须在主线程上执行。

## identifier

identifier为Bus对象的名字，debug用

## HandlerFinder

HandlerFinder用于在注册/反注册的时候查找Subscriber和Produce，后文会对其展开源码级别的解析。默认使用HandlerANNOTATED，表示使用注解来进行查找。

## 其他

除上述以外，Bus类还有两个成员变量handlersByType和producersByType:

```Java

/**
** 通过event的类型（class类型）来查找event handle。
*	键为 event类型  值为 事件订阅者集合
*	一个事件类型可以有多个事件订阅者
*/
private final ConcurrentMap<Class<?>, Set<EventHandler>> handlersByType =
	  new ConcurrentHashMap<Class<?>, Set<EventHandler>>();

/**
** 通过event的类型（class类型）来查找event producer。
*	键为 event类型  值为 事件生产者
*	一个事件类型，只能有一个事件生产者
*/
private final ConcurrentMap<Class<?>, EventProducer> producersByType =
	  new ConcurrentHashMap<Class<?>, EventProducer>();
	  
```

# 注册/反注册事件

如下所示要成为订阅者HandlerEvent，只需将其注册到bus，然后使用@Subscribe注解标记回调处理方法即可。回调方法要求可见性为public，有且仅有一个参数，类型为订阅的event。

```Java
class A {

    public A() {
        bus.register(this);
    }

    @Subscribe public void answerAvailable(HandlerEvent event) {
        // process event
    }
}
```
## @Subsrible

首先看一下@Subscribe注解:

```Java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface Subscribe {
}
```
RetentionPolicy.RUNTIME表示它是运行时的注解，ElementType.METHOD表示用于注解方法。

## register()

register流程：

```Java
public void register(Object object) {

	//1.检查当前线程是否符合ThreadEnforcer的设置
	if (object == null) {
	  throw new NullPointerException("Object to register must not be null.");
	}
	enforcer.enforce(this);

	//2.默认情况下，通过@Producer注解找到所有的事件生产者Producers
	Map<Class<?>, EventProducer> foundProducers = handlerFinder.findAllProducers(object);
	for (Class<?> type : foundProducers.keySet()) {

	  //2-1 判断object上的produce注册的event是否已经被别人注册过
	  final EventProducer producer = foundProducers.get(type);

	  //type存在则返回type对应的值 type不存在则将type的键值设为producer
	  EventProducer previousProducer = producersByType.putIfAbsent(type, producer);
	  //checking if the previous producer existed
	  if (previousProducer != null) {
		throw new IllegalArgumentException("Producer method for type " + type
		  + " found on type " + producer.target.getClass()
		  + ", but already registered by type " + previousProducer.target.getClass() + ".");
	  }


	  //2-2 如果没有被注册过，那么找出对应event的handler，触发一次回调
	  Set<EventHandler> handlers = handlersByType.get(type);
	  if (handlers != null && !handlers.isEmpty()) {
		for (EventHandler handler : handlers) {
		  dispatchProducerResultToHandler(handler, producer);
		}
	  }
	}

	//3. 找出object上用@Subscribe注解的方法
	Map<Class<?>, Set<EventHandler>> foundHandlersMap = handlerFinder.findAllSubscribers(object);
	for (Class<?> type : foundHandlersMap.keySet()) {
	  Set<EventHandler> handlers = handlersByType.get(type);
	  
	  
	  if (handlers == null) {
		
		//3-1，该event是第一次注册，那么新建一个CopyOnWriteArraySet用来保存handler和event的对应关系
		
		Set<EventHandler> handlersCreation = new CopyOnWriteArraySet<EventHandler>();
		handlers = handlersByType.putIfAbsent(type, handlersCreation);
		if (handlers == null) {
			handlers = handlersCreation;
		}
	  }
	  
	  //3-2,保存object中新增的event-handler对应关系
	  final Set<EventHandler> foundHandlers = foundHandlersMap.get(type);
	  if (!handlers.addAll(foundHandlers)) {
		throw new IllegalArgumentException("Object already registered.");
	  }
	}

	//4.检查object上的event是否存在对应的Producer，有则触发一次调用
	for (Map.Entry<Class<?>, Set<EventHandler>> entry : foundHandlersMap.entrySet()) {
	  Class<?> type = entry.getKey();
	  EventProducer producer = producersByType.get(type);
	  if (producer != null && producer.isValid()) {
		Set<EventHandler> foundHandlers = entry.getValue();
		for (EventHandler foundHandler : foundHandlers) {
		  if (!producer.isValid()) {
			break;
		  }
		  if (foundHandler.isValid()) {
			dispatchProducerResultToHandler(foundHandler, producer);
		  }
		}
	  }
	}
}
```
register方法主要做了三件事情：触发新的Producer；注册新的event-handler关系；触发旧的Producer。另外有两点要注意：

-	在保证线程安全的情况下，使用CopyOnWriteArraySet作为保存event和handler的容器，可以大大提高效率。
-	由于register方法没有加锁，所有在3-1中，尽管已经检查了handlers是否存在，但仍需使用putIfAbsent来保存handler。

## HandlerFinder

注意到Bus通过HandlerFinder来查找object上的producer和subscriber，接下来看一下HanderFinder的实现：

```Java
interface HandlerFinder {

  Map<Class<?>, EventProducer> findAllProducers(Object listener);

  Map<Class<?>, Set<EventHandler>> findAllSubscribers(Object listener);


  HandlerFinder ANNOTATED = new HandlerFinder() {
    @Override
    public Map<Class<?>, EventProducer> findAllProducers(Object listener) {
      return AnnotatedHandlerFinder.findAllProducers(listener);
    }

    @Override
    public Map<Class<?>, Set<EventHandler>> findAllSubscribers(Object listener) {
      return AnnotatedHandlerFinder.findAllSubscribers(listener);
    }
  };
}
```
其中findAllProducers方法返回某event type对应的EventProducers，findAllSubscribers返回某event type对应的EventHandler集合。

## EventProducer

EventProducer是producer方法的包装类，源码如下：

```Java
class EventProducer {
    final Object target;

    private final Method method;

    private final int hashCode;

    private boolean valid = true;

    EventProducer(Object target, Method method) {
        if (target == null) {
            throw new NullPointerException(
                "EventProducer target cannot be null.");
        }

        if (method == null) {
            throw new NullPointerException(
                "EventProducer method cannot be null.");
        }

        this.target = target;
        this.method = method;
        method.setAccessible(true);

        // 提前计算hashcode，以防每次调用hash()时消耗资源
        final int prime = 31;
        hashCode = ((prime + method.hashCode()) * prime) + target.hashCode();
    }

    public boolean isValid() {
        return valid;
    }
    
    // 应在object unregister时调用
    public void invalidate() {
        valid = false;
    }

    public Object produceEvent() throws InvocationTargetException {
        if (!valid) {
            throw new IllegalStateException(toString() +
                " has been invalidated and can no longer produce events.");
        }

        try {
            return method.invoke(target);
        } catch (IllegalAccessException e) {
            throw new AssertionError(e);
        } catch (InvocationTargetException e) {
            if (e.getCause() instanceof Error) {
                throw (Error) e.getCause();
            }

            throw e;
        }
    }
}
```

其中 produceEvent方法用于获得event。可以看出Otto要求produce方法不能有参数。

## EventHandler

EventHandler是一个event handler方法（事件回调）的包装类，源码如下：

```Java
class EventHandler {

    private final Object target;

    private final Method method;

    private final int hashCode;

    private boolean valid = true;

    EventHandler(Object target, Method method) {
        if (target == null) {
            throw new NullPointerException(
                "EventHandler target cannot be null.");
        }

        if (method == null) {
            throw new NullPointerException(
                "EventHandler method cannot be null.");
        }

        this.target = target;
        this.method = method;
        method.setAccessible(true);

        // Compute hash code eagerly since we know it will be used frequently and we cannot estimate the runtime of the
        // target's hashCode call.
        final int prime = 31;
        hashCode = ((prime + method.hashCode()) * prime) + target.hashCode();
    }

    public boolean isValid() {
        return valid;
    }

    public void invalidate() {
        valid = false;
    }

    public void handleEvent(Object event) throws InvocationTargetException {
        if (!valid) {
            throw new IllegalStateException(toString() +
                " has been invalidated and can no longer handle events.");
        }

        try {
            method.invoke(target, event);
        } catch (IllegalAccessException e) {
            throw new AssertionError(e);
        } catch (InvocationTargetException e) {
            if (e.getCause() instanceof Error) {
                throw (Error) e.getCause();
            }

            throw e;
        }
    }
}
```

其中handlEvent方法用于在object上调用handle方法（事件回调），传入event对象。Otto要求event handler方法只能有一个参数就是event handler类。

## dispatchProducerResultToHandler()

dispatchProducerResultToHandler方法用于将Producer产生的event分发给对应的handler，源码如下所示：

```Java
private void dispatchProducerResultToHandler(EventHandler handler, EventProducer producer) {
    Object event = null;
    try {
        event = producer.produceEvent();
    } catch(InvocationTargetException e) {
        throwRuntimeException("Producer " + producer + " threw an exception.", e);
    }
    if (event == null) {
        return;
    }
    dispatch(event, handler);
}

protected void dispatch(Object event, EventHandler wrapper) {
    try {
        wrapper.handleEvent(event);
    } catch(InvocationTargetException e) {
        throwRuntimeException("Could not dispatch event: " + event.getClass() + " to handler " + wrapper, e);
    }
}
```

主要使用了Producer的produceEvent()获取event对象后，调用EventHandler的handleEvent（）方法处理事件。

## unregister()

Bus类的unregister()方法用于解除目标对象和Bus之间的关联关系，包括对象上的producer方法，subscriber方法，源码如下所示：

```Java
public void unregister(Object object) {
    if (object == null) {
        throw new NullPointerException("Object to unregister must not be null.");
    }
    //1. 检查当前线程是否符合ThreadEnforcer的设置
    enforcer.enforce(this);

    //2. 默认情况下，通过注解在object上找出所有Producer，将其从producersByType中删除并标记为invalidate
    Map<Class<?>, EventProducer> producersInListener = handlerFinder.findAllProducers(object);
    for (Map.Entry<Class<?>, EventProducer> entry : producersInListener.entrySet()) {
        final Class<?> key = entry.getKey();
        EventProducer producer = getProducerForEventType(key);
        EventProducer value = entry.getValue();
        
        if (value == null || !value.equals(producer)) {
            throw new IllegalArgumentException(
            "Missing event producer for an annotated method. Is " + object.getClass() + " registered?");
        }
        producersByType.remove(key).invalidate();
    }
    
    //3. 默认情况下，找出object上用@Subscribe注解了的handler，将其从event集合中删除并标记为invalidate
    Map<Class<?>, Set<EventHandler>> handlersInListener = handlerFinder.findAllSubscribers(object);
    for (Map.Entry<Class<?>, Set<EventHandler>> entry : handlersInListener.entrySet()) {
        Set<EventHandler> currentHandlers = getHandlersForEventType(entry.getKey());
        Collection<EventHandler> eventMethodsInListener = entry.getValue();
        
        if (currentHandlers == null || !currentHandlers.containsAll(eventMethodsInListener)) {
            throw new IllegalArgumentException(
            "Missing event handler for an annotated method. Is " + object.getClass() + " registered?");
        }
        
        for (EventHandler handler : currentHandlers) {
            if (eventMethodsInListener.contains(handler)) {
                handler.invalidate();
            }
        }
        currentHandlers.removeAll(eventMethodsInListener);
    }
}
```

# 投递事件

## post()

简单的事件投递过程如下：

```Java
bus.post(new HandlerEvent(42));

或者

bus.post(getEvent);

@Producer
public HandlerEvent getEvent() {
	return new HandlerEvent(42);
}

```
下面来看下post方法实现的源码：

```Java
ublic void post(Object event) {
    if (event == null) {
        throw new NullPointerException("Event to post must not be null.");
    }
    //1. 检查当前线程是否符合ThreadEnforcer的设置
    enforcer.enforce(this);
    
    //2. 向上追溯event的所有父类
    Set<Class<?>>dispatchTypes = flattenHierarchy(event.getClass());
    
    //3. 当前event没有注册handler，则发送一个DeadEvent事件
    boolean dispatched = false;
    for (Class<?>eventType: dispatchTypes) {
        Set<EventHandler> wrappers = getHandlersForEventType(eventType);

        if (wrappers != null && !wrappers.isEmpty()) {
            dispatched = true;
            for (EventHandler wrapper: wrappers) {
                //3-1 将事件和handler放到分发队列里
                enqueueEvent(event, wrapper);
            }
        }
    }
    
    //4. 当前event没有注册handler，则发送一个DeadEvent事件
    if (!dispatched && !(event instanceof DeadEvent)) {
        post(new DeadEvent(this, event));
    }

    //5. 通知队列进行分发操作
    dispatchQueuedEvents();
}
```

注意两点：

-	发送一个Event时，订阅了Event父类的Subscriber方法也会被调用
-	事件被放到调用者所在线程的队列里依次分发

## flattenHierarchy()

进行post操作时，首先会通过flattenHierarchy方法获得event的所有父类或接口的集合：

```Java
  Set<Class<?>> flattenHierarchy(Class<?> concreteClass) {
    Set<Class<?>> classes = flattenHierarchyCache.get(concreteClass);
    if (classes == null) {
      Set<Class<?>> classesCreation = getClassesFor(concreteClass);
      classes = flattenHierarchyCache.putIfAbsent(concreteClass, classesCreation);
      if (classes == null) {
        classes = classesCreation;
      }
    }

    return classes;
  }
  
  //利用深度优先遍历导出了concreteClass的所有父类
  private Set<Class<?>> getClassesFor(Class<?> concreteClass) {
    List<Class<?>> parents = new LinkedList<Class<?>>();
    Set<Class<?>> classes = new HashSet<Class<?>>();

    parents.add(concreteClass);

	//深度优先遍历
    while (!parents.isEmpty()) {
      Class<?> clazz = parents.remove(0);
      classes.add(clazz);

      Class<?> parent = clazz.getSuperclass();
      if (parent != null) {
        parents.add(parent);
      }
    }
    return classes;
  }
```
## Dispatch Queue

通过post方法投递的event首先会放到当前线程所在的Dispatch Queue中，然后依次分发。Bus类有如下成员属性：

```Java
private final ThreadLocal<ConcurrentLinkedQueue<EventWithHandler>> eventsToDispatch =
    new ThreadLocal<ConcurrentLinkedQueue<EventWithHandler>>() {
        @Override protected ConcurrentLinkedQueue<EventWithHandler> initialValue() {
            return new ConcurrentLinkedQueue<EventWithHandler>();
        }
    };

```
eventsToDispatch是一个ThreadLocal对象，通过initialValue()方法，eventsToDispatch每次在新的线程上调用的时候都会生成新的ConcurrentLinkedQueue实例。event是通过enqueueEvent方法放到queue中的，下面看看equeueEvent()的实现：

```Java
protected void enqueueEvent(Object event, EventHandler handler) {
	eventsToDispatch.get().offer(new EventWithHandler(event, handler));
}
```
offer()方法会将EventWithHandler对象放到当前线程的queue的尾部。offer方法和add方法的区别在于，当无法插入（例如空间不够）情况下会返回false，而不是抛出异常。EventWithHandler类对event和handler的关系进行了简单的包装，实现如下：

```Java
static class EventWithHandler {
	final Object event;
	final EventHandler handler;

	public EventWithHandler(Object event, EventHandler handler) {
	  this.event = event;
	  this.handler = handler;
	}
}
```
接下来看看dispatchQueuedEvents方法的实现：

```Java
protected void dispatchQueuedEvents() {
    // don't dispatch if we're already dispatching, that would allow reentrancy and out-of-order events. Instead, leave
    // the events to be dispatched after the in-progress dispatch is complete.
    //1. 不能重复分发，否则会导致event的分发次序混乱
    if (isDispatching.get()) {
        return;
    }

    isDispatching.set(true);
    try {
        while (true) {
            //2. 依次取出EventWithHandler，并通过dispatch方法进行分发。
            EventWithHandler eventWithHandler = eventsToDispatch.get().poll();
            if (eventWithHandler == null) {
                break;
            }

            if (eventWithHandler.handler.isValid()) {
                dispatch(eventWithHandler.event, eventWithHandler.handler);
            }
        }
    } finally {
        isDispatching.set(false);
    }
}

  protected void dispatch(Object event, EventHandler wrapper) {
    try {
      wrapper.handleEvent(event);
    } catch (InvocationTargetException e) {
      throwRuntimeException(
          "Could not dispatch event: " + event.getClass() + " to handler " + wrapper, e);
    }
  }

```
值得注意的是，所有subscrible方法抛出的异常都会在这里捕获，捕获到异常以后event分发过程即停止，直到下一次在该线程上调用post为止。

# 结构图

Otto的总体结构如下表示

```Java
            +-------------------------+
            |Bus(ThreadLocal)         |
            |     +--------------+    |
            |     |EventProducers|    |
            |     |  +-------+   |  register  +-------+
            |     |  |Produce|   <----+-------+Produce|
            |     |  +-------+   |    |       +-------+
            |     |  +-------+   |    |
            |     |  |Produce|   |    |
            |     |  +-------+   |    |
            |     +--------------+    |
            |            |            |
            |          event          |
            |            |            |
 post(event)|    +-------v--------+   |
+----------------> Dispatch Queue |   |
            |    +-------+--------+   |
            |            |            |
            |          event          |
            |            |            |
            |     +------v------+     |
            |     |EventHandlers|     |
            |     | +---------+ |     |
            |     | |Subscribe| |   register  +---------+
            |     | +---------+ <-----+-------+Subscribe|
            |     | +---------+ |     |       +---------+
            |     | |Subscribe| |     |
            |     | +---------+ |     |
            |     +-------------+     |
            |                         |
            +-------------------------+
```


