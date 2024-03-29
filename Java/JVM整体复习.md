# 内存区域

- java是一门面向对象的语言，所以内存区域首先解决的也是类和对象的存储区域。
- 类数据存储的位置是方法区，存储的是方法以及静态的变量。
- 对象数据存储的位置是堆，存储的是对象的变量。
- 然后就是方法调用期间用到的存储位置虚拟机栈。
- 类似的还有native方法用的栈，本地方法栈。
- 最后是切换线程以及代码逻辑跳转时记录字节码执行行号的程序计数器。
- 内存区域分线程私有和线程公有，想一下就知道，方法区和堆是公有，其余是私有。



# HotSpot对象创建过程

## 创建步骤

1. 类加载检查

2. 分配内存

   - 给新对象分配堆内存，内存规整则使用指针碰撞的方式分配，调整指针位置；不规整使用空闲列表的方式分配。
   - 线程安全问题：如果分配内存时每次都同步，所有线程要停下手里的活，效率很低，用本地线程缓冲（TLAB），即每个线程预先分配一块内存，用完了再同步重新分新的TLAB。

3. 初始化

   所有对象的变量初始值赋为0.

4. 设置

   设置这个对象的hash值，这个对象是哪个类的实例，如何找到那个类。存到对象头中。

5. 执行init方法

   执行构造方法，以及变量的初始赋值。

## 内存布局

- 分为三块：对象头、示例数据、对齐填充。
- 对象头就是前面设置阶段存的那些数据，有三块：
  - hash值、GC分代年龄、线程锁等
  - 此对象是哪个类的实例，指向类数据的指针。
  - 如果对象是数组，记录数组的长度。
- 实例数据：就是存储的变量，包括父类继承下来的变量。
- 对齐填充，占位符。



# 类文件结构

- 类文件的基础单位是字节，它的基础结构是无符号数和表。
- 无符号数分为四中：u1,u2,u4,u8。分别表示1个字节、2个字节、4个字节和8个字节大小的无符号数。
- 表则是由无符号数，或者其他的表组成的结构。表的命名都是以info结尾。
- 类文件结构：魔数 - 版本号 - 常量池 - 访问标志 - 类/父类/接口 - 字段描述集合 - 方法描述集合 - 属性描述集合

#### String最大长度问题

常量池中的各个常量分各种类型，存为不同的表，比如int类型的存为CONSTANT_Integer_info，String类型的为CONSTANT_utf8_info，每个表中存储了一些数据，CONSTANT_utf8_info的为：

```
table CONSTANT_utf8_info {

    u1  tag; //代表是哪个类型的表，1表示是 CONSTANT_Utf8_info 类型表

    u2  length; //长度，就是这个表存的字符串有多长

    u1[] bytes; //具体的字符串数据

}
```

String用的CONSTANT_utf8_info表中用于存储字符串长度的 lengh 是一个u2的类型，也就是说他length这个值最大的值就是2个字节的无符号数最大值，也就是2<sup>16</sup> - 1，65535。所以String常量的最大值是65535。



# 类加载机制

## 类加载的时机

大体有三种情况：

##### 1. 创建对象的时候发现需要的类没有初始化

具体有两种情况：

1. new一个对象发现这个类还没初始化
2. 反射创建一个对象发现这个类还没初始化

##### 2. 用到属于类的东西的时候发现类还没初始化

就是静态的，存在方法区的那些，比如用到一个类的静态方法、静态变量时

##### 3. 初始化一个类时候发现父类还没初始化



## 类加载的过程

- 加载：把类从Class文件中加载打内存里
- 验证：验证Class文件的安全性、是否有错、是否符合虚拟机规范
- 准备：和对象创建阶段的初始化一样，就是所有的值赋为0
- 解析：讲常量池内的符号引用替换为直接引用，如果这个过程中发现使用到的类还没加载，会先去加载所需要的类。
- 初始化：执行`<clinit>`，这个过程中如果发现父类还没有初始化，会先去加载父类并初始化父类，然后再回来初始化自己。



## 类加载器

### java的三个默认的ClassLoader

- java默认有三个ClassLoader，分别是：BootstrapClassLoader，ExtClassLoader，AppClassLoader，这三个的顺序前一个是后一个的parent。
- ClassLoader的职责是加载指定path的类，这三个类按顺序分别负责<JAVA_HOME>/lib 、<JAVA_HOME>/lib/ext 以及classPath指定的目录的类。



### 双亲委派机制

双亲委派机制就是和Android的touch event的分发顺序刚好相反的一套机制，所有类加载器在加载的时候，先去找自己的parent去加载这个类，加载不了才会自己加载。

#### 双亲委派机制解决的问题

双亲委派机制解决了一个安全性的问题，就是避免java核心类被偷梁换柱。因为ClassLoader在创建的时候都会定义自己要加载的path，而这就是jvm在找对应的类对应的ClassLoader的重要指标。也就是说 AClassLoader说我去加载A路径下的类，那么在加载A路径的时候jvm就可能会找 AClassLoader去加载。

那么问题就来了，如果没有双亲委派机制，就有可能发生如下情况，攻击者写了一个和String相同全类名的类，以及一个自定义的ClassLoader，这个ClassLoader就声称自己要加载的String类所在的path，但是里面加载的是自己那个恶意的String类。

此时在使用一个正确的String的时候，jvm发现这个自定义的ClassLoader说了他要加载这个path，于是就可能会找这个自定义的ClassLoader去加载String类，从而被他偷梁换柱成功找到错误的String类。

但是有了双亲委派机制，在没有找到正确的ClassLoader时，普通的ClassLoader都会一直向上找到parent的ClassLoader，从最顶层的ClassLoader开始看是否负责加载这个类，这样核心类由于就会使用默认的正确的ClassLoader。

#### 自定义ClassLoader

自定义ClassLoader可以继承ClassLoader类并重写findClass()、loadClass()等方法，findClass()可以实现自己的一些逻辑，比如加载硬盘上其他位置的类，loadClass()可以在自定义ClassLoader使用时，不遵循双亲委派机制，直接自己自己去加载类。这部分例子详见文章：https://kaiwu.lagou.com/course/courseInfo.htm?courseId=67#/detail/pc?id=1859



### Android中的ClassLoader

Android中的ClassLoader和Java相比的一个不同就是，Android中的class文件会被打包成dex文件，Android中的ClassLoader也只会加载dex文件，不会直接加载class文件。

#### BaseDexClassLoader

实现加载 .dex 文件的基类。后面的两个ClassLoader都继承自 BaseDexClassLoader 。

#### PathClassLoader

只能加载系统apk以及已经被安装到手机中的 apk 里的 dex 文件。

#### DexClassLoader

可以加载任意来源的 dex 或 apk文件。也是热修复的基础。



### Android中的以ClassLoader为核心原理的热修复方案

发现bug后，新建java项目打一个jar包，其中包含一个相同全类名的类，这个类修复了bug，并且待修复的类的path，然后把jar包通过dex工具转换成dex包。这个包就是补丁包。

应用启动的时候去后端拉补丁包，发现有补丁，下载补丁包到本地sd卡，下载sdk卡后通过DexClassLoader加载补丁包的dex文件的path。等到下次加载的时候待修复的类的时候，就会加载指定的补丁包中的类了。

基本上是这个思路，但是细节还没有理解的很好，可能需要写个demo之类的。



# GC算法

## 什么时候回收垃圾

不同的虚拟机有不同的实现，但是基本就是两种情况：

1. 新对象分配内存失败时
2. 手动调用System.gc()时

具体到新生代和老年代的回收时机的分析在后面的回收算法的部分。



## 定位哪些是垃圾

- 可达性分析
- GCRoot
  - 正在运行的方法：虚拟机栈中引用的对象
  - 静态的东西：方法区中的静态引用指向的对象
  - 正在运行的线程对象
  - Native方法中JNI引用的对象



## 如何回收这些垃圾

### 标记清除算法

- 标记所有GCRoot及其间接引用的对象
- 清除这些对象

### 标记压缩算法

- 在标记清除算法的基础上，压缩存活对象，尽量减少内存碎片。
- 压缩阶段较耗时，效率较低。

### 复制算法

把已有内存氛围两块，每次只使用其中的一块，在垃圾回收时：

1. 标记所有GCRoot及其间接引用的对象。
2. 把标记的对象copy到另一区域，而且直接顺序往里面存就行了，内存自然就是规整的。
3. 存完所有存活对象，声明这一区域为使用的内存块。

用内存减少一般的代码，换来了不用压缩和整理内存，速度更快，属于用空间换了时间。

### JVM分代回收策略

##### 核心思想

1. 有些对象频繁创建销毁，需要频繁GC，这类的对象的回收要求算法效率高。这些对象就放在新生代里。
2. 新生代使用效率高的复制算法，用空间换时间。新生代由于不停的创建和销毁对象，触发GC会相对频繁一些，由于使用的算法效率高，所以整体不会太慢。
3. 有些对象基本长期驻守在内存中，较为稳定，大多数时候可以不用尝试回收这些对象，可以等内存不够用了再尝试回收。这些对象放在老年代里。
4. 老年代使用效率低的标记压缩算法。
5. 新生代的GC称为minorGC，老年代的GC称为MajorGC。都进行回收称为FullGC。不同的虚拟机有不同的实现，有的虚拟机MajorGC=FullGC。

##### 具体细节

1. 新生代有三个去，eden区，from区和to区，新来的对象都进入eden区，然后from区和to区轮流一个作为当前区，一个作为空闲区。
2. 当eden区内存不足或System.gc()的时候，触发MinorGC，把eden区以及from/to区中当前区存活的对象copy到空闲区，然后把当前区内存释放，空闲区成为当前区。也就是说经过一次MinorGC而继续存活的对象会在from区和to区来回倒。
3. 当新生代的对象经过一定次数（默认是15此）的GC以后，进入老年代。
4. 当老年代内存不足时，触发majorGC或fullGC。

