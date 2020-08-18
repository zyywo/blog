<!--markdown-->[RFC2328-OSPFv2](https://tools.ietf.org/html/rfc2328)

[Cisco-有编号和无编号的接口](https://www.cisco.com/c/en/us/support/docs/ip/hot-standby-router-protocol-hsrp/13786-20.html)

## OSPF的基本原理与实现

1. 宣告OSPF的路由器从所有启动OSPF协议的接口上发出Hello数据包。如果两台路由器共享一条公共数据链路，并且能够相互成功协商它们各自Hello数据包中所指定的某些参数，那么它们就成为了邻居（Neighbor）。

2. 邻接关系（Adjacency），它是在一些邻居路由器之间构成的。

3. 每一台路由器都会在所有形成邻接关系的邻居之间发送链路状态通告（Link State Advertisement， LSA）。LSA描述了路由器所有的链路、接口、路由器的邻居以及链路状态信息。

4. 每一台收到从邻居路由器发出的LSA的路由器都会把这些LSA记录在它的链路状态数据库中，并且发送一份LSA的拷贝给该路由器的其他所有邻居。

5. 通过LSA泛洪扩散到整个区域，所有的路由器都会形成同样的链路状态数据库。

7. 每一台路由器都会使用SPF算法构建出自己的路由表。

当链路状态数据库已经同步，邻居之间交换Hello数据包保活，并且每隔30分钟重传一次LSA。如果这期间有新的接口宣告到OSPF域中，不会立即产生LSA,需要等到和其他LSA一起发送。

### 邻居和邻接关系

在发送任何LSA之前，OSPF路由器都必须首先发现它们的邻居路由器并建立起邻接关系，并不是所有的邻居路由器都会成为邻接关系。

OSPF路由器利用Hello数据包通告它的路由器ID来开始建立邻居关系

Hello协议的目的：
    
- 它是发现邻居路由器的方法
- 在两台路由器成为邻居之前，需要通告这两台路由器必须相互认可的几个参数
- Hello数据包在邻居路由器之间担当keeplive的角色
- 它确保了邻居路由器之间的双向通信
- 它用来在一个广播网络或非广播多路访问（NBMA）网络上选取指定路由器（Designated Router, DR）和备份指定路由器（Backup Designated Router, BDR）

宣告OSPF的路由器周期性的从启动OSPF协议的每一个接口上发送Hello数据包，该周期称为Hello时间间隔（HelloInterval），它的配置是基于路由器的每一个接口的。

如果一台路由器在一个称为路由器无效时间间隔（RouterDeadInterval）的时间内还没有收到来自邻居的Hello数据包，那么它将宣告它的邻居无效。

每一个Hello数据包都包含以下信息：

- 始发路由器的路由器ID（RouterID）
- ***始发路由器的象区域ID（AreaID）**
- ***始发路由器接口的地址掩码**
- ***始发路由器接口的认证类型和认证信息**
- ***始发路由器接口的Hello间隔**
- ***始发路由器接口的路由器无效时间间隔**
- 路由器的优先级
- 指定路由器（DR）和备份指定路由器（BDR）
- ***标识可选性能的5个标记位**。E-bit选项代表OSPF的ExternalRoutingCapability，所有非stub区域的路由器都应置为1。
  
- 始发路由器的所有有效邻居的路由器ID

如果带*的参数不匹配，那么该数据包将被丢弃，而且无法成为邻居（不会创建发送方的邻居数据结构）。

如果路由器A收到了一个有效的Hello数据包，并在这个Hello数据包中发现了自己的路由器ID，那么路由器A就认为是双向通信（two-way communication）建立成功了。路由器A会立即向对方单播发送一个Hello包。

并不是所有的邻居路由器都会成为邻接关系，一个邻接关系的形成依赖于这两台邻居路由器所连接的网络类型。另外，网络类型也影响OSPF数据包传送的方式。

如果**数据库描述包中的MTU值**大于路由器接收端口的MTU值，这个数据库描述包会被丢弃（译注：RFC中用词是rejected）。邻接关系也无法建立（会卡在Excheange阶段）。

#### 网络类型

OSPF协议定义了5种网络类型：
1. 点到点网络（point-to-point）
2. 广播型网络（broadcast）
3. 非广播多路访问（NBMA）网络
4. 点到多点网络（point-to-multipoint）
5. 虚链路（virtual links）

- 点到点网络
  
  在点到点网络上的有效邻居总是可以形成邻接关系，OSPF数据包的目的地总是224.0.0.5，这个组播地址称为AllSPFRouters。
  
- 广播型网络
  
  会选举一台指定路由器和一台备份指定路由器。DR和BDR的OSPF数据包都发送到224.0.0.5。
  
  该类型网络上其他所有路由器的Hello数据包是以组播方式发送到AllSPFRouters（224.0.0.5），而链路状态更新数据包和链路状态确认数据包是发送到224.0.0.6，这个组播地址称为AllDRouters。
  
#### 指定路由器与备份指定路由器

指定路由器是路由器接口的特性，而不是整个路由器的特性。

网络中的每一台路由器都会与DR、BDR形成邻接关系，路由器的每个多点访问接口都有一个路由器的优先级（0～255），0优先级的路由器不能参与选举，也就是说不能成为DR或BDR。

#### 选举过程

选举过程概述：当一台OSPF路由器active并去发现它的邻居时，它将去检查有效的DR和BDR，如果DR和BDR存在的话，这台路由器会接受已经存在的DR和BDR；如果BDR不存在，将执行一个选举过程，选出具有最高优先级的路由器作为BDR。如果DR不存在，那么BDR路由器将被选为DR，然后再执行一个选举过程选取BDR路由器。

选举是非抢占的

一旦DR和BDR路由器选举成功，其他的路由器（DRothers）将只和DR及BDR形成邻接关系。所有的路由器将继续以组播方式发送Hello数据包到AllSPFRouters（224.0.0.5）,因此它们能够跟踪它们的邻居路由器。但是DRothers只以组播方式发送更新数据包到AllDRouters（224.0.0.6）。只有DR和BDR去侦听这个地址，而DR路由器将使用224.0.0.5泛洪扩散更新数据包到DRothers。

DR可以看作是一个虚拟的节点

假设路由器X准备计算一个网络中的DR和BDR，它会检查网络中与它建立双向通信的邻居列表，这个列表就是路由器X在这个网络中状态高于2-way的邻居集合。路由器X自身也包含在这个列表中。去除列表中那些没有资格选举DR的路由器（优先级为0的路由器）。对列表中剩余的路由器执行以下步骤：

(1) 记录当前网络中DR和BDR的值。

(2) 根据下面的步骤为网络计算新的BDR。

#### OSPF接口数据结构

- Type

  点到点（point-to-point，PTP）, 广播（broadcast）, 非广播多路访问（NBMA）, 点到多点（Point-to-MultiPoint）, 虚链路（virtual link）中的一种。
  
- State

  接口的功能状态，状态确定是否允许在接口上形成完全的邻接。状态按照接口状态机规则变化。

- IP interface address / mask

- Area ID

- HelloInterval

  这个接口发送两个Hello报文之间间隔的时间（秒）。接口会把这个时间放在Hello报文中通告出去。
  
- RouterDeadInterval

  不再收到邻居的Hello后，经过多少秒会宣布邻居Down。接口会把这个时间放在Hello报文中通告出去。
  
- InfTransDelay

  这个接口发送LSU报文需要的大概时间（秒），在发送之前，LSU中的每条LSA都要把老化时间（age）加上这个值。
  
- Router Priority

  选举DR/BDR用，越大优先级越高。接口会把这个值放在Hello报文中通告出去。
  
- Hello Timer

  用来周期性发送Hello报文的计时器。计时时间为HelloInterval秒。
  
- Wait Timer

  计时结束后接口会退出Waiting状态，加入DR/BDR选举过程中。计时时间为RouterDeadInterval秒。
  
- List of neighboring routers

  连接这个网络的所有路由器列表，按照Hello协议形成。
  
- DR / BDR

  指定路由器（Designated Router）和备份指定路由器（Backup Designated Router）。初始值都是0.0.0.0，意味着网络中没有DR/BDR。
  
- Interface output cost(S)

- RxmtInterval

  重传OSPF数据包所需要的时间
  
- Autype / Authentication key
  
  

#### OSPF接口状态机

参考：[RFC2328 - OSPFv2 - 9.1节](https://tools.ietf.org/html/rfc2328#page-68)

                                  +----+   UnloopInd   +--------+
                                  |Down|<--------------|Loopback|
                                  +----+               +--------+
                                     |
                                     |InterfaceUp
                          +-------+  |               +--------------+
                          |Waiting|<-+-------------->|Point-to-point|
                          +-------+                  +--------------+
                              |
                     WaitTimer|BackupSeen
                              |
                              |
                              |   NeighborChange
          +------+           +-+<---------------- +-------+
          |Backup|<----------|?|----------------->|DROther|
          +------+---------->+-+<-----+           +-------+
                    Neighbor  |       |
                    Change    |       |Neighbor
                              |       |Change
                              |     +--+
                              +---->|DR|
                                    +--+
                                    
##### 接口状态切换的事件

- **InterfaceUP**    更低级的协议检测到网络接口是可操作的，这会把接口移出Down状态。

- **WaitTimer**    Wait计时器超时，表明Waiting周期结束，在这之后才能选举DR或BDR。

- **BackupSeen**    路由器检测到网络中存在或不存在BDR。这通过两种方式完成：从邻居接收到的Hello数据包中声明邻居自己是BDR，或者从邻居接收到Hello数据包中声明邻居自己是DR，并且没有BDR。无论哪种情况，前提都是需要和邻居建立双向通信关系。这个事件标志着Waiting状态的结束。

- **NeighborChange**    指与接口建立双向关系的邻居集合发生了变化。DR和BDR需要重新选举。下面的邻居变化会引起NeighbroChange事件：

  - 与一个邻居建立了双向通信关系。也就是说，邻居转为2-way状态了。
  
  - 与一个邻居不再是双向通信关系了。也就是说，邻居转为Init或更低的状态了。
  
  - 一个双向关系的邻居声明邻居自己是DR或BDR。这可以通过邻居的Hello数据包发现。
  
  - 一个双向关系的邻居不再声明邻居自己是DR或BDR。这也是通过邻居的Hello数据包发现。
  
  - 双向关系邻居的路由器优先级发生了变化。这也是通过邻居的Hello数据包发现。
  
 - **LoopInd**    通过低层的协议发现接口是LoopBack的。
 
 - **UnloopInd**    通过低层的协议发现接口不再是LoopBack的。
 
 - **InterfaceDown**    通过低层的协议发现接口不可用，不管接口是什么状态，都会直接把接口转为Down状态。

#### OSPF邻居数据结构

- State

  邻居的功能状态，按照邻居状态机规则变化。
  
- Inactivity Timer
  
  这是一个时长为RouterDeadInterval的计时器，只要从邻居收到一个Hello数据包，该计时器就会被重置。如果计时器超时将宣告邻居失效。
  
- Master/Slave

  在ExStart状态下，邻居之间会协商master/slave。当两个邻居交换数据库时，msaster发送第一个数据库描述包，slave只允许回应master的数据库描述包。
  
- DD Sequence Number
  
  当前正在向邻居发送的数据库描述序列号。
  
- Last received Database Description packet
  
  最后收到的数据库描述包的 Initialize位，More位，Master/Slave位和可选项位，以及数据库描述包的序列号。用来确定下一个收到的数据库描述包是否重复。
  
- Neighbor ID

  邻居路由器的ID，该値是从邻居的Hello数据包中学习到的，而在virtual adjacency中是手工配置。
  
- Neighbor Priority
  
  邻居路由器的路由器优先级。包含在邻居的Hello数据包中，用来选举DR和BDR。
  
- Neighbor IP address

  邻居路由器的接口IP，当OSPF数据包以单播方式发送给邻居时，这个IP就是目的地址。
  
- Neighbor Options
  
  邻居路由器支持的一些可选的OSPF功能
  
- Neighbor's Designated Router

  邻居认为的DR路由器
  
- Neighbor's Backup Designated Router

  邻居认为的BDR路由器
  
- Link state retransmission list
  
  邻接关系建立后，已经泛洪出去但还没有得到确认的LSA的集合。当LSA还没有被确认或邻接关系还没破坏的时候，LSA将每经过RxmtIntrval的时间就重传一次。
  
- Database summary list

  链路状态数据库组成的LSA的列表，在邻居处于Exchange状态时，这个列表会发送给邻居。
  
- Link state request list

  为了同步链路状态数据库，需要从邻居接收哪些LSA的列表。这个列表在收到数据库描述数据包时被创建，然后路由器发送LSR给邻居。接收到期待的LSU时这个列表会减小，最终为空列表。
                                    
#### OSPF邻居状态机

参考：[RFC2328 - OSPFv2 - 10节](https://tools.ietf.org/html/rfc2328#page-80)
  
                                   +----+
                                   |Down|
                                   +----+
                                     |\\
                                     | \\Start
                                     |  \\      +-------+
                             Hello   |   +---->|Attempt|
                            Received |         +-------+
                                     |             |
                             +----+<-+             |HelloReceived
                             |Init|<---------------+
                             +----+<--------+
                                |           |
                                |2-Way      |1-Way
                                |Received   |Received
                                |           |
              +-------+         |        +-----+
              |ExStart|<--------+------->|2-Way|
              +-------+                  +-----+

              Figure 12: Neighbor state changes (Hello Protocol)

                                  +-------+
                                  |ExStart|
                                  +-------+
                                    |
                     NegotiationDone|
                                    +->+--------+
                                       |Exchange|
                                    +--+--------+
                                    |
                            Exchange|
                              Done  |
                    +----+          |      +-------+
                    |Full|<---------+----->|Loading|
                    +----+<-+              +-------+
                            |  LoadingDone     |
                            +------------------+

            Figure 13: Neighbor state changes (Database Exchange)
                
- Down

- Init

  接收到了邻居的Hello报文，但双向通信还没建立（如：路由器没有出现在邻居的Hello报文中）。
  
- 2-Way

  双向通信已建立。
  
- ExStart

  在这一状态下，邻居之间会建立主/从关系，并确定数据库描述数据包的序列号。
  
- Exchange

  在这一状态下下，路由器会向它的邻居发送数据库描述包，每个数据库描述包都有个序列号，邻居都要对这个包发送确认。同时，路由器也会发送链路状态请求数据包（LSR）给邻居，用来请求最新的LSA。
  
- Loading

  在这一状态下下，路由器会发送链路状态请求数据包（LSR）给邻居，用来请求那些在Exchange状态下发现但还没有收到的最新的LSA。
  
- Full

  在这一状态下下，邻接关系已完全建立。邻接关系会出现在Router-LSAs和Network-LSAs中。
  
#### 建立邻接关系

建立邻接关系的过程中，OSPF使用3种数据包类型：

- 数据库描述数据包（类型2）

- 链路状态请求数据包（类型3）

- 链路状态更新数据包（类型4）

在数据库描述数据包中有3个标记位用来管理邻接关系的建立过程：

- I位（Initial bit），当需要指明所发送的是第一个数据库描述数据包时，该位设置为1

- M位（More bit），当需要指明所发送还不是最后一个数据库描述数据包时，该位设置为1

- MS位（Master/Slave bit）， 当数据库描述数据包始发于一个主路由器时，该位设置为1

当两台路由器在ExStart状态开始协商主/从关系时，它们都将通过发送一个MS位置为1的空的数据库描述数据包来宣称自己是“主”路由器。这两个数据库描述数据包的数据库描述序号是由路由器根据当时使用到的序列号确定的。具有较低路由器ID的邻居路由器将成为“从”路由器，并且回复一个MS为0的数据库描述数据包——这个数据库描述数据包的序列号设置为“主”路由器的序列号，同时，这个数据库描述数据包也将是第一个携带LSA摘要信息的数据包。当主/从关系协商完成后，邻居状态也将转换到Exchange状态。

“主”路由器可以产生新的数据库描述序列号，“从”路由器只能使用“主”路由器产生的序列号，因此当“主”路由器描述完自己的链路状态，而“从”路由器还没有描述完时，“主”路由器会发送空的链路状态描述数据包，让“从”路由器有链路状态数据库序列号可以用。

#### 泛洪扩散

因为完全相同的链路状态数据库是正确操作OSPF的前提，因此LSA的泛洪扩散必须可靠的。

##### 可靠的泛洪扩散： 确认

 对于可靠的泛洪扩散来讲，每一个单独传送的LSA都必须被确认。确认分为隐式确认（Implicit Acknowledgment）或显式确认（Explicit Acknowledgment）。
 
 - **隐式确认**
   
   邻居可以通过向始发更新数据包的路由器回送包含那个LSA的拷贝信息的更新数据包，来作为对所接收的LSA的隐式确认。当邻居正打算向始发路由器发送更新数据包的时候，隐式确认比显式确认更有效。
   
 - **显式确认**
 
   指通过发送一个链路状态确认数据包来确认收到LSA。而且可以使用单个链路状态确认数据包来确认多个LSA。链路状态确认数据包只需要携带LSA的头部信息。
   
确认可以是有时延的或直接的。

##### 可靠的泛洪扩散：序列号、校验和、老化时间

OSPF使用线性的序列号空间和32位有符号的序列号，序列号的大小从InitialSequenceNumber（0x80000001）到MaxSequenceNumber（0x7fffffff）。当一台路由器始发一条LSA通告时，它将设置这个LSA的序列号为InitialSequenceNumber。每当这台路由器产生该LSA的一个新实例时，该路由器会将它的序列号增加1。如果LSA的序列号达到最大值且又需要创建这个LSA的一个新实例时，路由器就设置现有LSA的老化时间为MaxAge，并且重新泛洪扩散到所有的邻接节点来清除老的LSA。一旦所有的邻接邻居确认过这个老的LSA后，也就可以泛洪扩散这个含有InitialSequenceNumber序列号的LSA了。

校验和是一个16位整数。计算除了Age字段外的LSA其他部分，驻留在链路状态数据库中的LSA每隔5min将计算一次校验和。

老化时间（Age）是一个用来指明LSA的生存时间的16位无符号整数，以秒为单位。一台路由器在始发一个LSA时，它就把老化时间设置为0,而当泛洪扩散的LSA经过一台路由器时，LSA的老化时间就会增加InfTransDelay秒。当LSA驻留在路由器的数据库中时，LSA的老化时间同样也会增大。

### 区域（Area）

OSPF协议选用区域，可以减小链路状态数据库的大小，降低对路由器内存的消耗；链路状态数据库的减小也降低了路由器处理LSA时对CPU的消耗；LSA泛洪扩散也被限制在一个区域里面了。

区域是通过一个32位的区域ID来识别的。区域ID可以表示为一个十进制数字，也可以表示为一个点分十进制。
    
    0.0.0.16      = 16*256^0 = 16
    0.0.1.15      = 1*256^1 + 15*256^0 = 271
    192.168.30.29 = 192*256^3 + 168*256^2 + 30*256^1 + 29*256^0 = 3232243229
    
OSPF定义了下面3种与区域相关的通信量的类型：

- **域内通信量（Intra-Area Traffic）**
  
  单个区域内的路由器之间交换的数据包构成的通信量
  
- **域间通信量（Inter-Area Traffic）**

  不同区域的路由器之间交换的数据包构成的通信量
  
- **外部通信量（External Traffic）**

  OSPF域内的路由器和其他路由选择域的路由器之间交换的数据包构成的通信量
  
区域0是为骨干区域保留的区域ID号。骨干区域（Backbone Area）的任务是汇总每一个区域的网络拓扑到其他所有的区域。正是由于这个原因，所有的域间通信量都必须通过骨干区域，非骨干区域之间不能直接交换数据包。

#### 路由器的类型

- **内部路由器（Internal Router）**

  是指所有接口都属于同一个区域的路由器
  
- **区域边界路由器（Area Border Routers，ABR）**

  是指连接一个或者多个区域到骨干区域的路由器，并且这些路由器会作为域间通信量的路由网关。
  
- **骨干路由器（Backbone Router）**

  是指至少有一个接口是和骨干区域相连的路由器。
  
- **自主系统边界路由器（Autonomous System Boundary Router，ASBR）**

  可以认为是OSPF域外部通信量进入OSPF域的网关路由器，ASBR路由器是用来把从其他路由选择协议学习到的路由，通过路由选择重分配的方式注入到OSPF域的路由器。一台ASBR路由器可以是位于OSPF域内部的任何路由器。
  
#### 分段区域

分段区域（Partitioned area）是指由于链路失效而使一个区域的一个部分和其他部分隔离开来的情形

#### 虚链路

虚链路（Virtual link）是指一条通过一个非骨干区域连接到骨干区域的链路。

### 链路状态数据库

当LSA在路由器的链路状态数据库中，它们的老化时间是增大的，如果达到了最大生存时间，那么它们将从OSPF域中清除掉，为了防止正常的LSA达到最大生存周期而被清除掉，每隔30min（LSRefreshTime）始发这条LSA的路由器就将泛洪扩散这条LSA的一个新拷贝。这种机制就是链路状态重刷新（Link state refresh）。

#### LSA的类型

由于OSPF定义了多种路由器的类型，因而定义多种LSA通告的类型也是必要的。

- **路由器LSA（Router LSA），类型： 1**

  每个路由器都会在它属于的每个区域上产生路由器LSA，每个区域的路由器LSA都列出了所有连接到该区域的链路或接口。这个LSA只会在始发它们的区域内部进行泛洪扩散。
  
- **网络LSA（Network LSA），类型： 2**

  每一个多路访问网络中的指定路由器（DR）将会产生网络LSA通告，网络LSA只在产生这条网络LSA的区域内部进行泛洪扩散。网络LSA列出了所有与DR相连的路由器，包括DR自身。
  
- **网络汇总LSA（Network Summary LSA），类型： 3**

  是由ABR路由器始发的。ABR路由器将发送网络汇总LSA到一个区域，用来通告该区域外部的目的地址。
  
  当其他路由器从一台ABR路由器收到一条网络汇总LSA时，它并不运行SPF算法。相反地，它只是把通告的目的地连同计算所得的代价一起记录至路由表，这其实是距离矢量协议的行为，也是为什么要求所有的域间通信量都必须通过骨干区域的原因——这些区域其实构成了一个星形拓扑，避免出现环路。
  
- **ASBR汇总LSA（ASBR Summary LSA），类型： 4**

  也是由ABR路由器始发。ASBR汇总LSA除了所通告的目的地是一台ASBR路由器而不是一个网络外，其他的和网络汇总LSA都是一样的。

- **自主系统外部LSA（Autonomous System External LSA），类型： 5**

  或者称为**外部LSA（External LSA）**，是始发于ASBR路由器的，用来通告到达OSPF自主系统外部目的地或者OSPF自主系统外部的缺省路由的LSA。外部LSA通告将在整个自主系统中进行泛洪扩散。
  
- **NSSA外部LSA（NSSA External LSA），类型： 7**

  是指在非纯末梢区域（Not-So-Stubby Area）内始发于ASBR路由器的LSA通告。NSSA外部LSA通告几乎和自主系统外部LSA通告是相同的，只是NSSA外部LSA通告仅仅在始发这个NSSA外部LSA通告的非纯末梢区域内部进行扩散。
  
#### 末梢（Stub）区域

末梢区域是一个不允许AS外部LSA通告在其内部进行泛洪扩散的区域。如果在一个区域里没有学到类型5的LSA通告，那么类型4的LSA通告也是不必要的了。

位于末梢区域边界的ABR路由器将使用网络汇总LSA向这个区域通告一个简单的缺省路由（目的地址是0.0.0.0）。在区域内部路由器上，所有和域内或域间路由不能匹配的目的地址都将最终匹配这条缺省路由。由于缺省路由是由类型3的LSA通告传送的，因此它不会被通告到这个区域的外部去。

在末梢区域中有4个限制条件：

- 所有末梢区域内的路由器都会在它们的Hello数据包中把E-bit设置为0。

- 虚链路不能在一个末梢区域内进行配置，也不能穿过一个末梢区域。

- 末梢区域内的路由器不能是ASBR路由器。

- 一个末梢区域可以拥有多台ABR路由器，但是因为缺省路由的关系，区域内部路由器将不能确定哪一台路由器才是到达ASBR路由器的最优网关。

1. **完全末梢区域**

   由Cisco提出的，阻塞3,4,5类LSA，仅允许通告缺省路由的那一条3类LSA通过。
  
2. **非纯末梢区域**

    非纯末梢区域（Not-So-Stubby-Area，NSSA）允许外部路由通告到OSPF自主系统内部，而同时保留自主系统其余部分的末梢区域特征。为了做到这一点，在NSSA区域内的ASBR将始发类型7的LSA用来通告那些外部的目的网络。这些NSSA外部LSA将在整个NSSA区域中进行泛洪扩散，但是会在ABR路由器的地方被阻塞。
  
    NSSA外部LSA在它的头部有一个称为P-bit的标志。NSSA ASBR路由器可以设置或清除这个P-bit，如果一台NSSA ABR路由器收到一条P-bi设置为1的7类LSA数据包，那么它将把这条7类LSA转换成为类型5的LSA，并且将这条LSA泛洪到其他区域中去。如果这个P-bit被设置为0,那么将不会转换这条7类LSA，而且这条7类LSA携带的目的地址也不能通告到这个NSSA区域的外部。
  
| 区域类型             | 1和2  | 3     | 4     | 5      | 7      |
| :----------------- | :---: | :---: | :---: | :----: | :----: |
| 骨干区域（区域 0）    | 允许   | 允许   | 允许  | 允许    | 不允许  |
| 非骨干区域，非末梢区域 | 允许   | 允许   | 允许  | 允许    | 不允许  |
| 末梢区域            | 允许   | 允许   | 不允许 | 不允许  | 不允许  |
| 完全末梢区域         | 允许   | 不允许 | 不允许 | 不允许  | 不允许  |
| NSSA               | 允许   | 允许 | 允许    | 不允许  | 允许    |
   
### 路由表

#### 路径类型

每一条到达一个网络目的地的路由都可以被归类到4种路径类型中的一种：

- **区域内路径（Intra-area path）**    是指在路由器所在的区域内就可以到达目的地的路径。

- **区域间路径（Inter-area path）**    是指目的地在其他区域但是还在OSPF自主系统内的路径。

- **类型1的外部路径（Type 1 external path，E1）**    是指目的地在OSPF自主系统外部的路径。当一条外部路由重新分配到任何一个自主系统时，它都必须指定一个对就自主系统中的路由选择协议有意义的度量值。在OSPF协议里，ASBR路由器的责任是要给通告的外部路由指定一个代价值。对于类型1的外部路径来说，这个代价值是这条路由的外部代价加上到达ASBR路由器的路径代价之和。

- **类型2的外部路径（Type 2 external path，E2）**    也是指目的地在OSPF自主系统外部的路径，但是在计算外部路由的度量时不再计入到达ASBR路由器的内部路径代价。
   
### OSPF数据包格式

![OSPF数据包组成](https://raw.githubusercontent.com/zyywo/zyywo.pic/master/OSPF%E6%95%B0%E6%8D%AE%E5%8C%85%E7%9A%84%E5%B0%81%E8%A3%85%E7%BB%84%E6%88%90.png "OSPF数据包组成")


![OSPF包头](https://raw.githubusercontent.com/zyywo/zyywo.pic/master/OSPF%E5%8C%85%E5%A4%B4.png "OSPF包头")

OSPF数据包的IP包头TTL为1，OSPF数据包包头长度为24字节。

- **版本（Version）**
  
  是指OSPF的版本号。OSPF的版本号是2，对于IPv6的OSPF版本号是3。
  
- **类型**

  指出跟在头部后面的数据包类型
  
  |类型代码 | 描述        |
  | ----- | -----      |
  |1      | Hello      |
  |2      | 数据库描述   |
  |3      | 链路状态请求 |
  |4      | 链路状态更新 |
  |5      | 链路状态确认 |

- **数据包长度（Packet Length）**

  是指OSPF数据包的长度，包括数据包头部的长度，单位是字节。
  
- **路由器ID（Router ID）**

  是指始发路由器的ID
  
- **区域ID（Area ID）**

  始发数据包路由器所在的区域。如果数据包是在一个虚链路上发送的，那么区域ID就为0.0.0.0。
  
- **校验和（Checksum）**

  一个对整个数据包（包括包头）的标准IP校验和
  
- **认证类型（AuType）**

  是指正在使用的认证模式
  
  |认证类型代码（AuType）| 认证类型         |
  | ----------------- | --------------- |
  |0                  |空（没有认证）     |
  |1                  |简单（明文）口令认证|
  |2                  |加密校验和（MD5）  |
  
- **认证（Authentication）**

  如果Autype=0，将不检查这个认证字段，因此可以是任何内容
  
  如果Autype=1，这个字段将包含一个最长为64位的口令。
  
  如果Autype=2，这个认证字段将包括一个密钥ID、认证数据长度和一个加密序列号。这个认证消息摘要附加在OSPF数据包的尾部，不作为OSPF数据包本身的一部分（不影响OSPF数据包的长度，但影响IP数据包的长度）
  
- **密钥ID（Key ID）**

  标识认证算法和创建消息摘要使用的安全密钥
  
- **认证数据长度（Autentication Data Length）**

  指明附加在OSPF数据包尾部的消息摘要的长度，单位是字节。
  
- **加密序列号（Cryptographic Sequence Number**

  是一个不会减小的数字（达到最大值后再从0开始），用来防止重现攻击（Replay attacks）。
  
#### Hello数据包

  ![OSPF协议 Hello数据包](https://raw.githubusercontent.com/zyywo/zyywo.pic/master/OSPF%E7%9A%84Hello%E6%8A%A5%E6%96%87.png "OSPF协议 Hello数据包")
  
  Hello数据包是用来建立和维护邻接关系的。为了形成一种邻接关系，Hello数据包携带的参数必须和它的邻居保持一致。
  
- **网络掩码（Network Mask）**

  是指发送数据包的接口的网络掩码。如果接收数据包的接口的网络掩码和这个掩码不匹配，那么该数据包会被接收方丢弃。
  
- **Hello时间间隔（Hello Interval）**

  是指接口上发送Hello数据包的间隔时间，单位是秒。如果发送方和接收方的这个值不一致，那么它们无法建立邻居关系。一般是10s。
  
- **可选项（Option）**

  这个字段可以用来确保邻居之间的兼容性问题。一台路由器可以拒绝一台兼容性不匹配的路由器。
  
- **路由器优先级（Router Priority）**

  用来选举DR和BDR的。如果为0,那么始发路由器将没有资格选成DR和BDR。
  
- **路由器无效时间间隔（Router Dead Interval）**

  是指始发路由器在宣告邻居路由器无效之前，将要等待从邻居路由器发出的Hello数据包的时长，单位是秒。如果路由器收到Hello数据包中的该值和接收接口配置的RouterDeadInterval不匹配，那么这个Hello数据包将被丢弃。一般是Hello间隔的4倍。
  
- **指定路由器（DR）**

  是指网络上指定路由器的IP地址，注意，在DR选举过程中，这里只是始发路由器所认为的DR，不是最终选举出来的DR。只要Hello包能正常交互，所有路由器根据算法选出的DR/BDR必定一致。
  
- **备份指定路由器（BDR）**

  是指网络上备份指定路由器的IP地址。同样的，在BDR选举过程中，这里只是始发路由器所认为的BDR，不是最终选举出来的BDR。
  
- **邻居（Neighbor）**

  是一个递归字段，如果始发路由器在过去的一个RouterDeadInterval时间内，从网络上已经收到来自它的某些邻居的有效Hello数据包，那么将会在这个字段中列出所有这些邻居的RID。
  
#### 数据库描述数据包

数据库描述数据包用于正在建立的邻接关系。数据库描述数据包的一个主要目的是描述始发路由器数据库中的一些或者全部LSA信息，这个操作只需列出LSA的头部就可以。数据库描述数据包中包含了一个主/从控制关系的标志，用来管理这些数据包的交换。

![OSPF数据库描述数据包](https://raw.githubusercontent.com/zyywo/zyywo.pic/master/OSPF%E6%95%B0%E6%8D%AE%E5%BA%93%E6%8F%8F%E8%BF%B0%E6%8A%A5%E6%96%87.png "OSPF数据库描述数据包")

- **接口MTU（Interface MTU）**

  是指数据包在不分段的情况下，始发路由器接口可以发送的最大IP数据包大小，单位是字节。当数据包在虚链路上传送时，这个字段是为0x0000。
  
  如果数据库描述包中的MTU值大于路由器接收端口的MTU值，这个数据库描述包会被丢弃（译注：RFC中用词是rejected）。
  
- **可选项（Option）**

  路由器支持的一些可选能力。
  
- **I 位或称为初始位（Initial bit）**

  当发送的是一系列数据库描述数据包中的最初一个数据包时，该位设置为1。后续的数据库描述数据包把该位设置为0。
  
- **M 位或称为后继位（More bit）**

  当发送的数据包还不是一系列数据库描述数据包中的最后一个数据包时，将该位设置为1。最后的一个数据库描述数据包将把该位设置为0。
  
- **MS位， 或称为主/从位（Master/Slave bit）**

  在数据库同步过程中，该位设置为1，用来指明始发数据库描述数据包的路由器是一台“主”路由器，“从”路由器将设置该位为0。
  
- **数据库描述序列号（DD Sequence Number）**

  在数据库的同步过程中，用来确保路由器能够收到完整的数据库描述数据包序列。这个序列号将由“主”路由器在最初发送的数据库描述数据包中设置一些惟一的数值，而后续数据包的序列号将依次增加。
  
- **LSA头部（LSA Header）**

  列出了始发路由器的链路状态数据库中部分或全部LSA头部。LSA头部里包含有足够的信息可以惟一地标识一个LSA和一个LSA的具体实例。
  
#### 链路状态请求数据包

在数据库同步过程中如果收到了数据库描述数据包，路由器将会查看数据库描述数据包里有哪些LSA不在自己的数据库中，或者有哪些LSA比自己数据库中的LSA更新。然后将这些LSA记录在自己的链路状态请求列表中。接着，路由器会发送一个或多个链路状态请求数据包去向它的邻居请求发送在链路状态请求列表中的这些LSA副本。

一个数据包可以根据一个LSA头部的类型、ID和通告路由器进行惟一的标识，但是它不能请求这个LSA的具体实例（LSA的具体实例由LSA头部的序列号、校验和以及老化时间标识）。它的请求都是期望得到LSA的最新实例。

 ![OSPF链路状态请求数据包](https://raw.githubusercontent.com/zyywo/zyywo.pic/master/OSPF%E9%93%BE%E8%B7%AF%E7%8A%B6%E6%80%81%E8%AF%B7%E6%B1%82%E6%8A%A5%E6%96%87.png "OSPF链路状态请求数据包")
 
 - **链路状态类型（Link State Type）**
 
   指明LSA是一个路由器LSA、网络LSA还是其他类型的LSA。
   
 - **链路状态ID（Link State ID）**
 
   是LSA头部中的一个字段。
   
- **通告路由器（Advertising Router）**

  是指始发LSA通告的路由器的路由器ID。
  
#### 链路状态更新数据包

链路状态更新数据包是用于LSA的泛洪扩散和发送LSA去响应链路状态请求数据包的。OSPF数据包是不能离开发起它们的网络的（因为TTL=1）。因此，一个链路状态数据包可以携带一个或多个LSAA。接收LSA的邻居将负责在新的LS更新数据包中重新封装相关的LSA，从而进一步扩散到它自己的邻居。

![OSPF链路状态更新数据包](https://raw.githubusercontent.com/zyywo/zyywo.pic/master/OSPF%E9%93%BE%E8%B7%AF%E7%8A%B6%E6%80%81%E6%9B%B4%E6%96%B0%E6%95%B0%E6%8D%AE%E5%8C%85.png "OSPF链路状态更新数据包")

- **LSA数量（Number of LSA）**

  指出这个数据包中包含的LSA的数量。
  
- **链路状态通告（LSA）**

  是指携带的完整的LSA（不只是LSA头部）。每一个更新数据包都可以携带多个LSA。
  
#### 链路状态确认数据包

多个LSA可以通过单个链路状态确认数据包来确认。

![OSPF链路状态确认数据包](https://raw.githubusercontent.com/zyywo/zyywo.pic/master/OSPF%E9%93%BE%E8%B7%AF%E7%8A%B6%E6%80%81%E7%A1%AE%E8%AE%A4%E6%95%B0%E6%8D%AE%E5%8C%85.png "OSPF链路状态确认数据包")

### OSPF的LSA格式和产生方法

#### LSA的头部

在LSA头部中有3个字段可以惟一地识别每个LSA：类型、链路状态ID和通告路由器。另外，还有其他3个字段可以惟一地识别一个LSA的最新实例：老化时间、序列号和校验和。

![OSPF协议LSA头部](https://raw.githubusercontent.com/zyywo/zyywo.pic/master/OSPF%E7%9A%84LSA%E6%8A%A5%E6%96%87%E5%A4%B4%E9%83%A8.png "OSPF协议LSA头部")

- **老化时间（Age）**

  是指从发出LSA后所经历的时间，单位是秒。当泛洪扩散LSA时，在从每一台路由器接口转发出去时，LSA的老化时间都会增加一个InfTransDelay的秒数。当然，当LSA驻留在链路状态数据库内时，这个老化时间也会增大。
  
  LSA的老化时间不会超过MaxAge（1小时）。老化时间达到MaxAge的LSA不会被用在路由表计算中，并会重新被泛洪。
  
- **可选项（Option）**

  指出了在部分OSPF域中LSA能够支持的可选能力。
  
- **类型（Type）**

  就是LSA的类型。
  
- **链路状态ID（Link State ID）**

  用来指定LSA所描述的部分OSPF域（？）。这个字段根据不同的LSA类型取不同的值。
  
  |LS Type |Link State ID的值                                            |
  | ------ | ---------------------------------------------------------- |
  |1       |The originating router's Router ID                          |
  |2       |The IP interface address of the network's Designated Router |
  |3       |The destination network's IP address                        |
  |4       |The Router ID of the described AS boundary router           |
  |5       |The destination network's IP address                        |
  
  When an AS-external-LSA (LS Type = 5) is describing a default route, its Link State ID is set to DefaultDestination (0.0.0.0).
  
- **通告路由器（Advertising Router）**

  是指始发LSA的路由器的ID。
  
- **序列号（Sequence Number）**

  当LSA每次有新实例产生时，这个序列号就会增加。
  
- **校验和（Checksum）**

  这是一个除了Age字段之外，关于LSA的全部信息的校验和。
  
- **长度（Length）**

  是一个包含LSA头部在内的LSA长度，单位是字节。
  
#### 路由器LSA

  路由器LSA是由每一台路由器产生的。它列出了一台路由器的链路或接口，同时也列出了这些接口的状态和每一条链路的出站代价。路由器LSA只能在始发区域泛洪。
  
  路由器所属某个区域的所有接口必须在单个LSA内描述。
  
  ![OSPF的路由器LSA](https://raw.githubusercontent.com/zyywo/zyywo.pic/master/OSPF%E7%9A%84%E8%B7%AF%E7%94%B1%E5%99%A8LSA.png "OSPF的路由器LSA")
  
- **链路状态ID（Link State ID）**
  
  路由器LSA的链路状态ID是指始发路由器的路由器ID。
  
- **V，或虚链路端点位（Virtual Link Endpoint bit）**

  为1时，说明始发路由器是一条或多条具有完全邻接关系的虚链路的一个端点，这里被描述的区域是传送区域。
  
- **E，或外部位（External bit）**

  当始发路由器是一个ASBR路由器时，设置该位为1。
  
- **B，或边界位（Border bit）**

  当始发路由器是一个ABR路由器时，该位为1。
  
- **链路数量（Number of Links）**

  标明一个LSA所描述的路由器链路数量。

下面的字段用来描述路由器的（区域内的）每条链路。

每条链路都有**链路类型**字段，其他字段的值会根据**链路类型**的不同而变化。
  
- **链路类型（Link Type）**
  
  描述了链路所提供连接的一般类型。
  
  注意，路由器LSA把主机路由（host routes）归类为末梢网络来通告，且掩码是255.255.255.255。
  
  |链路类型 |连接                  |
  | -----  | ------------------  |
  |1       |点到点连接到另一台路由器 |
  |2       |连接到一个传送网络      |
  |3       |连接到一个末梢网络      |
  |4       |虚链路                |
  
  传送网络是指一个网络中有两个或更多的路由器
  
- **链路ID（Link ID）**

  这个字段的取值依赖于链路类型（Link Type）。用来标识链路连接的对象，This Link ID gives a name to the entity that is on the other end of the link.
  
  当连接的是一个也会产生LSA的对象时（如：路由器或传送网络），链路ID等于邻居LSA的链路状态ID（Link State ID）（不懂,P341）。
  
  |Link type |Description             |Link ID                                |
  | -------- | ---------------------- | ------------------------------------- |
  |1         |Point-to-point link     |Neighbor Router ID                     |
  |2         |Link to transit network |Interface address of Designated Router |
  |3         |Link to stub network    |IP network number                      |
  |4         |Virtual link            |Neighbor Router ID                     |
  
- **链路数据（Link Data）**

  这个字段的取值也是依赖于链路类型（Link Type）。
  
  当链路类型是**有编号的点到点链路**，**传送链路**，**虚链路**时，这个值取和网络相连的始发路由器接口的IP地址。有编号的点到点链路是指接口配有IP地址。
  
  当链路类型是**末梢网络**时，这个值取网络的IP地址掩码。
  
  当链路类型是**无编号的点到点链路**，这个值取接口的MIB-II ifIndex值。无编号的点到点链路是指接口没有地址，它是借用其他接口的地址。
  
- **TOS数量（Number of TOS）**

  RFC2328已经不再支持，为了兼容性才做保留，如果没有多余的TOS度量，这个值为0。
  
- **TOS**，**TOS度量（TOS Metric）**

  这两个字段是为了兼容以前的OSPF而保留。如果TOS数量（Number of TOS）为0，将没有这些字段。
  
##### 构造路由器LSA的过程

假设路由器想要在Area A内构造一个路由器LSA，它会检测自己的每个接口数据结构，为每个接口执行以下步骤：

1. 如果接口连接的网络不属于Area A，跳过这个接口，继续下一个接口。

2. 如果接口的状态是Down，跳过这个接口。

3. 如果接口的状态是Loopback，而且它没有连接到未编号点到点网络(也就是没有被未编号的点到点接口借用IP)，就在路由器LSA中添加一条类型3的链路，链路ID设置为接口的IP地址，链路数据设置为255.255.255.255，代价设置为0（思科设置为1）。

4. 其他的接口会根据OSPF接口的类型在LSA中添加不同的链路描述。

###### 描述点到点接口

对于点到点接口：

- 如果邻居路由器是全邻接（Full）状态，添加一条类型1（Poin-To-Point）链路。链路ID设置为邻居路由器的路由器ID。如果是有编号的点到点网络，链路数据设为接口IP地址。如果是未编号点到点网络，链路数据设为接口的MIB-II ifIndex值。代价设置为接口的出站代价。

- 另外，只要接口的状态是“Point-to-Point”（不管邻居路由器是什么状态），一条类型3（stub network）的链路应该被添加到LSA。有两种格式可以选择：

  - Option 1
    
    假设邻居路由器的IP地址已知（？），设置链路ID为邻居的IP地址，链路数据为255.255.255.255，代价为接口的出站代价。
    
  - Option 2
    
    如果一个子网分配到点到点链路，设置链路ID为子网的网络地址，链路数据为子网掩码，代价为接口出站代价。
    
###### 描述广播和NBMA接口

对于可操作的广播和NBMA接口，按照下面的步骤添加一条链路到网络LSA中：

如果接口的状态是Wating，添加一条类型3（stub network）的链路，链路ID设置为它连接的网络的网络号，链路数据设置为网络的网络地址掩码，代价为接口的出站代价。

否则，说明此时网络中已经选举出一台DR。

- 如果路由器和DR是全邻接或者它自身就是DR且至少有一台其他的路由器和它是全邻接，添加一条类型2（transit network）的链路，链路ID设置为网络中DR的接口IP地址（可能路由器自身就是DR），链路数据设置为路由器自己接口的IP地址，代价是接口的出站代价。

- 如果是其他的，按照接口状态是Wating处理。
  
###### 主机路由（Host Route）的定义

路由器LSA把主机路由（host routes）归类为末梢网络来通告，且掩码是255.255.255.255。

它们可以是路由器连接到点到点网络的接口、路由器的环回接口或直接连接到路由器的IP主机。
  
#### 网络LSA

网络LSA是指始发于指定路由器（DR）的。这些网络LSA将通告一个多路访问网络与这个网络相连的所有路由器（包括DR）。像路由器LSA一样，网络LSA也只能在始发这条网络LSA的区域内进行泛洪扩散。

![OSPF的网络LSA](https://raw.githubusercontent.com/zyywo/zyywo.pic/master/OSPF%E7%9A%84%E7%BD%91%E7%BB%9CLSA.png "OSPF的网络LSA")

- **链路状态ID（Link State ID**

  是指网络中DR路由器接口上的IP地址。
  
- **网络掩码（Network Mask）**

  指定这个网络上使用的地址或子网的掩码。
  
- **相连的路由器（Attached Router）**

  列出了多路访问网络上所有与DR形成完全邻接关系的路由器的路由器ID，以及DR路由器本身的路由器ID。
  
#### 汇总LSA－网络汇总LSA和ASBR汇总LSA

网络汇总LSA（类型3）和ASBR汇总LSA（类型4）具有同样的格式，惟一不同之处是它们的类型和链路状态ID。ABR路由器将产生这两种类型的汇总LSA。

网络汇总LSA通告的是一个区域外部的网络（包括缺省路由），而ASBR汇总LSA通告的是一个区域外部的ASBR路由器。这两种类型的LSA都只能泛洪扩散到单个区域。

![OSPF的汇总LSA](https://raw.githubusercontent.com/zyywo/zyywo.pic/master/OSPF%E7%9A%84%E6%B1%87%E6%80%BBLSA.png "OSPF的汇总LSA")

##### 网络汇总LSA

当一台ABR始发一条网络汇总LSA时，将包括从它本身到正在通告的这条LSA的目的地所耗费的代价。ABR路由器即使知道它有多条路由可以到达目的地，它也只会为这个目的地始发单条网络汇总LSA通告。因此，如果一台ABR路由器在它本身相连的区域内有多条路由可以到达目的地，那么它将只会始发单一的一条网络汇总LSA到骨干区域，而且这条网络汇总LSA是上述多条路由中代价最低的。

- **链路状态ID（Link State ID）**

  对于类型3的LSA来说，它是所通告的网络或子网的IP地址。
  
  对于类型4的LSA来说，它是所通告的ASBR路由器的路由器ID。
  
- **网络掩码（Network Mask）**

  在类型3的LSA中，是指所通告的网络的子网掩码或地址。
  
  在类型4的LSA中，这个字段没有实际意义，并被设置为0.0.0.0。
  
- **度量（Metric）**

  是指路由器到达目的地的路由代价。通常是到目的地路由的代价（直连的代价为0）、与目的地相连接口的代价之和。
  
##### 构造汇总LSA的过程

汇总LSA描述的目的地址为一个IP网络、一个AS边界路由器或一段IP地址。汇总LSA只能在单一区域内泛洪。被描述的目的地址位于区域外部但仍然属于自治系统。

汇总LSA由区域边界路由器（ABR)产生。通过下面的算法检查路由表数据结构，使精确的汇总路由通告给某个区域。注意，只有域内路由（intra-area routes）会被通告到骨干区域；域内路由和域间路由（inter-area routes）都会被通告到其他区域。

例如，为了决定哪些路由被通告到邻接的区域A，对每个路由条目进行以下处理（请记住，每个路由表条目都描述了一组
通往特定目的地的等价最佳路径）：

- 只有目的类型为一个网络或AS边界路由器（ASBR）的，才会由汇总LAS通告。如果路由条目的目的类型是区域边界路由器，检查下一个路由条目。

- AS外部路由决不会由汇总LSA通告。如果路由条目的路径类型（Path-type）有type 1 external或type 2external中的任意一个，检查下一个路由条目。

- 此外，如果与这组路径关联的区域是区域A本身，则不要为路径生成汇总LSA。

- 此外，如果与这组路径关联的下一跳属于区域A本身，则不要为路径生成汇总LSA。这是水平分隔的逻辑。

- 此外，如果路由目的是一个ASBR，仅且路由条目是去往这个ASBR的最优路径时会产生一个类型4的汇总LSA。链路状态ID是ASBR的路由器ID，代价为路由表中代价。注意：如果区域A被配置为一个stub区域，则不会产生这些LSA。

- 此外，如果目的类型是网络。又如果这是一条域间（inter-area）路由，则产生一个类型3的汇总LSA，链路状态ID是网络的地址（如果有必要，网络状态ID可以有一位或多位主机位置位），代价为路由表中的代价。

- 最后一种情况是去往一个网络的域内（intra-area）路由。这意味着这个网络处于与路由器直接相连的区域内。通常，这些信息在汇总LSA里出现之前必须被压缩。请记住，某个区域具有已配置的地址范围列表，每个范围包含[地址，掩码]，以及Advertise或DoNotAdvertise的状态指示。每个范围最多只产生一个Type 3汇总LSA。当范围的状态指示Advertise时，将生成Type 3汇总LSA，其Link State ID等于范围的地址（主机位可根据需要置位），并且代价等于组件网络中最大的那个。当范围的状态指示DoNotAdvertise时，Type 3汇总LSA会被抑制，并且组件网络对其他区域隐藏。

  (...省略...)
  
  In other words, the backbone’s configured ranges should be ignored when originating summary-LSAs into transit areas.

  TODO
  
#### 自主系统外部LSA

自主系统外部LSA是由ASBR路由器始发的。这些自主系统外部LSA是用来通告OSPF自主系统外部的目的网络的，这里也包括到达外部目的网络的缺省路由。

自主系统外部LSA可以泛洪到OSPF域中所有非末梢区域中去。

![OSPF的自主系统外部LSA](https://raw.githubusercontent.com/zyywo/zyywo.pic/master/OSPF%E7%9A%84%E8%87%AA%E4%B8%BB%E7%B3%BB%E7%BB%9F%E5%A4%96%E9%83%A8LSA.png "OSPF的自主系统外部LSA")

- **链路状态ID**

  自主系统外部LSA的链路状态ID是指目的地的IP地址。
  
- **网络掩码**

  是指所通告的目的地的子网掩码。
  
如果类型5的LSA正在通告的是一条缺省路由，那么链路状态ID和网络掩码字段都将被设置为0.0.0.0。
  
- **E，或称为外部度量位（External Metric bit）**

  用来指定这条路由使用的外部度量的类型。如果E-bit为0，那么度量类型就是E1；如果E-bit为1，那么度量类型就是E2。（E1，E2参考330页后再补充）
  
- **度量**

  是指路由的代价，由ASBR路由器设定。
  
- **转发地址（Forwarding Address）**

  是指到达所通告的目的地的数据包应该被转发到的地址。如果转发地址是0.0.0.0，那么数据包将被转发到始发ASBR上。
  
- **外部路由标志（External Route Tag）**

  是一个应用于外部路由的任意标志。OSPF协议本身并不使用这个字段，而是由外部路由来管理和控制。
  
可选地，TOS字段也可以和某个目的地关联。这些字段和前面讲述的是相同的，只是每一个TOS度量也都有自己的E-bit、转发地址和外部路由标志。

##### 构造自治系统外部LSA（AS-external-LSAs）

AS外部LSA描述到自治系统外部目的地的路由。大部分AS外部LSA描述的都是到特定外部目的地的路由，这时LSA的链路状态ID为目的网络的IP地址。然而，AS外部LSA也可能会为自治系统生成一条默认路由，这时LSA的状态ID为默认地址（0.0.0.0）。

AS外部LSA由AS边界路由器（ASBR）生成。ASBR为从其它路由协议学到的每条外部路由生成一个AS外部LSA。

AS外部LSA是惟一一种在整个自治系统泛洪的LSA类型，其他类型的LSA只会在单一区域内泛洪。但是，AS外部LSA不会被泛洪到或穿过末梢区域。

被通告的外部路由的代价有两种，类型为1的代价与链路状态代价相同；类型为2的代价被认为比任何AS内部路径的代价都大。

如果由AS外部LSA通告的路由变为了不可达，必须把LSA的age设置为MaxAge并在路由域内泛洪。

#### NSSA外部LSA

NSSA外部LSA是由一个NSSA区域内的ASBR路由器始发的。除了转发地址外，NSSA外部LSA的所有字段都和AS外部LSA的字段相同。

NSSA外部LSA仅仅在始发它们的一个非纯末梢区域内进行泛洪。

![OSPF的NSSA外部LSA](https://raw.githubusercontent.com/zyywo/zyywo.pic/master/OSPF%E7%9A%84NSSA%E5%A4%96%E9%83%A8LSA.png "OSPF的NSSA外部LSA")

- **转发地址**

  如果网络在一台NSSA ASBR路由器和邻接的自主系统之间是作为一条内部路由通告的，那么这个转发地址就是指这个网络的下一跳地址。如果网络不是作为一条内部路由通告，那么这个转发地址将是NSSA ASBR路由器的路由器ID。
  
### 可选项字段

可选字段是出现在每一个Hello数据包、数据库描述数据包和每一个LSA中的。可选字段允许路由器和其他路由器进行一些可选性能的通信。

![OSPF的可选项字段](https://raw.githubusercontent.com/zyywo/zyywo.pic/master/OSPF%E7%9A%84%E5%8F%AF%E9%80%89%E9%A1%B9%E5%AD%97%E6%AE%B5.png "OSPF的可选项字段")

- **DN**    用于基于MPLS的第3层虚拟专用网（VPN）。

- **O**    设置用来表明始发路由器支持Opaque LSA（类型9,类型10,类型11）

- **DC位**    当始发路由器具有支持按需电路上的OSPF的能力时，该位被设置为1。

- **EA位**    当始发路由器具有接收和转发外部属性LSA的能力（//TODO：需要详细解释）时，该位将被设置为1。

- **N位**    只用在Hello数据包中。一台路由器设置N-bit=1表明它支持NSSA外部LSA。如果N-bit=0,那么路由器将不接受和发送NSSA外部LSA。如果N-bit=1，那么E-bit必须设置为0。

- **P位**    只用在NSSA外部LSA的头部（由于这种情况，N-bit和P-bit可以使用在同一位置）（TODO:嘛意思）。该位将告诉一个非纯末梢区域中的ABR路由器将类型7的LSA转换为类型5的LSA。

- **MC位**    当始发路由器具有转发IP组播数据包的能力时，该位将被设置。

- **E位**    当始发路由器具有接受AS外部LSA的能力时，该位将被设置。在所有的AS外部LSA和所有始发于骨干区域以及非末梢区域的LSA中，该位将设置为1.而在所有始发于末梢区域的LSA中当，该位设置为0.另外，可以在Hello数据包中使用该位来表明一个接口具有接收和发送类型5的LSA的能力。E-bit不匹配的路由器将不能形成邻接关系，这个限制可以确保一个区域的所有路由器都同样地具有支持末梢区域的能力。

- **MT位**    设置了该位表示始发路由器支持多拓扑OSPF（MT-OSPF）。

[default-information originate](https://blog.csdn.net/weixin_33995481/article/details/91631654)

[default-information originate](https://blog.51cto.com/lcbingo/578728)	