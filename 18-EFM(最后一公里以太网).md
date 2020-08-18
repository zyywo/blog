<!--markdown-->最后一公里以太网（EFM：Ethernet in the First Mile）主要用于接入部分的以太网物理层规范及以太网管理和维护，是链路级的OAM（Operation and Management）。针对两台直连设备之间的链路，提供链路连通性检测功能，链路故障监控功能，远端故障通知功能和远端环回功能。

EFM的RFC链接：[EFM RFC](https://www.rfc-editor.org/rfc/rfc4878.txt)

#### 协议报文
EFM OAM工作在链路层，其协议报文被称为OAMPDU。

EFM是**慢速协议**（Slow Protocols），慢速协议报文的特点是不能被网桥转发。

![报文格式](http://ov57zbjqo.bkt.clouddn.com/efm_oam_pdu.jpeg)

EFM实体默认每秒发一个OAMPDU。

#### EFM配置
```
[系统视图]# efm enable #全局开启EFM
[接口视图]# efm enable #接口开启EFM
[接口视图]# efm trigger {if-down | error-down}
```

if-down的接口下除EFM协议报文外的所有其他报文都无法被转发。如果当前接口通过EFM检测到链路连通性故障恢复，则会恢复该接口的所有报文转发。

error-down会把接口的管理状态置为admin down，而且需要用户手工决定是否回切（可以通过额外配置设置自动恢复）。

#### 调试排错
`display efm session all`：查看efm状态

#### 参考文档
[802.1ag CFM/802.3ah EFM OAM/Y.1731 ETH OAM学习笔记](https://blog.csdn.net/fw0124/article/details/5831096)

[介绍EFM OAM和CFD以太网OAM技术](http://network.chinabyte.com/435/12331935_2.shtml)	