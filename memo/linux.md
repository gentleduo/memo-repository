# Linux发行版

## Red Hat Enterprise Linux

Red Hat Enterprise Linux（缩写为RHEL，Red Hat的企业版Linux）。Red Hat现在主要做服务器版的Linux开发，在版本上注重了性能和稳定性，以及对硬件的支持。由于企业版操作系统的开发周期较长，注重性能、稳定性和服务端软件支持，因此版本更新相对较缓慢。

RHEL大约1年发布一个新版本。其最新版本是2020年11月份发布的RHEL 8.3。

## Centos Linux

CentOS全名为“社区企业操作系统”（Community Enterprise Operating System）。它是来自于RHEL依照开放源代码规定发布的源代码所编译而成，两者的不同在于，CentOS并不包含封闭源代码软件。因此，CentOS不但可以自由使用，而且还能享受CentOS提供的免费长期升级和更新服务。这是CentOS的一个很大优势。目前Centos  Linux最新版本为Centos 8.3。

但是， 2020年12月08日CentOS 官方宣布CentOS Linux项目将停止，并推出CentOS Stream项目。并承诺：

（1）、CentOS Linux 7作为RHEL 7的复刻版本，将会延续当前的支持计划，于2020年第四季度停止更新，并于2024年6月30日停止维护(EOL，End Of Life)；

（2）、CentOS Linux 8作为RHEL 8的复刻版本，生命周期会缩短，将在2021年12月31日停止维护(EOL)，截止目前未看到该日期有延期的可能。

（3）、不会再提供CentOS Linux 9及后续版本，后续将会提供CentOS Stream版本。

centos停止维护后，可以使用其他redhat系的发行版本，比如：oracle linux，它的发行版本和redhat的发行版本也是完全对应起来的，下载地址：https://yum.oracle.com/oracle-linux-isos.html

## Fedora Linux

Fedora Core（缩写为FC）被Red Hat公司定位为新技术的测试平台，许多新的技术都会在FC中检验。如果稳定的话Red Hat公司则会考虑加入到Red Hat Enterprise Linux中。

Fedora对于用户而言，是一套功能完备、更新迅速的免费操作系统，因此，个人领域的应用，例如开发、体验新功能等可选择此发行版本。

## CentOS Stream Linux

Centos Stream是一个滚动发布的Linux发行版，它介于Fedora Linux的上游开发和RHEL的下游开发之间而存在。可以把CentOS Streams当成是用来体验最新红帽系Linux特性的一个版本，尝鲜使用。

CentOS 官方宣布CentOS Linux项目将停止后，CentOS未来将会从Red Hat Enterprise Linux(RHEL)复刻版本的CentOS Linux转向CentOS Stream。

CentOS Stream 是一个滚动升级版本，不再是Red Hat Enterprise Linux的复刻版本，对于系统的稳定性和兼容性可能无法得到保障，其在生产环境上的使用存在的风险未知；

## SuSE Linux

SUSE是德国最著名的Linux发行版，也享有很高的声誉，不过命运相当坎坷，经历了多次的收购，阻碍了SUSE的发展，不过社区项目openSUSE发展的不错，openSUSE网址：https://www.opensuse.org/ 。

虽然SUSE多次易主，但并不影响它的专业性，SUSE Linux现在欧洲Linux市场占有将近80%的份额，但在中国，Linux市场份额并不大，社区活跃度不是很高。

## Ubuntu Linux

Ubuntu（中文谐音为友帮拓、优般图、乌班图）是一个以桌面应用为主的Linux操作系统，基于Debian GNU/Linux，Ubuntu的目标在于为一般用户提供一个最新的、同时又相当稳定的主要由自由软件构建而成的操作系统。Ubuntu具有庞大的社区力量，活跃度很高，用户可以方便地从社区获得帮助。随着云计算的流行，ubuntu推出了一个云计算环境搭建的解决方案，这也更加促使了Ubuntu的流行度。

Ubuntu常用的三个版本：

Ubuntu Desktop（桌面版本，供开发和个人使用）

Ubuntu Server（服务器版本，生产服务器使用）

Ubuntu Cloud（基础云版本，主要是openstack）

## 发行版总结

上面主要介绍了几种最常见的Linux发行版本，其实Linux的发行版本还有很多，比较常见的还有Debian Linux、Gentoo Linux 等，以及国产深度deepin Linux和中标麒麟Linux等。

Linux发行版本无非是朝着这两个方面而来，一是服务器市场，二是桌面市场。

以Ubuntu Linux为代表的Linux发行版走的是桌面市场路线，虽然它们给用户带来很多惊喜，更新也很快，但是由于桌面市场有着Windows这样强劲的对手，因此Linux桌面发展不容乐观，目前Ubuntu Linux也开始向企业级服务器市场发力。

以Red Hat系列版本为代表的Linux发行版现在主要面向企业级Linux的服务器市场，重点开发Linux的企业版本，Linux两大发布厂商现在都走了Linux服务器市场的路线，可见Linux作为企业级服务器有着巨大的发展前途。本课程的讲述也是主要针对Linux在服务器下的各种应用展开的。

Linux运维初学者入门首选——Centos系列

1. CentOS现在拥有庞大的网络用户群体
2. CentOS系列版本可以轻松获得
3. CentOS应用范围广，具有典型性和代表性。

开发用户、桌面用户首选——Ubuntu Linux

1. 软件安装方便、支持度高
2. 硬件支持最好
3. 社区活跃

企业级应用首选——RHEL/Centos7/Oracle Linux系列

1. 性能稳定，以获取市场认可
2. 有厂商技术支持，使用有保障

# Linux服务器安装

## 下载

http://mirrors.163.com/centos/7/isos/x86_64/

https://developer.aliyun.com/mirror/

## RAID

### 概念

RAID 中主要有三个关键概念和技术：镜像（ Mirroring ）、数据条带（ Data Stripping ）和数据校验（ Data parity ） 

- 镜像，将数据复制到多个磁盘，一方面可以提高可靠性，另一方面可并发从两个或多个副本读取数据来提高读性能。显而易见，镜像的写性能要稍低， 确保数据正确地写到多个磁盘需要更多的时间消耗。
- 数据条带，将数据分片保存在多个不同的磁盘，多个数据分片共同组成一个完整数据副本，这与镜像的多个副本是不同的，它通常用于性能考虑。数据条带具有更高的并发粒度，当访问数据时，可以同时对位于不同磁盘上数据进行读写操作， 从而获得非常可观的 I/O 性能提升. 
- 数据校验，利用冗余数据进行数据错误检测和修复，冗余数据通常采用海明码、异或操作等算法来计算获得。利用校验功能，可以很大程度上提高磁盘阵列的可靠性和容错能力。不过，数据校验需要从多处读取数据并进行计算和对比，会影响系统性能。

### RAID的等级

根据运用或组合运用这三种技术的策略和架构，可以把 RAID分为不同的等级，以满足不同数据应用的需求。根据运用或组合运用这三种技术的策略和架构，可以把 RAID分为不同的等级，以满足不同数据应用的需求。根据运用或组合运用这三种技术的策略和架构，可以把 RAID分为不同的等级，以满足不同数据应用的需求。RAID 每一个等级代表一种实现方法和技术，等级之间并无高低之分。在实际应用中，应当根据用户的数据应用特点，综合考虑可用性、性能和成本来选择合适的 RAID 等级，以及具体的实现方式。

### RAID 0

称为条带模式,即把连续的数据分散到多个磁盘上存取。当系统有数据请求就可以被多个磁盘并行的执行，每个磁盘执行属于它自己的那部分数据请求。

 这种数据上的并行操作可以充分利用总线的带宽，显著提高磁盘整体存取性能。因为读取和写入是在设备上并行完成的，读取和写入性能将会增加，这通常是运行RAID0的主要原因。但RAID 0没有数据冗余，如果驱动器出现故障，那么将无法恢复任何数据。

![image](assets\linux-1.png)

### RAID 1

RAID 1可以用于两个或2xN个磁盘，并使用0块或更多的备用磁盘，每次写数据时会同时写入镜像盘。这种阵列可靠性很高，但其有效容量减小到总容量的一半，同时 这些磁盘的大小应该相等，否则总容量只具有最小磁盘的大小。

RAID 1的数据安全性在所有的RAID级别上来说是最好的。但是其磁盘的利用率却只有50%，是所有RAID级别中最低的。

![image](assets\linux-2.png)



### RAID 5

用简单的语言来表示，至少使用3块硬盘（也可以更多）组建RAID5磁盘阵列，当有数据写入硬盘的时候，按照1块硬盘的方式就是直接写入这块硬盘的磁道，如果是RAID5的话这次数据写入会根据算法分成3部分，然后写入这3块硬盘，写入的同时还会在这3块硬盘上写入校验信息，当读取写入的数据的时候会分别从3块硬盘上读取数据内容，再通过检验信息进行校验。当其中有1块硬盘出现损坏的时候,就从另外2块硬盘上存储的数据可以计算出第3块硬盘的数据内容。也就是说raid5这种存储方式只允许有一块硬盘出现故障，出现故障时需要尽快更换。当更换故障硬盘后，在故障期间写入的数据会进行重新校验。 如果在未解决故障又坏1块，那就是灾难性的了。
RAID5把数据和相对应的奇偶校验信息存储到组成RAID5的各个磁盘上，并且奇偶校验信息和相对应的数据分别存储于不同的磁盘上，其中任意N-1块磁盘上都存储完整的数据，也就是说有相当于一块磁盘容量的空间用于存储奇偶校验信息。因此当RAID5的一个磁盘发生损坏后，不会影响数据的完整性，从而保证了数据安全。当损坏的磁盘被替换后，RAID还会自动利用剩下奇偶校验信息去重建此磁盘上的数据，来保持RAID5的高可靠性。
做raid 5阵列所有磁盘容量必须一样大，当容量不同时，会以最小的容量为准。 最好硬盘转速一样，否则会影响性能，而且可用空间=磁盘数n-1，Raid 5 没有独立的奇偶校验盘，所有校验信息分散放在所有磁盘上， 只占用一个磁盘的容量

RAID 5特点：

1. 最少3块磁盘
2. 数据条带形式分布
3. 以奇偶校验作冗余
4. 适合多读少写的情景，是性能与数据冗余最佳的折中方案.

![image](assets\linux-3.png)

### RAID 10

RAID 10(又叫RAID 1+0)特点：

1. 最少需要4块磁盘
2. 先按RAID1分成两组，再分别对两组按RAID 0方式条带化 (先做镜象，然后再做条带)
3. 兼顾冗余(提供镜像存储)和性能(数据条带形分布)
4. 在实际应用中较为常用

![image](assets\linux-4.jpg)

### RAID 01

RAID 01(又叫RAID 0+1)特点：

最少需要4块磁盘

先按RAID 0分成两组，再分别对两组按RAID 1方式镜像（先做条带，然后再做镜象）

兼顾冗余(提供镜像存储)和性能(数据条带形分布)

![image](assets\linux-5.jpg)

### 总结

| **类型** | **读写性能**                                                 | **安全性**                     | **磁盘利用率**        | **成本** | **应用方面**                                                 |
| :------- | :----------------------------------------------------------- | :----------------------------- | :-------------------- | :------- | ------------------------------------------------------------ |
| RAID0    | 最好（因并行性而提高）                                       | 最差（完全无安全保障）         | 最高（100％）         | 最低     | 对安全性要求不是特别高、大文件写存储的系统                   |
| RAID1    | 读和单个磁盘无分别，写则要写两边                             | 最高（提供数据的百分之百备份） | 差（50％）            | 较高     | 适用于存放重要数据，如服务器和数据库存储等领域。             |
| RAID5    | 读：RAID  5＝RAID  0（相近似的数据读取速度）  <BR>写：RAID  5<对单个磁盘进行写入操作（多了一个奇偶校验信息写入） | RAID  5<RAID 1                 | RAID  5>RAID 1        | 中等     | 是一种存储性能、数据安全和存储成本兼顾的存储解决方案。       |
| RAID10   | 读：RAID10＝RAID0<BR>写：RAID10＝RAID1                       | RAID10＝RAID1                  | RAID10＝RAID1（50％） | 较高     | 集合了RAID0，RAID1的优点，但是空间上由于使用镜像，而不是类似RAID5的“奇偶校验信息”，磁盘利用率一样是50％ |

## Linux安装

### 主分区与逻辑分区

Linux下硬盘分区主要分为主分区(Primary Partion)和扩展分区(Extension Partion)两种，且主分区和扩展分区数目之和不能大于四个。主分区一经创建，格式化后可立即使用。扩展分区创建之后，无法直接格式化使用，必须再进行二次逻辑分区(Logical Partion)划分且格式化后才能使用。逻辑分区划分没有数量上的限制。
1到4 对应的是主分区(Primary Partition)或扩展分区(Extension Partition)。从5开始，对应的都是硬盘的逻辑分区(Logical Partition)。一块硬盘即使只有一个主分区，逻辑分区也是从5开始编号的，这点应特别注意。

![image](assets\linux-6.png)

### 分区与命名方案

Linux下是通过字母和数字的组合方式来标识硬盘的分区的，这点不同于Windows系统下使用类似“C盘”或者“C:”来标识硬盘分区。Linux的这种命名方案比起Windows更加灵活，表达的含义也更加清晰，完全可以通过分区标识来详细了解硬盘分区情况。同时Linux的这种硬盘命名方案是基于文件的，一般有如下文件名方式：/dev/hda2、 /dev/sdb3

- /dev：这是所有设备文件存放的目录。
- hd和sd：它们是分区的前两个字母，代表该分区所在的设备类型，其中hd代表IDE硬盘，sd代表SCSI硬盘。
- a：是分区命名的第3个字母，表示分区在哪个设备上。例如，/dev/hda代表第1个IDE硬盘，/dev/sdb则代表第2个SCSI硬盘，/dev/sdd则代表第4块SCSI硬盘，依此类推。
- 2：这个数字代表分区，Linux下前4个分区（主分区或者扩展分区）用数字1～4表示，逻辑分区从5开始，依此类推。例如，/dev/hda2表示第1块IDE硬盘的第2个主分区或者扩展分区，而/dev/sdb3表示第2块SCSI硬盘上的第3个主分区或者扩展分区，/dev/sdc6则表示第3块SCSI硬盘的第2个逻辑分区。

# Linux内存管理与监控

## Linux内存管理与监控

物理内存就是系统硬件提供的内存大小，是真正的内存，相对于物理内存，在linux下还有一个虚拟内存的概念，虚拟内存就是为了满足物理内存的不足而提出的策略，它是利用磁盘空间虚拟出的一块逻辑内存，用作虚拟内存的磁盘空间被称为交换空间（Swap Space）

 linux的内存管理采取的是分页存取机制，为了保证物理内存能得到充分的利用，内核会在适当的时候将物理内存中不经常使用的数据块自动交换到虚拟内存中，而将经常使用的信息保留到物理内存。

 Linux系统会不时的进行页面交换操作，以保持尽可能多的空闲物理内存。

 linux进行页面交换是有条件的，不是所有页面在不用时都交换到虚拟内存。

 交换空间的页面在使用时会首先被交换到物理内存，如果此时没有足够的物理内存来容纳这些页面，它们又会被马上交换出去，如此一来，虚拟内存中可能没有足够空间来存储这些交换页面，最终会导致linux出现假死机、服务异常等问题。

## 内存的监控

```bash
[root@server01 opt]# free
              total        used        free      shared  buff/cache   available
Mem:        2048668      283852     1692172        8584       72644     1643272
Swap:       1952764           0     1952764
```

从内核的角度来查看内存的状态

2048668 - 283852 - 72644 = 1692172

从应用层的角度来看系统内存的使用状态

1692172 + 72644 = 1764816

对于应用程序来说，buff/cache占有的内存是可用的，因为buff/cache是为了提高文件读取的性能，当应用程序需要用到内存的时候，buff/cache会很快地被回收，以供应用程序使用。

## buffers与cached的异同

buffers与cached都是内存操作，用来保存系统曾经打开过的文件以及文件属性信息，这样当操作系统需要读取某些文件时，会首先在buffers与cached内存区查找，如果找到，直接读出传送给应用程序，如果没有找到需要数据，才从磁盘读取，这就是操作系统的缓存机制，通过缓存，大大提高了操作系统的性能。但buffers与cached缓冲的内容却是不同的。

buffers表示块设备(block device)所占用的缓存页（page cache），包括直接读写块设备、以及文件系统元数据(metadata)如SuperBlock所使用的缓存页.

例子： 通过find命令扫描文件系统查看元数据信息，或者通过`cat /dev/sda1 > /dev/null`命令查看块设备信息观察 “buffers” 增加的情况。

cached表示普通文件所占用的缓存页（page cache），cached把读取过的数据保缓存起来，重新读取时若命中（找到需要的数据）就不要去读硬盘了，若没有命中就读硬盘。其中的数据会根据读取频率进行组织，把最频繁读取的内容放在最容易找到的位置，把不经常读取的内容不断往后排，直至从中删除。

例子：通过vi打开一个非常大的文件，看看cached的变化，然后再次vi这个文件，感觉一下两次打开的速度有何异同。

## 手动释放缓存cache

释放page cache：

echo 1 > /proc/sys/vm/drop_caches

实验：

当前Buffers以及Cached

```bash
[root@server01 opt]# cat /proc/meminfo
MemTotal:        2048668 kB
MemFree:         1706952 kB
MemAvailable:    1649488 kB
Buffers:               0 kB
Cached:            33660 kB
```

通过vim打开一个比较大的文件

```bash
[root@server01 opt]# ls -lhat BeijingPM20100101_20151231.csv
-rw-r--r-- 1 root root 164M 10月 17 17:57 BeijingPM20100101_20151231.csv
[root@server01 opt]# vim BeijingPM20100101_20151231.csv
```

此时再来看Buffers以及Cached

```bash
[root@server01 opt]# cat /proc/meminfo
MemTotal:        2048668 kB
MemFree:         1518664 kB
MemAvailable:    1605400 kB
Buffers:               0 kB
Cached:           221624 kB
```

执行清理：

```bash
[root@server01 opt]# echo 1 > /proc/sys/vm/drop_caches
```

此时再来看Buffers以及Cached

```bash
[root@server01 opt]# cat /proc/meminfo
MemTotal:        2048668 kB
MemFree:         1686192 kB
MemAvailable:    1634696 kB
Buffers:               0 kB
Cached:            37848 kB
```

释放文件节点（inodes）缓存和目录项缓存（dentries）

echo 1 > /proc/sys/vm/drop_caches

实验：

当前Buffers以及Cached

```bash
[root@server01 opt]# cat /proc/meminfo
MemTotal:        2048668 kB
MemFree:         1706720 kB
MemAvailable:    1649272 kB
Buffers:               0 kB
Cached:            33708 kB
```

执行查看块设备命令：

```bash
[root@server01 opt]# cat /dev/sda2 > /dev/null
```

此时再来看Buffers以及Cached

```bash
[root@server01 opt]# cat /proc/meminfo
MemTotal:        2048668 kB
MemFree:           75732 kB
MemAvailable:    1591812 kB
Buffers:         1590852 kB
Cached:            48172 kB
```

执行清理：

```bash
[root@server01 opt]# echo 1 > /proc/sys/vm/drop_caches
```

此时再来看Buffers以及Cached

```bash
[root@server01 opt]# cat /proc/meminfo
MemTotal:        2048668 kB
MemFree:         1707136 kB
MemAvailable:    1649768 kB
Buffers:               0 kB
Cached:            33584 kB
```

释放page cache、dentries和inodes缓存：

echo 3 > /proc/sys/vm/drop_caches

在手动释放内存前，需要使用sync指令，将所有未写的系统缓冲区写到磁盘中，包含已修改的 i-node、已延迟的块 I/O 和读写映射文件。否则在释放缓存的过程中，可能会丢失未保存的文件。

在centos7.4中通过，实验证明，1、3都可以释放Buffers和Cached，2不行

## 创建swap交换空间

创建交换空间所需的交换文件是一个普通的文件，但是，创建交换文件与创建普通文件不同，必须通过dd命令来完成，同时这个文件必须位于本地硬盘上，不能在网络文件系统（NFS）上创建swap交换文件。例如：

```bash
[root@server01 ~]# dd if=/dev/zero of=/data/swapfile bs=1024 count=65536
65536+0 records in
65536+0 records out
```

if＝输入文件，或者设备名称。

of＝输出文件或者设备名称。

ibs=bytes 表示一次读入bytes 个字节(即一个块大小为 bytes 个字节)。

obs=bytes 表示一次写bytes 个字节(即一个块大小为 bytes 个字节)。

bs＝bytes，同时设置读写块的大小，以bytes为单位，此参数可代替 ibs 和 obs。

count=blocks 仅拷贝blocks个块。

激活和使用swap

首先通过mkswap命令指定作为交换空间的设备或者文件：

```bash
[root@server01 ~]#mkswap /data/swapfile
```

通过swapon命令激活swap：

```bash
[root@server01 ~]#/usr/sbin/swapon /data/swapfile
```

## swap的优化

swappiness的值的大小对如何使用swap分区是有着很大的联系的。swappiness=0的时候表示最大限度使用物理内存，然后才是 swap空间，swappiness＝100的时候表示积极的使用swap分区，并且把内存上的数据及时的搬运到swap空间里面。linux的基本默认设置为60，具体如下：



cat /proc/sys/vm/swappiness

也就是说，你的内存在使用到100-60=40%的时候，就开始出现有交换分区的使用。



操作系统层面，要尽可能使用内存，对该参数进行调整。



临时调整的方法如下，调成10：

sysctl vm.swappiness=10



要想永久调整的话，需要将在/etc/sysctl.conf修改，加上：

cat /etc/sysctl.conf

vm.swappiness=10

# Linux系统性能评估

## cpu性能评估

### vmstat

vmstat是Virtual Meomory Statistics（虚拟内存统计）的缩写，很多linux发行版本都默认安装了此命令工具，利用vmstat命令可以对操作系统的内存信息、进程状态、CPU活动等进行监视，不足之处是无法对某个进程进行深入分析。vmstat使用语法如下： vmstat [-V] [-n] [delay [count]]

各个选项及参数含义如下：
-V：表示打印出版本信息，是可选参数。
-n：表示在周期性循环输出时，输出的头部信息仅显示一次。
delay：表示两次输出之间的间隔时间。
count：表示按照“delay”指定的时间间隔统计的次数。默认为1。

例如：
vmstat 3
	表示每3秒钟更新一次输出信息，循环输出，按ctrl+c停止输出。
vmstat 3 5
	表示每3秒更新一次输出信息，统计5次后停止输出。

```bash
[root@server01 opt]# vmstat 3
procs -----------memory---------- ---swap-- -----io---- -system-- ------cpu-----
 r  b   swpd   free   buff  cache   si   so    bi    bo   in   cs us sy id wa st
 2  0      0 1548748      0 168984    0    0 11663    69   93  170  0  1 97  2  0
 0  0      0 1549372      0 168936    0    0     0    11   82  153  0  0 99  0  0
 0  0      0 1549372      0 168936    0    0     0     0   53   90  0  0 100  0  0
 0  0      0 1549372      0 168936    0    0     0    50   60   92  0  0 100  0  0
```

*注意：第一行指的是从系统启动后到命令执行为止的一个信息，所以要从第二行开始看*

#### procs

队列信息

##### r

正在运行和等待cpu时间片的进程数，如果该值长期大于cpu核数的2到4倍，考虑cpu资源不足；

##### b

等待资源的进程数，比如等待IO资源；一般来说b的值比较的话，r列的值也会比较大，r列和b列是相互关联的，一般通过r列的值来判断cpu的资源使用情况；

#### memory

##### swpd

切换到内存交换区的数据的大小，单位：KB；但是swpd的值比较大的话并不能说明内存不足，还得结合swap栏中的si和so来看，如果si和so的值长期为0或者很小的话那就没问题，所以内存资源的使用情况主要是通过swap栏的si和so来判断。 

##### free

当前空闲的物理内存的大小，单位：KB。

##### buff

块设备(block device)所占用的缓存页（page cache），单位：KB。

##### cache

普通文件所占用的缓存页（page cache），单位：KB。

#### swap

##### si

表示虚拟内存的页导入，即从SWAP DISK交换到RAM

##### so

表示虚拟内存的页导出，即从RAM交换到SWAP DISK

#### io

##### bi

从块设备读入数据的总量，其实就是读磁盘

##### bo

写入块设备的总量，其实就是写磁盘

单单bi和bo比较大并不能说磁盘io有问题，必须结合iowait来综合判断，只有当bi和bo比较大并且cpu栏中的wa也比较大的时候，说明磁盘读写可能有性能瓶颈，

#### system

##### in

在某一个时间间隔中，观测到的每秒设备的中断数

##### cs

在某一个时间间隔中，产生的上下文切换的次数

当in和cs越大则内核消耗的cpu时间越多

#### cpu

##### us

表示CPU处在用户模式下的时间百分比

##### sy

表示CPU处在系统模式下的时间百分比

##### id

cpu空闲度

##### wa

iowait：表示CPU等待输入输出完成时间的百分比

##### st

虚拟器消耗的百分比，一般忽略不用考虑

### sar

sar命令很强大，是分析系统性能的重要工具之一，通过sar指令，可以全面的获取系统的CPU、运行队列、磁盘I/O、分页（交换区）、内存、CPU中断、网络等性能数据。sar使用格式为： 

sar [options] [-o filename] [interval [count] ] 

各个选项及参数含义如下：
options 为命令行选项，sar命令的选项很多，下面只列出常用选项:
-A：显示系统所有资源设备（CPU、内存、磁盘）的运行状况。 
-u：显示系统所有CPU在采样时间内的负载状态。 
-P：显示当前系统中指定CPU的使用情况。
-d：显示系统所有硬盘设备在采样时间内的使用状况。 
-r：显示系统内存在采样时间内的使用状况。 
-b：显示缓冲区在采样时间内的使用情况。 
-v：显示进程、文件、I节点和锁表状态。 
-n：显示网络运行状态。参数后面可跟DEV、EDEV、SOCK和FULL。DEV显示网络接口信息，EDEV显示网络错误的统计数据，SOCK显示套接字信息，FULL显示三个所有的信息。它们可以单独或者一起使用。
interval：表示采样间隔时间，是必须有的参数。
count：表示采样次数，是可选参数，默认值是1。

-u：表示统计所有cpu；3表示每隔3秒统计一次；5表示总共统计5次。具体每个参数的含义可通过sar --help查看

```bash
[root@server01 ~]# sar -u 3 5
Linux 3.10.0-514.el7.x86_64 (server01)  2022年10月18日  _x86_64_        (1 CPU)

08时11分38秒     CPU     %user     %nice   %system   %iowait    %steal     %idle
08时11分41秒     all      0.00      0.00      0.00      0.00      0.00    100.00
08时11分44秒     all      0.00      0.00      0.00      0.00      0.00    100.00
08时11分47秒     all      0.00      0.00      0.00      0.00      0.00    100.00
08时11分50秒     all      0.00      0.00      0.33      0.00      0.00     99.67
08时11分53秒     all      0.00      0.00      0.00      0.00      0.00    100.00
平均时间:     all      0.00      0.00      0.07      0.00      0.00     99.93
```

#### %user

CPU在用户态执行进程的时间百分比

#### %nice

CPU在用户态模式下，用于nice操作，所占用CPU总时间的百分比，即：改变过优先级的进程的CPU使用率

#### %system

CPU处在内核态执行进程的时间百分比

#### %iowait

CPU用于等待I/O操作占用CPU总时间的百分比

#### %steal

管理程序(hypervisor)为另一个虚拟进程提供服务而等待虚拟CPU的百分比

#### %idle

CPU空闲时间百分比

-P ALL：表示输出所有内核的信息；也可以通过0、1、2、3指示对应的cpu核，通过指定内核的统计信息

```bash
[root@server01 ~]# sar -P 0 3 5
Linux 3.10.0-514.el7.x86_64 (server01)  2022年10月18日  _x86_64_        (1 CPU)

08时24分32秒     CPU     %user     %nice   %system   %iowait    %steal     %idle
08时24分35秒       0      0.00      0.00      0.00      0.00      0.00    100.00
08时24分38秒       0      0.00      0.00      0.33      0.00      0.00     99.67
08时24分41秒       0      0.00      0.00      0.00      0.00      0.00    100.00
08时24分44秒       0      0.00      0.00      0.00      0.00      0.00    100.00
08时24分47秒       0      0.00      0.00      0.00      0.00      0.00    100.00
平均时间:       0      0.00      0.00      0.07      0.00      0.00     99.93
```

### iostat

iostat指令主要用于统计磁盘IO状态，但是也能查看cpu的使用信息，它的局限是只能显示系统所有cpu的平均信息

```bash
[root@server01 ~]# iostat -c 3 5
Linux 3.10.0-514.el7.x86_64 (server01)  2022年10月18日  _x86_64_        (1 CPU)

avg-cpu:  %user   %nice %system %iowait  %steal   %idle
           0.31    0.00    0.32    0.09    0.00   99.28

avg-cpu:  %user   %nice %system %iowait  %steal   %idle
           1.00    0.00    0.67    0.00    0.00   98.33

avg-cpu:  %user   %nice %system %iowait  %steal   %idle
           0.00    0.00    0.00    0.00    0.00  100.00

avg-cpu:  %user   %nice %system %iowait  %steal   %idle
           0.00    0.00    0.33    0.00    0.00   99.67

avg-cpu:  %user   %nice %system %iowait  %steal   %idle
           0.00    0.00    0.00    0.00    0.00  100.00
```

### uptime

uptime是监控系统性能最常用的一个命令，主要用于统计系统当前的运行状况，输出的信息依次为：系统现在的时间、系统从上次开机到现在运行了多长时间、系统目前有多少登录用户、系统在一分钟内、五分钟内、十五分钟内的平均负载。

```bash
[root@server01 ~]# uptime
 08:36:07 up 27 min,  1 user,  load average: 0.00, 0.01, 0.05
```

这里需要注意的是load average这个输出值，这三个值的大小一般不能大于系统cpu的核数，例如，系统有8个cpu，如果load average的三个值长期大于8，说明cpu很繁忙，负载很高，可能会影响系统性能，但是偶尔大于8可以不用担心，一般不会影响系统性能。

### htop

```bash
[root@server01 ~]# yum install htop
```

## 内存性能评估

### free

```bash
[root@server01 ~]# free
              total        used        free      shared  buff/cache   available
Mem:        2048668      305704     1233528        8520      509436     1569696
Swap:       1952764           0     1952764
```

### vmstat

```bash
[root@server01 ~]# vmstat 1 3
procs -----------memory---------- ---swap-- -----io---- -system-- ------cpu-----
 r  b   swpd   free   buff  cache   si   so    bi    bo   in   cs us sy id wa st
 2  0      0 1233768   2072 507452    0    0   100    95   71  173  1  0 99  0  0
 0  0      0 1233768   2072 507452    0    0     0     0   38   86  0  0 100  0  0
 0  0      0 1233768   2072 507452    0    0     0     0   35   85  0  0 100  0  0
```

### smem

```bash
[root@server01 ~]# smem
  PID User     Command                         Swap      USS      PSS      RSS
  538 root     /sbin/agetty --noclear tty1        0      176      206      832
 1668 root     /usr/sbin/anacron -s               0      348      383      780
  382 root     /usr/sbin/lvmetad -f               0      512      565     1440
  446 root     /sbin/auditd                       0      540      566     1124
  474 chrony   /usr/sbin/chronyd                  0      468      605     1608
  482 rpc      /sbin/rpcbind -w                   0      580      611     1252
  516 root     /usr/sbin/crond -n                 0      676      726     1608
  494 root     /usr/lib/systemd/systemd-lo        0      748      791     1672
  472 dbus     /bin/dbus-daemon --system -        0      712      800     1652
  364 root     /usr/lib/systemd/systemd-ud        0      812      888     1852
  541 root     /usr/sbin/sshd                     0      864      931     1384
  798 root     /usr/libexec/postfix/master        0     1244     1296     2268
  347 root     /usr/lib/systemd/systemd-jo        0      956     1500     2856
 1644 postfix  trivial-rewrite -n rewrite         0     1296     1604     4152
 1638 postfix  pickup -l -t unix -u               0     1300     1612     4216
  800 postfix  qmgr -l -t unix -u                 0     1400     1718     4324
 1057 postfix  cleanup -z -t unix -u              0     1424     1736     4336
  469 root     /usr/sbin/rsyslogd -n              0     1452     2047     3620
 1062 postfix  local -t unix                      0     1824     2138     4756
    1 root     /usr/lib/systemd/systemd --        0     2360     2473     3520
 1067 root     sshd: root@pts/0                   0     1332     2630     6112
 1424 root     sshd: root@pts/1                   0     1344     2642     6124
 1070 root     -bash                              0     2528     2983     4184
 1426 root     -bash                              0     2528     2983     4184
 1715 root     python /usr/bin/smem               0     5652     6344     7816
  470 polkitd  /usr/lib/polkit-1/polkitd -        0     9696    10259    12276
  487 root     /usr/bin/python -Es /usr/sb        0    12432    13776    16856
  854 root     /sbin/dhclient -H localhost        0    34352    34357    34524
  858 mysql    /usr/sbin/mysqld --daemoniz        0   187408   187532   188632
```

linux系统使用了virtual memory(虚拟内存) ,如果要准确的计算出一个进程实际使用的物理内存就不是那么的简单能做到的.只知道进程的虚拟内存大小其实没有多大的用处,因为没有办法获取到实际分配的物理内存大小

RSS-(resident set size)：进程占用物理内存大小，RSS是驻留集合大小，即进程所使用的非交换区的物理内存。

1. top命令也可以查询到,最常用的内存指标
2. 将各个进程中的RSS值相加后,一般都会超出整个系统的内存消耗,这是因为RSS中包含了各个进程之间的共享内存

PSS-(proportion set size)：比例集大小

1. 所有使用某共享库的程序均分该共享库占用的内存时,显然所有进程的PSS之和就是系统的内存的使用量,会更准确一些,他将共享内存的大小进行平均后,在分摊到各个进程上去.

USS-(unique set size)：进程独自占用内存

1. 只计算进程独自占用的内存大小,不包含任何共享的部分

smem -p

以百分比的形式报告内存使用情况,可以清楚观察每个进程占用西永的比重是多少

```bash
[root@server01 ~]# smem -p
  PID User     Command                         Swap      USS      PSS      RSS
  538 root     /sbin/agetty --noclear tty1    0.00%    0.01%    0.01%    0.04%
 1668 root     /usr/sbin/anacron -s           0.00%    0.02%    0.02%    0.04%
  382 root     /usr/sbin/lvmetad -f           0.00%    0.02%    0.03%    0.07%
  446 root     /sbin/auditd                   0.00%    0.03%    0.03%    0.05%
  474 chrony   /usr/sbin/chronyd              0.00%    0.02%    0.03%    0.08%
  482 rpc      /sbin/rpcbind -w               0.00%    0.03%    0.03%    0.06%
  516 root     /usr/sbin/crond -n             0.00%    0.03%    0.04%    0.08%
  494 root     /usr/lib/systemd/systemd-lo    0.00%    0.04%    0.04%    0.08%
  472 dbus     /bin/dbus-daemon --system -    0.00%    0.03%    0.04%    0.08%
  364 root     /usr/lib/systemd/systemd-ud    0.00%    0.04%    0.04%    0.09%
  541 root     /usr/sbin/sshd                 0.00%    0.04%    0.05%    0.07%
  798 root     /usr/libexec/postfix/master    0.00%    0.06%    0.06%    0.11%
  347 root     /usr/lib/systemd/systemd-jo    0.00%    0.05%    0.08%    0.14%
 1644 postfix  trivial-rewrite -n rewrite     0.00%    0.06%    0.08%    0.20%
 1638 postfix  pickup -l -t unix -u           0.00%    0.06%    0.08%    0.21%
  800 postfix  qmgr -l -t unix -u             0.00%    0.07%    0.08%    0.21%
 1057 postfix  cleanup -z -t unix -u          0.00%    0.07%    0.08%    0.21%
  469 root     /usr/sbin/rsyslogd -n          0.00%    0.07%    0.10%    0.18%
 1062 postfix  local -t unix                  0.00%    0.09%    0.10%    0.23%
    1 root     /usr/lib/systemd/systemd --    0.00%    0.12%    0.12%    0.17%
 1424 root     sshd: root@pts/1               0.00%    0.07%    0.13%    0.30%
 1067 root     sshd: root@pts/0               0.00%    0.07%    0.13%    0.30%
 1070 root     -bash                          0.00%    0.12%    0.15%    0.20%
 1426 root     -bash                          0.00%    0.12%    0.15%    0.20%
 1795 root     python /usr/bin/smem -p        0.00%    0.28%    0.31%    0.38%
  470 polkitd  /usr/lib/polkit-1/polkitd -    0.00%    0.47%    0.50%    0.60%
  487 root     /usr/bin/python -Es /usr/sb    0.00%    0.61%    0.67%    0.82%
  854 root     /sbin/dhclient -H localhost    0.00%    1.68%    1.68%    1.69%
  858 mysql    /usr/sbin/mysqld --daemoniz    0.00%    9.15%    9.15%    9.21%
```

smem -u

显示系统用户占用内存信息大小

```bash
[root@server01 ~]# smem -u -k
User     Count     Swap      USS      PSS      RSS
chrony       1        0   468.0K   605.0K     1.6M
rpc          1        0   580.0K   611.0K     1.2M
dbus         1        0   712.0K   800.0K     1.6M
postfix      5        0     7.1M     8.6M    21.3M
polkitd      1        0     9.5M    10.0M    12.0M
root        19        0    69.7M    76.9M   101.1M
mysql        1        0   183.0M   183.1M   184.2M
```

smem -P mysql

指定查看谋个用户进程使用内存大小

```bash
[root@server01 ~]# smem -P mysql
  PID User     Command                         Swap      USS      PSS      RSS
 1788 root     python /usr/bin/smem -P mys        0     4816     5515     7000
  858 mysql    /usr/sbin/mysqld --daemoniz        0   187408   187532   188632
```

## 磁盘I/O性能评估

### iostat

iostat是I/O statistics（输入/输出统计）的缩写，主要的功能是对系统的磁盘I/O操作进行监视。它的输出主要显示磁盘读写操作的统计信息，同时也会给出CPU使用情况。同vmstat一样，iostat也不能对某个进程进行深入分析，仅对系统的整体情况进行分析。 iostat使用语法如下：

 iostat [ -c | -d ] [ -k ] [ -t ] [ -x [ device ] ] [ interval [ count ] ]

各个选项及参数含义如下：
-c：显示CPU的使用情况。
-d：显示磁盘的使用情况。
-k：每秒以k bytes为单位显示数据。
-t：打印出统计信息开始执行的时间。
-x device：指定要统计的磁盘设备名称，默认为所有的磁盘设备。
interval：指定两次统计间隔的时间；
count：按照“interval”指定的时间间隔统计的次数。

```bash
[root@server01 ~]# iostat -d 3 5
Linux 3.10.0-514.el7.x86_64 (server01)  2022年10月18日  _x86_64_        (1 CPU)

Device:            tps    kB_read/s    kB_wrtn/s    kB_read    kB_wrtn
sda               2.56        54.53        54.70     258014     258821

Device:            tps    kB_read/s    kB_wrtn/s    kB_read    kB_wrtn
sda               0.00         0.00         0.00          0          0

Device:            tps    kB_read/s    kB_wrtn/s    kB_read    kB_wrtn
sda               0.00         0.00         0.00          0          0

Device:            tps    kB_read/s    kB_wrtn/s    kB_read    kB_wrtn
sda               0.00         0.00         0.00          0          0

Device:            tps    kB_read/s    kB_wrtn/s    kB_read    kB_wrtn
sda               2.68         0.00        13.21          0         39
```

*注意：第一行指的是从系统启动后到命令执行为止的一个信息，所以要从第二行开始看*

### iotop

iotop是一个用来监视磁盘I/O使用状况的top类工具，可以监测到哪一个程序使用的磁盘IO的实时信息。可以通过执行yum在线安装：yum -y install iotop，常用选项

-p 指定进程ID，显示该进程的IO情况

-u 指定用户名，显示该用户所有的进程IO情况

-P,--processes 只显示进程，默认iotop显示的是每个线程的io信息

-k,--kilobytes 以千字节显示

-t,--time 在每一行前添加一个当前的时间

iotop默认显示的是线程信息：TID，如果想要其显示进程信息可以直接按p键，或者通过iotop -P启动；在监控过程中按o键，可以实时监控有io的进程；

## 网络性能评估

### ping

```bash
[root@server01 ~]# ping 8.8.8.8
PING 8.8.8.8 (8.8.8.8) 56(84) bytes of data.
64 bytes from 8.8.8.8: icmp_seq=1 ttl=112 time=113 ms
64 bytes from 8.8.8.8: icmp_seq=2 ttl=112 time=102 ms
64 bytes from 8.8.8.8: icmp_seq=3 ttl=112 time=127 ms
64 bytes from 8.8.8.8: icmp_seq=4 ttl=112 time=100 ms
64 bytes from 8.8.8.8: icmp_seq=5 ttl=112 time=120 ms
^C
--- 8.8.8.8 ping statistics ---
21 packets transmitted, 21 received, 0% packet loss, time 20044ms
rtt min/avg/max/mdev = 85.566/106.601/128.894/13.081 ms

```

icmp_seq如果不连续，表示有丢包；time值显示了两台主机之间的网络延时情况，如果值很大，表示网络的延时很大，单位为毫秒；packet loss表示网络的丢包率，值越小，表示网络的质量越高。

### netstat

```bash
[root@server01 ~]# netstat -i
Kernel Interface table
Iface      MTU    RX-OK RX-ERR RX-DRP RX-OVR    TX-OK TX-ERR TX-DRP TX-OVR Flg
enp0s3    1500    27638      0      0 0         13957      0      0      0 BMRU
enp0s8    1500     7854      0      0 0          7453      0      0      0 BMRU
lo       65536      388      0      0 0           388      0      0      0 LRU
```

1. Iface：网络接口名称

2. MTU：最大传输单元

3. RX-OK：接收时，正确的数据包数

4. RX-ERR：接收时，产生错误的数据包数

5. RX-DRP：接收时，丢弃的数据包数

6. RX-OVR：接收时，由于过速而丢失的数据包数

7. TX-OK：发送时，正确的数据包数

8. TX-ERR：发送时，产生错误的数据包数

9. TX-DRP：发送时，丢弃的数据包数

10. TX-OVR：发送时，由于过速而丢失的数据包数

11. Flg：标志

    - B  已经设置了一个广播地址


    - L 该接口是一个回送设备


    - M 接收所有的数据包(混乱模式)


    - N 避免跟踪


    - O 在该接口上，禁用ARP


    - P 这是一个点到点连接


    - R 接口正在运行


    - U 接口处于"活动"状态

### tcpdump

#### 命令格式

```bash
[root@server01 ~]# tcpdump --help
tcpdump version 4.9.2
libpcap version 1.5.3
OpenSSL 1.0.2k-fips  26 Jan 2017
Usage: tcpdump [-aAbdDefhHIJKlLnNOpqStuUvxX#] [ -B size ] [ -c count ]
                [ -C file_size ] [ -E algo:secret ] [ -F file ] [ -G seconds ]
                [ -i interface ] [ -j tstamptype ] [ -M secret ] [ --number ]
                [ -Q|-P in|out|inout ]
                [ -r file ] [ -s snaplen ] [ --time-stamp-precision precision ]
                [ --immediate-mode ] [ -T type ] [ --version ] [ -V file ]
                [ -w file ] [ -W filecount ] [ -y datalinktype ] [ -z postrotate-command ]
                [ -Z user ] [ expression ]
```

tcpdump可以将网络中传送的数据包的header完全截获下来进行分析，它支持对网络层（net ip 端）、协议（tcp/udp）、主机（src/dst host）、网络或端口（port）的过滤，并提供and、or、not等逻辑语句来去掉无用的信息。tcpdump的常用选项如下：

#### 抓包选项

-c：指定要抓取的包数量。
-i interface：指定tcpdump需要监听的接口。默认会抓取第一个网络接口
-n：对地址以数字方式显式，否则显式为主机名，也就是说-n选项不做主机名解析。
-nn：除了-n的作用外，还把端口显示为数值，否则显示端口服务名。
-P：指定要抓取的包是流入还是流出的包。可以给定的值为"in"、"out"和"inout"，默认为"inout"。
-s len：设置tcpdump的数据包抓取长度为len，如果不设置默认将会是65535字节。对于要抓取的数据包较大时，长度设置不够可能会产生包截断，若出现包截断，所以一般会设置 -s 0；这样会按照包的大小截取数据；抓到的是完整的包数据。

#### 输出选项

-e：输出的每行中都将包括数据链路层头部信息，例如源MAC和目标MAC。
-q：快速打印输出。即打印很少的协议相关信息，从而输出行都比较简短。
-A：以ASCII方式显示包的内容，这个选项对文本格式的协议包很有用
-X：输出包的头部数据，会以16进制和ASCII两种方式同时输出。
-XX：输出包的头部数据，会以16进制和ASCII两种方式同时输出，更详细。
-v：当分析和打印的时候，产生详细的输出。
-vv：产生比-v更详细的输出。
-vvv：产生比-vv更详细的输出。

#### 其他功能性选项

-w：将抓包数据输出到文件中而不是标准输出。推荐使用-w t.out然后用-r t.out来看抓包信息。
-r：从给定的数据包文件中读取数据。使用"-"表示从标准输入中读取。

#### expression

==一个基本的表达式单元格式为"proto dir type ID"==

类型 type

host, net, port, portrange

例如：host 192.168.201.128  ,  net 128.3, port 20, portrange 6000-6008'

目标 dir

src,  dst,  src  or dst, src and dst

协议 proto

tcp， udp ， icmp，若未给定协议类型，则匹配所有可能的类型

示例：

```bash
[root@server01 ~]# tcpdump -n -i enp0s8 -c 5 src host 192.168.56.1
tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
listening on enp0s8, link-type EN10MB (Ethernet), capture size 262144 bytes
12:50:54.517640 IP 192.168.56.1.63223 > 192.168.56.110.ssh: Flags [.], ack 1732657228, win 8207, length 0
12:50:54.559283 IP 192.168.56.1.63223 > 192.168.56.110.ssh: Flags [.], ack 161, win 8212, length 0
12:50:54.600083 IP 192.168.56.1.63223 > 192.168.56.110.ssh: Flags [.], ack 321, win 8211, length 0
12:50:54.646512 IP 192.168.56.1.63223 > 192.168.56.110.ssh: Flags [.], ack 481, win 8211, length 0
12:50:54.688145 IP 192.168.56.1.63223 > 192.168.56.110.ssh: Flags [.], ack 641, win 8210, length 0
5 packets captured
5 packets received by filter
0 packets dropped by kernel
```

tcpdump常见的包携带的标志，即：Flags

S: S=SYC : 发送连接标志

P: P=PUSH:传送数据标志

F: F=FIN:连接关闭标志

ack:表示确认包

RST=RESET:异常关闭连接

.:表示没有任何标志



