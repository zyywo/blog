<!--markdown-->开启代理ARP后，路由器只要有去往`目的网络的路由`就会回应ARP请求。

cisco接口下默认启用代理ARP，使用下面的方法关闭代理ARP。
```
Router# configure terminal
Enter configuration commands, one per line.  End with CNTL/Z.
Router(config)# interface ethernet 0
Router(config-if)# no ip proxy-arp
Router(config-if)# ^Z
Router#
```

**代理ARP的优点**
1. 代理 ARP 的主要优点是它可以添加到网络上的单个路由器，而不会干扰网络上其他路由器的路由表。
2. 如果 IP 主机没有配置默认网关，或者没有任何路由智能功能，则必须在网络中使用代理 ARP。 

**代理ARP的缺点**
主机不了解其网络的物理详情，因而假定它是一个平面网络，它们只需通过发送 ARP 请求就能到达所有目标。但是，将 ARP 用于任何场合也有缺点。下面是一些缺点：
1. 它增加了网段中的 ARP 流量。
2. 主机需要更大的 ARP 表才能处理 IP 到 MAC 地址的映射。
3. 安全性可能遭到破坏。机器能声称是别的为了截断信息包，称“伪装的操作”。
4. 它不适用于不使用 ARP 进行地址解析的网络。
5. 它不通用于所有网络拓扑。例如，连接两个物理网络的多个路由器。

参考资料：
[代理 ARP - cisco](https://www.cisco.com/c/zh_cn/support/docs/ip/dynamic-address-allocation-resolution/13718-5.html)	