<!--markdown-->
### 2.1.1 Representation of non-broadcast networks

OSPF在非广播网络上有两种运行模式：`NBMA` 或 `Point-to-MultiPoint`。这两种方式用不同的方法处理Hello协议和泛洪。

在NBMA模式下， OSPF模仿在广播网络下的操作方法：选举一台DR

在Point-to-MultiPoint模式下，OSPF把router-to-router的连接作为point-to-point链路。不会选举DR，也不会产生网络LSA。

（...不完整）

## 10.5 Receiving Hello Packets

本章节描述了对接收到的Hello报文的详细处理过程。

通常来说，首先对接收到的OSPF报文进行IP头部、OSPF头部校验，接下来，把Hello报文中的**网络掩码**、**Hello间隔**和**RouterDead Internal的值**与接收端口上配置的值对比，任何一项不匹配都会中止处理进程并把报文丢弃。换句话说，以上值域是对邻接网络配置的准确描述，然而有个例外：在点到点和虚链路上，接收的Hello报文中的网络掩码应该被忽略。

接收端口只能连接到一个OSPF区域（可以是主干区域），Hello报文中的**E-bit**设置必须与区域的ExternalRoutingCapability相匹配。如果AS-External-LSAs不泛洪到或穿过区域（比如stub区域），接收到的Hello报文中，E-bit必须清零，否则，E-bit必须置1。该项不匹配时报文会被丢弃，Hello报文中Option域的其他设备项应该被忽略。

接下来，会尝试把Hello报文的源与接收端口的邻居做匹配。如果接收端口连接到广播网络、点到多点网络或NBMA网络，Hello报文的IP报头中的IP源地址就是源；如果接收端口连接到点对点链路或虚链路，Hello报文的OSPF报头中的路由器ID就是源。接口的当前邻居列表包含在接口的数据结构中。如果不能在邻居中找到匹配项（如，第一次发现邻居）。则创建一个邻居，这个邻居初始状态设置为Down。

当在广播网络、点到多点网络或NBMA网络接收到邻居的Hello报文时，设置邻居数据结构中的邻居ID等于报文的OSPF报头中的路由器ID。对于这些网络类型，邻居数据结构中的路由器优先级、邻居的DR值和邻居的BDR值设置为所接收到的Hello报文中的对应值；应注意这些字段中的更改，以便在下面的步骤中使用。当在点到点网络（不是虚链路）收到Hello报文，设置邻居数据结构的邻居IP地址为报文的IP源地址。

现在,将检查hello报文的其余部分,生成要提供给邻居和接口状态机的事件。这些状态机被指定执行或调度(参考4.4)。例如，通过执行下面指定的状态机，单个接收到的hello可能会影响几个邻居状态转换。

- 每个Hello报文都会导致邻居状态机以HelloReceived事件执行。

- 然后检查hello包中包含的邻居列表。如果路由器自身出现在列表中，应在收到事件 2-WayReceived的情况下执行邻居状态机。否则，应在收到事件 1-WayReceived的情况下执行邻居状态机，并且停止处理报文。

- 接下来，如果注意到邻居的路由器优先级字段中有更改，那么接收端口的接口状态机将计划以事件NeighborChange执行。

- 如果邻居路由器声明它自己是DR（Hello报文中DR字段的值等于邻居的IP地址）且报文中BDR的值为0.0.0.0，并且接收端口在Waiting状态，接收端口的接口状态机将执行BackupSenn事件。否则，如果邻居声明自己是DR，而它(指邻居)以前没有，或者邻居没有声明自己是DR，而它以前有，那么接收接口的状态机将计划以事件NeighborChange执行。

- 如果邻居路由器声明它自己是BDR（Hello报文中BDR字段的值等于邻居的IP地址）并且接收端口在Waiting状态，接收端口的接口状态机将执行BackupSenn事件。否则，如果邻居声明自己是BDR，而它(指邻居)以前没有，或者邻居没有声明自己是BDR，而它以前有，那么接收接口的状态机将计划以事件NeighborChange执行。

在NBMA网络上，收到Hello数据包也可能导致发送回一个Hello数据包用来响应，详细内容参考9.5.1。

## 10.6 Receiving Database Description Packets

本章节对接收到数据库描述报文的详细处理过程进行说明。

入栈的数据库描述报文已经通过输入报文处理（Section 8.2），并与一个邻居和接收端口相关联。是否应该接收数据库描述报文或者已经接收了数据库描述报文接下来应该对它做什么处理，取决于邻居的状态。

如果接收了一个数据库描述报文，报文中的以下字段应该保存到对应的邻居数据结构的“last received Database Description Packet“下：initialize（I），More（M）和master（MS）位，选项字段，还有数据库描述序列号（DD Sequence Number）。如果从邻居接收到的两个连续的数据库描述报文中，这些字段完全一致，对于下面的处理方式来说，第二个报文被视为是重复的。

如果数据库描述报文中Interface MTU表明一个IP报文的尺寸大于路由器接收端口不分片的尺寸(MTU)，这个数据库描述报文会被拒绝（rejected）。否则，取决于邻居的状态，有以下行为：

- **Down**：这个报文应该被拒绝。

- **Attempt**：这个报文应该被拒绝。

- **Init**：邻居状态机应该执行 2-way Received 事件。这会导致邻居的状态立即转换到 2-way 或 Exstart。如果新状态是Exstart，这个报文会继续按照下面Exstart状态的方式处理。

- **2-Way**：这个报文应该被忽略，数据库描述报文只为用来建立邻接关系[7]

- **ExStart**：如果接收到的报文满足以下条件，邻居状态应该执行Negotiation Done事件（状态会转到Exchange），报文的Option字段会记录到邻居数据结构的Neighbor Options字段，并且这个报文会加入到处理队列中。否则，报文被丢弃。

  1. initialize（I），More（M）和master（MS）位被置1，报文的负载为空。并且邻居的路由器ID比自己的路由器ID大，这种情况下，路由器自己为Slave，设置master/slabe位为slave，设置邻居数据结构的DD序列号为master指定的值。
  
  2. initialize（I）和master（MS）位为零，报文中的DD序列号等于邻居数据结构的DD（这表示确认），并且邻居的路由器ID比自己的ID小，这种情况，路由器自身是master。
  
- **Exchange**：这个阶段下，master收到重复的DD报文会直接丢弃，slave收到重复的DD报文会把自己最后发送的DD报文重新发送一次。否则（DD报文不是重复的）:

  1. 如果MS位的状态与master/slave协商时的状态不一致，产生邻居事件SeqnumberMismatch并停止处理这个报文。
  
  2. 如果initialize（I）位置1，产生邻居事件邻居事件SeqNumberMismatch并停止处理这个报文。
  
  3. 如果报文的Options字段与之前从邻居接收到不同（记录在邻居数据结构的Neighbor Options字段中），产生邻居事件SeqNumberMismatch，并停止处理报文。
  
  4. 数据描述报文必须按照报文中的DD序列号顺序处理。如果路由器是master，则下一个收到的报文中DD序列号应该等于邻居数据结构中的DD序列号。如果路由器是slave，则下一个收到的报文中DD序列号应该比邻居数据结构中的DD序列号大1.无论哪种情况，如果这个报文是队列中的下一个，它应该被接受并按照下面步骤处理报文中的内容。
  
  5. 否则，产生邻居事件SeqNumberMismatch并停止处理报文。
  
- **Loading or Full**：

	