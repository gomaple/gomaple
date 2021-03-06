# 第一章

## 进程和线程

进程是可执行程序在运行期，操作系统虚拟化对象。它包括了程序运行期间所使用的内存资源、指令计数器寄存器、文件句柄等。线程是进程中创建的执行单元。一个进程可以创建多个线程，每个线程拥有独立的栈资源、寄存器、指令计数器，共享进程的内存资源。

现代操作系统都是以线程作为调度执行的基本单元。

## 并发编程的优势

1.  可以充分利用多核cpu资源；
2.  并发框架，可以简化编程模型。比如servlet和RMI规范，开发人员只需要用开发单线程的视角编写程序，无需关注多线程的细节。

## 多线程开发的风险：

1.  安全性。缺乏足够同步的多线程程序，会出现不可预测的行为和结果。
2.  活跃性。可能出现死锁、活锁或者是饥饿的情况
3.  性能。多线程一方面可以提升程序的性能，另一方面也会导致应用的性能降低。
4.  安全性蔓延。所使用的框架本身是多线程性质的。框架通过回调的形式，将并发安全的影响范围传递到了应用代码中。

# 第二章 线程安全性

**线程安全性的来源是：共享可变变量。对抗这种风险的手段：**
>1.  将共享变为不共享 : 依赖面向对象的良好封装和线程封闭操作。
>2.  将可变变为不可变 ：不可变技巧
>3.  通过同步手段控制对变量的访问操作：明晰的不变性规则

## 竞态条件

本质：针对共享数据的非原子操作，在多线程并发的场景下，会出现数据一致性被破坏的情况。

分为两种：

-   共享数据的状态只有一个，代码对该共享状态执行非原子操作，例如：先检查再执行的操作。由于动作在硬件指令分为了两个阶段，就会产生多线程执行时的交叉。
    
-   更一般地，共享数据的状态有多个，当某个线程在操作其中一个状态的时候，另外一个线程可能同时在变更另外一个状态，此时共享对象的状态的一致性被破坏了。
    

之所以叫做竟态条件，**竟态**就是指状态处于多个线程竞争读取的情况，**条件**是指被竞争的状态是作为操作的条件参与线程动作的。

现象：程序的正确性取决于线程并发动作的顺序。

## 数据竞争

数据竞争是两个线程直接对共享变量执行操作。例如可能同时读写，同时写写。这里一定要有写动作的存在。

原子性的诉求：在操作共享对象的状态时，隐含着数据一致性的要求。

可见性的诉求：读取操作必须能够看到最新的值。

## 64位数据读写问题
JVM要求变量的读写操作是原子性的，但是针对64位数据的读操作和写操作可以是由两个步骤完成，尤其是在32位的系统中，由于总线长度就是32位的，必须操作两次才可以完成64位的读写动作。所以为了保证Long和double类型数值的安全性，必须使用volatile或者同步操作。


## 内置锁

每一个java对象都持有一个内置锁，用来简化锁对象的创建。每一个需要同步的方法或者代码块需要由一个锁对象来配合完成同步语义。

### synchronized
该关键字作用在类的非静态方法身上，则对应的锁对象是类的实例；作用在静态方法上，对应的锁对象是class对象。用于代码块时，需要明确指定锁对象

### 用锁来保护状态
* 一组相关联必须原子操作的状态，必须始终用同一个锁来进行同步控制
* JDK中的锁是可重入的
* 持有锁的时间过长，就很可能会导致活跃性的问题和性能问题

## 同步的作用
同步操作不仅仅是为了保证代码的原子性，同时也是为了保证共享状态的可见性。在没有同步的情况下，编译器、指令执行器、运行期写回动作都会对指令操作进行重排序，从而导致可见性问题，这其中内存重排序是最主要的原因。

## volatile关键字
**volatile变量对其他变量的可见性的影响，比volatile变量本身还要重要。**

volatile变量是一种轻量级的同步操作，它没有锁，与synchronized相比，volatile可以实现可见性的效果，但是没有原子性的效果。也就是说我们可以通过volatile关键字实现与同步代码块一样的操控内存可见性的能力。当然这种能力，也是有JMM所做出的要求。

对一个volatile变量的写操作，在可见性的效果上等价于退出同步代码块动作，在写动作之前的所有变量必须全部刷新回主存。但是这种高级的稍微隐含的可见性效果不易理解，有风险。

volatile合理的使用方式：
1. 确保voatile变量自身状态的可见性
2. 确保volatile变量所引用对象的状态的可见性
3. volatile变量不与其他变量一起形成不变形条件
4. volatile变量的写入操作是原子性的，即更新变量的值是直接写入的，不依赖原值。
5. 最常用的场景，使用volatile变量来表示某个状态，根据状态做出不同的行为。


# 第三章 对象的共享

## 对象发布

将一个对象共享就是将对象发布。对象共享的形式有多种：类的静态成员属性、非私有属性、   通过api暴露出的对象引用，或者将一个对象加入到某个已经共享出来的集合当中。还有一种发布对象状态的情况，是内部类实例，最常见的就是监听器listener模式。

只要对象通过上述情况发布后后，无论是否有多线程访问的情况，都必须要考虑此种风险的存在，在设计上就要考虑线程安全。

一个显而易见的对象发布形式就是构造函数。要保证的是在构造逻辑完成之前，不要将对象的引用提前发布出去。有两种常见提前发布了对象引用的情况：
  * 在构造函数中启动了异步线程，并将当前的this引用发布了出去
  * 工厂方法中的指令重排序。如双重检查的单例模式

## 线程封闭——限制共享

线程封闭就是将共享变量限制在线程内部，也就是抑制了变量的共享特性。

###  栈封闭
局部变量天然是线程封闭的。

### AD-HOC
ad-hoc线程封闭是指对象的线程封闭特性由应用程序自己来保证。常见的如各种池化技术，每一个池化的共享对象，只会分配一个线程使用。

### 单线程写volatiel
线程封闭的一个宽松的情景：只有一个线程写，其他都是读线程，就是将写操作封闭在一个线程中，这种情况下可以通过volatile关键字保证变量的可见性即可。

### ThreadLocal
__java语言中提供的线程封闭技术就是ThreadLocal类。

当在某个频繁执行的操作中，为了避免不停的重复获取临时对象，可以通过threadlocal技术将临时对象保存起来，线程内随时可以获得——最简单的可以通过将threadlocal对象设置为一个静态成员。

## 不变性——限制可变

### 什么是不可变对象
保证线程安全的另外一个手段就是依赖变量的不变性。不变性天然就是线程安全的。
在jVM规范中和JMM模型中没有明确的正式定义什么是不可变对象。所以只能从效果上对不可变对象做出简单的定义：创建后状态不再变化的对象就是不可变对象。


### final和不可变对象
一般在实践中，不可变对象设计为对象的所有状态都保存在final域中。这是因为借由final语法的特性，可以达成不可变对象的安全构造发布的效果。但是对象所有的数据域都是final域也无法保证对象就是不可变的对象——因为对于引用类型来说，final只是对引用的数值做了不可变的保证，但是引用具体指向的对象内部状态依然是不受约束的。

>我们回归到计算机存储的本质来看，所谓对象，其实就是各种基本类型的组合和再组合而已。那么对于对象的成员的访问，也还是对基本类型的一个访问。这个访问的过程，其实可以看成是两个阶段：
      1. 通过引用确定对象的起始地址
      2. 通过所访问的变量名，确定相对地址并读取并操作。
 所以这其中存在这两个阶段的可见性问题：首先引用的可见性。其次引用对象的变量的可见性。在缺少同步的情况下，某个对象的引用对其他线程保证了可见性是无法保证对象的成员的可见性的。


另一方面即使是可变对象，也尽可能的将对象的状态声明为final，这样可以减少对象的状态的维度，降低复杂度。

>正如除非需要更高的可见性，否则所有的字段都应该声明为private；除非需要更高的可变性，否则所有的字段都应该声明为final

## 安全发布

安全发布对象，要保证对象的引用以及对象的状态必须对其他线程同时可见，包括如下手段：
* 在静态初始化块中实例化一个对象 
* 将对象的引用保存在votatile域，或者使用AtomicReferance保存对象的引用
* 将对象的引用保存在final域中
* 对象的引用保存在由锁保护的域中，如同步容器。

# 第四章 组合的对象

