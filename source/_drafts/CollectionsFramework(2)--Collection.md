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

# 方法
![methods](/images/collections/collectionMethods.png)
Collection接口的实现类可以根据自身需求选择性实现或者不实现该接口定义的某些方法；也就是说，Collection接口的部分
方法有可能在他的实现类中不被支持。比如：Collections.UnmodifiableCollection中的add、remove、clear等操作类方法就都不被支持。
对于在实现类中不支持的方法，实现类可以抛出但不是必须抛出UnsupportedOperationException。比如：对不可变集合调用addAll(c)方法，
如果c是空集合（empty）就可以不抛UnsupportedOperationException异常而静默处理。

可选择性限制（optional-restrictions）
  - 有些Collection接口的实现类要求元素为非空（non null），对这些类执行contains(o)、remove(o)、
containsAll(c)、addAll(c)、removeAll(c)、retainAll(c)操作时，如果方法参数是null，或者参数中包含null元素，实现类可以选择抛NullPointerException或者直接返回false。
  - 有些Collection接口的实现类对元素的类型有要求，对这些类执行contains(o)、remove(o)、containsAll(c)、addAll(c)、removeAll(c)、
retainAll(c)操作时，如果参数类型不匹配或者参数中包含有类型不匹配的元素，实现类可以选择抛ClassCastException或者直接返回false。


- 构造方法<br>
接口没有构造方法，但是在java doc中明确说明，Collection接口的所有实现类都应当提供两个标准的构造方法
  - 无参的构造方法，生成空集合
  - 只有一个参数且参数类型为Collection<T>的构造方法，生成与参数元素一致的新集合。通过这个构造方法，用户可以复制任何集合，生成所需
 实现类型的等效集合。
- 具体方法<br>

