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
