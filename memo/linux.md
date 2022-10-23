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

### 软件选择

基本环境：带GUI的服务器，带有用于操作网络基础设施服务GUI的服务器。

已选环境的附件项：开发工具，基本开发环境。

### 安装位置

主要是对系统进行分区，分区方案主要有两种：标准分区和LVM，生产环境一般使用标准分区，因为LVM分区存在几个缺点：性能相对于标准分区读写性能会差很多，因为在文件系统之上又集成了一个LVM的管理层；其他LVM后期维护性很差，磁盘坏了后数据几乎没办法恢复。

#### 挂载点

/boot

系统引导程序，一般500M就足够了。然后设备类型一般选标准分区，文件系统一般选xfs，centos默认的文件系统格式就是xfs，在centos下xfs性能最好。

/var

主要存放系统日志，应用程序日志，大小一般为磁盘总容量的10%

/usr

主要存放应用程序，大小一般为磁盘总容量的10%

swap

swap指的是linux交换分区，是磁盘上的一块区域，可以是一个分区，也可以是一个文件，或者是两者的组合；swap类似于Windows的虚拟内存，就是当内存不足时，把一部分硬盘空间虚拟成内存使用，从而解决内存容量不足的情况。

| 内存大小         | swap大小     |
| ---------------- | ------------ |
| <16G             | =2倍物理内存 |
| 16G<物理内存<32G | =物理内存    |
| >32G             | 8G           |

/

根分区，大小一定要足够大，一般为磁盘总容量的20%

#### KDUMP

kdump是在系统崩溃、死锁或者死机的时候用来转储内存运行参数的一个工具和服务；需要分析内核崩溃原因的话可以开启。

# Linux系统基本结构与运行机制

## Linux控制台的使用

 Linux系统通过w命令查看连接到系统的终端信息，主要包括两大类：控制台终端(/dev/ttyn）虚拟终端(/dev/pts/n），虚拟终端一般是远程连接进来的

```bash
[root@server01 ~]# w
 17:32:15 up  1:25,  1 user,  load average: 0.00, 0.01, 0.03
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
root     pts/0    192.168.56.1     17:27    7.00s  0.08s  0.00s w
```

## Linux硬件资源管理

查看cpu信息

```bash
[root@server01 ~]# lscpu
[root@server01 ~]# cat /proc/cpuinfo
#要查看系统物理CPU的个数，可通过如下命令查看：
[root@server01 ~]# cat /proc/cpuinfo | grep "physical id" | sort | uniq | wc –l
#要查看每个物理CPU中Core的个数，可通过如下命令查看：
[root@server01 ~]# cat /proc/cpuinfo | grep "cpu cores"
#要查看系统所有逻辑CPU个数（所有物理CPU中Core的个数加上超线程个数），可通过如下命令查看：
[root@server01 ~]# cat /proc/cpuinfo | grep "processor" | wc –l
```

查看内存信息

```bash
[root@server01 ~]# lsmem
[root@server01 ~]# cat /proc/meminfo
```

查看硬盘信息

```bash
[root@server01 ~]# lsblk
[root@server01 ~]# fdisk -l
```

## 硬件与设备文件

在Linux系统下，硬件设备都是以文件的形式存在，因而不同硬件设备有不同的文件类型，我们把硬件与系统下相对应的文件称作设备文件。设备文件在外部设备与操作系统之间提供了一个接口，这样，用户使用外部设备就相当于使用普通文件一样。

设备文件在Linux系统下存放在/dev下面，设备文件的命名方式是主设备号加次设备号，主设备号说明设备类型，次设备号说明具体指哪一个设备。

U盘在Linux下被识别为SCSI设备，因此对应的设备文件为/dev/sdax，主设备号sd表示SCSI disk，a表示第一块SCSI设备。如果有第二块SCSI设备，那么对应的设备文件是/dev/sdb。x表示SCSI设备的相应分区编号。例如，/dev/sda1表示第一块SCSI设备的第一个分区，/dev/sdc5表示第三块SCSI设备的第一个逻辑分区。

光驱是我们最经常使用的外设之一。IDE光驱在Linux下对应的设备文件为/dev/had，表示在第一个IDE口（Master）的IDE光驱；SCSI光驱在Linux下对应的设备文件为/dev/srx，x表示SCSI ID。

## Linux下常见文件系统类型

文件系统类型就是分区的格式，对于不同的外设，Linux也提供了不同的文件类型。

1. msdos：DOS文件系统类型
2. vfat：支持长文件名的DOS分区文件系统类型，也可理解为Windows文件系统类型
3. iso9660：光盘格式文件系统类型
4. ext2/ext3/ext4：Linux下的主流文件系统类型
5. xfs：Linux下一种高性能的日志文件系统，在Centos7.x版本中成为默认文件系统

## Linux下设备的挂载与使用

Linux下挂载的命令是mount，格式如下：

mount -t 文件系统类型 设备名 挂载点

文件系统类型就是上面讲到的几种分区格式，设备名就是对应的设备文件，挂载点就是在Linux下指定的挂载目录，将设备指定到这个挂载目录后，以后访问这个挂载目录，就相当于访问这个设备了。Linux系统中有一个/mnt目录，专门用作临时挂载点（Mount Point）目录，主要用于系统管理员临时手动挂载一些媒体设备。此外，Linux系统中还有一个/media目录，此目录是一个自动挂载的目录，主要用于自动挂载光盘、U盘等移动设备。

```bash
# 挂载iso
[root@server01 ~]# mount /home/CentOS-7-x86_64-DVD-1611.iso /media -o loop
```

设备的卸载

umonut 挂载目录

## Linux文件系统结构与目录功能

### 经典树形目录

Linux系统设计中最优秀的特性之一就是将所有内容都以文件的形式展现出来，通过一个树形结构统一管理和组织这些文件。

![image](assets\linux-7.png)

### 目录功能介绍

#### /etc

这个目录主要用于存放系统管理相关的配置文件以及子目录，其中比较重要的有系统初始化文件/etc/rc、用户信息文件/etc/passwd等，相关网络配置文件和服务启动文件也均在这个目录下

#### /usr

此目录主要用于存放应用程序和文件。如果在系统安装的时候，选择了很多软件包，那么这些软件包默认会安装到此目录下，我们平时安装的一些软件，默认情况下也会安装到此目录内，因此这个目录一般比较大。

#### /var

此目录主要用于存放系统运行以及软件运行的日志信息

#### /dev

dev目录包含了系统所有的设备文件

#### /proc

此目录是一个虚拟目录，目录所有信息都是内存的映射，通过这个虚拟的内存映射目录，可以和内核内部数据结构进行交互，获取有关进程的有用信息，同时也可以在系统运行中修改内核参数。与其他目录不同，/proc存在于内存中，而不是硬盘上。

#### /boot

该目录存放的是启动Linux时的一些核心文件，具体包含一些镜像文件和链接文件，因此这个目录非常重要，如果遭到破坏，系统将无法启动。

#### /bin、/sbin

这两个目录存放的都是可执行的二进制文件，bin其实就是binary的缩写，/bin目录下存放的就是我们经常使用的Linux命令，例如文件操作命令ls、cd、cp，文本编辑命令vi、ed，磁盘操作命令dd、df、mount，等等。/sbin中的s是Spuer User的意思，也就是说只有超级用户才能执行这些命令，常见的如磁盘检查修复命令fcsk、磁盘分区命令fdisk、创建文件系统命令mkfs、关机命令shutdown和初始化系统命令init等。

#### /home

该目录是系统中每个用户的工作目录，在Linux系统中，每个用户都有自己的一个目录，而该目录一般是由用户的账号命名的，例如有一个用户ixdba，那么它的默认目录就是/home/ixdba。

#### /lib、/lib64

该目录中存放的是共享程序库和映像文件，可供很多程序使用。通过这些共享映射文件，每个程序就不必分别保存自己的库文件（这会增加占用的磁盘空间），Linux提供了一组可供所有程序使用的文件。在该目录中，还包含引导进程所需的静态库文件。

#### /root

该目录是Linux超级用户root的默认主目录，如果通过root登录系统，就会自动进入到此目录，一般用户没有进入这个目录的权限。

#### /tmp

该目录为临时文件目录，主要用于存放临时文件，这些临时文件可能会随时被删除，也可以随时删除。

## Linux初始化init系统

### init系统

Linux操作系统的启动首先从BIOS开始，接下来Linux引导程序将内核映像加载到内存，进行内核初始化，内核初始化的最后一步就是启动PID为1的init进程。这个进程是系统的第一个进程，它负责产生其它所有用户进程。仅仅将内核运行起来对使用操作系统来说毫无用处，因为对于操作系统来说内核只是基础，在内核之上还有很多服务的启动，只有服务启动之后才能使用操作系统。所以仅仅将init进程启动起来是不行的，所以接下来还有一系列的进程服务需要启动，所以还需要有一个系统去定义、管理和控制init进程的行为，并去负责组织运行许多独立或者相关的初始化工作，从而令操作系统进入用户设置的模式中运行。这个系统就是init系统，init系统和init进程是不同的概念。

大多数Linux发行版的init系统是和System V相兼容的，因此被称为sysvinit，这是最早也是最流行的init系统，在RHEL7.x/Centos7.x发行版本之前的系统中都采用sysvinit。sysvinit概念简单清晰，主要依赖于Shell脚本，但它一次一个串行地启动进程，决定了它的最大弱点：启动太慢。虽然在服务器上这个缺点不算什么，但是当Linux被应用到移动终端设备上时，这个缺点就变成了大问题。

upstart和systemd这两个是新一代init系统，以Ubuntu为代表的Linux发行版就采用的是upstart方式，而在RHEL7.x/Centos7.x版本中，已经默认开始采用systemd来管理系统。Upstart出现很早，而systemd出现较晚，但发展更快，大有取代upstart的趋势。systemd主要特点：并发处理所有服务，加速开机流程。

### init管理机制

init管理主要是针对于centos7以前的版本

init管理机制与特点
所有的服务启动脚本都放置于/etc/init.d/目录，基本上都是使用bash shell所写成的脚本程序，需要启动、关闭、重新启动、查看状态时，可以通过如下的方式来处理：
启动：/etc/init.d/daemon start
关闭：/etc/init.d/daemon stop
重启：/etc/init.d/daemon restart
查看状态：/etc/init.d/daemon status

init机制的系统运行级
init可以根据使用者自订的执行等级(runlevel)来唤醒不同的服务，以进入不同的操作模式。基本上 Linux 提供7个执行等级，分别是0, 1, 2...6 ，
比较重要的是：
init 0：关机模式
init 1：单人维护模式、
init 3：纯字符模式、
init 5：图形界面模式。
init 6：重启模式

init管理机制服务设置方式
各个执行等级的启动脚本是通过 /etc/rc.d/rc[0-6]/SXXdaemon 链接到 /etc/init.d/daemon 。即：管理在各个系统运行级中需要启动的服务
链接文件名 (SXXdaemon) 的功能为： S 为启动该服务，XX是数字，表示启动的顺序。由于有SXX的设定，因此在开机时可以依序执行所有需要的服务。
设置服务自启动： chkconfig daemon on
关闭服务自启动： chkconfig daemon off
查看服务是否自启动： chkconfig --list daemon
设置服务在指定运行级中自动启动： chkconfig --level 35 daemon on
将服务添加至系统运行级管理中： chkconfig --add daemon
将服务从系统运行级管理中移除：chkconfig --del daemon

### runlevel到target的改变

在RHEL7.x/Centos7.x版本中，由于采用了systemd管理体系，以前的运行级别（runlevel）的概念被新的运行目标（target）所取代，tartget的命名类似于“multi-user.target”这种形式，比如原来的运行级别3（runlevel3）对应于新的多用户目标“multi-user.target”，运行级别5（runlevel5）就对应于“graphical.target”。由于systemd机制中不再使用runlevle的概念，所以/etc/inittab(centos6中inittab的主要作用是定义操作系统默认的运行级别)也不再被系统使用。在新的systemd管理体系里，默认的target（相当于以前的默认运行级别）是通过软链来实现。

```bash
[root@server01 rc5.d]# cd ~
[root@server01 ~]# ll /lib/systemd/system/runlevel*.target
lrwxrwxrwx. 1 root root 15 7月  31 2017 /lib/systemd/system/runlevel0.target -> poweroff.target
lrwxrwxrwx. 1 root root 13 7月  31 2017 /lib/systemd/system/runlevel1.target -> rescue.target
lrwxrwxrwx. 1 root root 17 7月  31 2017 /lib/systemd/system/runlevel2.target -> multi-user.target
lrwxrwxrwx. 1 root root 17 7月  31 2017 /lib/systemd/system/runlevel3.target -> multi-user.target
lrwxrwxrwx. 1 root root 17 7月  31 2017 /lib/systemd/system/runlevel4.target -> multi-user.target
lrwxrwxrwx. 1 root root 16 7月  31 2017 /lib/systemd/system/runlevel5.target -> graphical.target
lrwxrwxrwx. 1 root root 13 7月  31 2017 /lib/systemd/system/runlevel6.target -> reboot.target
```

### systemd

systemd提供了一个非常强大的命令行工具systemctl，可能很多系统运维人员都已经非常熟悉基于sysvinit的服务管理方式，比如service、chkconfig命令，而systemd也能完成同样的管理任务，可以把systemctl看作是service和chkconfig的组合体。要查看、启动、停止、重启、启用或者禁用系统服务，都可以通过systemctl命令来实现 。

linux系统中有很多的system目录，常看到的有/etc/systemd/system、/lib/systemd/system以及/usr/lib/systemd/system；目录/lib/systemd/system 以及/usr/lib/systemd/system 其实指向的是同一目录，在/目录下执行命令 `ll` 即可知：

```bash
[root@server01 system]# ll /
总用量 56
drwxr-xr-x   3 root root    36 8月  25 09:51 app
lrwxrwxrwx.  1 root root     7 7月  31 2017 bin -> usr/bin
dr-xr-xr-x.  4 root root  4096 7月  31 2017 boot
drwxr-xr-x   3 root root    24 10月  7 09:33 data
drwxr-xr-x  18 root root  2960 10月 23 10:48 dev
drwxr-xr-x. 86 root root  8192 10月 23 10:48 etc
drwxrwxrwx   2 root root    22 3月  12 2019 extend_folder
drwxr-xr-x. 17 root root  4096 2月  10 2022 home
-rw-r--r--   1 root root 12153 8月  18 2017 jars
lrwxrwxrwx.  1 root root     7 7月  31 2017 lib -> usr/lib
lrwxrwxrwx.  1 root root     9 7月  31 2017 lib64 -> usr/lib64
drwxr-xr-x.  3 root root    38 10月 23 2018 media
drwxr-xr-x.  2 root root     6 3月   5 2018 mnt
drwxr-xr-x   2 root root     6 1月  17 2021 oldroot
drwxr-xr-x. 42 root root  8192 10月 22 21:17 opt
drwxr-xr-x   3 root root    22 3月  13 2018 output
dr-xr-xr-x  89 root root     0 10月 23 10:48 proc
dr-xr-x---. 21 root root  4096 10月 23 11:43 root
drwxr-xr-x  22 root root   700 10月 23 10:48 run
lrwxrwxrwx.  1 root root     8 7月  31 2017 sbin -> usr/sbin
drwxr-xr-x.  2 root root     6 11月  5 2016 srv
dr-xr-xr-x  13 root root     0 10月 23 10:48 sys
drwxrwxrwt. 12 root root  4096 10月 23 11:42 tmp
drwxr-xr-x. 13 root root   155 7月  31 2017 usr
drwxr-xr-x. 20 root root  4096 10月 23 10:48 var
```

/etc/systemd/system/(系统管理员安装的单元, 优先级更高)。在一般的使用场景下，每一个 Unit（服务等） 都有一个配置文件，告诉 Systemd 怎么启动这个 Unit 。Systemd 默认从目录`/etc/systemd/system/`读取配置文件。但是，里面存放的大部分文件都是符号链接，指向目录`/usr/lib/systemd/system/`，真正的配置文件存放在这个目录。 `systemctl enable` 命令用于在上面两个目录之间，建立符号链接关系。

```bash
# 比如配置docker service开机启动，其实真正的Unit在放在/usr/lib/systemd/system/目录下，/etc/systemd/system/目录只是存放指向该文件的软链接；
[root@server01 system]# systemctl enable docker   
Created symlink from /etc/systemd/system/multi-user.target.wants/docker.service to /usr/lib/systemd/system/docker.service.
[root@server01 system]# systemctl disable docker
Removed symlink /etc/systemd/system/multi-user.target.wants/docker.service.
# default.target设置的是系统默认运行级别，相当于原来init管理中的inittab
[root@server01 system]# ll /etc/systemd/system
总用量 4
drwxr-xr-x. 2 root root   31 8月  23 11:02 basic.target.wants
lrwxrwxrwx. 1 root root   37 7月  31 2017 default.target -> /lib/systemd/system/multi-user.target
drwxr-xr-x. 2 root root   87 7月  31 2017 default.target.wants
drwxr-xr-x  2 root root   53 8月   2 2019 docker.service.d_BAK
drwxr-xr-x. 2 root root   32 7月  31 2017 getty.target.wants
drwxr-xr-x. 2 root root 4096 10月 23 11:50 multi-user.target.wants
drwxr-xr-x  2 root root   51 3月  12 2019 sockets.target.wants
drwxr-xr-x  2 root root   89 5月  15 2018 sysinit.target.wants
drwxr-xr-x. 2 root root   44 7月  31 2017 system-update.target.wants
```

### systemd命令和sysvinit命令对比

| **sysvinit** **命令**              | **systemd** **命令**                | **备注**                                               |
| ---------------------------------- | ----------------------------------- | ------------------------------------------------------ |
| **service httpd start**            | systemctl  start httpd.service      | 启动httpd服务                                          |
| **service httpd stop**             | systemctl  stop httpd.service       | 关闭httpd服务                                          |
| **service** **httpd**  **restart** | systemctl  restart httpd.service    | 重启httpd服务，而不管httpd服务当前是否是启动或关闭状态 |
| **service httpd reload**           | systemctl  reload httpd.service     | 重新载入httpd配置信息而不中断服务                      |
| **service** **httpd**  **status**  | systemctl  status httpd.service     | 查看httpd服务的运行状态                                |
| **chkconfig httpd on**             | systemctl  enable httpd.service     | 设置httpd服务开机自启动                                |
| **chkconfig httpd off**            | systemctl  disable httpd.service    | 禁止httpd服务开机自启动                                |
| **chkconfig httpd**                | systemctl  is-enabled httpd.service | 检查httpd服务在当前环境下是启用还是禁用                |

# Linux常用命令及使用技巧

## shell简介

shell的本意是“壳”的意思，其实已经很形象地说明了shell在Linux系统中的作用。shell就是围绕在Linux内核之外的一个“壳”程序，用户在操作系统上完成的所有任务都是通过shell与Linux系统内核的交互来实现的。

Linux下除了默认的Bourne again shell（bash），还有很多其他的shell，例如C shell（csh）、Korn shell（ksh）、Bourne shell（sh）和Tenex C shell（tcsh）等。每个版本的shell功能基本相同，但各有千秋，现在的Linux系统发行版一般都以bash作为默认的shell。

为了加快命令的运行，同时更有效地定制shell程序，shell中定义了一些内置的命令，一般我们把shell自身解释执行的命令称为内置命令，例如下面我们将要讲到的cd、pwd、exit和echo等命令，都是属于bash的内置命令。

除了内置命令，Linux系统上还有很多可执行文件。可执行文件类似于Windows下的.exe文件，这些可执行文件也可以作为shell命令来执行。其实Linux上很多命令都不是shell的内置命令，例如ls就是一个可执行文件，存放在/bin/ls中。这些命令与shell内置命令不同，只有当它们被调用时，才由系统装入内存执行。

shell执行命令解释的具体过程为：用户在命令行输入命令提交后，shell程序首先检测是否为内置命令，如果是，就通过shell内部的解释器将命令解释为系统调用，然后提交给内核执行；如果不是shell内置的命令，那么shell会按照用户给出的路径或者根据系统环境变量的配置信息在硬盘寻找对应的命令，然后将其调入内存，最后再将其解释为系统调用，提交给内核执行。

## shell语法分析

### 命令格式

用户登录系统后，shell命令行启动。shell遵循一定的语法格式将用户输入的命令进行分析解释并传递给系统内核。shell命令的一般格式为：
command	[options]	[arguments]
根据习惯，我们一般把具有以上格式的字符串称为命令行。命令行是用户与shell之间对话的基本单位。
command：表示命令的名称。
options：表示命令的选项。
arguments：表示命令的参数
在命令行中，选项是包含一个或多个字母的代码，主要用于改变命令的执行方式。一般在选项前面有一个“-”符号，用于区别参数。

### 通配符

通配符主要是为了方便用户对文件或者目录的描述，例如用户仅仅需要以“.sh”结尾的文件时，使用通配符就能很方便地实现。各个版本的shell都有通配符，这些通配符是一些特殊的字符，用户可以在命令行的参数中使用这些字符，进行文件名或者路径名的匹配。

“*”、匹配任意一个或多个字符
“?”、匹配任意单一字符
“[]”、匹配任何包含在方括号内的单字符

```bash
# 列出当前目录下以数字开头，随后一个是任意字符，接着以“.conf”结尾的所有文件。
[root@server01 opt]# ls [0-9]?.conf
11.conf
# 列出当前目录下以x、y或z开头，最后以“.txt”结尾的文件。
[root@server01 opt]# ls [xyz]*.txt
x4adf.txt
```

### 引用

在bash中有很多特殊字符，这些字符本身就具有特殊含义。如果在shell的参数中使用它们，就会出现问题。Linux中使用了“引用”技术来忽略这些字符的特殊含义，引用技术就是通知shell将这些特殊字符当作普通字符处理。shell中用于引用的字符有转义字符 “\”、单引号” ‘’ ”、双引号 “ “” “。

如果将”\”放到特殊字符前面，shell就忽略这些特殊字符的原有含义，当作普通字符对待，例如：

```bash
[root@server01 opt]# vim C:\\backup
[root@server01 opt]# ll C:\backup
ls: 无法访问C:backup: 没有那个文件或目录
[root@server01 opt]# ll C:\\backup
-rw-r--r-- 1 root root 9 10月 23 13:36 C:\backup
# 将C:\backup重命名为backup。因为文件名中含有特殊字符，所有都使用了转义字符“\”。
[root@server01 opt]# mv C\:\\backup backup
```

单引号''，双引号""的区别是单引号''剥夺了所有字符的特殊含义，单引号''内就变成了单纯的字符。双引号""则大部分特殊字符可以当作普通字符处理，但是仍有一些特殊字符即使用双引号括起来，也仍然保留自己的特殊含义，比如参数替换：“$”、转义字符：“\”和命令替换：“`”

例如：

```bash
[root@server01 opt]# n=3;
# 使用$替换参数
[root@server01 opt]# echo "$n"     
3
# 使用``替换命令
[root@server01 opt]# echo "`date`"
2022年 10月 23日 星期日 15:10:01 CST
# 单引号''内就变成了单纯的字符
[root@server01 opt]# echo '$n' 
$n
[root@server01 opt]# echo '`date`'
`date`
```

## 系统管理与维护命令

### ls

显示指定工作目录下的内容，列出工作目录所含的文件及子目录。此命令与Windows下的dir类似。另外，Linux也提供了dir命令，用户也可以用dir命令代替ls命令。ls的语法如下：
ls [选项] [路径或文件]
常用选项：
-a	显示指定目录下的所有文件以及子目录，包含隐藏文件（Linux下将“.”开头的文件或者目录视为隐藏文档）
-l	除文件名称外，同时将文件或者子目录的权限、使用者和大小等信息详细列出
-S	以文件大小排序
-pF	在每个文件名后附上一个字符以说明该文件的类型。 “*”表示可执行的普通文件，“/”表示目录， “@” 表示符号链接，“|”表示FIFOs，“=”表示套接字（sockets）
常用组合：
ls –al、ll(ls –l) 、ls –Sl、	ls -apF

### passwd

用于设置用户口令。语法格式如下：passwd [用户名]

普通用户要修改自己的口令，可使用以下命令：passwd

超级用户root修改某个用户的口令时，使用以下命令：passwd

### su

su命令主要用于改变用户身份，其格式如下：
su [选项] [用户名]
-：加载相应用户下的环境变量（注释不是下划线是横杠）
-l：使目前的shell成为改变身份后用户默认的shell
-c：改变身份运行一个指令后就结束

```bash
# su后面的“-”就是加载root环境变量，如果直接输入su也可以转变为超级用户，但是由于没有加载root环境变量，因此某些指令可能无法执行，会提示“command not found”。
[root@server01 ~]# su -
上一次登录：日 10月 23 15:20:53 CST 2022pts/0 上
```

### dmesg

功能说明
kernel会将开机信息存储在ring buffer中。若是开机时来不及查看信息，可利用dmesg来查看。开机信息亦保存在/var/log目录中，名称为dmesg的文件里。其格式如下：
dmesg [选项]
-c	显示开机信息后，清除ring buffer信息
-s	设置缓冲区大小，默认设置为8192
主要用途:
dmesg经常用于系统异常诊断，当系统出现问题时，会第一时间将异常信息写入dmesg内存中，注意dmesg命令和/var/log/dmesg的区别；dmesg命令输出的是内存中实时的状态信息或者说是在系统缓冲区（ring buffer）中的信息；而/var/log/dmesg保存的是系统的开机信息。

### free

功能说明
free命令用来显示系统内存状态，具体包括系统物理内存、虚拟内存、共享内存和系统缓存。其格式如下：
free [选项] [-s （间隔秒数）]
-b	以Byte为单位显示内存使用情况
-m	以MB为单位显示内存使用情况
-K	以kB为单位显示内存使用情况
-h  以合适的单位显示内存使用情况，最大为三位数，自动计算对应的单位值
-s（间隔秒数）	根据指定的间隔秒数持续显示内存使用情况

### ps

功能说明
ps命令显示系统进程在瞬间的运行动态，其格式如下：
ps [选项]
ps的选项非常之多，这里我们仅仅介绍常用的选项
-a	显示所有用户的进程，包含每个程序的完整路径
-x	显示所有系统程序，包括那些没有终端的程序
-u	显示使用者的名称和起始时间
-f	 详细显示程序执行的路径
-l     代表长格式
-e	将除内核进程以外所有进程的信息写到标准输出

ps -eLf

- UID：用户ID
- PID：process id 进程id
- PPID: parent process id 父进程id
- LWP：表示这是个线程；要么是主线程(进程)，要么是线程
- NLWP: num of light weight process 轻量级进程数量，即线程数量
- STIME: start time 启动时间
- TIME: 占用的CPU总时间
- TTY：该进程是在哪个终端运行的;pts/0255代表虚拟终端，一般是远程连接的终端;tty1tty7 代表本地控制台终端
- CMD： 进程的启动命令

ps -elf

| 表头  | 含义                                                         |
| ----- | ------------------------------------------------------------ |
| F     | 进程标志，说明进程的权限，常见的标志有两个:<br/>1：进程可以被复制，但是不能被执行；<br/>4：进程使用超级用户权限； |
| S     | 进程状态。进程状态。常见的状态有以下几种：<br/>1.	-D：不可被唤醒的睡眠状态，通常用于 I/O 情况。<br/>2.	-R：该进程正在运行。<br/>3.	-S：该进程处于睡眠状态，可被唤醒。<br/>4.	-T：停止状态，可能是在后台暂停或进程处于除错状态。<br/>5.	-W：内存交互状态（从 2.6 内核开始无效）。<br/>6.	-X：死掉的进程（应该不会出现）。<br/>7.	-Z：僵尸进程。进程已经中止，但是部分程序还在内存当中。<br/>8.	-<：高优先级（以下状态在 BSD 格式中出现）。<br/>9.	-N：低优先级。<br/>10.	-L：被锁入内存。<br/>11.	-s：包含子进程。<br/>12.	-l：多线程（小写 L）。<br/>13.	-+：位于后台。 |
| UID   | 运行此进程的用户的 ID；                                      |
| PID   | 进程的 ID；                                                  |
| PPID  | 父进程的 ID；                                                |
| C     | 该进程的 CPU 使用率，单位是百分比；                          |
| PRI   | 进程的优先级，数值越小，该进程的优先级越高，越早被 CPU 执行； |
| NI    | 进程的优先级，数值越小，该进程越早被执行；                   |
| ADDR  | 该进程在内存的哪个位置；                                     |
| SZ    | 该进程占用多大内存；                                         |
| WCHAN | 该进程是否运行。"-"代表正在运行；                            |
| TTY   | 该进程由哪个终端产生；                                       |
| TIME  | 该进程占用 CPU 的运算时间，注意不是系统时间；                |
| CMD   | 产生此进程的命令名；                                         |

改变进程优先级

nice

按用户指定的优先级运行进程：nice [-n NI值] 命令

1. NI 范围是 -20~19。数值越大优先级越低
2. 普通用户调整 NI 值的范围是 0~19，而且只能调整自己的进程。
3. 普通用户只能调高 NI 值，而不能降低。如原本 NI 值为 0，则只能调整为大于 0。
4. 只有 root 用户才能设定进程 NI 值为负值，而且可以调整任何用户的进程。

renice

改变正在运行进程的优先级：renice [优先级] PID

ps -aux

- PID：进程号
- %CPU：用户可以查看某个进程占用了多少CPU
- %MEM：内存使用率
- VSZ：虚拟内存大小
- RSS：指明了当前实际占用了多少内存
- STAT：显示了进程当前的状态:
  - D：不可中断 进程
  - R：正在运行，或在队列中的进程
  - S：处于休眠状态
  - T：停止或被追踪
  - Z：僵尸进程
  - X：死掉的进程
  - <：高优先级
  - N：低优先级
  - s：包含子进程
  - +：位于后台的进程组

### top

第一行：top - 16:20:38 up 12 days,  5:24,  2 users,  load average: 0.04, 0.03, 0.05

- top：当前时间
- up：机器运行了多长时间
- users：当前登录用户数
- load average：系统负载，即任务队列的平均长度。三个数值分别为 1分钟、5分钟、15分钟前到现在的平均值。

这里具体需要关注的还是load average三个数值。先来说说定义吧：在一段时间内，CPU正在处理以及等待CPU处理的进程数之和。三个数字分别代表了1分钟，5分钟，15分钟的统计值，这个数值的确能反应服务器的负载情况。但是，这个数值高了也并不能直接代表这台机器的性能有问题，可能是因为正在进行CPU密集型的计算，也有可能是因为I/O问题导致运行队列堵了。所以，当我们看到这个数值飙升的时候，还得具体问题具体分析。大家都知道，一个CPU在一个时间片里面只能运行一个进程，CPU核数的多少直接影响到这台机器在同时间能运行的进程数。所以一般来说Load Average的数值别超过这台机器的总核数，就基本没啥问题。

第二行：Tasks: 127 total,   1 running, 126 sleeping,   0 stopped,   0 zombie

- Tasks：当前有多少进程
- running：正在运行的进程数
- sleeping：正在休眠的进程数
- stopped：停止的进程数
- zombie：僵尸进程数

这里running越多，服务器自然压力就越大。

第三行：%Cpu(s):  0.3 us,  0.7 sy,  0.0 ni, 99.0 id,  0.0 wa,  0.0 hi,  0.0si,  0.0 st

- us：用户空间占CPU的百分比（像shell程序、各种语言的编译器、各种应用、web服务器和各种桌面应用都算是运行在用户地址空间的进程，这些程序如果不是处于idle状态，那么绝大多数的CPU时间都是运行在用户态）
- sy： 内核空间占CPU的百分比（所有进程要使用的系统资源都是由Linux内核处理的，对于操作系统的设计来说，消耗在内核态的时间应该是越少越好，在实践中有一类典型的情况会使sy变大，那就是大量的IO操作，因此在调查IO相关的问题时需要着重关注它）
- ni：用户进程空间改变过优先级（ni是nice的缩写，可以通过nice值调整进程用户态的优先级，这里显示的ni表示调整过nice值的进程消耗掉的CPU时间，如果系统中没有进程被调整过nice值，那么ni就显示为0）
- id： 空闲CPU占用率
- wa： 等待输入输出的CPU时间百分比（和CPU的处理速度相比，磁盘IO操作是非常慢的，有很多这样的操作，比如，CPU在启动一个磁盘读写操作后，需要等待磁盘读写操作的结果。在磁盘读写操作完成前，CPU只能处于空闲状态。Linux系统在计算系统平均负载时会把CPU等待IO操作的时间也计算进去，所以在我们看到系统平均负载过高时，可以通过wa来判断系统的性能瓶颈是不是过多的IO操作造成的）
- hi： 硬中断占用百分比（硬中断是硬盘、网卡等硬件设备发送给CPU的中断消息，当CPU收到中断消息后需要进行适当的处理(消耗CPU时间)。）
- si：软中断占用百分比（软中断是由程序发出的中断，最终也会执行相应的处理程序，消耗CPU时间）
- st：steal time，被虚拟机占用的cpu的时间

第四行：KiB Mem : 1863012 total, 1286408 free,  216532 used, 360072 buff/cache

- total：物理内存总量
- free：空闲内存量
- used：使用的内存量
- buffer/cache：用作内核缓存的内存量

第五行：KiB Swap: 5242876 total, 7999484 free,     0 used. 1468240 avail Mem

- total：交换区内存总量
- free：空闲交换区总量
- used：使用的交换区总量
- buffer/cache：缓冲的交换区总量

第四第五行分别是内存信息和swap信息，所有程序的运行都是在内存中进行的，所以内存的性能对与服务器来说非常重要。不过当内存的free变少的时候，其实并不需要太紧张。真正需要看的是Swap中的used信息。Swap分区是由硬盘提供的交换区，当物理内存不够用的时候，操作系统才会把暂时不用的数据放到Swap中。所以当这个数值变高的时候，说明内存是真的不够用了。

进程信息：PID    USER    PR  NI  VIRT    RES   SHR   S  %CPU  %MEM     TIME+  COMMAND

- PID  	进程id
- USER	进程所有者的用户名
- PR	   	优先级
- NI		nice值，负值表示高优先级，正值表示低优先级
- VIRT	进程使用的虚拟内存总量，单位kb。VIRT=SWAP+RES
- RES		进程使用的、未被换出的物理内存大小，单位kb。RES=CODE+DATA
- SHR		共享内存大小，单位kb
- S		进程状态。D=不可中断的睡眠状态 R=运行 S=睡眠 T=跟踪/停止 Z=僵尸进程
- %CPU	上次更新到现在的CPU时间占用百分比
- %MEM	进程使用的物理内存百分比
- TIME+	进程使用的CPU时间总计，单位1/100秒
- COMMAND	命令名/命令行

默认情况下仅显示比较重要的 PID、USER、PR、NI、VIRT、RES、SHR、S、%CPU、%MEM、TIME+、COMMAND 列，还有一些参数，例如：

- PPID	父进程id
- GROUP   进程所有者的组名
- SWAP:	进程使用的虚拟内存中被换出的大小
- CODE	可执行代码占用的物理内存大小，单位kb
- DATA	可执行代码以外的部分(数据段+栈)占用的物理内存大小，单位kb
- nFLT	页面错误次数
- nDRT	最后一次写入到现在，被修改过的页面数。
- WCHAN	若该进程在睡眠，则显示睡眠中的系统函数名
- Flags	任务标志

PR(priority)进程优先级和NI(nice)优先级切换等级，都是优先级，有什么区别呢？如果，仔细观察，还会发现PR列的值是：rt或大于等于0的数字；NI列的值是：[-20,19]之间的数字，这些又代表什么意思呢？NI是代表nice的意思，是一个进程用户态的一个概念；PR代表priority优先级，是进程的实际优先级，是进程内核态的一个概念；对于一个普通任务进程来说，PR的值等于NI的值加20，即：PR=NI+20，所以，你就会发现，当进程的NI为0，PR就是20；NI为-20，PR就是0.平时启动的一个进程，如果没有特意去指定任务优先级的话，默认情况下，都是普通任务进程，NI的值为0。此外，对于一个实时任务进程来说，PR内核态优先级为rt(Realtime)，这种任务，在CPU中实时执行。

top命令常用的选项参数：

| 选项 | 功能                                                         |
| ---- | ------------------------------------------------------------ |
| -d   | 指定每两次屏幕信息刷新之间的时间间隔，如希望每秒刷新一次，则使用：top -d 1 |
| -p   | 通过指定PID来仅仅监控某个进程的状态                          |
| -S   | 指定累计模式                                                 |
| -s   | 使top命令在安全模式中运行。这将去除交互命令所带来的潜在危险  |
| -i   | 使top不显示任何闲置或者僵死的进程                            |
| -c   | 显示整个命令行而不只是显示命令名                             |

例如：

```bash
# 每隔3秒显式所有进程的资源占用情况
[root@server01 ~]# top
# 每隔1秒显式所有进程的资源占用情况
[root@server01 ~]# top -d 1
# 每隔3秒显式进程的资源占用情况，并显示进程的命令行参数(默认只有进程名)
[root@server01 ~]# top -c
# 每隔3秒显示pid是28820和pid是38830的两个进程的资源占用情况
[root@server01 ~]# top -p 28820 -p 38830
# 每隔2秒显示pid是69358的进程的资源使用情况，并显式该进程启动的命令行参数
[root@server01 ~]# top -d 2 -c -p 69358
```

top的交互命令

【1】敲top后，按键盘数字“1”可以监控每个逻辑CPU的状况：

【2】敲top后，输入u，然后输入用户名，则可以查看相应的用户进程；

【3】敲top后，top命令默认以K为单位显示内存大小，我们可以通过大写字母E来切换内存信息区域的显示单位，如下按一下E切换到MB

### date

top的交互命令显示或者修改系统时间与日期。只有超级用户才能用date命令设置时间，一般用户只能用date命令显示时间。
date命令的语法如下：
date [选项] 显示时间格式 (以+开头，后面接时间格式)

```bash
# 显示时间格式 (以+开头，后面接时间格式)
[root@server01 ~]# date  '+%Y-%m-%d'
2022-10-23
# 显示两天前的日期
[root@server01 ~]# date -d "2 days ago" +%Y-%m-%d
2022-10-21
# 显示两天后的日期
[root@server01 ~]# date -d "-2 days ago" +%Y-%m-%d
2022-10-25
# 修改日期
[root@server01 ~]# date -s 2019-08-07
2019年 08月 07日 星期三 00:00:00 CST
```

### uname

uname命令用来显示操作系统相关信息

### uptime

uptime命令用来输出系统任务队列信息

### last

列出目前与过去登入系统的用户相关信息。当执行last指令时，它会默认读取位于/var/log目录下名称为wtmp的文件，并把该给文件记录的登入系统的用户名单全部显示出来。

## 文件管理与编辑命令

### rm

功能说明：rm命令用来删除某个目录及其下的所有文件及子目录。对于链接文件，只是断开了链接，原文件保持不变。其格式如下：
rm [选项] 文件或者目录
-r	告诉rm将选项中列出的全部目录以及子目录还有文件均递归地删除，如果在选项中不指定“-r”选项，“rm”命令将不能删除目录
-f	忽略不存在的问题，也不给出提示
-i	交互式删除，即在删除前进行确认
注意：使用rm命令要特别小心，“rm –rf”组合要甚用，因为一旦文件被删除，就不能被恢复。Linux没有类似于Windows的回收站。因此，为了防止文件或者目录被误删除，可以使用rm的“-i”选项，来逐个确认要删除的文件。使用“-i”选项时，如果用户输入“y”，文件将被删除；如果输入其他任何信息，文件则不被删除。

### ln

功能说明：ln命令用来在文件或目录之间创建链接。Linux下的链接有两种，一种是硬链接（Hard Link），一种是符号链接（Symbolic Link），默认情况下ln命令产生的是硬链接。
硬链接：是指通过文件的索引节点来进行链接。
符号链接：也叫软链接，软链接类似于Windows中的快捷方式，因此软链接是一个指向真正的文件或者目录位置的符号连接。
ln命令的格式如下：
ln [选项]   源文件 	 目标链接名
-s	进行软链接（Symbolic Link）

```bash
# 创建硬链接前通过ll命令可以看到链接数目为1
[root@server01 app]# ll
总用量 4
drwxr-xr-x 3 root root 92 8月  20 2021 athn_cnmp_mgmt_service
-rw-r--r-- 1 root root  5 10月 23 18:18 source
# 通过ls -i查看源文件的inode编号
[root@server01 app]# ls -i source
51091626 source
# 通过ln命令创建source的硬链接link
[root@server01 app]# ln source link
# 通过ls -i查看链接文件的i-node编号，可知硬链接文件的inode跟源文件的inode是一致的
[root@server01 app]# ls -i link
51091626 link
# 创建硬链接后通过ll命令可以看到链接数目变为2，
# 硬链接：inode号相同、表示一个物理文件在虚拟文件系统中两个不同的path、将源文件删除后对创建的链接没有影响
[root@server01 app]# ll
总用量 8
drwxr-xr-x 3 root root 92 8月  20 2021 athn_cnmp_mgmt_service
-rw-r--r-- 2 root root  5 10月 23 18:18 link
-rw-r--r-- 2 root root  5 10月 23 18:18 source
# 软链接：inode不相同、软链接相当于windows中的一个快捷方式；删除源文件后，软链接指向就丢失了
[root@server01 app]# ln -s source soft-link
[root@server01 app]# ll
总用量 8
drwxr-xr-x 3 root root 92 8月  20 2021 athn_cnmp_mgmt_service
-rw-r--r-- 2 root root  5 10月 23 18:18 link
lrwxrwxrwx 1 root root  6 10月 23 19:51 soft-link -> source
-rw-r--r-- 2 root root  5 10月 23 18:18 source
```

### cp

功能说明：
cp命令用来将给出的文件或者目录拷贝到另一个文件或者目录中。cp与Windows下的copy命令类似，但是cp命令更加强大。其格式如下：
cp [选项] 源文件或目录 目标文件或目录
-a	在拷贝目录时使用。它保留所有的信息，包含文件链接、文件属性，并递归地拷贝目录
-r	若给出的源文件是一目录文件，此时cp将递归复制该目录下所有的子目录和文件。此时目标文件必须为一个目录名
-p	保留文件的修改时间和存取权限
-i	如果已经有相同文件名的目标文件，则提示用户是否覆盖

```bash
# 通过more ~/.bashrc可以看到，cp是在系统命令的基础上进行了alias，因此在复制文件时有同名的文件时会提示
[root@server01 app]# more ~/.bashrc
# .bashrc

# User specific aliases and functions

alias rm='rm -i'
alias cp='cp -i'
alias mv='mv -i'

# Source global definitions
if [ -f /etc/bashrc ]; then
        . /etc/bashrc
fi
export TERM=xterm
# 由于a目录和b目录下有同名的文件，因此会提示是否覆盖
[root@server01 app]# cp -r a/* b
cp：是否覆盖"b/1"？
# 如果想取消提示，则可以在cp命令前加转义字符\
[root@server01 app]# \cp -r a/* b
```

### find

功能说明
find命令用来在指定的路径下查找指定的文件。其格式如下：
find path-name  [ -options]  [-print –exec  -ok 命令 {}  \; ]
具体的选项说明如下。
path-name：find命令查找的目录路径，例如可以用“.”表示当前目录，用“/”表示系统根目录。
-options：find命令的这个选项主要用来控制搜索的方式。
-print：将搜索结果输出到标准输出。 -exec：对搜索出符合条件的文件执行所给出的Linux命令，而不询问用户是否需要执行该命令。{}表示shell命令的选项即为所查找到的文件。命令的末尾必须以“；”结束。
注意：格式要正确，“-exec 命令 {} \;”，在}和\之间一定要有空格才行。
-ok：对搜索出符合条件的文件执行所给出的Linux命令。与-exec不同的是，它会询问用户是否需要执行该命令。

举例：
查找系统中所有大小为0的普通文件，并列出它们的完整路径。
find / -type f -size 0 -exec  ls -al {} \;
查找系统/var/logs目录中修改时间在7天以前的普通文件，然后以交互方式删除。
find /var/log -type f -mtime  +7 -ok  rm {} \;
在系统根目录下，查找文件类型为普通文件，属于ixdba用户的，并且查找时不包含/usr/bin目录的文件名为iptables.sh的文件，并将结果输出到屏幕。
find / -path "/usr/bin" -prune -o -name "iptables.sh"  -user ixdba -type f -print
在系统根目录下查找不在/var/log和/usr/bin目录下的文件名为main.c的文件。
find /  \( -path /var/log -o -path /usr/bin \) -prune -o -name “main.c” -print 

### file

file命令用来显示文件的类型。对于长度为0的文件，将识别为空文件；对于符号连接文件，缺省情况下将显示符号连接引用的真实文件路径。

```bash
[root@server01 app]# file /opt/char.c     
/opt/char.c: C source, UTF-8 Unicode text
```

### stat

stat能更详细的查看文件状态信息

```bash
[root@server01 app]# stat /opt/char.c 
  文件："/opt/char.c"
  大小：436             块：8          IO 块：4096   普通文件
设备：802h/2050d        Inode：37792116    硬链接：1
权限：(0644/-rw-r--r--)  Uid：(    0/    root)   Gid：(    0/    root)
最近访问：2022-10-23 20:14:43.250300854 +0800
最近更改：2022-06-20 14:23:11.301141490 +0800
最近改动：2022-06-20 14:23:11.313141624 +0800
创建时间：-
```

### grep

grep命令是Linux下的文本过滤工具，grep根据指定的字符串，对文件的每一行进行搜索，如果找到了这个字符串，就输出该行的内容。其格式如下：
grep [选项] 需要查找的字符串 文件名
grep命令的选项有很多，这里列出最常使用的选项说明：
-c	只显示符合条件的行数，而不是显示被匹配到的内容。
-i	搜索时忽略大小写
-n	在显示的搜索结果上显示行号
-E    	支持扩展的正则表达式
-w	被匹配的文本只能是单词，而不能是单词中的某一部分。
-v ：	反过来（invert），只打印没有匹配的，而匹配的反而不打印。

```bash
[root@server01 ~]# grep -ni network anaconda-ks.cfg
16:# Network information
17:network  --bootproto=dhcp --device=enp0s3 --ipv6=auto --activate
18:network  --hostname=localhost.localdomain
```

grep家族总共有三个：grep，egrep，fgrep。
grep：标准grep命令，支持基本正则表达式
egrep：扩展grep命令，支持基本和扩展正则表达式，等价于grep –E
fgrep：快速grep命令，不支持正则表达式，按照字符串的字面意思进行匹配，等价于grep –F

### diff

diff命令用来比较文件的差异。diff以逐行的方式比较文本文件的异同，其格式如下：
diff [选项]  文件1  文件2
上面的命令执行后，会将比较后的不同之处以指定的形式列出，如下所示：
n1 a n3,n4 
n1,n2 d n3 
n1,n2 c n3,n4 
其中，字母"a"、"d"、"c"分别表示添加、删除及修改操作。而"n1"、"n2"表示在文件1中的行号，"n3"、"n4"表示在文件2中的行号。
在输出形式中，每一行后面将跟随受到影响的若干行。其中，以<开始的行属于文件1，以>开始的行属于文件2。

```bash
[root@server01 app]# cat a
hello
hi
hadoop flink
scala
spark
hudi
[root@server01 app]# cat b
hello
hi,
hadoop flink
scala
spark
# 2c2表示第一个文件中的第二行跟第二个文件中的第二行不一致
# < hi：表示第一个文件的第二行为hi
# > hi,：表示第二个文件的第二行为hi,
# 6d5表示第一个文件中第六行在第二个文件中不存在
[root@server01 app]# diff a b
2c2
< hi
---
> hi,
6d5
< hudi
```

## 压缩与解压缩命令

### mv

mv命令用来将文件或目录改名或将文件由一个目录移入另一个目录中。如果源类型和目标类型都是文件或者目录时，mv将进行目录重命名。如果源类型为文件，而目标类型为目录时，mv将进行文件的移动。如果源类型为目录，则目标类型只能是目录，不能是文件，此时完成目录重命名。其格式如下：
mv [选项] 源文件或目录  目标文件或目录
mv命令的选项及其说明：
-i	交互式操作，对已经存在的文件或目录覆盖时，系统会询问是否覆盖，用户输入“y”进行覆盖，输入“n”则不覆盖
-f	force 强制的意思，如果目标文件已经存在，不会询问而直接覆盖
-b	若需覆盖文件，则覆盖前先行备份。

### gzip/gunzip

将一般的文件进行压缩或者解压。压缩文件预设的扩展名为“.gz”，其实gunzip就是gzip的硬链接，因此无论是压缩或者解压都可以通过gzip来实现。注意：gzip只能对文件进行压缩，不能压缩目录，即使指定压缩的目录，也只能压缩目录内的所有文件。
格式如下：
gzip [选项] 压缩（解压缩）的文档名
-d	对压缩的文件进行解压
-t	检查压缩文档的完整性
-l	显示压缩文件的压缩信息，显示字段为压缩文档大小、未压缩文档大小、压缩比和未压缩文档名称

### bzip2/bunzip2

对文件进行压缩与解压缩。此命令类似于“gzip/gunzip”命令，只能对文件进行压缩。对于目录只能压缩目录下的所有文件，压缩完成后，在目录下生成以“.bz2”为后缀的压缩包。bunzip2其实是bzip2的符号链接，即软链接，因此压缩解压都可以通过bzip2实现。其格式如下：
bzip2 [选项] 要压缩或解压的文件
-d	执行解压缩，此时选项后面跟要解压缩的文件
-k	bzip2在压缩或解压缩后，会删除原始的文件，若要保留原始文件，可使用此选项
-t	测试“.bz2”压缩文件的完整性

### tar

tar是Linux下经常使用的归档工具，是对文件或者目录进行打包归档，归成一个文件，但是并不进行压缩。其格式如下：
tar [主选项＋辅助选项] 文件或者目录
tar命令的选项很多，这里列出一些经常用到的主选项
-c	创建新的文件
-t	列出档案文件中已经归档的文件列表
-x	从打包的档案文件中还原出文件
-z	调用gzip命令在文件打包的过程中进行压缩/解压文件
-j	调用bzip2命令在文件打包的过程中进行压缩/解压文件
-f	“-f”选项后面紧跟档案文件的存储设备，默认是磁盘，需要指定档案文件名；如果是磁带，只需指定磁带设备名即可。注意，在“-f”选项之后不能再跟任何其他选项，也就是说“-f”必须是tar命令的最后一个选项
-v	指定在创建归档文件过程中，显示各个归档文件的名称
-p	在文件归档的过程中，保持文件的属性不发生变化
--exclude file	在打包过程中，不将指定file文件打包

```bash
# 压缩
[root@server01 app]# tar -czvf athn_cnmp_mgmt_service.tar.gz athn_cnmp_mgmt_service
# 解压
[root@server01 app]# tar -xzvf athn_cnmp_mgmt_service.tar.gz -C /app/
# 仅解开athn_cnmp_mgmt_service.tar.gz压缩文件中的/athn_cnmp_mgmt_service/deploy.sh文件
[root@server01 app]# tar -xzvf athn_cnmp_mgmt_service.tar.gz athn_cnmp_mgmt_service/deploy.sh
# 或者
[root@server01 app]# tar -xzvf athn_cnmp_mgmt_service.tar.gz -C /app athn_cnmp_mgmt_service/deploy.sh
# 将/etc目录打包压缩后直接解压到/opt目录下，而不生成打包的档案文件。
[root@server01 ~]# tar -zcvf - /etc | tar -zxvf -  -C /opt
```

###  zip/unzip

将一般的文件或者目录进行压缩或者解压，默认生成以“.zip”为后缀的压缩包。zip命令类似于Windows中的winzip压缩程序。其格式如下：
zip [选项] 压缩文件名 需要压缩的文档列表
unzip [选项] 压缩文件名
-r		递归压缩，将指定目录下的所有文件以及子目录全部压缩
-d		从压缩文件内删除指定的文件
-x   “文件列表”	压缩时排除文件列表中指定的文件
-u		更新文件到压缩文件中

## 磁盘管理与维护命令

### df

功能说明
df命令用来检查Linux系统的磁盘空间占用情况。其格式如下：
df [选项]
选项含义
-h	以容易理解的格式输出文件系统分区占用情况，例如32kB、120MB、60GB
-k	以kB大小为单位输出文件系统分区占用情况
-m	以MB大小为单位输出文件系统分区占用情况
-i	列出文件系统分区的inodes信息
-T	显示磁盘分区的文件系统类型

### du

du命令用来显示文件或目录所占用的磁盘空间情况。其格式如下：
du [选项] 文件或目录
-sh	以人性化的格式显示文件或者目录大小，例如300MB、1.2GB等
-ks	以MB为单位显示文件或者目录大小

### fsck

fsck命令用来检查文件系统并尝试修复错误。其格式如下：
fsck  [-t <文件系统类型>] [设备名]
“-t <文件系统类型>”是指定要检查的文件系统类型。一般情况下可不添加此选项。
注意：在执行fsck命令修复某个文件系统时，这个文件系统对应的磁盘分区一定要处于卸载状态，磁盘分区在挂载状态下进行修复是极为不安全的，数据可能遭到破坏，也有可能损坏磁盘。

### mount/umount

挂载以及卸载指定的文件系统。
mount [选项] [-L<标签>] [-o<选项>] [-t<文件系统类型>] [设备名] [挂载点]
umount [挂载点]
选项:
-r	以只读方式加载设备
-w	以可读写模式加载设备，属于mount默认设置
-a	加载文件/etc/fstab中指定的所有设备
-L<标签>：标签其实就是磁盘分区标识的别名，标签可以随便起名，这样便于记忆，在Linux下磁盘分区的设备名比较难记，利用标签代替设备名，简单易记。
-o<选项>：指定加载文件系统时的选项， ro：以只读模式加载。rw：以可读写模式加载。remount,rw：以读写模式重新挂载
-t<文件系统类型>：指定设备的文件系统类型，常见文件系统类型有xfs/ext4/ext3/ext2/vfat/nfs等。
设备名：硬盘分区在Linux上的设备标识，类似于/dev/sda1、/dev/hda2等。
挂载点：Linux系统下指定的某个目录。

## 网络设置与维护命令

### ifconfig

ifconfig命令用来配置网络或显示当前网络接口状态。类似于Windows下的ipconfig命令，同时ifconfig命令必须以root用户来执行。其格式如下：
ifconfig  [选项] [interface] [inet|up|down|netmask|addr|broadcast]
在网卡enp0s3上配置两个IP地址，分别为192.168.60.136、192.168.66.138，子网掩码为255.255.255.0，使用以下命令：

```bash
[root@server01 ~]# ifconfig enp0s3 192.168.60.136 netmask 255.255.255.0
[root@server01 ~]# ifconfig enp0s3:0 192.168.66.138 netmask 255.255.255.0
# 修改网卡的MAC地址为新的MAC地址，使用以下命令：
[root@server01 ~]#ifconfig enp0s3 hw ether xx:xx:xx:xx:xx:xx
# 将网卡enp0s3禁用后再启用，使用以下命令：
[root@server01 ~]# ifconfig enp0s3 down
[root@server01 ~]# ifconfig enp0s3 up
```

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

# 附录

>**设置SecureCRT编码及字体**
>
>1. 进入Options->Global Options->Default Session->Edit Default Settings...
>
>2. 选择Terminal->Appearance
>
>   Current color scheme：White / Black
>
>   Normal font：黑体 14pt
>
>   Character encoding：UTF-8
>
>**设置SecureCRT使vim代码显示彩色**
>
>1. Options->Global Options->Default Session->Edit Default Settings...->Terminal->Emulation
>
>   Terminal选择Linux，然后勾选ANSI Color和Use color scheme
>
>2. 在/root/.bashrc中添加：export TERM=xterm
>
>3. 最后关闭CRT，再重新打开连接一下
