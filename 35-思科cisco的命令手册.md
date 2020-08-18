<!--markdown-->查看端口聚合信息

```
Switch#show etherchannel summary
Switch#show etherchannel load-balance   //查看负载信息
Switch#show etherchannel port-channel   //查看port-channel详细信息
Switch#show int trunk     查所有允许trunk 通道的VLAN
Switch#show int port-channel 1 trunk    查po1 允许通过的vlan  ，trunk 通道
#show standby ?
```

# 生成树命令

- `#show spanning-tree vlan 10`

    查看vlan10的生成树实例

- `(config)#spanning-tree mode {mst|pvst|rapid-pvst}`
  
    修改生成树的模式

- `(config)#spanning-tree vlan {vlan-id} priority {0-61440}`
  
    修改pvst生成树的优先级，数字越小优先级越高。


# 思科IOS的快捷键

`exit`：退回到上一级模式

`end`：可以在高于特权模式的模式下，直接退回到特权模式。

`Ctrl+A`：移动光标到行首

`Ctrl+E`：移动光标到行尾

`Ctrl+U`：清除整行内容

`Ctrl+Shift+6`：终止IOS的进程，如ping或traceroute

`Ctrl+C`：取消当前的命令并回到特权模式

`Ctrl+Z`：执行当前命令并回到特权模式

输入问号：ctrl+v，然后就可以输入问号了	