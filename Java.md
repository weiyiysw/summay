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

* HashMap:无序键值对，底层是数组+链表。
* LinkedHashMap:有序的，键值对，底层是双向链表
* TreeMap:有序的，底层是红黑树
* ConcurrentHashMap:JDK1.7使用分段锁，1.8使用CAS+synchronized实现

### Set

无序集合。不允许重复。常用实现：

* HashSet:底层实际上使用的是HashMap，存入的值是一个虚拟值。
* TreeSet:底层实际上是基于TreeMap的实现
* LinkedHashSet:

## 2. 多线程

什么是多线程？一个进程中，存在多个线程，并发的执行任务。多线程的目的就是为了更好的利用CPU资源。

并行与并发

* 并行：多个CPU实例或者多台机器同时执行一段处理逻辑，是真正的同时。
* 并发：通过CPU调度算法，让看上去是同时执行，实际从CPU的角度看并不是同时。并发往往在场景中有公用的资源，那么针对公用资源往往产生瓶颈。常用TPS和QPS来反应这个系统的处理能力。

![并行与并发](images/concurrent.png)

* 线程安全：一段代码，经过多线程使用，线程调度顺序不影响结果。线程不安全，则是线程的调度顺序会影响最终结果。
* 同步：通过人为的控制和调度，保证共享资源在多线程访问下成为线程安全，以此保证结果的准确。

### 线程的状态

![线程状态](images/thread_state.png)

### wait、notify、notifyAll

![](images/monitor.png)

synchronized, wait, notify 是任何对象都具有的同步工具。

wait、notify、notifyAll必须存在与synchronized块中，并且它们针对的是同一个监视器。

### synchronized

* 代码块：获取对象实例的monitor，如果实例相同，那么只有一个线程能执行。
* 直接作用于方法
  * 静态方法：锁定该类的monitor，不同的对象得进行同步。
  * 动态方法：锁定的是该类对象的monitor，不同的对象不影响。

#### volatile

多线程的内存模型：main memory（主存）、working memory（线程栈），在处理数据时，线程会把值从主存load到本地栈，完成操作后再save回去(volatile关键词的作用：每次针对该变量的操作都激发一次load and save)。

![volatile](images/volatile.png)

volatile还有另一个作用是禁止指令重排序。

针对多线程使用的变量如果不是volatile或者final修饰的，很有可能产生不可预知的结果（另一个线程修改了这个值，但是之后在某线程看到的是修改之前的值）。其实道理上讲同一实例的同一属性本身只有一个副本。但是多线程是会缓存值的，本质上，volatile就是不去缓存，直接取值。在线程安全的情况下加volatile会牺牲性能。

#### 线程类

* Thread类
* Runnable接口
* Callable接口

#### concurrent

##### ThreadLocal

##### 原子类

##### Lock

##### 容器类

##### 管理类

##### fork-join模型

##### 如何实现一个线程池

## 3. JVM

### Java内存区域

* 程序计数器：小块内存空间，当前线程所执行字节码的行号指示器。
* Java虚拟机栈：线程私有，生命周期与线程相同。
* 本地方法栈：为虚拟机使用到的native方法服务。
* Java堆：所有线程共享的内存区域，虚拟机启动时创建。
* 方法区：各个线程共享的内存区域。存储已被虚拟机加载的类信息、常量、静态变量、即时编译器编译后的代码等数据。
* 运行时常量池：方法区的一部分。
* 直接内存：并不是虚拟机运行时数据区的一部分。JDK1.4加入NIO，它可以是使用native函数库直接分配对外内存。

### 对象是否已死

* 引用计数法
* 可达性分析：可做gc roots对象
  * 虚拟机栈
  * 方法区类静态属性引用的对象
  * 方法区常量引用的对象
  * 本地方法栈JNI引用的对象

### 垃圾回收算法

* 标记-清除法
* 复制算法
* 标记-整理算法
* 分代收集算法

### 双亲委派模型



### Java内存模型



## 4. IO&NIO

## 5. 反射

#### 原理

Java反射机制是指在运行状态中，对于任意的一个类，都能够获取到这个类的所有属性和方法；对于任意一个对象，都能够调用它的任意一个方法和属性。这种动态获取信息以及动态调用对象的方法功能，就是Java的反射。

##### 获取Class方式

###### 1. Object类的getClass方法

```Java
// obj代表，代表一个类的对象
Class clz = obj.getClass()
```

###### 2.  静态class属性

```Java 
// Clazz代表任意一个类，
Class clz = Clazz.class
```

###### 3. forName方法

```Java
// forName方法，必须跟真实的带包名的类
try {
  Class clz = Class.forName("com.weiyiysw.Class");
} catch(ClassNotFoundException e) {
  e.printStackTrace();
}
```

总结：常用第三种方法。第一种方法，有对象了，获取类有点多余。第二种，必须导入包。第三种，传入一个字符串即可使用。

#### 使用

可以通过Class对象获取某个类的以下内容：

- 构造方法
- 成员变量
- 成员方法
- 访问成员

批量的方法： 

```Java
//所有”公有的”构造方法  
public Constructor[] getConstructors()
 
//获取所有的构造方法(包括私有、受保护、默认、公有) 
public Constructor[] getDeclaredConstructors() 
```

获取单个的方法，并调用： 

```Java
//获取单个的”公有的”构造方法
public Constructor getConstructor(Class… parameterTypes) 

//获取”某个构造方法”可以是私有的，或受保护、默认、公有； 
public Constructor getDeclaredConstructor(Class… parameterTypes)
```

 调用构造方法： 

```Java
 Constructor–>newInstance(Object… initargs) 
```

获取成员变量并调用： 

 批量的 

```java
// 获取所有的”公有字段” 
Field[] getFields()

// 获取所有字段，包括：私有、受保护、默认、公有；
Field[] getDeclaredFields()
```

获取单个的： 

```Java
// 获取某个”公有的”字段； 
public Field getField(String fieldName)

// 获取某个字段(可以是私有的) 
public Field getDeclaredField(String fieldName)
```

设置字段的值： 

```Java
// 参数说明： 
// obj:要设置的字段所在的对象； 
// value:要为字段设置的值；
Field –> public void set(Object obj,Object value)
```

 获取成员方法并调用： 

批量的： 

```Java
// 获取所有”公有方法”；（包含了父类的方法也包含Object类） 
public Method[] getMethods()
// 获取所有的成员方法，包括私有的(不包括继承的)
public Method[] getDeclaredMethods() 
```

获取单个的：

```Java
public Method getMethod(String name,Class<?>… parameterTypes): 
// 参数
// name : 方法名
// Class … : 形参的Class类型对象 
public Method getDeclaredMethod(String name,Class<?>… parameterTypes) 

```

调用方法： 

```Java
// 参数说明： 
// obj : 要调用方法的对象； 
// args:调用方式时所传递的实参；
Method –> public Object invoke(Object obj,Object… args): 
```

### 代理

#### 1. 静态代理

需要代理对象实现和目标对象一样的接口。

优点：不修改目标对象的前提扩展目标对象

缺点：冗余、不易维护

#### 2. 动态代理

利用了JDK API，动态的在内存中构建代理对象，从而实现对目标对象的代理功能。

动态代理对象不需要实现接口，但是目标对象必须实现接口。

动态代理本质是利用Java反射的能力，获取目标对象的类说明，通过类说明动态生成代理对象，由代理对象执行方法。

#### 3. cglib代理

通过第三方代码类库，运行时在内存中动态生成一个**子类**对象，从而实现目标对象功能的扩展。

Spring AOP使用此代理实现。此方法，会继承目标对象，所以需要重写方法。同时目标类，不能为final类。

## 6. 框架

### 1. Spring系列

#### Spring循环依赖

在Spring里，是由容器去创建对象，也就是通常所说的IOC。循环依赖是指，对象间相互依赖形成了环。比如，对象A依赖对象B，对象B依赖对象C，对象C依赖对象A，这样就构成了一个循环依赖。

循环依赖可以分为以下几种：

* 构造函数参数循环依赖：无法解决
* setter循环依赖
  * 单例模式：可解决
  * 非单例模式：无法解决

spring单例对象初始化的过程，大致可分为三步：

1. createBeanInstance：实例化bean，调用构造方法
2. populateBean：填充熟悉，主要对bean的熟悉进行填充
3. initializeBean：调用spring xml里配置的init方法，或者注解@PostConstruct声明的方法。

从单例bean的初始化的过程，可知，循环依赖是发生在第一步和第二步。

##### 构造器循环依赖

`this.singletonsCurrentlyInCreation.add(beanName)`将当前正要创建的bean记录在缓存中。Spring容器会将每一个正在创建的bean的标识符放到"当前创建bean池"，bean标识符在创建的过程中将一直保持在这个池中。因此，如果在创建bean的过程中发现自己已经在"当前创建bean池"里，则会抛出BeanCurrentlyInCreationException异常表示循环依赖。如果bean创建完成，那么将会从"当前创建bean池"移除。

##### setter单例循环依赖

Spring为解决单例循环依赖问题，使用了三级缓存。

~~~Java
/** Cache of singleton objects: bean name –> bean instance */
private final Map singletonObjects = new ConcurrentHashMap(256);
/** Cache of singleton factories: bean name –> ObjectFactory */
private final Map> singletonFactories = new HashMap>(16);
/** Cache of early singleton objects: bean name –> bean instance */
private final Map earlySingletonObjects = new HashMap(16);
~~~

三级缓存的作用：

~~~Java
singletonObjects: 单例对象的cache
singletonFactories: 单例对象工厂的cache
earlySingletonObjects: 提前曝光的单例对象cache
~~~

创建bean的代码如下：

~~~Java
protected Object getSingleton(String beanName, boolean allowEarlyReference) {
    Object singletonObject = this.singletonObjects.get(beanName);
    //isSingletonCurrentlyInCreation() 判断当前单例bean是否正在创建中
    if (singletonObject == null && isSingletonCurrentlyInCreation(beanName)) {
        synchronized (this.singletonObjects) {
            singletonObject = this.earlySingletonObjects.get(beanName);
            //allowEarlyReference 是否允许从singletonFactories中通过getObject拿到对象
            if (singletonObject == null && allowEarlyReference) {
                ObjectFactory<?> singletonFactory = this.singletonFactories.get(beanName);
                if (singletonFactory != null) {
                    singletonObject = singletonFactory.getObject();
                    //从singletonFactories中移除，并放入earlySingletonObjects中。
                    //其实也就是从三级缓存移动到了二级缓存
                    this.earlySingletonObjects.put(beanName, singletonObject);
                    this.singletonFactories.remove(beanName);
                }
            }
        }
    }
    return (singletonObject != NULL_OBJECT ? singletonObject : null);
}
~~~

首先从一级缓存 singletonObjects 获取，如果没有且当前指定的 beanName 正在创建，就再从二级缓存中 earlySingletonObjects 获取，如果还是没有获取到且运行 singletonFactories 通过 `getObject()` 获取，则从三级缓存 singletonFactories 获取，如果获取到则，通过其 `getObject()` 获取对象，并将其加入到二级缓存 earlySingletonObjects 中 从三级缓存 singletonFactories 删除。

##### 非单例循环依赖

对于“prototype”作用域bean, Spring 容器无法完成依赖注入，因为Spring 容器不进行缓 存“prototype”作用域的bean ，因此无法提前暴露一个创建中的bean 。

#### Spring事物分析

#### Spring拦截器原理

##### GET方法

##### POST方法

#### Spring MVC原理

#### Spring接口支持返回多格式

#### Spring里的设计模式

### 2. Netty

### 3. Disruptor

### 4. Vert.x

### 5. Guava

> [Guava](https://github.com/google/guava)GitHub仓库，可查看wiki，获取更多信息。

Guava项目包含了几个Google经常依赖使用的核心Java-Based项目：`collections` 、`caching`、`primitives support`、`concurrency libraries`、`common annotations`、`string processing`、`I/O`等。

#### 基本能力

##### 1. 使用时避免null

粗心的使用`null`会导致很多问题。谷歌调研了内部基本的代码，发现95%的集合都不会假定有null值存在。对开发者来说，遇到null时进行快速失败，而不是默默的接受将会更加有用。

使用Optional这个包装类型，Java8内置了该类型。

##### 2. Preconditions

Guava提供预先检查的能力，强烈建议静态的引入。每个方法都有三个变种：

1. 没有额外的参数。任何异常都会被抛出，并且不带有错误信息
2. 一个额外的 `Object` 参数。异常都会被抛出，并携带错误信息。
3. 一个额外的 `String` 参数，任意数量的 `Object` 的参数。多个`Object`参数和`printf`类似，但是在这个String参数里，只接受 `%s`指示。
   * 注意：`checkNotNull`, `checkArgument`, `checkState`三个方法有大量的基本类型的重载方法。使用基本类型的时候，避免自动装箱和类型转换开销。

~~~java
// 使用第三个变体的示例，这是静态引入了Preconditions
checkArgument(i >= 0, "Argument was %s but expected nonnegative", i);
~~~

| 方法签名（不包含额外的参数）                        | 说明                                                         | 失败时抛出的异常            |
| --------------------------------------------------- | ------------------------------------------------------------ | --------------------------- |
| `checkArgument(boolean)`                            | 检查 `boolean`是否是 `true`。验证传入方法的参数。            | `IllegalArgumentException`  |
| `checkNotNull(T)`                                   | 检查值是不是null。直接返回值，所以你可以直接在一行使用 `checkNotNull(value)` | `NullPointException`        |
| `checkState(boolean)`                               | 检查对象的一些状态，不依赖方法的参数。例如，一个 `Iterator`可以用它在调用 `remove`之前检查 `next`是否被调用。 | `IllegalStateException`     |
| `checkElementIndex(int index, int size)`            |                                                              | `IndexOutOfBoundsException` |
| `checkPositionIndex(int index, int size)`           |                                                              | `IndexOutOfBoundsException` |
| `checkPositionIndexs(int start, int end, int size)` |                                                              | `IndexOutOfBoundsException` |



#### 集合

##### 1. 不可变集合

##### 2. 新集合类型



### 6. Guice

[Guice](https://github.com/google/guice)是Google推出的轻量级DI框架，专注于DI，启动速度比Spring快。Guice包含以下几个要点。

- module: 将接口与接口的实现增加映射
- @Inject: 注解，引导guice进行注入。
- Injector: 获取module已配置的任意实例

## Bindings

Injector的工作是组装对象图。你给定类型来请求一个实例，Injector指示怎么去构建、解决依赖、并把所有都包装在一起。为了明确的解决依赖，使用**Bindings**配置Injector。

### 创建Bindings

创建Bindings，扩展`AbstractModule`并且覆盖`configure`方法即可。在方法体里，调用`bind()`方法确定每一个binding。只要你创建完了modules，将他们做为参数传入`Guice.createUInjector()`构建Injector。

Bindings有以下几种方式

1. linked bindings
2. instance bindings
3. @Providers methods
4. provider bindings
5. contructor bindings
6. untargetted bindings
7. built-in bindings
8. just-in-time binding

### 7. HiveMQ

## 7. 大数据

## 8. 消息队列

## 9. Redis



## 附录1：JVM调试

### jps

~~~shell
# jps 查看Java进程的pid
~~~

### jstat

~~~shell
# jstat 命令: 查看JVM的GC情况
# -gcutil 百分比、gc 实际占用内存
# usage: jstat -gcutil pid [间隔时间] [查询次数] 
# usage: jstat -gcutil pid [间隔时间] [查询次数]
~~~

`-gcutil`的参数含义

| Column | Description                            |
| :----- | :------------------------------------- |
| S0     | 第 0 个 survivor（幸存区）使用的百分比 |
| S1     | 第 1 个 survivor（幸存区）使用的百分比 |
| E      | `Eden` 区使用内存的百分比              |
| O      | 老生代内存使用的百分比                 |
| P/M    | `PermGen`/`MetaSpace` 的内存使用百分比 |
| YGC    | 程序启动以来 Young GC 发生的次数       |
| YGCT   | 程序启动以来 Young GC 共消耗的时间(s)  |
| FGC    | 程序启动以来 Full GC 发生的次数        |
| FGCT   | 程序启动以来 Full GC 共消耗的时间(s)   |
| GCT    | 程序启动以来 GC 的总用时(s)            |

`-gc`的输出列含义

| Column | Description                            |
| :----- | :------------------------------------- |
| SOC    | 第 0 个 Survivor 区的总空间 (KB).      |
| S1C    | 第 1 个 Survivor 区的总空间 (KB).      |
| S0U    | 第 0 个 Survivor 区已使用的空间 (KB).  |
| S1U    | 第 1 个 Survivor 区已使用的空间 (KB).  |
| EC     | Eden 区的总空间 (KB).                  |
| EU     | Eden 区已使用的空间 (KB).              |
| OC     | OldGen 的总空间 (KB).                  |
| OU     | OldGen 已使用的空间 (KB).              |
| PC/MC  | `PermGen`/`MetaSpace` 的总空间 (KB).   |
| PU/MU  | `PermGen`/`MetaSpace` 使用的空间 (KB). |
| YGC    | 程序启动以来 Young GC 发生的次数       |
| YGCT   | 程序启动以来 Young GC 共消耗的时间(s)  |
| FGC    | 程序启动以来 Full GC 发生的次数        |
| FGCT   | 程序启动以来 Full GC 共消耗的时间(s)   |
| GCT    | 程序启动以来 GC 的总用时(s)            |

### jmap

~~~shell
# to print java heap summary
> jmap -heap pid
~~~

### jstack

~~~shell
# jstack 打印堆栈信息
> jstack pid

# 需要结合top命令使用
~~~

#### top命令介绍

top命令是Linux下常用的性能分析工具，能够实时显示系统中各个进程的资源占用状况，类似于Windows的任务管理器。

top显示系统当前的进程和其他状况,是一个动态显示过程,即可以通过用户按键来不断刷新当前状态.如果在前台执行该命令,它将独占前台,直到用户终止该程序为止. 比较准确的说,top命令提供了实时的对系统处理器的状态监视.它将显示系统中CPU最“敏感”的任务列表.该命令可以按CPU使用.内存使用和执行时间对任务进行排序；而且该命令的很多特性都可以通过交互式命令或者在个人定制文件中进行设定. 

top输出某个特定进程pid并检查该进程内运行的线程状况：

~~~shell
top -Hp pid
~~~

