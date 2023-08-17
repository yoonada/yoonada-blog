---
title: JVM学习总结
date: 2023-07-25 16:28:47
tags: [JVM]
categories: [JVM]
---
### JVM是什么
JVM（Java Virtual Machine），Java虚拟机，是JRE的一部分。JVM有自己完善的硬件架构，还具备相应的指令系统，是实现跨平台的关键。
### JVM的组成
* 类加载子系统
* 运行时数据区
* 执行引擎
![](https://yoonada.oss-cn-shenzhen.aliyuncs.com/images/image-20230725165133830.png)
### 类加载子系统

#### 类加载器

类加载器是一个负责加载类的对象，用于实现类加载过程中的加载这一步

#### 职责
负责加载字节码文件（.class），而字节码文件存储于本地磁盘中，类加载子系统会以二进制流的方式把字节码文件从物理磁盘加载到内存中（方法去），生成Class对象，该Class对象是作为方法去的这个类的各种数据访问入口。
#### 类加载的生命周期

![](https://yoonada.oss-cn-shenzhen.aliyuncs.com/images/image-20230725170607947.png)

* 加载：查找并加载类的二进制数据
* 连接
  * 验证：确保被加载的类的正确性
  * 准备：为类的静态变量分配内存，并将其初始化为默认值
  * 解析：把类的符号转换为直接引用
* 初始化：为类的静态变量赋予正确的初始值，JVM负责对类进行初始化
* 使用
* 卸载：结束生命周期

#### 类加载器的层次

![](https://yoonada.oss-cn-shenzhen.aliyuncs.com/images/image-20230725171120062.png)

* 启动类加载器（Bootstrap ClassLoader）：**最顶层的加载类，由C++实现，主要用来加载 JDK 内部的核心类库**（ `%JAVA_HOME%/lib`目录下的 `rt.jar`、`resources.jar`、`charsets.jar`等 jar 包和类）以及被 `-Xbootclasspath`参数指定的路径下的所有类。该类加载器**无法**被Java程序直接引用。
* 扩展类加载器（Extension ClassLoader）：负责加载JAVA_HOME\lib\，该加载器可以被开发者直接使用。
* 应用程序类加载器：ApplicationClassLoader：面向我们用户的加载器，负责加载当前应用 classpath 下的所有 jar 包和类。
* 自定义类加载器

#### JVM有哪些类加载机制

* 全盘负责
* 父类委托
* 缓存机制
* 双亲委派机制

![](https://yoonada.oss-cn-shenzhen.aliyuncs.com/images/image-20230726111420785.png)

#### 双亲委派

如果一个类加载器收到了类加载的请求，它首先不会自己去尝试加载这个类，而是把请求委托给父加载器去完成，依次向上，因此，所有的类加载请求最终都应该被传递到顶层的启动类加载器中，只有当父加载器在它的搜索范围中没有找到所需的类时，即无法完成该加载，子加载器才会尝试自己去加载该类。

##### 双亲委派机制过程

* 在类加载的时候，系统会首先判断当前类是否被加载过。已经被加载的类会直接返回，否则才会尝试加载
* 类加载器在进行类加载的时候，它首先不会自己去尝试加载这个类，而是把这个请求委派给父类加载器去完成（调用父加载器 `loadClass()`方法来加载类）。这样的话，所有的请求最终都会传送到顶层的启动类加载器 `BootstrapClassLoader` 中。
* 只有当父加载器反馈自己无法完成这个加载请求（它的搜索范围中没有找到所需的类）时，子加载器才会尝试自己去加载（调用自己的 `findClass()` 方法来加载类）。

##### 双亲委派的好处

**双亲委派模型保证了Java程序的稳定运行，可以避免类的重复加载，也保证了Java的核心API不被篡改**。

如果没有使用双亲委派模型，而是每个类加载器加载自己的话就会出现一些问题，比如我们编写一个称为 `java.lang.Object` 类的话，那么程序运行的时候，系统就会出现两个不同的 `Object` 类。双亲委派模型可以保证加载的是 JRE 里的那个 `Object` 类，而不是你写的 `Object` 类。这是因为 `AppClassLoader` 在加载你的 `Object` 类时，会委托给 `ExtClassLoader` 去加载，而 `ExtClassLoader` 又会委托给 `BootstrapClassLoader`，`BootstrapClassLoader` 发现自己已经加载过了 `Object` 类，会直接返回，不会去加载你写的 `Object` 类。

### 运行时数据区

JVM在执行Java程序的过程中会把它管理的内存划分成若干个不同的数据区域

* 堆（Heap）
* 方法区（Method Area）
* 虚拟机栈（VM Stack）
* 本地方法栈（Native Method Stack）
* 程序计数器（Program Counter Register）

#### 线程私有

* 程序计数器
* 虚拟机栈
* 本地方法栈

#### 线程共享

* 堆
* 方法区

#### JDK1.7

![](https://yoonada.oss-cn-shenzhen.aliyuncs.com/images/image-20230726142110611.png)

#### JDK1.8

![](https://yoonada.oss-cn-shenzhen.aliyuncs.com/images/image-20230726142207011.png)

#### 程序计数器（Program Counter）

程序计数器是一块较小的内存区域，它**存储当前线程正在执行的Java字节码指令的地址或索引**。

#### 虚拟机栈（VM Stack）

Java虚拟机栈用于存储Java方法的局部变量、方法参数、部分计算结果和方法调用的栈帧信息

#### 本地方法栈（Native Method Stack）

本地方法栈与虚拟机栈类似，但它用于执行Native方法

#### 堆（Heap）

堆是JVM中最大的一块内存区域，用于存储Java对象实例以及数组。在程序运行期间，所有的对象实例都在Java堆中动态分配内存。被所有线程共享，是垃圾回收的主要区域。



Java堆是垃圾收集器管理的主要区域，因此也称为GC堆（Garbage Collected Heap）

##### 堆内存的划分

###### JDK7及其之前

* 新生代内存（Young Generation）
* 老生代（Old Generation）
* 永生代（Permanent Generation）

###### JDK8

* 新生代内存（Young Generation）
* 老生代（Old Generation）
* 元空间（MetaSpace）

![](https://yoonada.oss-cn-shenzhen.aliyuncs.com/images/image-20230726151527764.png)

###### JDK1.7和JDK1.8堆内存的区别

* JDK1.7及之前的版本使用永生代作为方法区的实现。永生代用于存储类的信息、常量、静态变量和即时编译器编译后的代码。因永生代的大小是有限的，且默认情况比较小，故而出经常出现空间溢出的错误
* JDK1.8引入了元空间来替代永生代，元空间并不在虚拟机中，而是使用本地内存。支持自动调整大小

总结：JDK 1.8的元空间相比于JDK 1.7的永久代在内存管理方面更加灵活和健壮，不易发生内存溢出问题，并且不再需要手动调整永久代的大小。

#### 方法区（Method Area）

方法区用于存储类的信息、常量、静态变量。线程共享

##### 运行时常量池

运行时常量池是方法区的一部分，用于存储编译时生成的各种字面量和符号引用

### 执行引擎

### JVM参数配置

#### 调优堆栈内存
* -Xms：最小堆内存
* -Xmx：最大堆内存
* -Xmn：新生代内存大小（Sun官方推荐配置为整个堆的3/8）
* -Xss：线程的栈内存大小

### 垃圾回收器
* 串行垃圾收集器
* 并行垃圾收集器
* CMS垃圾收集器
* G1垃圾收集器

#### 判断对象是否死亡
* 引用计数法：给每一个对象中添加一个计数器，一旦有地方引用了此对象，则该对象的计数器加1，如果引用失效了，则计数器减1。这样当计数器为0时，就代表此对象没有被任何地方引用。
* 可达性分析：通过一系列被称为“GCroot”的对象作为起始点，从这些节点开始向下搜索，走过的路径叫做引用链。如果一个对象没有通过引用链连接到GCroot节点，则证明此对象是不可用的

### Full GC 与 Minor GC
* Minor GC是指对JVM新生代执行的垃圾回收操作。在Minor GC中，只有新生代中的对象会被检查，不再被引用的对象将被回收。存活下来的对象会被移动到Survivor空间或晋升到老年代。Minor GC通常发生频繁，但回收的对象数量相对较少。
* Full GC是指对JVM老年代执行的垃圾回收操作。老年代是存放长时间存活的对象的内存区域。Full GC会对整个堆内存进行垃圾回收，包括新生代和老年代。这种类型的垃圾回收通常会导致较长的停顿时间，因为它需要检查和回收大量的对象。Full GC通常发生较少，但回收的对象数量相对较大。

#### 触发Full GC的情况
* 老年代内存不足：当老年代中没有足够的空间来分配一个大对象时，会先尝试进行Minor GC，如果任然无法获得足够的空间，则会触发Full GC
* 调用System.gc()：使用System.gc()方法不能保证立即进行垃圾回收，但是这个方法可以提示JVM进行垃圾回收。如果此时需要更多的内存空间，那么就可能会触发Full GC。
* Perm区空间不足：Perm区是存放类信息和常量池等元数据的区域，如果Perm区没有足够的空间来存放这些信息，就会触发Full GC。
* CMS GC出现Concurrent Mode Failure：CMS（Concurrent Mark Sweep）是一种以最小化停顿时间为目标的垃圾收集器，在CMS执行过程中，如果应用程序产生了大量更新，导致CMS回收速度跟不上对象生成速度，那么就可能会出现Concurrent Mode Failure，此时会启动Full GC来清理整个堆空间。
* 分配担保失败：在Minor GC后，如果survivor区无法容纳所有幸存对象，那么就要将部分幸存对象转移到老年代。如果老年代剩余空间不足以容纳这些对象，就需要进行Full GC。

* 老年代空间不足
* 永生代、元空间空间不足
* 调用System.gc()
* 
