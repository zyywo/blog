<!--markdown--># BGP简介

BGP的设计目的是连接不受自己管理控制的域中的邻居。通常无法信任这些邻居，必须利用路由策略仔细控制与这些邻居交换的信息。

入站路由宣告会影响出站流量，出站路由宣告会影响入站流量。

AS号64512～65535被保留用作私有用途。

- BGP仅发送单播消息，并且与每个对等体建立一个独立的点到点连接。

- BGP是一种为其点到点连接使用TCP（端口179）的应用层协议，依靠TCP的内在特性实现会话的维护功能（如确认、重传和排序）。

- BGP是一种矢量协议，通常称为路径矢量协议，而不是距离矢量协议。

- BGP路由利用AS_PATH路由属性来描述路径矢量，AS_PATH按序列出了到达目的端的路径所包含的AS号。

- AS_PATH属性是一种最短路径行列式，如果有多条路径去往同一个目的端，那么AS_PATH中AS号最少的路径就是最短路径。

- AS_PATH列表中的AS号可以实现环路检测机制。路由器收到BGP路由后，如果发现自己的AS号位于该路由的AS_PATH列表中，那么就认为出现了环路，从而丢弃该路由。

- 如果路由器与拥有不同AS号的邻居建立了BGP会话，那么就将该BGP会话称为EBGP会话，如果邻居与该路由器的AS号相同，那么就称为IBGP会话，此时分别将这两类邻居称为外部邻居和内部邻居。

## BGP消息类型

在建立BGP对等体之前，两个邻居必须执行标准的TCP三次握手进程，并打开到端口179的TCP连接。所有的BGP消息都采用单播方式经TCP连接传递给邻居。

BGP使用以下4种基本消息类型：

- Open（打开）消息

- Keepalive（保持激活）消息

- Update（更新）消息

- Notification（通告）消息

> 还有第五种BGP消息：Route Refresh（路由刷新），但是第五种消息不属于基本的BGP功能，因而可能不是所有的BGP路由器都支持该消息。

### Open 消息

TCP会话建立后，两个邻居之间需要发送Open消息，每个邻居都用该消息标识自己并指定BGP操作参数。Open消息包括以下信息。

- **BGP版本号（BGP version number）**

  该字段指定发起端正在运行的BGP版本号（2、3或4）。可以在配置会话时利用`neighbor version`命令要求邻居使用指定版本。

- **自治系统号（Autonomous system number）**

  该字段表示的是会话发起路由器的AS号，该信息可以确定BGP会话是EBGP会话或IBGP会话。

- **保持时间（Hold time）**

  该字段表示路由器在收到Keepalive消息或Update消息之前可以等待的最长时间（以秒为单位）。保持时间必须是0秒（此时必须不发送Keepalive消息）或至少3秒，IOS默认为180秒。如果邻居双方的保持时间不一致，那么将以较短的时间作为双方可以接受的保持时间。可以通过配置语句`timers bgp`来更改整个BGP进程的默认保持时间，也可以通过`neighbor timers`来更改指定邻居或对等体组的默认保持时间。

- **BGP标识符（BGP identifier）**

  该字段标识邻居的IPv4地址。IOS确定BGP标识符的过程与OSPF确定路由器ID的过程完全一致：使用数值最大的环回地址；如果环回接口没有配置IP地址，那么就选择数值最大的物理接口IP地址。也可以通过`BGP router-id`命令手工指定BGP标识符。

- **可选参数（Optional parameters）**

  该字段用来宣告验证、多协议支持以及路由刷新等可选支持能力。

### Keepalive 消息

如果路由器接受邻居发送的Open消息中指定的与参数，那么就响应一条Keepalive消息。此后IOS将默认每60秒发送一条Keepalive消息，或者是按照已协商的保持时间的1/3为周期发送Keepalive消息。

### Update 消息

Update消息用于宣告可行路由、撤销路由或两者。Update消息包括以下信息。

- **NLRI（Network Layer Reachability Information，网络层可达性信息）**

  该字段是一个或多个宣告目的端前缀及其长度的（长度，前缀）二元组。例如，如果宣告的是`206.193.160.0/19`,那么表示长度部分为`/19`，前缀部分为`206.193.160`。
  
- **路径属性（Path Attributes）**

  该属性提供了允许BGP选择最短路径、检测路由环路并确定路由策略的相关信息。
  
- **撤销路由（Withdrawn Routes）**

  描述已成为不可达且退出服务的目的端的（长度，前缀）二元组。
  
 虽然NLRI字段可以包含多个前缀，但每条Update消息仅描述单条BGP路径（这是因为路径属性仅描述单条路径，只是该路径可能会通过多个目的端）。
 
### Notification 消息
 
路由器检测到差错之后会发送Notification消息，并且总要关闭BGP连接。

## BGP有限状态机

下面的图和表给出了完整的BGP有限状态机以及触发状态迁移的各种输入事件（IE）

![BGP有限状态机](https://raw.githubusercontent.com/zyywo/zyywo.pic/master/BGP%E6%9C%89%E9%99%90%E7%8A%B6%E6%80%81%E6%9C%BA.png "BGP有限状态机")

| IE | 描述 |
| ---| ----|
| 1 | BGP启动（Start） |
| 2 | BGP终止（Stop） |
| 3 | BGP传输（Transport）连接打开 |
| 4 | BGP传输（Transport）连接关闭 |
| 5 | BGP传输（Transport）连接打开失败 |
| 6 | BGP传输（Transport）致命性错误 |
| 7 | ConnectRetry（连接重试）定时器到期 |
| 8 | Hold（保持）定时器到期 |
| 9 | Keepalive定时器到期 |
|10 | 接收Open消息 |
|11 | 接收Keepalive消息 |
|12 | 接收Update消息 |
|13 | 接收Notification消息 |

- **Idle（空闲）状态**

  BGP总是以Idle状态为起始点，该状态拒绝所有入站连接。如果发生差错，BGP进程将迁移到Idle状态。在第一次迁移到Idle之后，路由器会设置ConnectRetry定时器，在定时器到期时才会重新再启BGP。IOS的初始ConnectRetry时间为120秒，该值不可更改，以后每次ConnectRetry时间都是之前的两倍。

- **Connect（连接）状态**
  
  该状态下，BGP进程一直等待TCP连接的完成。如果TCP连接建立成功，BGP进程将会向邻居发送Open消息并进入OpenSent（打开发送）状态。如果TCP连接建立不成功，BGP进程将继续侦听由邻居初始化的连接、重置ConnectRety定时器，并迁移到Active（激活）状态。
  
  如果ConnectRetry定时器到期时仍处于Connect状态，则重置定时器，并再次尝试与邻居建立TCP连接，进程也将继续维持在Connect状态，其他输入事件将会让BGP进程迁移到Idle状态。

- **Active（激活）状态**
  
  该状态下，BGP进程会尝试与邻居初始化TCP连接。如果TCP连接连接建立成功，BGP进程会清除ConnectRetry定时器、完成初始化过程、向其邻居发送Open消息，并迁移到OpenSent（打开发送）状态。IOS默认的保持时间为180秒，可以通过`timers bgp statement`命令设置保持时间。

- **OpenSent（打开发送）状态**
  
  该状态下，已经发送了Open消息，BGP会一直等待直至侦听到来自邻居的Open消息。接收到Open消息后，会检查该消息的每个字段，如果存在差错，则会发送Notification消息并迁移到Idle状态。

  如果接收到的Open消息没有差错，则发送Keepalive消息并设置Keepalive定时器，此外还要协商保持时间，以确定一个较小的保持时间值，如果协商后的保持时间为零，则不启动保持定时器和Keepalive定时器。根据对等体的AS号，可以确定对等连接是内部连接还是外部连接，并迁移到OpenConfirm（打开确认）状态。

  如果收到断开TCP连接的请求，则本地进程将关闭BGP连接、重置ConnectRetry定时器、开始侦听由邻居发起的新连接，并迁移到Active状态。

- **OpenConfirm（打开确认）状态**

  该状态下，BGP进程等待Keepalive消息或Notification消息，如果收到的是Keepalive消息，则迁移到Established状态；如果接收到的Notification消息或断开TCP连接请求，则迁移到Idle状态。

  如果保持定时器到期，或检测到差错，或发生了终止事件，则向邻居发送一条Notification消息、关闭BGP连接，并将状态更改为Idle状态。

- **Established（建立）状态**

  该状态下，BGP对待连接已完全建立，对等体之间可以相互交换Update、Keepalive和Notification消息。如果接收到的是Update或Keepalive消息，则重新启动保持定时器（如果协商好的保持时间不是零）。如果接收到的是Notification消息，则迁移到Idle状态。其他事件（启动事件除外，因为该状态会忽略启动事件）将会让BGP进程发送一条Notification消息并迁移到空闲状态。

## 路径属性

路径属性有以下4类：

- **周知强制属性**
  
  ORIGIN, AS_PATH, NEXT_HOP

- **周知自选属性**

  LOCAL_PREF, ATOMIC_AGGREGATE

- **可选传递性属性**

  AGGREGATOR, COMMUNITY, EXTENDED COMMUNITY, AS4_PATY, AS4_AGGREGATOR

- **可选非传递性属性**

  MULTI_EXIT_DISC, ORIGINATOR_ID, CLUSTER_LIST, Multiprotocol Reachable NLRI, Multiprotocol Unreachable NLRI

周知属性包括强制属性（即必须包含在所有的BGP Update消息中）或自选属性（即可以包含在特定Update消息中，也可以不包含在特定Update消息中）。

如果可选属性是传递的，那么BGP进程就应该接受该属性中包含的Update消息（即使该进程并不支持该属性），而且应该将该属性传递给对等体。如果可选属性是非传递性的，那么无法识别该属性的BGP进程可以忽略该属性中包含的Update消息，而且不将该路径宣告给其他对等体。简单而言，就是属性可以通过或不可以路由器进行传递。

### OGRIGIN 属性

OGRIGN是一种周知强制属性，指定了路由更新的来源。如果BGP存在多条去往同一个目的端的路由时，那么就将OGRIGIN属性作为确定优选路由的要素之一。

### AS_PATH属性

AS_PATH属性是一种周知强制属性，该属性利用一串AS号来描述去往由NLRI（Network Layer Reachability Information，网络层可达性信息）指定的目的地的AS间路径或AS级路由。AS发起路由（在其AS内向外部邻居宣告目的地的NLRI）时，会将自己的AS号添加到AS_PATH中。

仅当Update消息发送给其他AS（也就是EBGP对等体）时，BGP路由器才会在AS_PATH中添加自己的AS号

### NEXT_HOP属性

该周知强制属性描述了去往所宣告目的端的路径上的下一跳路由器的IP地址。但是与通常的IGP不同，BGP NEXT_HOP属性所描述的IP地址并不总是邻居路由器的IP地址，其规则如下：

- 如果宣告路由器与接收路由器位于不同的自治系统中，那么NEXT_HOP是宣告路由器的接口IP地址；

- 如果宣告路由器与接收路由器位于同一自治系统中（内部对等体），且Update消息的NLRI指向的是同一AS内的目的地，那么NEXT_HOP是发起路由器的IP地址；

- 如果宣告路由器和接收路由器是内部对等体，且Update消息的NLRI指向的是不同AS内的目的地，那么NEXT_HOP是外部对等体（通过该对等体学习到该路由）的IP地址。

### 权重

权重是Cisco的专有BGP路径属性，仅对单台路由器内的路由器有效，无法与其他路由器进行通信。

## BGP决策进程

华为规定的决策进程如下:
1. 优选Prefferred_Value属性值最大的路由
2. 优选Local_Preference属性值最大的路由
3. 本地始发的BGP路由优于从其他对等体学习到的路由。其中本地始发的路由类型按优先级从高到低的排列是：`通过手工汇总的方式发布的路由`、`通过自动汇总的方式发布的路由`、`通过network命令发布的路由`以及`通过import-route命令发布的路由`。
4. 优选AS_Path属性值最短的路由
5. 优选Origin属性最优的路由。Origin属性值按优先级从高到低的排列是：IGP、EGP、及Incomplete。
6. 优选MED属性值最小的路由
7. 优选从EBGP对等体学来的路由（EBGP路由优先级高于IBGP路由）。
8. 优选到Next_Hop的IGP度量值最小的路由。
9. 优选Cluster_List最短的路由。
10. 优选Router-ID最小的设备通告的路由。
11. 优选具有最小IP地址（Peer命令所指定的地址）的对等体通告的路由。

## BGP消息格式

BGP消息是在TCP报文段中使用TCP端口179进行承载的，最大消息长度为4096个字节，最小长度为19个字节，所有的BGP消息都有一个相同的头部。

![BGP消息头部](https://raw.githubusercontent.com/zyywo/zyywo.pic/master/BGP%E6%B6%88%E6%81%AF%E5%A4%B4%E9%83%A8.png "BGP消息头部")

- **标志（marker）**：该字段长16字节，用于检测BGP对等体之间的同步丢失情况，并且在支持验证功能的情况下可以进行消息验证。目前所有的BGP实现都将该字段始终设置为全1,在消息头部保留该字段的原因是实现后向兼容性。

- **长度（length）**：该字段长2个字节，指示消息的全部长度（包括长度），以字节为单位。

- **类型（type）**：该字段长1字节，指示消息的类型。

|代码 |类型 |
|---  |---- |
|1    |Open（打开）|
|2    |Update（更新）|
|3    |Notification（通告）|
|4    |Keepalive（保持激活）|
|5    |Route Refresh（路由刷新）|

## Open消息

Open消息是TCP连接建立后发出的第一条消息，如果收到的Open消息是可接受的，那么就发送一条Keepalive消息，以确认该Open消息。确认了Open消息之后，BGP连接就处于Established（建立）状态，可以发送Update、Keepalive以及Notification消息。

![BGP Open消息的格式](https://raw.githubusercontent.com/zyywo/zyywo.pic/master/BGP%20Open%E6%B6%88%E6%81%AF%E7%9A%84%E6%A0%BC%E5%BC%8F.png "BGP Open消息的格式")

Open消息的最小长度（包含BGP消息头部）是29个字节。

BGP Open消息包含以下字段。

- **版本（Version）**：该字段长1个字节，用于指定发起方运行的BGP版本。

- **我的自治系统（My Autonomous System）**：该字段长2字节，用于指定发起方的AS号。

- **BGP标识符（BGP Identifier）**：是发起方的BGP路由ID，除非在BGP配置中指定路由器ID，否则IOS将路由器ID设置为最大的环回接口IP地址，如果没有 配置环回接口，那么就设置为最大的物理接口IP地址。

- **可选参数长度（Optional Parameters Length）**：该字段长1个字节，表示后面可选参数字段的长度（以字节为单位），如果该字段值为0,那么就表示该消息中无可选参数字段。

- **可选参数（Optional Parameters）**：该可变字段包含一个可选参数列表，每个参数都由一个长度为1字节的类型字段、一个长为1字节的长度字段以及一个包含参数值的可变长度字段组成。

## Update消息

该消息的作用是向对等体宣告一条可行路由或撤销多条不可行路由或者两者。
|BGP Update消息的格式|
|-----------|
|不可行路由的长度（2个字节）|
|被撤消路由（可变长度）   |
|全部路径属性长度（2个字节）|
|路径属性（可变长度）|
|网络层可达性信息（可变长度）|

- **不可行路由的长度（Unfeasible Routes Length）**：该字段长2个字节，用于指示后面被撤销路由（Withdraw Routes）字段的长度，该字段值为0时表示没有被撤销的路由，且Update消息中无被撤销路由字段。

- **被撤销路由（Withdraw Routes）**：该可变长度字段包含了一个要退出服务的路由列表，列表中的每条路由都以（长度，前缀）二元组形式加以表示，其中，长度表示前缀的长度，前缀表示被撤销路由的IP地址前缀。如果二元组中的长度部分为0,那么前缀部分将匹配所有路由。

- **全部路径属性的长度（Total Path Attribute Length）**：该字段长2个字节，用于指示后面的路径属性字段的长度（以字节为单位）。字段值为0时表示Update消息中未包含路径属性的NLRI。

- **路径属性（Path Attributes）**：该可变长度字段列出了与后面NLRI字段相关联的属性信息，每个路径属性都以可变长度的三元组（属性类型，属性长度，属性值）进行表示，该三元组中的属性类型部分是一个长为2个字节的字段，由4个标记位、4个未用位以及1个属性类型代码组成。下表列出了最常见的一些属性类型代码以及每种属性类型的可能属性值。

|  |  |  |  |  |  |  |  |              |
|--|--|--|--|--|--|--|--|--------------|
|O |T |P |E |U |U |U |U | 属性类型代码 |

  O：可选位。0=周知，1=可选。

  T：转接位。0=非转接，1=转接。

  P：部分位。0=可选转接属性完整，1=可选转接属性不完整。

  E：扩展长度位。0=属性长度为一个字节，1=属性长度为两个字节。

  U：未用

|类型代码 |属性类型 |属性值代码 |属性值 |
|---------|---------|-----------|-------|
|1        |ORIGIN   |0          |IGP    |
|         |         |1          |EGP    |
|         |         |2          |不完全 |
|2        |AS_PATH  |1          |AS_SET |
|         |         |2          |AS_SEQUENCE|
|         |         |3          |AS_CONFED_SET|
|         |         |4          |AS_CONFED_SEQUENCE|
|3        |NEXT_HOP |0          |下一跳IP地址|
|4        |MULTI_EXIT_DISC|0    |4个字节的MED|
|5        |LOCAL_PREF|0         |4个字节的LOCAL_PREF|
|6        |ATOMIC_AGGREGATER|0  |无     |
|7        |AGGREGATOR|0         |AS号及聚合设备的IP地址|
|8        |COMMUNITY|0          |4个字节团体标识符|
|9        |ORIGINATOR_ID|0      |4个字节的发起方路由器ID|


- **网络层可达性信息（Network Layer Reachability Information）**：该可变长度字段包含一个（长度，前缀）二元组，其中，长度部分以比特位为单位表示后面的前缀长度，前缀部分则是NLRI的IP地址前缀。如果长度部分取值为0,那么就表示前缀将匹配所有IP地址。

## Keepalive消息

Keepalive消息以保持时间的1/3（但不得小于1秒钟）为周期进行交换，如果协商后的保持时间为0,那么就不发送Keepalive消息。

Keepalive消息仅包含长度为19个字节的BGP消息头部，不包含其他数据。

## Notification消息

路由器检测到差错条件后就会发送该消息。该消息发送后，将立即关闭BGP连接。

## IBGP对等会话

同一AS中的每台BGP路由器之间必须建立一条直接的IBGP连接。 

### 使用Loopback接口建立IBGP邻居

如果在指定邻居时使用邻居的物理接口IP，那么在物理接口Down掉时，邻居关系就会断开，解决方案是在环回接口之间配置对等会话。

在环回接口之间配置IBGP对等会话不仅相要为IBGP会话远端指定邻居的环回地址，还必须将本地路由器的环回接口指定为IBGP会话的发起端（默认情况下，出站TCP会话源自出站物理接口地址）。

## 直连检查与EBGP多跳

通常情况下，EBGP对等体关系必须基于直连接口建立，因为缺省情况下，EBGP对等体之间发送的BGP协议报文的TTL值为1,这使得这些协议报文只能够被传送一跳。

绝大多数EBGP会话的标准实践方式是：EBGP会话运行在直连物理接口之间。IOS默认设置通过以下两种措施来确保EBGP对等体之间是直接相连的：

- 检查已配置邻居的IP地址，以确定该地址属于直连子网；

- 将发送给外部对等体且包含BGP消息的数据包的TTL值设为1。

如果外部BGP邻居是直连邻居，但邻居地址不属于本地子网（比如环回口之间建立会话），可以用`neighbor disable-connected-check`禁用IOS的直连检查特性。

如果两台路由器并未直连，EBGP会话必须穿越一台或多台没有运行BGP的路由器，可以使用`neighbor ebgp-multihop`更改发送给指定邻居的EBGP消息的默认TTL值。这条语句会自动禁用直连检查机制。

## 管理和保护BGP连接

`neighbor ttl-security`功能特性会改变接收到的EBGP消息包的可接受TTL值以及发送的BGP消息包的TTL值。比如`neighbor 192.168.1.225 ttl-security hops 1`语句会造成以下两个变化：

- 从邻居192.168.1.255接收的BGP消息包的TTL值必须是254甚至更大（大于255-1）；

- 将本地路由器发送给邻居192.168.1.255的BGP消息包的TTL值设置为255。

# BGP与NLRI（网络层可达性信息）

与IGP不同，BGP默认并不宣告任何可达性信息。因此，应该始终记着向BGP“注入”前缀。

## 利用network语句注入前缀

例：`network 192.168.100.0`，network只能宣告A类，B类，C类前缀，IOS的BGP实现将自动填充相应的8bit，16bit，24bit长度值，如果是其他地址可以使用network mask语句。

BGP的**network**与在接口上启用协议毫无关系，只是用来指定将要注入本地BGP进程的前缀。如果前缀是通过network语句指定的，那么BGP就会查询IP路由表，如果指定前缀不在表中，那么BGP就不会让前缀进入BGP表中，也就是说，除非路由器拥有去往指定目的地的有效路径，否则BGP不会注入前缀。

查看BGP表的语句：`show ip bgp`。

默认情况下，被注入前缀的IGP度量将会成为BGP路由的MED（MULTI_EXIT_DISC）属性，在BGP表中显示为Metric。

权重（weight）是IOS的专有属性，本地路由器注入前缀的默认权重是32768。

## 利用network mask语句注入前缀

例：`network 192.168.172.0 mask 255.255.255.252.0`，此条语句注入了前缀192.168.172.0，掩码255.255.252.0表示使用的前缀长度为22bit。

## 利用重分发注入前缀

默认情况是重分发IGP知道的所有路由，如果希望仅宣告部分IGP路由，那么就必须过滤其他路由。

```
router bgp 200
 no synchronization
 redistribute eigrp 200 router-map ROUTES_IN
 neighbor 192.168.1.226 remote-as 100
 no auto-summary

access-list 1 deny 192.168.1.224
access-list 1 deny 192.168.1.216
access-list 1 permit any

route-map ROUTES_IN premit 10
 match ip address 1
```
上例中增加路由过滤器以阻止注入前缀192.168.1.216/30和192.168.1.224/30，同时允许注入其他前缀。

被标记了`r`的前缀意味着BGP无法让前缀进入路由表（RIB）。`show ip bgp rib-failure`命令可以提供RIB-failure的详细原因。

对于没有进入路由表的前缀，BGP默认仍将该前缀宣告给其他对等体。`bgp suppress-inactive`语句可以阻止BGP这样做，即如果BGP表中的表项无法进入本地RIB，那么就不要将该前缀宣告给其他BGP对等体。

## 在IBGP拓扑结构中管理前缀（向EBGP或IBGP宣告路由的问题）

路由器向EBGP对等体宣告路由时，会将其出站接口地址添加为该路由的NEXT_HOP属性。默认情况下，路由器路向IBGP对等体宣告路由时，不会更改该路由的NEXT_HOP属性。

但实际网络中需要路由器将学自EBGP对等体的路由宣告给IBGP对等体时，路由器将NEXT_HOP更改为自己的某个地址，且该地址在本地AS内部已知（路由器ID，也就是环回接口地址）。

`neighbor next-hop-self`语句可以让路由器将宣告给邻居的所有路由的BGP NEXT_HOP属性都更改为自己的环回地址。

## IBGP与IGP同步

IGP同步是早期网络时代的产物，目前已经不再使用该功能，IOS12.2(8)T及以后的版本均默认**禁用同步功能**（**no synchronization**）。

同步规则的要求如下：

> 在学自IBGP邻居的路由进入本地路由表之前或者宣告给EBGP对等体之前，必须通过IGP知道该路由。






## IBGP水平分割
当路由器从一个IBGP对等体学习到某条BGP路由时，它将不能再把这条路由通告给任何IBGP对等体，这就是IBGP水平分割规则。为了解决因水平分割而导致的路由传递问题，可以在AS内部建立IBGP的对等体关系全互联模型（此外还有路由反射器和联邦两种解决方案）。

##### 配置
```
[BGP]# bestroute as-path-ignore //忽略AS-Path对路由优先级的影响
```

#TODO	