---
title: Threadlocal
category:
- JDK Source Code
- lang
date: 2017/03/20
copyright: true
---

# 1. Threadlocal源码
## 1.1 几个基本点
- Thread、Threadlocal、ThreadlocalMap、ThreadlocalMap.Entry的关系就好比老板（Thread）、助理（Threadlocal）、老板的公文包（ThreadlocalMap）、公文包里的文件夹（ThreadlocalMap.Entry）。
- 公文包（ThreadlocalMap）是老板（Thread）的，要么没有，要么只有一个。
  ```java
  //老板
  public class Thread implements Runnable {
     //公文包是老板的，而且如果有，就只有一个，默认没有
     ThreadLocal.ThreadLocalMap threadLocals = null;
  }
  ```

- 老板（Thread）可以有多个助理（Threadlocal），在老板的召唤下，助理们都可以往公文包（ThreadlocalMap）或存或取或移除自己的文件（ThreadlocalMap.Entry.value）。为了便于管理，助理们要把文件放进一个用自己身份标识了的文件夹（ThreadlocalMap.Entry），然后再把文件夹放进公文包。助理只能拿到自己的文件夹，也便只能管理自己文件夹里的文件。当然，助理在或存或取或移除文件之前，都要先判断老板有没有公文包，公文包里有没有用记身份标识了的文件夹。如果没有就要创建新的。
  ```java
  //助理
  public class ThreadLocal<T> {
    void createMap(Thread t, T firstValue) {//给老板创建新公文包
        t.threadLocals = new ThreadLocalMap(this, firstValue);
    }
    //公文包  
    static class ThreadLocalMap {
      //文件夹
      static class Entry extends WeakReference<ThreadLocal<?>> {
          //文件
          Object value;
          Entry(ThreadLocal<?> k, Object v) {//创建用助理身份标识了的文件夹，并把文件放进去
              super(k);
              value = v;
          }
      }
      ThreadLocalMap(ThreadLocal<?> firstKey, Object firstValue) {
          table = new Entry[INITIAL_CAPACITY];
          int i = firstKey.threadLocalHashCode & (INITIAL_CAPACITY - 1);
          table[i] = new Entry(firstKey, firstValue);//给助理自己创建新文件夹
          size = 1;
          setThreshold(INITIAL_CAPACITY);
      }
    }
  }
  ```
<!--more-->
- 老板（Thread）要操作某个文件（ThreadlocalMap.Entry.value）时就把管理这个文件的助理（Threadlocal）召唤到身边，由助理代为操作。助理操作自己放在当前老板公文包里的文件，要做的就是：在老板那拿到公文包（ThreadlocalMap），在公文包里拿到自己的文件夹（ThreadlocalMap.Entry），在文件夹里拿到文件。如果老板想和助理解除关系，就让这个助理把他的文件夹取出来连带文件一起移除。当有助理要存取文件或新建文件夹时，老板也会委托这个助理查看所有文件夹，如果发现有文件夹指向的助理已经die，也要把文件夹及文件从公文包里移除。
```java
public class ThreadLocal<T> {
   ThreadLocalMap getMap(Thread t) {//找老板要公文包
     return t.threadLocals;
   }

   public T get() {
       Thread t = Thread.currentThread();//当前老板
       ThreadLocalMap map = getMap(t);//在老板这要公文包
       if (map != null) {
           ThreadLocalMap.Entry e = map.getEntry(this);//找到用自己身份标识了的文件夹
           if (e != null) {
               @SuppressWarnings("unchecked")
               T result = (T)e.value;//拿到文件
               return result;
           }
       }
       return setInitialValue();
   }

   public void set(T value) {
       Thread t = Thread.currentThread();//找到当前的老板
       ThreadLocalMap map = getMap(t);//找老板要公文包
       if (map != null)
           map.set(this, value);//把文件放入用自己身份标识了的文件夹
       else
           createMap(t, value);//给老板买个公文包并把件放入用自己身份标识了的文件夹
   }

   public void remove() {
       ThreadLocalMap m = getMap(Thread.currentThread());//先找到当前的老板再找老板要公文包
       if (m != null)
           m.remove(this);//从公文包中把自己的文件夹取出
   }
}
```

- 助理（Threadlocal）对自己管理的文件的存取只能是替换存与整体取。每次存的时候都是整体替换（如果文件夹中已经有文件），取的时候都是先拿到文件夹，再从文件夹中取出全部文件（拿到存储对象的引用）。
  ```java
  static class ThreadLocalMap {
    //存文件
    private void set(ThreadLocal<?> key, Object value) {
        Entry[] tab = table;
        int len = tab.length;
        int i = key.threadLocalHashCode & (len-1);

        for (Entry e = tab[i];
             e != null;
             e = tab[i = nextIndex(i, len)]) {
            ThreadLocal<?> k = e.get();

            if (k == key) {//已经有用自己身份标识的文件夹
                e.value = value;//将手头的新文件替换旧文件
                return;
            }

            if (k == null) {//判断其他文件夹指向的助理是否存活
                replaceStaleEntry(key, value, i);
                return;
            }
        }

        //如果没有找到用自己身份标识的文件夹，创建一个，并把文件放进去。
        tab[i] = new Entry(key, value);
        int sz = ++size;
        if (!cleanSomeSlots(i, sz) && sz >= threshold)
            rehash();
    }
    //拿出文件夹
    private Entry getEntry(ThreadLocal<?> key) {
        int i = key.threadLocalHashCode & (table.length - 1);
        Entry e = table[i];
        if (e != null && e.get() == key)
            return e;
        else
            return getEntryAfterMiss(key, i, e);
    }
  }
  ```

- 助理（Threadlocal）也可以为多个老板（Thread）管理文件，只要有老板召唤（线程里有调用Threadlocal对象）。当然，这个时候助理操作的是放在当前这个老板（Thread.currentThread()）的公文包里的当前这个老板的文件，而不可能是其他老板的。

## 1.2 几个问题
### 1.2.1 关于变量副本与线程隔离
- 很多关于Threadlocal的技术博客里都有“变量副本”的说法：
 > 当使用ThreadLocal维护变量时，ThreadLocal为每个使用该变量的线程提供独立的变量副本，所以每一个线程都可以独立地改变自己的副本，而不会影响其它线程所对应的副本。  ---- [彻底理解ThreadLocal](http://blog.csdn.net/lufeng20/article/details/24314381)

 > ThreadLocal为变量在每个线程中都创建了一个副本。    ---- [Java并发编程：深入剖析ThreadLocal](http://www.cnblogs.com/dolphin0520/p/3920407.html)

 “变量副本”这个词应该来自对Threadlocal类的Java Doc的翻译：
 > This class provides thread-local variables.  These variables differ from
 their normal counterparts in that each thread that accesses one (via its
 {@code get} or {@code set} method) has its own, independently initialized
 copy of the variable.

 意译过来大概是这样：<br>这个类提供线程本地变量。这些变量与其他一般变量不同，通过它的get()或set()方法，每个线程都保存一份该变量的副本。
- 要厘清“变量副本”是什么，首先要厘清“变量”是什么。Thread的本地变量是指Threadlocal的set方法的参数value或者是initialValue方法的返回值？按引用的两篇文章的意思是这样的。而事实不是！线程本地变量（thread-local variable）应该就是指Threadlocal实例。
 - 这点在Threadlocal类的comments里有说明：
  > .... each thread that accesses one (via its {@code get} or {@code set} method) has its own，....

   这里的one和its都是指these variables，也就是说变量有get和set方法，对于用于存储的value而言，说Threadlocal有get和set方法才是合理的。
 - 在comments里的Threadlocal使用示例也有体现：
   ```java
   public class ThreadId {
       // Atomic integer containing the next thread ID to be assigned
       private static final AtomicInteger nextId = new AtomicInteger(0);

       // Thread local variable containing each thread's ID
       private static final ThreadLocal<Integer> threadId =
           new ThreadLocal<Integer>() {
               @Override protected Integer initialValue() {
                   return nextId.getAndIncrement();
           }
       };

       // Returns the current thread's unique ID, assigning it if necessary
       public static int get() {
           return threadId.get();
       }
   }
   ```
   示例中，对threadId的注释:Thread local variable containing each thread's ID足以表明threadId（Threadlocal实例）是线程本地变量（thread local variable）。
- 厘清了“变量”是指Threadlocal的实例，“变量副本”就好理解了。正像之前比喻的，助理（Threadlocal）可以为多个老板（Thread）服务，为多少老板服务，就会有多少个用自己身份标识了的保存在各个老板公文包里的文件夹。Threadlocal在一个Thread的ThreadlocalMap里做一次保存，就是一次copy。从源码可以知道，这个copy也只是概念上的copy，ThreadlocalMap的机制达到了不用真正copy每个Threadlocal，就有每个线程都有自己独立的变量副本的copy效果。
- 这个时候，就可以水到渠成地说：变量副本是线程隔离的，每个线程只能看到和操作自己的副本。客户在任何线程里调用同一个threadlocal的set、get等方法都只会返回在当然线程的数据。
- 如果把“变量”理解为Threadlocal的set方法的参数或initialValue方法的返回值，那么变量副本的说法说不通，变量副本是线程隔离的说法同样如此，因为在Threadlocal的源码中，没有对value参数进行拷贝（没有任何处理）。
- set方法的参数和initialValue方法的返回值作为实例对象，如果其引用能被其他线程拿到，就能被访问和操作，不会是线程隔离的。这种情况下value的线程安全取决于其对象本身，如果本身线程安全，那便是线程安全，如果不是线程安全，那也必定不是线程安全。这也是comments示例里nextId要用AtomicInteger类型的原因（如果把nextId声明为Integer，initialValue的返回值为nextId+1，那threadId势必会混乱）。另一方面，如果引用不会被其他线程拿到，只能被当前这一个线程访问，也就不存在线程安全问题。
  ```java
  public class ThreadSafety {

      static SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd HH:mm");

      static ThreadLocal<SimpleDateFormat> threadlocal = new ThreadLocal<SimpleDateFormat>() {
          protected SimpleDateFormat initialValue() {

              //方式1：sdf可以被多个线程访问，不是线程安全的
              //return sdf;

              //方式2：只能被当前这一个线程访问，不存在线程安全问题
              return new SimpleDateFormat("yyyy-MM-dd HH:mm");
          }
      };

      public static void main(String[] args) throws InterruptedException {

          new Thread(() -> {
              threadlocal.get().setTimeZone(TimeZone.getTimeZone("CTT"));//CTT - Asia/Shanghai
              printTimeZone();
              try {Thread.sleep(3000);} catch (Exception e) {}
              printTimeZone();
          }).start();

          Thread.sleep(1000);
          threadlocal.get().setTimeZone(TimeZone.getTimeZone("JST"));//JST - Asia/Tokyo
          printTimeZone();
      }

      private static void printTimeZone() {
          System.out.println(Thread.currentThread().getName() + ": timeZone=" + threadlocal.get().getTimeZone());

      }

  }
  ```
  方式1的执行结果：
  ```java
  Thread-0: timeZone=sun.util.calendar.ZoneInfo[id="CTT",offset=28800000,dstSavings=0,useDaylight=false,transitions=19,lastRule=null]
  main: timeZone=sun.util.calendar.ZoneInfo[id="JST",offset=32400000,dstSavings=0,useDaylight=false,transitions=10,lastRule=null]
  Thread-0: timeZone=sun.util.calendar.ZoneInfo[id="JST",offset=32400000,dstSavings=0,useDaylight=false,transitions=10,lastRule=null]
  ```
 方式2的执行结果
  ```java
  Thread-0: timeZone=sun.util.calendar.ZoneInfo[id="CTT",offset=28800000,dstSavings=0,useDaylight=false,transitions=19,lastRule=null]
  main: timeZone=sun.util.calendar.ZoneInfo[id="JST",offset=32400000,dstSavings=0,useDaylight=false,transitions=10,lastRule=null]
  Thread-0: timeZone=sun.util.calendar.ZoneInfo[id="CTT",offset=28800000,dstSavings=0,useDaylight=false,transitions=19,lastRule=null]
  ```

### 1.2.2 关于弱引用与内存泄漏
- 关于弱引用与内存泄漏的相关言论
 > ThreadLocalMap使用ThreadLocal的弱引用作为key，如果一个ThreadLocal没有外部强引用引用他，那么系统gc的时候，这个ThreadLocal势必会被回收，这样一来，ThreadLocalMap中就会出现key为null的Entry，就没有办法访问这些key为null的Entry的value，如果当前线程再迟迟不结束的话，这些key为null的Entry的value就会一直存在一条强引用链：Thread Ref -> Thread -> ThreaLocalMap -> Entry -> value永远无法回收，造成内存泄露。          
 ---- [解密ThreadLocal](http://qifuguang.me/2015/09/02/[Java%E5%B9%B6%E5%8F%91%E5%8C%85%E5%AD%A6%E4%B9%A0%E4%B8%83]%E8%A7%A3%E5%AF%86ThreadLocal/)

 > threadlocal里面使用了一个存在弱引用的map,当释放掉threadlocal的强引用以后,map里面的value却没有被回收.而这块value永远不会被访问到了. 所以存在着内存泄露. 最好的做法是将调用threadlocal的remove方法.threadlocal的生命周期中,都存在这些引用. 看下图: 实线代表强引用,虚线代表弱引用.
 ![Threadlocal弱引用关系](/images/threadlocal/reference.jpg)     ---- [ThreadLocal可能引起的内存泄露](http://www.cnblogs.com/onlywujun/p/3524675.html)


- 什么是内存泄漏
 > 内存泄漏也称作“存储渗漏”，用动态存储分配函数动态开辟的空间，在使用完毕后未释放，结果导致一直占据该内存单元。直到程序结束。（其实说白了就是该内存空间使用完毕之后未回收）即所谓内存泄漏。
 >  ---- [百度百科](http://baike.baidu.com/link?url=nEyGZ2NIAm86HqsZwM0tj6HXCXyB_tDcp6gDLY1zCMxhYyV6D9Em4oyQXd-g7Z3YQlSMWy5bMG9LV64Vq8y_gvzTUnOKyi6ZfqYTetrC_pxTB78_OZFyuKa69m6LfnB3)


- 造成内存泄漏的前提：没有对ThreadlocalMap的Entry对象进行有效管理，这个线程还迟迟不结束（销毁）。<br>只要对Entry对象进行了有效的防内存泄漏管理，就不会出现内存泄漏；只要线程对象快速结束，就不会出现内存泄露。迟迟不结束的线程，最常见的是线程池里的线程。
- 造成内存泄漏的直接原因是ThreadlocalMap的key使用了Threadlocal的弱引用？答案是否定的！<br>造成内存泄漏与key是强引用还是弱引用没有直接关系。即使Key被设计成Threadlocal的强引用，也避免不了引起内存泄漏的可能性。当Threadlocal Ref由Threadlocal1指向Threadlocal2之后，Entry的key和value都有通过Thread Refr的可达性，如果Thread一直存活，那么key和value就一直不会被回收，造成内存泄漏。![Threadlocal强引用关系](/images/threadlocal/reference-strong.jpg)
- 反而，使用ThreadLocal的弱引用作为key，是匠心之作。<br>再考虑上图中的情况，Threadlocal Ref由Threadlocal1指向Threadlocal2之后，事情已经变得不可控制了。用户已经拿不到Threadlocal1的引用，也便不能通过调用remove方法把自己从ThreadlocalMap中移除。而且，ThreadlocalMap想帮用户移除用Threadlocal1做key的Entry也不可能，因为遍历出来的Entry都没有特殊标记，不知道哪些有用，哪些没用。这个时候，Threadlocal1就必定是个内存泄漏，要想回收这部分内存，只能等线程结束，如果线程刚好是线程池里的线程，那就等OOM吧。<br>而把key设计成Threadlocal的弱引用后，在JVM的配合下，通过用户调用Threadlocal的get()或set()方法，或者往ThreadlocalMap存入新的变量副本时，ThreadlocalMap会探测到哪些Entry有用，哪些Entry没用，同时把没用的回收掉。<br>所以，用Threadlocal的弱引用来作为key是来帮助避免造成内存泄漏，而非促成内存泄漏。
- 当然，使用ThreadLocal的弱引用作为key的这个设计在很大程度避免了内存泄漏，但不能完全避免内存泄漏。<br>实例1:[ThreadLocal & Memory Leak
](http://stackoverflow.com/questions/17968803/threadlocal-memory-leak)<br>实例2:[A ThreadLocal Memory Leak](https://blog.codecentric.de/en/2008/09/a-threadlocal-memory-leak/)
- 如果你new了一个线程，能确定它会快速结束，那就放心用Threadlocal。皮之不存，毛将焉附。如果你创建了一个线程池，请谨慎使用Threadlocal。如果只是在一个别人给定的线程里工作，在了解线程的所有细节前，请谨慎使用Threadlocal。

# 2. Threadlocal应用
## 2.1 应用场景
 Threadlocal类Comments中关系应用场景的描述
 > ThreadLocal instances are typically private static fields in classes that wish to associate state with a thread (e.g.,a user ID or Transaction ID).<br>
如果开发者希望将某个状态（user ID或者transaction ID）与线程关联，则可以考虑使用ThreadLocal。ThreadLocal通常被声明为private static。


### 2.1.1 用Threadlocal在特定线程中共享对象。
比如，在web请求中，当前用户的userid一般只能在Controller中获取到，而userid在Service层乃至Model层都可能会被用到。<br>这时可以考虑用Threadlocal把userid绑定在Thread里，用于避免userid被频繁传递，达到共享userid的效果。当然，最后要调用remove方法，防止内存泄漏。参看[JFinal对Threadlocal的应用](#usage1)。
### 2.1.2 线程池里，用Threadlocal让对象得到复用。
比如，线程池里，如果线程每次被调用都会用到SimpleDateFormat的format方法，不管是每次都new一个新的SimpleDateFormat还是同步对一个全局SimpleDateFormat实例的访问都不是很好的方式。<br>这时可以考虑用Threadlocal为每个Thread绑定它自己的SimpleDateFormat，用于复用。参看[JFinal对Threadlocal的应用](#usage2)。当然，这时候也要考虑内存泄漏问题。如果是大对象就更要进行各种权衡，试想一下把10M的对象绑定到线程池的100个线程里。

## 2.2 JFinal对Threadlocal的应用
Threadlocald在JFinal3.0中有3次应用：com.jfinal.core.ActionReporter类，com.jfinal.plugin.activerecord.Config类，com.jfinal.plugin.redis.Cache类。
### 2.2.1 ActionReporter类
<span id="usage2">ActionReporter类对Threadlocal的应用</span>，就是为了让SimpleDateFormat在线程池的线程里得到复用。
  ```java
  public class ActionReporter {
    private static final ThreadLocal<SimpleDateFormat> sdf = new ThreadLocal<SimpleDateFormat>() {
      protected SimpleDateFormat initialValue() {
        return new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
      }
    };
    public static final void report(String target, Controller controller, Action action) {
  		StringBuilder sb = new StringBuilder("\nJFinal action report -------- ").append(sdf.get().format(new Date())).append(" ------------------------------\n");
      //省略其它代码
  	}	 
     //省略其它代码
  }
  ```
ActionReporter类用于开发模式下记录并输出请求信息。<br>开发模式下，每个web请求都会调用一次report()方法，恰好当前线程又是容器的线程池中的线程，便可以把SimpleDateFormat绑定到线程里复用。<br>虽然每次请求都只会调用一次sdf.get().format(new Date())，但是达到了复用Thread时复用SimpleDateFormat的效果，而不必为每次请求都new一个新的SimpleDateFormat。<br>如果一个web请求会多次调用sdf.get()方法，也一定会起到在这个线程里共享sdf的效果。<br>另外，这里没有显式地调用sdf.remove()方法，因为SimpleDateFormat对象每个请求都必用，在开发模式下没有例外，SimpleDateFormat对象可以（对象不大）也应该（每次都要用）随着Thread一起存活，直到当前应用terminate。<br>如果每次请求最后都调用sdf.remove()方法，那么每次调用sdf.get()方法都会转而调用initialValue()方法，最终每次都会new一个SimpleDateFormat对象，这样的话，和每次直接为线程new一个SimpleDateFormat对象就没有任何区别。
### 2.2.2 Config类和Cache类
<span id="usage2">Config类和Cache类都是为在特定线程中共享对象而设计的。</span>这里以Cache的threadLocalJedis为例。
  ```java
  public class Cache {
  	protected final ThreadLocal<Jedis> threadLocalJedis = new ThreadLocal<Jedis>();

  	public Jedis getThreadLocalJedis() {
  		return threadLocalJedis.get();
  	}

  	public void setThreadLocalJedis(Jedis jedis) {
  		threadLocalJedis.set(jedis);
  	}

  	public void removeThreadLocalJedis() {
  		threadLocalJedis.remove();
  	}
    //省略其它代码
  }
  ```
  ```java
  /**
   * RedisInterceptor 用于在同一线程中共享同一个 jedis 对象，提升性能.
   * 目前只支持主缓存 mainCache，若想更多支持，参考此拦截器创建新的拦截器
   * 改一下Redis.use() 为 Redis.use(otherCache) 即可
   */
  public class RedisInterceptor implements Interceptor {

  	/**
  	 * 通过继承 RedisInterceptor 类并覆盖此方法，可以指定
  	 * 当前线程所使用的 cache
  	 */
  	protected Cache getCache() {
  		return Redis.use();
  	}

  	public void intercept(Invocation inv) {
  		Cache cache = getCache();
  		Jedis jedis = cache.getThreadLocalJedis();
  		if (jedis != null) {
  			inv.invoke();
  			return ;
  		}

  		try {
  			jedis = cache.jedisPool.getResource();
  			cache.setThreadLocalJedis(jedis);
  			inv.invoke();
  		}
  		finally {
  			cache.removeThreadLocalJedis();
  			jedis.close();
  		}
  	}
  }
  ```
  调用threadLocalJedis的线程也是容器的线程池中的线程，但threadLocalJedis并不是每个线程每次都要用到，所以在每次调用的最后都调用cache.removeThreadLocalJedis()来防止内存泄漏。
# 3. Threadlocal设计
## 3.1 我来实现线程本地存储
### 3.1.1说明
实现是只考虑功能不考虑性能的实现。
### 3.1.2 需求
- 线程可以有多个本地变量，一个本地变量对应一个值（对象）
- 线程本地变量是线程隔离的，线程只是看见和操作自己的本地变量。

### 3.1.3 实现
#### Version 1.0
- MyThread类
 ```java
public class MyThread extends Thread {

    /**
     * 1. 线程可以有多个本地变量，一个本地变量对应一个值（对象），第一反应便是Map
     * 2. 并不是每个线程都一定会有threadlocalMap，没有必要在声明时就初始化。
     * 3. Map的key指定为String类型，value指定为Object类型。key即为本地变量，value即为本地变量对应的值
     */
    private Map<String, Object> threadlocalMap;

    /**
     * 1. 确保获取到的threadlocalMap是当前线程的threadlocalMap
     * 2. 第一次获取threadlocalMap时对其初始化
     *
     * @return
     */
    public Map<String, Object> getThreadlocalMap() {
        //隔离线程，确保获取到的threadlocalMap是当前线程的threadlocalMap
        MyThread curThread = (MyThread) currentThread();
        Map<String, Object> curThreadlocalMap = curThread.threadlocalMap;

        /*
         * 不论在哪个线程调用getThreadlocalMap()方法，都会通过上面的代码获取到当前正在执行的线程，
         * 而线程的线性执行，使得这个方法不存在并发，不需要同步
         *
        if(curThreadlocalMap == null){
            synchronized (curThread){
                if(curThreadlocalMap == null){
                    curThreadlocalMap = new HashMap<>();
                }
            }
        }*/
        if (curThreadlocalMap == null) {
            curThreadlocalMap = new HashMap<>();
        }

        return curThreadlocalMap;
    }

    public MyThread(Runnable runnable) {
        super(runnable);
    }
}
 ```

- 调用
 ```java
 public class Main {
    private static final AtomicInteger nextId = new AtomicInteger(0);
    private static final String threadId= UUID.randomUUID().toString();

    public static void main(String[] args) throws InterruptedException {
        new MyThread(()->{
            MyThread t =(MyThread)Thread.currentThread();
            putAndGetId(t,"group1");
            for (int i = 0; i < 10; i++) {
                new MyThread(()->{
                    MyThread t1=(MyThread)Thread.currentThread();
                    putAndGetId(t1,"group1");
                }).start();
            }
        }).start();

        Thread.sleep(2000);
        new MyThread(()->{
            MyThread t =(MyThread)Thread.currentThread();
            putAndGetId(t,"group2");

            for (int i = 0; i < 10; i++) {
                new MyThread(()->{
                    t.getThreadlocalMap();
                    putAndGetId(t,"group2");
                }).start();
            }
        }).start();

    }

    private static void putAndGetId(MyThread t,String threadGroup) {
        Map<String, Object> threadlocalMap = t.getThreadlocalMap();
        threadlocalMap.put(threadId, nextId.getAndIncrement());
        int id = (int)threadlocalMap.get(threadId);
        System.out.println("threadGroup:"+threadGroup+" threadlocalMap:"+threadlocalMap.hashCode()+" id:"+id+" threadName:"+t.getName());
    }
}
 ```

- 结果
group1的结果还正常，而group2的结果就混乱了。

 ```
 Connected to the target VM, address: '127.0.0.1:63895', transport: 'socket'
threadGroup:group1 threadlocalMap:1964588436 id:0 threadName:Thread-0
threadGroup:group1 threadlocalMap:1964588437 id:1 threadName:Thread-1
threadGroup:group1 threadlocalMap:1964588438 id:2 threadName:Thread-2
threadGroup:group1 threadlocalMap:1964588439 id:3 threadName:Thread-3
threadGroup:group1 threadlocalMap:1964588432 id:4 threadName:Thread-4
threadGroup:group1 threadlocalMap:1964588433 id:5 threadName:Thread-5
threadGroup:group1 threadlocalMap:1964588434 id:6 threadName:Thread-6
threadGroup:group1 threadlocalMap:1964588435 id:7 threadName:Thread-7
threadGroup:group1 threadlocalMap:1964588444 id:8 threadName:Thread-8
threadGroup:group1 threadlocalMap:1964588445 id:9 threadName:Thread-9
threadGroup:group1 threadlocalMap:1964588446 id:10 threadName:Thread-10
threadGroup:group2 threadlocalMap:1964588447 id:11 threadName:Thread-11
threadGroup:group2 threadlocalMap:1964588440 id:12 threadName:Thread-11
threadGroup:group2 threadlocalMap:1964588441 id:13 threadName:Thread-11
threadGroup:group2 threadlocalMap:1964588442 id:14 threadName:Thread-11
threadGroup:group2 threadlocalMap:1964588443 id:15 threadName:Thread-11
threadGroup:group2 threadlocalMap:1964588420 id:16 threadName:Thread-11
threadGroup:group2 threadlocalMap:1964588421 id:17 threadName:Thread-11
threadGroup:group2 threadlocalMap:1964588422 id:18 threadName:Thread-11
threadGroup:group2 threadlocalMap:1964588423 id:19 threadName:Thread-11
threadGroup:group2 threadlocalMap:1964588416 id:20 threadName:Thread-11
threadGroup:group2 threadlocalMap:1964588417 id:21 threadName:Thread-11
Disconnected from the target VM, address: '127.0.0.1:63895', transport: 'socket'
Process finished with exit code 0
 ```

- 结论
  - getThreadlocalMap()能实现线程本地变量隔离的关键在于有currentThread()方法。currentThread()方法，让任何其他Thread对象调用getThreadlocalMap()获取到的仍是当前线程的threadlocalMap。
  - 这个版本中，获取当前线程的threadlocalMap正确方式是用通过(MyThread)Thread.currentThread().getThreadlocalMap()，而不是通过otherThread.getThreadlocalMap()，虽然otherThread.getThreadlocalMap()获取到的还是当前线程的threadlocalMap，但是语义不清晰，客户会以为api获取到的是otherThread的threadlocalMap，不符合开发直觉。
  - 如果能阻止客户在当前线程调用otherThread.getThreadlocalMap()方法就好了。要解决这个问题，只能把对getThreadlocalMap()的访问封装起来，不让客户直接调用。

#### Version 2.0
- MyThread类
 ```java
 public class MyThread extends Thread {

    /**
     * 1. 线程可以有多个本地变量，一个本地变量对应一个值（对象），第一反应便是Map
     * 2. 并不是每个线程都一定会有threadlocalMap，没有必要在声明时就初始化。
     * 3. Map的key指定为String类型，value指定为Object类型。key即为本地变量，value即为本地变量对应的值
     *
     * since version 2.0
     * 为了配合MyThreadlocalodl类的泛型，将Map的key改为Object类型
     */
    private Map<Object, Object> threadlocalMap;

    /**
     * 1. 确保获取到的threadlocalMap是当前线程的threadlocalMap
     * 2. 第一次获取threadlocalMap时对其初始化
     *
     * since version 2.0
     * 3. 将方法设为包级私有
     */
     Map<Object, Object> getThreadlocalMap() {

        /*
         *since version 2.0
         *已经在MyThreadlocal类中确保是通过Thread.currentThread()获取threadlocalMap
         *
        //确保获取到的threadlocalMap是当前线程的threadlocalMap
        MyThread curThread = (MyThread) currentThread();
        Map<String, Object> curThreadlocalMap = curThread.threadlocalMap;*/

        /*
         * 不论在哪个线程调用getThreadlocalMap()方法，都会通过上面的代码获取到当前正在执行的线程，
         * 而线程的线性执行，使得这个方法不存在并发，不需要同步
         *
        if(curThreadlocalMap == null){
            synchronized (curThread){
                if(curThreadlocalMap == null){
                    curThreadlocalMap = new HashMap<>();
                }
            }
        }*/
        if (threadlocalMap == null) {
            threadlocalMap = new HashMap<>();
        }

        return threadlocalMap;
    }

    public MyThread(Runnable runnable) {
        super(runnable);
    }
}
 ```

- MyThreadlocal类
 ```java
 /**
 * Created by omed on 2017/3/12.
 * 封装getThreadlocalMap()访问的工具类
 */
public class MyThreadlocal<S,T> {

    public T get(S key) {
        return getThreadlocalMap().get(key);
    }

    public void set(S key,T value) {
        getThreadlocalMap().put(key,value);
    }

    /**
     * 确保是通过Thread.currentThread()获取threadlocalMap
     */
    private Map<S,T> getThreadlocalMap(){
       return  (Map<S,T>)((MyThread)Thread.currentThread()).getThreadlocalMap();
    }
}
 ```

- 调用
 ```java
 public class Main {
    private static final AtomicInteger nextId = new AtomicInteger(0);
    private static final String threadId = UUID.randomUUID().toString();

    public static void main(String[] args) throws InterruptedException {
        MyThreadlocal<String, Integer> idStorage = new MyThreadlocal<>();

        new MyThread(() -> {
            for (int i = 0; i < 10; i++) {
                new MyThread(() -> {
                    /*
                     * getThreadlocalMap()方法设置为包级私有且分包后
                     * 不能再调用((MyThread)Thread.currentThread()).getThreadlocalMap()
                     */
                    set(idStorage);
                    get(idStorage);
                }).start();
            }

            try {
                Thread.sleep(2000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            /*
             *验证set和get的是当前线程的id,且如果key相同则覆盖value
             */
            System.out.println("-------------------------------------------");
            set(idStorage);
            get(idStorage);
            set(idStorage);
            get(idStorage);

            /*
             * 验证线程的threadlocalMap可以存储多个值
             */
            System.out.println("-------------------------------------------");
            String newKey = UUID.randomUUID().toString();
            Integer newValue = 1024;
            idStorage.set(newKey,newValue);
            Integer integer = idStorage.get(newKey);
            System.out.println("threadName:"+Thread.currentThread().getName()+" newValue:"+integer);
            get(idStorage);
        }).start();
    }

    private static void set(MyThreadlocal<String, Integer> idStorage){
        idStorage.set(threadId, nextId.getAndIncrement());
    }

    private static void get(MyThreadlocal<String, Integer> idStorage){
        Integer id = idStorage.get(threadId);
        System.out.println("threadName:"+Thread.currentThread().getName()+" threadId:"+id);
    }
}
 ```

- 结果
 ```java
 threadName:Thread-2 threadId:1
 threadName:Thread-1 threadId:0
 threadName:Thread-4 threadId:2
 threadName:Thread-5 threadId:3
 threadName:Thread-6 threadId:4
 threadName:Thread-7 threadId:5
 threadName:Thread-10 threadId:6
 threadName:Thread-3 threadId:7
 threadName:Thread-8 threadId:8
 threadName:Thread-9 threadId:9
 -------------------------------------------
 threadName:Thread-0 threadId:10
 threadName:Thread-0 threadId:11
 -------------------------------------------
 threadName:Thread-0 newValue:1024
 threadName:Thread-0 threadId:11
 ```

- 结论
 - 基本已经满足需求
 - 有点不顺眼的是，MyThread类的threadlocalmap定义成了Map&lt;Object,Object&gt;类型，可是没必要再给MyThread引入泛型，毕竟并不是每一个Thread都一定会有thread-local variables 。
 - 同样要注意对Map的管理，防止内存泄漏。
 - 这样让用户自己管理key（本地变量），可以只有一个全局的MyThreadlocal对象，MyThreadlocal变成了一个纯粹的工具类，但同时增加了用户对key的管理难度，不利于协同开发，也有不安全因素。
  > 这种方法的问题在于，这些字符串键代表了一个共享的全局命名空间。要使这种方法可行，客户端提供的字符串键必须是唯一的：如果两个客户端各自决定为它们的线程局部变量使用同样的名称，它们实际上就无意中共享了这个变量，这样往往会导致两个客户端都失败。而且，安全性也差。恶意客户端可能有意地使用也另一个客户端相同的键，以便非法地访问其他客户端的数据。---- Effective Java 中文版（第2版） 第50条：如果其他类型更适合，则尽量避免使用字符串  P195
  > 永远不要让客户去做任何类库能够替客户完成的事情  ---- Effective Java 中文版（第2版） P47

## 3.2 Threadlocal设计
- Threadlocal实现的关键点在于有currentThread()方法来获取当前线程。
- 既然ThreadlocalMap是Thread的，那么为什么把ThreadlocalMap设计成Threadlocal的静态内部类，而非Thread的。下面这段话可能是一个很好的阐释：
> First it should be noted that ThreadLocalMap is a package-private class, thus it's not a part of the API, but an implementation detail which may change in future JDK versions if necessary.

 > Why it's not non-static? Just because non-static nested class (inner class) is bound to the concrete instance of the outer class. In our case it should be bound to the concrete ThreadLocal variable. This is just wrong: ThreadLocalMap is bounded to the thread and stores all the values of all ThreadLocal variables for given thread. Thus binding it to the concrete ThreadLocal instance is meaningless.

 > Maybe it looks more logical to make ThreadLocalMap the inner class of the Thread class. However ThreadLocalMap just does not need the Thread object to operate, so this would just add some unnecessary overhead. The reason it's located inside the ThreadLocal class is that the ThreadLocal is responsible for ThreadLocalMap creation. ThreadLocalMap is created for current thread only when the first ThreadLocal is set in this thread, but after that the same ThreadLocalMap is used for all other ThreadLocal variables.

 > Another alternative would be to hold ThreadLocalMap as separate package-private class in java.lang package. Nothing wrong with such option: it's just a matter of taste and depends on what developers want more: to group related functionality in single source file or to keep source file length smaller.

 >  ----   [Why design ThreadLocalMap as static nest class in ThreadLocal?](http://stackoverflow.com/questions/34781183/why-design-threadlocalmap-as-static-nest-class-in-threadlocal)

# 4. 结语
- 有错误或有疑义的地方，欢迎批评指定，会持续更新改正。
- 文中及其他一些测试代码地址：[JdkSourcecodeAnalysis-Threadlocal](https://github.com/oomeD/JdkSourcecodeAnalysis/tree/master/src/lang/threadlocal)
- 参考文章：
[彻底理解ThreadLocal](http://blog.csdn.net/lufeng20/article/details/24314381)
[ThreadLocal可能引起的内存泄露](http://www.cnblogs.com/onlywujun/p/3524675.html)
[Java并发编程：深入剖析ThreadLocal](http://www.cnblogs.com/dolphin0520/p/3920407.html)
[解密ThreadLocal](http://qifuguang.me/2015/09/02/[Java%E5%B9%B6%E5%8F%91%E5%8C%85%E5%AD%A6%E4%B9%A0%E4%B8%83]%E8%A7%A3%E5%AF%86ThreadLocal/)
[Is it really my job to clean up ThreadLocal resources when classes have been exposed to a thread pool?](http://stackoverflow.com/questions/13852632/is-it-really-my-job-to-clean-up-threadlocal-resources-when-classes-have-been-exp)
[Why design ThreadLocalMap as static nest class in ThreadLocal?](http://stackoverflow.com/questions/34781183/why-design-threadlocalmap-as-static-nest-class-in-threadlocal)
