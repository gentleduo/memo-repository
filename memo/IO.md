# 虚拟文件系统，文件描述符，IO重定向

## 文件描述符表、文件表、索引结点表

进程打开一个文件，会与三个表发生关联，分别是：文件描述符表、文件表、索引结点表。他们的存放地点：
每个进程都有一个属于自己的文件描述符表。
文件表存放在内核空间，由系统里的所有进程共享。
索引结点表也存放在内核空间，由所有进程所共享。

1. 文件描述符表：该表记录进程打开的文件。它的表项里面有一个指针，指向存放在内核空间的文件表中的一个表项。它向用户提供一个简单的文件描述符，使得用户可以通过方便地访问一个文件。当进程使用open打开一个文件时，内核就会在这个表中添加一个表项。如果对同一个文件打开多次，那么将有多个表项。
2. 文件表：文件表保存了进程对文件读写的偏移量。该表还保存了进程对文件的存取权限。比如，进程以O_RDONLY方式打开文件，这将记录到对应的文件表表项中。
3. 索引结点表(inode节点)：在文件系统中，也是有一个索引结点表的。这两个索引结点表有千丝万缕的关系。因为内存中的索引结点表的每一个表项都是从文件系统中读入的，并且两个索引结点表有一对一的关系。所以，内存中的索引结点表的每一个表项都对应一个具体的文件。

上面所说的三个表的功能，使得三个表紧密地联系在一起，文件描述符表项有一个指针指向文件表表项，文件表表项有一个指针指向索引结点表表项。

使用说明：

1. 不同的进程打开同一个文件：那么他们应该有各自对应的文件表表项。因为文件表表项记录了进程读写文件时的偏移量和存取权限。多个进程不可能共享一个文件偏移量。另外他们各自打开文件的权限也可能是不同的，有的是为了读、有的为了写，有的为了读写。所以，他们应该有不同的文件表表项。此外，因为是同一个文件，所以，多个进程会共享同一个索引结点表项。即他们的文件表表项指针会指向同一个索引结点
2. 使用dup函数复制一个文件描述符：复制得到的文件描述符和原描述符共享文件偏移量和一些状态。所以dup的作用仅仅是复制一个文件描述符表项，而不会复制一个文件表表项。（dup函数是一个很重要的函数。平时我们在shell里面通过 >  来进行重定向，就是通过dup函数来实现的。）
3. 同一个进程多次打开同一个文件：每打开一次同一个文件，内核就会在文件表中增加一个表项。这是因为每次open文件时使用了不同的读写权限，而读写权限是保存在文件表表项里面的。
4. 父进程使用fork创建子进程：由于fork一个子进程，子进程将复制父进程的绝大部分东西（除了进程ID、进程的父进程ID、一些时间属性、文件锁）。所以子进程复制了父进程的整个文件描述符表。

## 目录项、Inode、数据块

大部分的 Linux 文件系统（如 ext2、ext3 ）规定，一个文件由目录项、inode 和 数据块 组成：

- 目录项：包括文件名和 inode 节点号。
- Inode：又称文件索引节点，包含文件的基础信息以及数据块的指针。
- 数据块：包含文件的具体内容。

### Inode

理解inode，要从文件储存说起。文件储存在硬盘上，硬盘的最小存储单位叫做"扇区"（Sector），每个扇区储存512字节（相当于0.5KB）。操作系统读取硬盘的时候，不会一个扇区一个扇区地读取，这样效率太低，而是一次性连续读取多个扇区，即一次性读取一个"块"（block）。这种由多个扇区组成的"块"，是文件存取的最小单位。"块"的大小，最常见的是4KB，即连续8个sector组成一个block。文件数据都储存在"块"中，那么很显然，我们还必须找到一个地方储存文件的元信息，比如文件的创建者、文件的创建日期、文件的大小等等。这种储存文件元信息的区域就叫做inode，中文译名为"索引节点"。inode 包含文件的元信息，具体来说有以下内容：

- 文件的字节数。
- 文件拥有者的 User ID。
- 文件的 Group ID。
- 文件的读、写、执行权限。
- 文件的时间戳，共有三个：ctime 指 inode 上一次变动的时间，mtime 指文件内容上一次变动的时间，atime 指文件上一次打开的时间。
- 链接数，即有多少文件名指向这个 inode。
- 文件数据 block 的位置。

可以用stat命令，查看某个文件的inode信息：

```bash
[root@server01 202106_1_4_1]# stat age.bin 
  文件："age.bin"
  大小：30              块：8          IO 块：4096   普通文件
设备：802h/2050d        Inode：17678706    硬链接：1
权限：(0640/-rw-r-----)  Uid：(  995/clickhouse)   Gid：(  991/clickhouse)
最近访问：2022-11-19 16:49:04.708177961 +0800
最近更改：2022-11-19 13:32:54.623661103 +0800
最近改动：2022-11-19 13:32:54.623661103 +0800
创建时间：-
[root@server01 202106_1_4_1]# du -h age.bin 
4.0K    age.bin
# 硬盘的最小单位是扇区，通常一扇区为512B，实际上块并不存在，他只是因为文件系统如果一个扇区一个扇区的读数据太慢，所以在文件系统下提出了块的概念，一块一块的读可以大大提高效率，块大小=扇区大小*2^n。根据文件系统的不同，一个块可以是两个扇区也可以是四个。综上所述，文件读取时的最小单位是块。扇区是对硬盘而言，磁盘块是对文件系统而言。所以文件系统操作文件的最小单位是块，而磁盘的基本单位是扇区。而操作系统与内存打交道时才有了页的概念，因为如果采用内存的分页机制，内存被分为大小为4K的页面。也是一种虚拟单位。综上：操作系统操作内存是以页为基本单位、文件系统操作磁盘是以块为基本单位，而磁盘自身读写是以扇区为基本单位。
# 扇区为512字节，IO Block为4096，说明当前系统将8个扇区做为一个块。对于文件系统而言，它能读写的最小粒度为1个IO Block，此处为4K。
# Blocks指的是以512字节为单位的块的数量，或者理解为扇区数，因为一个扇区就是512字节。
# Blocks为8，那大小就是512*8=4K，和du展示的占用空间大小一致。即占用空间=Blocks*512。
```

总之，除了文件名以外的所有文件信息，都存在inode之中。当查看某个文件时，会先从inode表中查出文件属性及数据存放点，再从数据块中读取数据。文件存储结构示意图：

![image](assets\io-1.png)

inode的大小

inode也会消耗硬盘空间，所以硬盘格式化的时候，操作系统自动将硬盘分成两个区域。一个是数据区，存放文件数据；另一个是inode区（inode table），存放inode所包含的信息。每个inode节点的大小，一般是128字节或256字节。inode节点的总数，在格式化时就给定，一般是每1KB或每2KB就设置一个inode。假定在一块1GB的硬盘中，每个inode节点的大小为128字节，每1KB就设置一个inode，那么inode table的大小就会达到128MB，占整块硬盘的12.8%。查看每个硬盘分区的inode总数和已经使用的数量，可以使用df -i命令。

```bash
[root@server01 t_log]# df -i
文件系统         Inode 已用(I) 可用(I) 已用(I)% 挂载点
/dev/sda2      6359232  227718 6131514       4% /
devtmpfs        253690     328  253362       1% /dev
tmpfs           256083       1  256082       1% /dev/shm
tmpfs           256083     378  255705       1% /run
tmpfs           256083      16  256067       1% /sys/fs/cgroup
tmpfs           256083       1  256082       1% /run/user/995
tmpfs           256083       1  256082       1% /run/user/0
```

文件系统一开始就将inode与block规划好了，除非重新格式化，否则inode和block固定后就不会在发生变化。每个inode与block都与自己的编号，inode就是记录档案的属性，一个档案占用一个inode，同时记录此档案数据所在的block码。block则是实际记录档案的内容，若档案过大则会占用多个block。所以inode中存放block块的指针，并且在inode结构中，并不是只有一种指针，而是有多种。在inode表中将第1 - 12块指针当作直接块指针，由于一个指针只能指向一个block块，所以如一个block块有4K的存储空间，那么使用直接块指针的文件最大不能超过48K，如果文件大小超过48K就需要用到间接块指针，或双重间接块指针，或三重间接块指针。

直接指针

直接块指针就是inode中直接记录了block块号，直接指向block块，假如一个文件的基础信息存放在inode4中，而这个inode记录了该文件的数据存储在2，7，13，15这四个block中，此时操作系统可以根据存储顺序来读取磁盘中的block，数据的读取如下图所示

![image](assets\io-2.png)

间接块指针

间接块指针，其实和直接块指针大同小异，他们都是指针，只是直接块指针式直接指向的block块存储的是文件的数据，而间接块指针是将指向的block块继续当作指针块使用，所以间接指针指向的block块存储的是指针信息，它将继续指向下一个block块中存储的文件数据。

双重间接块指针

理解了间接指针块，那么双重间接指针块就容易理解了，它是在间接指针块的基础上将第二次指向的block块也当作指针使用，这样就可以指向更多的的block块。

三重间接块指针

三重间接块指针的道理就和双重的道理一样了，只是把第三次指向的block块也当作指针块当作指针使用，这样就可以指向更多的block块。

前面已经提到使用直接块指针的文件最大不能超过48K，假定一个指针块需要4个字节，一个block块有4K的存储空间，那么一个block块就可以存储1024个指针块，所以间接块指针的文件最大就可以存储`1024*4K=4M`的文件，以此类推使用双重间接块指针的文件最大存储空间就有`1024*1024*4K`=4G，使用三重间接块指针的文件最大存储空间就可以达到`1024*1024*1024*4K`=4T。文件是储存在硬盘上，硬盘的最小存储单位叫扇区。每个扇区能存储大小是0.5KB。进行硬盘分区后就需要将其是格式化。在Linux系统中ext4文件系统中将数据存储划分两个区域inode区与数据区，这样做的目的是提高文件的读写速度。操作系统读取硬盘的时候，不会一个个扇区地读取，这样效率太低，而是一次性连续读取多个扇区，即一次性读取一个块(block)，也即是上图中所示的簇。这种由多个扇区组成的块是文件存取的最小单位，块最常见的是4KB，也即8个扇区组成，文件里面写的数据都储存在块中。

inode表结构如图所示：

![image](assets\io-3.png)

### 目录项

普通文件是由inode+block构成，在Linux系统中，目录（directory）也是一种文件。打开目录，实际上就是打开目录文件。目录文件的结构非常简单，就是一系列目录项（dirent）的列表。每个目录项，由两部分组成：所包含文件的文件名，以及该文件名对应的inode号。通过目录项找到目录下文件的inode信息，然后就可以找到文件的block块及数据信息。如图所示：

![image](assets\io-4.png)

访问一个文件的时候，先进入目录，目录中有相应的目录项，目录项中有对应的文件名和inode号，通过里面的指针指向相应的数据块，这样就访问到文件的内容了。目录这种“特殊的文件”，可以简单地理解为是一张表，这张表里面存放了属于该目录的文件的文件名，以及所匹配的inode编号，它本身的数据也放在数据区中；读写一个文件时就这样来回的从inode和数据区之间切换。可以把文件比作一本书，inode相当于书的目录，数据区相当于书的内容，读书时得先查目录，这本书呢又放在读书馆的书架上，书架可以理解为是目录，看书前先查书架子的索引。

ls 命令只列出目录文件中的所有文件名：

```bash
[root@server01 t_log]# ls /etc
```

ls -i 命令列出整个目录文件，即文件名和 inode 号码：

```bash
[root@server01 t_log]# ls -i /etc
```

## kernel

## VFS

## FD

### lsof

lsof可以显示进程打开了哪些文件

比如：lsof -p $$

$$表示当前bash进程的进程ID号

echo $$和echo $BASHPID都是查看当前bash的进程ID，但是echo $$命令的优先级较高，比管道命令:|的优先级要高，

/proc：开机后内核中的变量及属性都会挂载在这里目录

/proc/$$/fd：当前bash进程打开的文件描述符

示例1：

1. 在当前cash下，利用exec命令使用文件描述符8读取ooxx.txt文件中的内容
2. cd到/proc/$$/fd目录，ll后可以见到当前进程除了0:标准输入、1:标准输出、2:标准错误等文件描述符外，又增加了文件描述符8
3. 使用lsof命令可以看FD(u表示读写,r表示只读)、TYPE、OFFSET等具体信息(-p:进程号 -op:进程读取的文件的偏移量)
4. 通过read a 0<&8命令读取当前进程的文件描述符中的数据作为read的输入，并赋给变量a(read命令的特点是读到换行符就结束)
5. 通过echo $a查看a的值
6. 通过lsof -op $$再一次确认文件描述符8的OFFSET，发现OFFSET的值已经改变了
7. 再开一个新的bash，同样用利用exec命令使用文件描述符6读取ooxx.txt文件中的内容
8. 然后用lsof -op $$查看OFFSET发现为0，说明在不同的bash(不同的进程)下读取同一个文件，会各自维护自己的OFFSET

```bash
[root@server03 opt]# cat ooxx.txt
11111111
22222223444

[root@server03 opt]# exec 8<ooxx.txt
[root@server03 opt]# cd /proc/$$/fd
[root@server03 fd]# ll
total 0
lrwx------. 1 root root 64 Jan 17 15:53 0 -> /dev/pts/0
lrwx------. 1 root root 64 Jan 17 15:53 1 -> /dev/pts/0
lrwx------. 1 root root 64 Jan 17 15:53 2 -> /dev/pts/0
lrwx------. 1 root root 64 Jan 17 15:56 255 -> /dev/pts/0
lr-x------. 1 root root 64 Jan 17 16:03 8 -> /opt/ooxx.txt
[root@server03 fd]# lsof -p $$
COMMAND  PID USER   FD   TYPE DEVICE  SIZE/OFF     NODE NAME
bash    1275 root  cwd    DIR    0,3         0    17930 /proc/1275/fd
bash    1275 root  rtd    DIR  253,0       252       64 /
bash    1275 root  txt    REG  253,0    960472 50332870 /usr/bin/bash
bash    1275 root  mem    REG  253,0 106176928    32642 /usr/lib/locale/locale-archive
bash    1275 root  mem    REG  253,0     61560  2114148 /usr/lib64/libnss_files-2.17.so
bash    1275 root  mem    REG  253,0   2156272    32652 /usr/lib64/libc-2.17.so
bash    1275 root  mem    REG  253,0     19248    32659 /usr/lib64/libdl-2.17.so
bash    1275 root  mem    REG  253,0    174520    32775 /usr/lib64/libtinfo.so.5.9
bash    1275 root  mem    REG  253,0    163312  2114136 /usr/lib64/ld-2.17.so
bash    1275 root  mem    REG  253,0     26970 33655849 /usr/lib64/gconv/gconv-modules.cache
bash    1275 root    0u   CHR  136,0       0t0        3 /dev/pts/0
bash    1275 root    1u   CHR  136,0       0t0        3 /dev/pts/0
bash    1275 root    2u   CHR  136,0       0t0        3 /dev/pts/0
bash    1275 root    8r   REG  253,0        22 17909683 /opt/ooxx.txt
bash    1275 root  255u   CHR  136,0       0t0        3 /dev/pts/0
[root@server03 fd]# lsof -op $$
COMMAND  PID USER   FD   TYPE DEVICE OFFSET     NODE NAME
bash    1275 root  cwd    DIR    0,3           17930 /proc/1275/fd
bash    1275 root  rtd    DIR  253,0              64 /
bash    1275 root  txt    REG  253,0        50332870 /usr/bin/bash
bash    1275 root  mem    REG  253,0           32642 /usr/lib/locale/locale-archive
bash    1275 root  mem    REG  253,0         2114148 /usr/lib64/libnss_files-2.17.so
bash    1275 root  mem    REG  253,0           32652 /usr/lib64/libc-2.17.so
bash    1275 root  mem    REG  253,0           32659 /usr/lib64/libdl-2.17.so
bash    1275 root  mem    REG  253,0           32775 /usr/lib64/libtinfo.so.5.9
bash    1275 root  mem    REG  253,0         2114136 /usr/lib64/ld-2.17.so
bash    1275 root  mem    REG  253,0        33655849 /usr/lib64/gconv/gconv-modules.cache
bash    1275 root    0u   CHR  136,0    0t0        3 /dev/pts/0
bash    1275 root    1u   CHR  136,0    0t0        3 /dev/pts/0
bash    1275 root    2u   CHR  136,0    0t0        3 /dev/pts/0
bash    1275 root    8r   REG  253,0    0t0 17909683 /opt/ooxx.txt
bash    1275 root  255u   CHR  136,0    0t0        3 /dev/pts/0
[root@server03 /]# read a 0<&8
[root@server03 /]# echo $a
11111111
[root@server03 /]# lsof -op $$
COMMAND  PID USER   FD   TYPE DEVICE OFFSET     NODE NAME
bash    1275 root  cwd    DIR  253,0              64 /
bash    1275 root  rtd    DIR  253,0              64 /
bash    1275 root  txt    REG  253,0        50332870 /usr/bin/bash
bash    1275 root  mem    REG  253,0           32642 /usr/lib/locale/locale-archive
bash    1275 root  mem    REG  253,0         2114148 /usr/lib64/libnss_files-2.17.so
bash    1275 root  mem    REG  253,0           32652 /usr/lib64/libc-2.17.so
bash    1275 root  mem    REG  253,0           32659 /usr/lib64/libdl-2.17.so
bash    1275 root  mem    REG  253,0           32775 /usr/lib64/libtinfo.so.5.9
bash    1275 root  mem    REG  253,0         2114136 /usr/lib64/ld-2.17.so
bash    1275 root  mem    REG  253,0        33655849 /usr/lib64/gconv/gconv-modules.cache
bash    1275 root    0u   CHR  136,0    0t0        3 /dev/pts/0
bash    1275 root    1u   CHR  136,0    0t0        3 /dev/pts/0
bash    1275 root    2u   CHR  136,0    0t0        3 /dev/pts/0
bash    1275 root    8r   REG  253,0    0t9 17909683 /opt/ooxx.txt
bash    1275 root  255u   CHR  136,0    0t0        3 /dev/pts/0

login as: root
root@192.168.56.112's password:
Last login: Mon Jan 17 15:53:41 2022 from 192.168.56.1
[root@server03 ~]# exec 6</opt/ooxx.txt
[root@server03 ~]# lsof -op $$
COMMAND  PID USER   FD   TYPE DEVICE OFFSET     NODE NAME
bash    1356 root  cwd    DIR  253,0        33574977 /root
bash    1356 root  rtd    DIR  253,0              64 /
bash    1356 root  txt    REG  253,0        50332870 /usr/bin/bash
bash    1356 root  mem    REG  253,0           32642 /usr/lib/locale/locale-arch                                                                                        ive
bash    1356 root  mem    REG  253,0         2114148 /usr/lib64/libnss_files-2.1                                                                                        7.so
bash    1356 root  mem    REG  253,0           32652 /usr/lib64/libc-2.17.so
bash    1356 root  mem    REG  253,0           32659 /usr/lib64/libdl-2.17.so
bash    1356 root  mem    REG  253,0           32775 /usr/lib64/libtinfo.so.5.9
bash    1356 root  mem    REG  253,0         2114136 /usr/lib64/ld-2.17.so
bash    1356 root  mem    REG  253,0        33655849 /usr/lib64/gconv/gconv-modu                                                                                        les.cache
bash    1356 root    0u   CHR  136,1    0t0        4 /dev/pts/1
bash    1356 root    1u   CHR  136,1    0t0        4 /dev/pts/1
bash    1356 root    2u   CHR  136,1    0t0        4 /dev/pts/1
bash    1356 root    6r   REG  253,0    0t0 17909683 /opt/ooxx.txt
bash    1356 root  255u   CHR  136,1    0t0        4 /dev/pts/1
```

## inode id

## 文件类型

### -：普通文件 REG（可执行、图片、文本）

### d：目录

### l：链接

1. 硬链接：inode号相同、表示一个物理文件在虚拟文件系统中两个不同的path、将源文件删除后对创建的链接没有影响，只是通过ll命令看到的链接数目变成1了。

   ```bash
   [root@localhost app]# vim src
   [root@localhost app]# ll
   total 4
   -rw-r--r--. 1 root root 13 Jan 15 15:08 src
   [root@localhost app]# ln src dest
   [root@localhost app]# ll
   total 8
   -rw-r--r--. 2 root root 13 Jan 15 15:08 dest
   -rw-r--r--. 2 root root 13 Jan 15 15:08 src
   [root@localhost app]# stat src
     File: ‘src’
     Size: 13              Blocks: 8          IO Block: 4096   regular file
   Device: fd00h/64768d    Inode: 1553        Links: 2
   Access: (0644/-rw-r--r--)  Uid: (    0/    root)   Gid: (    0/    root)
   Context: unconfined_u:object_r:default_t:s0
   Access: 2022-01-15 15:08:13.431000000 +0800
   Modify: 2022-01-15 15:08:13.431000000 +0800
   Change: 2022-01-15 15:08:22.190000000 +0800
    Birth: -
   [root@localhost app]# stat dest
     File: ‘dest’
     Size: 13              Blocks: 8          IO Block: 4096   regular file
   Device: fd00h/64768d    Inode: 1553        Links: 2
   Access: (0644/-rw-r--r--)  Uid: (    0/    root)   Gid: (    0/    root)
   Context: unconfined_u:object_r:default_t:s0
   Access: 2022-01-15 15:08:13.431000000 +0800
   Modify: 2022-01-15 15:08:13.431000000 +0800
   Change: 2022-01-15 15:08:22.190000000 +0800
    Birth: -
   [root@localhost app]# rm -rf src
   [root@localhost app]# ll
   total 4
   -rw-r--r--. 1 root root 13 Jan 15 15:08 dest
   
   ```

2. 软链接：inode不相同、软链接相当于windows中的一个快捷方式；删除源文件后，软链接指向就丢失了

   ```bash
   [root@localhost app]# vim src
   [root@localhost app]# ll
   total 4
   -rw-r--r--. 1 root root 16 Jan 15 15:18 src
   [root@localhost app]# ln -s src dest
   [root@localhost app]# ll
   total 4
   lrwxrwxrwx. 1 root root  3 Jan 15 15:19 dest -> src
   -rw-r--r--. 1 root root 16 Jan 15 15:18 src
   [root@localhost app]# stat src
     File: ‘src’
     Size: 16              Blocks: 8          IO Block: 4096   regular file
   Device: fd00h/64768d    Inode: 1554        Links: 1
   Access: (0644/-rw-r--r--)  Uid: (    0/    root)   Gid: (    0/    root)
   Context: unconfined_u:object_r:default_t:s0
   Access: 2022-01-15 15:18:53.294000000 +0800
   Modify: 2022-01-15 15:18:53.294000000 +0800
   Change: 2022-01-15 15:18:53.294000000 +0800
    Birth: -
   [root@localhost app]# stat dest
     File: ‘dest’ -> ‘src’
     Size: 3               Blocks: 0          IO Block: 4096   symbolic link
   Device: fd00h/64768d    Inode: 1553        Links: 1
   Access: (0777/lrwxrwxrwx)  Uid: (    0/    root)   Gid: (    0/    root)
   Context: unconfined_u:object_r:default_t:s0
   Access: 2022-01-15 15:19:06.777000000 +0800
   Modify: 2022-01-15 15:19:04.852000000 +0800
   Change: 2022-01-15 15:19:04.852000000 +0800
    Birth: -
    [root@localhost app]# rm -rf src
   [root@localhost app]# ll
   total 0
   lrwxrwxrwx. 1 root root 3 Jan 15 15:19 dest -> src
   [root@localhost app]# cat dest
   cat: dest: No such file or directory
   ```

3. 共同点：修改任意一方，另一方也会同步更新

### b：块设备

1. 创建一个100M的空的磁盘镜像文件 (/dev/zero填充的100个块大小为1M的磁盘)

2. 使用 losetup将磁盘镜像文件虚拟成块设备

3. 格式化/dev/loop0

4. 创建一个目录 mkdir -p /mnt/ooxx

5. 挂载块设备：将/dev/loop0挂载到/mnt/ooxx目录

6. 复制bash及所依赖的动态链接库，ldd命令可以列出动态库依赖关系

7. 使用chroot命令将根目录换成指定的目录

8.  卸载loop设备 

9. 注意：mount -o loop mydisk.img /mnt/ooxx 

   mount 命令的 -o loop 选项可以将任意一个 loopback 文件系统挂载。 

   上面的 mount 命令实际等价于下面两条命令： 

   losetup /dev/loop0 mydisk.img

   mount /dev/loop0 /mnt/ooxx

   因此实际上，mount -o loop 在内部已经默认的将文件和 /dev/loop0 挂载起来了。 

   然而对于第一种方法(mount -o loop)并不能适用于所有的场景。比如，我们想创建一个硬盘文件，然后对该文件进行分区，接着挂载其中一个子分区，这时就不能用 -o loop 这种方法了。因此必须如下做： 

   losetup /dev/loop0 loopfile.img

   fdisk /dev/loop0

```bash
[root@localhost app]# dd if=/dev/zero of=mydisk.img bs=1048576 count=100
100+0 records in
100+0 records out
104857600 bytes (105 MB) copied, 0.0606825 s, 1.7 GB/s
[root@localhost app]# losetup /dev/loop0 mydisk.img
[root@localhost app]# mke2fs /dev/loop0
mke2fs 1.42.9 (28-Dec-2013)
Discarding device blocks: done
Filesystem label=
OS type: Linux
Block size=1024 (log=0)
Fragment size=1024 (log=0)
Stride=0 blocks, Stripe width=0 blocks
25688 inodes, 102400 blocks
5120 blocks (5.00%) reserved for the super user
First data block=1
Maximum filesystem blocks=67371008
13 block groups
8192 blocks per group, 8192 fragments per group
1976 inodes per group
Superblock backups stored on blocks:
        8193, 24577, 40961, 57345, 73729

Allocating group tables: done
Writing inode tables: done
Writing superblocks and filesystem accounting information: done
[root@localhost app]# mkdir -p /mnt/ooxx
[root@localhost app]# mount -t ext2 /dev/loop0 /mnt/ooxx
[root@localhost app]# df -h
Filesystem               Size  Used Avail Use% Mounted on
/dev/mapper/centos-root   46G  1.1G   45G   3% /
devtmpfs                 1.9G     0  1.9G   0% /dev
tmpfs                    1.9G     0  1.9G   0% /dev/shm
tmpfs                    1.9G  8.4M  1.9G   1% /run
tmpfs                    1.9G     0  1.9G   0% /sys/fs/cgroup
/dev/sda1               1014M  143M  872M  15% /boot
tmpfs                    380M     0  380M   0% /run/user/0
/dev/loop0                97M  1.6M   91M   2% /mnt/ooxx
[root@localhost ooxx]# whereis bash
bash: /usr/bin/bash /usr/share/man/man1/bash.1.gz
[root@localhost ooxx]# ll
total 12
drwx------. 2 root root 12288 Jan 15 15:32 lost+found
[root@localhost ooxx]# mkdir bin
[root@localhost ooxx]# cp /bin/bash bin
[root@localhost bin]# ldd bash
        linux-vdso.so.1 =>  (0x00007ffede774000)
        libtinfo.so.5 => /lib64/libtinfo.so.5 (0x00007f2c974ab000)
        libdl.so.2 => /lib64/libdl.so.2 (0x00007f2c972a7000)
        libc.so.6 => /lib64/libc.so.6 (0x00007f2c96ee3000)
        /lib64/ld-linux-x86-64.so.2 (0x000056461103a000)
[root@localhost ooxx]# cd /mnt/ooxx/
[root@localhost ooxx]# mkdir lib64
[root@localhost ooxx]# cp /lib64/{libtinfo.so.5,libdl.so.2,libc.so.6,ld-linux-x86-64.so.2} lib64/
[root@localhost ooxx]# chroot ./
bash-4.2# echo $$
1507
bash-4.2# ls
bash: ls: command not found
bash-4.2# vi
bash: vi: command not found
bash-4.2# cat
bash: cat: command not found
bash-4.2# echo "hello word" > /abc.txt
bash-4.2# exit
exit
[root@localhost ooxx]# echo $$
1304
[root@localhost ooxx]# ll
total 18
-rw-r--r--. 1 root root    11 Jan 15 16:12 abc.txt
drwxr-xr-x. 2 root root  1024 Jan 15 16:06 bin
drwxr-xr-x. 2 root root  1024 Jan 15 16:09 lib64
drwx------. 2 root root 12288 Jan 15 15:32 lost+found
[root@localhost /]# umount /mnt/ooxx/
[root@localhost /]# losetup -d /dev/loop0
```

### c：字符设备 CHAR

### s：socket

```bash
[root@server03 ~]# echo $$
1277
[root@server03 ~]# cd /proc/1277/fd
[root@server03 fd]# ll
total 0
lrwx------. 1 root root 64 Jan 19 14:24 0 -> /dev/pts/0
lrwx------. 1 root root 64 Jan 19 14:24 1 -> /dev/pts/0
lrwx------. 1 root root 64 Jan 19 14:24 2 -> /dev/pts/0
lrwx------. 1 root root 64 Jan 19 14:34 255 -> /dev/pts/0
[root@server03 fd]# exec 8<> /dev/tcp/www.baidu.com/80
[root@server03 fd]# ll
total 0
lrwx------. 1 root root 64 Jan 19 14:24 0 -> /dev/pts/0
lrwx------. 1 root root 64 Jan 19 14:24 1 -> /dev/pts/0
lrwx------. 1 root root 64 Jan 19 14:24 2 -> /dev/pts/0
lrwx------. 1 root root 64 Jan 19 14:34 255 -> /dev/pts/0
lrwx------. 1 root root 64 Jan 19 14:35 8 -> socket:[18984]
[root@server03 fd]# lsof -op $$
COMMAND  PID USER   FD   TYPE DEVICE OFFSET     NODE NAME
bash    1277 root  cwd    DIR    0,3           17911 /proc/1277/fd
bash    1277 root  rtd    DIR  253,0              64 /
bash    1277 root  txt    REG  253,0        50332870 /usr/bin/bash
bash    1277 root  mem    REG  253,0         2114158 /usr/lib64/libresolv-2.17.so
bash    1277 root  mem    REG  253,0         2114146 /usr/lib64/libnss_dns-2.17.so
bash    1277 root  mem    REG  253,0           32642 /usr/lib/locale/locale-archive
bash    1277 root  mem    REG  253,0         2114148 /usr/lib64/libnss_files-2.17.so
bash    1277 root  mem    REG  253,0           32652 /usr/lib64/libc-2.17.so
bash    1277 root  mem    REG  253,0           32659 /usr/lib64/libdl-2.17.so
bash    1277 root  mem    REG  253,0           32775 /usr/lib64/libtinfo.so.5.9
bash    1277 root  mem    REG  253,0         2114136 /usr/lib64/ld-2.17.so
bash    1277 root  mem    REG  253,0        33655849 /usr/lib64/gconv/gconv-modules.cache
bash    1277 root    0u   CHR  136,0    0t0        3 /dev/pts/0
bash    1277 root    1u   CHR  136,0    0t0        3 /dev/pts/0
bash    1277 root    2u   CHR  136,0    0t0        3 /dev/pts/0
bash    1277 root    8u  IPv4  18984    0t0      TCP server03:57156->180.101.49.12:http (ESTABLISHED)
bash    1277 root  255u   CHR  136,0    0t0        3 /dev/pts/0
```

### p：pipeline

#### 重定向

ls athena1 > 1111 **2>&** 1

重定向描述符(无论是输入还是输出)它的左边放的是你要改变的这个程序它自身的文件描述符，且两者之间不能有空白符（在命令行中空白符极其敏感，会做split切割，切开之后它会认为文件描述符是前面命令的参数）。重定向右边是可以有空白符的，默认右边放的都是文件，但是如果右边放的是另外一个文件描述符必须在重定向描述符后面放一个&符号，并且之间不能有空格。

1. 通过命令ls ./ /ooxx显示当前目录和一个不存在的目录，那么结果会输出到两个文件描述符，一个标准输出，另一个是标准错误
2. 可是使用重定向将内容都输出到一个文件中，但是重定向默认是覆盖的并不是追加的
3. 另一个写法是用一个文件描述符指向另一个文件描述符，另一个文件描述符指向一个文件，但是要注意重定向描述符的绑定是有顺序的，比如当2指向1的时候，如果此时1还没有指向文件，指向的是屏幕，所以则2指向1，又指向了屏幕。接着再1指向文件的时候，2依然指向的屏幕。

```bash
[root@server03 opt]# ls ./ /ooxx
ls: cannot access /ooxx: No such file or directory
./:
1111  athena      DockerRuntime  DockerWorker  genius  kafka  ns.c        ooxx.txt  springcloud  tttt
app   containerd  DockerTmp      entrypoint.d  images  ns     OldRootDir  pcstat    te           zookeeper
[root@server03 opt]# ls ./ /ooxx 1> ls03.out 2> ls03.out
[root@server03 opt]# cat ls03.out
./:
1111
app
athena
containerd
DockerRuntime
DockerTmp
DockerWorker
entrypoint.d
genius
images
kafka
ls03.out
ns
ns.c
OldRootDir
ooxx.txt
pcstat
springcloud
te
tttt
zookeeper
[root@server03 opt]# ls ./ /ooxx 2>& 1 1> ls04.out
ls: cannot access /ooxx: No such file or directory
[root@server03 opt]# cat ls04.out
./:
1111
app
athena
containerd
DockerRuntime
DockerTmp
DockerWorker
entrypoint.d
genius
images
kafka
ls03.out
ls04.out
lso4.out
ns
ns.c
OldRootDir
ooxx.txt
pcstat
springcloud
te
tttt
zookeeper
[root@server03 opt]# ls ./ /ooxx 1> ls04.out 2>& 1
[root@server03 opt]# cat ./ls04.out
ls: cannot access /ooxx: No such file or directory
./:
1111
app
athena
containerd
DockerRuntime
DockerTmp
DockerWorker
entrypoint.d
genius
images
kafka
ls03.out
ls04.out
lso4.out
ns
ns.c
OldRootDir
ooxx.txt
pcstat
springcloud
te
tttt
zookeeper
```

#### 管道

前面的输出作为后面的输入

显示文件的第八行

```bash
[root@server03 opt]# cat test.txt
adfadsf
asdfadsf
adf12
311
23'
qdsf
adf'
12e
sdfa
123132
1231
12dsf
123
12
90rweifs
2jflas
[root@server03 opt]# head -8 test.txt | tail -1
12e
```

#### 父子进程

```bash
[root@server03 opt]# echo $$
1277
[root@server03 opt]# /bin/bash
[root@server03 opt]# echo $$
10024
[root@server03 opt]# pstree
systemd─┬─NetworkManager─┬─dhclient
        │                └─2*[{NetworkManager}]
        ├─agetty
        ├─auditd───{auditd}
        ├─crond
        ├─dbus-daemon───{dbus-daemon}
        ├─gssproxy───5*[{gssproxy}]
        ├─lvmetad
        ├─master─┬─pickup
        │        └─qmgr
        ├─polkitd───5*[{polkitd}]
        ├─rpcbind
        ├─rsyslogd───2*[{rsyslogd}]
        ├─sshd───sshd───bash───bash───pstree
        ├─systemd-journal
        ├─systemd-logind
        ├─systemd-udevd
        └─tuned───4*[{tuned}]
[root@server03 opt]# ps -ef | grep 1277
root      1277  1273  0 14:24 pts/0    00:00:00 -bash
root     10024  1277  0 17:25 pts/0    00:00:00 /bin/bash
root     10105 10024  0 17:27 pts/0    00:00:00 grep --color=auto 1277
```

##### export

由于进程隔离，所以父进程定义的变量在子进程中是看不到的，但是使用export将变量导出后，那么由这个进程

fork出的子进程都能看到这个变量了

```bash
[root@server03 opt]# x=100
[root@server03 opt]# echo $x
100
[root@server03 opt]# /bin/bash
[root@server03 opt]# echo $x

[root@server03 opt]# exit
exit
[root@server03 opt]# export x
[root@server03 opt]# /bin/bash
[root@server03 opt]# echo $x
100
```

##### 指令块

在linux中{}代表程序块。注意{的后面一定先先跟空格，然后命令必须以;结束

linux会启动新的进程来执行管道左右两边的命令或程序块等

echo $$和echo $BASHPID都是查看当前bash的进程ID，但是echo $$命令的优先级较高，比管道命令:|的优先级要高，

{ echo $$ ; ll ./ ; } | { cat ; }打印出来的是父进程的进程ID，因为linux在解释的时候先执行echo $$，所以$$会被先解析成当前进程的进程ID，然后才根据管道命令分别启动两个进程执行代码块

{ echo $BASHPID ; ll ./ ; } | { cat ; }打印出来的是子进程的进程ID，因为linux先会根据管道命令分别启动两个进程执行代码块，然后在子进程中分别执行两个代码块中的命令

{ echo $BASHPID ; read x ; } | { cat ; echo $BASHPID ; read y ; }父bash会启动两个子进程，前一个子进程的输出是后一个子进程的输入

```bash
[root@server03 ~]# { echo "123123123" ; echo "adfadsf" ; }
123123123
adfadsf
[root@server03 ~]# a=1
[root@server03 ~]# echo $a
1
[root@server03 ~]# { a=9 ; echo "adfadf" ; } | cat
adfadf
[root@server03 ~]# echo $a
1
[root@server03 ~]# { echo $$ ; ll ./ ; } | { cat ; }
1280
total 20
-rw-------. 1 root root  1339 Nov 14  2019 anaconda-ks.cfg
-rw-r--r--. 1 root root 14132 Nov 17 14:48 zookeeper.out
[root@server03 ~]# { echo $BASHPID ; ll ./ ; } | { cat ; }
1335
total 20
-rw-------. 1 root root  1339 Nov 14  2019 anaconda-ks.cfg
-rw-r--r--. 1 root root 14132 Nov 17 14:48 zookeeper.out
[root@server03 ~]# { echo $BASHPID ; read x ; } | { cat ; echo $BASHPID ; read y ; }
1391
[root@server03 ~]# ps -ef | grep 1280
root      1280  1276  0 17:58 pts/0    00:00:00 -bash
root      1391  1280  0 18:37 pts/0    00:00:00 -bash
root      1392  1280  0 18:37 pts/0    00:00:00 -bash
root      1395  1361  0 18:37 pts/1    00:00:00 grep --color=auto 1280
[root@server03 ~]# lsof -op 1391
COMMAND  PID USER   FD   TYPE DEVICE OFFSET     NODE NAME
bash    1391 root  cwd    DIR  253,0        33574977 /root
bash    1391 root  rtd    DIR  253,0              64 /
bash    1391 root  txt    REG  253,0        50332870 /usr/bin/bash
bash    1391 root  mem    REG  253,0           32642 /usr/lib/locale/locale-archive
bash    1391 root  mem    REG  253,0         2114148 /usr/lib64/libnss_files-2.17.so
bash    1391 root  mem    REG  253,0           32652 /usr/lib64/libc-2.17.so
bash    1391 root  mem    REG  253,0           32659 /usr/lib64/libdl-2.17.so
bash    1391 root  mem    REG  253,0           32775 /usr/lib64/libtinfo.so.5.9
bash    1391 root  mem    REG  253,0         2114136 /usr/lib64/ld-2.17.so
bash    1391 root  mem    REG  253,0        33655849 /usr/lib64/gconv/gconv-modules.cache
bash    1391 root    0u   CHR  136,0    0t0        3 /dev/pts/0
bash    1391 root    1w  FIFO    0,8    0t0    18787 pipe
bash    1391 root    2u   CHR  136,0    0t0        3 /dev/pts/0
bash    1391 root  255u   CHR  136,0    0t0        3 /dev/pts/0
[root@server03 ~]# lsof -op 1392
COMMAND  PID USER   FD   TYPE DEVICE OFFSET     NODE NAME
bash    1392 root  cwd    DIR  253,0        33574977 /root
bash    1392 root  rtd    DIR  253,0              64 /
bash    1392 root  txt    REG  253,0        50332870 /usr/bin/bash
bash    1392 root  mem    REG  253,0           32642 /usr/lib/locale/locale-archive
bash    1392 root  mem    REG  253,0         2114148 /usr/lib64/libnss_files-2.17.so
bash    1392 root  mem    REG  253,0           32652 /usr/lib64/libc-2.17.so
bash    1392 root  mem    REG  253,0           32659 /usr/lib64/libdl-2.17.so
bash    1392 root  mem    REG  253,0           32775 /usr/lib64/libtinfo.so.5.9
bash    1392 root  mem    REG  253,0         2114136 /usr/lib64/ld-2.17.so
bash    1392 root  mem    REG  253,0        33655849 /usr/lib64/gconv/gconv-modules.cache
bash    1392 root    0r  FIFO    0,8    0t0    18787 pipe
bash    1392 root    1u   CHR  136,0    0t0        3 /dev/pts/0
bash    1392 root    2u   CHR  136,0    0t0        3 /dev/pts/0
bash    1392 root  255u   CHR  136,0    0t0        3 /dev/pts/0
```

### eventpool

## pagecache

### pcstat工具安装

1. 在windows环境下使用代理下载pcstat.x86_64

   https://github.com/tobert/pcstat/raw/2014-05-02-01/pcstat.x86_64

2. 上产至linux服务器

3. mv pcstat.x86_64 pcstat

4. chmod 755 pcstat

5. cp ./pcstat /usr/bin

pcstat /bin/bash：查看/bin/bash有没有被缓存

重启服务器后使用pcstat /usr/sbin/ss查看ss有没有被缓存，发现Cached为0，然后使用完一次后再查看ss有没有被缓存，此时Cached为29，表示/usr/sbin/ss已经被kernel缓存住了。

```bash
[root@server03 ~]# pcstat /bin/bash
|-----------+----------------+------------+-----------+---------|
| Name      | Size           | Pages      | Cached    | Percent |
|-----------+----------------+------------+-----------+---------|
| /bin/bash | 960472         | 235        | 235       | 100.000 |
|-----------+----------------+------------+-----------+---------|
[root@server03 ~]# pcstat /usr/sbin/ss
|--------------+----------------+------------+-----------+---------|
| Name         | Size           | Pages      | Cached    | Percent |
|--------------+----------------+------------+-----------+---------|
| /usr/sbin/ss | 114784         | 29         | 0         | 000.000 |
|--------------+----------------+------------+-----------+---------|
[root@server03 ~]# ss -nltp | grep 8080
[root@server03 ~]# pcstat /usr/sbin/ss
|--------------+----------------+------------+-----------+---------|
| Name         | Size           | Pages      | Cached    | Percent |
|--------------+----------------+------------+-----------+---------|
| /usr/sbin/ss | 114784         | 29         | 29        | 100.000 |
|--------------+----------------+------------+-----------+---------|
```

### PageCache和BufferCache的区别

PageCache实际上是针对文件系统的，是文件的缓存，在文件层面上的数据（在文件系统构架里，组成一个文件的有两部分：文件的元数据，即FCB，还有文件的业务数据，即比如平时我们用记事本程序打开一个文件所看到的数据。“在文件层面上的数据”指的就是文件的业务数据）会缓存到PageCache。文件的逻辑层需要映射到实际的物理磁盘，这种映射关系由文件系统来完成。当PageCache的数据需要刷新时，PageCache中的数据交给BufferCache。

BufferCache是针对磁盘块（即扇区）的缓存（即BufferCache允许缓存的数据可以是来自一个磁盘上任意一个扇区的数据），也就是在没有文件系统的情况下（准确的表达是，无论有否文件系统的存在，设备驱动程序读取的数据（也即直接对磁盘进行操作的数据）都是缓存到BufferCache中的），直接对磁盘进行操作的数据会缓存到BufferCache中，例如，文件系统的元数据都会缓存到BufferCache中。

也就是说，即设备驱动程序对磁盘进行读写的基本单位是扇区，所以内存上对磁盘进行读写的基本单位是扇区。应用进程对PageCache进行读写的基本单位则是文件系统块。例如，BufferCache通过设备驱动程序从磁盘上读取一些数据（这些数据可能分散开的，也可能连续一起，如果更巧的话，正好连续成一个文件系统块的大小）是以扇区为基本单位，接着文件系统（感觉是内核）会将BufferCache映射到PageCache，之后，应用进程从PageCache上将这些数据读取到自己上时则是以文件系统块为基本单位的 。 
BufferCache算是属于硬盘分区这个层次的一部分，PageCache算是属于硬盘分区这个层次的上一层即文件系统这个层次的一部分。文件系统对磁盘有一个自己的格式划分（即以文件系统块为划分单位），但是文件系统（的进程）不能直接去读写磁盘（即就不可能以文件系统块为单位来直接读写磁盘上的数据），文件系统进程（内核进程）只能调用驱动程序去读写磁盘上的数据，驱动程序直接读写磁盘上的数据是以扇区为基本单位的，而直接对磁盘进行操作的数据则是（程序编程时规划）缓存到BufferCache中。
简单说来，BufferCache允许缓存的数据可以是来自一个磁盘上任意一个扇区的数据，而PageCache则规划就是用来缓存来自在文件系统下的任一文件的业务数据所存放的那些扇区。

### Linux内核PageCache和BufferCache关系及演化历史

#### 两类缓存各自的作用

##### Page Cache

Page Cache以Page为单位，缓存文件内容。缓存在Page Cache中的文件数据，能够更快的被用户读取。同时对于带buffer的写入操作，数据在写入到Page Cache中即可立即返回，而不需等待数据被实际持久化到磁盘，进而提高了上层应用读写文件的整体性能。

##### Buffer Cache

磁盘的最小数据单位为sector，每次读写磁盘都是以sector为单位对磁盘进行操作。sector大小跟具体的磁盘类型有关，有的为512Byte， 有的为4K Bytes。无论用户是希望读取1个byte，还是10个byte，最终访问磁盘时，都必须以sector为单位读取，如果裸读磁盘，那意味着数据读取的效率会非常低。同样，如果用户希望向磁盘某个位置写入(更新)1个byte的数据，他也必须整个刷新一个sector，言下之意，则是在写入这1个byte之前，我们需要先将该1byte所在的磁盘sector数据全部读出来，在内存中，修改对应的这1个byte数据，然后再将整个修改后的sector数据，一口气写入磁盘。为了降低这类低效访问，尽可能的提升磁盘访问性能，内核会在磁盘sector上构建一层缓存，它以sector的整数倍粒度为单位(block)，缓存部分sector数据在内存中，当有数据读取请求时，它能够直接从内存中将对应数据读出。当有数据写入时，它可以直接在内存中直接更新指定部分的数据，然后再通过异步方式，把更新后的数据写回到对应磁盘的sector中。这层缓存则是块缓存Buffer Cache。

#### 两类缓存的逻辑关系

从linux-2.6.18的内核源码来看，PageCache和BufferCache是一个事物的两种表现：对于一个Page而言，对上，它是某个File的一个PageCache，而对下，它同样是一个Device上的一组BufferCache。

![image][0]

File在地址空间上，以4K(pagesize)为单位进行切分，每一个4k都可能对应到一个page上（这里可能的含义是指，只有被缓存的部分，才会对应到page上，没有缓存的部分，则不会对应），而这个4k的在内存中被缓存的page，就是这个文件的一个PageCache。而对于落磁盘的一个文件而言，最终，这个4k的pagecache，还需要映射到一组磁盘block对应的buffercache上，假设block为1k，那么每个pagecache将对应一组(4个)buffercache，而每一个buffercache，则有一个对应的buffercache与deviceblock映射关系的描述符：buffer_head，这个描述符记录了这个buffercache对应的block在磁盘上的具体位置。

![image][1]

#### 两类缓存的演进历史

虽然，目前Linux Kernel代码中，Page Cache和Buffer Cache实际上是统一的，无论是文件的Page Cache还是Block的Buffer Cache最终都统一到Page上。但是，在阅读较老代码时，我们能够看出，这两块缓存的实现，原本是完全分开的。

##### 第一阶段：仅有Buffer Cache

 在Linux-0.11版本的代码中，我们会看到，buffer cache是完全独立的实现，甚至都还没有基于page作为内存单元，而是以原始指针的系形式出现。每一个block sector，在kernel内部对应一个独立的buffer cache单元，这个buffer cache单元通过buffer head来描述。

##### 第二阶段：Page Cache、Buffer Cache两者并存

到Linux-2.2版本时，磁盘文件访问的高速缓冲仍然是缓冲区高速缓冲(Buffer Cache)。其访问模式与上面Linux-0.11版本的访问逻辑基本类似。但此时，Buffer Cache已基于page来分配内存，buffer_head内部，已经有了关于所在page的一些信息，此时的buffer cache基于page来分配内存，但是与Page Cache完全独立，一点关系都没有。此时的Page Cache仅负责其中mmap部分的处理，而Buffer Cache实际上负责所有对磁盘的IO访问。从上面图中，我们也可看出其中一个问题：write绕过了Page Cache，这里导致了一个同步问题。当write发生时，有效数据是在Buffer Cache中，而不是在Page Cache中。这就导致mmap访问的文件数据可能存在不一致问题。为了解决这个问题，所有基于磁盘文件系统的write，都需要调用update_vm_cache()函数，该操作会修改write相关Buffer Cache对应的Page Cache。同样，正是这样Page Cache、Buffer Cache分离的设计，导致基于磁盘的文件，同一份数据，可能在Page Cache中有一份，而同时，却还在Buffer Cache中有一份。

##### 第三阶段：Page Cache、Buffer Cache两者融合

介于上述Page Cache、Buffer Cache分离设计的弊端，Linux-2.4版本中对Page Cache、Buffer Cache的实现进行了融合，融合后的Buffer Cache不再以独立的形式存在，Buffer Cache的内容，直接存在于Page Cache中，同时，保留了对Buffer Cache的描述符单元：buffer_head

### dirty

系统脏页配置

```bash
[root@server03 ~]# sysctl -a | grep dirty
sysctl: reading key "net.ipv6.conf.all.stable_secret"
sysctl: reading key "net.ipv6.conf.default.stable_secret"
sysctl: reading key "net.ipv6.conf.enp0s3.stable_secret"
sysctl: reading key "net.ipv6.conf.enp0s8.stable_secret"
sysctl: reading key "net.ipv6.conf.lo.stable_secret"
vm.dirty_background_bytes = 0
vm.dirty_background_ratio = 10
vm.dirty_bytes = 0
vm.dirty_expire_centisecs = 3000
vm.dirty_ratio = 30
vm.dirty_writeback_centisecs = 500
```

配置有如下特点：

1. 当脏数据达到可用内存的10%时唤醒回刷进程
2. 当脏数据达到可用内存的30%时，必须将所有脏数据提交到磁盘，同时所有新的I/O块都会被阻塞，直到脏数据被写入磁盘
3. 每隔5s唤醒一次回刷进程
4. 内存中脏数据存在时间超过30s则在下一次唤醒时回刷  
4. dirty跟pagecache是两个概念，操作系统在内存的使用未超过上限时，内核不会主动释放pagecache（在内存不够时内核会使用LRU算法淘汰一些未被引用的或者长时间未被使用的page，并且在淘汰前会判断是不是脏页，如果是脏页会先刷盘后再淘汰）。虽然在当到达内核设置的脏页刷盘阈值时会触发flush将脏页刷入磁盘，但是数据还是缓存在pagecache里面。

查看脏页数量

```bash
[root@server03 ~]# cat /proc/vmstat | grep dirty
nr_dirty 2
nr_dirty_threshold 573073
nr_dirty_background_threshold 191024
```

配置项说明：

1. vm.dirty_background_ratio = 10  是内存可以填充脏数据的百分比。这些脏数据稍后会写入磁盘，pdflush/flush/kdmflush这些后台进程会稍后清理脏数据。比如，内存为32G，那么有3.2G的脏数据可以待着内存里，超过3.2G的话就会有后台进程来清理。
2. vm.dirty_ratio = 30 是可以用脏数据填充的绝对最大系统内存量，当系统到达此点时，必须将所有脏数据提交到磁盘，同时所有新的I/O块都会被阻塞，直到脏数据被写入磁盘。这通常是长I/O卡顿的原因，但这也是保证内存中不会存在过量脏数据的保护机制。
3. vm.dirty_background_bytes和vm.dirty_bytes是另一种指定这些参数的方法。如果设置_bytes版本，则_ratio版本将变为0，反之亦然。
4. vm.dirty_expire_centisecs = 3000 指定脏数据能存活的时间。在这里它的值是30秒。当pdflush/flush/kdmflush在运行时，他们会检查是否有数据超过这个时限，如果有则会把它异步地写到磁盘中。毕竟数据在内存里待太久也会有丢失风险。（单位：厘秒，即：1/100s，所以默认3000为3000*1/3000秒即30秒）
5. vm.dirty_writeback_centisecs = 500 指定多长时间pdflush/flush/kdmflush这些进程会唤醒一次，然后检查是否有缓存需要清理。（单位：厘秒，即：1/100s，所以默认500为500*1/100秒即5秒）

pdflush写入硬盘由vm.dirty_background_ratio和vm.dirty_expire_centisecs一起来作用，一个表示大小比例，一个表示时间；即满足其中任何一个的条件都达到刷盘的条件。如果只有参数 vm.dirty_background_ratio ，也就是说cache中的数据需要超过这个阀值才会满足刷磁盘的条件；如果数据一直没有达到这个阀值，那相当于cache中的数据就永远无法持久化到磁盘，这种情况下，一旦服务器重启，那么cache中的数据必然丢失。结合以上情况，所以添加了一个数据过期时间参数。当数据量没有达到阀值，但是达到了我们设定的过期时间，同样可以实现数据刷盘。这样可以有效的解决上述存在的问题，其实这种设计在绝大部分框架中都有。

实验环境准备：

临时修改内核配置项目

sysctl -w vm.dirty_background_ratio=30

sysctl -w vm.dirty_writeback_centisecs=50000

sysctl -w vm.dirty_expire_centisecs=30000

永久修改内核配置项目

vim /etc/sysctl.conf

sysctl -w vm.dirty_background_ratio = 30

sysctl -w vm.dirty_writeback_centisecs = 50000

sysctl -w vm.dirty_expire_centisecs = 30000

sysctl -p /etc/sysctl.conf

查看内存中有多少脏数据

cat /proc/vmstat | egrep "dirty|writeback"

实验1：

- 启动一个/bin/bash，执行start.sh

```bash
[root@server03 io]# ./start.sh 1
```

start.sh

strace命令会追踪内核调用

```shell
#! /bin/bash

rm -fr *out*
javac OSFileIO.java
strace -ff -o out java OSFileIO $1
```

OSFileIO.java

```java
import java.io.BufferedOutputStream;
import java.io.File;
import java.io.FileOutputStream;
import java.nio.charset.StandardCharsets;

public class OSFileIO {

    static byte[] data = "123456789\n".getBytes(StandardCharsets.UTF_8);
    static String path = "/opt/io/out.txt";

    public static void main(String[] args) throws Exception {

        switch (args[0]) {
            case "0":
                testBasicFileIO();
                break;
            case "1":
                testBufferedFileIO();
                break;
            default:
        }
    }

    public static void testBasicFileIO() throws Exception {
        File file = new File(path);
        FileOutputStream out = new FileOutputStream(file);
        while (true) {
            out.write(data);
        }
    }

    public static void testBufferedFileIO() throws Exception {
        File file = new File(path);
        BufferedOutputStream out = new BufferedOutputStream(new FileOutputStream(file));
        while (true) {
            out.write(data);
        }
    }
}
```

- 启动命令：ll -h && pcstat out.txt观察缓存情况，

```bash

[root@server03 io]# ll -h && pcstat out.txt
total 4.0G
-rw-r--r--. 1 root root 1.4K Jan 21 09:23 OSFileIO.class
-rw-r--r--. 1 root root 1.1K Jan 21 08:55 OSFileIO.java
-rw-r--r--. 1 root root 9.5K Jan 21 09:23 out.1348
-rw-r--r--. 1 root root  18M Jan 21 09:23 out.1349
-rw-r--r--. 1 root root 4.2K Jan 21 09:23 out.1350
-rw-r--r--. 1 root root  930 Jan 21 09:23 out.1351
-rw-r--r--. 1 root root 1.1K Jan 21 09:23 out.1352
-rw-r--r--. 1 root root  974 Jan 21 09:23 out.1353
-rw-r--r--. 1 root root 9.1K Jan 21 09:23 out.1354
-rw-r--r--. 1 root root 6.2K Jan 21 09:23 out.1355
-rw-r--r--. 1 root root  930 Jan 21 09:23 out.1356
-rw-r--r--. 1 root root  54K Jan 21 09:23 out.1357
-rw-r--r--. 1 root root 2.2G Jan 21 09:23 out.txt
-rwxr-xr-x. 1 root root   82 Jan 20 10:39 start.sh
|----------+----------------+------------+-----------+---------|
| Name     | Size           | Pages      | Cached    | Percent |
|----------+----------------+------------+-----------+---------|
| out.txt  | 2358408780     | 575784     | 575784    | 100.000 |
|----------+----------------+------------+-----------+---------|
```

- 在页缓存还没有超出dirty_expire_centisecs、并且脏页缓存还没有达到工作内存的dirty_background_ratio的时候，强行断电关机

- 重新启动后观察out.txt，发现文件的大小为0，所有数据全部丢失

```bash

[root@server03 io]# ll -h
total 12K
-rw-r--r--. 1 root root 1.4K Jan 21 09:23 OSFileIO.class
-rw-r--r--. 1 root root 1.1K Jan 21 08:55 OSFileIO.java
-rw-r--r--. 1 root root    0 Jan 21 09:23 out.1348
-rw-r--r--. 1 root root    0 Jan 21 09:23 out.1349
-rw-r--r--. 1 root root    0 Jan 21 09:23 out.1350
-rw-r--r--. 1 root root    0 Jan 21 09:23 out.1351
-rw-r--r--. 1 root root    0 Jan 21 09:23 out.1352
-rw-r--r--. 1 root root    0 Jan 21 09:23 out.1353
-rw-r--r--. 1 root root    0 Jan 21 09:23 out.1354
-rw-r--r--. 1 root root    0 Jan 21 09:23 out.1355
-rw-r--r--. 1 root root    0 Jan 21 09:23 out.1356
-rw-r--r--. 1 root root    0 Jan 21 09:23 out.1357
-rw-r--r--. 1 root root    0 Jan 21 09:23 out.txt
-rwxr-xr-x. 1 root root   82 Jan 20 10:39 start.sh
```

实验2：

- 内核参数：

```basn
[root@localhost io]# sysctl -a | grep dirty
sysctl: reading key "net.ipv6.conf.all.stable_secret"
sysctl: reading key "net.ipv6.conf.default.stable_secret"
sysctl: reading key "net.ipv6.conf.enp0s3.stable_secret"
sysctl: reading key "net.ipv6.conf.enp0s8.stable_secret"
sysctl: reading key "net.ipv6.conf.lo.stable_secret"
vm.dirty_background_bytes = 0
vm.dirty_background_ratio = 10
vm.dirty_bytes = 0
vm.dirty_expire_centisecs = 3000
vm.dirty_ratio = 30
vm.dirty_writeback_centisecs = 500
```

- OSFileIO.java 

由于代码值输出100万次"123456789\n"后就进入阻塞状态，所以脏页的数量肯定不满足vm.dirty_background_ratio设定的值

```java
import java.io.BufferedOutputStream;
import java.io.File;
import java.io.FileOutputStream;
import java.nio.charset.StandardCharsets;

public class OSFileIO {


    static byte[] data = "123456789\n".getBytes(StandardCharsets.UTF_8);
    static String path = "/opt/io/out.txt";

    public static void main(String[] args) throws Exception {

        switch (args[0]) {
            case "0":
                testBasicFileIO();
                break;
            case "1":
                testBufferedFileIO();
                break;
            default:
        }
    }

    public static void testBasicFileIO() throws Exception {
        File file = new File(path);
        FileOutputStream out = new FileOutputStream(file);
//        while (true) {
//            out.write(data);
//        }
        for (int i = 0; i < 1000000; i++) {
            out.write(data);
        }
        System.in.read();
    }

    public static void testBufferedFileIO() throws Exception {
        File file = new File(path);
        BufferedOutputStream out = new BufferedOutputStream(new FileOutputStream(file));
//        while (true) {
//            out.write(data);
//        }
        for (int i = 0; i < 1000000; i++) {
            out.write(data);
        }
        System.in.read();
    }
}
```

- 启动start.sh  ./start.sh 1

```shell
#!/bin/bash

rm -fr *out*
javac OSFileIO.java
strace -ff -o out java OSFileIO $1
```

- 新开一个bash窗口使用ll -laht && cat /proc/vmstat | egrep "dirty|writeback"观察脏页情况，在vm.dirty_expire_centisecs设定的时间后(30S)，强行关机

```bash
[root@localhost io]# ll -laht && cat /proc/vmstat | egrep "dirty|writeback"
total 17M
-rw-r--r--. 1 root root 118K Jan 23 14:32 out.1393
-rw-r--r--. 1 root root 7.4K Jan 23 14:32 out.1386
-rw-r--r--. 1 root root 8.7K Jan 23 14:32 out.1390
-rw-r--r--. 1 root root 6.4K Jan 23 14:32 out.1391
-rw-r--r--. 1 root root 239K Jan 23 14:31 out.1385
-rw-r--r--. 1 root root 9.6M Jan 23 14:31 out.txt
drwxr-xr-x. 2 root root  240 Jan 23 14:31 .
-rw-r--r--. 1 root root  930 Jan 23 14:31 out.1392
-rw-r--r--. 1 root root  974 Jan 23 14:31 out.1389
-rw-r--r--. 1 root root 1.1K Jan 23 14:31 out.1388
-rw-r--r--. 1 root root  930 Jan 23 14:31 out.1387
-rw-r--r--. 1 root root 9.4K Jan 23 14:31 out.1384
-rw-r--r--. 1 root root 1.6K Jan 23 14:31 OSFileIO.class
-rw-r--r--. 1 root root 1.4K Jan 23 14:31 OSFileIO.java
-rwxr--r--. 1 root root   81 Jan 23 13:15 start.sh
drwxr-xr-x. 3 root root   16 Jan 23 13:06 ..
nr_dirty 4
nr_writeback 0
nr_writeback_temp 0
nr_dirty_threshold 261710
nr_dirty_background_threshold 87236
```

- 重新启动服务器，进入到/opt/io目录下，发现数据已经被刷入磁盘了，这是因为虽然不满足vm.dirty_background_ratio设定的阈值，但是达到了设定的过期时间vm.dirty_expire_centisecs，同样可以实现数据刷盘。

```bash

[root@localhost ~]# ll /opt/io/
total 17284
-rw-r--r--. 1 root root    1570 Jan 23 14:31 OSFileIO.class
-rw-r--r--. 1 root root    1425 Jan 23 14:31 OSFileIO.java
-rw-r--r--. 1 root root    9566 Jan 23 14:31 out.1384
-rw-r--r--. 1 root root  244291 Jan 23 14:31 out.1385
-rw-r--r--. 1 root root    7374 Jan 23 14:32 out.1386
-rw-r--r--. 1 root root     930 Jan 23 14:31 out.1387
-rw-r--r--. 1 root root    1054 Jan 23 14:31 out.1388
-rw-r--r--. 1 root root     974 Jan 23 14:31 out.1389
-rw-r--r--. 1 root root    8858 Jan 23 14:32 out.1390
-rw-r--r--. 1 root root    6462 Jan 23 14:32 out.1391
-rw-r--r--. 1 root root     930 Jan 23 14:31 out.1392
-rw-r--r--. 1 root root  226939 Jan 23 14:32 out.1393
-rw-r--r--. 1 root root 9999990 Jan 23 14:31 out.txt
-rwxr--r--. 1 root root      81 Jan 23 13:15 start.sh
```



# Socket编程BIO及TCP参数

实验1：

服务端代码:

```java
package org.duo.bio;

import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStream;
import java.io.InputStreamReader;
import java.net.InetSocketAddress;
import java.net.ServerSocket;
import java.net.Socket;
import java.net.StandardSocketOptions;
import java.util.Arrays;

public class SocketIOPropertites {

    private static final int RECEIVE_BUFFER = 10;
    private static final int SO_TIMEOUT = 0;
    private static final boolean REUSE_ADDR = false;
    // 请求的连接数量非常大，资源不够时，允许排队的连接数
    private static final int BACK_LOG = 2;
    private static final boolean CLI_KEEPALIVE = false;
    // 是否优先发送数据进行试探
    private static final boolean CLI_OOB = false;
    // 通过ss -natp可以看到Recv-Q，Recv-Q和CLI_REC_BUF的关系？
    private static final int CLI_REC_BUF = 20;
    private static final boolean CLI_REUSE_ADDR = false;
    // 通过ss -natp可以看到Send-Q，Send-Q和CLI_SEND_BUF的关系？
    private static final int CLI_SEND_BUF = 20;
    private static final boolean CLI_LINGER = true;
    private static final int CLI_LINGER_N = 0;
    // 客户端读取数据的超时时间，0：表示阻塞等待
    private static final int CLI_TIMEOUT = 0;
    private static final boolean CLI_NO_DELAY = false;

//    StandardSocketOptions.TCP_NODELAY

    public static void main(String[] args) {
        ServerSocket server = null;
        try {
            server = new ServerSocket();
            server.bind(new InetSocketAddress(9090), BACK_LOG);
            server.setReceiveBufferSize(RECEIVE_BUFFER);
            server.setReuseAddress(REUSE_ADDR);
            server.setSoTimeout(SO_TIMEOUT);
        } catch (IOException e) {
            e.printStackTrace();
        }
        System.out.println("server up use 9090!");

        while (true) {
            try {

                System.in.read();

                Socket client = server.accept();
                System.out.println("client port:" + client.getPort());

                client.setKeepAlive(CLI_KEEPALIVE);
                client.setOOBInline(CLI_OOB);
                client.setReceiveBufferSize(CLI_REC_BUF);
                client.setReuseAddress(CLI_REUSE_ADDR);
                client.setSendBufferSize(CLI_SEND_BUF);
                client.setSoLinger(CLI_LINGER, CLI_LINGER_N);
                client.setSoTimeout(CLI_TIMEOUT);
                client.setTcpNoDelay(CLI_NO_DELAY);

                new Thread(() -> {
                    while (true) {
                        try {
                            InputStream in = client.getInputStream();
                            BufferedReader reader = new BufferedReader(new InputStreamReader(in));
                            char[] data = new char[1024];
                            int num = reader.read(data);

                            if (num > 0) {
                                System.out.println("client read some data is :" + num + " val :" + new String(data, 0, num));
                            } else if (num == 0) {
                                System.out.println("client read nothing!");
                                continue;
                            } else {
                                System.out.println("client read -1...");
                                client.close();
                                break;
                            }
                        } catch (IOException e) {
                            e.printStackTrace();
                        }
                    }
                }).start();
            } catch (Exception e) {
                e.printStackTrace();
            }

        }
    }
}

```

客服端代码：

```java
package org.duo.bio;

import java.io.*;
import java.net.Socket;

public class SocketClient {

    public static void main(String[] args) {

        try {
            Socket client = new Socket("192.168.56.112", 9090);

            client.setSendBufferSize(20);
            client.setTcpNoDelay(false);
            OutputStream out = client.getOutputStream();

            InputStream in = System.in;
            BufferedReader reader = new BufferedReader(new InputStreamReader(in));

            while (true) {
                String line = reader.readLine();
                if (line != null) {
                    byte[] bb = line.getBytes();
                    for (byte b : bb) {
                        out.write(b);
                    }
                }
            }
        } catch (IOException ioException) {
            ioException.printStackTrace();
        }

    }
}
```

运行服务端代码：

```bash
[root@server03 ~]# ss -natp
State      Recv-Q Send-Q                                       Local Address:Port                                                      Peer Address:Port
LISTEN     0      128                                                      *:111                                                                  *:*                   users:(("rpcbind",pid=657,fd=4),("systemd",pid=1,fd=39))
LISTEN     0      128                                                      *:22                                                                   *:*                   users:(("sshd",pid=992,fd=3))
LISTEN     0      100                                              127.0.0.1:25                                                                   *:*                   users:(("master",pid=1231,fd=13))
ESTAB      0      0                                           192.168.56.112:22                                                        192.168.56.1:51140               users:(("sshd",pid=1388,fd=3))
ESTAB      0      64                                          192.168.56.112:22                                                        192.168.56.1:63220               users:(("sshd",pid=1412,fd=3))
ESTAB      0      0                                           192.168.56.112:22                                                        192.168.56.1:51105               users:(("sshd",pid=1301,fd=3))
ESTAB      0      0                                           192.168.56.112:22                                                        192.168.56.1:50697               users:(("sshd",pid=1063,fd=3))
LISTEN     0      128                                                     :::111                                                                 :::*                   users:(("rpcbind",pid=657,fd=6),("systemd",pid=1,fd=41))
LISTEN     0      128                                                     :::22                                                                  :::*                   users:(("sshd",pid=992,fd=4))
LISTEN     0      100                                                    ::1:25                                                                  :::*                   users:(("master",pid=1231,fd=14))
LISTEN     0      2                                                       :::9090                                                                :::*                   users:(("java",pid=1377,fd=5))
[root@server03 ~]# jps
1440 Jps
1377 SocketIOPropertites
[root@server03 ~]# lsof -op 1377
COMMAND  PID USER   FD   TYPE             DEVICE     OFFSET      NODE NAME
java    1377 root  cwd    DIR              253,0            100781673 /opt/io
java    1377 root  rtd    DIR              253,0                   64 /
java    1377 root  txt    REG              253,0             53223730 /usr/local/java/jdk1.8.0_144/bin/java
java    1377 root  mem    REG              253,0                32642 /usr/lib/locale/locale-archive
java    1377 root  mem    REG              253,0              5309195 /usr/local/java/jdk1.8.0_144/jre/lib/amd64/libnet.so
java    1377 root  mem    REG              253,0             83904872 /usr/local/java/jdk1.8.0_144/jre/lib/rt.jar
java    1377 root  mem    REG              253,0              5665416 /usr/local/java/jdk1.8.0_144/jre/lib/amd64/libzip.so
java    1377 root  mem    REG              253,0              2114148 /usr/lib64/libnss_files-2.17.so
java    1377 root  mem    REG              253,0              5665449 /usr/local/java/jdk1.8.0_144/jre/lib/amd64/libjava.so
java    1377 root  mem    REG              253,0              5309196 /usr/local/java/jdk1.8.0_144/jre/lib/amd64/libverify.so
java    1377 root  mem    REG              253,0              2114160 /usr/lib64/librt-2.17.so
java    1377 root  mem    REG              253,0                32663 /usr/lib64/libm-2.17.so
java    1377 root  mem    REG              253,0             35290966 /usr/local/java/jdk1.8.0_144/jre/lib/amd64/server/libjvm.so
java    1377 root  mem    REG              253,0                32652 /usr/lib64/libc-2.17.so
java    1377 root  mem    REG              253,0                32659 /usr/lib64/libdl-2.17.so
java    1377 root  mem    REG              253,0              5302306 /usr/local/java/jdk1.8.0_144/lib/amd64/jli/libjli.so
java    1377 root  mem    REG              253,0              2114156 /usr/lib64/libpthread-2.17.so
java    1377 root  mem    REG              253,0              2114136 /usr/lib64/ld-2.17.so
java    1377 root  mem    REG              253,0              1437184 /tmp/hsperfdata_root/1377
java    1377 root    0u   CHR              136,0        0t0         3 /dev/pts/0
java    1377 root    1u   CHR              136,0        0t0         3 /dev/pts/0
java    1377 root    2u   CHR              136,0        0t0         3 /dev/pts/0
java    1377 root    3r   REG              253,0 0t62392257  83904872 /usr/local/java/jdk1.8.0_144/jre/lib/rt.jar
java    1377 root    4u  unix 0xffff88020f703400        0t0     18888 socket
java    1377 root    5u  IPv6              18890        0t0       TCP *:websm (LISTEN)
[root@server03 ~]# tcpdump -nn -i enp0s8 port 9090
tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
listening on enp0s8, link-type EN10MB (Ethernet), capture size 262144 bytes
```

- 运行客户端代码，向服务器建立连接

```bash
[root@server01 io]# java org.duo.bio.SocketClient

```

- 在服务端运行tcpdump命令抓取客户端与服务器之间传输的数据包发现，客户端和服务器已经通过三次握手建立连接，并且通过ss命令可以看到，在内核状态中由客户端：192.168.56.110:54882发起的请求已经和服务器：192.168.56.112:9090建立好了连接，（状态是：ESTAB）但是这个socket没有分配给具体的进程去使用（即：没有被接受）。

```bash
[root@server03 ~]# tcpdump -nn -i enp0s8 port 9090
tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
listening on enp0s8, link-type EN10MB (Ethernet), capture size 262144 bytes
19:12:17.148445 IP 192.168.56.110.54882 > 192.168.56.112.9090: Flags [S], seq 1580823001, win 29200, options [mss 1460,sackOK,TS val 2993185 ecr 0,nop,wscale 7], length 0
19:12:17.148465 IP 192.168.56.112.9090 > 192.168.56.110.54882: Flags [S.], seq 188039735, ack 1580823002, win 1152, options [mss 1460,sackOK,TS val 3177786 ecr 2993185,nop,wscale 0], length 0
19:12:17.148681 IP 192.168.56.110.54882 > 192.168.56.112.9090: Flags [.], ack 1, win 229, options [nop,nop,TS val 2993186 ecr 3177786], length 0
[root@server03 ~]# ss -natp
State      Recv-Q Send-Q                                       Local Address:Port                                                      Peer Address:Port
LISTEN     0      128                                                      *:111                                                                  *:*                   users:(("rpcbind",pid=657,fd=4),("systemd",pid=1,fd=39))
LISTEN     0      128                                                      *:22                                                                   *:*                   users:(("sshd",pid=992,fd=3))
LISTEN     0      100                                              127.0.0.1:25                                                                   *:*                   users:(("master",pid=1231,fd=13))
ESTAB      0      0                                           192.168.56.112:22                                                        192.168.56.1:51140               users:(("sshd",pid=1388,fd=3))
ESTAB      0      64                                          192.168.56.112:22                                                        192.168.56.1:63220               users:(("sshd",pid=1412,fd=3))
ESTAB      0      0                                           192.168.56.112:22                                                        192.168.56.1:51105               users:(("sshd",pid=1301,fd=3))
ESTAB      0      0                                           192.168.56.112:22                                                        192.168.56.1:50697               users:(("sshd",pid=1063,fd=3))
LISTEN     0      128                                                     :::111                                                                 :::*                   users:(("rpcbind",pid=657,fd=6),("systemd",pid=1,fd=41))
LISTEN     0      128                                                     :::22                                                                  :::*                   users:(("sshd",pid=992,fd=4))
LISTEN     0      100                                                    ::1:25                                                                  :::*                   users:(("master",pid=1231,fd=14))
LISTEN     1      2                                                       :::9090                                                                :::*                   users:(("java",pid=1377,fd=5))
ESTAB      0      0                                    ::ffff:192.168.56.112:9090                                             ::ffff:192.168.56.110:54882
```

- 客户端向服务器发送数据包：

```bash
[root@server01 io]# java org.duo.bio.SocketClient
123131

```

通过tcpdump发现数据通过tcp传输到了服务器，并且通过ss命令可以看到该连接对应的Recv-Q为6，说明这6个字节已经被内核收到了。这就说明就算socket没有被具体的进程接受、没有对应的文件描述符，但是其实在内核中已经完成了数据的传输。这就是常说的tcp是面向连接的，在完成三次握手，双方开辟完资源后（在内核中开辟了缓冲区等），就算应用程序没有接受但是在内核中也会有资源去完成等待或者接受。

通过lsof -op PID发现由于服务端阻塞在System.in.read();还没有调用server.accept();所以还没有分配文件描述符

```bash
[root@server03 ~]# tcpdump -nn -i enp0s8 port 9090
tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
listening on enp0s8, link-type EN10MB (Ethernet), capture size 262144 bytes
19:12:17.148445 IP 192.168.56.110.54882 > 192.168.56.112.9090: Flags [S], seq 1580823001, win 29200, options [mss 1460,sackOK,TS val 2993185 ecr 0,nop,wscale 7], length 0
19:12:17.148465 IP 192.168.56.112.9090 > 192.168.56.110.54882: Flags [S.], seq 188039735, ack 1580823002, win 1152, options [mss 1460,sackOK,TS val 3177786 ecr 2993185,nop,wscale 0], length 0
19:12:17.148681 IP 192.168.56.110.54882 > 192.168.56.112.9090: Flags [.], ack 1, win 229, options [nop,nop,TS val 2993186 ecr 3177786], length 0
19:25:37.694947 IP 192.168.56.110.54882 > 192.168.56.112.9090: Flags [P.], seq 1:2, ack 1, win 229, options [nop,nop,TS val 3793738 ecr 3177786], length 1
19:25:37.694974 IP 192.168.56.112.9090 > 192.168.56.110.54882: Flags [.], ack 2, win 1151, options [nop,nop,TS val 3978333 ecr 3793738], length 0
19:25:37.695194 IP 192.168.56.110.54882 > 192.168.56.112.9090: Flags [P.], seq 2:7, ack 1, win 229, options [nop,nop,TS val 3793739 ecr 3978333], length 5
19:25:37.734871 IP 192.168.56.112.9090 > 192.168.56.110.54882: Flags [.], ack 7, win 1146, options [nop,nop,TS val 3978373 ecr 3793739], length 0
[root@server03 ~]# ss -natp
State      Recv-Q Send-Q                                       Local Address:Port                                                      Peer Address:Port
LISTEN     0      128                                                      *:111                                                                  *:*                   users:(("rpcbind",pid=657,fd=4),("systemd",pid=1,fd=39))
LISTEN     0      128                                                      *:22                                                                   *:*                   users:(("sshd",pid=992,fd=3))
LISTEN     0      100                                              127.0.0.1:25                                                                   *:*                   users:(("master",pid=1231,fd=13))
ESTAB      0      0                                           192.168.56.112:22                                                        192.168.56.1:51140               users:(("sshd",pid=1388,fd=3))
ESTAB      0      64                                          192.168.56.112:22                                                        192.168.56.1:63220               users:(("sshd",pid=1412,fd=3))
ESTAB      0      0                                           192.168.56.112:22                                                        192.168.56.1:51105               users:(("sshd",pid=1301,fd=3))
ESTAB      0      0                                           192.168.56.112:22                                                        192.168.56.1:50697               users:(("sshd",pid=1063,fd=3))
LISTEN     0      128                                                     :::111                                                                 :::*                   users:(("rpcbind",pid=657,fd=6),("systemd",pid=1,fd=41))
LISTEN     0      128                                                     :::22                                                                  :::*                   users:(("sshd",pid=992,fd=4))
LISTEN     0      100                                                    ::1:25                                                                  :::*                   users:(("master",pid=1231,fd=14))
LISTEN     1      2                                                       :::9090                                                                :::*                   users:(("java",pid=1377,fd=5))
ESTAB      6      0                                    ::ffff:192.168.56.112:9090                                             ::ffff:192.168.56.110:54882
[root@server03 ~]# lsof -op 1377
COMMAND  PID USER   FD   TYPE             DEVICE     OFFSET      NODE NAME
java    1377 root  cwd    DIR              253,0            100781673 /opt/io
java    1377 root  rtd    DIR              253,0                   64 /
java    1377 root  txt    REG              253,0             53223730 /usr/local/java/jdk1.8.0_144/bin/java
java    1377 root  mem    REG              253,0                32642 /usr/lib/locale/locale-archive
java    1377 root  mem    REG              253,0              5309195 /usr/local/java/jdk1.8.0_144/jre/lib/amd64/libnet.so
java    1377 root  mem    REG              253,0             83904872 /usr/local/java/jdk1.8.0_144/jre/lib/rt.jar
java    1377 root  mem    REG              253,0              5665416 /usr/local/java/jdk1.8.0_144/jre/lib/amd64/libzip.so
java    1377 root  mem    REG              253,0              2114148 /usr/lib64/libnss_files-2.17.so
java    1377 root  mem    REG              253,0              5665449 /usr/local/java/jdk1.8.0_144/jre/lib/amd64/libjava.so
java    1377 root  mem    REG              253,0              5309196 /usr/local/java/jdk1.8.0_144/jre/lib/amd64/libverify.so
java    1377 root  mem    REG              253,0              2114160 /usr/lib64/librt-2.17.so
java    1377 root  mem    REG              253,0                32663 /usr/lib64/libm-2.17.so
java    1377 root  mem    REG              253,0             35290966 /usr/local/java/jdk1.8.0_144/jre/lib/amd64/server/libjvm.so
java    1377 root  mem    REG              253,0                32652 /usr/lib64/libc-2.17.so
java    1377 root  mem    REG              253,0                32659 /usr/lib64/libdl-2.17.so
java    1377 root  mem    REG              253,0              5302306 /usr/local/java/jdk1.8.0_144/lib/amd64/jli/libjli.so
java    1377 root  mem    REG              253,0              2114156 /usr/lib64/libpthread-2.17.so
java    1377 root  mem    REG              253,0              2114136 /usr/lib64/ld-2.17.so
java    1377 root  mem    REG              253,0              1437184 /tmp/hsperfdata_root/1377
java    1377 root    0u   CHR              136,0        0t0         3 /dev/pts/0
java    1377 root    1u   CHR              136,0        0t0         3 /dev/pts/0
java    1377 root    2u   CHR              136,0        0t0         3 /dev/pts/0
java    1377 root    3r   REG              253,0 0t62392257  83904872 /usr/local/java/jdk1.8.0_144/jre/lib/rt.jar
java    1377 root    4u  unix 0xffff88020f703400        0t0     18888 socket
java    1377 root    5u  IPv6              18890        0t0       TCP *:websm (LISTEN)
```

- 在服务端输入回车后，由于调用了server.accept();就会分配一个文件描述符，然后客户端发送的被缓冲在内核的数据也就能通过分配给文件描述符读取到了。通过 ss -natp也可以看到之前的socket也被分配给了进程：1377；且通过lsof也可以看到进程：1377中新增了一个文件描述符：6

```bash
[root@server03 io]# java org.duo.bio.SocketIOPropertites
server up use 9090!

client port:54882
client read some data is :6 val :123131
[root@server03 ~]# ss -natp
State      Recv-Q Send-Q                                       Local Address:Port                                                      Peer Address:Port
LISTEN     0      128                                                      *:111                                                                  *:*                   users:(("rpcbind",pid=657,fd=4),("systemd",pid=1,fd=39))
LISTEN     0      128                                                      *:22                                                                   *:*                   users:(("sshd",pid=992,fd=3))
LISTEN     0      100                                              127.0.0.1:25                                                                   *:*                   users:(("master",pid=1231,fd=13))
ESTAB      0      0                                           192.168.56.112:22                                                        192.168.56.1:51140               users:(("sshd",pid=1388,fd=3))
ESTAB      0      64                                          192.168.56.112:22                                                        192.168.56.1:63220               users:(("sshd",pid=1412,fd=3))
ESTAB      0      0                                           192.168.56.112:22                                                        192.168.56.1:51105               users:(("sshd",pid=1301,fd=3))
ESTAB      0      0                                           192.168.56.112:22                                                        192.168.56.1:50697               users:(("sshd",pid=1063,fd=3))
LISTEN     0      128                                                     :::111                                                                 :::*                   users:(("rpcbind",pid=657,fd=6),("systemd",pid=1,fd=41))
LISTEN     0      128                                                     :::22                                                                  :::*                   users:(("sshd",pid=992,fd=4))
LISTEN     0      100                                                    ::1:25                                                                  :::*                   users:(("master",pid=1231,fd=14))
LISTEN     0      2                                                       :::9090                                                                :::*                   users:(("java",pid=1377,fd=5))
ESTAB      0      0                                    ::ffff:192.168.56.112:9090                                             ::ffff:192.168.56.110:54882               users:(("java",pid=1377,fd=6))
[root@server03 ~]# lsof -op 1377
COMMAND  PID USER   FD   TYPE             DEVICE     OFFSET      NODE NAME
java    1377 root  cwd    DIR              253,0            100781673 /opt/io
java    1377 root  rtd    DIR              253,0                   64 /
java    1377 root  txt    REG              253,0             53223730 /usr/local/java/jdk1.8.0_144/bin/java
java    1377 root  mem    REG              253,0                32642 /usr/lib/locale/locale-archive
java    1377 root  mem    REG              253,0              5309195 /usr/local/java/jdk1.8.0_144/jre/lib/amd64/libnet.so
java    1377 root  mem    REG              253,0             83904872 /usr/local/java/jdk1.8.0_144/jre/lib/rt.jar
java    1377 root  mem    REG              253,0              5665416 /usr/local/java/jdk1.8.0_144/jre/lib/amd64/libzip.so
java    1377 root  mem    REG              253,0              2114148 /usr/lib64/libnss_files-2.17.so
java    1377 root  mem    REG              253,0              5665449 /usr/local/java/jdk1.8.0_144/jre/lib/amd64/libjava.so
java    1377 root  mem    REG              253,0              5309196 /usr/local/java/jdk1.8.0_144/jre/lib/amd64/libverify.so
java    1377 root  mem    REG              253,0              2114160 /usr/lib64/librt-2.17.so
java    1377 root  mem    REG              253,0                32663 /usr/lib64/libm-2.17.so
java    1377 root  mem    REG              253,0             35290966 /usr/local/java/jdk1.8.0_144/jre/lib/amd64/server/libjvm.so
java    1377 root  mem    REG              253,0                32652 /usr/lib64/libc-2.17.so
java    1377 root  mem    REG              253,0                32659 /usr/lib64/libdl-2.17.so
java    1377 root  mem    REG              253,0              5302306 /usr/local/java/jdk1.8.0_144/lib/amd64/jli/libjli.so
java    1377 root  mem    REG              253,0              2114156 /usr/lib64/libpthread-2.17.so
java    1377 root  mem    REG              253,0              2114136 /usr/lib64/ld-2.17.so
java    1377 root  mem    REG              253,0              1437184 /tmp/hsperfdata_root/1377
java    1377 root    0u   CHR              136,0        0t0         3 /dev/pts/0
java    1377 root    1u   CHR              136,0        0t0         3 /dev/pts/0
java    1377 root    2u   CHR              136,0        0t0         3 /dev/pts/0
java    1377 root    3r   REG              253,0 0t30056619  83904872 /usr/local/java/jdk1.8.0_144/jre/lib/rt.jar
java    1377 root    4u  unix 0xffff88020f703400        0t0     18888 socket
java    1377 root    5u  IPv6              18890        0t0       TCP *:websm (LISTEN)
java    1377 root    6u  IPv6              20028        0t0       TCP server03:websm->server01:54882 (ESTABLISHED)
[root@server03 ~]#


```

## 总结

tcp是面向连接的可靠的传输协议，（注意这里的连接不是物理的，而是需要通过三次握手的）在三次握手完成后会在内核级开辟资源

socket是一个四元组：客户端IP+客户端PORT:服务器IP+服务器PORT

java进程在执行accept之后，可以拿到内核分配的文件描述符，然后在java中将得到的文件描述符包装成了socket对象，文件描述符在每个进程内都是唯一的。文件描述符可以理解为指向内核中的四元组。

## Recv-Q和Send-Q

当服务器启动的时候，会开启一个listen状态下的socket，用来接受客户端请求；当服务器接受到客户端的请求后会新建一个established状态的socket用于客户端和服务器之间的通信。而通过ss命令看到的Recv-Q和Send-Q在这两种类型的socket下含义是不相同的。

### 在LISTEN状态

在LISTEN状态：即服务器启动的ServerSocket；

1. 当client通过connect向server发出SYN包时，client会维护一个socket等待队列，而server会维护一个SYN队列；
2. 此时进入半链接的状态，如果socket等待队列满了，server则会丢弃，而client也会由此返回connection timeout；只要是client没有收到SYN+ACK，3s之后，client会再次发送，如果依然没有收到，9s之后会继续发送；
3. 半连接syn队列的长度由max(64,/proc/sys/net/ipv4/tcp_max_syn_backlog)决定；
4. 当server收到client的SYN包后，会返回SYN,ACK的包加以确认，client的TCP协议栈会唤醒socket等待队列，发出connect调用；
5. client返回ACK的包后，表示三次握手完成连接已经建立；server会进入一个新的叫accept的队列(即：Recv-Q)，该队列的长度为min(backlog,/proc/sys/net/core/somaxconn)，默认情况下，somaxconn的值为128，表示最多会有129个(即：队列中的128个+已经处于等待状态的1个)ESTAB状态的连接等待accept()，而backlog的值在java中可以通过ServerSocket的构造函数指定ServerSocket(int port, int backlog)；
6. 当accept队列(即：Recv-Q)满了之后，即使client继续向server发送ACK的包，也会不被响应，此时，server通过/proc/sys/net/ipv4/tcp_abort_on_overflow来决定如何返回，0表示直接丢丢弃该ACK，1表示发送RST通知client；相应的，client则会分别返回read timeout或者connection reset by peer；
7. Recv-Q表示的是进入accept队列的连接的个数；而Send-Q表示的是accept的队列的长度。

### 非LISTEN状态

非LISTEN状态：即ServerSocket在accpet之后得到的socket；

1. Recv-Q表示receive queue中的bytes数量；大小由/proc/sys/net/ipv4/tcp_rmem 决定
   - 该文件包含3个整数值，分别是：min，default，max 
   - Min：为TCP socket预留用于接收缓冲的内存数量，即使在内存出现紧张情况下TCP socket都至少会有这么多数量的内存用于接收缓冲。 
   - Default：为TCP socket预留用于接收缓冲的内存数量，默认情况下该值影响其它协议使用的 net.core.wmem中default的 值。该值决定了在tcp_adv_win_scale、tcp_app_win和tcp_app_win的默认值情况下，TCP 窗口大小为65535。 
   - Max：为TCP socket预留用于接收缓冲的内存最大值。该值不会影响 net.core.wmem中max的值，今天选择参数 SO_SNDBUF则不受该值影响。 
   - 缺省设置：4096 87380 6291456
2. Send-Q表示send queue中的bytes数量；大小由/proc/sys/net/ipv4/tcp_wmem 
   - 该文件包含3个整数值，分别是：min，default，max 
   - Min：为TCP socket预留用于发送缓冲的内存最小值。每个TCP socket都可以使用它。
   - Default：为TCP socket预留用于发送缓冲的内存数量，默认情况下该值会影响其它协议使用的net.core.wmem中default的 值，一般要低于net.core.wmem中default的值。 
   - Max：为TCP socket预留用于发送缓冲的内存最大值。该值不会影响net.core.wmem_max，今天选择参数SO_SNDBUF则不受该值影响。默认值为128K。 
   - 缺省设置：4096 16384 4194304

# 多路复用器

## NIO

NIO模型下的java代码：

```java
package org.duo.nio;

import java.io.IOException;
import java.net.InetSocketAddress;
import java.net.StandardSocketOptions;
import java.nio.ByteBuffer;
import java.nio.channels.ServerSocketChannel;
import java.nio.channels.SocketChannel;
import java.util.Arrays;
import java.util.LinkedList;

public class ServerSocketIO {

    public static void main(String[] args) throws IOException, InterruptedException {

        LinkedList<SocketChannel> clients = new LinkedList<>();

        ServerSocketChannel ss = ServerSocketChannel.open(); // listen socket
        ss.bind(new InetSocketAddress(9090));
        ss.configureBlocking(false); // 重点 OS NONBLOCKING

//        ss.setOption(StandardSocketOptions.TCP_NODELAY, false);

        while (true) {
//            Thread.sleep(1000);
            SocketChannel client = ss.accept();//连接socket
            if (client == null) {
//                System.out.println("null......");
            } else {
                client.configureBlocking(false); // 重点
                int port = client.socket().getPort();
                System.out.println("client port = " + port);
                clients.add(client);
            }

            ByteBuffer buffer = ByteBuffer.allocateDirect(4096);

            for (SocketChannel c : clients) {
                int num = c.read(buffer); //非阻塞
                if (num > 0) {
                    buffer.flip();
                    byte[] aaa = new byte[buffer.limit()];
                    buffer.get(aaa);
                    String b = new String(aaa);
                    System.out.println(c.socket().getPort() + " : " + b);
                    buffer.clear();
                }
            }
        }
    }
}
```

## select 

synchronous I/O multiplexing

int select(int nfds, fd_set *readfds, fd_set *writefds, fd_set *exceptfds, struct timeval *timeout);

描述：

select() and pselect() allow a program to monitor multiple file descriptors, waiting until one or more of the file descriptors become "ready" for some class of I/O operation (e.g., input possible). A file descriptor is considered ready if it is possible to perform the corresponding I/O operation (e.g., read(2)) without blocking.

参数：

An fd_set is a fixed size buffer.  Executing FD_CLR() or FD_SET() with a value of fd that is negative or is equal to or larger than FD_SETSIZE  will  result in undefined behavior.  Moreover, POSIX requires fd to be a valid file descriptor(fd_set 是一个固定大小的缓冲区。 当fd 为负数或等于或大于FD_SETSIZE 的值执行FD_CLR()或FD_SET()将导致未定义的行为。即：传入select的fd的数量不能超过FD_SETSIZE ，这就是select的限制)

注意在使用man查询内核函数的时候除了需要yum install man还需要yum install man-pages

## poll

wait for some event on a file descriptor

int poll(struct pollfd *fds, nfds_t nfds, int timeout);

描述：

poll() performs a similar task to select(2): it waits for one of a set of file descriptors to become ready to perform I/O.

小结：无论是NIO、SELECT、POLL都是要遍历所有的IO询问状态，不同的地方在于，NIO的时候要轮询所有的Channel然后依次调用read访问询问内核是否有数据达到，所以会发生n次系统调用；而SELECT会先通过一次系统调用(其实每次调用select也会遍历所有需要监听的fd，只是这个遍历是发生在内核中)拿到传入select中的所有Channel中有数据到达的m个Channel，然后再对这m个Channel进行读操作。假设有n个Channel，NIO的时间复杂度是O(n)而SELECT是O(1)+O(m)。

select、poll的弊端：

1. 每次都要重复传递fds
2. 每次内核被调用后，都要触发一次针对传入的fds的全量遍历的过程

## 中断

cpu中断的类型

1. 软中断：如果应用程序(即：app)想调用内核(即：kernel)的方法的话就会触发软中断，其实就是cpu从app中读到了int:80指令(CPU指令集中的一个)，然后再根据中断类型号从中断向量表(即中断服务程序入口地址表)中获取中断向量(存放中断服务程序的首地址称为中断向量)，最后将程序流程转向中断服务程序的入口地址(这个入口地址可能就是kernel的select，read函数在内存中的地址，可以理解为callback函数，这里就完成了一次syscall即：用户态、内核态的切换)。由于中断向量表是由是由操作系统在初始化阶段来填写，可以在操作系统层面灵活修改，因此，不同的系统的中断向量表可能是不同的。

   ```mermaid
   graph LR
   CPU[cpu] -->|读取到程序中的一个指令:int 80| APP[app]
   CPU[cpu] -->|中断类型号| TABLE[中断向量表]
   TABLE[中断向量表] -->|中断向量| CPU[cpu]
   CPU[cpu] -->|调用| KERNEL[kernel]
   ```

2. 硬中断：时钟中断，即在cpu中有一个晶振(晶体振荡器)，可能每秒钟会完成1万次有规律的振荡，每一次振荡就会产生一次中断，就会造成一次进程的切换(即多个进程轮流使用CPU，在一个时间片后切换到下一个进程。并且同样会根据中断类型号从中断向量表中获取中断向量，转向中断服务程序的入口地址：这里可能包括保护现场等一系列操作)。这就是单核的CPU也能同时运行多个程序的原因。

3. IO中断：鼠标、键盘、网卡。比如网卡中有数据到达之后就会触发中断；鼠标在桌面滑动的时候也会触发中断。

   网卡触发中断的级别：

   1. package：有数据到达就产生中断
   2. buffer：网卡缓冲区满了后就触发中断
   3. 轮询：数据太快，频繁触发中断造成不必要的资源浪费，所以会关闭中断然后周期性的读取数据

总结：

有中断就会有回调函数(即：中断服务程序)，在epoll之前的callback(即：nio、select、poll)只是将网卡发来的数据，执行内核的网络协议栈将数据放入FD的buffer中(这些都是在内核中完成)，所以某一时间从app(用户态)询问内核某一个或者多个FD是否有R/W事件的时候，会有状态返回。如果内核在callback处理中再加入将有事件到来的文件描述符放到另外一个集合中，那么用户再调用多轮复用器的时候内核就不需要再遍历了而是直接返回有事件的文件描述符的集合。

## epoll

epoll - I/O event notification facility

描述：

The  epoll API performs a similar task to poll(2): monitoring multiple file descriptors to see if I/O is possible on any of them.  The epoll API can be used either as an edge-triggered or a level-triggered interface and scales well to large numbers of watched file descriptors.

epoll实际下会调用下面三个函数

### epoll_create

open an epoll file descriptor

描述：

On success, these system calls return a nonnegative file descriptor.

返回值：

On success, these system calls return a nonnegative file descriptor.  On error, -1 is returned, and errno is set to indicate the error.

在内核中开辟空间，管理文件描述符，这就避免了select的时候重复传递fds

### epoll_ctl

control interface for an epoll descriptor

描述：

This  system  call  performs control operations on the epoll(7) instance referred to by the file descriptor epfd.  It requests that the operation op be per formed for the target file descriptor, fd.

返回值：

When successful, epoll_ctl() returns zero.  When an error occurs, epoll_ctl() returns -1 and errno is set appropriately.

向epoll_create创建的文件描述符所指向的空间中增加、修改、删除文件描述符

### epoll_wait

wait for an I/O event on an epoll file descriptor

描述：

The  epoll_wait()  system call waits for events on the epoll(7) instance referred to by the file descriptor epfd.  The memory area pointed to by events will contain the events that will be available for the caller.  Up to maxevents are returned by epoll_wait().  The maxevents argument must be greater than zero.

返回值：

When successful, epoll_wait() returns the number of file descriptors ready for the requested I/O, or zero if no file  descriptor  became  ready  during  the requested timeout milliseconds.  When an error occurs, epoll_wait() returns -1 and errno is set appropriately.

在epoll的多路复用器实现方式中，中断的回调函数(中断向量)里面已经将有事件到达的文件描述符移到了一个集合中，所以在调用epoll_wait的时候就不需要再循环遍历了，可以直接拿到有事件到达的文件描述符的集合。

```mermaid
graph LR
listen[listen] -->|返回:fd4| epoll_create[epoll_create] -->|返回:fd6| epoll_ctl[eopll_ctl] -->|ADD fd4到fd6| epoll_wait[eopll_wait] -->|返回有事件到达的FD的集合| event_list[event_list]
```

### 实验1

使用poll模式启动server，并且在客户端关闭socket后，服务端不关闭socket，观察tcp状态

SocketMultiplexingSingleThread.java

```java
package org.duo.nio;

import java.io.IOException;
import java.net.InetSocketAddress;
import java.nio.ByteBuffer;
import java.nio.channels.SelectionKey;
import java.nio.channels.Selector;
import java.nio.channels.ServerSocketChannel;
import java.nio.channels.SocketChannel;
import java.util.Iterator;
import java.util.Set;

/**
 * 多路复用器监听的文件描述符的上限：
 *
 * /proc/sys/fs/epoll/max_user_watches (since Linux 2.6.28)
 * This  specifies  a limit on the total number of file descriptors that a user can register across all epoll instances on the system.  The limit is per
 *  real user ID.  Each registered file descriptor costs roughly 90 bytes on a 32-bit kernel, and roughly 160 bytes on a 64-bit kernel.   Currently,  the
 *  default value for max_user_watches is 1/25 (4%) of the available low memory, divided by the registration cost in bytes.
 *
 * 这指定了每个用户可以在系统上的所有epoll实例中注册的文件描述符总数的限制。在32位内核上大约需要90个字节，在64位内核上大约需要160个字节。
 * 目前，max_user_watches的默认值为可用内存的1/25(4%)除以注册成本（以字节为单位）
 */
public class SocketMultiplexingSingleThread {

    private ServerSocketChannel server = null;
    // 内核的多路复用器被java抽象成了Selector (select poll epoll)
    // 可以根据JVM启动参数设置该java进程是使用poll还是epoll：
    // -Djava.nio.channels.spi.SelectorProvider=sun.nio.ch.EPollSelectorProvider
    // -Djava.nio.channels.spi.SelectorProvider=sun.nio.ch.PollSelectorProvider
    private Selector selector = null;
    int port = 9090;

    public void initServer() {
        try {
            // 以下的三步会在内核中创建一个listen状态的FD4(server)；
            server = ServerSocketChannel.open();
            server.configureBlocking(false);
            server.bind(new InetSocketAddress(port));

            /*
            java会优先选择epoll，但是可以通过设置-D进行修正
            如果在epoll模型下，open相当于调用内核的epoll_create创建了一个新的文件描述符FD5(之后注册在该selector中的fd实际上是放在fd5中)
             */
            selector = Selector.open(); //
            /*
            如果使用的是select或者poll的模型：会在jvm里开辟一个数据将fd4放进去，
            如果使用的是epoll模型：那么相当于调用内核的epoll_ctl(fd5,ADD,fd4,EPOLLIN)
             */
            server.register(selector, SelectionKey.OP_ACCEPT);
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    public void start() {
        initServer();
        System.out.println("服务器启动了......");
        try {
            while (true) {
                Set<SelectionKey> keys = selector.keys();
                System.out.println(keys.size() + " size");
                /*
                1.调用多路复用器(select,poll or epoll)
                如果使用的是select,poll模型：select其实调的是内核的select(fds)方法
                如果使用的是epoll模型：其实调的是内核的epoll_wait
                2.select的参数可以带时间，表示阻塞多少毫秒；如果为零，则无限期阻塞； 不能为负
                可以调用此选择器的wakeup方法结束阻塞
                 */
                while (selector.select(500) > 0) {
                    // 返回有状态的文件描述符的集合
                    Set<SelectionKey> selectionKeys = selector.selectedKeys();
                    Iterator<SelectionKey> iter = selectionKeys.iterator();
                    // 因此多路复用器返回的只是状态，应用程序还得通过系统调用将内核缓冲池中的数据读到应用程序自己的内存里面。
                    while (iter.hasNext()) {
                        SelectionKey key = iter.next();
                        iter.remove();
                        if (key.isAcceptable()) {
                            /*
                            如果接受一个新的连接：
                            如果使用的是select,poll模型：因为它们内核没有空间，那么jvm会将它保存在和前面fd4一样的集合中
                            如果使用的是epoll模型：那么相当于调用内核的epoll_ctl(fd5,ADD,fd6,EPOLLIN)
                             */
                            acceptHandler(key);
                        } else if (key.isReadable()) {
                            readHandler(key);
                        }
                    }
                }
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    public void acceptHandler(SelectionKey key) {

        try {
            ServerSocketChannel ssc = (ServerSocketChannel) key.channel();
            SocketChannel client = ssc.accept();
            client.configureBlocking(false);
            ByteBuffer buffer = ByteBuffer.allocateDirect(8192);
            client.register(selector, SelectionKey.OP_READ, buffer);
            System.out.println("--------------------------------------------------");
            System.out.println("新的客户端：" + client.getRemoteAddress());
            System.out.println("--------------------------------------------------");
        } catch (IOException ioException) {
            ioException.printStackTrace();
        }
    }

    public void readHandler(SelectionKey key) {

        SocketChannel client = (SocketChannel) key.channel();
        ByteBuffer buffer = (ByteBuffer) key.attachment();
        buffer.clear();
        int read = 0;
        try {
            while (true) {
                read = client.read(buffer);
                if (read > 0) {
                    buffer.flip();
                    while (buffer.hasRemaining()) {
                        client.write(buffer);
                    }
                    buffer.clear();
                } else if (read == 0) {
                    break;
                } else {
                    //client.close();
                    break;
                }
            }
        } catch (IOException ioException) {
            ioException.printStackTrace();
        }

    }

    public static void main(String[] args) {

        SocketMultiplexingSingleThread thread = new SocketMultiplexingSingleThread();
        thread.start();
    }
}
```

1. 使用poll模式启动server

```bash
[root@server03 io]# javac org/duo/nio/SocketMultiplexingSingleThread.java && strace -ff -o poll java -Djava.nio.channels.spi.SelectorProvider=sun.nio.ch.PollSelectorProvider org.duo.nio.SocketMultiplexingSingleThread
服务器启动了......
```

2.在同一台服务器上使用nc工具连接server，并测试数据传输

```bash
[root@server03 ~]# nc localhost 9090
hello world
hello world
```

3.使用ss观察tcp状态：可以看到有以下两个文件描述符

- 进程:2288(nc进程)中有一个local:52774到local:9090的文件描述符，状态是ESTAB
- 进程:2278(java进程)中有一个local:9090到local:52774的文件描述符，状态是ESTAB

```bash
[root@server03 ~]# ss -natp
State      Recv-Q Send-Q                                       Local Address:Port                                                      Peer Address:Port
LISTEN     0      128                                                      *:111                                                                  *:*                   users:(("rpcbind",pid=656,fd=4),("systemd",pid=1,fd=40))
LISTEN     0      128                                                      *:22                                                                   *:*                   users:(("sshd",pid=998,fd=3))
LISTEN     0      100                                              127.0.0.1:25                                                                   *:*                   users:(("master",pid=1224,fd=13))
ESTAB      0      0                                           192.168.56.112:22                                                        192.168.56.1:50008               users:(("sshd",pid=1310,fd=3))
ESTAB      0      0                                           192.168.56.112:22                                                        192.168.56.1:50145               users:(("sshd",pid=1913,fd=3))
ESTAB      0      0                                           192.168.56.112:22                                                        192.168.56.1:50147               users:(("sshd",pid=1945,fd=3))
ESTAB      0      64                                          192.168.56.112:22                                                        192.168.56.1:50171               users:(("sshd",pid=2242,fd=3))
LISTEN     0      128                                                     :::111                                                                 :::*                   users:(("rpcbind",pid=656,fd=6),("systemd",pid=1,fd=42))
LISTEN     0      128                                                     :::22                                                                  :::*                   users:(("sshd",pid=998,fd=4))
LISTEN     0      100                                                    ::1:25                                                                  :::*                   users:(("master",pid=1224,fd=14))
LISTEN     0      50                                                      :::9090                                                                :::*                   users:(("java",pid=2278,fd=4))
ESTAB      0      0                                                      ::1:52774                                                              ::1:9090                users:(("nc",pid=2288,fd=3))
ESTAB      0      0                                                      ::1:9090                                                               ::1:52774               users:(("java",pid=2278,fd=7))
```

4.关闭客户端的socket连接：

```bash
[root@server03 ~]# nc localhost 9090
hello world
hello world
^C
[root@server03 ~]#
```

5.再次观察tcp状态：可以看到由于服务器没有关闭socket，所以主动关闭socket的一方会一直停留在FIN-WAIT-2状态而服务器端则一直停留在CLOSE-WAIT状态

```bash
[root@server03 ~]# ss -natp
State      Recv-Q Send-Q                                       Local Address:Port                                                      Peer Address:Port
LISTEN     0      128                                                      *:111                                                                  *:*                   users:(("rpcbind",pid=656,fd=4),("systemd",pid=1,fd=40))
LISTEN     0      128                                                      *:22                                                                   *:*                   users:(("sshd",pid=998,fd=3))
LISTEN     0      100                                              127.0.0.1:25                                                                   *:*                   users:(("master",pid=1224,fd=13))
ESTAB      0      0                                           192.168.56.112:22                                                        192.168.56.1:50008               users:(("sshd",pid=1310,fd=3))
ESTAB      0      0                                           192.168.56.112:22                                                        192.168.56.1:50145               users:(("sshd",pid=1913,fd=3))
ESTAB      0      64                                          192.168.56.112:22                                                        192.168.56.1:50147               users:(("sshd",pid=1945,fd=3))
ESTAB      0      0                                           192.168.56.112:22                                                        192.168.56.1:50171               users:(("sshd",pid=2242,fd=3))
LISTEN     0      128                                                     :::111                                                                 :::*                   users:(("rpcbind",pid=656,fd=6),("systemd",pid=1,fd=42))
LISTEN     0      128                                                     :::22                                                                  :::*                   users:(("sshd",pid=998,fd=4))
LISTEN     0      100                                                    ::1:25                                                                  :::*                   users:(("master",pid=1224,fd=14))
LISTEN     0      50                                                      :::9090                                                                :::*                   users:(("java",pid=2278,fd=4))
FIN-WAIT-2 0      0                                                      ::1:52774                                                              ::1:9090
CLOSE-WAIT 0      0                                                      ::1:9090                                                               ::1:52774               users:(("java",pid=2278,fd=7))
```

分析strace跟踪的内核调用结果：

1. socket(AF_INET6, SOCK_STREAM, IPPROTO_IP) = 4

   ServerSocketChannel.open()会在内核中创建一个文件描述符:4

2. fcntl(4, F_SETFL, O_RDWR|O_NONBLOCK)    = 0 

   设置为非阻塞，对应java中的：server.configureBlocking(false);

3. bind(4, {sa_family=AF_INET6, sin6_port=htons(9090), inet_pton(AF_INET6, "::", &sin6_addr), sin6_flowinfo=htonl(0), sin6_scope_id=0}, 28) = 0

   listen(4, 50) 

   将文件描述符:4绑定9090端口，对应java中的：server.bind(new InetSocketAddress(port));

4. 在poll模型中中，selector = Selector.open();和server.register(selector, SelectionKey.OP_ACCEPT);没有系统调用，都是在jvm中完成。

5. poll([{fd=5, events=POLLIN}, {fd=4, events=POLLIN}], 2, 500) = 1 ([{fd=4, revents=POLLIN}])

   有客户端连接进来后触发poll返回1，对应java中的：selector.select(500) 返回1

6. accept(4, {sa_family=AF_INET6, sin6_port=htons(52780), inet_pton(AF_INET6, "::1", &sin6_addr), sin6_flowinfo=htonl(0), sin6_scope_id=0}, [28]) = 7

   accept返回一个新的客户端连接(文件描述符:7)，相当于Java中的serverSocketChannel.accept()后得到一个SocketChannel

7. fcntl(7, F_SETFL, O_RDWR|O_NONBLOCK)    = 0

   对文件描述符:7做非阻塞处理，相当于java中的client.configureBlocking(false);

8. poll([{fd=5, events=POLLIN}, {fd=4, events=POLLIN}, {fd=7, events=POLLIN}], 3, 500) = 1 ([{fd=7, revents=POLLIN}])

   表示当文件描述符:7中有事件的时候，多路复用器在执行select时会返回文件描述符:7

### 实验2

使用poll模式启动server，并且在客户端关闭socket后，服务端也正常关闭socket，观察tcp状态

SocketMultiplexingSingleThread.java

```java
package org.duo.nio;

import java.io.IOException;
import java.net.InetSocketAddress;
import java.nio.ByteBuffer;
import java.nio.channels.SelectionKey;
import java.nio.channels.Selector;
import java.nio.channels.ServerSocketChannel;
import java.nio.channels.SocketChannel;
import java.util.Iterator;
import java.util.Set;

/**
 * 多路复用器监听的文件描述符的上限：
 *
 * /proc/sys/fs/epoll/max_user_watches (since Linux 2.6.28)
 * This  specifies  a limit on the total number of file descriptors that a user can register across all epoll instances on the system.  The limit is per
 *  real user ID.  Each registered file descriptor costs roughly 90 bytes on a 32-bit kernel, and roughly 160 bytes on a 64-bit kernel.   Currently,  the
 *  default value for max_user_watches is 1/25 (4%) of the available low memory, divided by the registration cost in bytes.
 *
 * 这指定了每个用户可以在系统上的所有epoll实例中注册的文件描述符总数的限制。在32位内核上大约需要90个字节，在64位内核上大约需要160个字节。
 * 目前，max_user_watches的默认值为可用的内存的1/25(4%)除以注册成本（以字节为单位）
 */
public class SocketMultiplexingSingleThread {

    private ServerSocketChannel server = null;
    // 内核的多路复用器被java抽象成了Selector (select poll epoll)
    // 可以根据JVM启动参数设置该java进程是使用poll还是epoll：
    // -Djava.nio.channels.spi.SelectorProvider=sun.nio.ch.EPollSelectorProvider
    // -Djava.nio.channels.spi.SelectorProvider=sun.nio.ch.PollSelectorProvider
    private Selector selector = null;
    int port = 9090;

    public void initServer() {
        try {
            // 以下的三步会在内核中创建一个listen状态的FD4(server)；
            server = ServerSocketChannel.open();
            server.configureBlocking(false);
            server.bind(new InetSocketAddress(port));

            /*
            java会优先选择epoll，但是可以通过设置-D进行修正
            如果在epoll模型下，open相当于调用内核的epoll_create创建了一个新的文件描述符FD5(之后注册在该selector中的fd实际上是放在fd5中)
             */
            selector = Selector.open(); //
            /*
            如果使用的是select或者poll的模型：会在jvm里开辟一个数据将fd4放进去，
            如果使用的是epoll模型：那么相当于调用内核的epoll_ctl(fd5,ADD,fd4,EPOLLIN)
             */
            server.register(selector, SelectionKey.OP_ACCEPT);
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    public void start() {
        initServer();
        System.out.println("服务器启动了......");
        try {
            while (true) {
                Set<SelectionKey> keys = selector.keys();
//                System.out.println(keys.size() + " size");
                /*
                1.调用多路复用器(select,poll or epoll)
                如果使用的是select,poll模型：select其实调的是内核的select(fds)方法
                如果使用的是epoll模型：其实调的是内核的epoll_wait
                2.select的参数可以带时间，表示阻塞多少毫秒；如果为零，则无限期阻塞； 不能为负
                可以调用此选择器的wakeup方法结束阻塞
                 */
                while (selector.select(500) > 0) {
                    // 返回有状态的文件描述符的集合
                    Set<SelectionKey> selectionKeys = selector.selectedKeys();
                    Iterator<SelectionKey> iter = selectionKeys.iterator();
                    // 因此多路复用器返回的只是状态，应用程序还得通过系统调用将内核缓冲池中的数据读到应用程序自己的内存里面。
                    while (iter.hasNext()) {
                        SelectionKey key = iter.next();
                        iter.remove();
                        if (key.isAcceptable()) {
                            /*
                            如果接受一个新的连接：
                            如果使用的是select,poll模型：因为它们内核没有空间，那么jvm会将它保存在和前面fd4一样的集合中
                            如果使用的是epoll模型：那么相当于调用内核的epoll_ctl(fd5,ADD,fd6,EPOLLIN)
                             */
                            acceptHandler(key);
                        } else if (key.isReadable()) {
                            readHandler(key);
                        }
                    }
                }
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    public void acceptHandler(SelectionKey key) {

        try {
            ServerSocketChannel ssc = (ServerSocketChannel) key.channel();
            SocketChannel client = ssc.accept();
            client.configureBlocking(false);
            ByteBuffer buffer = ByteBuffer.allocateDirect(8192);
            client.register(selector, SelectionKey.OP_READ, buffer);
            System.out.println("--------------------------------------------------");
            System.out.println("新的客户端：" + client.getRemoteAddress());
            System.out.println("--------------------------------------------------");
        } catch (IOException ioException) {
            ioException.printStackTrace();
        }
    }

    public void readHandler(SelectionKey key) {

        SocketChannel client = (SocketChannel) key.channel();
        ByteBuffer buffer = (ByteBuffer) key.attachment();
        buffer.clear();
        int read = 0;
        try {
            while (true) {
                read = client.read(buffer);
                if (read > 0) {
                    buffer.flip();
                    while (buffer.hasRemaining()) {
                        client.write(buffer);
                    }
                    buffer.clear();
                } else if (read == 0) {
                    break;
                } else {
                    client.close();
                    break;
                }
            }
        } catch (IOException ioException) {
            ioException.printStackTrace();
        }

    }

    public static void main(String[] args) {

        SocketMultiplexingSingleThread thread = new SocketMultiplexingSingleThread();
        thread.start();
    }
}
```

1. 使用epoll模式启动server

   ```bash
   [root@server03 io]# javac org/duo/nio/SocketMultiplexingSingleThread.java && strace -ff -o epoll java org.duo.nio.SocketMultiplexingSingleThread
   服务器启动了......
   ```

2. 在同一台服务器上使用nc工具连接server，并测试数据传输

   ```bash
   [root@server03 ~]# nc localhost 9090
   hello world
   hello world
   ```

3. 使用ss观察tcp状态：可以看到有以下两个文件描述符

   - 进程:2396(nc进程)中有一个local:52778到local:9090的文件描述符，状态是ESTAB

   - 进程:2384(java进程)中有一个local:9090到local:52778的文件描述符，状态是ESTAB

   ```bash
   [root@server03 ~]# ss -natp
   State      Recv-Q Send-Q                                       Local Address:Port                                                      Peer Address:Port
   LISTEN     0      128                                                      *:111                                                                  *:*                   users:(("rpcbind",pid=656,fd=4),("systemd",pid=1,fd=40))
   LISTEN     0      128                                                      *:22                                                                   *:*                   users:(("sshd",pid=998,fd=3))
   LISTEN     0      100                                              127.0.0.1:25                                                                   *:*                   users:(("master",pid=1224,fd=13))
   ESTAB      0      0                                           192.168.56.112:22                                                        192.168.56.1:50008               users:(("sshd",pid=1310,fd=3))
   ESTAB      0      0                                           192.168.56.112:22                                                        192.168.56.1:50145               users:(("sshd",pid=1913,fd=3))
   ESTAB      0      0                                           192.168.56.112:22                                                        192.168.56.1:50147               users:(("sshd",pid=1945,fd=3))
   ESTAB      0      64                                          192.168.56.112:22                                                        192.168.56.1:50171               users:(("sshd",pid=2242,fd=3))
   LISTEN     0      128                                                     :::111                                                                 :::*                   users:(("rpcbind",pid=656,fd=6),("systemd",pid=1,fd=42))
   LISTEN     0      128                                                     :::22                                                                  :::*                   users:(("sshd",pid=998,fd=4))
   LISTEN     0      100                                                    ::1:25                                                                  :::*                   users:(("master",pid=1224,fd=14))
   LISTEN     0      50                                                      :::9090                                                                :::*                   users:(("java",pid=2384,fd=4))
   ESTAB      0      0                                                      ::1:9090                                                               ::1:52778               users:(("java",pid=2384,fd=7))
   ESTAB      0      0                                                      ::1:52778                                                              ::1:9090                users:(("nc",pid=2396,fd=3))
   ```

4. 关闭客户端的socket连接

   ```bash
   [root@server03 ~]# nc localhost 9090
   hello world
   hello world
   ^C
   [root@server03 ~]#
   
   ```

5. 再次观察tcp状态：客户端进程中的local:52778到local:9090的文件描述符先是进入TIME-WAIT状态，在2MSL后关闭。

   ```bash
   [root@server03 ~]# ss -natp
   State      Recv-Q Send-Q                                       Local Address:Port                                                      Peer Address:Port
   LISTEN     0      128                                                      *:111                                                                  *:*                   users:(("rpcbind",pid=656,fd=4),("systemd",pid=1,fd=40))
   LISTEN     0      128                                                      *:22                                                                   *:*                   users:(("sshd",pid=998,fd=3))
   LISTEN     0      100                                              127.0.0.1:25                                                                   *:*                   users:(("master",pid=1224,fd=13))
   ESTAB      0      0                                           192.168.56.112:22                                                        192.168.56.1:50008               users:(("sshd",pid=1310,fd=3))
   ESTAB      0      0                                           192.168.56.112:22                                                        192.168.56.1:50145               users:(("sshd",pid=1913,fd=3))
   ESTAB      0      0                                           192.168.56.112:22                                                        192.168.56.1:50147               users:(("sshd",pid=1945,fd=3))
   ESTAB      0      64                                          192.168.56.112:22                                                        192.168.56.1:50171               users:(("sshd",pid=2242,fd=3))
   LISTEN     0      128                                                     :::111                                                                 :::*                   users:(("rpcbind",pid=656,fd=6),("systemd",pid=1,fd=42))
   LISTEN     0      128                                                     :::22                                                                  :::*                   users:(("sshd",pid=998,fd=4))
   LISTEN     0      100                                                    ::1:25                                                                  :::*                   users:(("master",pid=1224,fd=14))
   LISTEN     0      50                                                      :::9090                                                                :::*                   users:(("java",pid=2384,fd=4))
   TIME-WAIT  0      0                                                      ::1:52778  
   [root@server03 ~]# ss -natp
   State      Recv-Q Send-Q                                       Local Address:Port                                                      Peer Address:Port
   LISTEN     0      128                                                      *:111                                                                  *:*                   users:(("rpcbind",pid=656,fd=4),("systemd",pid=1,fd=40))
   LISTEN     0      128                                                      *:22                                                                   *:*                   users:(("sshd",pid=998,fd=3))
   LISTEN     0      100                                              127.0.0.1:25                                                                   *:*                   users:(("master",pid=1224,fd=13))
   ESTAB      0      0                                           192.168.56.112:22                                                        192.168.56.1:50008               users:(("sshd",pid=1310,fd=3))
   ESTAB      0      0                                           192.168.56.112:22                                                        192.168.56.1:50145               users:(("sshd",pid=1913,fd=3))
   ESTAB      0      0                                           192.168.56.112:22                                                        192.168.56.1:50147               users:(("sshd",pid=1945,fd=3))
   ESTAB      0      64                                          192.168.56.112:22                                                        192.168.56.1:50171               users:(("sshd",pid=2242,fd=3))
   LISTEN     0      128                                                     :::111                                                                 :::*                   users:(("rpcbind",pid=656,fd=6),("systemd",pid=1,fd=42))
   LISTEN     0      128                                                     :::22                                                                  :::*                   users:(("sshd",pid=998,fd=4))
   LISTEN     0      100                                                    ::1:25                                                                  :::*                   users:(("master",pid=1224,fd=14))
   LISTEN     0      50                                                      :::9090                                                                :::*                   users:(("java",pid=2384,fd=4))
   ```

分析strace跟踪的内核调用结果：

1. socket(AF_INET6, SOCK_STREAM, IPPROTO_IP) = 4

   ServerSocketChannel.open()会在内核中创建一个文件描述符:4

2. fcntl(4, F_SETFL, O_RDWR|O_NONBLOCK)    = 0

   设置为非阻塞，对应java中的：server.configureBlocking(false);

3. bind(4, {sa_family=AF_INET6, sin6_port=htons(9090), inet_pton(AF_INET6, "::", &sin6_addr), sin6_flowinfo=htonl(0), sin6_scope_id=0}, 28) = 0

   listen(4, 50)

   将文件描述符:4绑定9090端口，对应java中的：server.bind(new InetSocketAddress(port));

4.  epoll_create(256)                       = 7

   epoll_create创建了一个新的文件描述符(该进程的eventpoll，用户管理该进程的所有文件描述符)，对应java中的：selector = Selector.open();

5. epoll_ctl(7, EPOLL_CTL_ADD, 4, {EPOLLIN, {u32=4, u64=6744024928841891844}}) = 0

   将之前生成的文件描述符:4(ServerSocketChannel)注册到文件描述符:7(eventpoll)中

6. epoll_wait(7, [{EPOLLIN, {u32=4, u64=6744024928841891844}}], 8192, 500) = 1

   有客户端连接进来后触发poll返回1，对应java中的：selector.select(500) 返回1

7.  accept(4, {sa_family=AF_INET6, sin6_port=htons(52782), inet_pton(AF_INET6, "::1", &sin6_addr), sin6_flowinfo=htonl(0), sin6_scope_id=0}, [28]) = 8

   accept返回一个新的客户端连接(文件描述符:8)，相当于Java中的serverSocketChannel.accept()后得到一个SocketChannel

8. fcntl(8, F_SETFL, O_RDWR|O_NONBLOCK)    = 0

   对文件描述符:8做非阻塞处理，相当于java中的client.configureBlocking(false);

9. epoll_ctl(7, EPOLL_CTL_ADD, 8, {EPOLLIN, {u32=8, u64=8}}) = 0

   将客户端连接:8(SocketChannel)注册到文件描述符:7(eventpoll)中

10. epoll_wait(7, [{EPOLLIN, {u32=8, u64=8}}], 8192, 500) = 1

    表示当文件描述符:8中有事件的时候，多路复用器在执行select时会返回文件描述符:8

### 实验3

SocketMultiplexingMultiThread.java

```java
package org.duo.nio;

import java.io.IOException;
import java.net.InetSocketAddress;
import java.nio.ByteBuffer;
import java.nio.channels.SelectionKey;
import java.nio.channels.Selector;
import java.nio.channels.ServerSocketChannel;
import java.nio.channels.SocketChannel;
import java.util.Iterator;
import java.util.Set;

/**
 * 多路复用器监听的文件描述符的上限：
 * <p>
 * /proc/sys/fs/epoll/max_user_watches (since Linux 2.6.28)
 * This  specifies  a limit on the total number of file descriptors that a user can register across all epoll instances on the system.  The limit is per
 * real user ID.  Each registered file descriptor costs roughly 90 bytes on a 32-bit kernel, and roughly 160 bytes on a 64-bit kernel.   Currently,  the
 * default value for max_user_watches is 1/25 (4%) of the available low memory, divided by the registration cost in bytes.
 * <p>
 * 这指定了每个用户可以在系统上的所有epoll实例中注册的文件描述符总数的限制。在32位内核上大约需要90个字节，在64位内核上大约需要160个字节。
 * 目前，max_user_watches的默认值为可用的内存的1/25(4%)除以注册成本（以字节为单位）
 */
public class SocketMultiplexingMultiThread {

    private ServerSocketChannel server = null;
    // 内核的多路复用器被java抽象成了Selector (select poll epoll)
    // 可以根据JVM启动参数设置该java进程是使用poll还是epoll：
    // -Djava.nio.channels.spi.SelectorProvider=sun.nio.ch.EPollSelectorProvider
    // -Djava.nio.channels.spi.SelectorProvider=sun.nio.ch.PollSelectorProvider
    private Selector selector = null;
    int port = 9090;

    public void initServer() {
        try {
            // 以下的三步会在内核中创建一个listen状态的FD4(server)；
            server = ServerSocketChannel.open();
            server.configureBlocking(false);
            server.bind(new InetSocketAddress(port));

            /*
            java会优先选择epoll，但是可以通过设置-D进行修正
            如果在epoll模型下，open相当于调用内核的epoll_create创建了一个新的文件描述符FD5(之后注册在该selector中的fd实际上是放在fd5中)
             */
            selector = Selector.open(); //
            /*
            如果使用的是select或者poll的模型：会在jvm里开辟一个数据将fd4放进去，
            如果使用的是epoll模型：那么相当于调用内核的epoll_ctl(fd5,ADD,fd4,EPOLLIN)
             */
            server.register(selector, SelectionKey.OP_ACCEPT);
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    public void start() {
        initServer();
        System.out.println("服务器启动了......");
        try {
            while (true) {
                Set<SelectionKey> keys = selector.keys();
//                System.out.println(keys.size() + " size");
                /*
                1.调用多路复用器(select,poll or epoll)
                如果使用的是select,poll模型：select其实调的是内核的select(fds)方法
                如果使用的是epoll模型：其实调的是内核的epoll_wait
                2.select的参数可以带时间，表示阻塞多少毫秒；如果为零，则无限期阻塞； 不能为负
                可以调用此选择器的wakeup方法结束阻塞
                 */
                while (selector.select(50) > 0) {
                    // 返回有状态的文件描述符的集合
                    Set<SelectionKey> selectionKeys = selector.selectedKeys();
                    Iterator<SelectionKey> iter = selectionKeys.iterator();
                    // 因此多路复用器返回的只是状态，应用程序还得通过系统调用将内核缓冲池中的数据读到应用程序自己的内存里面。
                    while (iter.hasNext()) {
                        SelectionKey key = iter.next();
                        // 从本次调用select后返回的集合中删除，并没有取消多路复用器对该通道的监听，所以在下一次选择操作时如果该通道有事件达到还是会包含在集合中
                        iter.remove();
                        if (key.isAcceptable()) {
                            /*
                            如果接受一个新的连接：
                            如果使用的是select,poll模型：因为它们内核没有空间，那么jvm会将它保存在和前面fd4一样的集合中
                            如果使用的是epoll模型：那么相当于调用内核的epoll_ctl(fd5,ADD,fd6,EPOLLIN)
                             */
                            acceptHandler(key);
                        } else if (key.isReadable()) {
                            /*
                            选择器将取消对该键对应通道的监听，在下一次选择操作时，该通道不会再作为选择器监听的对象：
                            在kernel层会发生一次系统调用：epoll_ctl(7, EPOLL_CTL_DEL, 8, 0x7fba8da263f0) = 0，表示将监听的FD从多路复用器创建的文件描述符代表的空间中删除掉
                            它是选择器对象的同步方法，因此如果与涉及同一选择器的取消或选择操作同时调用，会阻塞
                             */
//                            key.cancel();
                            readHandler(key);
                        } else if (key.isWritable()) {
                            // 写事件：只要文件描述符的Send-Q(通过ss -natp)为空或者没有满，selector.select就会返回该文件描述符(可以进行写操作)
                            // 但是什么时候写不是依赖Send-Q是不是有空间，而是应用程序先要准备好要写入的数据后，再去看Send-Q是否有空间。
                            // 所以read事件在服务一启动后就需要注册，而write事件是在应用程序准备好要写入的数据后再注册。
                            // 如果一开始就注册write事件，那么就会进入死循环，一直被调起。
//                            key.cancel();
                            writeHandler(key);
                        }
                    }
                }
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    public void acceptHandler(SelectionKey key) {

        try {
            ServerSocketChannel ssc = (ServerSocketChannel) key.channel();
            SocketChannel client = ssc.accept();
            client.configureBlocking(false);
            ByteBuffer buffer = ByteBuffer.allocateDirect(8192);
            client.register(selector, SelectionKey.OP_READ, buffer);
            System.out.println("--------------------------------------------------");
            System.out.println("新的客户端：" + client.getRemoteAddress());
            System.out.println("--------------------------------------------------");
        } catch (IOException ioException) {
            ioException.printStackTrace();
        }
    }

    public void readHandler(SelectionKey key) {
        new Thread(() -> {
            System.out.println("Read Handler");
            SocketChannel client = (SocketChannel) key.channel();
            ByteBuffer buffer = (ByteBuffer) key.attachment();
            buffer.clear();
            int read = 0;
            try {
                while (true) {
                    read = client.read(buffer);
                    System.out.println(Thread.currentThread().getName() + " " + read);
                    if (read > 0) {
                        // 在读到了客户端发来的数据之后再注册写事件
                        client.register(key.selector(), SelectionKey.OP_WRITE, buffer);
                    } else if (read == 0) {
                        break;
                    } else {
                        client.close();
                        break;
                    }
                }
            } catch (IOException ioException) {
                ioException.printStackTrace();
            }
        }).start();
    }

    public void writeHandler(SelectionKey key) {
        new Thread(() -> {
            System.out.println("Write Handler");
            SocketChannel client = (SocketChannel) key.channel();
            ByteBuffer buffer = (ByteBuffer) key.attachment();
            buffer.flip();
            while (buffer.hasRemaining()) {
                try {
                    client.write(buffer);
                } catch (IOException ioException) {
                    ioException.printStackTrace();
                }
            }
            try {
                Thread.sleep(2000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            buffer.clear();
            key.cancel();
            try {
                client.close();
            } catch (IOException ioException) {
                ioException.printStackTrace();
            }
        }).start();
    }

    public static void main(String[] args) {

        SocketMultiplexingMultiThread thread = new SocketMultiplexingMultiThread();
        thread.start();
    }
}
```

1. 运行SocketMultiplexingMultiThread.java

   ```bash
   [root@server03 io]# javac org/duo/nio/SocketMultiplexingMultiThread.java && strace -ff -o cancel java org.duo.nio.SocketMultiplexingMultiThread
   服务器启动了......
   ```

2. 使用nc连接到服务端

   ```bash
   [root@server03 io]# nc 127.0.0.1 9090
   
   
   ```

3. 发送数据

   ```bash
   [root@server03 io]# nc 127.0.0.1 9090
   123
   
   
   ```

4. 观察服务端的输出：可以看到ReadHandler和WriteHandler都被多次调起

   ```bash 
   [root@server03 io]# javac org/duo/nio/SocketMultiplexingMultiThread.java && strace -ff -o cancel java org.duo.nio.SocketMultiplexingMultiThread
   服务器启动了......
   --------------------------------------------------
   新的客户端：/127.0.0.1:53338
   --------------------------------------------------
   Read Handler
   Thread-0 4
   Thread-0 0
   Read Handler
   Thread-1 0
   Write Handler
   Write Handler
   Write Handler
   Write Handler
   Write Handler
   Write Handler
   Write Handler
   Write Handler
   Write Handler
   Write Handler
   Write Handler
   Write Handler
   Write Handler
   Write Handler
   Write Handler
   Write Handler
   Write Handler
   Write Handler
   Write Handler
   Write Handler
   Write Handler
   .........
   ```

原因分析：

1. ReadHandler方法中通过线程的方式去读取数据，所以主线程不会阻塞会立即返回，重新进行select处理；
2. 在主线程中的多路复用器会再次调用select的时候，由于在readHandler中新起的线程中还没有完成处理完客户端发送过来的数据，所以在事件集合中会包含相同的read事件；即在线程开始处理读取时间到处理完的这段时差内，由于主线程是不阻塞的，所以这个read事件会被重复触发（解决方案，在readHandler之前将监听的文件描述符的事件从多路复用器中移除）；
3. WriteHandler被重复触发的原因：在ReadHandler中往多路复用器注册完write事件的监听后，只要Send-Q中有剩余空间，write事件就会被触发（解决方案，在writeHandler之前将监听的文件描述符的事件从多路复用器中移除）。

解决方案：

1. 如果采用R/W分开在不同的线程中处理的话(同一FD的R/W是非线性的)，为了避免R/W处理被重复触发的问题，在每次R/W处理之前都需要通过一次系统调用将事件先从多路复用器中移除，读取完后再通过一次系统调用注册进去，这样的话增加了很多的系统调用。
2. 如果既想利用多核又想减少系统调用，可将N个FD分组，每一个组一个selector，然后将一个selector放到一个线程上(这样同一个FD的R/W肯定是在同一个线程内处理，是线性的处理，就不需要增加cancel操作，从而达到大幅减少系统调用的目的)；多个selector并行。
3. 上述2的逻辑就是分治，在这个基础上还可以进一步细化：可以用一个线程的selector只关注accept，然后把接受的客户端的FD分配给其他线程的selector。

### 实验4

多selector并行

MainThread.java

```java
package org.duo.reactor;

public class MainThread {

    public static void main(String[] args) {

        // 创建I/O Thread：一个或多个：
        // 混杂模式：每个selector都会被分配client进行R/W
        // 也可以将listen单独绑定一个selector，client分配到其他的selector
        SelectorThreadGroup selectorThreadGroup = new SelectorThreadGroup(3);

        // 将监听的server注册到某一个selector上
        selectorThreadGroup.bind(9999);
    }
}
```

SelectorThreadGroup.java

```java
package org.duo.reactor;

import java.io.IOException;
import java.net.InetSocketAddress;
import java.nio.channels.*;
import java.util.concurrent.atomic.AtomicInteger;

public class SelectorThreadGroup {

    SelectorThread[] selectorThread;
    ServerSocketChannel serverSocketChannel;
    AtomicInteger threadId = new AtomicInteger(0);

    SelectorThreadGroup(int num) {

        selectorThread = new SelectorThread[num];
        for (int i = 0; i < num; i++) {
            selectorThread[i] = new SelectorThread(this);
            // selectorThread运行后，会在执行select的地方阻塞，由于此时selector中还没有注册任何事件，如果不进行wakeup，那么会永久阻塞
            new Thread(selectorThread[i]).start();
        }
    }

    public void bind(int port) {
        try {
            serverSocketChannel = ServerSocketChannel.open();
            serverSocketChannel.configureBlocking(false);
            serverSocketChannel.bind(new InetSocketAddress(port));
            // 注册到哪个selector上呢？
            nextSelector(serverSocketChannel);
        } catch (IOException ioException) {
            ioException.printStackTrace();
        }
    }

    /**
     * ServerSocketChannel和SocketChannel都复用这个方法
     *
     * @param channel
     */
    public void nextSelector(Channel channel) {

        // 混杂模式
//        SelectorThread selectorThread = next();
//
////        // channel有可能是server也有可能是client
////        ServerSocketChannel server = (ServerSocketChannel)channel;
////        try {
////            // 由于在selectorThread中已经调用了Selector的select方法，最终调用的是：sun.nio.ch.SelectorImpl.lockAndDoSelect
////            // 而ServerSocketChannel的register最终会调用也会sun.nio.ch.SelectorImpl#register
////            // 而lockAndDoSelect和register在SelectorImpl类中是同步方法，selectorThread在执行select方法的时候已经持有了该selector的锁，所以在这里在执行下面的方法时会因为获取不到锁而阻塞。
////            server.register(selectorThread.selector, SelectionKey.OP_ACCEPT);
////            // wakeup可以使selector的select方法立刻返回，从而释放锁
////            // 如果将wakeup放在register之前执行，那么有可能selectorThread在被唤醒后register线程还没有完成注册前就又执行到了select方法的地方，那么在register线程中注册的事件还是无法被监听到
////            // 如果wakeup放在register之后执行，那么有可能先执行selectorThread的select方法，然后再执行register线程的register方法，那么register线程将会被阻塞住，从而无法再执行wakeup方法
////            // 所以将wakeup放在register的前后都不行，涉及到多线程之间的通信最好使用队列
////            selectorThread.selector.wakeup();
////        } catch (ClosedChannelException e) {
////            e.printStackTrace();
////        }
//
//        // 1.通过队列传递数据
//        selectorThread.linkedBlockingQueue.add(channel);
//        // 2.通过打断阻塞，让对应的线程自己在打断后完成事件的注册
//        selectorThread.selector.wakeup();

        // listen单独绑定一个selector，client分配到其他的selector
        try {
            if (channel instanceof ServerSocketChannel) {
                selectorThread[0].linkedBlockingQueue.put(channel);
                selectorThread[0].selector.wakeup();
            } else if (channel instanceof SocketChannel) {
                SelectorThread selectorThread = next();
                selectorThread.linkedBlockingQueue.add(channel);
                selectorThread.selector.wakeup();
            }
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }

    private SelectorThread next() {

//        // 混杂模式
//        int index = threadId.incrementAndGet() % selectorThread.length;
//        return selectorThread[index];

        // listen单独绑定一个selector，client分配到其他的selector
        int index = threadId.incrementAndGet() % (selectorThread.length - 1);
        return selectorThread[index + 1];
    }
}
```

SelectorThread.java

```java
package org.duo.reactor;

import java.io.IOException;
import java.nio.ByteBuffer;
import java.nio.channels.*;
import java.util.Iterator;
import java.util.Set;
import java.util.concurrent.LinkedBlockingQueue;

public class SelectorThread implements Runnable {

    //每个线程对应一个selector，多线程情况下，并发客户端被分配到某一个selector上
    //注意：每个客户端只绑定一个selector
    Selector selector = null;
    LinkedBlockingQueue<Channel> linkedBlockingQueue = new LinkedBlockingQueue<>();
    SelectorThreadGroup selectorThreadGroup;

    SelectorThread(SelectorThreadGroup selectorThreadGroup) {
        try {
            this.selectorThreadGroup = selectorThreadGroup;
            selector = Selector.open();
        } catch (IOException ioException) {
            ioException.printStackTrace();
        }
    }

    @Override
    public void run() {

        while (true) {
            try {
                /*
                如果当前线程执行select，没有事件到达，那么会进入阻塞状态
                而另外一个线程往相同的多路服务器中增加了对其他文件描述符的监听事件
                由于当前线程中只能监听执行select时刻的多路复用器中的文件描述符，无法监听之后由其他线程追加的文件描述符中的事件，因此有可能当前线程会进入永久阻塞的状态
                所以会有selector中会有wakeup方法，唤醒某个阻塞的文件描述符，并且返回值为0
                 */
//                System.out.println(Thread.currentThread().getName() + " before select......" + selector.keys().size());
                int nums = selector.select(); //阻塞
//                System.out.println(Thread.currentThread().getName() + " after select......" + selector.keys().size());
                // 处理selectedKeys
                if (nums > 0) {
                    Set<SelectionKey> keys = selector.selectedKeys();
                    Iterator<SelectionKey> iter = keys.iterator();
                    // 同一文件描述符的R/W一定是在同一线程中处理
                    while (iter.hasNext()) {
                        SelectionKey key = iter.next();
                        iter.remove();
                        // 多线程中：新的客户端注册在哪里呢？
                        if (key.isAcceptable()) {
                            acceptHandler(key);
                        } else if (key.isReadable()) {
                            readHandler(key);
                        } else if (key.isWritable()) {
                            writeHandler(key);
                        }
                    }
                }
                // 由每个线程自己注册：serverSocketChannel、SocketChannel
                if (!linkedBlockingQueue.isEmpty()) {
                    Channel channel = (Channel) linkedBlockingQueue.take();
                    if (channel instanceof ServerSocketChannel) {
                        ServerSocketChannel server = (ServerSocketChannel) channel;
                        server.register(selector, SelectionKey.OP_ACCEPT);
                        System.out.println(Thread.currentThread().getName() + " register listen");
                    } else if (channel instanceof SocketChannel) {
                        SocketChannel client = (SocketChannel) channel;
                        ByteBuffer buffer = ByteBuffer.allocateDirect(4098);
                        client.register(selector, SelectionKey.OP_READ, buffer);
                        System.out.println(Thread.currentThread().getName() + " register client: " + client.getRemoteAddress());
                    }
                }
            } catch (IOException | InterruptedException ioException) {
                ioException.printStackTrace();
            }
        }
    }

    private void acceptHandler(SelectionKey key) {

        try {
            System.out.println(Thread.currentThread().getName() + " acceptHandler......");
            ServerSocketChannel ssc = (ServerSocketChannel) key.channel();
            SocketChannel client = ssc.accept();
            client.configureBlocking(false);
            // 多线程中需要选择一个多路复用器register
            selectorThreadGroup.nextSelector(client);
        } catch (IOException ioException) {
            ioException.printStackTrace();
        }
    }

    public void readHandler(SelectionKey key) {

        SocketChannel client = (SocketChannel) key.channel();
        ByteBuffer buffer = (ByteBuffer) key.attachment();
        buffer.clear();
        int read = 0;
        try {
            while (true) {
                read = client.read(buffer);
                if (read > 0) {
                    buffer.flip();
                    while (buffer.hasRemaining()) {
                        client.write(buffer);
                    }
                    buffer.clear();
                } else if (read == 0) { // 等于0：没有读取到任何数据
                    break;
                } else {//如果小于0：客户端断开了连接，或者出现异常等情况
                    System.out.println("client: " + client.getRemoteAddress() + " closed......");
                    key.cancel();
                    break;
                }
            }
        } catch (IOException ioException) {
            ioException.printStackTrace();
        }
    }

    public void writeHandler(SelectionKey key) {
    }
}
```

### 实验5

多selector并行（使用boss group、worker group）

MainThread.java

```java
package org.duo.reactor;

public class MainThread {

    public static void main(String[] args) {

        // 创建I/O Thread：一个或多个：
        // 混杂模式：每个selector都会被分配client进行R/W
        // 也可以将listen单独绑定一个boss group，client绑定一个worker group
        SelectorThreadGroup bossGroup = new SelectorThreadGroup(3);
        SelectorThreadGroup workerGroup = new SelectorThreadGroup(3);
        bossGroup.setWorker(workerGroup);
        // 将监听的server注册到某一个selector上
        bossGroup.bind(9999);
//        bossGroup.bind(7777);
//        bossGroup.bind(6666);
    }
}
```

SelectorThreadGroup.java

```java
package org.duo.reactor;

import java.io.IOException;
import java.net.InetSocketAddress;
import java.nio.channels.*;
import java.util.concurrent.atomic.AtomicInteger;

public class SelectorThreadGroup {

    SelectorThread[] selectorThread;
    ServerSocketChannel serverSocketChannel;
    AtomicInteger threadId = new AtomicInteger(0);
    // 初始化都是boss组，如果没有设置worker那么就是混杂模式：client会平均分配到该组中的每一个selector
    // 如果设置了worker那么：listen会被注册在boss组，而client会被注册到该SelectorThreadGroup对象中的workerSelectorThreadGroup对象中
    SelectorThreadGroup workerSelectorThreadGroup = this;

    public void setWorker(SelectorThreadGroup selectorThreadGroup) {
        this.workerSelectorThreadGroup = selectorThreadGroup;
    }

    SelectorThreadGroup(int num) {

        selectorThread = new SelectorThread[num];
        for (int i = 0; i < num; i++) {
            selectorThread[i] = new SelectorThread(this);
            // selectorThread运行后，会在执行select的地方阻塞，由于此时selector中还没有注册任何事件，如果不进行wakeup，那么会永久阻塞
            new Thread(selectorThread[i]).start();
        }
    }

    public void bind(int port) {
        try {
            serverSocketChannel = ServerSocketChannel.open();
            serverSocketChannel.configureBlocking(false);
            serverSocketChannel.bind(new InetSocketAddress(port));
            // 注册到哪个selector上呢？
            nextSelector(serverSocketChannel);
        } catch (IOException ioException) {
            ioException.printStackTrace();
        }
    }

    /**
     * ServerSocketChannel和SocketChannel都复用这个方法
     *
     * @param channel
     */
    public void nextSelector(Channel channel) {

//        // channel有可能是server也有可能是client
//        ServerSocketChannel server = (ServerSocketChannel)channel;
//        try {
//            // 由于在selectorThread中已经调用了Selector的select方法，最终调用的是：sun.nio.ch.SelectorImpl.lockAndDoSelect
//            // 而ServerSocketChannel的register最终会调用也会sun.nio.ch.SelectorImpl#register
//            // 而lockAndDoSelect和register在SelectorImpl类中是同步方法，selectorThread在执行select方法的时候已经持有了该selector的锁，所以在这里在执行下面的方法时会因为获取不到锁而阻塞。
//            server.register(selectorThread.selector, SelectionKey.OP_ACCEPT);
//            // wakeup可以使selector的select方法立刻返回，从而释放锁
//            // 如果将wakeup放在register之前执行，那么有可能selectorThread在被唤醒后register线程还没有完成注册前就又执行到了select方法的地方，那么在register线程中注册的事件还是无法被监听到
//            // 如果wakeup放在register之后执行，那么有可能先执行selectorThread的select方法，然后再执行register线程的register方法，那么register线程将会被阻塞住，从而无法再执行wakeup方法
//            // 所以将wakeup放在register的前后都不行，涉及到多线程之间的通信最好使用队列
//            selectorThread.selector.wakeup();
//        } catch (ClosedChannelException e) {
//            e.printStackTrace();
//        }

        try {
            if (channel instanceof ServerSocketChannel) {
                SelectorThread selectorThread = nextListen();
                // 1.通过队列传递数据
                selectorThread.linkedBlockingQueue.put(channel);
                // 2.通过打断阻塞，让对应的线程自己在打断后完成事件的注册
                selectorThread.selector.wakeup();
            } else if (channel instanceof SocketChannel) {
                SelectorThread selectorThread = nextClient();
                // 1.通过队列传递数据
                selectorThread.linkedBlockingQueue.put(channel);
                // 2.通过打断阻塞，让对应的线程自己在打断后完成事件的注册
                selectorThread.selector.wakeup();
            }
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }

    private SelectorThread nextListen() {

        int index = threadId.incrementAndGet() % selectorThread.length;
        return selectorThread[index];
    }

    private SelectorThread nextClient() {

        int index = workerSelectorThreadGroup.threadId.incrementAndGet() % workerSelectorThreadGroup.selectorThread.length;
        return workerSelectorThreadGroup.selectorThread[index];
    }
}
```

SelectorThread.java

```java
package org.duo.reactor;

import java.io.IOException;
import java.nio.ByteBuffer;
import java.nio.channels.*;
import java.util.Iterator;
import java.util.Set;
import java.util.concurrent.LinkedBlockingQueue;

public class SelectorThread implements Runnable {

    //每个线程对应一个selector，多线程情况下，并发客户端被分配到某一个selector上
    //注意：每个客户端只绑定一个selector
    Selector selector = null;
    LinkedBlockingQueue<Channel> linkedBlockingQueue = new LinkedBlockingQueue<>();
    SelectorThreadGroup selectorThreadGroup;

    SelectorThread(SelectorThreadGroup selectorThreadGroup) {
        try {
            this.selectorThreadGroup = selectorThreadGroup;
            selector = Selector.open();
        } catch (IOException ioException) {
            ioException.printStackTrace();
        }
    }

    @Override
    public void run() {

        while (true) {
            try {
                /*
                如果当前线程执行select，没有事件到达，那么会进入阻塞状态
                而另外一个线程往相同的多路服务器中增加了对其他文件描述符的监听事件
                由于当前线程中只能监听执行select时刻的多路复用器中的文件描述符，无法监听之后由其他线程追加的文件描述符中的事件，因此有可能当前线程会进入永久阻塞的状态
                所以会有selector中会有wakeup方法，唤醒某个阻塞的文件描述符，并且返回值为0
                 */
//                System.out.println(Thread.currentThread().getName() + " before select......" + selector.keys().size());
                int nums = selector.select(); //阻塞
//                System.out.println(Thread.currentThread().getName() + " after select......" + selector.keys().size());
                // 处理selectedKeys
                if (nums > 0) {
                    Set<SelectionKey> keys = selector.selectedKeys();
                    Iterator<SelectionKey> iter = keys.iterator();
                    // 同一文件描述符的R/W一定是在同一线程中处理
                    while (iter.hasNext()) {
                        SelectionKey key = iter.next();
                        iter.remove();
                        // 多线程中：新的客户端注册在哪里呢？
                        if (key.isAcceptable()) {
                            acceptHandler(key);
                        } else if (key.isReadable()) {
                            readHandler(key);
                        } else if (key.isWritable()) {
                            writeHandler(key);
                        }
                    }
                }
                // 由每个线程自己注册：serverSocketChannel、SocketChannel
                if (!linkedBlockingQueue.isEmpty()) {
                    Channel channel = (Channel) linkedBlockingQueue.take();
                    if (channel instanceof ServerSocketChannel) {
                        ServerSocketChannel server = (ServerSocketChannel) channel;
                        server.register(selector, SelectionKey.OP_ACCEPT);
                        System.out.println(Thread.currentThread().getName() + " register listen");
                    } else if (channel instanceof SocketChannel) {
                        SocketChannel client = (SocketChannel) channel;
                        ByteBuffer buffer = ByteBuffer.allocateDirect(4098);
                        client.register(selector, SelectionKey.OP_READ, buffer);
                        System.out.println(Thread.currentThread().getName() + " register client: " + client.getRemoteAddress());
                    }
                }
            } catch (IOException | InterruptedException ioException) {
                ioException.printStackTrace();
            }
        }
    }

    private void acceptHandler(SelectionKey key) {

        try {
            System.out.println(Thread.currentThread().getName() + " acceptHandler......");
            ServerSocketChannel ssc = (ServerSocketChannel) key.channel();
            SocketChannel client = ssc.accept();
            client.configureBlocking(false);
            // 多线程中需要选择一个多路复用器register
            selectorThreadGroup.nextSelector(client);
        } catch (IOException ioException) {
            ioException.printStackTrace();
        }
    }

    public void readHandler(SelectionKey key) {

        SocketChannel client = (SocketChannel) key.channel();
        ByteBuffer buffer = (ByteBuffer) key.attachment();
        buffer.clear();
        int read = 0;
        try {
            while (true) {
                read = client.read(buffer);
                if (read > 0) {
                    buffer.flip();
                    while (buffer.hasRemaining()) {
                        client.write(buffer);
                    }
                    buffer.clear();
                } else if (read == 0) { // 等于0：没有读取到任何数据
                    break;
                } else {//如果小于0：客户端断开了连接，或者出现异常等情况
                    System.out.println("client: " + client.getRemoteAddress() + " closed......");
                    key.cancel();
                    break;
                }
            }
        } catch (IOException ioException) {
            ioException.printStackTrace();
        }
    }

    public void writeHandler(SelectionKey key) {
    }
}
```

# tcp/ip

## 三次握手过程：

1. 主机A发送位码为SYN＝1,随机产生Seq Number=XXX的数据包到服务器，主机B由SYN＝1知道，A要求建立联机，主机A的状态变为SYN_SENT；
2. 主机B收到请求后要确认联机信息，向A发送Ack Number=(主机A的Seq+1),SYN=1,ACK=1,随机产生Seq Number=YYY的包，此时主机B的状态变为SYN_RCVD；
3. 主机A收到后检查Ack Number是否正确，即第一次发送的Seq Number+1,以及位码ACK是否为1，若正确，主机A状态变为ESTABLISHED；主机A会再发送Ack Number=(主机B的Seq Number+1),ACK=1，主机B收到后确认Ack Number与ACK=1，若正确，主机B状态变为ESTABLISHED，连接建立成功；

## 四次挥手的过程：

1. 首先客户端想要释放连接，向服务器端发送一段TCP报文，其中：标记位为FIN，表示“请求释放连接“；随后客户端进入FIN-WAIT-1阶段，即半关闭阶段。并且停止在客户端到服务器端方向上发送数据，但是客户端仍然能接收从服务器端传输过来的数据。

   注意：这里不发送的是正常连接时传输的数据(非确认报文)，而不是一切数据，所以客户端仍然能发送ACK确认报文。

2. 服务器端接收到从客户端发出的TCP报文之后，确认了客户端想要释放连接，随后服务器端结束ESTABLISHED阶段，进入CLOSE-WAIT阶段（半关闭状态）并返回一段TCP报文，其中：标记位为ACK，表示“接收到客户端发送的释放连接的请求”；客户端收到从服务器端发出的TCP报文之后，确认了服务器收到了客户端发出的释放连接请求，随后客户端结束FIN-WAIT-1阶段，进入FIN-WAIT-2阶段，此时连接已经断开了一半了。如果服务器还有数据要发送给客户端，就会继续发送；

3. 服务器在将数据发送完成后将释放服务器端到客户端方向上的连接，于是再次向客户端发出一段TCP报文，其中：标记位为FIN，表示“已经准备好释放连接了”。随后服务器端结束CLOSE-WAIT阶段，进入LAST-ACK阶段。并且停止在服务器端到客户端的方向上发送数据，但是服务器端仍然能够接收从客户端传输过来的数据。

4. 客户端收到从服务器端发出的TCP报文，确认了服务器端已做好释放连接的准备，结束FIN-WAIT-2阶段，进入TIME-WAIT阶段，并向服务器端发送一段报文，其中：标记位为ACK，表示“接收到服务器准备好释放连接的信号”。随后客户端开始在TIME-WAIT阶段等待2MSL。（TIME-WAIT出现在主动关闭连接的一方）

## 为什么“握手”是三次，“挥手”却要四次？

TCP建立连接时之所以只需要"三次握手"，是因为在第二次"握手"过程中，服务器端发送给客户端的TCP报文是以SYN与ACK作为标志位的。SYN是请求连接标志，表示服务器端同意建立连接；ACK是确认报文，表示告诉客户端，服务器端收到了它的请求报文。即SYN建立连接报文与ACK确认接收报文是在同一次"握手"当中传输的，所以"三次握手"不多也不少，正好让双方明确彼此信息互通。

TCP释放连接时之所以需要“四次挥手”,是因为FIN释放连接报文与ACK确认接收报文是分别由第二次和第三次"握手"传输的。为何建立连接时一起传输，释放连接时却要分开传输？建立连接时，被动方服务器端结束CLOSED阶段进入“握手”阶段并不需要任何准备，可以直接返回SYN和ACK报文，开始建立连接。释放连接时，被动方服务器，突然收到主动方客户端释放连接的请求时并不能立即释放连接，因为还有必要的数据需要处理，所以服务器先返回ACK确认收到报文，经过CLOSE-WAIT阶段准备好释放连接之后，才能返回FIN释放连接报文。

## 为什么客户端在TIME-WAIT阶段要等2MSL?

为的是确认服务器端是否收到客户端发出的ACK确认报文。当客户端发出最后的ACK确认报文时，并不能确定服务器端能够收到该段报文。所以客户端在发送完ACK确认报文之后，会设置一个时长为2MSL的计时器。MSL指的是Maximum Segment Lifetime：一段TCP报文在传输过程中的最大生命周期。2MSL即是服务器端发出为FIN报文和客户端发出的ACK确认报文所能保持有效的最大时长。服务器端在1MSL内没有收到客户端发出的ACK确认报文，就会再次向客户端发出FIN报文；如果客户端在2MSL内，再次收到了来自服务器端的FIN报文，说明服务器端由于各种原因没有接收到客户端发出的ACK确认报文。客户端再次向服务器端发出ACK确认报文，计时器重置，重新开始2MSL的计时；否则客户端在2MSL内没有再次收到来自服务器端的FIN报文，说明服务器端正常接收了ACK确认报文，客户端可以进入CLOSED阶段，完成“四次挥手”。所以，客户端要经历时长为2SML的TIME-WAIT阶段；这也是为什么客户端比服务器端晚进入CLOSED阶段的原因。

MSL是Maximum-Segment-Lifetime英文的缩写，中文可以译为“报文最大生存时间”，他是任何报文在网络上存在的最长时间，超过这个时间报文将被丢弃。因为tcp报文（segment）是ip数据报（datagram）的数据部分，而ip头中有一个TTL域，TTL是time-to-live的缩写，中文可以译为“生存时间”，这个生存时间是由源主机设置初始值但不是存的具体时间，而是存储了一个ip数据报可以经过的最大路由数，每经过一个处理他的路由器此值就减1，当此值为0则数据报将被丢弃，同时发送ICMP报文通知源主机。RFC-793中规定MSL为2分钟，实际应用中常用的是30秒，1分钟和2分钟等。比如在linux上，这个限制时间无法调整，写死为1分钟了，定义在include/net/tcp.h

## SYN和FIN占一个序列号的原因：

TCP是一个支持可靠数据传输的网络协议，怎么做到可靠传输？主要是靠“确认”这个步骤来做到的，也就是ACK号。用ACK号来表达我这边已经收到了你传过来的东西，注意这里的东西是一个广义的概念，包含了数据和命令两种内容。在可靠的TCP传输过程中，用于建立和释放这个可靠通道的东西就是命令。这两和过程都是需要双方主动参与和确认回复的，当一方说我想开始一段可靠连接，并且给予了相关自己的情况数据，另外一方，则需要对这一命令进行确认，确认只能通过确认号来做。这个事情在断开连接的时候也是一样的。我们知道TCP除了SYN和FIN还有其它的标志，为什么他们不需要占用一个序列号呢？首先，ACK就不用说了，它本身就是为了确认这个动作而生的，如果再给它一个序列号，就意味着还要给这一序列号进行ACK，这是一个很奇怪的事情。PSH,URG是一个属性，一个附加在一段数据传输上的属性，它不属于命令或数据，RST比较特殊，它似乎是一个命令，但是基本上如果需要用这个命令的时候，TCP的可靠性也基本没有了，所以对这个命令进行确认已经无意义啦。总的来说，TCP是用“确认”这个手段来保证可靠的，在TCP整个过程中，我们需要确认SYN,FIN,两个命令，以及数据传输，这样才能保证可靠。

## 序列号和确认号

序列号（Sequence Number）代表的是发出的并且被对方确认好的数据长度

确认号（Acknowledgment Number）代表的是自己接受到的并且确认好的数据长度

## TIME_WAIT有关的参数

### TIME_WAIT存在的两个原因

1. 防止上一个TCP连接的延迟的数据包（发起关闭，但关闭没完成），被接收后，影响到新的TCP连接。（唯一连接确认方式为四元组：源IP地址、目的IP地址、源端口、目的端口），包的序列号也有一定作用，会减少问题发生的几率，但无法完全避免。尤其是较大接收windows size的快速（回收）连接。

   示例：SEQ=3的报文在上一次TCP连接中发生超时重传(重传的包服务器收到了，但是超时的包还在网络中)。在原的连接中断后，又重新创建了一个和原来连接四元组完全相同的连接，此时上次超时的SEQ=3的报文经过一段时间后又达到了远端主机(并且恰好本次连接的序列号SEQ也到3了)，因为是上次连接的数据包所以肯定不是当前连接所需要的数据，于是就对当前连接造成了影响。

   ![image][2]

2. 另外一个作用是，当最后一个ACK丢失时，远程连接进入LAST-ACK状态，它可以确保远程已经关闭当前TCP连接。如果没有TIME-WAIT状态，当远程仍认为这个连接是有效的（没有收到ACK或RST），则会继续与其通讯，导致这个连接会被重新打开。当远程收到一个SYN 时，会回复一个RST包，因为这SEQ不对，那么新的连接将无法建立成功，报错终止。(如果远程因为最后一个ACK包丢失，导致停留在LAST-ACK状态，将影响新建立具有相同四元组的TCP连接。)

   ![image][3]

   注意：RST(reset-重置)：重置连接标志，用于重置由于主机崩溃或其他原因而出现错误的连接。或者用于拒绝非法的报文段和拒绝连接请求。

在机器上执行以下任意命令，就可以看到这个此机器上的TIME_WAIT的数量

```bash
[root@server01 ~]# netstat -apn | grep TIME_WAIT | wc -l
0
[root@server01 ~]# ss -ant  | grep TIME-WAIT | wc -l
0
[root@server01 ~]# cat /proc/net/sockstat
sockets: used 171
TCP: inuse 4 orphan 0 tw 0 alloc 8 mem 1
UDP: inuse 6 mem 0
UDPLITE: inuse 0
RAW: inuse 0
FRAG: inuse 0 memory 0
[root@server01 ~]#
```

### net.ipv4.tcp_timestamps

#### 开启时间戳选项

要开启tcp这个选项，需要将内核参数`net.ipv4.tcp_timestamps`设置成1(`sysctl -w net.ipv4.tcp_timestamps=1`)，可以通过以下两个命令来查看当前的内核参数设置:

```bash
$ sysctl net.ipv4.tcp_timestamps
net.ipv4.tcp_timestamps = 1
$ cat /proc/sys/net/ipv4/tcp_timestamps
1
```

#### 实现细节

假设A与B建立连接，在建立连接伊始A发送的SYN包中，就会带上时间戳字段TSval，B在回复SYN/ACK中会将A发来的TSval放在TSecr(echo reply)中，并同时带上新的时间戳字段TSval。

#### 实际抓包

在使用tcpdump抓包中，可以很清晰地看到选项字段里带的TSval和TSecr：

```bash
[root@server01 home]# tcpdump -nn -i enp0s8 port 8085
tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
listening on enp0s8, link-type EN10MB (Ethernet), capture size 262144 bytes
09:20:13.558754 IP 192.168.56.112.55326 > 192.168.56.110.8085: Flags [S], seq 3940056058, win 29200, options [mss 1460,sackOK,TS val 4294787354 ecr 0,nop,wscale 7], length 0
09:20:13.558772 IP 192.168.56.110.8085 > 192.168.56.112.55326: Flags [S.], seq 208694481, ack 3940056059, win 28960, options [mss 1460,sackOK,TS val 431562 ecr 4294787354,nop,wscale 7], length 0
09:20:13.558953 IP 192.168.56.112.55326 > 192.168.56.110.8085: Flags [.], ack 1, win 229, options [nop,nop,TS val 4294787354 ecr 431562], length 0
09:20:13.559045 IP 192.168.56.112.55326 > 192.168.56.110.8085: Flags [P.], seq 1:181, ack 1, win 229, options [nop,nop,TS val 4294787354 ecr 431562], length 180
09:20:13.559051 IP 192.168.56.110.8085 > 192.168.56.112.55326: Flags [.], ack 181, win 235, options [nop,nop,TS val 431562 ecr 4294787354], length 0
09:20:13.560312 IP 192.168.56.110.8085 > 192.168.56.112.55326: Flags [P.], seq 1:18, ack 181, win 235, options [nop,nop,TS val 431564 ecr 4294787354], length 17
09:20:13.560488 IP 192.168.56.112.55326 > 192.168.56.110.8085: Flags [.], ack 18, win 229, options [nop,nop,TS val 4294787356 ecr 431564], length 0
09:20:13.560533 IP 192.168.56.110.8085 > 192.168.56.112.55326: Flags [P.], seq 18:284, ack 181, win 235, options [nop,nop,TS val 431564 ecr 4294787356], length 266
09:20:13.560667 IP 192.168.56.112.55326 > 192.168.56.110.8085: Flags [.], ack 284, win 237, options [nop,nop,TS val 4294787356 ecr 431564], length 0
09:20:13.560673 IP 192.168.56.110.8085 > 192.168.56.112.55326: Flags [P.], seq 284:319, ack 181, win 235, options [nop,nop,TS val 431564 ecr 4294787356], length 35
09:20:13.560755 IP 192.168.56.110.8085 > 192.168.56.112.55326: Flags [F.], seq 319, ack 181, win 235, options [nop,nop,TS val 431564 ecr 4294787356], length 0
09:20:13.560912 IP 192.168.56.112.55326 > 192.168.56.110.8085: Flags [.], ack 319, win 237, options [nop,nop,TS val 4294787356 ecr 431564], length 0
09:20:13.561004 IP 192.168.56.112.55326 > 192.168.56.110.8085: Flags [F.], seq 181, ack 320, win 237, options [nop,nop,TS val 4294787356 ecr 431564], length 0
09:20:13.561011 IP 192.168.56.110.8085 > 192.168.56.112.55326: Flags [.], ack 182, win 235, options [nop,nop,TS val 431564 ecr 4294787356], length 0

```

而在没有开启时间戳选项(B没有开启)的交互中，A收到B的回包不带时间戳选项，后续再发的包，也就不再带时间戳选项了：

```bash
[root@server01 home]# sysctl -w net.ipv4.tcp_timestamps=0
net.ipv4.tcp_timestamps = 0
[root@server01 home]# nohup python3 tcp_app.py >log 2>&1 &
[3] 1127
[root@server01 home]# tcpdump -nn -i enp0s8 port 8085
tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
listening on enp0s8, link-type EN10MB (Ethernet), capture size 262144 bytes
09:23:27.983691 IP 192.168.56.112.55328 > 192.168.56.110.8085: Flags [S], seq 866788712, win 29200, options [mss 1460,sackOK,TS val 14483 ecr 0,nop,wscale 7], length 0
09:23:27.983711 IP 192.168.56.110.8085 > 192.168.56.112.55328: Flags [S.], seq 4050425417, ack 866788713, win 29200, options [mss 1460,nop,nop,sackOK,nop,wscale 7], length 0
09:23:27.983902 IP 192.168.56.112.55328 > 192.168.56.110.8085: Flags [.], ack 1, win 229, length 0
09:23:27.983997 IP 192.168.56.112.55328 > 192.168.56.110.8085: Flags [P.], seq 1:181, ack 1, win 229, length 180
09:23:27.984003 IP 192.168.56.110.8085 > 192.168.56.112.55328: Flags [.], ack 181, win 237, length 0
09:23:27.985356 IP 192.168.56.110.8085 > 192.168.56.112.55328: Flags [P.], seq 1:18, ack 181, win 237, length 17
09:23:27.985544 IP 192.168.56.112.55328 > 192.168.56.110.8085: Flags [.], ack 18, win 229, length 0
09:23:27.985576 IP 192.168.56.110.8085 > 192.168.56.112.55328: Flags [P.], seq 18:284, ack 181, win 237, length 266
09:23:27.985710 IP 192.168.56.112.55328 > 192.168.56.110.8085: Flags [.], ack 284, win 237, length 0
09:23:27.985717 IP 192.168.56.110.8085 > 192.168.56.112.55328: Flags [P.], seq 284:319, ack 181, win 237, length 35
09:23:27.985842 IP 192.168.56.112.55328 > 192.168.56.110.8085: Flags [.], ack 319, win 237, length 0
09:23:27.985869 IP 192.168.56.110.8085 > 192.168.56.112.55328: Flags [F.], seq 319, ack 181, win 237, length 0
09:23:27.986113 IP 192.168.56.112.55328 > 192.168.56.110.8085: Flags [F.], seq 181, ack 320, win 237, length 0
09:23:27.986121 IP 192.168.56.110.8085 > 192.168.56.112.55328: Flags [.], ack 182, win 237, length 0
```

#### 注意点

1. TSval并非真正的时间戳，而是由时间戳依据一定算法算出来的一个值，与时间戳有同等的特性，即随时间单调递增；
2. 只有在TCP连接的客户端和服务器端都打开`net.ipv4.tcp_timestamps`，时间戳选项才会生效;
3. 关于TSecr，一说只有在ACK标志的包中才会带，由于实际中基本没有不带ACK的包(除了第一个sync)，所以无法验证;
4. 如果一个ACK包是回复之前收到的多个数据包，则此时的TSecr取值算法可参考[此](https://www.freesoft.org/CIE/RFC/1323/10.htm)，一般使用最早收到的那个TSval。

#### 作用

##### 精确计算RTT(Round-Trip Time)

在没有时间戳时计算RTT使用的方法是在包发送时记录下时间，RTT为收到ACK的时间减去发送时记录的时间。这种方法在出现丢失重传时，会导致RTT计算出现偏差，因为不确定ACK的回包是因为收到了最开始发的包，还是收到了重传后的包。而时间戳选项可以很方便的使用TSecr来计算精准的RTT，当然，由于TSval并非真正的时间戳，所以计算时并非直接相减，而是使用相应的算法计算出RTT。

##### PAWS(Protection Against Wrapped Sequence numbers)

TCP的头部信息中序列号占用4个字节，即每传输4G的数据之后，序列号又要从头开始了(考虑到开始时随机选取的序列号，这个数字一般比4G小)。考虑在一个高速网络中，某一个数据包A发生了超时重传，过了一段时间，此时序列号已经过了一轮又回到A了，之前丢失的包如果此时被收到就会被当成合法的包加以使用，这就是PAWS要解决的问题。在添加了时间戳的选项的包中，PAWS在处理逻辑中添加了一条如下规则：如果收到的包的TSval小于最近一次收到的时间戳，则认为是不合法的，这就保证之前的包不会被当成合法的包。(这中间还有一些细节的处理，比如何时更新最近一次收到的时间戳，对于重传情况的处理等，可以参考具体的RFC文档)

注意：当使用高速网络时，在一次TCP链接的数据传送中序号极可能被重复使用。例如，当使用1.5Mbit/s的速度发送报文段时，序号重复要6小时以上。但若用2.5Gbit/s的速率发送报文段，则不到14秒钟序号就会重复。

### net.ipv4.tcp_tw_reuse

#### 现象

众所周知，可用的端口号有65535个，而实际能用的端口数还受`net.ipv4.ip_local_port_range`和`net.ipv4.ip_local_reserved_ports`影响。为了探究reuse选项对TIME_WAIT的影响，可以进行如下实验：

1. 在server01中启动一个web服务；
2. 将server03的可用端口改成10个；
3. 并关闭server03的reuse选项；
4. 接下来从server03向server01连续发起请求
5. 可以发现很快就会报错：curl: (7) Failed connect to 192.168.56.110:8085/getapp; Cannot assign requested address

这个步骤有时能一直请求，需要抓包看一下主动断开的是哪一方，如果是服务端主动断开，则TIME_WAIT在服务端，所以你的请求会持续成功

```bash
[root@server01 home]# nohup python3 tcp_app.py >console.log 2>&1 &
[1] 3277
```

```bash
[root@server03 entrypoint.d]# sysctl net.ipv4.ip_local_port_range
net.ipv4.ip_local_port_range = 32768    60999
[root@server03 entrypoint.d]# sysctl -w net.ipv4.ip_local_port_range="34000 34009"
net.ipv4.ip_local_port_range = 34000 34009
[root@server03 entrypoint.d]# sysctl -w net.ipv4.tcp_tw_reuse=0
net.ipv4.tcp_tw_reuse = 0
[root@server03 entrypoint.d]#  for ((i=0;i<1000;i++)); do if [ "$(curl -sL -w '%{http_code}' 192.168.56.110:8085/getapp -o /dev/null)" = "200" ]; then echo "Success"; else echo "Fail"; fi; done;
Success
Success
Success
Success
Success
Success
Success
Success
Success
Success
Success
Success
Success
Success
Success
Success
Success
Success
Success
Success
Success
Success
Success
Success
Success
Success
Success
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
......
```

接下来打开reuse再试一次：这次出现了很奇怪的现象，发现成功发起了10次请求之后与之前一样开始报错，但是在大约过了1秒左右，又开始能成功请求10个，然后继续报错，如此往复。这个现象与之前网上查阅到的 “如果开启reuse，那么TIME_WAIT将在1秒之后重用” 这个说法很吻合。以上实验的基础是服务端和客户端都打开了tcp_timestamps这个选项，如果某一方关闭timestamps，reuse还能起作用吗？从结果上看，如果关闭了timestamps选项，则reuse也不起作用了，与没有打开reuse现象一样，即在前10次成功请求之后的请求全都报错了。

```bash
[root@server03 entrypoint.d]# sysctl -w net.ipv4.tcp_tw_reuse=1
net.ipv4.tcp_tw_reuse = 1
[root@server03 entrypoint.d]#  for ((i=0;i<1000;i++)); do if [ "$(curl -sL -w '%{http_code}' 192.168.56.110:8085/getapp -o /dev/null)" = "200" ]; then echo "Success"; else echo "Fail"; fi; done;
Success
Success
Success
Success
Success
Success
Success
Success
Success
Success
Success
Success
Success
Success
Success
Success
Success
Success
Success
Success
Success
Success
Success
Success
Success
Success
Success
Success
Success
Success
Success
Success
Success
Success
Success
Success
Success
Success
Success
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Success
Success
Success
Success
Success
Success
Success
Success
Success
Success
Success
Success
Success
Success
Success
Success
Success
Success
Success
Success
Success
Success
Success
Success
Success
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Success
Success
Success
Success
Success
Success
Success
Success
Success
Success
Success
Success
Success
Success
Success
Success
Success
Success
Success
Success
Success
Success
Success
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
```

#### 代码

tcp相关的内核代码错综复杂，目前还无法从头到尾梳理一遍，只能从现象上找到对应的代码来佐证，所以我们从这个报错入手。Cannot assign requested address这个在内核代码中并没有找到对应的字符串，只是从众多的注释上看，可以知道EADDRNOTAVAIL这个错误码就代表了这个报错。另外一方面，搜索tcp_tw_reuse这个关键字，我们可以找到以下代码：

net/ipv4/tcp_ipv4.c

```c
int tcp_twsk_unique(struct sock *sk, struct sock *sktw, void *twp)
{
	const struct tcp_timewait_sock *tcptw = tcp_twsk(sktw);
	struct tcp_sock *tp = tcp_sk(sk);

	/* With PAWS, it is safe from the viewpoint
	   of data integrity. Even without PAWS it is safe provided sequence
	   spaces do not overlap i.e. at data rates <= 80Mbit/sec.

	   Actually, the idea is close to VJ's one, only timestamp cache is
	   held not per host, but per port pair and TW bucket is used as state
	   holder.

	   If TW bucket has been already destroyed we fall back to VJ's scheme
	   and use initial timestamp retrieved from peer table.
	 */
	if (tcptw->tw_ts_recent_stamp &&
	    (twp == NULL || (sysctl_tcp_tw_reuse &&
			     get_seconds() - tcptw->tw_ts_recent_stamp > 1))) {
		tp->write_seq = tcptw->tw_snd_nxt + 65535 + 2;
		if (tp->write_seq == 0)
			tp->write_seq = 1;
		tp->rx_opt.ts_recent	   = tcptw->tw_ts_recent;
		tp->rx_opt.ts_recent_stamp = tcptw->tw_ts_recent_stamp;
		sock_hold(sktw);
		return 1;
	}

	return 0;
}
```

而此函数的调用处，在返回0时会返回`EADDRNOTAVAIL`错误码，所以这块可以猜到应该就是判断TIME_WAIT是否可以重用的代码

net/ipv4/inet_hashtables.c

```c
static int __inet_check_established(struct inet_timewait_death_row *death_row,
				    struct sock *sk, __u16 lport,
				    struct inet_timewait_sock **twp)
{
	// skip something
	/* Check TIME-WAIT sockets first. */
	sk_nulls_for_each(sk2, node, &head->twchain) {
		tw = inet_twsk(sk2);

		if (INET_TW_MATCH(sk2, net, hash, acookie,
					saddr, daddr, ports, dif)) {
			if (twsk_unique(sk, sk2, twp)) // 这个函数也就是上面看到的tcp_twsk_unique函数
				goto unique;
			else
				goto not_unique;
		}
	}
	// skip something
unique:
	// skip something
	return 0;

not_unique:
	spin_unlock(lock);
	return -EADDRNOTAVAIL;
}
```

回过头来，我们看一下`tcp_twsk_unique`返回1的条件：

**`tcptw->tw_ts_recent_stamp`**： 搜索这个变量的赋值情况，都是在saw_tstamp为真是才会赋值，猜想这个saw_tstamp即为是否打开了timestamps选项，所以这个`tcptw->tw_ts_recent_stamp`只有在打开timestamps才会有值；

**`twp == NULL || (sysctl_tcp_tw_reuse && get_seconds() - tcptw->tw_ts_recent_stamp > 1)`**，这个twp不考虑是什么，如果要让sysctl_tcp_tw_reuse选项发生作用，这个twp必须不为空，所以这个条件的意思就是如果开启了reuse选项，并且当前时间(get_seconds)是在tw_ts_recent_stamp这个时间一秒之后，则为真；这是否与之前我们看到的现象1秒之后所有请求成功了相关呢？

以上是从现象再搜索代码猜测的结果，细节上应该还有些出入，但大概的情况应该也是这样了。

#### 优化TIME_WAIT

1. 启用net.ipv4.tcp_tw_reuse后，如果新的时间戳，比以前存储的时间戳更大，那么linux将会从TIME-WAIT状态的存活连接中，选取一个，重新分配给新的连接出去的TCP连接。

   个人理解新的时间戳和以前存储的时间戳的含义：

   新的时间戳指：当前SYN请求发送时的TSval值

   以前存储的时间戳指：从远端服务收到的最后一次ACK包中的TSecr值(即回复客户端关闭socket请求的FIN包的ACK)

   由于客户端处于TIME_WAIT状态所以可以保证客户端的FIN服务器已经收到也可以保证客户端已经收到了服务器发来的FIN，并且由于当前要发送SYN请求发送时的TSval值大于以前存储的时间戳就能保证此次SYN是新的连接请求，于是复用TIME-WAIT状态的存活连接能保证通信的安全性(唯一有可能存在的问题是客户端回复服务器发来的FIN的ACK包有可能丢)。也就是说，TIME_WAIT状态下其实客服端和服务器的socket都已经是close状态了，只有当是SYN请求、并且时间戳大于以前存储的时间戳的数据包，并且在tcp_tw_reuse开启的情况下才能使用连接复用机制将数据包传输到远端，如果不是SYN请求、或者时间戳小于以前存储的时间戳的数据包，那么会认为是丢包导致的重传而由于此时socket已经关闭所以不会将数据包传输出去。

2. 当reuse开启后、客户端复用TIME_WAIT状态下的连接连接远端服务可能出现的情况，

   - 当处于LAST_ACK状态下的远端服务器正常收到了客户端(同四元组)的最后一个ACK(即服务器发送给客户端的socket关闭请求:FIN的ACK)的话，因为服务器此时已经关闭了该四元组对应的连接，那么在客户端复用此连接发送SYN不会有问题，服务器还是会通过正常的三次握手建立连接。
   - 当远端服务器由于最后一个ACK丢失而处于LAST_ACK状态的话，此时客户端发起了对同一四元组的新连接，并且是复用之前同一四元组的TIME_WAIT状态的连接；新连接发送的SYN会被远端服务器忽略并且不会有RST应答(不理解，网上的解释是由于timestamps的原因，个人理解：当双方关闭timestamps选项的时候服务器会有RST应答；而当双方都开启timestamps的时候服务器会忽略这个SYN)，由于远端服务器没有收到ACK所以会重传FIN，因为客户端此时处于SYN_SENT状态所以会回复RST让服务器端跳过LAST_ACK状态。而客户端最初的SYN因为没有应答会在一秒后重新发送，此时的服务器由于同一四元组的socket已经关闭了于是可以通过正常的三次握手建立连接。

#### 总结

对于`net.ipv4.tcp_tw_reuse`，其作用是在TIME_WAIT状态1秒之后即可重用端口，达到快速回收TIME_WAIT端口的作用，避免出现无端口可用的情况，但是reuse的生效条件是通信双方都开启了timestamps选项。

### net.ipv4.tcp_tw_recycle

#### 实验1：

1. 开启net.ipv4.tcp_tw_reuse关闭net.ipv4.tcp_tw_recycle
2. 由于在TIME_WAIT状态1秒之后即可重用端口，达到快速回收TIME_WAIT端口的作用，所有在一秒之后又会有成功的请求

```bash
[root@server03 entrypoint.d]# sysctl -w net.ipv4.tcp_tw_reuse=1                                                                                                         net.ipv4.tcp_tw_reuse = 1
[root@server03 entrypoint.d]# sysctl -w net.ipv4.tcp_tw_recycle=1
net.ipv4.tcp_tw_recycle = 1
[root@server03 entrypoint.d]#  for ((i=0;i<1000;i++)); do if [ "$(curl -sL -w '%{http_code}' 192.168.56.110:8085/getapp -o /dev/null)" = "200" ]; then echo "Success"; else echo "Fail"; fi; done;
Success
Success
Success
Success
Success
Success
Success
Success
Success
Success
Success
Success
Success
Success
Success
Success
Success
Success
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
......
```



```bash
[root@server03 ~]# ss -nat | grep TIME-WAIT
TIME-WAIT  0      0      192.168.56.112:34005              192.168.56.110:8085
TIME-WAIT  0      0      192.168.56.112:34009              192.168.56.110:8085
TIME-WAIT  0      0      192.168.56.112:34004              192.168.56.110:8085
TIME-WAIT  0      0      192.168.56.112:34001              192.168.56.110:8085
TIME-WAIT  0      0      192.168.56.112:34007              192.168.56.110:8085
TIME-WAIT  0      0      192.168.56.112:34003              192.168.56.110:8085
TIME-WAIT  0      0      192.168.56.112:34000              192.168.56.110:8085
TIME-WAIT  0      0      192.168.56.112:34008              192.168.56.110:8085
TIME-WAIT  0      0      192.168.56.112:34006              192.168.56.110:8085
TIME-WAIT  0      0      192.168.56.112:34002              192.168.56.110:8085
[root@server03 ~]# ss -nat | grep TIME-WAIT
TIME-WAIT  0      0      192.168.56.112:34005              192.168.56.110:8085
TIME-WAIT  0      0      192.168.56.112:34009              192.168.56.110:8085
TIME-WAIT  0      0      192.168.56.112:34004              192.168.56.110:8085
TIME-WAIT  0      0      192.168.56.112:34001              192.168.56.110:8085
TIME-WAIT  0      0      192.168.56.112:34007              192.168.56.110:8085
TIME-WAIT  0      0      192.168.56.112:34003              192.168.56.110:8085
TIME-WAIT  0      0      192.168.56.112:34000              192.168.56.110:8085
TIME-WAIT  0      0      192.168.56.112:34008              192.168.56.110:8085
TIME-WAIT  0      0      192.168.56.112:34006              192.168.56.110:8085
TIME-WAIT  0      0      192.168.56.112:34002              192.168.56.110:8085
[root@server03 ~]# ss -nat | grep TIME-WAIT
TIME-WAIT  0      0      192.168.56.112:34005              192.168.56.110:8085
TIME-WAIT  0      0      192.168.56.112:34009              192.168.56.110:8085
TIME-WAIT  0      0      192.168.56.112:34004              192.168.56.110:8085
TIME-WAIT  0      0      192.168.56.112:34001              192.168.56.110:8085
TIME-WAIT  0      0      192.168.56.112:34007              192.168.56.110:8085
TIME-WAIT  0      0      192.168.56.112:34003              192.168.56.110:8085
TIME-WAIT  0      0      192.168.56.112:34000              192.168.56.110:8085
TIME-WAIT  0      0      192.168.56.112:34008              192.168.56.110:8085
TIME-WAIT  0      0      192.168.56.112:34006              192.168.56.110:8085
TIME-WAIT  0      0      192.168.56.112:34002              192.168.56.110:8085
[root@server03 ~]# ss -nat | grep TIME-WAIT
TIME-WAIT  0      0      192.168.56.112:34005              192.168.56.110:8085
TIME-WAIT  0      0      192.168.56.112:34009              192.168.56.110:8085
TIME-WAIT  0      0      192.168.56.112:34004              192.168.56.110:8085
TIME-WAIT  0      0      192.168.56.112:34001              192.168.56.110:8085
TIME-WAIT  0      0      192.168.56.112:34007              192.168.56.110:8085
TIME-WAIT  0      0      192.168.56.112:34003              192.168.56.110:8085
TIME-WAIT  0      0      192.168.56.112:34000              192.168.56.110:8085
TIME-WAIT  0      0      192.168.56.112:34008              192.168.56.110:8085
TIME-WAIT  0      0      192.168.56.112:34006              192.168.56.110:8085
TIME-WAIT  0      0      192.168.56.112:34002              192.168.56.110:8085
[root@server03 ~]# ss -nat | grep TIME-WAIT
TIME-WAIT  0      0      192.168.56.112:34005              192.168.56.110:8085
TIME-WAIT  0      0      192.168.56.112:34009              192.168.56.110:8085
TIME-WAIT  0      0      192.168.56.112:34004              192.168.56.110:8085
TIME-WAIT  0      0      192.168.56.112:34001              192.168.56.110:8085
TIME-WAIT  0      0      192.168.56.112:34007              192.168.56.110:8085
TIME-WAIT  0      0      192.168.56.112:34003              192.168.56.110:8085
TIME-WAIT  0      0      192.168.56.112:34000              192.168.56.110:8085
TIME-WAIT  0      0      192.168.56.112:34008              192.168.56.110:8085
TIME-WAIT  0      0      192.168.56.112:34006              192.168.56.110:8085
TIME-WAIT  0      0      192.168.56.112:34002              192.168.56.110:8085
[root@server03 ~]# ss -nat | grep TIME-WAIT
TIME-WAIT  0      0      192.168.56.112:34005              192.168.56.110:8085
TIME-WAIT  0      0      192.168.56.112:34009              192.168.56.110:8085
TIME-WAIT  0      0      192.168.56.112:34004              192.168.56.110:8085
TIME-WAIT  0      0      192.168.56.112:34001              192.168.56.110:8085
TIME-WAIT  0      0      192.168.56.112:34007              192.168.56.110:8085
TIME-WAIT  0      0      192.168.56.112:34003              192.168.56.110:8085
TIME-WAIT  0      0      192.168.56.112:34000              192.168.56.110:8085
TIME-WAIT  0      0      192.168.56.112:34008              192.168.56.110:8085
TIME-WAIT  0      0      192.168.56.112:34006              192.168.56.110:8085
TIME-WAIT  0      0      192.168.56.112:34002              192.168.56.110:8085
[root@server03 ~]# ss -nat | grep TIME-WAIT
TIME-WAIT  0      0      192.168.56.112:34005              192.168.56.110:8085
TIME-WAIT  0      0      192.168.56.112:34009              192.168.56.110:8085
TIME-WAIT  0      0      192.168.56.112:34004              192.168.56.110:8085
TIME-WAIT  0      0      192.168.56.112:34001              192.168.56.110:8085
TIME-WAIT  0      0      192.168.56.112:34007              192.168.56.110:8085
TIME-WAIT  0      0      192.168.56.112:34003              192.168.56.110:8085
TIME-WAIT  0      0      192.168.56.112:34000              192.168.56.110:8085
TIME-WAIT  0      0      192.168.56.112:34008              192.168.56.110:8085
TIME-WAIT  0      0      192.168.56.112:34006              192.168.56.110:8085
TIME-WAIT  0      0      192.168.56.112:34002              192.168.56.110:8085
[root@server03 ~]# ss -nat | grep TIME-WAIT
[root@server03 ~]#

```

#### 实验2：

1. 关闭net.ipv4.tcp_tw_reuse开启net.ipv4.tcp_tw_recycle
2. 由于开启了recycle选项，同样可以达到可以快速回收TIME_WAIT的端口的效果，他与reuse的不同在于reuse在ss中还能看到TIME_WAIT，只是可以复用这些端口，而recycle是直接回收了，使用ss可能已经看不到了。
3. 如果关闭timestamps：发现这些TIME_WAIT又回来了，所以也是在timestamps打开的情况下recycle才能生效！

```bash
[root@server03 entrypoint.d]# sysctl -w net.ipv4.tcp_tw_reuse=0
net.ipv4.tcp_tw_reuse = 0
[root@server03 entrypoint.d]# sysctl -w net.ipv4.tcp_tw_recycle=1
net.ipv4.tcp_tw_recycle = 1
[root@server03 entrypoint.d]# sysctl -w net.ipv4.ip_local_port_range="34000 34009"
net.ipv4.ip_local_port_range = 34000 34009
[root@server03 entrypoint.d]#  for ((i=0;i<1000;i++)); do if [ "$(curl -sL -w '%{http_code}' 192.168.56.110:8085/getapp -o /dev/null)" = "200" ]; then echo "Success"; else echo "Fail"; fi; done;
Success
Success
Success
Success
Success
Success
Success
Success
Success
Success
Success
Success
Success
Success
Success
Success
Success
Success
Success
Success
Success
Success
Success
Success
Success
Success
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Success
Success
Success
Success
Success
Success
Success
Success
Success
Success
Success
Success
Success
Success
Success
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Success
Success
Success
Success
Success
Success
Success
Success
Success
Success
Success
Success
Success
Success
Success
Success
Success
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Success
Success
Success
Success
Success
Success
Success
Success
Success
Success
Success
Success
Success
Success
Success
Success
Success
Success
Success
Success
Success
Success
Success
Success
Success
Success
Success
Success
Success
Success
Success
Success
Success
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Success
Success
Success
Success
Success
Success
Success
Success
Success
Success
Success
Success
Success
Success
Success
Success
Success
Success
Success
Success
Success
Success
Success
Success
Success
Success
Success
Success
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Success
Success
Success
Success
Success
Success
Success
Success
Success
Success
Success
Success
Success
Success
Success
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
Fail
[root@server03 entrypoint.d]#
```



```bash
[root@server03 ~]# ss -nat | grep TIME-WAIT
TIME-WAIT  0      0      192.168.56.112:34005              192.168.56.110:8085
TIME-WAIT  0      0      192.168.56.112:34009              192.168.56.110:8085
TIME-WAIT  0      0      192.168.56.112:34004              192.168.56.110:8085
TIME-WAIT  0      0      192.168.56.112:34001              192.168.56.110:8085
TIME-WAIT  0      0      192.168.56.112:34007              192.168.56.110:8085
TIME-WAIT  0      0      192.168.56.112:34003              192.168.56.110:8085
TIME-WAIT  0      0      192.168.56.112:34000              192.168.56.110:8085
TIME-WAIT  0      0      192.168.56.112:34008              192.168.56.110:8085
TIME-WAIT  0      0      192.168.56.112:34006              192.168.56.110:8085
TIME-WAIT  0      0      192.168.56.112:34002              192.168.56.110:8085
[root@server03 ~]# ss -nat | grep TIME-WAIT
TIME-WAIT  0      0      192.168.56.112:34005              192.168.56.110:8085
TIME-WAIT  0      0      192.168.56.112:34009              192.168.56.110:8085
TIME-WAIT  0      0      192.168.56.112:34004              192.168.56.110:8085
TIME-WAIT  0      0      192.168.56.112:34001              192.168.56.110:8085
TIME-WAIT  0      0      192.168.56.112:34007              192.168.56.110:8085
TIME-WAIT  0      0      192.168.56.112:34003              192.168.56.110:8085
TIME-WAIT  0      0      192.168.56.112:34000              192.168.56.110:8085
TIME-WAIT  0      0      192.168.56.112:34008              192.168.56.110:8085
TIME-WAIT  0      0      192.168.56.112:34006              192.168.56.110:8085
TIME-WAIT  0      0      192.168.56.112:34002              192.168.56.110:8085
[root@server03 ~]# ss -nat | grep TIME-WAIT
[root@server03 ~]# ss -nat | grep TIME-WAIT
[root@server03 ~]# ss -nat | grep TIME-WAIT
[root@server03 ~]# ss -nat | grep TIME-WAIT
[root@server03 ~]#
```

#### 代码

net/ipv4/tcp_minisocks.c

```c
/*
 * Move a socket to time-wait or dead fin-wait-2 state.
 */
void tcp_time_wait(struct sock *sk, int state, int timeo)
{
	struct inet_timewait_sock *tw = NULL;
	const struct inet_connection_sock *icsk = inet_csk(sk);
	const struct tcp_sock *tp = tcp_sk(sk);
	int recycle_ok = 0;

	if (tcp_death_row.sysctl_tw_recycle && tp->rx_opt.ts_recent_stamp)
		recycle_ok = icsk->icsk_af_ops->remember_stamp(sk);

	if (tcp_death_row.tw_count < tcp_death_row.sysctl_max_tw_buckets)
		tw = inet_twsk_alloc(sk, state);

	if (tw != NULL) {
		// skip something

		if (recycle_ok) {
			tw->tw_timeout = rto;
		} else {
			tw->tw_timeout = TCP_TIMEWAIT_LEN;
			if (state == TCP_TIME_WAIT)
				timeo = TCP_TIMEWAIT_LEN;
		}

		inet_twsk_schedule(tw, &tcp_death_row, timeo,
				   TCP_TIMEWAIT_LEN);
		inet_twsk_put(tw);
	} else {
		/* Sorry, if we're out of memory, just CLOSE this
		 * socket up.  We've got bigger problems than
		 * non-graceful socket closings.
		 */
		LIMIT_NETDEBUG(KERN_INFO "TCP: time wait bucket table overflow\n");
	}

	tcp_update_metrics(sk);
	tcp_done(sk);
}
```

留意recycle_ok变量，从条件上可以看出，如果开启了recycle选项，并且`tp->rx_opt.ts_recent_stamp`不为空，则recycle_ok为真，继而对应的TIME_WAIT超时时间为rto，否则为TCP_TIMEWAIT_LEN。

这里有三个问题：

1. **tp->rx_opt.ts_recent_stamp**这个变量的值是什么：从代码中搜索，这个值的赋值为之前我们看到的tw_ts_recent_stamp，即开启timestamps选项时收到的最近一个包的时间戳，只有开启了timestamps这个变量才有值
2. **rto**和**TCP_TIMEWAIT_LEN**是多少：TCP_TIMEWAIT_LEN很容易搜索到是60秒，而rto呢？rto为Retransmission TimeOut，即重传超时，他是一个动态计算的值，在网络较好的情况下这个值一般都小于1秒，具体的算法可以查看参考文章。
3. **tcp_v4_remember_stamp**何时返回真：这里有一个`inet_getpeer(inet->daddr, 1)`函数，如果取到了peer信息，则返回真，否则返回0。

上面的代码，我们还能看到另外一个内核选项的作用`net.ipv4.tcp_max_tw_buckets`，当TIME_WAIT超过max_tw_buckets数量时，就不会再转入TIME_WAIT状态，而是报一条overflow的报错，这个报错可以在系统的/var/log/message里看到。

#### 总结

对于`net.ipv4.tcp_tw_recycle`选项，其作用是在将TIME_WAIT的超时时间设置成rto，而非60秒，而rto一般情况下会小于1秒，所以recycle经常能够快速回收处理TIME_WAIT状态的端口。而timestamps同样必须打开recycle才能生效。另外一种不生效的情况是`inet_getpeer`函数无法获取到对应的信息时，recycle也不会生效。

### 如何测试TIME_WAIT的超时时间

使用curl的`--local-port`选项可以大概地看出TIME_WAIT的超时时间

```bash
for ((i=0;i< 1000;i++)); do date; curl --local-port 54539  http://192.168.56.110; sleep 1; done;
```

查看两次正常返回的时间差即为TIME_WAIT状态的超时时间，这个在linux上是宏定义的60秒，无法修改。

### 开启recycle对于NAT网络的影响

对于服务器来说，如果同时开启了recycle和timestamps选项，则会开启一种称之为per-host的PAWS机制。与PAWS机制一样，per-host的PAWS机制是针对同一来源的包，只接收时间戳大于最近一次收到时间戳的包。但对于NAT网络来说，服务器认为的同一个来源，在NAT网关后面可能是多台客户机，而这些机器无法保证在时间戳上的单调递增，从而导致了某些客户机连接失败的情况，这也是作为服务端不推荐打开recycle的原因。

### 总结

在任何情况下打开reuse就够了，recycle不管是做为服务端还是做为客户端都不建议打开，除非你知道这意味着什么。而对于无法控制的服务端并且没有开启timestamps选项，可以通过减少tw_buckets来降低端口不可用的情况，但这相当于去掉了TIME_WAIT机制，带来的副作用可想而知。



[0]:data:image/png;base64,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B6OhRaPRVA2nFQqFY8eOvfjii8eOHSsUCsxBpVL553/+50KhcNdddzU1NQFrMBrFM888c++99/JOYrFYKpW68sorU6lULBbjDEEQsFSCIKAmCAKWXDKZLBaLSqnW1lbCSSaThUJBKfX+978fePrpp2+44Qbf9/+/miAImJsLL7wwFotNT08rpZLJJA0nkUgAxWIxCALqWxAE1ARBwNmuu+66u+66a2RkJAgCVo1HHnkEuP7664MgIJy1a9fatu153muvvWZZFmf49Kc//a1vfeuRRx75yle+EovFWEKJRKJcLmutY7EYxirTXsNpzz//fKFQeL6Gt3X48OEvfelLd911V1NTUwSjIVSr1bvvvpvZrV+//mMf+9iDDz740EMP/d3f/d3mzZtjsRirVTKZBIrFIqElk0mgUCgAhULhhhtu8H3/5ptv/tKXvsQ8pVIpoFgs0ohisZhlWeUaVqxUKhWLxUqlUrFYZHUol8tPP/20ZVlbt25lIcRiMaBUKnG2WCy2efNm3/cfeughllYsFgOmp6cxVr329vaPfvSjX//61x977LEvfOELnZ2d0WiUWRw+fHhgYABYg9EQvvOd77z22mucIxaLXVsTi8WoCYKAuhHUsOTi8ThQKpWCICCc1tZWYGpqSim1efPmcrl84403fu973wuCgHlKJpNPPPGEUuq6666jESWTyUKhUCwWr7rqKupYEATUBEHAOTZv3vz9739/eHj47//+71kFDh486Pv+5s2b165dGwQBoSUSiWKx6Hlea2srZ/v0pz998ODBb3/725///OdZQolEAtBaB0GAYdRccMEFG2uAsbGx8fHxsbExzvHoo4/+zd/8TQSjIRw9epRz7NixY2Bg4CMf+UgsFsM4QyqVAgqFAqG1trYCP/3pT7du3Voulzdv3vy9732P89La2gocOXKEBpVIJACtNSvZtddeCzz55JOsDj/4wQ+AG264gQWSSCSAUqnEOa655ppUKjU9PX3w4EGWUDweB6ampjCMt9LZ2XnHHXcMDAysW7eOs1Wr1UqlsgajIbzxxhucI5fLNTU1bdy48dJLL6UuBTUsudbWVqBUKgVBQDjJZBI4dOjQqVOnrrnmmgMHDlx44YVBEDB/ra2tQLFYDIKARtTa2gqUSqUgCKhjQRBQEwQB59i8ebNlWYcPHy6Xy2vXrqWhTU9PP/3005ZlXXfddUEQsBAuv/xyQGsdBAHn+MhHPlIoFB588MHrrruOpRKLxQDP84IgwDDeyvPPP5/L5Y4dO8Y5KpVKBKMhXHbZZZzjxIkT+/bt+/CHP3zPPfc8+eSTr776KkbN2prp6elXXnmFcGKxGHDq1Kn29vZHHnnEsizOVzKZBIrFIg0qHo8DR44cob698sorwNq1a3kra9euveqqq3zff/rpp2l0Bw8e9H3/2muvXbt2LQskkUgAU1NTvJVPfOITlmU98cQTzz//PEslkUgApVIJwzjbiy+++C//8i99fX233377s88+yyzWYDSESy65hNkdqvmnf/qn9vb2jo6O9vb2K664gjoQ1LAcksnk4cOHlVJXXXUV58v3/RtvvJGa73znO5dcckkQBJyvyy+/fO3ata+88ornebFYjIaTSCSAUqkUBAF17MSJE4Bt20EQ8Fb+8i//8umnn37sscc++MEP0tAeffRR4Prrrw+CgAUSj8cBz/OCIOAcF1544Sc+8YnvfOc73//+97/2ta+xJOLxOFAqlYIgwFj1Xn311eeff/6/aqanp5mDNRiryfM1wMUXX9ze3p5MJqWUyWSyqamJpRUEATVBELAcWltbDx8+rLV+3/vex3nxfb+3t/fw4cO/93u/99prr7388stBEBBOMpk8fPhwoVC49tpraTjvfe97gWKxGAQBdS+o4a1ce+21//AP//DEE08EQUDjmp6efvrppy3L+uAHPxgEAQskHo8DpVIpCALeysc//vHvfOc7Dz744J49eyzLYvHF43Fgenr6tddesywLY/X5xS9+cfTo0SNHjjz//PMvvvgi87QGY1V69dVXn62hJhaLXXHFFX/0R38kpbziiisuvfRSGl1rayvws5/97IYbbuC89PX1Pfnkk7FYrKur66GHHjpy5Aihtbe3Hz58uFQq0YgSiQQwPT3t+75lWaxY7e3tiUSiVCo9//zz7e3tNKjHHnsMuO666yzLYuEkEglgamqKWbS2tl577bVPPvnkgw8++OlPf5olkUgkSqXS9PR0IpHAWAVefPHFY8eOvfTSS0qpYrFYrVYJYQ2GAdM1zz77LDUXX3zxFVdcceWVV15++eWxWOzKK69kcQQ1LIf3vve9wJEjR4IgYP527tz52GOPrV27dmho6PDhww899NCRI0eCICCcdevWAUeOHAmCgEbU2tp65MiR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
[1]:data:image/png;base64,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

[2]:data:image/png;base64,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
[3]:data:image/png;base64,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JJQkukpbCDQOgVWrVkXJhhB2THDGzQEHHGByu3btGpebYguBFBAQCaUAurIUAlEEFi9eDLv4lg3e999/35ENVs5ee+1lvWotWrSIpqAQIVCNCIiEqrHWVObqRmD27NlRvvn0008d3+y6666HHXaY8U1136pKLwTqQ0AkVB9Cui4EGorA559/HiUbbJ0NN9zQCIZf9iE1edNNN21oPoonBKoYAZFQFVeeil45CCxZsiTgG+tYc8YNTLPHHnsY37Rs2bJySq6SCIF0ERAJpYu/cq8+BObOnRuM3OD9+OOPjWD43WmnnQ466CDzrrOOtmesvipWicuJgEionGgrrypDIEo2mDvsCmgEg5Wz7777nnDCCXg333zzKrs3FVcIVAYCIqHKqAeVIlUEli1b5vONk9mxxvENJ/eYHHtIR6rFV+ZCoIoREAlVceWp6A1AgO04sWacM77hVGxHNttvv33//v3N+7Wv6QVpAMaKIgSKQEDvWBFgSbW6EJg5c2ZANnibNWsGwdh8AZbdHHvssXi32GKL6ro1lVYI1AwCIqGaqcrs3ggrbKJkQ8hmm23m+ObQQw8146aQTT+zC6XuXAiUHQGRUNkhV4aNQIB+syjfzJs3z5ENAoe5mTf2XMFGZK6oQkAIlB4BkVDpMVWKJUGAHWuMb9w0AbyrV692fLP77rsPGjSIjrWtt966JDkqESEgBMqPgEio/Jgrx68gwBlLUbIhpG3btm6l54ABA4x7CPxKZHmEgBCocgREQlVegVVV/I8++ijKN7NmzXJkA9Pss88+5t1ggw2q6uZUWCEgBBqCgEioIagpTr0IQC3wjd+Thhejx/ENB4MefvjhZt/Um5oUhIAQqFUEREK1WrMx9wUNMDestNPDOEgtatzAPa1atYJgzO2///7GPRwpHVMsBQkBIZBhBERCmaj8f//737fffvu666575513NviG2R4tMG4gG9xarlnzv0ePHuZlo+gGZ6SIQkAIZAcBkVCN1zWsc9VVV7311lvbbrvtxIkTC7zbOXPmRPlm6dKljm923nnnQw45xLxNmzYtMFmpCQEhIAQCBERCASA14v3www+HDh36+9//fvny5ZwysPHGG48YMSLaEceM5yjZEMKkAMc3vXv3PvHEE/Gy9rNG0NFtCAEhUDEIiIQqpipKVBAo5NJLL3300UfZlHPFihWkyuk1N998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
