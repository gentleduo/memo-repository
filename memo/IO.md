

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



管道：前一个命令的输出为第二个命令的输入



linux中每启动一个bash代表一个进程，在当前bash中再启动一个/bin/bash那么相当于再fork出来一个进程，并且跟原来的进程是父子关系(可以通过pstree观察)

由于进程间的隔离机制，所以在父进程中定义的变量，在子进程中是看不到的，比如:

在父进程中定义变量a

a=1

echo $a

在当前bash下启动一个新的bash

/bin/bash

此时在子进程中a是没有定义的

echo $a

再退回到父进程中，export a之后，在进入子进程的话，a就可见了

exit

export a

/bin/bash

echo $a









[0]:data:image/png;base64,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B6OhRaPRVA2nFQqFY8eOvfjii8eOHSsUCsxBpVL553/+50KhcNdddzU1NQFrMBrFM888c++99/JOYrFYKpW68sorU6lULBbjDEEQsFSCIKAmCAKWXDKZLBaLSqnW1lbCSSaThUJBKfX+978fePrpp2+44Qbf9/+/miAImJsLL7wwFotNT08rpZLJJA0nkUgAxWIxCALqWxAE1ARBwNmuu+66u+66a2RkJAgCVo1HHnkEuP7664MgIJy1a9fatu153muvvWZZFmf49Kc//a1vfeuRRx75yle+EovFWEKJRKJcLmutY7EYxirTXsNpzz//fKFQeL6Gt3X48OEvfelLd911V1NTUwSjIVSr1bvvvpvZrV+//mMf+9iDDz740EMP/d3f/d3mzZtjsRirVTKZBIrFIqElk0mgUCgAhULhhhtu8H3/5ptv/tKXvsQ8pVIpoFgs0ohisZhlWeUaVqxUKhWLxUqlUrFYZHUol8tPP/20ZVlbt25lIcRiMaBUKnG2WCy2efNm3/cfeughllYsFgOmp6cxVr329vaPfvSjX//61x977LEvfOELnZ2d0WiUWRw+fHhgYABYg9EQvvOd77z22mucIxaLXVsTi8WoCYKAuhHUsOTi8ThQKpWCICCc1tZWYGpqSim1efPmcrl84403fu973wuCgHlKJpNPPPGEUuq6666jESWTyUKhUCwWr7rqKupYEATUBEHAOTZv3vz9739/eHj47//+71kFDh486Pv+5s2b165dGwQBoSUSiWKx6Hlea2srZ/v0pz998ODBb3/725///OdZQolEAtBaB0GAYdRccMEFG2uAsbGx8fHxsbExzvHoo4/+zd/8TQSjIRw9epRz7NixY2Bg4CMf+UgsFsM4QyqVAgqFAqG1trYCP/3pT7du3Voulzdv3vy9732P89La2gocOXKEBpVIJACtNSvZtddeCzz55JOsDj/4wQ+AG264gQWSSCSAUqnEOa655ppUKjU9PX3w4EGWUDweB6ampjCMt9LZ2XnHHXcMDAysW7eOs1Wr1UqlsgajIbzxxhucI5fLNTU1bdy48dJLL6UuBTUsudbWVqBUKgVBQDjJZBI4dOjQqVOnrrnmmgMHDlx44YVBEDB/ra2tQLFYDIKARtTa2gqUSqUgCKhjQRBQEwQB59i8ebNlWYcPHy6Xy2vXrqWhTU9PP/3005ZlXXfddUEQsBAuv/xyQGsdBAHn+MhHPlIoFB588MHrrruOpRKLxQDP84IgwDDeyvPPP5/L5Y4dO8Y5KpVKBKMhXHbZZZzjxIkT+/bt+/CHP3zPPfc8+eSTr776KkbN2prp6elXXnmFcGKxGHDq1Kn29vZHHnnEsizOVzKZBIrFIg0qHo8DR44cob698sorwNq1a3kra9euveqqq3zff/rpp2l0Bw8e9H3/2muvXbt2LQskkUgAU1NTvJVPfOITlmU98cQTzz//PEslkUgApVIJwzjbiy+++C//8i99fX233377s88+yyzWYDSESy65hNkdqvmnf/qn9vb2jo6O9vb2K664gjoQ1LAcksnk4cOHlVJXXXUV58v3/RtvvJGa73znO5dcckkQBJyvyy+/fO3ata+88ornebFYjIaTSCSAUqkUBAF17MSJE4Bt20EQ8Fb+8i//8umnn37sscc++MEP0tAeffRR4Prrrw+CgAUSj8cBz/OCIOAcF1544Sc+8YnvfOc73//+97/2ta+xJOLxOFAqlYIgwFj1Xn311eeff/6/aqanp5mDNRiryfM1wMUXX9ze3p5MJqWUyWSyqamJpRUEATVBELAcWltbDx8+rLV+3/vex3nxfb+3t/fw4cO/93u/99prr7388stBEBBOMpk8fPhwoVC49tpraTjvfe97gWKxGAQBdS+o4a1ce+21//AP//DEE08EQUDjmp6efvrppy3L+uAHPxgEAQskHo8DpVIpCALeysc//vHvfOc7Dz744J49eyzLYvHF43Fgenr6tddesywLY/X5xS9+cfTo0SNHjjz//PMvvvgi87QGY1V69dVXn62hJhaLXXHFFX/0R38kpbziiisuvfRSGl1rayvws5/97IYbbuC89PX1Pfnkk7FYrKur66GHHjpy5Aihtbe3Hz58uFQq0YgSiQQwPT3t+75lWaxY7e3tiUSiVCo9//zz7e3tNKjHHnsMuO666yzLYuEkEglgamqKWbS2tl577bVPPvnkgw8++OlPf5olkUgkSqXS9PR0IpHAWAVefPHFY8eOvfTSS0qpYrFYrVYJYQ2GAdM1zz77LDUXX3zxFVdcceWVV15++eWxWOzKK69kcQQ1LIf3vve9wJEjR4IgYP527tz52GOPrV27dmho6PDhww899NCRI0eCICCcdevWAUeOHAmCgEbU2tp65MiR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
[1]:data:image/png;base64,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

