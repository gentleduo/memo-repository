# 前言

apache所有软件的下载地址（包含各种历史版本）：

https://archive.apache.org/dist/

# Zookeeper

## 概念

Zookeeper是一个开源的分布式协调服务器，主要用来解决分布式集群中应用系统的一致性问题和数据管理问题

## 特点

Zookeeper本质上是一个分布式文件系统，适合存放小文件，也可以理解为一个数据库，Zookeeper中存储的其实是一个又一个Znode，Znode是Zookeeper中的节点。

1. Znode是有路径的，例如/data/host1，/data/host2，这个路径也可以理解为是Znode的Name
2. Znode也可以携带数据，例如说某个Znode的路径是/data/host1，其值是一个字符串"192.168.0.1"

正因为Znode的特性，所以Zookeeper可以对外提供出一个类似于文件系统的视图，可以通过操作文件系统的方式操作Zookeeper

1. 使用路径获取Znode
2. 获取Znode携带的数据
3. 修改Znode携带的数据
4. 删除Znode
5. 添加Znode

## 架构

### Leader

一个Zookeeper集群同一时间只会有一个实际工作的Leader，它会发起并维护与各Follwer及Observer间的心跳。所有的写操作必须要通过Leader完成再由Leader将写操作广播给其他服务器。

1. 集群的核心，集群内部各个服务器的调度者
2. 负责进行投票选举
3. 处理事务性（写操作）请求
4. 参与集群投票

### Follwer

一个Zookeeper集群可能同时存在多个Follwer，它会响应Leader的心跳。Follwer可直接处理并返回客户端的读请求，同时会将写请求转发给Leader处理，并且负责在Leader处理写请求时对请求进行投票。

1. 接受客户端请求，并向客户端返回结果
2. 处理客户端非事物性（读操作）请求
3. 转发事务性请求给Leader
4. 参与投票选举

### Observer

观察者角色与Follwer类似，但无投票权。

1. 接受客户端请求，并向客户端返回结果
2. 处理客户端非事物性（读操作）请求
3. 转发事务性请求给Leader
4. 不参与投票选举

## 应用场景

### 数据发布/订阅

发布/订阅一般有两种设计模式：推模式和拉模式，服务端主动将数据更新发送给所有订阅的客户端称为推模式；客户端主动请求获取最新数据称为拉模式。

Zookeeper采用了推拉结合的模式，客户端向服务器注册自己需要关注的节点，一旦该节点数据发生变更，那么服务器就会向相应的客户端推送Watcher事件通知，客户端接收到此通知后，主动到服务端获取最新的数据。

### 命名服务

命名服务是分布式系统中较为常见的一类场景，分布式系统中，被命名的实体通常可以是集群中的机器、提供的服务地址或远程服务对象等，通过命名服务，客户端可以根据指定名字来获取资源的实体，在分布式环境中，上层应用仅仅需要一个全局唯一名字。Zookeeper可以实现一套分布式全局唯一ID的分配机制。

### 分布式协调/通知

ZooKeeper中特有Watcher注册与异步通知机制，能够很好的实现分布式环境下不同系统之间的通知与协调，实现对数据变更的实时处理。使用方法通常是不同系统都对ZooKeeper上同一个Znode进行Watcher注册，监听Znode的变化（包括Znode本身内容及子节点的），若数据节点发生了变化，那么所有订阅该节点的客户端都能够接收到相应的Watcher通知，并作出相应处理。

在大多数分布式系统中，系统间的通信主要包括：心跳检测、工作进度汇报和系统调度。

### 分布式锁

分布式锁用于控制分布式系统之间同步访问共享资源的一种方式，可以保证不同系统访问一个或一组资源时的一致性，主要分为排它锁和共享锁。

#### 排它锁

又称写锁或者独占锁，若事物T1对数据对象O1加上了排它锁，那么在整个加锁期间，只允许事物T1对O1进行读取和更新操作，其他任何事物都不能再对这个数据对象进行任何类型的操作，直到T1释放了排它锁。

##### 定义锁

```
/exclusive_lock/lock
```

##### 实现方式

利用zookeeper的同级节点的唯一性特性，在需要获取排他锁时，所有的客户端试图通过调用create()接口，在/exclusive_lock节点下创建临时子节点/exclusive_lock/lock，最终只有一个客户端能创建成功，那么此客户端就获得了分布式锁。同时，所有没有获取到锁的客户端可以在/exclusive_lock节点上注册一个子节点变更的watcher监听事件，以便重新争取获得锁。

#### 共享锁

又称读锁。如果事务T1对数据对象O1加上了共享锁，那么当前事务只能对O1进行读取操作，其他事务也只能对这个数据对象加共享锁，直到该数据对象上的所有共享锁都释放。

##### 定义锁

```
/shared_lock/[hostname]-请求类型W/R-序号
```

##### 实现方式

1. 客户端调用create方法创建类似定义锁方式的临时顺序节点。
2. 客户端调用getChildren接口来获取所有已创建的子节点列表。
3. 判断是否获得锁，对于读请求如果所有比自己小的子节点都是读请求或者没有比自己序号小的子节点，表明已经成功获取共享锁，同时开始执行度逻辑。对于写请求，如果自己不是序号最小的子节点，那么就进入等待。
4. 如果没有获取到共享锁，读请求向比自己序号小的最后一个写请求节点注册watcher监听，写请求向比自己序号小的最后一个节点注册watcher监听。

#### 释放锁

在获取锁的客户端宕机或者正常完成业务逻辑都会导致临时节点的删除，此时，所有在/exclusive_lock节点上注册监听的客户端都会收到通知，可以重新发起分布式锁获取。

### 分布式队列

## 选举机制

### Serverid

编号越大在选择算法中的权重越大。比如有三台服务器，编号分别是1,2,3；则myid为3的服务器权重最大。

### Zxid

服务器中存放的最大数据ID。值越大说明数据越新，在选举算法中数据越新权重越大。

### Epoch

逻辑时钟。或者叫投票的次数，同一轮投票过程中的逻辑时钟值是相同的。每投完一次票这个数据就会增加，然后与接收到的其它服务器返回的投票信息中的数值相比，根据不同的值做出不同的判断。

### Server状态

选举状态：

1. LOOKING，竞选状态。
2. FOLLOWING，随从状态，同步leader状态，参与投票。
3. OBSERVING，观察状态,同步leader状态，不参与投票。
4. LEADING，领导者状态。

### 选举流程

#### 流程

##### 发出投票

节点启动时，都会将自己作为leader服务器来进行投票。

##### 接受各个服务器的投票

集群的每个服务器收到投票后，首先判断该投票的有效性，如检查是否是本轮投票，是否来自LOOKING状态的服务器。

##### 处理投票

针对每个投票，服务器都需要将别人的投票和自己的投票进行PK，PK规则如下：

优先检查ZXID：ZXID比较大的服务器优先作为leader。

如果ZXID相同：比较Serverid，Serverid较大的服务器作为leader服务器。

##### 统计投票

每次投票后，服务器都会统计投票信息，判断是否有机器得到的票数已经超过半数，如果有的话该机器当选leader。

##### 改变服务器状态

一旦确定了leader，每个服务器就会更新自己的状态，如果是follower，那么就变更为FOLLOWING，如果是leader，就变更为LEADING。



#### 集群刚启动

1. 服务器1启动：发起一次选举，服务器1投自己一票，此时服务器1票数一票，不够半数以上（3票），选举无法完成。投票结果：服务器1为1票。服务器1状态保持为LOOKING。

2. 服务器2启动：发起一次选举，服务器1和2分别投自己一票，此时服务器1发现服务器2的id比自己大，更改选票投给服务器2。投票结果：服务器1为0票，服务器2为2票。服务器1，2状态保持LOOKING

3. 服务器3启动：发起一次选举，服务器1、2、3先投自己一票，然后因为服务器3的id最大，两者更改选票投给为服务器3；投票结果：服务器1为0票，服务器2为0票，服务器3为3票。此时服务器3的票数已经超过半数（3票），服务器3当选Leader。服务器1，2更改状态为FOLLOWING，服务器3更改状态为LEADING。

4. 服务器4启动：发起一次选举，此时服务器1，2，3已经不是LOOKING 状态，不会更改选票信息。交换选票信息结果：服务器3为3票，服务器4为1票。此时服务器4服从多数，更改选票信息为服务器3。服务器4并更改状态为FOLLOWING。

5. 服务器5启动：与服务器4一样投票给3，此时服务器3一共5票，服务器5为0票。服务器5并更改状态为FOLLOWING。

6. 最终的结果：服务器3是 Leader，状态为 LEADING；其余服务器是 Follower，状态为 FOLLOWING。

#### 运行时期

在Zookeeper运行期间Leader和非Leader各司其职，当有非Leader服务器宕机或加入不会影响Leader，但是一旦Leader服务器挂了，那么整个Zookeeper集群将暂停对外服务，会触发新一轮的选举。初始状态下服务器3当选为Leader，假设现在服务器3故障宕机了，此时每个服务器上zxid可能都不一样，server1为99，server2为102，server4为100，server5为101。

集群 Leader 节点故障运行期选举与初始状态投票过程基本类似，大致可以分为以下几个步骤：

1. 状态变更。Leader 故障后，余下的非 Observer 服务器都会将自己的服务器状态变更为LOOKING，然后开始进入Leader选举过程。

2. 每个Server会发出投票。

3. 接收来自各个服务器的投票，如果其他服务器的数据比自己的新会改投票。

4. 处理和统计投票，每一轮投票结束后都会统计投票，超过半数即可当选。

5. 改变服务器的状态，宣布当选。

运行器Leader故障后选举流程

1. 第一次投票，每台机器都会将票投给自己。

2. 接着每台机器都会将自己的投票发给其他机器，如果发现其他机器的zxid比自己大，那么就需要改投票重新投一次。比如server1 收到了三张票，发现server2的xzid为102，pk一下发现自己输了，后面果断改投票选server2为老大。

##  安装

Zookeeper官网：https://zookeeper.apache.org/
下载地址：https://archive.apache.org/dist/zookeeper/

### 修改配置文件

```bash
[root@server01 conf]# cp zoo_sample.cfg zoo.cfg
[root@server01 conf]# vim zoo.cfg

# 修改如下内容
# 保留多少个快照
autopurge.snapRetainCount=3
# 日志多少个小时清理一次
autopurge.purgeInterval=1
dataDir=/opt/zookeeper/data
dataLogDir=/opt/zookeeper/log
# 集群中服务器地址
# 其中id为一个数字，表示zk进程的id，这个id也是dataDir目录下myid文件的内容
# host是该zk进程所在的IP地址，port1表示follower和leader交换消息所使用的端口，port2表示选举leader所使用的端口。
server.1=server01:2888:3888
server.2=server02:2888:3888
server.3=server03:2888:3888
```

### 添加myid配置

```bash
[root@server01 conf]# mkdir -p -m 755 /opt/zookeeper/data
[root@server01 conf]# mkdir -p -m 755 /opt/zookeeper/log
[root@server01 conf]# cd /opt/zookeeper/data
[root@server01 conf]# vim myid

# 添加如下内容
1
# 修改其他机器的配置文件：在server02上：修改myid为：2；在server03上：修改myid为：3
```

### 启动集群

```bash
# 启动
[root@server01 conf]# zkServer.sh start
# 查看集群状态，主从信息
[root@server01 conf]# zkServer.sh status
```

## 数据模型

Zookeeper的数据模型，在结构上和标准文件系统非常相似，拥有一个层次的命名空间，都是采用树形层次结构。Zookeeper树中的每个节点被称为一个Znode。和文件系统的目录树一样，Zookeeper树中的每个节点可以拥有子节点。

1. Znode 兼具文件和目录两种特点。既像文件一样维护着数据、元信息、ACL、时间戳等数据结构，又像目录一样可以作为路径标识的一部分，并可以具有子Znode。用户对Znode具有增、删、查、改等操作。

2. Znode存储数据大小的限制。Zookeeper虽然可以关联一些数据，但并没有被设计为常规的数据库或者大数据存储，相反的是，它用来管理调度数据，比如分布式应用中的配置文件信息、状态信息、汇集位置等等。这些数据的共同特征是它们都是很小的数据，通常以KB为大小单位，Zookeeper的服务器和客户端都被设计为严格检查并限制每个Znode的数据大小最多1M，常规使用中应该远小于此值。

3. Znode通过路径引用。如同Unix中的文件路径，路径必须是绝对的。因此他们必须由斜杠符开头。除此之外，他么必须是唯一的，也就是说每一个路径只有一个表示，因此这些路径不能改变。/zookeeper节点是默认节点，用来保存关键配置信息。

4. 每个Znode由3部分组成：

   stat：此为状态信息，描述为该Znode的版本，权限等信息；

   data：与该Znode关联的数据；

   children：该Znode下的子节点。

## 节点类型

### 临时节点

该节点的生命周期依赖于创建它们的会话。一旦会话结束，临时节点就将被自动删除，当然也可以手动删除。临时节点不允许拥有子节点。（ephemeral）

### 永久节点

该节点的生命周期不依赖于会话，并且只有在客户端显示执行删除操作的时候，他们才能被删除。（persistent）

### 序列化

Znode还有一个序列化特性，如果创建的时候指定的话，该Znode的名字后面会自动追加一个不断增加的序列号。序列号对于此节点的父节点来说是唯一的，这样便会记录每个子节点创建的先后顺序。它的格式为"%10d"(10位数字，没有数值的数位用0补充，例如“0000000001”)。

### 目录节点形式

1. PERSISTENT

   永久节点

2. PERSISTENT_SEQUENTIAL

   永久节点、序列化

3. EPHEMERAL

   临时节点

4. EPHEMERAL_SEQUENTIAL

   临时节点、序列化

## 命令操作

1. 创建永久节点：create /hello world
2. 创建临时节点：create -e /abc 123
3. 创建永久序列化节点：create -s /zhangsan boy
4. 创建临时序列化节点：create -e -s /lisi boy
5. 修改节点数据：set /hello zookeeper
6. 获取节点信息：get /hello
   - dataVersion：数据版本号，每次对节点进行set操作，dataVersion都会增加1。
   - cversion：子节点的版本号。当Znode的子节点有变化时，cversion的值就会增加1。
   - aclVersion：ACL的版本号。
   - cZxid：Znode创建时的事物id。
   - mZxid：Znode被修改的事物id，即每次对Znode的修改都会更新mZxid。对于Zookeeper来说，每次的变化都会产生一个唯一的事物id，zxid(Zookeeper Transaction Id)。通过zxid可以确定更新操作的先后顺序。例如，如果zxid1小于zxid2说明zxid1操作先于zxid2发生，zxid对于整个zk都是唯一的。
   - ctime：节点创建时的时间戳
   - mtime：节点最新一次更新发生时的时间戳
   - ephemeralOwner：如果该节点为临时节点，ephemeralOwner的值表示与该节点绑定的session id，如果不是ephemeralOwner的值为0。
7. 删除节点(如果要删除的节点有子Znode则无法删除)：delete /hello
8. 删除节点(如果有子Znode则递归删除)：rmr /abc
9. 列出历史记录：histroy

## Watch机制

类似于数据库的触发器，对某个Znode设置Watcher，当Znode发生变化的时候，WatchManager会调用对应的Watcher。

当Znode发生删除、修改、创建、子节点修改的时候，对应的Watcher会得到通知。

Watcher的特点：

1. 一个Watcher只会被触发一次，如果需要继续监听，则需要再次添加Watcher
2. Watcher得到的事件是被封装过的，包括三个内容：keeperState，eventType，path

### keeperState

| 属性          | 说明                     |
| ------------- | ------------------------ |
| SyncConnected | 客户端与服务器正常连接时 |
| Disconnected  | 客户端与服务器断开连接时 |
| Expired       | 会话session失效时        |
| AuthFailed    | 身份认证失败时           |

### eventType

| 枚举类型                       | 说明                             |
| ------------------------------ | -------------------------------- |
| None                           | 无                               |
| NodeCreated	Watcher         | 监听的数据节点被创建时           |
| NodeDeleted	Watcher         | 监听的数据节点被删除时           |
| NodeDataChanged	Watcher     | 监听的数据节点内容发生变更时     |
| NodeChildrenChanged	Watcher | 监听的数据节点的子节点发生变化时 |

## JavaAPI操作

可以使用一套Zookeeper客户端框架Curator，可以解决很多Zookeeper客户端非常底层的细节开发问题。

Curator包含几个包：

1. curator-framework：对zookeeper的底层api的一些封装；
2. curator-recipes：封装了一些高级特性，如：Cache事件监听、选举、分布式锁、分布式计数器等。

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

#### 设计构思

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

1. 分区：对输出的Key-Value对进行分区(map阶段)
2. 排序：对不同分区的数据按照相同的Key排序(map阶段)
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

