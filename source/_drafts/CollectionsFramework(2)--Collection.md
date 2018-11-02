---
title: Collections Framework(2) -- Interface:Collection
category:
- JDK Source Code
- Collections Framework
date: 2018/10/27
---

# 总览
- 基于jdk8。
- Collection接口是集合框架的根接口，抽象了所有集合的共性。
- Collection接口没有直接实现类，只有一个抽象实现类AbstractCollection。
- Collection接口继承Iterable接口
- Collection接口所有实现类都实现序列化接口

# 方法
![methods](/images/collections/collectionMethods.png)

可选择性实现(optional-operation)
  - 实现类可以根据自身需求选择性实现或者不实现该接口定义的某些方法；也就是说，Collection接口的部分
方法有可能在他的实现类中不被支持。比如：Collections.UnmodifiableCollection中的add、remove、clear等操作类方法就都不被支持。
  - 对于在实现类中不支持的方法，实现类可以抛出但不是必须抛出UnsupportedOperationException。比如：对不可变集合调用addAll(c)方法，
如果c是空集合（empty）就可以不抛UnsupportedOperationException而静默处理。

可选择性限制(optional-restrictions)
  - 如果实现类要求元素为非空（non-null），对这些类执行contains(o)、containsAll(c)等操作时，如果方法参数是null，或者参数中包含null元素，实现类可以选择抛NullPointerException或者直接返回false。
  - 如果实现类对元素的类型有要求，对这些类执行contains(o)、containsAll(c)等操作时，如果参数类型不匹配或者参数中包含有类型不匹配的元素，实现类可以选择抛ClassCastException或者直接返回false。

如果实现类没有线程安全机制，则所有方法都非线程安全，包括接口的默认方法(default method：不管是接口自身定义还是继承而来)。如果实现类有线程安全机制，也应重写(override)默认方法以保证线程安全。

接口的很多方法都基本于Object的hashCode()和equals(o)方法定义。比如：contains(o)、remove(o)等，这些方法的执行结果也取决于元素的hashCode()和equals(o)的执行结果。

要注意处理和避免自引用。不管是直接自引用还是间接自引用，执行像clone()、equals()、hashCode()、toString()这些方法（递归遍历）时，都会出错。

- 构造方法<br>
Collection接口的所有实现类都应当提供两个标准的构造方法
  - 无参的构造方法，生成空集合
  - 只有一个参数且参数类型为Collection<T>的构造方法，生成与参数元素一致的新集合。通过这个构造方法，用户可以复制任何集合，生成所需
 实现类型的等效集合。
- 具体方法<br>

集合查询操作：
  - int size()
  返回集合元素个数，如果元素个数大于(Integer.MAX_VALUE = 2147483647)也只返回Integer.MAX_VALUE。在多线程操作情况下，返回的元素个数可能为脏数据。比如:ConcurrentLinkedQueue的size()方法。
  - boolean isEmpty()
  判断集合是不是没有元素的空集合，是返回true，不是返回false。
  - boolean contains(Object o)
  当且仅当集合里包含至少一个与o相同元素时返回true。
  元素是否相同取决于元素的hashCode()和equals()方法。
  适用optional-restrictions，如果参数o的类型与集合泛型类型不匹配，可以选择抛ClassCastException，也可以选择直接返回false。
  适用optional-restrictions，对于不允许null元素的集合，如果o为null，可以选择抛NullPointerException，也可以选择直接返回false。
  - Iterator<E> iterator()
  继承自Iterable接口。返回集合的迭代器对象。
  迭代器返回元素的顺序不一定有序。其中List、Deque、NavigableSet子接口的实现类及LinkedHashSet类按一定顺序返回元素，其余的实现类都不是。
  - Object[] toArray()
  将集合转换成数组。数组元素的顺序与迭代器返回元素(Iterator.next())的顺序保持一致。
  返回的数组是新分配的数组，与原集合是两个不相关的对象，即使原集合底层是由数组实现(像ArrayList)。两个对象间没有引用关系，对这两个对象的操作也互不影响（不指对对象里元素的操作）。
  - <T> T[] toArray(T[] a)
  将集合转换成指定类型T的数组。数组元素的顺序与迭代器返回元素(Iterator.next())的顺序保持一致。
  T的类型必须与集合中元素类型相同或者是集合中元素的父类型，否则抛ArrayStoreException。
  返回的数组是新分配的数组，与原集合是两个不相关的对象，即使原集合底层是由数组实现(像ArrayList)。两个对象间没有引用关系，对这两个对象的操作也互不影响（不指对对象里元素的操作）。
  如果a.length大于等于集合的size，则将集合元素放入a中，此时就将a用作返回值；如果a.length小于集合的size，则重新new一个同样类型的length足够大(length=c.size())的新数组用来承载集合的元素并用作返回值。

集合修改操作
  - boolean add(E e)
  适用optional-operation，实现类可以选择不支持此方法而抛出UnsupportedOperationException。
  实现类必须对是否允许null元素以及是否允许重复元素有明确定义。
  当集合不允许有重复元素，而集合又已包含e元素时，方法返回false。当方法执行完，集合对象有变动时，返回true。其他情况都应抛出异常（不适用optional-restrictions）。
  如果元素类型不匹配，抛出ClassCastException。
  如果集合元素要求non-null而e==null，抛出NullPointerException。
  如果元素的某些属性不符合集合元素的要求，抛出IllegalArgumentException。比如:如果在开发需求中，要求ArrayList<String>中的元素不为空串，而e为空串，则此时
  如果集合当前状态不对（插入限制）无法添加元素，抛出IllegalStateException。
  
  - boolean remove(Object o)
  适用optional-operation，实现类可以选择不支持此方法而抛出UnsupportedOperationException。
  从集合中移除一个与o相同（通过元素的hashCode()与equals(o)方法比较）的元素。如果集合中有多个与o相同的元素，也只移除一个。
  元素移除成功则返回true。
  适用optional-restrictions，如果参数o的类型与集合泛型类型不匹配，可以选择抛ClassCastException，也可以选择直接返回false。
  适用optional-restrictions，对于不允许null元素的集合，如果o为null，可以选择抛NullPointerException，也可以选择直接返回false。
  
集合批处理
  - boolean containsAll(Collection<?> c)
  如果c==null，抛出NullPointerException。
  如果集合c中的所有元素都包含在当前集合中，返回true。元素比较方法与contain(o)一致。
  适用optional-restrictions，如果集合c中有元素的类型与当前集合泛型类型不匹配，可以选择抛ClassCastException，也可以选择直接返回false。
  适用optional-restrictions，如果集合c中包含null元素，而当前集合要求元素为non-null，可以选择抛NullPointerException，也可以选择直接返回false。
  
  - boolean addAll(Collection<? extends E> c)
  适用optional-operation，实现类可以选择不支持此方法而抛出UnsupportedOperationException。
  
  
  
  
  

