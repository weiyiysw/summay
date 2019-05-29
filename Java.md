# Java

## 1. 集合

![Java集合框架图](images/collection.gif)

![集合框架图2](images/collection2.png)

Java中，集合是存储对象最常用的一种方式。学习和熟练使用集合，对开发工作有事半功倍的效果。集合主要分三种类型，分别是List、Map以及Set。

> 数组虽然也可以存储对象，但数组长度固定；集合的长度是可变的。数组可以存储基本类型，但集合只能存储对象。

### List

特点：有序集合，可以存放重复的内容。常用的有两种实现：

* ArrayList：底层是数组，可随机访问，查找方便。
* LinkedList：底层是双向链表，增删方便，查找慢。

以上两种实现，根据其底层使用的数据结构，均有自己的特点。

可以用LinkedList实现LRU算法。

#### 常见问题

1. ArrayList里移除元素，可使用Iterator的remove，不可以使用list remove方法。

### Map

特点：Key-Value键值对。常用实现：

* HashMap:
* LinkedHashMap:
* TreeMap:
* ConcurrentHashMap:

### Set

无序集合，

## 2. 多线程

### wait、notify、notifyAll

### synchronized



## 3. JVM

## 4. IO&NIO

## 5. 反射

## 6. 框架

### 1. Spring系列

### 2. Netty

### 3. Disruptor

### 4. Vert.x

### 5. Guava

## 7. 大数据

