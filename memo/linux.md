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

1. N (N>=3) 块盘组成阵列，一份数据产生N-1个条带，同时还有1份校验数据,共N份数据在N块盘上循环均衡存储
2. N块盘同时读写，读性能很高，但由于有校验机制的问题，写性能相对不高；
3. (N-1) /N磁盘利用率（有一块是用来校验的）；
4. 可靠性高，允许坏1块盘，不影响所有数据。

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

进入/etc/yum.repos.d目录，新建配置文件Redhat.repo

```bash
[root@server01 ~]# cd /etc/yum.repos.d/
[root@server01 yum.repos.d] mv CentOS-Base.repo CentOS-Base.repo_bak
[root@server01 yum.repos.d]# vim Redhat.repo
[Redhat]
name=Redhat
baseurl=file:///media
enabled=1
gpgcheck=0
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
启动：/etc/init.d/XXXdaemon start
关闭：/etc/init.d/XXXdaemon stop
重启：/etc/init.d/XXXdaemon restart
查看状态：/etc/init.d/XXXdaemon status

```bash
# /etc/init.d/目录下存放的是所有的启动脚本
[root@server01 init.d]# ll /etc/init.d/
总用量 44
-rwxr-xr-x  1 root root 10835 10月  6 2020 clickhouse-server
-rw-r--r--. 1 root root 15131 9月  12 2016 functions
-rwxr-xr-x. 1 root root  2989 9月  12 2016 netconsole
-rwxr-xr-x. 1 root root  6643 9月  12 2016 network
-rw-r--r--. 1 root root  1160 11月  7 2016 README
# 比如：查看clickhouse-server的状态
[root@server01 init.d]# /etc/init.d/clickhouse-server status
clickhouse-server service is running
# 具体怎么启动哪些服务要结合下面的系统运行等级来看
```

init机制的系统运行级
init可以根据使用者自订的执行等级(runlevel)来唤醒不同的服务，以进入不同的操作模式。基本上 Linux 提供7个执行等级，分别是0, 1, 2...6 ，
比较重要的是：
init 0：关机模式
init 1：单人维护模式、
init 3：纯字符模式、
init 5：图形界面模式。
init 6：重启模式

init管理机制服务设置方式
各个执行等级的启动脚本是通过 /etc/rc.d/rc[0-6]/SXXdaemon 链接到 /etc/init.d/daemon 。即：管理在各个系统运行等级中需要启动的服务
链接文件名 (SXXdaemon) 的功能为： S 为启动该服务，XX是数字，表示启动的顺序。由于有SXX的设定，因此在开机时可以依序执行所有需要的服务。

```bash
# 比如在init 3这个运行等级下，会依次执行/etc/init.d/目录下的network、clickhouse-server脚本
# 注：K50netconsole中的K代表的含义未知
[root@server01 init.d]# ll /etc/rc.d/rc3.d/
总用量 0
lrwxrwxrwx. 1 root root 20 7月  31 2017 K50netconsole -> ../init.d/netconsole
lrwxrwxrwx. 1 root root 17 7月  31 2017 S10network -> ../init.d/network
lrwxrwxrwx  1 root root 27 11月 11 15:45 S50clickhouse-server -> ../init.d/clickhouse-server
```

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

systemd提供了一个非常强大的命令行工具systemctl，可能很多系统运维人员都已经非常熟悉基于sysvinit的服务管理方式，比如service、chkconfig命令，而systemd也能完成同样的管理任务，可以把systemctl看作是service和chkconfig的组合体。要查看、启动、停止、重启、启用或者禁用系统服务，都可以通过systemctl命令来实现 。Systemd使用单元(Units)来管理系统服务和程序。系统单元使用配置文件来控制其相关操作。单元配置文件有三种类型：默认单元配置文件，系统特定的单元配置文件和运行时的单元配置文件。三种类型的单元配置文件所在路径： 

1. 默认单元配置文件：/usr/lib/systemd/system；当安装新软件包时，在安装过程中，单元配置文件会在该目录中生成

2. 运行时的配置文件 /run/systemd/system；分别在units启动和停止时，会自动生成和删除。

3. 系统特定的配置文件 /etc/systemd/system；包含定制的单元配置。通过这些配置文件，用户可以覆盖units的默认行为。

当对系统服务和程序的状态进行任何更改时，例如：start,stop,enable,和disable时，systemd读取并执行其单元配置文件。按照以下顺序检查单元配置文件。系统特定的单元配置文件、运行时单元配置文件、默认单元配置文件。例如，如果一个units配置文件在着三个路径下面都存在，则仅使用系统特定的配置文件：/etc/systemd/system。

对于multi-user.target来说，需要把/lib/systemd/system/multi-user.target.wants下面的service和/etc/systemd/system/multi-user.target.wants下面的service都执行了。这两个目录是叠加的关系。/etc/systemd/system/multi-user.target.wants/，这里的服务是用户自定义的systemd的开机启动项。目录/usr/lib/systemd/system/multi-user.target.wants/，是系统开机启用的systemd。如果想disable那些位于/etc/systemd/system/multi-user.target.wants下面的服务，只需要执行 systemctl disable xx.service即可，执行的结果是删除xx.service 在/etc/systemd/system/multi-user.target.wants下面的软连接。disable之后，如果想enable这个服务，需要执行systemctl enable xx.service。如果想disable那些位于/lib/systemd/system/multi-user.target.wants下面的服务，需要执行 systemctl mask yy.service即可，执行的结果是在/etc/systemd/system目录下面创建一个软连接，yy.service -----> /dev/null，mask之后，如果想重新生成这个服务，需要执行systemctl unmask yy.service

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
# 查看systemd的默认target：
[root@server01 system]# systemctl get-default     
multi-user.target
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
ps命令显示系统进程在瞬间的运行动态，CMD栏的内容被中括号括起来的进程一般都是系统进行，其格式如下：
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

```bash
# 查看某个进程的启动时间
[root@server01 limits.d]# ps -eo pid,lstart,etime | grep 542
  542 Thu Oct 27 13:14:23 2022    01:34:26
# 使用ps查看JAVA进程使用的实际内存和虚拟内存：
[root@server01 limits.d]# ps -p ${pid} -o rss,vsz  
RSS     VSZ
7152568 17485844
```

获取占用CPU资源最多的10个进程

ps aux|sort -nr -k3

参数解析

- sort #排序命令 
- -nr #默认使用字符串排序n代表使用数值进行排序 默认从小到大排序 r代表反向排序 
- -k3 #以第3列进行排序

这里会有一个问题，就是：第一行：head也参与的排序

![image](assets\linux-65.png)

把输入第一行删除，然后剩余的行参与排序并取前10位

ps aux|grep -v PID|sort -nr -k3|head -n10

![image](assets\linux-66.png)

grep参数解析

-v 反向查找，比如 grep -v "grep" 就是查找不含有 grep 字段的行; 

如需要显示PID，则先运行输出第一行然后再进行排序

ps aux|head -n1;ps aux|grep -v PID|sort -nr -k3|head -n10

![image](assets\linux-67.png)

同理输出内存占用多的进程，内存参数在第四行

ps aux|head -n1;ps aux|grep -v PID|sort -nr -k4|head -n10

![image](assets\linux-68.png)

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

【4】敲top后，输入f，可以定制显示的列。通过光标移动到想要显示的列，然后按d或者Space，然后按q退出。

【5】敲top后，然后按下大写M按照内存MEM排序，按下大写P按照CPU排序。

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
# 在linux中可以通过":角标"的方式给一块网卡绑定多个ip地址
[root@server01 ~]# ifconfig enp0s3:0 192.168.66.138 netmask 255.255.255.0
# 修改网卡的MAC地址为新的MAC地址，使用以下命令：
[root@server01 ~]#ifconfig enp0s3 hw ether xx:xx:xx:xx:xx:xx
# 将网卡enp0s3禁用后再启用，使用以下命令：
[root@server01 ~]# ifconfig enp0s3 down
[root@server01 ~]# ifconfig enp0s3 up
```

### ip

```bash
# 设置IP地址，可以使用下列ip命令：
[root@server01 ~]# ip addr add 192.168.100.193/24 dev enp0s3
# 查看IP地址：
[root@server01 ~]#ip addr show enp0s3
# 删除IP地址，只需用del代替add
[root@server01 ~]#ip addr del 192.168.100.193/24 dev enp0s3
# 列出路由表条目
[root@server01 ~]#ip route show
# 查看路由包来自的接口
[root@server01 ~]#ip route get 172.16.213.51
# 激活网络接口
[root@server01 ~]# ip link set eth0 up
# 停止网络接口
[root@server01 ~]# ip link set eth0 down
# 监控netlink消息
[root@server01 ~]# ip monitor all
# 显示网络统计信息
[root@server01 ~]# ip -s link
# 设置默认网关
[root@server01 ~]# ip route add default via 192.168.1.254
```

### scp

scp就是secure copy，用于将文件或者目录从一个Linux系统拷贝到另一个Linux系统下。scp传输数据用的是SSH协议，保证了数据传输的安全。其格式如下：
scp  远程用户名@ip地址:文件的绝对路径 本地Linux系统路径
scp  本地Linux系统文件路径 远程用户名@ip地址:远程系统文件绝对路径名

```bash
[root@server01 ~]#scp /home/ixdba/etc.tar.gz root@192.168.60.168:/tmp
[root@server01 ~]#scp root@192.168.60.133:/home/ixdba/etc.tar.gz  /tmp
[root@server01 ~]#scp –r  /etc  root@192.168.60.135:/opt
```

### traceroute

traceroute [选项] [远程主机名或者IP地址] [数据包大小]
-i <网络接口>	使用指定的网络接口发送数据包
-w<超时秒数>	设置等待远程主机回应的时间
-s<来源ip>	设置本地主机发送数据包的IP地址
traceroute是利用ICMP连接的，有些网络设备（比如防火墙）可能会屏蔽ICMP通过的权限，因此也会出现节点没有回应的状态，如果在指定的时间内（这里我们设置的是10秒），traceroute检测不到某个路由节点的回应信息，就在屏幕输出“*”，表示此节点无法通过。

```bash
[root@server01 ~]# traceroute www.sina.com
traceroute to www.sina.com (101.89.125.239), 30 hops max, 60 byte packets
 1  10.0.2.2 (10.0.2.2)  0.162 ms  0.104 ms  0.076 ms
 2  * * *
 3  * * *
```

### mtr

![image](assets\linux-8.png)

mtr是 Linux中有一个非常棒的网络连通性判断工具，它结合了ping, traceroute,nslookup 的相关特性.
Loss%列就是对应IP行的丢包率了，值得一提的是，只有最后的目标丢包才算是真正的丢包
Last列则是最后一次返回的延迟，按毫秒计算的
Avg列是所有返回时延的一个平均值
Best列是最快的一次返回时延
Wrst列是最长的一次返回时延
StDev列是标准偏差。

### wget

wget命令用来从网络上下载某个软件，这个命令对于能够连接到互联网的Linux系统作用非常大，可以直接从网络下载需要的软件。其格式如下：
wget [要下载软件的网址]

```bash
# wget -c断点续传
[root@server01 ~]# wget -c http://cn.wordpress.org/wordpress-3.1-zh_CN.zip
# 使用wget -O下载并以不同的文件名保存
[root@server01 ~]# wget -O wordpress.zip http://www.centos.bz/download.php?id=1080 
# 使用wget –limit -rate限速下载
[root@server01 ~]# wget –limit-rate=300k http://cn.wordpress.org/wordpress-3.1-zh_CN.zip 
# 使用wget -b后台下载 
[root@server01 ~]# wget -b http://cn.wordpress.org/wordpress-3.1-zh_CN.zip
# 下载https
[root@server01 ~]# wget --no-cookie --no-check-certificate https://pypi.python.org/source/s/setuptools/setuptools-0.6c11.tar.gz
```

### telnet

telnet命令通过telnet协议与远程的主机通信或者获取远程主机对应端口的信息。与Windows下的telnet完成相同的功能。其格式如下：
telnet  主机名或者IP地址 端口

```bash
# 如果ip后面不跟端口的话，默认连接23端口
[root@localhost ~]# telnet 192.168.60.88
# 查看某台Linux系统的22和80端口是否打开以及分别开启了什么服务，使用以下命令
[root@localhost ~]# telnet 192.168.60.88 22
[root@localhost ~]# telnet www.ixdba.net 80
# 查看某台Linux系统的22和80端口是否打开以及分别开启了什么服务，使用以下命令：
[root@localhost ~]# telnet 192.168.60.88 22
Trying 192.168.60.88...
Connected to 192.168.60.88.
Escape character is '^]'.
SSH-2.0-OpenSSH_6.6.1
# 从这里可以看出，在“192.168.60.88”的22端口运行着SSH服务，对应的SSH版本为SSH-2.0-OpenSSH_6.6.1。
```

### netstat

netstat命令是一个监控TCP/IP网络的非常有用的工具，它可以显示路由表、实际的网络连接以及每一个网络接口设备的状态信息。
-a或--all：显示所有连线中的Socket（所有状态）； 
-A<网络类型>或--<网络类型>：列出该网络类型连线中的相关地址； 
-c或--continuous：持续列出网络状态； 
-C或--cache：显示路由器配置的快取信息； 
-e或--extend：显示网络其他相关信息； 
-F或--fib：显示FIB； 
-g或--groups：显示多重广播功能群组组员名单； 
-h或--help：在线帮助； 
-i或--interfaces：显示网络界面信息表单； 
-l或--listening：显示监控中的服务器的Socket； 
-M或--masquerade：显示伪装的网络连线； 
-n或--numeric：直接使用ip地址，而不通过域名服务器； 
-N或--netlink或--symbolic：显示网络硬件外围设备的符号连接名称； 
-o或--timers：显示计时器； 
-p或--programs：显示正在使用Socket的程序识别码和程序名称； 
-r或--route：显示Routing Table； 
-s或--statistice：显示网络工作信息统计表； 
-t或--tcp：显示TCP传输协议的连线状况； 
-u或--udp：显示UDP传输协议的连线状况； 
-v或--verbose：显示指令执行过程； 
-V或--version：显示版本信息； 
-w或--raw：显示RAW传输协议的连线状况； 
-x或--unix：此参数的效果和指定"-A unix"参数相同； 
--ip或--inet：此参数的效果和指定"-A inet"参数相同。

### ss

ss是Socker-Statistics的缩写，是一款非常适用、快速、跟踪显示的网络套接字的新工具。它和netstat显示的内容类似，但它比netstat更加强大。当服务器的socket连接数量变得非常大时，无论是使用netstat命令还是直接cat/proc/net/tcp，执行速度都会很慢。而用ss可以快速、有效的执行并得到结果。ss利用到了TCP协议栈中tcp_diag。tcp_diag是一个用于分析统计的模块，可以获得Linux内核中第一手的信息，这就确保了ss的快捷高效。当然，如果你的系统中没有tcp_diag，ss也可以正常运行，只是效率会变得稍慢。yum install iproute iproute-doc；语法格式 ss [OPTION]... [FILTER]
常用选项
-t: tcp协议相关；
-u: udp协议相关；
-w: 裸套接字相关；
-x： unix sock相关；
-l: listen状态的连接；
-a: 显示所有sockets信息；
-n: 数字格式；
-p: 相关的程序及PID；
-e: 扩展的信息；
-m：内存用量；
-o：计时器信息；
-s：显示当前sockets的统计信息的摘要；
-i：显示系统内部tcp连接；
-r：解析主机名；
-4：仅显示IPv4的sockets连接；
-6：仅显示IPv6的sockets连接；

## 文本编辑工具

### vi

#### 三种模式

1、命令行模式：用户在用vi编辑文件时，最初进入的为该模式。可以进行复制、粘贴等操作。
2、插入模式：命令行模式下按i键就会进入编辑模式，此模式下，敲入的都是文件内容。编辑完成之后，按Esc键退出插入模式，回到命令行模式；
3、底行模式：命令行模式下按：就会进入底行模式。可以进行文件的保存、退出、查找、替换、列出行号等 

#### 模式切换命令

| 命令       | 功能                                                         |
| ---------- | ------------------------------------------------------------ |
| i I        | Insert<br>i:进入编辑状态，从当前光标之前的位置开始插入键盘输入的字符<br>I:进入编辑状态，鼠标所在行的行首位置开始插入键盘输入的字符 |
| a A        | Append<br>a:进入编辑状态，从当前光标之后的位置开始插入键盘输入的字符<br>A:进入编辑状态，鼠标所在行的行尾位置开始插入键盘输入的字符 |
| o O        | Open<br>o:进入编辑状态，从光标所在行的下面插入一新行，光标移到该新行的行首，以后键盘输入的字符将插入到光标位置<br>O:进入编辑状态，从光标所在行的上面插入一新行，光标移到该新行的行首，以后键盘输入的字符将插入到光标位置 |
| ESC        | 进入命令状态                                                 |
| :! Command | 在vi中执行外部命令Command，按回车键可以返回vi继续工作        |

#### 拷贝与粘贴命令

| 命令    | 功能                                                         |
| ------- | ------------------------------------------------------------ |
| [N]x    | (Expurgate)删除从光标位置开始的连续N个字符（并复制到编辑缓冲区） |
| [N]dd   | (Delete)删除从光标位置开始的连续N行（并复制到编辑缓冲区）    |
| [N]yy   | (Yank)复制从光标位置开始的连续N行到编辑缓冲区                |
| p或P    | (Put)从编辑缓冲区复制文本到当前光标位置（即粘贴）            |
| y0      | 将光标至行首的字符拷入剪贴板                                 |
| y$      | 将光标至行尾的字符拷入剪贴板                                 |
| d0      | 将光标至行首的字符剪切入剪贴板                               |
| d$      | 将光标至行尾的字符剪切入剪贴板                               |
| range y | 块复制<br>比如：输入:.,$y再按回车，表示复制从当前行到末行的内容。 |
| range d | 块剪切<br>比如：输入:2,5d再按回车，表示删除从第二行到第五行内容。 |
| ctrl+v  | 进入块选择模式，选择完成后，按y复制，按p粘贴                 |
| shift+v | 进入行选择模式，选择完成后，按y复制，按p粘贴                 |
| u       | (Undo)取消上一次操作（即恢复功能）                           |
| Ctrl+r  | 恢复上一步被撤销的操作                                       |
| gg=G    | 自动调格式                                                   |
| ggdG    | 全部删除                                                     |

#### 保存和退出命令

| 命令    | 功能                                                         |
| ------- | ------------------------------------------------------------ |
| :q      | (Quit)退出没有修改的文件（若文件被修改了而没有保存，则此命令无效） |
| :q!     | 强制退出，且不保存修改过的部分                               |
| :w      | (Write)保存文件，但不退出                                    |
| :x      | (Exit)保存文件并退出                                         |
| :w File | 另存为File给出的文件名，不退出                               |
| :r File | (Read) 读入File指定的文件内容插入到光标位置                  |

#### 光标命令

| 命令          | 功能                                            |
| ------------- | ----------------------------------------------- |
| h             | 方向键，向左移动光标一个字符的位置，相当于键“←” |
| j             | 方向键，向下移动光标到下一行的位置，相当于键“↓” |
| k             | 方向键，向上移动光标到上一行的位置，相当于键“↑” |
| l             | 方向键，向右移动光标一个字符的位置，相当于键“→” |
| :N            | 移动光标到第N行（N待定）                        |
| gg/1G         | 移动光标到文件的第1行                           |
| G             | 移动光标到文件的最后1行                         |
| :set number   | 设置显示行号                                    |
| :set nonumber | 取消显示行号                                    |

#### 查找命令

| 命令      | 功能                                                         |
| --------- | ------------------------------------------------------------ |
| /[string] | 查找命令<br>n 继续查找 <br>N 反向继续查找 <br>支持正则表达式比如：/^the    /end$ |

#### 替换命令

利用:s 命令可以实现字符串的替换。

| 命令               | 功能                                                         |
| ------------------ | ------------------------------------------------------------ |
| :s/str1/str2/      | 替换当前行第一个str1                                         |
| :s/str1/str2/g     | 替换当前行的所有str1                                         |
| :.,$ s/str1/str2/g | 替换当前行到最后一行中每一行的str1<br>在每次替换前提示确认则可以加参数'c'例如：:.,$ s/str1/str2/gc |
| :1,$ s/str1/str2/g | 替换第一行到最后一行中每一行的str1                           |
| :%s/str1/str2/g    | 替换第一行到最后一行中每一行的str1                           |

#### vim列编辑模式

添加一列：
1）vim 打开文件，并移动光标到要添加列的起始行
2）按下ctrl+v，打开visual模式
3）通过光标向下选中你要添加内容的位置
4）按下I（即shift＋i）键，然后输入你要插入的内容
5）按下ESC键，大概1s后，你就能看到内容加上了
删除一列：
1）vim打开文件，并移动光标到要删除列所在的启示位置
2）按下ctrl＋v，进入visual模式
3）通过移动光标，选中你要删除的区域
4）按下d键完成删除

#### Vim分屏模式

分屏启动Vim
使用大写的O参数来垂直分屏。
vim -On file1 file2 ...
使用小写的o参数来水平分屏。
vim -on file1 file2 ...
注释: n是数字，表示分成几个屏。

分屏
上下分割，并打开一个新的文件。
:sp filename
左右分割，并打开一个新的文件。
:vsp filename

移动光标
把光标移到右边的屏。
Ctrl+w l
把光标移到左边的屏中。
Ctrl+w h
把光标移到上边的屏中。
Ctrl+w k
把光标移到下边的屏中。
Ctrl+w j
把光标移到下一个的屏中。.
Ctrl+w w  注：也可以按住cntl键，同时按下两次w键

# Linux下软件的安装与管理

## 源码安装

源码安装方式的优点
由于Linux操作系统开放源代码，因而在其上安装的软件大部分也都是开源软件，例如apache、tomcat、php等软件。开源软件基本都提供源码下载，源码安装的方式；源码安装的好处是用户可以自己定制软件功能，安装需要的模块，不需要的功能可以不用安装，此外，用户还可以自己选择安装路径，方便管理，卸载软件也很方便，只需删除对应的安装目录即可。

源码方式编译安装过程
源码安装软件一般有以下几个步骤组成：下载解压源码、分析安装平台环境（configure）、编译安装软件（make、make install）。
configure文件一般是个可执行文件，可以在当前目录下直接输入“./configure”进行软件安装的环境测试，如果提示缺少某些安装包，就需要进行安装，直到测试通过。通常的，源码安装都需要GCC或者CC编译器，这些编译器一般在安装系统时定制安装包中的开发工具选项下。
make是我们经常用到的编译命令，对于一个包含很多源文件的应用程序，使用make和makefile工具可以简单快速的解决各个源文件之间复杂的依赖关系，同时，make工具还可以自动完成所有源码文件的编译工作，并且可以只对上次编译后修改过的文件进行增量编译。
make工具最主要的功能就是通过makefile文件来实现的，makefile文件是按照某种语法来进行编写的，文件中定义了各个源文件之间的依赖关系，并说明了如何编译源文件并生成可执行文件，它通过描述各个源程序之间的关系让make工具自动完成编译工作。

源码包安装注意事项
通过源码方式安装软件，需要把开发工具等基础模块安装好，比如 gcc、gcc-c++、libgcc、glibc、make、automake等开发工具或基础包；还要安装一些相应的开发包，一般是文件名包括dev的，比如glibc-devel、gettext-devel、；还有一些开发库，比如以lib开头的开发库。
源代码一般以file.tar.gz file.tar.bz2打包，file.tar.gz和file.tar.bz2格式的解包命令如下；
[root@localhost ~]# tar jxvf file.tar.bz2
[root@localhost ~]# tar zxvf file.tar.gz
解开一个包后，进入解压包，一般都能发现README（或reame)和INSTALL( 或install)；或doc（或DOC)目录。这些都是安装说明。
configure 比较重要的一个参数是 --prefix ，用--prefix 参数，我们可以指定软件安装目录；当我们不需要这个软件时，直接删除软件的目录就行了。

## RPM包方式安装

### RPM包管理工具介绍

RPM是Red Hat Package Manager的缩写，本意就是Redhat软件包管理，是最先由Redhat公司开发出来的linux下软件包管理工具，由于这种软件管理方式非常方便，逐渐被其它linux发行商所借用，现在已经成为linux平台下通用的软件包管理方式，例如Fedora 、Redhat、suse等主流linux发行版本都默认采用了这种软件包管理方式。
以“.rpm”结尾的软件包，就是RPM文件。每个RPM文件中包含了已经编译好的二进制可执行文件，其实就是将软件源码文件进行编译安装，然后进行封装，就成了RPM文件。例如：gcc-4.8.5-16.el7.x86_64.rpm
RPM包管理方式的优点是：安装简单方便，因为软件已经编译完成打包完毕，安装只是个验证环境和解压的过程。
RPM包管理方式的缺点是对操作系统环境的依赖很大，它要求RPM包的安装环境必须与RPM包封装时的环境相一致或相当。还需要满足安装时与系统某些软件包的依赖关系，例如需要安装A软件，但是A软件需要系统有B和C软件的支持，那么就必须先安装B和C软件，然后才能安装A软件。

### RPM包种类和命令

RPM包的封装格式一般有两种，分别是RPM和SRPM，SRPM包也是一种RPM，但是它包含了编译时的源码文件和一些编译指定的参数文件，因而在使用的时候需要重新进行编译，通常SRPM对应的RPM文件类似与“xxxxxxxx.src.rpm”格式。
下面介绍两种RPM包对应的文件名含义，以rpm文件：openssh-7.4p1-11.el7.x86_64.rpm为例，其中：
“openssh”表示软件的名称；
“7.4p1”表示软件的版本号；
“11”表示软件更新发行的次数；
“el7”表示适用的操作系统平台
“x86_64”表示适合的硬件平台；
“.rpm”是rpm软件包的标识。
一般的RPM封装包的命名格式都有这六个部分组成，由于SRPM包是需要编译才能使用的，因此没有上面显示项中对应的平台选项，其它与RPM包命令格式完全一样。

### RPM工具的使用

#### 安装软件包

命令格式：rpm -i [辅助选项] file1.rpm file2.rpm…..fileN.rpm  
主选项含义如下：
-i：install的意思，就是安装软件。也可以使用“--install”。
参数说明：file1.rpm file2.rpm…..filen.rpm是指定将要安装RPM包的文件名，可以多个文件一起安装。
辅助选项含义如下（选项很多，我们只列出常用选项）：
-v：显示附加信息。
-h：安装时输出标记“#”。
--test：只对安装进行测试，并不实际安装。
--nodeps：不检查软件之间的依赖关系。加入此选项可能会导致软件不可用。
--force：忽略软件包以及软件冲突。

#### 查询软件包

命令格式：rpm -q [辅助选项] package1……packageN
常用选项含义如下：
-q：query的意思，也可以使用“--query”。
辅助选项含义如下：
-a：列出所有已安装的包
-f：查询操作系统中某个文件属于哪个对应的rpm软件包。
-p：查询以“.rpm”为后缀的软件包安装后对应的包名称
-l：显示软件包中的所有文件列表。此选项后面跟软件包安装后对应的包名，切记不是以“.rpm”为后缀的rpm包。
-i：显示软件包的概要信息，例如软件名称、版本、适应平台、大小等等。此选项后面跟完整的包名，切忌不是以“.rpm”为后缀的rpm包。

```bash
# 查询是否安装某个软件
[root@localhost 1]# rpm -qa | grep vim
vim-filesystem-7.4.160-1.el7.x86_64
vim-enhanced-7.4.160-1.el7.x86_64
vim-common-7.4.160-1.el7.x86_64
vim-minimal-7.4.160-1.el7.x86_64

# 查询命令属于哪个软件
[root@localhost 1]# which passwd 
/usr/bin/passwd
[root@localhost 1]# rpm -qf /usr/bin/passwd
passwd-0.79-4.el7.x86_64

# 查看通过yum（或者yum源）安装的软件所在的目录
# 如果是直接通过yum install安装的软件，用如下方式查找
[root@localhost ~]# rpm -ql python
```

#### 更新软件包

命令格式：rpm -U [辅助选项] file1.rpm……fileN.rpm
主选项含义如下：
-U：upgrade的意思，可以使用“--upgrade”代替。
参数说明：file1.rpm……fileN.rpm表示需要升级的rpm文件包。

#### 删除软件包

命令格式：rpm -e [辅助选项] package1……packageN
主选项含义如下：
-e：erase的意思，也可以用“--erase”代替。
参数说明：package1……packageN表示已经安装的软件包名称。
辅助选项含义如下：
--test：只执行删除的测试。
--nodeps：不检查依赖性。

#### 验证未安装的软件包文件

发行的RPM格式的软件包是否值得信任，是否损坏，我们可以通过RPM提供的选项进行验证。RPM软件包一般使用 Gnu 隐私卫士（或称 GPG）来签名，从而帮助使用者肯定下载软件包的可信任性。
命令格式：rpm -K file1.rpm……fileN.rpm
主选项含义如下：
-K： checksig的意思，也可以用“--checksig”代替。这个选项用来检查 RPM 软件包文件的md5校验和GPG签名。

#### 安装xxxxxx.src.rpm的方法

这里以my-package.src.rpm名称为例，在Centos7.4的 x86_64系统平台下进行介绍src.rpm软件包基本安装方法，步骤如下： 
1）、执行rpm -i my-package.src.rpm 命令
2）、切换目录：cd /root/rpmbuild/SPECS（在Centos5.x以及之前版本的路径是/usr/src/redhat/SPECS，从Centos6.x以及之后版本路径变为/root/rpmbuild/SPECS）
3）、执行rpmbuild 操作
rpmbuild  -bb my-package.specs （my-package.specs是一个和软件包同名的specs文件）。
此时，在/root/rpmbuild/RPMS/x86_64这个目录下，有一个或者多个已经生成的rpm包，这个就是已经编译好的可执行rpm文件。 
4）、执行安装命令：rpm -ivh new-package.rpm。

## yum方式在线安装软件

yum是yellowdog updater modified 的缩写，yellow dog（黄狗）也是一个 Linux 的 发行版本，只不过Redhat公司是将这种升级技术利用到自己的发行版上就形成了现在的 yum。 yum是进行linux系统下软件安装和升级常用的一个工具，通过yum工具配合互联网即可实现软件的便捷安装和自动升级。

### yum的安装与配置

以Centos7.x为例，检查yum是否已经安装，执行如下命令：
[root@localhost ~]# rpm -qa|grep yum
如果没有任何显示，表示系统中还没有安装yum工具，yum安装包在Centos系统光盘中可以找到，执行如下指令进行安装：
[root@localhost ~]# rpm -ivh yum-*.noarch.rpm
安装yum需要python-elementtree、python-sqlite、urlgrabber、yumconf等软件包的支持，这些软件包在Centos Linux系统安装光盘均可找到，如果在安装yum过程中出现软件包之间的依赖性，只需按照依赖提示寻找相应软件包安装即可，直到yum包安装成功。

### yum的配置

yum工具安装完毕，接下来的工作是进行yum的配置，yum的配置文件有主配置文件/etc/yum.conf、资源库配置目录/etc/yum.repos.d，yum安装后，默认的一些资源库配置可能无法使用，可能需要修改。

### yum的特点

安装方便，自动解决增加或删除rpm包时遇到的依赖性问题。 
可以同时配置多个资源库（Repository）
配置文件简单明了（/etc/yum.conf、/etc/yum.repos.d/CentOS-Base.repo）
保持与RPM数据库的一致性
注意：yum会自动下载所有所需的升级资源包并默认放置在/var/cache/yum目录下，当第一次使用yum或yum资源库更新时，软件升级所需的时间可能较长。

### yum的基本用法

通过yum安装和删除RPM包
安装rpm包,如dhcp。命令如下：
[root@localhost ~]#yum install curl
删除rpm包,包括与该包有依赖性的包，命令如下：
[root@localhost ~]#yum remove gettext-devel
注意:同时会提示删除intltool
检查可更新的rpm包，命令如下：
[root@localhost ~]#yum check-update
更新所有的rpm包，命令如下：
[root@localhost ~]#yum update
列出资源库中特定的可以安装或更新以及已经安装的rpm包的信息，命令如下：
[root@localhost ~]#yum info openssh
[root@localhost ~]#yum info perl*
列出资源库中特定的可以安装或更新以及已经安装的rpm包，命令如下：
[root@localhost ~]#yum list sendmail
[root@localhost ~]#yum list gcc*
搜索匹配特定字符的rpm包的详细信息，命令如下：
[root@localhost ~]#yum search wget
清除缓存中旧的rpm头文件和包文件，命令如下：
[root@localhost ~]#yum clean 或
[root@localhost ~]#yum clean all

### yum只下载软件不安装的两种方法

通过yum自带一个工具：yumdownloader
yumdownloader  gcc
使用yum的一个插件：yum-downloadonly
yum -y install --downloadonly --downloaddir=/tmp  httpd
--downloadonly 说明只下载
--downloaddir  指定安装到哪个目录下

### yum加速插件yum-fastestmirror

yum-fastestmirror插件可以自动选择速度最快的mirror
配置文件:
/etc/yum/pluginconf.d/fastestmirror.conf
其中，yum镜像的速度测试记录文件
/var/cache/yum/timedhosts.txt
安装加速插件：
yum  install yum-plugin-fastestmirror

### 更换系统默认yum源为阿里云yum源

备份原来的yum源
mv /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.backup
下载阿里云的yum
wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo
清理缓存
yum clean all
生成新的缓存
yum makecache

### 几个不错的yum源

#### EPEL源

EPEL，全称是企业版Linux附加软件包，是一个由特别兴趣小组创建、维护并管理的，针对红帽企业版Linux(RHEL)及其衍生发行版(例如CentOS、Scientific Linux)的一个高质量附加软件包项目。 其官方网址为：http://fedoraproject.org/wiki/EPEL/zh-cn， EPEL的软件包不会与企业版Linux官方源中的软件包发生冲突，或者互相替换文件。
相关的EPEL软件包可以从EPEL官方网站下载到，针对Centos系统，有EL5、EL6、EL7三个版本，分别针对Centos5.x、Centos6.x、Centos7.x三个系列版本

#### rpmforge源

rpmforge是一个第三方的软件源仓库，也是CentOS官方社区推荐的第三方yum源，它为CentOS系统提供了超过10000个软件包，被CentOS社区认为是最安全也是最稳定的一个软件仓库。但是由于这个安装源不是CentOS本身的组成部分，因此要使用rpmforge，需要手动下载并安装。
rpmforge的官方网站是http://repoforge.org/ ，可以在http://repoforge.org/use/ 下载RHEL/Centos各个版本的“rpmforge-release”包，这样就可以使用RPMForge提高的丰富软件了。

## 二进制包安装方式

Linux下二进制格式的软件是指事先已经在各种平台编译安装好相关软件，然后压缩打包，在安装时只需解压或者执行安装可执行文件即可。二进制软件包的优点是安装简单、容易，缺点是缺乏灵活性，相应的软件包执行在对应平台下安装，离开这个环境软件就无法运行。
二进制软件包提供了很多类型的打包方式，最常见的就是RPM格式的包，还有以“*.tar.gz、*.tgz、*.bz2“等形式的二进制软件包。
这种格式的软件包，安装其实就是简单的解压过程，根据不同的软件打包格式，用对应的解压命令解压即可。
对于*.tar.gz软件格式，解压命令如下：
tar -zxvf xxxxxx.tar.gz
对于*.bz2软件格式，解压命令如下： 
tar  -jxvf	xxxxxx.tar.bz2
常见的jdk安装包、tomcat安装包等，都输入二进制包的安装方式。

# Linux用户权限管理与文件权限管理

## 用户权限管理与文件权限管理

### 用户与角色分类

超级用户：拥有对系统的最高管理权限，默认是root用户。
普通用户：只能对自己目录下的文件进行访问和修改，具有登录系统的权限，例如www用户、ftp用户等。
虚拟用户：也叫“伪”用户，这类用户最大的特点是不能登录系统，它们的存在主要是方便系统管理，满足相应的系统进程对文件属主的要求。例如系统默认的bin、adm、nobody用户等

### 用户和组以及关系

一对一：即一个用户可以存在一个组中，也可以是组中的唯一成员。
一对多：即一个用户可以存在多个用户组中。那么此用户具有多个组的共同权限。
多对一：多个用户可以存在一个组中，这些用户具有和组相同的权限。
多对多：多个用户可以存在多个组中。其实就是上面三个对应关系的扩展。

### 用户和组相关的配置文件

/etc/passwd文件：系统用户配置文件，是用户管理中最重要的一个文件。这个文件记录了Linux系统中每个用户的一些基本属性，并且对所有用户可读。

```bash
[root@server01 ~]# vim /etc/passwd
# 用户名：密码：UID（用户ID）：GID（组ID）：描述性信息：主目录：默认Shell
# root用户的UID和GID都是0，如果要将一个普通用户改成一个root用户只需要将普通用的UID和GID都改成0即可
# useradd -u 0 -o -g 0 user
# -u：手工指定用户的 UID，注意 UID 的范围（不要小于 500）。
# -g：GID
# -o：允许创建的用户的 UID 相同。例如，执行 "useradd -u 0 -o usertest" 命令建立用户 usertest，它的 UID 和 root 用户的 UID 相同，都是 0；
root:x:0:0:root:/root:/bin/bash
# 默认shell为：/sbin/nologin的用户是虚拟用户
bin:x:1:1:bin:/bin:/sbin/nologin
daemon:x:2:2:daemon:/sbin:/sbin/nologin
adm:x:3:4:adm:/var/adm:/sbin/nologin
lp:x:4:7:lp:/var/spool/lpd:/sbin/nologin
```

/etc/shadow文件：用户影子文件，由于/etc/passwd文件是所有用户都可读的，这样就导致了用户的密码容易出现泄露，因此，linux将用户的密码信息从/etc/passwd中分离出来，单独的放到了一个文件中，这个文件就是/etc/shadow，该文件只有root用户拥有读权限。

```bash
[root@server01 ~]# vim /etc/shadow
# 如果第二个字段为：*、!!等特殊字符的话，说明该用户是不能登录系统的虚拟用户
root:$6$Z1/.wLxU$I3/6siTdlBoNtJi/QpS6tpobdpTXbI3DYrwQKlY5jyTpCIqfkvPU2rq4CPnV3FGu/LnG7NIBT38e8XocPmDLL.:19188:0:99999:7:::
bin:*:17110:0:99999:7:::
daemon:*:17110:0:99999:7:::
dbus:!!:17378::::::
polkitd:!!:17378::::::
tss:!!:17378::::::
sshd:!!:17378::::::
postfix:!!:17378::::::
chrony:!!:17378::::::
```

/etc/group文件：用户组配置文件，用户组的所有信息都存放在此文件中。

```bash
[root@server01 ~]# vim /etc/group
# 组名:口令:组标识号:组内用户列表
# 组内用户列表：是属于这个组的所有用户的列表，不同用户之间用逗号(,)分隔。这个用户组可能是用户的主组，也可能是附加组。此字段列出每个群组包含的所有用户。需要注意的是，如果该用户组是这个用户的初始组，则该用户不会写入这个字段，可以这么理解，该字段显示的用户都是这个用户组的附加用户。
root:x:0:
bin:x:1:
daemon:x:2:
sys:x:3:
adm:x:4:
tty:x:5:
disk:x:6:
lp:x:7:
mem:x:8:
kmem:x:9:
wheel:x:10:admin
cdrom:x:11:
mail:x:12:postfix
```

/etc/login.defs文件：用来定义创建一个用户时的默认设置，比如指定用户的UID和GID的范围，用户的过期时间、是否需要创建用户主目录等等。

```bash
[root@server01 ~]# vim /etc/login.defs
# 创建用户时，要在目录/var/spool/mail中创建一个用户mail文件
MAIL_DIR        /var/spool/mail

#密码最大有效期99999   
PASS_MAX_DAYS   99999
#两次修改密码的最小间隔时间0
PASS_MIN_DAYS   0
#密码最小长度，对于root无效
PASS_MIN_LEN    5
#密码过期前多少天开始提示
PASS_WARN_AGE   7

#创建用户时不指定UID的话自动UID的范围
UID_MIN                  1000
#用户ID的最小值
UID_MAX                 60000
#系统uid的最小值
SYS_UID_MIN               201
#系统uid的最大值
SYS_UID_MAX               999

#用户组GID的最小值
GID_MIN                  1000
#用户组GID的最大值
GID_MAX                 60000
#系统用户组GID的最小值
SYS_GID_MIN               201
#系统用户组GID的最大值
SYS_GID_MAX               999

#允许使用useradd的时候创建用户家目录
CREATE_HOME     yes
#权限掩码设置为077（默认为022，注释里面有这方面介绍）
UMASK           077
#使用命令 userdel 删除用户时，是否删除用户的初始组，默认是删除
USERGROUPS_ENAB yes
#指定 Linux 用户的密码使用 SHA512 散列模式加密，这是新的密码加密模式，原先的 Linux只能用 DES 或 MD5 方式加密
ENCRYPT_METHOD SHA512
```

/etc/default/useradd文件：定义了新建用户的一些默认属性，比如用户的主目录、使用的shell等等，通过更改此文件，可以改变创建新用户的默认属性值。

```bash
[root@server01 ~]# vim /etc/default/useradd
# 默认用户组
GROUP=100
# 用户主目录
HOME=/home
# 帐号是否过期	
INACTIVE=-1
# 帐号终止日期
EXPIRE=
# 默认使用哪个shell
SHELL=/bin/bash
# 模板目录，骨架目录
SKEL=/etc/skel
# 是否创建邮箱文件
CREATE_MAIL_SPOOL=yes
```

/etc/skel文件：目录定义了新建用户在主目录下默认的配置文件，更改/etc/skel目录下的内容就可以改变新建用户默认主目录的配置文件信息。

### 添加、切换、删除用户组命令

#### groupadd

用来新建一个用户组。语法格式为：
groupadd -g gid  groupname
-g：指定新建用户组的GID号，该GID号必须唯一，不能和其它用户组的GID号重复。

#### newgrp

如果一个用户同时属于多个用户组，那么用户可以在用户组之间切换，以便具有其他用户组的权限，newgrp主要用于在多个用户组之间进行切换。

#### groupdel

表示删除用户组，语法格式为：
groupdel [群组名称]
当需要从系统上删除用户组时，可用groupdel指令来完成这项工作。如果该用户组中仍包括某些用户，则必须先删除这些用户后，然后才能删除用户组。

### 添加、修改和删除用户命令

#### useradd

useradd的使用语法
useradd  [-u uid [-o]] [-g group] [-G group,...]
                [-d home] [-s shell] [-c comment]
                [-f inactive] [-e expire ] name
各个选项具体含义如下：

1.  -u uid：即用户标识号，此标识号必须唯一。
2.  -g group：指定新建用户登录时所属的默认组，或者叫主组。此群组必须已经存在。
3.  -G group：指定新建用户的附加组，此群组必须已经存在。附加组是相对与主组而言的，当一个用户同时是多个组中的成员时，登录时的默认组成为主组，而其它组称为附加组。
4.  -d home：指定新建用户的默认主目录，如果不指定，系统会在/etc/default/useradd文件指定的目录下创建用户主目录。
5.  -s shell：指定新建用户使用的默认shell，如果不指定，系统以/etc/default/useradd文件中定义的shell作为新建用户的默认shell。

```bash
[root@server01 opt]# groupadd appuser
[root@server01 opt]# useradd -g appuser appuser
```

#### usermod

usermod的使用语法
usermod用来修改用户的账户属性信息，使用语法如下：
usermod  [-u uid [-o]] [-g group] [-G group,...]
                [-d 主目录 ] [-s shell] [-L|-U] Name
各个选项具体含义如下：

1.  -u uid：指定用户新的UID值，此值必须为唯一的ID值，除非用-o选项。
2.  -g group：修改用户所属的组名为新的用户组名，此用户组名必须已经存在。
3.  -G group：修改用户所属的附加组。
4.  -d 主目录：修改用户登录时的主目录。
5.  -s shell：修改用户登录系统后默认使用的shell
6.  -L：锁定用户密码，使密码无效。
7.  -U：解除密码锁定。

#### userdel

userdel的使用语法
Userdel用来删除一个用户，若指定“-r”参数不但删除用户，同时删除用户的主目录以及目录下的所有文件。语法格式为：
`userdel [-r][用户帐号]`

### 用户及组信息查询

用于显示用户的ID，以及所属群组的ID

1.语法：

id [参数] [用户名称]

2.功能：

显示用户以及所属群组的实际与有效ID。若两个ID相同，则仅显示实际ID。若仅指定用户名称，则显示目前用户的ID。

3.参数：

1. -g或--group 　显示用户所属群组的ID。 
2. -G或--groups 　显示用户所属附加群组的ID。 
3. -n或--name 　显示用户，所属群组或附加群组的名称。 
4. -r或--real 　显示实际ID。 
5. -u或--user 　显示用户ID。

```bash
# 查看指定用户的组信息
[root@server01 ~]# id admin
uid=1000(admin) gid=1000(admin) 组=1000(admin),10(wheel)
# 查看目前系统中所有的用户及组信息
[root@server01 ~]# vipw
root:x:0:0:root:/root:/bin/bash
bin:x:1:1:bin:/bin:/sbin/nologin
daemon:x:2:2:daemon:/sbin:/sbin/nologin
adm:x:3:4:adm:/var/adm:/sbin/nologin
lp:x:4:7:lp:/var/spool/lpd:/sbin/nologin
sync:x:5:0:sync:/sbin:/bin/sync
shutdown:x:6:0:shutdown:/sbin:/sbin/shutdown
halt:x:7:0:halt:/sbin:/sbin/halt
mail:x:8:12:mail:/var/spool/mail:/sbin/nologin
operator:x:11:0:operator:/root:/sbin/nologin
games:x:12:100:games:/usr/games:/sbin/nologin
ftp:x:14:50:FTP User:/var/ftp:/sbin/nologin
nobody:x:99:99:Nobody:/:/sbin/nologin
```

### 文件权限属性解读

在Linux中常见的有7种文件类型：
普通文件（-表示）
目录（d表示）
字符设备文件（c）
块设备文件（b）
套接字文件（s）
管道（p）
符号链接文件（l）

>字符设备、块设备
>
>在LINUX里面，设备类型分为：字符设备、块设备以及网络设备， PCI是一种和ISA为一类的总线结构，归属于网络驱动设备
>字符设备、块设备主要区别是：在对字符设备发出读/写请求时，实际的硬件I/O一般就紧接着发生了，而块设备则不然，它利用一块系统内存作为缓冲区，当用户进程对设备请求能满足用户的要求时，就返回请求的数据，如果不能就调用请求函数来进行实际的I/O操作，因此，块设备主要是针对磁盘等慢速设备设计的，以免消耗过多的CPU时间来等待
>系统中能够随机（不需要按顺序）访问固定大小数据片（chunks）的设备被称作块设备，这些数据片就称作块。最常见的块设备是硬盘，除此以外，还有软盘驱动器、CD-ROM驱动器和闪存等等许多其他块设备。注意，它们都是以安装文件系统的方式使用的——这也是块设备的一般访问方式。
>另一种基本的设备类型是字符设备。字符设备按照字符流的方式被有序访问，像串口和键盘就都属于字符设备。如果一个硬件设备是以字符流的方式被访问的话，那就应该将它归于字符设备；反过来，如果一个设备是随机（无序的）访问的，那么它就属于块设备。
>这两种类型的设备的根本区别在于它们是否可以被随机访问——换句话说就是，能否在访问设备时随意地从一个位置跳转到另一个位置。举个例子，键盘这种设备提供的就是一个数据流，当你敲入"fox" 这个字符串时，键盘驱动程序会按照和输入完全相同的顺序返回这个由三个字符组成的数据流。如果让键盘驱动程序打乱顺序来读字符串，或读取其他字符，都是没有意义的。所以键盘就是一种典型的字符设备，它提供的就是用户从键盘输入的字符流。对键盘进行读操作会得到一个字符流，首先是"f"，然后是"o"，最后是"x"，最终是文件的结束（EOF）。当没人敲键盘时，字符流就是空的。硬盘设备的情况就不大一样了。硬盘设备的驱动可能要求读取磁盘上任意块的内容，然后又转去读取别的块的内容，而被读取的块在磁盘上位置不一定要连续，所以说硬盘可以被随机访问，而不是以流的方式被访问，显然它是一个块设备。

通过“ls –al”可以显示文件或者目录的权限信息，看下面的输出：
[root@localhost oracle]# ls -al
drwxr-xr--   3   oracle oinstall   4096  Oct 30  2008   oradata 

![image](assets\linux-9.png)

第一列显示文档类型与执行权限，有十个字符组成，分为4个部分，下面将文档oradata权限分解

![image](assets\linux-10.png)

文档类型部分：
当为“d”时，表示目录；当为“l”时表示软链接；当为“-”时表示文件；当为“c”时表示串行端口字符设备文件；当为“b”时表示可供存储的块设备文件。由此可知，oradata是一个目录。
在接下来的三个部分中，三个字符为一组，每个字符的含义为：“r”表示只读，即read；“w”表示可写，即write；“x”表示可执行，即execute；“-”表示无此权限，即为空。
User部分：
第二部分是对文档所有者（user）权限的设定，“rwx”表示用户对oradata目录有读、写和执行的所有权限。
Group部分：
第三部分是对文档所属用户组（group）权限的设定，“r-x”表示用户组对oradata目录有读和执行的权限，但是没有写的权限。
Others部分：
第四部分是对文档拥有者之外的其它用户权限的设定，“r--”表示其它用户或用户组对oradata目录只有读的权限。

[root@localhost oracle]# ls -al
drwxr-xr--   3 oracle oinstall  4096 Oct 30  2008  oradata 	
第二列显示的是文档的链接数，这个链接数就是硬链接的概念，即多少个文件指向同一个索引节点
第三列显示了文档所属的用户和用户组，也就是文档是属于哪个用户以及哪个用户组所有。
第四列显示的是文档的大小，默认显示的是以bytes为单位，但是也可以通过命令的参数修改显示的单位，例如可以通过“ls -sh”组合人性化的显示文档的大小。对于目录，通常只显示文件系统默认block的大小。
第五列显示文档最后一次的修改日期，通常以月、日、时、分的方式显示，如果文档修改时间距离现在已经很远了，会使用月、日、年的方式显示。
第六列显示的是文档名称，linux下以“.”开头的文件是隐藏文件，同理以“.”开头的目录是隐藏目录，隐藏文档只有通过ls命令的“-a”选项才能显示。

### 利用chown改变属主和属组

chown就是change owner的意思，主要作用就是改变文件或者目录的所有者，而所有者包含用户和用户组，其实chown就是对文件所属的用户和用户组进行的一系列设置。
chown使用的一般语法为：
[root@localhost ~]#chown [-R] 用户名称 文件或目录
[root@localhost ~]#chown [-R] 用户名称:用户组组名称 文件或目录
参数说明：
-R : 进行递归式的权限更改，也就是将目录下的所有文件、子目录都更新成为指定的用户组权限。常常用于变更某一目录的情况。

### 利用chmod改变访问权限

chmod用于改变文件或目录的访问权限。该命令有两种用法。一种是包含字母和操作符表达式的字符设定法；另一种是包含数字的数字设定法。

#### 字符设定法

使用语法为：
chmod [who] [+ | - | =] [mode] 文件名
who表示操作对象，可以是下面字母中的任何一个或者它们的组合。

1. u 表示“用户（user）”，即文件或目录的所有者。
2. g 表示“用户组（group）”，即文件或目录所属的用户组。
3. o 表示“其他（others）用户”。 
4. a 表示“所有（all）用户”。它是系统默认值。

操作符号含义如下：

1. “+”表示添加某个权限。
2. “-”表示取消某个权限。
3. “=”表示赋予给定的权限，同时取消文档以前的所有权限。

mode表示可以执行的权限，可以是“r“（只读）、“w”（可写）和“x”（可执行），以及它们的组合。

```bash
[root@server01 var]# chmod u=rwx,g+w,o+x log.log
```

#### 数字设定法

首先了解一下用数字表示属性的含义：
0表示没有任何权限
1表示有可执行权限，与上面字符表示法中的“x”有相同的含义。
2表示有可写权限，与“w”对应
4表示有可读权限，对应与“r“
如果想让文件的属主拥有读和写的权限，可以通过4（可读）+2（可写）=6（可读可写）的方式来实现，那么用数字6就表示拥有读写权限。

使用语法：
chmod [属主权限的数字组合] [用户组权限的数字组合] [其它用户权限的数字组合]  文件名

```bash
[root@server01 var]# chmod 644 log.log
```

### Linux用户与环境变量

#### 常见系统环境变量

PATH、PWD、BASH、LANG、USER、HOSTNAME、HOME、SHELL

#### 定义环境变量

ENVIRON-VARIABLE=value       #环境变量赋值
export ENVIRON-VARIABLE      #声明环境变量；环境变量赋值后只有声明完才能生效
注意：环境变量可以在命令行中设置，但用户注销时这些值将丢失，环境变量均为大写，必须用export命令导出。

#### 清除环境变量

unset 环境变量名

#### 显示环境变量内容

env命令可以列出已经定义的环境变量
echo命令
echo $环境变量名

#### 环境变量与配置文件 

bash相关的配置文件主要分为全局配置文件和个人配置文件

全局配置文件：

1. /etc/profile：用来设置系统环境参数，比如$PATH. 这里面的环境变量是对系统内所有用户生效的。

2. /etc/profile.d/*.sh：全部用户、任何方式登录都有效

3. /etc/bashrc：这个文件设置系统bash shell相关的东西，对系统内所有用户生效。只要用户运行bash命令，那么这里面的东西就在起作用。

个人配置文件

1. ~/.bash_profile：用来设置一些环境变量，功能和/etc/profile 类似，但是这个是针对用户来设定的，也就是说，你在/home/user1/.bash_profile 中设定了环境变量，那么这个环境变量只针对 user1 这个用户生效；~/.bash_profile 是交互式、login 方式进入 bash 运行的，意思是只有用户登录时才会生效。

2. ~/.bashrc：作用类似于/etc/bashrc, 只是~/.bashrc只针对当前用户生效，不对其他用户生效。~/.bashrc 是交互式 non-login 方式进入 bash 运行的，用户不一定登录，只要以该用户身份运行命令行就会读取该文件。

| 概念        | 举例                                                         | 举例                                                         |
| ----------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 登陆shell   | 用户登陆时，输入用户名和密码登陆的shell，或bash --login命令打开的shell | /etc/profile->/etc/profile.d/*.sh->~/.bash_profile->~/.bashrc->/etc/bashrc |
| 非登陆shell | 用户登陆时不需要系统认证打开的shell，或bash命令打开的shell   | ~/.bashrc->/etc/bashrc->/etc/profile.d/*.sh（测试下来，虽然不执行profile，但是会继承上一个shell的全部变量） |
| 交互shell   | 提供命令提示符等待用户输入命令的是交互shell模式              | 用户登录后，在终端上输入命令，shell 立即执行用户提交的命令。当用户退出后，shell 也终止了。 |
| 非交互shell | 直接运行脚本文件是非交互shell模式                            | 即 shell 与用户不存在交互，而是以 shell script的方式执行的。 |

命令bash启动shell时，是可以通过选项改变其行为的，(sh 基本兼容bash这些设定)

```bash
bash [长选项] [选项] [脚本]
```

常用选项：

| 选项           | 含义                                                         |
| -------------- | ------------------------------------------------------------ |
| -i             | shell在交互模式下运行                                        |
| -l             | shell作为登陆shell                                           |
| -r             | 启动受限shell                                                |
| --             | 选项结束标志，后面的内容当做文件名或参数，即使他们以-开头    |
| --login        | 同-l                                                         |
| --noprofile    | 阻止读取初始化文件/etc/profile、~/.bash_profile、~/.bash_login、~/.profile |
| --norc         | 在交互式shell，阻止读取初始化文件~/.bashrc。如果shell以sh调用的话，该选项默认是打开的。 |
| --recfile file | 在交互式shell，指定初始化文件是file而不是~/.bashrc           |
| --version      | 版本信息                                                     |

查看登录式：shopt login_shell

```bash
# 新起一个bash，为非登录shell
[root@server01 opt]# bash
[root@server01 opt]# shopt login_shell            
login_shell     off
# 指定--login参数，新起一个bash，为登录shell
[root@server01 opt]# bash --login
[root@server01 opt]# shopt login_shell
login_shell     on
# 新起一个bash：
[root@server01 opt]# ssh server01
Last login: Thu Apr  6 19:08:09 2023
[root@server01 ~]# shopt login_shell
login_shell     on
```

查看交互式法1：echo $PS1 # 非空则表示交互式

```bash
[root@server01 ~]# echo $PS1 
[\u@\h \W]\$
[root@server01 ~]# cat test.sh
echo $PS1
shopt login_shell
[root@server01 ~]# sh test.sh 

login_shell     off
```

交互/非交互/登录/非登录shell这些组合情况，在脚本的调用方面有区别；而且对于bash与sh也存在差异；

下面以bash为例：

示例1：

```bash
# 交互式的登录shell和非交互式shell，首先会继承上一个shell的所有变量，然后会按下面这个流程执行：
# /etc/profile->/etc/profile.d/*.sh->~/.bash_profile->~/.bashrc->/etc/bashrc
# 所有可以看到 /etc/profile文件末尾设置的环境变量被加载了两遍。（非交互式的登录效果一样）
[root@server01 opt]# bash -il test.sh
/usr/local/rvm/gems/ruby-2.6.0/bin:/usr/local/rvm/gems/ruby-2.6.0@global/bin:/usr/local/rvm/rubies/ruby-2.6.0/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/usr/local/rvm/bin:/usr/local/java/jdk1.8.0_144/bin:/usr/local/zookeeper/bin:/usr/local/hadoop-2.7.5/bin:/usr/local/hadoop-2.7.5/sbin:/usr/local/hive/bin:/usr/local/sqoop/bin:/usr/local/flume/bin:/usr/local/hbase-2.0.0/bin:/usr/local/flink-1.6.1/bin:/usr/local/node/bin:/root/bin:/usr/local/java/jdk1.8.0_144/bin:/usr/local/zookeeper/bin:/usr/local/hadoop-2.7.5/bin:/usr/local/hadoop-2.7.5/sbin:/usr/local/hive/bin:/usr/local/sqoop/bin:/usr/local/flume/bin:/usr/local/hbase-2.0.0/bin:/usr/local/flink-1.6.1/bin:/usr/local/node/bin:/root/bin
# 交互式的非登录shell和非交互式的非登录shell，同样会继承上一个shell的所有变量，然后会按下面这个流程执行：
# ~/.bashrc->/etc/bashrc->/etc/profile.d/*.sh
# 因为非登录shell不会执行/etc/profile，所以/etc/profile文件末尾设置的环境变量没有被重复加载。（非交互式的非登录效果一样）
[root@server01 opt]# bash -i test.sh 
/usr/local/rvm/gems/ruby-2.6.0/bin:/usr/local/rvm/gems/ruby-2.6.0@global/bin:/usr/local/rvm/rubies/ruby-2.6.0/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/usr/local/rvm/bin:/usr/local/java/jdk1.8.0_144/bin:/usr/local/zookeeper/bin:/usr/local/hadoop-2.7.5/bin:/usr/local/hadoop-2.7.5/sbin:/usr/local/hive/bin:/usr/local/sqoop/bin:/usr/local/flume/bin:/usr/local/hbase-2.0.0/bin:/usr/local/flink-1.6.1/bin:/usr/local/node/bin:/root/bin
```

示例2：

在server02的/root/.bashrc文件中增加一个环境变量：

```bash
# .bashrc

# User specific aliases and functions

alias rm='rm -i'
alias cp='cp -i'
alias mv='mv -i'

# Source global definitions
if [ -f /etc/bashrc ]; then
        . /etc/bashrc
fi
export PATH=$PATH:/usr/local/hadoop-2.7.5/bin
```

在server02的/opt目录下，创建如下shell脚本

```bash
[root@localhost opt]# cat test.sh 
echo $PATH
```

在server01下执行如下命令：

```bash
# 由于是通过ssh直接连接server02后直接通过bash打开shell，所以没有父shell，此外通过bash打开shell默认是非登录式的所以会按照：~/.bashrc->/etc/bashrc->/etc/profile.d/*.sh这个流程执行，因此/etc/profile和~/.bash_profile是不会被执行的。
[root@server01 opt]# ssh server02 "bash /opt/test.sh"
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/usr/local/hadoop-2.7.5/bin
# 如果通过bash -l打开shell(shell作为登陆shell)，则会按照/etc/profile->/etc/profile.d/*.sh->~/.bash_profile->~/.bashrc->/etc/bashrc的流程执行，那么/etc/profile里面配置的环境变量就会生效
[root@server01 opt]# ssh server02 "bash -l /opt/test.sh"
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/usr/local/hadoop-2.7.5/bin:/usr/local/java/jdk1.8.0_144/bin:/usr/local/zookeeper/bin:/usr/local/hadoop-2.7.5/bin:/usr/local/hadoop-2.7.5/sbin:/usr/local/flume/bin:/usr/local/hbase-2.0.0/bin:/usr/local/flink-1.6.1/bin:/usr/local/hadoop-2.7.5/bin:/root/bin
[root@server01 opt]# 
```



![image](assets\linux-11.png)

#### 添加用户环境变量

#此.bash_profile是elasticsearch用户下的环境变量配置文件
[elasticsearch@localhost ~]$ cat .bash_profile
export JAVA_HOME=/usr/local/jdk1.8.0_04
export PATH=$PATH:$JAVA_HOME/bin: $ANT_HOME/bin: $HOME/bin
配置文件环境变量文件后，执行source命令马上生效。
[elasticsearch@localhost ~]$source .bash_profile

### sudo的使用

sudo命令的配置文件为/etc/sudoers，编辑这个文件有个单独的命令visudo(这个文件最好不要使用vim命令来打开)，因为一旦语法写错会造成严重的后果，这个工具会替你检查你写的语法，这个文件的语法遵循以下格式：who where whom command

/etc/sudoers文件默认给 root 用户定义了一条规则：
root    ALL=(ALL)       ALL

root　　　　表示 root 用户。
ALL　　 　　表示从任何的主机上都可以执行，也可以这样 192.168.100.0/24。
(ALL) 　         是以谁的身份来执行，ALL就代表可以以任何人的身份来执行命令。
ALL 　　　　表示任何命令。
那么整条规则就是 root用户可以在任何主机以任何人的身份来执行所有的命令。

iivey    192.168.10.0/24=(root)    /usr/sbin/useradd
上面的配置只允许iivey在 192.168.10.0/24 网段上连接主机并且以root权限执行useradd 命令。

www    ALL=(root)    NOPASSWD:ALL,!/usr/bin/passwd [A-Za-z]*,!/usr/bin/passwd root,!/bin/su
允许www用户执行所有命令，除了passwd后加任意字符、passwd root和su这三类操作。

%wheel    ALL=(ALL)    ALL
给一个组设置权限，上面的例子是给wheel组下的所有用户赋root权限

%wheel    ALL=(ALL)    NOPASSWD: ALL
 NOPASSWD: ALL：不输入密码执行任何命令（这个代表的是由用户在执行sudo命令的时候不需要输密码、注意这个密码只的是用户自己的密码，不是root账号的密码）

## 磁盘存储管理

### 设备文件的分类

在Linux下的/dev目录中有大量的设备文件，根据设备文件的不同，又分为字符设备文件和块设备文件。
字符设备文件的存取是以字符流的方式来进行的，一次传送一个字符。常见的有打印机，终端（TTY）、绘图仪和磁带设备等等，字符设备文件有时也被称为“raw” 设备文件。
块设备文件是以数据块的方式来存取的，最常见的设备就是磁盘。系统通过块设备文件存取数据的时候，先从内存中的buffer中读或写数据。而不是直接传送数据到物理磁盘。这种方式有效的提高了磁盘的I/O性能。

### MBR和GPT

磁盘在进行格式化的时候，通常需要选择分区格式，而磁盘有两种分区方式：MBR格式与GPT格式

MBR（Master Boot Record）：

MBR又叫做主引导扇区，是计算机开机后访问磁盘时读取的首个扇区，即位于硬盘的0号柱面(Cylinder)、0号磁头(Side)、1号扇区(Sector)。了解柱面、磁头、扇区。该扇区占 512 字节（bytes）。它由三个部分组成：

1. 主引导程序446bytes（boot loader，即主引导记录，现在linux一般由grub2作为boot loader）

2. 硬盘分区表64bytes（Disk Partition Table，存放磁盘分区数据的表）

3. 结束标志位2bytes（固定为十六进制的55AA）。

MBR与GPT的区别在于硬盘分区表，这是由于MBR分区方式的缺陷导致的：硬盘分区表是用来记录硬盘里面有多少个分区以及每一分区的大小，一共占 64 字节，即 16*4 ，所以最多只有4  个分区信息可以写到第一个扇区中，所以就称这4个分区为4个主分区 ( primary partion )，每个分区占16  bytes。在有限的空间内，需要记录分区表的详细情况：

![image](assets\linux-64.jpg)

可见分区表只有4个字节存储分区的总扇区数，最大能表示2的32次方的扇区个数，按每扇区512字节计算，每个分区最大不能超过2TB。随着时代发展，硬盘容量不断扩展，使得之前定义的每个扇区512字节不再是那么的合理，于是将每个扇区512字节改为每个扇区4096个字节，这意味着MBR的有效容量上限提升到16TiB，伴随NTFS成为了标准的硬盘文件系统，同时这样的扇区大小也和其文件系统的默认分配单元大小（簇）匹配，提高了系统检索效率。但是传统硬盘的每个扇区固定是512字节，硬盘厂商为了保证与操作系统兼容性，也会将扇区模拟成512B扇区，这样又会造成4k对齐的问题。有时候四个主分区会不够用，所以会将其中一个分区作为扩展分区 ( extension partion )。扩展分区相当于一个指针（即只记录分区大小位置信息），用来指向某个有记录信息的分区，所以是不能直接存储数据的。由于操作系统的限制，扩展分区最多只有一个。其分区表项指定扩展分区的起始位置和长度，在其中最开始扇区（EBR）和MBR相同位置（0x1BE）放置另外一个分区表，一般称为扩展分区表。扩展分区表的第一项指定扩展分区目前的逻辑分区信息，如果还有更多的逻辑分区，扩展分区表的第二项指定下一个EBR的位置，否则为0。最后的两个分区表项总是为0。通过这种方式，一个硬盘上的分区数目就没有限制了。

GPT（GUID partition table）

作为新一代的磁盘分区形式，GPT磁盘具有分区大小、分区数量等的优势。GPT是一个实体硬盘的分区表的结构布局的标准。它是可扩展固件接口（UEFI）标准的一部分，被用于替代BIOS系统中使用4字节来存储逻辑块地址和分区大小信息的主引导记录(MBR)分区表。GPT标准使用8字节用于记录逻辑块地址，因此，GPT分区格式在同等逻辑块大小的情况下，比MBR分区格式支持更大的硬盘空间。出处于兼容性与安全性方面的考虑，GPT分区格式保留传统MBR，位于LBA0（第一个逻辑扇区），用于防止不支持GPT的硬盘管理软件错误识别并破坏硬盘数据。在这个MBR中，只有一个标志为0xEE的分区，以此表示这块硬盘使用GPT分区格式。不支持GPT分区格式的软件，会识别出未知类型的分区；支持GPT分区格式的软件，可正确识别GPT分区磁盘。

1. GPT分区表，没有扩展分区与逻辑分区的概念，所有分区都是主分区。

2. GPT分区表可以划分出128个分区。

3. GPT分区表的每个分区的最大容量是18EB（1EB = 1024PB = 1,048,576TB），这么大，不用考虑硬盘容量太大的问题了。

4. GPT分区表需要与使用UEFI的电脑配合。

MBR和GPT之间的区别

1. GPT支持大于2TB的分区，而传统的MBR磁盘（512k）不支持。

2. GPT方式支持磁盘划分128个分区，每个分区可达18EB容量；而MBR方式（512k）支持4个主分区（或者也可以是三个主分区加一个扩展分区和无限划分的逻辑卷）。每个分区可达2TB容量

3. GPT磁盘具有更高的性能，这是因为分区表的复制和循环冗余校验（CRC）保护机制来实现的。与MBR磁盘分区不同的是，GPT磁盘将系统相关的重要数据存放于分区中，而不是未分区或隐藏的扇区中。

4. GPT磁盘具有冗余的主分区表和备份分区表，可以优化分区数据结构的完整性。

通常来说，MBR和BIOS（MBR+BIOS）、GPT和UEFI（GPT+UEFI）是相辅相成的。这对于某些操作系统（例如 Windows）是强制性的，但是对于其他操作系统（例如 Linux）来说是可以选择的。

### UEFI和BIOS

UEFI（Unified Extensible Firmware Interface)：全称“统一的可扩展固件接口”，它定义了一种在操作系统和平台固件之间的接口标准。这种接口用于操作系统自动从预启动的操作环境（在系统启动之后，但是操作系统开始运作之前）加载到一种操作系统上，从而使开机程序化繁为简，节省时间。需要注意，UEFI最准确的说它仅是一种规范，不同厂商根据该规范对UEFI的实现，并做出PC固件后,该固件就称为UEFI固件。
BIOS（基本输入和输出系统），是最古老的一种系统固件和接口，采用汇编语言进行编程，并使用中断来执行输入/输出操作，在出现之初即确定了PC生态系统的基本框架。
UEFI比BIOS先进在三个方面：

1. 读取分区表
2. 访问某些特定文件系统中的文件
3. 执行特定格式的代码【可以说UEFI像一个简易的操作系统】

### Linux操作系统GPT-UEFI支持列表

其中，1=BIOS+MBR，2=UEFI+GPT

![image](assets\linux-12.png)

### 利用fdisk工具划分磁盘分区

fdisk是linux下一款功能强大的磁盘分区管理工具，可以观察硬盘的使用情况，也可以对磁盘进行分割，linux下类似与fdisk的工具还有cfdisk、parted等，fdisk工具不支持GPT，要是GPT格式的分区，需要使用另一个GNU发布的强大分区工具parted，下面会介绍。
常用组合：fdisk -l（查看分区情况）
fdisk的使用分为两个部分，查询部分和交互操作部分。通过fdisk device即可进入命令交互操作界面。交互界面下的常用命令含义如下：

1. d：删除一个分区
2. l：查看指定分区的分区表信息
3. m：显示fdisk每个交互命令的详细含义
4. n：增加一个新的分区
5. p：显示分区信息
6. q：退出交互操作，不保存操作的内容
7. t：改变分区类型
8. w：写分区表信息到硬盘，保存操作退出
9. L：查看Linux支持的分区格式

### 利用parted工具规划磁盘分区

对于GPT格式的分区，fdisk工具是无能为力的，同时，fdisk工具对分区是有大小限制的，它只能划分小于2T的磁盘。但是现在的磁盘空间很多都已经是远远大于2T，此时就需要另外一个磁盘管理工具parted来完成大于2T的磁盘分区工作。查看系统是否有parted命令，如果没有，执行如下命令直接安装即可：yum -y install parted
parted交互模式下常用的一些参数：

1. mklabel	创建分区表， 也就是设置使用msdos还是使用gpt格式。例如：mklabel gpt，表示设定分区表为gpt格式。
2. mkpart	创建新分区命令。使用格式为：mkpart PART-TYPE  [FS-TYPE]  START  END，其中，PART-TYPE，表示分区类型，主要有primary（主分区），extended（扩展分区），logical（逻辑区），其中，扩展分区和逻辑分区只针对msdos分区表。
3. fs-type，表示文件系统类型，主要有fat32，NTFS，ext2，ext3等，可不填写。
4. start，表示分区的起始位置。
5. end，表示分区的结束位置。
6. print	输出分区信息，可简写为p。该功能有3个选项：
   1. free，显示该盘的所有信息，并显示磁盘剩余空间。
   2. number， 显示指定的分区的信息。
   3. all或list， 显示所有磁盘信息。
7. rm	删除分区。命令格式 rm  number 。例如：rm 2 就是将编号为3的分区删除。
8. select	选择设备。当输入parted命令后直接回车进入交互模式时，默认设置的是系统的第一块硬盘，如果系统有多块硬盘，需要用select命令选择要操作的硬盘。例如：select /dev/sdb

### LVM

LVM，是Logical Volume Manager的缩写，中文意思是逻辑卷管理，它是linux下对磁盘分区进行管理的一种机制，LVM是建立在磁盘分区和文件系统之间的一个逻辑层，管理员利用LVM可以在磁盘不用重新分区的情况下动态的调整分区的大小。如果系统新增了一块硬盘，通过LVM就可以将新增的硬盘空间直接扩展到原来的磁盘分区上。

1. 物理存储设备（physical media）：指系统的存储设备文件，比如：/dev/sda、/dev/hdb
2. 物理卷（physical volume）：简称PV
3. 卷组（Volume Group）:简称VG
4. 逻辑卷（logical volume）：简称LV
5. PE（physical extent）：PV中可以分配的最小存储单元称为PE
6. LE（logical extent）：LV中可以分配的最小存储单元称为LE

![image](assets\linux-13.png)

```bash
# 创建pv
[root@server01 ~]# pvcreate /dev/sdb1 /dev/sdb2 /dev/sdc1
# 查看pv信息
[root@server01 ~]# pvdisplay
# 创建vg
[root@server01 ~]# vgcreate vg1 /dev/sdb1 /dev/sdb2 /dev/sdc1
# 查看vg信息
[root@server01 ~]# vgdisplay
# 激活vg1
[root@server01 ~]# vgchange -a y vg1
# 创建lv，-L后面跟lv的大小
[root@server01 ~]# lvcreate -L 25G -n lv1 vg1
# 创建lv，-l后面跟的是PE的数量
[root@server01 ~]# lvcreate -L 1278 -n lv2 vg1
# 查看lv信息
[root@server01 ~]# lvdisplay
# 格式化lv1
[root@server01 ~]# mkfs.xfs /dev/vg1/lv1
# 格式化lv2
[root@server01 ~]# mkfs.ext4 /dev/vg1/lv2
# 挂载lv1
[root@server01 ~]# mount /dev/vg1/lv1 /data1
# 挂载lv2
[root@server01 ~]# mount /dev/vg1/lv2 /data2
#########################################################################################
# 扩充vg
[root@server01 ~]# vgextend vg1 /dev/sdc2
#########################################################################################
# 扩充lv1
[root@server01 ~]# lvextend -l +1000 /dev/vg1/lv1
# 在线调整大小
[root@server01 ~]# xfs_growfs /data1
# 扩充lv2
[root@server01 ~]# lvextend -L +10G /dev/vg1/lv2
# 在线调整大小
[root@server01 ~]# resize2fs /dev/vg1/lv2
#########################################################################################
# 删除lv
[root@server01 ~]# umount /data1
[root@server01 ~]# lvremove -L +10G /dev/vg1/lv1
[root@server01 ~]# umount /data2
[root@server01 ~]# lvremove -L +10G /dev/vg1/lv2
# 删除vg
[root@server01 ~]# vgremove vg1
# 删除pv
[root@server01 ~]# pvremove /dev/sdb1
[root@server01 ~]# pvremove /dev/sdb2
[root@server01 ~]# pvremove /dev/sdc1
[root@server01 ~]# pvremove /dev/sdc2
```

## 文件系统管理

### 线上业务系统选择文件系统标准

Linux下常见的有DOS文件系统类型msdos，windows下的FAT系列（fat16和FAT32）和NTFS文件系统，光盘文件系统ISO-9660，单一文件系统ext2和日志文件系统ext3、ext4、xfs，集群文件系统gfs（Red Hat Global File System）、ocfs2（oracle cluster File System）、虚拟文件系统（比如 /proc），网络文件系统（NFS）。

1. 读操作频繁，同时小文件众多的应用对于此类应用，选择ext4文件系统都是不错的选择。由于ext3的目录结构是线型的，因此当一个目录下文件较多时，ext3的性能就下降比较多。而ext4的延迟分配、多块分配和盘区功能，使ext4非常适合大量小文件的操作，因此，从性能方面考虑，对于小规模文件密集型应用，ext4文件系统是首选。而如果从性能和安全性方面综合考虑的话，xfs文件系统是比较好的选择。大量实践证明，如果业务环境是对文件要进行大量的创建和删除操作的话，ext4是更高效的文件系统，接下来依次是xfs、ext3。例如网站应用，邮件系统等，都可使用ext4文件系统来达到最优性能。
2. 写操作频繁的应用，如果是一些大数据文件操作，同时，应用本身需要大量日志写操作，那么，xfs文件系统是最佳选择，根据实际应用经验，对xfs、ext4、ext3块写入性能对比，整体上性能差不多，但在效率上（CPU利用率）最好的是xfs，接下来依次是ext4和ext3。
3. 对性能要求不高、数据安全要求不高的业务，对于这类应用，ext3/ext2文件系统是比较好的选择，因为ext2没有日志记录功能，这样就节省了很多磁盘性能。例如linux系统下的/tmp分区就可以采用ext2文件系统。

### NFS

NFS的全称是Network FileSystem，即网络文件系统，NFS主要实现的功能是让网络上的不同操作系统之间共享数据。NFS首先在远程服务端（共享数据的操作系统）共享出文件或者目录，然后远端共享出来的文件或者目录就可以通过挂载（mount）的方式挂接到本地的不同操作系统上，最后，本地系统就可以很方便的使用远端提供的文件服务，操作起来像在本地操作一样。从而实现了数据的共享。

![image](assets\linux-14.png)

#### NFS Server端的配置

NFS的主要配置文件只有一个/etc/exports，配置非常简单，设置格式为：
共享资源路径   [主机地址]   [选项]
例如：下面是某系统/etc/exports的设置：
/webdata  *(sync,rw,all_squash)
/tmp　　　　　*(rw,no_root_squash) 
/home/share 　192.168.1.*(rw,root_squash)　　 *(ro) 
/opt/data 　　192.168.1.18(rw) 
共享资源路径：就是要共享出来的目录或者磁盘分区。例如上面的/tmp、/home/share目录等，这些目录存在于NFS Server端，以供NFS Client挂载使用。
主机地址：设定允许使用NFS Server共享资源的客户端主机地址，主机地址可以是主机名、域名、IP地址等，支持匹配。

选项：下面是可用的各个选项含义：

1. ro： 即为：read only，也就是客户端主机对共享资源仅仅有读权限。
2. rw： 即为：read write，也就是客户端主机对共享资源有读、写权限。
3. sync：资料同步写入磁盘中。默认选择。
4. async：资料会先暂时存放在内存中，不会直接写入硬盘。
5. all_squash：将远程访问的所有普通用户及所属组都映射为匿名用户或用户组（nfsnobody）；

6. no_all_squash：与all_squash取反（默认设置）；

7. root_squash：将root用户及所属组都映射为匿名用户或用户组（默认设置）；

8. no_root_squash：与rootsquash取反；

9. anonuid=xxx：将远程访问的所有用户都映射为匿名用户，并指定该用户为本地用户（UID=xxx）；

10. anongid=xxx：将远程访问的所有用户组都映射为匿名用户组账户，并指定该匿名用户组账户为本地用户组账户（GID=xxx）；

利用exportfs命令即可让修改生效
重新mount 文件/etc/exports中分享出来的目录，显示mount过程，操作如下：
[root@NFS Server ~]# exportfs  -rv 
-r ：重新mount /etc/exports中分享出来的目录。 
-v ：在 export 的時候，将详细的信息输出到屏幕上。

#### NFS客户端的设定

客户端系统也是linux，首先需要在客户端安装nfs-utils和rpcbind两个服务
[root@localhost ~]# yum -y install nfs-utils
[root@localhost ~]# systemctl start rpcbind 
[root@localhost ~]# systemctl enable rpcbind 
关闭NFS Server上开启了防火墙：
[root@NFS Server ~]# systemctl stop firewalld
[root@NFS Server ~]# systemctl disable firewalld
客户端要使用NFS Server提供的共享资源，使用mount命令挂载就可以了：
挂载的格式： 
[root@localhost ~]#mount -t nfs Hostname(orIP):/directory  /mountpoint
Hostname：用来指定NFS Server的地址，可以是IP地址或主机名。
/directory：表示NFS Server共享出来的目录资源。
/mountpoint：表示客户端主机指定的挂载点。通常是一个空目录。
例如：
[root@localhost ~]#mount -t nfs 192.168.60.133:/mydata  /data/nfs

### 使用extundelete恢复误删除的文件

反删除工具简介
在Linux下，基于开源的数据恢复工具有很多，常见的有debugfs、R-Linux、ext3grep、extundelete等，比较常用的有ext3grep和extundelete，这两个工具的恢复原理基本一样，只是extundelete功能更加强大。
ext3grep仅支持ext3文件系统的恢复，恢复速度较慢，而extundelete可以恢复ext3/ext4文件系统的数据，并且恢复速度很快。
extundelete官网：http://extundelete.sourceforge.net/

恢复原理
extundelete首先会通过文件系统的inode信息（根目录的inode一般为2）来获得当前文件系统下所有文件的信息，包括存在的和已经删除的文件，然后利用inode信息结合日志去查询该inode所在的block位置，包括直接块，间接块等信息。最后利用dd命令将这些信息备份出来，从而恢复数据文件。

extundelete的安装与使用
[root@cloud1 app]#tar jxvf  extundelete-0.2.4.tar.bz2
[root@cloud1 app]#cd extundelete-0.2.4
[root@cloud1 extundelete-0.2.4]#./configure
[root@cloud1 extundelete-0.2.4]#make
[root@cloud1 extundelete-0.2.4]#make install
成功安装extundelete后，会在系统中生成一个extundelete可执行文件

extundelete常用选项：
--restore-inode ino[,ino,...]，恢复命令参数，表示恢复节点“ino”的文件，恢复的文件会自动放在当前目录下的RESTORED_FILES文件夹中，使用节点编号作为扩展名。
--restore-file 'path'，恢复命令参数，表示将恢复指定路径的文件，并把恢复的文件放在当前目录下的RECOVERED_FILES目录中。
--restore-files 'path'	，恢复命令参数，表示将恢复在路径中已列出的所有文件。
--restore-all，恢复命令参数，表示将尝试恢复所有目录和文件。

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

# Linux进程管理与监控

## 进程的概念与分类

进程的的基本定义是：在自身的虚拟地址空间运行的一个独立的程序，从操作系统的角度来看，所有在系统上运行的东西，都可以称为一个进程。
进程的分类

1. 系统进程：可以执行内存资源分配和进程切换等管理工作；而且，该进程的运行不受用户的干预，即使是root用户也不能干预系统进程的运行。

2. 用户进程：通过执行用户程序、应用程序或内核之外的系统程序而产生的进程，此类进程可以在用户的控制下运行或关闭。

   - 交互进程：由一个shell终端启动的进程，在执行过程中，需要与用户进行交互操作，可以运行于前台，也可以运行在后台。

   - 批处理进程：该进程是一个进程集合，负责按顺序启动其他的进程。

   - 守护进程：守护进程是一直运行的一种进程，经常在linux系统启动时启动，在系统关闭时终止。例如httpd进程，一直处于运行状态，等待用户的访问。还有经常用的crond进程，这个进程类似与windows的计划任务，可以周期性的执行用户设定的某些任务。

## 进程的监控与管理

在linux系统中，进程ID（用PID表示）是区分不同进程的唯一标识，它们的大小是有限制的，最大ID为32768，用UID和GID分别表示启动这个进程的用户和用户组。所有的进程都是PID为1的init进程（centos7.x版本是systemd进程）的后代。内核在系统启动的最后阶段启动init进程，因而，这个进程是linux下所有进程的父进程，用PPID表示父进程。
常用的进程管理命令有：
ps、top、lsof、pgrep、kill、killall

### lsof

```bash
# lsof -p PID：PID是进程号，通过进程号显示程序打开的所有文件及相关进程
[root@server01 ~]# lsof -p 1065
COMMAND  PID USER   FD   TYPE             DEVICE SIZE/OFF     NODE NAME
sshd    1065 root  cwd    DIR                8,2     4096       64 /
sshd    1065 root  rtd    DIR                8,2     4096       64 /
sshd    1065 root  txt    REG                8,2   819640 50916842 /usr/sbin/sshd
sshd    1065 root  DEL    REG                0,4             15932 /dev/zero
sshd    1065 root  mem    REG                8,2   510416 50342739 /usr/lib64/libfreeblpriv3.so
sshd    1065 root  mem    REG                8,2    15480 16873174 /usr/lib64/security/pam_lastlog.so
# lsof -c PNAME：PNAME是进程名称，通过进程名称显示程序打开的所有文件及相关进程
[root@server01 ~]# lsof -c sshd
COMMAND  PID USER   FD   TYPE             DEVICE SIZE/OFF     NODE NAME
sshd     531 root  cwd    DIR                8,2     4096       64 /
sshd     531 root  rtd    DIR                8,2     4096       64 /
sshd     531 root  txt    REG                8,2   819640 50916842 /usr/sbin/sshd
sshd     531 root  mem    REG                8,2    62184 51936217 /usr/lib64/libnss_files-2.17.so
sshd     531 root  mem    REG                8,2    44448 51936229 /usr/lib64/librt-2.17.so
sshd     531 root  mem    REG                8,2    15688 50379752 /usr/lib64/libkeyutils.so.1.5
# 通过lsof也可以查询每个进程打开的文件句柄数目
[root@server01 ~]# lsof -c sshd | wc -l
126
# 查看那些进程占用/usr/lib64/libdl-2.17.so
[root@server01 ~]# lsof | grep /usr/lib64/libdl-2.17.so
# lsof -i  通过监听指定的协议、端口、主机等信息，显示符合条件的进程信息。
# 查看占用22端口的进程的信息
[root@server01 ~]#  lsof -i :22
COMMAND  PID USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
sshd     531 root    3u  IPv4  13820      0t0  TCP *:ssh (LISTEN)
sshd     531 root    4u  IPv6  13829      0t0  TCP *:ssh (LISTEN)
sshd    1065 root    3u  IPv4  15881      0t0  TCP server01:ssh->192.168.56.1:65496 (ESTABLISHED)
[root@server01 ~]#  lsof -i tcp:22
COMMAND  PID USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
sshd     531 root    3u  IPv4  13820      0t0  TCP *:ssh (LISTEN)
sshd     531 root    4u  IPv6  13829      0t0  TCP *:ssh (LISTEN)
sshd    1065 root    3u  IPv4  15881      0t0  TCP server01:ssh->192.168.56.1:65496 (ESTABLISHED)
# 通过pgrep命令获取进程名称对应的所有进程的PID
[root@server01 ~]# pgrep sshd
531
1065
```

### pidof

```bash
[root@server01 ~]# pidof sshd
1134 528
```

### kill

用kill终止一个进程
kill命令的使用语法为：
kill [信号类型] 进程PID
信号类型有很多种，可以通过kill –l查看所有信号类型。常用的信号类型有SIGKILL，对应的数字为9，还有SIGTERM和SIGINT，对应的数字分别为15和2。
kill -0 进程PID：试探进程是否存在，如果存在就什么也不做，否则提示进程不存在
kill -9 进程PID：表示强制结束进程
kill -2 进程PID：表示结束进程，但是并不是强制性的，常用的ctrl+c组合键发出的就是一个kill -2的信号。
kill -15 进程PID：表示正常结束进程，是kill的缺省选项，也就是kill不加任何信号类型时，默认类型就是15。

killall也是关闭进程的一个命令，与kill不同的是，killall后面跟的是进程的名字，而不是进程的PID，因而，killall可以终止一组进程。
killall的使用语法为：
killall [信号类型] 进程名称
信号类型：与kill命令中信号类型的含义相同。
进程名称：进程对应的名称，例如java、httpd、mysqld、sshd、sendmail等

### fuser

查看文件被哪个进程占用

```bash
# 安装
[root@server01 ~]# yum install -y psmisc
# 查看某个进程的pid
[root@server01 ~]# fuser /bin/bash
/usr/bin/bash:        1136e
# 查看当前目录正在被哪些进程在使用
[root@server01 ~]# fuser -uv ./
                     用户     进程号 权限   命令
/root:               root       1136 ..c.. (root)bash
                     root       1372 ..c.. (root)ntpdate
# 列占用指定端口的进程号
[root@server01 ~]# fuser -n tcp 9009
9009/tcp:             1037
# 查看/lib/gcc/x86_64-redhat-linux/4.8.2/libgcc_s.so正在被哪些进程在使用
[root@server01 ~]# fuser -uv /lib/gcc/x86_64-redhat-linux/4.8.2/libgcc_s.so
                     用户     进程号 权限   命令
/usr/lib64/libgcc_s-4.8.5-20150702.so.1:
                     root          1 ....m (root)systemd
                     root        347 ....m (root)systemd-journal
                     root        365 ....m (root)lvmetad
                     root        367 ....m (root)systemd-udevd
                     root        469 ....m (root)rsyslogd
                     polkitd     470 ....m (polkitd)polkitd
                     rpc         484 ....m (rpc)rpcbind
                     root        490 ....m (root)systemd-logind
                     root        495 ....m (root)tuned
                     root        755 ....m (root)master
                     postfix     776 ....m (postfix)pickup
                     postfix     777 ....m (postfix)qmgr
                     mysql       784 ....m (mysql)mysqld
                     root        832 ....m (root)dhclient
                     root       1134 ....m (root)sshd
                     postfix    1241 ....m (postfix)cleanup
                     postfix    1243 ....m (postfix)trivial-rewrite
                     postfix    1244 ....m (postfix)local
# 查看/proc这个目录有哪些进程在使用
[root@server01 ~]# fuser -uv /proc
                     用户     进程号 权限   命令
/proc:               root     kernel mount (root)/proc
# 哪些进程在进行/proc文件系统的读取
[root@server01 ~]# fuser -muv /proc
                     用户     进程号 权限   命令
/proc:               root     kernel mount (root)/proc
                     root          1 f.... (root)systemd
                     root        347 f.... (root)systemd-journal
                     clickhouse   1039 f.... (clickhouse)clickhouse-serv
# 杀死/home占用home目录的所有进程
[root@server01 ~]# fuser -mki /home
```

## 任务调度进程crond的使用

### crond的概念和分类

crond是linux下用来周期性的执行某种任务或等待处理某些事件的一个守护进程，与windows下的计划任务类似。Linux下的任务调度分为两类，系统任务调度和用户任务调度。
系统任务调度：系统周期性所要执行的工作，比如写缓存数据到硬盘、日志清理等。在/etc目录下有一个crontab文件，这个就是系统任务调度的配置文件。
用户任务调度：用户定期要执行的工作，比如用户数据备份、定时邮件提醒等。用户可以使用 crontab 工具来定制自己的计划任务。所有用户定义的crontab 文件都被保存在 /var/spool/cron目录中。其文件名与用户名一致。

### crontab常用的使用格式

crontab [-u user] [file] 
crontab [-u user] [-e|-l|-r |-i]
选项含义如下：

1. -u user：用来设定某个用户的crontab服务，例如，“-u ixdba”表示设定ixdba用户的crontab服务，此参数一般有root用户来运行。
2. file：file是命令文件的名字,表示将file做为crontab的任务列表文件并载入crontab。如果在命令行中没有指定这个文件，crontab命令将接受标准输入（键盘）上键入的命令，并将它们载入crontab。
3. -e：编辑某个用户的crontab文件内容。如果不指定用户，则表示编辑当前用户的crontab文件。
4. -l：显示某个用户的crontab文件内容，如果不指定用户，则表示显示当前用户的crontab文件内容。
5. -r：从/var/spool/cron目录中删除某个用户的crontab文件，如果不指定用户，则默认删除当前用户的crontab文件。
6. -i：在删除用户的crontab文件时给确认提示。

### crontab文件的含义

用户所建立的crontab文件中，每一行都代表一项任务，每行的每个字段代表一项设置，它的格式共分为六个字段，前五段是时间设定段，第六段是要执行的命令段，格式如下：
minute   hour   day   month   week   command
其中：
minute： 表示分钟，可以是从0到59之间的任何整数。
hour：表示小时，可以是从0到23之间的任何整数。
day：表示日期，可以是从1到31之间的任何整数。
month：表示月份，可以是从1到12之间的任何整数。
week：表示星期几，可以是从0到7之间的任何整数，这里的0或7代表星期日。
command：要执行的命令，可以是系统命令，也可以是自己编写的脚本文件。
在以上各个字段中，还可以使用以下特殊字符：

```
星号（*）：代表所有可能的值，例如month字段如果是星号，则表示在满足其它字段的制约条件后每月都执行该命令操作。
逗号（,）：可以用逗号隔开的值指定一个列表范围，例如，“1,2,5,7,8,9”
中杠（-）：可以用整数之间的中杠表示一个整数范围，例如“2-6”表示“2,3,4,5,6”
正斜线（/）：可以用正斜线指定时间的间隔频率，例如“0-23/2”表示每两小时执行一次。同时正斜线可以和星号一起使用，例如*/10，如果用在minute字段，表示每十分钟执行一次。
```

### crontab应用举例

0 */3 * * * /usr/local/apache2/apachectl restart
表示每隔3个小时重启apache服务一次。

30 3 * * 6 /webdata/bin/backup.sh
表示每周六的3点30分执行/webdata/bin/backup.sh脚本的操作。

0 0 1,20 * *  fsck /dev/sdb8
表示每个月的1号和20号检查/dev/sdb8磁盘设备。

10 5 */5 * *  echo "">/usr/local/apache2/log/access_log
表示每个月的5号、10号、15号、20号、25号、30号的5点10分执行清理apache日志操作。

```bash
# 查看crontab是否被执行的日志
[root@server01 var]# tail -f /var/log/cron
Oct 26 16:30:01 localhost CROND[1474]: (root) CMD (/usr/lib64/sa/sa1 1 1)
Oct 26 16:31:01 localhost CROND[1484]: (root) CMD (/usr/sbin/ntpdate ntp4.aliyun.com;)
Oct 26 16:32:01 localhost CROND[1489]: (root) CMD (/usr/sbin/ntpdate ntp4.aliyun.com;)
```

### 注意事项

1. 有时需要用到Crontab的定时任务去执行脚本，但是发现通过命令（./test.sh)执行Shell文件的时候，可以获取Linux的环境变量;可是通过Crontab做的定时任务，无法获取。

   问题剖析：	

   crontab不会缺省的从用户profile文件中读取环境变量参数，经常导致在手工执行某个 脚本时是成功的，但是到crontab中试图让它定期执行时就是会出错。crontab执行环境在/etc/crontab，具体配置如下：

   ```sh
   SHELL=/bin/bash
   PATH=/sbin:/bin:/usr/sbin:/usr/bin
   MAILTO=root
   
   # For details see man 4 crontabs
   
   # Example of job definition:
   # .---------------- minute (0 - 59)
   # |  .------------- hour (0 - 23)
   # |  |  .---------- day of month (1 - 31)
   # |  |  |  .------- month (1 - 12) OR jan,feb,mar,apr ...
   # |  |  |  |  .---- day of week (0 - 6) (Sunday=0 or 7) OR sun,mon,tue,wed,thu,fri,sat
   # |  |  |  |  |
   # *  *  *  *  * user-name  command to be executed
   ```

   配置解释：

   前三行是用来配置crond任务运行的环境变量
   第一行SHELL变量指定了系统要使用哪个shell，这里是bash；
   第二行PATH变量指定了系统执行命令的路径；
   第三行MAILTO变量指定了crond的任务执行信息将通过电子邮件发送给root用户，如果MAILTO变量的值为空，则表示不发送任务执行信息给用户；

   几种解决办法：

   1、在Shell文件里面获取环境变量值的路径写成绝对路径，别用环境变量的路径值。例如获取CPU的使用情况 通过绝对路径/proc/cpuinfo 来获取值；

   2、在即将执行的Shell脚本缺省的#!/bin/sh开头换行后的第一行加入：source /etc/profile或者. /etc/profile

   3、在/etc/crontab中添加环境变量，也可以在执行对应的命令之前，加入一条命令，使得环境变量生效，例如：

   ```bash
   0 * * * * . /etc/profile;/bin/sh /var/www/java/audit_no_count/bin/restart_audit.sh
   ```

   备注：在corntable 中执行多条语句时，用分号“；”隔开。故以上例子就是先执行. /etc/profile; 这条命令，然后再运行sh脚本。

2. 注意清理系统用户的邮件日志，可以在crontab文件中设置如下形式，忽略日志输出：0 */3 * * * /usr/local/apache2/apachectl restart >/dev/null 2>&1

3. 系统级任务调度与用户级任务调度，系统级任务调度主要完成系统的一些维护操作(比如定时重启机器），用户级任务调度主要完成用户自定义的一些任务，可以将用户级任务调度放到系统级任务调度来完成（不建议这么做），但是反过来却不行。

# Linux日志分析与故障排查

## Linux系统日志与分类

1. 内核及系统日志：这种日志数据由系统服务syslog统一管理，根据其主配置文件"/etc/rsyslog.conf"中的设置决定将内核消息及各种系统程序消息记录到什么位置。（在centos7以前为/etc/syslog.conf）
2. 用户日志：这种日志数据用于记录Linux系统用户登录及退出系统的相关信息，包括用户名、登录的终端、登录时间、来源主机、正在使用的进程操作等。
3. 程序日志：有些应用程序运会选择自己来独立管理一份日志文件（而不是交给syslog服务管理），用于记录本程序运行过程中的各种事件信息。

## Linux下日志文件解读

Linux系统本身和大部分服务器程序的日志文件默认情况下都放置在目录“/var/log”中。
/var/log/messages：公共日志文件，记录Linux内核消息及各种应用程序的公共日志信息，包括启动、IO错误、网络错误、程序故障等。对于未使用独立日志文件的应用程序或服务，一般都可以从该文件获得相关的事件记录信息。 
/var/log/cron：记录crond计划任务产生的事件消息。 
/var/log/dmesg： 包含内核缓冲信息（kernel ring buffer）。在系统启动时，会在屏幕上显示许多与硬件有关的信息。此文件记录的信息时系统上次启动时的信息。而用dmesg命令可查看本次系统启动时与硬件有关的信息，以及内核缓冲信息。 
/var/log/maillog：记录进入或发出系统的电子邮件活动。
/var/log/boot.log  记录系统启动时的软件日志信息。
/var/log/secure：记录用户远程登录、认证过程中的事件信息。 
/var/log/wtmp：记录系统所有登录进入和退出纪录。 可执行last命令查看
/var/log/btmp：记录错误登录进入系统的日志信息，可执行lastb命令查看。
/var/log/lastlog：记录最近成功登录的事件和最后一次不成功的登录事件。可执行lastlog命令查看。

## 忘记root密码的解决案例

这个问题出现的几率是很高的，不过，在linux下解决这个问题也很简单，只需重启linux系统，然后引导进入linux的单用户模式（init 1），由于单用户模式是不需要输入登录密码的，因此，可以直接登录系统，修改root密码即可解决问题。重点内容：如何重启系统，进入单用户模式（centos6.x和centox7.x方式不同）

/boot/grub/grub.conf系统启动时的引导文件解读(在centos7对应的是/etc/grub2.conf)
default=0：  定义了没有选择内核菜单的启动项时选择第一个内核启动。
timeout=5 ： 定义了没有任何操作时5s的超时时间。
splashimage=(hd0,0)/grub/splash.xpm.gz   定义了开机时内核选择菜单的背景图片，可以不写这一行，但是写错也会导致机器无法启动！
hidemenu：   隐藏内核选择菜单，按任意键出现选择菜单，可以不写这一行。
title：内核名字标题。
root(hd0,0) ：  相对下面的内核和initrd全局定义root为第一块磁盘的第一个分区，此处的root不是真的root，而是开机时的/boot 分区，因为bootloader开机还没有加载内核以及/分区。加载的内核需要通过/boot分区加载/分区和内核。
kernel ：  定义内核文件位置，向内核文件传递必要的参数。并且指定  /  分区所在的位置
initrd：  包括加载根分区的必要的驱动以及可以在内存当中解压释放出虚根用于加载真正的内核文件

## 系统无法启动故障案例

### root文件系统破坏，导致系统无法启动故障案例

这种情况多由于异常断电、不正常关机，引起文件系统结构不一致导致的。此种问题发生，在系统启动的时候，屏幕会显示：
checking root filesystem
/dev/sdb5 contains a file system with errors, check forced
/dev/sdb5: UNEXPECTED INCONSISTENCY; RUN fsck MANUALLY
(i.e., without -a or -p options)  FAILED
Press enter for maintenance
(or type Control-D to continue):
give root password for maintenance
从这个错误可以看出，系统根分区文件系统出现了问题，系统在启动时无法自动修复，然后进入到了一个交互界面，提示用户进行系统修复。
解决方法：
输入root密码后进入系统修复模式，在修复模式下，可以执行fsck命令（在修复前需要先umount），例如：
[root@localhost /]# fsck .ext4 -y  /dev/sdb5

### /etc/fstab文件丢失，导致系统无法启动案例

/etc/fstab文件存放了系统中文件系统的相关信息，在linux启动时，系统会读取此文件，自动挂载linux的各个分区，如果此文件配置错误，或者丢失，就会导致系统无法启动，具体的故障现象是在检测mount partition时出现：starting system logger此后系统启动就停止了。
解决方法：
利用linux rescue修复模式登录系统，进而获取分区和挂载点信息，重构/etc/fstab文件。

```bash
# 查看分区对应的挂载点、UUID等详细的分区信息
[root@server01 log]# tune2fs -l /dev/sda2
```

### Linux系统无法启动的通用解决方案

（1）进入单用户模式或援救模式（rescue），修复分区错误或者备份数据，然后修复或重新安装系统。
（2）如果是linux的引导程序出现问题，那么也可以通过光盘引导的方式进入linux修复模式，然后修改对应的引导程序或者重新安装引导程序。
（3）如果linux内核崩溃或者丢失，同样可以先进入linux rescue模式下，然后加载root分区，最后重新编译内核。

## Read-only file system

现象：java.lang.RuntimeException: Cannot make directory: file:/www/data/html/2018-03-21
思路：可能是服务器磁盘故障（磁盘空间满了或者磁盘无法写入了）
原因：磁盘分区出现了问题，导致文件系统结构不一致，文件系统关闭了写功能，需要修复文件系统结构：
[root@localhost ~]# umount /www/data
[root@localhost ~]# fsck -y  /dev/sdb1

## su命令切换用户带来的问题

故障现象：su: warning: cannot change directory to /home/oracle: Permission denied
解决思路：

1. 用户目录/home/oracle权限问题
2. su程序执行权限问题
3. 序依赖的共享库权限问题
4. selinux问题导致
5. 系统根空间问题

产生原因：根目录权限问题导致，修改根目录权限即可 [root@localhost ~]#chmod 555 /

```bash
# 查看用户目录权限：700表示用户对于用户目录拥有读写及执行权限，所以没有问题
[root@server01 home]# ls -al | grep elasticsearch
drwx------   7 elasticsearch elasticsearch      177 10月 25 16:24 elasticsearch
# 查看su命令的权限，用户、用户组及其他用户都拥有su命令的执行权限，所以没有问题
[root@server01 home]# ls -al /bin/su
-rwsr-xr-x 1 root root 32208 3月  14 2019 /bin/su
# 查看su命令依赖的动态库的权限
[root@server01 home]# ldd /bin/su
        linux-vdso.so.1 =>  (0x00007fff344fa000)
        libpam.so.0 => /lib64/libpam.so.0 (0x00007f4722275000)
        libpam_misc.so.0 => /lib64/libpam_misc.so.0 (0x00007f4722071000)
        libc.so.6 => /lib64/libc.so.6 (0x00007f4721cad000)
        libaudit.so.1 => /lib64/libaudit.so.1 (0x00007f4721a84000)
        libdl.so.2 => /lib64/libdl.so.2 (0x00007f4721880000)
        /lib64/ld-linux-x86-64.so.2 (0x00007f4722696000)
        libcap-ng.so.0 => /lib64/libcap-ng.so.0 (0x00007f4721679000)
# selinux问题
[root@server01 home]# vim /etc/selinux/config
# This file controls the state of SELinux on the system.
# SELINUX= can take one of these three values:
#     enforcing - SELinux security policy is enforced.
#     permissive - SELinux prints warnings instead of enforcing.
#     disabled - No SELinux policy is loaded.
# 查看SELINUX是否启用，目前SELINUX是disabled状态(如果是启用的话就是enforcing)，所以没有问题
SELINUX=disabled
# SELINUXTYPE= can take one of three two values:
#     targeted - Targeted processes are protected,
#     minimum - Modification of targeted policy. Only selected processes are protected. 
#     mls - Multi Level Security protection.
SELINUXTYPE=targeted
# 查看根目录权限：0666；在查看其他正常系统的根目录的权限为：0555，所以问题就出在根目录上，修改根目录权限即可：chmod 555 /
[root@server01 home]# stat /
  文件："/"
  大小：4096            块：8          IO 块：4096   目录
设备：802h/2050d        Inode：64          硬链接：22
权限：(0666/drw-rw-rw-)  Uid：(    0/    root)   Gid：(    0/    root)
最近访问：2022-10-26 13:03:18.324000000 +0800
最近更改：2022-08-25 17:44:48.959909299 +0800
最近改动：2022-08-25 17:44:48.959909299 +0800
创建时间：-
```

## Too many open files

对于服务器来说，file-max和ulimit都需要设置，否则会出现文件描述符耗尽的问题。

### max-file

max-file 表示系统级别的能够打开的文件句柄的数量。是对整个系统的限制，并不是针对用户的。

```bash
# 临时修改系统级打开最大文件句柄的数量的方法
[root@server01 ~]# sysctl -w fs.file-max=640000
# 系统级打开最大文件句柄的数量永久生效的修改方法
# 修改文件/etc/sysctl.conf
[root@server01 ~]# vim /etc/sysctl.conf
# 文件末尾加入配置内容：fs.file-max = 640000
fs.file-max = 64000
# 然后执行命令，使修改配置立即生效。
[root@server01 ~]# sysctl -p
fs.file-max = 640000
```

### ulimit

ulimit -n 控制进程级别能够打开的文件句柄的数量。提供对shell及其启动的进程的可用文件句柄的控制。这是进程级别的。参数：

-a	显示当前系统所有的limit资源信息。 
-H	设置硬资源限制，一旦设置不能增加。
-S	设置软资源限制，设置后可以增加，但是不能超过硬资源设置。
-c	最大的core文件的大小，以 blocks 为单位。
-f	进程可以创建文件的最大值，以blocks 为单位.
-d	进程最大的数据段的大小，以Kbytes 为单位。
-m	最大内存大小，以Kbytes为单位。
-n	可以打开的最大文件描述符的数量。
-s	线程栈大小，以Kbytes为单位。
-p	管道缓冲区的大小，以Kbytes 为单位。
-u	用户最大可用的进程数。
-v	进程最大可用的虚拟内存，以Kbytes 为单位。
-t	最大CPU占用时间，以秒为单位。
-l	最大可加锁内存大小，以Kbytes 为单位。

```bash
# ulimit -a显示的是所有的limit资源信息：比如：core file size表示的是限制内核文件的大小、data seg size表示的是最大数据大小。
[root@server01 ~]# ulimit -a
core file size          (blocks, -c) 0
data seg size           (kbytes, -d) unlimited
scheduling priority             (-e) 0
file size               (blocks, -f) unlimited
pending signals                 (-i) 7927
max locked memory       (kbytes, -l) unlimited
max memory size         (kbytes, -m) unlimited
open files                      (-n) 1024
pipe size            (512 bytes, -p) 8
POSIX message queues     (bytes, -q) 819200
real-time priority              (-r) 0
stack size              (kbytes, -s) 8192
cpu time               (seconds, -t) unlimited
max user processes              (-u) 7927
virtual memory          (kbytes, -v) unlimited
file locks                      (-x) unlimited
# 设置的是当前shell的当前用户的打开的最大限制
[root@server01 ~]# ulimit -n 65536
# 进程级打开文件句柄数量永久生效的修改方法，修改limits.conf文件，limits.conf的格式如下：
# <domain>　　 <type>　　 <item> 　　 <value>
# domain ==>
# username或者@groupname 设置需要被限制的用户名或组，组名前面加@和用户名区别；也可以用通配符*来做所有用户的限制。
# type ==>
# soft 指的是当前系统生效的设置值（警告）
# hard 表明系统中所能设定的最大值（错误）
# soft 的限制不能比hard限制高，- 表明同时设置了 soft 和 hard 的值。
# item ==>
# core - 限制内核文件的大小（KB）
# date - 最大数据大小（KB）
# fsize - 最大文件大小（KB）
# memlock - 最大锁定内存地址空间（KB）
# nofile - 打开的文件描述符的最大数目**（经常设置）**
# rss - 最大持久设置大小（KB）
# stack - 最大堆栈大小（KB）
# cpu - 最大CPU时间（min）
# noproc - 进程最大数量，注意最大进程数在/etc/security/limits.d/20-nproc.conf这个配置文件中也有设置，所以必须两边都改才会生效。
# as - 地址空间限制（KB）
# maxlogins - 此用户的最大登录数量
# maxsyslogins - 在系统上登录的最大数目
# priority - 优先级运行用户进程
# locks -  文件的最大数量锁定用户可容纳
# sigpending - 最大挂起信号的数量
# msgqueue - 通过POSIX消息队列使用的最大内存（字节）
# nice - 最大不错优先允许提高到值：[-20，19]
# rtprio - 最大实时优先
# 修改以后，需要重新登录才能生效。
[root@server01 ~]# vim /etc/security/limits.conf
* - nofile 65536
```

## Linux网络故障处理思路和经验

检查网线状态

```bash
[root@server01 limits.d]# ethtool enp0s3   
Settings for enp0s3:
        Supported ports: [ TP ]
        Supported link modes:   10baseT/Half 10baseT/Full 
                                100baseT/Half 100baseT/Full 
                                1000baseT/Full 
        Supported pause frame use: No
        Supports auto-negotiation: Yes
        Advertised link modes:  10baseT/Half 10baseT/Full 
                                100baseT/Half 100baseT/Full 
                                1000baseT/Full 
        Advertised pause frame use: No
        Advertised auto-negotiation: Yes
        Speed: 1000Mb/s
        Duplex: Full
        Port: Twisted Pair
        PHYAD: 0
        Transceiver: internal
        Auto-negotiation: on
        MDI-X: off (auto)
        Supports Wake-on: umbg
        Wake-on: d
        Current message level: 0x00000007 (7)
                               drv probe link
        Link detected: yes  #yes表示网卡的链路是连通的
```

检查网卡状态

```bash
# 通过ethtool -i查看网卡驱动
[root@server01 limits.d]# ethtool -i enp0s8
driver: e1000
version: 7.3.21-k8-NAPI
firmware-version: 
expansion-rom-version: 
bus-info: 0000:00:08.0
supports-statistics: yes
supports-test: yes
supports-eeprom-access: yes
supports-register-dump: yes
supports-priv-flags: no
# 通过lsmod查看驱动是否正常加载
[root@server01 limits.d]# lsmod | grep e1000
e1000                 137544  0
# ifconfig 查看网卡状态，flags中UP表示网卡是激活状态
[root@server01 limits.d]# ifconfig
enp0s3: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 10.0.2.15  netmask 255.255.255.0  broadcast 10.0.2.255
        inet6 fe80::a00:27ff:fe8f:a57b  prefixlen 64  scopeid 0x20<link>
        ether 08:00:27:8f:a5:7b  txqueuelen 1000  (Ethernet)
        RX packets 967  bytes 91405 (89.2 KiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 982  bytes 81125 (79.2 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
        
enp0s8: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.56.110  netmask 255.255.255.0  broadcast 192.168.56.255
        inet6 fe80::a00:27ff:fef3:c0da  prefixlen 64  scopeid 0x20<link>
        ether 08:00:27:f3:c0:da  txqueuelen 1000  (Ethernet)
        RX packets 953  bytes 103881 (101.4 KiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 561  bytes 126677 (123.7 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        inet6 ::1  prefixlen 128  scopeid 0x10<host>
        loop  txqueuelen 1  (Local Loopback)
        RX packets 242  bytes 13552 (13.2 KiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 242  bytes 13552 (13.2 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
# 查看系统中网卡数量
[root@server01 limits.d]# lspci|grep  Ethernet
00:03.0 Ethernet controller: Intel Corporation 82540EM Gigabit Ethernet Controller (rev 02)
00:08.0 Ethernet controller: Intel Corporation 82540EM Gigabit Ethernet Controller (rev 02)
```

检查网卡配置文件

/etc/sysconfig/network-scripts/

RHEL/Centos7.0以及之后的系统版本中，所有网络设置和管理都统一由NetworkManager服务来维护，相对于旧的/etc/init.d/network脚本管理方式，NetworkManager是动态的、事件驱动的网络管理服务。
关闭NetworkManager服务的命令如下：
[root@localhost network-scripts]# systemctl  stop NetworkManager
[root@localhost network-scripts]# systemctl  disable  NetworkManager
ip<-----ifconfig
ss<-----netstat

检查DNS解析文件是否设置正确
/etc/resolve.conf：可指定需要的域名服务器（注意NetworkManager会自动管理DNS解析文件）
nslookup命令诊断DNS解析功能

检查系统防火墙iptables和selinux的状态
iptables -L -n
more  /etc/selinux/config	（setenforce 1|0）

检查网络连通性以及路由信息
通过ping命令查看网络连通性
ping www.ixdba.net
通过route命令检查系统路由表信息是否正确
route -n
通过mtr、traceroute命令检查远程路由信息
mtr www.163.com
通过telnet、netstat命令检查主机端口与服务状态
telnet www.ixdba.net 80
netstat -antlp

# Linux线上服务器运维技巧与优化经验

## 线上Linux服务器基础优化

1. 最小化安装系统

   仅安装需要的，按需安装、不用不装，必须安装的有开发包、基本网络包、基本应用包。

2. ssh登录系统策略

   ```bash
   [root@server01 security]# vim /etc/ssh/sshd_config
   #SSH链接默认端口，修改默认22端口为1万以上端口号，避免被扫描和攻击。
   Port 22221
   #不使用DNS反查，可提高ssh连接速度
   UseDNS no
   #关闭GSSAPI验证，可提高ssh连接速度
   GSSAPIAuthentication no
   #禁止root账号远程登陆
   PermitRootLogin no
   ```

3. selinux， iptables策略设置

   ```bash
   [root@server01 security]# cat  /etc/selinux/config
   
   # This file controls the state of SELinux on the system.
   # SELINUX= can take one of these three values:
   #     enforcing - SELinux security policy is enforced.
   #     permissive - SELinux prints warnings instead of enforcing.
   #     disabled - No SELinux policy is loaded.
   # enforcing 开启状态、permissive 提醒的状态 、disabled 关闭状态
   # 命令行关闭：setenforce  0（临时生效）
   SELINUX=disabled
   # SELINUXTYPE= can take one of three two values:
   #     targeted - Targeted processes are protected,
   #     minimum - Modification of targeted policy. Only selected processes are protected. 
   #     mls - Multi Level Security protection.
   SELINUXTYPE=targeted 
   ```

4. 更新yum源及必要软件安装

   ```bash
   # 更新系统所有软件和内核
   [root@server01 security]# yum update
   ```

5. 定时自动更新服务器时间

   ```bash
   # 阿里云时间服务器
   [root@server01 /]# crontab -e
   # 设置BIOS/硬件时间定时更新：/sbin/hwclock -w
   /usr/sbin/ntpdate ntp1.aliyun.com >> /var/log/ntp.log 2>&1; /sbin/hwclock -w
   ```

6. 重要文件加锁

   ```bash
   # 加锁
   [root@server01 ~]# chattr +i  /etc/sudoer
   [root@server01 ~]# chattr +i  /etc/shadow
   [root@server01 ~]# chattr +i  /etc/passwd
   [root@server01 ~]# chattr +i  /etc/grub2.cfg
   # 查看锁状态
   [root@server01 /]# lsattr /etc/sudoers
   ----i----------- /etc/sudoers
   # 解锁
   [root@server01 /]# chattr -i /etc/sudoers 
   ```

7. 系统资源参数优化

   ```bash
   [root@server01 ~]# vim /etc/security/limits.conf
   [root@server01 ~]# vim /etc/security/limits.d/20-nproc.conf

## 系统安全与网络安全

### 常见攻击类型

口令暴力破解攻击
拒绝服务攻击（DDoS）
应用程序漏洞攻击（挂马、SQL注入）

### 防范攻击策略

![image](assets\linux-15.png)

### 操作系统常用安全策略

#### 密码登录安全

复杂密码+普通用户ssh登录
密钥认证方式远程登录
openssh配置文件/etc/ssh/sshd_config，注意关注下面几个配置项：
Port 22
AuthorizedKeysFile      .ssh/authorized_keys
PermitRootLogin no
GSSAPIAuthentication no
UseDNS no

#### 端口与服务安全

在linux操作系统下，系统共定义了65536个可用端口，这些端口又分为两个部分，以1024作为分割点， 分别是“只有root用户才能启用的port”和“客户端的port”：
0-1023端口，都需要以root身份才能启用，可以通过查阅linux下/etc/services文件得到端口与服务的对应列表。比如20、21端口是预留给ftp服务的，23端口是预留给telnet服务的，25是预留给mail服务的，而80是预留给www服务的。
1024以上（包含1024）的端口主要是给客户端软件使用的，这些端口都是有软件随机分配的，大于或者等于1024的端口的启用不受root用户的控制，例如，经常使用的mysql数据库，服务的默认端口是3306，而这个端口就是由mysql用户启用的。Oracle数据库默认的监听端口是1521，也是由oracle用户启动的。

#### 软件安全

yum update glibc
yum update openssl

#### 禁止ping操作

```bash
[root@server01 /]# echo "1" > /proc/sys/net/ipv4/icmp_echo_ignore_all
```

#### 设定tcp_wrappers防火墙

tcp_Wrappers的使用很简单，仅仅两个配置文件：/etc/hosts.allow和/etc/hosts.deny

## Linux软件防火墙iptables

### iptables的概念

iptables是linux系统内嵌的一个防火墙软件（封包过滤式防火墙），它集成在系统内核中，因此执行效率非常的高，iptables通过设置一些封包过滤规则，来定义什么数据可以接收，什么数据需要剔除，因此，用户通过iptables可以对进出计算机的数据包进行IP过滤，以达到保护主机的目的。iptables是有最基本的多个表格（tables）组成的，而且每个表格的用途都不一样，在每个表格中，又定义了多个链（chain），通过这些链可以设置相应的规则和策略.

### iptables的功能组成

![image](assets\linux-16.png)

### filter表

iptables有3种常用的表选项，包括管理本机数据进出的filter、管理防火墙内部主机nat 和改变不同包及包头内容的mangle。
filter表一般用于的信息包过滤，内置了INPUT、OUTPUT和FORWARD链。

1. INPUT链	主要是对外部数据包进入linux系统进行信息过滤
2. OUTPUT链	主要是对内部linux系统所要发送的数据包进行信息过滤
3. FORWARD链	将外面过来的数据包传递到内部计算机中

### NAT表

NAT表主要用处是网络地址转换，即Network Address Translation，缩写为NAT，它包含PREROUTING、POSTROUTING和OUTPUT链。

1. PREROUTING链：是在数据包刚刚到达防火墙时，根据需要改变它的目的地址。例如DNAT操作，就是通过一个合法的公网IP地址，通过对防火墙的访问，重定向到防火墙内的其它计算机（DMZ区域），也就是说通过防火墙改变了访问的目的地址，以使数据包能重定向到指定的主机。
2. POSTROUTING链：在包就要离开防火墙之前改变其源地址，例如SNAT操作，屏蔽了本地局域网主机的信息，本地主机通过防火墙连接到internet，这样在internet上看到的本地主机的来源地址都是同一个IP，屏蔽了来源主机地址信息。
3. OUTPUT链：改变了本地产生包的目的地址。

### 防火墙规则的查看与清除

```bash
# 列出当前系统filter表的链信息
[root@server01 /]# iptables -L -n
Chain INPUT (policy ACCEPT)
target     prot opt source               destination         

Chain FORWARD (policy ACCEPT)
target     prot opt source               destination         

Chain OUTPUT (policy ACCEPT)
target     prot opt source               destination   
# 列出当前系统nat表的链信息
[root@server01 /]# iptables -t nat -L -n
Chain PREROUTING (policy ACCEPT)
target     prot opt source               destination         

Chain INPUT (policy ACCEPT)
target     prot opt source               destination         

Chain OUTPUT (policy ACCEPT)
target     prot opt source               destination         

Chain POSTROUTING (policy ACCEPT)
target     prot opt source               destination 
# 清除本机防火墙的所有规则设定：下面三条指令可以清除防火墙的所有规则，但是不能清除预设的默认规则（policy）
[root@server01 /]# iptables -F
[root@server01 /]# iptables -X
[root@server01 /]# iptables -Z
```

### iptables的执行流程

![image](assets\linux-17.png)

### 制定防火墙规则

#### 设置预设规则（policy）

-P	--policy，定义策略，针对filter table有INPUT,OUTPUT,FORWARD3条链可选。
注意，这里的”P”为大写
-t	后面接table，常见的有filter、NAT等，如果没有用“-t”指定table，则默认使用filter表

```bash
# 预设规则先将进来的数据报全部drop，然后出去的数据报全部通过
# 由于防火墙是先读自定义规则，最后才去看预设规则，所以不用担心数据包进不来，可以在自定义规则中让它进来即可
[root@server01 ~]# iptables -P INPUT DROP
[root@server01 ~]# iptables -P OUTPUT ACCEPT
[root@server01 ~]# iptables -P FORWARD ACCEPT
```

#### 针对ip/网络、网络接口、tcp,udp协议的过滤规则

iptables设置语法如下：

```bash
[root@server01 ~]# iptables [-t tables] [-AI 链] [-io 网络接口] [-p 协议] [-s 来源IP/网络] [-d 目标IP/网络] -j [ACCEPT|DROP|REJECT|REDIRECT]
```

-A	新增加一条规则，放到已有规则的最后面
-I	插入一条规则，如果没有指定插入规则的顺序，则新插入的变成第一条规则，A和I后面跟链：INPUT、OUTPUT、FORWARD
-i	指定数据包进入的那个网络接口。linux下常见的有eth0、eth1、lo等等。此参数一般与INPUT链配合使用
-o	指定数据包传出的那个网络接口。经常与OUTPUT链配合使用
-p	指定此规则适用的协议，常用的协议有tcp、udp、icmp以及all
-s	指定来源IP或者网络，也就是限定数据包来源的IP或者网络，可以单独指定某个IP，也可以指定某段网络
-d	指定目标IP或者网络，跟参数“-s”类似
-j	此参数后面指定要执行的动作，主要的动作有接受(ACCEPT)、抛弃(DROP)及记录(LOG)
	ACCEPT	接受该数据包
	DROP	直接丢弃该数据包，不给客户端任何回应。
-m  指定iptables使用的扩展模块，常用的有tcp模块等
--sport 端口范围	限制来源的端口号码，端口号码可以是连续的，例如 1024:65535
--dport 端口范围	限制目标的端口号码

```bash
# --dport为目标端口，即服务器上的端口
# 整条规则的含义：仅仅允许客户端192.168.56.1通过tcp连接服务器的22端口
[root@server01 /]# iptables -A INPUT -s 192.168.56.1 -p tcp -m tcp --dport 22 -j ACCEPT
# 此时由于设置预设规则中的INPUT为ACCEPT，所以上面设置的规则没有效果，必须将INPUT链设置为DROP状态
[root@server01 ~]# iptables -L -n
Chain INPUT (policy ACCEPT)
target     prot opt source               destination         
ACCEPT     tcp  --  192.168.56.1         0.0.0.0/0            tcp dpt:22

Chain FORWARD (policy ACCEPT)
target     prot opt source               destination         

Chain OUTPUT (policy ACCEPT)
target     prot opt source               destination 
# 只有将INPUT链设置为DROP状态上面的规则才能生效
[root@server01 ~]# iptables -P INPUT DROP
[root@server01 ~]# iptables -L -n        
Chain INPUT (policy DROP)
target     prot opt source               destination         
ACCEPT     tcp  --  192.168.56.1         0.0.0.0/0            tcp dpt:22

Chain FORWARD (policy ACCEPT)
target     prot opt source               destination         

Chain OUTPUT (policy ACCEPT)
target     prot opt source               destination
# 执行上述规则后，在192.168.56.111上无法通过ssh连接server01了
[root@server02 ~]# ssh server01
ssh: connect to host server01 port 22: Connection timed out
# 允许所有客户端访问服务器的80端口
[root@server01 ~]# iptables -A INPUT -p tcp -m tcp --dport 80 -j ACCEPT 
# 允许192.168.56.1访问服务器的任意端口
[root@server01 ~]# iptables -A INPUT -s 192.168.56.1 -j ACCEPT
# 将来自网络接口lo的数据包，全部接受
[root@server01 ~]# iptables -A INPUT -i lo -j ACCEPT
# 允许通过icmp协议访问主机的数据包通过
[root@server01 ~]# iptables -A INPUT -i enp0s8 -p icmp -j ACCEPT
# 允许局域网内192.168.56.0/24的所有主机访问我们这个服务器，除了192.168.56.111这台主机：
[root@server01 ~]#iptables -A INPUT -i enp0s8 -s 192.168.56.111  -j DROP
[root@server01 ~]#iptables -A INPUT -i enp0s8 -s 192.168.56.0/24 -j  ACCEPT
# 允许来自222.91.99.0/28的1024:65535端口范围的主机可以通过22端口连接linux服务器
[root@server01 ~]#iptables -A INPUT -i eth0 -p tcp -s  222.91.99.0/28  --sport 1024:65534 --dport 22 -j  ACCEPT
```

#### 针对数据状态模块的过滤规则

数据状态模块机制是iptables中特殊的一部分，严格来说不应该叫状态机制，因为它只是一种连接跟踪机制。连接跟踪可以让filter table知道某个特定连接的状态。

```bash
[root@server01 ~]# iptables -A INPUT -m state/mac --state NEW/ESTABLISHED/RELATED/INVALID
```

-m	iptables的几个模块选项，常见的有：

1. state：状态模块
2. mac：网卡硬件地址（hardware address）

--state	一些数据封包的状态，主要有：

1. NEW：某个连接的第一个包。
2. ESTABLISHED：表示该封包属于某个已经建立的链接。
3. RELATED：当一个连接和某个已处于ESTABLISHED状态的连接有关系时，就被认为是RELATED的了。
4. INVALID：表示数据包不能被识别属于哪个连接或没有任何状态。

```bash
# 只要是已建立的连接或者相关数据包就予以通过，不能识别或者没有任何状态的数据包全部丢弃，设置如下规则：
[root@server01 ~]#iptables -A INPUT -m   state --state RELATED,ESTABLISHED -j ACCEPT
[root@server01 ~]#iptables -A INPUT -m   state --state INVALID -j DROP
```

#### NAT表SNAT操作

```bash
# 比如有两台机器：172.16.213.78和172.16.213.51，
# 172.16.213.78不能上网；
# 172.16.213.51可以上网，172.16.213.51中连接外网的IP为171.84.4.131
# 那么可以在172.16.213.51中添加一条规则：
[root@localhost ~]# iptables -t nat -A POSTROUTING -s 172.16.213.78/32 -j SNAT --to-source 171.84.4.131
# 然后再在172.16.213.78中添加一条路由规则：
[root@localhost ~]# route add default gw 172.16.213.51
```

## rsync数据镜像工具与应用案例

### rsync功能介绍

rsync是Linux系统下的数据镜像备份工具，通过rsync可以将本地系统数据通过网络备份到任何远程主机上。
rsync有如下特性：
可以镜像保存整个目录树和文件系统
可以增量同步数据，文件传输效率高，因而同步时间很短。
可以保持原有文件的权限、时间等属性。
加密传输数据，保证了数据的安全性。
rsync有两种应用模式：client/server模式、client/client模式。

### rsync的client/server模式

client/server模式下，是在server端启动一个服务端口，然后客户端来连接这个端口，进行数据的同步和传输。
服务端设置：
rsync服务端的配置文件为/etc/rsyncd.conf
uid = nobody
gid = nobody
use chroot = no
max connections = 10
pid file = /var/run/rsyncd.pid
lock file = /var/run/rsync.lock
log file = /var/log/rsyncd.log
[ixdba]
path = /webdata
comment = ixdba file
ignore errors
read only = true
list = false
uid = root
gid = root
auth users = backup
secrets file = /etc/server.pass

uid    此选项指定当该模块传输文件时守护进程应该具有的用户ID，默认值是“nobody”。
gid   此选项指定当该模块传输文件时守护进程应该具有的用户组ID。默认值为“nobody”。
max connections  此选项指定模块的最大并发连接数量，以保护服务器，超过限制的连接请求，将被暂时限制。默认值是0，也就是没有限制。
pid file 此选项用来指定rsync守护进程对应的PID文件路径。
lock file 此选择指定支持max connections的锁文件，默认值是/var/run/rsyncd.lock。
log file  此选项指定了rsync的日志输出文件路径。
[ixdba] 表示定义一个模块的开始，ixdba就是对应的模块名称。
path  此选项用来指定需要备份的文件或目录，必填项，这里指定的目录为/webdata。
list  此选项设定当客户请求可以使用的模块列表时，该模块是否被列出。默认值是true，如果需要建立隐藏的模块。可以设置为false。
auth users 此选项用来定义可以连接该模块的用户名，多个用户用空格或逗号分隔开。需要注意的是这里的用户和Linux系统用户没有任何关系。这里指定的用户是backup。
secrets file  此选项指定一个包含“用户名:密码”格式的文件，用户名就是“auth users”选项定义的用户，密码可以随便指定，只要和客户端的secrets file对应起来就行。只有在auth users被定义时，该文件才起作用。系统默认没有这个文件，自己手动创建一个即可。

服务端执行如下指令启动rsync守护进程：

```bash
[root@server01 ~]# /usr/local/bin/rsync --daemon
[root@server01 ~]# ps -ef|grep rsync
root     20278     1  0 16:29 ?        00:00:00 /usr/local/bin/rsync --daemon
```

客户端拉取：

```bash
[root@server01 ~]# /usr/local/bin/rsync -vzrtopg --delete --progress backup@192.168.56.110::ixdba  /data  --password-file=/etc/server.pass
```

1. “--vzrtopg”选项中v是“-verbose”，即详细模式输出，z表示“--compress” 即对备份的文件在传输时进行压缩处理，r表示“--recursive”，也就是对子目录以递归模式处理。t即“--times”，用来保持文件时间信息，o即“--owner”用来保持文件属主信息。p即“--perms”用来保持文件权限，g即“--group”用来保持文件的属组信息。
2. “--delete”选项指定以rsync服务端为基准进行数据镜像同步，也就是要保持rsync服务端目录与客户端目录的完全一致性。
3. “--progress”选项用于显示数据镜像同步的过程。
4. “backup@192.168.60.253::ixdba” 表示对服务器192.168.60.253中的ixdba模块进行备份，也就是指定备份的模块，backup表示使用“backup”这个用户对该模块进行备份。
5. “/data”用于指定备份文件在客户端机器上的存放路径，也就是将备份的文件存放在备份机的/data目录下。
6. “--password-file=/etc/server.pass”用来指定客户机上存放的密码文件位置，这样在客户端执行同步命令时就无需输入交互密码了，注意，这个密码文件的名称和位置可以随意指定，但是在客户机上必须存在此文件，文件的内容仅仅为备份用户的密码，这里指的是backup的密码。

```bash
# 推送模式 相当于scp
[root@server01 ~]# rsync -vzrtopg --delete --progress   Python-3.6.5.tgz  root@172.16.213.233:/mnt
# 拉取模式  相当于scp
[root@server01 ~]# rsync -vzrtopg --delete --progress   root@172.16.213.233:/mnt/Python-3.6.5.tgz  /app/
# 默认情况下rsync走的是ssh协议，22端口，如果ssh是非默认的22端口，那么可以添加“-e“选项：
[root@server01 ~]# rsync -vzrtopg --delete --progress -e 'ssh -p 9090'  Python-3.6.5.tgz  root@172.16.213.233:/mnt
[root@server01 ~]# rsync -vzrtopg --delete --progress -e 'ssh -p 9090'   root@172.16.213.233:/mnt/Python-3.6.5.tgz  /app/
```

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

以百分比的形式报告内存使用情况,可以清楚观察每个进程占用的比重是多少

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

指定查看某个用户进程使用内存大小

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

### 磁盘读写速度查询

```bash
# 通过往磁盘中写入8GB数据来测试磁盘的读写速度
[root@server03 ~]# dd if=/dev/zero of=$PWD/1.img bs=1G count=8 oflag=dsync
8+0 records in
8+0 records out
8589934592 bytes (8.6 GB) copied, 123.37 s, 69.6 MB/s
```

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

## dstat 

dstat 可以监测CPU、磁盘、网络流量、IO、内存等，是一个全能的系统信息统计工具。可以替代 vmstat、iostat、netstat、nfsstat 、ifstat 等命令。如果系统没有安装，可以通过sudo yum install dstat来安装。

dstat 支持即时刷新，有着彩色的界面，数据指标更加直观明了。

### 默认

默认情况将输出CPU、磁盘、网络、IO、内存 等统计信息。

```bash
[root@server02 ~]# dstat
You did not select any stats, using -cdngy by default.
----total-cpu-usage---- -dsk/total- -net/total- ---paging-- ---system--
usr sys idl wai hiq siq| read  writ| recv  send|  in   out | int   csw 
  2   2  95   1   0   0| 948k  377k|   0     0 |   0     0 | 234   626 
  0   0 100   0   0   0|   0     0 | 864B 1722B|   0     0 | 155   186 
  0   0 100   0   0   0|   0     0 | 864B 1194B|   0     0 | 110   157 
  0   1  99   0   0   0|   0     0 | 864B 1194B|   0     0 | 105   142 
  0   0 100   0   0   0|   0    96k| 864B 1194B|   0     0 | 134   165 
  0   1  99   0   0   0|   0     0 | 864B 1194B|   0     0 | 126   157 
  1   0  99   0   0   0|   0     0 | 864B 1194B|   0     0 | 116   155 
  0   0 100   0   0   0|   0     0 | 864B 1194B|   0     0 | 115   158 
```

### 查看CPU

usr用户占比，sys系统占比，idl空闲占比，wai等待次数；hiq硬中断次数，siq软中断次数。

```bash
[root@server02 ~]# dstat -c
----total-cpu-usage----
usr sys idl wai hiq siq
  2   2  95   1   0   0
  0   0 100   0   0   0
  0   0  99   0   0   1
```

### 查看磁盘I/O

```bash
[root@server02 ~]# dstat -d
-dsk/total-
 read  writ
 829k  357k
   0   232k
   0     0 
   0     0 
   0     0 
```

### 查看CPU平均负载

```bash
[root@server02 ~]# dstat -l
---load-avg---
 1m   5m  15m 
0.02 0.05 0.05
0.02 0.05 0.05
0.02 0.05 0.05
0.02 0.05 0.05
```

### 查看网卡流量

```bash
[root@server02 ~]# dstat -n
-net/total-
 recv  send
   0     0 
 864B 1050B
 924B 1062B
 864B 1002B
 924B 1062B
```

# Linux虚拟内存管理

在程序中编写业务逻辑代码的时候，往往需要引用这些创建出来的数据结构，并通过这些引用对相关数据结构进行业务处理。当程序运行起来之后就变成了进程，而这些业务数据结构的引用在进程的视角里全都都是虚拟内存地址，因为进程无论是在用户态还是在内核态能够看到的都是虚拟内存空间，物理内存空间被操作系统所屏蔽进程是看不到的。进程通过虚拟内存地址访问这些数据结构的时候，虚拟内存地址会在内存管理子系统中被转换成物理内存地址，通过物理内存地址就可以访问到真正存储这些数据结构的物理内存了。随后就可以对这块物理内存进行各种业务操作，从而完成业务逻辑。

## 为什么要使用虚拟地址访问内存

假设现在没有虚拟内存地址，在程序中对内存的操作全都都是使用物理内存地址，在这种情况下，程序员就需要精确的知道每一个变量在内存中的具体位置，需要手动对物理内存进行布局，明确哪些数据存储在内存的哪些位置，除此之外还需要考虑为每个进程究竟要分配多少内存？内存紧张的时候该怎么办？如何避免进程与进程之间的地址冲突？等等一系列复杂且琐碎的细节。如果在单进程系统中比如嵌入式设备上开发应用程序，系统中只有一个进程，这单个进程独享所有的物理资源包括内存资源。在这种情况下，上述提到的这些直接使用物理内存的问题可能还好处理一些，但是仍然具有很高的开发门槛。然而在现代操作系统中往往支持多个进程，需要处理多进程之间的协同问题，在多进程系统中直接使用物理内存地址操作内存所带来的上述问题就变得非常复杂了。这里笔者为大家举一个简单的例子来说明在多进程系统中直接使用物理内存地址的复杂性。比如现在有这样一个简单的 Java 程序。

```java
public static void main(String[] args) throws Exception {

    string i = args[0];
}
```

在程序代码相同的情况下，用这份代码同时启动三个JVM进程，暂时将进程依次命名为a,b,c。这三个进程用到的代码是一样的，都是提前写好的，可以被多次运行。由于是直接操作物理内存地址，假设变量i保存在0x354这个物理地址上。这三个进程运行起来之后，同时操作这个0x354物理地址，这样这个变量i的值不就混乱了吗？三个进程就会出现变量的地址冲突。

![image](assets\linux-18.png)

所以在直接操作物理内存的情况下，需要知道每一个变量的位置都被安排在了哪里，而且还要注意和多个进程同时运行的时候，不能共用同一个地址，否则就会造成地址冲突。现实中一个程序会有很多的变量和函数，这样一来给它们都需要计算一个合理的位置，还不能与其他进程冲突，这就很复杂了。那么该如何解决这个问题呢？可以利用程序的局部性原理

>程序局部性原理表现为：时间局部性和空间局部性。时间局部性是指如果程序中的某条指令一旦执行，则不久之后该指令可能再次被执行；如果某块数据被访问，则不久之后该数据可能再次被访问。空间局部性是指一旦程序访问了某个存储单元，则不久之后，其附近的存储单元也将被访问。

从程序局部性原理的描述中我们可以得出这样一个结论：进程在运行之后，对于内存的访问不会一下子就要访问全部的内存，相反进程对于内存的访问会表现出明显的倾向性，更加倾向于访问最近访问过的数据以及热点数据附近的数据。根据这个结论我们就清楚了，无论一个进程实际可以占用的内存资源有多大，根据程序局部性原理，在某一段时间内，进程真正需要的物理内存其实是很少的一部分，只需要为每个进程分配很少的物理内存就可以保证进程的正常执行运转。而虚拟内存的引入正是要解决上述的问题，虚拟内存引入之后，进程的视角就会变得非常开阔，每个进程都拥有自己独立的虚拟地址空间，进程与进程之间的虚拟内存地址空间是相互隔离，互不干扰的。每个进程都认为自己独占所有内存空间，自己想干什么就干什么。系统上还运行了哪些进程和我没有任何关系。这样一来就可以将多进程之间协同的相关复杂细节统统交给内核中的内存管理模块来处理，极大地解放了程序员的负担。这一切都是因为虚拟内存能够提供内存地址空间的隔离，极大地扩展了可用空间。

![image](assets\linux-19.png)

这样进程就以为自己独占了整个内存空间资源，给进程产生了所有内存资源都属于它自己的幻觉，这其实是CPU和操作系统使用的一个障眼法罢了，任何一个虚拟内存里所存储的数据，本质上还是保存在真实的物理内存里的。只不过内核帮我们做了虚拟内存到物理内存的这一层映射，将不同进程的虚拟地址和不同内存的物理地址映射起来。当CPU访问进程的虚拟地址时，经过地址翻译硬件将虚拟地址转换成不同的物理地址，这样不同的进程运行的时候，虽然操作的是同一虚拟地址，但其实背后写入的是不同的物理地址，这样就不会冲突了。

## 进程虚拟内存空间

首先是一个进程运行起来是为了执行我们交代给进程的工作，执行这些工作的步骤我们通过程序代码事先编写好，然后编译成二进制文件存放在磁盘中，CPU会执行二进制文件中的机器码来驱动进程的运行。所以在进程运行之前，这些存放在二进制文件中的机器码需要被加载进内存中，而用于存放这些机器码的虚拟内存空间叫做代码段。

在程序运行起来之后，总要操作变量吧，在程序代码中我们通常会定义大量的全局变量和静态变量，这些全局变量在程序编译之后也会存储在二进制文件中，在程序运行之前，这些全局变量也需要被加载进内存中供程序访问。所以在虚拟内存空间中也需要一段区域来存储这些全局变量。

- 那些在代码中被我们指定了初始值的全局变量和静态变量在虚拟内存空间中的存储区域我们叫做数据段。
- 那些没有指定初始值的全局变量和静态变量在虚拟内存空间中的存储区域我们叫做BSS段。这些未初始化的全局变量被加载进内存之后会被初始化为0值。

上面介绍的这些全局变量和静态变量都是在编译期间就确定的，但是我们程序在运行期间往往需要动态的申请内存，所以在虚拟内存空间中也需要一块区域来存放这些动态申请的内存，这块区域就叫做堆。注意这里的堆指的是OS堆并不是JVM中的堆。

除此之外，程序在运行过程中还需要依赖动态链接库，这些动态链接库以.so文件的形式存放在磁盘中，比如C程序中的glibc，里边对系统调用进行了封装。glibc库里提供的用于动态申请堆内存的malloc函数就是对系统调用sbrk和mmap的封装。这些动态链接库也有自己的对应的代码段，数据段，BSS段，也需要一起被加载进内存中。还有用于内存文件映射的系统调用mmap，会将文件与内存进行映射，那么映射的这块内存（虚拟内存）也需要在虚拟地址空间中有一块区域存储。这些动态链接库中的代码段，数据段，BSS 段，以及通过mmap系统调用映射的共享内存区，在虚拟内存空间的存储区域叫做文件映射与匿名映射区。

最后我们在程序运行的时候总该要调用各种函数吧，那么调用函数过程中使用到的局部变量和函数参数也需要一块内存区域来保存。这一块区域在虚拟内存空间中叫做栈。

综上所述，内核根据进程运行的过程中所需要不同种类的数据而为其开辟了对应的地址空间。分别为：

- 用于存放进程程序二进制文件中的机器指令的代码段
- 用于存放程序二进制文件中定义的全局变量和静态变量的数据段和 BSS 段。
- 用于在程序运行过程中动态申请内存的堆。
- 用于存放动态链接库以及内存映射区域的文件映射与匿名映射区。
- 用于存放函数调用过程中的局部变量和函数参数的栈。

![image](assets\linux-20.png)

## Linux进程虚拟内存空间

上面介绍了进程虚拟内存空间中各个内存区域的一个大概分布，在此基础之上，分别从32位和64位机器上看下在Linux系统中进程虚拟内存空间的真实分布情况。

### 32位机器上进程虚拟内存空间分布

在32位机器上，指针的寻址范围为2^32，所能表达的虚拟内存空间为4GB。所以在32位机器上进程的虚拟内存地址范围为：0x00000000-0xFFFFFFFF。
其中用户态虚拟内存空间为3GB，虚拟内存地址范围为：0x00000000-0xC000000。
内核态虚拟内存空间为1GB，虚拟内存地址范围为：0xC000000-0xFFFFFFFF。

![image](assets\linux-21.png)

但是用户态虚拟内存空间中的代码段并不是从0x00000000地址开始的，而是从0x08048000地址开始。
0x00000000到0x08048000这段虚拟内存地址是一段不可访问的保留区，因为在大多数操作系统中，数值比较小的地址通常被认为不是一个合法的地址，这块小地址是不允许访问的。比如在C语言中通常会将一些无效的指针设置为NULL，指向这块不允许访问的地址。
保留区的上边就是代码段和数据段，它们是从程序的二进制文件中直接加载进内存中的，BSS段中的数据也存在于二进制文件中，因为内核知道这些数据是没有初值的，所以在二进制文件中只会记录BSS段的大小，在加载进内存时会生成一段0填充的内存空间。
紧挨着BSS段的上边就是经常使用到的堆空间，从图中的红色箭头可以知道在堆空间中地址的增长方向是从低地址到高地址增长。
内核中使用start_brk标识堆的起始位置，brk标识堆当前的结束位置。当堆申请新的内存空间时，只需要将brk指针增加对应的大小，回收地址时减少对应的大小即可。比如当通过malloc向内核申请很小的一块内存时（128K之内），就是通过改变brk位置实现的。
堆空间的上边是一段待分配区域，用于扩展堆空间的使用。接下来就来到了文件映射与匿名映射区域。进程运行时所依赖的动态链接库中的代码段，数据段，BSS段就加载在这里。还有调用mmap映射出来的一段虚拟内存空间也保存在这个区域。注意：在文件映射与匿名映射区的地址增长方向是从高地址向低地址增长。
接下来用户态虚拟内存空间的最后一块区域就是栈空间了，在这里会保存函数运行过程所需要的局部变量以及函数参数等函数调用信息。栈空间中的地址增长方向是从高地址向低地址增长。每次进程申请新的栈地址时，其地址值是在减少的。
在内核中使用start_stack标识栈的起始位置，RSP寄存器中保存栈顶指针stackpointer，RBP寄存器中保存的是栈基地址。
在栈空间的下边也有一段待分配区域用于扩展栈空间，在栈空间的上边就是内核空间了，进程虽然可以看到这段内核空间地址，但是就是不能访问。

### 64位机器上进程虚拟内存空间分布

上小节中介绍的32位虚拟内存空间布局和本小节即将要介绍的64位虚拟内存空间布局都可以通过cat/proc/pid/maps或者pmappid来查看某个进程的实际虚拟内存布局。在32位机器上，指针的寻址范围为2^32，所能表达的虚拟内存空间为4GB。在64位机器上，指针的寻址范围为2^64，所能表达的虚拟内存空间为16EB。虚拟内存地址范围为：0x00000000000000000000-0xFFFFFFFFFFFFFFFF。但是在现实情况中根本不会用到这么大范围的内存空间，事实上在目前的64位系统下只使用了48位来描述虚拟内存空间，寻址范围为2^48，所能表达的虚拟内存空间为256TB。其中低128T表示用户态虚拟内存空间，虚拟内存地址范围为：0x0000000000000000-0x00007FFFFFFFF000。高128T表示内核态虚拟内存空间，虚拟内存地址范围为：0xFFFF800000000000-0xFFFFFFFFFFFFFFFF。这样一来就在用户态虚拟内存空间与内核态虚拟内存空间之间形成了一段0x00007FFFFFFFF000-0xFFFF800000000000的地址空洞，这个空洞被叫做canonicaladdress空洞。

![image](assets\linux-22.png)

那么这个canonical-address空洞是如何形成的呢？在64位机器上的指针寻址范围为2^64，但是在实际使用中只使用了其中的低48位来表示虚拟内存地址，那么这多出的高16位就形成了这个地址空洞。在低128T的用户态地址空间：0x0000000000000000-0x00007FFFFFFFF000范围中，所以虚拟内存地址的高16位全部为0。如果一个虚拟内存地址的高16位全部为0，那么就可以直接判断出这是一个用户空间的虚拟内存地址。同样的道理，在高128T的内核态虚拟内存空间：0xFFFF800000000000-0xFFFFFFFFFFFFFFFF范围中，所以虚拟内存地址的高16位全部为1。也就是说内核态的虚拟内存地址的高16位全部为1，如果一个试图访问内核的虚拟地址的高16位不全为1，则可以快速判断这个访问是非法的。这个高16位的空闲地址被称为canonical。如果虚拟内存地址中的高16位全部为0（表示用户空间虚拟内存地址）或者全部为1（表示内核空间虚拟内存地址），这种地址的形式叫做canonicalform，对应的地址称作canonical-address。那么处于canonical-address空洞：0x00007FFFFFFFF000-0xFFFF800000000000范围内的地址的高16位不全为0也不全为1。如果某个虚拟地址落在这段canonical-address空洞区域中，那就是既不在用户空间，也不在内核空间，肯定是非法访问了。未来也可以利用这块canonical-address空洞，来扩展虚拟内存地址的范围，比如扩展到56位。在理解了canonical-address这个概念之后，再来看下64位Linux系统下的真实虚拟内存空间布局情况：

![image](assets\linux-23.png)

从上图中可以看出64位系统中的虚拟内存布局和32位系统中的虚拟内存布局大体上是差不多的。主要不同的地方有三点：

1. 就是前边提到的由高16位空闲地址造成的canonical-address空洞。在这段范围内的虚拟内存地址是不合法的，因为它的高16位既不全为0也不全为1，不是一个canonical-address，所以称之为canonical-address空洞。
2. 在代码段跟数据段的中间还有一段不可以读写的保护段，它的作用是防止程序在读写数据段的时候越界访问到代码段，这个保护段可以让越界访问行为直接崩溃，防止它继续往下运行。
3. 用户态虚拟内存空间与内核态虚拟内存空间分别占用128T，其中低128T分配给用户态虚拟内存空间，高128T分配给内核态虚拟内存空间。

## 进程虚拟内存空间的管理

进程的虚拟内存空间管理，首先是进程在内核中的描述符task_struct结构

```c
struct task_struct {
    // 进程id
    pid_t        pid;
    // 用于标识线程所属的进程 pid
    pid_t        tgid;
    // 进程打开的文件信息
    struct files_struct    *files;
    // 内存描述符表示进程虚拟地址空间
    struct mm_struct    *mm;
    // .......... 省略 .......
}
```

在进程描述符task_struct结构中，有一个专门描述进程虚拟地址空间的内存描述符mm_struct结构，这个结构体中包含了前边几个小节中介绍的进程虚拟内存空间的全部信息。每个进程都有唯一的mm_struct结构体，也就是前边提到的每个进程的虚拟地址空间都是独立，互不干扰的。当调用fork()函数创建进程的时候，表示进程地址空间的mm_struct结构会随着进程描述符task_struct的创建而创建。

```c
long _do_fork(unsigned long clone_flags,
        unsigned long stack_start,
        unsigned long stack_size,
        int __user *parent_tidptr,
        int __user *child_tidptr,
        unsigned long tls)
{
  //......... 省略 ..........
  struct pid *pid;
  struct task_struct *p;

  //......... 省略 ..........
  // 为进程创建 task_struct 结构，用父进程的资源填充 task_struct 信息
  p = copy_process(clone_flags, stack_start, stack_size,
       child_tidptr, NULL, trace, tls, NUMA_NO_NODE);

  //......... 省略 ..........
}
```

随后会在copy_process函数中创建task_struct结构，并拷贝父进程的相关资源到新进程的task_struct结构里，其中就包括拷贝父进程的虚拟内存空间mm_struct结构。这里可以看出子进程在新创建出来之后它的虚拟内存空间是和父进程的虚拟内存空间一模一样的，直接拷贝过来。

```c
static __latent_entropy struct task_struct *copy_process(
          unsigned long clone_flags,
          unsigned long stack_start,
          unsigned long stack_size,
          int __user *child_tidptr,
          struct pid *pid,
          int trace,
          unsigned long tls,
          int node)
{

    struct task_struct *p;
    // 创建 task_struct 结构
    p = dup_task_struct(current, node);

    //....... 初始化子进程 ...........

    //....... 开始继承拷贝父进程资源  .......      
    // 继承父进程打开的文件描述符
    retval = copy_files(clone_flags, p);
    // 继承父进程所属的文件系统
    retval = copy_fs(clone_flags, p);
    // 继承父进程注册的信号以及信号处理函数
    retval = copy_sighand(clone_flags, p);
    retval = copy_signal(clone_flags, p);
    // 继承父进程的虚拟内存空间
    retval = copy_mm(clone_flags, p);
    // 继承父进程的 namespaces
    retval = copy_namespaces(clone_flags, p);
    // 继承父进程的 IO 信息
    retval = copy_io(clone_flags, p);

    //...........省略.........
    // 分配 CPU
    retval = sched_fork(clone_flags, p);
    // 分配 pid
    pid = alloc_pid(p->nsproxy->pid_ns_for_children);

    //..........省略.........
}
```

这里重点关注copy_mm函数，正是在这里完成了子进程虚拟内存空间mm_struct结构的的创建以及初始化。

```c
static int copy_mm(unsigned long clone_flags, struct task_struct *tsk)
{
    // 子进程虚拟内存空间，父进程虚拟内存空间
    struct mm_struct *mm, *oldmm;
    int retval;

    //...... 省略 ......

    tsk->mm = NULL;
    tsk->active_mm = NULL;
    // 获取父进程虚拟内存空间
    oldmm = current->mm;
    if (!oldmm)
        return 0;

    //...... 省略 ......
    // 通过 vfork 或者 clone 系统调用创建出的子进程（线程）和父进程共享虚拟内存空间
    if (clone_flags & CLONE_VM) {
        // 增加父进程虚拟地址空间的引用计数
        mmget(oldmm);
        // 直接将父进程的虚拟内存空间赋值给子进程（线程）
        // 线程共享其所属进程的虚拟内存空间
        mm = oldmm;
        goto good_mm;
    }

    retval = -ENOMEM;
    // 如果是 fork 系统调用创建出的子进程，则将父进程的虚拟内存空间以及相关页表拷贝到子进程中的 mm_struct 结构中。
    mm = dup_mm(tsk);
    if (!mm)
        goto fail_nomem;

    good_mm:
    // 将拷贝出来的父进程虚拟内存空间 mm_struct 赋值给子进程
    tsk->mm = mm;
    tsk->active_mm = mm;
    return 0;

    //...... 省略 ......
}
```

由于本小节中举的示例是通过fork()函数创建子进程的情形，所以这里先占时忽略if(clone_flags&CLONE_VM)这个条件判断逻辑，copy_mm函数首先会将父进程的虚拟内存空间current->mm赋值给指针oldmm。然后通过dup_mm函数将父进程的虚拟内存空间以及相关页表拷贝到子进程的mm_struct结构中。最后将拷贝出来的mm_struct赋值给子进程的task_struct结构。

>通过fork()函数创建出的子进程，它的虚拟内存空间以及相关页表相当于父进程虚拟内存空间的一份拷贝，直接从父进程中拷贝到子进程中。

而当通过vfork或者clone系统调用创建出的子进程，首先会设置CLONE_VM标识，这样来到copy_mm函数中就会进入if (clone_flags & CLONE_VM)条件中，在这个分支中会将父进程的虚拟内存空间以及相关页表直接赋值给子进程。这样一来父进程和子进程的虚拟内存空间就变成共享的了。也就是说父子进程之间使用的虚拟内存空间是一样的，并不是一份拷贝。子进程共享了父进程的虚拟内存空间，这样子进程就变成了熟悉的线程，是否共享地址空间几乎是进程和线程之间的本质区别。Linux内核并不区别对待它们，线程对于内核来说仅仅是一个共享特定资源的进程而已。

内核线程和用户态线程的区别就是内核线程没有相关的内存描述符mm_struct，内核线程对应的task_struct结构中的mm域指向Null，所以内核线程之间调度是不涉及地址空间切换的。当一个内核线程被调度时，它会发现自己的虚拟地址空间为Null，虽然它不会访问用户态的内存，但是它会访问内核内存，聪明的内核会将调度之前的上一个用户态进程的虚拟内存空间mm_struct直接赋值给内核线程，因为内核线程不会访问用户空间的内存，它仅仅只会访问内核空间的内存，所以直接复用上一个用户态进程的虚拟地址空间就可以避免为内核线程分配mm_struct和相关页表的开销，以及避免内核线程之间调度时地址空间的切换开销。

> 父进程与子进程的区别，进程与线程的区别，以及内核线程与用户态线程的区别其实都是围绕着这个 mm_struct 展开的。

### 内核如何划分用户态和内核态虚拟内存空间

进程的虚拟内存空间分为两个部分：一部分是用户态虚拟内存空间，另一部分是内核态虚拟内存空间。那么用户态的地址空间和内核态的地址空间在内核中是如何被划分的呢？这就用到了进程的内存描述符mm_struct结构体中的task_size变量，task_size定义了用户态地址空间与内核态地址空间之间的分界线。

```c
struct mm_struct {
    unsigned long task_size;  /* size of task vm space */
}
```

通过前边小节的内容介绍可知在32位系统中用户态虚拟内存空间为3GB，虚拟内存地址范围为：0x00000000-0xC000000。内核态虚拟内存空间为1GB，虚拟内存地址范围为：0xC000000 - 0xFFFFFFFF。

![image](assets\linux-24.png)

32位系统中用户地址空间和内核地址空间的分界线在0xC000000地址处，那么自然进程的mm_struct结构中的task_size为0xC000000。再来看下内核在/arch/x86/include/asm/page_32_types.h文件中关于TASK_SIZE的定义。

```c
/*
 * User space process size: 3GB (default).
 */
#define TASK_SIZE    __PAGE_OFFSET
```

如下图所示：__PAGE_OFFSET 的值在 32 位系统下为  0xC000 000。

![image](assets\linux-25.png)

而在64位系统中，只使用了其中的低48位来表示虚拟内存地址。其中用户态虚拟内存空间为低128T，虚拟内存地址范围为：0x0000000000000000-0x00007FFFFFFFF000。内核态虚拟内存空间为高128T，虚拟内存地址范围为：0xFFFF800000000000-0xFFFFFFFFFFFFFFFF。

![image](assets\linux-26.png)

64位系统中用户地址空间和内核地址空间的分界线在0x00007FFFFFFFF000地址处，那么自然进程的mm_struct结构中的task_size为0x00007FFFFFFFF000。再来看下内核在/arch/x86/include/asm/page_64_types.h文件中关于TASK_SIZE的定义。

```c
#define TASK_SIZE    (test_thread_flag(TIF_ADDR32) ? \
          IA32_PAGE_OFFSET : TASK_SIZE_MAX)

#define TASK_SIZE_MAX    task_size_max()

#define task_size_max()    ((_AC(1,UL) << __VIRTUAL_MASK_SHIFT) - PAGE_SIZE)

#define __VIRTUAL_MASK_SHIFT  47
```

来看下在64位系统中内核如何来计算TASK_SIZE，在task_size_max()的计算逻辑中1左移47位得到的地址是0x0000800000000000，然后减去一个PAGE_SIZE（默认为4K），就是0x00007FFFFFFFF000，共128T。所以在64位系统中的TASK_SIZE为0x00007FFFFFFFF000。

> 这里可以看出，64 位虚拟内存空间的布局是和物理内存页page的大小有关的，物理内存页page默认大小 PAGE_SIZE为4K。

PAGE_SIZE 定义在 /arch/x86/include/asm/page_types.h文件中：

```c
/* PAGE_SHIFT determines the page size */
#define PAGE_SHIFT    12
#define PAGE_SIZE    (_AC(1,UL) << PAGE_SHIFT)
```

而内核空间的起始地址是0xFFFF 8000 0000 0000 。在 0x00007FFFFFFFF000 - 0xFFFF 8000 0000 0000之间的内存区域就是在前面64位机器上进程虚拟内存空间分布小节中介绍的canonical address空洞。

### 内核如何布局进程虚拟内存空间

接下来介绍内核是如何划分进程虚拟内存空间中的这些内存区域的

![image](assets\linux-27.png)

前边提到，内核中采用了一个叫做内存描述符的mm_struct结构体来表示进程虚拟内存空间的全部信息。那么就到mm_struct结构体内部去寻找下相关的线索。

```c
struct mm_struct {
    unsigned long task_size;    /* size of task vm space */
    unsigned long start_code, end_code, start_data, end_data;
    unsigned long start_brk, brk, start_stack;
    unsigned long arg_start, arg_end, env_start, env_end;
    unsigned long mmap_base;  /* base of mmap area */
    unsigned long total_vm;    /* Total pages mapped */
    unsigned long locked_vm;  /* Pages that have PG_mlocked set */
    unsigned long pinned_vm;  /* Refcount permanently increased */
    unsigned long data_vm;    /* VM_WRITE & ~VM_SHARED & ~VM_STACK */
    unsigned long exec_vm;    /* VM_EXEC & ~VM_WRITE & ~VM_STACK */
    unsigned long stack_vm;    /* VM_STACK */

    //...... 省略 ........
}
```

内核中用mm_struct结构体中的上述属性来定义上图中虚拟内存空间里的不同内存区域。

start_code和end_code定义代码段的起始和结束位置，程序编译后的二进制文件中的机器码被加载进内存之后就存放在这里。

start_data和end_data定义数据段的起始和结束位置，二进制文件中存放的全局变量和静态变量被加载进内存中就存放在这里。

后面紧挨着的是BSS段，用于存放未被初始化的全局变量和静态变量，这些变量在加载进内存时会生成一段0填充的内存区域（BSS段），BSS段的大小是固定的，

下面就是OS堆了，在堆中内存地址的增长方向是由低地址向高地址增长，start_brk定义堆的起始位置，brk定义堆当前的结束位置。

> 使用 malloc 申请小块内存时（低于 128K），就是通过改变 brk 位置调整堆大小实现的。

接下来就是内存映射区，在内存映射区内存地址的增长方向是由高地址向低地址增长，mmap_base 定义内存映射区的起始地址。进程运行时所依赖的动态链接库中的代码段，数据段，BSS 段以及调用mmap映射出来的一段虚拟内存空间就保存在这个区域。

start_stack是栈的起始位置在RBP寄存器中存储，栈的结束位置也就是栈顶指针stackpointer在RSP寄存器中存储。在栈中内存地址的增长方向也是由高地址向低地址增长。

arg_start和arg_end是参数列表的位置，env_start和env_end是环境变量的位置。它们都位于栈中的最高地址处。

![image](assets\linux-28.png)

在mm_struct结构体中除了上述用于划分虚拟内存区域的变量之外，还定义了一些虚拟内存与物理内存映射内容相关的统计变量，操作系统会把物理内存划分成一页一页的区域来进行管理，所以物理内存到虚拟内存之间的映射也是按照页为单位进行的。这部分后续会详细介绍，这里只需要有个概念就行。mm_struct 结构体中的 total_vm 表示在进程虚拟内存空间中总共与物理内存映射的页的总数。

> 注意映射这个概念，它表示只是将虚拟内存与物理内存建立关联关系，并不代表真正的分配物理内存。

当内存吃紧的时候，有些页可以换出到硬盘上，而有些页因为比较重要，不能换出。locked_vm就是被锁定不能换出的内存页总数，pinned_vm表示既不能换出，也不能移动的内存页总数。data_vm表示数据段中映射的内存页数目，exec_vm是代码段中存放可执行文件的内存页数目，stack_vm是栈中所映射的内存页数目，这些变量均是表示进程虚拟内存空间中的虚拟内存使用情况。现在关于内核如何对进程虚拟内存空间进行布局的内容我们已经清楚了，那么布局之后划分出的这些虚拟内存区域在内核中又是如何被管理的呢？接着往下看

### 内核如何管理虚拟内存区域

在上面的介绍中，知道内核是通过一个mm_struct结构的内存描述符来表示进程的虚拟内存空间的，并通过task_size域来划分用户态虚拟内存空间和内核态虚拟内存空间。而在划分出的这些虚拟内存空间里边又包含了许多特定的虚拟内存区域，比如：代码段，数据段，堆，内存映射区，栈。那么这些虚拟内存区域在内核中又是如何表示的呢？本小节中，将介绍一个新的结构体vm_area_struct，正是这个结构体描述了这些虚拟内存区域VMA（virtual memory area）。

```c
struct vm_area_struct {

    unsigned long vm_start;    /* Our start address within vm_mm. */
    unsigned long vm_end;    /* The first byte after our end address
             within vm_mm. */
    /*
   * Access permissions of this VMA.
   */
    pgprot_t vm_page_prot;
    unsigned long vm_flags;  

    struct anon_vma *anon_vma;  /* Serialized by page_table_lock */
    struct file * vm_file;    /* File we map to (can be NULL). */
    unsigned long vm_pgoff;    /* Offset (within vm_file) in PAGE_SIZE
             units */  
    void * vm_private_data;    /* was vm_pte (shared mem) */
    /* Function pointers to deal with this struct. */
    const struct vm_operations_struct *vm_ops;
}
```

每个vm_area_struct结构对应于虚拟内存空间中的唯一虚拟内存区域VMA，vm_start指向了这块虚拟内存区域的起始地址（最低地址），vm_start本身包含在这块虚拟内存区域内。vm_end指向了这块虚拟内存区域的结束地址（最高地址），而vm_end本身包含在这块虚拟内存区域之外，所以vm_area_struct结构描述的是[vm_start，vm_end)这样一段左闭右开的虚拟内存区域。

![image](assets\linux-29.png)

### 定义虚拟内存区域的访问权限和行为规范

vm_page_prot和vm_flags都是用来标记vm_area_struct结构表示的这块虚拟内存区域的访问权限和行为规范。
上边小节中也提到，内核会将整块物理内存划分为一页一页大小的区域，以页为单位来管理这些物理内存，每页大小默认4K。而虚拟内存最终也是要和物理内存一一映射起来的，所以在虚拟内存空间中也有虚拟页的概念与之对应，虚拟内存中的虚拟页映射到物理内存中的物理页。无论是在虚拟内存空间中还是在物理内存中，内核管理内存的最小单位都是页。
vm_page_prot偏向于定义底层内存管理架构中页这一级别的访问控制权限，它可以直接应用在底层页表中，它是一个具体的概念。
虚拟内存区域 VMA 由许多的虚拟页 (page) 组成，每个虚拟页需要经过页表的转换才能找到对应的物理页面。页表中关于内存页的访问权限就是由vm_page_prot决定的。
vm_flags则偏向于定于整个虚拟内存区域的访问权限以及行为规范。描述的是虚拟内存区域中的整体信息，而不是虚拟内存区域中具体的某个独立页面。它是一个抽象的概念。可以通过vma->vm_page_prot=vm_get_page_prot(vma->vm_flags)实现到具体页面访问权限vm_page_prot的转换。
列举一些常用到的vm_flags：VM_READ，VM_WRITE，VM_EXEC定义了虚拟内存区域是否可以被读取，写入，执行等权限。
比如代码段这块内存区域的权限是可读，可执行，但是不可写。数据段具有可读可写的权限但是不可执行。堆则具有可读可写，可执行的权限（Java中的字节码存储在堆中，所以需要可执行权限），栈一般是可读可写的权限，一般很少有可执行权限。而文件映射与匿名映射区存放了共享链接库，所以也需要可执行的权限。

![image](assets\linux-30.png)

VM_SHARD用于指定这块虚拟内存区域映射的物理内存是否可以在多进程之间共享，以便完成进程间通讯。
VM_IO的设置表示这块虚拟内存区域可以映射至设备IO空间中。通常在设备驱动程序执行mmap进行IO空间映射时才会被设置。
VM_RESERVED的设置表示在内存紧张的时候，这块虚拟内存区域非常重要，不能被换出到磁盘中。
VM_SEQ_READ的设置用来暗示内核，应用程序对这块虚拟内存区域的读取是会采用顺序读的方式进行，内核会根据实际情况决定预读后续的内存页数，以便加快下次顺序访问速度。
VM_RAND_READ的设置会暗示内核，应用程序会对这块虚拟内存区域进行随机读取，内核则会根据实际情况减少预读的内存页数甚至停止预读。

通过这一系列的介绍，可以看到vm_flags就是定义整个虚拟内存区域的访问权限以及行为规范，而内存区域中内存的最小单位为页（4K），虚拟内存区域中包含了很多这样的虚拟页，对于虚拟内存区域VMA设置的访问权限也会全部复制到区域中包含的内存页中。

### 关联内存映射中的映射关系

接下来的三个属性anon_vma，vm_file，vm_pgoff分别和虚拟内存映射相关，虚拟内存区域可以映射到物理内存上，也可以映射到文件中，映射到物理内存上称之为匿名映射，映射到文件中称之为文件映射。那么这个映射关系在内核中该如何表示呢？这就用到了vm_area_struct结构体中的上述三个属性。

当调用malloc申请内存时，如果申请的是小块内存（低于128K）则会使用do_brk()系统调用通过调整堆中的brk指针大小来增加或者回收堆内存。
如果申请的是比较大块的内存（超过128K）时，则会调用mmap在虚拟内存空间中的文件映射与匿名映射区创建出一块VMA内存区域（这里是匿名映射）。这块匿名映射区域就用struct anon_vma结构表示。
当调用mmap进行文件映射时，vm_file属性就用来关联被映射的文件。这样一来虚拟内存区域就与映射文件关联了起来。vm_pgoff则表示映射进虚拟内存中的文件内容，在文件中的偏移。当然在匿名映射中，vm_area_struct结构中的vm_file就为null，vm_pgoff也就没有了意义。
vm_private_data则用于存储VMA中的私有数据。具体的存储内容和内存映射的类型有关，暂不展开论述。

### 针对虚拟内存区域的相关操作

struct vm_area_struct结构中还有一个vm_ops用来指向针对虚拟内存区域VMA的相关操作的函数指针。

```c
struct vm_operations_struct {
    void (*open)(struct vm_area_struct * area);
    void (*close)(struct vm_area_struct * area);
    vm_fault_t (*fault)(struct vm_fault *vmf);
    vm_fault_t (*page_mkwrite)(struct vm_fault *vmf);

    //..... 省略 .......
}
```

- 当指定的虚拟内存区域被加入到进程虚拟内存空间中时，open 函数会被调用
- 当虚拟内存区域 VMA 从进程虚拟内存空间中被删除时，close 函数会被调用
- 当进程访问虚拟内存时，访问的页面不在物理内存中，可能是未分配物理内存也可能是被置换到磁盘中，这时就会产生缺页异常，fault 函数就会被调用。
- 当一个只读的页面将要变为可写时，page_mkwrite 函数会被调用。

struct vm_operations_struct结构中定义的都是对虚拟内存区域VMA的相关操作函数指针。

### 虚拟内存区域在内核中是如何被组织的

在上一小节中，介绍了内核中用来表示虚拟内存区域VMA的结构体struct vm_area_struct，并详细为剖析了struct vm_area_struct中的一些重要的关键属性。现在已经熟悉了这些虚拟内存区域，那么接下来就是分析在内核中这些虚拟内存区域是如何被组织的。

继续来到struct vm_area_struct结构中，来看一下与组织结构相关的一些属性：

```c
struct vm_area_struct {

    struct vm_area_struct *vm_next, *vm_prev;
    struct rb_node vm_rb;
    struct list_head anon_vma_chain; 
    struct mm_struct *vm_mm;  /* The address space we belong to. */

    unsigned long vm_start;     /* Our start address within vm_mm. */
    unsigned long vm_end;       /* The first byte after our end address
                       within vm_mm. */
    /*
     * Access permissions of this VMA.
     */
    pgprot_t vm_page_prot;
    unsigned long vm_flags; 

    struct anon_vma *anon_vma;  /* Serialized by page_table_lock */
    struct file * vm_file;      /* File we map to (can be NULL). */
    unsigned long vm_pgoff;     /* Offset (within vm_file) in PAGE_SIZE
                       units */ 
    void * vm_private_data;     /* was vm_pte (shared mem) */
    /* Function pointers to deal with this struct. */
    const struct vm_operations_struct *vm_ops;
}
```

在内核中其实是通过一个struct vm_area_struct结构的双向链表将虚拟内存空间中的这些虚拟内存区域VMA串联起来的。
vm_area_struct结构中的vm_next，vm_prev指针分别指向VMA节点所在双向链表中的后继节点和前驱节点，内核中的这个VMA双向链表是有顺序的，所有VMA节点按照低地址到高地址的增长方向排序。
双向链表中的最后一个VMA节点的vm_next指针指向NULL，双向链表的头指针存储在内存描述符struct mm_struct结构中的mmap中，正是这个mmap串联起了整个虚拟内存空间中的虚拟内存区域。

```c
struct mm_struct {
    struct vm_area_struct *mmap;    /* list of VMAs */
}
```

在每个虚拟内存区域VMA中又通过struct vm_area_struct中的vm_mm指针指向了所属的虚拟内存空间 mm_struct。

![image](assets\linux-31.png)

可以通过cat /proc/pid/maps或者pmappid查看进程的虚拟内存空间布局以及其中包含的所有内存区域。这两个命令背后的实现原理就是通过遍历内核中的这个vm_area_struct双向链表获取的。内核中关于这些虚拟内存区域的操作除了遍历之外还有许多需要根据特定虚拟内存地址在虚拟内存空间中查找特定的虚拟内存区域。尤其在进程虚拟内存空间中包含的内存区域VMA比较多的情况下，使用红黑树查找特定虚拟内存区域的时间复杂度是O(logN)，可以显著减少查找所需的时间。所以在内核中，同样的内存区域vm_area_struct会有两种组织形式，一种是双向链表用于高效的遍历，另一种就是红黑树用于高效的查找。每个VMA区域都是红黑树中的一个节点，通过structvm_area_struct结构中的vm_rb将自己连接到红黑树中。而红黑树中的根节点存储在内存描述符struct mm_struct中的mm_rb中：

```c
struct mm_struct {
     struct rb_root mm_rb;
}
```

![image](assets\linux-32.png)

## 程序编译后的二进制文件如何映射到虚拟内存空间中

进程的虚拟内存空间mm_struct以及这些虚拟内存区域vm_area_struct的初始化创建过程：首先程序代码编译之后会生成一个ELF格式的二进制文件，这个二进制文件中包含了程序运行时所需要的元信息，比如程序的机器码，程序中的全局变量以及静态变量等。这个ELF格式的二进制文件中的布局和前边讲的虚拟内存空间中的布局类似，也是一段一段的，每一段包含了不同的元数据。磁盘文件中的段叫做Section，内存中的段叫做Segment，也就是内存区域。磁盘文件中的这些Section会在进程运行之前加载到内存中并映射到内存中的Segment。通常是多个Section映射到一个Segment。比如磁盘文件中的.text，.rodata等一些只读的Section，会被映射到内存的一个只读可执行的Segment里（代码段）。而.data，.bss等一些可读写的Section，则会被映射到内存的一个具有读写权限的Segment里（数据段，BSS段）。那么这些ELF格式的二进制文件中的Section是如何加载并映射进虚拟内存空间的呢？内核中完成这个映射过程的函数是load_elf_binary，这个函数的作用很大，加载内核的是它，启动第一个用户态进程init的是它，fork完了以后，调用exec运行一个二进制程序的也是它。当exec运行一个二进制程序的时候，除了解析ELF的格式之外，另外一个重要的事情就是建立上述提到的内存映射。

```c

static int load_elf_binary(struct linux_binprm *bprm)
{
    //...... 省略 ........
    // 设置虚拟内存空间中的内存映射区域起始地址 mmap_base
    setup_new_exec(bprm);

    //...... 省略 ........
    // 创建并初始化栈对应的 vm_area_struct 结构。
    // 设置 mm->start_stack 就是栈的起始地址也就是栈底，并将 mm->arg_start 是指向栈底的。
    retval = setup_arg_pages(bprm, randomize_stack_top(STACK_TOP),
                             executable_stack);

    //...... 省略 ........
    // 将二进制文件中的代码部分映射到虚拟内存空间中
    error = elf_map(bprm->file, load_bias + vaddr, elf_ppnt,
                    elf_prot, elf_flags, total_size);

    //...... 省略 ........
    // 创建并初始化堆对应的的 vm_area_struct 结构
    // 设置 current->mm->start_brk = current->mm->brk，设置堆的起始地址 start_brk，结束地址 brk。 起初两者相等表示堆是空的
    retval = set_brk(elf_bss, elf_brk, bss_prot);

    //...... 省略 ........
    // 将进程依赖的动态链接库 .so 文件映射到虚拟内存空间中的内存映射区域
    elf_entry = load_elf_interp(&loc->interp_elf_ex,
                                interpreter,
                                &interp_map_addr,
                                load_bias, interp_elf_phdata);

    //...... 省略 ........
    // 初始化内存描述符 mm_struct
    current->mm->end_code = end_code;
    current->mm->start_code = start_code;
    current->mm->start_data = start_data;
    current->mm->end_data = end_data;
    current->mm->start_stack = bprm->p;

    //...... 省略 ........
}

```

- setup_new_exec 设置虚拟内存空间中的内存映射区域起始地址 mmap_base
- setup_arg_pages 创建并初始化栈对应的 vm_area_struct 结构。置 mm->start_stack 就是栈的起始地址也就是栈底，并将 mm->arg_start 是指向栈底的。
- elf_map 将 ELF 格式的二进制文件中.text ，.data，.bss 部分映射到虚拟内存空间中的代码段，数据段，BSS 段中。
- set_brk 创建并初始化堆对应的的 vm_area_struct 结构，设置 current->mm->start_brk = current->mm->brk，设置堆的起始地址 start_brk，结束地址 brk。 起初两者相等表示堆是空的。
- load_elf_interp 将进程依赖的动态链接库 .so 文件映射到虚拟内存空间中的内存映射区域
- 初始化内存描述符 mm_struct

## 内核虚拟内存空间

不同进程之间的虚拟内存空间是相互隔离的，彼此之间相互独立，相互感知不到其他进程的存在。使得进程以为自己拥有所有的内存资源。而内核态虚拟内存空间是所有进程共享的，不同进程进入内核态之后看到的虚拟内存空间全部是一样的。

>用户态是指进程在用户代码中运行。
>
>内核态是指进程进入内核代码，执行内核的代码。

![image](assets\linux-19.png)

比如上图中的进程a，进程b，进程c分别在各自的用户态虚拟内存空间中访问虚拟地址x。由于进程之间的用户态虚拟内存空间是相互隔离相互独立的，虽然在进程a，进程b，进程c访问的都是虚拟地址x但是看到的内容却是不一样的（背后可能映射到不同的物理内存中）。但是当进程a，进程b，进程c进入到内核态之后情况就不一样了，由于内核虚拟内存空间是各个进程共享的，所以它们在内核空间中看到的内容全部是一样的，比如进程a，进程b，进程c在内核态都去访问虚拟地址y。这时它们看到的内容就是一样的了。

> 这里澄清一个经常被误解的概念：由于内核会涉及到物理内存的管理，所以很多人会想当然地认为只要进入了内核态就开始使用物理地址了，这就大错特错了，千万不要这样理解，进程进入内核态之后使用的仍然是虚拟内存地址，只不过在内核中使用的虚拟内存地址被限制在了内核态虚拟内存空间范围中

在清楚了这个基本概念之后，下面分别从32位体系和64位体系下为大家介绍内核态虚拟内存空间的布局。

### 32位体系内核虚拟内存空间布局

在前边《内核如何划分用户态和内核态虚拟内存空间》中提到，内核在/arch/x86/include/asm/page_32_types.h文件中通过TASK_SIZE将进程虚拟内存空间和内核虚拟内存空间分割开来。__PAGE_OFFSET的值在32位系统下为0xC000000，在32位体系结构下进程用户态虚拟内存空间为3GB，虚拟内存地址范围为：0x00000000-0xC000000。内核态虚拟内存空间为1GB，虚拟内存地址范围为：0xC000000-0xFFFFFFFF。本小节主要关注0xC000000-0xFFFFFFFF这段虚拟内存地址区域也就是内核虚拟内存空间的布局情况。

#### 直接映射区

在总共大小1G的内核虚拟内存空间中，位于最前边有一块896M大小的区域，被称之为直接映射区或者线性映射区，地址范围为3G--3G+896m。之所以这块896M大小的区域称为直接映射区或者线性映射区，是因为这块连续的虚拟内存地址会映射到0-896M这块连续的物理内存上。也就是说3G--3G+896m这块896M大小的虚拟内存会直接映射到0-896M这块896M大小的物理内存上，这块区域中的虚拟内存地址直接减去0xC0000000(3G)就得到了物理内存地址。所以称这块区域为直接映射区。为了理解，假设现在机器上的物理内存为4G大小：

![image](assets\linux-33.png)

> 虽然这块区域中的虚拟地址是直接映射到物理地址上，但是内核在访问这段区域的时候还是走的虚拟内存地址，内核也会为这块空间建立映射页表。关于页表的概念笔者后续再讲解，这里只需要简单理解为页表保存了虚拟地址到物理地址的映射关系即可。

这里只需要记得内核态虚拟内存空间的前896M区域是直接映射到物理内存中的前896M区域中的，直接映射区中的映射关系是一比一映射。映射关系是固定的不会改变。接下来就看一下这块直接映射区域在物理内存中究竟存的是什么内容：

1. 在这段896M大小的物理内存中，前1M已经在系统启动的时候被系统占用，1M之后的物理内存存放的是内核代码段，数据段，BSS段（这些信息起初存放在ELF格式的二进制文件中，在系统启动的时候被加载进内存）。可以通过cat/proc/iomem命令查看具体物理内存布局情况。
2. 当使用fork系统调用创建进程的时候，内核会创建一系列进程相关的描述符，比如之前提到的进程的核心数据结构task_struct，进程的内存空间描述符mm_struct，以及虚拟内存区域描述符vm_area_struct等。这些进程相关的数据结构也会存放在物理内存前896M的这段区域中，当然也会被直接映射至内核态虚拟内存空间中的3G--3G+896m这段直接映射区域中。
3. 当进程被创建完毕之后，在内核运行的过程中，会涉及内核栈的分配，内核会为每个进程分配一个固定大小的内核栈（一般是两个页大小，依赖具体的体系结构），每个进程的整个调用链必须放在自己的内核栈中，内核栈也是分配在直接映射区。
4. 与进程用户空间中的栈不同的是，内核栈容量小而且是固定的，用户空间中的栈容量大而且可以动态扩展。内核栈的溢出危害非常巨大，它会直接悄无声息的覆盖相邻内存区域中的数据，破坏数据。
5. 通过以上内容的介绍了解到内核虚拟内存空间最前边的这段896M大小的直接映射区如何与物理内存进行映射关联，并且清楚了直接映射区主要用来存放哪些内容。

接下来再次从功能划分的角度介绍下这块直接映射区域。

内核对物理内存的管理都是以页为最小单位来管理的，每页默认4K大小，理想状况下任何种类的数据页都可以存放在任何页框中，没有什么限制。比如：存放内核数据，用户数据，缓冲磁盘数据等。但是实际的计算机体系结构受到硬件方面的限制制约，间接导致限制了页框的使用方式。比如在X86体系结构下，ISA总线的DMA（直接内存存取）控制器，只能对内存的前16M进行寻址，这就导致了ISA设备不能在整个32位地址空间中执行DMA，只能使用物理内存的前16M进行DMA操作。因此直接映射区的前16M专门让内核用来为DMA分配内存，这块16M大小的内存区域称之为ZONE_DMA。用于DMA的内存必须从ZONE_DMA区域中分配。而直接映射区中剩下的部分也就是从16M到896M（不包含896M）这段区域，称之为ZONE_NORMAL。从字面意义上可以了解到，这块区域包含的就是正常的页框（使用没有任何限制）。ZONE_NORMAL由于也是属于直接映射区的一部分，对应的物理内存16M到896M这段区域也是被直接映射至内核态虚拟内存空间中的3G+16M到3G+896M这段虚拟内存上。

![image](assets\linux-34.png)

> 注意这里的 ZONE_DMA 和 ZONE_NORMAL 是内核针对物理内存区域的划分。

现在物理内存中的前896M的区域也就是前边介绍的ZONE_DMA和ZONE_NORMAL区域到内核虚拟内存空间的映射就介绍完了，它们都是采用直接映射的方式，一比一进行映射。

#### ZONE_HIGHMEM 高端内存

而物理内存896M以上的区域被内核划分为ZONE_HIGHMEM区域，被称之为高端内存。本例中的物理内存假设为4G，高端内存区域为4G-896M=3200M，那么这块3200M大小的ZONE_HIGHMEM区域该如何映射到内核虚拟内存空间中呢？由于内核虚拟内存空间中的前896M虚拟内存已经被直接映射区所占用，而在32体系结构下内核虚拟内存空间总共也就1G的大小，这样一来内核剩余可用的虚拟内存空间就变为了1G-896M=128M。显然物理内存中3200M大小的ZONE_HIGHMEM区域无法继续通过直接映射的方式映射到这128M大小的虚拟内存空间中。这样一来物理内存中的ZONE_HIGHMEM区域就只能采用动态映射的方式映射到128M大小的内核虚拟内存空间中，也就是说只能动态的一部分一部分的分批映射，先映射正在使用的这部分，使用完毕解除映射，接着映射其他部分。知道了ZONE_HIGHMEM区域的映射原理，接着往下看这128M大小的内核虚拟内存空间究竟是如何布局的？

![image](assets\linux-35.png)

内核虚拟内存空间中的3G+896M这块地址在内核中定义为high_memory，high_memory往上有一段8M大小的内存空洞。空洞范围为：high_memory到VMALLOC_START。VMALLOC_START定义在内核源码/arch/x86/include/asm/pgtable_32_areas.h文件中：

```c
#define VMALLOC_OFFSET  (8 * 1024 * 1024)

#define VMALLOC_START  ((unsigned long)high_memory + VMALLOC_OFFSET)

```

#### vmalloc 动态映射区

接下来 VMALLOC_START 到 VMALLOC_END 之间的这块区域成为动态映射区。采用动态映射的方式映射物理内存中的高端内存。

```c
#ifdef CONFIG_HIGHMEM
# define VMALLOC_END  (PKMAP_BASE - 2 * PAGE_SIZE)
#else
# define VMALLOC_END  (LDT_BASE_ADDR - 2 * PAGE_SIZE)
#endif

```

![image](assets\linux-36.png)

和用户态进程使用malloc申请内存一样，在这块动态映射区内核是使用vmalloc进行内存分配。由于之前介绍的动态映射的原因，vmalloc分配的内存在虚拟内存上是连续的，但是物理内存是不连续的。通过页表来建立物理内存与虚拟内存之间的映射关系，从而可以将不连续的物理内存映射到连续的虚拟内存上。

> 由于 vmalloc 获得的物理内存页是不连续的，因此它只能将这些物理内存页一个一个地进行映射，在性能开销上会比直接映射大得多。
>
> 关于vmalloc分配内存的相关实现原理，后面再讲解，这里只需要明白它在哪块虚拟内存区域中活动即可。

#### 永久映射区

![image](assets\linux-37.png)

而在PKMAP_BASE到FIXADDR_START之间的这段空间称为永久映射区。在内核的这段虚拟地址空间中允许建立与物理高端内存的长期映射关系。比如内核通过alloc_pages()函数在物理内存的高端内存中申请获取到的物理内存页，这些物理内存页可以通过调用kmap映射到永久映射区中。

> LAST_PKMAP 表示永久映射区可以映射的页数限制。

```c
#define PKMAP_BASE    \
  ((LDT_BASE_ADDR - PAGE_SIZE) & PMD_MASK)

#define LAST_PKMAP 1024

```

#### 固定映射区

![image](assets\linux-38.png)

内核虚拟内存空间中的下一个区域为固定映射区，区域范围为：FIXADDR_START到FIXADDR_TOP。FIXADDR_START和FIXADDR_TOP定义在内核源码/arch/x86/include/asm/fixmap.h文件中：

```c
#define FIXADDR_START    (FIXADDR_TOP - FIXADDR_SIZE)

extern unsigned long __FIXADDR_TOP; // 0xFFFF F000
#define FIXADDR_TOP  ((unsigned long)__FIXADDR_TOP)

```

在内核虚拟内存空间的直接映射区中，直接映射区中的虚拟内存地址与物理内存前896M的空间的映射关系都是预设好的，一比一映射。在固定映射区中的虚拟内存地址可以自由映射到物理内存的高端地址上，但是与动态映射区以及永久映射区不同的是，在固定映射区中虚拟地址是固定的，而被映射的物理地址是可以改变的。也就是说，有些虚拟地址在编译的时候就固定下来了，是在内核启动过程中被确定的，而这些虚拟地址对应的物理地址不是固定的。采用固定虚拟地址的好处是它相当于一个指针常量（常量的值在编译时确定），指向物理地址，如果虚拟地址不固定，则相当于一个指针变量。那为什么会有固定映射这个概念呢?比如：在内核的启动过程中，有些模块需要使用虚拟内存并映射到指定的物理地址上，而且这些模块也没有办法等待完整的内存管理模块初始化之后再进行地址映射。因此，内核固定分配了一些虚拟地址，这些地址有固定的用途，使用该地址的模块在初始化的时候，将这些固定分配的虚拟地址映射到指定的物理地址上去。

#### 临时映射区

![image](assets\linux-39.png)

比如在Buffered IO模式下进行文件写入的时候，在下图中的第四步，内核会调用iov_iter_copy_from_user_atomic函数将用户空间缓冲区DirectByteBuffer中的待写入数据拷贝到pagecache中。

![image](assets\linux-40.png)

但是内核又不能直接进行拷贝，因为此时从page cache中取出的缓存页page是物理地址，而在内核中是不能够直接操作物理地址的，只能操作虚拟地址。那怎么办呢？所以就需要使用kmap_atomic将缓存页临时映射到内核空间的一段虚拟地址上，这段虚拟地址就位于内核虚拟内存空间中的临时映射区上，然后将用户空间缓存区DirectByteBuffer中的待写入数据通过这段映射的虚拟地址拷贝到page cache中的相应缓存页中。这时文件的写入操作就已经完成了。由于是临时映射，所以在拷贝完成之后，调用kunmap_atomic将这段映射再解除掉。

```c
size_t iov_iter_copy_from_user_atomic(struct page *page,
                                      struct iov_iter *i, unsigned long offset, size_t bytes)
{
    // 将缓存页临时映射到内核虚拟地址空间的临时映射区中
    char *kaddr = kmap_atomic(page), 
    *p = kaddr + offset;
    // 将用户缓存区 DirectByteBuffer 中的待写入数据拷贝到文件缓存页中
    iterate_all_kinds(i, bytes, v,
                      copyin((p += v.iov_len) - v.iov_len, v.iov_base, v.iov_len),
                      memcpy_from_page((p += v.bv_len) - v.bv_len, v.bv_page,
                                       v.bv_offset, v.bv_len),
                      memcpy((p += v.iov_len) - v.iov_len, v.iov_base, v.iov_len)
                     )
        // 解除内核虚拟地址空间与缓存页之间的临时映射，这里映射只是为了临时拷贝数据用
        kunmap_atomic(kaddr);
    return bytes;
}
```

#### 32位体系结构下Linux虚拟内存空间整体布局

![image](assets\linux-41.png)

### 64位体系内核虚拟内存空间布局

内核虚拟内存空间在32位体系下只有1G大小，实在太小了，因此需要精细化的管理，于是按照功能分类划分除了很多内核虚拟内存区域，这样就显得非常复杂。到了64位体系下，内核虚拟内存空间的布局和管理就变得容易多了，因为进程虚拟内存空间和内核虚拟内存空间各自占用128T的虚拟内存，实在是太大了，可以在这里边随意挥霍。因此在64位体系下的内核虚拟内存空间与物理内存的映射就变得非常简单，由于虚拟内存空间足够的大，即便是内核要访问全部的物理内存，直接映射就可以了，不在需要用到之前介绍的高端内存那种动态映射方式。

内核在/arch/x86/include/asm/page_64_types.h文件中通过TASK_SIZE将进程虚拟内存空间和内核虚拟内存空间分割开来。

```c
#define TASK_SIZE    (test_thread_flag(TIF_ADDR32) ? \
          IA32_PAGE_OFFSET : TASK_SIZE_MAX)

#define TASK_SIZE_MAX    task_size_max()

#define task_size_max()    ((_AC(1,UL) << __VIRTUAL_MASK_SHIFT) - PAGE_SIZE)

#define __VIRTUAL_MASK_SHIFT  47

```

> 64 位系统中的 TASK_SIZE 为 0x00007FFFFFFFF000

![image](assets\linux-42.png)

在64位系统中，只使用了其中的低48位来表示虚拟内存地址。其中用户态虚拟内存空间为低128T，虚拟内存地址范围为：0x0000 0000 0000 0000 - 0x0000 7FFF FFFF F000。内核态虚拟内存空间为高128T，虚拟内存地址范围为：0xFFFF 8000 0000 0000 - 0xFFFF FFFF FFFF FFFF。本节主要关注0xFFFF 8000 0000 0000 - 0xFFFF FFFF FFFF FFFF这段内核虚拟内存空间的布局情况。

![image](assets\linux-43.png)

64位内核虚拟内存空间从0xFFFF 8000 0000 0000开始到0xFFFF 8800 0000 0000这段地址空间是一个8T大小的内存空洞区域。
紧着着8T大小的内存空洞下一个区域就是64T大小的直接映射区。这个区域中的虚拟内存地址减去PAGE_OFFSET就直接得到了物理内存地址。
PAGE_OFFSET变量定义在/arch/x86/include/asm/page_64_types.h文件中：

```c
#define __PAGE_OFFSET_BASE      _AC(0xffff880000000000, UL)
#define __PAGE_OFFSET           __PAGE_OFFSET_BASE

```

从图中VMALLOC_START到VMALLOC_END的这段区域是32T大小的vmalloc映射区，这里类似用户空间中的堆，内核在这里使用vmalloc系统调用申请内存。

VMALLOC_START和VMALLOC_END变量定义在/arch/x86/include/asm/pgtable_64_types.h文件中：

```c
#define __VMALLOC_BASE_L4  0xffffc90000000000UL

#define VMEMMAP_START    __VMEMMAP_BASE_L4

#define VMALLOC_END    (VMALLOC_START + (VMALLOC_SIZE_TB << 40) - 1)

```

从VMEMMAP_START开始是1T大小的虚拟内存映射区，用于存放物理页面的描述符structpage结构用来表示物理内存页。

VMEMMAP_START变量定义在/arch/x86/include/asm/pgtable_64_types.h文件中：

```c
#define __VMEMMAP_BASE_L4  0xffffea0000000000UL

# define VMEMMAP_START    __VMEMMAP_BASE_L4

```

从 `__START_KERNEL_map开始是大小为512M的区域用于存放内核代码段、全局变量、BSS等。这里对应到物理内存开始的位置，减去__START_KERNEL_map`就能得到物理内存的地址。这里和直接映射区有点像，但是不矛盾，因为直接映射区之前有8T的空洞区域，早就过了内核代码在物理内存中加载的位置。

__START_KERNEL_map变量定义在/arch/x86/include/asm/page_64_types.h文件中：

```c
#define __START_KERNEL_map  _AC(0xffffffff80000000, UL)

```

#### 64位体系结构下Linux虚拟内存空间整体布局

![image](assets\linux-44.png)

## 到底什么是物理内存地址

接着分析物理内存，平时所称的内存也叫随机访问存储器（random-access memory）也叫RAM。而RAM分为两类：
一类是静态RAM（SRAM），这类SRAM用于CPU高速缓存L1 Cache，L2 Cache，L3 Cache。其特点是访问速度快，访问速度为1-30个时钟周期，但是容量小，造价高。

![image](assets\linux-45.png)

另一类则是动态RAM(DRAM)，这类DRAM用于我们常说的主存上，其特点的是访问速度慢（相对高速缓存），访问速度为50-200个时钟周期，但是容量大，造价便宜些（相对高速缓存）。

内存由一个一个的存储器模块（memory module）组成，它们插在主板的扩展槽上。常见的存储器模块通常以 64 位为单位（ 8 个字节）传输数据到存储控制器上或者从存储控制器传出数据。

![image](assets\linux-46.png)

如图所示内存条上黑色的元器件就是存储器模块（memory module）。多个存储器模块连接到存储控制器上，就聚合成了主存。

![image](assets\linux-47.png)

而 DRAM 芯片就包装在存储器模块中，每个存储器模块中包含 8 个 DRAM 芯片，依次编号为 0 - 7 。

![image](assets\linux-48.png)

而每一个 DRAM 芯片的存储结构是一个二维矩阵，二维矩阵中存储的元素我们称为超单元（supercell），每个 supercell 大小为一个字节（8 bit）。每个 supercell 都由一个坐标地址（i，j）。

> i 表示二维矩阵中的行地址，在计算机中行地址称为 RAS (row access strobe，行访问选通脉冲)。j 表示二维矩阵中的列地址，在计算机中列地址称为 CAS (column access strobe,列访问选通脉冲)。

下图中的 supercell 的 RAS = 2，CAS = 2。

![image](assets\linux-49.png)

DRAM 芯片中的信息通过引脚流入流出 DRAM 芯片。每个引脚携带 1 bit 的信号。

图中 DRAM 芯片包含了两个地址引脚( `addr` )，因为我们要通过 RAS，CAS 来定位要获取的 supercell 。还有 8 个数据引脚（`data`），因为 DRAM 芯片的 IO 单位为一个字节（8 bit），所以需要 8 个 data 引脚从 DRAM 芯片传入传出数据。

> 注意这里只是为了解释地址引脚和数据引脚的概念，实际硬件中的引脚数量是不一定的。

### DRAM 芯片的访问

现在就以读取上图中坐标地址为（2，2）的 supercell 为例，来说明访问 DRAM 芯片的过程。

![image](assets\linux-50.png)

1. 首先存储控制器将行地址 RAS = 2 通过地址引脚发送给 DRAM 芯片。
2. DRAM 芯片根据 RAS = 2 将二维矩阵中的第二行的全部内容拷贝到内部行缓冲区中。
3. 接下来存储控制器会通过地址引脚发送 CAS = 2 到 DRAM 芯片中。
4. DRAM 芯片从内部行缓冲区中根据 CAS = 2 拷贝出第二列的 supercell 并通过数据引脚发送给存储控制器。

> DRAM 芯片的 IO 单位为一个 supercell ，也就是一个字节(8 bit)。

### CPU 如何读写主存

前边介绍了内存的物理结构，以及如何访问内存中的 DRAM 芯片获取 supercell 中存储的数据（一个字节）。本小节来介绍下 CPU 是如何访问内存的：

![image](assets\linux-51.png)

CPU 与内存之间的数据交互是通过总线（bus）完成的，而数据在总线上的传送是通过一系列的步骤完成的，这些步骤称为总线事务（bus transaction）。

其中数据从内存传送到 CPU 称之为读事务（read transaction），数据从 CPU 传送到内存称之为写事务（write transaction）。

总线上传输的信号包括：地址信号，数据信号，控制信号。其中控制总线上传输的控制信号可以同步事务，并能够标识出当前正在被执行的事务信息：

- 当前这个事务是到内存的？还是到磁盘的？或者是到其他 IO 设备的？
- 这个事务是读还是写？
- 总线上传输的地址信号（物理内存地址），还是数据信号（数据）？。

> **这里需要注意总线上传输的地址均为物理内存地址**。比如：在 MESI 缓存一致性协议中当 CPU core0 修改字段 a 的值时，其他 CPU 核心会在总线上嗅探字段 a 的**物理内存地址**，如果嗅探到总线上出现字段 a 的**物理内存地址**，说明有人在修改字段 a，这样其他 CPU 核心就会失效字段 a 所在的 cache line 。

如上图所示，其中系统总线是连接 CPU 与 IO bridge 的，存储总线是来连接 IO bridge 和主存的。

IO bridge 负责将系统总线上的电子信号转换成存储总线上的电子信号。IO bridge 也会将系统总线和存储总线连接到 IO 总线（磁盘等 IO 设备）上。这里我们看到 IO bridge 其实起的作用就是转换不同总线上的电子信号。

### CPU 从内存读取数据过程

假设 CPU 现在需要将物理内存地址为 A 的内容加载到寄存器中进行运算。

> 需要注意的是 CPU 只会访问虚拟内存，在操作总线之前，需要把虚拟内存地址转换为物理内存地址，总线上传输的都是物理内存地址，这里省略了虚拟内存地址到物理内存地址的转换过程，这部分内容笔者会在后续文章的相关章节详细为大家讲解，这里我们聚焦如果通过物理内存地址读取内存数据。

![image](assets\linux-52.png)

首先 CPU 芯片中的总线接口会在总线上发起读事务（read transaction）。 该读事务分为以下步骤进行：

1. CPU 将物理内存地址 A 放到系统总线上。随后 IO bridge 将信号传递到存储总线上。
2. 主存感受到存储总线上的地址信号并通过存储控制器将存储总线上的物理内存地址 A 读取出来。
3. 存储控制器通过物理内存地址 A 定位到具体的存储器模块，从 DRAM 芯片中取出物理内存地址 A 对应的数据 X。
4. 存储控制器将读取到的数据 X 放到存储总线上，随后 IO bridge 将存储总线上的数据信号转换为系统总线上的数据信号，然后继续沿着系统总线传递。
5. CPU 芯片感受到系统总线上的数据信号，将数据从系统总线上读取出来并拷贝到寄存器中。

以上就是 CPU 读取内存数据到寄存器中的完整过程。

但是其中还涉及到一个重要的过程，这里我们还是需要摊开来介绍一下，那就是存储控制器如何通过物理内存地址 A 从主存中读取出对应的数据 X 的？

接下来结合前边介绍的内存结构以及从 DRAM 芯片读取数据的过程，来总体介绍下如何从主存中读取数据。

### 如何根据物理内存地址从主存中读取数据

前边介绍到，当主存中的存储控制器感受到了存储总线上的地址信号时，会将内存地址从存储总线上读取出来。随后会通过内存地址定位到具体的存储器模块。而每个存储器模块中包含了 8 个 DRAM 芯片，编号从 0 - 7 。

![image](assets\linux-53.png)

存储控制器会将物理内存地址转换为 DRAM 芯片中 supercell 在二维矩阵中的坐标地址(RAS，CAS)。并将这个坐标地址发送给对应的存储器模块。随后存储器模块会将 RAS 和 CAS 广播到存储器模块中的所有 DRAM 芯片。依次通过 (RAS，CAS) 从 DRAM0 到 DRAM7 读取到相应的 supercell 。

![image](assets\linux-54.png)

一个 supercell 存储了一个字节（ 8 bit ） 数据，这里从 DRAM0 到 DRAM7 依次读取到了 8 个 supercell 也就是 8 个字节，然后将这 8 个字节返回给存储控制器，由存储控制器将数据放到存储总线上。

CPU 总是以 word size 为单位从内存中读取数据，在 64 位处理器中的 word size 为 8 个字节。64 位的内存每次只能吞吐 8 个字节。

> CPU 每次会向内存读写一个 cache line 大小的数据（ 64 个字节），但是内存一次只能吞吐 8 个字节。

所以在物理内存地址对应的存储器模块中，DRAM0 芯片存储第一个低位字节（ supercell ），DRAM1 芯片存储第二个字节，......依次类推 DRAM7 芯片存储最后一个高位字节。

![image](assets\linux-55.png)

由于存储器模块中这种由 8 个 DRAM 芯片组成的物理存储结构的限制，内存读取数据只能是按照物理内存地址，8 个字节 8 个字节地顺序读取数据。所以说内存一次读取和写入的单位是 8 个字节。

![image](assets\linux-56.png)

而且在程序员眼里连续的物理内存地址实际上在物理上是不连续的。因为这连续的 8 个字节其实是存储于不同的 DRAM 芯片上的。每个 DRAM 芯片存储一个字节（supercell）

### CPU 向内存写入数据过程

假设 CPU 要将寄存器中的数据 X 写到物理内存地址 A 中。同样的道理，CPU 芯片中的总线接口会向总线发起写事务（write transaction）。写事务步骤如下：

1. CPU 将要写入的物理内存地址 A 放入系统总线上。
2. 通过 IO bridge 的信号转换，将物理内存地址 A 传递到存储总线上。
3. 存储控制器感受到存储总线上的地址信号，将物理内存地址 A 从存储总线上读取出来，并等待数据的到达。
4. CPU 将寄存器中的数据拷贝到系统总线上，通过 IO bridge 的信号转换，将数据传递到存储总线上。
5. 存储控制器感受到存储总线上的数据信号，将数据从存储总线上读取出来。
6. 存储控制器通过内存地址 A 定位到具体的存储器模块，最后将数据写入存储器模块中的 8 个 DRAM 芯片中。

## malloc是如何分配内存的

实际上，malloc()并不是系统调用，而是C库里的函数，用于动态分配内存。malloc申请内存的时候，会有两种方式向操作系统申请堆内存。

1. 方式一：通过brk()**系统调用**从堆分配内存
2. 方式二：通过mmap()**系统调用**在文件映射区域分配内存；

所以，brk和mmap才是内核函数，malloc只是库函数

方式一实现的方式很简单，就是通过 brk() 函数将「堆顶」指针向高地址移动，获得新的内存空间。如下图：

![image](assets\linux-57.png)

方式二通过 mmap() 系统调用中「私有匿名映射」的方式，在文件映射区分配一块内存，也就是从文件映射区“偷”了一块内存。如下图：

![image](assets\linux-58.png)

malloc() 源码里默认定义了一个阈值：

- 如果用户分配的内存小于 128 KB，则通过 brk() 申请内存；
- 如果用户分配的内存大于 128 KB，则通过 mmap() 申请内存；

malloc()分配的是虚拟内存；如果分配后的虚拟内存没有被访问的话，是不会将虚拟内存不会映射到物理内存，这样就不会占用物理内存了。只有在访问已分配的虚拟地址空间的时候，操作系统通过查找页表，发现虚拟内存对应的页没有在物理内存中，就会触发缺页中断，然后操作系统会建立虚拟内存和物理内存之间的映射关系。

malloc()在分配内存的时候，并不是老老实实按用户预期申请的字节数来分配内存空间大小，而是会预分配更大的空间作为内存池。具体会预分配多大的空间，跟malloc使用的内存管理器有关系，就以malloc默认的内存管理器（Ptmalloc2）来分析。用下面这个代码，通过malloc申请1字节的内存时，看看操作系统实际分配了多大的内存空间。

```c
#include <stdio.h>
#include <malloc.h>

int main() {
        printf("使用cat /proc/%d/maps查看内存分配\n",getpid());

        //申请1字节的内存
        void *addr = malloc(1);
        printf("此1字节的内存起始地址：%x\n", addr);
        printf("使用cat /proc/%d/maps查看内存分配\n",getpid());

        //将程序阻塞，当输入任意字符时才往下执行
        getchar();

        //释放内存
        free(addr);
        printf("释放了1字节的内存，但heap堆并不会释放\n");

        getchar();
        return 0;
}
```

编译&执行代码：

```bash
[root@server01 opt]# gcc alloc_addr.c -o alloc_addr
[root@server01 opt]# ./alloc_addr 
使用cat /proc/1451/maps查看内存分配
此1字节的内存起始地址：1a12010
使用cat /proc/1451/maps查看内存分配
```

可以通过 /proc/PID/maps 文件查看进程的内存分布情况。我在 maps 文件通过此 1 字节的内存起始地址过滤出了内存地址的范围。

```bash
[root@server01 ~]# cat /proc/1451/maps | grep 1a120
01a12000-01a33000 rw-p 00000000 00:00 0                                  [heap]
```

这个例子分配的内存小于128KB，所以是通过brk()系统调用向堆空间申请的内存，因此可以看到最右边有[heap]的标识。可以看到，堆空间的内存地址范围是00d73000-00d94000，这个范围大小是132KB，也就说明了malloc(1)实际上预分配132K字节的内存。但是程序里打印的内存起始地址是d73010，而maps文件显示堆内存空间的起始地址是d73000，为什么会多出来0x10（16字节）呢？这个问题，后面会说。

在上面的进程往下执行，看看通过 free() 函数释放内存后，堆内存还在吗？

```bash
[root@server01 opt]# ./alloc_addr 
使用cat /proc/1451/maps查看内存分配
此1字节的内存起始地址：1a12010
使用cat /proc/1451/maps查看内存分配

释放了1字节的内存，但heap堆并不会释放
```

从下图可以看到，通过 free 释放内存后，堆内存还是存在的，并没有归还给操作系统。

```bash
[root@server01 ~]# cat /proc/1451/maps | grep 1a120
01a12000-01a33000 rw-p 00000000 00:00 0                                  [heap]

```

这是因为与其把这1字节释放给操作系统，不如先缓存着放进malloc的内存池里，当进程再次申请1字节的内存时就可以直接复用，这样速度快了很多。当然，当进程退出后，操作系统就会回收进程的所有资源。上面说的free内存后堆内存还存在，是针对malloc通过brk()方式申请的内存的情况。如果malloc通过mmap方式申请的内存，free释放内存后就会归归还给操作系统。

改一下代码，通过malloc申请128KB字节的内存，来使得malloc通过mmap方式来分配内存。

```c
#include <stdio.h>
#include <malloc.h>

int main() {
        //申请1字节的内存
        void *addr = malloc(128*1024);
        printf("此128KB字节的内存起始地址：%x\n", addr);
        printf("使用cat /proc/%d/maps查看内存分配\n",getpid());

        //将程序阻塞，当输入任意字符时才往下执行
        getchar();

        //释放内存
        free(addr);
        printf("释放了128KB字节的内存，内存也归还给了操作系统\n");

        getchar();
        return 0;
}
```

编译&执行代码：

```bash
[root@server01 opt]# gcc alloc_addr.c -o alloc_addr
[root@server01 opt]# ./alloc_addr 
此128KB字节的内存起始地址：38fff010
使用cat /proc/1636/maps查看内存分配
```

查看进程的内存的分布情况，可以发现最右边没有 [head] 标志，说明是通过 mmap 以匿名映射的方式从文件映射区分配的匿名内存。

```bash
[root@server01 ~]# cat /proc/1636/maps | grep 38fff
7fb738fff000-7fb739023000 rw-p 00000000 00:00 0 
```

然后释放掉这个内存看看：

```bash
[root@server01 opt]# ./alloc_addr 
此128KB字节的内存起始地址：38fff010
使用cat /proc/1636/maps查看内存分配

释放了128KB字节的内存，内存也归还给了操作系统
```

再次查看该 128 KB 内存的起始地址，可以发现已经不存在了，说明归还给了操作系统。

```bash
[root@server01 ~]# cat /proc/1636/maps | grep 38fff
[root@server01 ~]# 
```

总结：

1. malloc通过brk()方式申请的内存，free释放内存的时候，并不会把内存归还给操作系统，而是缓存在malloc的内存池中，待下次使用；
2. malloc通过mmap()方式申请的内存，free释放内存的时候，会把内存归还给操作系统，内存得到真正的释放。

malloc中存在sbrk和mmap两种内存申请方式的原因：

因为向操作系统申请内存，是要通过系统调用的，执行系统调用是要进入内核态的，然后在回到用户态，运行态的切换会耗费不少时间。所以，申请内存的操作应该避免频繁的系统调用，如果都用mmap来分配内存，等于每次都要执行系统调用。另外，因为mmap分配的内存每次释放的时候，都会归还给操作系统，于是每次mmap分配的虚拟地址都是缺页状态的，然后在第一次访问该虚拟地址的时候，就会触发缺页中断。也就是说，频繁通过mmap分配的内存话，不仅每次都会发生运行态的切换，还会发生缺页中断（在第一次访问虚拟地址后），这样会导致CPU消耗较大。为了改进这两个问题，malloc通过brk()系统调用在堆空间申请内存的时候，由于堆空间是连续的，所以直接预分配更大的内存来作为内存池，当内存释放的时候，就缓存在内存池中。等下次在申请内存的时候，就直接从内存池取出对应的内存块就行了，而且可能这个内存块的虚拟地址与物理地址的映射关系还存在，这样不仅减少了系统调用的次数，也减少了缺页中断的次数，这将大大降低CPU的消耗。也就是说使用brk方式的话，通过一次系统调用申请到的内存可能会被多次使用。

为什么不全部使用brk来分配呢？前面提到通过brk从堆空间分配的内存，并不会归还给操作系统，那么那考虑这样一个场景。如果连续申请了10k，20k，30k这三片内存，如果10k和20k这两片释放了，变为了空闲内存空间，如果下次申请的内存小于30k，那么就可以重用这个空闲内存空间。但是如果下次申请的内存大于30k，没有可用的空闲内存空间，必须向OS申请，实际使用内存继续增大。因此，随着系统频繁地malloc和free，尤其对于小块内存，堆内将产生越来越多不可用的碎片，导致“内存泄露”。而这种“泄露”现象使用valgrind是无法检测出来的。所以，malloc实现中，充分考虑了sbrk和mmap行为上的差异及优缺点，默认分配大块内存(128KB)才使用mmap分配内存空间。

![image](assets\linux-59.png)

free()函数只传入一个内存地址，为什么能知道要释放多大的内存？
前面提到，malloc返回给用户态的内存起始地址比进程的堆空间起始地址多了16字节吗？这个多出来的16字节就是保存了该内存块的描述信息，比如有该内存块的大小。

![image](assets\linux-60.png)

这样当执行free()函数时，free会对传入进来的内存地址向左偏移16字节，然后从这个16字节的分析出当前的内存块的大小，自然就知道要释放多大的内存了。







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
