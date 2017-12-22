---
title: Android手机连接编译器失败，端口号被占用
excerpt: 编译
categories:
  - topics
tag: 扩展了解  
---

> 如果本文帮助到你，本人不胜荣幸，如果浪费了你的时间，本人深感抱歉。
希望用最简单的大白话来帮助那些像我一样的人。如果有什么错误，请一定指出，以免误导大家、也误导我。
本文来自：http://www.jianshu.com/users/320f9e8f7fc9/latest_articles
感谢您的关注。

Java的内存是不用我们开发者自己来管理的，这个大家都知道，但是那它到底是怎么运作的呢？
我们都知道GC，也就是垃圾回收机制，但到底什么是GC。
我们一起来看看。

#### 什么是GC 
**垃圾回收**是一种自动的存储管理机制。当一些被占用的内存不再需要时，就应该予以释放，以让出空间，这种存储资源管理，称为垃圾回收（garbage collection）。垃圾回收器可以让程序员减轻许多负担，也减少程序员犯错的机会。
我们来主要看看Java的gc机制。

整个Java堆可以切割成为三个部分：

1.  Young(年轻代)：
    1. Eden（伊利园）：存放新生对象。
    2. Survivor（幸存者）：存放经过垃圾回收没有被清除的对象。
2.  Tenured(老年代)：对象多次回收没有被清除，则移到该区块。
3.  Perm：存放加载的类别还有方法对象。

#### GC会造成什么影响
在开始学习GC之前你应该知道一个词：stop-the-world。不管选择哪种GC算法，stop-the-world都是不可避免的。
也就是说，当垃圾回收开始清理资源时，其余的所有线程都会被停止。所以，我们要做的就是尽可能的让它执行的时间变短。如果清理的时间过长，在我们的应用程序中就能感觉到明显的卡顿。

#### 什么情况下GC会执行
因为它对系统影响很明显，所以它到底在什么时候执行呢？

总的来说,有两个条件会触发主GC:

1. 当应用程序空闲时,即没有应用线程在运行时,GC会被调用。因为GC在优先级最低的线程中进行,所以当应用忙时,GC线程就不会被调用,但以下条件除外。
2. Java堆内存不足时,GC会被调用。当应用线程在运行,并在运行过程中创建新对象,若这时内存空间不足,JVM就会强制地调用GC线程,以便回收内存用于新的分配。若GC一次之后仍不能满足内存分配的要求,JVM会再进行两次GC作进一步的尝试,若仍无法满足要求,则 JVM将报“out of memory”的错误,Java应用将停止。

由于是否进行主GC由JVM根据系统环境决定,而系统环境在不断的变化当中,所以主GC的运行具有不确定性,无法预计它何时必然出现,但可以确定的是对一个长期运行的应用来说,其主GC是反复进行的。 

#### 垃圾回收的一般步骤
之前已经了解到Java堆被主要分成三个部分，而垃圾回收主要是在Young（年轻代）和Tenured（老年代）工作。
而 年轻代 又包括 Eden（伊利园）和两个Survivor（幸存者）。
下面我们就来看看这些空间是如何进行交互的：

1、首先，所有新生成的对象都是放在年轻代的Eden分区的，初始状态下两个Survivor分区都是空的。年轻代的目标就是尽可能快速的收集掉那些生命周期短的对象。

 ![](http://upload-images.jianshu.io/upload_images/1689895-654a5844f0f18097.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

2、当Eden区满的的时候，小垃圾收集就会被触发。

![](http://upload-images.jianshu.io/upload_images/1689895-e2a92ebc88281bf4.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

3、当Eden分区进行清理的时候，会把引用对象移动到第一个Survivor分区，无引用的对象删除。

![](http://upload-images.jianshu.io/upload_images/1689895-71e380d505f45a04.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

4、在下一个小垃圾收集的时候，在Eden分区中会发生同样的事情：无引用的对象被删除，引用对象被移动到另外一个Survivor分区（S1）。此外，从上次小垃圾收集过程中第一个Survivor分区（S0）移动过来的对象**年龄增加**，然后被移动到S1。当所有的幸存对象移动到S1以后，S0和Eden区都会被清理。注意到，此时的Survivor分区存储有不同年龄的对象。

![](http://upload-images.jianshu.io/upload_images/1689895-6b9c3b08b0012e43.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

5、在下一个小垃圾收集，同样的过程反复进行。然而，此时Survivor分区的角色**发生了互换**，引用对象被移动到S0，幸存对象年龄增大。Eden和S1被清理。

![](http://upload-images.jianshu.io/upload_images/1689895-0c1e29b68b0a2322.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

6、这幅图展示了从年轻代到老年代的提升。当进行一个小垃圾收集之后，如果此时年老对象此时到达了某一个个年龄阈值（例子中使用的是8），JVM会把他们从年轻代提升到老年代。

![](http://upload-images.jianshu.io/upload_images/1689895-a59c4dd20f78007b.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

7、随着小垃圾收集的持续进行，对象将会被持续提升到老年代。

![](http://upload-images.jianshu.io/upload_images/1689895-625ef8f9aa75eabf.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

8、这样几乎涵盖了年轻一代的整个过程。最终，在老年代将会进行大垃圾收集，这种收集方式会清理-压缩老年代空间。

![](http://upload-images.jianshu.io/upload_images/1689895-5a33e76d5979ce16.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

<br />
也就是说，刚开始会先在新生代内部反复的清理，顽强不死的移到老生代清理，最后都清不出空间，就爆炸了。

#### 与堆配置相关的参数

|参数    |    描述|
|----|----|
|-Xms|    JVM启动的时候设置初始堆的大小|
|-Xmx|    设置最大堆的大小             |
|-Xmn|    设置年轻代的大小             |
|-XX:PermSize|    设置持久代的初始的大小|
|-XX:MaxPermSize|    设置持久代的最大值|

#### 优化建议：
根据GC的工作原理，我们可以通过一些技巧和方式，让GC运行更加有效率

1. 最基本的建议就是尽早释放无用对象的引用。大多数程序员在使用临时变量的时候，都是让引用变量在退出活动域（scope）后，自动设置为 null。好的做法是：如果程序允许，尽早将不用的引用对象赋为null，这样可以加速GC的工作；
2. 尽量少用finalize函数。finalize函数是Java提供给程序员一个释放对象或资源的机会。但是，它会加大GC的工作量，因此尽量少采用finalize方式回收资源；
3. 如果需要使用经常使用的图片，可以使用SoftReference类型。它可以尽可能将图片保存在内存中，供程序调用，而不引起OutOfMemory；
4. 注意集合数据类型，包括数组，树，图，链表等数据结构，这些数据结构对GC来说，回收更为复杂，所以使用结束应立即置为null，不要等堆在一起。另外，注意一些全局的变量，以及一些静态变量。这些变量往往容易引起悬挂对象（dangling reference），造成内存浪费；
5. 当程序**有一定的等待时间**(注意，是有一定等待时间时)，程序员可以手动执行System.gc()，通知GC运行，但是Java语言规范并不保证GC一定会执行。使用增量式GC可以缩短Java程序的暂停时间。**System.gc()；   Runtime.getRuntime().gc() 这个方法对资源消耗较大尽量不要手动去调用这个方法，不然可能引起程序的明显卡顿**；
6. 尽量使用StringBuffer,而不用String来累加字符串
7. 能用基本类型如int,long,就不用Integer,Long对象。基本类型变量占用的内存资源比相应对象占用的少得多；
8. 尽量少用静态对象变量，静态变量属于全局变量,不会被GC回收,它们会一直占用内存；
9. 分散对象创建或删除的时间，集中在短时间内大量创建新对象,特别是大对象,会导致突然需要大量内存,JVM在面临这种情况时,只能进行主GC,以回收内存或整合内存碎片,从而增加主GC的频率。

#### 合理使用 软引用 和 弱引用

**清除** 将引用对象的 referent 域设置为 null ，并将引用类在堆中引用的对象声明为 可结束的。
**StrongReference** 强引用：正常的对象，一直不会清理，直到爆炸。
**SoftReference** 软引用：内存不够的时候，遇到了就清了。
**WeakReference** 弱引用：只要遇到了就清了，注意：可能清了好几次，都没遇到。
**PhantomReference** 虚引用：虚顾名思义就是没有的意思，建立虚引用之后通过get方法返回结果始终为null。

PhantomReference 必须与 ReferenceQueue 类一起使用。需要 ReferenceQueue 是因为它能够充当通知机制。当垃圾收集器确定了某个对象是虚可及对象时， PhantomReference 对象就被放在它的 ReferenceQueue 上。将 PhantomReference 对象放在 ReferenceQueue 上也就是一个通知，表明 PhantomReference 对象引用的对象已经结束，可供收集了。这使您能够刚好在对象占用的内存被回收之前采取行动。

给个软引用的例子：

    //首先定义一个HashMap，保存软引用对象。
    private Map<String, SoftReference<Bitmap>> imageCache = new HashMap<String, SoftReference<Bitmap>>();
     
    //再来定义一个方法，保存Bitmap的软引用到HashMap。
    public void addBitmapToCache(String path) {
        // 强引用的Bitmap对象
        Bitmap bitmap = BitmapFactory.decodeFile(path);
        // 软引用的Bitmap对象
        SoftReference<Bitmap> softBitmap = new SoftReference<Bitmap>(bitmap);
        // 添加该对象到Map中使其缓存
        imageCache.put(path, softBitmap);
    }
 
 
    //获取的时候，可以通过SoftReference的get()方法得到Bitmap对象。
    public Bitmap getBitmapByPath(String path) {
        // 从缓存中取软引用的Bitmap对象
        SoftReference<Bitmap> softBitmap = imageCache.get(path);
        // 判断是否存在软引用
        if (softBitmap == null) {
            return null;
        }
        // 取出Bitmap对象，如果由于内存不足Bitmap被回收，将取得空
        Bitmap bitmap = softBitmap.get();
        return bitmap;
    }
    
    
<hr />

参考及学习链接：

http://www.cnblogs.com/shudonghe/p/3457990.html
Java 垃圾回收机制技术详解
http://blog.csdn.net/lu1005287365/article/details/52475957
Java 垃圾回收机制 (GC) 详解、意义、建议
http://blog.csdn.net/ithomer/article/details/6252552
Java 内存模型及GC原理
http://www.codeceo.com/article/java-gc-1.html
Java GC专家系列1：理解Java垃圾回收
http://www.codeceo.com/article/java-gc-2.html
Java GC专家系列2：Java 垃圾回收的监控
http://www.codeceo.com/article/java-gc-learn.html
Java GC 专家系列3：GC调优实践

[Android性能优化-内存泄漏的8个Case](http://mp.weixin.qq.com/s?__biz=MzAxMTI4MTkwNQ==&mid=2650822597&idx=1&sn=462b116f97623f239ecf667d3bdef446&chksm=80b7835bb7c00a4d4cbc9f7e19829d9a99f3cf58c1bc43dace16ffec58c98668927c9fa8dcda&mpshare=1&scene=1&srcid=04209wySV2j2LglhLKccZpHV#rd)

http://blog.csdn.net/huoyin/article/details/5891998
Java中三个引用类SoftReference 、 WeakReference 和 PhantomReference的区别