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

## inode id

## 文件类型

1. -：普通文件 REG（可执行、图片、文本）

2. d：目录

3. l：链接

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

4. b：块设备

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

5. c：字符设备 CHAR

6. s：socket

7. p：pipeline

8. eventpool：

## lsof

lsof可以显示进程打开了哪些文件

比如：lsof -p $$

$$表示当前bash进程的进程ID号

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

## pagecache

pcstat工具安装

1. 在windows环境下使用代理下载pcstat.x86_64

   https://github.com/tobert/pcstat/raw/2014-05-02-01/pcstat.x86_64

2. 上产至linux服务器

3. mv pcstat.x86_64 pcstat

4. chmod 755 pcstat

5. cp ./pcstat /usr/bin

## dirty







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



在linux中{}代表程序块。注意{的后面一定先先跟空格，然后命令必须以;结束

当管道的一端是程序块的时候，linux会启动一个新的进程来执行程序块里的逻辑

echo $$和echo $BASHPID都是查看当前bash的进程ID，但是echo $$命令的优先级较高，比管道命令:|的优先级要高，

{ echo $$ ; ll ./ } | { cat ; }打印出来的是父进程的进程ID，因为linux在解释的时候先执行echo $$，所以$$会被先解析成当前进程的进程ID，然后才根据管道命令分别启动两个进程执行代码块

{ echo $BASHPID ; ll ./ ; } | { cat ; }打印出来的是子进程的进程ID，因为linux先会根据管道命令分别启动两个进程执行代码块，然后在子进程中分别执行两个代码块中的命令

echo $$

echo $BASHPID

{ echo $$ ; ll ./ } | { cat ; }

{ echo $BASHPID ; ll ./ ; } | { cat ; }