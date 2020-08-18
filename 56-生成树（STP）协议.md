<!--markdown-->STP为IEEE标准协议，并且有多个协议版本，版本与协议号的对应关系如下：

Common Spanning Tree (CST)  =  IEEE 802.1D

Rapid Spanning Tree Protocol (RSTP)  =  IEEE 802.1w

Per-VLAN Spanning-Tree plus (PVST+)   =   Per-VLAN EEE 802.1D

Rapid PVST+  =  Per-VLAN IEEE 802.1w

Multiple Spanning Tree Protocol (MSTP)   =    IEEE 802.1s 

STP在发送数据包测试网络是否有多条链路，是靠发送bridge protocol data units (BPDUs)来完成的，同台交换机发出去的BPDU都被做上了相同的标记，只要任何交换机从多个接口收到相同标记的BPDU，就表示网络中有冗余链路，因此需要STP断开多余链路。BPDU数据包里面有以下信息：

- 根交换机的bridge ID。

- 发送交换机的bridge ID 。

- 到根交换机的Path Cost。

- 发送接口以及优先级。

- Hello、forward delay、max-age时间。

同台交换机发出的BPDU，bridge ID都是一样的，因为是用来标识自己的，其中bridge ID由两部分组成：Bridge优先级和MAC地址，默认优先级为32678。

交换机上的每个端口也是有优先级的，默认为128，范围为0-255。
 
注：**在STP协议中，所有优先级数字越小，表示优先级越高，数字越大，优先级越低。**

# 选举规则

STP在计算网络时，需要在网络中选举出**根交换机（Root）**，**根端口（Root Port）**，以及**指定端口(Designated Port)**，才能保证网络的无环，选举规则分别如下：

**根交换机（Root）**

    在同一个三层网络中需要选举，即一个广播域内要选举，并且一个网络中只能选举一台根交换机。Birdge-ID中优先级最高（即数字最小）的为根交换机，优先级范围为0-65535，如果优先级相同，则MAC地址越小的为根交换机。

**根端口（Root Port）**

    所有非根交换机都要选举，非根交换机上选举的根端口就是普通交换机去往根交换机的唯一链路，选举规则为 到根交换机的Path Cost值最小的链路，如果多条链路到达根交换机的Path Cost值相同，则选举上一跳交换机Bridge-ID最小的链路，如果是经过的同一台交换机，则上一跳交换机Bridge-ID也是相同的，再选举对端端口优先级最小的链路，如果到达对端的多个端口优先级相同，最后选举交换机 对端端口号码最小的链路。

**指定端口(Designated Port)**

     在每个二层网段都要选举，也就是在每个冲突域需要选举，简单地理解为每条连接交换机的物理线路的两个端口中，有一个要被选举为指定端口，每个网段选举指定端口后，就能保证每个网段都有链路能够到达根交换机，选举规则和选举根端口一样，即：到根交换机的Path Cost值最小的链路，如果多条链路到达根交换机的Path Cost值相同，则选举上一跳交换机Bridge-ID最小的链路，如果是经过的同一台交换机，则上一跳交换机Bridge-ID也是相同的，再选举对端端口优先级最小的链路，如果到达对端的多个端口优先级相同，最后选举交换机 对端端口号码最小的链路。

 在STP选出根交换机，根端口以及指定端口后，其它所有端口全部被Block，为了防止环路，所以Block端口只有在根端口或指定端口失效的时候才有可能被启用。

#TODO

 转载自：[China-CCIE STP](http://www.china-ccie.com/ccie/lilun/switching/switching.html#6)	