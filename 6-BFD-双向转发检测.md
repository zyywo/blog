<!--markdown-->BFD（Bidirectional Forwarding Detection，双向转发检测），是一种实现网络可靠性的机制，它可以被用于快速检测网络中的链路状况、IP可达性等。BFD可以与多种协议或机制进行联动，以确保它们更加可靠的工作，例如静态路由、OSPF、IS-IS、BGP、VRRP、PIM、及MPLS LSP等。

##### 配置
```
[系统视图] bfd //全局启用BFD
[系统试图] quit
[系统视图] bfd atob bind peer-ip default-ip interface g0/0/0
[bfd视图] discriminator local 10 //本端标识符
[bfd视图] discriminator remote 20 //远端标识符
[bfd视图] commit //确认修改

```
###### 配置说明
`bfd atob bind peer-ip default-ip interface g0/0/0`：创建BFD会话，对端IP使用默认组播地址
##### 信息查看
`display bfd session all verbose`：显示BFD的详细信息	