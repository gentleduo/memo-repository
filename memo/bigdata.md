# Hadoop

## 环境安装

### 修改主机名

/etc/sysconfig/network

### 设置ip和域名映射

/etc/hosts

### 关闭防火墙和SELinux

#### 防火墙

```bash
#关闭防火墙
[root@server01 ~]# systemctl stop firewalld.service
#启动防火墙
[root@server01 ~]# systemctl start firewalld.service
#重启防火墙
[root@server01 ~]# systemctl restart firewalld.service
#在开机时启用防火墙
[root@server01 ~]# systemctl enable firewalld.service
#禁止在开机时启动防火墙
[root@server01 ~]# systemctl disable firewalld.service
#查看状态
[root@server01 ~]# systemctl status firewalld.service
```

#### SELinux

SELinux是Linux的一种安全子系统

Linux中的权限管理是针对于文件的，而不是针对进程的，也就是说，如果root启动了某个进程，则这个进程可以操作任何一个文件，

SELinux在Linux的文件权限之外，增加了对进程的限制，进程只能在进程允许的范围内操作资源

关闭SELinux的原因：如果开启了SELinux，需要非常复杂的配置，才能正常使用系统，在非生产环境一般不使用SELinux

SELinux的工作模式

enforcing      #强制模式

permissive   #宽容模式

disable          #关闭

修改SELinux的配置文件：/etc/selinux/config

### 免密登录

为了让两个linux机器之间使用ssh不需要用户名和密码。采用了数字签名RSA或者DSA来完成这个操作。

例如：假设A为客户机器，B为目标机；要达到的目的：A机器ssh登录B机器无需输入密码；也就是说B机器授权A机器可以免密访问自己。所以最后需要B机器来确认是否是A机器发过来的登录请求（A将公钥发送给B，B随机生成一段文本并用A的公钥加密后给A，A用私钥解密后将文本返回给B，B验证文本是否正确，从而确认是不是A机器的登录请求）。

具体配置步骤如下：

1. 通过root用户登录A机器，然后执行ssh-keygen -t rsa命令，将会生成密钥文件和私钥文件id_rsa,id_rsa.pub或id_dsa,id_dsa.pub（一路回车，既可完成生成私钥和公钥）
2. 将.pub文件复制到B机器的.ssh目录，并cat id_dsa.pub >> /root/.ssh/authorized_keys（或者也可以直接使用ssh-copy-id server02，那么会直接现在B机器上生成authorized_keys文件，并且将公钥复制到authorized_keys文件中，如果有多台机器需要免密登录B的话，只需要在其他机器上也重复执行ssh-copy-id server02即可）
3. 大功告成，从A机器登录B机器的目标账户，不再需要密码了；（直接运行 #ssh B机器的IP ）

具体通信流程如下：

1. 先在B节点配置A节点的公钥
2. A节点请求免密登录B节点
3. B节点使用A节点提供的公钥加密一段随机文本
4. A节点使用私钥解密，并发回给B节点
5. B节点验证文本是否正确

注意：上面只将公钥拷贝并且追加到了/root/.ssh/authorized_keys中所以只是在A机器中用root用户登陆B机器时免密，而其他B机器的用户（如：admin）从A机器登陆B机器的时候还是需要密码，要想在A机器用B机器的用户admin登陆的时候也实现免密登陆则需要将A机器生成的公钥也拷贝追加到/home/admin/.ssh/authorized_keys中

### 配置时钟同步

#### 方式1

所有主机和同一台主机的时间保持同步

```bash
# 重启ntpd服务
[root@server03 /]# systemctl enable ntpd
[root@server03 /]# systemctl restart ntpd
# 修改时间同步服务器ip
[root@server03 /]# vim /etc/ntp.conf
# 同步时间
[root@server03 /]# ntpq -p
```

#### 方式2

通过网络，所有主机和时间同步服务器保持同步（前提是服务器必须连网）

```bash
# 安装
[root@server02 .ssh]# yum install ntp
# 启动定时服务
[root@server03 .ssh]# crontab -e
# 在输入界面输入以下定时任务：它表示每一分钟都会执行/usr/sbin/ntpdate命令，跟ntp4.aliyun.com服务器做一次时间同步
*/1 * * * * /usr/sbin/ntpdate ntp4.aliyun.com;
```

#### crontab

linux中crontab命令用于设置周期性被执行的指令，该命令从标准输入设备读取指令，并将其存放于“crontab”文件中，以供之后读取和执行。cron  系统调度进程。可以使用它在每天的非高峰负荷时间段运行作业，或在一周或一月中的不同时段运行。cron是系统主要的调度进程，可以在无需人工干预的情况下运行作业。

##### 参数

crontab [-u username] [-l|-e|-r]

-u: 只有root才能进行这个任务，也即帮其他用户新建/删除crontab工作调度;

-e: 编辑crontab 的工作内容;

-l: 查阅crontab的工作内容;

-r: 删除所有的crontab的工作内容，若仅要删除一项，请用-e去编辑。 

##### 定时任务设置 

直接输入命令crontab -e 或者编辑文件/etc/crontab 就可以直接设置定时任务。

##### 定时任务格式

格式：*   *   *   *   *  command

​           分  时 日 月 周 命令 

1. 第1列表示分钟1～59 每分钟用*或者 */1表示

2. 第2列表示小时1～23（0表示0点）

3. 第3列表示日期1～31

4. 第4列表示月份1～12

5. 第5列标识号星期0～6（0表示星期天）

6. 第6列要运行的命令 

星号（*）：代表所有可能的值，例如month字段如果是星号，则表示在满足其它字段的制约条件后每月都执行该命令操作。

逗号（,）：可以用逗号隔开的值指定一个列表范围，例如，“1,2,5,7,8,9”。

中杠（-）：可以用整数之间的中杠表示一个整数范围，例如“2-6”表示“2,3,4,5,6”。

正斜线（/）：可以用正斜线指定时间的间隔频率，例如“0-23/2”表示每两小时执行一次。同时正斜线可以和星号一起使用，例如*/10，如果用在minute字段，表示每十分钟执行一次。

##### 实例

```shell
30 21 * * * /usr/local/etc/rc.d/lighttpd restart //每晚的21:30重启apache。

45 4 1,10,22 * * /usr/local/etc/rc.d/lighttpd restart //每月1、10、22日的4:45重启apache。

10 1 * * 6,0 /usr/local/etc/rc.d/lighttpd restart //每周六、周日的1:10重启apache。

0,30 18-23 * * * /usr/local/etc/rc.d/lighttpd restart //每天18:00至23:00之间每隔30分钟重启apache

0 23 * * 6 /usr/local/etc/rc.d/lighttpd restart //每星期六的11:00 pm重启apache。

* */1 * * * /usr/local/etc/rc.d/lighttpd restart //每一小时重启apache

* 23-7/1 * * * /usr/local/etc/rc.d/lighttpd restart //晚上11点到早上7点之间，每隔一小时重启apache

0 11 4 * mon-wed /usr/local/etc/rc.d/lighttpd restart //每月的4号与每周一到周三的11点重启apache

0 4 1 jan * /usr/local/etc/rc.d/lighttpd restart //一月一号的4点重启apache

*/30 * * * * /usr/sbin/ntpdate 210.72.145.44 //每半小时同步一下时间
```

### lrzsz

```bash
# 在linux里可代替ftp上传和下载。centos服务器，可直接yum -y install lrzsz 程序会自动安装好，然后如你要下载者sz [找到你要下载的文件] 如果你要上传，者rz 浏览找到你本机要上传的文件。
[root@server03 /]# yum -y install lrzsz
# 该命令无法在putty界面使用，可以在SecureCRT终端中使用
[root@server03 /]# rz -E
```

### mysql安装

第一 在线安装mysql相关的软件包

```bash
[root@server01 ~]# yum install mysql mysql-server mysql-devel
```

第二 启动mysql服务

```bash
[root@server01 ~]# /etc/init.d/mysqld start
```

第三 通过mysql安装自带脚本进行设置

```bash
[root@server01 ~]# /usr/bin/mysql_secure_installation
```

第四 进入mysql的客户端然后进行授权

```bash
# 'root'@'%'表示：允许root用户从任何ip访问数据库
mysql> grant all privileges on *.* to 'root'@'%' identified by '123456' with grant option;
mysql> flush privileges;
```

## Hadoop 2.X架构

### NameNode与ResourceManager单节点架构

#### 文件系统

##### NameNode

集群当中的主节点，主要用于管理集群当中的各种数据

##### secondaryNameNode

主要用于hadoop当中元数据信息的辅助管理

##### DataNode

集群当中的从节点，主要用于存储集群当中的各种数据

#### 数据计算

##### ResourceManager

接受用户的计算请求任务，并负责集群的资源分配

##### NodeManager

负责执行主节点Appmaster分配的任务

### NameNode单节点与ResourceManager高可用架构

#### 数据计算

##### ResourceManager

接受用户的计算请求任务，并负责集群的资源分配，以及计算任务的划分，通过zookeeper实现ResourceManager的高可用。

##### NodeManager

负责执行主节点Appmaster分配的任务

### NameNode高可用与ResourceManager单节点

#### 文件系统

##### NameNode

集群当中的主节点，主要用于管理集群当中的各种数据，其中nameNode可以有两个，形成高可用状态

##### DataNode

集群当中的从节点，主要用于存储集群当中的各种数据

##### JournalNode

文件系统元数据信息管理

### NameNode与ResourceManager高可用架构

#### 文件系统

##### NameNode

集群当中的主节点，主要用于管理集群当中的各种数据，其中nameNode可以有两个，形成高可用状态

##### DataNode

集群当中的从节点，主要用于存储集群当中的各种数据

##### JournalNode

文件系统元数据信息管理

#### 数据计算

##### ResourceManager

接受用户的计算请求任务，并负责集群的资源分配，以及计算任务的划分，通过zookeeper实现ResourceManager的高可用。

##### NodeManager

负责执行主节点Appmaster分配的任务

## 配置文件

### core-site.xml

```xml
<configuration>

    <!--  指定集群的文件系统类型:分布式文件系统 -->
	<property>
		<name>fs.default.name</name>
		<value>hdfs://server01:8020</value>
	</property>
    <!--  指定临时文件存储目录 -->
	<property>
		<name>hadoop.tmp.dir</name>
		<value>/usr/local/hadoop-2.7.5/hadoopDatas/tempDatas</value>
	</property>
	<!--  缓冲区大小，实际工作中根据服务器性能动态调整 -->
	<property>
		<name>io.file.buffer.size</name>
		<value>4096</value>
	</property>
	<!--  开启hdfs的垃圾桶机制，删除掉的数据可以从垃圾桶中回收，单位分钟 -->
	<property>
		<name>fs.trash.interval</name>
		<value>10080</value>
	</property>
</configuration>
```

### hdfs-site.xml

```xml
<configuration>

	 <property>
			<name>dfs.namenode.secondary.http-address</name>
			<value>server01:50090</value>
	</property>
	<!-- 指定namenode的访问地址和端口 -->
	<property>
		<name>dfs.namenode.http-address</name>
		<value>server01:50070</value>
	</property>
	<!-- 指定namenode元数据的存放位置 -->
	<property>
		<name>dfs.namenode.name.dir</name>
		<value>file:///usr/local/hadoop-2.7.5/hadoopDatas/namenodeDatas,file:///usr/local/hadoop-2.7.5/hadoopDatas/namenodeDatas2</value>
	</property>
	<!--  定义dataNode数据存储的节点位置，实际工作中，一般先确定磁盘的挂载目录，然后多个目录用，进行分割  -->
	<property>
		<name>dfs.datanode.data.dir</name>
		<value>file:///usr/local/hadoop-2.7.5/hadoopDatas/datanodeDatas,file:///usr/local/hadoop-2.7.5/hadoopDatas/datanodeDatas2</value>
	</property>
	<!-- 指定namenode日志文件的存放目录 -->
	<property>
		<name>dfs.namenode.edits.dir</name>
		<value>file:///usr/local/hadoop-2.7.5/hadoopDatas/nn/edits</value>
	</property>
	<property>
		<name>dfs.namenode.checkpoint.dir</name>
		<value>file:///usr/local/hadoop-2.7.5/hadoopDatas/snn/name</value>
	</property>
	<property>
		<name>dfs.namenode.checkpoint.edits.dir</name>
		<value>file:///usr/local/hadoop-2.7.5/hadoopDatas/dfs/snn/edits</value>
	</property>
	<!-- 文件切片的副本个数-->
	<property>
		<name>dfs.replication</name>
		<value>3</value>
	</property>
	<!-- 设置HDFS的文件权限-->
	<property>
		<name>dfs.permissions</name>
		<value>true</value>
	</property>
	<!-- 设置一个文件切片的大小：128M-->
	<property>
		<name>dfs.blocksize</name>
		<value>134217728</value>
	</property>
</configuration>
```

### hadoop-env.sh

```shell
export JAVA_HOME=/usr/local/java/jdk1.8.0_144
```

### mapred-site.xml

```xml
<configuration>
    
	<!-- 开启MapReduce小任务模式 -->
	<property>
		<name>mapreduce.job.ubertask.enable</name>
		<value>true</value>
	</property>
	<!-- 设置历史任务的主机和端口 -->
	<property>
		<name>mapreduce.jobhistory.address</name>
		<value>server01:10020</value>
	</property>
	<!-- 设置网页访问历史任务的主机和端口 -->
	<property>
		<name>mapreduce.jobhistory.webapp.address</name>
		<value>server01:19888</value>
	</property>
</configuration>
```

### yarn-site.xml

```xml
<configuration>
 
	<!-- 配置yarn主节点的位置 -->
	<property>
		<name>yarn.resourcemanager.hostname</name>
		<value>server01</value>
	</property>
	<property>
		<name>yarn.nodemanager.aux-services</name>
		<value>mapreduce_shuffle</value>
	</property>
	<!-- 开启日志聚合功能 -->
	<property>
		<name>yarn.log-aggregation-enable</name>
		<value>true</value>
	</property>
	<!-- 设置聚合日志在hdfs上的保存时间 -->
	<property>
		<name>yarn.log-aggregation.retain-seconds</name>
		<value>604800</value>
	</property>
	<!-- 设置yarn集群的内存分配方案 -->
	<property>    
		<name>yarn.nodemanager.resource.memory-mb</name>    
		<value>20480</value>
	</property>
	<property>  
        	 <name>yarn.scheduler.minimum-allocation-mb</name>
         	<value>2048</value>
	</property>
	<property>
		<name>yarn.nodemanager.vmem-pmem-ratio</name>
		<value>2.1</value>
	</property>
</configuration>
```

### mapred-env.sh

```shell
export JAVA_HOME=/usr/local/java/jdk1.8.0_144
```

### slaves

```tex
server01
server02
server03
```

### 创建存储目录

```shell
mkdir -p /usr/local/hadoop-2.7.5/hadoopDatas/tempDatas
mkdir -p /usr/local/hadoop-2.7.5/hadoopDatas/namenodeDatas
mkdir -p /usr/local/hadoop-2.7.5/hadoopDatas/namenodeDatas2
mkdir -p /usr/local/hadoop-2.7.5/hadoopDatas/datanodeDatas
mkdir -p /usr/local/hadoop-2.7.5/hadoopDatas/datanodeDatas2
mkdir -p /usr/local/hadoop-2.7.5/hadoopDatas/nn/edits
mkdir -p /usr/local/hadoop-2.7.5/hadoopDatas/snn/name
mkdir -p /usr/local/hadoop-2.7.5/hadoopDatas/dfs/snn/edits
```

## 启动集群

```bash
# 集群第一次启动时格式化namenode(注意：只有当第一次启动时才格式化)
[root@server01 hadoop-2.7.5]# cd /usr/local/hadoop-2.7.5
[root@server01 hadoop-2.7.5]# bin/hdfs namenode -format
# 启动HDFS
[root@server01 hadoop-2.7.5]# sbin/start-dfs.sh
# 启动YARN
[root@server01 hadoop-2.7.5]# sbin/start-yarn.sh
# 启动历史任务服务，可以在historyserver的日志中查看运行情况
[root@server01 hadoop-2.7.5]# sbin/mr-jobhistory-daemon.sh start historyserver

# 停止历史任务服务
[root@server01 hadoop-2.7.5]# sbin/mr-jobhistory-daemon.sh stop historyserver
# 停止YARN
[root@server01 hadoop-2.7.5]# sbin/stop-yarn.sh
# 停止HDFS
[root@server01 hadoop-2.7.5]# sbin/stop-dfs.sh
```

三个端口查看界面

查看hdfs：http://server01:50070/explorer.html

查看yarn集群：http://server01:8088/cluster 

查看历史完成的任务：http://server01:19888/jobhistory

## HDFS

### 定义

HDFS（Hadoop  Distributed  File  System）是 Apache Hadoop 项目的一个子项目. Hadoop 非常适于存储大型数据 (比如 TB 和 PB), 其就是使用 HDFS 作为存储系统. HDFS 使用多台计算机存储文件, 并且提供统一的访问接口, 像是访问一个普通文件系统一样使用分布式文件系统. 

### 应用场景

#### 适合的应用场景

1. 存储非常大的文件：这里非常大指的是几百M、G、或者TB级别，需要高吞吐量，对延时没有要求。
2. 采用流式的数据访问方式: 即一次写入、多次读取，数据集经常从数据源生成或者拷贝一次，然后在其上做很多分析工作 。
3. 运行于商业硬件上: Hadoop不需要特别贵的机器，可运行于普通廉价机器，可以处节约成本
4. 需要高容错性
5. 为数据存储提供所需的扩展能力

#### 不适合的应用场景

1. 低延时的数据访问

   对延时要求在毫秒级别的应用，不适合采用HDFS。HDFS是为高吞吐数据传输设计的,因此可能牺牲延时

2. 大量小文件 

   文件的元数据保存在NameNode的内存中， 整个文件系统的文件数量会受限于NameNode的内存大小。 
   经验而言，一个文件/目录/文件块一般占有150字节的元数据内存空间。如果有100万个文件，每个文件占用1个文件块，则需要大约300M的内存。因此十亿级别的文件数量在现有商用机器上难以支持。

3. 多方读写，需要任意的文件修改 

   HDFS采用追加（append-only）的方式写入数据。不支持文件任意offset的修改。不支持多个写入器（writer）

### 架构

HDFS是一个 主/从（Mater/Slave）体系结构 ， HDFS由四部分组成，HDFS Client、NameNod e、DataNode和Secondary NameNode。

#### Client

就是客户端

1. 文件切分。文件上传 HDFS 的时候，Client 将文件切分成 一个一个的Block，然后进行存 储。 
2. 与 NameNode 交互，获取文件的位置信息。
3. 与 DataNode 交互，读取或者写入数据。
4. Client 提供一些命令来管理 和访问HDFS，比如启动或者关闭HDFS。

#### NameNode

就是 master，它是一个主管、管理者

1. 管理 HDFS 的名称空间 
2. 管理数据块（Block）映射信息 
3. 配置副本策略 
4. 处理客户端读写请求

#### DataNode

就是Slave。NameNode 下达命令，DataNode 执行实际的操作

1. 存储实际的数据块
2. 执行数据块的读/写操作

#### Secondary NameNode

并非 NameNode 的热备。当NameNode 挂掉的时候，它并不 能马上替换 NameNode 并提供服务。

1. 辅助 NameNode，分担其工作量
2.  定期合并 fsimage和fsedits，并推送给NameNode
3.  在紧急情况下，可辅助恢复 NameNode

### 命令行

```bash
#格式：hdfs dfs -ls URI
#作用：类似于Linux的ls命令，显示文件列表
[root@server01 hadoop-2.7.5]# hdfs dfs -ls /
Found 1 items
drwxrwx---   - root supergroup          0 2022-08-25 12:24 /tmp
#格式:dfs dfs -ls -R URI
#作用:在整个目录下递归执行ls, 与UNIX中的ls-R类似
[root@server01 hadoop-2.7.5]# hdfs dfs -ls -R /
drwxrwx---   - root supergroup          0 2022-08-25 12:24 /tmp
drwxrwx---   - root supergroup          0 2022-08-25 12:24 /tmp/hadoop-yarn
drwxrwx---   - root supergroup          0 2022-08-25 12:24 /tmp/hadoop-yarn/staging
drwxrwx---   - root supergroup          0 2022-08-25 12:24 /tmp/hadoop-yarn/staging/history
drwxrwx---   - root supergroup          0 2022-08-25 12:24 /tmp/hadoop-yarn/staging/history/done
drwxrwxrwt   - root supergroup          0 2022-08-25 12:24 /tmp/hadoop-yarn/staging/history/done_intermediate
# 格式：hdfs dfs [-p] -mkdir <paths>
# 作用：以<paths>中的URI作为参数，创建目录。使用-p参数可以递归创建目录
[root@server01 hadoop-2.7.5]# hdfs dfs -mkdir -p /dir1
# 格式：hdfs dfs -put <localsrc > ... <dst>
# 作用：将单个的源文件src或者多个源文件srcs从本地文件系统拷贝到目标文件系统中（<dst>对应的路径）。也可以从标准输入中读取输入，写入目标文件系统中
[root@server01 opt]# hdfs dfs -put ./a.txt /dir1
# 格式：hdfs dfs -moveFromLocal <localsrc>   <dst>
# 作用：和put命令类似，但是源文件localsrc拷贝之后自身被删除
[root@server01 opt]# hdfs dfs -moveFromLocal ./a.txt /dir2
# 格式：hdfs dfs  -get [-ignorecrc ] [-crc] <src> <localdst>
# 作用：将文件拷贝到本地文件系统。 CRC 校验失败的文件通过-ignorecrc选项拷贝。 文件和CRC校验和可以通过-CRC选项拷贝
[root@server01 opt]# hdfs dfs -get /dir1/a.txt /opt
# 格式：hdfs dfs -mv URI   <dest>
# 作用：将hdfs上的文件从原路径移动到目标路径（移动之后文件删除），该命令不能夸文件系统,即：不能把本地的文件移动到hdfs上面去
[root@server01 opt]# hdfs dfs -mv /dir1/a.txt /dir2
# 格式：hdfs dfs -rm [-r] 【-skipTrash】 URI 【URI 。。。】
# 作用：删除参数指定的文件，参数可以有多个。此命令只删除文件和非空目录。如果指定-skipTrash选项，那么在回收站可用的情况下，该选项将跳过回收站而直接删除文件；否则，在回收站可用时，在HDFS Shell中执行此命令，会将文件暂时放到回收站中。
[root@server01 opt]# hdfs dfs -rm -r /dir2
# 格式：hdfs  dfs  -cp URI [URI ...] <dest>
# 作用：将文件拷贝到目标路径中。如果<dest>  为目录的话，可以将多个文件拷贝到该目录下。
# -f 选项将覆盖目标，如果它已经存在。
# -p 选项将保留文件属性（时间戳、所有权、许可、ACL、XAttr）。
[root@server01 opt]# hdfs dfs -cp /dir1/a.txt /dir2/b.txt
# 格式：hdfs dfs  -cat URI [uri ...]
# 作用：将参数所指示的文件内容输出到stdout
[root@server01 opt]# hdfs dfs -cat /dir1/a.txt
# 格式：hdfs dfs -chmod [-R] URI[URI  ...]
# 作用：改变文件权限。如果使用  -R 选项，则对整个目录有效递归执行。使用这一命令的用户必须是文件的所属用户，或者超级用户。
[root@server01 opt]# hdfs dfs -chmod -R 777 /dir1/a.txt
# 格式：hdfs   dfs  -chmod  [-R]  URI[URI  ...]
# 作用：改变文件的所属用户和用户组。如果使用  -R 选项，则对整个目录有效递归执行。使用这一命令的用户必须是文件的所属用户，或者超级用户。
[root@server01 opt]# hdfs dfs -chown -R hadoop:hadoop /dir1/a.txt
# 格式: hdfs dfs -appendToFile <localsrc> ... <dst>
# 作用: 追加一个或者多个文件到hdfs指定文件中.也可以从命令行读取输入.
[root@server01 opt]# hdfs dfs -appendToFile a.xml b.xml /big.xml
# 格式: hdfs dfs -getmerge <dst> ... <localsrc>
# 作用: 小文件合并
[root@server01 opt]# hdfs dfs -getmerge /*.txt /total.txt
```

#### 限额配置

在多人共用HDFS的环境下，配置设置非常重要。特别是在Hadoop处理大量资料的环境，如 果没有配额管理，很容易把所有的空间用完造成别人无法存取。Hdfs的配额设定是针对目录 而不是针对账号，可以 让每个账号仅操作某一个目录，然后对目录设置配置。 hdfs文件的限额配置允许我们以文件个数，或者文件大小来限制我们在某个目录下上传的文 件数量或者文件内容总量，以便达到我们类似百度网盘网盘等限制每个用户允许上传的最大 的文件的量。

##### 数量限额

```bash
# 查看配额信息
[root@server01 opt]#  hdfs dfs -count -q -h /dir1
        none             inf            none             inf            1            1             11.0 K /dir1
# 给该文件夹下面设置最多上传两个文件，发现只能上传一个文件，也就是说给一个目录设置限额为N的时候最多只能放N-1个，文件夹也被当作一个文件了。
[root@server01 opt]# hdfs dfsadmin -setQuota 2 /dir1
# 清除文件数量限制
[root@server01 opt]# hdfs dfsadmin -clrQuota /dir1
```

##### 大小限额

```bash
# 在设置空间配额时，设置的空间至少是block_size * 3大小
[root@server01 opt]# hdfs dfsadmin -setSpaceQuota 384M /dir1
# 清除空间配额限制
[root@server01 opt]# hdfs dfsadmin -clrSpaceQuota /dir1
```

### 安全模式

安全模式是hadoop的一种保护机制，用于保证集群中的数据块的安全性。当集群启动的时 候，会首先进入安全模式。当系统处于安全模式时会检查数据块的完整性。假设我们设置的副本数（即参数dfs.replication）是3，那么在datanode上就应该有3个副本存 在，假设只存在2个副本，那么比例就是2/3=0.666。hdfs默认的副本率0.999。我们的副本率 0.666明显小于0.999，因此系统会自动的复制副本到其他dataNode，使得副本率不小于0.999。 如果系统中有5个副本，超过我们设定的3个副本，那么系统也会删除多于的2个副本。在安全模式状态下，文件系统只接受读数据请求，而不接受删除、修改等变更请求。在，当 整个系统达到安全标准时，HDFS自动离开安全模式。

```bash
# 查看安全模式状态
[root@server01 opt]# hdfs dfsadmin -safemode get
# 进入安全模式
[root@server01 opt]# hdfs dfsadmin -safemode enter
# 离开安全模式
[root@server01 opt]# hdfs dfsadmin -safemode leave
```

## 元数据辅助管理

当Hadoop的集群当中,NameNode的所有元数据信息都保存在了FsImage与Eidts文件当中,这两个文件就记录了所有的数据的元数据信息, 元数据信息的保存目录配置在了 hdfssite.xml 当中

```xml
<property>
    <name>dfs.namenode.name.dir</name>    
    <value>file:///usr/local/hadoop2.7.5/hadoopDatas/namenodeDatas,    
          file:///usr/local/hadoop2.7.5/hadoopDatas/namenodeDatas2
    </value>
</property>
<property>
     <name>dfs.namenode.edits.dir</name>
     <value>file:///usr/local/hadoop2.7.5/hadoopDatas/nn/edits</value
</property>>
```

### FsImage和Edits详解

#### edits

1. edits 存放了客户端最近一段时间的操作日志
2. 客户端对HDFS进行写文件时会首先被记录在edits文件中
3. edits修改时元数据也会更新

文件信息查看：使用命令 hdfs oev

```bash
[root@server01 current]# cd /usr/local/hadoop-2.7.5/hadoopDatas/nn/edits/current
[root@server01 current]# ll
总用量 2064
-rw-r--r-- 1 root root 1048576 8月  25 12:14 edits_0000000000000000001-0000000000000000001
-rw-r--r-- 1 root root     743 8月  25 13:09 edits_0000000000000000002-0000000000000000011
-rw-r--r-- 1 root root    1308 8月  25 14:09 edits_0000000000000000012-0000000000000000031
-rw-r--r-- 1 root root 1048576 8月  25 14:09 edits_inprogress_0000000000000000032
-rw-r--r-- 1 root root       3 8月  25 14:09 seen_txid
-rw-r--r-- 1 root root     206 8月  25 12:11 VERSION
[root@server01 current]# hdfs oev -i edits_0000000000000000001-0000000000000000001 -p XML -o myedit.xml
```

#### fsimage

1. NameNode中关于元数据的镜像,一般称为检查点,fsimage存放了一份比较完整的元数据信息
2. 因为fsimage是NameNode的完整的镜像,如果每次都加载到内存生成树状拓扑结构，这是非常耗内存和CPU,所以一般开始时对NameNode的操作都放在edits中
3. fsimage内容包含了NameNode管理下的所有DataNode文件及文件block及block所在的DataNode的元数据信息.
4. 随着edits内容增大,就需要在一定时间点和fsimage合并

文件信息查看：使用命令 hdfs oiv

```bash
[root@server01 current]# cd /usr/local/hadoop-2.7.5/hadoopDatas/namenodeDatas/current
[root@server01 current]# ll
总用量 24
-rw-r--r-- 1 root root 858 8月  25 14:09 fsimage_0000000000000000031
-rw-r--r-- 1 root root  62 8月  25 14:09 fsimage_0000000000000000031.md5
-rw-r--r-- 1 root root 858 8月  25 15:09 fsimage_0000000000000000033
-rw-r--r-- 1 root root  62 8月  25 15:09 fsimage_0000000000000000033.md5
-rw-r--r-- 1 root root   3 8月  25 15:09 seen_txid
-rw-r--r-- 1 root root 206 8月  25 12:11 VERSION
[root@server01 current]# hdfs oiv -i fsimage_0000000000000000031 -p XML -o fsimage.xml
```

### SecondaryNameNode

SecondaryNameNode定期合并fsimage和edits,把edits控制在一个范围内

1. SecondaryNameNode 通知 NameNode 切换 editlog
2. SecondaryNameNode 从 NameNode 中获得 fsimage 和 editlog(通过http方式)
3. SecondaryNameNode 将 fsimage 载入内存, 然后开始合并 editlog, 合并之后成为新的fsimage
4. SecondaryNameNode 将新的 fsimage 发回给 NameNode
5. NameNode 用新的 fsimage 替换旧的 fsimage

#### 配置SecondaryNameNode

hdfs-site.xml

```xml
<property>
    <name>dfs.namenode.secondary.http-address</name>
    <value>server01:50090</value>
</property>
```

core-site.xml（不配置保持默认也可以）

```xml
<!-- 多久记录一次 HDFS 镜像, 默认 1小时 -->
<property>
    <name>fs.checkpoint.period</name>
    <value>3600</value>
</property>
<!-- 一次记录多大, 默认 64M -->
<property>
    <name>fs.checkpoint.size</name>
    <value>67108864</value>
</property>
```

#### 特点

1. 完成合并的是SecondaryNameNode,会请求NameNode停止使用edits,暂时将新写操作放入一个新的文件中edits.new
2. SecondaryNameNode从NameNode中通过HttpGET获得edits,因为要和fsimage合并,所以也是通过HttpGet的方式把fsimage加载到内存,然后逐一执行具体对文件系统的操作,与fsimage合并,生成新的fsimage,然后通过HttpPOST的方式把fsimage发送给NameNode.NameNode从SecondaryNameNode获得了fsimage后会把原有的fsimage替换为新的fsimage,把edits.new变成edits.同时会更新fstime
3. Hadoop进入安全模式时需要管理员使用dfsadmin的savenamespace来创建新的检查点
4. SecondaryNameNode在合并edits和fsimage时需要消耗的内存和NameNode差不多,所以一般把NameNode和SecondaryNameNode放在不同的机器上

## HDFS的API操作

### 配置Windows下Hadoop环境

在windows系统需要配置hadoop运行环境，否则直接运行代码会出现以下问题:

缺少winutils.exe

Could not locate executable null \bin\winutils.exe in the hadoop binaries 

缺少hadoop.dll

Unable to load native-hadoop library for your platform… using builtin-Java classes where applicable 

#### 步骤

1. 将hadoop2.7.5文件夹拷贝到一个没有中文没有空格的路径下面
2. 在windows上面配置hadoop的环境变量：HADOOP_HOME，并将%HADOOP_HOME%\bin添加到path中
3. 把hadoop2.7.5文件夹中bin目录下的hadoop.dll文件放到系统盘:C:\Windows\System32 目录
4. 关闭windows重启

## MapReduce

### 介绍

MapReduce思想在生活中处处可见。或多或少都曾接触过这种思想。MapReduce的思想核心 是“分而治之”，适用于大量复杂的任务处理场景（大规模数据处理场景）。Map负责“分”，即把复杂的任务分解为若干个“简单的任务”来并行处理。可以进行拆分的 前提是这些小任务可以并行计算，彼此间几乎没有依赖关系。Reduce负责“合”，即对map阶段的结果进行全局汇总。

### 设计思想

MapReduce是一个分布式运算程序的编程框架，核心功能是将用户编写的业务逻辑代码和自 带默认组件整合成一个完整的分布式运算程序，并发运行在Hadoop集群上。MapReduce设计并提供了统一的计算框架，为程序员隐藏了绝大多数系统层面的处理细节。 为程序员提供一个抽象和高层的编程接口和框架。程序员仅需要关心其应用层的具体计算问 题，仅需编写少量的处理应用本身计算问题的程序代码。如何具体完成这个并行计算任务所 相关的诸多系统层细节被隐藏起来,交给计算框架去处理：Map和Reduce为程序员提供了一个清晰的操作接口抽象描述。一个完整的mapreduce程序在分布式运行时有三类实例进程：

1. MRAppMaster 负责整个程序的过程调度及状态协调
2. MapTask 负责map阶段的整个数据处理流程
3. ReduceTask 负责reduce阶段的整个数据处理流程

### 编程规范

MapReduce的开发一共有八个步骤,其中Map阶段分为2个步骤，Shule阶段4个步骤，Reduce阶段分为2个步骤

Map阶段2个步骤

1. 设置InputFormat类,将数据切分为Key-Value(K1和V1，K1表示这一行相对于文件的偏移量，V1表示这一行数据的内容)对,输入到第二步
2. 自定义Map逻辑,将第一步的结果转换成另外的Key-Value（K2和V2，K2一般为单词，V2一般为数量1）对,输出结果

因此在我们编写代码实现MapReduce的时候，在继承Mapper类的时候，有四个泛型分分别是：K1、V1、K2、V2；分别代表的就是上面的两组Key-Value，所以整个Map、Shuffle、Reduce就是将一个输入的Key-Value形式转成输出Key-Value形式的过程

Shuffle阶段4个步骤

1. 分区：对输出的Key-Value对进行分区，默认的分区编号为：key的hashcode%ReduceTask个数(map阶段)
2. 排序：对不同分区的数据按照相同的Key排序，mapreduce会自动根据key进行排序，是默认行为，如果要自定义排序的话只需要将key实现WritableComparable接口并且重写compareTo方法即可(map阶段)
3. 规约：(可选)对分组过的数据初步规约,降低数据的网络拷贝(map阶段)
4. 分组：对数据进行分组,相同Key的Value放入一个集合中(reduce阶段)

Reduce阶段2个步骤

1. 对多个Map任务的结果进行排序以及合并,编写Reduce函数实现自己的逻辑,对输入的Key-Value进行处理,转为新的Key-Value（K3和V3）输出
2. 设置OutputFormat处理并保存Reduce输出的Key-Value数据

### WordCount

WordCountMapper.java

```java
package org.duo.mapreduce;

import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Counter;
import org.apache.hadoop.mapreduce.Mapper;

import java.io.IOException;

/*
  四个泛型解释:
    KEYIN :K1的类型
    VALUEIN: V1的类型

    KEYOUT: K2的类型
    VALUEOUT: V2的类型
 */
public class WordCountMapper extends Mapper<LongWritable,Text, Text , LongWritable> {

    //map方法就是将K1和V1 转为 K2和V2
    /*
      参数:
         key    : K1   行偏移量
         value  : V1   每一行的文本数据
         context ：表示上下文对象
     */
    /*
      如何将K1和V1 转为 K2和V2
        K1         V1
        0   hello,world,hadoop
        15  hdfs,hive,hello
       ---------------------------

        K2            V2
        hello         1
        world         1
        hdfs          1
        hadoop        1
        hello         1
     */
    @Override
    protected void map(LongWritable key, Text value, Context context) throws IOException, InterruptedException {
        Text text = new Text();
        LongWritable longWritable = new LongWritable();
        //1:将一行的文本数据进行拆分
        String[] split = value.toString().split(",");
        //2:遍历数组，组装 K2 和 V2
        for (String word : split) {
            //3:将K2和V2写入上下文
            text.set(word);
            longWritable.set(1);
            context.write(text, longWritable);
        }

    }
}
```

WordCountReducer.java

```java
package org.duo.mapreduce;

import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Reducer;

import java.io.IOException;
/*
  四个泛型解释:
    KEYIN:  K2类型
    VALULEIN: V2类型

    KEYOUT: K3类型
    VALUEOUT:V3类型
 */

public class WordCountReducer extends Reducer<Text,LongWritable,Text,LongWritable> {
    //reduce方法作用: 将新的K2和V2转为 K3和V3 ，将K3和V3写入上下文中
    /*
      参数:
        key ： 新K2
        values： 集合 新 V2
        context ：表示上下文对象

        ----------------------
        如何将新的K2和V2转为 K3和V3
        新  K2         V2
            hello      <1,1,1>
            world      <1,1>
            hadoop     <1>
        ------------------------
           K3        V3
           hello     3
           world     2
           hadoop    1

     */

    @Override
    protected void reduce(Text key, Iterable<LongWritable> values, Context context) throws IOException, InterruptedException {
        long count = 0;
       //1:遍历集合，将集合中的数字相加，得到 V3
        for (LongWritable value : values) {
             count += value.get();
        }
        //2:将K3和V3写入上下文中
        context.write(key, new LongWritable(count));
    }
}
```

JobMain.java

```java
package org.duo.mapreduce;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.conf.Configured;
import org.apache.hadoop.fs.FileSystem;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.lib.input.TextInputFormat;
import org.apache.hadoop.mapreduce.lib.output.TextOutputFormat;
import org.apache.hadoop.util.Tool;
import org.apache.hadoop.util.ToolRunner;

import java.net.URI;

public class JobMain extends Configured implements Tool {

    //该方法用于指定一个job任务
    @Override
    public int run(String[] args) throws Exception {

        //1:创建一个job任务对象
        Job job = Job.getInstance(super.getConf(), "wordcount");
        //如果打包运行出错，则需要加该配置
        job.setJarByClass(JobMain.class);
        //2:配置job任务对象(八个步骤)

        //第一步:指定文件的读取方式和读取路径
        job.setInputFormatClass(TextInputFormat.class);
        TextInputFormat.addInputPath(job, new Path("hdfs://server01:8020/wordcount"));
        //TextInputFormat.addInputPath(job, new Path("file:///D:\\mapreduce\\input"));

        //第二步:指定Map阶段的处理方式和数据类型
        job.setMapperClass(WordCountMapper.class);
        //设置Map阶段K2的类型
        job.setMapOutputKeyClass(Text.class);
        //设置Map阶段V2的类型
        job.setMapOutputValueClass(LongWritable.class);

        //第三，四，五，六 采用默认的方式

        //第七步：指定Reduce阶段的处理方式和数据类型
        job.setReducerClass(WordCountReducer.class);
        //设置K3的类型
        job.setOutputKeyClass(Text.class);
        //设置V3的类型
        job.setOutputValueClass(LongWritable.class);

        //第八步: 设置输出类型
        job.setOutputFormatClass(TextOutputFormat.class);
        //设置输出的路径
        Path path = new Path("hdfs://server01:8020/wordcount_out");
        TextOutputFormat.setOutputPath(job, path);
        //TextOutputFormat.setOutputPath(job, new Path("file:///D:\\mapreduce\\output"));

        //获取FileSystem
        FileSystem fileSystem = FileSystem.get(new URI("hdfs://server01:8020"), new Configuration());
        //判断目录是否存在
        boolean bl2 = fileSystem.exists(path);
        if (bl2) {
            //删除目标目录
            fileSystem.delete(path, true);
        }

        //等待任务结束
        boolean bl = job.waitForCompletion(true);

        return bl ? 0 : 1;
    }

    public static void main(String[] args) throws Exception {

        Configuration configuration = new Configuration();

        //启动job任务
        int run = ToolRunner.run(configuration, new JobMain(), args);
        System.exit(run);

    }
}
```

### 运行模式

#### 集群运行模式

1. 将MapReduce程序提交给Yarn集群, 分发到很多的节点上并发执行
2. 处理的数据和输出结果应该位于HDFS文件系统
3. 提交集群的实现步骤: 将程序打成JAR包，并上传，然后在集群上用hadoop命令启动

```bash
[root@server01 opt]# hadoop jar original-hadoop-1.0-SNAPSHOT.jar org.duo.mapreduce.JobMain
```

#### 本地运行模式

1. MapReduce 程序是被提交给 LocalJobRunner 在本地以单进程的形式运行

2. 处理的数据及输出结果可以在本地文件系统, 也可以在hdfs上

3. 怎样实现本地运行? 写一个程序, 不要带集群的配置文件, 本质是程序的conf中是否有

   mapreduce.framework.name=local 以及yarn.resourcemanager.hostname=local 参数

4. 本地模式非常便于进行业务逻辑的 Debug , 只要在 Eclipse 中打断点即可

```java
configuration.set("mapreduce.framework.name","local");
configuration.set(" yarn.resourcemanager.hostname","local");
TextInputFormat.addInputPath(job,new Path("file:///F:\\wordcount\\input"));
TextOutputFormat.setOutputPath(job,new Path("file:///F:\\wordcount\\output"));
```

### 分区

在MapReduce中,通过我们指定分区,会将同一个分区的数据发送到同一个Reduce当中进行处理例如:为了数据的统计,可以把一批类似的数据发送到同一个Reduce当中,在同一个Reduce当中统计相同类型的数据,就可以实现类似的数据分区和统计等其实就是相同类型的数据,有共性的数据,送到一起去处理Reduce当中默认的分区只有一个

### 计数器

计数器是收集作业统计信息的有效手段之一，用于质量控制或应用级统计。计数器还可辅助诊断系统故障。如果需要将日志信息传输到map或reduce任务，更好的方法通常是看能否用一个计数器值来记录某一特定事件的发生。对于大型分布式作业而言，使用计数器更为方便。除了因为获取计数器值比输出日志更方便，还有根据计数器值统计特定事件的发生次数要比分析一堆日志文件容易得多。

hadoop内置计数器列表

| 名称                   | 类                                                           |
| ---------------------- | ------------------------------------------------------------ |
| MapReduce任务计数器    | org.apache.hadoop.mapreduce.TaskCounter                      |
| 文件系统计数器         | org.apache.hadoop.mapreduce.FileSystemCounter                |
| FileInputFormat计数器  | org.apache.hadoop.mapreduce.lib.input.FileInputFormatCounter |
| FileOutputFormat计数器 | org.apache.hadoop.mapreduce.lib.output.FileOutputFormatCounter |
| 作业计数器             | org.apache.hadoop.mapreduce.JobCounter                       |

每次mapreduce执行完成之后，会看到一些日志记录出来，其中最重要的一些日志记录，所有的这些都是MapReduce的计数器的功能。

自定义计数器：统计map接收到的数据记录条数

第一种方式：通过context上下文对象获取计数器，在map端使用计数器进行统计

第二种方式：通过enum枚举类型来定义计数器 统计reduce端数据的输入的key有多少个

PartitionMapper.java

```java
package org.duo.partition;

import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.NullWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Counter;
import org.apache.hadoop.mapreduce.Mapper;

import java.io.IOException;
/*
  K1: 行偏移量 LongWritable
  V1: 行文本数据  Text

  K2: 行文本数据 Text
  V2:  NullWritable
 */

public class PartitionMapper extends Mapper<LongWritable, Text, Text, NullWritable> {

    //map方法将K1和V1转为K2和V2
    @Override
    protected void map(LongWritable key, Text value, Context context) throws IOException, InterruptedException {

        //方式1：定义计数器
        Counter counter = context.getCounter("MR_COUNTER", "partition_counter");
        //每次执行该方法，则计数器变量的值加1
        counter.increment(1L);
        context.write(value, NullWritable.get());
    }
}
```

MyPartitioner.java

```java
package org.duo.partition;

import org.apache.hadoop.io.NullWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Partitioner;

public class MyPartitioner extends Partitioner<Text, NullWritable> {

    /*
      1：定义分区规则
      2:返回对应的分区编号
     */
    @Override
    public int getPartition(Text text, NullWritable nullWritable, int i) {

        //1:拆分行文本数据(K2),获取中奖字段的值
        String[] split = text.toString().split("\t");
        String numStr = split[5];

        //2:判断中奖字段的值和15的关系，然后返回对应的分区编号
        if (Integer.parseInt(numStr) > 15) {
            return 1;
        } else {
            return 0;
        }

    }
}
```

PartitionerReducer.java

```java
package org.duo.partition;

import org.apache.hadoop.io.NullWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Reducer;

import java.io.IOException;

/*
  K2:  Text
  V2:  NullWritable

  K3:  Text
  V3: NullWritable
 */
public class PartitionerReducer extends Reducer<Text, NullWritable, Text, NullWritable> {

    public static enum Counter {
        MY_INPUT_RECOREDS, MY_INPUT_BYTES
    }

    @Override
    protected void reduce(Text key, Iterable<NullWritable> values, Context context) throws IOException, InterruptedException {
        //方式2：使用枚枚举来定义计数器
        context.getCounter(Counter.MY_INPUT_RECOREDS).increment(1L);
        context.write(key, NullWritable.get());
    }
}
```

JobMain.java

```java
package org.duo.partition;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.conf.Configured;
import org.apache.hadoop.fs.FileSystem;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.NullWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.lib.input.TextInputFormat;
import org.apache.hadoop.mapreduce.lib.output.TextOutputFormat;
import org.apache.hadoop.util.Tool;
import org.apache.hadoop.util.ToolRunner;

import java.net.URI;

public class JobMain extends Configured implements Tool {

    @Override
    public int run(String[] args) throws Exception {

        //1:创建job任务对象
        Job job = Job.getInstance(super.getConf(), "partition_maperduce");

        //2:对job任务进行配置(八个步骤)
        //第一步:设置输入类和输入的路径
        job.setInputFormatClass(TextInputFormat.class);
        TextInputFormat.addInputPath(job, new Path("hdfs://server01:8020/input"));
        //TextInputFormat.addInputPath(job, new Path("file:///D:\\input"));
        //第二步:设置Mapper类和数据类型（K2和V2）
        job.setMapperClass(PartitionMapper.class);
        job.setMapOutputKeyClass(Text.class);
        job.setMapOutputValueClass(NullWritable.class);

        //第三步，指定分区类
        job.setPartitionerClass(MyPartitioner.class);
        //第四, 五，六步
        //第七步:指定Reducer类和数据类型(K3和V3)
        job.setReducerClass(PartitionerReducer.class);
        job.setOutputValueClass(Text.class);
        job.setOutputValueClass(NullWritable.class);
        //设置ReduceTask的个数
        job.setNumReduceTasks(2);

        //第八步:指定输出类和输出路径
        job.setOutputFormatClass(TextOutputFormat.class);
        Path path = new Path("hdfs://server01:8020/out");
        TextOutputFormat.setOutputPath(job, new Path("hdfs://server01:8020/out"));
        //TextOutputFormat.setOutputPath(job, new Path("file:///D:\\out\\partition_out3"));

        //获取FileSystem
        FileSystem fileSystem = FileSystem.get(new URI("hdfs://server01:8020"), new Configuration());
        //判断目录是否存在
        boolean bl2 = fileSystem.exists(path);
        if (bl2) {
            //删除目标目录
            fileSystem.delete(path, true);
        }

        //3:等待任务结束
        boolean bl = job.waitForCompletion(true);

        return bl ? 0 : 1;
    }

    public static void main(String[] args) throws Exception {
        Configuration configuration = new Configuration();
        //启动job任务
        int run = ToolRunner.run(configuration, new JobMain(), args);
        System.exit(run);
    }
}
```

### 排序和序列化

1. 序列化(Serialization)是指把结构化对象转化为字节流
2. 反序列化(Deserialization)是序列化的逆过程.把字节流转为结构化对象.当要在进程间传递对象或持久化对象的时候,就需要序列化对象成字节流,反之当要将接收到或从磁盘读取的字节流转换为对象,就要进行反序列化
3. Java的序列化(Serializable)是一个重量级序列化框架,一个对象被序列化后,会附带很多额外的信息(各种校验信息,header,继承体系等）,不便于在网络中高效传输.所以,Hadoop自己开发了一套序列化机制(Writable),精简高效.不用像Java对象类一样传输多层的父子关系,需要哪个属性就传输哪个属性值,大大的减少网络传输的开销
4. Writable是Hadoop的序列化格式,Hadoop定义了这样一个Writable接口.一个类要支持可序列化只需实现这个接口即可
5. 另外Writable有一个子接口是WritableComparable,WritableComparable是既可实现序列化,也可以对key进行比较,我们这里可以通过自定义Key实现WritableComparable来实现我们的排序功能

数据格式：

```markdown
a 1
a 9
b 3
a 7
b 8
b 10
a 5
```

要求:

- 第一列按照字典顺序进行排列 
- 第一列相同的时候, 第二列按照升序进行排列

解决思路:

- 将 Map 端输出的 <key,value> 中的 key 和 value 组合成一个新的 key (newKey), value值不变
- 这里就变成 <(key,value),value> , 在针对 newKey 排序的时候, 如果 key 相同, 就再对value进行排序

SortBean.java

```java
package org.duo.sort;

import org.apache.hadoop.io.WritableComparable;

import java.io.DataInput;
import java.io.DataOutput;
import java.io.IOException;

public class SortBean implements WritableComparable<SortBean> {

    private String word;
    private int num;

    public String getWord() {
        return word;
    }

    public void setWord(String word) {
        this.word = word;
    }

    public int getNum() {
        return num;
    }

    public void setNum(int num) {
        this.num = num;
    }

    @Override
    public String toString() {
        return word + "\t" + num;
    }

    //实现比较器，指定排序的规则
    /*
      规则:
        第一列(word)按照字典顺序进行排列    //  aac   aad
        第一列相同的时候, 第二列(num)按照升序进行排列
     */
    @Override
    public int compareTo(SortBean sortBean) {
        //先对第一列排序: Word排序
        int result = this.word.compareTo(sortBean.word);
        //如果第一列相同，则按照第二列进行排序
        if (result == 0) {
            return this.num - sortBean.num;
        }
        return result;
    }

    //实现序列化
    @Override
    public void write(DataOutput out) throws IOException {
        out.writeUTF(word);
        out.writeInt(num);
    }

    //实现反序列
    @Override
    public void readFields(DataInput in) throws IOException {
        this.word = in.readUTF();
        this.num = in.readInt();
    }
}
```

SortMapper.java

```java
package org.duo.sort;

import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.NullWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Mapper;

import java.io.IOException;

public class SortMapper extends Mapper<LongWritable,Text,SortBean,NullWritable> {
    /*
      map方法将K1和V1转为K2和V2:

      K1            V1
      0            a  3
      5            b  7
      ----------------------
      K2                         V2
      SortBean(a  3)         NullWritable
      SortBean(b  7)         NullWritable
     */
    @Override
    protected void map(LongWritable key, Text value, Context context) throws IOException, InterruptedException {
        //1:将行文本数据(V1)拆分，并将数据封装到SortBean对象,就可以得到K2
        String[] split = value.toString().split("\t");

        SortBean sortBean = new SortBean();
        sortBean.setWord(split[0]);
        sortBean.setNum(Integer.parseInt(split[1]));

        //2:将K2和V2写入上下文中
        context.write(sortBean, NullWritable.get());
    }
}
```

SortReducer.java

```java
package org.duo.sort;

import org.apache.hadoop.io.NullWritable;
import org.apache.hadoop.mapreduce.Reducer;

import java.io.IOException;

public class SortReducer extends Reducer<SortBean,NullWritable,SortBean,NullWritable> {

    //reduce方法将新的K2和V2转为K3和V3
    @Override
    protected void reduce(SortBean key, Iterable<NullWritable> values, Context context) throws IOException, InterruptedException {
       context.write(key, NullWritable.get());
    }
}
```

JobMain.java

```java
package org.duo.sort;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.conf.Configured;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.NullWritable;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.lib.input.TextInputFormat;
import org.apache.hadoop.mapreduce.lib.output.TextOutputFormat;
import org.apache.hadoop.util.Tool;
import org.apache.hadoop.util.ToolRunner;

public class JobMain extends Configured implements Tool {

    @Override
    public int run(String[] args) throws Exception {

        //1:创建job对象
        Job job = Job.getInstance(super.getConf(), "mapreduce_sort");

        //2:配置job任务(八个步骤)
        //第一步:设置输入类和输入的路径
        job.setInputFormatClass(TextInputFormat.class);
        ///TextInputFormat.addInputPath(job, new Path("hdfs://node01:8020/input/sort_input"));
        TextInputFormat.addInputPath(job, new Path("file:///D:\\input\\sort_input"));

        //第二步: 设置Mapper类和数据类型
        job.setMapperClass(SortMapper.class);
        job.setMapOutputKeyClass(SortBean.class);
        job.setMapOutputValueClass(NullWritable.class);

        //第三，四，五，六

        //第七步：设置Reducer类和类型
        job.setReducerClass(SortReducer.class);
        job.setOutputKeyClass(SortBean.class);
        job.setOutputValueClass(NullWritable.class);

        //第八步: 设置输出类和输出的路径
        job.setOutputFormatClass(TextOutputFormat.class);
        //TextOutputFormat.setOutputPath(job, new Path("hdfs://node01:8020/out/sort_out"));
        TextOutputFormat.setOutputPath(job, new Path("file:///D:\\out\\sort_out"));

        //3:等待任务结束
        boolean bl = job.waitForCompletion(true);

        return bl ? 0 : 1;
    }

    public static void main(String[] args) throws Exception {

        Configuration configuration = new Configuration();
        //启动job任务
        int run = ToolRunner.run(configuration, new JobMain(), args);
        System.exit(run);
    }
}
```

### 规约

每一个map都可能会产生大量的本地输出，Combiner的作用就是对map端的输出先做一次合并，以减少在map和reduce节点之间的数据传输量，以提高网络IO性能，是MapReduce的一种优化手段之一

1. combiner是MR程序中Mapper和Reducer之外的一种组件
2. combiner组件的父类就是Reducer
3. combiner和reducer的区别在于运行的位置
   - Combiner是在每一个maptask所在的节点运行
   - Reducer是接收全局所有Mapper的输出结果
4. combiner的意义就是对每一个maptask的输出进行局部汇总，以减小网络传输量

实现步骤

1. 自定义一个combiner继承Reducer，重写reduce方法
2. 在job中设置job.setCombinerClass(CustomCombiner.class)

combiner能够应用的前提是不能影响最终的业务逻辑，而且，combiner的输出kv应该跟reducer的输入kv类型要对应起来

MyCombiner.java

```java
package org.duo.combiner;

import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Reducer;

import java.io.IOException;

public class MyCombiner extends Reducer<Text, LongWritable, Text, LongWritable> {

    /*
       key : hello
       values: <1,1,1,1>
     */
    @Override
    protected void reduce(Text key, Iterable<LongWritable> values, Context context) throws IOException, InterruptedException {

        long count = 0;
        //1:遍历集合，将集合中的数字相加，得到 V3
        for (LongWritable value : values) {
            count += value.get();
        }
        //2:将K3和V3写入上下文中
        context.write(key, new LongWritable(count));
    }
}
```

WordCountMapper.java

```java
package org.duo.combiner;

import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Mapper;

import java.io.IOException;

/*
  四个泛型解释:
    KEYIN :K1的类型
    VALUEIN: V1的类型

    KEYOUT: K2的类型
    VALUEOUT: V2的类型
 */
public class WordCountMapper extends Mapper<LongWritable, Text, Text, LongWritable> {

    //map方法就是将K1和V1 转为 K2和V2
    /*
      参数:
         key    : K1   行偏移量
         value  : V1   每一行的文本数据
         context ：表示上下文对象
     */
    /*
      如何将K1和V1 转为 K2和V2
        K1         V1
        0   hello,world,hadoop
        15  hdfs,hive,hello
       ---------------------------

        K2            V2
        hello         1
        world         1
        hdfs          1
        hadoop        1
        hello         1
     */
    @Override
    protected void map(LongWritable key, Text value, Context context) throws IOException, InterruptedException {

        Text text = new Text();
        LongWritable longWritable = new LongWritable();
        //1:将一行的文本数据进行拆分
        String[] split = value.toString().split(",");

        //2:遍历数组，组装 K2 和 V2
        for (String word : split) {
            //3:将K2和V2写入上下文
            text.set(word);
            longWritable.set(1);
            context.write(text, longWritable);
        }

    }
}
```

WordCountReducer.java

```java
package org.duo.combiner;

import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Reducer;

import java.io.IOException;
/*
  四个泛型解释:
    KEYIN:  K2类型
    VALULEIN: V2类型

    KEYOUT: K3类型
    VALUEOUT:V3类型
 */

public class WordCountReducer extends Reducer<Text, LongWritable, Text, LongWritable> {

    //reduce方法作用: 将新的K2和V2转为 K3和V3 ，将K3和V3写入上下文中
    /*
      参数:
        key ： 新K2
        values： 集合 新 V2
        context ：表示上下文对象

        ----------------------
        如何将新的K2和V2转为 K3和V3
        新  K2         V2
            hello      <1,1,1>
            world      <1,1>
            hadoop     <1>
        ------------------------
           K3        V3
           hello     3
           world     2
           hadoop    1

     */

    @Override
    protected void reduce(Text key, Iterable<LongWritable> values, Context context) throws IOException, InterruptedException {
        long count = 0;
        //1:遍历集合，将集合中的数字相加，得到 V3
        for (LongWritable value : values) {
            count += value.get();
        }
        //2:将K3和V3写入上下文中
        context.write(key, new LongWritable(count));
    }
}
```

JobMain.java

```java
package org.duo.combiner;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.conf.Configured;
import org.apache.hadoop.fs.FileSystem;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.lib.input.TextInputFormat;
import org.apache.hadoop.mapreduce.lib.output.TextOutputFormat;
import org.apache.hadoop.util.Tool;
import org.apache.hadoop.util.ToolRunner;

import java.net.URI;

public class JobMain extends Configured implements Tool {

    //该方法用于指定一个job任务
    @Override
    public int run(String[] args) throws Exception {

        //1:创建一个job任务对象
        Job job = Job.getInstance(super.getConf(), "wordcount");
        //如果打包运行出错，则需要加该配置
        job.setJarByClass(JobMain.class);
        //2:配置job任务对象(八个步骤)

        //第一步:指定文件的读取方式和读取路径
        job.setInputFormatClass(TextInputFormat.class);
        //TextInputFormat.addInputPath(job, new Path("hdfs://node01:8020/wordcount"));
        TextInputFormat.addInputPath(job, new Path("file:///D:\\input\\combiner_input"));

        //第二步:指定Map阶段的处理方式和数据类型
        job.setMapperClass(WordCountMapper.class);
        //设置Map阶段K2的类型
        job.setMapOutputKeyClass(Text.class);
        //设置Map阶段V2的类型
        job.setMapOutputValueClass(LongWritable.class);

        //第三（分区），四 （排序）
        //第五步: 规约(Combiner)
        job.setCombinerClass(MyCombiner.class);
        //第六步 分布

        //第七步：指定Reduce阶段的处理方式和数据类型
        job.setReducerClass(WordCountReducer.class);
        //设置K3的类型
        job.setOutputKeyClass(Text.class);
        //设置V3的类型
        job.setOutputValueClass(LongWritable.class);

        //第八步: 设置输出类型
        job.setOutputFormatClass(TextOutputFormat.class);
        //设置输出的路径
        TextOutputFormat.setOutputPath(job, new Path("file:///D:\\out\\combiner_out"));

        //等待任务结束
        boolean bl = job.waitForCompletion(true);

        return bl ? 0 : 1;
    }

    public static void main(String[] args) throws Exception {
        Configuration configuration = new Configuration();

        //启动job任务
        int run = ToolRunner.run(configuration, new JobMain(), args);
        System.exit(run);

    }
}
```

### 综合案例

#### 统计求和

统计每个手机号的上行数据包总和，下行数据包总和，上行总流量之和，下行总流量之和分析：以手机号码作为key值，上行流量，下行流量，上行总流量，下行总流量四个字段作为value值，然后以这个key，和value作为map阶段的输出，reduce阶段的输入

| 序号 | 字段        | 类型   | 描述                   |
| ---- | ----------- | ------ | ---------------------- |
| 0    | reportTime  | long   | 记录报告时间戳         |
| 1    | msisdn      | String | 手机号码               |
| 2    | apmac       | String | AP mac                 |
| 3    | acmac       | String | AC mac                 |
| 4    | host        | String | 访问的网址             |
| 5    | siteType    | String | 网址种类               |
| 6    | upPackNum   | long   | 上行数据包数，单位：个 |
| 7    | downPackNum | long   | 下行数据报数，单位：个 |
| 8    | upPayLoad   | long   | 上行总流量。单位：byte |
| 9    | downPayLoad | long   | 下行总流量。单位：byte |
| 10   | httpStatus  | String | HTTP Response状态      |

FlowBean.java

```java
package org.duo.flowcount.count;

import org.apache.hadoop.io.Writable;

import java.io.DataInput;
import java.io.DataOutput;
import java.io.IOException;

public class FlowBean implements Writable {

    private Integer upFlow;  //上行数据包数
    private Integer downFlow;  //下行数据包数
    private Integer upCountFlow; //上行流量总和
    private Integer downCountFlow;//下行流量总和

    public Integer getUpFlow() {
        return upFlow;
    }

    public void setUpFlow(Integer upFlow) {
        this.upFlow = upFlow;
    }

    public Integer getDownFlow() {
        return downFlow;
    }

    public void setDownFlow(Integer downFlow) {
        this.downFlow = downFlow;
    }

    public Integer getUpCountFlow() {
        return upCountFlow;
    }

    public void setUpCountFlow(Integer upCountFlow) {
        this.upCountFlow = upCountFlow;
    }

    public Integer getDownCountFlow() {
        return downCountFlow;
    }

    public void setDownCountFlow(Integer downCountFlow) {
        this.downCountFlow = downCountFlow;
    }

    @Override
    public String toString() {
        return upFlow +
                "\t" + downFlow +
                "\t" + upCountFlow +
                "\t" + downCountFlow;
    }

    //序列化方法
    @Override
    public void write(DataOutput out) throws IOException {
        out.writeInt(upFlow);
        out.writeInt(downFlow);
        out.writeInt(upCountFlow);
        out.writeInt(downCountFlow);
    }

    //反序列化
    @Override
    public void readFields(DataInput in) throws IOException {
        this.upFlow = in.readInt();
        this.downFlow = in.readInt();
        this.upCountFlow = in.readInt();
        this.downCountFlow = in.readInt();
    }
}
```

FlowCountMapper.java

```java
package org.duo.flowcount.count;

import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Mapper;

import java.io.IOException;

public class FlowCountMapper extends Mapper<LongWritable, Text, Text, FlowBean> {

    /*
      将K1和V1转为K2和V2:
      K1              V1
      0               1363157985059 	13600217502	00-1F-64-E2-E8-B1:CMCC	120.196.100.55	www.baidu.com	综合门户	19	128	1177	16852	200
     ------------------------------
      K2              V2
      13600217502     FlowBean(19	128	1177	16852)
     */
    @Override
    protected void map(LongWritable key, Text value, Context context) throws IOException, InterruptedException {
        //1:拆分行文本数据,得到手机号--->K2
        String[] split = value.toString().split("\t");
        String phoneNum = split[1];

        //2:创建FlowBean对象,并从行文本数据拆分出流量的四个四段,并将四个流量字段的值赋给FlowBean对象
        FlowBean flowBean = new FlowBean();

        flowBean.setUpFlow(Integer.parseInt(split[6]));
        flowBean.setDownFlow(Integer.parseInt(split[7]));
        flowBean.setUpCountFlow(Integer.parseInt(split[8]));
        flowBean.setDownCountFlow(Integer.parseInt(split[9]));

        //3:将K2和V2写入上下文中
        context.write(new Text(phoneNum), flowBean);

    }
}
```

FlowCountReducer.java

```java
package org.duo.flowcount.count;

import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Reducer;

import java.io.IOException;

public class FlowCountReducer extends Reducer<Text, FlowBean, Text, FlowBean> {

    @Override
    protected void reduce(Text key, Iterable<FlowBean> values, Context context) throws IOException, InterruptedException {

        //1:遍历集合,并将集合中的对应的四个字段累计
        Integer upFlow = 0;  //上行数据包数
        Integer downFlow = 0;  //下行数据包数
        Integer upCountFlow = 0; //上行流量总和
        Integer downCountFlow = 0;//下行流量总和

        for (FlowBean value : values) {
            upFlow += value.getUpFlow();
            downFlow += value.getDownFlow();
            upCountFlow += value.getUpCountFlow();
            downCountFlow += value.getDownCountFlow();
        }

        //2:创建FlowBean对象,并给对象赋值  V3
        FlowBean flowBean = new FlowBean();
        flowBean.setUpFlow(upFlow);
        flowBean.setDownFlow(downFlow);
        flowBean.setUpCountFlow(upCountFlow);
        flowBean.setDownCountFlow(downCountFlow);

        //3:将K3和V3下入上下文中
        context.write(key, flowBean);
    }
}

```

JobMain.java

```java
package org.duo.flowcount.count;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.conf.Configured;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.lib.input.TextInputFormat;
import org.apache.hadoop.mapreduce.lib.output.TextOutputFormat;
import org.apache.hadoop.util.Tool;
import org.apache.hadoop.util.ToolRunner;

public class JobMain extends Configured implements Tool {

    //该方法用于指定一个job任务
    @Override
    public int run(String[] args) throws Exception {
        //1:创建一个job任务对象
        Job job = Job.getInstance(super.getConf(), "mapreduce_flowcount");
        //如果打包运行出错，则需要加该配置
        job.setJarByClass(JobMain.class);
        //2:配置job任务对象(八个步骤)

        //第一步:指定文件的读取方式和读取路径
        job.setInputFormatClass(TextInputFormat.class);
        //TextInputFormat.addInputPath(job, new Path("hdfs://node01:8020/wordcount"));
        TextInputFormat.addInputPath(job, new Path("file:///D:\\input\\flowcount_input"));

        //第二步:指定Map阶段的处理方式和数据类型
        job.setMapperClass(FlowCountMapper.class);
        //设置Map阶段K2的类型
        job.setMapOutputKeyClass(Text.class);
        //设置Map阶段V2的类型
        job.setMapOutputValueClass(FlowBean.class);


        //第三（分区），四 （排序）
        //第五步: 规约(Combiner)
        //第六步 分组


        //第七步：指定Reduce阶段的处理方式和数据类型
        job.setReducerClass(FlowCountReducer.class);
        //设置K3的类型
        job.setOutputKeyClass(Text.class);
        //设置V3的类型
        job.setOutputValueClass(FlowBean.class);

        //第八步: 设置输出类型
        job.setOutputFormatClass(TextOutputFormat.class);
        //设置输出的路径
        TextOutputFormat.setOutputPath(job, new Path("file:///D:\\out\\flowcount_out"));


        //等待任务结束
        boolean bl = job.waitForCompletion(true);

        return bl ? 0 : 1;
    }

    public static void main(String[] args) throws Exception {
        Configuration configuration = new Configuration();

        //启动job任务
        int run = ToolRunner.run(configuration, new JobMain(), args);
        System.exit(run);

    }
}
```

#### 上行流量倒序排序

以需求一的输出数据作为排序的输入数据，自定义FlowBean,以FlowBean为map输出的key，以手机号作为Map输出的value，因为MapReduce程序会对Map阶段输出的key进行排序

#### 手机号码分区

将不同的手机号分到不同的数据文件的当中去，需要自定义分区来实现，

FlowBean.java

```java
package org.duo.flowcount.partition;

import org.apache.hadoop.io.Writable;

import java.io.DataInput;
import java.io.DataOutput;
import java.io.IOException;

public class FlowBean implements Writable {

    private Integer upFlow;  //上行数据包数
    private Integer downFlow;  //下行数据包数
    private Integer upCountFlow; //上行流量总和
    private Integer downCountFlow;//下行流量总和

    public Integer getUpFlow() {
        return upFlow;
    }

    public void setUpFlow(Integer upFlow) {
        this.upFlow = upFlow;
    }

    public Integer getDownFlow() {
        return downFlow;
    }

    public void setDownFlow(Integer downFlow) {
        this.downFlow = downFlow;
    }

    public Integer getUpCountFlow() {
        return upCountFlow;
    }

    public void setUpCountFlow(Integer upCountFlow) {
        this.upCountFlow = upCountFlow;
    }

    public Integer getDownCountFlow() {
        return downCountFlow;
    }

    public void setDownCountFlow(Integer downCountFlow) {
        this.downCountFlow = downCountFlow;
    }

    @Override
    public String toString() {
        return upFlow +
                "\t" + downFlow +
                "\t" + upCountFlow +
                "\t" + downCountFlow;
    }

    //序列化方法
    @Override
    public void write(DataOutput out) throws IOException {
        out.writeInt(upFlow);
        out.writeInt(downFlow);
        out.writeInt(upCountFlow);
        out.writeInt(downCountFlow);
    }

    //反序列化
    @Override
    public void readFields(DataInput in) throws IOException {
        this.upFlow = in.readInt();
        this.downFlow = in.readInt();
        this.upCountFlow = in.readInt();
        this.downCountFlow = in.readInt();
    }
}
```

FlowCountMapper.java

```java
package org.duo.flowcount.partition;

import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Mapper;

import java.io.IOException;

public class FlowCountMapper extends Mapper<LongWritable, Text, Text, FlowBean> {

    /*
      将K1和V1转为K2和V2:
      K1              V1
      0               1363157985059 	13600217502	00-1F-64-E2-E8-B1:CMCC	120.196.100.55	www.baidu.com	综合门户	19	128	1177	16852	200
     ------------------------------
      K2              V2
      13600217502     FlowBean(19	128	1177	16852)
     */
    @Override
    protected void map(LongWritable key, Text value, Context context) throws IOException, InterruptedException {

        //1:拆分行文本数据,得到手机号--->K2
        String[] split = value.toString().split("\t");
        String phoneNum = split[1];

        //2:创建FlowBean对象,并从行文本数据拆分出流量的四个四段,并将四个流量字段的值赋给FlowBean对象
        FlowBean flowBean = new FlowBean();

        flowBean.setUpFlow(Integer.parseInt(split[6]));
        flowBean.setDownFlow(Integer.parseInt(split[7]));
        flowBean.setUpCountFlow(Integer.parseInt(split[8]));
        flowBean.setDownCountFlow(Integer.parseInt(split[9]));

        //3:将K2和V2写入上下文中
        context.write(new Text(phoneNum), flowBean);
    }
}
```

FlowCountPartition.java

```java
package org.duo.flowcount.partition;

import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Partitioner;

public class FlowCountPartition extends Partitioner<Text, FlowBean> {

    /*
      该方法用来指定分区的规则:
        135 开头数据到一个分区文件
        136 开头数据到一个分区文件
        137 开头数据到一个分区文件
        其他分区

       参数:
         text : K2   手机号
         flowBean: V2
         i   : ReduceTask的个数
     */
    @Override
    public int getPartition(Text text, FlowBean flowBean, int i) {

        //1:获取手机号
        String phoneNum = text.toString();

        //2:判断手机号以什么开头,返回对应的分区编号(0-3)
        if (phoneNum.startsWith("135")) {
            return 0;
        } else if (phoneNum.startsWith("136")) {
            return 1;
        } else if (phoneNum.startsWith("137")) {
            return 2;
        } else {
            return 3;
        }
    }
}
```

FlowCountReducer.java

```java
package org.duo.flowcount.partition;

import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Reducer;

import java.io.IOException;

public class FlowCountReducer extends Reducer<Text, FlowBean, Text, FlowBean> {

    @Override
    protected void reduce(Text key, Iterable<FlowBean> values, Context context) throws IOException, InterruptedException {

        //1:遍历集合,并将集合中的对应的四个字段累计
        Integer upFlow = 0;  //上行数据包数
        Integer downFlow = 0;  //下行数据包数
        Integer upCountFlow = 0; //上行流量总和
        Integer downCountFlow = 0;//下行流量总和

        for (FlowBean value : values) {
            upFlow += value.getUpFlow();
            downFlow += value.getDownFlow();
            upCountFlow += value.getUpCountFlow();
            downCountFlow += value.getDownCountFlow();
        }

        //2:创建FlowBean对象,并给对象赋值  V3
        FlowBean flowBean = new FlowBean();
        flowBean.setUpFlow(upFlow);
        flowBean.setDownFlow(downFlow);
        flowBean.setUpCountFlow(upCountFlow);
        flowBean.setDownCountFlow(downCountFlow);

        //3:将K3和V3下入上下文中
        context.write(key, flowBean);
    }
}
```

JobMain.java

```java
package org.duo.flowcount.partition;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.conf.Configured;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.lib.input.TextInputFormat;
import org.apache.hadoop.mapreduce.lib.output.TextOutputFormat;
import org.apache.hadoop.util.Tool;
import org.apache.hadoop.util.ToolRunner;

public class JobMain extends Configured implements Tool {

    //该方法用于指定一个job任务
    @Override
    public int run(String[] args) throws Exception {

        //1:创建一个job任务对象
        Job job = Job.getInstance(super.getConf(), "mapreduce_flow_partiton");
        //如果打包运行出错，则需要加该配置
        job.setJarByClass(JobMain.class);
        //2:配置job任务对象(八个步骤)

        //第一步:指定文件的读取方式和读取路径
        job.setInputFormatClass(TextInputFormat.class);
        //TextInputFormat.addInputPath(job, new Path("hdfs://node01:8020/wordcount"));
        TextInputFormat.addInputPath(job, new Path("file:///D:\\input\\flowpartition_input"));

        //第二步:指定Map阶段的处理方式和数据类型
        job.setMapperClass(FlowCountMapper.class);
        //设置Map阶段K2的类型
        job.setMapOutputKeyClass(Text.class);
        //设置Map阶段V2的类型
        job.setMapOutputValueClass(FlowBean.class);

        //第三（分区），四 （排序）
        job.setPartitionerClass(FlowCountPartition.class);
        //第五步: 规约(Combiner)
        //第六步 分组

        //第七步：指定Reduce阶段的处理方式和数据类型
        job.setReducerClass(FlowCountReducer.class);
        //设置K3的类型
        job.setOutputKeyClass(Text.class);
        //设置V3的类型
        job.setOutputValueClass(FlowBean.class);

        //设置reduce个数
        job.setNumReduceTasks(4);

        //第八步: 设置输出类型
        job.setOutputFormatClass(TextOutputFormat.class);
        //设置输出的路径
        TextOutputFormat.setOutputPath(job, new Path("file:///D:\\out\\flowpartiton_out"));

        //等待任务结束
        boolean bl = job.waitForCompletion(true);

        return bl ? 0 : 1;
    }

    public static void main(String[] args) throws Exception {
        Configuration configuration = new Configuration();

        //启动job任务
        int run = ToolRunner.run(configuration, new JobMain(), args);
        System.exit(run);

    }
}
```

### Reduce端实现JOIN

#### 需求

假如数据量巨大，两表的数据是以文件的形式存储在 HDFS 中, 需要用 MapReduce 程序来 实现以下 SQL 查询运算

```sql
select a.id,a.date,b.name,b.category_id,b.price from t_order a left join t_product b on a.pid = b.id
```

#### 商品表

| id    | pname  | category_id | price |
| ----- | ------ | ----------- | ----- |
| P0001 | 小米5  | 1000        | 2000  |
| P0002 | 锤子T1 | 1000        | 3000  |

#### 订单数据表

| id   | date     | pid   | amount |
| ---- | -------- | ----- | ------ |
| 1001 | 20150710 | P0001 | 2      |
| 1002 | 20150710 | P0002 | 3      |

#### 实现步骤

通过将关联的条件作为map输出的key，将两表满足join条件的数据并携带数据所来源的文件信息，发往同一个reduce task，在reduce中进行数据的串联

ReduceJoinMapper.java

```java
package org.duo.reduce_join;

import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.InputSplit;
import org.apache.hadoop.mapreduce.Mapper;
import org.apache.hadoop.mapreduce.lib.input.FileSplit;

import java.io.IOException;

/*
  K1:  LongWritable
  V1:  Text

  K2: Text  商品的id
  V2: Text  行文本信息(商品的信息)
 */
public class ReduceJoinMapper extends Mapper<LongWritable, Text, Text, Text> {

    /*
   product.txt     K1                V1
                    0                 p0001,小米5,1000,2000
   orders.txt      K1                V1
                   0                1001,20150710,p0001,2
           -------------------------------------------
                  K2                 V2
                 p0001              p0001,小米5,1000,2000
                 p0001              1001,20150710,p0001,2

     */
    @Override
    protected void map(LongWritable key, Text value, Context context) throws IOException, InterruptedException {

        //1:判断数据来自哪个文件
        FileSplit fileSplit = (FileSplit) context.getInputSplit();
        String fileName = fileSplit.getPath().getName();
        if (fileName.equals("product.txt")) {
            //数据来自商品表
            //2:将K1和V1转为K2和V2,写入上下文中
            String[] split = value.toString().split(",");
            String productId = split[0];

            context.write(new Text(productId), value);

        } else {
            //数据来自订单表
            //2:将K1和V1转为K2和V2,写入上下文中
            String[] split = value.toString().split(",");
            String productId = split[2];

            context.write(new Text(productId), value);
        }
    }
}
```

ReduceJoinReducer.java

```java
package org.duo.reduce_join;

import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Reducer;

import java.io.IOException;

public class ReduceJoinReducer extends Reducer<Text, Text, Text, Text> {

    @Override
    protected void reduce(Text key, Iterable<Text> values, Context context) throws IOException, InterruptedException {

        //1:遍历集合,获取V3 (first +second)
        String first = "";
        String second = "";
        for (Text value : values) {
            if (value.toString().startsWith("p")) {
                first = value.toString();
            } else {
                if ("".equals(second)) {
                    second = value.toString();
                } else {
                    second = second + "," + value.toString();
                }
            }
        }
        //2:将K3和V3写入上下文中
        context.write(key, new Text(first + "\t" + second));
    }
}
```

JobMain.java

```java
package org.duo.reduce_join;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.conf.Configured;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.lib.input.TextInputFormat;
import org.apache.hadoop.mapreduce.lib.output.TextOutputFormat;
import org.apache.hadoop.util.Tool;
import org.apache.hadoop.util.ToolRunner;

public class JobMain extends Configured implements Tool {

    @Override
    public int run(String[] args) throws Exception {

        //1:获取Job对象
        Job job = Job.getInstance(super.getConf(), "reduce_join");

        //2:设置job任务
        //第一步:设置输入类和输入路径
        job.setInputFormatClass(TextInputFormat.class);
        TextInputFormat.addInputPath(job, new Path("file:///D:\\input\\reduce_join_input"));

        //第二步:设置Mapper类和数据类型
        job.setMapperClass(ReduceJoinMapper.class);
        job.setMapOutputKeyClass(Text.class);
        job.setMapOutputValueClass(Text.class);

        //第三,四,五,六

        //第七步:设置Reducer类和数据类型
        job.setReducerClass(ReduceJoinReducer.class);
        job.setOutputKeyClass(Text.class);
        job.setOutputValueClass(Text.class);

        //第八步:设置输出类和输出的路径
        job.setOutputFormatClass(TextOutputFormat.class);
        TextOutputFormat.setOutputPath(job, new Path("file:///D:\\out\\reduce_join_out"));

        //3:等待job任务结束
        boolean bl = job.waitForCompletion(true);

        return bl ? 0 : 1;
    }

    public static void main(String[] args) throws Exception {
        Configuration configuration = new Configuration();

        //启动job任务
        int run = ToolRunner.run(configuration, new JobMain(), args);

        System.exit(run);
    }
}
```

### Map端实现JOIN

#### 概述

适用于关联表中有小表的情形.使用分布式缓存,可以将小表分发到所有的map节点，这样，map节点就可以在本地对自己所读到的大表数据进行join并输出最终结果，可以大大提高join操作的并发度，加快处理速度

#### 实现步骤

先在mapper类中预先定义好小表，进行join

MapJoinMapper.java

```java
package org.duo.map_join;

import org.apache.hadoop.fs.FSDataInputStream;
import org.apache.hadoop.fs.FileSystem;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Mapper;

import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;
import java.net.URI;
import java.util.HashMap;

public class MapJoinMapper extends Mapper<LongWritable, Text, Text, Text> {

    private HashMap<String, String> map = new HashMap<>();

    //第一件事情:将分布式缓存的小表数据读取到本地Map集合(setup方法只会执行一次)
    @Override
    protected void setup(Context context) throws IOException, InterruptedException {

        //1:获取分布式缓存文件列表
        URI[] cacheFiles = context.getCacheFiles();
        //2:获取指定的分布式缓存文件的文件系统(FileSystem)
        FileSystem fileSystem = FileSystem.get(cacheFiles[0], context.getConfiguration());
        //3:获取文件的输入流
        FSDataInputStream inputStream = fileSystem.open(new Path(cacheFiles[0]));
        //4:读取文件内容, 并将数据存入Map集合
        //4.1 将字节输入流转为字符缓冲流FSDataInputStream --->BufferedReader
        BufferedReader bufferedReader = new BufferedReader(new InputStreamReader(inputStream));
        //4.2 读取小表文件内容,以行位单位,并将读取的数据存入map集合
        String line = null;
        while ((line = bufferedReader.readLine()) != null) {
            String[] split = line.split(",");
            map.put(split[0], line);
        }
        //5:关闭流
        bufferedReader.close();
        fileSystem.close();
    }

    //第二件事情:对大表的处理业务逻辑,而且要实现大表和小表的join操作
    @Override
    protected void map(LongWritable key, Text value, Context context) throws IOException, InterruptedException {

        //1:从行文本数据中获取商品的id: p0001 , p0002  得到了K2
        String[] split = value.toString().split(",");
        String productId = split[2];  //K2

        //2:在Map集合中,将商品的id作为键,获取值(商品的行文本数据) ,将value和值拼接,得到V2
        String productLine = map.get(productId);
        String valueLine = productLine + "\t" + value.toString(); //V2
        //3:将K2和V2写入上下文中
        context.write(new Text(productId), new Text(valueLine));
    }
}
```

JobMain.java

```java
package org.duo.map_join;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.conf.Configured;
import org.apache.hadoop.filecache.DistributedCache;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.lib.input.TextInputFormat;
import org.apache.hadoop.mapreduce.lib.output.TextOutputFormat;
import org.apache.hadoop.util.Tool;
import org.apache.hadoop.util.ToolRunner;

import java.net.URI;

public class JobMain extends Configured implements Tool {

    @Override
    public int run(String[] args) throws Exception {

        //1:获取job对象
        Job job = Job.getInstance(super.getConf(), "map_join_job");

        //2:设置job对象(将小表放在分布式缓存中)
        //将小表放在分布式缓存中
        // DistributedCache.addCacheFile(new URI("hdfs://node01:8020/cache_file/product.txt"), super.getConf());
        job.addCacheFile(new URI("hdfs://node01:8020/cache_file/product.txt"));

        //第一步:设置输入类和输入的路径
        job.setInputFormatClass(TextInputFormat.class);
        TextInputFormat.addInputPath(job, new Path("file:///D:\\input\\map_join_input"));
        //第二步:设置Mapper类和数据类型
        job.setMapperClass(MapJoinMapper.class);
        job.setMapOutputKeyClass(Text.class);
        job.setMapOutputValueClass(Text.class);

        //第八步:设置输出类和输出路径
        job.setOutputFormatClass(TextOutputFormat.class);
        TextOutputFormat.setOutputPath(job, new Path("file:///D:\\out\\map_join_out"));

        //3:等待任务结束
        boolean bl = job.waitForCompletion(true);
        return bl ? 0 : 1;
    }

    public static void main(String[] args) throws Exception {
        Configuration configuration = new Configuration();
        //启动job任务
        int run = ToolRunner.run(configuration, new JobMain(), args);
        System.exit(run);
    }
}
```

### 自定义InputFormat

#### 需求

无论hdfs还是mapreduce，对于小文件都有损效率，实践中，又难免面临处理大量小文件的场景，此时，就需要有相应解决方案

#### 分析

小文件的优化无非以下几种方式：

1. 在数据采集的时候，就将小文件或小批数据合成大文件再上传HDFS
2. 在业务处理之前，在HDFS上使用mapreduce程序对小文件进行合并
3. 在mapreduce处理时，可采用combineInputFormat提高效率

#### 实现

本例实现的是上述第二种方式程序的核心机制：自定义一个InputFormat，改写RecordReader，实现一次读取一个完整文件封装为KV，在输出时使用SequenceFileOutPutFormat输出合并文件

MyRecordReader.java

```java
package org.duo.inputformat;

import org.apache.commons.io.IOUtils;
import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.FSDataInputStream;
import org.apache.hadoop.fs.FileSystem;
import org.apache.hadoop.io.BytesWritable;
import org.apache.hadoop.io.NullWritable;
import org.apache.hadoop.mapreduce.InputSplit;
import org.apache.hadoop.mapreduce.RecordReader;
import org.apache.hadoop.mapreduce.TaskAttemptContext;
import org.apache.hadoop.mapreduce.lib.input.FileSplit;

import java.io.IOException;

public class MyRecordReader extends RecordReader<NullWritable, BytesWritable> {

    private Configuration configuration = null;
    private FileSplit fileSplit = null;
    private boolean processed = false;
    private BytesWritable bytesWritable = new BytesWritable();
    private FileSystem fileSystem = null;
    private FSDataInputStream inputStream = null;

    //进行初始化工作
    @Override
    public void initialize(InputSplit inputSplit, TaskAttemptContext taskAttemptContext) throws IOException, InterruptedException {

        //获取文件的切片
        fileSplit = (FileSplit) inputSplit;
        //获取Configuration对象
        configuration = taskAttemptContext.getConfiguration();
    }

    //该方法用于获取K1和V1
    /*
     K1: NullWritable
     V1: BytesWritable
     */
    @Override
    public boolean nextKeyValue() throws IOException, InterruptedException {

        if (!processed) {
            //1:获取源文件的字节输入流
            //1.1 获取源文件的文件系统 (FileSystem)
            fileSystem = FileSystem.get(configuration);
            //1.2 通过FileSystem获取文件字节输入流
            inputStream = fileSystem.open(fileSplit.getPath());
            //2:读取源文件数据到普通的字节数组(byte[])
            byte[] bytes = new byte[(int) fileSplit.getLength()];
            IOUtils.readFully(inputStream, bytes, 0, (int) fileSplit.getLength());
            //3:将字节数组中数据封装到BytesWritable ,得到v1
            bytesWritable.set(bytes, 0, (int) fileSplit.getLength());
            processed = true;
            return true;
        }
        return false;
    }

    //返回K1
    @Override
    public NullWritable getCurrentKey() throws IOException, InterruptedException {
        return NullWritable.get();
    }

    //返回V1
    @Override
    public BytesWritable getCurrentValue() throws IOException, InterruptedException {
        return bytesWritable;
    }

    //获取文件读取的进度
    @Override
    public float getProgress() throws IOException, InterruptedException {
        return 0;
    }

    //进行资源释放
    @Override
    public void close() throws IOException {
        inputStream.close();
        fileSystem.close();
    }
}
```

MyInputFormat.java

```java
package org.duo.inputformat;

import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.BytesWritable;
import org.apache.hadoop.io.NullWritable;
import org.apache.hadoop.mapreduce.InputSplit;
import org.apache.hadoop.mapreduce.JobContext;
import org.apache.hadoop.mapreduce.RecordReader;
import org.apache.hadoop.mapreduce.TaskAttemptContext;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;

import java.io.IOException;

public class MyInputFormat extends FileInputFormat<NullWritable, BytesWritable> {

    @Override
    public RecordReader<NullWritable, BytesWritable> createRecordReader(InputSplit inputSplit, TaskAttemptContext taskAttemptContext) throws IOException, InterruptedException {

        //1:创建自定义RecordReader对象
        MyRecordReader myRecordReader = new MyRecordReader();
        //2:将inputSplit和context对象传给MyRecordReader
        myRecordReader.initialize(inputSplit, taskAttemptContext);
        return myRecordReader;
    }

    /*
     设置文件是否可以被切割
     */
    @Override
    protected boolean isSplitable(JobContext context, Path filename) {
        return false;
    }
}
```

SequenceFileMapper.java

```java
package org.duo.inputformat;

import org.apache.hadoop.io.BytesWritable;
import org.apache.hadoop.io.NullWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.InputSplit;
import org.apache.hadoop.mapreduce.Mapper;
import org.apache.hadoop.mapreduce.lib.input.FileSplit;

import java.io.IOException;

public class SequenceFileMapper extends Mapper<NullWritable, BytesWritable, Text, BytesWritable> {

    @Override
    protected void map(NullWritable key, BytesWritable value, Context context) throws IOException, InterruptedException {

        //1:获取文件的名字,作为K2
        FileSplit fileSplit = (FileSplit) context.getInputSplit();
        String fileName = fileSplit.getPath().getName();
        //2:将K2和V2写入上下文中
        context.write(new Text(fileName), value);
    }
}
```

JobMain.java

```java
package org.duo.inputformat;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.conf.Configured;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.BytesWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.lib.input.TextInputFormat;
import org.apache.hadoop.mapreduce.lib.output.SequenceFileOutputFormat;
import org.apache.hadoop.mapreduce.lib.output.TextOutputFormat;
import org.apache.hadoop.util.Tool;
import org.apache.hadoop.util.ToolRunner;

public class JobMain extends Configured implements Tool {

    @Override
    public int run(String[] args) throws Exception {

        //1:获取job对象
        Job job = Job.getInstance(super.getConf(), "sequence_file_job");

        //2:设置job任务
        //第一步:设置输入类和输入的路径
        job.setInputFormatClass(MyInputFormat.class);
        MyInputFormat.addInputPath(job, new Path("file:///D:\\input\\myInputformat_input"));

        //第二步:设置Mapper类和数据类型
        job.setMapperClass(SequenceFileMapper.class);
        job.setMapOutputKeyClass(Text.class);
        job.setMapOutputValueClass(BytesWritable.class);

        //第七步: 不需要设置Reducer类,但是必须设置数据类型
        job.setOutputKeyClass(Text.class);
        job.setOutputValueClass(BytesWritable.class);

        //第八步:设置输出类和输出的路径
        job.setOutputFormatClass(SequenceFileOutputFormat.class);
        SequenceFileOutputFormat.setOutputPath(job, new Path("file:///D:\\out\\myinputformat_out"));

        //3:等待job任务执行结束
        boolean bl = job.waitForCompletion(true);
        return bl ? 0 : 1;
    }

    public static void main(String[] args) throws Exception {

        Configuration configuration = new Configuration();
        int run = ToolRunner.run(configuration, new JobMain(), args);
        System.exit(run);
    }
}
```

### 自定义outputFormat

#### 需求

现在有一些订单的评论数据，需求，将订单的好评与差评进行区分开来，将最终的数据分开到不同的文件夹下面去（分区是输出到不同文件，这里的需求是要输出到不同的文件夹），其中数据第九个字段表示好评，中评，差评。0：好评，1：中评，2：差评

#### 分析

程序的关键点是要在一个mapreduce程序中根据数据的不同输出两类结果到不同目录，这类灵活的输出需求可以通过自定义outputformat来实现

#### 实现

1. 在mapreduce中访问外部资源
2. 自定义outputformat，改写其中的recordwriter，改写具体输出数据的方法write()

MyRecordWriter.java

```java
package org.duo.outputformat;

import org.apache.hadoop.fs.FSDataOutputStream;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.IOUtils;
import org.apache.hadoop.io.NullWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.RecordWriter;
import org.apache.hadoop.mapreduce.TaskAttemptContext;

import java.io.IOException;

public class MyRecordWriter extends RecordWriter<Text, NullWritable> {

    private FSDataOutputStream goodCommentsOutputStream;
    private FSDataOutputStream badCommentsOutputStream;

    public MyRecordWriter() {
    }

    public MyRecordWriter(FSDataOutputStream goodCommentsOutputStream, FSDataOutputStream badCommentsOutputStream) {
        this.goodCommentsOutputStream = goodCommentsOutputStream;
        this.badCommentsOutputStream = badCommentsOutputStream;
    }

    /**
     * @param text         行文本内容
     * @param nullWritable
     * @throws IOException
     * @throws InterruptedException
     */
    @Override
    public void write(Text text, NullWritable nullWritable) throws IOException, InterruptedException {

        //1:从行文本数据中获取第9个字段
        String[] split = text.toString().split("\t");
        String numStr = split[9];

        //2:根据字段的值,判断评论的类型,然后将对应的数据写入不同的文件夹文件中
        if (Integer.parseInt(numStr) <= 1) {
            //好评或者中评
            goodCommentsOutputStream.write(text.toString().getBytes());
            goodCommentsOutputStream.write("\r\n".getBytes());
        } else {
            //差评
            badCommentsOutputStream.write(text.toString().getBytes());
            badCommentsOutputStream.write("\r\n".getBytes());
        }
    }

    @Override
    public void close(TaskAttemptContext taskAttemptContext) throws IOException, InterruptedException {
        IOUtils.closeStream(goodCommentsOutputStream);
        IOUtils.closeStream(badCommentsOutputStream);
    }
}
```

MyOutputFormat.java

```java
package org.duo.outputformat;

import org.apache.hadoop.fs.FSDataOutputStream;
import org.apache.hadoop.fs.FileSystem;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.NullWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.RecordWriter;
import org.apache.hadoop.mapreduce.TaskAttemptContext;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;

import java.io.IOException;

public class MyOutputFormat extends FileOutputFormat<Text, NullWritable> {

    @Override
    public RecordWriter<Text, NullWritable> getRecordWriter(TaskAttemptContext taskAttemptContext) throws IOException, InterruptedException {

        //1:获取目标文件的输出流(两个)
        FileSystem fileSystem = FileSystem.get(taskAttemptContext.getConfiguration());
        FSDataOutputStream goodCommentsOutputStream = fileSystem.create(new Path("file:///D:\\out\\good_comments\\good_comments.txt"));
        FSDataOutputStream badCommentsOutputStream = fileSystem.create(new Path("file:///D:\\out\\bad_comments\\bad_comments.txt"));
        //2:将输出流传给MyRecordWriter
        MyRecordWriter myRecordWriter = new MyRecordWriter(goodCommentsOutputStream, badCommentsOutputStream);
        return myRecordWriter;
    }
}
```

MyOutputFormatMapper.java

```java
package org.duo.outputformat;

import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.NullWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Mapper;

import java.io.IOException;

public class MyOutputFormatMapper extends Mapper<LongWritable, Text, Text, NullWritable> {

    @Override
    protected void map(LongWritable key, Text value, Context context) throws IOException, InterruptedException {
        context.write(value, NullWritable.get());
    }
}
```

JobMain.java

```java
package org.duo.outputformat;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.conf.Configured;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.NullWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.lib.input.TextInputFormat;
import org.apache.hadoop.util.Tool;
import org.apache.hadoop.util.ToolRunner;

public class JobMain extends Configured implements Tool {

    @Override
    public int run(String[] args) throws Exception {

        //1:获取job对象
        Job job = Job.getInstance(super.getConf(), "myoutputformat_job");

        //2:设置job任务
        //第一步:设置输入类和输入的路径
        job.setInputFormatClass(TextInputFormat.class);
        TextInputFormat.addInputPath(job, new Path("file:///D:\\input\\myoutputformat_input"));

        //第二步:设置Mapper类和数据类型
        job.setMapperClass(MyOutputFormatMapper.class);
        job.setMapOutputKeyClass(Text.class);
        job.setMapOutputValueClass(NullWritable.class);

        //第八步:设置输出类和输出的路径
        job.setOutputFormatClass(MyOutputFormat.class);
        MyOutputFormat.setOutputPath(job, new Path("file:///D:\\out\\myoutputformat_out"));

        //3:等待任务结束
        boolean bl = job.waitForCompletion(true);
        return bl ? 0 : 1;
    }

    public static void main(String[] args) throws Exception {

        Configuration configuration = new Configuration();
        int run = ToolRunner.run(configuration, new JobMain(), args);
        System.exit(run);
    }
}
```

### 自定义分组

分组是mapreduce当中reduce端的一个功能组件，主要的作用是决定哪些数据作为一组，调用一次reduce的逻辑，默认是每个不同的key，作为多个不同的组，每个组调用一次reduce逻辑，我们可以自定义分组实现不同的key作为同一个组，调用一次reduce逻辑

#### 需求

有如下订单数据

| 订单id        | 商品id | 成交金额 |
| ------------- | ------ | -------- |
| Order_0000001 | Pdt_01 | 222.8    |
| Order_0000001 | Pdt_05 | 25.8     |
| Order_0000002 | Pdt_03 | 522.8    |
| Order_0000002 | Pdt_04 | 122.4    |
| Order_0000002 | Pdt_05 | 722.4    |
| Order_0000003 | Pdt_01 | 222.8    |

现在需要求出每一个订单中成交金额最大的一笔交易（或者求交易金额的topN）

#### 分析

1. 利用“订单id和成交金额”作为key，可以将map阶段读取到的所有订单数据按照id分区，按照金额排序，发送到reduce
2. 在reduce端利用分组将订单id相同的kv聚合成组，然后取第一个即是最大值

#### 实现

##### 定义OrderBean

定义一个OrderBean，里面定义两个字段，第一个字段是orderId，第二个字段是金额（注意金额一定要使用Double或者DoubleWritable类型，否则没法按照金额顺序排序）

OrderBean.java

```java
package org.duo.grouping;

import org.apache.hadoop.io.WritableComparable;

import java.io.DataInput;
import java.io.DataOutput;
import java.io.IOException;

public class OrderBean implements WritableComparable<OrderBean> {

    private String orderId;
    private Double price;

    public String getOrderId() {
        return orderId;
    }

    public void setOrderId(String orderId) {
        this.orderId = orderId;
    }

    public Double getPrice() {
        return price;
    }

    public void setPrice(Double price) {
        this.price = price;
    }

    @Override
    public String toString() {
        return orderId + "\t" + price;
    }

    //指定排序规则
    @Override
    public int compareTo(OrderBean orderBean) {
        //先比较订单ID,如果订单ID一致,则排序订单金额(降序)
        int i = this.orderId.compareTo(orderBean.orderId);
        if (i == 0) {
            i = this.price.compareTo(orderBean.price) * -1;
        }

        return i;
    }

    //实现对象的序列化
    @Override
    public void write(DataOutput out) throws IOException {
        out.writeUTF(orderId);
        out.writeDouble(price);
    }

    //实现对象反序列化
    @Override
    public void readFields(DataInput in) throws IOException {
        this.orderId = in.readUTF();
        this.price = in.readDouble();
    }
}
```

##### 定义Mapper类

GroupMapper.java

```java
package org.duo.grouping;

import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Mapper;

import java.io.IOException;

public class GroupMapper extends Mapper<LongWritable, Text, OrderBean, Text> {

    @Override
    protected void map(LongWritable key, Text value, Context context) throws IOException, InterruptedException {

        //1:拆分行文本数据,得到订单的ID,订单的金额
        String[] split = value.toString().split("\t");

        //2:封装OrderBean,得到K2
        OrderBean orderBean = new OrderBean();
        orderBean.setOrderId(split[0]);
        orderBean.setPrice(Double.valueOf(split[2]));

        //3:将K2和V2写入上下文中
        context.write(orderBean, value);
    }
}
```

##### 自定义分区

自定义分区，按照订单id进行分区，把所有订单id相同的数据，都发送到同一个reduce中去

OrderPartition.java

```java
package org.duo.grouping;

import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Partitioner;

public class OrderPartition extends Partitioner<OrderBean, Text> {

    /**
     * 分区规则: 根据订单的ID实现分区
     * 如果不自定义OrderPartition的话，默认会按照orderBean的hashCode来分区，那么相同的订单id就会被分在不同的分区
     *
     * @param orderBean K2
     * @param text      V2
     * @param i         ReduceTask个数
     * @return 返回分区的编号
     */
    @Override
    public int getPartition(OrderBean orderBean, Text text, int i) {
        return (orderBean.getOrderId().hashCode() & 2147483647) % i;
    }
}
```

##### 自定义分组

按照自己的逻辑进行分组，通过比较相同的订单id，将相同的订单id放到一个组里面去，进过分组之后当中的数据，已经全部是排好序的数据，只需要取前topN即可

OrderGroupComparator.java

```java
package org.duo.grouping;

/*

  1: 继承WriteableComparator
  2: 调用父类的有参构造
  3: 指定分组的规则(重写方法)
 */

import org.apache.hadoop.io.WritableComparable;
import org.apache.hadoop.io.WritableComparator;

// 1: 继承WriteableComparator
public class OrderGroupComparator extends WritableComparator {

    // 2: 调用父类的有参构造
    public OrderGroupComparator() {
        super(OrderBean.class, true);
    }

    //3: 指定分组的规则(重写方法)
    @Override
    public int compare(WritableComparable a, WritableComparable b) {

        //3.1 对形参做强制类型转换
        OrderBean first = (OrderBean) a;
        OrderBean second = (OrderBean) b;
        //3.2 指定分组规则
        return first.getOrderId().compareTo(second.getOrderId());
    }
}
```

##### 定义Reducer类

GroupReducer.java

```java
package org.duo.grouping;

import org.apache.hadoop.io.NullWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Reducer;

import java.io.IOException;

public class GroupReducer extends Reducer<OrderBean, Text, Text, NullWritable> {

    @Override
    protected void reduce(OrderBean key, Iterable<Text> values, Context context) throws IOException, InterruptedException {

        int i = 0;
        for (Text value : values) {
            context.write(value, NullWritable.get());
            i++;
            if (i >= 2) {
                break;
            }
        }
    }
}
```

##### main函数

JobMain.java

```java
package org.duo.grouping;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.conf.Configured;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.NullWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.lib.input.TextInputFormat;
import org.apache.hadoop.mapreduce.lib.output.TextOutputFormat;
import org.apache.hadoop.util.Tool;
import org.apache.hadoop.util.ToolRunner;

public class JobMain extends Configured implements Tool {

    @Override
    public int run(String[] args) throws Exception {

        //1:获取Job对象
        Job job = Job.getInstance(super.getConf(), "mygroup_job");

        //2:设置job任务
        //第一步:设置输入类和输入路径
        job.setInputFormatClass(TextInputFormat.class);
        TextInputFormat.addInputPath(job, new Path("file:///D:\\input\\mygroup_input"));

        //第二步:设置Mapper类和数据类型
        job.setMapperClass(GroupMapper.class);
        job.setMapOutputKeyClass(OrderBean.class);
        job.setMapOutputValueClass(Text.class);

        //第三,四,五,六
        //设置分区
        job.setPartitionerClass(OrderPartition.class);
        //设置分组
        job.setGroupingComparatorClass(OrderGroupComparator.class);

        //第七步:设置Reducer类和数据类型
        job.setReducerClass(GroupReducer.class);
        job.setOutputKeyClass(Text.class);
        job.setOutputValueClass(NullWritable.class);

        //第八步:设置输出类和输出的路径
        job.setOutputFormatClass(TextOutputFormat.class);
        TextOutputFormat.setOutputPath(job, new Path("file:///D:\\out\\mygroup_out"));

        //3:等待job任务结束
        boolean bl = job.waitForCompletion(true);

        return bl ? 0 : 1;
    }

    public static void main(String[] args) throws Exception {

        Configuration configuration = new Configuration();
        //启动job任务
        int run = ToolRunner.run(configuration, new JobMain(), args);
        System.exit(run);
    }
}
```

## yarn资源调度

### 介绍

yarn是hadoop集群当中的资源管理系统模块，从hadoop2.0开始引入yarn模块,yarn可为各类计算框架提供资源的管理和调度,主要用于管理集群当中的资源（主要是服务器的各种硬件资源，包括CPU，内存，磁盘，网络IO等）以及调度运行在yarn上面的各种任务。yarn核心出发点是为了分离资源管理与作业监控，实现分离的做法是拥有一个全局的资源管理（ResourceManager，RM），以及每个应用程序对应一个的应用管理器（ApplicationMaster，AM）总结一句话就是说：yarn主要就是为了调度资源，管理任务等。

### 组件

YARN总体上是Master/Slave结构 ，主要由ResourceManager、NodeManager、 ApplicationMaster和Container等几个组件构成。

#### ResourceManager(RM) 

负责处理客户端请求,对各NM上的资源进行统一管理和调度。给ApplicationMaster分配空闲的Container运行并监控其运行状态。主要由两个组件构成：调度器和应用程序管理器：

1. 调度器(Scheduler)：调度器根据容量、队列等限制条件，将系统中的资源分配给各个正在运行的应用程序。调度器仅根据各个应用程序的资源需求进行资源分配，而资源分配单位是Container。Shceduler不负责监控或者跟踪应用程序的状态。总之，调度器根据应用程序的资源要求，以及集群机器的资源情况，为应用程序分配封装在Container中的资源。

   hadoop支持的调度方式

   1. FIFO Scheduler

      把任务按提交的顺序排成一个队列，这是一个先进先出队列，在进行资源分配的时候，先给队列中最头上的任务进行分配资源，待最头上任务需求满足后再给下一个分配，以此类推。FIFOScheduler是最简单也是最容易理解的调度器，也不需要任何配置，但它并不适用于共享集群。大的任务可能会占用所有集群资源，这就导致其它任务被阻塞。

   2. Capacity Scheduler

      Capacity调度器允许多个组织共享整个集群，每个组织可以获得集群的一部分计算能力。通过为每个组织分配专门的队列，然后再为每个队列分配一定的集群资源，这样整个集群就可以通过设置多个队列的方式给多个组织提供服务了。除此之外，队列内部又可以垂直划分，这样一个组织内部的多个成员就可以共享这个队列资源了，在一个队列内部，资源的调度是采用的是先进先出(FIFO)策略。（容量调度器是apache版本默认使用的调度器）

   3. Fair Scheduler

      Fair调度器的设计目标是为所有的应用分配公平的资源（对公平的定义可以通过参数来设置）。公平调度在也可以在多个队列间工作。举个例子，假设有两个用户A和B，他们分别拥有一个队列。当A启动一个job而B没有任务时，A会获得全部集群资源；当B启动一个job后，A的job会继续运行，不过一会儿之后两个任务会各自获得一半的集群资源。如果此时B再启动第二个job并且其它job还在运行，则它将会和B的第一个job共享B这个队列的资源，也就是B的两个job会用于四分之一的集群资源，而A的job仍然用于集群一半的资源，结果就是资源最终在两个用户之间平等的共享。（公平调度器，CDH版本的hadoop默认使用的调度器）

   使用哪种调度器取决于yarn-site.xml当中的：yarn.resourcemanager.scheduler.class 这个属性的配置

2. 应用程序管理器(Applications Manager)：应用程序管理器负责管理整个系统中所有应用程序，包括应用程序提交、与调度器协商资源以启动ApplicationMaster、监控ApplicationMaster运行状态并在失败时重新启动等，跟踪分给的Container的进度、状态也是其职责。

#### NodeManager(NM)

NodeManager是每个节点上的资源和任务管理器。它会定时地向ResourceManager汇报本节点上的资源使用情况和各个Container的运行状态；同时会接收并处理来自ApplicationMaster的Container启动/停止等请求。

#### ApplicationMaster(AM)

用户提交的应用程序均包含一个ApplicationMaster，负责应用的监控，跟踪应用执行状态，重启失败任务等。ApplicationMaster是应用框架，它负责向ResourceManager协调资源，并且与NodeManager协同工作完成Task的执行和监控。

#### Container

Container是YARN中的资源抽象，它封装了某个节点上的多维度资源，如内存、CPU、磁盘、网络等，当ApplicationMaster向ResourceManager申请资源时，ResourceManager为ApplicationMaster返回的资源便是用Container表示的。

### 参数

设置container分配最小内存，给应用程序container分配的最小内存

yarn.scheduler.minimum-allocation-mb 1024 

设置container分配最大内存，给应用程序container分配的最大内存

yarn.scheduler.maximum-allocation-mb 8192 

设置每个container的最小虚拟内核个数，每个container默认给分配的最小的虚拟内核个数

yarn.scheduler.minimum-allocation-vcores 1 

设置每个container的最大虚拟内核个数，每个container可以分配的最大的虚拟内核的个数

yarn.scheduler.maximum-allocation-vcores 32 

设置NodeManager可以分配的内存大小，nodemanager可以分配的最大内存大小，默认8192Mb

yarn.nodemanager.resource.memory-mb 8192 

定义每台机器的内存使用大小

yarn.nodemanager.resource.memory-mb 8192

定义交换区空间可以使用的大小，交换区空间就是讲一块硬盘拿出来做内存使用,这里指定的是nodemanager的2.1倍

yarn.nodemanager.vmem-pmem-ratio 2.1 

# Hive

## 数据仓库

### 概念

英文名称为Data Warehouse，可简写为DW或DWH。数据仓库的目的是构建面向分析的集成化数据环境，为企业提供决策支持（Decision Support）。数据仓库是存数据的，企业的各种数据往里面存，主要目的是为了分析有效数据，后续会基于它产出供分析挖掘的数据，或者数据应用需要的数据，如企业的分析性报告和各类报表等。可以理解为：面向分析的存储系统。

### 特征

数据仓库是面向主题的（Subject-Oriented ）、集成的（Integrated）、非易失的（Non-Volatile）和时变的（Time-Variant ）数据集合，用以支持管理决策。

#### 面向主题

数据仓库是面向主题的,数据仓库通过一个个主题域将多个业务系统的数据加载到一起，为了各个主题（如：用户、订单、商品等）进行分析而建，操作型数据库是为了支撑各种业务而建立。

#### 集成性

数据仓库会将不同源数据库中的数据汇总到一起,数据仓库中的综合数据不能从原有的数据库系统直接得到。因此在数据进入数据仓库之前，必然要经过统一与整合，这一步是数据仓库建设中最关键、最复杂的一步(ETL)，要统一源数据中所有矛盾之处，如字段的同名异义、异名同义、单位不统一、字长不一致，等等。

ETL，是英文Extract-Transform-Load的缩写，用来描述将数据从来源端经过抽取（extract）、转换（transform）、加载（load）至目的端的过程。ETL一词较常用在数据仓库，但其对象并不限于数据仓库。

#### 非易失性

操作型数据库主要服务于日常的业务操作，使得数据库需要不断地对数据实时更新，以便迅速获得当前最新数据，不至于影响正常的业务运作。在数据仓库中只要保存过去的业务数据，不需要每一笔业务都实时更新数据仓库，而是根据商业需要每隔一段时间把一批较新的数据导入数据仓库。数据仓库的数据反映的是一段相当长的时间内历史数据的内容，是不同时点的数据库的集合，以及基于这些快照进行统计、综合和重组的导出数据。数据仓库中的数据一般仅执行查询操作，很少会有删除和更新。但是需定期加载和刷新数据。

#### 时变性

数据仓库包含各种粒度的历史数据。数据仓库中的数据可能与某个特定日期、星期、月份、季度或者年份有关。数据仓库的目的是通过分析企业过去一段时间业务的经营状况，挖掘其中隐藏的模式。虽然数据仓库的用户不能修改数据，但并不是说数据仓库的数据是永远不变的。分析的结果只能反映过去的情况，当业务变化后，挖掘出的模式会失去时效性。因此数据仓库的数据需要定时更新，以适应决策的需要。

### 数据库与数据仓库的区别

数据库与数据仓库的区别实际讲的是OLTP与OLAP 的区别。

#### 操作型处理

联机事务处理OLTP（On-Line Transaction Processing，），也可以称面向交易的处理系统，它是针对具体业务在数据库联机的日常操作，通常对少数记录进行查询、修改。用户较为关心操作的响应时间、数据的安全性、完整性和并发支持的用户数等问题。传统的数据库系统作为数据管理的主要手段，主要用于操作型处理。

#### 分析型处理

联机分析处理 OLAP（On-Line Analytical Processing）一般针对某些主题的历史数据进行分析，支持管理决策。

数据仓库的出现，并不是要取代数据库。

1. 数据库是面向事务的设计，数据仓库是面向主题设计的。
2. 数据库一般存储业务数据，数据仓库存储的一般是历史数据。
3. 数据库设计是尽量避免冗余，一般针对某一业务应用进行设计，比如一张简单的User表，记录用户名、密码等简单数据即可，符合业务应用，但是不符合分析。数据仓库在设计是有意引入冗余，依照分析需求，分析维度、分析指标进行设计。
4. 数据库是为捕获数据而设计，数据仓库是为分析数据而设计。

数据仓库，是在数据库已经大量存在的情况下，为了进一步挖掘数据资源、为了决策需要而产生的，它决不是所谓的“大型数据库”

### 分层架构

按照数据流入流出的过程，数据仓库架构可分为三层——源数据、数据仓库、数据应用。数据仓库的数据来源于不同的源数据，并提供多样的数据应用，数据自下而上流入数据仓库后向上层开放应用，而数据仓库只是中间集成化数据管理的一个平台。

#### 源数据层（ODS）

此层数据无任何更改，直接沿用外围系统数据结构和数据，不对外开 放；为临时存储层，是接口数据的临时存储区域，为后一步的数据处理做准备。来源一般包括：点击流日志(Click Stream)、数据库数据(OLTP)、文档数据(Documents)、其它。

#### 数据仓库层（DW）

也称为细节层，DW层的数据应该是一致的、准确的、干净的数据，即对源系统数据进行了清洗（去除了杂质）后的数据。

#### 数据应用层（DA或APP）

前端应用直接读取的数据源；根据报表、专题分析需求而计算 生成的数据。

数据仓库从各数据源获取数据及在数据仓库内的数据转换和流动都可以认为是ETL（抽取 Extra, 转化Transfer, 装载Load）的过程，ETL是数据仓库的流水线，也可以认为是数据仓库的 血液，它维系着数据仓库中数据的新陈代谢，而数据仓库日常的管理和维护工作的大部分精 力就是保持ETL的正常和稳定。

## Hive的基本概念

### 简介

Hive是基于Hadoop的一个数据仓库工具，可以将结构化的数据文件映射为一张数据库表，并提供类SQL查询功能。其本质是将SQL转换为MapReduce的任务进行运算，底层由HDFS来提供数据的存储，说白了hive可以理解为一个将SQL转换为MapReduce的任务的工具，甚至更进一步可以说hive就是一个MapReduce的客户端。

1. 采用类SQL语法去操作数据，提供快速开发的能力。
2. 避免了去写MapReduce，减少开发人员的学习成本。
3. 功能扩展很方便。

### Hive与Hadoop的关系

Hive利用HDFS存储数据，利用MapReduce查询分析数据

```mermaid
graph LR
  A(客户端)-->|发出sql|B[Hive处理转换成MapReduce]
  B[Hive处理转换成MapReduce]-->|提交任务到Hadoop|C(MapReduce运行)
```

## 安装

下载地址：

http://archive.apache.org/dist/hive/

### 上传并解压安装包

### 安装mysql

### 修改hive的配置文件

hive-env.sh

```sh
HADOOP_HOME=/usr/local/hadoop-2.7.5/
export HIVE_CONF_DIR=/usr/local/hive/conf
```

hive-site.xml

$HIVE_CONF_DIR目录下只有hive-default.xml.template文件，用户如果没有自定义配置文件的话就使用hive-default.xml

```xml
<?xml version="1.0" encoding="UTF-8" standalone="no"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<configuration>
<property>
      <name>javax.jdo.option.ConnectionUserName</name>
      <value>root</value>
  </property>
  <property>
      <name>javax.jdo.option.ConnectionPassword</name>
      <value>123456</value>
  </property>
  <property>
      <name>javax.jdo.option.ConnectionURL</name>
      <value>jdbc:mysql://server01:3306/hive?createDatabaseIfNotExist=true&amp;useSSL=false</value>
  </property>
  <property>
      <name>javax.jdo.option.ConnectionDriverName</name>
      <value>com.mysql.jdbc.Driver</value>
  </property>
  <property>
      <name>hive.metastore.schema.verification</name>
      <value>false</value>
  </property>
  <property>
    <name>datanucleus.schema.autoCreateAll</name>
    <value>true</value>
  </property>
  <property>
    <name>hive.server2.thrift.bind.host</name>
    <value>server01</value>
  </property>
  <!--设置reduce个数-->
  <property>
    <name>mapreduce.job.reduces</name>
    <value>3</value>
  </property>
  <!-- 开启Map输出阶段压缩-->
  <property>
    <name>hive.exec.compress.intermediate</name>
    <value>true</value>
  </property>
  <property>
    <name>mapreduce.map.output.compress</name>
    <value>true</value>
  </property>
  <property>
    <name>mapreduce.map.output.compress.codec</name>
    <value>org.apache.hadoop.io.compress.SnappyCodec</value>
  </property>
  <!--开启Reduce输出阶段压缩-->
  <property>
    <name>hive.exec.compress.output</name>
    <value>true</value>
  </property>
  <property>
    <name>mapreduce.output.fileoutputformat.compress</name>
    <value>true</value>
  </property>
  <property>
    <name>mapreduce.output.fileoutputformat.compress.codec</name>
    <value>org.apache.hadoop.io.compress.SnappyCodec</value>
  </property>
  <property>
    <name>mapreduce.output.fileoutputformat.compress.type</name>
    <value>BLOCK</value>
  </property>
</configuration>
```

### 添加mysql驱动包

hive使用mysql作为元数据存储，必然需要连接mysql数据库，所以添加一个mysql的连接驱动包到hive的安装目录：/usr/local/hive/lib/下，然后就可以准备启动hive了

### 配置hive的环境变量

```sh
export HIVE_HOME=/usr/local/hive
export PATH=$PATH:$HIVE_HOME/bin
```

## 交互方式

### bin/hive

```hive
[root@server01 /]# hive
hive> show databases;
OK
default
Time taken: 1.373 seconds, Fetched: 1 row(s)
hive>
```

### 使用sql语句或者sql脚本进行交互

```hive
[root@server01 /]# hive -e "show databases;"
which: no hbase in (/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/usr/local/java/jdk1.8.0_144/bin:/usr/local/hadoop-2.7.5/bin:/usr/local/hadoop-2.7.5/sbin:/usr/local/hive/bin:/root/bin)
SLF4J: Class path contains multiple SLF4J bindings.
SLF4J: Found binding in [jar:file:/usr/local/hive/lib/log4j-slf4j-impl-2.4.1.jar!/org/slf4j/impl/StaticLoggerBinder.class]
SLF4J: Found binding in [jar:file:/usr/local/hadoop-2.7.5/share/hadoop/common/lib/slf4j-log4j12-1.7.10.jar!/org/slf4j/impl/StaticLoggerBinder.class]
SLF4J: See http://www.slf4j.org/codes.html#multiple_bindings for an explanation.
SLF4J: Actual binding is of type [org.apache.logging.slf4j.Log4jLoggerFactory]

Logging initialized using configuration in jar:file:/usr/local/hive/lib/hive-common-2.1.1.jar!/hive-log4j2.properties Async: true
OK
default
Time taken: 1.757 seconds, Fetched: 1 row(s)
```

```hive
[root@server01 /]# cd /opt
[root@server01 opt]# vim hive.sql
create database if not exists mytest;
use mytest;
create table stu(id int,name string);
[root@server01 opt]# hive -f /opt/hive.sql
which: no hbase in (/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/usr/local/java/jdk1.8.0_144/bin:/usr/local/hadoop-2.7.5/bin:/usr/local/hadoop-2.7.5/sbin:/usr/local/hive/bin:/root/bin)
SLF4J: Class path contains multiple SLF4J bindings.
SLF4J: Found binding in [jar:file:/usr/local/hive/lib/log4j-slf4j-impl-2.4.1.jar!/org/slf4j/impl/StaticLoggerBinder.class]
SLF4J: Found binding in [jar:file:/usr/local/hadoop-2.7.5/share/hadoop/common/lib/slf4j-log4j12-1.7.10.jar!/org/slf4j/impl/StaticLoggerBinder.class]
SLF4J: See http://www.slf4j.org/codes.html#multiple_bindings for an explanation.
SLF4J: Actual binding is of type [org.apache.logging.slf4j.Log4jLoggerFactory]

Logging initialized using configuration in jar:file:/usr/local/hive/lib/hive-common-2.1.1.jar!/hive-log4j2.properties Async: true
OK
Time taken: 2.319 seconds
OK
Time taken: 0.037 seconds
OK
Time taken: 0.627 seconds
```

## 基本操作

### 数据库操作

#### 创建数据库

```hive
hive> create database if not exists myhive;
OK
Time taken: 1.59 seconds
```

说明：hive的表存放位置模式是由hive-site.xml当中的一个属性指定的

```xml
<name>hive.metastore.warehouse.dir</name>
<value>/user/hive/warehouse</value>
```

#### 创建数据库并指定位置

```hive
hive> create database myhive2 location '/myhive2';
```

#### 设置数据库键值对信息

```hive
hive> create database foo with dbproperties ('owner'='gentleduo','date'='20220830');
hive> describe database extended foo;
hive> alter database foo set dbproperties ('owner'='itheima');
```

#### 查看数据库更多详细信息

```hive
hive> desc database extended myhive2;
```

#### 删除数据库

删除一个空数据库，如果数据库下面有数据表，那么就会报错

```hive
hive> drop database foo;
```

强制删除数据库，包含数据库下面的表一起删除

```hive
hive> drop database myhive2;
```

### 数据库表操作

#### 语法

```sql
create [external] table [if not exists] table_name (
col_name data_type [comment '字段描述信息']
col_name data_type [comment '字段描述信息'])
[comment '表的描述信息']
[partitioned by (col_name data_type,...)]
[clustered by (col_name,col_name,...)]
[sorted by (col_name [asc|desc],...) into num_buckets buckets]
[row format row_format]
[storted as ....]
[location '指定表的路径']
```

#### 说明

##### create table

创建一个指定名字的表。如果相同名字的表已经存在，则抛出异常；用户可以用 IF NOT EXISTS 选项来忽略这个异常。

##### external

可以让用户创建一个外部表，在建表的同时指定一个指向实际数据的路径 （LOCATION），Hive 创建内部表时，会将数据移动到数据仓库指向的路径；若创建外部 表，仅记录数据所在的路径，不对数据的位置做任何改变。在删除表的时候，内部表的 元数据和数据会被一起删除，而外部表只删除元数据，不删除数据。

##### comment

表示注释,默认不能使用中文

##### partitioned by

表示使用表分区,一个表可以拥有一个或者多个分区，每一个分区单独存在一个目录下

##### clustered by

对于每一个表分文件， Hive可以进一步组织成桶，也就是说桶是更为细粒度的数据范围划分。Hive也是 针对某一列进行桶的组织

##### sorted by

指定排序字段和排序规则

##### row format

指定表文件字段分隔符，hive默认的分隔符为：'\001'；

注意：在java中定义字符时，

1. 如果用整型来表示的话表示的是该字符的ascii码值，

2. 如果用字符的话，如果有数字的话只能用0到9的数字，如果有超过9的数值的话必须加转义字符(反斜杠)，表示的是八进制的ascii码，否则会报错，

   ```java
   package com.duo.test;
   
   import com.duo.client.AnalyticsEngineSDK;
   
   public class Test {
       public static void main(String[] args) {
           // 在ascii码中，110代表的是字符n
           char a = 110;
           System.out.println(a);
           // 由于将字符6赋值给b，所以此时的b代表的就是字符6，而不是ascii码6代表的字符
           char b = '6';
           System.out.println(b);
           // 将数值大于9的数赋值给字符会报错
           //char c = '64';
           // 将数值大于9的数赋值转义后赋值给字符，相当于将八进制的ascii码赋值给字符，此时的64表示的是八进制为64的ascii码代表的字符
           // 由于此时转义符后面只能跟八进制，所以 c = '\18', c = '\91'都是非法的(八进制中不存在8，和9的数字)
           char c = '\64';
           System.out.println(c);
           // 八进制的64等于十进制的52，所以c和d代表的字符是相等的
           char d = 52;
           System.out.println(d);
           // 在字符串中，以转义符为开头后面加数字代表的也是八进制的ascii码代表的字符
           // 比如"\60"，等于十进制的48，在ascii码中代表的是0；
           System.out.println("\60");
       }
   }
   ```

##### storted as

指定表文件的存储格式, 常用格式:SEQUENCEFILE, TEXTFILE, RCFILE,如果文件 数据是纯文本，可以使用 STORED AS TEXTFILE。如果数据需要压缩，使用 storted as SEQUENCEFILE

##### location

指定表文件的存储路径

####  内部表的操作

创建表时,如果没有使用external关键字,则该表是内部表（managed table）

Hive建表字段类型

| 分类     | 类型      | 描述                                           | 字面量示例                                                   |
| -------- | --------- | ---------------------------------------------- | ------------------------------------------------------------ |
| 原始类型 | BOOLEAN   | true/false                                     | TRUE                                                         |
|          | TINYINT   | 1字节的有符号整数,-128~127                     | 1Y                                                           |
|          | SMALLINT  | 2个字节的有符号整数，-32768~32767              | 1S                                                           |
|          | INT       | 4个字节的带符号整数                            | 1                                                            |
|          | BIGINT    | 8字节带符号整数                                | 1L                                                           |
|          | FLOAT     | 4字节单精度浮点数                              | 1.0                                                          |
|          | DOUBLE    | 8字节双精度浮点数                              | 1.0                                                          |
|          | DEICIMAL  | 任意精度的带符号小数                           | 1.0                                                          |
|          | STRING    | 字符串，变长                                   | “a”,’b’                                                      |
|          | VARCHAR   | 变长字符串                                     | “a”,’b’                                                      |
|          | CHAR      | 固定长度字符串                                 | “a”,’b’                                                      |
|          | BINARY    | 字节数组                                       |                                                              |
|          | TIMESTAMP | 时间戳，毫秒值精度                             | 122327493795                                                 |
|          | DATE      | 日期                                           | ‘2016-03-29’                                                 |
|          | INTERVAL  | 时间频率间隔                                   |                                                              |
| 复杂类型 | ARRAY     | 有序的的同类型的集合                           | array(1,2)                                                   |
|          | MAP       | key-value,key必须为原始类型，value可以任意类型 | map(‘a’,1,’b’,2)                                             |
|          | STRUCT    | 字段集合,类型可以不同                          | struct(‘1’,1,1.0),named_stract(‘col1’,’1’,’col2’,1,’clo3’,1.0) |
|          | UNION     | 在有限取值范围内的一个值                       | create_union(1,’a’,63)                                       |

##### 建表入门

```sql
use myhive;
create table stu(id int,name string);
insert into stu values (1,"zhangsan"); #插入数据
select * from stu;
```

##### 创建表并指定字段之间的分隔符

```sql
create  table if not exists stu2(id int ,name string) row format delimited fields terminated by '\t';
```

##### 创建表并指定表文件的存放路径

```sql
create  table if not exists stu2(id int ,name string) row format delimited fields terminated by '\t' location '/user/stu2';
```

##### 根据查询结果创建表

```sql
create table stu3 as select * from stu2; # 通过复制表结构和表内容创建新表
```

##### 根据已经存在的表结构创建表

```sql
create table stu4 like stu;
```

##### 查询表的详细信息

```sql
desc formatted stu2;
```

##### 删除表

```sql
drop table stu4;
```

#### 外部表的操作

##### 说明

外部表因为是指定其他的hdfs路径的数据加载到表当中来，所以hive表会认为自己不完全独占 这份数据，所以删除hive表的时候，数据仍然存放在hdfs当中，不会删掉.

##### 内部表和外部表的使用场景

每天将收集到的网站日志定期流入HDFS文本文件（各部门共享）。在外部表（原始日志表）的基础上各部门再根据各自的需要做大量 的统计分析，在这个过程中用到的中间表、结果表则使用内部表存储，数据通过SELECT+INSERT进入各部门的内部表。

##### 案例

分别创建老师与学生表外部表，并向表中加载数据

###### 创建老师表

```hive
hive> use myhive;
OK
Time taken: 1.11 seconds
hive> create external table teacher (t_id string,t_name string) row format delimited fields terminated by '\t';
OK
Time taken: 0.855 seconds

```

###### 创建学生表

```hive
hive> create external table student (s_id string,s_name string,s_birth string , s_sex string ) row format delimited fields terminated by '\t';
OK
Time taken: 0.173 seconds
```

###### 加载数据

1. 方式1

   将hdfs文件映射成表

   ```bash
   [root@server01 opt]# vim teacher.txt
   1       zhangsan
   2       lisi
   3       wangwu
   # 将文件直接传到hdfs的表目录：/user/hive/warehouse/myhive.db/teacher后，hive会自动将hdfs文件映射成一张表
   [root@server01 opt]# hdfs dfs -put teacher.txt /user/hive/warehouse/myhive.db/teacher
   ```

   查询

   ```hive
   hive> select * from teacher;
   OK
   1       zhangsan
   2       lisi
   3       wangwu
   Time taken: 1.462 seconds, Fetched: 3 row(s)
   ```

2. 方式2

   加载数据

   ```hive
   hive> load data local inpath '/opt/bigdata/hive/student.csv' into table student;
   Loading data to table myhive.student
   OK
   Time taken: 1.027 seconds
   hive> select * from student;
   OK
   01      赵雷    1990-01-01      男
   02      钱电    1990-12-21      男
   03      孙风    1990-05-20      男
   04      李云    1990-08-06      男
   05      周梅    1991-12-01      女
   06      吴兰    1992-03-01      女
   07      郑竹    1989-07-01      女
   08      王菊    1990-01-20      女
   Time taken: 0.208 seconds, Fetched: 8 row(s)
   ```

   加载数据并覆盖已有数据

   ```hive
   hive> load data local inpath '/opt/bigdata/hive/student.csv' overwrite into table student;
   Loading data to table myhive.student
   Moved: 'hdfs://server01:8020/user/hive/warehouse/myhive.db/student/student.csv' to trash at: hdfs://server01:8020/user/root/.Trash/Current
   OK
   Time taken: 0.517 seconds
   hive> drop table student;
   OK
   Time taken: 3.192 seconds
   hive> select * from student;
   FAILED: SemanticException [Error 10001]: Line 1:14 Table not found 'student'
   hive> create external table student (s_id string,s_name string,s_birth string , s_sex string ) row format delimited fields terminated by '\t';
   OK
   Time taken: 0.105 seconds
   hive> select * from student;
   OK
   01      赵雷    1990-01-01      男
   02      钱电    1990-12-21      男
   03      孙风    1990-05-20      男
   04      李云    1990-08-06      男
   05      周梅    1991-12-01      女
   06      吴兰    1992-03-01      女
   07      郑竹    1989-07-01      女
   08      王菊    1990-01-20      女
   Time taken: 0.128 seconds, Fetched: 8 row(s)
   ```

   删除外部表只会删元数据，真实的数据不会删除，当重新创建已经删除的外部表后，hive又会自动建立映射关系。说表和表数据只是一个映射关系，当表被删除后数据还是在的。

   从hdfs文件系统向表中加载数据（需要提前将数据上传到hdfs文件系统）

   ```bash
   [root@server01 hive]# cd /opt/bigdata/hive
   [root@server01 hive]# hdfs dfs -mkdir -p /hivedatas
   [root@server01 hive]# hdfs dfs -put techer.csv /hivedatas/
   ```

   ```hive
   hive> load data inpath '/hivedatas/techer.csv' overwrite into table teacher;
   Loading data to table myhive.teacher
   OK
   Time taken: 0.366 seconds
   ```


### 分区表操作

在大数据中，最常用的一种思想就是分治，可以把大的文件切割划分成一个个的小的文件，这样每次操作一个小的文件就会很容易了，同样的道理，在hive当中也是支持这种思想的，就是可以把大的数据，按照每月，或者天进行切分成一个个的小的文件,存放在不同的文件夹中.

#### 创建分区表

```hive
hive> create table score(s_id string,c_id string, s_score int) partitioned by (month string) row format delimited fields terminated by '\t';
OK
Time taken: 0.234 seconds
```

#### 创建多分区表

会在hdfs中创建多级目录

```hive
hive> create table score2 (s_id string,c_id string, s_score int) partitioned by (year string,month string,day string) row format delimited fields terminated by '\t';
OK
Time taken: 0.249 seconds
```

#### 加载数据到分区表中

```hive
hive> load data local inpath '/opt/bigdata/hive/score.csv' into table score partition (month='202207');
Loading data to table myhive.score partition (month=202207)
OK
Time taken: 0.696 seconds
hive> load data local inpath '/opt/bigdata/hive/score.csv' into table score partition (month='202208');
Loading data to table myhive.score partition (month=202208)
OK
Time taken: 0.651 seconds
```

#### 加载数据到多分区表中

```hive
hive> load data local inpath '/opt/bigdata/hive/score.csv' into table score2 partition (year='2022',month='08',day='01');
Loading data to table myhive.score2 partition (year=2022, month=08, day=01)
OK
Time taken: 0.768 seconds
```

#### 查询

```hive
hive> select * from score;
OK
01      01      80      202207
01      02      90      202207
01      03      99      202207
02      01      70      202207
02      02      60      202207
02      03      80      202207
03      01      80      202207
03      02      80      202207
03      03      80      202207
04      01      50      202207
04      02      30      202207
04      03      20      202207
05      01      76      202207
05      02      87      202207
06      01      31      202207
06      03      34      202207
07      02      89      202207
07      03      98      202207
01      01      80      202208
01      02      90      202208
01      03      99      202208
02      01      70      202208
02      02      60      202208
02      03      80      202208
03      01      80      202208
03      02      80      202208
03      03      80      202208
04      01      50      202208
04      02      30      202208
04      03      20      202208
05      01      76      202208
05      02      87      202208
06      01      31      202208
06      03      34      202208
07      02      89      202208
07      03      98      202208
Time taken: 0.154 seconds, Fetched: 36 row(s)
hive> select * from score where month = '202207';
OK
01      01      80      202207
01      02      90      202207
01      03      99      202207
02      01      70      202207
02      02      60      202207
02      03      80      202207
03      01      80      202207
03      02      80      202207
03      03      80      202207
04      01      50      202207
04      02      30      202207
04      03      20      202207
05      01      76      202207
05      02      87      202207
06      01      31      202207
06      03      34      202207
07      02      89      202207
07      03      98      202207
Time taken: 0.153 seconds, Fetched: 18 row(s)
```

#### 多分区联合查询

```hive
hive> select * from score where month = '202207' union all select * from score where month = '202208';
WARNING: Hive-on-MR is deprecated in Hive 2 and may not be available in the future versions. Consider using a different execution engine (i.e. spark, tez) or using Hive 1.X releases.
Query ID = root_20220831131344_f817f033-5e24-4b8b-9597-89d063125af6
Total jobs = 1
Launching Job 1 out of 1
Number of reduce tasks is set to 0 since there's no reduce operator
Job running in-process (local Hadoop)
2022-08-31 13:13:47,450 Stage-1 map = 100%,  reduce = 0%
Ended Job = job_local1450407751_0001
MapReduce Jobs Launched:
Stage-Stage-1:  HDFS Read: 3370 HDFS Write: 1772 SUCCESS
Total MapReduce CPU Time Spent: 0 msec
OK
01      01      80      202207
01      02      90      202207
01      03      99      202207
02      01      70      202207
02      02      60      202207
02      03      80      202207
03      01      80      202207
03      02      80      202207
03      03      80      202207
04      01      50      202207
04      02      30      202207
04      03      20      202207
05      01      76      202207
05      02      87      202207
06      01      31      202207
06      03      34      202207
07      02      89      202207
07      03      98      202207
01      01      80      202208
01      02      90      202208
01      03      99      202208
02      01      70      202208
02      02      60      202208
02      03      80      202208
03      01      80      202208
03      02      80      202208
03      03      80      202208
04      01      50      202208
04      02      30      202208
04      03      20      202208
05      01      76      202208
05      02      87      202208
06      01      31      202208
06      03      34      202208
07      02      89      202208
07      03      98      202208
Time taken: 2.699 seconds, Fetched: 36 row(s)
```

#### 查看分区

```hive
hive> show partitions score;
OK
month=202207
month=202208
Time taken: 0.103 seconds, Fetched: 2 row(s)
```

#### 添加分区

```hive
hive> alter table score add partition(month='202209');
OK
Time taken: 0.164 seconds
```

#### 删除分区

```hive
hive> alter table score drop partition(month='202209');
Moved: 'hdfs://server01:8020/user/hive/warehouse/myhive.db/score/month=202209' to trash at: hdfs://server01:8020/user/root/.Trash/Current
Dropped the partition month=202209
OK
Time taken: 0.326 seconds
```

#### 综合练习

在有一个文件score.csv文件，存放在集群的这个目录下/scoredatas/month=20220801，这个文件每天都会生成，存放到对应的日期文件夹下面去，文件别人也需要公用，不能移动。需求，创建hive对应的表，并将数据加载到表中，进行数据统计分析，且删除表之后，数据不能删除

##### 数据准备

```bash
[root@server01 hive]# hdfs dfs -mkdir -p /scoredatas/month=20220801
[root@server01 hive]# hdfs dfs -put /opt/bigdata/hive/score.csv /scoredatas/month=20220801/
```

##### 创建外部分区表，并指定文件数据存放目录

```hive
hive> create external table score3(s_id string, c_id string,s_score int) partitioned by (month string) row format delimited fields terminated by '\t' location '/scoredatas';
OK
Time taken: 0.135 seconds
hive> select * from score3;
OK
Time taken: 0.104 seconds
```

##### 进行表的修复

建立表与数据文件之间的一个关系映射。

```hive
hive> msck repair table score3;
OK
Partitions not in metastore:    score3:month=20220801
Repair: Added partition to metastore score3:month=20220801
Time taken: 0.21 seconds, Fetched: 2 row
```

注意：如果创建的表是不带分区的则不需要进行表修复。例如：

```bash
[root@server01 hive]# hdfs dfs -mkdir -p /scorelog
[root@server01 hive]# hdfs dfs -put /opt/bigdata/hive/score.csv /scorelog/
```

```hive
hive> create external table score4(s_id string, c_id string,s_score int) row format delimited fields terminated by '\t' location '/scorelog';
OK
Time taken: 0.069 seconds
hive> select * from score4;
OK
01      01      80
01      02      90
01      03      99
02      01      70
02      02      60
02      03      80
03      01      80
03      02      80
03      03      80
04      01      50
04      02      30
04      03      20
05      01      76
05      02      87
06      01      31
06      03      34
07      02      89
07      03      98
Time taken: 0.084 seconds, Fetched: 18 row(s)
```

### 分桶表操作

分桶，就是将数据按照指定的字段进行划分到多个文件当中去,分桶就是MapReduce中的分区.

#### 开启Hive的分桶功能

```hive
hive> set hive.enforce.bucketing=true;
```

#### 设置Reduce个数

```hive
hive> set mapreduce.job.reduces=3;
```

#### 查看设置reduce个数

```hive
hive> set mapreduce.job.reduces;
mapreduce.job.reduces=3
```

#### 创建分桶表

```hive
hive> create table course (c_id string,c_name string,t_id string) clustered by(c_id) into 3 buckets row format delimited fields terminated by '\t';
OK
Time taken: 0.137 seconds
```

由于分桶表的数据加载需要通过mapreduce，所以通过hdfs dfs -put文件或者通过load data都不行，只能通过insert overwrite，创建普通表，并通过insert overwriter的方式将普通表的数据通过查询的方式加载到分桶表当中去。

#### 创建普通表

```hive
hive> create table course_common (c_id string,c_name string,t_id string) row format delimited fields terminated by '\t';
OK
Time taken: 0.134 seconds
```

#### 普通表中加载数据

```hive
hive> load data local inpath '/opt/bigdata/hive/course.csv' into table course_common;
Loading data to table myhive.course_common
OK
Time taken: 0.314 seconds
```

#### 通过insert overwrite给桶表中加载数据

```hive
hive> insert overwrite table course select * from course_common cluster by(c_id);
WARNING: Hive-on-MR is deprecated in Hive 2 and may not be available in the future versions. Consider using a different execution engine (i.e. spark, tez) or using Hive 1.X releases.
Query ID = root_20220831142835_0e879fe0-e237-47d9-b636-4ef502ab4445
Total jobs = 2
Launching Job 1 out of 2
Number of reduce tasks not specified. Defaulting to jobconf value of: 3
In order to change the average load for a reducer (in bytes):
  set hive.exec.reducers.bytes.per.reducer=<number>
In order to limit the maximum number of reducers:
  set hive.exec.reducers.max=<number>
In order to set a constant number of reducers:
  set mapreduce.job.reduces=<number>
Job running in-process (local Hadoop)
2022-08-31 14:28:37,391 Stage-1 map = 100%,  reduce = 100%
Ended Job = job_local1299401726_0002
Launching Job 2 out of 2
Number of reduce tasks determined at compile time: 3
In order to change the average load for a reducer (in bytes):
  set hive.exec.reducers.bytes.per.reducer=<number>
In order to limit the maximum number of reducers:
  set hive.exec.reducers.max=<number>
In order to set a constant number of reducers:
  set mapreduce.job.reduces=<number>
Job running in-process (local Hadoop)
2022-08-31 14:28:39,918 Stage-2 map = 100%,  reduce = 67%
2022-08-31 14:28:40,969 Stage-2 map = 100%,  reduce = 100%
Ended Job = job_local123090463_0003
Loading data to table myhive.course
MapReduce Jobs Launched:
Stage-Stage-1:  HDFS Read: 8516 HDFS Write: 3700 SUCCESS
Stage-Stage-2:  HDFS Read: 8516 HDFS Write: 4192 SUCCESS
Total MapReduce CPU Time Spent: 0 msec
OK
Time taken: 5.688 seconds
```

## 修改表结构

### 重命名

```hive
hive> alter table score4 rename to score5;
OK
Time taken: 0.162 seconds
```

### 查询表结构

```hive
hive> desc score5;
OK
s_id                    string
c_id                    string
s_score                 int
Time taken: 0.05 seconds, Fetched: 3 row(s)
```

### 增加/修改列信息

#### 添加列

```hive
hive> alter table score5 add columns (mycol string, mysco int);
OK
Time taken: 0.127 seconds
hive> desc score5;
OK
s_id                    string
c_id                    string
s_score                 int
mycol                   string
mysco                   int
Time taken: 0.048 seconds, Fetched: 5 row(s)
```

#### 更新列

```hive
hive> alter table score5 change column mysco mysconew int;
OK
Time taken: 0.176 seconds
hive> desc score5;
OK
s_id                    string
c_id                    string
s_score                 int
mycol                   string
mysconew                int
Time taken: 0.024 seconds, Fetched: 5 row(s)
```

### 删除表

```hive
hive> drop table score5;
OK
Time taken: 0.172 seconds
```

## 查询语法

### SELECT

```hive
SELECT [ALL | DISTINCT] select_expr, select_expr, ...
FROM table_reference
[WHERE where_condition]
[GROUP BY col_list [HAVING condition]]
[CLUSTER BY col_list
| [DISTRIBUTE BY col_list] [SORT BY| ORDER BY col_list]
]
[LIMIT number]
```

1. order by 会对输入做全局排序，因此只有一个reducer，会导致当输入规模较大时，需要较长的计算时间。
2. sort by不是全局排序，其在数据进入reducer前完成排序。因此，如果用sort by进行排序，并且设置mapred.reduce.tasks>1，则sort by只保证每个reducer的输出有序，不保证全局有序。
3. distribute by(字段)根据指定的字段将数据分到不同的reducer（即：分区），且分发算法是hash散列。
4. cluster by(字段) 除了具有distribute by的功能外，还会对该字段进行排序。

因此，如果distribute 和sort字段是同一个时，此时， cluster by = distribute by + sort by

#### 全表查询

```hive
hive> select * from score;
```

#### 选择特定列

```hive
hive> select s_id ,c_id from score;
```

#### 列别名

```hive
hive> select s_id as myid ,c_id from score;
```

### 常用函数

#### 求总行数（count）

```hive
hive> select count(1) from score;
```

#### 求分数的最大值（max）

```hive
hive> select max(s_score) from score;
```

#### 求分数的最小值（min）

```hive
hive> select min(s_score) from score;
```

#### 求分数的总和（sum）

```hive
hive> select sum(s_score) from score;
```

#### 求分数的平均值（avg）

```hive
hive> select avg(s_score) from score;
```

### LIMIT

典型的查询会返回多行数据。LIMIT子句用于限制返回的行数。

```hive
hive> select * from score limit 3;
OK
01      01      80      202207
01      02      90      202207
01      03      99      202207
Time taken: 0.152 seconds, Fetched: 3 row(s)
```

### LIKE和RLIKE

使用LIKE运算选择类似的值，选择条件可以包含字符或数字:

1. % 代表零个或多个字符(任意个字符)。
2. _ 代表一个字符。

RLIKE子句是Hive中这个功能的一个扩展，其可以通过Java的正则表达式这个更强大的语言来指定匹配条件。

查找以8开头的所有成绩

```hive
hive> select * from score where s_score like '8%';
OK
01      01      80      202207
02      03      80      202207
03      01      80      202207
03      02      80      202207
03      03      80      202207
05      02      87      202207
07      02      89      202207
01      01      80      202208
02      03      80      202208
03      01      80      202208
03      02      80      202208
03      03      80      202208
05      02      87      202208
07      02      89      202208
Time taken: 0.175 seconds, Fetched: 14 row(s)
```

查找第二个数值为9的所有成绩数据

```hive
hive> select * from score where s_score like '_9%';
OK
01      03      99      202207
07      02      89      202207
01      03      99      202208
07      02      89      202208
Time taken: 0.157 seconds, Fetched: 4 row(s)
```

查找s_id中含1的数据

```hive
hive> select * from score where s_id rlike '[1]';
OK
01      01      80      202207
01      02      90      202207
01      03      99      202207
01      01      80      202208
01      02      90      202208
01      03      99      202208
Time taken: 0.127 seconds, Fetched: 6 row(s)
```

### GROUP BY

GROUP BY语句通常会和聚合函数一起使用，按照一个或者多个列队结果进行分组，然后对每 个组执行聚合操作。

计算每个学生的平均分数

```hive
hive> select s_id ,avg(s_score) from score group by s_id;
WARNING: Hive-on-MR is deprecated in Hive 2 and may not be available in the future versions. Consider using a different execution engine (i.e. spark, tez) or using Hive 1.X releases.
Query ID = root_20220831161108_570580c6-2548-4a5d-ae23-30ec93c62b66
Total jobs = 1
Launching Job 1 out of 1
Number of reduce tasks not specified. Defaulting to jobconf value of: 3
In order to change the average load for a reducer (in bytes):
  set hive.exec.reducers.bytes.per.reducer=<number>
In order to limit the maximum number of reducers:
  set hive.exec.reducers.max=<number>
In order to set a constant number of reducers:
  set mapreduce.job.reduces=<number>
Job running in-process (local Hadoop)
2022-08-31 16:11:09,949 Stage-1 map = 100%,  reduce = 100%
Ended Job = job_local1835751581_0009
MapReduce Jobs Launched:
Stage-Stage-1:  HDFS Read: 29588 HDFS Write: 4684 SUCCESS
Total MapReduce CPU Time Spent: 0 msec
OK
03      80.0
06      32.5
01      89.66666666666667
04      33.333333333333336
07      93.5
02      70.0
05      81.5
Time taken: 1.472 seconds, Fetched: 7 row(s)
```

### HAVING

having与where不同点：

1. where针对表中的列发挥作用，查询数据；having针对查询结果中的列发挥作用，筛选数据。
2. where后面不能写分组函数，而having后面可以使用分组函数。
3. having只用于group by分组统计语句。

求每个学生平均分数大于85的人

```hive
hive> select s_id ,avg(s_score) avgscore from score group by s_id having avgscore > 85;
WARNING: Hive-on-MR is deprecated in Hive 2 and may not be available in the future versions. Consider using a different execution engine (i.e. spark, tez) or using Hive 1.X releases.
Query ID = root_20220831161302_eac518da-324d-4c68-8ff9-f3ff0335611f
Total jobs = 1
Launching Job 1 out of 1
Number of reduce tasks not specified. Defaulting to jobconf value of: 3
In order to change the average load for a reducer (in bytes):
  set hive.exec.reducers.bytes.per.reducer=<number>
In order to limit the maximum number of reducers:
  set hive.exec.reducers.max=<number>
In order to set a constant number of reducers:
  set mapreduce.job.reduces=<number>
Job running in-process (local Hadoop)
2022-08-31 16:13:04,429 Stage-1 map = 100%,  reduce = 100%
Ended Job = job_local364563648_0010
MapReduce Jobs Launched:
Stage-Stage-1:  HDFS Read: 30884 HDFS Write: 4684 SUCCESS
Total MapReduce CPU Time Spent: 0 msec
OK
01      89.66666666666667
07      93.5
Time taken: 1.565 seconds, Fetched: 2 row(s)
```

### JOIN

Hive支持通常的SQL JOIN语句，但是只支持等值连接，不支持非等值连接。

#### 内连接

内连接：只有进行连接的两个表中都存在与连接条件相匹配的数据才会被保留下来。

```hive
hive> select * from teacher t inner join course c on t.t_id = c.t_id;
WARNING: Hive-on-MR is deprecated in Hive 2 and may not be available in the future versions. Consider using a different execution engine (i.e. spark, tez) or using Hive 1.X releases.
Query ID = root_20220831161427_5150cf98-ced4-48b4-95ae-682aafebae53
Total jobs = 1
SLF4J: Class path contains multiple SLF4J bindings.
SLF4J: Found binding in [jar:file:/usr/local/hive/lib/log4j-slf4j-impl-2.4.1.jar!/org/slf4j/impl/StaticLoggerBinder.class]
SLF4J: Found binding in [jar:file:/usr/local/hadoop-2.7.5/share/hadoop/common/lib/slf4j-log4j12-1.7.10.jar!/org/slf4j/impl/StaticLoggerBinder.class]
SLF4J: See http://www.slf4j.org/codes.html#multiple_bindings for an explanation.
SLF4J: Actual binding is of type [org.apache.logging.slf4j.Log4jLoggerFactory]
2022-08-31 16:14:38     Starting to launch local task to process map join;      maximum memory = 518979584
2022-08-31 16:14:39     Dump the side-table for tag: 0 with group count: 3 into file: file:/tmp/root/bbef36fd-2385-4292-978a-9fdf19b2da63/hive_2022-08-31_16-14-27_645_7608515339673598536-1/-local-10004/HashTable-Stage-3/MapJoin-mapfile10--.hashtable
2022-08-31 16:14:39     Uploaded 1 File to: file:/tmp/root/bbef36fd-2385-4292-978a-9fdf19b2da63/hive_2022-08-31_16-14-27_645_7608515339673598536-1/-local-10004/HashTable-Stage-3/MapJoin-mapfile10--.hashtable (344 bytes)
2022-08-31 16:14:39     End of local task; Time Taken: 1.711 sec.
Execution completed successfully
MapredLocal task succeeded
Launching Job 1 out of 1
Number of reduce tasks is set to 0 since there's no reduce operator
Job running in-process (local Hadoop)
2022-08-31 16:14:43,245 Stage-3 map = 100%,  reduce = 0%
Ended Job = job_local1489247584_0012
MapReduce Jobs Launched:
Stage-Stage-3:  HDFS Read: 16155 HDFS Write: 2342 SUCCESS
Total MapReduce CPU Time Spent: 0 msec
OK
03      王五    03      英语    03
01      张三    02      数学    01
02      李四    01      语文    02
Time taken: 15.605 seconds, Fetched: 3 row(s)
```

#### 左外连接

左外连接：JOIN操作符左边表中符合WHERE子句的所有记录将会被返回。 查询老师对应的课 程

```hive
hive> select * from teacher t left join course c on t.t_id = c.t_id;
WARNING: Hive-on-MR is deprecated in Hive 2 and may not be available in the future versions. Consider using a different execution engine (i.e. spark, tez) or using Hive 1.X releases.
Query ID = root_20220831161709_a6cadbf6-0d79-4bed-a89d-b64d664c3c0f
Total jobs = 1
SLF4J: Class path contains multiple SLF4J bindings.
SLF4J: Found binding in [jar:file:/usr/local/hive/lib/log4j-slf4j-impl-2.4.1.jar!/org/slf4j/impl/StaticLoggerBinder.class]
SLF4J: Found binding in [jar:file:/usr/local/hadoop-2.7.5/share/hadoop/common/lib/slf4j-log4j12-1.7.10.jar!/org/slf4j/impl/StaticLoggerBinder.class]
SLF4J: See http://www.slf4j.org/codes.html#multiple_bindings for an explanation.
SLF4J: Actual binding is of type [org.apache.logging.slf4j.Log4jLoggerFactory]
2022-08-31 16:17:19     Starting to launch local task to process map join;      maximum memory = 518979584
2022-08-31 16:17:21     Dump the side-table for tag: 1 with group count: 3 into file: file:/tmp/root/bbef36fd-2385-4292-978a-9fdf19b2da63/hive_2022-08-31_16-17-09_865_1618300336243957525-1/-local-10004/HashTable-Stage-3/MapJoin-mapfile21--.hashtable
2022-08-31 16:17:21     Uploaded 1 File to: file:/tmp/root/bbef36fd-2385-4292-978a-9fdf19b2da63/hive_2022-08-31_16-17-09_865_1618300336243957525-1/-local-10004/HashTable-Stage-3/MapJoin-mapfile21--.hashtable (353 bytes)
2022-08-31 16:17:21     End of local task; Time Taken: 1.729 sec.
Execution completed successfully
MapredLocal task succeeded
Launching Job 1 out of 1
Number of reduce tasks is set to 0 since there's no reduce operator
Job running in-process (local Hadoop)
2022-08-31 16:17:24,850 Stage-3 map = 100%,  reduce = 0%
Ended Job = job_local954173857_0014
MapReduce Jobs Launched:
Stage-Stage-3:  HDFS Read: 8183 HDFS Write: 1171 SUCCESS
Total MapReduce CPU Time Spent: 0 msec
OK
01      张三    02      数学    01
02      李四    01      语文    02
03      王五    03      英语    03
Time taken: 14.991 seconds, Fetched: 3 row(s)
```

#### 右外连接

右外连接：JOIN操作符右边表中符合WHERE子句的所有记录将会被返回。

```hive
hive> select * from teacher t right join course c on t.t_id = c.t_id;
WARNING: Hive-on-MR is deprecated in Hive 2 and may not be available in the future versions. Consider using a different execution engine (i.e. spark, tez) or using Hive 1.X releases.
Query ID = root_20220831161934_9f194288-0fea-43b9-9f2b-b4a0952cf632
Total jobs = 1
SLF4J: Class path contains multiple SLF4J bindings.
SLF4J: Found binding in [jar:file:/usr/local/hive/lib/log4j-slf4j-impl-2.4.1.jar!/org/slf4j/impl/StaticLoggerBinder.class]
SLF4J: Found binding in [jar:file:/usr/local/hadoop-2.7.5/share/hadoop/common/lib/slf4j-log4j12-1.7.10.jar!/org/slf4j/impl/StaticLoggerBinder.class]
SLF4J: See http://www.slf4j.org/codes.html#multiple_bindings for an explanation.
SLF4J: Actual binding is of type [org.apache.logging.slf4j.Log4jLoggerFactory]
2022-08-31 16:19:44     Starting to launch local task to process map join;      maximum memory = 518979584
2022-08-31 16:19:46     Dump the side-table for tag: 0 with group count: 3 into file: file:/tmp/root/bbef36fd-2385-4292-978a-9fdf19b2da63/hive_2022-08-31_16-19-34_144_6731761041383703760-1/-local-10004/HashTable-Stage-3/MapJoin-mapfile30--.hashtable
2022-08-31 16:19:46     Uploaded 1 File to: file:/tmp/root/bbef36fd-2385-4292-978a-9fdf19b2da63/hive_2022-08-31_16-19-34_144_6731761041383703760-1/-local-10004/HashTable-Stage-3/MapJoin-mapfile30--.hashtable (344 bytes)
2022-08-31 16:19:46     End of local task; Time Taken: 1.803 sec.
Execution completed successfully
MapredLocal task succeeded
Launching Job 1 out of 1
Number of reduce tasks is set to 0 since there's no reduce operator
Job running in-process (local Hadoop)
2022-08-31 16:19:49,663 Stage-3 map = 100%,  reduce = 0%
Ended Job = job_local212957694_0015
MapReduce Jobs Launched:
Stage-Stage-3:  HDFS Read: 16647 HDFS Write: 2342 SUCCESS
Total MapReduce CPU Time Spent: 0 msec
OK
03      王五    03      英语    03
01      张三    02      数学    01
02      李四    01      语文    02
Time taken: 15.522 seconds, Fetched: 3 row(s)
```

#### 多表连接

连接 n个表，至少需要n-1个连接条件。例如：连接三个表，至少需要两个连接条件。 多表连接查询，查询老师对应的课程，以及对应的分数，对应的学生

```hive
hive> select * from teacher t left join course c on t.t_id = c.t_id left join score s on s.c_id = c.c_id left join student stu on s.s_id = stu.s_id;
WARNING: Hive-on-MR is deprecated in Hive 2 and may not be available in the future versions. Consider using a different execution engine (i.e. spark, tez) or using Hive 1.X releases.
Query ID = root_20220831162044_665f46af-81c3-45ad-b30d-ad2f46ae29ae
Total jobs = 1
SLF4J: Class path contains multiple SLF4J bindings.
SLF4J: Found binding in [jar:file:/usr/local/hive/lib/log4j-slf4j-impl-2.4.1.jar!/org/slf4j/impl/StaticLoggerBinder.class]
SLF4J: Found binding in [jar:file:/usr/local/hadoop-2.7.5/share/hadoop/common/lib/slf4j-log4j12-1.7.10.jar!/org/slf4j/impl/StaticLoggerBinder.class]
SLF4J: See http://www.slf4j.org/codes.html#multiple_bindings for an explanation.
SLF4J: Actual binding is of type [org.apache.logging.slf4j.Log4jLoggerFactory]
2022-08-31 16:20:54     Starting to launch local task to process map join;      maximum memory = 518979584
2022-08-31 16:20:56     Dump the side-table for tag: 1 with group count: 8 into file: file:/tmp/root/bbef36fd-2385-4292-978a-9fdf19b2da63/hive_2022-08-31_16-20-44_052_8952827809831279621-1/-local-10006/HashTable-Stage-7/MapJoin-mapfile41--.hashtable
2022-08-31 16:20:56     Uploaded 1 File to: file:/tmp/root/bbef36fd-2385-4292-978a-9fdf19b2da63/hive_2022-08-31_16-20-44_052_8952827809831279621-1/-local-10006/HashTable-Stage-7/MapJoin-mapfile41--.hashtable (607 bytes)
2022-08-31 16:20:56     Dump the side-table for tag: 1 with group count: 3 into file: file:/tmp/root/bbef36fd-2385-4292-978a-9fdf19b2da63/hive_2022-08-31_16-20-44_052_8952827809831279621-1/-local-10006/HashTable-Stage-7/MapJoin-mapfile51--.hashtable
2022-08-31 16:20:56     Uploaded 1 File to: file:/tmp/root/bbef36fd-2385-4292-978a-9fdf19b2da63/hive_2022-08-31_16-20-44_052_8952827809831279621-1/-local-10006/HashTable-Stage-7/MapJoin-mapfile51--.hashtable (887 bytes)
2022-08-31 16:20:56     Dump the side-table for tag: 1 with group count: 3 into file: file:/tmp/root/bbef36fd-2385-4292-978a-9fdf19b2da63/hive_2022-08-31_16-20-44_052_8952827809831279621-1/-local-10006/HashTable-Stage-7/MapJoin-mapfile61--.hashtable
2022-08-31 16:20:56     Uploaded 1 File to: file:/tmp/root/bbef36fd-2385-4292-978a-9fdf19b2da63/hive_2022-08-31_16-20-44_052_8952827809831279621-1/-local-10006/HashTable-Stage-7/MapJoin-mapfile61--.hashtable (353 bytes)
2022-08-31 16:20:56     End of local task; Time Taken: 2.39 sec.
Execution completed successfully
MapredLocal task succeeded
Launching Job 1 out of 1
Number of reduce tasks is set to 0 since there's no reduce operator
Job running in-process (local Hadoop)
2022-08-31 16:20:59,769 Stage-7 map = 100%,  reduce = 0%
Ended Job = job_local1814042043_0016
MapReduce Jobs Launched:
Stage-Stage-7:  HDFS Read: 8360 HDFS Write: 1171 SUCCESS
Total MapReduce CPU Time Spent: 0 msec
OK
01      张三    02      数学    01      01      02      90      202207  01      赵雷    1990-01-01      男
01      张三    02      数学    01      02      02      60      202207  02      钱电    1990-12-21      男
01      张三    02      数学    01      03      02      80      202207  03      孙风    1990-05-20      男
01      张三    02      数学    01      04      02      30      202207  04      李云    1990-08-06      男
01      张三    02      数学    01      05      02      87      202207  05      周梅    1991-12-01      女
01      张三    02      数学    01      07      02      89      202207  07      郑竹    1989-07-01      女
01      张三    02      数学    01      01      02      90      202208  01      赵雷    1990-01-01      男
01      张三    02      数学    01      02      02      60      202208  02      钱电    1990-12-21      男
01      张三    02      数学    01      03      02      80      202208  03      孙风    1990-05-20      男
01      张三    02      数学    01      04      02      30      202208  04      李云    1990-08-06      男
01      张三    02      数学    01      05      02      87      202208  05      周梅    1991-12-01      女
01      张三    02      数学    01      07      02      89      202208  07      郑竹    1989-07-01      女
02      李四    01      语文    02      01      01      80      202207  01      赵雷    1990-01-01      男
02      李四    01      语文    02      02      01      70      202207  02      钱电    1990-12-21      男
02      李四    01      语文    02      03      01      80      202207  03      孙风    1990-05-20      男
02      李四    01      语文    02      04      01      50      202207  04      李云    1990-08-06      男
02      李四    01      语文    02      05      01      76      202207  05      周梅    1991-12-01      女
02      李四    01      语文    02      06      01      31      202207  06      吴兰    1992-03-01      女
02      李四    01      语文    02      01      01      80      202208  01      赵雷    1990-01-01      男
02      李四    01      语文    02      02      01      70      202208  02      钱电    1990-12-21      男
02      李四    01      语文    02      03      01      80      202208  03      孙风    1990-05-20      男
02      李四    01      语文    02      04      01      50      202208  04      李云    1990-08-06      男
02      李四    01      语文    02      05      01      76      202208  05      周梅    1991-12-01      女
02      李四    01      语文    02      06      01      31      202208  06      吴兰    1992-03-01      女
03      王五    03      英语    03      01      03      99      202207  01      赵雷    1990-01-01      男
03      王五    03      英语    03      02      03      80      202207  02      钱电    1990-12-21      男
03      王五    03      英语    03      03      03      80      202207  03      孙风    1990-05-20      男
03      王五    03      英语    03      04      03      20      202207  04      李云    1990-08-06      男
03      王五    03      英语    03      06      03      34      202207  06      吴兰    1992-03-01      女
03      王五    03      英语    03      07      03      98      202207  07      郑竹    1989-07-01      女
03      王五    03      英语    03      01      03      99      202208  01      赵雷    1990-01-01      男
03      王五    03      英语    03      02      03      80      202208  02      钱电    1990-12-21      男
03      王五    03      英语    03      03      03      80      202208  03      孙风    1990-05-20      男
03      王五    03      英语    03      04      03      20      202208  04      李云    1990-08-06      男
03      王五    03      英语    03      06      03      34      202208  06      吴兰    1992-03-01      女
03      王五    03      英语    03      07      03      98      202208  07      郑竹    1989-07-01      女
Time taken: 15.722 seconds, Fetched: 36 row(s)
```

大多数情况下，Hive会对每对JOIN连接对象启动一个MapReduce任务。本例中会首先启动一个MapReduce job对表techer和表course进行连接操作，然后会再启动一个MapReduce job将第一个MapReduce job的输出和表score;进行连接操作。

### ORDER BY

全局排序，一个reduce

1. 使用 ORDER BY 子句排序 ASC（ascend）: 升序（默认） DESC（descend）: 降序
2. ORDER BY 子句在SELECT语句的结尾。

```hive
hive> SELECT * FROM student s LEFT JOIN score sco ON s.s_id = sco.s_id ORDER BY sco.s_score DESC;
WARNING: Hive-on-MR is deprecated in Hive 2 and may not be available in the future versions. Consider using a different execution engine (i.e. spark, tez) or using Hive 1.X releases.
Query ID = root_20220831162600_4cf2c222-3a33-4442-82f9-183034d6f623
Total jobs = 1
SLF4J: Class path contains multiple SLF4J bindings.
SLF4J: Found binding in [jar:file:/usr/local/hive/lib/log4j-slf4j-impl-2.4.1.jar!/org/slf4j/impl/StaticLoggerBinder.class]
SLF4J: Found binding in [jar:file:/usr/local/hadoop-2.7.5/share/hadoop/common/lib/slf4j-log4j12-1.7.10.jar!/org/slf4j/impl/StaticLoggerBinder.class]
SLF4J: See http://www.slf4j.org/codes.html#multiple_bindings for an explanation.
SLF4J: Actual binding is of type [org.apache.logging.slf4j.Log4jLoggerFactory]
2022-08-31 16:26:11     Starting to launch local task to process map join;      maximum memory = 518979584
2022-08-31 16:26:12     Dump the side-table for tag: 1 with group count: 7 into file: file:/tmp/root/bbef36fd-2385-4292-978a-9fdf19b2da63/hive_2022-08-31_16-26-00_468_3536626744031857119-1/-local-10005/HashTable-Stage-2/MapJoin-mapfile71--.hashtable
2022-08-31 16:26:12     Uploaded 1 File to: file:/tmp/root/bbef36fd-2385-4292-978a-9fdf19b2da63/hive_2022-08-31_16-26-00_468_3536626744031857119-1/-local-10005/HashTable-Stage-2/MapJoin-mapfile71--.hashtable (951 bytes)
2022-08-31 16:26:12     End of local task; Time Taken: 1.743 sec.
Execution completed successfully
MapredLocal task succeeded
Launching Job 1 out of 1
Number of reduce tasks determined at compile time: 1
In order to change the average load for a reducer (in bytes):
  set hive.exec.reducers.bytes.per.reducer=<number>
In order to limit the maximum number of reducers:
  set hive.exec.reducers.max=<number>
In order to set a constant number of reducers:
  set mapreduce.job.reduces=<number>
Job running in-process (local Hadoop)
2022-08-31 16:26:15,821 Stage-2 map = 100%,  reduce = 100%
Ended Job = job_local938150601_0017
MapReduce Jobs Launched:
Stage-Stage-2:  HDFS Read: 17120 HDFS Write: 2342 SUCCESS
Total MapReduce CPU Time Spent: 0 msec
OK
01      赵雷    1990-01-01      男      01      03      99      202208
01      赵雷    1990-01-01      男      01      03      99      202207
07      郑竹    1989-07-01      女      07      03      98      202207
07      郑竹    1989-07-01      女      07      03      98      202208
01      赵雷    1990-01-01      男      01      02      90      202207
01      赵雷    1990-01-01      男      01      02      90      202208
07      郑竹    1989-07-01      女      07      02      89      202208
07      郑竹    1989-07-01      女      07      02      89      202207
05      周梅    1991-12-01      女      05      02      87      202207
05      周梅    1991-12-01      女      05      02      87      202208
02      钱电    1990-12-21      男      02      03      80      202208
01      赵雷    1990-01-01      男      01      01      80      202208
02      钱电    1990-12-21      男      02      03      80      202207
01      赵雷    1990-01-01      男      01      01      80      202207
03      孙风    1990-05-20      男      03      03      80      202208
03      孙风    1990-05-20      男      03      02      80      202208
03      孙风    1990-05-20      男      03      01      80      202208
03      孙风    1990-05-20      男      03      03      80      202207
03      孙风    1990-05-20      男      03      02      80      202207
03      孙风    1990-05-20      男      03      01      80      202207
05      周梅    1991-12-01      女      05      01      76      202207
05      周梅    1991-12-01      女      05      01      76      202208
02      钱电    1990-12-21      男      02      01      70      202207
02      钱电    1990-12-21      男      02      01      70      202208
02      钱电    1990-12-21      男      02      02      60      202207
02      钱电    1990-12-21      男      02      02      60      202208
04      李云    1990-08-06      男      04      01      50      202207
04      李云    1990-08-06      男      04      01      50      202208
06      吴兰    1992-03-01      女      06      03      34      202207
06      吴兰    1992-03-01      女      06      03      34      202208
06      吴兰    1992-03-01      女      06      01      31      202207
06      吴兰    1992-03-01      女      06      01      31      202208
04      李云    1990-08-06      男      04      02      30      202207
04      李云    1990-08-06      男      04      02      30      202208
04      李云    1990-08-06      男      04      03      20      202207
04      李云    1990-08-06      男      04      03      20      202208
08      王菊    1990-01-20      女      NULL    NULL    NULL    NULL
Time taken: 15.361 seconds, Fetched: 37 row(s)
```

### SORT BY

每个MapReduce内部进行排序，对全局结果集来说不是排序。

设置reduce个数

```hive
hive> set mapreduce.job.reduces=3;
```

查询成绩按照成绩降序排列

```hive
hive> select * from score sort by s_score;
WARNING: Hive-on-MR is deprecated in Hive 2 and may not be available in the future versions. Consider using a different execution engine (i.e. spark, tez) or using Hive 1.X releases.
Query ID = root_20220831162958_f0e55d69-8970-493f-9ade-c1e56b21cf66
Total jobs = 1
Launching Job 1 out of 1
Number of reduce tasks not specified. Defaulting to jobconf value of: 3
In order to change the average load for a reducer (in bytes):
  set hive.exec.reducers.bytes.per.reducer=<number>
In order to limit the maximum number of reducers:
  set hive.exec.reducers.max=<number>
In order to set a constant number of reducers:
  set mapreduce.job.reduces=<number>
Job running in-process (local Hadoop)
2022-08-31 16:29:59,951 Stage-1 map = 100%,  reduce = 100%
Ended Job = job_local670389688_0018
MapReduce Jobs Launched:
Stage-Stage-1:  HDFS Read: 35536 HDFS Write: 4684 SUCCESS
Total MapReduce CPU Time Spent: 0 msec
OK
04      03      20      202208
06      01      31      202207
06      03      34      202208
04      01      50      202207
02      02      60      202207
02      01      70      202208
03      02      80      202207
02      03      80      202207
03      01      80      202207
03      03      80      202207
03      01      80      202208
02      03      80      202208
01      01      80      202208
05      02      87      202208
07      03      98      202208
07      03      98      202207
04      03      20      202207
04      02      30      202207
04      02      30      202208
02      01      70      202207
03      02      80      202208
03      03      80      202208
05      02      87      202207
07      02      89      202208
07      02      89      202207
01      02      90      202208
01      02      90      202207
01      03      99      202207
06      01      31      202208
06      03      34      202207
04      01      50      202208
02      02      60      202208
05      01      76      202208
05      01      76      202207
01      01      80      202207
01      03      99      202208
Time taken: 1.504 seconds, Fetched: 36 row(s)
```

将查询结果导入到文件中（按照成绩降序排列）

```hive
hive> insert overwrite local directory '/opt/bigdata/hive/sort_result' select * from score sort by s_score;
WARNING: Hive-on-MR is deprecated in Hive 2 and may not be available in the future versions. Consider using a different execution engine (i.e. spark, tez) or using Hive 1.X releases.
Query ID = root_20220831163143_69f14887-68b6-44d7-b9c6-809bd4f0d112
Total jobs = 1
Launching Job 1 out of 1
Number of reduce tasks not specified. Defaulting to jobconf value of: 3
In order to change the average load for a reducer (in bytes):
  set hive.exec.reducers.bytes.per.reducer=<number>
In order to limit the maximum number of reducers:
  set hive.exec.reducers.max=<number>
In order to set a constant number of reducers:
  set mapreduce.job.reduces=<number>
Job running in-process (local Hadoop)
2022-08-31 16:31:45,095 Stage-1 map = 100%,  reduce = 100%
Ended Job = job_local44799780_0019
Moving data to local directory /opt/bigdata/hive/sort_result
MapReduce Jobs Launched:
Stage-Stage-1:  HDFS Read: 36832 HDFS Write: 4684 SUCCESS
Total MapReduce CPU Time Spent: 0 msec
OK
Time taken: 1.325 seconds
```

### DISTRIBUTE BY

Distribute By：类似MR中partition，进行分区，结合sort by使用。注意，Hive要求DISTRIBUTE BY语句要写在SORT BY语句之前。对于distribute by进行测试，一定要分配多reduce进行处理，否则无法看到distribute by的效
果。

案例实操：先按照学生id进行分区，再按照学生成绩进行排序。

设置reduce的个数，将我们对应的s_id划分到对应的reduce当中去

```hive
hive> set mapreduce.job.reduces=7;
```

通过distribute by进行数据的分区

```hive
hive> insert overwrite local directory '/opt/bigdata/hive/sort_result1' select * from score distribute by s_id sort by s_score;
WARNING: Hive-on-MR is deprecated in Hive 2 and may not be available in the future versions. Consider using a different execution engine (i.e. spark, tez) or using Hive 1.X releases.
Query ID = root_20220831163514_7e190db6-af8a-4ec6-89e7-f9958dd7d449
Total jobs = 1
Launching Job 1 out of 1
Number of reduce tasks not specified. Defaulting to jobconf value of: 7
In order to change the average load for a reducer (in bytes):
  set hive.exec.reducers.bytes.per.reducer=<number>
In order to limit the maximum number of reducers:
  set hive.exec.reducers.max=<number>
In order to set a constant number of reducers:
  set mapreduce.job.reduces=<number>
Job running in-process (local Hadoop)
2022-08-31 16:35:15,554 Stage-1 map = 100%,  reduce = 100%
Ended Job = job_local879845742_0020
Moving data to local directory /opt/bigdata/hive/sort_result1
MapReduce Jobs Launched:
Stage-Stage-1:  HDFS Read: 76256 HDFS Write: 9368 SUCCESS
Total MapReduce CPU Time Spent: 0 msec
OK
Time taken: 1.375 seconds
```

### CLUSTER BY

当distribute by和sort by字段相同时，可以使用cluster by方式。cluster by除了具有distribute by的功能外还兼具sort by的功能。但是排序只能是倒序排序，不能指定排序规则为ASC或者DESC。

以下两种写法等价：

```hive
hive> select * from score cluster by s_id;
hive> select * from score distribute by s_id sort by s_id;
```

## 参数配置

开发Hive应用时，不可避免地需要设定Hive的参数。设定Hive的参数可以调优HQL代码的执行 效率，或帮助定位问题。对于一般参数，有以下三种设定方式：

### 配置文件

- 用户自定义配置文件：$HIVE_CONF_DIR/hive-site.xml
- 默认配置文件： $HIVE_CONF_DIR/hive-default.xml

用户自定义配置会覆盖默认配置。另外，Hive也会读入Hadoop的配置，因为Hive是作为Hadoop的客户端启动的，Hive的配置会 覆盖Hadoop的配置。配置文件的设定对本机启动的所有Hive进程都有效。

### 命令行参数

启动Hive（客户端或Server方式）时，可以在命令行添加-hiveconf param=value 来设定参数，例如：

```bash
[root@server01 sort_result2]# hive -hiveconf hive.root.logger=INFO,console
```

这一设定对本次启动的Session（对于Server方式启动，则是所有请求的Sessions）有效。

### 参数声明

```hive
hive> set mapreduce.job.reduces;
mapreduce.job.reduces=3
```

这一设定的作用域也是session级的。这个mapreduce.job.reduces也可以在配置文件中设置。

上述三种设定方式的优先级依次递增。即参数声明覆盖命令行参数，命令行参数覆盖配置文 件设定。注意某些系统级的参数，例如log4j相关的设定，必须用前两种方式设定，因为那些 参数的读取在Session建立以前已经完成了。

## 函数

### 内置函数

内容较多，见《Hive官方文档》

https://cwiki.apache.org/confluence/display/Hive/LanguageManual+UDF

查看系统自带的函数

```hive
hive> show functions;
```

显示自带的函数的用法

```hive
hive> desc function upper;
OK
upper(str) - Returns str with all characters changed to uppercase
Time taken: 0.049 seconds, Fetched: 1 row(s)
```

详细显示自带的函数的用法

```hive
hive> desc function extended upper;
OK
upper(str) - Returns str with all characters changed to uppercase
Synonyms: ucase
Example:
  > SELECT upper('Facebook') FROM src LIMIT 1;
  'FACEBOOK'
Time taken: 0.008 seconds, Fetched: 5 row(s)
```

字符串连接函数： concat

```hive
hive> select concat('abc','def','ghi');
OK
abcdefghi
Time taken: 0.07 seconds, Fetched: 1 row(s)
```

带分隔符字符串连接函数：concat_ws

```hive
hive> select concat_ws(',','abc','def','gh');
OK
abc,def,gh
Time taken: 0.04 seconds, Fetched: 1 row(s)
```

类型转换：cast

```hive
hive> select cast(1.5 as int);
OK
1
Time taken: 0.045 seconds, Fetched: 1 row(s)
```

json解析函数：get_json_object

```hive
hive> select get_json_object('{"name":"jack","age":"20"}','$.name');
OK
jack
Time taken: 0.084 seconds, Fetched: 1 row(s)
```

URL解析函数：parse_url

```hive
hive>  select parse_url('http://facebook.com/path1/p.php?k1=v1&k2=v2#Ref1','HOST');
OK
facebook.com
Time taken: 0.041 seconds, Fetched: 1 row(s)
```

### 自定义函数

Hive 自带了一些函数，比如：max/min等，当Hive提供的内置函数无法满足你的业务处 理需要时，此时就可以考虑使用用户自定义函数(UDF)。根据用户自定义函数类别分为以下三种：

- UDF（User-Defined-Function）

  一进一出

- UDAF（User-Defined Aggregation Function）

  聚集函数，多进一出

  类似于： count / max / min

- UDTF（User-Defined Table-Generating Functions）

  一进多出

  如 lateral view explore()

注意事项：

1. UDF必须要有返回类型，可以返回null，但是返回类型不能为void；
2. UDF中常用Text/LongWritable等类型，不推荐使用java类型；

#### UDF 开发实例

UDF：用户定义（普通）函数，只对单行数值产生作用；

UDF只能实现一进一出的操作。

##### 创建Maven工程

```xml
<dependencies>
    <!-- https://mvnrepository.com/artifact/org.apache.hive/hive-exec -->
    <dependency>
        <groupId>org.apache.hive</groupId>
        <artifactId>hive-exec</artifactId>
        <version>2.1.1</version>
    </dependency>
    <!-- https://mvnrepository.com/artifact/org.apache.hadoop/hadoop-common -->
    <dependency>
        <groupId>org.apache.hadoop</groupId>
        <artifactId>hadoop-common</artifactId>
        <version>2.7.5</version>
    </dependency>
</dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.0</version>
                <configuration>
                    <source>1.8</source>
                    <target>1.8</target>
                    <encoding>UTF-8</encoding>
                </configuration>
            </plugin>
        </plugins>
    </build>
```

##### 开发Java类集成UDF

```java
public class CustomUDF extends UDF{
    public Text evaluate(final Text str){
        String tmp_str = str.toString();
        if(str != null && !tmp_str.equals("")){
          String str_ret =   tmp_str.substring(0, 1).toUpperCase() +
tmp_str.substring(1);
          return  new Text(str_ret);
       }
        return  new Text("");
   }
}
```

##### 项目打包，并上传到hive的lib目录下

##### 添加jar包

```hive
hive> add jar /usr/local/hive/lib/my_upper.jar;
```

##### 设置函数与自定义函数关联

```hive
hive> create temporary function my_upper as 'org.duo.udf.CustomUDF';
```

##### 使用自定义函数

```hive
hive> select my_upper('abc');
```

#### UDTF

User-Defined Table-Generating Functions，用户定义表生成函数，用来解决输入一行输出多行。

继承org.apache.hadoop.hive.ql.udf.generic.GenericUDTF，实现initialize, process, close三个方法：UDTF首先会调用initialize方法，此方法返回UDTF的返回行的信息（返回个数，类型）；初始化完成后，会调用process方法,真正的处理过程在process函数中，在process中，每一次forward()调用产生一行，如果产生多列可以将多个列的值放在一个数组中，然后将该数组传入到forward()函数；最后close()方法调用，对需要清理的方法进行清理

UDTF有两种使用方法，一种直接放到select后面，一种和lateral view一起使用。以explode函数为例来说明(explode函数可以将一个array或者map展开，其中explode(array)使得结果中将array列表里的每个元素生成一行；explode(map)使得结果中将map里的每一对元素作为一行，key为一列，value为一列)

创建表数据：

/opt/weblog/test_message.txt

```markdown
001,allen,usa|china|japan,1|3|7
002,kobe,usa|england|japan,2|3|5
```

创建测试表：

```hive
create table test_message(id int,name string,location array<string>,city array<int>) row format delimited fields terminated by ","
collection items terminated by '|';
```

导入数据：

```hive
load data local inpath "/opt/weblog/test_message.txt" into table test_message;
```

当使用UDTF函数的时候,hive只允许对拆分字段进行访问的。(使用UDTF函数相当于新建了一张表)

```hive
hive> select location[1] from test_message;
OK
china
england
Time taken: 0.214 seconds, Fetched: 2 row(s)
hive> select explode(location) from test_message;
OK
usa
china
japan
usa
england
japan
Time taken: 0.279 seconds, Fetched: 6 row(s)
hive> select name,explode(location) from test_message;
FAILED: SemanticException [Error 10081]: UDTF's are not supported outside the SELECT clause, nor nested in expressions
```

lateral view为侧视图，意义是为了配合UDTF来使用，把某一行数据拆分成多行数据，不加lateral view的UDTF只能提取单个字段拆分，并不能塞会原来数据表中，加上lateral view就可以将拆分的单个字段数据与原始表数据关联上。lateral view explode 相当于一个拆分location字段的虚表,然后与原表进行关联。(subview为视图别名,lc为指定新列别名)

```hive
hive> select name,subview.lc from test_message lateral view explode(location) subview as lc;
OK
allen   usa
allen   china
allen   japan
kobe    usa
kobe    england
kobe    japan
Time taken: 0.095 seconds, Fetched: 6 row(s)
```

#### UDAF

User- Defined Aggregation Funcation；用户定义聚合函数，可对多行数据产生作用；等同与SQL中常用的SUM()，AVG()，也是聚合函数；UDAF实现多进一出。

继承org.apache.hadoop.hive.ql.exec.UDAF。

## 数据压缩

在实际工作当中，hive当中处理的数据，一般都需要经过压缩，前面已经配置过hadoop的压缩，这里的hive也是一样的可以使用压缩来节省MR处理的网络带宽

### MR支持的压缩编码

| 压缩格式 | 工具  | 算法    | 文件扩展名 | 是否可切分 |
| -------- | ----- | ------- | ---------- | ---------- |
| DEFAULT  | 无    | DEFAULT | .deflate   | 否         |
| Gzip     | gzip  | DEFAULT | .gz        | 否         |
| bzip2    | bzip2 | bzip2   | .bz2       | 是         |
| LZO      | lzop  | LZO     | .lzo       | 否         |
| LZ4      | 无    | LZ4     | .lz4       | 否         |
| Snappy   | 无    | Snappy  | .snappy    | 否         |

为了支持多种压缩/解压缩算法，Hadoop引入了编码/解码器，如下表所示

| 压缩格式 | 对应的编码/解码器                          |
| -------- | ------------------------------------------ |
| DEFLATE  | org.apache.hadoop.io.compress.DefaultCodec |
| gzip     | org.apache.hadoop.io.compress.GzipCodec    |
| bzip2    | org.apache.hadoop.io.compress.BZip2Codec   |
| LZO      | com.hadoop.compression.lzo.LzopCodec       |
| LZ4      | org.apache.hadoop.io.compress.Lz4Codec     |
| Snappy   | org.apache.hadoop.io.compress.SnappyCodec  |

压缩性能的比较

| 压缩算法 | 原始文件大小 | 压缩文件大小 | 压缩速度 | 解压速度 |
| -------- | ------------ | ------------ | -------- | -------- |
| gzip     | 8.3GB        | 1.8GB        | 17.5MB/s | 58MB/s   |
| bzip2    | 8.3GB        | 1.1GB        | 2.4MB/s  | 9.5MB/s  |
| LZO      | 8.3GB        | 2.9GB        | 49.3MB/s | 74.6MB/s |

### 压缩配置参数

要在Hadoop中启用压缩，可以配置如下参数（mapred-site.xml文件中）：

| 数                                                | 默认值                                                       | 阶段        | 建议                                         |
| ------------------------------------------------- | ------------------------------------------------------------ | ----------- | :------------------------------------------- |
| io.compression.codecs   （在core-site.xml中配置） | org.apache.hadoop.io.compress.DefaultCodec, org.apache.hadoop.io.compress.GzipCodec, org.apache.hadoop.io.compress.BZip2Codec,org.apache.hadoop.io.compress.Lz4Codec | 输入压缩    | Hadoop使用文件扩展名判断是否支持某种编解码器 |
| mapreduce.map.output.compress                     | false                                                        | mapper输出  | 这个参数设为true启用压缩                     |
| mapreduce.map.output.compress.codec               | org.apache.hadoop.io.compress.DefaultCodec                   | mapper输出  | 使用LZO、LZ4或snappy编解码器在此阶段压缩数据 |
| mapreduce.output.fileoutputformat.compress        | false                                                        | reducer输出 | 这个参数设为true启用压缩                     |
| mapreduce.output.fileoutputformat.compress.codec  | org.apache.hadoop.io.compress. DefaultCodec                  | reducer输出 | 使用标准工具或者编解码器，如gzip和bzip2      |
| mapreduce.output.fileoutputformat.compress.type   | RECORD                                                       |             |                                              |

### Hive开启Map输出阶段压缩

开启map输出阶段压缩可以减少job中map和Reduce task间数据传输量。具体配置如下：

**案例实操：**

1）开启hive中间传输数据压缩功能

~~~sql
hive> set hive.exec.compress.intermediate=true;
~~~

2）开启mapreduce中map输出压缩功能

~~~sql
hive> set mapreduce.map.output.compress=true;
~~~

3）设置mapreduce中map输出数据的压缩方式

~~~sql
hive> set mapreduce.map.output.compress.codec=org.apache.hadoop.io.compress.SnappyCodec;
~~~

4）执行查询语句

```hive
hive> select count(1) from score;
```

### Hive开启Reduce输出阶段压缩

当Hive将输出写入到表中时，输出内容同样可以进行压缩。属性hive.exec.compress.output控制着这个功能。用户可能需要保持默认设置文件中的默认值false，这样默认的输出就是非压缩的纯文本文件了。用户可以通过在查询语句或执行脚本中设置这个值为true，来开启输出结果压缩功能。

**案例实操**：

1）开启hive最终输出数据压缩功能

~~~sql
hive> set hive.exec.compress.output=true;
~~~

2）开启mapreduce最终输出数据压缩

~~~sql
hive> set mapreduce.output.fileoutputformat.compress=true;
~~~

3）设置mapreduce最终数据输出压缩方式

~~~sql
hive> set mapreduce.output.fileoutputformat.compress.codec = org.apache.hadoop.io.compress.SnappyCodec;
~~~

4）设置mapreduce最终数据输出压缩为块压缩

~~~sql
hive> set mapreduce.output.fileoutputformat.compress.type=BLOCK;
~~~

5）测试一下输出结果是否是压缩文件

```hive
hive> insert overwrite local directory '/opt/bigdata/hive/snappy' select * from score distribute by s_id sort by s_id desc;
```

## 数据存储格式

Hive支持的存储数的格式主要有：TEXTFILE（行式存储） 、SEQUENCEFILE(行式存储)、 ORC（列式存储）、PARQUET（列式存储）。

### 列式存储和行式存储

行存储的特点： 查询满足条件的一整行数据的时候，列存储则需要去每个聚集的字段找到对 应的每个列的值，行存储只需要找到其中一个值，其余的值都在相邻地方，所以此时行存储 查询的速度更快。

列存储的特点： 因为每个字段的数据聚集存储，在查询只需要少数几个字段的时候，能大大 减少读取的数据量；每个字段的数据类型一定是相同的，列式存储可以针对性的设计更好的 设计压缩算法。

TEXTFILE和SEQUENCEFILE的存储格式都是基于行存储的； ORC和PARQUET是基于列式存储的。

### 常用数据存储格式

#### TEXTFILE格式

默认格式，数据不做压缩，磁盘开销大，数据解析开销大。可结合Gzip、Bzip2使用

#### ORC格式

Orc (Optimized Row Columnar)是hive 0.11版里引入的新的存储格式。每个Orc文件由1个或多个stripe组成，每个stripe250MB大小，每个Stripe里有三部分 组成，分别是Index Data,Row Data,Stripe Footer：

1. indexData：某些列的索引数据 
2. rowData :真正的数据存储 
3. StripFooter：stripe的元数据信息

#### PARQUET格式

Parquet是面向分析型业务的列式存储格式，由Twitter和Cloudera合作开发，Parquet文件是以二进制方式存储的，所以是不可以直接读取的，文件中包括该文件的数据和 元数据，因此Parquet格式文件是自解析的。通常情况下，在存储Parquet数据的时候会按照Block大小设置行组的大小，由于一般情况下每一个Mapper任务处理数据的最小单位是一个Block，这样可以把每一个行组由一个Mapper任务处理，增大任务执行并行度。

## 文件存储格式与数据压缩结合

### 压缩比

#### TextFile

##### 创建表，存储数据格式为TEXTFILE

```hive
hive> create table log_text ( track_time string, url string, session_id string, referer string, ip string, end_user_id string, city_id string ) ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t' STORED AS TEXTFILE ;
```

##### 向表中加载数据

```hive
hive> load data local inpath '/opt/bigdata/hive/log.data' into table log_text ;
Loading data to table myhive.log_text
OK
Time taken: 1.681 seconds
```

##### 查看表中数据大小

```hive
hive> dfs -du -h /user/hive/warehouse/myhive.db/log_text;
18.1 M  /user/hive/warehouse/myhive.db/log_text/log.data
```

#### ORC

##### 创建表，存储数据格式为ORC

```hive
hive> create table log_orc ( track_time string, url string, session_id string, referer string, ip string, end_user_id string, city_id string ) ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t' STORED AS orc ;
OK
Time taken: 0.164 seconds
```

##### 向表中加载数据

```hive
hive> insert into table log_orc select * from log_text ;
WARNING: Hive-on-MR is deprecated in Hive 2 and may not be available in the future versions. Consider using a different execution engine (i.e. spark, tez) or using Hive 1.X releases.
Query ID = root_20220831195238_dd666792-5a33-42a4-b27f-6f2487b0fb2c
Total jobs = 1
Launching Job 1 out of 1
Number of reduce tasks is set to 0 since there's no reduce operator
Job running in-process (local Hadoop)
2022-08-31 19:52:40,647 Stage-1 map = 0%,  reduce = 0%
2022-08-31 19:52:43,668 Stage-1 map = 100%,  reduce = 0%
Ended Job = job_local225775947_0003
Stage-4 is selected by condition resolver.
Stage-3 is filtered out by condition resolver.
Stage-5 is filtered out by condition resolver.
Moving data to directory hdfs://server01:8020/user/hive/warehouse/myhive.db/log_orc/.hive-staging_hive_2022-08-31_19-52-38_145_6955550278273650885-1/-ext-10000
Loading data to table myhive.log_orc
MapReduce Jobs Launched:
Stage-Stage-1:  HDFS Read: 19015527 HDFS Write: 21928808 SUCCESS
Total MapReduce CPU Time Spent: 0 msec
OK
Time taken: 6.039 seconds
```

##### 查看表中数据大小

```hive
hive> dfs -du -h /user/hive/warehouse/myhive.db/log_orc;
2.8 M  /user/hive/warehouse/myhive.db/log_orc/000000_0
```

#### Parquet

##### 创建表，存储数据格式为parquet

```hive
hive> create table log_parquet ( track_time string, url string, session_id string, referer string, ip string, end_user_id string, city_id string ) ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t' STORED AS PARQUET ;
OK
Time taken: 0.209 seconds
```

##### 向表中加载数据

```hive
hive> insert into table log_parquet select * from log_text ;
WARNING: Hive-on-MR is deprecated in Hive 2 and may not be available in the future versions. Consider using a different execution engine (i.e. spark, tez) or using Hive 1.X releases.
Query ID = root_20220831195410_085df567-8b76-48e5-9574-03d22199fbb7
Total jobs = 3
Launching Job 1 out of 3
Number of reduce tasks is set to 0 since there's no reduce operator
Job running in-process (local Hadoop)
2022-08-31 19:54:11,711 Stage-1 map = 0%,  reduce = 0%
SLF4J: Failed to load class "org.slf4j.impl.StaticLoggerBinder".
SLF4J: Defaulting to no-operation (NOP) logger implementation
SLF4J: See http://www.slf4j.org/codes.html#StaticLoggerBinder for further details.
2022-08-31 19:54:14,722 Stage-1 map = 100%,  reduce = 0%
Ended Job = job_local1188880581_0004
Stage-4 is selected by condition resolver.
Stage-3 is filtered out by condition resolver.
Stage-5 is filtered out by condition resolver.
Moving data to directory hdfs://server01:8020/user/hive/warehouse/myhive.db/log_parquet/.hive-staging_hive_2022-08-31_19-54-10_140_7840170999287515047-1/-ext-10000
Loading data to table myhive.log_parquet
MapReduce Jobs Launched:
Stage-Stage-1:  HDFS Read: 38030486 HDFS Write: 35649792 SUCCESS
Total MapReduce CPU Time Spent: 0 msec
OK
Time taken: 5.024 seconds
```

##### 查看表中数据大小

```hive
hive> dfs -du -h /user/hive/warehouse/myhive.db/log_parquet;
13.1 M  /user/hive/warehouse/myhive.db/log_parquet/000000_0
```

#### 存储文件的压缩比总结

ORC > Parquet > textFile

### 查询速度

#### TextFile

```hive
hive> select count(*) from log_text;
WARNING: Hive-on-MR is deprecated in Hive 2 and may not be available in the future versions. Consider using a different execution engine (i.e. spark, tez) or using Hive 1.X releases.
Query ID = root_20220831195652_1668d4bb-3e42-4a1d-976b-f873345bc230
Total jobs = 1
Launching Job 1 out of 1
Number of reduce tasks determined at compile time: 1
In order to change the average load for a reducer (in bytes):
  set hive.exec.reducers.bytes.per.reducer=<number>
In order to limit the maximum number of reducers:
  set hive.exec.reducers.max=<number>
In order to set a constant number of reducers:
  set mapreduce.job.reduces=<number>
Job running in-process (local Hadoop)
2022-08-31 19:56:54,076 Stage-1 map = 100%,  reduce = 100%
Ended Job = job_local1661940817_0005
MapReduce Jobs Launched:
Stage-Stage-1:  HDFS Read: 114090894 HDFS Write: 71299584 SUCCESS
Total MapReduce CPU Time Spent: 0 msec
OK
100000
Time taken: 1.971 seconds, Fetched: 1 row(s)
```

#### ORC

```hive
hive> select count(*) from log_orc;
WARNING: Hive-on-MR is deprecated in Hive 2 and may not be available in the future versions. Consider using a different execution engine (i.e. spark, tez) or using Hive 1.X releases.
Query ID = root_20220831195718_751c7627-d016-48b4-80be-17d68b17b952
Total jobs = 1
Launching Job 1 out of 1
Number of reduce tasks determined at compile time: 1
In order to change the average load for a reducer (in bytes):
  set hive.exec.reducers.bytes.per.reducer=<number>
In order to limit the maximum number of reducers:
  set hive.exec.reducers.max=<number>
In order to set a constant number of reducers:
  set mapreduce.job.reduces=<number>
Job running in-process (local Hadoop)
2022-08-31 19:57:20,318 Stage-1 map = 100%,  reduce = 100%
Ended Job = job_local1388576768_0006
MapReduce Jobs Launched:
Stage-Stage-1:  HDFS Read: 114124144 HDFS Write: 71299584 SUCCESS
Total MapReduce CPU Time Spent: 0 msec
OK
100000
Time taken: 1.407 seconds, Fetched: 1 row(s)
```

#### Parquet

```hive
hive> select count(*) from log_parquet;
WARNING: Hive-on-MR is deprecated in Hive 2 and may not be available in the future versions. Consider using a different execution engine (i.e. spark, tez) or using Hive 1.X releases.
Query ID = root_20220831195750_534a427d-0cbb-4c16-bed1-4624c6b5c1fc
Total jobs = 1
Launching Job 1 out of 1
Number of reduce tasks determined at compile time: 1
In order to change the average load for a reducer (in bytes):
  set hive.exec.reducers.bytes.per.reducer=<number>
In order to limit the maximum number of reducers:
  set hive.exec.reducers.max=<number>
In order to set a constant number of reducers:
  set mapreduce.job.reduces=<number>
Job running in-process (local Hadoop)
2022-08-31 19:57:51,528 Stage-1 map = 0%,  reduce = 0%
2022-08-31 19:57:52,534 Stage-1 map = 100%,  reduce = 100%
Ended Job = job_local1724600551_0007
MapReduce Jobs Launched:
Stage-Stage-1:  HDFS Read: 141567992 HDFS Write: 71299584 SUCCESS
Total MapReduce CPU Time Spent: 0 msec
OK
100000
Time taken: 2.424 seconds, Fetched: 1 row(s)
```

#### 存储文件的查询速度总结

ORC > TextFile > Parquet

### ORC存储指定压缩方式

官网：https://cwiki.apache.org/confluence/display/Hive/LanguageManual+ORC

#### ORC存储方式的压缩：

| Key                      | Default    | Notes                                                        |
| ------------------------ | ---------- | ------------------------------------------------------------ |
| orc.compress             | `ZLIB`     | high level compression (one of NONE, ZLIB, SNAPPY)           |
| orc.compress.size        | 262,144    | number of bytes in each compression chunk                    |
| orc.stripe.size          | 67,108,864 | number of bytes in each stripe                               |
| orc.row.index.stride     | 10,000     | number of rows between index entries (must be >= 1000)       |
| orc.create.index         | true       | whether to create row indexes                                |
| orc.bloom.filter.columns | ""         | comma separated list of column names for which bloom filter should be created |
| orc.bloom.filter.fpp     | 0.05       | false positive probability for bloom filter (must >0.0 and <1.0) |

#### 创建一个非压缩的的ORC存储方式

##### 建表语句

```hive
hive> create table log_orc_none( track_time string, url string, session_id string, referer string, ip string, end_user_id string, city_id string ) ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t' STORED AS orc tblproperties ("orc.compress"="NONE");
OK
Time taken: 0.173 seconds
```

##### 插入数据

```hive
hive> insert into table log_orc_none select * from log_text ;
WARNING: Hive-on-MR is deprecated in Hive 2 and may not be available in the future versions. Consider using a different execution engine (i.e. spark, tez) or using Hive 1.X releases.
Query ID = root_20220831200111_6fade8cc-2754-49c3-b988-62f87256e111
Total jobs = 1
Launching Job 1 out of 1
Number of reduce tasks is set to 0 since there's no reduce operator
Job running in-process (local Hadoop)
2022-08-31 20:01:13,153 Stage-1 map = 0%,  reduce = 0%
2022-08-31 20:01:14,159 Stage-1 map = 100%,  reduce = 0%
Ended Job = job_local1422694790_0008
Stage-4 is selected by condition resolver.
Stage-3 is filtered out by condition resolver.
Stage-5 is filtered out by condition resolver.
Moving data to directory hdfs://server01:8020/user/hive/warehouse/myhive.db/log_orc_none/.hive-staging_hive_2022-08-31_20-01-11_679_8596868928907176395-1/-ext-10000
Loading data to table myhive.log_orc_none
MapReduce Jobs Launched:
Stage-Stage-1:  HDFS Read: 89798875 HDFS Write: 43714105 SUCCESS
Total MapReduce CPU Time Spent: 0 msec
OK
Time taken: 2.9 seconds
```

##### 查看插入后数据

```hive
hive> dfs -du -h /user/hive/warehouse/myhive.db/log_orc_none;
7.7 M  /user/hive/warehouse/myhive.db/log_orc_none/000000_0
```

#### 创建一个SNAPPY压缩的ORC存储方式

##### 建表语句

```hive
hive> dfs -du -h /user/hive/warehouse/myhive.db/log_orc_none;
7.7 M  /user/hive/warehouse/myhive.db/log_orc_none/000000_0
hive> create table log_orc_snappy( track_time string, url string, session_id string, referer string, ip string, end_user_id string, city_id string ) ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t' STORED AS orc tblproperties ("orc.compress"="SNAPPY");
OK
Time taken: 0.127 seconds
```

##### 插入数据

```hive
hive> insert into table log_orc_snappy select * from log_text ;
WARNING: Hive-on-MR is deprecated in Hive 2 and may not be available in the future versions. Consider using a different execution engine (i.e. spark, tez) or using Hive 1.X releases.
Query ID = root_20220831200248_4151155d-f785-49c0-8f62-57bb41330798
Total jobs = 1
Launching Job 1 out of 1
Number of reduce tasks is set to 0 since there's no reduce operator
Job running in-process (local Hadoop)
2022-08-31 20:02:50,238 Stage-1 map = 0%,  reduce = 0%
2022-08-31 20:02:51,242 Stage-1 map = 100%,  reduce = 0%
Ended Job = job_local870189240_0009
Stage-4 is selected by condition resolver.
Stage-3 is filtered out by condition resolver.
Stage-5 is filtered out by condition resolver.
Moving data to directory hdfs://server01:8020/user/hive/warehouse/myhive.db/log_orc_snappy/.hive-staging_hive_2022-08-31_20-02-48_737_7439308229440473484-1/-ext-10000
Loading data to table myhive.log_orc_snappy
MapReduce Jobs Launched:
Stage-Stage-1:  HDFS Read: 108813839 HDFS Write: 47680867 SUCCESS
Total MapReduce CPU Time Spent: 0 msec
OK
Time taken: 2.944 seconds
```

##### 查看插入后数据

```hive
hive> dfs -du -h /user/hive/warehouse/myhive.db/log_orc_snappy ;
3.8 M  /user/hive/warehouse/myhive.db/log_orc_snappy/000000_0
```

### 存储方式和压缩总结：

在实际的项目开发当中，hive表的数据存储格式一般选择：orc或parquet。压缩方式一般选 择snappy。

## 调优

### Fetch抓取

Hive中对某些情况的查询可以不必使用MapReduce计算。例如：SELECT * FROM score;在这种情况下，Hive可以简单地读取score对应的存储目录下的文件，然后输出查询结果到控制台。通过设置hive.fetch.task.conversion参数,可以控制查询语句是否走MapReduce，该参数默认为more

1. 把hive.fetch.task.conversion设置成none，然后执行查询语句，都会执行mapreduce程序。

   ```hive
   hive> set hive.fetch.task.conversion=none;
   hive> select * from score;
   hive> select s_score from score;
   hive> select s_score from score limit 3;

2. 把hive.fetch.task.conversion设置成more，然后执行查询语句，如下查询方式都不会执行mapreduce程序。

   ```hive
   hive> set hive.fetch.task.conversion=more;
   hive> select * from score;
   hive> select s_score from score;
   hive> select s_score from score limit 3;
   ```

### 本地模式

大多数的Hadoop Job是需要Hadoop提供的完整的可扩展性来处理大数据集的。不过，有时Hive的输入数据量是非常小的。在这种情况下，为查询触发执行任务时消耗可能会比实际job的执行时间要多的多。对于大多数这种情况，Hive可以通过本地模式在单台机器上处理所有的任务。对于小数据集，执行时间可以明显被缩短。用户可以通过设置hive.exec.mode.local.auto的值为true，来让Hive在适当的时候自动启动这个优化，该参数默认为false。

1. 开启本地模式，并执行查询语句

   ```hive
   hive> set hive.exec.mode.local.auto=true;
   hive> select * from score cluster by s_id;
   ```

   

2. 关闭本地模式，并执行查询语句

   ```hive
   hive> set hive.exec.mode.local.auto=false;
   hive> select * from score cluster by s_id;
   ```

### MapJoin

如果不指定MapJoin或者不符合MapJoin的条件，那么Hive解析器会在Reduce阶段完成join,容易发生数据倾斜。可以用MapJoin把小表全部加载到内存在map端进行join，避免reducer处理，该参数默认为true。

开启MapJoin参数设置：

1. 设置自动选择Mapjoin

   ```hive
   hive> set hive.auto.convert.join = true;
   ```

2. 大表小表的阈值设置（默认25M以下认为是小表）：

   ```hive
   hive> set hive.mapjoin.smalltable.filesize=25123456;
   ```

### Group By

默认情况下，Map阶段同一Key数据分发给一个reduce，当一个key数据过大时就倾斜了。并不是所有的聚合操作都需要在Reduce端完成，很多聚合操作都可以先在Map端进行部分聚合，最后在Reduce端得出最终结果，该参数默认为true。

1. 是否在Map端进行聚合，默认为True

   ```hive
   hive> set hive.map.aggr = true;
   ```

2. 在Map端进行聚合操作的条目数目

   ```hive
   hive> hive.groupby.mapaggr.checkinterval = 100000;
   ```

3. 有数据倾斜的时候进行负载均衡（默认是false）

   ```hive
   hive> hive.groupby.skewindata = true;
   ```

当选项设定为 true，生成的查询计划会有两个MR Job：

1. 第一个MR Job中，Map的输出结果会随机分布到Reduce中，每个Reduce做部分聚合操作，并输出结果，这样处理的结果是相同的Group By Key有可能被分发到不同的Reduce中，从而达到负载均衡的目的；
2. 第二个MR Job再根据预处理的数据结果按照Group By Key分布到Reduce中（这个过程可以保证相同的Group By Key被分布到同一个Reduce中），最后完成最终的聚合操作。

### Count(distinct)

数据量小的时候无所谓，数据量大的情况下，由于COUNT DISTINCT操作需要用一个Reduce Task来完成，这一个Reduce需要处理的数据量太大，就会导致整个Job很难完成，一般COUNT DISTINCT使用先GROUP BY再COUNT的方式替换：

```hive
hive> select count(distinct s_id) from score;
hive> select count(s_id) from (select id from score group by s_id) a;
```

### 笛卡尔积

尽量避免笛卡尔积，即避免join的时候不加on条件，或者无效的on条件，Hive只能使用1个reducer来完成笛卡尔积。

### 动态分区调整

往hive分区表中插入数据时，hive提供了一个动态分区功能，其可以基于查询参数的位置去推断分区的名称，从而建立分区。使用Hive的动态分区，需要进行相应的配置。

Hive的动态分区是以第一个表的分区规则，来对应第二个表的分区规则，将第一个表的所有分区，全部拷贝到第二个表中来，第二个表在加载数据的时候，不需要指定分区了，直接用第一个表的分区即可

开启动态分区参数设置

1. 开启动态分区功能（默认true，开启）

   ```hive
   hive> set hive.exec.dynamic.partition=true;
   ```

2. 设置为非严格模式（动态分区的模式，默认strict，表示必须指定至少一个分区为静态分区，nonstrict模式表示允许所有的分区字段都可以使用动态分区。）

   ```hive
   hive> set hive.exec.dynamic.partition.mode=nonstrict;
   ```

3. 在所有执行MR的节点上，最大一共可以创建多少个动态分区。

   ```hive
   hive> set hive.exec.max.dynamic.partitions=1000;
   ```

4. 在每个执行MR的节点上，最大可以创建多少个动态分区。该参数需要根据实际的数据来设定。

   ```hive
   hive> set hive.exec.max.dynamic.partitions.pernode=100;
   ```

5. 整个MR Job中，最大可以创建多少个HDFS文件。

   ```hive
   hive> set hive.exec.max.created.files=100000;
   ```

6. 当有空分区生成时，是否抛出异常。一般不需要设置。

   ```hive
   hive> set hive.error.on.empty.partition=false;
   ```

案例操作

需求：将ori中的数据按照时间(如：20111231234568)，插入到目标表ori_partitioned的相应分区中。

1. 准备数据原表

   ```hive
   hive> create table ori_partitioned(id bigint, time bigint, uid string, keyword string, url_rank int, click_num int, click_url string) PARTITIONED BY (p_time bigint) row format delimited fields terminated by '\t';
   OK
   Time taken: 0.74 seconds
   hive> load data local inpath '/opt/bigdata/hive/small_data' into  table ori_partitioned partition (p_time='20111230000010');
   Loading data to table myhive.ori_partitioned partition (p_time=20111230000010)
   OK
   Time taken: 2.698 seconds
   hive> load data local inpath '/opt/bigdata/hive/small_data' into  table ori_partitioned partition (p_time='20111230000011');
   Loading data to table myhive.ori_partitioned partition (p_time=20111230000011)
   OK
   Time taken: 1.063 seconds
   ```

2. 创建目标分区表

   ```hive
   hive> create table ori_partitioned_target(id bigint, time bigint, uid string, keyword string, url_rank int, click_num int, click_url string) PARTITIONED BY (p_time STRING) row format delimited fields terminated by '\t';
   OK
   Time taken: 0.164 seconds
   ```

3. 向目标分区表加载数据

   如果按照之前介绍的往指定一个分区中Insert数据，那么这个需求很不容易实现。这时候就需要使用动态分区来实现。

   ```hive
   hive> INSERT overwrite TABLE ori_partitioned_target PARTITION (p_time) SELECT id, time, uid, keyword, url_rank, click_num, click_url, p_time FROM ori_partitioned;
   WARNING: Hive-on-MR is deprecated in Hive 2 and may not be available in the future versions. Consider using a different execution engine (i.e. spark, tez) or using Hive 1.X releases.
   Query ID = root_20220901141116_eb4d677c-b925-4756-9aa7-cebbe8f53b24
   Total jobs = 3
   Launching Job 1 out of 3
   Number of reduce tasks is set to 0 since there's no reduce operator
   Job running in-process (local Hadoop)
   2022-09-01 14:11:20,568 Stage-1 map = 0%,  reduce = 0%
   2022-09-01 14:11:24,624 Stage-1 map = 100%,  reduce = 0%
   Ended Job = job_local885366507_0001
   Stage-4 is selected by condition resolver.
   Stage-3 is filtered out by condition resolver.
   Stage-5 is filtered out by condition resolver.
   Moving data to directory hdfs://server01:8020/user/hive/warehouse/myhive.db/ori_partitioned_target/.hive-staging_hive_2022-09-01_14-11-16_513_8999722180766104875-1/-ext-10000
   Loading data to table myhive.ori_partitioned_target partition (p_time=null)
   
   
            Time taken to load dynamic partitions: 0.443 seconds
            Time taken for adding to write entity : 0.001 seconds
   MapReduce Jobs Launched:
   Stage-Stage-1:  HDFS Read: 24036712 HDFS Write: 38207837 SUCCESS
   Total MapReduce CPU Time Spent: 0 msec
   OK
   Time taken: 9.284 seconds
   ```

   注意：在SELECT子句的最后几个字段，必须对应前面PARTITION (p_time)中指定的分区字段，包括顺序。

4. 查看分区

   ```hive
   hive> show partitions ori_partitioned_target; 
   ```

### 并行执行

Hive会将一个查询转化成一个或者多个阶段。这样的阶段可以是MapReduce阶段、抽样阶段、合并阶段、limit阶段。或者Hive执行过程中可能需要的其他阶段。默认情况下，Hive一次只会执行一个阶段。不过，某个特定的job可能包含众多的阶段，而这些阶段可能并非完全互相依赖的，也就是说有些阶段是可以并行执行的，这样可能使得整个job的执行时间缩短。不过，如果有更多的阶段可以并行执行，那么job可能就越快完成。通过设置参数hive.exec.parallel值为true，就可以开启并发执行。不过，在共享集群中，需要注意下，如果job中并行阶段增多，那么集群利用率就会增加，该参数的默认值为false。

```hive
hive> hive.exec.parallel = true;
```

### 严格模式

Hive提供了一个严格模式，可以防止用户执行那些可能意向不到的不好的影响的查询。开启严格模式需要修改hive.mapred.mode值为strict，开启严格模式可以禁止3种类型的查询，该参数默认为非严格模式nonstrict 。

```hive
hive> set hive.mapred.mode = strict;
```

1. 对于分区表，在where语句中必须含有分区字段作为过滤条件来限制范围，否则不允许执行`。换句话说，就是用户不允许扫描所有分区。进行这个限制的原因是，通常分区表都拥有非常大的数据集，而且数据增加迅速。没有进行分区限制的查询可能会消耗令人不可接受的巨大资源来处理这个表。

2. 对于使用了order by语句的查询，要求必须使用limit语句`。因为order by为了执行排序过程会将所有的结果数据分发到同一个Reducer中进行处理，强制要求用户增加这个LIMIT语句可以防止Reducer额外执行很长一段时间。

3. 限制笛卡尔积的查询`。对关系型数据库非常了解的用户可能期望在执行JOIN查询的时候不使用ON语句而是使用where语句，这样关系数据库的执行优化器就可以高效地将WHERE语句转化成那个ON语句。不幸的是，Hive并不会执行这种优化，因此，如果表足够大，那么这个查询就会出现不可控的情况。

### JVM重用

JVM重用是Hadoop调优参数的内容，其对Hive的性能具有非常大的影响，特别是对于很难避免小文件的场景或task特别多的场景，这类场景大多数执行时间都很短。Hadoop的默认配置通常是使用派生JVM来执行map和Reduce任务的。这时JVM的启动过程可能会造成相当大的开销，尤其是执行的job包含有成百上千task任务的情况。JVM重用可以使得JVM实例在同一个job中重新使用N次。N的值可以在Hadoop的mapred-site.xml文件中进行配置。通常在10-20之间，具体多少需要根据具体业务场景测试得出。

在hive当中通过mapred.job.reuse.jvm.num.tasks来设置jvm重用：

```hive
hive> set  mapred.job.reuse.jvm.num.tasks = 10;
```

这个功能的缺点是，开启JVM重用将一直占用使用到的task插槽，以便进行重用，直到任务完成后才能释放。如果某个“不平衡的”job中有某几个reduce task执行的时间要比其他Reduce task消耗的时间多的多的话，那么保留的插槽就会一直空闲着却无法被其他的job使用，直到所有的task都结束了才会释放。

### 推测执行

在分布式集群环境下，因为程序Bug（包括Hadoop本身的bug），负载不均衡或者资源分布不均等原因，会造成同一个作业的多个任务之间运行速度不一致，有些任务的运行速度可能明显慢于其他任务（比如一个作业的某个任务进度只有50%，而其他所有任务已经运行完毕），则这些任务会拖慢作业的整体执行进度。为了避免这种情况发生，Hadoop采用了推测执行（Speculative Execution）机制，它根据一定的法则推测出“拖后腿”的任务，并为这样的任务启动一个备份任务，让该任务与原始任务同时处理同一份数据，并最终选用最先成功运行完成任务的计算结果作为最终结果。

```hive
hive> set mapred.map.tasks.speculative.execution=true
hive> set mapred.reduce.tasks.speculative.execution=true
hive> set hive.mapred.reduce.tasks.speculative.execution=true;
```

关于调优这些推测执行变量，还很难给一个具体的建议。如果用户对于运行时的偏差非常敏感的话，那么可以将这些功能关闭掉。如果用户因为输入数据量很大而需要执行长时间的map或者Reduce task的话，那么启动推测执行造成的浪费是非常巨大大。

# Impala

## 基本介绍

impala是cloudera提供的一款高效率的sql查询工具，提供实时的查询效果，官方测试性能比hive快10到100倍，其sql查询比sparkSQL还要更加快速，号称是当前大数据领域最快的查询sql工具，

impala是参照谷歌的新三篇论文（Caffeine--网络搜索引擎、Pregel--分布式图计算、Dremel--交互式分析工具）当中的Dremel实现而来，其中旧三篇论文分别是（BigTable，GFS，MapReduce）分别对应HBase、HDFS以及MapReduce。

impala是基于hive并使用内存进行计算，兼顾数据仓库，具有实时，批处理，多并发等优点。

## Impala与Hive关系

impala是基于hive的大数据分析查询引擎，直接使用hive的元数据库metadata，意味着impala元数据都存储在hive的metastore当中，并且impala兼容hive的绝大多数sql语法。所以需要安装impala的话，必须先安装hive，保证hive安装成功，并且还需要启动hive的metastore服务。

```bash
[root@server01 tmp]# nohup hive --service metastore >> ~/metastore.log 2>&1 &
```

Hive元数据包含用Hive创建的database、table等元信息。元数据存储在关系型数据库中，如Derby、MySQL等。

客户端连接metastore服务，metastore再去连接MySQL数据库来存取元数据。有了metastore服务，就可以有多个客户端同时连接，而且这些客户端不需要知道MySQL数据库的用户名和密码，只需要连接metastore 服务即可。

Hive适合于长时间的批处理查询分析，而Impala适合于实时交互式SQL查询。可以先使用hive进行数据转换处理，之后使用Impala在Hive处理后的结果数据集上进行快速的数据分析。

## Impala与Hive异同

Impala 与Hive都是构建在Hadoop之上的数据查询工具各有不同的侧重适应面，但从客户端使用来看Impala与Hive有很多的共同之处，如数据表元数据、ODBC/JDBC驱动、SQL语法、灵活的文件格式、存储资源池等。

但是Impala跟Hive最大的优化区别在于：没有使用 MapReduce进行并行计算，虽然MapReduce是非常好的并行计算框架，但它更多的面向批处理模式，而不是面向交互式的SQL执行。与 MapReduce相比，Impala把整个查询分成一执行计划树，而不是一连串的MapReduce任务，在分发执行计划后，Impala使用拉式获取数据的方式获取结果，把结果数据组成按执行树流式传递汇集，减少的了把中间结果写入磁盘的步骤，再从磁盘读取数据的开销。Impala使用服务的方式避免每次执行查询都需要启动的开销，即相比Hive没了MapReduce启动时间。

### Impala使用的优化技术

1. 使用LLVM产生运行代码，针对特定查询生成特定代码，同时使用Inline的方式减少函数调用的开销，加快执行效率。(C++特性)
2. 充分利用可用的硬件指令（SSE4.2）。
3. 更好的IO调度，Impala知道数据块所在的磁盘位置能够更好的利用多磁盘的优势，同时Impala支持直接数据块读取和本地代码计算checksum。
4. 通过选择合适数据存储格式可以得到最好性能（Impala支持多种存储格式）。
5. 最大使用内存，中间结果不写磁盘，及时通过网络以stream的方式传递。

### 执行计划

#### Hive

依赖于MapReduce执行框架，执行计划分成 map->shuffle->reduce->map->shuffle->reduce…的模型。如果一个Query会 被编译成多轮MapReduce，则会有更多的写中间结果。由于MapReduce执行框架本身的特点，过多的中间过程会增加整个Query的执行时间。

#### Impala

把执行计划表现为一棵完整的执行计划树，可以更自然地分发执行计划到各个Impalad执行查询，而不用像Hive那样把它组合成管道型的 map->reduce模式，以此保证Impala有更好的并发性和避免不必要的中间sort与shuffle。

### 数据流

#### Hive

采用推的方式，每一个计算节点计算完成后将数据主动推给后续节点。

#### Impala

用拉的方式，后续节点通过getNext主动向前面节点要数据，以此方式数据可以流式的返回给客户端，且只要有1条数据被处理完，就可以立即展现出来，而不用等到全部处理完成，更符合SQL交互式查询使用。

### 内存使用

#### Hive

在执行过程中如果内存放不下所有数据，则会使用外存，以保证Query能顺序执行完。每一轮MapReduce结束，中间结果也会写入HDFS中，同样由于MapReduce执行架构的特性，shuffle过程也会有写本地磁盘的操作。

#### Impala

在遇到内存放不下数据时，版本1.0.1是直接返回错误，而不会利用外存，以后版本应该会进行改进。这使用得Impala目前处理Query会受到一定的限制，最好还是与Hive配合使用。

### 调度

#### Hive

任务调度依赖于Hadoop的调度策略。

#### Impala

调度由自己完成，目前只有一种调度器simple-schedule，它会尽量满足数据的局部性，扫描数据的进程尽量靠近数据本身所在的物理机器。调度器 目前还比较简单，在SimpleScheduler::GetBackend中可以看到，现在还没有考虑负载，网络IO状况等因素进行调度。但目前 Impala已经有对执行过程的性能统计分析，应该以后版本会利用这些统计信息进行调度吧。

### 容错

#### Hive

依赖于Hadoop的容错能力。

#### Impala

在查询过程中，没有容错逻辑，如果在执行过程中发生故障，则直接返回错误（这与Impala的设计有关，因为Impala定位于实时查询，一次查询失败， 再查一次就好了，再查一次的成本很低）。

### 适用面

#### Hive

复杂的批处理查询任务，数据转换任务。

#### Impala

实时数据分析，因为不支持UDF，能处理的问题域有一定的限制，与Hive配合使用,对Hive的结果数据集进行实时分析。

## Impala架构

Impala主要由Impalad、 State Store、Catalogd和CLI组成。

### Impalad

与DataNode运行在同一节点上，由Impalad进程表示，它接收客户端的查询请求（接收查询请求的Impalad为Coordinator，Coordinator通过JNI调用java前端解释SQL查询语句，生成查询计划树，再通过调度器把执行计划分发给具有相应数据的其它Impalad进行执行），读写数据，并行执行查询，并把结果通过网络流式的传送回给Coordinator，由Coordinator返回给客户端。同时Impalad也与State Store保持连接，用于确定哪个Impalad是健康和可以接受新的工作。在Impalad中启动三个ThriftServer: beeswax_server（连接客户端），hs2_server（借用Hive元数据）， be_server（Impalad内部使用）和一个ImpalaServer服务。

### Impala State Store

跟踪集群中的Impalad的健康状态及位置信息，由statestored进程表示，它通过创建多个线程来处理Impalad的注册订阅和与各Impalad保持心跳连接，各Impalad都会缓存一份State Store中的信息，当State Store离线后（Impalad发现State Store处于离线时，会进入recovery模式，反复注册，当State Store重新加入集群后，自动恢复正常，更新缓存数据）因为Impalad有State Store的缓存仍然可以工作，但会因为有些Impalad失效了，而已缓存数据无法更新，导致把执行计划分配给了失效的Impalad，导致查询失败。

### CLI

CLI: 提供给用户查询使用的命令行工具（Impala Shell使用python实现），同时Impala还提供了Hue，JDBC， ODBC使用接口。

### Catalogd

Catalogd：作为metadata访问网关，从Hive Metastore等外部catalog中获取元数据信息，放到impala自己的catalog结构中。impalad执行ddl命令时通过catalogd由其代为执行，该更新则由statestored广播。

## Impala查询处理过程

Impalad分为Java前端与C++处理后端，接受客户端连接的Impalad即作为这次查询的Coordinator，Coordinator通过JNI调用Java前端对用户的查询SQL进行分析生成执行计划树。

Java前端产生的执行计划树以Thrift数据格式返回给C++后端（Coordinator）（执行计划分为多个阶段，每一个阶段叫做一个PlanFragment，每一个PlanFragment在执行时可以由多个Impalad实例并行执行(有些PlanFragment只能由一个Impalad实例执行,如聚合操作)，整个执行计划为一执行计划树）。

Coordinator根据执行计划，数据存储信息（Impala通过libhdfs与HDFS进行交互。通过hdfsGetHosts方法获得文件数据块所在节点的位置信息），通过调度器（现在只有simple-scheduler, 使用round-robin算法）Coordinator::Exec对生成的执行计划树分配给相应的后端执行器Impalad执行（查询会使用LLVM进行代码生成，编译，执行），通过调用GetNext()方法获取计算结果。

如果是insert语句，则将计算结果通过libhdfs写回HDFS当所有输入数据被消耗光，执行结束，之后注销此次查询服务。

# Sqoop

## 介绍 

Apache Sqoop是在Hadoop生态体系和RDBMS体系之间传送数据的一种工具。来自于Apache软件基金会提供。
Sqoop工作机制是将导入或导出命令翻译成mapreduce程序来实现。在翻译出的mapreduce中主要是对inputformat和outputformat进行定制。

Hadoop生态系统包括：HDFS、Hive、Hbase等

RDBMS体系包括：Mysql、Oracle、DB2等

Sqoop可以理解为：“SQL 到 Hadoop 和 Hadoop 到SQL”。

站在Apache立场看待数据流转问题，可以分为数据的导入导出:

Import：数据导入。RDBMS----->Hadoop

Export：数据导出。Hadoop---->RDBMS

## 安装

### 环境变量

```sh
export SQOOP_HOME=/usr/local/sqoop
export PATH=$PATH:$SQOOP_HOME/bin
```

### 配置文件修改

```bash
[root@server01 conf]# cd $SQOOP_HOME/conf
[root@server01 conf]# mv sqoop-env-template.sh sqoop-env.sh
[root@server01 conf]# vim sqoop-env.sh
#修改如下三个配置项
export HADOOP_COMMON_HOME=/usr/local/hadoop-2.7.5
export HADOOP_MAPRED_HOME=/usr/local/hadoop-2.7.5
export HIVE_HOME=/usr/local/hive
#加入mysql的jdbc驱动包
[root@server01 conf]# cp /usr/local/hive/lib/mysql-connector-java-5.1.38.jar $SQOOP_HOME/lib/
#验证启动
[root@server01 conf]# sqoop list-databases --connect jdbc:mysql://localhost:3306/ --username root --password 123456
```

## 全量导入hdfs

```bash
[root@server01 conf]# sqoop import --connect jdbc:mysql://server01:3306/userdb --username root --password 123456 --delete-target-dir --fields-terminated-by '\t' --split-by id --target-dir /sqoopresult1 --table emp --m 2
```

在HDFS上默认用逗号,分隔emp表的数据和字段。可以通过--fields-terminated-by '\t'来指定分隔符；

--split-by id通常配合-m 2参数使用。用于指定根据哪个字段进行划分并启动多少个maptask。首先sqoop会向关系型数据库比如mysql发送一个命令:select max(id),min(id) from test。然后会把max、min之间的区间平均分为10分，最后10个并行的map去找数据库，导数据就正式开始

## 全量导入hive

### 方式一：先复制表结构到hive中再导入数据

复制表结构：

```hive
[root@server01 conf]# sqoop create-hive-table --connect jdbc:mysql://server01:3306/userdb --table emp_add --username root --password 123456 --hive-table myhive.emp_add_sp
```

--table emp_add为mysql中的数据库sqoopdb中的表。

--hive-table emp_add_sp 为hive中新建的表名称。

从关系数据库导入文件到hive中：

```hive
[root@server01 conf]# sqoop import --connect jdbc:mysql://server01:3306/userdb --username root --password 123456 --table emp_add --hive-table myhive.emp_add_sp --hive-import --m 1
```

### 方式二：直接复制表结构数据到hive中

包括表结构和数据会一起导入，并且在hive中会自动创建同名的表

```hive
[root@server01 conf]# sqoop import --connect jdbc:mysql://server01:3306/userdb --username root --password 123456 --table emp_conn --hive-import --m 1 --hive-database myhive;
```

## 导入表数据子集

### where过滤

--where可以指定从关系数据库导入数据时的查询条件。它执行在数据库服务器相应的SQL查询，并将结果存储在HDFS的目标目录。

```bash
[root@server01 conf]# sqoop import --connect jdbc:mysql://server01:3306/userdb --username root --password 123456 --where "city ='sec-bad'" --target-dir /wherequery --table emp_add --m 1
```

### query查询

```hive
[root@server01 conf]# sqoop import --connect jdbc:mysql://server01:3306/userdb --username root --password 123456 --target-dir /wherequery12 --query 'select id,name,deg from emp WHERE  id>1203 and $CONDITIONS' --split-by id --fields-terminated-by '\001' --m 2
```

注意事项：

- 使用sql语句来进行查找是不能加参数--table 
- 并且必须要添加where条件，
- 并且where条件后面必须带一个$CONDITIONS 这个字符串，
- 并且这个sql语句必须用单引号，不能用双引号

## 增量导入

-check-column (col)	

用来指定一些列，这些列在增量导入时用来检查这些数据是否作为增量数据进行导入，和关系型数据库中的自增字段及时间戳类似。 

注意:这些被指定的列的类型不能使任意字符类型，如char、varchar等类型都是不可以的，同时-- check-column可以去指定多个列。

--incremental (mode)	

append：追加，比如对大于last-value指定的值之后的记录进行追加导入。lastmodified：最后的修改时间，追加

last-value指定的日期之后的记录

--last-value (value)

指定自从上次导入后列的最大值（大于该指定的值），也可以自己设定某一值

### Append模式增量导入

执行以下指令先将数据导入

```bash
[root@server01 ~]# sqoop import --connect jdbc:mysql://server01:3306/userdb --username root --password 123456 --target-dir /appendresult --table emp --m 1
```

在mysql的emp中插入2条数据:

```sql
insert into `userdb`.`emp` (`id`, `name`, `deg`, `salary`, `dept`) values ('1206', 'allen', 'admin', '30000', 'tp');
insert into `userdb`.`emp` (`id`, `name`, `deg`, `salary`, `dept`) values ('1207', 'woon', 'admin', '40000', 'tp');
```

执行如下的指令，实现增量的导入

```bash
[root@server01 ~]# sqoop import --connect jdbc:mysql://server01:3306/userdb --username root --password 123456 --table emp --m 1 --target-dir /appendresult --incremental append --check-column id --last-value 1205;
```

### Lastmodified模式:append增量导入

首先创建一个customer表，指定一个时间戳字段，此处的时间戳设置为在数据的产生和更新时都会发生改变。

```sql
CREATE TABLE customertest(id INT,NAME VARCHAR(20),last_mod TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP);
```

分别插入如下记录:

```sql
insert into customertest(id,name) values(1,'neil');
insert into customertest(id,name) values(2,'jack');
insert into customertest(id,name) values(3,'martin');
insert into customertest(id,name) values(4,'tony');
insert into customertest(id,name) values(5,'eric');

```

执行sqoop指令将数据全部导入hdfs:

```bash
[root@server01 ~]# sqoop import --connect jdbc:mysql://server01:3306/userdb --username root --password 123456 --target-dir /lastmodifiedresult --table customertest --m 1
```

再次插入一条数据进入customertest表

```sql
INSERT INTO customertest(id,NAME) VALUES(6,'james');
```

使用incremental的方式进行增量的导入

```bash
[root@server01 ~]# sqoop import --connect jdbc:mysql://server01:3306/userdb --username root --password 123456 --table customertest --target-dir /lastmodifiedresult --check-column last_mod --incremental lastmodified --last-value "2022-09-02 12:32:36" --m 1 --append
```

此处发现此处插入了2条数据，这是为什么呢？ 这是因为采用lastmodified模式去处理增量时，会将大于等于last-value值的数据当做增量插入

### Lastmodified模式:merge-key增量导入

使用lastmodified模式进行增量处理要指定增量数据是以append模式(附加)还是merge-key(合并)模式添加

接下来使用merge-by的模式进行增量更新

首先更新id为1的name字段。

```sql
UPDATE customertest SET NAME = 'Neil' WHERE id = 1;
```

执行如下指令，把id字段作为merge-key

```bash
sqoop import --connect jdbc:mysql://server01:3306/userdb --username root --password 123456 --table customertest --target-dir /lastmodifiedresult --check-column last_mod --incremental lastmodified --last-value "2022-09-02 12:56:54" --m 1 --merge-key id
```

由于merge-key这种模式是进行了一次完整的mapreduce操作，因此最终我们在lastmodifiedresult文件夹下可以看到生成的为part-r-00000这样的文件，会发现id=1的name已经得到修改，同时新增了id=6的数据

## Sqoop导出

将数据从Hadoop生态体系导出到RDBMS数据库导出前，目标表必须存在于目标数据库中。

export有三种模式：

1. 默认操作是从将文件中的数据使用INSERT语句插入到表中。
2. 更新模式：Sqoop将生成UPDATE替换数据库中现有记录的语句。
3. 调用模式：Sqoop将为每条记录创建一个存储过程调用。

### 默认模式

默认情况下，sqoop export将每行输入记录转换成一条INSERT语句，添加到目标数据库表中。如果数据库中的表具有约束条件（例如，其值必须唯一的主键列）并且已有数据存在，则必须注意避免插入违反这些约束条件的记录。如果INSERT语句失败，导出过程将失败。此模式主要用于将记录导出到可以接收这些结果的空表中。通常用于全表数据导出。导出时可以是将Hive表中的全部记录或者HDFS数据（可以是全部字段也可以部分字段）导出到Mysql目标表。

准备HDFS数据，在HDFS文件系统中“/emp_data/”目录的下创建一个文件emp_data.txt

```markdown
1201,gopal,manager,50000,TP
1202,manisha,preader,50000,TP
1203,kalil,php dev,30000,AC
1204,prasanth,php dev,30000,AC
1205,kranthi,admin,20000,TP
1206,satishp,grpdes,20000,GR
```

手动创建mysql中的目标表

```sql
CREATE TABLE employee ( 
   id INT NOT NULL PRIMARY KEY, 
   NAME VARCHAR(20), 
   deg VARCHAR(20),
   salary INT,
   dept VARCHAR(10));
```

执行导出命令

```bash
sqoop export --connect jdbc:mysql://server01:3306/userdb --username root --password 123456 --table employee --columns id,name,deg,salary,dept --export-dir /emp_data/
```

### 更新导出(updateonly)

-- update-key，更新标识，即根据某个字段进行更新，例如id，可以指定多个更新标识的字段，多个字段之间用逗号分隔。

-- updatemod，指定updateonly（默认模式），仅仅更新已存在的数据记录，不会插入新纪录。

在HDFS文件系统中“/updateonly_1/”目录的下创建一个文件updateonly_1.txt：

```markdown
1201,gopal,manager,50000
1202,manisha,preader,50000
1203,kalil,php dev,30000
```

手动创建mysql中的目标表

```mysql
mysql> USE userdb;
mysql> CREATE TABLE updateonly ( 
   id INT NOT NULL PRIMARY KEY, 
   name VARCHAR(20), 
   deg VARCHAR(20),
   salary INT);
```

先执行全部导出操作

```bash
sqoop export \
--connect jdbc:mysql://server01:3306/userdb \
--username root \
--password 123456 \
--table updateonly \
--export-dir /updateonly_1/
```

新增一个文件updateonly_2.txt：修改了前三条数据并且新增了一条记录

```markdown
1201,gopal,manager,1212
1202,manisha,preader,1313
1203,kalil,php dev,1414
1204,allen,java,1515
```

```bash
[root@server01 opt]# hdfs dfs -mkdir /updateonly_2
[root@server01 opt]# hdfs dfs -put updateonly_2.txt /updateonly_2
```

执行更新导出

```bash
sqoop export \
--connect jdbc:mysql://server01:3306/userdb \
--username root \
--password 123456 \
--table updateonly \
--export-dir /updateonly_2/ \
--update-key id \
--update-mode updateonly
```

### 更新导出(allowinsert)

-- update-key，更新标识，即根据某个字段进行更新，例如id，可以指定多个更新标识的字段，多个字段之间用逗号分隔。

-- updatemod，指定allowinsert，更新已存在的数据记录，同时插入新纪录。实质上是一个insert & update的操作。

创建一个文件allowinsert_1.txt：

```markdown
1201,gopal,manager,50000
1202,manisha,preader,50000
1203,kalil,php dev,30000
```

上传至hdfs：/allowinsert_1/目录下

```bash
[root@server01 opt]# hdfs dfs -mkdir /allowinsert_1
[root@server01 opt]# hdfs dfs -put ./allowinsert_1.txt /allowinsert_1
```

手动创建mysql中的目标表

```sql
mysql> USE userdb;
mysql> CREATE TABLE allowinsert ( 
   id INT NOT NULL PRIMARY KEY, 
   name VARCHAR(20), 
   deg VARCHAR(20),
   salary INT);
```

先执行全部导出操作

```bash
sqoop export \
--connect jdbc:mysql://server01:3306/userdb \
--username root \
--password 123456 \
--table allowinsert \
--export-dir /allowinsert_1/
```

创建allowinsert_2.txt，修改了前三条数据并且新增了一条记录

```markdown
1201,gopal,manager,1212
1202,manisha,preader,1313
1203,kalil,php dev,1414
1204,allen,java,1515
```

上传至hdfs：/allowinsert_2/目录下

```bash
[root@server01 opt]# hdfs dfs -mkdir /allowinsert_2
[root@server01 opt]# hdfs dfs -put ./allowinsert_2.txt /allowinsert_2
```

执行更新导出

```bash
sqoop export \
--connect jdbc:mysql://server01:3306/userdb \
--username root --password 123456 \
--table allowinsert \
--export-dir /allowinsert_2/ \
--update-key id \
--update-mode allowinsert
```

## Sqoop job作业

创建好的job不会立即执行，需要通过执行作业命令才会触发作业的执行，所以Sqoop job作业适合定时任务场景。

### 创建作业(--create)

```bash
sqoop job --create gentleduojob1 -- import --connect jdbc:mysql://server01:3306/userdb \
--username root \
--password 123456 \
--target-dir /sqoopjob1 \
--table emp --m 1
```

注意import前要有空格

### 验证作业 (--list)

```bash
sqoop job --list
```

### 执行作业 (--exec)

```bash
sqoop job --exec itcastjob1
```

### job的免密输入

sqoop在创建job时，如果使用--password将出现警告，并且每次都要手动输入密码才能执行job；

使用--password-file参数，可以避免输入mysql密码，sqoop规定密码文件必须存放在HDFS上，并且权限必须是400。

```bash
[root@server01 opt]# echo -n "123456" > itcastmysql.pwd
[root@server01 opt]# hdfs dfs -mkdir -p /input/sqoop/pwd/
[root@server01 opt]# hdfs dfs -put itcastmysql.pwd /input/sqoop/pwd/
[root@server01 opt]# hdfs dfs -chmod 400 /input/sqoop/pwd/itcastmysql.pwd
```

在sqoop的sqoop-site.xml文件中添加如下配置项：

```xml
<property>
    <name>sqoop.metastore.client.record.password</name>
    <value>true</value>
    <description>If true, allow saved passwords in the metastore.
    </description>
</property>
```

创建sqoop job

```bash
sqoop job --create gentleduojob2 -- import --connect jdbc:mysql://server01:3306/userdb \
--username root \
--password-file /input/sqoop/pwd/itcastmysql.pwd \
--target-dir /sqoopjob2 \
--table emp --m 1
```

执行job

```bash
sqoop job -exec gentleduojob2
```

# Flume

## 概述

Flume是Cloudera提供的一个高可用的，高可靠的，分布式的海量日志采集、聚合和传输的软件。

Flume的核心是把数据从数据源(source)收集过来，再将收集到的数据送到指定的目的地(sink)。为了保证输送的过程一定成功，在送到目的地(sink)之前，会先缓存数据(channel),待数据真正到达目的地(sink)后，flume在删除自己缓存的数据。

Flume支持定制各类数据发送方，用于收集各类型数据；同时，Flume支持定制各种数据接受方，用于最终存储数据。一般的采集需求，通过对flume的简单配置即可实现。针对特殊场景也具备良好的自定义扩展能力。因此，flume可以适用于大部分的日常数据采集场景。

当前Flume有两个版本。Flume 0.9X版本的统称Flume OG（original generation），Flume1.X版本的统称Flume NG（next generation）。由于Flume NG经过核心组件、核心配置以及代码架构重构，与Flume OG有很大不同，使用时请注意区分。改动的另一原因是将Flume纳入 apache 旗下，Cloudera Flume 改名为 Apache Flume。

## 运行机制

Flume系统中核心的角色是agent，agent本身是一个Java进程，一般运行在日志收集节点。每一个agent相当于一个数据传递员，内部有三个组件：

1. Source：采集源，用于跟数据源对接，以获取数据；
2. Sink：下沉地，采集数据的传送目的，用于往下一级agent传递数据或者往最终存储系统传递数据；
3. Channel：agent内部的数据传输通道，用于从source将数据传递到sink；

在整个数据的传输的过程中，流动的是event，它是Flume内部数据传输的最基本单元。event将传输的数据进行封装。如果是文本文件，通常是一行记录，event也是事务的基本单位。event从source，流向channel，再到sink，本身为一个字节数组，并可携带headers(头信息)信息。event代表着一个数据的最小完整单元，从外部数据源来，向外部的目的地去。一个完整的event包括：event headers、event body、event信息，其中event信息就是flume收集到的日记记录。

## 安装部署

上传安装包到数据源所在节点上，然后解压，最后进入flume的目录，修改conf下的flume-env.sh，在里面配置JAVA_HOME，添加环境变量：

```properties
export FLUME_HOME=/usr/local/flume
export PATH=$PATH:$FLUME_HOME/bin
```

先用一个最简单的例子来测试一下程序环境是否正常

先在flume的conf目录下新建一个文件：netcat-logger.conf

```properties
#从网络端口接收数据，下沉到logger
# Name the components on this agent
a1.sources = r1
a1.sinks = k1
a1.channels = c1

# Describe/configure the source
a1.sources.r1.type = netcat
a1.sources.r1.bind = localhost
a1.sources.r1.port = 44444

# Describe the sink
a1.sinks.k1.type = logger

# Use a channel which buffers events in memory
a1.channels.c1.type = memory
a1.channels.c1.capacity = 1000
a1.channels.c1.transactionCapacity = 100

# Bind the source and sink to the channel
a1.sources.r1.channels = c1
a1.sinks.k1.channel = c1
```

启动agent去采集数据

```bash
[root@server01 conf]# flume-ng agent --conf /usr/local/flume/conf --conf-file /usr/local/flume/conf/netcat-logger.conf --name a1 -Dflume.root.logger=INFO,console
Info: Sourcing environment configuration script /usr/local/flume/conf/flume-env.sh
```

- --conf指定配置文件的目录
- --conf-file指定采集方案路径
- --name  agent进程名字 要跟采集方案中保持一致

测试

```bash
[root@server01 ~]# telnet localhost 44444
```

## 简单案例

### 采集目录到HDFS

采集需求：服务器的某特定目录下，会不断产生新的文件，每当有新文件出现，就需要把文件采集到HDFS中去根据需求，首先定义以下3大要素

1. 采集源，即source——监控文件目录 :  spooldir
2. 下沉目标，即sink——HDFS文件系统  :  hdfs sink
3. source和sink之间的传递通道——channel，可用file channel 也可以用内存channel

配置文件编写

```properties
# Name the components on this agent
a1.sources = r1
a1.sinks = k1
a1.channels = c1

# Describe/configure the source
##注意：不能往监控目中重复丢同名文件
a1.sources.r1.type = spooldir
a1.sources.r1.spoolDir = /opt/logs
a1.sources.r1.fileHeader = true

# Describe the sink
a1.sinks.k1.type = hdfs
a1.sinks.k1.channel = c1
# 控制hdfs中的文件夹以多少时间间隔滚动，以下述为例：就会每10分钟生成一个文件夹
# 当时间为2015-10-16 17:38:59时候，hdfs.path依然会被解析为：/flume/events/20151016/17:30/00
# 因为设置的是舍弃10分钟内的时间，因此，该目录每10分钟新生成一个。
a1.sinks.k1.hdfs.path = /flume/events/%y-%m-%d/%H%M/
a1.sinks.k1.hdfs.round = true
a1.sinks.k1.hdfs.roundValue = 10
a1.sinks.k1.hdfs.roundUnit = minute
a1.sinks.k1.hdfs.filePrefix = events-
# roll控制写入hdfs的文件以何种方式进行滚动
# 如果三个都配置，谁先满足谁触发滚动，如果不想以某种属性滚动，设置为0即可
# sink间隔多长将临时文件滚动成最终目标文件，单位：秒；如果设置成0，则表示不根据时间来滚动文件；默认值：30。注：滚动（roll）指的是，hdfs sink将临时文件重命名成最终目标文件，并新打开一个临时文件来写入数据；
a1.sinks.k1.hdfs.rollInterval = 3 #以时间间隔
# 当临时文件达到该大小（单位：bytes）时，滚动成目标文件；如果设置成0，则表示不根据临时文件大小来滚动文件；默认值：1024。
a1.sinks.k1.hdfs.rollSize = 20 #以文件大小
# 当events数据达到该数量时候，将临时文件滚动成目标文件；当events数据达到该数量时候，将临时文件滚动成目标文件；默认值：10。
a1.sinks.k1.hdfs.rollCount = 5 #以event个数
a1.sinks.k1.hdfs.batchSize = 1
a1.sinks.k1.hdfs.useLocalTimeStamp = true
#生成的文件类型，默认是Sequencefile，可用DataStream，则为普通文本
a1.sinks.k1.hdfs.fileType = DataStream

# Use a channel which buffers events in memory
a1.channels.c1.type = memory
a1.channels.c1.capacity = 1000
a1.channels.c1.transactionCapacity = 100

# Bind the source and sink to the channel
a1.sources.r1.channels = c1
a1.sinks.k1.channel = c1
```

Channel参数解释：

1. capacity：默认该通道中最大的可以存储的event数量
2. trasactionCapacity：每次最大可以从source中拿到或者送到sink中的event数量

注意其监控的文件夹下面不能有同名文件的产生，如果有，报错且罢工，后续就不再进行数据的监视采集了

启动命令

```bash
flume-ng agent -c /usr/local/flume/conf -f /usr/local/flume/conf/spool-hdfs.conf -n a1 -Dflume.root.logger=INFO,console
```

### 采集文件到HDFS

采集需求：比如业务系统使用log4j生成的日志，日志内容不断增加，需要把追加到日志文件中的数据实时采集到hdfs
根据需求，首先定义以下3大要素

1. 采集源，即source——监控文件内容更新 :  exec  ‘tail -F file’
2. 下沉目标，即sink——HDFS文件系统  :  hdfs sink
3. Source和sink之间的传递通道——channel，可用file channel 也可以用 内存channel

配置文件编写

```properties
# Name the components on this agent
a1.sources = r1
a1.sinks = k1
a1.channels = c1

# Describe/configure the source
a1.sources.r1.type = exec
a1.sources.r1.command = tail -F /opt/logs/test.log
a1.sources.r1.channels = c1

# Describe the sink
a1.sinks.k1.type = hdfs
a1.sinks.k1.channel = c1
a1.sinks.k1.hdfs.path = /flume/tailout/%y-%m-%d/%H-%M/
a1.sinks.k1.hdfs.filePrefix = events-
a1.sinks.k1.hdfs.round = true
a1.sinks.k1.hdfs.roundValue = 10
a1.sinks.k1.hdfs.roundUnit = minute
a1.sinks.k1.hdfs.rollInterval = 0
a1.sinks.k1.hdfs.rollSize = 10485760
a1.sinks.k1.hdfs.rollCount = 0
a1.sinks.k1.hdfs.batchSize = 1
a1.sinks.k1.hdfs.useLocalTimeStamp = true
#生成的文件类型，默认是Sequencefile，可用DataStream，则为普通文本
a1.sinks.k1.hdfs.fileType = DataStream

# Use a channel which buffers events in memory
a1.channels.c1.type = memory
a1.channels.c1.capacity = 1000
a1.channels.c1.transactionCapacity = 100

# Bind the source and sink to the channel
a1.sources.r1.channels = c1
a1.sinks.k1.channel = c1
```

启动命令

```bash
[root@server01 conf]# flume-ng agent -c /usr/local/flume/conf -f /usr/local/flume/conf/tail-hdfs.conf -n a1 -Dflume.root.logger=INFO,console
```

模拟数据实时产生

```bash
[root@server01 logs]# while true; do date >> /opt/logs/test.log;sleep 1;done
```

### Taildir Source

在flume1.7版本之后，提供了一个非常好用的TaildirSource，使用这个source，可以监控一个目录，并且使用正则表达式匹配该目录中的文件名进行实时收集。

配置文件编写：taildir_source_avro_sink.conf

```properties
# Name the components on this agent
a1.sources = r1
a1.sinks = k1
a1.channels = c1

# Describe/configure the source
a1.sources.r1.type = TAILDIR
a1.sources.r1.channels = c1
a1.sources.r1.positionFile = /opt/logs/taildir_position.json
a1.sources.r1.filegroups = f1 f2
a1.sources.r1.filegroups.f1 = /opt/logs/example.log
a1.sources.r1.filegroups.f2 = /opt/logs/taildir/.*log.*

# Describe the sink
a1.sinks.k1.type = hdfs
a1.sinks.k1.channel = c1
a1.sinks.k1.hdfs.path = /flume/taildir/%y-%m-%d/%H-%M/
a1.sinks.k1.hdfs.filePrefix = weblog-
a1.sinks.k1.hdfs.round = true
a1.sinks.k1.hdfs.roundValue = 10
a1.sinks.k1.hdfs.roundUnit = minute
a1.sinks.k1.hdfs.rollInterval = 0
# 128M
a1.sinks.k1.hdfs.rollSize = 134217728
a1.sinks.k1.hdfs.rollCount = 0
# HDFS sink文件滚动属性：基于文件闲置时间策略
# 如果文件在hdfs.idleTimeout秒的时间里都是闲置的，即：没有任何数据写入，那么当前文件关闭，滚动到下一个文件
a1.sinks.k1.hdfs.idleTimeout = 20
# HDFS sink文件滚动属性：基于hdfs文件副本数
# 默认值：和hdfs的副本数一致
# hdfs.minBlockReplicas是为了让flume感知不到hdfs的块复制，这样滚动方式配置（比如时间间隔、文件大小、events数量等）才不会受影响。
# 假如hdfs的副本为3.那么配置的滚动时间为10秒，那么在第二秒的时候，flume检测到hdfs在复制块，那么这时候flume就会滚动，这样导致flume的滚动方式受到影响。所以通常hdfs.minBlockReplicas配置为1，就检测不到副本的复制了。但是hdfs的副本还是3
a1.sinks.k1.hdfs.minBlockReplicas = 1
a1.sinks.k1.hdfs.batchSize = 1
a1.sinks.k1.hdfs.useLocalTimeStamp = true
#生成的文件类型，默认是Sequencefile，可用DataStream，则为普通文本
a1.sinks.k1.hdfs.fileType = DataStream

# Use a channel which buffers events in memory
a1.channels.c1.type = memory
a1.channels.c1.capacity = 1000
a1.channels.c1.transactionCapacity = 100

# Bind the source and sink to the channel
a1.sources.r1.channels = c1
a1.sinks.k1.channel = c1
```

启动命令

1. filegroups:指定filegroups，可以有多个，以空格分隔；（TailSource可以同时监控tail多个目录中的文件）
2. positionFile:配置检查点文件的路径，检查点文件会以json格式保存tail命令已经执行到的的位置，当程序异常终止并重新启动后，会从positionFile文件中记录的上次采集到的位置之后开始采集，解决了断点不能续传的缺陷。
3. filegroups.filegroupName：配置每个filegroup的文件绝对路径，文件名可以用正则表达式匹配

````bash
[root@server01 flume]# flume-ng agent -c /usr/local/flume/conf -f /usr/local/flume/conf/taildir_source_avro_sink.conf -n a1 -Dflume.root.logger=INFO,console
````

## load-balance

负载均衡是用于解决一台机器(一个进程)无法解决所有请求而产生的一种算法。Load balancing Sink Processor能够实现load balance功能，如下面的示例：Agent1是一个路由节点，负责将Channel暂存的Event均衡到对应的多个Sink组件上，而每个Sink组件分别连接到一个独立的Agent上

flume的负载均衡

1. 所谓的负载均衡 用于解决一个进程或者程序处理不了所有请求 多个进程一起处理的场景
2. 同一个请求只能交给一个进行处理 避免数据重复
3. 如何分配请求就涉及到了负载均衡的算法：轮询（round_robin）  随机（random）  权重

flume串联跨网络传输数据

1. avro sink  

2. avro source

3. 使用上述两个组件指定绑定的端口ip 就可以满足数据跨网络的传递 通常用于flume串联架构中

exec-avro.conf

```properties
#agent1 name
agent1.channels = c1
agent1.sources = r1
agent1.sinks = k1 k2

#set gruop
agent1.sinkgroups = g1

#set channel
agent1.channels.c1.type = memory
agent1.channels.c1.capacity = 1000
agent1.channels.c1.transactionCapacity = 100

agent1.sources.r1.channels = c1
agent1.sources.r1.type = exec
agent1.sources.r1.command = tail -F /opt/logs/123.log

# set sink1
agent1.sinks.k1.channel = c1
agent1.sinks.k1.type = avro
agent1.sinks.k1.hostname = server02
agent1.sinks.k1.port = 52020

# set sink2
agent1.sinks.k2.channel = c1
agent1.sinks.k2.type = avro
agent1.sinks.k2.hostname = server03
agent1.sinks.k2.port = 52020

#set sink group
agent1.sinkgroups.g1.sinks = k1 k2

#set failover
agent1.sinkgroups.g1.processor.type = load_balance
agent1.sinkgroups.g1.processor.backoff = true
agent1.sinkgroups.g1.processor.selector = round_robin
agent1.sinkgroups.g1.processor.selector.maxTimeOut=10000

```

avro-logger.conf(server02)

```properties
# Name the components on this agent
a1.sources = r1
a1.sinks = k1
a1.channels = c1

# Describe/configure the source
a1.sources.r1.type = avro
a1.sources.r1.channels = c1
a1.sources.r1.bind = server02
a1.sources.r1.port = 52020

# Describe the sink
a1.sinks.k1.type = logger

# Use a channel which buffers events in memory
a1.channels.c1.type = memory
a1.channels.c1.capacity = 1000
a1.channels.c1.transactionCapacity = 100

# Bind the source and sink to the channel
a1.sources.r1.channels = c1
a1.sinks.k1.channel = c1
```

avro-logger.conf(server03)

```properties
# Name the components on this agent
a1.sources = r1
a1.sinks = k1
a1.channels = c1

# Describe/configure the source
a1.sources.r1.type = avro
a1.sources.r1.channels = c1
a1.sources.r1.bind = server03
a1.sources.r1.port = 52020

# Describe the sink
a1.sinks.k1.type = logger

# Use a channel which buffers events in memory
a1.channels.c1.type = memory
a1.channels.c1.capacity = 1000
a1.channels.c1.transactionCapacity = 100

# Bind the source and sink to the channel
a1.sources.r1.channels = c1
a1.sinks.k1.channel = c1
```

启动server02上的agent

```bash
[root@server02 conf]# flume-ng agent -c /usr/local/flume/conf -f /usr/local/flume/conf/avro-logger.conf -n a1 -Dflume.root.logger=INFO,console
```

启动server03上的agent

```bash
[root@server03 conf]# flume-ng agent -c /usr/local/flume/conf -f /usr/local/flume/conf/avro-logger.conf -n a1 -Dflume.root.logger=INFO,console
```

启动agent1

```bash
[root@server01 conf]# flume-ng agent -c /usr/local/flume/conf -f /usr/local/flume/conf/exec-avro.conf -n agent1 -Dflume.root.logger=INFO,console
```

模拟实时数据

```bash
[root@server01 logs]# while true; do date >> /opt/logs/123.log;sleep 1;done
```

## failover

Failover Sink Processor能够实现failover功能，具体流程类似load balance，但是内部处理机制与load balance完全不同。
Failover Sink Processor维护一个优先级Sink组件列表，只要有一个Sink组件可用，Event就被传递到下一个组件。故障转移机制的作用是将失败的Sink降级到一个池，在这些池中它们被分配一个冷却时间，随着故障的连续，在重试之前冷却时间增加。一旦Sink成功发送一个事件，它将恢复到活动池。 Sink具有与之相关的优先级，数量越大，优先级越高。

flume failover

1. 容错又称之为故障转移  容忍错误的发生。
2. 通常用于解决单点故障 给容易出故障的地方设置备份
3. 备份越多 容错能力越强  但是资源的浪费越严重

exec-avro.conf

```properties
#agent1 name
agent1.channels = c1
agent1.sources = r1
agent1.sinks = k1 k2

#set gruop
agent1.sinkgroups = g1

#set channel
agent1.channels.c1.type = memory
agent1.channels.c1.capacity = 1000
agent1.channels.c1.transactionCapacity = 100

agent1.sources.r1.channels = c1
agent1.sources.r1.type = exec
agent1.sources.r1.command = tail -F /opt/logs/456.log

# set sink1
agent1.sinks.k1.channel = c1
agent1.sinks.k1.type = avro
agent1.sinks.k1.hostname = server02
agent1.sinks.k1.port = 52020

# set sink2
agent1.sinks.k2.channel = c1
agent1.sinks.k2.type = avro
agent1.sinks.k2.hostname = server03
agent1.sinks.k2.port = 52020

#set sink group
agent1.sinkgroups.g1.sinks = k1 k2

#set failover
agent1.sinkgroups.g1.processor.type = failover
agent1.sinkgroups.g1.processor.priority.k1 = 10
agent1.sinkgroups.g1.processor.priority.k2 = 1
agent1.sinkgroups.g1.processor.maxpenalty = 10000
```

avro-logger.conf(server02)

```properties
# Name the components on this agent
a1.sources = r1
a1.sinks = k1
a1.channels = c1

# Describe/configure the source
a1.sources.r1.type = avro
a1.sources.r1.channels = c1
a1.sources.r1.bind = server02
a1.sources.r1.port = 52020

# Describe the sink
a1.sinks.k1.type = logger

# Use a channel which buffers events in memory
a1.channels.c1.type = memory
a1.channels.c1.capacity = 1000
a1.channels.c1.transactionCapacity = 100

# Bind the source and sink to the channel
a1.sources.r1.channels = c1
a1.sinks.k1.channel = c1
```

avro-logger.conf(server03)

```properties
# Name the components on this agent
a1.sources = r1
a1.sinks = k1
a1.channels = c1

# Describe/configure the source
a1.sources.r1.type = avro
a1.sources.r1.channels = c1
a1.sources.r1.bind = server03
a1.sources.r1.port = 52020

# Describe the sink
a1.sinks.k1.type = logger

# Use a channel which buffers events in memory
a1.channels.c1.type = memory
a1.channels.c1.capacity = 1000
a1.channels.c1.transactionCapacity = 100

# Bind the source and sink to the channel
a1.sources.r1.channels = c1
a1.sinks.k1.channel = c1
```

启动server02上的agent

```bash
[root@server02 conf]# flume-ng agent -c /usr/local/flume/conf -f /usr/local/flume/conf/avro-logger.conf -n a1 -Dflume.root.logger=INFO,console
```

启动server03上的agent

```ba
[root@server03 conf]# flume-ng agent -c /usr/local/flume/conf -f /usr/local/flume/conf/avro-logger.conf -n a1 -Dflume.root.logger=INFO,console
```

启动agent1

```bash
[root@server01 conf]# flume-ng agent -c /usr/local/flume/conf -f /usr/local/flume/conf/exec-avro.conf -n agent1 -Dflume.root.logger=INFO,console
```

模拟实时数据

```bash
[root@server01 logs]# while true; do date >> /opt/logs/456.log;sleep 1;done
```

## Flume拦截器

### 日志的采集和汇总

#### 案例场景

A、B两台日志服务机器实时生产日志主要类型为access.log、nginx.log、web.log 

#### 现在要求

把A、B 机器中的access.log、nginx.log、web.log 采集汇总到C机器上然后统一收集到hdfs中。
但是在hdfs中要求的目录为：

- /source/logs/access/20160101/**
- /source/logs/nginx/20160101/**
- /source/logs/web/20160101/**

#### 静态拦截器

```markdown
如果没有使用静态拦截器
Event: { headers:{} body:  36 Sun Jun  2 18:26 }

使用静态拦截器之后 自己添加kv标识对
Event: { headers:{type=access} body:  36 Sun Jun  2 18:26 }
Event: { headers:{type=nginx} body:  36 Sun Jun  2 18:26 }
Event: { headers:{type=web} body:  36 Sun Jun  2 18:26 }
```

后续在存放数据的时候可以使用flume的规则语法获取到拦截器添加的kv内容

```markdown
%{type}
```

exec_source_avro_sink.conf

```properties
# Name the components on this agent
a1.sources = r1 r2 r3
a1.sinks = k1
a1.channels = c1

# Describe/configure the source
a1.sources.r1.type = exec
a1.sources.r1.command = tail -F /opt/logs/access.log
a1.sources.r1.interceptors = i1
# static 拦截器的功能就是往采集到的数据的header中插入自己定义的key-value对
a1.sources.r1.interceptors.i1.type = static
a1.sources.r1.interceptors.i1.key = type
a1.sources.r1.interceptors.i1.value = access

a1.sources.r2.type = exec
a1.sources.r2.command = tail -F /opt/logs/nginx.log
a1.sources.r2.interceptors = i2
a1.sources.r2.interceptors.i2.type = static
a1.sources.r2.interceptors.i2.key = type
a1.sources.r2.interceptors.i2.value = nginx

a1.sources.r3.type = exec
a1.sources.r3.command = tail -F /opt/logs/web.log
a1.sources.r3.interceptors = i3
a1.sources.r3.interceptors.i3.type = static
a1.sources.r3.interceptors.i3.key = type
a1.sources.r3.interceptors.i3.value = web

# Describe the sink
a1.sinks.k1.type = avro
a1.sinks.k1.hostname = server02
a1.sinks.k1.port = 41414

# Use a channel which buffers events in memory
a1.channels.c1.type = memory
a1.channels.c1.capacity = 200000
a1.channels.c1.transactionCapacity = 2000

# Bind the source and sink to the channel
a1.sources.r1.channels = c1
a1.sources.r2.channels = c1
a1.sources.r3.channels = c1
a1.sinks.k1.channel = c1
```

avro_source_hdfs_sink.conf

```properties
#定义agent名， source、channel、sink的名称
a1.sources = r1
a1.sinks = k1
a1.channels = c1

#定义source
a1.sources.r1.type = avro
a1.sources.r1.bind = server02
a1.sources.r1.port =41414

#添加时间拦截器
a1.sources.r1.interceptors = i1
a1.sources.r1.interceptors.i1.type = org.apache.flume.interceptor.TimestampInterceptor$Builder


#定义channels
a1.channels.c1.type = memory
a1.channels.c1.capacity = 200000
a1.channels.c1.transactionCapacity = 2000

#定义sink
a1.sinks.k1.type = hdfs
a1.sinks.k1.hdfs.path=hdfs://server01:8020/source/logs/%{type}/%Y%m%d
a1.sinks.k1.hdfs.filePrefix =events
a1.sinks.k1.hdfs.fileType = DataStream
a1.sinks.k1.hdfs.writeFormat = Text
#时间类型
#a1.sinks.k1.hdfs.useLocalTimeStamp = true
#生成的文件不按条数生成
a1.sinks.k1.hdfs.rollCount = 0
#生成的文件不按时间生成
a1.sinks.k1.hdfs.rollInterval = 0
#生成的文件按大小生成
a1.sinks.k1.hdfs.rollSize  = 10485760
#批量写入hdfs的个数
a1.sinks.k1.hdfs.batchSize = 2000
flume操作hdfs的线程数（包括新建，写入等）
a1.sinks.k1.hdfs.threadsPoolSize=10
#操作hdfs超时时间
a1.sinks.k1.hdfs.callTimeout=30000

#组装source、channel、sink
a1.sources.r1.channels = c1
a1.sinks.k1.channel = c1
```

启动server02上的agent(avro_source_hdfs_sink.conf)

```bash
[root@server02 conf]# flume-ng agent -c /usr/local/flume/conf -f /usr/local/flume/conf/avro_source_hdfs_sink.conf -n a1 -Dflume.root.logger=INFO,console
```

启动server01上的agent(exec_source_avro_sink.conf)

```bash
[root@server02 conf]# flume-ng agent -c /usr/local/flume/conf -f /usr/local/flume/conf/exec_source_avro_sink.conf -n a1 -Dflume.root.logger=INFO,console
```

模拟实时数据

```bash
[root@server01 logs]# while true; do echo "access access....." >> /opt/logs/access.log;sleep 0.5;done
```

```bash
[root@server01 logs]# while true; do echo "web web....." >> /opt/logs/web.log;sleep 0.5;done
```

```bash
[root@server01 logs]# while true; do echo "nginx nginx....." >> /opt/logs/nginx.log;sleep 0.5;done
```

### 自定义拦截器

Flume是Cloudera提供的一个高可用的，高可靠的，分布式的海量日志采集、聚合和传输的系统，Flume支持在日志系统中定制各类数据发送方，用于收集数据；同时，Flume提供对数据进行简单处理，并写到各种数据接受方（可定制）的能力。Flume有各种自带的拦截器，比如：TimestampInterceptor、HostInterceptor、RegexExtractorInterceptor等，通过使用不同的拦截器，实现不同的功能。但是以上的这些拦截器，不能改变原有日志数据的内容或者对日志信息添加一定的处理逻辑，当一条日志信息有几十个甚至上百个字段的时候，在传统的Flume处理下，收集到的日志还是会有对应这么多的字段，也不能对你想要的字段进行对应的处理。

根据实际业务的需求，为了更好的满足数据在应用层的处理，通过自定义Flume拦截器，过滤掉不需要的字段，并对指定字段加密处理，将源数据进行预处理。减少了数据的传输量，降低了存储的开销。

编写 java 代码，自定义拦截器，内容包括：

1. 定义一个类CustomParameterInterceptor实现Interceptor接口。
2. 在CustomParameterInterceptor类中定义变量，这些变量是需要到 Flume的配置文件中进行配置使用的。每一行字段间的分隔符(fields_separator)、通过分隔符分隔后，所需要列字段的下标（indexs）、多个下标使用的分隔符（indexs_separator)、多个下标使用的分隔符（indexs_separator)。
3. 添加CustomParameterInterceptor的有参构造方法。并对相应的变量进行处理。将配置文件中传过来的unicode编码进行转换为字符串。
4. 写具体的要处理的逻辑intercept()方法，一个是单个处理的，一个是批量处理。
5. 接口中定义了一个内部接口Builder，在configure方法中，进行一些参数配置。并给出，在flume的conf中没配置一些参数时，给出其默认值。通过其builder方法，返回一个CustomParameterInterceptor对象。
6. 定义一个静态类，类中封装MD5加密方法
7. 通过以上步骤，自定义拦截器的代码开发已完成，然后打包成jar， 放到Flume的根目录下的lib中

CustomParameterInterceptor.java

```java
package org.duo.interceptor;

import com.google.common.base.Charsets;
import org.apache.flume.Context;
import org.apache.flume.Event;
import org.apache.flume.interceptor.Interceptor;

import java.security.MessageDigest;
import java.security.NoSuchAlgorithmException;
import java.util.ArrayList;
import java.util.List;
import java.util.regex.Matcher;
import java.util.regex.Pattern;

import static org.duo.interceptor.CustomParameterInterceptor.Constants.*;

/**
 * Created by gentleduo
 */
public class CustomParameterInterceptor implements Interceptor {


    /**
     * The field_separator.指明每一行字段的分隔符
     */
    private final String fields_separator;

    /**
     * The indexs.通过分隔符分割后，指明需要那列的字段 下标
     */
    private final String indexs;

    /**
     * The indexs_separator. 多个下标的分隔符
     */
    private final String indexs_separator;

    /**
     * The encrypted_field_index. 需要加密的字段下标
     */
    private final String encrypted_field_index;


    /**
     *
     */
    public CustomParameterInterceptor(String fields_separator,
                                      String indexs, String indexs_separator, String encrypted_field_index) {
        String f = fields_separator.trim();
        String i = indexs_separator.trim();
        this.indexs = indexs;
        this.encrypted_field_index = encrypted_field_index.trim();
        if (!f.equals("")) {
            f = UnicodeToString(f);
        }
        this.fields_separator = f;
        if (!i.equals("")) {
            i = UnicodeToString(i);
        }
        this.indexs_separator = i;
    }

    /*
     *
     * \t 制表符 ('\u0009')
     *
     */

    public static String UnicodeToString(String str) {
        Pattern pattern = Pattern.compile("(\\\\u(\\p{XDigit}{4}))");
        Matcher matcher = pattern.matcher(str);
        char ch;
        while (matcher.find()) {
            ch = (char) Integer.parseInt(matcher.group(2), 16);
            str = str.replace(matcher.group(1), ch + "");
        }
        return str;
    }

    /*
     * @see org.apache.flume.interceptor.Interceptor#intercept(org.apache.flume.Event)
     */
    public Event intercept(Event event) {
        if (event == null) {
            return null;
        }
        try {
            String line = new String(event.getBody(), Charsets.UTF_8);
            String[] fields_spilts = line.split(fields_separator);
            String[] indexs_split = indexs.split(indexs_separator);
            String newLine = "";
            for (int i = 0; i < indexs_split.length; i++) {
                int parseInt = Integer.parseInt(indexs_split[i]);
                //对加密字段进行加密
                if (!"".equals(encrypted_field_index) && encrypted_field_index.equals(indexs_split[i])) {
                    newLine += StringUtils.GetMD5Code(fields_spilts[parseInt]);
                } else {
                    newLine += fields_spilts[parseInt];
                }

                if (i != indexs_split.length - 1) {
                    newLine += fields_separator;
                }
            }
            event.setBody(newLine.getBytes(Charsets.UTF_8));
            return event;
        } catch (Exception e) {
            return event;
        }
    }

    /*
     * @see org.apache.flume.interceptor.Interceptor#intercept(java.util.List)
     */
    public List<Event> intercept(List<Event> events) {
        List<Event> out = new ArrayList<Event>();
        for (Event event : events) {
            Event outEvent = intercept(event);
            if (outEvent != null) {
                out.add(outEvent);
            }
        }
        return out;
    }

    /*
     * @see org.apache.flume.interceptor.Interceptor#initialize()
     */
    public void initialize() {
        // TODO Auto-generated method stub

    }

    /*
     * @see org.apache.flume.interceptor.Interceptor#close()
     */
    public void close() {
        // TODO Auto-generated method stub

    }


    public static class Builder implements Interceptor.Builder {

        /**
         * The fields_separator.指明每一行字段的分隔符
         */
        private String fields_separator;

        /**
         * The indexs.通过分隔符分割后，指明需要那列的字段 下标
         */
        private String indexs;

        /**
         * The indexs_separator. 多个下标下标的分隔符
         */
        private String indexs_separator;

        /**
         * The encrypted_field. 需要加密的字段下标
         */
        private String encrypted_field_index;

        /*
         * @see org.apache.flume.conf.Configurable#configure(org.apache.flume.Context)
         */
        public void configure(Context context) {
            fields_separator = context.getString(FIELD_SEPARATOR, DEFAULT_FIELD_SEPARATOR);
            indexs = context.getString(INDEXS, DEFAULT_INDEXS);
            indexs_separator = context.getString(INDEXS_SEPARATOR, DEFAULT_INDEXS_SEPARATOR);
            encrypted_field_index = context.getString(ENCRYPTED_FIELD_INDEX, DEFAULT_ENCRYPTED_FIELD_INDEX);

        }

        /*
         * @see org.apache.flume.interceptor.Interceptor.Builder#build()
         */
        public Interceptor build() {

            return new CustomParameterInterceptor(fields_separator, indexs, indexs_separator, encrypted_field_index);
        }
    }


    /**
     * The Class Constants.
     */
    public static class Constants {
        /**
         * The Constant FIELD_SEPARATOR.
         */
        public static final String FIELD_SEPARATOR = "fields_separator";

        /**
         * The Constant DEFAULT_FIELD_SEPARATOR.
         */
        public static final String DEFAULT_FIELD_SEPARATOR = " ";

        /**
         * The Constant INDEXS.
         */
        public static final String INDEXS = "indexs";

        /**
         * The Constant DEFAULT_INDEXS.
         */
        public static final String DEFAULT_INDEXS = "0";

        /**
         * The Constant INDEXS_SEPARATOR.
         */
        public static final String INDEXS_SEPARATOR = "indexs_separator";

        /**
         * The Constant DEFAULT_INDEXS_SEPARATOR.
         */
        public static final String DEFAULT_INDEXS_SEPARATOR = ",";

        /**
         * The Constant ENCRYPTED_FIELD_INDEX.
         */
        public static final String ENCRYPTED_FIELD_INDEX = "encrypted_field_index";

        /**
         * The Constant DEFAUL_TENCRYPTED_FIELD_INDEX.
         */
        public static final String DEFAULT_ENCRYPTED_FIELD_INDEX = "";

        /**
         * The Constant PROCESSTIME.
         */
        public static final String PROCESSTIME = "processTime";
        /**
         * The Constant PROCESSTIME.
         */
        public static final String DEFAULT_PROCESSTIME = "a";

    }


    /**
     * 字符串md5加密
     */
    public static class StringUtils {
        // 全局数组
        private final static String[] strDigits = {"0", "1", "2", "3", "4", "5",
                "6", "7", "8", "9", "a", "b", "c", "d", "e", "f"};

        // 返回形式为数字跟字符串
        private static String byteToArrayString(byte bByte) {
            int iRet = bByte;
            // System.out.println("iRet="+iRet);
            if (iRet < 0) {
                iRet += 256;
            }
            int iD1 = iRet / 16;
            int iD2 = iRet % 16;
            return strDigits[iD1] + strDigits[iD2];
        }

        // 返回形式只为数字
        private static String byteToNum(byte bByte) {
            int iRet = bByte;
            System.out.println("iRet1=" + iRet);
            if (iRet < 0) {
                iRet += 256;
            }
            return String.valueOf(iRet);
        }

        // 转换字节数组为16进制字串
        private static String byteToString(byte[] bByte) {
            StringBuffer sBuffer = new StringBuffer();
            for (int i = 0; i < bByte.length; i++) {
                sBuffer.append(byteToArrayString(bByte[i]));
            }
            return sBuffer.toString();
        }

        public static String GetMD5Code(String strObj) {
            String resultString = null;
            try {
                resultString = new String(strObj);
                MessageDigest md = MessageDigest.getInstance("MD5");
                // md.digest() 该函数返回值为存放哈希值结果的byte数组
                resultString = byteToString(md.digest(strObj.getBytes()));
            } catch (NoSuchAlgorithmException ex) {
                ex.printStackTrace();
            }
            return resultString;
        }
    }
}
```

数据：/opt/logs_intercept/testdata

```properties
13601249301     100     200     300     400     500     600     700
13601249302     100     200     300     400     500     600     700
13601249303     100     200     300     400     500     600     700
13601249304     100     200     300     400     500     600     700
13601249305     100     200     300     400     500     600     700
13601249306     100     200     300     400     500     600     700
13601249307     100     200     300     400     500     600     700
13601249308     100     200     300     400     500     600     700
13601249309     100     200     300     400     500     600     700
13601249310     100     200     300     400     500     600     700
13601249311     100     200     300     400     500     600     700
13601249312     100     200     300     400     500     600     700
13601249313     100     200     300     400     500     600     700
```

spool-interceptor-hdfs.conf

```properties
a1.channels = c1
a1.sources = r1
a1.sinks = s1

#channel
a1.channels.c1.type = memory
a1.channels.c1.capacity=1000
a1.channels.c1.transactionCapacity=200

#source
a1.sources.r1.channels = c1
a1.sources.r1.type = spooldir
a1.sources.r1.spoolDir = /opt/logs_intercept/
a1.sources.r1.batchSize= 50
a1.sources.r1.inputCharset = UTF-8

a1.sources.r1.interceptors =i1 i2
a1.sources.r1.interceptors.i1.type =org.duo.interceptor.CustomParameterInterceptor$Builder
a1.sources.r1.interceptors.i1.fields_separator=\\u0009
a1.sources.r1.interceptors.i1.indexs =0,1,3,5,6
a1.sources.r1.interceptors.i1.indexs_separator =\\u002c
a1.sources.r1.interceptors.i1.encrypted_field_index =0

a1.sources.r1.interceptors.i2.type = org.apache.flume.interceptor.TimestampInterceptor$Builder


#sink
a1.sinks.s1.channel = c1
a1.sinks.s1.type = hdfs
a1.sinks.s1.hdfs.path =hdfs://server01:8020/intercept/%Y%m%d
a1.sinks.s1.hdfs.filePrefix = itcasr
a1.sinks.s1.hdfs.fileSuffix = .dat
a1.sinks.s1.hdfs.rollSize = 10485760
a1.sinks.s1.hdfs.rollInterval =20
a1.sinks.s1.hdfs.rollCount = 0
a1.sinks.s1.hdfs.batchSize = 2
a1.sinks.s1.hdfs.round = true
a1.sinks.s1.hdfs.roundUnit = minute
a1.sinks.s1.hdfs.threadsPoolSize = 25
a1.sinks.s1.hdfs.useLocalTimeStamp = true
a1.sinks.s1.hdfs.minBlockReplicas = 1
a1.sinks.s1.hdfs.fileType =DataStream
a1.sinks.s1.hdfs.writeFormat = Text
a1.sinks.s1.hdfs.callTimeout = 60000
a1.sinks.s1.hdfs.idleTimeout =60


############################################
bin/flume-ng agent -c conf -f conf/spool-interceptor-hdfs.conf -n a1 -Dflume.root.logger=INFO,console
```

启动agent

```bash
[root@server01 conf]# flume-ng agent -c /usr/local/flume/conf -f /usr/local/flume/conf/spool-interceptor-hdfs.conf -n a1 -Dflume.root.logger=INFO,console
```

## Flume自定义Source

Source是负责接收数据到FlumeAgent的组件。Source组件可以处理各种类型、各种格式的日志数据，包括avro、thrift、exec、jms、spoolingdirectory、netcat、sequencegenerator、syslog、http、legacy。官方提供的source类型已经很多，但是有时候并不能满足实际开发当中的需求，此时我们就需要根据实际需求自定义某些source。如：实时监控MySQL，从MySQL中获取数据传输到HDFS或者其他存储框架，所以此时需要我们自己实现MySQLSource。官方也提供了自定义 source 的接口：官网说明：https://flume.apache.org/FlumeDeveloperGuide.html#source

根据官方说明自定义 mysqlsource 需要继承 AbstractSource 类并实现Configurable 和 PollableSource 接口。
实现相应方法：

1. getBackOffSleepIncrement() //暂不用
2. getMaxBackOffSleepInterval() //暂不用
3. configure(Context context) //初始化 context
4. process() //获取数据（从 mysql 获取数据，业务处理比较复杂，所以我们定义一个专门的类——QueryMysql 来处理跟 mysql 的交互），封装成 event 并写入 channel，这个方法被循环调用
5. stop() //关闭相关的资源

创建mysql数据库以及mysql数据库表

```sql
CREATE DATABASE `mysqlsource`;

USE `mysqlsource`;

/*Table structure for table `flume_meta` */
DROP TABLE
IF EXISTS `flume_meta`;

CREATE TABLE `flume_meta` (
	`source_tab` VARCHAR (255) NOT NULL,
	`currentIndex` VARCHAR (255) NOT NULL,
	PRIMARY KEY (`source_tab`)
) ENGINE = INNODB DEFAULT CHARSET = utf8;


/*Table structure for table `student` */
DROP TABLE
IF EXISTS `student`;

CREATE TABLE `student` (
	`id` INT (11) NOT NULL AUTO_INCREMENT,
	`name` VARCHAR (255) NOT NULL,
	PRIMARY KEY (`id`)
) ENGINE = INNODB AUTO_INCREMENT = 5 DEFAULT CHARSET = utf8;

/*Data for the table `student` */
INSERT INTO `student` (`id`, `name`)
VALUES
	(1, 'zhangsan'),
	(2, 'lisi'),
	(3, 'wangwu'),
	(4, 'zhaoliu');
```

QueryMySql.java

```java
package org.duo.flumesource;

import org.apache.flume.Context;
import org.apache.flume.conf.ConfigurationException;
import org.apache.http.ParseException;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.sql.*;
import java.util.ArrayList;
import java.util.List;
import java.util.Properties;

public class QueryMySql {
    private static final Logger LOG = LoggerFactory.getLogger(QueryMySql.class);

    private int runQueryDelay, //两次查询的时间间隔
            startFrom,            //开始id
            currentIndex,	     //当前id
            recordSixe = 0,      //每次查询返回结果的条数
            maxRow;                //每次查询的最大条数


    private String table,       //要操作的表
            columnsToSelect,     //用户传入的查询的列
            customQuery,          //用户传入的查询语句
            query,                 //构建的查询语句
            defaultCharsetResultSet;//编码集

    //上下文，用来获取配置文件
    private Context context;

    //为定义的变量赋值（默认值），可在flume任务的配置文件中修改
    private static final int DEFAULT_QUERY_DELAY = 10000;
    private static final int DEFAULT_START_VALUE = 0;
    private static final int DEFAULT_MAX_ROWS = 2000;
    private static final String DEFAULT_COLUMNS_SELECT = "*";
    private static final String DEFAULT_CHARSET_RESULTSET = "UTF-8";

    private static Connection conn = null;
    private static PreparedStatement ps = null;
    private static String connectionURL, connectionUserName, connectionPassword;

    //加载静态资源
    static {
        Properties p = new Properties();
        try {
            p.load(QueryMySql.class.getClassLoader().getResourceAsStream("jdbc.properties"));
            connectionURL = p.getProperty("dbUrl");
            connectionUserName = p.getProperty("dbUser");
            connectionPassword = p.getProperty("dbPassword");
            Class.forName(p.getProperty("dbDriver"));
        } catch (Exception e) {
            LOG.error(e.toString());
        }
    }

    //获取JDBC连接
    private static Connection InitConnection(String url, String user, String pw) {
        try {
            Connection conn = DriverManager.getConnection(url, user, pw);
            if (conn == null)
                throw new SQLException();
            return conn;
        } catch (SQLException e) {
            e.printStackTrace();
        }
        return null;
    }

    //构造方法
    QueryMySql(Context context) throws ParseException {
        //初始化上下文
        this.context = context;

        //有默认值参数：获取flume任务配置文件中的参数，读不到的采用默认值
        this.columnsToSelect = context.getString("columns.to.select", DEFAULT_COLUMNS_SELECT);
        this.runQueryDelay = context.getInteger("run.query.delay", DEFAULT_QUERY_DELAY);
        this.startFrom = context.getInteger("start.from", DEFAULT_START_VALUE);
        this.defaultCharsetResultSet = context.getString("default.charset.resultset", DEFAULT_CHARSET_RESULTSET);

        //无默认值参数：获取flume任务配置文件中的参数
        this.table = context.getString("table");
        this.customQuery = context.getString("custom.query");
        connectionURL = context.getString("connection.url");
        connectionUserName = context.getString("connection.user");
        connectionPassword = context.getString("connection.password");
        conn = InitConnection(connectionURL, connectionUserName, connectionPassword);

        //校验相应的配置信息，如果没有默认值的参数也没赋值，抛出异常
        checkMandatoryProperties();
        //获取当前的id
        currentIndex = getStatusDBIndex(startFrom);
        //构建查询语句
        query = buildQuery();
    }

    //校验相应的配置信息（表，查询语句以及数据库连接的参数）
    private void checkMandatoryProperties() {
        if (table == null) {
            throw new ConfigurationException("property table not set");
        }
        if (connectionURL == null) {
            throw new ConfigurationException("connection.url property not set");
        }
        if (connectionUserName == null) {
            throw new ConfigurationException("connection.user property not set");
        }
        if (connectionPassword == null) {
            throw new ConfigurationException("connection.password property not set");
        }
    }

    //构建sql语句
    private String buildQuery() {
        String sql = "";
        //获取当前id
        currentIndex = getStatusDBIndex(startFrom);
        LOG.info(currentIndex + "");
        if (customQuery == null) {
            sql = "SELECT " + columnsToSelect + " FROM " + table;
        } else {
            sql = customQuery;
        }
        StringBuilder execSql = new StringBuilder(sql);
        //以id作为offset
        if (!sql.contains("where")) {
            execSql.append(" where ");
            execSql.append("id").append(">").append(currentIndex);
            return execSql.toString();
        } else {
            int length = execSql.toString().length();
            return execSql.toString().substring(0, length - String.valueOf(currentIndex).length()) + currentIndex;
        }
    }

    //执行查询
    List<List<Object>> executeQuery() {
        try {
            //每次执行查询时都要重新生成sql，因为id不同
            customQuery = buildQuery();
            //存放结果的集合
            List<List<Object>> results = new ArrayList<>();
            if (ps == null) {
                //
                ps = conn.prepareStatement(customQuery);
            }
            ResultSet result = ps.executeQuery(customQuery);
            while (result.next()) {
                //存放一条数据的集合（多个列）
                List<Object> row = new ArrayList<>();
                //将返回结果放入集合
                for (int i = 1; i <= result.getMetaData().getColumnCount(); i++) {
                    row.add(result.getObject(i));
                }
                results.add(row);
            }
            LOG.info("execSql:" + customQuery + "\nresultSize:" + results.size());
            return results;
        } catch (SQLException e) {
            LOG.error(e.toString());
            // 重新连接
            conn = InitConnection(connectionURL, connectionUserName, connectionPassword);
        }
        return null;
    }

    //将结果集转化为字符串，每一条数据是一个list集合，将每一个小的list集合转化为字符串
    List<String> getAllRows(List<List<Object>> queryResult) {
        List<String> allRows = new ArrayList<>();
        if (queryResult == null || queryResult.isEmpty())
            return allRows;
        StringBuilder row = new StringBuilder();
        for (List<Object> rawRow : queryResult) {
            Object value = null;
            for (Object aRawRow : rawRow) {
                value = aRawRow;
                if (value == null) {
                    row.append(",");
                } else {
                    row.append(aRawRow.toString()).append(",");
                }
            }
            allRows.add(row.toString());
            row = new StringBuilder();
        }
        return allRows;
    }

    //更新offset元数据状态，每次返回结果集后调用。必须记录每次查询的offset值，为程序中断续跑数据时使用，以id为offset
    void updateOffset2DB(int size) {
        //以source_tab做为KEY，如果不存在则插入，存在则更新（每个源表对应一条记录）
        String sql = "insert into flume_meta(source_tab,currentIndex) VALUES('"
                + this.table
                + "','" + (recordSixe += size)
                + "') on DUPLICATE key update source_tab=values(source_tab),currentIndex=values(currentIndex)";
        LOG.info("updateStatus Sql:" + sql);
        execSql(sql);
    }

    //执行sql语句
    private void execSql(String sql) {
        try {
            ps = conn.prepareStatement(sql);
            LOG.info("exec::" + sql);
            ps.execute();
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }

    //获取当前id的offset
    private Integer getStatusDBIndex(int startFrom) {
        //从flume_meta表中查询出当前的id是多少
        String dbIndex = queryOne("select currentIndex from flume_meta where source_tab='" + table + "'");
        if (dbIndex != null) {
            return Integer.parseInt(dbIndex);
        }
        //如果没有数据，则说明是第一次查询或者数据表中还没有存入数据，返回最初传入的值
        return startFrom;
    }

    //查询一条数据的执行语句(当前id)
    private String queryOne(String sql) {
        ResultSet result = null;
        try {
            ps = conn.prepareStatement(sql);
            result = ps.executeQuery();
            while (result.next()) {
                return result.getString(1);
            }
        } catch (SQLException e) {
            e.printStackTrace();
        }
        return null;
    }

    //关闭相关资源
    void close() {
        try {
            ps.close();
            conn.close();
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }

    int getCurrentIndex() {
        return currentIndex;
    }

    void setCurrentIndex(int newValue) {
        currentIndex = newValue;
    }

    int getRunQueryDelay() {
        return runQueryDelay;
    }

    String getQuery() {
        return query;
    }

    String getConnectionURL() {
        return connectionURL;
    }

    private boolean isCustomQuerySet() {
        return (customQuery != null);
    }

    Context getContext() {
        return context;
    }

    public String getConnectionUserName() {
        return connectionUserName;
    }

    public String getConnectionPassword() {
        return connectionPassword;
    }

    String getDefaultCharsetResultSet() {
        return defaultCharsetResultSet;
    }
}

```

MySqlSource.java

```java
package org.duo.flumesource;

import org.apache.flume.Context;
import org.apache.flume.Event;
import org.apache.flume.EventDeliveryException;
import org.apache.flume.PollableSource;
import org.apache.flume.conf.Configurable;
import org.apache.flume.event.SimpleEvent;
import org.apache.flume.source.AbstractSource;
import org.slf4j.Logger;

import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;

import static org.slf4j.LoggerFactory.*;

public class MySqlSource extends AbstractSource implements Configurable, PollableSource {

    //打印日志
    private static final Logger LOG = getLogger(MySqlSource.class);
    //定义sqlHelper
    private QueryMySql sqlSourceHelper;

    @Override
    public long getBackOffSleepIncrement() {
        return 0;
    }

    @Override
    public long getMaxBackOffSleepInterval() {
        return 0;
    }

    @Override
    public void configure(Context context) {
        //初始化
        sqlSourceHelper = new QueryMySql(context);
    }

    @Override
    public Status process() throws EventDeliveryException {
        try {
            //查询数据表
            List<List<Object>> result = sqlSourceHelper.executeQuery();
            //存放event的集合
            List<Event> events = new ArrayList<>();
            //存放event头集合
            HashMap<String, String> header = new HashMap<>();
            //如果有返回数据，则将数据封装为event
            if (!result.isEmpty()) {
                List<String> allRows = sqlSourceHelper.getAllRows(result);
                Event event = null;
                for (String row : allRows) {
                    event = new SimpleEvent();
                    event.setBody(row.getBytes());
                    event.setHeaders(header);
                    events.add(event);
                }
                //将event写入channel
                this.getChannelProcessor().processEventBatch(events);
                //更新数据表中的offset信息
                sqlSourceHelper.updateOffset2DB(result.size());
            }
            //等待时长
            Thread.sleep(sqlSourceHelper.getRunQueryDelay());
            return Status.READY;
        } catch (InterruptedException e) {
            LOG.error("Error procesing row", e);
            return Status.BACKOFF;
        }
    }

    @Override
    public synchronized void stop() {
        LOG.info("Stopping sql source {} ...", getName());
        try {
            //关闭资源
            sqlSourceHelper.close();
        } finally {
            super.stop();
        }
    }
}
```

mysqlsource.conf

```properties
a1.sources = r1
a1.sinks = k1
a1.channels = c1

# Describe/configure the source
a1.sources.r1.type = org.duo.flumesource.MySqlSource
a1.sources.r1.connection.url = jdbc:mysql://server01:3306/userdb
a1.sources.r1.connection.user = root
a1.sources.r1.connection.password = 123456
a1.sources.r1.table = student
a1.sources.r1.columns.to.select = *
a1.sources.r1.incremental.column.name = id
a1.sources.r1.incremental.value = 0
a1.sources.r1.run.query.delay=3000

# Describe the sink
a1.sinks.k1.type = logger

# Describe the channel
a1.channels.c1.type = memory
a1.channels.c1.capacity = 1000
a1.channels.c1.transactionCapacity = 100

# Bind the source and sink to the channel
a1.sources.r1.channels = c1
a1.sinks.k1.channel = c1
```

启动flume

```bash
[root@server01 conf]# flume-ng agent -c /usr/local/flume/conf/ -f /usr/local/flume/conf/mysqlsource.conf -n a1 -Dflume.root.logger=INFO,console
```

## Flume自定义Sink

同自定义 source 类似，对于某些 sink 如果没有我们想要的，我们也可以自定义 sink 实现将数据保存到我们想要的地方去，例如 kafka，或者 mysql，或者文件等等都可以

需求：从网络端口当中发送数据，自定义 sink，使用 sink 从网络端口接收数据，然后将数据保存到本地文件当中去。

MySink.java

```java
package org.duo.flumesink;

import org.apache.commons.io.FileUtils;
import org.apache.flume.*;
import org.apache.flume.conf.Configurable;
import org.apache.flume.sink.AbstractSink;

import java.io.*;

public class MySink extends AbstractSink implements Configurable {
    private Context context ;
    private String filePath = "";
    private String fileName = "";
    private File fileDir;

    //这个方法会在初始化调用，主要用于初始化我们的Context，获取我们的一些配置参数
    @Override
    public void configure(Context context) {
        try {
            this.context = context;
            filePath = context.getString("filePath");
            fileName = context.getString("fileName");
            fileDir = new File(filePath);
            if(!fileDir.exists()){
                fileDir.mkdirs();
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
    //这个方法会被反复调用
    @Override
    public Status process() throws EventDeliveryException {
        Event event = null;
        Channel channel = this.getChannel();
        Transaction transaction = channel.getTransaction();
        transaction.begin();
        while(true){
            event = channel.take();
            if(null != event){
                break;
            }
        }
        byte[] body = event.getBody();
        String line = new String(body);
        try {
            FileUtils.write(new File(filePath+File.separator+fileName),line,true);
            transaction.commit();
        } catch (IOException e) {
            transaction.rollback();
            e.printStackTrace();
            return Status.BACKOFF;
        }finally {
            transaction.close();
        }
        return Status.READY;
    }
}
```

filesink.conf

```properties
a1.sources = r1
a1.sinks = k1
a1.channels = c1
# Describe/configure the source
a1.sources.r1.type = netcat
a1.sources.r1.bind = server01
a1.sources.r1.port = 5678
a1.sources.r1.channels = c1
# # Describe the sink
a1.sinks.k1.type = org.duo.flumesink.MySink
a1.sinks.k1.filePath=/opt/logs
a1.sinks.k1.fileName=filesink.txt
# # Use a channel which buffers events in memory
a1.channels.c1.type = memory
a1.channels.c1.capacity = 1000
a1.channels.c1.transactionCapacity = 100
# # Bind the source and sink to the channel
a1.sources.r1.channels = c1
a1.sinks.k1.channel = c1

```

启动flume

```bash
[root@server01 conf]# flume-ng agent -c /usr/local/flume/conf -f /usr/local/flume/conf/filesink.conf -n a1 -Dflume.root.logger=INFO,console
```

# Azkaban

## 工作流

### 工作流产生背景

工作流（Workflow），指“业务过程的部分或整体在计算机应用环境下的自动化”。是对工作流程及其各操作步骤之间业务规则的抽象、概括描述。工作流解决的主要问题是：为了实现某个业务目标，利用计算机软件在多个参与者之间按某种预定规则自动传递文档、信息或者任务。一个完整的数据分析系统通常都是由多个前后依赖的模块组合构成的：数据采集、数据预处理、数据分析、数据展示等。各个模块单元之间存在时间先后依赖关系，且存在着周期性重复。为了很好地组织起这样的复杂执行计划，需要一个工作流调度系统来调度执行。

### 工作流调度实现方式

简单的任务调度：直接使用linux的crontab来定义,但是缺点也是比较明显，无法设置依赖。复杂的任务调度：自主开发调度平台，使用开源调度系统，比如azkaban、ApacheOozie、Cascading、Hamake等。其中知名度比较高的是ApacheOozie，但是其配置工作流的过程是编写大量的XML配置，而且代码复杂度比较高，不易于二次开发。

### 工作流调度工具之间对比

下面的表格对四种hadoop工作流调度器的关键特性进行了比较，尽管这些工作流调度器能够解决的需求场景基本一致，但在设计理念，目标用户，应用场景等方面还是存在显著的区别，在做技术选型的时候，可以提供参考。

| 特性                 | Hamake               | Oozie             | Azkaban                        | Cascading |
| -------------------- | -------------------- | ----------------- | ------------------------------ | --------- |
| 工作流描述语言       | XML                  | XML (xPDL based)  | text file with key/value pairs | Java API  |
| 依赖机制             | data-driven          | explicit          | explicit                       | explicit  |
| 是否要 web 容器      | No                   | Yes               | Yes                            | no        |
| 进度跟踪             | console/log messages | web page          | web page                       | Java API  |
| Hadoop job 调度 支持 | no                   | yes               | yes                            | no        |
| 运行模式             | command line utility | daemon            | daemon                         | Java API  |
| Pig 支持             | yes                  | yes               | yes                            | yes       |
| 事件通知             | no                   | no                | no                             | yes       |
| 需要安装             | no                   | yes               | yes                            | no        |
| 支持的 hadoop 版本   | 0.18+                | 0.20+             | currently unknown              | 0.18+     |
| 重试支持             | no                   | workflownode evel | yes                            | yes       |
| 运行任意命令         | yes                  | yes               | yes                            | yes       |
| Amazon EMR 支 持     | yes                  | no                | currently unknown              | yes       |

## Azkaban介绍

Azkaban是由linkedin（领英）公司推出的一个批量工作流任务调度器，用于在一个工作流内以一个特定的顺序运行一组工作和流程。Azkaban使用job配置文件建立任务之间的依赖关系，并提供一个易于使用的web用户界面维护和跟踪你的工作流。

Azkaban 功能特点：

1. 提供功能清晰，简单易用的 Web UI 界面
2. 提供 job 配置文件快速建立任务和任务之间的依赖关系
3. 提供模块化和可插拔的插件机制，原生支持 command、Java、Hive、Pig、Hadoop
4. 基于 Java 开发，代码结构清晰，易于二次开发

## Azkaban原理架构

1. mysql 服务器: 存储元数据，如项目名称、项目描述、项目权限、任务状态、SLA 规则等
2. AzkabanWebServer:对外提供 web 服务，使用户可以通过 web 页面管理。职责包括项目管理、权限授权、任务调度、监控 executor。
3. AzkabanExecutorServer:负责具体的工作流的提交、执行。

## Azkaban三种部署模式

### solo server mode

该模式中 webServer 和 executorServer 运行在同一个进程中，进程名是AzkabanSingleServer。使用自带的 H2 数据库。这种模式包含 Azkaban 的所有特性，但一般用来学习和测试。

### two-server mode

该模式使用 MySQL 数据库， Web Server 和 Executor Server 运行在不同的进程中。

### multiple-executor mode

该模式使用 MySQL 数据库， Web Server 和 Executor Server 运行在不同的机器中。且有多个 Executor Server。该模式适用于大规模应用。

## Azkaban源码编译

Azkaban3.x在安装前需要自己编译成二进制包。并且提前安装好Maven、Ant、Node等软件，具体请参考附件资料

### 编译环境

```bash
yum install –y git
yum install –y gcc-c++
```

### 下载源码解压

```bash
wget https://github.com/azkaban/azkaban/archive/3.51.0.tar.gz
tar -zxvf 3.51.0.tar.gz 
cd ./azkaban-3.51.0/
```

### 编译源码

```bash
# Gradle是一个基于Apache Ant和Apache Maven的项目自动化构建工具。-x test 跳过测试。（注意联网下载jar可能会失败、慢）
./gradlew build installDist -x test
```

### 编译后安装包路径

编译成功之后就可以在指定的路径下取得对应的安装包了：

#solo-server模式安装包路径

azkaban-solo-server/build/distributions/

#two-server模式和multiple-executor模式web-server安装包路径

azkaban-web-server/build/distributions/

#two-server模式和multiple-executor模式exec-server安装包路径

azkaban-exec-server/build/distributions/

#数据库相关安装包路径

azkaban-db/build/distributions/

## 安装部署

### solo-server模式部署

### two-server模式部署

#### 节点规划

| HOST     | 角色                            |
| -------- | ------------------------------- |
| server01 | MySQL                           |
| server02 | web‐server和exec‐server不同进程 |

#### mysql配置初始化

server01：

Mysql上创建对应的库、增加权限、创建表

```bash
[root@server01 local]# cd /usr/local
[root@server01 local]# tar -zxvf azkaban-db-0.1.0-SNAPSHOT.tar.gz
[root@server01 local]# mv azkaban-db-0.1.0-SNAPSHOT azkaban-db
[root@server01 local]# mysql -u root -p
mysql> CREATE DATABASE azkaban_two_server;
mysql> use azkaban_two_server;
mysql> source /usr/local/azkaban-db/create-all-sql-0.1.0-SNAPSHOT.sql;
```

#### web-server服务器配置

server02：

```bash
[root@server02 local]# tar -zxvf azkaban-web-server-0.1.0-SNAPSHOT.tar.gz
[root@server02 local]# mv azkaban-web-server-0.1.0-SNAPSHOT azkaban-web-server
[root@server02 local]# tar -zxvf azkaban-exec-server-0.1.0-SNAPSHOT.tar.gz
[root@server02 local]# mv azkaban-exec-server-0.1.0-SNAPSHOT azkaban-exec-server
```

生成SSL证书：

```bash
#运行此命令后,会提示输入当前生成keystore的密码及相应信息,输入的密码请记住(所有密码统一以123456输入)。完成上述工作后,将在当前目录生成keystore证书文件,将keystore拷贝到 azkaban web服务器根目录中。
[root@server02 azkaban-web-server]# keytool -keystore keystore -alias jetty -genkey -keyalg RSA
```

配置conf/azkaban.properties：

```properties
 # 修改时区，注意后面不要有空格
default.timezone.id=Asia/Shanghai
# Azkaban Jetty server properties.
# 开启使用ssl 并且设置端口
jetty.use.ssl=true
jetty.ssl.port=8443
# Azkaban Executor settings
# 指定本机Executor的运行端口（增加）
executor.host=localhost
executor.port=12321
# KeyStore for SSL ssl相关配置（增加）
# 如果keystore密码在azkaban web服务器根目录下，写文件名即可，否则要写文件全路径
jetty.keystore=keystore
jetty.password=123456
jetty.keypassword=123456
jetty.truststore=keystore
jetty.trustpassword=123456
# Azkaban mysql settings by default. Users should configure their own username and password.修改数据库信息
database.type=mysql
mysql.port=3306
mysql.host=server01
mysql.database=azkaban_two_server
mysql.user=root
mysql.password=123456
mysql.numconnections=100
#Multiple Executor
azkaban.use.multiple.executors=true
# 关闭内存检测：如果不注释的话，它会去检测运行的服务器是否有3G的内存
#azkaban.executorselector.filters=StaticRemainingFlowSize,MinimumFreeMemory,CpuStatus
azkaban.executorselector.comparator.NumberOfAssignedFlowComparator=1
azkaban.executorselector.comparator.Memory=1
azkaban.executorselector.comparator.LastDispatched=1
azkaban.executorselector.comparator.CpuUsage=1
```

新建commonprivate.properties

```bash
[root@server02 azkaban-web-server]# cd /usr/local/azkaban-web-server
[root@server02 azkaban-web-server]# mkdir -p plugins/jobtypes
[root@server02 azkaban-web-server]# vim commonprivate.properties
azkaban.native.lib=false
execute.as.user=false
memCheck.enabled=false
```

#### exec-server服务器配置

配置conf/azkaban.properties：

```properties
# 修改时区
default.timezone.id=Asia/Shanghai
# Where the Azkaban web server is located，修改web-server的url
azkaban.webserver.url=https://server02:8443
# Azkaban mysql settings by default. Users should configure their own username and password.修改数据库信息
database.type=mysql
mysql.port=3306
mysql.host=server01
mysql.database=azkaban_two_server
mysql.user=root
mysql.password=123456
mysql.numconnections=100
# Azkaban Executor settings，添加azkaban-exec执行端口
executor.maxThreads=50
executor.port=12321
executor.flow.threads=30
```

启动集群：

先启动exec-server

```bash
[root@server02 azkaban-exec-server]# cd /usr/local/azkaban-exec-server
[root@server02 azkaban-exec-server]# bin/start-exec.sh
```

需要手动激活executor

```bash
[root@server02 azkaban-exec-server]# cd /usr/local/azkaban-exec-server
[root@server02 azkaban-exec-server]# curl -G "server02:$(<./executor.port)/executor?action=activate" && echo
{"status":"success"}
```

再启动web-server

```bash
[root@server02 azkaban-exec-server]# cd /usr/local/azkaban-web-server
[root@server02 azkaban-web-server]# bin/start-web.sh
```

### multiple-executor模式部署

multiple-executor模式是多个executor Server分布在不同服务器上，只需要将azkaban-exec-server安装包拷贝到不同机器上即可组成分布式。

```bash
[root@server02 local]# scp -r azkaban-exec-server/ server03:/usr/local/
```

先启动exec-server

```bas
[root@server03 ~]# cd /usr/local/azkaban-exec-server/
[root@server03 azkaban-exec-server]# bin/start-exec.sh
```

手动激活executor

```bash
[root@server03 azkaban-exec-server]# curl -G "server03:$(<./executor.port)/executor?action=activate" && echo
{"status":"success"}
```

## 使用实战

### Shell command调度

创建job描述文件：command.job

```markdown
#command.job
type=command
command=sh hello.sh
```

创建shell脚本：hello.sh

```shell
#!/bin/bash
date  > /root/hello.txt
```

通过azkaban的web管理平台创建project并上传job压缩包，启动执行该job

### job依赖调度

创建有依赖关系的多个job描述

通过dependencies来配置所依赖的job(job文件的第一行# XXX.job来表示job的名称)

第一个job：foo.job

```markdown
# foo.job
type=command
command=echo foo
```

第二个job：bar.job

```markdown
# bar.job
type=command
dependencies=foo
command=echo bar
```

第三个job：duo.job

```markdown
# duo.job
type=command
dependencies=bar
command=echo duo
```

### HDFS任务调度

fs.job

```markdown
# fs.job
type=command
command=sh hdfs.sh
```

hdfs.sh

注意下的hadoop命令要写全路径，因为shell脚本启动相当新起了一个进程不会加载/etc/profile里面的环境变量，但是也可以在shell脚本中通过source命令加载一下/etc/profile里面的环境变量。

```sh
#!/bin/bash
/usr/local/hadoop-2.7.5/bin/hadoop fs -mkdir /azaz666
```

### 定时任务调度

通过页面配置

# 网站流量日志分析

## 背景

每个网站都有自己存在的目的和意义。除了政府和公益类网站之外，大多数网站的目的都是为了产生货币收入，说白了就是赚钱。要创建出用户需要的网站就必须进行网站分析，通过分析，找出用户实际需求，构建出符合用户需求的网站。

## 意义

网站分析，可以帮助网站管理员、运营人员、推广人员等实时获取网站流量信息，并从流量来源、网站内容、网站访客特性等多方面提供网站分析的数据依据。从而帮助提高网站流量，提升网站用户体验，让访客更多的沉淀下来变成会员或客户，通过更少的投入获取最大化的收入。

事实上网站分析设计的内容非常广泛，由很多部分组成。每一部分都可以单独作为一个分析项目：

1. 首先，网站分析是网站的眼睛。是从网站的营销角度看到的网站分析。在这部分中，网站分析的主要对象是访问者，访问者在网站中的行为以及不同流量之间的关系。
2. 其次，网站分析是整个网站的神经系统。这是从产品和架构的角度看到的网站分析。在这部分中，网站分析的主要对象是网站的逻辑和结构，网站的导航结构是否合理，注册购买流程的逻辑是否顺畅。
3. 最后，网站分析是网站的大脑，在这部门中，网站分析的主要分析对象是投资回报率（ROI）。也就是说在现有的情况下，如何合理的分配预算和资源以完成网站的目标。

终极意义：改善网站的运营，获取更高投资回报率（ROI）。赚更多的钱。

## 如何进行网站分析

### 网站流量质量分析（流量分析）

流量对于每个网站来说都是很重要，但流量并不是越多越好，应该更加看重流量的质量，换句话来说就是流量可以为我们带来多少收入。

### 网站流量多维度细分（流量分析）

细分是指通过不同维度对指标进行分割，查看同一个指标在不同维度下的表 现，进而找出有问题的那部分指标，对这部分指标进行优化。指标是访问量，就是我们常说的流量。在来源维度、媒介维度、时间维 度、位置维度等维度下，我们可以对访问量进行单独或者重叠的多维度细分。

### 网站内容及导航分析（内容分析）

对于所有网站来说，页面都可以被划分为三个类别：导航页、功能页、内容页。首页和列表页都是典型的导航页；站内搜索页面、注册表单页面和购物车页面都是典型的功能页，而产品详情页、新闻和文章页都是典型的内容页。导航页的目的是引导访问者找到信息，功能页的目的是帮助访问者完成特定任务，内容页的目的是向访问者展示信息并帮助访问者进行决策。比如从内容导航分析中，以下两类行为就是网站运营者不希望看到的行为：

1. 第一个问题：访问者从导航类页面（首页）进入，还没有看到内容类页面（详 情页）之前就从导航类页面（列表页）离开网站。在这次访问中，访问者并没有 完成任务，导航类页面也没有将访问者带入到内容类页面（详情页）中。因此， 需要分析导航类页面（列表页）造成访问者中途离开的原因。
2.  第二个问题：访问者从导航类页面（首页或列表页）进入网站，从内容类页 面（详情页）又返回到导航类页面（首页）。看似访问者在这次访问中完成了任 务（如果浏览内容页就是这个网站的最终目标的话），但其实访问者返回首页是 在开始一次新的导航或任务。说明需要分析内容页的最初设计，并考虑中内容页 提供交叉的信息推荐。

### 网站转化以及漏斗分析（转化分析）

转化，指网站业务流程中的一个封闭渠道，引导用户按照流程最终实现业务 目标（比如商品成交）；在这个渠道中，我们希望访问者一路向前，不要回头也 不要离开，直到完成转化目标。漏斗模型则是指进入渠道的用户在各环节递进过程中逐渐流失的形象描述。对于转化渠道，主要进行两部分的分析：访问者的流失和迷失。

#### 转化中的阻力的流失

转化的阻力是造成访问者流失的主要原因之一。这里的阻力包括：

1. 错误的设计、错误的引导
2. 错误的设计包括访问者在转化过程中找不到下一步操作的按钮，无法确认订 单信息，或无法完成支付等。 
3. 错误的引导包括在支付过程中提供很多离开的渠道链接，如不恰当的商品或 者活动推荐、对支付环节中专业名称的解释、帮助信息等内容。

造成流失的原因很多，如：

1. 不恰当的商品或活动推荐
2. 对支付环节中专业名词的解释、帮助信息等内容不当

#### 访问者的迷失

造成迷失的主要原因是转化流量设计不合理，访问者在特定阶段得不到需要 的信息，并且不能根据现有的信息作出决策，比如在线购买演唱会门票，直 到支付也没看到在线选座的提示，这时候就很可能会产生迷失，返回查看。

## 整体技术流程及架构

### 数据采集

数据采集概念，目前行业会有两种解释： 

1. 一是数据从无到有产生的过程（服务器打印的 log、自定义采集的日志等） 叫做数据采集； 
2. 另一方面也有把通过使用 Flume等工具把数据采集搬运到指定位置的这个 过程叫做数据采集。 

关于具体含义要结合语境具体分析，明白语境中具体含义即可。

#### 网站流量日志数据获取

随着网站在技术和运营上的不断技术发展，人们对数据的要求越来越高，以 求实现更加精细的运营来提升网站的质量。所以数据的获取方式也随着网站技术 的进步和人们对网站数据需求的加深而不断地发展。从使用发展来看，主要分为 2 类：网站日志文件（Log files）和页面埋点 js 自定义采集。 

##### 网站日志文件

记录网站日志文件的方式是最原始的数据获取方式，主要在服务端完成，在 网站的应用服务器配置相应的写日志的功能就能够实现，很多 web 应用服务器自 带日志的记录功能。如 Nginx 的 access.log 日志等。优点是获取数据时不需要对页面做相关处理，可以直接开始统计相关请求信 息，缺点在于有些信息无法采集，比如用户在页面端的操作（如点击、ajax 的使 用等）无法记录。限制了一些指标的统计和计算。

##### 页面埋点js自定义采集

自定义采集用户行为数据，通过在页面嵌入自定义的 javascript 代码来获取 用户的访问行为（比如鼠标悬停的位置，点击的页面组件等），然后通过 ajax 请 求到后台记录日志，这种方式所能采集的信息会更加全面。在实际操作中，有以下几个方面的数据可以自定义的采集：

1. 系统特征：比如所采用的操作系统、浏览器、域名和访问速度等。 
2. 访问特征：包括点击的 URL、所点击的“页面标签”及标签的属性等。 
3. 来源特征：包括来访 URL，来访 IP 等。 
4. 产品特征：包括所访问的产品编号、产品类别、产品颜色、产品价格、产品 利润、产品数量和特价等级等。

###### 原理分析

首先，用户的行为会触发浏览器对被统计页面的一个http请求，比如打开某网页、点击某个按钮或者链接，页面中的埋点javascript代码会被执行。埋点是指：在网页中预先加入小段javascript代码，这个代码片段一般会动态创建一个script标签，并将src属性指向一个单独的js文件，此时这个单独的js文件会被浏览器请求到并执行，这个js往往就是真正的数据收集脚本。数据收集完成后，js会请求一个后端的数据收集脚本，这个脚本一般是一个伪装成图片的动态脚本程序，js会将收集到的数据通过http参数的方式传递给后端脚本，后端脚本解析参数并按固定格式记录到访问日志，同时可能会在http响应中给客户端种植一些用于追踪的cookie。

###### 方案1

静态页面、js放nginx服务器中

index.html

```html
<!DOCTYPE html>
<html>
	<head>
		<meta charset="UTF-8">
		<title>welcom to itheima</title>	
	
		<script type="text/javascript">
        // _maq 是全局数组，收集各种配置信息，这里面可能会包括用户自定义的事件跟踪、业务数据（如电子商务网站的商品编号等）等。
		var _maq = _maq || [];
		//_maq.push([['_setAccount', 'AllenWoon'],['_setAvalue', '123456']]);
		_maq.push(['_setAccount', 'AllenWoon'],['_setAvalue', '123456']);
        // js自调用匿名函数
        // 格式： (function(){})();
        // 第一对括号向脚本返回未命名的函数；后一对空括号立即执行返回的未命名函数，括号内为匿名函数的参数。
        // 自调用匿名函数的好处是，避免重名，自调用匿名函数只会在运行时执行一次，一般用于初始化。
		(function() {
            // 这段代码的主要目的就是引入一个外部的js文件（duo.js），方式是通过 document.createElement方法创建一个script并根据协议（http或https）将src指向对应的duo.js，最后将这个元素插入页面的dom树上。
            // 注意duo.async = true的意思是异步调用外部js文件，即不阻塞浏览器的解析，待外部js下载完成后异步执行。这个属性是HTML5新引入的。
			var duo = document.createElement('script'); 
			duo.type = 'text/javascript';
			duo.async = true;
			duo.src = 'http://192.168.56.111/duo.js';
			var s = document.getElementsByTagName('script')[0]; 
			s.parentNode.insertBefore(duo, s);
		})();
		</script>	
	</head>
	<body>
		<h1 align="center">网站流量日志分析</h1>
	</body>
</html>

```

添加埋点js，放入nginx静态资源目录nginx/html 下

duo.js

```javascript
(function () {
    var params = {};
    //通过浏览器内置javascript对象收集信息，如页面title（通过document.title）、referrer（上一跳url，通过document.referrer）、用户显示器分辨率（通过windows.screen）、cookie信息（通过document.cookie）等等一些信息。
    //Document对象数据
    if(document) {
        params.domain = document.domain || ''; 
        params.url = document.URL || ''; 
        params.title = document.title || ''; 
        params.referrer = document.referrer || ''; 
    }   
    //Window对象数据
    if(window && window.screen) {
        params.sh = window.screen.height || 0;
        params.sw = window.screen.width || 0;
        params.cd = window.screen.colorDepth || 0;
    }   
    //navigator对象数据
    if(navigator) {
        params.lang = navigator.language || ''; 
    }
    //解析_maq数组，收集配置信息。这里面可能会包括用户自定义的事件跟踪、业务数据（如电子商务网站的商品编号等）等。
    if(_maq) {
        for(var i in _maq) {
            switch(_maq[i][0]) {
                case '_setAccount':
                    params.account = _maq[i][1];
                    break;
                case '_setAvalue':
                    params.avalue = _maq[i][1];
                    break;
                default:
                    break;
            }   
        }   
    }   
    //将上面两步收集的数据按预定义格式解析并拼接（get 请求参数）
    var args = ''; 
    for(var i in params) {
        if(args != '') {
            args += '&';
        }   
        args += i + '=' + encodeURIComponent(params[i]);
    }   

    //通过Image对象请求后端脚本
    //javascript请求后端脚本常用的方法是ajax，但是ajax是不能跨域请求的。一种通用的方法是js脚本创建一个Image对象，将Image对象的src属性指向后端脚本并携带参数，此时即实现了跨域请求后端。这也是后端脚本为什么通常伪装成gif文件的原因。
    var img = new Image(1, 1); 
    img.src = 'http://192.168.56.111/log.gif?' + args;
})();
```

nginx_web.conf

```properties
worker_processes  2;

events {
    worker_connections  1024;
}

http {
    include       mime.types;
    default_type  application/octet-stream;

	log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';
					  
    log_format user_log_format "$msec||$remote_addr||$status||$body_bytes_sent||$u_domain||$u_url||$u_title||$u_referrer||$u_sh||$u_sw||$u_cd||$u_lang||$http_user_agent||$u_account||$u_avalue";
    
    sendfile        on;  #允许sendfile方式传输文件，默认为off

    keepalive_timeout  65; #连接超时时间，默认为75s

    server {
        listen       80;
        server_name  localhost;
		location /log.gif {
			#伪装成gif文件
			default_type image/gif;    
			#nginx本身记录的access_log，日志格式为main
			access_log  logs/access.log  main;
		
			access_by_lua "
				-- 用户跟踪cookie名为__utrace
				local uid = ngx.var.cookie___utrace        
				if not uid then
					-- 如果没有则生成一个跟踪cookie，算法为md5(时间戳+IP+客户端信息)
					uid = ngx.md5(ngx.now() .. ngx.var.remote_addr .. ngx.var.http_user_agent)
				end 
				ngx.header['Set-Cookie'] = {'__utrace=' .. uid .. '; path=/'}
				if ngx.var.arg_domain then
				-- 通过subrequest到/i-log记录日志，将参数和用户跟踪cookie带过去
					ngx.location.capture('/i-log?' .. ngx.var.args .. '&utrace=' .. uid)
				end 
			";  
		
			#此请求资源本地不缓存
			add_header Expires "Fri, 01 Jan 1980 00:00:00 GMT";
			add_header Pragma "no-cache";
			add_header Cache-Control "no-cache, max-age=0, must-revalidate";
		
			#返回一个1×1的空gif图片
			empty_gif;
		}   
	
		location /i-log {
			#内部location，不允许外部直接访问
			internal;
		
			#设置变量，注意需要unescape
			set_unescape_uri $u_domain $arg_domain;
			set_unescape_uri $u_url $arg_url;
			set_unescape_uri $u_title $arg_title;
			set_unescape_uri $u_referrer $arg_referrer;
			set_unescape_uri $u_sh $arg_sh;
			set_unescape_uri $u_sw $arg_sw;
			set_unescape_uri $u_cd $arg_cd;
			set_unescape_uri $u_lang $arg_lang;
			set_unescape_uri $u_account $arg_account;
			set_unescape_uri $u_avalue $arg_avalue;
		
			#打开subrequest（子请求）日志
			log_subrequest on;
			#自定义采集的日志，记录数据到user_defined.log
			access_log logs/user_defined.log user_log_format;
		
			#输出空字符串
			echo '';
		}	
	
    }
}
```

###### 方案2

静态页面、js放nginx服务器中

index2.html

```html
<!DOCTYPE html>
<html>
	<head>
		<meta charset="UTF-8">
		<title>gentleduo</title>
    </head>
	<body>
		<h1 align="center">网站流量日志分析</h1>
		
		<a href="page1.html" target="_blank" clstag="click|index|page1">这是点击1</a><br/>
		<a href="page2.html" target="_blank" clstag="click|index|page2">这是点击2</a>
		
		<script type="text/javascript">
		
			var _maq = _maq || [];
		    _maq.push(['_setAccount', 'AllenWoon']);
			var aList = document.getElementsByTagName("a");
			
			for (var i = 0,mylength = aList.length; i<mylength; i++) {
				aList[i].addEventListener('click',function(){
					var clstag = this.attributes["clstag"].nodeValue;
					var _a_value = this.text;
					_maq.push(['_a_value',_a_value]);
					clstag = clstag.split('|');
					
					for (i in clstag){ 
						_maq.push(['type'+i, clstag[i]]);
						sendRequest();
					}
				},false);
			}

			function sendRequest(){
				var ma = document.createElement('script'); 
				ma.type = 'text/javascript';
				ma.async = true;
				ma.src = 'http://192.168.56.111/duo2.js';
				var s = document.getElementsByTagName('script')[0]; 
				s.parentNode.insertBefore(ma, s);
			}
		</script>
	</body>
</html>
```

page1.html

```html
<!DOCTYPE html>
<html>
	<head>
		<meta charset="UTF-8">
		<title>page1</title>	
		<script type="text/javascript">
		var _maq = _maq || [];
		//_maq.push(['_setAccount', 'AllenWoon']);
		_maq.push(['_setAccount', 'AllenWoon'],['_setAvalue', 'testValue1']);
		(function() {
			var ma = document.createElement('script'); 
			ma.type = 'text/javascript';
			ma.async = true;
			ma.src = 'http://192.168.56.111/duo2.js';
			var s = document.getElementsByTagName('script')[0]; 
			s.parentNode.insertBefore(ma, s);
		})();
		</script>		
	</head>
	<body>
		<h1 align="center">网站流量日志分析</h1>
		<h1 align="center">Page 1</h1>
	</body>
</html>
```

page2.html

```html
<!DOCTYPE html>
<html>
	<head>
		<meta charset="UTF-8">
		<title>page2</title>	
		<script type="text/javascript">
		var _maq = _maq || [];
		//_maq.push(['_setAccount', 'AllenWoon']);
		_maq.push(['_setAccount', 'AllenWoon'],['_setAvalue', 'testValue2']);
		(function() {
			var ma = document.createElement('script'); 
			ma.type = 'text/javascript';
			ma.async = true;
			ma.src = 'http://192.168.56.111/duo2.js';
			var s = document.getElementsByTagName('script')[0]; 
			s.parentNode.insertBefore(ma, s);
		})();
		</script>	
	</head>
	<body>
		<h1 align="center">网站流量日志分析</h1>
		<h1 align="center">page 2</h1>
	</body>
</html>
```

duo2.js

```javascript
(function () {
    var params = {};
    //Document对象数据
    if(document) {
        params.domain = document.domain || ''; 
        params.url = document.URL || ''; 
        params.title = document.title || ''; 
        params.referrer = document.referrer || ''; 
    }   
    //Window对象数据
    if(window && window.screen) {
        params.sh = window.screen.height || 0;
        params.sw = window.screen.width || 0;
        params.cd = window.screen.colorDepth || 0;
    }   
    //navigator对象数据
    if(navigator) {
        params.lang = navigator.language || ''; 
    }   
    //解析_maq配置
    if(_maq) {
        for(var i in _maq) {
            switch(_maq[i][0]) {
                case '_setAccount':
                    params.account = _maq[i][1];
                    break;
				case '_a_value':
                    params.avalue = _maq[i][1];
                    break;
				case 'type0':
                    params.type0 = _maq[i][1];
                    break;
				case 'type1':
                    params.type1 = _maq[i][1];
                    break;
				case 'type2':
                    params.type2 = _maq[i][1];
                    break;
                default:
                    break;
            }   
        }   
    }   
    //拼接参数串
    var args = ''; 
    for(var i in params) {
        if(args != '') {
            args += '&';
        }   
        args += i + '=' + encodeURIComponent(params[i]);
    }   
 
    //通过Image对象请求后端脚本
    var img = new Image(1, 1); 
    img.src = 'http://192.168.56.111/log.gif?' + args;
})();
```

nginx_web2.conf

```properties
worker_processes  2;

events {
    worker_connections  1024;
}


http {
    include       mime.types;
    default_type  application/octet-stream;

	log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';
  	
    log_format user_log_format "$msec||$remote_addr||$status||$body_bytes_sent||$u_domain||$u_url||$u_title||$u_referrer||$u_sh||$u_sw||$u_cd||$u_lang||$http_user_agent||$u_account||$u_avalue||$u_type0||$u_type1||$u_type2";
    
    sendfile        on;

    keepalive_timeout  65;

    server {
        listen       80;
        server_name  localhost;
		location /log.gif {
			#伪装成gif文件
			default_type image/gif;    
			#nginx本身记录的access_log，日志格式为main
			access_log  logs/access.log  main;
		
			access_by_lua "
				-- 用户跟踪cookie名为__utrace
				local uid = ngx.var.cookie___utrace        
				if not uid then
					-- 如果没有则生成一个跟踪cookie，算法为md5(时间戳+IP+客户端信息)
					uid = ngx.md5(ngx.now() .. ngx.var.remote_addr .. ngx.var.http_user_agent)
				end 
				ngx.header['Set-Cookie'] = {'__utrace=' .. uid .. '; path=/'}
				if ngx.var.arg_domain then
				-- 通过subrequest到/i-log记录日志，将参数和用户跟踪cookie带过去
					ngx.location.capture('/i-log?' .. ngx.var.args .. '&utrace=' .. uid)
				end 
			";  
		
			#此请求不缓存
			add_header Expires "Fri, 01 Jan 1980 00:00:00 GMT";
			add_header Pragma "no-cache";
			add_header Cache-Control "no-cache, max-age=0, must-revalidate";
		
			#返回一个1×1的空gif图片
			empty_gif;
		}   
	
		location /i-log {
			#内部location，不允许外部直接访问
			internal;
		
			#设置变量，注意需要unescape
			set_unescape_uri $u_domain $arg_domain;
			set_unescape_uri $u_url $arg_url;
			set_unescape_uri $u_title $arg_title;
			set_unescape_uri $u_referrer $arg_referrer;
			set_unescape_uri $u_sh $arg_sh;
			set_unescape_uri $u_sw $arg_sw;
			set_unescape_uri $u_cd $arg_cd;
			set_unescape_uri $u_lang $arg_lang;
			set_unescape_uri $u_account $arg_account;
			set_unescape_uri $u_avalue $arg_avalue;
			set_unescape_uri $u_type0 $arg_type0;
			set_unescape_uri $u_type1 $arg_type1;
			set_unescape_uri $u_type2 $arg_type2;
		
			#打开subrequest（子请求）日志
			log_subrequest on;
			#自定义采集的日志，记录数据到user_defined.log
			access_log logs/user_defined.log user_log_format;
		
			#输出空字符串
			echo '';
		}	
	
    }
}
```

###### 方案3

动态web工程、js放web项目中，nginx只用于日志采集。方案3中有如下几个概念：

1. 用户/访客：表示同一浏览器代表的用户。唯一标识用户。注意：由于这个项目没有后台所以无法通过服务器种植的方式设置cookie，而且由于可能每个网站对用户/访客的定义都不一样从而导致对新增用户的计算方式也不一样。方案3中对用户的定义是：是否使用同一个浏览器来访问网站：在用户第一次使用某个浏览器访问网站的时候，会在该浏览器的cookie中生成一个唯一标识用户身份的uuid，并且过期时间设置为10年，所以10年内用户再使用同一浏览器来访问该网站就算老用户了。
2. 会员：表示网站的一个正常的会员用户。
3. 会话：一段时间内的连续操作，就是一个会话中的所有操作。
4. Pv：访问页面的数量

数据参数说明

| 参数名称 | 类型   | 描述                       |
| -------- | ------ | -------------------------- |
| en       | string | 事件名称, eg: e_pv         |
| ver      | string | 版本号, eg: 0.0.1          |
| pl       | string | 平台, eg: website          |
| sdk      | string | Sdk类型, eg: js            |
| b_rst    | string | 浏览器分辨率，eg: 1800*678 |
| b_iev    | string | 浏览器信息useragent        |
| u_ud     | string | 用户/访客唯一标识符        |
| l        | string | 客户端语言                 |
| u_mid    | string | 会员id，和业务系统一致     |
| u_sd     | string | 会话id                     |
| c_time   | string | 客户端时间                 |
| p_url    | string | 当前页面的url              |
| p_ref    | string | 上一个页面的url            |
| tt       | string | 当前页面的标题             |
| ca       | string | Event事件的Category名称    |
| ac       | string | Event事件的action名称      |
| kv_*     | string | Event事件的自定义属性      |
| du       | string | Event事件的持续时间        |
| oid      | string | 订单id                     |
| on       | string | 订单名称                   |
| cua      | string | 支付金额                   |
| cut      | string | 支付货币类型               |
| pt       | string | 支付方式                   |

web/js/analytics.js

```javascript
(function() {
	var CookieUtil = {
		// get the cookie of the key is name
		get : function(name) {
			// 定义三个变量：cookieName、cookieStart以及cookieValue，在定义的同时给其赋值，
			// 第二个和第三个变量定义时由于使用","跟在第一个变量定义的后面，所以可以省略的关键字var
			var cookieName = encodeURIComponent(name) + "=", cookieStart = document.cookie
					.indexOf(cookieName), cookieValue = null;
			if (cookieStart > -1) {
				var cookieEnd = document.cookie.indexOf(";", cookieStart);
				if (cookieEnd == -1) {
					cookieEnd = document.cookie.length;
				}
				cookieValue = decodeURIComponent(document.cookie.substring(
						cookieStart + cookieName.length, cookieEnd));
			}
			return cookieValue;
		},
		// set the name/value pair to browser cookie
		set : function(name, value, expires, path, domain, secure) {
			var cookieText = encodeURIComponent(name) + "="
					+ encodeURIComponent(value);

			if (expires) {
				// set the expires time
				var expiresTime = new Date();
				expiresTime.setTime(expires);
				cookieText += ";expires=" + expiresTime.toGMTString();
			}

			if (path) {
				cookieText += ";path=" + path;
			}

			if (domain) {
				cookieText += ";domain=" + domain;
			}

			if (secure) {
				cookieText += ";secure";
			}

			document.cookie = cookieText;
		},
		setExt : function(name, value) {
			this.set(name, value, new Date().getTime() + 315360000000, "/");
		}
	};

	// 主体，其实就是tracker js
	var tracker = {
		// config
		clientConfig : {
			serverUrl : "http://server02/log.gif",
			sessionTimeout : 360, // 360s -> 6min
			maxWaitTime : 3600, // 3600s -> 60min -> 1h
			ver : "1"
		},

		cookieExpiresTime : 315360000000, // cookie过期时间，10年

		columns : {
			// 发送到服务器的列名称
			eventName : "en",
			version : "ver",
			platform : "pl",
			sdk : "sdk",
			uuid : "u_ud",
			memberId : "u_mid",
			sessionId : "u_sd",
			clientTime : "c_time",
			language : "l",
			userAgent : "b_iev",
			resolution : "b_rst",
			currentUrl : "p_url",
			referrerUrl : "p_ref",
			title : "tt",
			orderId : "oid",
			orderName : "on",
			currencyAmount : "cua",
			currencyType : "cut",
			paymentType : "pt",
			category : "ca",
			action : "ac",
			kv : "kv_",
			duration : "du"
		},

		keys : {
			pageView : "e_pv",
			chargeRequestEvent : "e_crt",
			launch : "e_l",
			eventDurationEvent : "e_e",
			sid : "bftrack_sid",
			uuid : "bftrack_uuid",
			mid : "bftrack_mid",
			preVisitTime : "bftrack_previsit",

		},

		/**
		 * 获取会话id
		 */
		getSid : function() {
			return CookieUtil.get(this.keys.sid);
		},

		/**
		 * 保存会话id到cookie
		 */
		setSid : function(sid) {
			if (sid) {
				CookieUtil.setExt(this.keys.sid, sid);
			}
		},

		/**
		 * 获取uuid，从cookie中
		 */
		getUuid : function() {
			return CookieUtil.get(this.keys.uuid);
		},

		/**
		 * 保存uuid到cookie
		 */
		setUuid : function(uuid) {
			if (uuid) {
				CookieUtil.setExt(this.keys.uuid, uuid);
			}
		},

		/**
		 * 获取memberID
		 */
		getMemberId : function() {
			return CookieUtil.get(this.keys.mid);
		},

		/**
		 * 设置mid
		 */
		setMemberId : function(mid) {
			if (mid) {
				CookieUtil.setExt(this.keys.mid, mid);
			}
		},

		startSession : function() {
			// 加载js就触发的方法
			if (this.getSid()) {
				// 会话id存在，表示uuid也存在
				if (this.isSessionTimeout()) {
					// 会话过期,产生新的会话
					this.createNewSession();
				} else {
					// 会话没有过期，更新最近访问时间
					this.updatePreVisitTime(new Date().getTime());
				}
			} else {
				// 会话id不存在，表示uuid也不存在
				this.createNewSession();
			}
			this.onPageView();
		},

        // 当用户第一次访问网站的时候触发该事件
		onLaunch : function() {
			// 触发launch事件
			var launch = {};
			launch[this.columns.eventName] = this.keys.launch; // 设置事件名称
			this.setCommonColumns(launch); // 设置公用columns
			this.sendDataToServer(this.parseParam(launch)); // 最终发送编码后的数据
		},

        // 当用户访问页面/刷新页面的时候触发该事件
		onPageView : function() {
			// 触发page view事件
			if (this.preCallApi()) {
				var time = new Date().getTime();
				var pageviewEvent = {};
				pageviewEvent[this.columns.eventName] = this.keys.pageView;
				pageviewEvent[this.columns.currentUrl] = window.location.href; // 设置当前url
				pageviewEvent[this.columns.referrerUrl] = document.referrer; // 设置前一个页面的url
				pageviewEvent[this.columns.title] = document.title; // 设置title
				this.setCommonColumns(pageviewEvent); // 设置公用columns
				this.sendDataToServer(this.parseParam(pageviewEvent)); // 最终发送编码后的数据
				this.updatePreVisitTime(time);
			}
		},

        // 当用户下订单的时候触发该事件，该事件需要后台程序主动调用。
		onChargeRequest : function(orderId, name, currencyAmount, currencyType, paymentType) {
			// 触发订单产生事件
			if (this.preCallApi()) {
				if (!orderId || !currencyType || !paymentType) {
					this.log("订单id、货币类型以及支付方式不能为空");
					return;
				}

				if (typeof (currencyAmount) == "number") {
					// 金额必须是数字
					var time = new Date().getTime();
					var chargeRequestEvent = {};
					chargeRequestEvent[this.columns.eventName] = this.keys.chargeRequestEvent;
					chargeRequestEvent[this.columns.orderId] = orderId;
					chargeRequestEvent[this.columns.orderName] = name;
					chargeRequestEvent[this.columns.currencyAmount] = currencyAmount;
					chargeRequestEvent[this.columns.currencyType] = currencyType;
					chargeRequestEvent[this.columns.paymentType] = paymentType;
					this.setCommonColumns(chargeRequestEvent); // 设置公用columns
					this.sendDataToServer(this.parseParam(chargeRequestEvent)); // 最终发送编码后的数据
					this.updatePreVisitTime(time);
				} else {
					this.log("订单金额必须是数字");
					return;
				}
			}
		},

        // 当访客/用户触发业务定义的事件后，前端程序调用该方法。
		onEventDuration : function(category, action, map, duration) {
			// 触发event事件
			if (this.preCallApi()) {
				if (category && action) {
					var time = new Date().getTime();
					var event = {};
					event[this.columns.eventName] = this.keys.eventDurationEvent;
					event[this.columns.category] = category;
					event[this.columns.action] = action;
					if (map) {
						for ( var k in map) {
							if (k && map[k]) {
								event[this.columns.kv + k] = map[k];
							}
						}
					}
					if (duration) {
						event[this.columns.duration] = duration;
					}
					this.setCommonColumns(event); // 设置公用columns
					this.sendDataToServer(this.parseParam(event)); // 最终发送编码后的数据
					this.updatePreVisitTime(time);
				} else {
					this.log("category和action不能为空");
				}
			}
		},

		/**
		 * 执行对外方法前必须执行的方法
		 */
		preCallApi : function() {
			if (this.isSessionTimeout()) {
				// 如果为true，表示需要新建
				this.startSession();
			} else {
				this.updatePreVisitTime(new Date().getTime());
			}
			return true;
		},

		sendDataToServer : function(data) {
			
			//alert(data);
			
			// 发送数据data到服务器，其中data是一个字符串
			var that = this;
			var i2 = new Image(1, 1);// <img src="url"></img>
			i2.onerror = function() {
				// 这里可以进行重试操作
			};
			i2.src = this.clientConfig.serverUrl + "?" + data;
		},

		/**
		 * 往data中添加发送到日志收集服务器的公用部分
		 */
		setCommonColumns : function(data) {
			data[this.columns.version] = this.clientConfig.ver;
			data[this.columns.platform] = "website";
			data[this.columns.sdk] = "js";
			data[this.columns.uuid] = this.getUuid(); // 设置用户id
			data[this.columns.memberId] = this.getMemberId(); // 设置会员id
			data[this.columns.sessionId] = this.getSid(); // 设置sid
			data[this.columns.clientTime] = new Date().getTime(); // 设置客户端时间
			data[this.columns.language] = window.navigator.language; // 设置浏览器语言
			data[this.columns.userAgent] = window.navigator.userAgent; // 设置浏览器类型
			data[this.columns.resolution] = screen.width + "*" + screen.height; // 设置浏览器分辨率
		},

		/**
		 * 创建新的会员，并判断是否是第一次访问页面，如果是，进行launch事件的发送。
		 */
		createNewSession : function() {
			var time = new Date().getTime(); // 获取当前操作时间
			// 1. 进行会话更新操作
			var sid = this.generateId(); // 产生一个session id
			this.setSid(sid);
			this.updatePreVisitTime(time); // 更新最近访问时间
			// 2. 进行uuid查看操作
			if (!this.getUuid()) {
				// uuid不存在，先创建uuid，然后保存到cookie，最后触发launch事件
				var uuid = this.generateId(); // 产品uuid
				this.setUuid(uuid);
				this.onLaunch();
			}
		},

		/**
		 * 参数编码返回字符串
		 */
		parseParam : function(data) {
			var params = "";
			for ( var e in data) {
				if (e && data[e]) {
					params += encodeURIComponent(e) + "="
							+ encodeURIComponent(data[e]) + "&";
				}
			}
			if (params) {
				return params.substring(0, params.length - 1);
			} else {
				return params;
			}
		},

		/**
		 * 产生uuid
		 */
		generateId : function() {
			var chars = '0123456789ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz';
			var tmpid = [];
			var r;
			tmpid[8] = tmpid[13] = tmpid[18] = tmpid[23] = '-';
			tmpid[14] = '4';

			for (i = 0; i < 36; i++) {
				if (!tmpid[i]) {
					r = 0 | Math.random() * 16;
					tmpid[i] = chars[(i == 19) ? (r & 0x3) | 0x8 : r];
				}
			}
			return tmpid.join('');
		},

		/**
		 * 判断这个会话是否过期，查看当前时间和最近访问时间间隔时间是否小于this.clientConfig.sessionTimeout<br/>
		 * 如果是小于，返回false;否则返回true。
		 */
		isSessionTimeout : function() {
			var time = new Date().getTime();
			var preTime = CookieUtil.get(this.keys.preVisitTime);
			if (preTime) {
				// 最近访问时间存在,那么进行区间判断
				return time - preTime > this.clientConfig.sessionTimeout * 1000;
			}
			return true;
		},

		/**
		 * 更新最近访问时间
		 */
		updatePreVisitTime : function(time) {
			CookieUtil.setExt(this.keys.preVisitTime, time);
		},

		/**
		 * 打印日志
		 */
		log : function(msg) {
			console.log(msg);
		},

	};

	// 对外暴露的方法名称
	window.__AE__ = {
		startSession : function() {
			tracker.startSession();
		},
		onPageView : function() {
			tracker.onPageView();
		},
		onChargeRequest : function(orderId, name, currencyAmount, currencyType, paymentType) {
			tracker.onChargeRequest(orderId, name, currencyAmount, currencyType, paymentType);
		},
		onEventDuration : function(category, action, map, duration) {
			tracker.onEventDuration(category, action, map, duration);
		},
		setMemberId : function(mid) {
			tracker.setMemberId(mid);
		}
	};

	// 自动加载方法
	var autoLoad = function() {
		// 进行参数设置
		var _aelog_ = _aelog_ || window._aelog_ || [];
		var memberId = null;
		for (i = 0; i < _aelog_.length; i++) {
			// 如果_aelog_[i][0]等于"memberId"的话执行后面的赋值语句：memberId = _aelog_[i][1]
			// ==会进行类型的转换之后再判断两者是否相等，而===不会进行数据类型的转换，先判断两边的数据类型是否相等，如果数据类型相等的话才会进行接下来的判断，再进行等式两边值得判断，可以理解为只有等式两边是全等（数据类型相同，值相同）的时候结果才会是true，否则全为false。
			_aelog_[i][0] === "memberId" && (memberId = _aelog_[i][1]);
		}
		// 根据是给定memberid，设置memberid的值
		// 如果memberId为真(即：memberId不为空)则执行后续的setMemberId
		memberId && __AE__.setMemberId(memberId);
		// 启动session
		__AE__.startSession();
	};

	autoLoad();
})();
```

web/demo.jsp

```jsp
<%@ page contentType="text/html; charset=utf-8" pageEncoding="utf-8"%>
<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
<html>
<head>
<meta http-equiv="Content-Type" content="text/html; charset=utf-8">
<title>测试页面1</title>
<script type="text/javascript" src="./js/analytics.js"></script>
</head>
<body>
	测试页面1<br/>
	跳转到:
	<a href="demo.jsp">demo</a>
	<a href="demo2.jsp">demo2</a>
	<a href="demo3.jsp">demo3</a>
	<a href="demo4.jsp">demo4</a>
</body>
</html>
```

web/demo2.jsp

```jsp
<%@ page contentType="text/html; charset=utf-8" pageEncoding="utf-8"%>
<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
<html>
<head>
<meta http-equiv="Content-Type" content="text/html; charset=utf-8">
<title>测试页面2</title>
<script type="text/javascript" src="./js/analytics.js"></script>
</head>
<body>
	测试页面2
	<br/>
	<label>orderid: 123456</label><br>
	<label>orderName: 测试订单123456</label><br/>
	<label>currencyAmount: 524.01</label><br/>
	<label>currencyType: RMB</label><br/>
	<label>paymentType: alipay</label><br/>
	<button onclick="__AE__.onChargeRequest('123456','测试订单123456',524.01,'RMB','alipay')">触发chargeRequest事件</button><br/>
	跳转到:
	<a href="demo.jsp">demo</a>
	<a href="demo2.jsp">demo2</a>
	<a href="demo3.jsp">demo3</a>
	<a href="demo4.jsp">demo4</a>
</body>
</html>
```

demo3.jsp

```jsp
<%@ page contentType="text/html; charset=utf-8" pageEncoding="utf-8"%>
<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
<html>
<head>
<meta http-equiv="Content-Type" content="text/html; charset=utf-8">
<title>测试页面3</title>
<script type="text/javascript" src="./js/analytics.js"></script>
</head>
<body>
	测试页面3<br/>
	<label>category: event的category名称</label><br/>
	<label>action: event的action名称</label><br/>
	<label>map: {"key1":"value1", "key2":"value2"}</label><br/>
	<label>duration: 1245</label><br/>
	<button onclick="__AE__.onEventDuration('event的category名称','event的action名称', {'key1':'value1','key2':'value2'}, 1245)">触发带map和duration的事件</button><br/>
	<button onclick="__AE__.onEventDuration('event的category名称','event的action名称')">触发不带map和duration的事件</button><br/>
	跳转到:
	<a href="demo.jsp">demo</a>
	<a href="demo2.jsp">demo2</a>
	<a href="demo3.jsp">demo3</a>
	<a href="demo4.jsp">demo4</a>
</body>
</html>
```

demo4.jsp

```jsp
<%@ page contentType="text/html; charset=utf-8" pageEncoding="utf-8"%>
<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
<html>
<head>
<meta http-equiv="Content-Type" content="text/html; charset=utf-8">
<title>测试页面4</title>
<script type="text/javascript">
(function(){
	var _aelog_ = _aelog_ || window._aelog_ || [];
	// 设置_aelog_相关属性
	_aelog_.push(["memberId","zhangsan"]);
	window._aelog_ = _aelog_;
	(function(){
	    var aejs = document.createElement('script');
	    aejs.type = 'text/javascript';
	    aejs.async = true;
	    aejs.src = './js/analytics.js';
	    var script = document.getElementsByTagName('script')[0];
	    script.parentNode.insertBefore(aejs, script);
	})();
})();
</script>
</head>
<body>
	测试页面4<br/>
	在本页面设置memberid为zhangsan<br/>
	跳转到:
	<a href="demo.jsp">demo</a>
	<a href="demo2.jsp">demo2</a>
	<a href="demo3.jsp">demo3</a>
	<a href="demo4.jsp">demo4</a>
</body>
</html>
```

web/WEB-INF/web.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
         version="4.0">
    <display-name>log_analyse</display-name>
    <welcome-file-list>
        <welcome-file>index.html</welcome-file>
        <welcome-file>index.htm</welcome-file>
        <welcome-file>index.jsp</welcome-file>
        <welcome-file>default.html</welcome-file>
        <welcome-file>default.htm</welcome-file>
        <welcome-file>default.jsp</welcome-file>
    </welcome-file-list>
</web-app>
```

com.duo.client.AnalyticsEngineSDK

```java
package com.duo.client;

import java.io.UnsupportedEncodingException;
import java.net.URLEncoder;
import java.util.HashMap;
import java.util.Map;
import java.util.logging.Level;
import java.util.logging.Logger;

/**
 * 分析引擎sdk java服务器端数据收集
 */
public class AnalyticsEngineSDK {
	// 日志打印对象
	private static final Logger log = Logger.getGlobal();
	// 请求url的主体部分
	public static final String accessUrl = "http://server02/log.gif";
	private static final String platformName = "java_server";
	private static final String sdkName = "jdk";
	private static final String version = "1";

	/**
	 * 触发订单支付成功事件，发送事件数据到服务器
	 * 
	 * @param orderId
	 *            订单支付id
	 * @param memberId
	 *            订单支付会员id
	 * @return 如果发送数据成功(加入到发送队列中)，那么返回true；否则返回false(参数异常&添加到发送队列失败).
	 */
	public static boolean onChargeSuccess(String orderId, String memberId) {
		try {
			if (isEmpty(orderId) || isEmpty(memberId)) {
				// 订单id或者memberid为空
				log.log(Level.WARNING, "订单id和会员id不能为空");
				return false;
			}
			// 代码执行到这儿，表示订单id和会员id都不为空。
			Map<String, String> data = new HashMap<String, String>();
			data.put("u_mid", memberId);
			data.put("oid", orderId);
			data.put("c_time", String.valueOf(System.currentTimeMillis()));
			data.put("ver", version);
			data.put("en", "e_cs");
			data.put("pl", platformName);
			data.put("sdk", sdkName);
			// 创建url
			String url = buildUrl(data);
			// 发送url&将url加入到队列
			SendDataMonitor.addSendUrl(url);
			return true;
		} catch (Throwable e) {
			log.log(Level.WARNING, "发送数据异常", e);
		}
		return false;
	}

	/**
	 * 触发订单退款事件，发送退款数据到服务器
	 * 
	 * @param orderId
	 *            退款订单id
	 * @param memberId
	 *            退款会员id
	 * @return 如果发送数据成功，返回true。否则返回false。
	 */
	public static boolean onChargeRefund(String orderId, String memberId) {
		try {
			if (isEmpty(orderId) || isEmpty(memberId)) {
				// 订单id或者memberid为空
				log.log(Level.WARNING, "订单id和会员id不能为空");
				return false;
			}
			// 代码执行到这儿，表示订单id和会员id都不为空。
			Map<String, String> data = new HashMap<String, String>();
			data.put("u_mid", memberId);
			data.put("oid", orderId);
			data.put("c_time", String.valueOf(System.currentTimeMillis()));
			data.put("ver", version);
			data.put("en", "e_cr");
			data.put("pl", platformName);
			data.put("sdk", sdkName);
			// 构建url
			String url = buildUrl(data);
			// 发送url&将url添加到队列中
			SendDataMonitor.addSendUrl(url);
			return true;
		} catch (Throwable e) {
			log.log(Level.WARNING, "发送数据异常", e);
		}
		return false;
	}

	/**
	 * 根据传入的参数构建url
	 * 
	 * @param data
	 * @return
	 * @throws UnsupportedEncodingException
	 */
	private static String buildUrl(Map<String, String> data)
			throws UnsupportedEncodingException {
		StringBuilder sb = new StringBuilder();
		sb.append(accessUrl).append("?");
		for (Map.Entry<String, String> entry : data.entrySet()) {
			if (isNotEmpty(entry.getKey()) && isNotEmpty(entry.getValue())) {
				sb.append(entry.getKey().trim())
						.append("=")
						.append(URLEncoder.encode(entry.getValue().trim(), "utf-8"))
						.append("&");
			}
		}
		return sb.substring(0, sb.length() - 1);// 去掉最后&
	}

	/**
	 * 判断字符串是否为空，如果为空，返回true。否则返回false。
	 * 
	 * @param value
	 * @return
	 */
	private static boolean isEmpty(String value) {
		return value == null || value.trim().isEmpty();
	}

	/**
	 * 判断字符串是否非空，如果不是空，返回true。如果是空，返回false。
	 * 
	 * @param value
	 * @return
	 */
	private static boolean isNotEmpty(String value) {
		return !isEmpty(value);
	}
}
```

com.duo.client.SendDataMonitor

```java
package com.duo.client;

import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;
import java.net.HttpURLConnection;
import java.net.URL;
import java.util.concurrent.BlockingQueue;
import java.util.concurrent.LinkedBlockingQueue;
import java.util.logging.Level;
import java.util.logging.Logger;

/**
 * 发送url数据的监控者，用于启动一个单独的线程来发送数据
 */
public class SendDataMonitor {
    // 日志记录对象
    private static final Logger log = Logger.getGlobal();
    // 队列，用户存储发送url
    private BlockingQueue<String> queue = new LinkedBlockingQueue<String>();
    // 用于单列的一个类对象
    private static SendDataMonitor monitor = null;

    private SendDataMonitor() {
        // 私有构造方法，进行单列模式的创建
    }

    /**
     * 获取单列的monitor对象实例
     *
     * @return
     */
    public static SendDataMonitor getSendDataMonitor() {
        if (monitor == null) {
            synchronized (SendDataMonitor.class) {
                if (monitor == null) {
                    monitor = new SendDataMonitor();

                    Thread thread = new Thread(new Runnable() {

                        @Override
                        public void run() {
                            // 线程中调用具体的处理方法
                            SendDataMonitor.monitor.run();
                        }
                    });
                    // 测试的时候，不设置为守护模式
                    // thread.setDaemon(true);
                    thread.start();
                }
            }
        }
        return monitor;
    }

    /**
     * 添加一个url到队列中去
     *
     * @param url
     * @throws InterruptedException
     */
    public static void addSendUrl(String url) throws InterruptedException {
        getSendDataMonitor().queue.put(url);
    }

    /**
     * 具体执行发送url的方法
     */
    private void run() {
        while (true) {
            try {
                String url = this.queue.take();
                // 正式的发送url
                HttpRequestUtil.sendData(url);
            } catch (Throwable e) {
                log.log(Level.WARNING, "发送url异常", e);
            }
        }
    }

    /**
     * 内部类，用户发送数据的http工具类
     *
     * @author root
     */
    public static class HttpRequestUtil {
        /**
         * 具体发送url的方法
         *
         * @param url
         * @throws IOException
         */
        public static void sendData(String url) throws IOException {
            HttpURLConnection con = null;
            BufferedReader in = null;

            try {
                URL obj = new URL(url); // 创建url对象
                con = (HttpURLConnection) obj.openConnection(); // 打开url连接
                // 设置连接参数
                con.setConnectTimeout(5000); // 连接过期时间
                con.setReadTimeout(5000); // 读取数据过期时间
                con.setRequestMethod("GET"); // 设置请求类型为get

                System.out.println("发送url:" + url);
                // 发送连接请求
                in = new BufferedReader(new InputStreamReader(
                        con.getInputStream()));
                // TODO: 这里考虑是否可以
            } finally {
                try {
                    if (in != null) {
                        in.close();
                    }
                } catch (Throwable e) {
                    // nothing
                }
                try {
                    con.disconnect();
                } catch (Throwable e) {
                    // nothing
                }
            }
        }
    }
}
```

com.duo.test.Test

```java
package com.duo.test;

import com.duo.client.AnalyticsEngineSDK;
public class Test {
	public static void main(String[] args) {
		AnalyticsEngineSDK.onChargeSuccess("orderid123", "zhangsan");
		AnalyticsEngineSDK.onChargeRefund("orderid456", "lisi");
	}
}
```

nginx_web3.conf

```properties

#user  nobody;
worker_processes  1;

#error_log  logs/error.log;
#error_log  logs/error.log  notice;
#error_log  logs/error.log  info;

#pid        logs/nginx.pid;


events {
    worker_connections  1024;
}

# load modules compiled as Dynamic Shared Object (DSO)
#
#dso {
#    load ngx_http_fastcgi_module.so;
#    load ngx_http_rewrite_module.so;
#}

http {
    include       mime.types;
    default_type  application/octet-stream;

    #log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
    #                  '$status $body_bytes_sent "$http_referer" '
    #                  '"$http_user_agent" "$http_x_forwarded_for"';

    log_format my_format '$remote_addr^A$msec^A$http_host^A$request_uri';

    #access_log  logs/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    #keepalive_timeout  0;
    keepalive_timeout  65;

    #gzip  on;

    server {
        listen       80;
        server_name  localhost;

        #charset koi8-r;

        #access_log  logs/host.access.log  main;

        location / {
            root   html;
            index  index.html index.htm;
        }

        location = /log.gif {
            default_type image/gif;
            access_log logs/custom_access.log my_format;
        }


        #error_page  404              /404.html;

        # redirect server error pages to the static page /50x.html
        #
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
    }
}
```

### 数据预处理

数据预处理（data preprocessing）是指在正式处理以前对数据进行的一些处理，保证后续正式处理的数据是格式统一、干净规则的结构化数据。现实世界中数据大体上都是不完整，不一致的脏数据，无法直接进行数据分析，或者说不利于分析。为了提高数据分析的质量和便捷性产生了数据预处理技术。数据预处理有多种方法：数据清理，数据集成，数据变换等。这些数据处理技术在正式数据分析之前使用，大大提高了后续数据分析的质量与便捷，降低实际分析所需要的时间。技术上原则来说，任何可以接受数据经过处理输出数据的语言技术都可以用来进行数据预处理。比如java、Python、shell等。本项目中通过MapReduce程序对采集到的原始日志数据进行预处理，比如数据清洗，日期格式整理，滤除不合法数据等，并且梳理成点击流模型数据。使用MapReduce的好处在于：一是java语言熟悉度高，有很多开源的工具库便于数据处理，二是MR可以进行分布式的计算，并发处理效率高。

预处理过程中有些编程小技巧需要注意：

1. 如果涉及多属性值数据传递 通常可建立与之对应的javabean携带数据传递注意要实现Hadoop序列化机制---writable接口。
2. 有意识的把javabean中toString方法重写，以\001进行分割，方便后续数据入hive映射方便。
3. 如涉及不符合本次分析的脏数据，往往采用逻辑删除，也就是自定义标记位，比如使用1或者0来表示数据是否有效，而不是直接物理删除。

WebLogBean.java

```java
package org.duo.weblog;

import org.apache.hadoop.io.Writable;

import java.io.DataInput;
import java.io.DataOutput;
import java.io.IOException;

/**
 * 对接外部数据的层，表结构定义最好跟外部数据源保持一致
 * 术语： 贴源表
 * @author
 *
 */
public class WebLogBean implements Writable {

	private boolean valid = true;// 判断数据是否合法
	private String remote_addr;// 记录客户端的ip地址
	private String remote_user;// 记录客户端用户名称,忽略属性"-"
	private String time_local;// 记录访问时间与时区
	private String request;// 记录请求的url与http协议
	private String status;// 记录请求状态；成功是200
	private String body_bytes_sent;// 记录发送给客户端文件主体内容大小
	private String http_referer;// 用来记录从那个页面链接访问过来的
	private String http_user_agent;// 记录客户浏览器的相关信息

	
	public void set(boolean valid,String remote_addr, String remote_user, String time_local, String request, String status, String body_bytes_sent, String http_referer, String http_user_agent) {
		this.valid = valid;
		this.remote_addr = remote_addr;
		this.remote_user = remote_user;
		this.time_local = time_local;
		this.request = request;
		this.status = status;
		this.body_bytes_sent = body_bytes_sent;
		this.http_referer = http_referer;
		this.http_user_agent = http_user_agent;
	}

	public String getRemote_addr() {
		return remote_addr;
	}

	public void setRemote_addr(String remote_addr) {
		this.remote_addr = remote_addr;
	}

	public String getRemote_user() {
		return remote_user;
	}

	public void setRemote_user(String remote_user) {
		this.remote_user = remote_user;
	}

	public String getTime_local() {
		return this.time_local;
	}

	public void setTime_local(String time_local) {
		this.time_local = time_local;
	}

	public String getRequest() {
		return request;
	}

	public void setRequest(String request) {
		this.request = request;
	}

	public String getStatus() {
		return status;
	}

	public void setStatus(String status) {
		this.status = status;
	}

	public String getBody_bytes_sent() {
		return body_bytes_sent;
	}

	public void setBody_bytes_sent(String body_bytes_sent) {
		this.body_bytes_sent = body_bytes_sent;
	}

	public String getHttp_referer() {
		return http_referer;
	}

	public void setHttp_referer(String http_referer) {
		this.http_referer = http_referer;
	}

	public String getHttp_user_agent() {
		return http_user_agent;
	}

	public void setHttp_user_agent(String http_user_agent) {
		this.http_user_agent = http_user_agent;
	}

	public boolean isValid() {
		return valid;
	}

	public void setValid(boolean valid) {
		this.valid = valid;
	}

	@Override
	public String toString() {
		StringBuilder sb = new StringBuilder();
		sb.append(this.valid);
		sb.append("\001").append(this.getRemote_addr());
		sb.append("\001").append(this.getRemote_user());
		sb.append("\001").append(this.getTime_local());
		sb.append("\001").append(this.getRequest());
		sb.append("\001").append(this.getStatus());
		sb.append("\001").append(this.getBody_bytes_sent());
		sb.append("\001").append(this.getHttp_referer());
		sb.append("\001").append(this.getHttp_user_agent());
		return sb.toString();
	}

	@Override
	public void readFields(DataInput in) throws IOException {
		this.valid = in.readBoolean();
		this.remote_addr = in.readUTF();
		this.remote_user = in.readUTF();
		this.time_local = in.readUTF();
		this.request = in.readUTF();
		this.status = in.readUTF();
		this.body_bytes_sent = in.readUTF();
		this.http_referer = in.readUTF();
		this.http_user_agent = in.readUTF();

	}

	@Override
	public void write(DataOutput out) throws IOException {
		out.writeBoolean(this.valid);
		out.writeUTF(null==remote_addr?"":remote_addr);
		out.writeUTF(null==remote_user?"":remote_user);
		out.writeUTF(null==time_local?"":time_local);
		out.writeUTF(null==request?"":request);
		out.writeUTF(null==status?"":status);
		out.writeUTF(null==body_bytes_sent?"":body_bytes_sent);
		out.writeUTF(null==http_referer?"":http_referer);
		out.writeUTF(null==http_user_agent?"":http_user_agent);

	}

}
```

WebLogParser.java

```java
package org.duo.weblog;

import java.text.ParseException;
import java.text.SimpleDateFormat;
import java.util.Locale;
import java.util.Set;

public class WebLogParser {

	public static SimpleDateFormat df1 = new SimpleDateFormat("dd/MMM/yyyy:HH:mm:ss", Locale.US);
	public static SimpleDateFormat df2 = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss", Locale.US);

	public static WebLogBean parser(String line) {
		WebLogBean webLogBean = new WebLogBean();
		String[] arr = line.split(" ");
		if (arr.length > 11) {
			webLogBean.setRemote_addr(arr[0]);
			webLogBean.setRemote_user(arr[1]);
			String time_local = formatDate(arr[3].substring(1));
			if(null==time_local || "".equals(time_local)) time_local="-invalid_time-";
			webLogBean.setTime_local(time_local);
			webLogBean.setRequest(arr[6]);
			webLogBean.setStatus(arr[8]);
			webLogBean.setBody_bytes_sent(arr[9]);
			webLogBean.setHttp_referer(arr[10]);

			//如果useragent元素较多，拼接useragent
			if (arr.length > 12) {
				StringBuilder sb = new StringBuilder();
				for(int i=11;i<arr.length;i++){
					sb.append(arr[i]);
				}
				webLogBean.setHttp_user_agent(sb.toString());
			} else {
				webLogBean.setHttp_user_agent(arr[11]);
			}

			if (Integer.parseInt(webLogBean.getStatus()) >= 400) {// 大于400，HTTP错误
				webLogBean.setValid(false);
			}
			
			if("-invalid_time-".equals(webLogBean.getTime_local())){
				webLogBean.setValid(false);
			}
		} else {
			webLogBean=null;
		}

		return webLogBean;
	}

	public static void filtStaticResource(WebLogBean bean, Set<String> pages) {
		if (!pages.contains(bean.getRequest())) {
			bean.setValid(false);
		}
	}
        //格式化时间方法
	public static String formatDate(String time_local) {
		try {
			return df2.format(df1.parse(time_local));
		} catch (ParseException e) {
			return null;
		}

	}

}
```

WeblogPreProcess.java

```java
package org.duo.weblog;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.NullWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.Mapper;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;

import java.io.IOException;
import java.util.HashSet;
import java.util.Set;

/**
 * 处理原始日志，过滤出真实pv请求 转换时间格式 对缺失字段填充默认值 对记录标记valid和invalid
 */

public class WeblogPreProcess {

    static class WeblogPreProcessMapper extends Mapper<LongWritable, Text, Text, NullWritable> {
        // 用来存储网站url分类数据
        Set<String> pages = new HashSet<String>();
        Text k = new Text();
        NullWritable v = NullWritable.get();

        /**
         * 从外部配置文件中加载网站的有用url分类数据 存储到maptask的内存中，用来对日志数据进行过滤
         */
        @Override
        protected void setup(Context context) throws IOException, InterruptedException {
            pages.add("/about");
            pages.add("/black-ip-list/");
            pages.add("/cassandra-clustor/");
            pages.add("/finance-rhive-repurchase/");
            pages.add("/hadoop-family-roadmap/");
            pages.add("/hadoop-hive-intro/");
            pages.add("/hadoop-zookeeper-intro/");
            pages.add("/hadoop-mahout-roadmap/");

        }

        @Override
        protected void map(LongWritable key, Text value, Context context) throws IOException, InterruptedException {

            String line = value.toString();
            WebLogBean webLogBean = WebLogParser.parser(line);
            if (webLogBean != null) {
                // 过滤js/图片/css等静态资源
                WebLogParser.filtStaticResource(webLogBean, pages);
                /* if (!webLogBean.isValid()) return; */
                k.set(webLogBean.toString());
                context.write(k, v);
            }
        }

    }

    public static void main(String[] args) throws Exception {

        String inPath = args[0];
        String outpath = args[1];

        Configuration conf = new Configuration();
        Job job = Job.getInstance(conf);

        job.setJarByClass(WeblogPreProcess.class);

        job.setMapperClass(WeblogPreProcessMapper.class);

        job.setOutputKeyClass(Text.class);
        job.setOutputValueClass(NullWritable.class);

//		 FileInputFormat.setInputPaths(job, new Path(args[0]));
//		 FileOutputFormat.setOutputPath(job, new Path(args[1]));
        FileInputFormat.setInputPaths(job, new Path(inPath));
        FileOutputFormat.setOutputPath(job, new Path(outpath));

        job.setNumReduceTasks(0);

        boolean res = job.waitForCompletion(true);
        System.exit(res ? 0 : 1);
    }
}
```

#### 点击流模型数据

点击流（Click Stream）是指用户在网站上持续访问的轨迹。注重用户浏览网站的整个流程。用户对网站的每次访问包含了一系列的点击动作行为，这些点击行为数据就构成了点击流数据（Click Stream Data），它代表了用户浏览网站的整个流程。

点击流和网站日志是两个不同的概念，点击流是从用户的角度出发，注重用户浏览网站的整个流程；而网站日志是面向整个站点，它包含了用户行为数据、服务器响应数据等众多日志信息，我们通过对网站日志的分析可以获得用户的点击流数据。

点击流模型完全是业务模型，相关概念由业务指定而来。由于大量的指标统计从点击流模型中更容易得出，所以在预处理阶段，可以使用MapReduce程序来生成点击流模型的数据。
在点击流模型中，存在着两种模型数据：PageViews、Visits。

##### pageviews

Pageviews模型数据专注于用户每次会话（session）的识别，以及每次session内访问了几步和每一步的停留时间。在网站分析中，通常把前后两条访问记录时间差在30分钟以内算成一次会话。如果超过30分钟，则把下次访问算成新的会话开始。大致步骤如下：

1. 在所有访问日志中找出该用户的所有访问记录
2. 把该用户所有访问记录按照时间正序排序
3. 计算前后两条记录时间差是否为30分钟
4. 如果小于30分钟，则是同一会话session的延续
5. 如果大于30分钟，则是下一会话session的开始
6. 用前后两条记录时间差算出上一步停留时间
7. 最后一步和只有一步的  业务默认指定页面停留时间60s

PageViewsBean.java

```java
package org.duo.weblog.pageviews;

import org.apache.hadoop.io.Writable;

import java.io.DataInput;
import java.io.DataOutput;
import java.io.IOException;

public class PageViewsBean implements Writable {

	private String session;
	private String remote_addr;
	private String timestr;
	private String request;
	private int step;
	private String staylong;
	private String referal;
	private String useragent;
	private String bytes_send;
	private String status;

	public void set(String session, String remote_addr, String useragent, String timestr, String request, int step, String staylong, String referal, String bytes_send, String status) {
		this.session = session;
		this.remote_addr = remote_addr;
		this.useragent = useragent;
		this.timestr = timestr;
		this.request = request;
		this.step = step;
		this.staylong = staylong;
		this.referal = referal;
		this.bytes_send = bytes_send;
		this.status = status;
	}

	public String getSession() {
		return session;
	}

	public void setSession(String session) {
		this.session = session;
	}

	public String getRemote_addr() {
		return remote_addr;
	}

	public void setRemote_addr(String remote_addr) {
		this.remote_addr = remote_addr;
	}

	public String getTimestr() {
		return timestr;
	}

	public void setTimestr(String timestr) {
		this.timestr = timestr;
	}

	public String getRequest() {
		return request;
	}

	public void setRequest(String request) {
		this.request = request;
	}

	public int getStep() {
		return step;
	}

	public void setStep(int step) {
		this.step = step;
	}

	public String getStaylong() {
		return staylong;
	}

	public void setStaylong(String staylong) {
		this.staylong = staylong;
	}

	public String getReferal() {
		return referal;
	}

	public void setReferal(String referal) {
		this.referal = referal;
	}

	public String getUseragent() {
		return useragent;
	}

	public void setUseragent(String useragent) {
		this.useragent = useragent;
	}

	public String getBytes_send() {
		return bytes_send;
	}

	public void setBytes_send(String bytes_send) {
		this.bytes_send = bytes_send;
	}

	public String getStatus() {
		return status;
	}

	public void setStatus(String status) {
		this.status = status;
	}

	@Override
	public void readFields(DataInput in) throws IOException {
		this.session = in.readUTF();
		this.remote_addr = in.readUTF();
		this.timestr = in.readUTF();
		this.request = in.readUTF();
		this.step = in.readInt();
		this.staylong = in.readUTF();
		this.referal = in.readUTF();
		this.useragent = in.readUTF();
		this.bytes_send = in.readUTF();
		this.status = in.readUTF();

	}

	@Override
	public void write(DataOutput out) throws IOException {
		out.writeUTF(session);
		out.writeUTF(remote_addr);
		out.writeUTF(timestr);
		out.writeUTF(request);
		out.writeInt(step);
		out.writeUTF(staylong);
		out.writeUTF(referal);
		out.writeUTF(useragent);
		out.writeUTF(bytes_send);
		out.writeUTF(status);

	}

}
```

ClickStreamPageView.java

```java
package org.duo.weblog.pageviews;

import org.apache.commons.beanutils.BeanUtils;
import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.NullWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.Mapper;
import org.apache.hadoop.mapreduce.Reducer;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;
import org.duo.weblog.WebLogBean;

import java.io.IOException;
import java.text.ParseException;
import java.text.SimpleDateFormat;
import java.util.*;


/**
 * 将清洗之后的日志梳理出点击流pageviews模型数据
 * <p>
 * 输入数据是清洗过后的结果数据
 * <p>
 * 区分出每一次会话，给每一次visit（session）增加了session-id（随机uuid）
 * 梳理出每一次会话中所访问的每个页面（请求时间，url，停留时长，以及该页面在这次session中的序号）
 * 保留referral_url，body_bytes_send，useragent
 *
 * @author
 */
public class ClickStreamPageView {

    static class ClickStreamMapper extends Mapper<LongWritable, Text, Text, WebLogBean> {

        Text k = new Text();
        WebLogBean v = new WebLogBean();

        @Override
        protected void map(LongWritable key, Text value, Context context) throws IOException, InterruptedException {

            String line = value.toString();

            String[] fields = line.split("\001");
            if (fields.length < 9) return;
            //将切分出来的各字段set到weblogbean中
            //fields[0].equals("true")
            v.set("true".equals(fields[0]) ? true : false, fields[1], fields[2], fields[3], fields[4], fields[5], fields[6], fields[7], fields[8]);
            //只有有效记录才进入后续处理
            if (v.isValid()) {
                //此处用ip地址来标识用户
                k.set(v.getRemote_addr());
                context.write(k, v);
            }
        }
    }

    static class ClickStreamReducer extends Reducer<Text, WebLogBean, NullWritable, Text> {

        Text v = new Text();

        @Override
        protected void reduce(Text key, Iterable<WebLogBean> values, Context context) throws IOException, InterruptedException {
            ArrayList<WebLogBean> beans = new ArrayList<WebLogBean>();

//			for (WebLogBean b : values) {
//				beans.add(b);
//			}

            // 先将一个用户的所有访问记录中的时间拿出来排序
            try {
                for (WebLogBean bean : values) {
                    WebLogBean webLogBean = new WebLogBean();
                    try {
                        BeanUtils.copyProperties(webLogBean, bean);
                    } catch (Exception e) {
                        e.printStackTrace();
                    }
                    beans.add(webLogBean);
                }


                //将bean按时间先后顺序排序
                Collections.sort(beans, new Comparator<WebLogBean>() {

                    @Override
                    public int compare(WebLogBean o1, WebLogBean o2) {
                        try {
                            Date d1 = toDate(o1.getTime_local());
                            Date d2 = toDate(o2.getTime_local());
                            if (d1 == null || d2 == null)
                                return 0;
                            return d1.compareTo(d2);
                        } catch (Exception e) {
                            e.printStackTrace();
                            return 0;
                        }
                    }

                });

                /**
                 * 以下逻辑为：从有序bean中分辨出各次visit，并对一次visit中所访问的page按顺序标号step
                 * 核心思想：
                 * 就是比较相邻两条记录中的时间差，如果时间差<30分钟，则该两条记录属于同一个session
                 * 否则，就属于不同的session
                 *
                 */

                int step = 1;
                String session = UUID.randomUUID().toString();
                for (int i = 0; i < beans.size(); i++) {
                    WebLogBean bean = beans.get(i);
                    // 如果仅有1条数据，则直接输出
                    if (1 == beans.size()) {

                        // 设置默认停留时长为60s
                        v.set(session + "\001" + key.toString() + "\001" + bean.getRemote_user() + "\001" + bean.getTime_local() + "\001" + bean.getRequest() + "\001" + step + "\001" + (60) + "\001" + bean.getHttp_referer() + "\001" + bean.getHttp_user_agent() + "\001" + bean.getBody_bytes_sent() + "\001"
                                + bean.getStatus());
                        context.write(NullWritable.get(), v);
                        session = UUID.randomUUID().toString();
                        break;
                    }

                    // 如果不止1条数据，则将第一条跳过不输出，遍历第二条时再输出
                    if (i == 0) {
                        continue;
                    }

                    // 求近两次时间差
                    long timeDiff = timeDiff(toDate(bean.getTime_local()), toDate(beans.get(i - 1).getTime_local()));
                    // 如果本次-上次时间差<30分钟，则输出前一次的页面访问信息

                    if (timeDiff < 30 * 60 * 1000) {

                        v.set(session + "\001" + key.toString() + "\001" + beans.get(i - 1).getRemote_user() + "\001" + beans.get(i - 1).getTime_local() + "\001" + beans.get(i - 1).getRequest() + "\001" + step + "\001" + (timeDiff / 1000) + "\001" + beans.get(i - 1).getHttp_referer() + "\001"
                                + beans.get(i - 1).getHttp_user_agent() + "\001" + beans.get(i - 1).getBody_bytes_sent() + "\001" + beans.get(i - 1).getStatus());
                        context.write(NullWritable.get(), v);
                        step++;
                    } else {

                        // 如果本次-上次时间差>30分钟，则输出前一次的页面访问信息且将step重置，以分隔为新的visit
                        v.set(session + "\001" + key.toString() + "\001" + beans.get(i - 1).getRemote_user() + "\001" + beans.get(i - 1).getTime_local() + "\001" + beans.get(i - 1).getRequest() + "\001" + (step) + "\001" + (60) + "\001" + beans.get(i - 1).getHttp_referer() + "\001"
                                + beans.get(i - 1).getHttp_user_agent() + "\001" + beans.get(i - 1).getBody_bytes_sent() + "\001" + beans.get(i - 1).getStatus());
                        context.write(NullWritable.get(), v);
                        // 输出完上一条之后，重置step编号
                        step = 1;
                        session = UUID.randomUUID().toString();
                    }

                    // 如果此次遍历的是最后一条，则将本条直接输出
                    if (i == beans.size() - 1) {
                        // 设置默认停留市场为60s
                        v.set(session + "\001" + key.toString() + "\001" + bean.getRemote_user() + "\001" + bean.getTime_local() + "\001" + bean.getRequest() + "\001" + step + "\001" + (60) + "\001" + bean.getHttp_referer() + "\001" + bean.getHttp_user_agent() + "\001" + bean.getBody_bytes_sent() + "\001" + bean.getStatus());
                        context.write(NullWritable.get(), v);
                    }
                }

            } catch (ParseException e) {
                e.printStackTrace();

            }

        }

        private String toStr(Date date) {
            SimpleDateFormat df = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss", Locale.US);
            return df.format(date);
        }

        private Date toDate(String timeStr) throws ParseException {
            SimpleDateFormat df = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss", Locale.US);
            return df.parse(timeStr);
        }

        private long timeDiff(String time1, String time2) throws ParseException {
            Date d1 = toDate(time1);
            Date d2 = toDate(time2);
            return d1.getTime() - d2.getTime();

        }

        private long timeDiff(Date time1, Date time2) throws ParseException {

            return time1.getTime() - time2.getTime();

        }

    }

    public static void main(String[] args) throws Exception {

        Configuration conf = new Configuration();
        Job job = Job.getInstance(conf);

        job.setJarByClass(ClickStreamPageView.class);

        job.setMapperClass(ClickStreamMapper.class);
        job.setReducerClass(ClickStreamReducer.class);

        job.setMapOutputKeyClass(Text.class);
        job.setMapOutputValueClass(WebLogBean.class);

        job.setOutputKeyClass(Text.class);
        job.setOutputValueClass(Text.class);

        FileInputFormat.setInputPaths(job, new Path("D:\\out\\weblog"));
        FileOutputFormat.setOutputPath(job, new Path("D:\\out\\pageviews"));

        job.waitForCompletion(true);

    }
}
```

##### visit

Visit模型专注于每次会话session内起始、结束的访问情况信息。比如用户在某一个会话session内，进入会话的起始页面和起始时间，会话结束是从哪个页面离开的，离开时间，本次session总共访问了几个页面等信息。大致步骤如下：

1. 在pageviews模型上进行梳理
2. 在每一次回收session内所有访问记录按照时间正序排序
3. 第一天的时间页面就是起始时间页面
4. 业务指定最后一条记录的时间页面作为离开时间和离开页面

VisitBean.java

```java
package org.duo.weblog.visits;

import org.apache.hadoop.io.Writable;

import java.io.DataInput;
import java.io.DataOutput;
import java.io.IOException;

public class VisitBean implements Writable {

	private String session;
	private String remote_addr;
	private String inTime;
	private String outTime;
	private String inPage;
	private String outPage;
	private String referal;
	private int pageVisits;

	public void set(String session, String remote_addr, String inTime, String outTime, String inPage, String outPage, String referal, int pageVisits) {
		this.session = session;
		this.remote_addr = remote_addr;
		this.inTime = inTime;
		this.outTime = outTime;
		this.inPage = inPage;
		this.outPage = outPage;
		this.referal = referal;
		this.pageVisits = pageVisits;
	}

	public String getSession() {
		return session;
	}

	public void setSession(String session) {
		this.session = session;
	}

	public String getRemote_addr() {
		return remote_addr;
	}

	public void setRemote_addr(String remote_addr) {
		this.remote_addr = remote_addr;
	}

	public String getInTime() {
		return inTime;
	}

	public void setInTime(String inTime) {
		this.inTime = inTime;
	}

	public String getOutTime() {
		return outTime;
	}

	public void setOutTime(String outTime) {
		this.outTime = outTime;
	}

	public String getInPage() {
		return inPage;
	}

	public void setInPage(String inPage) {
		this.inPage = inPage;
	}

	public String getOutPage() {
		return outPage;
	}

	public void setOutPage(String outPage) {
		this.outPage = outPage;
	}

	public String getReferal() {
		return referal;
	}

	public void setReferal(String referal) {
		this.referal = referal;
	}

	public int getPageVisits() {
		return pageVisits;
	}

	public void setPageVisits(int pageVisits) {
		this.pageVisits = pageVisits;
	}

	@Override
	public void readFields(DataInput in) throws IOException {
		this.session = in.readUTF();
		this.remote_addr = in.readUTF();
		this.inTime = in.readUTF();
		this.outTime = in.readUTF();
		this.inPage = in.readUTF();
		this.outPage = in.readUTF();
		this.referal = in.readUTF();
		this.pageVisits = in.readInt();

	}

	@Override
	public void write(DataOutput out) throws IOException {
		out.writeUTF(session);
		out.writeUTF(remote_addr);
		out.writeUTF(inTime);
		out.writeUTF(outTime);
		out.writeUTF(inPage);
		out.writeUTF(outPage);
		out.writeUTF(referal);
		out.writeInt(pageVisits);

	}

	@Override
	public String toString() {
		return session + "\001" + remote_addr + "\001" + inTime + "\001" + outTime + "\001" + inPage + "\001" + outPage + "\001" + referal + "\001" + pageVisits;
	}
}
```

ClickStreamVisit.java

```java
package org.duo.weblog.visits;

import org.apache.commons.beanutils.BeanUtils;
import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.NullWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.Mapper;
import org.apache.hadoop.mapreduce.Reducer;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;
import org.duo.weblog.pageviews.PageViewsBean;

import java.io.IOException;
import java.util.ArrayList;
import java.util.Collections;
import java.util.Comparator;

/**
 * 输入数据：pageviews模型结果数据
 * 从pageviews模型结果数据中进一步梳理出visit模型
 * sessionid  start-time   out-time   start-page   out-page   pagecounts  ......
 * 
 * @author
 *
 */
public class ClickStreamVisit {

	// 以session作为key，发送数据到reducer
	static class ClickStreamVisitMapper extends Mapper<LongWritable, Text, Text, PageViewsBean> {

		PageViewsBean pvBean = new PageViewsBean();
		Text k = new Text();

		@Override
		protected void map(LongWritable key, Text value, Context context) throws IOException, InterruptedException {

			String line = value.toString();
			String[] fields = line.split("\001");
			int step = Integer.parseInt(fields[5]);
			//(String session, String remote_addr, String timestr, String request, int step, String staylong, String referal, String useragent, String bytes_send, String status)
			//299d6b78-9571-4fa9-bcc2-f2567c46df3472.46.128.140-2013-09-18 07:58:50/hadoop-zookeeper-intro/160"https://www.google.com/""Mozilla/5.0"14722200
			pvBean.set(fields[0], fields[1], fields[2], fields[3],fields[4], step, fields[6], fields[7], fields[8], fields[9]);
			k.set(pvBean.getSession());
			context.write(k, pvBean);

		}

	}

	static class ClickStreamVisitReducer extends Reducer<Text, PageViewsBean, NullWritable, VisitBean> {

		@Override
		protected void reduce(Text session, Iterable<PageViewsBean> pvBeans, Context context) throws IOException, InterruptedException {

			// 将pvBeans按照step排序
			ArrayList<PageViewsBean> pvBeansList = new ArrayList<PageViewsBean>();
			for (PageViewsBean pvBean : pvBeans) {
				PageViewsBean bean = new PageViewsBean();
				try {
					BeanUtils.copyProperties(bean, pvBean);
					pvBeansList.add(bean);
				} catch (Exception e) {
					e.printStackTrace();
				}
			}

			Collections.sort(pvBeansList, new Comparator<PageViewsBean>() {

				@Override
				public int compare(PageViewsBean o1, PageViewsBean o2) {

					return o1.getStep() > o2.getStep() ? 1 : -1;
				}
			});

			// 取这次visit的首尾pageview记录，将数据放入VisitBean中
			VisitBean visitBean = new VisitBean();
			// 取visit的首记录
			visitBean.setInPage(pvBeansList.get(0).getRequest());
			visitBean.setInTime(pvBeansList.get(0).getTimestr());
			// 取visit的尾记录
			visitBean.setOutPage(pvBeansList.get(pvBeansList.size() - 1).getRequest());
			visitBean.setOutTime(pvBeansList.get(pvBeansList.size() - 1).getTimestr());
			// visit访问的页面数
			visitBean.setPageVisits(pvBeansList.size());
			// 来访者的ip
			visitBean.setRemote_addr(pvBeansList.get(0).getRemote_addr());
			// 本次visit的referal
			visitBean.setReferal(pvBeansList.get(0).getReferal());
			visitBean.setSession(session.toString());

			context.write(NullWritable.get(), visitBean);

		}

	}

	public static void main(String[] args) throws Exception {
		Configuration conf = new Configuration();
		Job job = Job.getInstance(conf);

		job.setJarByClass(ClickStreamVisit.class);

		job.setMapperClass(ClickStreamVisitMapper.class);
		job.setReducerClass(ClickStreamVisitReducer.class);

		job.setMapOutputKeyClass(Text.class);
		job.setMapOutputValueClass(PageViewsBean.class);

		job.setOutputKeyClass(NullWritable.class);
		job.setOutputValueClass(VisitBean.class);
		
		
//		FileInputFormat.setInputPaths(job, new Path(args[0]));
//		FileOutputFormat.setOutputPath(job, new Path(args[1]));
		FileInputFormat.setInputPaths(job, new Path("/test/pageviews"));
		FileOutputFormat.setOutputPath(job, new Path("/test/visitout"));
		
		boolean res = job.waitForCompletion(true);
		System.exit(res?0:1);

	}

}
```

### 数据仓库设计

#### 维度建模基本概念

维度模型是数据仓库领域大师Ralph Kimall所倡导，他的《数据仓库工具箱》，是数据仓库工程领域最流行的数仓建模经典。维度建模以分析决策的需求出发构建模型，构建的数据模型为分析需求服务，因此它重点解决用户如何更快速完成分析需求，同时还有较好的大规模复杂查询的响应性能。维度建模是专门应用于分析型数据库、数据仓库、数据集市建模的方法。数据集市可以理解为是一种"小型数据仓库"。

##### 事实表

发生在现实世界中的操作型事件，其所产生的可度量数值，存储在事实表中。从最低的粒度级别来看，事实表行对应一个度量事件，反之亦然。事实表表示对分析主题的度量。比如一次购买行为们就可以理解为是一个事实。

例如：订单表就是一个事实表，你可以理解他就是在现实中发生的一次操作型事件，每完成一个订单，就会在订单中增加一条记录。

事实表的特征：表里没有存放实际的内容，他是一堆主键的集合，这些ID分别能对应到维度表中的一条记录。事实表包含了与各维度表相关联的外键，可与维度表关联。事实表的度量通常是数值类型，且记录数会不断增加，表数据规模迅速增长。

##### 维度表

每个维度表都包含单一的主键列。维度表的主键可以作为与之关联的任何事实表的外键，当然，维度表行的描述环境应与事实表行完全对应。维度表通常比较宽，是扁平型非规范表，包含大量的低粒度的文本属性。

维度表示你要对数据进行分析时所用的一个量,比如你要分析产品销售情况, 你可以选择按类别来进行分析,或按区域来分析。这样的按..分析就构成一个维度。用户表、商家表、时间表这些都属于维度表，这些表都有一个唯一的主键，然后在表中存放了详细的数据信息。

总的说来，在数据仓库中不需要严格遵守规范化设计原则。因为数据仓库的主导功能就是面向分析，以查询为主，不涉及数据更新操作。事实表的设计是以能够正确记录历史信息为准则，维度表的设计是以能够以合适的角度来聚合主题内容为准则。

###### 星型模型

星形模式(Star Schema)是最常用的维度建模方式。星型模式是以事实表为中心，所有的维度表直接连接在事实表上，像星星一样。星形模式的维度建模由一个事实表和一组维表成，且具有以下特点：

1. 维表只和事实表关联，维表之间没有关联；
2. 每个维表主键为单列，且该主键放置在事实表中，作为两边连接的外键；
3. 以事实表为核心，维表围绕核心呈星形分布；

###### 雪花模型

雪花模式(Snowflake Schema)是对星形模式的扩展。雪花模式的维度表可以拥有其他维度表的，虽然这种模型相比星型更规范一些，但是由于这种模型不太容易理解，维护成本比较高，而且性能方面需要关联多层维表，性能也比星型模型要低。所以一般不是很常用。（维度表可以继续关联维度表 ，不利于后期维护，尽量避免演化成该种模型）

###### 星座模型

星座模式是星型模式延伸而来，星型模式是基于一张事实表的，而星座模式是基于多张事实表的，而且共享维度信息。前面介绍的两种维度建模方法都是多维表对应单事实表，但在很多时候维度空间内的事实表不止一个，而一个维表也可能被多个事实表用到。在业务发展后期，绝大部分维度建模都采用的是星座模式。（多个事实表多，个维度表，某些维度表可以共用 ，企业数仓发展中后期常见的模型）

### 数据入库

预处理完的结构化数据通常会导入到Hive数据仓库中，建立相应的库和表与之映射关联。这样后续就可以使用HiveSQL针对数据进行分析。因此这里所说的入库是把数据加进面向分析的数据仓库，而不是数据库。因项目中数据格式比较清晰简明，可以直接load进入数据仓库。实际中，入库过程有个更加专业的叫法—ETL。ETL是将业务系统的数据经过抽取、清洗转换之后加载到数据仓库的过程，目的是将企业中的分散、零乱、标准不统一的数据整合到一起，为企业的决策提供分析依据。

ETL的设计分三部分：数据抽取、数据的清洗转换、数据的加载。在设计ETL的时候我们也是从这三部分出发。数据的抽取是从各个不同的数据源抽取到ODS(Operational Data Store，操作型数据存储)中——这个过程也可以做一些数据的清洗和转换)，在抽取的过程中需要挑选不同的抽取方法，尽可能的提高ETL的运行效率。ETL三个部分中，花费时间最长的是“T”(Transform，清洗、转换)的部分，一般情况下这部分工作量是整个ETL的2/3。数据的加载一般在数据清洗完了之后直接写入DW(Data Warehousing，数据仓库)中去。

ETL工作的实质就是从各个数据源提取数据，对数据进行转换，并最终加载填充数据到数据仓库维度建模后的表中。只有当这些维度/事实表被填充好，ETL工作才算完成。

#### 创建ODS层数据表

```hive
create database if not exists weblog;
```

原始数据表：对应mr清洗完之后的数据，而不是原始日志数据

```hive
drop table if exists ods_weblog_origin;
create table ods_weblog_origin(
valid string,
remote_addr string,
remote_user string,
time_local string,
request string,
status string,
body_bytes_sent string,
http_referer string,
http_user_agent string)
partitioned by (datestr string)
row format delimited
fields terminated by '\001';
```

点击流pageview表

```hive
drop table if exists ods_click_pageviews;
create table ods_click_pageviews(
session string,
remote_addr string,
remote_user string,
time_local string,
request string,
visit_step string,
page_staylong string,
http_referer string,
http_user_agent string,
body_bytes_sent string,
status string)
partitioned by (datestr string)
row format delimited
fields terminated by '\001';
```

点击流visit表

```hive
drop table if exists ods_click_stream_visit;
create table ods_click_stream_visit(
session     string,
remote_addr string,
inTime      string,
outTime     string,
inPage      string,
outPage     string,
referal     string,
pageVisits  int)
partitioned by (datestr string)
row format delimited
fields terminated by '\001';
```

维度表

```hive
drop table if exists t_dim_time;
create table t_dim_time(date_key int,year string,month string,day string,hour string) row format delimited fields terminated by ',';
```

导入清洗结果数据到贴源数据表ods_weblog_origin

```hive
load data local inpath '/opt/weblog/preprocessed/' overwrite into table ods_weblog_origin partition(datestr='20181101');
```

导入点击流模型pageviews数据到ods_click_pageviews表

```hive
load data local inpath '/opt/weblog/pageviews' overwrite into table ods_click_pageviews partition(datestr='20181101');
```

导入点击流模型visit数据到ods_click_stream_visit表

```hive
load data local inpath '/opt/weblog/visits' overwrite into table ods_click_stream_visit partition(datestr='20181101');
```

时间维度表数据导入

```hive
load data local inpath '/opt/weblog/dim_time' overwrite into table t_dim_time;
```

#### 明细表、宽表、窄表

事实表的数据中，有些属性共同组成了一个字段（糅合在一起），比如年月日时分秒构成了时间,当需要根据某一属性进行分组统计的时候，需要截取拼接之类的操作，效率极低。为了分析方便，可以事实表中的一个字段切割提取多个属性出来构成新的字段，因为字段变多了，所以称为宽表，原来的成为窄表。又因为宽表的信息更加清晰明细，所以也可以称之为明细表。

#### 明细表（宽表）实现

```hive
drop table dw_weblog_detail;
create table dw_weblog_detail(
valid           string, --有效标识
remote_addr     string, --来源IP
remote_user     string, --用户标识
time_local      string, --访问完整时间
daystr          string, --访问日期
timestr         string, --访问时间
month           string, --访问月
day             string, --访问日
hour            string, --访问时
request         string, --请求的url
status          string, --响应码
body_bytes_sent string, --传输字节数
http_referer    string, --来源url
ref_host        string, --来源的host
ref_path        string, --来源的路径
ref_query       string, --来源参数query
ref_query_id    string, --来源参数query的值
http_user_agent string --客户终端标识
)
partitioned by(datestr string);
```

通过查询插入数据到明细宽表：dw_weblog_detail中：至于插入什么样的数据完全取决于查询语句返回的结果，因此在查询的时候就需要使用hive的函数进行字段的扩宽操作。

1. 时间字段的拓宽：substring（time_local）
2. 来源url字段拓宽：
   1. parse_url_tuple，是hive内置的udtf函数 ，可以处理标准的url格式数据，提取相关属性：host、path、query等
   2. regexp_replace，是hive内置的正则替换函数

```hive
insert into table dw_weblog_detail partition(datestr='20181101')
select c.valid,c.remote_addr,c.remote_user,c.time_local,
substring(c.time_local,0,10) as daystr,
substring(c.time_local,12) as tmstr,
substring(c.time_local,6,2) as month,
substring(c.time_local,9,2) as day,
substring(c.time_local,12,2) as hour,
c.request,c.status,c.body_bytes_sent,c.http_referer,c.ref_host,c.ref_path,c.ref_query,c.ref_query_id,c.http_user_agent
from
(SELECT 
a.valid,a.remote_addr,a.remote_user,a.time_local,
a.request,a.status,a.body_bytes_sent,a.http_referer,a.http_user_agent,b.ref_host,b.ref_path,b.ref_query,b.ref_query_id 
FROM ods_weblog_origin a LATERAL VIEW parse_url_tuple(regexp_replace(http_referer, "\"", ""), 'HOST', 'PATH','QUERY', 'QUERY:id') b as ref_host, ref_path, ref_query, ref_query_id) c;
```

### 数据分析

根据需求使用Hive SQL分析语句，得出指标各种统计结果。

#### 基础指标

1. PageView浏览次数（PV）:用户每打开1个网站页面，记录1个PV。用户多次打开同一页面PV累计多次。通俗解释就是页面被加载的总次数。

   ```hive
   select count(*) as pv from  dw_weblog_detail t where t.datestr="20181101" and t.valid = "true";
   ```

2. Unique Visitor独立访客（UV）: 1天之内，访问网站的不重复用户数（以浏览器cookie为依据），一天内同一访客多次访问网站只被计算1次。

   ```hive
   select count(distinct remote_addr) as uv from dw_weblog_detail t where t.datestr="20181101"; 
   ```

3. 访问次数（VV）：访客从进入网站到离开网站的一系列活动记为一次访问，也称会话(session),1次访问(会话)可能包含多个PV。

   ```hive
   select count(t.session) as vv from ods_click_stream_visit t where t.datestr="20181101";
   ```

4. IP：1天之内，访问网站的不重复IP数。

   ```hive
   select count(distinct remote_addr) as ip from dw_weblog_detail t where t.datestr="20181101"; 
   ```

例子：张三今天上午来到网站打开3个页面  下午来到网站打开2个页面  晚上又来到网站打开5个页面

pv:页面加载总次数     10

uv:独立访客数             1

vv:会话次数                  3

ip:                                  1

指标入库：

```hive
drop table dw_webflow_basic_info; 
create table dw_webflow_basic_info(month string,day string, pv bigint,uv bigint,ip bigint,vv bigint) partitioned by(datestr string);
```

```hive
insert into table dw_webflow_basic_info partition(datestr="20181101")
select '201811','01',a.*,b.* from
(select count(*) as pv,count(distinct remote_addr) as uv,count(distinct remote_addr) as ips 
from dw_weblog_detail
where datestr ='20181101') a join 
(select count(distinct session) as vvs from ods_click_stream_visit where datestr ="20181101") b;
```

#### 复合指标

1. 平均访问频度: 一天之内人均会话数, 总的会话次数(vv)/总的独立访客数(uv)
2. 人均浏览页数（平均访问深度）：一天之内人均浏览页面数，总的页面浏览数(pv)/总的独立访客数(uv)
3. 平均访问时长：平均每次会话停留的时间。总的会话停留时间/会话次数(vv)
4. 跳出率: 用户到网站上仅浏览了一个页面就离开的访问次数与所有访问次数的百分比

#### 多维数据分析

1. 维度：指的是看待问题的角度
2. 本质：基于多个不同的维度的聚集 计算出某种度量值（count  sum  max  mix  topN）
3. 重点：确定维度 维度就是sql层面分组的字段
4. 技巧：按xxx  每xxx   各xxx

##### 时间维度

```hive
--计算每小时的pvs
drop table dw_pvs_everyhour_oneday;
create table dw_pvs_everyhour_oneday(month string,day string,hour string,pvs bigint) partitioned by(datestr string);
insert into table dw_pvs_everyhour_oneday partition(datestr='20181101')
select a.month as month,a.day as day,a.hour as hour,count(*) as pvs from dw_weblog_detail a
where  a.datestr='20181101' group by a.month,a.day,a.hour;
--计算每天的pvs
drop table dw_pvs_everyday;
create table dw_pvs_everyday(pvs bigint,month string,day string);
insert into table dw_pvs_everyday
select count(*) as pvs,a.month as month,a.day as day from dw_weblog_detail a
group by a.month,a.day;
```

##### 来访维度

```hive
--统计每小时各来访url产生的pv量，查询结果存入：( "dw_pvs_referer_everyhour" )
drop table dw_pvs_referer_everyhour;
create table dw_pvs_referer_everyhour(referer_url string,referer_host string,month string,day string,hour string,pv_referer_cnt bigint) partitioned by(datestr string);
insert into table dw_pvs_referer_everyhour partition(datestr='20181101')
select http_referer,ref_host,month,day,hour,count(1) as pv_referer_cnt
from dw_weblog_detail 
group by http_referer,ref_host,month,day,hour 
having ref_host is not null
order by hour asc,day asc,month asc,pv_referer_cnt desc;
--统计每小时各来访host的产生的pv数并排序
drop table dw_pvs_refererhost_everyhour;
create table dw_pvs_refererhost_everyhour(ref_host string,month string,day string,hour string,ref_host_cnts bigint) partitioned by(datestr string);
insert into table dw_pvs_refererhost_everyhour partition(datestr='20181101')
select ref_host,month,day,hour,count(1) as ref_host_cnts
from dw_weblog_detail 
group by ref_host,month,day,hour 
having ref_host is not null
order by hour asc,day asc,month asc,ref_host_cnts desc;
```

##### 终端维度

数据中能够反映出用户终端信息的字段是http_user_agent。User Agent也简称UA。它是一个特殊字符串头，是一种向访问网站提供所使用的浏览器类型及版本、操作系统及版本、浏览器内核、等信息的标识。例如：

```html
User-Agent,Mozilla/5.0 (Windows NT 6.3; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/58.0.3029.276 Safari/537.36
```

上述UA信息就可以提取出以下的信息：

```markdown
chrome 58.0、浏览器	chrome、浏览器版本	58.0、系统平台	windows
浏览器内核	webkit
```

想要从网站日志数据中分析一下操作系统、浏览器、版本号使用情况。可是hive中的函数不能直接解析useragent,于是只能写一个UDF来解析。

pom.xml

```xml
引入依赖
<dependencies>
        <dependency>
            <groupId>org.apache.hive</groupId>
            <artifactId>hive-exec</artifactId>
            <version>1.2.1</version>
        </dependency>
        <dependency>
            <groupId>org.apache.hadoop</groupId>
            <artifactId>hadoop-common</artifactId>
            <version>2.7.4</version>
        </dependency>
        <dependency>
           <groupId>eu.bitwalker</groupId>
           <artifactId>UserAgentUtils</artifactId>
           <version>1.21</version>
        </dependency>
    </dependencies>
```

UAParseUDF.java

```java
package org.duo.hive.udf;

import eu.bitwalker.useragentutils.UserAgent;

public class UAParseUDF extends UDF{

    public String evaluate(final String userAgent){
        StringBuilder builder = new StringBuilder();
        UserAgent ua = new UserAgent(userAgent);
        builder.append(ua.getOperatingSystem()+"\t"+ua.getBrowser()+"\t"+ua.getBrowserVersion());
        return  (builder.toString());
    }
}
```

#### 分组topN问题

##### 分组函数(row number)

###### row number

用于在分好的组内根据排序的字段依次不重复的打上步骤号，返回构成一个新的字段。（不考虑数据重复性）

语法：row_number() over (partition by xxx order by xxx) rank，rank为分组的别名，相当于新增一个字段为rank。

###### partition by

用于指定根据什么字段分组

###### order by

用于指定分组内根据什么字段进行排序

```hive
--需求：按照时间维度，统计一天内各小时产生最多pvs的来源topN
drop table dw_pvs_refhost_topn_everyhour;
create table dw_pvs_refhost_topn_everyhour(
hour string,
toporder string,
ref_host string,
ref_host_cnts string
)partitioned by(datestr string);
insert into table dw_pvs_refhost_topn_everyhour partition(datestr='20181101')
select t.hour,t.od,t.ref_host,t.ref_host_cnts from
 (select ref_host,ref_host_cnts,concat(month,day,hour) as hour,
row_number() over (partition by concat(month,day,hour) order by ref_host_cnts desc) as od 
from dw_pvs_refererhost_everyhour) t where od<=3;
```

###### RANK,DENSE_RANK

```mark
cookie1,2018-04-10,1
cookie1,2018-04-11,5
cookie1,2018-04-12,7
cookie1,2018-04-13,3
cookie1,2018-04-14,2
cookie1,2018-04-15,4
cookie1,2018-04-16,4
cookie2,2018-04-10,2
cookie2,2018-04-11,3
cookie2,2018-04-12,5
cookie2,2018-04-13,6
cookie2,2018-04-14,3
cookie2,2018-04-15,9
cookie2,2018-04-16,7
```

```hive
CREATE TABLE orgduo_t2 (
cookieid string,
createtime string,   --day 
pv INT
) ROW FORMAT DELIMITED 
FIELDS TERMINATED BY ',' 
stored as textfile;
```

```hive
load data local inpath '/opt/orgduo_t2.dat' into table orgduo_t2;
```

```hive
SELECT 
cookieid,
createtime,
pv,
RANK() OVER(PARTITION BY cookieid ORDER BY pv desc) AS rn1,
DENSE_RANK() OVER(PARTITION BY cookieid ORDER BY pv desc) AS rn2,
ROW_NUMBER() OVER(PARTITION BY cookieid ORDER BY pv DESC) AS rn3 
FROM orgduo_t2; 
```

1. RANK() 考虑数据的重复性，重复的数据会挤占后续的标号；
2. DENSE_RANK() 考虑数据的重复性，重复的数据不会挤占后续的标号；
3. ROW_NUMBER() 不考虑数据的重复性。

###### NTILE

ntile可以看成是：把有序的数据集合平均分配到指定的数量（num）个桶中, 将桶号分配给每一行。如果不能平均分配，则优先分配较小编号的桶，并且各个桶中能放的行数最多相差1。语法是：ntile (num)  over ([partition_clause]  order_by_clause)  as xxx 然后可以根据桶号，选取前或后 n分之几的数据。

```hive
SELECT 
cookieid,
createtime,
pv,
NTILE(2) OVER(PARTITION BY cookieid ORDER BY createtime) AS rn1,
NTILE(3) OVER(PARTITION BY cookieid ORDER BY createtime) AS rn2,
NTILE(4) OVER(ORDER BY createtime) AS rn3
FROM orgduo_t2 
ORDER BY cookieid,createtime;
```

比如，统计一个cookie，pv数最多的前1/3的天

```hive
SELECT 
cookieid,
createtime,
pv,
NTILE(3) OVER(PARTITION BY cookieid ORDER BY pv DESC) AS rn 
FROM orgduo_t2;
--其中rn = 1 的记录，就是我们想要的结果
```

##### 分组函数和聚合函数配合使用

```markd
cookie1,2018-04-10,1
cookie1,2018-04-11,5
cookie1,2018-04-12,7
cookie1,2018-04-13,3
cookie1,2018-04-14,2
cookie1,2018-04-15,4
cookie1,2018-04-16,4
```

```hive
create table orgduo_t1(
cookieid string,
createtime string,   --day 
pv int
) row format delimited 
fields terminated by ',';
```

```hive
load data local inpath '/opt/orgduo_t1.dat' into table orgduo_t1;
```

###### SUM

```hive
--分组内从起点到当前行的pv累积，如，11号的pv1=10号的pv+11号的pv, 12号=10号+11号+12号
select cookieid,createtime,pv,sum(pv) over(partition by cookieid order by createtime) as pv1 from orgduo_t1;
```

```hive
--同pv1
select cookieid,createtime,pv,sum(pv) over(partition by cookieid order by createtime rows between unbounded preceding and current row) as pv2 from orgduo_t1;
```

```hive
--分组内(cookie1)所有的pv累加
select cookieid,createtime,pv,sum(pv) over(partition by cookieid) as pv3 from orgduo_t1;
```

```hive
--分组内当前行+往前3行，如，11号=10号+11号， 12号=10号+11号+12号，13号=10号+11号+12号+13号， 14号=11号+12号+13号+14号
select cookieid,createtime,pv,sum(pv) over(partition by cookieid order by createtime rows between 3 preceding and current row) as pv4 from orgduo_t1;
```

```hive
--分组内当前行+往前3行+往后1行，如，14号=11号+12号+13号+14号+15号=5+7+3+2+4=21
select cookieid,createtime,pv,sum(pv) over(partition by cookieid order by createtime rows between 3 preceding and 1 following) as pv5 from orgduo_t1;
```

```hive
--分组内当前行+往后所有行，如，13号=13号+14号+15号+16号=3+2+4+4=13，14号=14号+15号+16号=2+4+4=10
select cookieid,createtime,pv,sum(pv) over(partition by cookieid order by createtime rows between current row and unbounded following) as pv6 from orgduo_t1;
```

- 如果不指定rows between,默认为从起点到当前行;
- 如果不指定order by，则将分组内所有值累加;
- 关键是理解rows between含义,也叫做window子句：
  - preceding：往前
  - following：往后
  - current row：当前行
  - unbounded：起点
  - unbounded preceding 表示从前面的起点
  - unbounded following：表示到后面的终点

AVG，MIN，MAX，和SUM用法一样

```hive
select cookieid,createtime,pv,avg(pv) over(partition by cookieid order by createtime rows between unbounded preceding and current row) as pv2 from orgduo_t1;
```

```hive
select cookieid,createtime,pv,max(pv) over(partition by cookieid order by createtime rows between unbounded preceding and current row) as pv2 from orgduo_t1;
```

```hive
select cookieid,createtime,pv,min(pv) over(partition by cookieid order by createtime rows between unbounded preceding and current row) as pv2 from orgduo_t1;
```

##### 分组函数(CUME_DIST)

CUME_DIST和PERCENT_RANK属于序列分析函数，注意： 序列函数不支持WINDOW子句

```markdown
d1,user1,1000
d1,user2,2000
d1,user3,3000
d2,user4,4000
d2,user5,5000
```

```hive
CREATE TABLE orgduo_t3 (
dept STRING,
userid string,
sal INT
) ROW FORMAT DELIMITED 
FIELDS TERMINATED BY ',' 
stored as textfile;
```

```hive
load data local inpath '/opt/orgduo_t3.dat' into table orgduo_t3;
```

CUME_DIST和order by的排序顺序有关系：小于等于当前值的行数/分组内总行数。比如，统计小于等于当前薪水的人数，所占总人数的比例

```hive
SELECT 
dept,
userid,
sal,
CUME_DIST() OVER(ORDER BY sal) AS rn1,--没有partition语句 所有的数据位于一组
CUME_DIST() OVER(PARTITION BY dept ORDER BY sal) AS rn2 
FROM orgduo_t3;
```

```markdown
rn1: 没有partition,所有数据均为1组，总行数为5，
     第一行：小于等于1000的行数为1，因此，1/5=0.2
     第三行：小于等于3000的行数为3，因此，3/5=0.6
rn2: 按照部门分组，dpet=d1的行数为3,
     第二行：小于等于2000的行数为2，因此，2/3=0.6666666666666666
```

PERCENT_RANK：分组内当前行的RANK值-1/分组内总行数-1；经调研该函数显示现实意义不明朗，有待于继续考证

```hive
SELECT 
dept,
userid,
sal,
PERCENT_RANK() OVER(ORDER BY sal) AS rn1,   --分组内
RANK() OVER(ORDER BY sal) AS rn11,          --分组内RANK值
SUM(1) OVER(PARTITION BY NULL) AS rn12,     --分组内总行数
PERCENT_RANK() OVER(PARTITION BY dept ORDER BY sal) AS rn2 
FROM orgduo_t3;
```

```markdown
rn1: rn1 = (rn11-1) / (rn12-1) 
	   第一行,(1-1)/(5-1)=0/4=0
	   第二行,(2-1)/(5-1)=1/4=0.25
	   第四行,(4-1)/(5-1)=3/4=0.75
rn2: 按照dept分组，
     dept=d1的总行数为3
     第一行，(1-1)/(3-1)=0
     第三行，(3-1)/(3-1)=1
```

##### 分组函数(LAG)

```markdown
cookie1,2018-04-10 10:00:02,url2
cookie1,2018-04-10 10:00:00,url1
cookie1,2018-04-10 10:03:04,1url3
cookie1,2018-04-10 10:50:05,url6
cookie1,2018-04-10 11:00:00,url7
cookie1,2018-04-10 10:10:00,url4
cookie1,2018-04-10 10:50:01,url5
cookie2,2018-04-10 10:00:02,url22
cookie2,2018-04-10 10:00:00,url11
cookie2,2018-04-10 10:03:04,1url33
cookie2,2018-04-10 10:50:05,url66
cookie2,2018-04-10 11:00:00,url77
cookie2,2018-04-10 10:10:00,url44
cookie2,2018-04-10 10:50:01,url55
```

```hive
CREATE TABLE orgduo_t4 (
cookieid string,
createtime string,  --页面访问时间
url STRING       --被访问页面
) ROW FORMAT DELIMITED 
FIELDS TERMINATED BY ',' 
stored as textfile;
```

```hive
load data local inpath '/opt/orgduo_t4.dat' into table orgduo_t4;
```

LAG(col,n,DEFAULT) 用于统计窗口内往上第n行值：第一个参数为列名，第二个参数为往上第n行（可选，默认为1），第三个参数为默认值（当往上第n行为NULL时候，取默认值，如不指定，则为NULL）

```hive
SELECT cookieid,
createtime,
url,
ROW_NUMBER() OVER(PARTITION BY cookieid ORDER BY createtime) AS rn,
LAG(createtime,1,'1970-01-01 00:00:00') OVER(PARTITION BY cookieid ORDER BY createtime) AS last_1_time,
LAG(createtime,2) OVER(PARTITION BY cookieid ORDER BY createtime) AS last_2_time 
FROM orgduo_t4;
```

```markdown
last_1_time: 指定了往上第1行的值，default为'1970-01-01 00:00:00'  
             			 cookie1第一行，往上1行为NULL,因此取默认值 1970-01-01 00:00:00
             			 cookie1第三行，往上1行值为第二行值，2015-04-10 10:00:02
             			 cookie1第六行，往上1行值为第五行值，2015-04-10 10:50:01
last_2_time: 指定了往上第2行的值，为指定默认值
						 cookie1第一行，往上2行为NULL
						 cookie1第二行，往上2行为NULL
						 cookie1第四行，往上2行为第二行值，2015-04-10 10:00:02
						 cookie1第七行，往上2行为第五行值，2015-04-10 10:50:01
```

LEAD(col,n,DEFAULT) 用于统计窗口内往下第n行值：第一个参数为列名，第二个参数为往下第n行（可选，默认为1），第三个参数为默认值（当往下第n行为NULL时候，取默认值，如不指定，则为NULL）

```hive
SELECT cookieid,
createtime,
url,
ROW_NUMBER() OVER(PARTITION BY cookieid ORDER BY createtime) AS rn,
LEAD(createtime,1,'1970-01-01 00:00:00') OVER(PARTITION BY cookieid ORDER BY createtime) AS next_1_time,
LEAD(createtime,2) OVER(PARTITION BY cookieid ORDER BY createtime) AS next_2_time 
FROM orgduo_t4;
```

FIRST_VALUE：取分组内排序后，截止到当前行，第一个值

```hive
SELECT cookieid,
createtime,
url,
ROW_NUMBER() OVER(PARTITION BY cookieid ORDER BY createtime) AS rn,
FIRST_VALUE(url) OVER(PARTITION BY cookieid ORDER BY createtime) AS first1 
FROM orgduo_t4;
```

LAST_VALUE：取分组内排序后，截止到当前行，最后一个值

```hive
SELECT cookieid,
createtime,
url,
ROW_NUMBER() OVER(PARTITION BY cookieid ORDER BY createtime) AS rn,
LAST_VALUE(url) OVER(PARTITION BY cookieid ORDER BY createtime) AS last1 
FROM orgduo_t4;
```

如果想要取分组内排序后最后一个值，则需要变通一下：

```hive
SELECT cookieid,
createtime,
url,
ROW_NUMBER() OVER(PARTITION BY cookieid ORDER BY createtime) AS rn,
LAST_VALUE(url) OVER(PARTITION BY cookieid ORDER BY createtime) AS last1,
FIRST_VALUE(url) OVER(PARTITION BY cookieid ORDER BY createtime DESC) AS last2 
FROM orgduo_t4 
ORDER BY cookieid,createtime;
```

##### 分组函数(GROUPING SETS)

```mark
2018-03,2018-03-10,cookie1
2018-03,2018-03-10,cookie5
2018-03,2018-03-12,cookie7
2018-04,2018-04-12,cookie3
2018-04,2018-04-13,cookie2
2018-04,2018-04-13,cookie4
2018-04,2018-04-16,cookie4
2018-03,2018-03-10,cookie2
2018-03,2018-03-10,cookie3
2018-04,2018-04-12,cookie5
2018-04,2018-04-13,cookie6
2018-04,2018-04-15,cookie3
2018-04,2018-04-15,cookie2
2018-04,2018-04-16,cookie1
```

```hive
CREATE TABLE orgduo_t5 (
month STRING,
day STRING, 
cookieid STRING 
) ROW FORMAT DELIMITED 
FIELDS TERMINATED BY ',' 
stored as textfile;
```

```hive
load data local inpath '/opt/orgduo_t5.dat' into table orgduo_t5;
```

GROUPING SETS：grouping sets是一种将多个group by逻辑写在一个sql语句中的便利写法。等价于将不同维度的GROUP BY结果集进行UNION ALL。GROUPING__ID，表示结果属于哪一个分组集合。

```hive
SELECT 
month,
day,
COUNT(DISTINCT cookieid) AS uv,
GROUPING__ID 
FROM orgduo_t5 
GROUP BY month,day 
GROUPING SETS (month,day) 
ORDER BY GROUPING__ID;
```

grouping_id表示这一组结果属于哪个分组集合，根据grouping sets中的分组条件month，day，1是代表month，2是代表day。等价于：

```hive
SELECT month,NULL,COUNT(DISTINCT cookieid) AS uv,1 AS GROUPING__ID FROM orgduo_t5 GROUP BY month UNION ALL 
SELECT NULL as month,day,COUNT(DISTINCT cookieid) AS uv,2 AS GROUPING__ID FROM orgduo_t5 GROUP BY day;
```

再如：

```hive
SELECT 
month,
day,
COUNT(DISTINCT cookieid) AS uv,
GROUPING__ID 
FROM orgduo_t5 
GROUP BY month,day 
GROUPING SETS (month,day,(month,day)) 
ORDER BY GROUPING__ID;
```

等价于：

```hive
SELECT month,NULL,COUNT(DISTINCT cookieid) AS uv,1 AS GROUPING__ID FROM orgduo_t5 GROUP BY month 
UNION ALL 
SELECT NULL,day,COUNT(DISTINCT cookieid) AS uv,2 AS GROUPING__ID FROM orgduo_t5 GROUP BY day
UNION ALL 
SELECT month,day,COUNT(DISTINCT cookieid) AS uv,3 AS GROUPING__ID FROM orgduo_t5 GROUP BY month,day;
```

CUBE：立方体数据立方体多维数据分析；举个例子：某个事情有A、B、C三个维度，根据这三个维度进行组合分析，共有多少种情况？这些情况加起来就是所谓多维分析中数据立方体。

```markdown
没有维度：[]
一个维度：[A]  [B]  [C]
两个维度：[AB] [AC] [BC]
三个维度：[ABC]
共有8个结果。

规律：假如有n个维度 所有的维度组合情况是2的n次方
```

根据GROUP BY的维度的所有组合进行聚合。

```hive
SELECT 
month,
day,
COUNT(DISTINCT cookieid) AS uv,
GROUPING__ID 
FROM orgduo_t5 
GROUP BY month,day 
WITH CUBE 
ORDER BY GROUPING__ID;
```

等价于

```hive
SELECT NULL,NULL,COUNT(DISTINCT cookieid) AS uv,0 AS GROUPING__ID FROM orgduo_t5
UNION ALL 
SELECT month,NULL,COUNT(DISTINCT cookieid) AS uv,1 AS GROUPING__ID FROM orgduo_t5 GROUP BY month 
UNION ALL 
SELECT NULL,day,COUNT(DISTINCT cookieid) AS uv,2 AS GROUPING__ID FROM orgduo_t5 GROUP BY day
UNION ALL 
SELECT month,day,COUNT(DISTINCT cookieid) AS uv,3 AS GROUPING__ID FROM orgduo_t5 GROUP BY month,day;
```

ROLLUP：是CUBE的子集，以最左侧的维度为主，从该维度进行层级聚合。

```hive
SELECT 
month,
day,
COUNT(DISTINCT cookieid) AS uv,
GROUPING__ID  
FROM orgduo_t5 
GROUP BY month,day
WITH ROLLUP 
ORDER BY GROUPING__ID;
```

等价于

```hive
SELECT NULL,NULL,COUNT(DISTINCT cookieid) AS uv,0 AS GROUPING__ID FROM orgduo_t5
UNION ALL 
SELECT month,NULL,COUNT(DISTINCT cookieid) AS uv,1 AS GROUPING__ID FROM orgduo_t5 GROUP BY month 
UNION ALL 
SELECT month,day,COUNT(DISTINCT cookieid) AS uv,3 AS GROUPING__ID FROM orgduo_t5 GROUP BY month,day;
```

#### 热门页面统计

统计每日最热门的页面top10

```hive
drop table dw_hotpages_everyday;
create table dw_hotpages_everyday(day string,url string,pvs string);
insert into table dw_hotpages_everyday
select '20181101',a.request,a.request_counts from
(select request as request,count(request) as request_counts from dw_weblog_detail where datestr='20181101' group by request having request is not null) a
order by a.request_counts desc limit 10;
```

#### 独立访客

```hive
drop table dw_user_dstc_ip_h;
create table dw_user_dstc_ip_h(
remote_addr string,
pvs      bigint,
hour     string);
insert into table dw_user_dstc_ip_h 
select remote_addr,count(1) as pvs,concat(month,day,hour) as hour 
from dw_weblog_detail
Where datestr='20181101'
group by concat(month,day,hour),remote_addr
order by hour asc,pvs desc;
```

#### 每日新访客

```hive
--历日去重访客累积表
drop table dw_user_dsct_history;
create table dw_user_dsct_history(
day string,
ip string
) 
partitioned by(datestr string);
--每日新访客表
drop table dw_user_new_d;
create table dw_user_new_d (
day string,
ip string
) 
partitioned by(datestr string);
```

```hive
--每日新用户插入新访客表
insert into table dw_user_new_d partition(datestr='20181101')
select tmp.day as day,tmp.today_addr as new_ip from
(
select today.day as day,today.remote_addr as today_addr,old.ip as old_addr 
from 
(select distinct remote_addr as remote_addr,"20181101" as day from dw_weblog_detail where datestr="20181101") today
left outer join 
dw_user_dsct_history old
on today.remote_addr=old.ip
) tmp
where tmp.old_addr is null;
```

```hive
--每日新用户追加到累计表
insert into table dw_user_dsct_history partition(datestr='20181101')
select day,ip from dw_user_new_d where datestr='20181101';
```

#### 回头/单次访客统计

```hive
--  回头/单次访客统计
drop table dw_user_returning;
create table dw_user_returning(
day string,
remote_addr string,
acc_cnt string)
partitioned by (datestr string);
insert overwrite table dw_user_returning partition(datestr='20181101')
select tmp.day,tmp.remote_addr,tmp.acc_cnt
from
(select '20181101' as day,remote_addr,count(session) as acc_cnt from ods_click_stream_visit group by remote_addr) tmp
where tmp.acc_cnt>1;
```

#### 级联求和

第一列表示服务员，第二列表示月份，第三列表示该服务员在这个月获得的小费

```markdown
A,2015-01,5
A,2015-01,15
B,2015-01,5
A,2015-01,8
B,2015-01,25
A,2015-01,5
A,2015-02,4
A,2015-02,6
B,2015-02,10
B,2015-02,5
A,2015-03,7
A,2015-03,9
B,2015-03,11
B,2015-03,6
```

```hive
create table t_salary_detail(username string,month string,salary int) row format delimited fields terminated by ',';
```

```hive
load data local inpath '/opt/t_salary_detail.dat' into table t_salary_detail;
```

需求：统计每个用户该月累积获得多少小费？

第一步：先求个用户的月总金额

```hive
select username,month,sum(salary) as salary from t_salary_detail group by username,month;
```

```markdown
+-----------+----------+---------+-
| username  |  month   | salary  |   累计金额
+-----------+----------+---------+-
| A         | 2015-01  | 33      |    33
| A         | 2015-02  | 10      |    43
| A         | 2015-03  | 16      |    59
| B         | 2015-01  | 33      |    33
| B         | 2015-02  | 15      |    48
| B         | 2015-03  | 17      |    65
+-----------+----------+---------+--+
```

第二步：inner join自己，由于内连接会产生笛卡尔积，而只有两个表中username相等并且第一个表的月大于等于第二个表的月的数据对最终的计算结果才是有意义的，所以会加两个过滤条件。

```hive
select A.*,B.* FROM
(select username,month,sum(salary) as salary from t_salary_detail group by username,month) A 
inner join 
(select username,month,sum(salary) as salary from t_salary_detail group by username,month) B
on
A.username=B.username
where B.month <= A.month
```

```markd
+-------------+----------+-----------+-------------+----------+-----------+--+
| a.username  | a.month  | a.salary  | b.username  | b.month  | b.salary  |
+-------------+----------+-----------+-------------+----------+-----------+--+
| A           | 2015-01  | 33        | A           | 2015-01  | 33        |
| A           | 2015-02  | 10        | A           | 2015-01  | 33        |
| A           | 2015-02  | 10        | A           | 2015-02  | 10        |
| A           | 2015-03  | 16        | A           | 2015-01  | 33        |
| A           | 2015-03  | 16        | A           | 2015-02  | 10        |
| A           | 2015-03  | 16        | A           | 2015-03  | 16        |
| B           | 2015-01  | 30        | B           | 2015-01  | 30        |
| B           | 2015-02  | 15        | B           | 2015-01  | 30        |
| B           | 2015-02  | 15        | B           | 2015-02  | 15        |
| B           | 2015-03  | 17        | B           | 2015-01  | 30        |
| B           | 2015-03  | 17        | B           | 2015-02  | 15        |
| B           | 2015-03  | 17        | B           | 2015-03  | 17        |
+-------------+----------+-----------+-------------+----------+-----------+--+
```

第三步：从上一步的结果中进行分组查询，分组的字段是a.username a.month，求月累计值：将b.month <= a.month的所有b.salary求和即可

```hive
select A.username,A.month,max(A.salary) as salary,sum(B.salary) as accumulate
from 
(select username,month,sum(salary) as salary from t_salary_detail group by username,month) A 
inner join 
(select username,month,sum(salary) as salary from t_salary_detail group by username,month) B
on
A.username=B.username
where B.month <= A.month
group by A.username,A.month
order by A.username,A.month;
```

```markdown
+-------------+----------+---------+-------------+--+
| a.username  | a.month  | salary  | accumulate  |
+-------------+----------+---------+-------------+--+
| A           | 2015-01  | 33      | 33          |
| A           | 2015-02  | 10      | 43          |
| A           | 2015-03  | 16      | 59          |
| B           | 2015-01  | 30      | 30          |
| B           | 2015-02  | 15      | 45          |
| B           | 2015-03  | 17      | 62          |
+-------------+----------+---------+-------------+--+
```

#### 漏斗模型转化分析

```hive
create table dw_oute_numbs(step string,numbs int);
insert into dw_oute_numbs values ("step1",1029); 
insert into dw_oute_numbs values ("step2",1029); 
insert into dw_oute_numbs values ("step3",1028); 
insert into dw_oute_numbs values ("step4",1018); 
```

查询每一步骤相对于路径起点人数的比例

```hive
--每一步的人数/第一步的人数==每一步相对起点人数比例
select tmp.rnstep,tmp.rnnumbs/tmp.rrnumbs as ratio
from
(
select rn.step as rnstep,rn.numbs as rnnumbs,rr.step as rrstep,rr.numbs as rrnumbs  from dw_oute_numbs rn
inner join 
dw_oute_numbs rr) tmp
where tmp.rrstep='step1';
```

查询每一步骤相对于上一步骤的漏出率

```hive
--首先通过自join表过滤出每一步跟上一步的记录
select rn.step as rnstep,rn.numbs as rnnumbs,rr.step as rrstep,rr.numbs as rrnumbs  from dw_oute_numbs rn
inner join 
dw_oute_numbs rr
where cast(substr(rn.step,5,1) as int)=cast(substr(rr.step,5,1) as int)-1;
```

```hive
--然后就可以非常简单的计算出每一步相对上一步的漏出率
select tmp.rrstep as step,tmp.rrnumbs/tmp.rnnumbs as leakage_rate
from
(
select rn.step as rnstep,rn.numbs as rnnumbs,rr.step as rrstep,rr.numbs as rrnumbs  from dw_oute_numbs rn
inner join 
dw_oute_numbs rr) tmp
where cast(substr(tmp.rnstep,5,1) as int)=cast(substr(tmp.rrstep,5,1) as int)-1;
```

汇总以上两种指标

```hive
select abs.step,abs.numbs,abs.rate as abs_ratio,rel.rate as leakage_rate
from 
(
select tmp.rnstep as step,tmp.rnnumbs as numbs,tmp.rnnumbs/tmp.rrnumbs as rate
from
(
select rn.step as rnstep,rn.numbs as rnnumbs,rr.step as rrstep,rr.numbs as rrnumbs  from dw_oute_numbs rn
inner join 
dw_oute_numbs rr) tmp
where tmp.rrstep='step1'
) abs
left outer join
(
select tmp.rrstep as step,tmp.rrnumbs/tmp.rnnumbs as rate
from
(
select rn.step as rnstep,rn.numbs as rnnumbs,rr.step as rrstep,rr.numbs as rrnumbs  from dw_oute_numbs rn
inner join 
dw_oute_numbs rr) tmp
where cast(substr(tmp.rnstep,5,1) as int)=cast(substr(tmp.rrstep,5,1) as int)-1
) rel
on abs.step=rel.step;
```

### 数据导出

Sqoop支持直接从Hive表到RDBMS表的导出操作，也支持HDFS到RDBMS表的操作，鉴于此，有如下两种方案：

1. 从Hive表到RDBMS表的直接导出：效率较高，相当于直接在Hive表与RDBMS表的进行数据更新，但无法做精细的控制。
2. 从Hive到HDFS再到RDBMS表的导出：需要先将数据从Hive表导出到HDFS，再从HDFS将数据导入到RDBMS。虽然比直接导出多了一步操作，但是可以实现对数据的更精准的操作，特别是在从Hive表导出到HDFS时，可以进一步对数据进行字段筛选、字段加工、数据过滤操作，从而使得HDFS上的数据更“接近”或等于将来实际要导入RDBMS表的数据，提高导出速度。

#### 全量导出数据到mysql

Hive------->HDFS

```hive
insert overwrite directory '/weblog/export/dw_pvs_referer_everyhour' row format
delimited fields terminated by '\001' STORED AS textfile select referer_url,hour,pv_referer_cnt from dw_pvs_referer_everyhour where datestr = "20181101";
```

mysql中手动创建目标表

```mysql
create database weblog;
use weblog;
create table dw_pvs_referer_everyhour(
   hour varchar(10), 
   referer_url text, 
   pv_referer_cnt bigint);
```

HDFS-------->Mysql

```bash
sqoop export \
--connect jdbc:mysql://server01:3306/weblog \
--username root --password 123456 \
--table dw_pvs_referer_everyhour \
--fields-terminated-by '\001' \
--columns referer_url,hour,pv_referer_cnt \
--export-dir /weblog/export/dw_pvs_referer_everyhour;
```

注意：columns选项当且仅当数据文件中的数据与表结构一致时（包括字段顺序）可以不指定；否则应按照数据文件中各个字段与目标的字段的映射关系来指定该参数

#### 增量导出数据到mysql

mysql中手动创建目标表

```mysql
create table dw_webflow_basic_info(
monthstr varchar(20),
daystr varchar(10),
pv bigint,
uv bigint,
ip bigint,
vv bigint
);
```

先进行全量导出 把hive中该表的数据全部导出至mysql中

```bash
sqoop export \
--connect jdbc:mysql://server01:3306/weblog \
--username root --password 123456 \
--table dw_webflow_basic_info \
--fields-terminated-by '\001' \
--export-dir /user/hive/warehouse/weblog.db/dw_webflow_basic_info/datestr=20181101/
```

手动模拟在hive新增数据  表示增量数据

```hive
insert into table dw_webflow_basic_info partition(datestr="20181103") values("201811","03",14250,1341,1341,96);
```

利用sqoop进行增量导出

```bash
sqoop export \
--connect jdbc:mysql://server01:3306/weblog \
--username root \
--password 123456 \
--table dw_webflow_basic_info \
--fields-terminated-by '\001' \
--update-key monthstr,daystr \
--update-mode allowinsert \
--export-dir /user/hive/warehouse/weblog.db/dw_webflow_basic_info/datestr=20181103/
```

在sql中update-key用于指定更新时检查判断依据，可以多个字段，中间用,分割，如果检查的字段在hive中有更新 mysql目标表中没有，那么sqoop就会执行更新操作

#### 定时增量数据的导出操作

手动导入增量数据

```hive
insert into table dw_webflow_basic_info partition(datestr="20181104") values("201811","04",10137,1129,1129,103);
```

编写增量导出的shell脚本

```shell
#!/bin/bash
export SQOOP_HOME=/usr/local/sqoop

#判断是否传入了参数，如果传入了日期则按照传入的日期导出，如果没有传入的话则导入当前日期的前一天的数据
if [ $# -eq 1 ]
then
    execute_date=`date --date="${1}" +%Y%m%d`
else
    execute_date=`date -d'-1 day' +%Y%m%d`
fi

echo "execute_date:"${execute_date}

table_name="dw_webflow_basic_info"
hdfs_dir=/user/hive/warehouse/weblog.db/dw_webflow_basic_info/datestr=${execute_date}
mysql_db_pwd=123456
mysql_db_name=root

echo 'sqoop start'
$SQOOP_HOME/bin/sqoop export \
--connect "jdbc:mysql://server01:3306/weblog" \
--username $mysql_db_name \
--password $mysql_db_pwd \
--table $table_name \
--fields-terminated-by '\001' \
--update-key monthstr,daystr \
--update-mode allowinsert \
--export-dir $hdfs_dir

echo 'sqoop end'
```

执行脚本导出

```bash
[root@server01 weblog]# ./sqoop_export.sh 20181104
```

配合定时调度工具完成周期性定时调度

- linux crontab
- 开源azkaban  oozie

### 工作流调度

业务目标完成会包含各个不同的步骤 步骤之间或者步骤内部往往存在着依赖关系，甚至需要周期性重复性执行，这时候就需要设定工作流，指导工作按照设定的流程进行。

1. ​	简单工作流  使用linux crontab

2. ​	复杂工作流  自己开发软件 或者使用开源免费的调度软件 azkaban

#### 数据预处理定时调度

数据预处理模块按照数据处理过程和业务需求，可以分成3个步骤执行：数据预处理清洗、点击流模型之pageviews、点击流模型之visit。并且3个步骤之间存在着明显的依赖关系，使用azkaban定时周期性执行将会非常方便。

预处理的MapReduce打成3个jar：

1. weblog_preprocess.jar
2. weblog_click_pageviews.jar
3. weblog_click_visits.jar

编写azkaban调度job设置dependence依赖

weblog_preprocess.job

```bash
# weblog_preprocess.job
type=command
command=/usr/local/hadoop-2.7.5/bin/hadoop jar weblog_preprocess.jar /input /preprocess
```

weblog_click_pageviews.job

```bash
# weblog_click_pageviews.job
type=command
dependencies=weblog_preprocess
command=/usr/local/hadoop-2.7.5/bin/hadoop jar weblog_click_pageviews.jar /preprocess /pageviews
```

weblog_click_visits.job

```bash
# weblog_click_visits.job
type=command
dependencies=weblog_click_pageviews
command=/usr/local/hadoop-2.7.5/bin/hadoop jar weblog_click_visits.jar /pageviews /visit
```

#### 数据入库定时调度

load-weblog.sh

```shell
#!/bin/bash

export HIVE_HOME=/usr/local/hive

if [ $# -eq 1 ]
then
    datestr=`date --date="${1}" +%Y%m%d`
else
    datestr=`date -d'-1 day' +%Y%m%d`
fi

HQL="load data inpath '/preprocess/' into table weblog.ods_weblog_origin partition(datestr='${datestr}')"

echo "开始执行load......"
$HIVE_HOME/bin/hive -e "$HQL"
echo "执行完毕......"

```

load-weblog.job

```bash
# load-weblog.job
type=command
command=sh load-weblog.sh
```

#### 数据统计计算定时调度

1execute_hive_sql_detail.sh

```shell
#!/bin/bash
HQL="
drop table itheima.dw_user_dstc_ip_h1;
create table itheima.dw_user_dstc_ip_h1(
remote_addr string,
pvs      bigint,
hour     string);

insert into table itheima.dw_user_dstc_ip_h1 
select remote_addr,count(1) as pvs,concat(month,day,hour) as hour 
from weblog.dw_weblog_detail
Where datestr='20181101'
group by concat(month,day,hour),remote_addr;
"
echo $HQL
/usr/local/hive/bin/hive -e "$HQL"
```

1execute_hive_sql_detail.job

```bash
# 1execute_hive_sql_detail.job
type=command
command=sh 1execute_hive_sql_detail.sh
```



### 数据可视化

将分析所得数据结果进行数据可视化，一般通过图表进行展示。数据可视化可以帮你更容易的解释趋势和统计数据。

echars示例：

```html
<!DOCTYPE html>
<html>
	<head>
		<meta charset="utf-8" />
		<title></title>
		<!-- 在页面上引入echarts.js -->
		<script type="text/javascript" src="js/echarts.js"></script>
	</head>
	<body>
		<!-- 在页面创建一个dom容器 有高有宽的范围 -->
		<div id="main" style="width: 600px;height:400px;"></div>
		<script type="text/javascript">
			/* 选择容器使用echarts api创建echarts 实例 */
			var myChart = echarts.init(document.getElementById('main'));
			/* 根据业务需求去echarts官网寻找对应的图形样式  复制其option */
			var option = {
				title: {
					text: 'ECharts 入门示例'
				},
				tooltip: {},
				legend: {
					data: ['销量']
				},
				xAxis: {
					data: ["衬衫", "羊毛衫", "雪纺衫", "裤子", "高跟鞋", "袜子"]
				},
				yAxis: {},
				series: [{
					name: '销量',
					type: 'bar',
					data: [5, 20, 36, 10, 10, 20]
				}]
			};
			/* 把option设置到创建的echarts 实例中 */
			myChart.setOption(option);
		</script>
	</body>
</html>
```

# Hbase

## 基本介绍

hbase是bigtable的开源java版本。是建立在hdfs之上，提供高可靠性、高性能、列存储、可伸缩、实时读写nosql的数据库系统。它介于nosql和RDBMS之间，仅能通过主键(rowkey)和主键的range来检索数据，仅支持单行事务(可通过hive支持来实现多表join等复杂操作)。

## 基础架构

### HMaster

#### 功能

1. 监控RegionServer
2. 处理RegionServer故障转移
3. 处理元数据的变更
4. 处理region的分配或移除
5. 在空闲时间进行数据的负载均衡
6. 通过Zookeeper发布自己的位置给客户端

### RegionServer

#### 功能

1. 负责存储HBase的实际数据
2. 处理分配给它的Region
3. 刷新缓存到HDFS
4. 维护HLog
5. 执行压缩
6. 负责处理Region分片

#### 组件

1. Write-Ahead logs

   HBase的修改记录，当对HBase读写数据的时候，数据不是直接写进磁盘，它会在内存中保留一段时间（时间以及数据量阈值可以设定）。但把数据保存在内存中可能有更高的概率引起数据丢失，为了解决这个问题，数据会先写在一个叫做Write-Ahead logfile的文件中，然后再写入内存中。所以在系统出现故障的时候，数据可以通过这个日志文件重建。

2. HFile

   这是在磁盘上保存原始数据的实际的物理文件，是实际的存储文件。

3. Store

   HFile存储在Store中，一个Store对应HBase表中的一个列族。

4. MemStore

   顾名思义，就是内存存储，位于内存中，用来保存当前的数据操作，所以当数据保存在WAL中之后，RegsionServer会在内存中存储键值对。

5. Region

   Hbase表的分片，HBase表会根据RowKey值被切分成不同的region存储在RegionServer中，在一个RegionServer中可以有多个不同的region。

## 集群搭建

### 修改配置文件

hbase-env.sh

```properties
#设置JAVA_HOME
export JAVA_HOME=/usr/local/java/jdk1.8.0_144
#是否使用hbase自带的zookeeper，这里选否
export HBASE_MANAGES_ZK=false
```

hbase-site.xml

```xml
<configuration>
       <property>
                <name>hbase.rootdir</name>
                <value>hdfs://server01:8020/hbase</value>
        </property>
        <property>
                <name>hbase.cluster.distributed</name>
                <value>true</value>
        </property>
   <!-- 0.98后的新变动，之前版本没有.port,默认端口为60000 -->
        <property>
                <name>hbase.master.port</name>
                <value>16000</value>
        </property>
        <property>
                <name>hbase.zookeeper.quorum</name>
                <value>server01:2181,server02:2181,server03:2181</value>
        </property>

        <property>
                <name>hbase.zookeeper.property.dataDir</name>
                <value>/opt/zookeeper/data</value>
        </property>
</configuration>
```

regionservers

```properties
server01
server02
server03
```

创建back-masters配置文件，实现HMaster的高可用

backup-masters

```properties
server02
```

### 安装包分发到其他机器

```bash
[root@server01 local]# scp -r hbase-2.0.0/ server02:$PWD
[root@server01 local]# scp -r hbase-2.0.0/ server03:$PWD
```

### 三台机器创建软连接

因为hbase需要读取hadoop的core-site.xml以及hdfs-site.xml当中的配置文件信息，所以三台机器都要执行以下命令创建软连接

```bash
[root@server01 local]# ln -s /usr/local/hadoop-2.7.5/etc/hadoop/core-site.xml /usr/local/hbase-2.0.0/conf/core-site.xml
[root@server01 local]# ln -s /usr/local/hadoop-2.7.5/etc/hadoop/hdfs-site.xml /usr/local/hbase-2.0.0/conf/hdfs-site.xml
```

### 添加环境变量

/etc/profile

```properties
export HBASE_HOME=/usr/local/hbase-2.0.0
export PATH=$PATH:$HBASE_HOME/bin
```

### 集群启动

```bash
[root@server01 conf]# start-hbase.sh
```

另外一种启动方式：单节点进行启动

```bash
#启动HMaster命令
[root@server01 conf]# hbase-daemon.sh start master
#启动HMaster命令
[root@server01 conf]# hbase-daemon.sh start regionserver
```

### 页面访问

http://server01:16010/master-status

## hbase表模型

| rowKey | Column  Family1 userInfo     store1 store2 |      |      |          | Column  Family2 addressInfo     store3 |          |             |                                   |        |                   | Column FamilyN... |      |      |      | timeStamp                                               | versionNum     版本号 |
| ------ | ------------------------------------------ | ---- | ---- | -------- | -------------------------------------- | -------- | ----------- | --------------------------------- | ------ | ----------------- | ----------------- | ---- | ---- | ---- | ------------------------------------------------------- | --------------------- |
|        | name                                       | age  | sex  | password | address                                | from     | phone       | email                             | salary | regtime           | ...               | ...  | ...  | ...  |                                                         |                       |
| 1      | zhangsan     lisi     wangwu     zholiu    | 18   | 1    | 123456   | 地球村                                 | 火星     | 13612345678 | [666@163.com](mailto:666@163.com) | 500    | 2018-12-20 12:23  |                   |      |      |      | 1545307281     1578456985     1596325874     1598745632 | 1                     |
| 2      | 李四                                       | 28   | 1    | 123456   | 地球村                                 | 月球     | 13612345678 | [667@163.com](mailto:667@163.com) | 600    | 2018-12-21  12:23 |                   |      |      |      | 1545393681                                              | 2                     |
| 3      | 黄晓明                                     | 58   | 1    | 123456   | 地球村                                 | 土星     | 13612345678 | [668@163.com](mailto:668@163.com) | 800    | 2018-12-21  12:23 |                   |      |      |      | 1545480081                                              | 3                     |
| 4      | 按住啦baby                                 | 25   | 0    | 123456   | 地球村                                 | 韩国整容 | 13612345678 | [669@163.com](mailto:669@163.com) | 15000  | 2018-12-22  12:23 |                   |      |      |      | 1545566481                                              | 1                     |

owKey：行键，每一条数据都是使用行键来进行唯一标识的

columnFamily：列族。列族下面可以有很多列

column：列的概念。每一个列都必须归属于某一个列族

timestamp：时间戳，每条数据都会有时间戳的概念

versionNum：版本号，每条数据都会有版本号，每次数据变化，版本号都会进行更新

创建一张HBase表最少需要两个条件：表名  +  列族名

注意：rowkey是在插入数据的时候自己指定的，列名也是在插入数据的时候动态指定的，时间戳是插入数据的时候，系统自动生成的，versionNum是系统自动维护的

### Row Key

与nosql数据库们一样,row key是用来检索记录的主键。访问hbase table中的行，只有三种方式：

1. 通过单个row key访问
2. 通过row key的range
3. 全表扫描

Row key行键 (Row key)可以是任意字符串(最大长度是 64KB，实际应用中长度一般为 10-100bytes)，在hbase内部，row key保存为字节数组。Hbase会对表中的数据按照rowkey排序(字典顺序)，设计key时，要充分排序存储这个特性，将经常一起读取的行存储放到一起。(位置相关性)。行的一次读写是原子操作 (不论一次读写多少列)。

### Column Family

hbase表中的每个列，都归属与某个列族。列族是表的schema的一部分(而列不是)，必须在使用表之前定义。列名都以列族作为前缀。例如courses:history ， courses:math 都属于 courses 这个列族。访问控制、磁盘和内存的使用统计都是在列族层面进行的。列族越多，在取一行数据时所要参与IO、搜寻的文件就越多，所以，如果没有必要，不要设置太多的列族。

### Column

列族下面的具体列，属于某一个ColumnFamily,类似于mysql当中创建的具体的列。列是插入数据的时候动态指定的。

### 时间戳

HBase中通过row和columns确定的为一个存贮单元称为cell。每个cell都保存着同一份数据的多个版本。版本通过时间戳来索引。时间戳的类型是 64位整型。时间戳可以由hbase(在数据写入时自动)赋值，此时时间戳是精确到毫秒的当前系统时间。时间戳也可以由客户显式赋值。如果应用程序要避免数据版本冲突，就必须自己生成具有唯一性的时间戳。每个cell中，不同版本的数据按照时间倒序排序，即最新的数据排在最前面。

为了避免数据存在过多版本造成的的管理 (包括存贮和索引)负担，hbase提供了两种数据版本回收方式：

1. 保存数据的最后n个版本
2. 保存最近一段时间内的版本（设置数据的生命周期TTL）。

用户可以针对每个列族进行设置。

### Cell

由{row key, column( =<family> + <label>), version} 唯一确定的单元。cell中的数据是没有类型的，全部是字节码形式存贮。

### VersionNum

数据的版本号，每条数据可以有多个版本号，默认值为系统时间戳，类型为Long

## shell操作

```bash
[root@server01 conf]# hbase shell
```

查看帮助命令：help

查看数据库中有哪些表：list

### 创建表

```sql
create 'user', 'info', 'data'
#或者
create 'user', {NAME => 'info', VERSIONS => '3'}，{NAME => 'data'}
```

### 添加数据

```sql
--向user表中插入信息，row key为rk0001，列族info中添加name列标示符，值为zhangsan
put 'user', 'rk0001', 'info:name', 'zhangsan'
--向user表中插入信息，row key为rk0001，列族info中添加gender列标示符，值为female
put 'user', 'rk0001', 'info:gender', 'female'
--向user表中插入信息，row key为rk0001，列族info中添加age列标示符，值为20
put 'user', 'rk0001', 'info:age', 20
--向user表中插入信息，row key为rk0001，列族data中添加pic列标示符，值为picture
put 'user', 'rk0001', 'data:pic', 'picture'
```

### 查询数据

hbase当中数据的查询：

第一种查询方式：  get  rowkey   直接获取某一条数据

第二种查询方式  ：  scan  startRow   stopRow   范围值扫描

第三种查询方式：scan   tableName   全表扫描

```sql
--获取user表中row key为rk0001的所有信息
--获取user表中row key为rk0001的所有信息
get 'user', 'rk0001'
--查看rowkey下面的某个列族的信息
--获取user表中row key为rk0001，info列族的所有信息
get 'user', 'rk0001', 'info'
--查看rowkey指定列族指定字段的值
--获取user表中row key为rk0001，info列族的name、age列标示符的信息
get 'user', 'rk0001', 'info:name', 'info:age'
--查看rowkey指定多个列族的信息
--获取user表中row key为rk0001，info、data列族的信息
get 'user', 'rk0001', 'info', 'data'
--或者
get 'user', 'rk0001', {COLUMN => ['info', 'data']}
--或者
get 'user', 'rk0001', {COLUMN => ['info:name', 'data:pic']}
--指定rowkey与列值查询
--获取user表中row key为rk0001，cell的值为zhangsan的信息
get 'user', 'rk0001', {FILTER => "ValueFilter(=, 'binary:zhangsan')"}
--指定rowkey与列值模糊查询
--获取user表中row key为rk0001，列标示符中含有a的信息
get 'user', 'rk0001', {FILTER => "(QualifierFilter(=,'substring:a'))"}
--继续插入一批数据
put 'user', 'rk0002', 'info:name', 'fanbingbing'
put 'user', 'rk0002', 'info:gender', 'female'
put 'user', 'rk0002', 'info:nationality', '中国'
get 'user', 'rk0002', {FILTER => "ValueFilter(=, 'binary:中国')"}
--查询所有数据
--查询user表中的所有信息
scan 'user'
--列族查询
--查询user表中列族为info的信息
scan 'user', {COLUMNS => 'info'}
scan 'user', {COLUMNS => 'info', RAW => true, VERSIONS => 5}
scan 'user', {COLUMNS => 'info', RAW => true, VERSIONS => 3}
--多列族查询
--查询user表中列族为info和data的信息
scan 'user', {COLUMNS => ['info', 'data']}
scan 'user', {COLUMNS => ['info:name', 'data:pic']}
--指定列族与某个列名查询
--查询user表中列族为info、列标示符为name的信息
scan 'user', {COLUMNS => 'info:name'}
--指定列族与列名以及限定版本查询
--查询user表中列族为info、列标示符为name的信息,并且版本最新的5个
scan 'user', {COLUMNS => 'info:name', VERSIONS => 5}
--指定多个列族与按照数据值模糊查询
--查询user表中列族为info和data且列标示符中含有a字符的信息
scan 'user', {COLUMNS => ['info', 'data'], FILTER => "(QualifierFilter(=,'substring:a'))"}
--rowkey的范围值查询
--查询user表中列族为info，rk范围是[rk0001, rk0003)的数据
scan 'user', {COLUMNS => 'info', STARTROW => 'rk0001', ENDROW => 'rk0003'}
--指定rowkey模糊查询
--查询user表中row key以rk字符开头的
scan 'user',{FILTER=>"PrefixFilter('rk')"}
--指定数据范围值查询
--查询user表中指定范围的数据
scan 'user', {TIMERANGE => [1392368783980, 1392380169184]}
```

### 更新数据库

```bash
--更新数据值
--更新操作同插入操作一模一样，只不过有数据就更新，没数据就添加

--更新版本号
--将user表的f1列族版本号改为5
alter 'user', NAME => 'info', VERSIONS => 5

--删除数据以及删除表操作
--指定rowkey以及列名进行删除
--删除user表row key为rk0001，列标示符为info:name的数据
delete 'user', 'rk0001', 'info:name'
--指定rowkey，列名以及字段值进行删除
--删除user表row key为rk0001，列标示符为info:name，timestamp为1392383705316的数据
delete 'user', 'rk0001', 'info:name', 1392383705316
--删除一个列族
alter 'user', NAME => 'info', METHOD => 'delete'
--或者
alter 'user', 'delete' => 'info'
--清空表数据
truncate 'user'
--删除表
--首先需要先让该表为disable状态，使用命令：
disable 'user'
--然后才能drop这个表，使用命令：
--如果直接drop表，会报错：Drop the named table. Table must first be disabled
drop 'user'
--统计一张表有多少行数据
count 'user'
```

### 管理命令

```sql
--显示服务器状态
status 'server01'
--显示HBase当前用户
whoami
--显示当前所有的表
list
--统计指定表的记录数
count 'user'
--展示表结构信息
describe 'user'
--检查表是否存在，适用于表量特别多的情况
exists 'user'
--检查表是否启用或禁用
is_enabled 'user'
is_disabled 'user'
--为当前表增加列族
alter 'user', NAME => 'CF2', VERSIONS => 2
--为当前表删除列族
alter 'user', 'delete' => 'CF2'
--禁用一张表/启用一张表
disable 'user'
enable 'user'
--删除一张表，记得在删除表之前必须先禁用
drop 'user'
--truncate命令相当于连续执行这三条命令：禁用表-删除表-创建表，
truncate
```

## JavaAPI

pom.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>

<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>org.duo</groupId>
    <artifactId>hadoop</artifactId>
    <version>1.0-SNAPSHOT</version>

    <name>hadoop</name>
    <!-- FIXME change it to the project's website -->
    <url>http://www.example.com</url>

    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <maven.compiler.source>1.8</maven.compiler.source>
        <maven.compiler.target>1.8</maven.compiler.target>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.apache.hadoop</groupId>
            <artifactId>hadoop-common</artifactId>
            <version>2.7.5</version>
        </dependency>
        <dependency>
            <groupId>org.apache.hadoop</groupId>
            <artifactId>hadoop-client</artifactId>
            <version>2.7.5</version>
        </dependency>
        <dependency>
            <groupId>org.apache.hadoop</groupId>
            <artifactId>hadoop-hdfs</artifactId>
            <version>2.7.5</version>
        </dependency>
        <dependency>
            <groupId>org.apache.hadoop</groupId>
            <artifactId>hadoop-mapreduce-client-core</artifactId>
            <version>2.7.5</version>
        </dependency>
        <dependency>
            <groupId>org.apache.flume</groupId>
            <artifactId>flume-ng-sdk</artifactId>
            <version>1.8.0</version>
        </dependency>
        <dependency>
            <groupId>org.apache.flume</groupId>
            <artifactId>flume-ng-core</artifactId>
            <version>1.8.0</version>
        </dependency>
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>5.1.38</version>
        </dependency>
        <dependency>
            <groupId>org.apache.commons</groupId>
            <artifactId>commons-lang3</artifactId>
            <version>3.6</version>
        </dependency>
        <dependency>
            <groupId>org.apache.hive</groupId>
            <artifactId>hive-exec</artifactId>
            <version>1.2.1</version>
        </dependency>
        <dependency>
            <groupId>eu.bitwalker</groupId>
            <artifactId>UserAgentUtils</artifactId>
            <version>1.21</version>
        </dependency>
        <!-- https://mvnrepository.com/artifact/org.apache.hbase/hbase-client -->
        <!-- 2.0.0的版本maven在构建的时候会报错，有些依赖下载不下来，所以这里选择2.2.0 -->
        <dependency>
            <groupId>org.apache.hbase</groupId>
            <artifactId>hbase-client</artifactId>
            <version>2.2.0</version>
        </dependency>
        <!-- https://mvnrepository.com/artifact/org.apache.hbase/hbase-server -->
        <dependency>
            <groupId>org.apache.hbase</groupId>
            <artifactId>hbase-server</artifactId>
            <version>2.2.0</version>
        </dependency>
        <!-- https://mvnrepository.com/artifact/org.apache.hbase/hbase-mapreduce -->
        <dependency>
            <groupId>org.apache.hbase</groupId>
            <artifactId>hbase-mapreduce</artifactId>
            <version>2.2.0</version>
        </dependency>
    </dependencies>
    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.1</version>
                <configuration>
                    <source>1.8</source>
                    <target>1.8</target>
                    <encoding>UTF-8</encoding>
                    <!--    <verbal>true</verbal>-->
                </configuration>
            </plugin>
            <!--maven-jar-plugin，默认的打包插件，用来打普通的project JAR包，它只是编译src/main/java和下的java文件src/main/resources/.它不包含依赖项JAR文件.-->
            <!--maven-shade-plugin，用来打可执行JAR包，也就是所谓的fat JAR包；-->
            <!--maven-assembly-plugin，支持自定义的打包结构，也可以定制依赖项等。-->
            <!--            <plugin>
                            <groupId>org.apache.maven.plugins</groupId>
                            <artifactId>maven-jar-plugin</artifactId>
                            <version>2.4</version>
                            <configuration>
                                <archive>
                                    <manifest>
                                        <addClasspath>true</addClasspath>
                                        <classpathPrefix>lib/</classpathPrefix>
                                        <mainClass>org.duo.weblog.visits.ClickStreamVisit</mainClass>
                                    </manifest>
                                </archive>
                            </configuration>
                        </plugin>-->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-shade-plugin</artifactId>
                <version>2.4.3</version>
                <executions>
                    <execution>
                        <phase>package</phase>
                        <goals>
                            <goal>shade</goal>
                        </goals>
                        <configuration>
                            <minimizeJar>true</minimizeJar>
                        </configuration>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>
</project>
```

HBaseOperate.java

```java
package org.duo.hbase;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.hbase.Cell;
import org.apache.hadoop.hbase.CompareOperator;
import org.apache.hadoop.hbase.HBaseConfiguration;
import org.apache.hadoop.hbase.TableName;
import org.apache.hadoop.hbase.client.*;
import org.apache.hadoop.hbase.filter.*;
import org.apache.hadoop.hbase.util.Bytes;

import java.io.IOException;
import java.util.ArrayList;
import java.util.List;

public class HBaseOperate {

    private Connection connection;
    private Configuration configuration;

    private Table table;

    /**
     * 初始化的操作
     */
    public void initTable() throws IOException {

        //获取连接
        configuration = HBaseConfiguration.create();
        configuration.set("hbase.zookeeper.quorum", "server01:2181,server02:2181,server03:2181");
        connection = ConnectionFactory.createConnection(configuration);
        table = connection.getTable(TableName.valueOf("myuser"));
    }

    public void closeTable() throws IOException {
        table.close();
    }

    public static void main(String[] args) throws IOException {

        HBaseOperate hBaseOperate = new HBaseOperate();
        hBaseOperate.initTable();
        //hBaseOperate.createTable();
        // hBaseOperate.addData();
        //hBaseOperate.insertBatchData();
        //hBaseOperate.getData();
        // hBaseOperate.scanRange();
        //hBaseOperate.filterStudy();
        //hBaseOperate.hbasePage();
        //hBaseOperate.filterList();
        hBaseOperate.deleteData();
        hBaseOperate.closeTable();
    }

    /**
     * 创建hbase表 myuser，带有两个列族 f1  f2
     */
    public void createTable() throws IOException {

        //连接hbase集群
        Configuration configuration = HBaseConfiguration.create();
        //指定hbase的zk连接地址
        configuration.set("hbase.zookeeper.quorum", "server01:2181,server02:2181,server03:2181");
        Connection connection = ConnectionFactory.createConnection(configuration);
        //获取管理员对象
        Admin admin = connection.getAdmin();
        TableDescriptorBuilder myuser = TableDescriptorBuilder.newBuilder(TableName.valueOf("myuser"));
        // 给表添加列族，指定两个列族  f1   f2
        ColumnFamilyDescriptor f1 = ColumnFamilyDescriptorBuilder.of("f1");
        ColumnFamilyDescriptor f2 = ColumnFamilyDescriptorBuilder.of("f2");
        // 将两个列族设置到  myuser里面去
        myuser.setColumnFamily(f1);
        myuser.setColumnFamily(f2);
        TableDescriptor build = myuser.build();
        //创建表
        admin.createTable(build);
        admin.close();
        connection.close();
    }


    /***
     * 向表当中添加数据
     */
    public void addData() throws IOException {
        //获取连接
        Configuration configuration = HBaseConfiguration.create();
        configuration.set("hbase.zookeeper.quorum", "server01:2181,server02:2181,server03:2181");
        Connection connection = ConnectionFactory.createConnection(configuration);
        //获取表对象
        Table myuser = connection.getTable(TableName.valueOf("myuser"));
        Put put = new Put("0001".getBytes());
        put.addColumn("f1".getBytes(), "id".getBytes(), Bytes.toBytes(1));
        put.addColumn("f1".getBytes(), "name".getBytes(), Bytes.toBytes("张三"));
        put.addColumn("f1".getBytes(), "age".getBytes(), Bytes.toBytes(18));
        put.addColumn("f2".getBytes(), "address".getBytes(), Bytes.toBytes("地球人"));
        put.addColumn("f2".getBytes(), "phone".getBytes(), Bytes.toBytes("15845678952"));
        myuser.put(put);
        //关闭表
        myuser.close();

    }

    public void insertBatchData() throws IOException {

        //获取连接
        Configuration configuration = HBaseConfiguration.create();
        configuration.set("hbase.zookeeper.quorum", "server01:2181,server02:2181,server03:2181");
        Connection connection = ConnectionFactory.createConnection(configuration);
        //获取表
        Table myuser = connection.getTable(TableName.valueOf("myuser"));
        //创建put对象，并指定rowkey
        Put put = new Put("0002".getBytes());
        put.addColumn("f1".getBytes(), "id".getBytes(), Bytes.toBytes(1));
        put.addColumn("f1".getBytes(), "name".getBytes(), Bytes.toBytes("曹操"));
        put.addColumn("f1".getBytes(), "age".getBytes(), Bytes.toBytes(30));
        put.addColumn("f2".getBytes(), "sex".getBytes(), Bytes.toBytes("1"));
        put.addColumn("f2".getBytes(), "address".getBytes(), Bytes.toBytes("沛国谯县"));
        put.addColumn("f2".getBytes(), "phone".getBytes(), Bytes.toBytes("16888888888"));
        put.addColumn("f2".getBytes(), "say".getBytes(), Bytes.toBytes("helloworld"));

        Put put2 = new Put("0003".getBytes());
        put2.addColumn("f1".getBytes(), "id".getBytes(), Bytes.toBytes(2));
        put2.addColumn("f1".getBytes(), "name".getBytes(), Bytes.toBytes("刘备"));
        put2.addColumn("f1".getBytes(), "age".getBytes(), Bytes.toBytes(32));
        put2.addColumn("f2".getBytes(), "sex".getBytes(), Bytes.toBytes("1"));
        put2.addColumn("f2".getBytes(), "address".getBytes(), Bytes.toBytes("幽州涿郡涿县"));
        put2.addColumn("f2".getBytes(), "phone".getBytes(), Bytes.toBytes("17888888888"));
        put2.addColumn("f2".getBytes(), "say".getBytes(), Bytes.toBytes("talk is cheap , show me the code"));

        Put put3 = new Put("0004".getBytes());
        put3.addColumn("f1".getBytes(), "id".getBytes(), Bytes.toBytes(3));
        put3.addColumn("f1".getBytes(), "name".getBytes(), Bytes.toBytes("孙权"));
        put3.addColumn("f1".getBytes(), "age".getBytes(), Bytes.toBytes(35));
        put3.addColumn("f2".getBytes(), "sex".getBytes(), Bytes.toBytes("1"));
        put3.addColumn("f2".getBytes(), "address".getBytes(), Bytes.toBytes("下邳"));
        put3.addColumn("f2".getBytes(), "phone".getBytes(), Bytes.toBytes("12888888888"));
        put3.addColumn("f2".getBytes(), "say".getBytes(), Bytes.toBytes("what are you 弄啥嘞！"));

        Put put4 = new Put("0005".getBytes());
        put4.addColumn("f1".getBytes(), "id".getBytes(), Bytes.toBytes(4));
        put4.addColumn("f1".getBytes(), "name".getBytes(), Bytes.toBytes("诸葛亮"));
        put4.addColumn("f1".getBytes(), "age".getBytes(), Bytes.toBytes(28));
        put4.addColumn("f2".getBytes(), "sex".getBytes(), Bytes.toBytes("1"));
        put4.addColumn("f2".getBytes(), "address".getBytes(), Bytes.toBytes("四川隆中"));
        put4.addColumn("f2".getBytes(), "phone".getBytes(), Bytes.toBytes("14888888888"));
        put4.addColumn("f2".getBytes(), "say".getBytes(), Bytes.toBytes("出师表你背了嘛"));

        Put put5 = new Put("0006".getBytes());
        put5.addColumn("f1".getBytes(), "id".getBytes(), Bytes.toBytes(5));
        put5.addColumn("f1".getBytes(), "name".getBytes(), Bytes.toBytes("司马懿"));
        put5.addColumn("f1".getBytes(), "age".getBytes(), Bytes.toBytes(27));
        put5.addColumn("f2".getBytes(), "sex".getBytes(), Bytes.toBytes("1"));
        put5.addColumn("f2".getBytes(), "address".getBytes(), Bytes.toBytes("哪里人有待考究"));
        put5.addColumn("f2".getBytes(), "phone".getBytes(), Bytes.toBytes("15888888888"));
        put5.addColumn("f2".getBytes(), "say".getBytes(), Bytes.toBytes("跟诸葛亮死掐"));


        Put put6 = new Put("0007".getBytes());
        put6.addColumn("f1".getBytes(), "id".getBytes(), Bytes.toBytes(5));
        put6.addColumn("f1".getBytes(), "name".getBytes(), Bytes.toBytes("xiaobubu—吕布"));
        put6.addColumn("f1".getBytes(), "age".getBytes(), Bytes.toBytes(28));
        put6.addColumn("f2".getBytes(), "sex".getBytes(), Bytes.toBytes("1"));
        put6.addColumn("f2".getBytes(), "address".getBytes(), Bytes.toBytes("内蒙人"));
        put6.addColumn("f2".getBytes(), "phone".getBytes(), Bytes.toBytes("15788888888"));
        put6.addColumn("f2".getBytes(), "say".getBytes(), Bytes.toBytes("貂蝉去哪了"));

        List<Put> listPut = new ArrayList<Put>();
        listPut.add(put);
        listPut.add(put2);
        listPut.add(put3);
        listPut.add(put4);
        listPut.add(put5);
        listPut.add(put6);

        myuser.put(listPut);
        myuser.close();
    }

    /**
     * 查询rowkey为0003的人，所有的列
     */
    public void getData() throws IOException {
        Get get = new Get("0003".getBytes());
        get.addFamily("f1".getBytes());
//        get.addColumn("f1".getBytes(), "id".getBytes());
        //Result是一个对象，封装了我们所有的结果数据
        Result result = table.get(get);
        //获取0003这条数据所有的cell值
        List<Cell> cells = result.listCells();
        for (Cell cell : cells) {
            //获取列族的名称
            String familyName = Bytes.toString(cell.getFamilyArray(), cell.getFamilyOffset(), cell.getFamilyLength());
            //获取列的名称
            String columnName = Bytes.toString(cell.getQualifierArray(), cell.getQualifierOffset(), cell.getQualifierLength());
            if (familyName.equals("f1") && columnName.equals("id") || columnName.equals("age")) {
                int value = Bytes.toInt(cell.getValueArray(), cell.getValueOffset(), cell.getValueLength());
                System.out.println("列族名为" + familyName + "列名为" + columnName + "列的值为" + value);
            } else {
                String value = Bytes.toString(cell.getValueArray(), cell.getValueOffset(), cell.getValueLength());
                System.out.println("列族名为" + familyName + "列名为" + columnName + "列的值为" + value);
            }
        }
    }

    /**
     * 按照rowkey进行范围值的扫描
     * 扫描rowkey范围是0004到0006的所有的值
     */
    public void scanRange() throws IOException {
        Scan scan = new Scan();
        //设置我们起始和结束rowkey,范围值扫描是包括前面的，不包括后面的
        // scan.setStartRow("0004".getBytes());
        //scan.setStopRow("0006".getBytes());
        //返回多条数据结果值都封装在resultScanner里面了
        ResultScanner scanner = table.getScanner(scan);
        for (Result result : scanner) {
            List<Cell> cells = result.listCells();
            for (Cell cell : cells) {
                String rowkey = Bytes.toString(cell.getRowArray(), cell.getRowOffset(), cell.getRowLength());
                //获取列族名
                String familyName = Bytes.toString(cell.getFamilyArray(), cell.getFamilyOffset(), cell.getFamilyLength());
                String columnName = Bytes.toString(cell.getQualifierArray(), cell.getQualifierOffset(), cell.getQualifierLength());
                if (familyName.equals("f1") && columnName.equals("id") || columnName.equals("age")) {
                    int value = Bytes.toInt(cell.getValueArray(), cell.getValueOffset(), cell.getValueLength());
                    System.out.println("数据的rowkey为" + rowkey + "    数据的列族名为" + familyName + "    列名为" + columnName + "   列值为" + value);
                } else {
                    String value = Bytes.toString(cell.getValueArray(), cell.getValueOffset(), cell.getValueLength());
                    System.out.println("数据的rowkey为" + rowkey + "    数据的列族名为" + familyName + "    列名为" + columnName + "   列值为" + value);
                }
            }
        }
    }

    /**
     * 使用rowFilter查询比0003小的所有的数据
     */
    public void filterStudy() throws IOException {

        Scan scan = new Scan();

        //查询rowkey比0003小的所有的数据
        //RowFilter rowFilter = new RowFilter(CompareOperator.LESS, new BinaryComparator(Bytes.toBytes("0003")));
        //scan.setFilter(rowFilter);

        //查询比f2列族小的所有的列族里面的数据
        //FamilyFilter f2 = new FamilyFilter(CompareOperator.LESS, new SubstringComparator("f2"));
        //scan.setFilter(f2);

        //只查询name列的值
        //QualifierFilter name = new QualifierFilter(CompareOperator.EQUAL, new SubstringComparator("name"));
        //scan.setFilter(name);

        //查询value值当中包含8的所有的数据
        //ValueFilter valueFilter = new ValueFilter(CompareOperator.EQUAL, new SubstringComparator("8"));
        //scan.setFilter(valueFilter);

        //查询name值为刘备的数据，单列值过滤器 SingleColumnValueFilter，列值排除过滤器SingleColumnValueExcludeFilter；
        //与SingleColumnValueFilter相反，列值排除过滤器SingleColumnValueExcludeFilter，会排除掉指定的列，其他的列全部返回
        //SingleColumnValueFilter singleColumnValueFilter = new SingleColumnValueFilter("f1".getBytes(), "name".getBytes(), CompareOperator.EQUAL, "刘备".getBytes());
        //scan.setFilter(singleColumnValueFilter);

        //查询rowkey以00开头所有的数据
        PrefixFilter prefixFilter = new PrefixFilter("00".getBytes());
        scan.setFilter(prefixFilter);

        //返回多条数据结果值都封装在resultScanner里面了
        ResultScanner scanner = table.getScanner(scan);
        for (Result result : scanner) {
            List<Cell> cells = result.listCells();
            for (Cell cell : cells) {
                String rowkey = Bytes.toString(cell.getRowArray(), cell.getRowOffset(), cell.getRowLength());
                //获取列族名
                String familyName = Bytes.toString(cell.getFamilyArray(), cell.getFamilyOffset(), cell.getFamilyLength());
                String columnName = Bytes.toString(cell.getQualifierArray(), cell.getQualifierOffset(), cell.getQualifierLength());
                if (familyName.equals("f1") && columnName.equals("id") || columnName.equals("age")) {
                    int value = Bytes.toInt(cell.getValueArray(), cell.getValueOffset(), cell.getValueLength());
                    System.out.println("数据的rowkey为" + rowkey + "    数据的列族名为" + familyName + "    列名为" + columnName + "   列值为" + value);
                } else {
                    String value = Bytes.toString(cell.getValueArray(), cell.getValueOffset(), cell.getValueLength());
                    System.out.println("数据的rowkey为" + rowkey + "    数据的列族名为" + familyName + "    列名为" + columnName + "   列值为" + value);
                }
            }
        }
    }

    /**
     * 实现hbase的分页的功能
     */
    public void hbasePage() throws IOException {

        int pageNum = 3;
        int pageSize = 2;
        if (pageNum == 1) {

            Scan scan = new Scan();
            //如果是查询第一页数据，就按照空来进行扫描
            scan.withStartRow("".getBytes());
            PageFilter pageFilter = new PageFilter(pageSize);
            scan.setFilter(pageFilter);

            ResultScanner scanner = table.getScanner(scan);
            for (Result result : scanner) {
                byte[] row = result.getRow();
                System.out.println(Bytes.toString(row));
            }
        } else {
            String startRow = "";
            //计算我们前两页的数据的最后一条，再加上一条，就是第三页的起始rowkey
            Scan scan = new Scan();
            scan.withStartRow("".getBytes());
            PageFilter pageFilter = new PageFilter((pageNum - 1) * pageSize + 1);
            scan.setFilter(pageFilter);
            ResultScanner scanner = table.getScanner(scan);
            for (Result result : scanner) {
                byte[] row = result.getRow();
                startRow = Bytes.toString(row);
            }
            //获取第三页的数据
            scan.withStartRow(startRow.getBytes());
            PageFilter pageFilter1 = new PageFilter(pageSize);
            scan.setFilter(pageFilter1);
            ResultScanner scanner1 = table.getScanner(scan);
            for (Result result : scanner1) {
                byte[] row = result.getRow();
                System.out.println(Bytes.toString(row));
            }
        }
    }

    /**
     * 多过滤器综合查询
     * 需求：使用SingleColumnValueFilter查询f1列族，name为刘备的数据，并且同时满足rowkey的前缀以00开头的数据（PrefixFilter）
     */
    public void filterList() throws IOException {

        SingleColumnValueFilter singleColumnValueFilter = new SingleColumnValueFilter("f1".getBytes(), "name".getBytes(), CompareOperator.EQUAL, "刘备".getBytes());
        PrefixFilter prefixFilter = new PrefixFilter("00".getBytes());
        //使用filterList来实现多过滤器综合查询
        FilterList filterList = new FilterList(singleColumnValueFilter, prefixFilter);

        Scan scan = new Scan();
        scan.setFilter(filterList);
        ResultScanner scanner = table.getScanner(scan);
        for (Result result : scanner) {
            List<Cell> cells = result.listCells();
            for (Cell cell : cells) {
                String rowkey = Bytes.toString(cell.getRowArray(), cell.getRowOffset(), cell.getRowLength());
                //获取列族名
                String familyName = Bytes.toString(cell.getFamilyArray(), cell.getFamilyOffset(), cell.getFamilyLength());
                String columnName = Bytes.toString(cell.getQualifierArray(), cell.getQualifierOffset(), cell.getQualifierLength());
                if (familyName.equals("f1") && columnName.equals("id") || columnName.equals("age")) {
                    int value = Bytes.toInt(cell.getValueArray(), cell.getValueOffset(), cell.getValueLength());
                    System.out.println("数据的rowkey为" + rowkey + "    数据的列族名为" + familyName + "    列名为" + columnName + "   列值为" + value);
                } else {
                    String value = Bytes.toString(cell.getValueArray(), cell.getValueOffset(), cell.getValueLength());
                    System.out.println("数据的rowkey为" + rowkey + "    数据的列族名为" + familyName + "    列名为" + columnName + "   列值为" + value);
                }
            }
        }
    }

    /**
     * 根据rowkey删除某一条数据
     */
    public void deleteData() throws IOException {

        Delete delete = new Delete("0007".getBytes());
        table.delete(delete);

    }


    /**
     * 删除表操作
     */
    public void deleteTable() throws IOException {

        //获取管理员对象
        Admin admin = connection.getAdmin();
        //禁用表
        admin.disableTable(TableName.valueOf("myuser"));
        //删除表
        admin.deleteTable(TableName.valueOf("myuser"));
    }
}
```

## 物理存储

### 整体结构

1. Table中的所有行都按照row key的字典序排列。
2. Table 在行的方向上分割为多个Hregion。
3. region按大小分割的(默认10G)，每个表一开始只有一个region，随着数据不断插入表，region不断增大，当增大到一个阀值的时候，Hregion就会等分会两个新的Hregion。当table中的行不断增多，就会有越来越多的Hregion。
4. Hregion是Hbase中分布式存储和负载均衡的最小单元。最小单元就表示不同的Hregion可以分布在不同的HRegion server上。但一个Hregion是不会拆分到多个server上的。
5. HRegion虽然是负载均衡的最小单元，但并不是物理存储的最小单元。事实上，HRegion由一个或者多个Store组成，每个store保存一个column family。每个Strore又由一个memStore和0至多个StoreFile组成。

### STORE FILE

StoreFile以HFile格式保存在HDFS上。HFile由以下几部分组成：

#### Data Block

保存表中的数据，这部分可以被压缩。每个Data块除了开头的Magic以外就是一个个KeyValue对拼接而成, Magic内容就是一些随机数字，目的是防止数据损坏。HFile里面的每个KeyValue对就是一个简单的byte数组。但是这个byte数组里面包含了很多项，并且有固定的结构。具体结构：开始是两个固定长度的数值，分别表示Key的长度和Value的长度。紧接着是Key，开始是固定长度的数值，表示RowKey的长度，紧接着是 RowKey，然后是固定长度的数值，表示Family的长度，然后是Family，接着是Qualifier，然后是两个固定长度的数值，表示Time Stamp和Key Type（Put/Delete）。Value部分没有这么复杂的结构，就是纯粹的二进制数据了。

#### Meta Block

保存用户自定义的kv对，可以被压缩

#### File Info

记录了文件的一些Meta信息，例如：AVG_KEY_LEN, AVG_VALUE_LEN, LAST_KEY, COMPARATOR, MAX_SEQ_ID_KEY等，不被压缩，用户也可以在这一部分添加自己的元信息

#### Data Block Index

Data Block的索引。每条索引的key是被索引的block的第一条记录的key

#### Meta Block Index

Meta Block的索引

#### Trailer

这一段是定长的。保存了每一段的偏移量，读取一个HFile时，会首先 读取Trailer，Trailer保存了每个段的起始位置(段的Magic Number用来做安全check)，然后，DataBlock Index会被读取到内存中，这样，当检索某个key时，不需要扫描整个HFile，而只需从内存中找到key所在的block，通过一次磁盘io将整个 block读取到内存中，再找到需要的key。DataBlock Index采用LRU机制淘汰。

### Memstore

一个region由多个store组成，每个store包含一个列族的所有数据；Store包括位于内存的memstore和位于硬盘的storefile；写操作先写入memstore,当memstore中的数据量达到某个阈值，Hregionserver启动flashcache进程写入storefile,每次写入形成单独一个storefile；当storefile的总大小超过一定阈值后，会把当前的region分割成两个，并由Hmaster分配给相应的region服务器，实现负载均衡；客户端检索数据时，先在memstore找，找不到再找storefile。

### HLog(WAL log)

WAL 意为Write ahead log(http://en.wikipedia.org/wiki/Write-ahead_logging)，类似mysql中的binlog,用来 做灾难恢复时用，Hlog记录数据的所有变更,一旦数据修改，就可以从log中进行恢复。每个Region Server维护一个Hlog,而不是每个Region一个。这样不同region(来自不同table)的日志会混在一起，这样做的目的是不断追加单个文件相对于同时写多个文件而言，可以减少磁盘寻址次数，因此可以提高对table的写性能。带来的麻烦是，如果一台region server下线，为了恢复其上的region，需要将region server上的log进行拆分，然后分发到其它region server上进行恢复。HLog文件就是一个普通的Hadoop Sequence File：

1. HLog Sequence File 的Key是HLogKey对象，HLogKey中记录了写入数据的归属信息，除了table和region名字外，同时还包括 sequence number和timestamp，timestamp是”写入时间”，sequence number的起始值为0，或者是最近一次存入文件系统中sequence number。
2. HLog Sequece File的Value是HBase的KeyValue对象，即对应HFile中的KeyValue。

WAL的持久化等级分为如下四个等级：

1. SKIP_WAL：只写缓存，不写HLog日志。这种方式因为只写内存，因此可以极大的提升写入性能，但是数据有丢失的风险。在实际应用过程中并不建议设置此等级，除非确认不要求数据的可靠性。

2. ASYNC_WAL：异步将数据写入HLog日志中。

3. SYNC_WAL：同步将数据写入日志文件中，需要注意的是数据只是被写入文件系统中，即：PageCache，并没有真正落盘。

4. FSYNC_WAL：同步将数据写入日志文件并强制落盘。最严格的日志写入等级，可以保证数据不会丢失，但是性能相对比较差。

5. USER_DEFAULT：默认如果用户没有指定持久化等级，HBase使用SYNC_WAL等级持久化数据。

### 总结

1. 在HBase中每个表一开始只有一个region，随着数据不断插入表，region不断增大，当增大到一个阀值的时候，Hregion就会等分会两个新的Hregion。新插入的数据会存储在新的Hregion中，所以HBase底层不是列式存储，不同行的相同列不一定存储在一起。
2. Hregion由多个store组成，每个store包含一个列族的所有数据，即：同一列族中的所有列数据肯定存储在一个store中，不会存在一个列族中的不同列数据分散存储在不同store的情况；但是由于数据不断表中，会生成新的Hregion，所以会造成同一列族不同RowKey的数据存储在不同的Hregion中。

## 读写过程 

### 读请求

1. 首先通过zookeeper获取一张特殊表：meta表的位置信息，meta表中保存了包含所有创建的表的位置信息；
2. 从meta表中获取要访问的表所在的HRegionServer；
3. 访问对应的HRegionServer，然后扫描所在HRegionServer的Memstore和Storefile来查询数据。

### 写请求

1. Client也是先访问zookeeper，找到Meta表，并获取Meta表元数据，确定当前将要写入的数据所对应的HRegion和HRegionServer服务器；

2. Client向该HRegionServer服务器发起写入数据请求，然后HRegionServer收到请求并响应；

3. Client先把数据写入到HLog，以防止数据丢失；

4. 然后将数据写入到Memstore；

5. 如果HLog和Memstore均写入成功，则这条数据写入成功；

6. 如果Memstore达到阈值，会把Memstore中的数据flush到Storefile中；

7. 当Storefile越来越多，会触发Compact合并操作，把过多的Storefile合并成一个大的HFile；

8. 当HFile越来越大，Region也会越来越大，达到阈值后，会触发Split操作，将Region一分为二；

细节描述：
hbase使用MemStore和StoreFile存储对表的更新。数据在更新时首先写入Log(WAL log)和内存(MemStore)中，MemStore中的数据是排序的，当MemStore累计到一定阈值时，就会创建一个新的MemStore，并 且将老的MemStore添加到flush队列，由单独的线程flush到磁盘上，成为一个StoreFile。于此同时，系统会在zookeeper中记录一个redo point，表示这个时刻之前的变更已经持久化了。当系统出现意外时，可能导致内存(MemStore)中的数据丢失，此时使用Log(WAL log)来恢复checkpoint之后的数据。

StoreFile是只读的，一旦创建后就不可以再修改。因此Hbase的更新其实是不断追加的操作。当一个Store中的StoreFile达到一定的阈值后，就会进行一次合并(minor_compact, major_compact),将对同一个key的修改合并到一起，形成一个大的StoreFile，当StoreFile的大小达到一定阈值后，又会对 StoreFile进行split，等分为两个StoreFile。由于对表的更新是不断追加的，compact时，需要访问Store中全部的StoreFile和MemStore，将他们按rowkey进行合并，由于StoreFile和MemStore都是经过排序的，并且StoreFile带有内存中索引，合并的过程还是比较快。

hbase的数据是存储在hdfs中的，hdfs是一个适合一次写入，多次读取的文件系统，而hbase适合实时的增删改查操作，原因如下：

1. 首先，没有所谓的插入、修改和删除的区别，增删改都是带着版本号的追加操作；
2. 其次，当HLog和Memstore均写入成功，则返回写入成功，即：并没有真正写入hdfs中，而HLog是顺序写所以很快，可以理解为将一次随机写转化为了一次顺序写加一次内存写；
3. 当一个Store中的StoreFile的个数达到一定的阈值后会进行一次compact，在compact的过程中会按照rowkey进行合并。

## Region管理

### region分配

任何时刻，一个region只能分配给一个region server。master记录了当前有哪些可用的region server。以及当前哪些region分配给了哪些region server，哪些region还没有分配。当需要分配的新的region，并且有一个region server上有可用空间时，master就给这个region server发送一个装载请求，把region分配给这个region server。region server得到请求后，就开始对此region提供服务。

### region server上线

master使用zookeeper来跟踪region server状态。当某个region server启动时，会首先在zookeeper上的server目录下建立代表自己的znode。由于master订阅了server目录上的变更消息，当server目录下的文件出现新增或删除操作时，master可以得到来自zookeeper的实时通知。因此一旦region server上线，master能马上得到消息。

### region server下线

当region server下线时，它和zookeeper的会话断开，zookeeper而自动释放代表这台server的文件上的独占锁。master就可以确定：

1. region server和zookeeper之间的网络断开了。
2. region server挂了。

无论哪种情况，region server都无法继续为它的region提供服务了，此时master会删除server目录下代表这台region server的znode数据，并将这台region server的region分配给其它还活着的同志。

## Master工作机制

### master上线

master启动进行以下步骤：

1. 从zookeeper上获取唯一一个代表active master的锁，用来阻止其它master成为master。
2. 扫描zookeeper上的server父节点，获得当前可用的region server列表。
3. 和每个region server通信，获得当前已分配的region和region server的对应关系。
4. 扫描.META.region的集合，计算得到当前还未分配的region，将他们放入待分配region列表。

### master下线

由于master只维护表和region的元数据，而不参与表数据IO的过程，master下线仅导致所有元数据的修改被冻结(无法创建删除表，无法修改表的schema，无法进行region的负载均衡，无法处理region 上下线，无法进行region的合并，唯一例外的是region的split可以正常进行，因为只有region server参与)，表的数据读写还可以正常进行。因此master下线短时间内对整个hbase集群没有影响。从上线过程可以看到，master保存的信息全是可以冗余信息（都可以从系统其它地方收集到或者计算出来）因此，一般hbase集群中总是有一个master在提供服务，还有一个以上的‘master’在等待时机抢占它的位置。

## HBase三个重要机制

### flush机制

1. hbase.regionserver.global.memstore.size（默认：堆大小的40%）

   regionServer的全局memstore的大小，超过该大小会触发flush到磁盘的操作,默认是堆大小的40%,而且regionserver级别的flush会阻塞客户端读写

2. hbase.hregion.memstore.flush.size（默认：128M）

   单个region里memstore的缓存大小，超过那么整个HRegion就会flush, 

3. hbase.regionserver.optionalcacheflushinterval（默认：1h）

   内存中的文件在自动刷新之前能够存活的最长时间

4. hbase.regionserver.global.memstore.size.lower.limit（默认：堆大小 * 0.4 * 0.95）

   有时候集群的“写负载”非常高，写入量一直超过flush的量，这时，我们就希望memstore不要超过一定的安全设置。在这种情况下，写操作就要被阻塞一直到memstore恢复到一个“可管理”的大小, 这个大小就是默认值是堆大小 * 0.4 * 0.95，也就是当regionserver级别的flush操作发送后,会阻塞客户端写,一直阻塞到整个regionserver级别的memstore的大小为 堆大小 * 0.4 *0.95为止

5. hbase.hregion.preclose.flush.size（默认为：5M）

   当一个 region 中的 memstore 的大小大于这个值的时候，我们又触发 了 close.会先运行“pre-flush”操作，清理这个需要关闭的memstore，然后 将这个 region 下线。当一个 region 下线了，我们无法再进行任何写操作。 如果一个 memstore 很大的时候，flush  操作会消耗很多时间。"pre-flush" 操作意味着在 region 下线之前，会先把 memstore 清空。这样在最终执行 close 操作的时候，flush 操作会很快。

6. hbase.hstore.compactionThreshold（默认：超过3个）

   一个store里面允许存的hfile的个数，超过这个个数会被写到新的一个hfile里面 也即是每个region的每个列族对应的memstore在fulsh为hfile的时候，默认情况下当超过3个hfile的时候就会 对这些文件进行合并重写为一个新文件，设置个数越大可以减少触发合并的时间，但是每次合并的时间就会越长

### compact机制

把小的storeFile文件合并成大的Storefile文件。清理过期的数据，包括删除的数据，将数据的版本号保存为3个。

### split机制

当Region达到阈值，会把过大的Region一分为二。默认一个HFile达到10Gb的时候就会进行切分。

## HBase与MapReduce的集成

HBase当中的数据最终都是存储在HDFS上面的，HBase天生的支持MR的操作，可以通过MR直接处理HBase当中的数据，并且MR可以将处理后的结果直接存储到HBase当中去：

http://hbase.apache.org/2.0/book.html#mapreduce

### HBase Table ==> HBase Table

读取HBase当中一张表的数据，然后将数据写入到HBase当中的另外一张表当中去。注意：我们可以使用TableMapper与TableReducer来实现从HBase当中读取与写入数据

HBaseSourceMapper.java

```java
package org.duo.hbase.tableOperate;

import org.apache.hadoop.hbase.Cell;
import org.apache.hadoop.hbase.CellUtil;
import org.apache.hadoop.hbase.client.Put;
import org.apache.hadoop.hbase.client.Result;
import org.apache.hadoop.hbase.io.ImmutableBytesWritable;
import org.apache.hadoop.hbase.mapreduce.TableMapper;
import org.apache.hadoop.hbase.util.Bytes;
import org.apache.hadoop.io.Text;

import java.io.IOException;
import java.util.List;

/**
 * 负责读取myuser表当中的数据
 * 如果mapper类需要读取hbase表数据，那么我们mapper类需要继承TableMapper这样的一个类
 * 将key2   value2定义成 text  和put类型
 * text里面装rowkey
 * put装我们需要插入的数据
 */

public class HBaseSourceMapper extends TableMapper<Text, Put> {

    /**
     * @param key     rowkey
     * @param value   result对象，封装了我们一条条的数据
     * @param context 上下文对象
     * @throws IOException
     * @throws InterruptedException 需求：读取myuser表当中f1列族下面的name和age列
     */
    @Override
    protected void map(ImmutableBytesWritable key, Result value, Context context) throws IOException, InterruptedException {
        //获取到rowkey的字节数组
        byte[] bytes = key.get();
        String rowkey = Bytes.toString(bytes);

        Put put = new Put(bytes);

        //获取到所有的cell
        List<Cell> cells = value.listCells();
        for (Cell cell : cells) {
            //获取cell对应的列族
            byte[] familyBytes = CellUtil.cloneFamily(cell);
            //获取对应的列
            byte[] qualifierBytes = CellUtil.cloneQualifier(cell);
            //这里判断我们只需要f1列族，下面的name和age列
            if (Bytes.toString(familyBytes).equals("f1") && Bytes.toString(qualifierBytes).equals("name") || Bytes.toString(qualifierBytes).equals("age")) {
                put.add(cell);
            }

        }
        //将数据写出去
        if (!put.isEmpty()) {
            context.write(new Text(rowkey), put);
        }

    }
}
```

HBaseSinkReducer.java

```java
package org.duo.hbase.tableOperate;

import org.apache.hadoop.hbase.client.Put;
import org.apache.hadoop.hbase.io.ImmutableBytesWritable;
import org.apache.hadoop.hbase.mapreduce.TableReducer;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Reducer;

import java.io.IOException;

/**
 * 负责将数据写入到myuser2
 */
public class HBaseSinkReducer extends TableReducer<Text, Put, ImmutableBytesWritable> {
    @Override
    protected void reduce(Text key, Iterable<Put> values, Context context) throws IOException, InterruptedException {
        for (Put put : values) {
            context.write(new ImmutableBytesWritable(key.toString().getBytes()), put);
        }
    }
}
```

HBaseMain.java

```java
package org.duo.hbase.tableOperate;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.conf.Configured;
import org.apache.hadoop.hbase.HBaseConfiguration;
import org.apache.hadoop.hbase.client.Put;
import org.apache.hadoop.hbase.client.Scan;
import org.apache.hadoop.hbase.mapreduce.TableMapReduceUtil;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.util.Tool;
import org.apache.hadoop.util.ToolRunner;

import javax.swing.plaf.nimbus.AbstractRegionPainter;

public class HBaseMain extends Configured implements Tool {

    @Override
    public int run(String[] args) throws Exception {

        Job job = Job.getInstance(super.getConf(), "hbaseMR");
        //打包运行，必须设置main方法所在的主类
        job.setJarByClass(HBaseMain.class);

        Scan scan = new Scan();
        //定义我们的mapper类和reducer类
        /**
         * String table, Scan scan,
         Class<? extends TableMapper> mapper,
         Class<?> outputKeyClass,
         Class<?> outputValueClass, Job job,
         boolean addDependencyJars
         */
        TableMapReduceUtil.initTableMapperJob("myuser", scan, HBaseSourceMapper.class, Text.class, Put.class, job, false);
        //使用工具类初始化reducer类
        TableMapReduceUtil.initTableReducerJob("myuser2", HBaseSinkReducer.class, job);
        boolean b = job.waitForCompletion(true);
        return b ? 0 : 1;
    }

    //程序入口类
    public static void main(String[] args) throws Exception {
        //Configuration conf, Tool tool, String[] args
        Configuration configuration = HBaseConfiguration.create();
        configuration.set("hbase.zookeeper.quorum", "server01:2181,server02:2181,server03:2181");
        int run = ToolRunner.run(configuration, new HBaseMain(), args);
        System.exit(run);
    }
}
```

### HDFS ==> HBase Table

读取HDFS文件，写入到HBase表当中去

准备数据文件，并将数据文件上传到HDFS上面去

user.txt

```markdown
0007    zhangsan        18
0008    lisi    25
0009    wangwu  20
```

```bash
[root@server01 opt]# hdfs dfs -mkdir -p /hbase/input
[root@server01 opt]# hdfs dfs -put user.txt /hbase/input
# mr运行时可能会有权限问题，所以这里需要修改文件夹的访问权限
[root@server01 opt]# hdfs dfs -chmod -R 777 /hbase

```



HDFSMapper.java

```java
package org.duo.hbase.hdfsOperate;

import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.NullWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Mapper;

import java.io.IOException;

/**
 * 通过这个mapper读取hdfs上面的文件，然后进行处理
 */
public class HDFSMapper extends Mapper<LongWritable, Text, Text, NullWritable> {

    @Override
    protected void map(LongWritable key, Text value, Context context) throws IOException, InterruptedException {

        //读取到数据之后不做任何处理，直接将数据写入到reduce里面去进行处理
        context.write(value, NullWritable.get());

    }
}
```

HBaseWriteReducer.java

```java
package org.duo.hbase.hdfsOperate;

import org.apache.hadoop.hbase.client.Put;
import org.apache.hadoop.hbase.io.ImmutableBytesWritable;
import org.apache.hadoop.hbase.mapreduce.TableReducer;
import org.apache.hadoop.io.NullWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Reducer;

import java.io.IOException;

public class HBaseWriteReducer extends TableReducer<Text, NullWritable, ImmutableBytesWritable> {

    /**
     * 0007    zhangsan        18
     * 0008    lisi    25
     * 0009    wangwu  20
     *
     * @param key
     * @param values
     * @param context
     * @throws IOException
     * @throws InterruptedException
     */
    @Override
    protected void reduce(Text key, Iterable<NullWritable> values, Context context) throws IOException, InterruptedException {

        String[] split = key.toString().split("\t");
        Put put = new Put(split[0].getBytes());
        put.addColumn("f1".getBytes(), "name".getBytes(), split[1].getBytes());
        put.addColumn("f1".getBytes(), "age".getBytes(), split[2].getBytes());
        //将我们的数据写出去，key3是ImmutableBytesWritable，这个里面装的是rowkey
        //然后将写出去的数据封装到put对象里面去了
        context.write(new ImmutableBytesWritable(split[0].getBytes()), put);
    }
}
```

HdfsHBaseMain.java

```java
package org.duo.hbase.hdfsOperate;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.conf.Configured;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.hbase.HBaseConfiguration;
import org.apache.hadoop.hbase.mapreduce.TableMapReduceUtil;
import org.apache.hadoop.io.NullWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.lib.input.TextInputFormat;
import org.apache.hadoop.util.Tool;
import org.apache.hadoop.util.ToolRunner;

import javax.swing.plaf.nimbus.AbstractRegionPainter;

public class HdfsHBaseMain extends Configured implements Tool {

    @Override
    public int run(String[] args) throws Exception {
        //获取job对象
        Job job = Job.getInstance(super.getConf(), "hdfs2Hbase");

        //第一步：读取文件，解析成key，value对
        job.setInputFormatClass(TextInputFormat.class);
        TextInputFormat.addInputPath(job, new Path("hdfs://server01:8020/hbase/input"));

        //第二步：自定义map逻辑，接受k1,v1，转换成为k2  v2进行输出
        job.setMapperClass(HDFSMapper.class);
        job.setMapOutputKeyClass(Text.class);
        job.setMapOutputValueClass(NullWritable.class);

        //分区，排序，规约，分组

        //第七步：设置reduce类
        TableMapReduceUtil.initTableReducerJob("myuser2", HBaseWriteReducer.class, job);

        boolean b = job.waitForCompletion(true);

        return b ? 0 : 1;
    }

    public static void main(String[] args) throws Exception {
        Configuration configuration = HBaseConfiguration.create();
        configuration.set("hbase.zookeeper.quorum", "server01:2181,server02:2181,server03:2181");
        int run = ToolRunner.run(configuration, new HdfsHBaseMain(), args);
        System.exit(run);
    }
}
```

总结：如果读取hbase里面的数据  mapper类需要继承TableMapper；如果需要读取hdfs上面的文本文件数据，mapper类需要继承Mapper；如果要将reduce程序处理完的数据，保存到hbase里面去，reduce类，一定要继承TableReducer

### bulkload ==> HBase

通过bulkload的方式批量加载数据到HBase当中去。加载数据到HBase当中去的方式多种多样，可以使用HBase的javaAPI或者使用sqoop将我们的数据写入或者导入到HBase当中去，但是这些方式不是慢就是在导入的过程的占用Region资源导致效率低下，也可以通过MR的程序，将我们的数据直接转换成HBase的最终存储格式HFile，然后直接load数据到HBase当中去即可；HBase中每张Table在根目录（/HBase）下用一个文件夹存储，Table名为文件夹名，在Table文件夹下每个Region同样用一个文件夹存储，每个Region文件夹下的每个列族也用文件夹存储，而每个列族下存储的就是一些HFile文件，HFile就是HBase数据在HFDS下存储格式，所以HBase存储文件最终在hdfs上面的表现形式就是HFile，如果可以直接将数据转换为HFile的格式，那么HBase就可以直接读取加载HFile格式的文件，就可以直接读取了。优点：

1. 导入过程不占用Region资源

2. 能快速导入海量的数据

3. 节省内存

将hdfs上面的这个路径/hbase/input/user.txt的数据文件，转换成HFile格式，然后load到myuser2这张表里面去：

HDFSReadMapper.java

```java
package org.duo.hbase.bulkOperate;

import org.apache.hadoop.hbase.client.Put;
import org.apache.hadoop.hbase.io.ImmutableBytesWritable;
import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Mapper;

import java.io.IOException;

public class HDFSReadMapper extends Mapper<LongWritable, Text, ImmutableBytesWritable, Put> {

    /**
     * 0007    zhangsan        18
     * 0008    lisi    25
     * 0009    wangwu  20
     *
     * @param key
     * @param value
     * @param context
     * @throws IOException
     * @throws InterruptedException
     */
    @Override
    protected void map(LongWritable key, Text value, Context context) throws IOException, InterruptedException {

        String[] split = value.toString().split("\t");
        Put put = new Put(split[0].getBytes());
        put.addColumn("f1".getBytes(), "name".getBytes(), split[1].getBytes());
        put.addColumn("f1".getBytes(), "age".getBytes(), split[2].getBytes());
        context.write(new ImmutableBytesWritable(split[0].getBytes()), put);
    }
}
```

BulkLoadMain.java

```java
package org.duo.hbase.bulkOperate;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.conf.Configured;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.hbase.HBaseConfiguration;
import org.apache.hadoop.hbase.TableName;
import org.apache.hadoop.hbase.client.Connection;
import org.apache.hadoop.hbase.client.ConnectionFactory;
import org.apache.hadoop.hbase.client.Put;
import org.apache.hadoop.hbase.client.Table;
import org.apache.hadoop.hbase.io.ImmutableBytesWritable;
import org.apache.hadoop.hbase.mapreduce.HFileOutputFormat2;
import org.apache.hadoop.hdfs.DFSUtil;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.lib.input.TextInputFormat;
import org.apache.hadoop.util.Tool;
import org.apache.hadoop.util.ToolRunner;

public class BulkLoadMain extends Configured implements Tool {

    @Override
    public int run(String[] args) throws Exception {

        Configuration conf = super.getConf();

        //获取job对象
        Job job = Job.getInstance(conf, "bulkLoad");

        Connection connection = ConnectionFactory.createConnection(conf);

        Table table = connection.getTable(TableName.valueOf("myuser2"));

        //读取文件
        job.setInputFormatClass(TextInputFormat.class);
        TextInputFormat.addInputPath(job, new Path("hdfs://server01:8020/hbase/input"));

        job.setMapperClass(HDFSReadMapper.class);
        job.setMapOutputKeyClass(ImmutableBytesWritable.class);
        job.setMapOutputValueClass(Put.class);

        //将数据输出成为HFile格式

        //Job job, Table table, RegionLocator regionLocator
        //配置增量的添加数据
        HFileOutputFormat2.configureIncrementalLoad(job, table, connection.getRegionLocator(TableName.valueOf("myuser2")));
        //设置输出classs类，决定了我们输出数据格式
        job.setOutputFormatClass(HFileOutputFormat2.class);
        //设置输出路径
        HFileOutputFormat2.setOutputPath(job, new Path("hdfs://server01:8020/hbase/hfile_out"));

        boolean b = job.waitForCompletion(true);

        return b ? 0 : 1;
    }

    public static void main(String[] args) throws Exception {

        Configuration configuration = HBaseConfiguration.create();
        configuration.set("hbase.zookeeper.quorum", "server01:2181,server02:2181,server03:2181");
        int run = ToolRunner.run(configuration, new BulkLoadMain(), args);
        System.exit(run);
    }
}
```

LoadData.java

```java
package org.duo.hbase.bulkOperate;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.hbase.HBaseConfiguration;
import org.apache.hadoop.hbase.TableName;
import org.apache.hadoop.hbase.client.Admin;
import org.apache.hadoop.hbase.client.Connection;
import org.apache.hadoop.hbase.client.ConnectionFactory;
import org.apache.hadoop.hbase.client.Table;
import org.apache.hadoop.hbase.mapreduce.LoadIncrementalHFiles;

public class LoadData {

    public static void main(String[] args) throws Exception {

        Configuration configuration = HBaseConfiguration.create();
        configuration.set("hbase.zookeeper.property.clientPort", "2181");
        configuration.set("hbase.zookeeper.quorum", "server01,server02,server03");
        Connection connection = ConnectionFactory.createConnection(configuration);
        Admin admin = connection.getAdmin();
        Table table = connection.getTable(TableName.valueOf("myuser2"));
        LoadIncrementalHFiles load = new LoadIncrementalHFiles(configuration);
        load.doBulkLoad(new Path("hdfs://server01:8020/hbase/hfile_out"), admin, table, connection.getRegionLocator(TableName.valueOf("myuser2")));
    }
}
```

## HBase与Hive的对比

### Hive

Hive的本质其实就相当于将HDFS中已经存储的文件在Mysql中做了一个双射关系，以方便使用HQL去管理查询。用于数据分析、清洗，Hive适用于离线的数据分析和清洗，延迟较高。基于HDFS、MapReduce，Hive存储的数据依旧在DataNode上，编写的HQL语句终将是转换为MapReduce代码执行。

### HBase

nosql数据库，是一种面向列存储的非关系型数据库。用于存储结构化和非结构话的数据，适用于单表非关系型数据的存储，不适合做关联查询，类似JOIN等操作。基于HDFS，数据持久化存储的体现形式是Hfile，存放于DataNode中，被ResionServer以region的形式进行管理。延迟较低，接入在线业务使用，面对大量的企业数据，HBase可以直线单表大量数据的存储，同时提供了高效的数据访问速度。

### 总结

Hive和Hbase是两种基于Hadoop的不同技术，Hive是一种类SQL的引擎，并且运行MapReduce任务，Hbase是一种在Hadoop之上的NoSQL的Key/vale数据库。这两种工具是可以同时使用的。就像用Google来搜索，用FaceBook进行社交一样，Hive可以用来进行统计查询，HBase可以用来进行实时查询，数据也可以从Hive写到HBase，或者从HBase写回Hive。

## Hive与HBase的整合

Hive与HBase各有千秋，各自有着不同的功能，但是归根接地，hive与hbase的数据最终都是存储在hdfs上面的，一般为了存储磁盘的空间，不会将一份数据存储到多个地方，导致磁盘空间的浪费，可以直接将数据存入hbase，然后通过hive整合hbase直接使用sql语句分析hbase里面的数据即可，非常方便。

### 环境配置

1. 拷贝hbase的五个依赖jar包到hive的lib目录下(也可以创建软链接吗)

   ```bash
   [root@server01 opt]# ln -s /usr/local/hbase-2.0.0/lib/hbase-client-2.0.0.jar /usr/local/hive/lib/hbase-client-2.0.0.jar
   [root@server01 opt]# ln -s /usr/local/hbase-2.0.0/lib/hbase-hadoop2-compat-2.0.0.jar /usr/local/hive/lib/hbase-hadoop2-compat-2.0.0.jar
   [root@server01 opt]# ln -s /usr/local/hbase-2.0.0/lib/hbase-hadoop-compat-2.0.0.jar /usr/local/hive/lib/hbase-hadoop-compat-2.0.0.jar
   [root@server01 opt]# ln -s /usr/local/hbase-2.0.0/lib/hbase-it-2.0.0.jar /usr/local/hive/lib/hbase-it-2.0.0.jar
   [root@server01 opt]# ln -s /usr/local/hbase-2.0.0/lib/hbase-server-2.0.0.jar /usr/local/hive/lib/hbase-server-2.0.0.jar
   ```

2. 修改hive-site.xml配置文件添加以下配置

   ```xml
     <property>
       <name>hive.zookeeper.quorum</name>
       <value>server01,server02,server03</value>
     </property>
     <property>
       <name>hbase.zookeeper.quorum</name>
       <value>server01,server02,server03</value>
     </property>
   ```

3. 修改hive-env.sh配置文件添加以下配置

   ```sh
   export HBASE_HOME=/usr/local/hbase-2.0.0
   ```

### Hive ==> HBase

创建hive数据库与hive对应的数据库表

```hive
create database course;
use course;
create external table if not exists course.score(id int,cname string,score int) row format delimited fields terminated by '\t' stored as textfile ;
```

准备数据内容如下：vim hive-hbase.txt

```markdown
1       zhangsan        80
2       lisi    60
3       wangwu  30
4       zhaoliu 70
```

进入hive客户端进行加载数据

```hive
load data local inpath '/opt/hive-hbase.txt' into table score;
```

创建一个hive的管理表与hbase当中的表进行映射，hive管理表当中的数据，都会存储到hbase上面去

```hive
create table course.hbase_score(id int,cname string,score int)  stored by 'org.apache.hadoop.hive.hbase.HBaseStorageHandler'  with serdeproperties("hbase.columns.mapping" = "cf:name,cf:score") tblproperties("hbase.table.name" = "hbase_score");
```

通过insert  overwrite select插入数据

```hive
insert overwrite table course.hbase_score select id,cname,score from course.score;
```

hbase当中查看表hbase_score

```sql
scan 'hbase_score'
```

### HBase ==> Hive

HBase当中创建表并手动插入加载一些数据

```sql
create 'hbase_hive_score',{ NAME =>'cf'}
put 'hbase_hive_score','1','cf:name','zhangsan'
put 'hbase_hive_score','1','cf:score', '95'
put 'hbase_hive_score','2','cf:name','lisi'
put 'hbase_hive_score','2','cf:score', '96'
put 'hbase_hive_score','3','cf:name','wangwu'
put 'hbase_hive_score','3','cf:score', '97'
```

建立hive的外部表，映射HBase当中的表以及字段

```hive
CREATE external TABLE course.hbase2hive(id int, name string, score int) STORED BY 'org.apache.hadoop.hive.hbase.HBaseStorageHandler' WITH SERDEPROPERTIES ("hbase.columns.mapping" = ":key,cf:name,cf:score") TBLPROPERTIES("hbase.table.name" ="hbase_hive_score");
```

hive当中查看表hbase2hive

```hive
select * from hbase2hive;
```

## 预分区

HBase在创建表的时候默认不进行分区的，即：所有的region都会放在一台RegionServer上；

可通过访问：http://server01:16010/master-status ==> Table Details ==> myuser ==> 勾选：Ascending和ShowDetailName&Start/End Key然后按Reorder按钮，查看分区情况

### 优点

1. 增加数据读写效率
2. 负载均衡，防止数据倾斜
3. 方便集群容灾调度region
4. 优化Map数量

### 如何预分区

每一个region维护着startRow与endRowKey，如果加入的数据符合某个region维护的rowKey范围，则该数据交给这个region维护

#### 手动指定预分区

```sql
create 'staff','info','partition1',SPLITS => ['1000','2000','3000','4000']
```

#### 使用16进制算法生成预分区

```sql
create 'staff2','info','partition2',{NUMREGIONS => 15, SPLITALGO => 'HexStringSplit'}
```

### 使用JavaAPI创建预分区

```java
/**
 * 通过javaAPI进行HBase的表的创建以及预分区操作
 */
public void hbaseSplit() throws IOException {
    //获取连接
    Configuration configuration = HBaseConfiguration.create();
    configuration.set("hbase.zookeeper.quorum", "server01:2181,server02:2181,server03:2181");
    Connection connection = ConnectionFactory.createConnection(configuration);
    Admin admin = connection.getAdmin();
    //自定义算法，产生一系列Hash散列值存储在二维数组中
    byte[][] splitKeys = {{1,2,3,4,5},{'a','b','c','d','e'}};


    //通过HTableDescriptor来实现我们表的参数设置，包括表名，列族等等
    HTableDescriptor hTableDescriptor = new HTableDescriptor(TableName.valueOf("staff3"));
    //添加列族
    hTableDescriptor.addFamily(new HColumnDescriptor("f1"));
    //添加列族
    hTableDescriptor.addFamily(new HColumnDescriptor("f2"));
    admin.createTable(hTableDescriptor,splitKeys);
    admin.close();

}
```

## rowKey设计原则

HBase是三维有序存储的，通过rowkey（行键），column key（column family和qualifier）和TimeStamp（时间戳）这个三个维度可以对HBase中的数据进行快速定位。HBase中rowkey可以唯一标识一行记录，在HBase查询的时候，有以下几种方式：

1. 通过get方式，指定rowkey获取唯一一条记录
2. 通过scan方式，设置startRow和stopRow参数进行范围匹配
3. 全表扫描，即直接扫描整张表中所有行记录

### 长度原则

rowkey是一个二进制码流，可以是任意字符串，最大长度64kb，实际应用中一般为10-100bytes，以byte[]形式保存，一般设计成定长。建议越短越好，不要超过16个字节，原因如下：

1. 数据的持久化文件HFile中是按照KeyValue存储的，如果rowkey过长，比如超过100字节，1000w行数据，光rowkey就要占用100*1000w=10亿个字节，将近1G数据，这样会极大影响HFile的存储效率；
2. MemStore将缓存部分数据到内存，如果rowkey字段过长，内存的有效利用率就会降低，系统不能缓存更多的数据，这样会降低检索效率。

### 散列原则

如果rowkey按照时间戳的方式递增，不要将时间放在二进制码的前面，建议将rowkey的高位作为散列字段，由程序随机生成，低位放时间字段，这样将提高数据均衡分布在每个RegionServer，以实现负载均衡的几率。如果没有散列字段，首字段直接是时间信息，所有的数据都会集中在一个RegionServer上，这样在数据检索的时候负载会集中在个别的RegionServer上，造成热点问题，会降低查询效率。

### 唯一原则

必须在设计上保证其唯一性，rowkey是按照字典顺序排序存储的，因此，设计rowkey的时候，要充分利用这个排序的特点，将经常读取的数据存储到一块，将最近可能会被访问的数据放到一块。

### 什么是热点

HBase中的行是按照rowkey的字典顺序排序的，这种设计优化了scan操作，可以将相关的行以及会被一起读取的行存取在临近位置，便于scan。然而糟糕的rowkey设计是热点的源头。 热点发生在大量的client直接访问集群的一个或极少数个节点（访问可能是读，写或者其他操作）。大量访问会使热点region所在的单个机器超出自身承受能力，引起性能下降甚至region不可用，这也会影响同一个RegionServer上的其他region，由于主机无法服务其他region的请求。 设计良好的数据访问模式以使集群被充分，均衡的利用。为了避免写热点，设计rowkey使得不同行在同一个region，但是在更多数据情况下，数据应该被写入集群的多个region，而不是一个。下面是一些常见的避免热点的方法以及它们的优缺点：

#### 加盐

这里所说的加盐不是密码学中的加盐，而是在rowkey的前面增加随机数，具体就是给rowkey分配一个随机前缀以使得它和之前的rowkey的开头不同。分配的前缀种类数量应该和你想使用数据分散到不同的region的数量一致。加盐之后的rowkey就会根据随机生成的前缀分散到各个region上，以避免热点。

#### 哈希

哈希会使同一行永远用一个前缀加盐。哈希也可以使负载分散到整个集群，但是读却是可以预测的。使用确定的哈希可以让客户端重构完整的rowkey，可以使用get操作准确获取某一个行数据。

#### 反转

第三种防止热点的方法时反转固定长度或者数字格式的rowkey。这样可以使得rowkey中经常改变的部分（最没有意义的部分）放在前面。这样可以有效的随机rowkey，但是牺牲了rowkey的有序性。反转rowkey的例子以手机号为rowkey，可以将手机号反转后的字符串作为rowkey，这样的就避免了以手机号那样比较固定开头导致热点问题。

#### 时间戳反转

一个常见的数据处理问题是快速获取数据的最近版本，使用反转的时间戳作为rowkey的一部分对这个问题十分有用，可以用 Long.Max_Value - timestamp 追加到key的末尾，例如key、reverse_timestamp, key的最新值可以通过scan key获得key的第一条记录，因为HBase中rowkey是有序的，第一条记录是最后录入的数据。

#### 其他

尽量减少行键和列族的大小在HBase中，value永远和它的key一起传输的。当具体的值在系统间传输时，它的rowkey，列名，时间戳也会一起传输。如果你的rowkey和列名很大，这个时候它们将会占用大量的存储空间。列族尽可能越短越好，最好是一个字符。冗长的属性名虽然可读性好，但是更短的属性名存储在HBase中会更好。

## 协处理器

### observer

Observer 类似于传统数据库中的触发器，当发生某些事件的时候这类协处理器会被 Server 端调用。Observer Coprocessor 就是一些散布在 HBase Server 端代码中的 hook 钩子， 在固定的事件发生时被调用。比如： put 操作之前有钩子函数 prePut，该函数在 put 操作执行前会被 Region Server 调用；在 put 操作之后则有 postPut 钩子函数。以Hbase2.0.0版本为例，它提供了三种观察者接口：

1. RegionObserver：提供客户端的数据操纵事件钩子： Get、 Put、 Delete、 Scan 等。
2. WALObserver：提供 WAL 相关操作钩子。
3. MasterObserver：提供 DDL-类型的操作钩子。如创建、删除、修改数据表等。

### Endpoint

Endpoint协处理器类似传统数据库中的存储过程，客户端可以调用这些Endpoint协处理器执行一段Server端代码，并将Server端代码的结果返回给客户端进一步处理，最常见的用法就是进行聚集操作。如果没有协处理器，当用户需要找出一张表中的最大数据，即max聚合操作，就必须进行全表扫描，在客户端代码内遍历扫描结果，并执行求最大值的操作。这样的方法无法利用底层集群的并发能力，而将所有计算都集中到Client端统一执行，势必效率低下。利用Coprocessor，用户可以将求最大值的代码部署到HBaseServer端，HBase将利用底层cluster的多个节点并发执行求最大值的操作。即在每个Region范围内执行求最大值的代码，将每个Region的最大值在RegionServer端计算出，仅仅将该max值返回给客户端。在客户端进一步将多个Region的最大值进一步处理而找到其中的最大值。这样整体的执行效率就会提高很多

### 总结

1. Observer 允许集群在正常的客户端操作过程中可以有不同的行为表现
2. Endpoint 允许扩展集群的能力，对客户端应用开放新的运算命令
3. observer 类似于 RDBMS 中的触发器，主要在服务端工作
4. endpoint 类似于 RDBMS 中的存储过程，主要在 client 端工作
5. observer 可以实现权限管理、优先级设置、监控、 ddl 控制、 二级索引等功能
6. endpoint 可以实现 min、 max、 avg、 sum、 distinct、 group by 等功能

### 协处理器加载方式 

协处理器的加载方式有两种，静态加载方式（ Static Load） 和动态加载方式（ Dynamic Load）。 静态加载的协处理器称之为 ystem Coprocessor，动态加载的协处理器称 之为Table Coprocessor

#### 静态加载 

通过修改hbase-site.xml这个文件来实现， 启动全局aggregation，能过操纵所有的表上的数据。只需要添加如下代码：

```xml
<property>
	<name>hbase.coprocessor.user.region.classes</name>
	<value>org.apache.hadoop.hbase.coprocessor.AggregateImplementation</value>
</property>
```

为所有table 加载了一个 cp class，可以用” ,”分割加载多个 class

#### 动态加载

启用表 aggregation，只对特定的表生效。通过 HBase Shell 来实现。

```hive
--disable 指定表
disable 'mytable'
--添加 aggregation
alter 'mytable', METHOD => 'table_att','coprocessor'=>
'|org.apache.Hadoop.hbase.coprocessor.AggregateImplementation|1001|'
--重启指定表
enable 'mytable'
```

协处理器卸载

```hive
--disable 指定表
disable 'mytable'
--添加 aggregation
alter 'mytable', METHOD => 'table_att',NAME=>'coprocessor$1'
--重启指定表
enable 'mytable'
```

### Observer应用

通过协处理器Observer实现hbase当中一张表插入数据，然后通过协处理器，将数据复制一份保存到另外一张表当中去，但是只取当第一张表当中的部分列数据保存到第二张表当中去

HBase当中创建第一张表proc1，表名proc1，并只有一个列族info

```sql
create 'proc1','info'
```

Hbase当中创建第二张表proc2，作为目标表，将第一张表当中插入数据的部分列，使用协处理器，复制到'proc2表当中来

```sql
create 'proc2','info'
```

开发HBase的协处理器

```java
package org.duo.hbase.ObserverOperate;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.hbase.*;
import org.apache.hadoop.hbase.client.*;
import org.apache.hadoop.hbase.coprocessor.ObserverContext;
import org.apache.hadoop.hbase.coprocessor.RegionCoprocessor;
import org.apache.hadoop.hbase.coprocessor.RegionCoprocessorEnvironment;
import org.apache.hadoop.hbase.coprocessor.RegionObserver;
import org.apache.hadoop.hbase.util.Bytes;
import org.apache.hadoop.hbase.wal.WALEdit;

import java.io.IOException;
import java.util.List;
import java.util.Optional;

public class MyProcessor implements RegionObserver, RegionCoprocessor {

    static Connection connection = null;
    static Table table = null;

    //使用静态代码块来创建连接对象，避免频繁的创建连接对象
    static {
        Configuration conf = HBaseConfiguration.create();
        conf.set("hbase.zookeeper.quorum", "server01:2181");
        try {
            connection = ConnectionFactory.createConnection(conf);
            table = connection.getTable(TableName.valueOf("proc2"));
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    private RegionCoprocessorEnvironment env = null;
    //定义列族名
    private static final String FAMAILLY_NAME = "info";
    //定义列名
    private static final String QUALIFIER_NAME = "name";

    //2.0加入该方法，否则无法生效
    @Override
    public Optional<RegionObserver> getRegionObserver() {
        // Extremely important to be sure that the coprocessor is invoked as a RegionObserver
        return Optional.of(this);
    }

    /**
     * 初始化协处理器环境
     *
     * @param e
     * @throws IOException
     */
    @Override
    public void start(CoprocessorEnvironment e) throws IOException {
        env = (RegionCoprocessorEnvironment) e;
    }

    @Override
    public void stop(CoprocessorEnvironment e) throws IOException {
        // nothing to do here
    }

    /**
     * 覆写prePut方法，在我们数据插入之前进行拦截，
     *
     * @param e
     * @param put        put对象里面封装了我们需要插入到目标表的数据
     * @param edit
     * @param durability
     * @throws IOException
     */
    @Override
    public void prePut(final ObserverContext<RegionCoprocessorEnvironment> e,
                       final Put put, final WALEdit edit, final Durability durability)
            throws IOException {
        try {
            //通过put对象获取插入数据的rowkey
            byte[] rowBytes = put.getRow();
            String rowkey = Bytes.toString(rowBytes);
            //获取我们插入数据的name字段的值

            List<Cell> list = put.get(Bytes.toBytes(FAMAILLY_NAME), Bytes.toBytes(QUALIFIER_NAME));
            //判断如果没有获取到info列族，和name列，直接返回即可
            if (list == null || list.size() == 0) {
                return;
            }
            //获取到info列族，name列对应的cell
            Cell cell2 = list.get(0);

            //通过cell获取数据值
            String nameValue = Bytes.toString(CellUtil.cloneValue(cell2));
            //创建put对象，将数据插入到proc2表里面去
            Put put2 = new Put(rowkey.getBytes());
            put2.addColumn(Bytes.toBytes(FAMAILLY_NAME), Bytes.toBytes(QUALIFIER_NAME), nameValue.getBytes());
            table.put(put2);
            table.close();
        } catch (Exception e1) {
            return;
        }
    }
}
```

将项目打成jar包，并上传到HDFS上面

```bash
[root@server01 ~]# hdfs dfs -mkdir /processor
[root@server01 ~]# hdfs dfs -put /opt/processor.jar /processor
```

将打好的jar包挂载到proc1表当中去

```sql
describe 'proc1'
alter 'proc1',METHOD => 'table_att','Coprocessor'=>'hdfs://server01:8020/processor/processor.jar|org.duo.hbase.ObserverOperate.MyProcessor|1001|'
describe 'proc1'
```

proc1表当中添加数据

```sql
put 'proc1','0001','info:name','zhangsan'
put 'proc1','0001','info:age','28'
put 'proc1','0002','info:name','lisi'
put 'proc1','0002','info:age','25'
```

查看proc2表的插入结果

```sql
scan  'proc2'
```

## 二级索引

由于HBase的查询比较弱，如果需要实现类似于  select  name,salary,count(1),max(salary) from user  group  by name,salary order  by  salary 等这样的复杂性的统计需求，基本上不可能，或者说比较困难，所以在使用HBase的时候，一般都会借助二级索引的方案来进行实现；HBase的一级索引就是rowkey，只能通过rowkey进行检索。如果相对hbase里面列族的列列进行一些组合查询，就需要采用HBase的二级索引方案来进行多条件的查询。 （二级索引，即：将经常要查询的数据冗余存储一份到mysql、es等查询性能较高的数据库中，实现空间换时间）

1. MapReduce方案 
2. ITHBASE（Indexed-Transanctional HBase）方案 
3. IHBASE（Index HBase）方案 
4. Hbase Coprocessor(协处理器)方案 
5. Solr+hbase方案
6. CCIndex（complementalclustering index）方案

常见的二级索引我们一般可以借助各种其他的方式来实现，例如Phoenix或者solr或者ES等

## 调优

### 通用优化

1. NameNode的元数据备份使用SSD
2. 定时备份NameNode上的元数据，每小时或者每天备份，如果数据极其重要，可以5~10分钟备份一次。备份可以通过定时任务复制元数据目录即可。
3. 为NameNode指定多个元数据目录，使用dfs.name.dir或者dfs.namenode.name.dir指定。一个指定本地磁盘，一个指定网络磁盘。这样可以提供元数据的冗余和健壮性，以免发生故障。
4. 设置dfs.namenode.name.dir.restore为true，允许尝试恢复之前失败的dfs.namenode.name.dir目录，在创建checkpoint时做此尝试，如果设置了多个磁盘，建议允许。
5. NameNode节点必须配置为RAID1（镜像盘）结构。
   1. Standalone：最普遍的单磁盘储存方式。
   2. Cluster：集群储存是通过将数据分布到集群中各节点的存储方式,提供单一的使用接口与界面,使用户可以方便地对所有数据进行统一使用与管理。
   3. Hot swap：用户可以再不关闭系统,不切断电源的情况下取出和更换硬盘,提高系统的恢复能力、拓展性和灵活性。
   4. Raid0：Raid0是所有raid中存储性能最强的阵列形式。其工作原理就是在多个磁盘上分散存取连续的数据,这样,当需要存取数据是多个磁盘可以并排执行,每个磁盘执行属于它自己的那部分数据请求,显著提高磁盘整体存取性能。但是不具备容错能力,适用于低成本、低可靠性的台式系统。
   5. Raid1：又称镜像盘,把一个磁盘的数据镜像到另一个磁盘上,采用镜像容错来提高可靠性,具有raid中最高的数据冗余能力。存数据时会将数据同时写入镜像盘内,读取数据则只从工作盘读出。发生故障时,系统将从镜像盘读取数据,然后再恢复工作盘正确数据。这种阵列方式可靠性极高,但是其容量会减去一半。广泛用于数据要求极严的应用场合,如商业金融、档案管理等领域。只允许一颗硬盘出故障。
   6. Raid0+1：将Raid0和Raid1技术结合在一起,兼顾两者的优势。在数据得到保障的同时,还能提供较强的存储性能。不过至少要求4个或以上的硬盘，但也只允许一个磁盘出错。是一种三高技术。
   7. Raid5：Raid5可以看成是Raid0+1的低成本方案。采用循环偶校验独立存取的阵列方式。将数据和相对应的奇偶校验信息分布存储到组成RAID5的各个磁盘上。当其中一个磁盘数据发生损坏后,利用剩下的磁盘和相应的奇偶校验信息 重新恢复/生成丢失的数据而不影响数据的可用性。至少需要3个或以上的硬盘。适用于大数据量的操作。成本稍高、储存性强、可靠性强的阵列方式。
   8. RAID还有其他方式。
6. 保持NameNode日志目录有足够的空间，这些日志有助于帮助你发现问题。
7. 因为Hadoop是IO密集型框架，所以尽量提升存储的速度和吞吐量（类似位宽）。

### Linux优化

1. 开启文件系统的预读缓存可以提高读取速度

   blockdev --setra 32768 /dev/sda

2. 关闭进程睡眠池

   sysctl -w vm.swappiness=0

3. 调整ulimit上限，默认值为比较小的数字

### HDFS优化（hdfs-site.xml）

1. 保证RPC调用会有较多的线程数

   属性：dfs.namenode.handler.count
   解释：该属性是NameNode服务默认线程数，的默认值是10，根据机器的可用内存可以调整为50~100
   属性：dfs.datanode.handler.count
   解释：该属性默认值为10，是DataNode的处理线程数，如果HDFS客户端程序读写请求比较多，可以调高到15~20，设置的值越大，内存消耗越多，不要调整的过高，一般业务中，5~10即可。

2. 副本数的调整

   属性：dfs.replication
   解释：如果数据量巨大，且不是非常之重要，可以调整为2~3，如果数据非常之重要，可以调整为3~5。

3. 文件块大小的调整

   属性：dfs.blocksize
   解释：块大小定义，该属性应该根据存储的大量的单个文件大小来设置，如果大量的单个文件都小于100M，建议设置成64M块大小，对于大于100M或者达到GB的这种情况，建议设置成256M，一般设置范围波动在64M~256M之间。

### MapReduce优化（mapred-site.xml）

1. Job任务服务线程数调整

   mapreduce.jobtracker.handler.count
   该属性是Job任务线程数，默认值是10，根据机器的可用内存可以调整为50~100

2. Http服务器工作线程数

   属性：mapreduce.tasktracker.http.threads
   解释：定义HTTP服务器工作线程数，默认值为40，对于大集群可以调整到80~100

3. 文件排序合并优化

   属性：mapreduce.task.io.sort.factor
   解释：文件排序时同时合并的数据流的数量，这也定义了同时打开文件的个数，默认值为10，如果调高该参数，可以明显减少磁盘IO，即减少文件读取的次数。

4. 设置任务并发

   属性：mapreduce.map.speculative
   解释：该属性可以设置任务是否可以并发执行，如果任务多而小，该属性设置为true可以明显加快任务执行效率，但是对于延迟非常高的任务，建议改为false，这就类似于迅雷下载。

5. MR输出数据的压缩

   属性：mapreduce.map.output.compress、mapreduce.output.fileoutputformat.compress
   解释：对于大集群而言，建议设置Map-Reduce的输出为压缩的数据，而对于小集群，则不需要。

6. 优化Mapper和Reducer的个数

   属性：
   mapreduce.tasktracker.map.tasks.maximum
   mapreduce.tasktracker.reduce.tasks.maximum
   解释：以上两个属性分别为一个单独的Job任务可以同时运行的Map和Reduce的数量。
   设置上面两个参数时，需要考虑CPU核数、磁盘和内存容量。假设一个8核的CPU，业务内容非常消耗CPU，那么可以设置map数量为4，如果该业务不是特别消耗CPU类型的，那么可以设置map数量为40，reduce数量为20。这些参数的值修改完成之后，一定要观察是否有较长等待的任务，如果有的话，可以减少数量以加快任务执行，如果设置一个很大的值，会引起大量的上下文切换，以及内存与磁盘之间的数据交换，这里没有标准的配置数值，需要根据业务和硬件配置以及经验来做出选择。在同一时刻，不要同时运行太多的MapReduce，这样会消耗过多的内存，任务会执行的非常缓慢，我们需要根据CPU核数，内存容量设置一个MR任务并发的最大值，使固定数据量的任务完全加载到内存中，避免频繁的内存和磁盘数据交换，从而降低磁盘IO，提高性能。

### HBase优化

1. 在HDFS的文件中追加内容

   属性：dfs.support.append
   文件：hdfs-site.xml、hbase-site.xml
   解释：开启HDFS追加同步，可以优秀的配合HBase的数据同步和持久化。默认值为true。

2. 优化DataNode允许的最大文件打开数

   属性：dfs.datanode.max.transfer.threads
   文件：hdfs-site.xml
   解释：HBase一般都会同一时间操作大量的文件，根据集群的数量和规模以及数据动作，设置为4096或者更高。默认值：4096

3. 优化延迟高的数据操作的等待时间

   属性：dfs.image.transfer.timeout
   文件：hdfs-site.xml
   解释：如果对于某一次数据操作来讲，延迟非常高，socket需要等待更长的时间，建议把该值设置为更大的值（默认60000毫秒），以确保socket不会被timeout掉。

4. 优化数据的写入效率

   属性：
   mapreduce.map.output.compress
   mapreduce.map.output.compress.codec
   文件：mapred-site.xml
   解释：开启这两个数据可以大大提高文件的写入效率，减少写入时间。第一个属性值修改为true，第二个属性值修改为：org.apache.hadoop.io.compress.GzipCodec

5. 优化DataNode存储

   属性：dfs.datanode.failed.volumes.tolerated
   文件：hdfs-site.xml
   解释：默认为0，意思是当DataNode中有一个磁盘出现故障，则会认为该DataNode shutdown了。如果修改为1，则一个磁盘出现故障时，数据会被复制到其他正常的DataNode上，当前的DataNode继续工作。

6. 设置RPC监听数量

   属性：hbase.regionserver.handler.count
   文件：hbase-site.xml
   解释：默认值为30，用于指定RPC监听的数量，可以根据客户端的请求数进行调整，读写请求较多时，增加此值。

7. 优化HStore文件大小

   属性：hbase.hregion.max.filesize
   文件：hbase-site.xml
   解释：默认值10737418240（10GB），如果需要运行HBase的MR任务，可以减小此值，因为一个region对应一个map任务，如果单个region过大，会导致map任务执行时间过长。该值的意思就是，如果HFile的大小达到这个数值，则这个region会被切分为两个Hfile。

8. 优化hbase客户端缓存

   属性：hbase.client.write.buffer
   文件：hbase-site.xml
   解释：用于指定HBase客户端缓存，增大该值可以减少RPC调用次数，但是会消耗更多内存，反之则反之。一般我们需要设定一定的缓存大小，以达到减少RPC次数的目的。

9. 指定scan.next扫描HBase所获取的行数

   属性：hbase.client.scanner.caching
   文件：hbase-site.xml
   解释：用于指定scan.next方法获取的默认行数，值越大，消耗内存越大。

## namespace

### 基本介绍

在HBase中，namespace命名空间指对一组表的逻辑分组，类似RDBMS中的database，方便对表在业务上划分。Apache HBase从0.98.0, 0.95.2两个版本号開始支持namespace级别的授权操作，HBase全局管理员能够创建、改动和回收namespace的授权。

### 作用

1. 配额管理：限制一个namespace可以使用的资源，包括region和table
2. 命名空间安全管理：提供了另一个层面的多租户安全管理
3. Region服务器组：一个命名或一张表，可以被固定到一组RegionServers上，从而保证了数据隔离性

### 基本操作

创建namespace

```sql
create_namespace 'nametest'  
```

查看namespace

```sql
describe_namespace 'nametest'
```

列出所有namespace

```sql
list_namespace
```

在namespace下创建表

```sql
create 'nametest:testtable', 'fm1'
```

查看namespace下的表

```sql
list_namespace_tables 'nametest'
```

删除namespace

```sql
drop_namespace 'nametest'
```

## 数据版本的确界

### 数据的确界

在Hbase当中，可以为数据设置上界和下界，其实就是定义数据的历史版本保留多少个，通过自定义历史版本保存的数量，我们可以实现历史多个版本的数据的查询

版本的下界：默认的版本下界是0，即禁用。row版本使用的最小数目是与生存时间（TTL Time To Live）相结合的，并且根据实际需求可以有0或更多的版本，使用0，即只有1个版本的值写入cell。

版本的上界：之前默认的版本上界是3，也就是一个row保留3个副本（基于时间戳的插入）。该值不要设计的过大，一般的业务不会超过100。如果cell中存储的数据版本号超过了3个，再次插入数据时，最新的值会将最老的值覆盖。（现版本已默认为1）

### 数据的TTL

在实际工作当中经常会遇到有些数据过了一段时间我们可能就不需要了，那么这时候我们可以使用定时任务去定时的删除这些数据，或者我们也可以使用Hbase的TTL（Time  To  Live）功能，让我们的数据定期的会进行清除

在scan的时候，显示的都是最新的一个版本，使用JavaAPI可以查询到较早的版本；过期时间TTL可以针对列族进行设置，也可以针对某一条put对象进行设置；最大版本以及最小版本，也就是上界与下界，只能够针对列族进行设置。

确界是和TTL配合使用的，比如下界为3，TTL为30秒，针对于某个值更新5次，在一分钟之后再查，其实它还是会保留最后的三个版本，所以说TTL只针对于超过下界的那些值会在TTL时间之后会被清理掉

HBaseVersionAndTTL.java

```java
package org.duo.hbase;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.hbase.*;
import org.apache.hadoop.hbase.client.*;
import org.apache.hadoop.hbase.util.Bytes;

import java.io.IOException;

public class HBaseVersionAndTTL {

    public static void main(String[] args) throws IOException, InterruptedException {

        //操作hbase，向hbase表当中添加一条数据，并且设置数据的上界以及下界，以及设置数据的TTL过期时间
        Configuration configuration = HBaseConfiguration.create();
        configuration.set("hbase.zookeeper.quorum", "server01:2181,server02:2181,server03:2181");
        //获取连接
        Connection connection = ConnectionFactory.createConnection(configuration);

        //创建hbase表
        Admin admin = connection.getAdmin();
        //判断如果hbase表不存在，那么就创建
        if (!admin.tableExists(TableName.valueOf("version_hbase"))) {
            //指定表名
            HTableDescriptor hTableDescriptor = new HTableDescriptor(TableName.valueOf("version_hbase"));
            //为表指定列族名
            HColumnDescriptor f1 = new HColumnDescriptor("f1");
            //针对列族设置版本的上界以及版本下界
            f1.setMinVersions(3);
            f1.setMaxVersions(5);  //设置我们版本的上界
            f1.setTimeToLive(30);  //设置我们f1列族下面所有的列，最大的存活时间是30s
            //为我们表添加列族
            hTableDescriptor.addFamily(f1);
            admin.createTable(hTableDescriptor);
        }

        Table version_hbase = connection.getTable(TableName.valueOf("version_hbase"));

       /* Put put = new Put("1".getBytes());
        //如果需要往里面保存多个版本，一定要带上时间戳
        put.addColumn("f1".getBytes(), "name".getBytes(), System.currentTimeMillis(), "zhangsan".getBytes());

        Thread.sleep(1000);
        Put put2 = new Put("1".getBytes());
        //如果需要往里面保存多个版本，一定要带上时间戳
        put2.addColumn("f1".getBytes(), "name".getBytes(), System.currentTimeMillis(), "zhangsan2".getBytes());

        Thread.sleep(1000);
        Put put3 = new Put("1".getBytes());
        //如果需要往里面保存多个版本，一定要带上时间戳
        put3.addColumn("f1".getBytes(), "name".getBytes(), System.currentTimeMillis(), "zhangsan3".getBytes());

        Thread.sleep(1000);
        Put put4 = new Put("1".getBytes());
        //如果需要往里面保存多个版本，一定要带上时间戳
        put4.addColumn("f1".getBytes(), "name".getBytes(), System.currentTimeMillis(), "zhangsan4".getBytes());


        Thread.sleep(1000);
        Put put5 = new Put("1".getBytes());
        //如果需要往里面保存多个版本，一定要带上时间戳
        put5.addColumn("f1".getBytes(), "name".getBytes(), System.currentTimeMillis(), "zhangsan5".getBytes());

        Thread.sleep(1000);
        Put put6 = new Put("1".getBytes());
        //可以针对某一条数据设置过期时间
        //put6.setTTL(3000);
        //如果需要往里面保存多个版本，一定要带上时间戳
        put6.addColumn("f1".getBytes(), "name".getBytes(), System.currentTimeMillis(), "zhangsan6".getBytes());

        //将所有的数据都put到version_hbase这个表里面去
        version_hbase.put(put);
        version_hbase.put(put2);
        version_hbase.put(put3);
        version_hbase.put(put4);
        version_hbase.put(put5);
        version_hbase.put(put6);*/

        Get get = new Get("1".getBytes());
        get.setMaxVersions();//如果不带任何参数，表示将数据的所有的版本全部都获取到,如果带上参数，表示我们获取指定个版本的数据
        Result result = version_hbase.get(get);
        //获取所有的cell
        Cell[] cells = result.rawCells();
        for (Cell cell : cells) {
            System.out.println(Bytes.toString(CellUtil.cloneValue(cell)));
        }

        version_hbase.close();
        connection.close();
    }
}
```

# Flink

## 介绍

Stateful Computations over Data Streams，即数据流上的有状态的计算。

Data Streams 

Flink认为有界数据集是无界数据流的一种特例，所以说有界数据集也是一种数据流，事件流也是一种数据流。Everything is streams，即Flink可以用来处理任何的数据，可以支持批处理、流处理、AI、MachineLearning等等。

Stateful Computations

即有状态计算。有状态计算是最近几年来越来越被用户需求的一个功能。比如说一个网站一天内访问UV数，那么这个UV数便为状态。Flink提供了内置的对状态的一致性的处理，即如果任务发生了Failover，其状态不会丢失、不会被多算少算，同时提供了非常高的性能。

无界流

意思很明显，只有开始没有结束。必须连续的处理无界流数据，也即是在事件注入之后立即要对其进行处理。不能等待数据到达了再去全部处理，因为数据是无界的并且永远不会结束数据注入。处理无界流数据往往要求事件注入的时候有一定的顺序性，例如可以以事件产生的顺序注入，这样会使得处理结果完整。

有界流

也即是有明确的开始和结束的定义。有界流可以等待数据全部注入完成了再开始处理。注入的顺序不是必须的了，因为对于一个静态的数据集，我们是可以对其进行排序的。有界流的处理也可以称为批处理。

其它特点:

1. 性能优秀(尤其在流计算领域)
2. 高可扩展性
3. 支持容错
4. 纯内存式的计算引擎，做了内存管理方面的大量优化
5. 支持eventime的处理
6. 支持超大状态的Job(在阿里巴巴中作业的state大小超过TB的是非常常见的)
7. 支持exactly-once的处理。

## 环境安装

https://archive.apache.org/dist/flink/

### 伪分布环境部署

单机模式

![image][1]

- Flink程序需要提交给Job Client
- Job Client将作业提交给Job Manager
- Job Manager负责协调资源分配和作业执行。 资源分配完成后，任务将提交给相应的Task Manager
- Task Manager启动一个线程以开始执行。Task Manager会向Job Manager报告状态更改。例如开始执行，正在进行或已完成。 
- 作业执行完成后，结果将发送回客户端（Job Client）

```bash
[root@server01 local]# tar -zxvf flink-1.6.1-bin-hadoop27-scala_2.11.tgz
[root@server01 local]# cd /usr/local/flink-1.6.1/bin
[root@server01 bin]# ./start-cluster.sh
# 运行测试任务
[root@server01 bin]# /usr/local/flink-1.6.1/bin/flink run /usr/local/flink-1.6.1/examples/batch/WordCount.jar --input /usr/local/zookeeper/zookeeper.out --output /opt/flink_data
```

### Standalone模式

独立模式，Flink自带集群

![image][2]

- client客户端提交任务给JobManager
- JobManager负责Flink集群计算资源管理，并分发任务给TaskManager执行
- TaskManager定期向JobManager汇报状态

flink-conf.yaml

```yaml
# jobManager 的IP地址
jobmanager.rpc.address: server01

# JobManager 的端口号
jobmanager.rpc.port: 6123

# JobManager JVM heap 内存大小
jobmanager.heap.size: 1024m

# TaskManager JVM heap 内存大小
taskmanager.heap.size: 1024m

# 每个TaskManager提供的任务slots数量大小,如果有3个taskmanager一共有6个TaskSlot；是静态资源，即cpu的核数；
taskmanager.numberOfTaskSlots: 2

#是否进行预分配内存，默认不进行预分配，这样在我们不使用flink集群时候不会占用集群资源
taskmanager.memory.preallocate: false

# 程序默认并行计算的个数，默认的并行度为1，6个TaskSlot只用了1个，有5个空闲；是动态的概念，指程序运行时实际使用的并发能力，所以parallelism的值不能大于numberOfTaskSlots X taskmanager个数
parallelism.default: 6

#JobManager的Web界面的端口（默认：8081）
jobmanager.web.port: 8081

#配置每个taskmanager生成的临时文件目录（选配）
taskmanager.tmp.dirs: /usr/local/flink-1.6.1/tmp
```

slaves

```properties
server01
server02
server03
```

修改/etc/profile系统环境变量配置文件，添加HADOOP_CONF_DIR目录

```properties
export HADOOP_CONF_DIR=/usr/local/hadoop-2.7.5/etc/hadoop/
export FLINK_HOME=/usr/local/flink-1.6.1
export PATH=$PATH:$FLINK_HOME/bin
```

将修改好的配置文件复制到flink其他节点

```bash
[root@server01 conf]# cd /usr/local/flink-1.6.1/conf
[root@server01 conf]# scp flink-conf.yaml server02:$PWD
[root@server01 conf]# scp flink-conf.yaml server02:$PWD
[root@server01 conf]# scp slaves server02:$PWD
[root@server01 conf]# scp slaves server03:$PWD
[root@server01 conf]# cd /etc/
[root@server01 conf]# scp profile server02:$PWD
[root@server01 conf]# scp profile server03:$PWD
```

启动hadoop的yarn和hdfs以及flink集群

```bash
[root@server01 etc]# start-all.sh
[root@server01 etc]# start-cluster.sh
```

测试：

```bash
[root@server01 etc]# hdfs dfs -mkdir -p /test/input
[root@server01 etc]# hdfs dfs -put /usr/local/flink-1.6.1/README.txt /test/input
[root@server01 etc]# flink run /usr/local/flink-1.6.1/examples/batch/WordCount.jar --input hdfs://server01:8020/test/input/README.txt --output hdfs://server01:8020/test/output2/result.txt
```

### Standalone高可用HA模式

从上述架构图中，可发现JobManager存在单点故障，一旦JobManager出现意外，整个集群无法工作。所以，为了确保集群的高可用，需要搭建Flink的HA。

![image][3]

在flink-conf.yaml中添加zookeeper配置

```yaml
#开启HA，使用文件系统作为快照存储
state.backend: filesystem
#启用检查点，可以将快照保存到HDFS
state.backend.fs.checkpointdir: hdfs://server01:8020/flink-checkpoints
#使用zookeeper搭建高可用
high-availability: zookeeper
# 存储JobManager的元数据到HDFS
high-availability.storageDir: hdfs://server01:8020/flink/ha/
high-availability.zookeeper.quorum: server01:2181,server02:2181,server03:2181
```

将配置过的HA的flink-conf.yaml分发到另外两个节点

```bash
[root@server01 conf]# cd /usr/local/flink-1.6.1/conf
[root@server01 conf]# scp flink-conf.yaml server02:$PWD
[root@server01 conf]# scp flink-conf.yaml server03:$PWD
```

到server02中修改flink-conf.yaml中的配置，将JobManager设置为自己节点的名称(切记搭建HA，需要将第二个节点的jobmanager.rpc.address修改为server02)

```yaml
jobmanager.rpc.address: server02
```

在server01的masters配置文件中添加多个节点

```properties
server01:8081
server02:8081
```

分发masters配置文件到另外两个节点

```bash
[root@server01 conf]# cd /usr/local/flink-1.6.1/conf
[root@server01 conf]# scp masters server02:$PWD
[root@server01 conf]# scp masters server03:$PWD
```

启动zookeeper集群

启动HDFS集群

启动flink集群

### Yarn集群环境

计算资源统一由Hadoop YARN管理，修改Hadoop的yarn-site.xml，添加该配置表示内存超过分配值，是否将任务杀掉。默认为true。运行Flink程序，很容易超过分配的内存。

```xml
<property>
	<name>yarn.nodemanager.vmem-check-enabled</name>
    <value>false</value>
</property>
```

分发yarn-site.xml到其它服务器节点

```bash
[root@server01 conf]# cd /usr/local/hadoop-2.7.5/etc/hadoop/
[root@server01 hadoop]# scp yarn-site.xml server02:$PWD
[root@server01 hadoop]# scp yarn-site.xml server03:$PWD
```

启动zookeeper集群

启动HDFS集群

启动flink集群

#### yarn-session

Flink运行在YARN上，可以使用yarn-session来快速提交作业到YARN集群，

![image][4]

交互过程如下：

1. 上传jar包和配置文件到HDFS集群上

2. 申请资源和请求AppMaster容器

3. Yarn分配资源AppMaster容器，并启动JobManager

   JobManager和ApplicationMaster运行在同一个container上。一旦他们被成功启动，AppMaster就知道JobManager的地址（AM它自己所在的机器）。它就会为TaskManager生成一个新的Flink配置文件（他们就可以连接到JobManager）。这个配置文件也被上传到HDFS上。此外，AppMaster容器也提供了Flink的web服务接口。YARN所分配的所有端口都是临时端口，这允许用户并行执行多个Flink

4. 申请worker资源，启动TaskManager

   ![image][5]

yarn-session提供两种模式: 会话模式和分离模式

##### 会话模式

![image][6]

1. 使用Flink中的yarn-session（yarn客户端），会启动两个必要服务JobManager和TaskManager
2. 客户端通过yarn-session提交作业
3. yarn-session会一直启动，不停地接收客户端提交的作用
4. 有大量的小作业，适合使用这种方式

启动yarn-session

```bash
# -n 表示申请2个容器，
# -s 表示每个容器启动多少个slot
# -tm 表示每个TaskManager申请800M内存
# -d 表示以后台程序方式运行
[root@server01 hadoop]# yarn-session.sh -n 2 -tm 800 -s 1 -d
```

yarn-session.sh脚本可以携带的参数:

```shell
yarn-session.sh脚本可以携带的参数:
   Required
     -n,--container <arg>               分配多少个yarn容器 (=taskmanager的数量)  
   Optional
     -D <arg>                        动态属性
     -d,--detached                    独立运行 （以分离模式运行作业）
     -id,--applicationId <arg>            YARN集群上的任务id，附着到一个后台运行的yarn session中
     -j,--jar <arg>                      Path to Flink jar file
     -jm,--jobManagerMemory <arg>     JobManager的内存 [in MB] 
     -m,--jobmanager <host:port>        指定需要连接的jobmanager(主节点)地址  
                                    使用这个参数可以指定一个不同于配置文件中的jobmanager  
     -n,--container <arg>               分配多少个yarn容器 (=taskmanager的数量) 
     -nm,--name <arg>                 在YARN上为一个自定义的应用设置一个名字
     -q,--query                        显示yarn中可用的资源 (内存, cpu核数) 
     -qu,--queue <arg>                 指定YARN队列
     -s,--slots <arg>                   每个TaskManager使用的slots数量
     -st,--streaming                   在流模式下启动Flink
     -tm,--taskManagerMemory <arg>    每个TaskManager的内存 [in MB] 
     -z,--zookeeperNamespace <arg>     针对HA模式在zookeeper上创建NameSpace
```

使用flink提交任务

```bash
[root@server01 hadoop]# cd /usr/local/flink-1.6.1
[root@server01 hadoop]# flink run examples/batch/WordCount.jar
```

如果程序运行完了，可以使用yarn application -kill application_id杀掉任务

```bash
[root@server01 flink-1.6.1]# yarn application -kill application_1663404082372_0001
```

##### 分离模式

![image][7]

1. 直接提交任务给YARN
2. 大作业，适合使用这种方式

使用flink直接提交任务

```bash
[root@server01 flink-1.6.1]# flink run -m yarn-cluster -yn 2 ./examples/batch/WordCount.jar
```

## 架构

### 基石

#### checkpoint

首先是Checkpoint机制，这是Flink最重要的一个特性。Flink基于Chandy-Lamport算法实现了一个分布式的一致性的快照，从而提供了一致性的语义。Chandy-Lamport算法实际上在1985年的时候已经被提出来，但并没有被很广泛的应用，而Flink则把这个算法发扬光大了。Spark最近在实现Continuestreaming，Continuestreaming的目的是为了降低它处理的延时，其也需要提供这种一致性的语义，最终采用Chandy-Lamport这个算法，说明Chandy-Lamport算法在业界得到了一定的肯定。

#### State

提供了一致性的语义之后，Flink为了让用户在编程时能够更轻松、更容易地去管理状态，还提供了一套非常简单明了的StateAPI，包括里面的有ValueState、ListState、MapState，近期添加了BroadcastState，使用StateAPI能够自动享受到这种一致性的语义。

#### Time

除此之外，Flink还实现了Watermark的机制，能够支持基于事件的时间的处理，或者说基于系统时间的处理，能够容忍数据的迟到、容忍乱序的数据。

#### Window

另外流计算中一般在对流数据进行操作之前都会先进行开窗，即基于一个什么样的窗口上做这个计算。Flink提供了开箱即用的各种窗口，比如滑动窗口、滚动窗口、会话窗口以及非常灵活的自定义的窗口。

### 组件栈

Flink是一个分层架构的系统，每一层所包含的组件都提供了特定的抽象，用来服务于上层组件：

从下至上：

- 部署层：Flink 支持本地运行、能在独立集群或者在被 YARN 管理的集群上运行， 也能部署在云上。
- 运行时：Runtime层提供了支持Flink计算的全部核心实现，为上层API层提供基础服务。
- API：DataStream、DataSet、Table、SQL API。
- 扩展库：Flink 还包括用于复杂事件处理，机器学习，图形处理和 Apache Storm 兼容性的专用代码库。

### 数据流编程模型

Flink 提供了不同的抽象级别以开发流式或批处理应用：

- 最底层提供了有状态流。它将通过 过程函数（Process Function）嵌入到 DataStream API 中。它允许用户可以自由地处理来自一个或多个流数据的事件，并使用一致、容错的状态。除此之外，用户可以注册事件时间和处理事件回调，从而使程序可以实现复杂的计算。

- DataStream / DataSet API 是 Flink 提供的核心 API ，DataSet 处理有界的数据集，DataStream 处理有界或者无界的数据流。用户可以通过各种方法（map / flatmap / window / keyby / sum / max / min / avg / join 等）将数据进行转换 / 计算。


- Table API 是以 表 为中心的声明式 DSL，其中表可能会动态变化（在表达流数据时）。Table API 提供了例如 select、project、join、group-by、aggregate 等操作，使用起来却更加简洁（代码量更少）。你可以在表与 DataStream/DataSet 之间无缝切换，也允许程序将 Table API 与 DataStream 以及 DataSet 混合使用。


- Flink 提供的最高层级的抽象是 SQL 。这一层抽象在语法与表达能力上与 Table API 类似，但是是以 SQL查询表达式的形式表现程序。SQL 抽象与 Table API 交互密切，同时 SQL 查询可以直接在 Table API 定义的表上执行。

### 程序结构

Flink程序的基本构建块是流和转换（请注意，Flink的DataSet API中使用的DataSet也是内部流 ）。从概念上讲，流是（可能永无止境的）数据记录流，而转换是将一个或多个流作为一个或多个流的操作。输入，并产生一个或多个输出流。Flink 应用程序结构：

- Source: 数据源，Flink 在流处理和批处理上的 source 大概有 4 类：基于本地集合的 source、基于文件的 source、基于网络套接字的 source、自定义的 source。自定义的 source 常见的有 Apache kafka、RabbitMQ 等，当然你也可以定义自己的 source。

- Transformation：数据转换的各种操作，有 Map / FlatMap / Filter / KeyBy / Reduce / Fold / Aggregations / Window / WindowAll / Union / Window join / Split / Select等，操作很多，可以将数据转换计算成你想要的数据。

- Sink：接收器，Flink 将转换计算后的数据发送的地点 ，你可能需要存储下来，Flink 常见的 Sink 大概有如下几类：写入文件、打印出来、写入 socket 、自定义的 sink 。自定义的 sink 常见的有 Apache kafka、RabbitMQ、MySQL、ElasticSearch、Apache Cassandra、Hadoop FileSystem 等，同理你也可以定义自己的 sink。

### 并行数据流

Flink程序在执行的时候，会被映射成一个Streaming Dataflow，一个Streaming Dataflow是由一组Stream和Transformation Operator组成的。在启动时从一个或多个Source Operator开始，结束于一个或多个Sink Operator。

Flink程序本质上是并行的和分布式的，在执行过程中，一个流(stream)包含一个或多个流分区，而每一个operator包含一个或多个operator子任务。操作子任务间彼此独立，在不同的线程中执行，甚至是在不同的机器或不同的容器上。operator子任务的数量是这一特定operator的并行度。相同程序中的不同operator有不同级别的并行度。

一个Stream可以被分成多个Stream的分区，也就是Stream Partition。一个Operator也可以被分为多个Operator Subtask。每一个Operator Subtask都是在不同的线程当中独立执行的。一个Operator的并行度，就等于Operator Subtask的个数。而一个Stream的并行度就等于它生成的Operator的并行度。

数据在两个operator之间传递的时候有两种模式：

One to One模式：两个operator用此模式传递的时候，会保持数据的分区数和数据的排序；如上图中的Source1到Map1，它就保留的Source的分区特性，以及分区元素处理的有序性。

Redistributing （重新分配）模式：这种模式会改变数据的分区数；每个一个operator subtask会根据选择transformation把数据发送到不同的目标subtasks,比如keyBy()会通过hashcode重新分区,broadcast()和rebalance()方法会随机重新分区； 

### Task和Operator chain

Flink的所有操作都称之为Operator，客户端在提交任务的时候会对Operator进行优化操作，能进行合并的Operator会被合并为一个Operator，合并后的Operator称为Operator chain，实际上就是一个执行链，每个执行链会在TaskManager上一个独立的线程中执行。

### 任务调度与执行

1. 当Flink执行executor会自动根据程序代码生成DAG数据流图
2. ActorSystem创建Actor将数据流图发送给JobManager中的Actor
3. JobManager会不断接收TaskManager的心跳消息，从而可以获取到有效的TaskManager
4. JobManager通过调度器在TaskManager中调度执行Task（在Flink中，最小的调度单元就是task，对应就是一个线程）
5. 在程序运行过程中，task与task之间是可以进行数据传输的

- Job Client

  - 主要职责是提交任务, 提交后可以结束进程, 也可以等待结果返回
  - Job Client 不是 Flink 程序执行的内部部分，但它是任务执行的起点。 
  - Job Client 负责接受用户的程序代码，然后创建数据流，将数据流提交给 Job Manager 以便进一步执行。 执行完成后，Job Client 将结果返回给用户

- JobManager`	

  - 主要职责是调度工作并协调任务做检查点
  - 集群中至少要有一个 master，master 负责调度 task，协调checkpoints 和容错，
  - 高可用设置的话可以有多个 master，但要保证一个是 leader, 其他是standby; 
  - Job Manager 包含 Actor System、Scheduler、CheckPoint 三个重要的组件
  - JobManager从客户端接收到任务以后, 首先生成优化过的执行计划, 再调度到TaskManager中执行

- TaskManager
  - 主要职责是从JobManager处接收任务, 并部署和启动任务, 接收上游的数据并处理

  - Task Manager 是在 JVM 中的一个或多个线程中执行任务的工作节点。

  - TaskManager在创建之初就设置好了Slot, 每个Slot可以执行一个任务

###  任务槽（task-slot）

每个TaskManager是一个JVM的进程, 可以在不同的线程中执行一个或多个子任务。 

为了控制一个worker能接收多少个task。worker通过task slot来进行控制（一个worker至少有一个task slot）。

每个task slot表示TaskManager拥有资源的一个固定大小的子集。 

flink将进程的内存进行了划分到多个slot中。

例如：有2个TaskManager，每个TaskManager有3个slot的，则每个slot占有1/3的内存。

内存被划分到不同的slot之后可以获得如下好处: 

- TaskManager最多能同时并发执行的任务是可以控制的，那就是3个，因为不能超过slot的数量。

- slot有独占的内存空间，这样在一个TaskManager中可以运行多个不同的作业，作业之间不受影响。

### 槽共享（Slot Sharing）

默认情况下，Flink允许子任务共享插槽，即使它们是不同任务的子任务，只要它们来自同一个作业。结果是一个槽可以保存作业的整个管道。允许插槽共享有两个主要好处：

- 只需计算Job中最高并行度（parallelism）的task slot,只要这个满足，其他的job也都能满足。
- 资源分配更加公平，如果有比较空闲的slot可以将更多的任务分配给它。图中若没有任务槽共享，负载不高的Source/Map等subtask将会占据许多资源，而负载较高的窗口subtask则会缺乏资源。
- 有了任务槽共享，可以将基本并行度（base parallelism）从2提升到6.提高了分槽资源的利用率。同时它还可以保障TaskManager给subtask的分配的slot方案更加公平。

### 统一的流处理与批处理

在大数据处理领域，批处理任务与流处理任务一般被认为是两种不同的任务，一个大数据框架一般会被设计为只能处理其中一种任务：

- Storm只支持流处理任务
- MapReduce、Spark只支持批处理任务
- Spark Streaming是Apache Spark之上支持流处理任务的子系统，看似是一个特例，其实并不是——Spark Streaming采用了一种micro-batch的架构，即把输入的数据流切分成细粒度的batch，并为每一个batch数据提交一个批处理的Spark任务，所以Spark Streaming本质上还是基于Spark批处理系统对流式数据进行处理，和Storm等完全流式的数据处理方式完全不同。
- Flink通过灵活的执行引擎，能够同时支持批处理任务与流处理任务

在执行引擎这一层，流处理系统与批处理系统最大不同在于节点间的数据传输方式：

- 对于一个流处理系统，其节点间数据传输的标准模型是：
  - 当一条数据被处理完成后，序列化到缓存中，然后立刻通过网络传输到下一个节点，由下一个节点继续处理

- 对于一个批处理系统，其节点间数据传输的标准模型是：
  - 当一条数据被处理完成后，序列化到缓存中，并不会立刻通过网络传输到下一个节点，当缓存写满，就持久化到本地硬盘上，当所有数据都被处理完成后，才开始将处理后的数据通过网络传输到下一个节点

这两种数据传输模式是两个极端，对应的是流处理系统对低延迟的要求和批处理系统对高吞吐量的要求

Flink的执行引擎采用了一种十分灵活的方式，同时支持了这两种数据传输模型

- Flink以固定的缓存块为单位进行网络数据传输，用户可以通过设置缓存块超时值指定缓存块的传输时机。如果缓存块的超时值为0，则Flink的数据传输方式类似上文所提到流处理系统的标准模型，此时系统可以获得最低的处理延迟


- 如果缓存块的超时值为无限大，则Flink的数据传输方式类似上文所提到批处理系统的标准模型，此时系统可以获得最高的吞吐量


- 同时缓存块的超时值也可以设置为0到无限大之间的任意值。缓存块的超时阈值越小，则Flink流处理执行引擎的数据处理延迟越低，但吞吐量也会降低，反之亦然。通过调整缓存块的超时阈值，用户可根据需求灵活地权衡系统延迟和吞吐量

## 批处理

### 环境搭建

pom.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.itheima</groupId>
    <artifactId>flink-base</artifactId>
    <version>1.0-SNAPSHOT</version>

    <properties>
        <maven.compiler.source>1.8</maven.compiler.source>
        <maven.compiler.target>1.8</maven.compiler.target>
        <encoding>UTF-8</encoding>
        <scala.version>2.11</scala.version>
        <flink.version>1.6.1</flink.version>
        <hadoop.version>2.7.5</hadoop.version>
    </properties>

    <dependencies>

        <!--Flink TableAPI-->
        <dependency>
            <groupId>org.apache.flink</groupId>
            <artifactId>flink-table_${scala.version}</artifactId>
            <version>${flink.version}</version>
        </dependency>
        <!--导入scala的依赖-->
        <dependency>
            <groupId>org.apache.flink</groupId>
            <artifactId>flink-scala_${scala.version}</artifactId>
            <version>${flink.version}</version>
        </dependency>

        <dependency>
            <groupId>org.apache.flink</groupId>
            <artifactId>flink-connector-kafka-0.10_${scala.version}</artifactId>
            <version>${flink.version}</version>
        </dependency>

        <!--模块二 流处理-->
        <dependency>
            <groupId>org.apache.flink</groupId>
            <artifactId>flink-streaming-scala_${scala.version}</artifactId>
            <version>${flink.version}</version>
        </dependency>
        <dependency>
            <groupId>org.apache.flink</groupId>
            <artifactId>flink-streaming-java_${scala.version}</artifactId>
            <version>${flink.version}</version>
        </dependency>

        <!--对象和json 互相转换的-->
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>fastjson</artifactId>
            <version>1.2.44</version>
        </dependency>

        <dependency>
            <groupId>org.apache.flink</groupId>
            <artifactId>flink-runtime-web_2.11</artifactId>
            <version>${flink.version}</version>
        </dependency>

        <dependency>
            <groupId>org.apache.hadoop</groupId>
            <artifactId>hadoop-client</artifactId>
            <version>${hadoop.version}</version>
        </dependency>

        <!-- 指定mysql-connector的依赖 -->
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>5.1.38</version>
        </dependency>

    </dependencies>


    <build>
        <sourceDirectory>src/main/scala</sourceDirectory>
        <testSourceDirectory>src/test/scala</testSourceDirectory>
        <plugins>

            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>2.5.1</version>
                <configuration>
                    <source>${maven.compiler.source}</source>
                    <target>${maven.compiler.target}</target>
                    <!--<encoding>${project.build.sourceEncoding}</encoding>-->
                </configuration>
            </plugin>

            <plugin>
                <groupId>net.alchim31.maven</groupId>
                <artifactId>scala-maven-plugin</artifactId>
                <version>3.2.0</version>
                <executions>
                    <execution>
                        <goals>
                            <goal>compile</goal>
                            <goal>testCompile</goal>
                        </goals>
                        <configuration>
                            <args>
                                <!--<arg>-make:transitive</arg>-->
                                <arg>-dependencyfile</arg>
                                <arg>${project.build.directory}/.scala_dependencies</arg>
                            </args>

                        </configuration>
                    </execution>
                </executions>
            </plugin>


            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-shade-plugin</artifactId>
                <version>2.3</version>
                <executions>
                    <execution>
                        <phase>package</phase>
                        <goals>
                            <goal>shade</goal>
                        </goals>
                        <configuration>
                            <filters>
                                <filter>
                                    <artifact>*:*</artifact>
                                    <excludes>
                                        <!--
                                        zip -d learn_spark.jar META-INF/*.RSA META-INF/*.DSA META-INF/*.SF
                                        -->
                                        <exclude>META-INF/*.SF</exclude>
                                        <exclude>META-INF/*.DSA</exclude>
                                        <exclude>META-INF/*.RSA</exclude>
                                    </excludes>
                                </filter>
                            </filters>
                            <transformers>
                                <transformer
                                        implementation="org.apache.maven.plugins.shade.resource.ManifestResourceTransformer">
                                    <mainClass>com.itheima.batch.BatchFromCollection</mainClass>
                                </transformer>
                            </transformers>
                        </configuration>
                    </execution>

                </executions>
            </plugin>

            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-jar-plugin</artifactId>
                <version>2.6</version>
                <configuration>
                    <archive>
                        <manifest>
                            <addClasspath>true</addClasspath>
                            <classpathPrefix>lib/</classpathPrefix>
                            <mainClass>com.itheima.env.BatchRemoteEven</mainClass>
                        </manifest>
                    </archive>
                </configuration>
            </plugin>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-dependency-plugin</artifactId>
                <version>2.10</version>
                <executions>
                    <execution>
                        <id>copy-dependencies</id>
                        <phase>package</phase>
                        <goals>
                            <goal>copy-dependencies</goal>
                        </goals>
                        <configuration>
                            <outputDirectory>${project.build.directory}/lib</outputDirectory>
                        </configuration>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>

</project>
```

### Flink程序结构

Flink 应用程序结构主要包含三部分,Source/Transformation/Sink

- Source: 数据源，Flink 在流处理和批处理上的 source 大概有 4 类：
  - 基于本地集合的 source
  - 基于文件的 source
  - 基于网络套接字的 source
  - 自定义的 source。自定义的 source 常见的有 Apache kafka、Amazon Kinesis Streams、RabbitMQ、Twitter Streaming API、Apache NiFi 等，当然你也可以定义自己的 source。

- Transformation：数据转换的各种操作，有 Map / FlatMap / Filter / KeyBy / Reduce / Fold / Aggregations / Window / WindowAll / Union / Window join / Split / Select / Project 等，操作很多，可以将数据转换计算成你想要的数据。


- Sink：接收器，Flink 将转换计算后的数据发送的地点 ，你可能需要存储下来，Flink 常见的 Sink 大概有如下几类：
  - 写入文件、
  - 打印输出、
  - 写入 socket 、
  - 自定义的 sink 。自定义的 sink 常见的有 Apache kafka、RabbitMQ、MySQL、ElasticSearch、Apache Cassandra、Hadoop FileSystem 等，同理你也可以定义自己的 Sink。

### DataSource

DataSource：数据来源。Flink 做为一款流式计算框架，它可用来做批处理，即处理静态的数据集、历史的数据集；也可以用来做流处理，即实时的处理些实时数据流，实时的产生数据流结果，只要数据源源不断的过来，Flink 就能够一直计算下去，这个 Data Sources 就是数据的来源地。

Flink在批处理中常见的source主要有两大类。

-   基于本地集合的source（Collection-based-source）

-   基于文件的source（File-based-source）


#### 基于本地集合的Source

在Flink中最常见的创建本地集合的DataSet方式有三种。

1.  使用env.fromElements()，这种方式也支持Tuple，自定义对象等复合形式。

2.  使用env.fromCollection(),这种方式支持多种Collection的具体类型。

3.  使用env.generateSequence(),这种方法创建基于Sequence的DataSet

```scala
import org.apache.flink.api.scala.ExecutionEnvironment
import scala.collection.mutable
import scala.collection.mutable.{ArrayBuffer, ListBuffer}

/**
  * 读取集合中的批次数据
  */
object BatchFromCollection {
  def main(args: Array[String]): Unit = {

    //获取flink执行环境
    val env = ExecutionEnvironment.getExecutionEnvironment

    //导入隐式转换
    import org.apache.flink.api.scala._

    //0.用element创建DataSet(fromElements)
    val ds0: DataSet[String] = env.fromElements("spark", "flink")
    ds0.print()

    //1.用Tuple创建DataSet(fromElements)
    val ds1: DataSet[(Int, String)] = env.fromElements((1, "spark"), (2, "flink"))
    ds1.print()

    //2.用Array创建DataSet
    val ds2: DataSet[String] = env.fromCollection(Array("spark", "flink"))
    ds2.print()

    //3.用ArrayBuffer创建DataSet
    val ds3: DataSet[String] = env.fromCollection(ArrayBuffer("spark", "flink"))
    ds3.print()

    //4.用List创建DataSet
    val ds4: DataSet[String] = env.fromCollection(List("spark", "flink"))
    ds4.print()

    //5.用List创建DataSet
    val ds5: DataSet[String] = env.fromCollection(ListBuffer("spark", "flink"))
    ds5.print()

    //6.用Vector创建DataSet
    val ds6: DataSet[String] = env.fromCollection(Vector("spark", "flink"))
    ds6.print()

    //7.用Queue创建DataSet
    val ds7: DataSet[String] = env.fromCollection(mutable.Queue("spark", "flink"))
    ds7.print()

    //8.用Stack创建DataSet
    val ds8: DataSet[String] = env.fromCollection(mutable.Stack("spark", "flink"))
    ds8.print()

    //9.用Stream创建DataSet（Stream相当于lazy List，避免在中间过程中生成不必要的集合）
    val ds9: DataSet[String] = env.fromCollection(Stream("spark", "flink"))
    ds9.print()

    //10.用Seq创建DataSet
    val ds10: DataSet[String] = env.fromCollection(Seq("spark", "flink"))
    ds10.print()

    //11.用Set创建DataSet
    val ds11: DataSet[String] = env.fromCollection(Set("spark", "flink"))
    ds11.print()

    //12.用Iterable创建DataSet
    val ds12: DataSet[String] = env.fromCollection(Iterable("spark", "flink"))
    ds12.print()

    //13.用ArraySeq创建DataSet
    val ds13: DataSet[String] = env.fromCollection(mutable.ArraySeq("spark", "flink"))
    ds13.print()

    //14.用ArrayStack创建DataSet
    val ds14: DataSet[String] = env.fromCollection(mutable.ArrayStack("spark", "flink"))
    ds14.print()

    //15.用Map创建DataSet
    val ds15: DataSet[(Int, String)] = env.fromCollection(Map(1 -> "spark", 2 -> "flink"))
    ds15.print()

    //16.用Range创建DataSet
    val ds16: DataSet[Int] = env.fromCollection(Range(1, 9))
    ds16.print()

    //17.用fromElements创建DataSet
    val ds17: DataSet[Long] = env.generateSequence(1, 9)
    ds17.print()
  }
}
```

#### 基于文件的Source

Flink支持直接从外部文件存储系统中读取文件的方式来创建Source数据源,Flink支持的方式有以下几种:

1.  读取本地文件数据
2.  读取HDFS文件数据
3.  读取CSV文件数据
4.  读取压缩文件
5.  遍历目录

##### 读取本地文件

```scala
import org.apache.flink.api.scala.{DataSet, ExecutionEnvironment, GroupedDataSet}

/**
  * 读取文件中的批次数据
  */
object BatchFromFile {
  def main(args: Array[String]): Unit = {
    //使用readTextFile读取本地文件
    //初始化环境
    val environment: ExecutionEnvironment = ExecutionEnvironment.getExecutionEnvironment

    //加载数据
    val datas: DataSet[String] = environment.readTextFile("D://intellij-workspace//hadoop//flink//src//main//resources//data.txt")

    //触发程序执行
    datas.print()
  }
}
```

##### 读取HDFS文件数据

```scala
import org.apache.flink.api.scala.{DataSet, ExecutionEnvironment, GroupedDataSet}

/**
  * 读取文件中的批次数据
  */
object BatchFromFile {
  def main(args: Array[String]): Unit = {
    //使用readTextFile读取本地文件
    //初始化环境
    val environment: ExecutionEnvironment = ExecutionEnvironment.getExecutionEnvironment
    //加载数据 
    val datas: DataSet[String] = environment.readTextFile("hdfs://server01:8020/README.txt")
    //触发程序执行
    datas.print()
  }
}
```

##### 读取CSV文件数据

```scala
package org.duo.batch

import org.apache.flink.api.scala._

/**
 * 读取CSV文件中的批次数据
 */
object BatchFromCsvFile {

  def main(args: Array[String]): Unit = {

    // env
    val env: ExecutionEnvironment = ExecutionEnvironment.getExecutionEnvironment

    // 加载CSV文件, csv是以,分割的文本内容
    case class Subject(id:Long,name:String)
    val caseClassDataSet: DataSet[Subject] = env.readCsvFile[Subject]("D:\\intellij-workspace\\hadoop\\flink\\src\\main\\resources\\subject.csv")

    caseClassDataSet.print()
  }
}
```

##### 读取压缩文件

对于以下压缩类型，不需要指定任何额外的inputformat方法，flink可以自动识别并且解压。但是，压缩文件可能不会并行读取，可能是顺序读取的，这样可能会影响作业的可伸缩性。

| 压缩格式 | 扩展名    | 并行化 |
| -------- | --------- | ------ |
| DEFLATE  | .deflate  | no     |
| GZIP     | .gz .gzip | no     |
| Bzip2    | .bz2      | no     |
| XZ       | .xz       | no     |

```scala
package org.duo.batch

import org.apache.flink.api.scala.ExecutionEnvironment

/**
 * 读取压缩文件的数据
 */
object BatchFromCompressFile {

  def main(args: Array[String]): Unit = {
    //初始化环境
    val env = ExecutionEnvironment.getExecutionEnvironment
    //加载数据
    val result = env.readTextFile("D:\\intellij-workspace\\hadoop\\flink\\src\\main\\resources\\wordcount.txt.gz")
    //触发程序执行
    result.print()
  }
}
```

##### 遍历目录

flink支持对一个文件目录内的所有文件，包括所有子目录中的所有文件的遍历访问方式。对于从文件中读取数据，当读取的数个文件夹的时候，嵌套的文件默认是不会被读取的，只会读取第一个文件，其他的都会被忽略。所以我们需要使用recursive.file.enumeration进行递归读取

```scala
package org.duo.batch

import org.apache.flink.api.scala.{DataSet, ExecutionEnvironment}
import org.apache.flink.configuration.Configuration

/**
 * 遍历目录的批次数据
 */
object BatchFromFolder {

  def main(args: Array[String]): Unit = {

    // env
    val env: ExecutionEnvironment = ExecutionEnvironment.getExecutionEnvironment

    // 读取目录
    def params: Configuration = new Configuration()
    params.setBoolean("recursive.file.enumeration",true)
    val folderDataSet: DataSet[String] = env.readTextFile("D:\\intellij-workspace\\hadoop\\flink\\src\\main\\resources\\").withParameters(params)

    folderDataSet.print()

  }
}
```

### Transformation

| ransformation   | 说明                                                         |
| --------------- | ------------------------------------------------------------ |
| map             | 将DataSet中的每一个元素转换为另外一个元素                    |
| flatMap         | 将DataSet中的每一个元素转换为0...n个元素                     |
| mapPartition    | 将一个分区中的元素转换为另一个元素                           |
| filter          | 过滤出来一些符合条件的元素                                   |
| reduce          | 可以对一个dataset或者一个group来进行聚合计算，最终聚合成一个元素 |
| reduceGroup     | 将一个dataset或者一个group聚合成一个或多个元素               |
| aggregate       | 按照内置的方式来进行聚合。例如：SUM/MIN/MAX..                |
| distinct        | 去重                                                         |
| join            | 将两个DataSet按照一定条件连接到一起，形成新的DataSet         |
| union           | 将两个DataSet取并集，并不会去重                              |
| rebalance       | 让每个分区的数据均匀分布，避免数据倾斜                       |
| partitionByHash | 按照指定的key进行hash分区                                    |
| sortPartition   | 指定字段对分区中的数据进行排序                               |

#### map

将DataSet中的每一个元素转换为另外一个元素

```scala
package org.duo.trans

import org.apache.flink.api.scala._

/**
 * 使用map操作，将以下数据
 *
 * "1,张三", "2,李四", "3,王五", "4,赵六"
 *
 * 转换为一个scala的样例类。
 *
 * map是一对一
 */

object MapTrans {

  case class Person(id: String, name: String)

  def main(args: Array[String]): Unit = {

    // env
    val env: ExecutionEnvironment = ExecutionEnvironment.getExecutionEnvironment

    // 加载本地集合
    val listDataSet: DataSet[String] = env.fromCollection(List("1,张三", "2,李四", "3,王五", "4,赵六"))

    // map
    val personDataSet: DataSet[Person] = listDataSet.map {
      text: String =>
        val arr: Array[String] = text.split(",")
        Person(arr(0), arr(1))
    }

    // 打印
    personDataSet.print()
  }
}
```

#### flatMap

将DataSet中的每一个元素转换为0...n个元素

```scala
package org.duo.trans

import org.apache.flink.api.scala._

/**
 * 张三,中国,江西省,南昌市
 *
 * 张三,中国
 * 张三,中国江西省
 * 张三,中国江西省南昌市
 */
object FlatmapTrans {

  def main(args: Array[String]): Unit = {

    // 批处理环境
    val env: ExecutionEnvironment = ExecutionEnvironment.getExecutionEnvironment

    // 去加载本地数据
    val listDataSet: DataSet[String] = env.fromCollection(List(
      "张三,中国,江西省,南昌市",
      "李四,中国,河北省,石家庄市",
      "Tom,America,NewYork,Manhattan"
    ))

    // 使用flatmap进行转换
    val dataSet: DataSet[Product] = listDataSet.flatMap {
      text => {
        var array = text.split(",")
        List(
          (array(0), array(1)),
          (array(0), array(1), array(2)),
          (array(0), array(1), array(2), array(3)))
      }
    }
    dataSet.print()
  }
}
```

#### mapPartition

将一个分区中的元素转换为另一个元素

```scala
package org.duo.trans

import org.apache.flink.api.scala._

/**
 * 使用mapPartition操作，将以下数据
 *
 * "1,张三", "2,李四", "3,王五", "4,赵六"
 *
 * 转换为一个scala的样例类。
 */

/**
 * 开发步骤:
 * 1. 创建样例类 Person(id,name)
 * 2. 创建ENV
 * 3. 加载本地集合
 * 4. mapPartition
 * 5. 打印
 */
case class Person(id: String, name: String)

object MapPartitionTrans {

  def main(args: Array[String]): Unit = {

    val env: ExecutionEnvironment = ExecutionEnvironment.getExecutionEnvironment

    val listDataSet: DataSet[String] = env.fromCollection(List("1,张三", "2,李四", "3,王五", "4,赵六"))

    // mapPartition
    val mapDataSet: DataSet[Person] = listDataSet.mapPartition {

      // 开启redis或者mysql的链接
      iterable => {

        iterable.map {
          text =>
            val arrs: Array[String] = text.split(",")
            Person(arrs(0), arrs(1))
        }
      }
      // 关闭redis或者mysql的链接
    }
    mapDataSet.print()
  }
}
```

#### filter

过滤出来一些符合条件的元素

```scala
package org.duo.trans

import org.apache.flink.api.scala._

/**
 * 过滤出来以下以`h`开头的单词。
 *
 * "hadoop", "hive", "spark", "flink"
 */
object FilterTrans {

  def main(args: Array[String]): Unit = {
    // 1. 创建批处理环境
    val env: ExecutionEnvironment = ExecutionEnvironment.getExecutionEnvironment

    // 2. Source 加载本地集合
    val words: DataSet[String] = env.fromCollection(List("hadoop", "hive", "spark", "flink"))

    // 3. Trans filter 过滤出来以h开头的单词
    val h_words: DataSet[String] = words.filter(_.startsWith("h"))

    // 4. sink 打印输出
    h_words.print()
  }
}
```

#### reduce

可以对一个dataset或者一个group来进行聚合计算，最终聚合成一个元素

```scala
package org.duo.trans

import org.apache.flink.api.scala._

/**
 * 请将以下元组数据，使用`reduce`操作聚合成一个最终结果
 *
 * ("java" , 1) , ("java", 1) ,("java" , 1)
 *
 * 将上传元素数据转换为`("java",3)`
 */
object ReduceTrans {

  def main(args: Array[String]): Unit = {

    // env
    val env: ExecutionEnvironment = ExecutionEnvironment.getExecutionEnvironment
    // load list
    val list: DataSet[(String, Int)] = env.fromCollection(List(("java", 1), ("java", 1), ("java", 1)))

    // reduce
    // p1 ("java" , 1) p2 ("java", 1) ("java",2)
    // p1 ("java",2) p2 ("java" , 1) ("java",3)
    val reduceDataSet: DataSet[(String, Int)] = list.reduce {
      (p1, p2) => (p1._1, p1._2 + p2._2)
    }

    // print
    reduceDataSet.print()
  }
}
```

#### reduceGroup

可以对一个dataset或者一个group来进行聚合计算，最终聚合成一个元素

reduce和reduceGroup的区别

* reduce是将数据一个个拉取到另外一个节点，然后再执行计算
* reduceGroup是先在每个group所在的节点上执行计算，然后再拉取

```scala
package org.duo.trans

import org.apache.flink.api.scala._

/**
 * 请将以下元组数据，下按照单词使用`groupBy`进行分组，再使用`reduceGroup`操作进行单词计数
 *
 * ("java" , 1) , ("java", 1) ,("scala" , 1)
 */
object ReduceGroupTrans {

  def main(args: Array[String]): Unit = {

    // env
    val env = ExecutionEnvironment.getExecutionEnvironment
    // load list
    val list: DataSet[(String, Int)] = env.fromCollection(List(("java", 1), ("java", 1), ("scala", 1)))

    // groupby
    val groupDataSet: GroupedDataSet[(String, Int)] = list.groupBy(_._1)

    // reduceGroup
    val reduceDataSet: DataSet[(String, Int)] = groupDataSet.reduceGroup {
      iter =>
        iter.reduce {
          (p1, p2) => (p1._1, p1._2 + p2._2)
        }
    }

    // print
    reduceDataSet.print()
  }

}
```

#### aggregate

按照内置的方式来进行聚合, Aggregate只能作用于元组上。例如：SUM/MIN/MAX..

```scala
package org.duo.trans

import org.apache.flink.api.java.aggregation.Aggregations
import org.apache.flink.api.scala._

/**
  * 请将以下元组数据，使用`aggregate`操作进行单词统计
  *
  * ("java" , 1) , ("java", 1) ,("scala" , 1)
  */
object AggregateTrans {

  def main(args: Array[String]): Unit = {

    // env
    val env = ExecutionEnvironment.getExecutionEnvironment
    // load list
    val list: DataSet[(String, Int)] = env.fromCollection(List(("java", 1), ("java", 1), ("scala", 1)))

    // groupby
    val groupDataSet: GroupedDataSet[(String, Int)] = list.groupBy(0)

    // aggregate
    val aggregateDataSet: AggregateDataSet[(String, Int)] = groupDataSet.aggregate(Aggregations.SUM,1)

    // print
    aggregateDataSet.print()

  }

}
```

要使用aggregate，只能使用字段索引名或索引名称来进行分组groupBy(0)，否则会报错

#### distinct

去除重复的数据

```scala
package org.duo.trans

import org.apache.flink.api.scala._

/**
 * 请将以下元组数据，使用`distinct`操作去除重复的单词
 *
 * ("java" , 1) , ("java", 2) ,("scala" , 1)
 *
 * 去重得到
 * ("java", 1), ("scala", 1)
 */
object DistinctTrans {

  def main(args: Array[String]): Unit = {

    // env
    val env = ExecutionEnvironment.getExecutionEnvironment

    // load list
    val list: DataSet[(String, Int)] = env.fromCollection(List(("java", 1), ("java", 1), ("java", 2), ("scala", 1)))

    // distinct 0: 根据元组的第一个元素进行去重
    val distinctDataSet: DataSet[(String, Int)] = list.distinct(0, 1)
    // print
    distinctDataSet.print()
  }
}
```

#### join

使用join可以将两个DataSet连接起来

```scala
package org.duo.trans

import org.apache.flink.api.scala._

/**
 * 两个CSV文件 进行join
 */

case class Score(id: String, name: String, subjecid: String, score: String)

case class Subject(id: String, name: String)

object JoinTrans {

  def main(args: Array[String]): Unit = {
    // env
    val env: ExecutionEnvironment = ExecutionEnvironment.getExecutionEnvironment

    // readCsvFile
    val scoreDataSet: DataSet[Score] = env.readCsvFile[Score]("D://intellij-workspace//hadoop//flink//src//main//resources//score.csv")
    val subjectDataSet: DataSet[Subject] = env.readCsvFile[Subject]("D://intellij-workspace//hadoop//flink//src//main//resources//subject.csv")

    // join
    // A.join(B).where(A的哪一个元素).equalTo(和B的哪个元素相等)
    val joinDataSet: JoinDataSet[Score, Subject] = scoreDataSet.join(subjectDataSet).where(2).equalTo(0)

    // print
    joinDataSet.print()
  }
}
```



#### union

将两个DataSet取并集，不会去重。

```scala
package org.duo.trans

import org.apache.flink.api.scala._

/**
 * 将以下数据进行取并集操作
 *
 * 数据集1
 *
 * "hadoop", "hive", "flume"
 *
 * 数据集2
 *
 * "hadoop", "hive", "spark"
 */
object UnionTrans {

  def main(args: Array[String]): Unit = {
    
    // env
    val env = ExecutionEnvironment.getExecutionEnvironment

    // load list
    val dataset1: DataSet[String] = env.fromCollection(List("hadoop", "hive", "flume"))
    val dataset2: DataSet[String] = env.fromCollection(List("hadoop", "hive", "spark"))

    // union
    val newDataSet: DataSet[String] = dataset1.union(dataset2).distinct()

    // print
    newDataSet.print()
  }

}
```

#### rebalance

Flink会产生数据倾斜，rebalance会使用轮询的方式将数据均匀打散，这是处理数据倾斜最好的选择。

```scala
package org.duo.trans

import org.apache.flink.api.common.functions.RichMapFunction
import org.apache.flink.api.scala._

/**
 * 1. 构建批处理运行环境
 * 2. 使用`env.generateSequence`创建0-100的并行数据
 * 3. 使用`fiter`过滤出来`大于8`的数字
 * 4. 使用map操作传入`RichMapFunction`，将当前子任务的ID和数字构建成一个元组
 * 在RichMapFunction中可以使用`getRuntimeContext.getIndexOfThisSubtask`获取子任务序号
 * 5. 打印测试
 */
object RebanlanceTrans {

  def main(args: Array[String]): Unit = {

    // 1. 构建批处理运行环境
    val env = ExecutionEnvironment.getExecutionEnvironment
    //   2. 使用`env.generateSequence`创建0-100的并行数据
    val seqDataSet: DataSet[Long] = env.generateSequence(0, 100)

    //   3. 使用`fiter`过滤出来`大于8`的数字
    // 如果不进行rebanlance，那么数据在各分区的分布是不均匀的
    // val filterDataSet: DataSet[Long] = seqDataSet.filter(_ > 8)
    // rebanlance会使用轮询的方式将数据均匀打散，按照数据的哈希模以并行度（Parallelism），并行度未设置的情况下默认为cpu核数
    val filterDataSet: DataSet[Long] = seqDataSet.filter(_ > 8).rebalance()
    //   4. 使用map操作传入`RichMapFunction`，将当前子任务的ID和数字构建成一个元组
    // 在RichMapFunction中可以使用`getRuntimeContext.getIndexOfThisSubtask`获取子任务序号
    val mapDataSet: DataSet[(Int, Long)] = filterDataSet.map(new RichMapFunction[Long, (Int, Long)] {
      // value: 是每次遍历的数字
      override def map(value: Long): (Int, Long) = {
        (getRuntimeContext.getIndexOfThisSubtask, value)
      }
    })

    // 5. 打印测试
    mapDataSet.print()
  }

}
```

#### PartitionByHash

按照指定的key进行hash分区

```scala
package org.duo.trans

import org.apache.flink.api.scala._

/**
  * 基于以下列表数据来创建数据源，并按照hashPartition进行分区，然后输出到文件。
  *
  * List(1,1,1,1,1,1,1,2,2,2,2,2)
  */
object PartitionByHashTrans {

  def main(args: Array[String]): Unit = {

    // 1. 创建批处理环境
    val env = ExecutionEnvironment.getExecutionEnvironment

    // 2. 设置并行度为2
    env.setParallelism(2)

    // 3. 加载本地集合
    val listDataSet: DataSet[Int] = env.fromCollection(List(1,1,1,1,1,1,1,2,2,2,2,2))

    // 4. 进行分区
    val hashDataSet: DataSet[Int] = listDataSet.partitionByHash(_.toString)

    // 5. 写入文件
    hashDataSet.writeAsText("./data/patitions")

    // 6. 打印输出
    hashDataSet.print()

  }
}
```

```scala
package org.duo.trans

import org.apache.flink.api.scala._

/**
 * 基于以下列表数据来创建数据源，并按照hashPartition进行分区，然后输出到文件。
 *
 * List(1,1,1,1,1,1,1,2,2,2,2,2)
 */
object PartitionByHashTrans_Tuple {

  def main(args: Array[String]): Unit = {

    // 1. 创建批处理环境
    val env = ExecutionEnvironment.getExecutionEnvironment

    // 2. 设置并行度为2
    env.setParallelism(2)

    // 3. 加载本地集合
    val listDataSet = env.fromCollection(
      List(("flink", 1), ("spark", 2), ("flume", 3)))

    // 4. 进行分区
    val hashDataSet: DataSet[(String, Int)] = listDataSet.partitionByHash(0)

    // 5. 写入文件
    hashDataSet.writeAsText("./data/patitions")

    // 6. 打印输出
    hashDataSet.print()

  }
}
```

#### sortPartition

指定字段对分区中的数据进行排序

```scala
package org.duo.trans

import org.apache.flink.api.common.operators.Order
import org.apache.flink.api.scala._

/**
  * 按照以下列表来创建数据集
  *
  * List("hadoop", "hadoop", "hadoop", "hive", "hive", "spark", "spark", "flink")
  *
  * 对分区进行排序后，输出到文件。
  */
object SortPartitionTrans {

  def main(args: Array[String]): Unit = {

    // 1. 创建批处理环境
    val env = ExecutionEnvironment.getExecutionEnvironment

    env.setParallelism(1)

    // 2. 加载本地集合
    val listDataSet: DataSet[String] = env.fromCollection(List("hadoop", "hadoop", "hadoop", "hive", "hive", "spark", "spark", "flink"))

    // 3. 排序
    val sortDataSet: DataSet[String] = listDataSet.sortPartition((x: String) =>x,Order.DESCENDING)

    // 4. 写入文件
    sortDataSet.writeAsText("./data/sort_output")

    // 5. 打印输出
    sortDataSet.print()

  }

}
```

### Sink

#### 基于本地集合

```scala
package org.duo.sink

import org.apache.flink.api.scala._

/**
  * 基于下列数据,分别 进行打印输出,error输出,collect()
  *
  * ```
  * (19, "zhangsan", 178.8),
  * (17, "lisi", 168.8),
  * (18, "wangwu", 184.8),
  * (21, "zhaoliu", 164.8)
  * ```
  */
object BatchSinkCollection {

  def main(args: Array[String]): Unit = {
    // 1. env
    val env=ExecutionEnvironment.getExecutionEnvironment

    // 2. 加载集合
    val listDataSet: DataSet[(Int, String, Double)] = env.fromCollection(List(
      (19, "zhangsan", 178.8),
      (17, "lisi", 168.8),
      (18, "wangwu", 184.8),
      (21, "zhaoliu", 164.8)))

    // 3. 打印输出 错误输出 collect
    listDataSet.print()

    listDataSet.printToErr()

    val tuples: Seq[(Int, String, Double)] = listDataSet.collect()

    println(tuples)

  }
}
```

#### 基于文件

-   flink支持多种存储设备上的文件，包括本地文件，hdfs文件等。

-   flink支持多种文件的存储格式，包括text文件，CSV文件等。

-   writeAsText()：TextOuputFormat - 将元素作为字符串写入行。字符串是通过调用每个元素的toString()方法获得的。

```scala
package org.duo.sink

import org.apache.flink.api.scala._
import org.apache.flink.core.fs.FileSystem

/**
  * 基于下列数据,写入到文件中
  *
  * Map(1 -> "spark", 2 -> "flink")
  */
object BatchSinkFile {

  def main(args: Array[String]): Unit = {

    // 1. env
    val env= ExecutionEnvironment.getExecutionEnvironment

    // 2. load map
    val mapDataSet: DataSet[(Int, String)] = env.fromCollection(Map(1 -> "spark", 2 -> "flink"))

    // 3. write as text setParallelism(4) 如果并行度大于1 会输出多个文件,test就是一个目录,如果并行度为1,那么test就是文件
    mapDataSet.setParallelism(4).writeAsText("./data/sink/test",FileSystem.WriteMode.OVERWRITE)

    // 4. 执行程序

    mapDataSet.print()
    env.execute("sinkFile")
  }

}
```

写入hdfs中

```scala
import org.apache.flink.api.scala.{DataSet, ExecutionEnvironment}
import org.apache.flink.core.fs.FileSystem.WriteMode
import org.apache.flink.api.scala._

/**
  * 将数据写入本地文件
  */
object BatchSinkFile {
    
  def main(args: Array[String]): Unit = {
      
    //1.定义环境
    val env = ExecutionEnvironment.getExecutionEnvironment

    //2.定义数据 stu(age,name,height)
    val stu: DataSet[(Int, String, Double)] = env.fromElements(
      (19, "zhangsan", 178.8),
      (17, "lisi", 168.8),
      (18, "wangwu", 184.8),
      (21, "zhaoliu", 164.8)
    )
    val ds1: DataSet[Map[Int, String]] = env.fromElements(Map(1 -> "spark", 2 -> "flink"))
    //1.TODO 写入到本地，文本文档,NO_OVERWRITE模式下如果文件已经存在，则报错，OVERWRITE模式下如果文件已经存在，则覆盖 
    ds1.setParallelism(1).writeAsText("hdfs://server01:8020/a", WriteMode.OVERWRITE)
    env.execute()
  }
}
```

## 执行环境

### 本地执行

Flink支持两种不同的本地执行：

1. LocalExecutionEnvironment 是启动完整的Flink运行时（Flink Runtime），包括 JobManager 和 TaskManager 。 这种方式包括内存管理和在集群模式下执行的所有内部算法。
2. CollectionEnvironment是在 Java 集合（Java Collections）上执行 Flink 程序。 此模式不会启动完整的Flink运行时（Flink Runtime），因此执行的开销非常低并且轻量化。 例如一个DataSet.map()变换，会对Java list中所有元素应用 map()函数。

#### local环境

LocalEnvironment是Flink程序本地执行的句柄。可使用它，独立或嵌入其他程序在本地JVM中运行Flink程序。本地环境通过该方法实例化ExecutionEnvironment.createLocalEnvironment()。默认情况下，启动的本地线程数与计算机的CPU个数相同。也可以指定所需的并行性。本地环境可以配置为使用enableLogging()/登录到控制台disableLogging()。在大多数情况下，ExecutionEnvironment.getExecutionEnvironment()是更好的方式。LocalEnvironment当程序在本地启动时（命令行界面外），该方法会返回一个程序，并且当程序由命令行界面调用时，它会返回一个预配置的群集执行环境。

```scala
package org.duo.en

import java.util.Date

import org.apache.flink.api.scala._

object BatchLocalEven {

  def main(args: Array[String]): Unit = {
    // 开始时间
    val startTime = new Date().getTime
    // env
    val localEnv: ExecutionEnvironment = ExecutionEnvironment.createLocalEnvironment(2)
    val collectEnv: ExecutionEnvironment = ExecutionEnvironment.createCollectionsEnvironment

    // load list
    val listDataSet: DataSet[Int] = localEnv.fromCollection(List(1, 2, 3, 4))

    // print
    listDataSet.print()
    // 开始时间
    val endTime = new Date().getTime
    println(endTime - startTime)
  }

}
```

#### 集合环境

使用集合的执行CollectionEnvironment是执行Flink程序的低开销方法。这种模式的典型用例是自动化测试，调试和代码重用。用户也可以使用为批处理实施的算法，以便更具交互性的案例。请注意，基于集合的Flink程序的执行仅适用于适合JVM堆的小数据。集合上的执行不是多线程的，只使用一个线程

```scala
package org.duo.env

import org.apache.flink.api.scala.{DataSet, ExecutionEnvironment, createTypeInformation}

import java.util.Date

/**
 * local环境
 */
object BatchCollectionsEven {

  def main(args: Array[String]): Unit = {
    // 开始时间
    var start_time = new Date().getTime
    //TODO 初始化本地执行环境
    val env = ExecutionEnvironment.createCollectionsEnvironment
    val list: DataSet[String] = env.fromCollection(List("1", "2"))

    list.print()

    // 结束时间
    var end_time = new Date().getTime
    println(end_time - start_time) //单位毫秒
  }
}
```

### 集群执行

通过IDE,直接在远程环境上执行Flink Java程序。

使用命令行提交

```bash
flink run ./examples/batch/WordCount.jar   --input file:///home/user/hamlet.txt --output file:///home/user/wordcount_out
```

通过IDE,直接在远程环境上执行Flink Java程序。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>hadoop</artifactId>
        <groupId>org.duo</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>flink</artifactId>

    <properties>
        <maven.compiler.source>8</maven.compiler.source>
        <maven.compiler.target>8</maven.compiler.target>
        <encoding>UTF-8</encoding>
        <scala.version>2.11</scala.version>
        <flink.version>1.6.1</flink.version>
        <hadoop.version>2.7.5</hadoop.version>
    </properties>

    <dependencies>

        <!--Flink TableAPI-->
        <dependency>
            <groupId>org.apache.flink</groupId>
            <artifactId>flink-table_${scala.version}</artifactId>
            <version>${flink.version}</version>
        </dependency>
        <!--导入scala的依赖-->
        <dependency>
            <groupId>org.apache.flink</groupId>
            <artifactId>flink-scala_${scala.version}</artifactId>
            <version>${flink.version}</version>
        </dependency>

        <dependency>
            <groupId>org.apache.flink</groupId>
            <artifactId>flink-connector-kafka-0.10_${scala.version}</artifactId>
            <version>${flink.version}</version>
        </dependency>

        <!--模块二 流处理-->
        <dependency>
            <groupId>org.apache.flink</groupId>
            <artifactId>flink-streaming-scala_${scala.version}</artifactId>
            <version>${flink.version}</version>
        </dependency>
        <dependency>
            <groupId>org.apache.flink</groupId>
            <artifactId>flink-streaming-java_${scala.version}</artifactId>
            <version>${flink.version}</version>
        </dependency>

        <!--对象和json 互相转换的-->
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>fastjson</artifactId>
            <version>1.2.44</version>
        </dependency>

        <dependency>
            <groupId>org.apache.flink</groupId>
            <artifactId>flink-runtime-web_2.11</artifactId>
            <version>${flink.version}</version>
        </dependency>

        <dependency>
            <groupId>org.apache.hadoop</groupId>
            <artifactId>hadoop-client</artifactId>
            <version>${hadoop.version}</version>
        </dependency>

        <!-- 指定mysql-connector的依赖 -->
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>5.1.38</version>
        </dependency>

    </dependencies>


    <build>
        <sourceDirectory>src/main/scala</sourceDirectory>
        <testSourceDirectory>src/test/scala</testSourceDirectory>
        <plugins>

            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>2.5.1</version>
                <configuration>
                    <source>${maven.compiler.source}</source>
                    <target>${maven.compiler.target}</target>
                    <!--<encoding>${project.build.sourceEncoding}</encoding>-->
                </configuration>
            </plugin>

            <plugin>
                <groupId>net.alchim31.maven</groupId>
                <artifactId>scala-maven-plugin</artifactId>
                <version>3.2.0</version>
                <executions>
                    <execution>
                        <goals>
                            <goal>compile</goal>
                            <goal>testCompile</goal>
                        </goals>
                        <configuration>
                            <args>
                                <!--<arg>-make:transitive</arg>-->
                                <arg>-dependencyfile</arg>
                                <arg>${project.build.directory}/.scala_dependencies</arg>
                            </args>

                        </configuration>
                    </execution>
                </executions>
            </plugin>


            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-shade-plugin</artifactId>
                <version>2.4.3</version>
                <executions>
                    <execution>
                        <phase>package</phase>
                        <goals>
                            <goal>shade</goal>
                        </goals>
                        <configuration>
                            <filters>
                                <filter>
                                    <artifact>*:*</artifact>
                                    <excludes>
                                        <!--
                                        zip -d learn_spark.jar META-INF/*.RSA META-INF/*.DSA META-INF/*.SF
                                        -->
                                        <exclude>META-INF/*.SF</exclude>
                                        <exclude>META-INF/*.DSA</exclude>
                                        <exclude>META-INF/*.RSA</exclude>
                                    </excludes>
                                </filter>
                            </filters>
                            <transformers>
                                <transformer
                                        implementation="org.apache.maven.plugins.shade.resource.ManifestResourceTransformer">
                                    <mainClass>org.duo.batch.BatchFromCollection</mainClass>
                                </transformer>
                            </transformers>
                        </configuration>
                    </execution>

                </executions>
            </plugin>

            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-jar-plugin</artifactId>
                <version>2.6</version>
                <configuration>
                    <archive>
                        <manifest>
                            <addClasspath>true</addClasspath>
                            <classpathPrefix>lib/</classpathPrefix>
                            <mainClass>org.duo.env.BatchRemoteEnv</mainClass>
                        </manifest>
                    </archive>
                </configuration>
            </plugin>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-dependency-plugin</artifactId>
                <version>2.10</version>
                <executions>
                    <execution>
                        <id>copy-dependencies</id>
                        <phase>package</phase>
                        <goals>
                            <goal>copy-dependencies</goal>
                        </goals>
                        <configuration>
                            <outputDirectory>${project.build.directory}/lib</outputDirectory>
                        </configuration>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>

</project>
```

```scala
package org.duo.env

import org.apache.flink.api.common.operators.Order
import org.apache.flink.api.scala._

object BatchRemoteEnv {

  def main(args: Array[String]): Unit = {

   /**
    * 1.创建远程执行环境。远程环境将程序（部分）发送到集群以执行。请注意，程序中使用的所有文件路径都必须可以从集群中访问。除非通过[[ExecutionEnvironment.setParallelism（）]显式设置并行度，否则执行将使用集群的默认并行度。 
    * @param host  JobManager的ip或域名
    * @param port  JobManager的端口
    * @param jarFiles 包含需要发送到集群的代码的JAR文件。如果程序使用用户定义的函数、用户定义的输入格式或任何库，则必须在JAR文件中提供这些函数。
    */
    val env = ExecutionEnvironment.createRemoteEnvironment("server01", 8081, "D:\\intellij-workspace\\hadoop\\flink\\target\\flink-1.0-SNAPSHOT.jar")

    // 2. 读取hdfs中csv文件,转换为元组
    val csvFile: DataSet[(Long, String, Long, Double)] = env.readCsvFile[(Long, String, Long, Double)]("hdfs://server01:8020/flink-datas/score.csv")

    // 3. 根据元组的姓名分组,以成绩降序,取第一个值
    val result: DataSet[(Long, String, Long, Double)] = csvFile.groupBy(1).sortGroup(3, Order.DESCENDING).first(1)
    //#############################################################################################################################################
    // 如果使用result.print会报错：org.apache.flink.runtime.rest.util.RestClientException: Response was not valid JSON, but plain-text:
    // 原因还没找到
    ////#############################################################################################################################################
    // 4. 打印
    //result.print()

    println(result.toString);
  }
}
```

## 广播变量

Flink支持广播。可以将数据广播到TaskManager上，数据存储到内存中。数据存储在内存中，这样可以减缓大量的shuffle操作；比如在数据join阶段，不可避免的就是大量的shuffle操作，我们可以把其中一个dataSet广播出去，一直加载到taskManager的内存中，可以直接在内存中拿数据，避免了大量的shuffle，导致集群性能下降；广播变量创建后，它可以运行在集群中的任何function上，而不需要多次传递给集群节点。另外需要记住，不应该修改广播变量，这样才能确保每个节点获取到的值都是一致的。一句话解释，可以理解为是一个公共的共享变量，我们可以把一个dataset 数据集广播出去，然后不同的task在节点上都能够获取到，这个数据在每个节点上只会存在一份。如果不使用broadcast，则在每个节点中的每个task中都需要拷贝一份dataset数据集，比较浪费内存(也就是一个节点中可能会存在多份dataset数据)。

![image][8]

* 可以理解广播就是一个公共的共享变量
* 将一个数据集广播后，不同的Task都可以在节点上获取到
* 每个节点只存一份
* 如果不使用广播，每一个Task都会拷贝一份数据集，造成内存资源浪费

用法

* 在需要使用广播的操作后，使用withBroadcastSet创建广播
* 在操作中，使用getRuntimeContext.getBroadcastVariable`[广播数据类型]`(`广播名`)获取广播变量

操作步骤

```scala
1：初始化数据
DataSet<Integer> toBroadcast = env.fromElements(1, 2, 3)
2：广播数据
.withBroadcastSet(toBroadcast, "broadcastSetName");
3：获取数据
Collection<Integer> broadcastSet = getRuntimeContext().getBroadcastVariable("broadcastSetName");
```

示例

创建一个学生数据集，包含以下数据

```html
|学生ID | 姓名 |
|------|------|
List((1, "张三"), (2, "李四"), (3, "王五"))
```

将该数据，发布到广播。

再创建一个成绩数据集， 

```html
|学生ID | 学科 | 成绩 |
|------|------|-----|
List( (1, "语文", 50),(2, "数学", 70), (3, "英文", 86))
```

请通过广播获取到学生姓名，将数据转换为

```html
List( ("张三", "语文", 50),("李四", "数学", 70), ("王五", "英文", 86))
```

**步骤**

1. 获取批处理运行环境
2. 分别创建两个数据集
3. 使用RichMapFunction对成绩数据集进行map转换
4. 在数据集调用map方法后，调用withBroadcastSet将学生数据集创建广播
5. 实现RichMapFunction
   - 将成绩数据(学生ID，学科，成绩) -> (学生姓名，学科，成绩)
   - 重写open方法中，获取广播数据
   - 导入scala.collection.JavaConverters._隐式转换
   - 将广播数据使用asScala转换为Scala集合，再使用toList转换为scala List集合
   - 在map方法中使用广播进行转换
6. 打印测试

```scala
package org.duo.broad

import org.apache.flink.api.common.functions.RichMapFunction
import org.apache.flink.api.scala._
import org.apache.flink.configuration.Configuration

object BroadCastVal {

  def main(args: Array[String]): Unit = {

    // 1. 创建env
    val env = ExecutionEnvironment.getExecutionEnvironment

    // 2. 加载两个集合
    // List((1, "张三"), (2, "李四"), (3, "王五"))  => 广播变量
    // List( (1, "语文", 50),(2, "数学", 70), (3, "英文", 86))
    // List( ("张三", "语文", 50),("李四", "数学", 70), ("王五", "英文", 86))
    val stuDataSet: DataSet[(Int, String)] = env.fromCollection(List((1, "张三"), (2, "李四"), (3, "王五")))
    val scoreDataSet: DataSet[(Int, String, Int)] = env.fromCollection(List((1, "语文", 50), (2, "数学", 70), (3, "英文", 86)))
    // 3. 遍历成绩集合
   /* 所有Flink函数类都有其Rich版本。它与常规函数的不同在于，可以获取运行环境的上下文，并拥有一些生命周期方法，所以可以实现更复杂的功能。也有意味着提供了更多的，更丰富的功能。例如：RichMapFunction
    * 默认生命周期方法, 初始化方法, 在每个并行度上只会被调用一次, 而且先被调用
    * 默认生命周期方法, 最后一个方法, 做一些清理工作, 在每个并行度上只调用一次, 而且是最后被调用
    * getRuntimeContext()方法提供了函数的RuntimeContext的一些信息，例如函数执行的并行度，任务的名字，以及state状态. 开发人员在需要的时候自行调用获取运行时上下文对象.
    */
    // IN  OUT
    val resultDataSet: DataSet[(String, String, Int)] = scoreDataSet.map(new RichMapFunction[(Int, String, Int), (String, String, Int)] {

      var stuList: List[(Int, String)] = null

      override def map(value: (Int, String, Int)): (String, String, Int) = {
        // 在map方法中 我们要进行数据的对比
        // 1. 获取学生ID
        val stuID = value._1

        // 2. 过滤出id相等数据
        val tuples: List[(Int, String)] = stuList.filter((x: (Int, String)) => stuID == x._1)

        // 3. 构建新的元组
        (tuples(0)._2, value._2, value._3)

      }

      // open方法会在map方法之前执行
      override def open(parameters: Configuration): Unit = {

        import scala.collection.JavaConverters._
        // 获取广播变量
        stuList = getRuntimeContext.getBroadcastVariable[(Int, String)]("stuDataSet").asScala.toList
      }
    }).withBroadcastSet(stuDataSet, "stuDataSet")


    // 4. 打印
    resultDataSet.print()


  }

}
```

- 广播出去的变量存放在每个节点的内存中，直到程序结束，这个数据集不能太大
- withBroadcastSet需要在要使用到广播的操作后调用
- 需要手动导入scala.collection.JavaConverters._将Java集合转换为scala集合

## 累加器

Accumulator即累加器，与MapReduce counter的应用场景差不多，都能很好地观察task在运行期间的数据变化，可以在Flink job任务中的算子函数中操作累加器，但是只能在任务执行结束之后才能获得累加器的最终结果。Flink现在有以下内置累加器。每个累加器都实现了Accumulator接口。

1. IntCounter
2. LongCounter
3. DoubleCounter

操作步骤

```scala
1：创建累加器
private IntCounter numLines = new IntCounter(); 
2：注册累加器
getRuntimeContext().addAccumulator("num-lines", this.numLines);
3：使用累加器
this.numLines.add(1); 
4：获取累加器的结果
myJobExecutionResult.getAccumulatorResult("num-lines")
```

示例

遍历下列数据, 打印出单词的总数

```
"a","b","c","d"
```

开发步骤:

1. 获取批处理环境
2. 加载本地集合
3. map转换
   1. 定义累加器
   2. 注册累加器
   3. 累加数据
4. 数据写入到文件中
5. 执行任务,获取任务执行结果对象(JobExecutionResult)
6. 获取累加器数值
7. 打印数值

```scala
package org.duo.broad

import org.apache.flink.api.common.accumulators.IntCounter
import org.apache.flink.api.common.functions.RichMapFunction
import org.apache.flink.api.scala.ExecutionEnvironment
import org.apache.flink.configuration.Configuration

/**
 * counter 累加器
 */
object BatchDemoCounter {

  def main(args: Array[String]): Unit = {
    //获取执行环境
    val env = ExecutionEnvironment.getExecutionEnvironment

    import org.apache.flink.api.scala._

    val data = env.fromElements("a", "b", "c", "d")

    val res = data.map(new RichMapFunction[String, String] {
      //1：定义累加器
      val numLines = new IntCounter

      override def open(parameters: Configuration): Unit = {
        super.open(parameters)
        //2:注册累加器
        getRuntimeContext.addAccumulator("num-lines", this.numLines)
      }

      var sum = 0;

      override def map(value: String) = {
        //如果并行度为1，使用普通的累加求和即可，但是设置多个并行度，则普通的累加求和结果就不准了
        sum += 1;
        System.out.println("sum：" + sum);
        this.numLines.add(1)
        value
      }

    }).setParallelism(1)
    //    res.print();
    res.writeAsText("d:\\data\\count0")
    //#############################################################################
    //env.execute代码报错，解决方案不明
    //java.lang.NoSuchMethodError: akka.actor.ActorSystem.terminate()Lscala/concurrent/Future;
    //#############################################################################
    val jobResult = env.execute("BatchDemoCounterScala")
    //    //3：获取累加器
    val num = jobResult.getAccumulatorResult[Int]("num-lines")
    println("num:" + num)
  }
}
```

## 分布式缓存

提供了一个类似于Hadoop的分布式缓存，让并行运行实例的函数可以在本地访问。这个功能可以被使用来分享外部静态的数据，例如：机器学习的逻辑回归模型等！缓存的使用流程：使用ExecutionEnvironment实例对本地的或者远程的文件（例如：HDFS上的文件）,为缓存文件指定一个名字注册该缓存文件。当程序执行时候，Flink会自动将复制文件或者目录到所有worker节点的本地文件系统中，函数可以根据名字去该节点的本地文件系统中检索该文件！

注意:广播是将变量分发到各个worker节点的内存上，分布式缓存是将文件缓存到各个worker节点上

操作步骤：

```scala
1：注册一个文件
env.registerCachedFile("hdfs:///path/to/your/file", "hdfsFile")  
2：访问数据
File myFile = getRuntimeContext().getDistributedCache().getFile("hdfsFile");
```

示例：

遍历下列数据, 并在open方法中获取缓存的文件

```scala
a,b,c,d
```

```scala
package org.duo.env

import org.apache.commons.io.FileUtils
import org.apache.flink.api.common.functions.RichMapFunction
import org.apache.flink.api.scala.ExecutionEnvironment
import org.apache.flink.configuration.Configuration

object BatchDisCache {

  def main(args: Array[String]): Unit = {

    //获取执行环境
    val env = ExecutionEnvironment.getExecutionEnvironment

    //隐式转换
    import org.apache.flink.api.scala._

    //1:注册文件
    env.registerCachedFile("D:\\intellij-workspace\\bigdata\\flink\\src\\main\\resources\\data.txt", "b.txt")

    //读取数据
    val data = env.fromElements("a", "b", "c", "d")
    val result = data.map(new RichMapFunction[String, String] {

      override def open(parameters: Configuration): Unit = {
        super.open(parameters)
        //访问数据
        val myFile = getRuntimeContext.getDistributedCache.getFile("b.txt")
        val lines = FileUtils.readLines(myFile)
        val it = lines.iterator()
        while (it.hasNext) {
          val line = it.next();
          println("line:" + line)
        }
      }

      override def map(value: String) = {
        value
      }
    })
    result.print()
  }
}
```

## 流处理

### DataSource

Flink中你可以使用StreamExecutionEnvironment.getExecutionEnvironment创建流处理的执行环境；也可以通过StreamExecutionEnvironment.addSource(source)来为你的程序添加数据来源。Flink已经提供了若干实现好了的sourcefunctions，当然也可以通过实现SourceFunction来自定义非并行的source或者实现ParallelSourceFunction接口或者扩展RichParallelSourceFunction来自定义并行的source。

####  基于集合的source

```scala
import org.apache.flink.streaming.api.scala.{DataStream, StreamExecutionEnvironment}
import org.apache.flink.streaming.api.scala._
import scala.collection.immutable.{Queue, Stack}
import scala.collection.mutable
import scala.collection.mutable.{ArrayBuffer, ListBuffer}

object StreamingDemoFromCollectionSource {
    
  def main(args: Array[String]): Unit = {
    val senv = StreamExecutionEnvironment.getExecutionEnvironment
    //0.用element创建DataStream(fromElements)
    val ds0: DataStream[String] = senv.fromElements("spark", "flink")
    ds0.print()

    //1.用Tuple创建DataStream(fromElements)
    val ds1: DataStream[(Int, String)] = senv.fromElements((1, "spark"), (2, "flink"))
    ds1.print()

    //2.用Array创建DataStream
    val ds2: DataStream[String] = senv.fromCollection(Array("spark", "flink"))
    ds2.print()

    //3.用ArrayBuffer创建DataStream
    val ds3: DataStream[String] = senv.fromCollection(ArrayBuffer("spark", "flink"))
    ds3.print()

    //4.用List创建DataStream
    val ds4: DataStream[String] = senv.fromCollection(List("spark", "flink"))
    ds4.print()

    //5.用List创建DataStream
    val ds5: DataStream[String] = senv.fromCollection(ListBuffer("spark", "flink"))
    ds5.print()

    //6.用Vector创建DataStream
    val ds6: DataStream[String] = senv.fromCollection(Vector("spark", "flink"))
    ds6.print()

    //7.用Queue创建DataStream
    val ds7: DataStream[String] = senv.fromCollection(Queue("spark", "flink"))
    ds7.print()

    //8.用Stack创建DataStream
    val ds8: DataStream[String] = senv.fromCollection(Stack("spark", "flink"))
    ds8.print()

    //9.用Stream创建DataStream（Stream相当于lazy List，避免在中间过程中生成不必要的集合）
    val ds9: DataStream[String] = senv.fromCollection(Stream("spark", "flink"))
    ds9.print()

    //10.用Seq创建DataStream
    val ds10: DataStream[String] = senv.fromCollection(Seq("spark", "flink"))
    ds10.print()

    //11.用Set创建DataStream(不支持)
    //val ds11: DataStream[String] = senv.fromCollection(Set("spark", "flink"))
    //ds11.print()

    //12.用Iterable创建DataStream(不支持)
    //val ds12: DataStream[String] = senv.fromCollection(Iterable("spark", "flink"))
    //ds12.print()

    //13.用ArraySeq创建DataStream
    val ds13: DataStream[String] = senv.fromCollection(mutable.ArraySeq("spark", "flink"))
    ds13.print()

    //14.用ArrayStack创建DataStream
    val ds14: DataStream[String] = senv.fromCollection(mutable.ArrayStack("spark", "flink"))
    ds14.print()

    //15.用Map创建DataStream(不支持)
    //val ds15: DataStream[(Int, String)] = senv.fromCollection(Map(1 -> "spark", 2 -> "flink"))
    //ds15.print()

    //16.用Range创建DataStream
    val ds16: DataStream[Int] = senv.fromCollection(Range(1, 9))
    ds16.print()

    //17.用fromElements创建DataStream
    val ds17: DataStream[Long] = senv.generateSequence(1, 9)
    ds17.print()

    senv.execute(this.getClass.getName)
  }
}
```

#### 基于文件的source

```scala
package org.duo.stream.datasource

import org.apache.flink.streaming.api.scala.{DataStream, StreamExecutionEnvironment}

object DataSource_CSV {

  def main(args: Array[String]): Unit = {

    // 1. 流处理环境
    val env = StreamExecutionEnvironment.getExecutionEnvironment
    // 2. 读取HDFS中的CSV文件
    val csvDataStream: DataStream[String] = env.readTextFile("hdfs://server01:8020/flink-datas/score.csv")
    // 3. 打印
    csvDataStream.print()
    // 4. 执行任务
    env.execute()
  }
}
```

#### 基于网络套接字的source

```scala
package org.duo.stream.datasource

import org.apache.flink.streaming.api.scala.{DataStream, StreamExecutionEnvironment}
import org.apache.flink.api.scala._

object DataSource_Socket {

  def main(args: Array[String]): Unit = {

    //1. 创建流式环境
    val env = StreamExecutionEnvironment.getExecutionEnvironment

    // 在Linux中，使用`nc -lk 端口号`监听端口，并发送单词
    // 2. 构建socket数据源
    val socketDataStream: DataStream[String] = env.socketTextStream("server01", 9999)

    // 3. 数据转换,空格切分
    val mapDataStream: DataStream[String] = socketDataStream.flatMap(
      line => line.split(" ")
    )

    // 4. 打印
    mapDataStream.print()

    // 5. 执行任务
    env.execute()
  }
}
```

#### 自定义source

通过去实现SourceFunction或者它的子类RichSourceFunction类来自定义实现一些自定义的source，Kafka创建source数据源类FlinkKafkaConsumer011也是采用类似的方式。

```scala
package org.duo.stream.datasource

import java.util.UUID
import java.util.concurrent.TimeUnit

import org.apache.flink.streaming.api.functions.source.{RichSourceFunction, SourceFunction}
import org.apache.flink.streaming.api.scala.{DataStream, StreamExecutionEnvironment}
import org.apache.flink.api.scala._

import scala.util.Random

object DataSource_Custom {

  def main(args: Array[String]): Unit = {

    //    1. 创建订单样例类
    // 订单ID`、`用户ID`、`订单金额`、`时间戳`)
    case class Order(id: String, userId: String, money: Int, time: Long)
    //      2. 获取流处理环境
    val env = StreamExecutionEnvironment.getExecutionEnvironment
    //      3. 创建自定义数据源
    val customDataStream: DataStream[Order] = env.addSource(new RichSourceFunction[Order] {

      override def run(ctx: SourceFunction.SourceContext[Order]): Unit = {
        //      - 循环1000次

        for (i <- 0 until 1000) {
          //    - 随机构建订单信息
          // - 随机生成订单ID（UUID）
          val id = UUID.randomUUID().toString
          // - 随机生成用户ID（0-2）
          val userId = Random.nextInt(3).toString
          // - 随机生成订单金额（0-100）
          val money = Random.nextInt(101)
          // - 时间戳为当前系统时间
          val time = System.currentTimeMillis()
          //    - 上下文收集数据
          ctx.collect(Order(id, userId, money, time))
          //    - 每隔一秒执行一次循环
          TimeUnit.SECONDS.sleep(1)
        }
      }

      override def cancel(): Unit = {
      }
    })

    //    4. 打印数据
    customDataStream.print()

    //      5. 执行任务
    env.execute()
  }
}
```

#### 使用Kafka作为数据源

```scala
package org.duo.stream.datasource

import java.util.Properties
import org.apache.flink.api.common.serialization.SimpleStringSchema
import org.apache.flink.streaming.api.scala.{DataStream, StreamExecutionEnvironment}
import org.apache.flink.streaming.connectors.kafka.{FlinkKafkaConsumer010, FlinkKafkaConsumer011}
import org.apache.kafka.clients.CommonClientConfigs
import org.apache.flink.api.scala._

object DataSource_Kafka {

  def main(args: Array[String]): Unit = {

    // 1. 创建流式环境
    val env = StreamExecutionEnvironment.getExecutionEnvironment

    //2 .指定kafak相关信息
    val kafkaCluster = "server01:9092,server02:9092,server03:9092"
    val kafkaTopic = "test-topic-1"

    // 3. 创建Kafka数据流
    val props = new Properties()
    props.setProperty(CommonClientConfigs.BOOTSTRAP_SERVERS_CONFIG, kafkaCluster)
    val flinkKafkaConsumer = new FlinkKafkaConsumer011[String](kafkaTopic, new SimpleStringSchema(), props)

    //4 .设置数据源
    val kafkaDataStream: DataStream[String] = env.addSource(flinkKafkaConsumer)

    // 5. 打印数据
    kafkaDataStream.print()

    // 6.执行任务
    env.execute()
  }

}
```

#### 使用MySQL作为数据源

```scala
package org.duo.stream.datasource

import java.sql.{Connection, DriverManager, PreparedStatement, ResultSet}

import org.apache.flink.streaming.api.functions.source.{RichSourceFunction, SourceFunction}
import org.apache.flink.streaming.api.scala.{DataStream, StreamExecutionEnvironment}
import org.apache.flink.api.scala._

object DataSource_MySql {

  //  1. 自定义Source,继承自RichSourceFunction
  class MySql_Source extends RichSourceFunction[(Long, String, String, String)] {

    //  2. 实现run方法
    override def run(ctx: SourceFunction.SourceContext[(Long, String, String, String)]): Unit = {
      //    1. 加载驱动
      Class.forName("com.mysql.jdbc.Driver")
      //    2. 创建连接
      val connection: Connection = DriverManager.getConnection("jdbc:mysql://server01:3306/kgwebdb?useSSL=false", "root", "123456")
      //    3. 创建PreparedStatement
      val sql = "select id,username,password,nickname from sys_user"
      val ps: PreparedStatement = connection.prepareStatement(sql)
      //    4. 执行查询
      val resultSet: ResultSet = ps.executeQuery()
      //    5. 遍历查询结果,收集数据
      while (resultSet.next()) {
        val id = resultSet.getLong("id")
        val username = resultSet.getString("username")
        val password = resultSet.getString("password")
        val name = resultSet.getString("nickname")

        // 收集数据
        ctx.collect((id, username, password, name))
      }

    }

    override def cancel(): Unit = {

    }
  }

  def main(args: Array[String]): Unit = {
    // 1. env
    val env = StreamExecutionEnvironment.getExecutionEnvironment

    // 2 使用自定义Source
    val mySqlDataStream: DataStream[(Long, String, String, String)] = env.addSource(new MySql_Source)

    // 3. 打印结果
    mySqlDataStream.print()

    // 4. 执行任务
    env.execute()
  }
}
```



















[1]:data:image/png;base64,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
[2]:data:image/png;base64,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
[3]:data:image/png;base64,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wGJ/CzlffoU/5ExVOZaIiIiIiPQampYlUbEi89uYNTQN9798FA1xH+Ozb34Tl16hkdXSeV9faEL/839BgtOJ4tvHYsqQq6O9SyIiIiIiIhGl4I5ETUZiIt64dzqef+dtlJypw5E37dHeJYlBt3zrSjw0bBgeSEvHN/v1i/buiIiIiIiIRJyCOxJ1/zjsOvzjsOuivRsiIiIiIiIiMUk9d0REREREREREYpiCOyIiIiIiIiIiMUzBHRERERERERGRGKbgjoiIiIiIiIhIDFNwR0REREREREQkhim4IyIiIiIiIiISwxTcERERERERERGJYQruiIiIiIiIiIjEMAV3RERERERERERimII7IiIiIiIiIiIxTMEdEREREREREZEYpuCOiIiIiIiIiEgMU3BHRERERERERCSGKbgjIiIiIiIiIhLD+kZ7B0RERHqrvt9IwKGm96O9GyLdSt9vJER7F0RERGKOgjsiIiJRMv6/90Z7F0RERESkB1BZloiIiIiIiIhIDFNwR0REREREREQkhim4IyIiIiIiIiISwxTcERERERERERGJYQruiIiIiIiIiIjEMAV3RERERERERERimII7IiIiIiIiIiIxTMEdEREREREREZEYpuCOiIiIiIiIiEgMU3BHRERERERERCSGKbgjIiIiIiIiIhLDFNwREREREREREYlhCu6IiIiIiIiIiMSwvtHeARERERGRgDmbAXs9f89IASwJ0d0fERGRbkDBHRERERGJHf3igOXFQIPD+/ZRGfxpSQAWzgDSkiK/byIiIlGisiwRERERiR3xccCyOW1vr7Tz3wcOINka+f0SERGJIgV3RERERCS2ZI8Ackb7vm/ZXAaAREREehEFd0REREQk9iye1bbfjjURSB4Unf0RERGJIgV3RERERCT2WBOBR2Z4/33/eGBWIVBUHr39EhERiQIFd0REREQkNs2cAGSm8/eFM4CHc4HtPwFOvgU88DhQXRvd/RMREYkQBXdEREREJHYtm8tJWdOz+XeyFdi8CJiXCzy6CVhdwvHpIiIiPZiCOyIiIq1V1wIlB6K9FyK9k60GKLUF/viMFGDTora3T8wCXljFvjz3Le/cNkVERGKMgjsiIiIGVwuwYS9X+62J0d4bkd7JmgjsOQYUrAPqGgN7jr/pWJYE9uXZvKjz2xQREYkhCu6IiIgAzBaYthQ4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
[4]:data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAA1YAAAGCCAIAAABsDsUsAAAgAElEQVR4AeydB3xUVdrGz1ANitRIiSAEbLCsEAOh6CoiggrEgAICfrZNsO2quKIIaGiLZcVeSOx0WIgBVJQFdZUWYgBZY4NQQ4uAiBqq8z3nnntn7rRkJpnJzJ157k8z9557ynv+54Q8855ms9vtghcJkAAJkAAJkAAJkEAsEagWS5VlXUmABEiABEiABEiABCQBSkD2AxIgARIgARIgARKIOQKUgDHX5KwwCZAACZAACZAACVACsg+QAAmQAAmQAAmQQMwRoASMuSZnhUmABEiABEiABEiAEpB9gARIgARIgARIgARijgAlYMw1OStMAiRAAiRAAiRAApSA7AMkQAIkQAIkQAIkEHMEKAFjrslZYRIgARIgARIgARKgBGQfIAESIAESIAESIIGYI0AJGHNNzgqTAAmQAAmQAAmQQA0iIAESCBGBTz/9dN26dcePHw9R/lGTbe3atVNSUnr27Bk1NWJFSIAESCDyCdjsdnvkW0kLScBaBI4ePdr3ppvWb9168oILRA1+0Sqv9U6dqvnDD53btFm2YEHdunXLi833JEACJEACQSBACRgEiMyCBNwI9Ojbd3X16mJgmls4H8sisCin++nTq5YtKysO35EACZAACQSJAOcCBgkksyEBgwDGf+H/o/4zePj9OTAN3EDP7wSMSAIkQAIkUHEClIAVZ8eUJOCVAOb/yfFfXoETADfQCzwdU5AACZAACQRMgBIwYGRMQAJlE5DrPzj/r2xGvt7WqMHVM77YMJwESIAEgkuAEjC4PJkbCZAACZAACZAACViAACWgBRqJJpIACZAACZAACZBAcAlQAgaXJ3MjARIgARIgARIgAQsQoAS0QCPRRBIgARIgARIgARIILgFKwODyZG4kQAIkQAIkQAIkYAEClIAWaCSaSAIkQAIkQAIkQALBJUAJGFyezI0ESIAESIAESIAELECAEtACjUQTSYAESIAESIAESCC4BCgBg8uTuZEACZAACZAACZCABQhQAlqgkWgiCZAACZAACZAACQSXACVgcHkyNxIgARIgARIgARKwAAFKQAs0Ek0kARIgARIgARIggeASoAQMLk/mRgIkQAIkQAIkQAIWIEAJaIFGookkQAIkQAIkQAIkEFwClIDB5cncSIAESIAESIAESMACBCgBLdBINJEESIAESIAESIAEgkuAEjC4PJkbCZAACZAACZAACViAACWgBRqJJpIACZAACZAACZBAcAlQAgaXJ3MjARIgARIgARIgAQsQoAS0QCPRRBIgARIgARIgARIILgFKwODyZG4kQAIkQAIkQAIkYAEClIAWaCSaSAIkQAIkQAIkQALBJUAJGFyezI0ESIAESIAESIAELECAEtACjUQTSYAESIAESIAESCC4BCgBg8uTuZEACZAACZAACZCABQhQAlqgkWgiCZAACZAACZAACQSXACVgcHkyNxIgARIgARIgARKwAAFKQAs0Ek0kARIgARIgARIggeASoAQMLk/mRgIkQAIkQAIkQAIWIEAJaIFGookkQAIkQAIkQAIkEFwClIDB5cncSIAESIAESIAESMACBCgBLdBINJEESIAESIAESIAEgkuAEjC4PJkbCZAACZAACZAACViAACWgBRqJJpIACZAACZAACZBAcAlQAgaXJ3MjARIgARIgARIgAQsQoAS0QCPRRBIgARIgARIgARIILgFKwODyZG4kQAIkQAIkQAIkYAEClIAWaCSaSAIkQAIkQAIkQALBJUAJGFyezI0ESIAESIAESIAELECAEtACjUQTSYAESIAESIAESCC4BCgBg8uTuZEACZAACZAACZCABQhQAlqgkWgiCZAACZAACZAACQSXQI3gZsfcSIAEvBM4lC9G54mn7xENtff2o2L6IyJpquhST6j7hFGif1tn2rxskZVnPHbSE6qY+X9y5oMYiCkGy3zMl9fk5gi8JwESIAESiG0C9ALGdvuz9lVGoGGySP1RrNqiF3j4e5F/ra7bcC86iNxPxCFXa1IfEW9ky/8yaor5a53vkm0uj84Xrne+krvG4hMJkAAJkEBsEqAEjM12Z63DQaDHcJH7rV7wqlki4y/6/ZYNImmYFIhbjng3q20nkV/ifNV7mBDviTwfkZ3xjDu35EYwP0mABEiABGKZACVgLLc+6161BKQjcKWUbhgUzr1KdwHat4usk6JtPdGuj8j6r3eDoBdTLza9OlsMvkNkzXL3GppiuNyak29dJv6arv33qp4cxriFIPGSB90DkfB1wxMJm/9qJEfMvDwZWb3FALSe2xLdBkdWjuQuxvGBBEiABEggPAQ4FzA83FlqjBKAzlsOR+BmkTFYJ1D0nUhOkhMEG1wk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8rZjx47vv/8ei3/POuusCDCKJpAACZAACZBAFBLgXMAobFQrVmnRokWY9tekSZP58+d3797dilWgzSRAAiRAAiRgIQKUgBZqrOg0ddWqVRB/+/fvx5qPgQMHRmclWSsSIAESIAESiDACHAiOsAaJJXO2bNmCs92GDBnSv3//r776ivovliF58VgAACAASURBVBqfdSUBEiABEggzAUrAMDdAbBaP89NwttvFF1983nnnbd++/e67745NDqw1CZAACZAACYSLACVguMjHbrnPPfcc1nwcOXIEC36x+LcGj9CN3b7AmpMACZAACYSNACVg2NDHYMFz5szB2W5ffPHFxx9/jMW/5557bgxCYJX9IrBr9i04yW7o7B1CFGbizpaR51c6RooUAqoFp6yJFHtoBwmQgAcBSkAPJAwIAYGVK1fibLeXX375mWeeweLfSy+9NASFMMsIIbDvgzQp2uRVbfKS8o0y4muCr/zolYhROnOAzTZw2p5Aslg9UVWlMhpUK1fLxiyJlEhCcIAVl5rYL7A+q6lUtapXoKWrTCtC0qc5fEECJBAeApSA4eEeO6V+8803w4YNy8jIuOWWW7D497rrroudusdoTXetnPu+UXX7+AHl6htH/Hn/XhiQODMK8f9z2xY/FKlrdoWfPKECspcHw6E1foXDgtLP5850Lcu/p33bNvkX0XustS/bbO0neH/nf2j5JFsMm2G328d28z9PxiQBEqhiApSAVQw8hor76aefHnzwwZSUlA4dOqjFvzFU+diu6mVr8ccf15qXwKFD6/PKpKGU0JSJk4XIeejtcmWWVDD65eIJM1yJeKd52pyON4TM2y+EjKBJn5yHEvQ4Qphyk3E8r7UroZaGTJ42SIhxzzjch8qLdnfeOt0YwxJf4SrftJsGCvv4JfqI9r6VC5eIwTeOcJZpqoLJVWmqyMDn9+S9bGvWDwob2lqvlxAOb6JjuFx5LqfMke5YVwlemNntbyhxyKztqons9oIBdmWCqSBFDMHKpIFvrJt7i4Iu2XqS9Ga5aSDYHYvJJKc/Ug/US5w9558osDKeVydX3pEACfgiYPw7wE8SCCYBdbbH/fffX1JSEsx8mVelCeCfgkrnUU4G3+iOMyFskxaXE3fv0hukItlhXyv1oiP+zllSG2lKReWWvg4ZaZrS/G+ZHt81fPJqRHXaIOOnTSveLwty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k9wuSD2vm5nbWskdLh0zGcyppHt7pcumfRiTxWQG/HPsnL2gTUNMuticzuf9YfgFO+mzONYv9xnN/EL5NYMyZ8OcLe+tQoDd2yotVWV2sktUGepACgqlFzAQOxjXjQDO9oDn75tvvhk3btzw4cPd3vLR8gSwR3rqvS5TI+S8CG1RiLluNoyvaavXPScbyDV3i81xnfdwbzfRjmdUQVjAIVe1a7Ma9NkLGLEy7S/oTGncKQ0np0Boy0EwZUIlzIBrP9+IVOYnBgGCNWejzHL4MhIJsHtHYquE1SZ2ibDi91W4zW53PcNXi4jjZTMLCsQAtwFXX5kwvEIEFi/JTEp64okn3BLv2bMH4m/OnDkQf1j84faWj5FPgL8+FW8jH78UFc+QKYNNgN274kSjtHuzS1i3S3AguOJtF/SUp0+fnjhxYuvWrc8880ys+aD+CzphZkgCJEACJEACJKAIUAJGSk+YPn06xN/WrVv/97//YfHv2WefHSmW0Q4SIAESIAESIIGoI8C5gOFvUpztgZHfRo0azZ49+7LLLgu/QbSABEiABEiABEgg2glQAoazhXfv3t2/f3/8xDkfN954YzhNYdkkQAIkQAIkQAKxRIADweFr7V9/xT4vffv2xeJf6r/wNQNLJgESIAESIIFYJEAJGL5WP+usBx988N577w2fBSyZBEiABEiABEggRglQAoaz4atXrx7O4lk2CZAACZAACZBArBKgBIzVlme9SYAESIAESIAEYpgAJWAMNz6rTgIkQAIkQAIkEKsEKAFjteVZbxIgARIgARIggRgmQAkYw43PqpMACZAACZAACcQqAUrAWG151psESIAESIAESCCGCQSyNfTiJWLx4hhmVbmqDxggBvSvXBZMTQIkQAIkQAIkQALBIRCIBESJAzqL/p2DU3JM5bJkfUxVl5UlARIgARIgARKIcAIcCI7wBqJ5JEACJEACJEACJBB8AtaUgIeLGqbnCWEToqRl+ofisN0XmFp5K8TSPb7eMpwESIAESIAESIAEYpNAgAPBYYFUtFFMXe0secxdooHziXckQAIkQAIkQAIkQAKBErCCBESdknuJkRc663bYcRu/M/s6xwNvSIAESIAESIAESIAE/CFgzYFgf2rGOCRAAiRAAiRAAiRAAj4IWF0COucCxi99R6zfmzA9S6S/KtI/cp8geLioEcI5L9BH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
[5]:data:image/png;base64,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
[6]:data:image/png;base64,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
[7]:data:image/png;base64,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QGHiDgAAAEBh4g4AAABAYeIOAAAAQGHiDgAAAEBh4g4AAABAYeIOAAAAQGHiDgAAAEBh4g4AAABAYeIOAAAAQGHiDgAAAEBh4g4AAABAYeIOAAAAQGHiDgAAAEBh4g4AAABAYeIOAAAAQGHiDgAAAEBh4g4AAABAYeIOAAAAQGHiDgAAAEBh4g4AAABAYeIOAAAAQGHiDgAAAEBh4g4AAABAYeIOAAAAQGHiDgAAAEBh4g4AAABAYeIOAAAAQGHiDgAAAEBh4g4AAABAYeIOAAAAQGHiDgAAAEBh4g4AAABAYeIOAAAAQGHiDgAAAEBh4g4AAABAYeIOAAAAQGHiDgAAAEBh4g4AAABAYeIOAAAAQGHiDgAAAEBh4g4AAABAYeIOAAAAQGHiDgAAAEBh4g4AAABAYeIOAAAAQGHiDgAAAEBh4g4AAABAYeIOAAAAQGHiDgAAAEBh4g4AAABAYeIOAAAAQGHiDgAAAEBh4g4AAABAYeIOAAAAQGHiDgAAAEBh4g4AAABAYeIOAAAAQGHiDgAAAEBh4g4AAABAYeIOAAAAQGHiDgAAAEBh4g4AAABAYeIOAAAAQGHiDgAAAEBh4g4AAABAYeIOAAAAQGHiDgAAAEBh4g4AAABAYUNDIz/PyuVr5z0PAAAAAE5paOTn+X/TnOnZAoH4gwAAAABJRU5ErkJggg==
[8]:data:image/png;base64,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
