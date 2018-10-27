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
- Collection接口继承自Iterable接口

# 方法
![methods](/images/collections/collectionMethods.png)
- 构造方法
接口没有构造方法，但是在java doc中明确说明，Collection接口的所有实现类都应当提供两个标准的构造方法
 - 无参的构造方法，生成空集合
 - 只有一个参数且参数类型为Collection<T>的构造方法，生成与参数元素一致的新集合。通过这个构造方法，用户可以复制任何集合，生成所需
 实现类型的等效集合。
