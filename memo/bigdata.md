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

对于每一个表分文件， Hive可以进一步组织成桶，也就是说桶是更为细粒 度的数据范围划分。Hive也是 针对某一列进行桶的组织

##### sorted by

指定排序字段和排序规则

##### row format

指定表文件字段分隔符

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

#### 开启Hive的分桶功能及

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

当distribute by和sort by字段相同时，可以使用cluster by方式。cluster by除了具有distribute by的功能外还兼具sort by的功能。但是排序只能是倒序排序，不 能指定排序规则为ASC或者DESC。

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
