---
title: Threadlocal
category:
- JDK源码
- lang
date: 2017/03/13
---

# 1. Threadlocal源码
## 1.1 几个基本点
- Thread、Threadlocal、ThreadlocalMap、ThreadlocalMap.Entry的关系就好比老板（Thread）、助理（Threadlocal）、老板的公文包（ThreadlocalMap）、公文包里的文件夹（ThreadlocalMap.Entry）。
- 公文包（ThreadlocalMap）是老板（Thread）的，要么没有，要么只有一个。
  ```java
  public class Thread implements Runnable {
     //公文包是老板的，而且如果有，就只有一个，默认没有
     ThreadLocal.ThreadLocalMap threadLocals = null;
  }
  ```

- 老板（Thread）可以有多个助理（Threadlocal），在老板的指挥下，助理们都可以往公文包（ThreadlocalMap）或存或取或移除自己的文件（ThreadlocalMap.Entry.value）。为了便于管理，助理们要把文件放进一个用自己身份标识了的文件夹（ThreadlocalMap.Entry），然后再把文件夹放进公文包。助理只能拿到自己的文件夹，也便只能管理自己文件夹里的文件。当然，助理在或存或取或移除文件之前，都要先判断老板有没有公文包，公文包里有没有用记身份标识了的文件夹。如果没有就要创建新的。
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

- 老板（Thread）要操作某个文件（ThreadlocalMap.Entry.value）时就把管理这个文件的助理（Threadlocal）召唤到身边，由助理代为操作。助理操作自己放在当前老板公文包里的文件，要做的就是：在老板那拿到公文包（ThreadlocalMap），在公文包里拿到自己的文件夹（ThreadlocalMap.Entry），在文件夹里拿到文件。如果老板想和助理解除关系，那就把标识了该助理身份的文件夹取出来连带文件一起仍了吧。
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

            if (k == null) {
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
- 很多技术博客里都有“变量副本”的说法：
 > 当使用ThreadLocal维护变量时，ThreadLocal为每个使用该变量的线程提供独立的变量副本，所以每一个线程都可以独立地改变自己的副本，而不会影响其它线程所对应的副本。  ---- [彻底理解ThreadLocal](http://blog.csdn.net/lufeng20/article/details/24314381)

 > ThreadLocal为变量在每个线程中都创建了一个副本。    ---- [Java并发编程：深入剖析ThreadLocal](http://www.cnblogs.com/dolphin0520/p/3920407.html)

 “变量副本”这个词应该来自对Threadlocal类的Java Doc的翻译：
 > This class provides thread-local variables.  These variables differ from
 their normal counterparts in that each thread that accesses one (via its
 {@code get} or {@code set} method) has its own, independently initialized
 copy of the variable.

 意译过来大概是这样：<br>
Threadlocal提供了线程本地变量。这些变量与其他一般变量不同，通过它的get()或set()方法，每个线程就有它自己的，独立初始化了的变量副本。<br>
- 要厘清“变量副本”是什么，首先要厘清“变量”是什么。Thread的本地变量是指Threadlocal的set方法的参数value？按引用的两篇文章的意思是这样的。而事实不是！线程本地变量（thread-local variable）应该就是指Threadlocal实例。
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
- 厘清了“变量”是指Threadlocal的实例，“变量副本”就好理解了。正像之前比喻的，助理（Threadlocal）可以为多个老板（Thread）服务，多少老板服务，就会有多少个用自己身份标识了的放在各个老板公文包里的文件夹。每个文件夹的标识，即每个Entry的key就是变量副本，Threadlocal在一个Thread的ThreadlocalMap里做一次保存，就是一次copy。
- 这个时候，就可以水到渠成地说：变量副本是线程隔离的，每个线程只能看到和操作自己的副本。因为线程只能看到和操作自己的变量副本，所以客户在线程里调用threadlocal的set、get等方法都只会返回当然线程的数据，而不用担心在窜到别的线程。
- 如果把“变量”理解为Threadlocal的set方法的参数value，那么变量副本的说法是说不通的，在Threadlocal的源码中，没有对value参数进行拷贝（没有任何处理）；变量副本是线程隔离的说法更是说不通的，因为Threadlocal类没有对value有任何处理。作为对象，value仍然可以被其他线程访问和操作，至于是不是线程安全的，这取决于value对象本身，如果其本身是线程安全的，那便是线程安全的，如果不是线程安全的，那也必定不是线程安全的。这也是comments示例里nextId要用AtomicInteger类型的原因。


### 1.2.2 关于弱引用与内存泄漏

# 2. Threadlocal应用
## 2.1 应用场景
## 2.1 JFinal对Threadlocal的应用

# 3. Threadlocal设计
## 3.1 我来实现线程本地变量存储
### 3.1.1 需求
- 线程本地变量可以存储多个不同的键值对。
- 线程本地变量是线程隔离的，线程只是看见和操作自己的本地变量。

### 3.1.2 实现
#### Version 1.0
- MyThread类
 ```java
public class MyThread extends Thread {

    /**
     * 1. 存储多个不同的键值对，第一反应便是Map
     * 2. 并不是每个线程都一定会有threadlocalMap，没有必要在声明时就初始化。
     * 3. Map的key指定为String类型，value指定为Object类型。这明显不是一个好实现
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
     * 1. 存储多个不同的键值对，第一反应便是Map
     * 2. 并不是每个线程都一定会有threadlocalMap，没有必要在声明时就初始化。
     * 3. Map的key指定为String类型，value指定为Object类型。这明显不是一个好实现
     *
     * from version 2.0
     * 为了配合MyThreadlocalodl类的泛型，将Map的key改为Object类型
     */
    private Map<Object, Object> threadlocalMap;

    /**
     * 1. 确保获取到的threadlocalMap是当前线程的threadlocalMap
     * 2. 第一次获取threadlocalMap时对其初始化
     *
     * from version 2.0
     * 3. 将方法设为包级私有
     */
     Map<Object, Object> getThreadlocalMap() {

        /*
         *from version 2.0
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
                     * 不能再能过((MyThread)Thread.currentThread()).getThreadlocalMap()调用
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

## 3.2 大师的设计
## 3.3 Threadlocal设计模式应用

# 4. 结语
- 有错误或有疑义的地方，欢迎批评指定，会持续更新改正。
- 文中代码地址：
- 参考文章
 - [Java并发编程：深入剖析ThreadLocal](http://www.cnblogs.com/dolphin0520/p/3920407.html)
 - [解密ThreadLocal](http://qifuguang.me/2015/09/02/[Java%E5%B9%B6%E5%8F%91%E5%8C%85%E5%AD%A6%E4%B9%A0%E4%B8%83]%E8%A7%A3%E5%AF%86ThreadLocal/)
 - [Is it really my job to clean up ThreadLocal resources when classes have been exposed to a thread pool?](http://stackoverflow.com/questions/13852632/is-it-really-my-job-to-clean-up-threadlocal-resources-when-classes-have-been-exp)
