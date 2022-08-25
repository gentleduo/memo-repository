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

