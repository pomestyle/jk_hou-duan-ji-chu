# Java虚拟机

## 前言

Java是一种跨平台语言，可以在win , mac os linux 系统上运行，可以在手机上运行。

屏蔽了操作系统和不同Cpu的差异性。之所以Java有这种神奇的特性是因为Java编译的字节码文件不是直接运行在底层操作系统之上的，这点和c、c++ 这种语言不一样，编译好的字节码未见运行在Java虚拟机上（即JVM），JVM屏蔽了底层系统的不同，给Java字节码文件创建了一个统一的运行时环境。

JVM和普通应用程序一样，都运行在操作系统之上，启动后就是一个普通的进程，JVM 的全程是 Java Virtual Machine 。



## JVM的组成

JVM由类加载器，运行时数据区，执行引擎三部分构成。

![JVM](https://static001.geekbang.org/resource/image/de/3a/de1e68c3f603d1a7ab07d22a3761673a.png)

而运行时数据区又分为方法区，堆区，Java栈，程序计数器。

### 方法区

方法区主要存放从磁盘加载进来的字节码

### 堆区

程序在运行时期创建的Java对象存放在堆区（堆区存放的Java对象属性，即成员变量。）。

### Java 栈

程序在运行时期创建的Java对象的地址保存在栈中，方法里的变量也存放在栈中。

### 程序计数器

程序计数器一开始在main方法的第一行，jvm的执行引擎根据这个位置去方法区加载这个代码指令，CPU解析并且执行。

若果main方法里面有调用了其他方法，当线程进入其他方法的时候，会在Java栈中为这个方法创建一个新的栈帧，当线程在这个方法里面执行的时候，方法内部的变量都会存放在这个栈帧里，方法执行完毕推出的时候，就把这个扎针从Java栈中出栈，这样子当前栈帧，也就是堆栈的栈顶又回到了main放的栈帧，使用这个变量可以继续执行main方法。这样子即使方法变量相同，JVM也不会将方法变量弄错。

![](https://static001.geekbang.org/resource/image/a3/d9/a3de9184bfbd97546c291067d3106cd9.png)

## Java 的线程

从栈的角度去理解，方法内定义的变量都会被都会被每个方法的线程放入自己的栈中，现成的栈是隔离到的，所以这些变量一定是线程安全的。

另外堆中的对象也是安全的。



在Web 开发中，Servlet线程并不是安全的，但是如果将所有的变量定义在方法中，线程就是安全的。



## JVM 垃圾回收

JVM 不仅能屏蔽底层操作系统的差异性，还可以管理自身的内存，进行垃圾回收。清理自己本身不需要的垃圾对象，释放内存资源，提高系统运行。

JVM 回收垃圾有三种方法

![回收前](https://static001.geekbang.org/resource/image/91/f7/91e9bd4f5370fc22ec90ea7e093f3bf7.png)

回收前



### 1、清理

将垃圾对象在占据的内存清掉，实际上JVM并没有清理这些垃圾，而是将它放在一个空闲列表中，当程序需要的时候在进行分配。



![](https://static001.geekbang.org/resource/image/fc/03/fc259afcfb7bce6276c04656d4da8203.png)

### 2、压缩

从堆空间头部开始，将存活的对象拷贝到一段连续的空间中，剩下的空间就是连续的空闲空间。



![](https://static001.geekbang.org/resource/image/70/20/7040ed39531687afcb17f6b444101420.png)



### 3、复制

将堆空间分成两部分，一部分创建对象，当这个部分空间用完的时候，将标记过得可用对象复制到另一个空间中。

From区域和to区域,对象从from区域复制到to区域后，交换地址引用，from区域继续创建对象，直到满。

![](https://static001.geekbang.org/resource/image/7b/94/7b6a99a9bd7f9941ea4ae19a738cde94.png)





### 具体回收垃圾

当方法进栈出栈过程结束，期间创建的对象，对象引用放在栈中，存活时间很短，这个对象就是去引用了，成为垃圾。

对于这种情况，JVM将堆分为新城代（Young） 和 老年代 （Old）两个区域，差UN宫颈癌你对象的时候，只在新生代创建，新生代空间不足的时候启动垃圾回收，这样处理内存比较小，回收速度也快。新生代经过几次垃圾回收还没有被回收的对象将会被复制到老年区。



如果老年代空间满了，新生代无法将多次存活的对象复制进去，就会对新生代和老年代内存空间进行一次全量垃圾回收，也就是Full Gc。

由此可见，合理的设置老年代和新生代空间比例对JVM垃圾回收的性能有很大影响。

`JVM 设置老年代和新生代比例的参数是-XX:NewRatio`

![具体垃圾回收](https://static001.geekbang.org/resource/image/a5/f4/a5c3dbd6e992822d253f4d16b05555f4.png)



#### JVM 四种垃圾回收

##### 1、Serial 串行垃圾回收器

`JVM最早期的垃圾回收器，只有一个线程执行垃圾回收。`

##### 2、Parallel 并行垃圾回收器

`启动多线程执行垃圾回收。如果 JVM 运行在多核 CPU 上，那么显然并行垃圾回收要比串行垃圾回收效率高。`

在串行和并行垃圾回收过程中，当垃圾回收线程工作的时候，必须要停止用户线程的工作,否则会乱标记，造成对象错乱。

##### 3、CMS 并发垃圾回收器

`在垃圾回收的某些阶段，垃圾回收线程和用户线程可以并发运行，因此对用户线程的影响较小。Web 应用这类对用户响应时间比较敏感的场景，适用 CMS 垃圾回收器。`

##### 5 、G1 垃圾回收器

`它将整个堆空间分成多个子区域，然后在这些子区域上各自独立进行垃圾回收，在回收过程中垃圾回收线程和用户线程也是并发运行。G1 综合了以前几种垃圾回收器的优势，适用于各种场景，是未来主要的垃圾回收器。`



###### 5种垃圾回收器对比

![](https://static001.geekbang.org/resource/image/49/34/492f81e739aba5664ebaf0e08b467134.png)



