# 基本概念

## 程序

可执行文件

## 进程

操作系统进行资源分配的基本单位，进程是静态的概念(静态资源:CPU,内存,外部设备等)

## 线程

线程是调度执行的基本单位，多个线程共享同一个进程里面的所有的资源，线程是动态的概念。一个程序在运行的时候会产生不同的执行路径(同时在运行的多条路径)就叫多线程。双击可执行文件后会进入内存，变成一个进程，进程是如何执行的呢？(程序是如何执行的)真正开始执行是以线程为单位来执行，操作系统会找到主线程，然后将主线程的代码一条一条交给CPU来执行。如果主线程运行中开启了其他线程，再来线程之间的来回切换。

## 线程切换(Context Switch)

CPU的组成包括如下几个部分：

1. ALU:计算单元
2. Registers:寄存器组,用于存储数据
3. PC:程序计数器（Program Counter）也是一种寄存器，保存了将要取出的下一条指令的内存地址，指令取出后，就会更新该寄存器指向下一条指令
4. Cache:用来缓存内存中的数据，避免直接从内存中获取，提升CPU的运算周期效率。

程序执行过程：指令放PC，数据放Registers，然后ALU进行计算；线程切换则是将PC和Registers中正在执行的线程的数据保存到Cache中，然后将另一个线程的数据load到CPU的PC和Registers中来。CPU只负责计算，操作系统负责具体指令和数据是属于哪个线程的。

设置线程数量的一般计算公式：N_threads = N_cpu * U_cpu * (1 + W/C)

1. N_cpu是处理器的核的数目，可以通过Runtime.getRuntime().availableProcessors()得到
2. U_cpu是期望的CPU利用率(该值应该介于0到1之间)
3. W/C是等待时间与计算时间的比率

## 纤程/协程

绿色线程，用户管理的(而不是OS管理的)线程

## 并发(Concurrent)

当有多个线程在操作时,如果系统只有一个CPU,则它根本不可能真正同时进行一个以上的线程,它只能把CPU运行时间划分成若干个时间段,再将时间段分配给各个线程执行,在一个时间段的线程代码运行时,其它线程处于挂起状态.这种方式我们称之为并发(Concurrent).

## 并行(Parallel)

当系统有一个以上CPU时,则线程的操作有可能非并发.当一个CPU执行一个线程时,另一个CPU可以执行另一个线程,两个线程互不抢占CPU资源,可以同时进行,这种方式我们称之为并行(Parallel)

## 调用状态

阻塞（Blocking）和非阻塞（Non-Blocking）

## 通信机制

同步（synchronous）和异步（asynchronous）

## 锁

monitor

## 临界区

critical section 每个线程中访问临界资源(临界资源是一次仅允许一个线程使用的共享资源)的那段代码称为临界区。每次只准许一个线程进入临界区，进入后不允许其他线程进入，即：临界区是被synchronized锁定的代码区域，如果临界区执行时间长、语句多，叫做锁的粒度比较粗

# CPU缓存

## 硬件内存架构

计算机在执行程序的时候，每条指令都是在CPU中执行的，而执行的时候，又免不了要和数据打交道。而计算机上面的数据，是存放在主存当中的，也就是计算机的物理内存。以多核CPU为例，每个CPU核都包含一组「CPU寄存器」，这些寄存器本质上是在CPU内存中。CPU在这些寄存器上执行操作的速度要比在主内存(RAM)中执行的速度快得多。因为CPU速率高，内存速率慢，为了让存储体系可以跟上CPU的速度，所以中间又加上Cache层，就是我们说的「CPU高速缓存」。

## CPU多级缓存

由于CPU的运算速度远远超越了1级缓存的数据I\O能力，CPU厂商又引入了多级的缓存结构。通常L1、L2是每个CPU核有一个，L3是多个核共用一个。

1. 各种寄存器，用来存储本地变量和函数参数，访问一次需要1cycle，耗时小于1ns；
2. L1 Cache，一级缓存，本地core的缓存，分成32K的数据缓存L1d和32k指令缓存L1i，访问L1需要3cycles，耗时大约1ns；
3. L2 Cache，二级缓存，本地core的缓存，被设计为L1缓存与共享的L3缓存之间的缓冲，大小为256K，访问L2需要12cycles，耗时大约3ns；
4. L3 Cache，三级缓存，在同插槽的所有core共享L3缓存，分为多个2M的段，访问L3需要38cycles，耗时大约12ns；

大致可以得出结论，缓存层级越接近于CPUcore，容量越小，速度越快，同时其造价也更贵。所以为了支撑更多的热点数据，同时追求最高的性价比，多级缓存架构应运而生。

## CacheLine

Cache又是由很多个「缓存行」(CacheLine)组成的。CacheLine是Cache和RAM交换数据的最小单位。Cache存储数据是固定大小为单位的，称为一个CacheEntry，这个单位称为CacheLine或CacheBlock。给定Cache容量大小和cache-line-size的情况下，它能存储的条目个数(number-of-cache-entries)就是固定的。因为Cache是固定大小的，所以它从主内存获取数据也是固定大小。对于X86来讲，是64Bytes。对于ARM来讲，较旧的架构的CacheLine是32Bytes，但一次内存访存只访问一半的数据也不太合适，所以它经常是一次填两个CacheLine，叫做DoubleFill。

例如：遍历一个长度为16的long数组data[16]，原始数据自然存在于主内存中，访问过程描述如下

1. 访问data[0]，CPUcore尝试访问CPUCache，未命中。
2. 尝试访问主内存，操作系统一次访问的单位是一个CacheLine的大小—64字节，这意味着：既从主内存中获取到了data[0]的值，同时将data[0]~data[7]加入到了CPUCache之中。
3. 访问data[1]~data[7]，CPUcore尝试访问CPUCache，命中直接返回。
4. 访问data[8]，CPUcore尝试访问CPUCache，未命中。
5. 尝试访问主内存。重复步骤2

实验验证：

```java
public class MainClass {

	static long[][] arr;

	/*
	 * 使用了横向遍历和纵向遍历两种顺序对这个二位数组进行遍历，遍历总次数相同，只不过循环的方向不同，观察他们的耗时。
	 * 多次运行后可以发现：横向遍历的耗时大约为15ms，纵向遍历的耗时大约为40ms，前者比后者快了1倍有余。
	 * 上述现象出现的原因就是CPU-Cache&Cache-Line。因为横向遍历arr= new long[1024 * 1024][8]
	 * 每次内层寻访中的数据都存储在连续内存地址中，所以一次访问会将剩下7个数据一起加入CPUCache中，
	 * 而纵向访问内存循环的数据不是在连续地址中所以每一次都要从内存中取。
	 */
	public static void main(String[] args) throws InterruptedException {
		arr = new long[1024 * 1024][8];
		long sum = 0;
		// 横向遍历
		long marked = System.currentTimeMillis();
		for (int i = 0; i < 1024 * 1024; i += 1) {
			for (int j = 0; j < 8; j++) {
				sum += arr[i][j];
			}
		}
		System.out.println("Loop times:" + (System.currentTimeMillis() - marked) + "ms");
		sum = 0;
		marked = System.currentTimeMillis();
		// 纵向遍历
		for (int i = 0; i < 8; i += 1) {
			for (int j = 0; j < 1024 * 1024; j++) {
				sum += arr[j][i];
			}
		}
		System.out.println("Loop times:" + (System.currentTimeMillis() - marked) + "ms");
	}
}
```

## 伪共享

伪共享指的是多个线程同时读写同一个缓存行的不同变量时导致的CPU缓存失效。尽管这些变量之间没有任何关系，但由于在主内存中邻近，存在于同一个缓存行之中，它们的相互覆盖会导致频繁的缓存未命中，需要重新刷缓存，引发性能下降。伪共享问题的解决方法便是字节填充，只需要保证不同线程的变量存在于不同的CacheLine即可，使用多余的字节来填充可以做点这一点，这样就不会出现伪共享问题。

实验验证：

```java
package org.duo.cacheline;

import java.util.concurrent.CountDownLatch;

public class CacheLinePadding {
    public static long count = 10_0000_0000L;

    /**
     * 1) 不加volatile情况，使不使用缓存对齐，由于StoreBuffer和Invalidate Queues的存在对性能影响不大
     * 2) 加volatile
     * 使用缓存对齐:由于两个对象处于两个不同的缓存行内，所以使用缓存锁，并且每个线程锁定不同的缓存行不存在锁竞争所以性能高
     * 不使用缓存对齐:由于两个对象处于同一缓存行内，所以使用总线锁，并且存在锁竞争相当于串行所以性能低
     * <p>
     * 加不加volatile缓存一致性协议都存在。只不过加了volatile之后由于缓存行被锁定，所以另一个线程要等待锁的释放，并且通过缓存一致性协议读取最新的值
     */
    private static class T {
        private long p1, p2, p3, p4, p5, p6, p7;
        public volatile long x = 0L;
        //public long x = 0L;
        private long p9, p10, p11, p12, p13, p14, p15;
    }

    public static T[] arr = new T[2];

    static {
        arr[0] = new T();
        arr[1] = new T();
    }

    public static void main(String[] args) throws InterruptedException {
        CountDownLatch latch = new CountDownLatch(2);
        Thread t1 = new Thread(new Runnable() {
            @Override
            public void run() {
                for (long i = 0; i < count; i++) {
                    arr[0].x = i;
                }
                latch.countDown();
            }
        });
        Thread t2 = new Thread(new Runnable() {
            @Override
            public void run() {
                for (long i = 0; i < count; i++) {
                    arr[1].x = i;
                }
                latch.countDown();
            }
        });
        final long start = System.nanoTime();
        t1.start();
        t2.start();
        latch.await();
        System.out.println((System.nanoTime() - start) / 100_0000);

    }
}
```

## 缓存带来的问题

缓存的加入是为了解决CPU运算能力和内存读写能力的不匹配问题，简单来说就是为了提升资源利用率。那么在多CPU（一个CPU对应一个或者多个核心）或者多核心下，每个核心都会有一个一级缓存或者二级缓存，也就是说一二级缓存是核心独占的（类似JMM模型，线程的工作内存是线程独占的，主内存是共享的）而三级缓存和主内存是共享的，这样就将导致CPU缓存一致性问题。多个cache与内存之间的数据同步该怎么做？主要有两类：总线锁和缓存一致性协议

## 总线锁

在早期的CPU当中，是通过在总线上加LOCK#锁的形式来解决缓存不一致的问题。因为 CPU 和其他部件进行通信都是通过总线来进行的，如果对总线加锁的话，也就是说阻塞了其他 CPU 对其他部件访问（如内存），从而使得只能有一个 CPU 能使用内存。但是这种方式会有一个问题，由于在锁住总线期间，其他 CPU无法访问内存，导致效率低下。所以就出现了缓存一致性协议。

## 缓存一致性协议(MESI)

缓存一致性协议可以分为两类：“窥探（snooping）”协议和“基于目录的（directory-based）”协议，MESI协议属于一种"窥探协议"。窥探协议的基本思想所有cache与内存，cache与cache（是的，cache之间也会有数据传输）之间的传输都发生在一条共享的总线上，而所有的cpu都能看到这条总线，同一个指令周期中，只有一个cache可以读写内存，所有的内存访问都要经过仲裁（arbitrate）。窥探协议的思想是，cahce不但与内存通信时和总线打交道，而且它会不停地窥探总线上发生的数据交换，跟踪其他cache在做什么。所以当一个cache代表它所属的cpu去读写内存时，其它cpu都会得到通知，它们以此来使自己的cache保持同步。

并非所有情况都会使用缓存一致性协议，如被操作的数据不能被缓存在CPU内部或操作数据跨越多个缓存行(状态无法标识)，则处理器仍然会调用总线锁定

每个 Cache line 有两个标志位：dirty 和 valid 标志来标记缓存行的有4个状态，MESI是指4种状态的首字母。协议中最重要的内容有两部分：cache line的状态以及消息通知机制：

### Cache Line的状态

- M(Modified)：描述：该CacheLine有效，数据被修改了，和内存中的数据不一致，数据只存在于本Cache中。CacheLine只有处于Exclusive状态才能被修改。此外，已修改CacheLine如果被丢弃或标记为Invalid，那么先要把它的内容回写到内存中。监听任务：必须时刻监听所有试图读该缓存行相对于主存的操作，这种操作必须在CPU将该缓存行写回主存并将状态变成S（共享）状态之前被延迟执行。
- E(Exclusive)：描述：该CacheLine有效，数据和内存中的数据一致，数据只存在于本Cache中。独占该内存地址，其他处理器的CacheLine不能同时持有它，如果其他处理器原本也持有同一CacheLine，那么它会马上变成"Invalid"状态。监听任务：缓存行也必须监听其它缓存读主存中该缓存行的操作，一旦有这种操作，该缓存行需要变成S（共享）状态。
- S(Shared)：描述：该Cache line有效，数据和内存中的数据一致，数据存在于很多Cache中。处于该状态下的cache line只能被cpu读取，不能写入，因为此时还没有独占。监听任务：缓存行也必须监听其它缓存使该缓存行无效或者独享该缓存行的请求，并将该缓存行变成无效（Invalid）。
- I(Invalid)：描述：该Cache line无效。监听任务：无

对于M和E状态而言总是精确的，他们在和该缓存行的真正状态是一致的。而S状态可能是非一致的，如果一个缓存将处于S状态的缓存行作废了，而另一个缓存实际上可能已经独享了该缓存行，但是该缓存却不会将该缓存行升迁为E状态，这是因为其它缓存不会广播他们作废掉该缓存行的通知，同样由于缓存并没有保存该缓存行的copy的数量，因此（即使有这种通知）也没有办法确定自己是否已经独享了该缓存行。

cpu有读取数据的动作，有独占的动作，有独占后更新数据的动作，有更新数据之后回写内存的动作，根据”窥探协议“的规范，每个动作都需要通知到其他cpu，于是有以下的消息机制：

### 消息通知机制

- Read，cpu发起读取数据请求，请求中包含需要读取的数据地址。
- Read Response，作为Read消息的响应，该消息可能是内存响应的，也可能是某cpu响应的(比如该地址在某cpu cache Line中为Modified状态，则该cpu必须返回该地址的最新数据)。
- Invalidate，cpu发起”我要独占一个cache line，其他cpu请失效对应的cache line“的消息，消息中包含了内存地址，所有的其它cpu需要将对应cache line置为Invalid状态。
- Invalidate ACK，收到Invalidate消息的cpu在将对应cache line置为Invalid后，返回Invalid ACK。
- Read Invalidate，相当于Read消息+Invalidate消息，即取得数据并且独占它，将收到一个Read Response和所有其它cpu的Invalidate ACK。
- Write back，写回消息，即将状态为Modified的cache line写回到内存，通常在该行将被替换时使用。现代cpu cache基本都采用”写回(Write Back)”而非”直写(Write Through)”的方式。

结合cache line状态以及消息机制，来看看cpu之间是如何协作的。假设有个四核cpu系统，每个cpu只有一个cache line，每个cache line大小为1个字节，内存地址空间一共两个字节的数据，地址分别为0x0和0x8，有如下操作序列:

|                        | cup0      | cup1       | cup2        | cup3      | main   | memory |
| ---------------------- | --------- | ---------- | ----------- | --------- | ------ | ------ |
|                        | cacheline | cacheline  | cacheline   | cacheline | 0x0    | 0x8    |
| cpu0:read from 0x0     | 0/shared  |            |             |           | latest | latest |
| cpu3:read from 0x0     | 0/shared  |            |             | 0/shared  | latest | latest |
| cpu0:read from 0x8     | 8/shared  |            |             | 0/shared  | latest | latest |
| cpu2:exclusive 0x0     | 8/shared  |            | 0/exclusive |           | latest | latest |
| cpu2:modified 0x0      | 8/shared  |            | 0/modified  |           | expire | latest |
| cpu1:atomic inc in 0x0 | 8/shared  | 0/modified |             |           | expire | latest |
| cpu1:read from 0x8     | 8/shared  | 8/shared   |             |           | latest | latest |

1. 初始状态，4个cpu的cache line都为Invalid状态（黑色表示Invalid）。
2. cpu0发送Read消息，加载0x0的数据，数据从内存返回，cache line状态变为Shared。
3. cpu3发送Read消息，加载0x0的数据，数据从内存返回，cache line状态变为Shared。
4. cpu0发送Read消息，加载0x8的数据，导致cache line被替换，由于之前状态为Shared，即与内存中数据一致，可直接覆盖，而无需回写。
5. cpu2发送Read Invalidate消息，从内存返回最新数据，cpu3返回Invalidate ACK，并将状态变为Invalid，cpu2获得独占权，状态变为Exclusive。
6. cpu2修改cache line中的数据，cache line状态为Modified，同时内存中0x0的数据过期。
7. cpu1 对地址0x0的数据执行原子(atomic)递增操作，发出Read Invalidate消息，cpu2将返回Read Response(而不是内存)，包含最新数据，并返回Invalidate ACK，同时cache line状态变为Invalid。最后cpu1获得独占权，cache line状态变为Modified，数据为递增后的数据，而内存中的数据仍然为过期状态。
8. cpu1 加载0x8的数据，此时cache line将被替换，由于之前状态为Modified，因此需要先执行写回操作，此时内存中0x0的数据得以更新。

这就是缓存一致性协议，一个状态机，仅此而已。因为该协议的存在，每个cpu就可以放心操作属于自己的cache，而不需要担心本地cache中的数据会不会已经被其他cpu修改。但在缓存一致性协议的框架下工作性能不高，但这并不是协议的问题，协议本身逻辑很严谨。性能差在哪里？假如某数据存在于其他cpu的cache中，那自己每次需要修改数据时，都需要发送ReadInvalidate消息，除了等待最新数据的返回，还需要等待其他cpu的InvalidateACK才能继续执行其他指令，这是一种同步行为，于是便有了cpu的优化：处理器指令重排(指令还未结束便去执行其它指令的行为称之为，指令重排)

### 处理器指令重排

指令重排的实现:硬件层面的优化——Store Buffer, Invalid Queue，以及软件层面的优化——内存屏障指令。

#### Store Buffer(存储缓存)

StoreBuffer即存储缓存。位于内核和缓存之间。当处理器需要处理将计算结果写入在缓存中处于shared状态的数据时，需要通知其他内核将该缓存置为 Invalid（无效），引入StoreBuffer后将不再需要处理器去等待其他内核的响应结果，只需要把修改的数据写到StoreBuffer，通知其他内核，然后当前内核即可去执行其它指令。当收到其他内核的响应结果后，再把StoreBuffer中的数据写回缓存，并修改状态为M。（很类似分布式中，数据一致性保障的异步确认）

StoreBuffer带来的问题，例如有如下代码：

```java
//初始状态下，假设a，b值都为0，并且a存在cpu1的cache line中(Shared状态)，代码在cpu0中执行，
a = 1;
b = a + 1;
assert(b == 2);
```


可能出现如下操作序列:

1. cpu0 要写入a，将a=1写入store buffer，并发出Read Invalidate消息，继续其他指令。
2. cpu1 收到Read Invalidate，返回Read Response(包含a=0的cache line)和Invalidate ACK，cpu0 收到Read Response，更新cache line(a=0)。
3. cpu0 开始执行b=a+1，此时cache line中还没有加载b，于是发出Read Invalidate消息，从内存加载b=0，同时cache line中已有a=0，于是得到b=1，状态为Modified状态。
4. cpu0 得到 b=1，断言失败。
5. cpu0 将store buffer中的a=1推送到cache line，然而为时已晚。

造成这个问题的根源在于对同一个cpu存在对a的两份拷贝，一份在cache，一份在store buffer，而cpu计算b=a+1时，a和b的值都来自cache。仿佛代码的执行顺序发生了变化：b = a + 1在a = 1之前执行。

store buffer可能导致破坏程序顺序的问题，硬件工程师在store buffer的基础上，又实现了”store forwarding”技术: cpu可以直接从store buffer中加载数据，即支持将cpu存入store buffer的数据传递(forwarding)给后续的加载操作，而不经由cache。虽然解决了同一个cpu读写数据的问题，但是在并发程序中还是会有问题。例如有如下代码：

```java
//初始状态下，假设a，b值都为0，a存在于cpu1的cache中，b存在于cpu0的cache中，均为Exclusive状态，cpu0执行foo函数，cpu1执行bar函数
void foo() {
    a = 1;
    b = 1;
}
void bar() {
    while (b == 0) continue;
    assert(a == 1)
}
```

可能出现如下操作序列:

1. cpu1执行while(b == 0)，由于cpu1的Cache中没有b，发出Read b消息
2. cpu0执行a=1，由于cpu0的cache中没有a，因此它将a(当前值1)写入到store buffer并发出Read Invalidate a消息
3. cpu0执行b=1，由于b已经存在在cache中，且为Exclusive状态，因此可直接执行写入
4. cpu0收到Read b消息，将cache中的b(当前值1)返回给cpu1，将b写回到内存，并将cache Line状态改为Shared
5. cpu1收到包含b的cache line，结束while (b == 0)循环
6. cpu1执行assert(a == 1)，由于此时cpu1 cache line中的a仍然为0并且有效(Exclusive)，断言失败
7. cpu1收到Read Invalidate a消息，返回包含a的cache line，并将本地包含a的cache line置为Invalid，然而已经为时已晚。
8. cpu0收到cpu1传过来的cache line，然后将store buffer中的a(当前值1)刷新到cache line

出现这个问题的原因在于cpu不知道a,b之间的数据依赖，cpu0对a的写入需要和其他cpu通信，因此有延迟，而对b的写入直接修改本地cache就行，因此b比a先在cache中生效，导致cpu1读到b=1时，a还存在于store buffer中。从代码的角度来看，仿佛foo函数中b = 1又在a = 1之前执行。

foo函数的代码，即使是store forwarding也阻止不了它被cpu“重排”，虽然这并没有影响foo函数的正确性，但会影响到所有依赖foo函数赋值顺序的线程。到目前为止，可以发现”指令重排“的其中一个本质，cpu为了优化指令的执行效率，引入了store buffer（forwarding），而又因此导致了指令执行顺序的变化。要保证这种顺序一致性，靠硬件是优化不了了，需要在软件层面支持，没错，cpu提供了写屏障（write memory barrier）指令，Linux操作系统将写屏障指令封装成了smp_wmb()函数，cpu执行smp_mb()的思路是，会先把当前store buffer中的数据刷到cache之后，再执行屏障后的“写入操作”，该思路有两种实现方式: 一是简单地刷store buffer，但如果此时远程cache line没有返回，则需要等待，二是将当前store buffer中的条目打标，然后将屏障后的“写入操作”也写到store buffer中，cpu继续干其他的事，当被打标的条目全部刷到cache line，之后再刷后面的条目。以第二种实现逻辑为例，看看以下代码执行过程：

```java
//初始状态下，假设a，b值都为0，a存在于cpu1的cache中，b存在于cpu0的cache中，均为Exclusive状态，cpu0执行foo函数，cpu1执行bar函数
void foo() {
    a = 1;
    smp_wmb()
    b = 1;
}
void bar() {
    while (b == 0) continue;
    assert(a == 1)
}
```

1. cpu1执行while(b == 0)，由于cpu1的cache中没有b，发出Read b消息。
2. cpu0执行a=1，由于cpu0的cache中没有a，因此它将a(当前值1)写入到store buffer并发出Read Invalidate a消息。
3. cpu0看到smp_wmb()内存屏障，它会标记当前store buffer中的所有条目(即a=1被标记)。
4. cpu0执行b=1，尽管b已经存在在cache中(Exclusive)，但是由于store buffer中还存在被标记的条目，因此b不能直接写入，只能先写入store buffer中。
5. cpu0收到Read b消息，将cache中的b(当前值0)返回给cpu1，并将cache line状态改为Shared。
6. cpu1收到包含b的cache line，继续while (b == 0)循环。
7. cpu1收到Read Invalidate a消息，返回包含a的cache line，并将本地的cache line置为Invalid。
8. cpu0收到cpu1传过来的包含a的cache line，然后将store buffer中的a(当前值1)刷新到cache line，并且将cache line状态置为Modified。
9. 由于cpu0的store buffer中被标记的条目已经全部刷新到cache，此时cpu0可以尝试将store buffer中的b=1刷新到cache，但是由于包含B的cache line已经不是Exclusive而是Shared，因此需要先发Invalidate b消息。
10. cpu1收到Invalidate b消息，将包含b的cache line置为Invalid，返回Invalidate ACK。
11. cpu1继续执行while(b == 0)，此时b已经不在cache中，因此发出Read消息。
12. cpu0收到Invalidate ACK，将store buffer中的b=1写入Cache。
13. cpu0收到Read消息，返回包含b新值的cache line。
14. cpu1收到包含b的cache line，可以继续执行while(b == 0)，终止循环，然后执行assert(a == 1)，此时a不在其cache中，因此发出Read消息。
15. cpu0收到Read消息，返回包含a新值的cache line。
16. cpu1收到包含a的cache line，断言为真。

#### Invalidate Queues(失效队列)

简单说处理器修改数据时，需要通知其它内核将该缓存中的数据置为Invalid（失效），我们将该数据放到了Store Bufferes处理。那收到失效指令的这些内核会立即处理这种失效消息吗？答案是不会的，因为就算是一个内核缓存了该数据并不意味着马上要用，这些内核会将失效通知放到Invalidate Queues，然后快速返回Invalidate Acknowledge消息。后续收到失效通知的内核将会从该queues中逐个处理该命令。

加入了invalid queue之后，cpu在处理任何cache line的MSEI状态前，都必须先看invalid queue中是否有该cache line的Invalid消息没有处理。另外，它也再一次破坏了内存的一致性。看看以下代码执行过程：

```java
//初始状态下，假设a, b的初始值为0，a在cpu0，cpu1中均为Shared状态，b在cpu0独占(Exclusive状态)，cpu0执行foo，cpu1执行bar
void foo() {
    a = 1;
    smp_wmb()
    b = 1;
}
void bar() {
    while (b == 0) continue;
    assert(a == 1)
}
```

1. cpu0执行a=1，由于其有包含a的cache line，将a写入store buffer，并发出Invalidate a消息。
2. cpu1执行while(b == 0)，它没有b的cache，发出Read b消息。
3. cpu1收到cpu0的Invalidate a消息，将其放入Invalidate Queue，返回Invalidate ACK。
4. cpu0收到Invalidate ACK，将store buffer中的a=1刷新到cache line，标记为Modified。
5. cpu0看到smp_wmb()内存屏障，但是由于其store buffer为空，因此它可以直接跳过该语句。
6. cpu0执行b=1，由于其cache独占b，因此直接执行写入，cache line标记为Modified。
7. cpu0收到cpu1发的Read b消息，将包含b的cache line写回内存并返回该cache line，本地的cache line标记为Shared。
8. cpu1收到包含b(当前值1)的cache line，结束while循环。
9. cpu1执行assert(a == 1)，由于其本地有包含a旧值的cache line，读到a初始值0，断言失败。
10. cpu1这时才处理Invalid Queue中的消息，将包含a旧值的cache line置为Invalid。

问题在于第9步中cpu1在读取a的cache line时，没有先处理Invalid Queue中该cache line的Invalid操作，怎么办？其实cpu还提供了**读屏障指令**，Linux将其封装成smp_rmb()函数，将该函数插入到bar函数中，就像这样：

```java
void foo() {
    a = 1;
    smp_wmb()
    b = 1;
}
void bar() {
    while (b == 0) continue;
    smp_rmb()
    assert(a == 1)
}
```

和smp_wmb()类似，cpu执行smp_rmb()的时，会先把当前invalidate queue中的数据处理掉之后，再执行屏障后的“读取操作”；smp_mb(): 同时具有读屏障和写屏障功能。

### 总结

MESI协议，可以保证缓存的一致性，但是无法保证实时性。为了解决此类问题，CPU提供了一种通过软件告知CPU什么指令不能重排，什么指令能重排的机制，就是内存屏蔽。不同的CPU架构对内存屏障的实现是不尽相同的，Linux操作系统面向cpu抽象出了自己的一套内存屏障函数，它们分别是：

- smp_rmb(): 在invalid queue的数据被刷完之后再执行屏障后的读操作。
- smp_wmb(): 在store buffer的数据被刷完之后再执行屏障后的写操作。
- smp_mb(): 同时具有读屏障和写屏障功能。

## volatile与lock前缀指令

实际通过hsdis工具将代码转换为汇编指令来看volatile的变量在赋值前都加了lock指令，下面详细分析一下lock指令的作用和为什么加上lock指令后就能保证volatile关键字的内存可见性：

### lock指令做了什么

在修改内存操作时，使用LOCK前缀去调用加锁的读-修改-写操作，这种机制用于多处理器系统中处理器之间进行可靠的通讯，具体描述如下：

1. 在Pentium和早期的IA-32处理器中，LOCK前缀会使处理器执行当前指令时产生一个LOCK#信号，这种总是引起显式总线锁定出现
2. 在Pentium4、Inter Xeon和P6系列处理器中，加锁操作是由高速缓存锁或总线锁来处理。如果内存访问有高速缓存且只影响一个单独的高速缓存行，那么操作中就会调用高速缓存锁，而系统总线和系统内存中的实际区域内不会被锁定。同时，这条总线上的其它Pentium4、Intel Xeon或者P6系列处理器就回写所有已修改的数据并使它们的高速缓存失效，以保证系统内存的一致性。如果内存访问没有高速缓存且/或它跨越了高速缓存行的边界，那么这个处理器就会产生LOCK#信号，并在锁定操作期间不会响应总线控制请求。（从Pentium 4，Intel Xeon及P6处理器开始，intel在原有总线锁的基础上做了一个很有意义的优化：如果要访问的内存区域（area of memory）在lock前缀指令执行期间已经在处理器内部的缓存中被锁定（即包含该内存区域的缓存行当前处于独占或以修改状态），并且该内存区域被完全包含在单个缓存行（cache line）中，那么处理器将直接执行该指令。由于在指令执行期间该缓存行会一直被锁定，其它处理器无法读/写该指令要访问的内存区域，因此能保证指令执行的原子性。这个操作过程叫做缓存锁定（cache locking），缓存锁定将大大降低lock前缀指令的执行开销，但是当多处理器之间的竞争程度很高或者指令访问的内存地址**未对齐**时，仍然会锁住总线。）

IA-32架构提供了几种机制用来强化或弱化内存排序模型，以处理特殊的编程情形。这些机制包括：

1. I/O指令、加锁指令、LOCK前缀以及串行化指令等，强制在处理器上进行较强的排序
2. SFENCE指令（在Pentium III中引入）和LFENCE指令、MFENCE指令（在Pentium4和Intel Xeon处理器中引入）提供了某些特殊类型内存操作的排序和串行化功能
   - sfence:  store| 在sfence指令前的写操作当必须在sfence指令后的写操作前完成。
   - lfence：load | 在lfence指令前的读操作当必须在lfence指令后的读操作前完成。
   - mfence：modify/mix | 在mfence指令前的读写操作当必须在mfence指令后的读写操作前完成。

这些机制可以通过下面的方式使用：总线上的内存映射设备和其它I/O设备通常对向它们缓冲区写操作的顺序很敏感，I/O指令（IN指令和OUT指令）以下面的方式对这种访问执行强写操作的排序。在执行了一条I/O指令之前，处理器等待之前的所有指令执行完毕以及所有的缓冲区都被都被写入了内存。只有取指令和页表查询能够越过I/O指令，后续指令要等到I/O指令执行完毕才开始执行。

通过上述描述，可以得到lock指令的几个作用：

1. 锁总线，其它CPU对内存的读写请求都会被阻塞，直到锁释放，不过实际后来的处理器都采用锁缓存替代锁总线，因为锁总线的开销比较大，锁总线期间其他CPU没法访问内存
2. lock后的写操作会回写已修改的数据，同时让其它CPU相关缓存行失效，从而重新从主存中加载最新的数据
3. 不是内存屏障却能完成类似内存屏障的功能，阻止屏障两边的指令重排序

### 由lock指令回看volatile变量读写

工作内存Work Memory其实就是对CPU寄存器和高速缓存的抽象，或者说每个线程的工作内存也可以简单理解为CPU寄存器和高速缓存。

那么当写两条线程Thread-A与Threab-B同时操作主存中的一个volatile变量i时，Thread-A写了变量i，那么：

- Thread-A发出LOCK#指令

- 发出的LOCK#指令锁总线（或锁缓存行），同时让Thread-B高速缓存中的缓存行内容失效
- Thread-A向主存回写最新修改的i

Thread-B读取变量i，那么：

- Thread-B发现对应地址的缓存行被锁了，等待锁的释放，缓存一致性协议会保证它读取到最新的值

### 总结

volatile通过LOCK#指令锁总线(或缓存行)，同时让其他线程高速缓存中的缓存行内容失效，然后向主存回写最新修改后的变量值，其他线程读取变量时发现对应地址的缓存行被锁，会等待锁的释放，然后通过缓存一致性协议保证读取到最新的值(即CPU的数据读取还是使用缓存一致性协议的)。加不加volatile缓存一致性协议(MESI)都存在，不加volatile由于MESI协议只可以保证缓存的一致性，但是无法保证实时性，所以会产生可见性问题。

# 原子性

# 可见性

# 有序性

## 指令重排序

| 名称                    | 说明                                                         |
| ----------------------- | ------------------------------------------------------------ |
| 取指 IF                 | (InstrucTIon Fetch)从内存中取出指令                          |
| 译码和取寄存器操作数 ID | (InstrucTIon Decode)把指令送到指令译码器进行译码，产生相应控制信号 |
| 执行或者有效地址计算 EX | (InstrucTIon Execute)指令执行，在执行阶段的最常见部件为算术逻辑部件运算器(ArithmeTIc Logical Unit，ALU) |
| 存储器访问 MEM          | 访存（Memory Access）是指存储器访问指令将数据从存储器中读出，或者写入存储器的过程 |
| 写回 WB                 | 写回（Write-Back）是指将指令执行的结果写回通用寄存器组的过程 |

单周期处理器(时钟周期1000ps)

```
IF ID EX MEM WB
               IF ID EX MEM WB
			                  IF ID EX MEM WB
```

流水线处理器(时钟周期200ps)指令不是一步可以执行完毕的，每个步骤涉及的硬件可能不同，指令的某一步骤在执行的时候，只会占用某个类型的硬件，例如：当A指令在取指的时候，B指令是可以进行译码、执行或者写回等操作的。所以可以使用流水线技术来执行指令。可以看到，当第2条指令执行时，第1条指令只是完成了取值操作。假如每个步骤需要1毫秒，那么如果指令2等待指令1执行完再执行，就需要等待5毫秒。而使用流水线后，只需要等待1毫秒。

```
IF ID EX MEM WB
   IF ID EX  MEM WB
      IF ID  EX  MEM WB
```

例如：A = B + C的处理，在ADD指令上的大叉表示一个中断，也就是在这里停顿了一下，因为R2中的数据还没准备好。由于ADD的延迟，后面的指令都要慢一个节拍。

```
LW  R1,B     IF ID EX MEM WB                     //LW表示load，把B的值加载到R1寄存器中
LW  R2,C        IF ID EX  MEM WB                 //LW表示load，把C的值加载到R2寄存器中
ADD R3,R1,R2       IF ID  X   EX MEM WB          //ADD是加法，把R1、R2的值相加，并存放到R3中
SW  A,R3              IF  X   ID EX  MEM WB      //SW表示store存储，将R3寄存器的值保存到变量A中
```

再看复杂一点的情况

```
A = B + C 
D = E + F
LW  R1,B     IF ID EX MEM WB                     
LW  R2,C        IF ID EX  MEM WB                 
ADD R3,R1,R2       IF ID  X   EX MEM WB          
SW  A,R3              IF  X   ID EX  MEM WB      
LW  R4,E                  X   IF ID  EX  MEM WB 
LW  R5,F                         IF  ID  EX  MEM WB
ADD R6,R4,R5                         IF  ID  X   EX  MEM WB
SW  D,R6                                 IF  X   ID  EX  MEM WB
```

可见上图中有不少停顿。为了减少停顿，我们只需要将LW R4,E和LW R5,F移动到前面执行。

```
LW  R1,B     IF ID EX MEM WB                     
LW  R2,C        IF ID EX  MEM WB
LW  R4,E           IF ID  EX  MEM WB                
ADD R3,R1,R2          IF  ID  EX  MEM WB
LW  R5,F                  IF  ID  EX  MEM WB       
SW  A,R3                      IF  ID  EX  MEM WB      
ADD R6,R4,R5                      IF  ID  EX  MEM WB
SW  D,R6                              IF  ID  EX  MEM WB
```

可见指令重排序对提高CPU性能十分必要，但是要遵循happens-before规则

1. 程序顺序原则：一个线程内保证语义的串行性(比如a=1;b=a+1;)
2. volatile规则：volatile变量的写，先发生于读，这保证了volatile变量的可见性
3. 锁规则：解锁（unlock）必然发生在随后的加锁（lock）前
4. 传递性：A先于B，B先于C，那么A必然先于C
5. 线程的start()方法先于它的每一个动作
6. 线程的所有操作先于线程的终结（Thread.join()）
7. 线程的中断（interrupt()）先于被中断线程的代码
8. 对象的构造函数执行结束先于finalize()方法

指令重排实例：

```java
package org.duo.ordering;

public class Ordering {
    private static int x = 0, y = 0;
    private static int a = 0, b = 0;

    public static void main(String[] args) throws InterruptedException {
        int i = 0;
        for (; ; ) {
            i++;
            x = 0;
            y = 0;
            a = 0;
            b = 0;
            Thread one = new Thread(new Runnable() {
                public void run() {
                    //由于线程one先启动，下面这句话让它等一等线程two. 可根据自己电脑的实际性能适当调整等待时间.
                    //shortWait(100000);
                    a = 1;
                    x = b;
                }
            });

            Thread other = new Thread(new Runnable() {
                public void run() {
                    b = 1;
                    y = a;
                }
            });
            one.start();
            other.start();
            one.join();
            other.join();
            String result = "第" + i + "次 (" + x + "," + y + "）";
            if (x == 0 && y == 0) {
                System.err.println(result);
                break;
            } else {
                //System.out.println(result);
            }
        }
    }
    public static void shortWait(long interval) {
        long start = System.nanoTime();
        long end;
        do {
            end = System.nanoTime();
        } while (start + interval >= end);
    }
}

```

## 如何保证特定情况下不乱序

### 硬件层面

1. SFENCE指令（在Pentium III中引入）和LFENCE指令、MFENCE指令
2. LOCK前缀指令是一个Full Barrier，执行时会锁住内存子系统来确保执行顺序，甚至跨多个CPU。Software Locks通常使用了内存屏障或原子指令来实现变量可见性和保持程序顺序

### JVM级别如何规范（JSR133）

1. LoadLoad屏障：对于这样的语句Load1; LoadLoad; Load2， 在Load2及后续读取操作要读取的数据被访问前，保证Load1要读取的数据被读取完毕。
2. StoreStore屏障：对于这样的语句Store1; StoreStore; Store2，在Store2及后续写入操作执行前，保证Store1的写入操作对其它处理器可见。
3. LoadStore屏障：对于这样的语句Load1; LoadStore; Store2，在Store2及后续写入操作被刷出前，保证Load1要读取的数据被读取完毕。
4. StoreLoad屏障：对于这样的语句Store1; StoreLoad; Load2， 在Load2及后续所有读取操作执行前，保证Store1的写入对所有处理器可见。

### volatile的实现细节

1. 字节码层面：ACC_VOLATILE

2. JVM层面：volatile内存区的读写 都加屏障

3. OS和硬件层面：在windows系统中利用lock指令实现(有可能linux的实现又不一样，JVM只定义了规范，在不同的操作系统及硬件上的实现也各不相同)

4. 通过反汇编Java字节码，查看汇编层面对volatile关键字做了什么(windows系统)

   - 下载hsdis工具

   - 将hsdis-amd64.dll与hsdis-amd64.lib两个文件放在%JAVA_HOME%\jre\bin\server路径下

   - 跑main函数之前，加入如下虚拟机参数：

     -server

     -Xcomp

     -XX:+UnlockDiagnosticVMOptions

     -XX:+PrintAssembly

     -XX:CompileCommand=compileonly,*LazySingleton.getInstance

     (它表示将LazySingleton类中的getInstance方法转换为汇编指令)


```java
public class LazySingleton {
	private static volatile LazySingleton instance = null;
	public static LazySingleton getInstance() {
		if (instance == null) {
			instance = new LazySingleton();
		}
		return instance;
	}
	public static void main(String[] args) {
		LazySingleton.getInstance();
	}
}
```

### synchronized实现细节

1. 字节码层面：ACC_SYNCHRONIZED、monitorenter、monitorexit
2. JVM层面：C C++ 调用了操作系统提供的同步机制
3. OS和硬件层面：X86 : lock cmpxchg / xxx

