---
title: Threadlocal
category:
- JDK源码
- lang
date: 2017/03/13
---

# 1. Threadlocal源码
## 1.1 老板、助理与公文包
- Thread、Threadlocal、ThreadlocalMap三者的关系就好比老板（Thread）、助理（Threadlocal）、老板的公文包（ThreadlocalMap）。公文包由助理管理，但并不由助理保管，并且老板只有一个公文包。ThreadlocalMap的引用由Thread持有，Threadlocal并不是ThreadlocalMap的所有者，只是对ThreadlocalMap代为管理。
 ```java
 public class Thread implements Runnable {
    //公文包是老板的，而且如果有，就只有一个。
    ThreadLocal.ThreadLocalMap threadLocals = null;
 }
 ```

- 助理（Threadlocal）只能往公文包（ThreadlocaMap）里放一个文件夹（ThreadlocalMap.Entry），并用自己的身份做标识，而替老板管理的文件（Entry.value）就放在这个文件夹中了。
   ```java
   static class ThreadLocalMap {
     //文件夹在公文包中
     static class Entry extends WeakReference<ThreadLocal<?>> {
         Object value;//替老板管理的文件，可以是一份文件，也可以是一摞文件。对文件的操作老板有个要求，要么是全部替换，要么是全部拿出

         Entry(ThreadLocal<?> k, Object v) {//用助理身份标识了的文件夹
             super(k);
             value = v;
         }
     }
   }
   ```

- 助理（Threadlocal）每次用公文包（ThreadlocalMap）都要先找到老板（Thread），再从老板那里取。当然，如果在要存文件的时候老板还没有公文包，那就给老板买个新的。Threadlocal所有导出api（包括set,get,remove方法）都是先获取当前线程对象，再通过对象获取ThreadlocalMap，如果没有就create一个。
 ```java
public class ThreadLocal<T> {
    void createMap(Thread t, T firstValue) {//给老板买新公文包
     t.threadLocals = new ThreadLocalMap(this, firstValue);//把件放入用自己身份标识了的文件夹
    }

    ThreadLocalMap getMap(Thread t) {//找老板要公文包
      return t.threadLocals;
    }

    public T get() {
        Thread t = Thread.currentThread();//找到当前的老板
        ThreadLocalMap map = getMap(t);//找老板要公文包
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

- 老板（Thread）可以有多个助理（Threadlocal），只要拥有助理证（是Thradlocal的实例）而且和老板碰见了（线程调用了有Threadlocal对象引用的方法，碰见了就意味着要调用Threadlocal的set、get或者remove方法）了，就能成为也一定会成为老板的助理或者已经是老板的助理了。所有助理共同管理老板唯一的公文包，但都只能操作自己的文件夹。正因为如此，一切有条不紊地进行着。

- 老板（Thread）不用关心，也不用记住自己有多少个助理（Threadlocal）。如果真想知道，打开公文包（ThreadLocalMap），看看公文包里文件夹(ThreadLocalMap.Entry)的个数，及文件夹上面的信息就可以了。

- 同样地，助理（Threadlocal）也可以为多个老板（Thread）服务,也不用理会自己到底服务了多少老板，只要一与老板碰见，就找老板拿到公文包（ThreadLocalMap），要么拿出文件操作一翻（get），要么替换之前的所有文件(set)，或者干脆把自己的文件夹从公文包里取出(remove)暂时不为这个老板服务（下次碰见这个老板还是要再次把文件夹放进老板的公文包）。

## 1.2 几个问题
### 1.2.1 关于创建变量副本与线程隔离
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
