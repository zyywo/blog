<!--markdown-->#### 基本命令

`display bridge mac-address`：查看设备自身的MAC地址

`display as access configuration`：查看as设备的信息

`display interface ethernet brief`：查看以太网接口的简要信息

`display device manufacture-info`：查看设备制造信息，包括SN号

`display version`：查看运行时间、软件版本

`display vap`：查看无线VAP信息

`display efm session all`：查看efm状态

`display icmp statistics`：查看icmp收发状态

`display ap-address info`：查看ap的地址信息

`display mac-address [interface]`： 查看接口学到的mac表项

`display cpu-usage`：查看cpu使用率，diagnose模式下可以显示详细的进程

`display wlan wsta peak-statistics`：diagnose模式下查看用户periodic情况

`reset factory-configuration`：恢复出厂设置

`reset counters interface`：<用户视图>下，清除接口的统计信息

`switchover mode nonstop-routing`：不间断路由

#### 调试命令
`terminal debugging`：开启终端调试功能

`display logbuffer`：查看设备的日志

    cd log_file
    more log.log    查看更多的日志

`reset logbuffer`：清除设备日志

`display save time`：显示配置有无保存

调试时关闭不需要的打印干扰：
```sh
undo terminal alarm
undo terminal logging
undo terminal trapping
```

##### 使用trace

    <Huawei> terminal monitor
    <Huawei> terminal debug
    <Huawei> system-view
    [Huawei] trace enable
    [Huawei] trace access-user object 1 mac-address {mac}
    [Huawei] undo trace enable //得到信息后关闭trace//得到信息后关闭trace

#### 设备升级
`display startup`：查看本次启动的信息

`startup system-software {file.cc}`：升级软件

`startup patch {patch.pat}`：升级补丁

`delete {file}`：删除flash中的文件，不会释放空间，可以通过回收站恢复

`reset recycle-bin`：清空回收站，释放flash空间

升级版本后，可使用`check startup crc next`：检查下次启动的文件是否正常

`<HUAWEI> undo workmode`： 从云管理模式切换为普通模式

#### 交换机光口无法关闭自协商
在万兆口使用`undo negotiation auto`关闭自协商时，提示：Error:This port or this mode does not support this command.

由于万兆口都是默认强制为万兆的，所以万兆口下面没有自协商模式，无法关闭自协商，其他口可以正常执行。S5700系列的交换机万兆口下都无法关闭自协商。

#### 交换机的AS状态不刷新
这可能是因为AS与控制器之间的CAPWAP隧道异常导致的。
1. 删除控制器上指定的capwap源接口:`undo capwap source interface {interface}`
2. 再重新配置capwap源接口，让AS与控制器重新建立管理隧道，刷新数据。

#### 边缘端口与BPDU保护
启用边缘端口特性，使该端口不参与stp计算，并快速过渡之转发状态，命令:`[接口模式]# stp edged-port enable`

启用bpdu保护特性，当边缘端口接收到bpdu报文时，交换机会shutdown该端口，同时记录事件，如果不启用该特性，当边缘端口接收到bpdu报文时会转化为非边缘端口，并重新计算生成树，从而引起网络动荡。命令：`[接口模式]# stp bpdu-protection`

要恢复被shutdown的bpdu受保护边缘端口，可手工`undo shutdown`，或采用自动恢复的方式：系统视图下执行命令`error-down auto-recovery cause cause-item interval {time_of_second}`

#### 报错信息

`Error:failed to run this command because the connection was closed by remote host`

这个log表示用户级别太低，没有权限执行这个命令，提升用户级别可以解决。

#### 用户在线探测，用户同步
**user-detect interval 60 retry 2**
> 用户探测报文发送周期为60s（默认15s），重试2次(默认3次)，达到最大时间后（例子中是180s）没有收到用户的响应报文，认为用户下线。

**access-user arp-detect vlan 1 ip-address 1.1.1.1 mac-address aabb-ccdd-eeff**
>设置用户在线探测报文的源IP、源MAC及VLAN。如果在周期内收到用户的ARP回复，认定用户在线。

**usr-sync interval 600**
>接入设备每600s把所有在线用户的MAC通过capwap隧道发送给核心设备。

#### AAA认证
AAA可以通过域来对用户进行管理，不同的域可以关联不同的认证、授权和计费方案
```sh
display domain #查看设备上存在的域
display domain name default_admin #查看default_domain域的详细情况
dispaly authentication-scheme default #查看认证方案default的详细情况
```

对于单MAC多IP的终端，多个IP地址必须都通过认证终端才能上线。
使能`ip-static-user enable`功能，满足单MAC多IP地址终端的上线需求。

##### 命令行快捷键
`ctrl + a`：移动到行头

`ctrl + b`：向左移动一个字符

`ctrl + f`：向右移动一个字符	