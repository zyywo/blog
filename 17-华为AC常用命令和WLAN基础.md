<!--markdown--># AC的一些操作命令

* **display wlan calibrate statistics**
  >输出信息描述：
  >
  >signal environment deteriaration 信号环境恶化次数
  >
  >power/channel calibration 功率/信道调整次数
  
* **display ap neghbor ap-id 233 radio 1**
    >查看AP指定射频的邻居信息
    
* **display ap sta-signal strength ap-id 1**
    >查看AP上STA的平均信号强度
    
* **display ap traffic statistics wireless**
    >查看AP射频上的报文统计数

* **display vap ap-id {ap-id}**
    >查看某个AP开放的SSID

* **display vap all**
    >查看每个AP开放的SSID

* **display radio ap-id {ap-id}**
    >查看AP的信道信息（包括信道的带宽、信号强度、关联数量、信道利用率、工作模式等

# 离线添加AP上线的基本配置

离线添加的AP需要有以下基本配置才会上线：

1. IP地址方式改为static，默认是DHCP方式
2. IP：AP需要一个IP来建立CAPWAP隧道。否则AP不会尝试建立CAPWAP隧道，AC上会显示AP是Idle状态。另外如果没有配置AC，会以广播方式请求建立；如果有AC，又分为同网段与不同网段两种情况。

AC需要以下配置：
1. 指定CAPWAP源：如果没有指定CAPWAP源，AC不会响应AP的CAPWAP建立请求。**AC设置CAPWAP源后重启才会生效**。
2. 认证模式：默认是MAC认证
2. 添加AP：如`ap-id 0 ap-mac aabb-ccdd-eeff`会添加一个MAC为aabb-ccdd-eeff的AP，AP认证通过后AC会自动添加type-id和sn。

#### Notes
1. 如果AP和AC在同一网段，AP会ARP得到AC的MAC，然后向AC单播{二层，三层？}CAPWAP协议的discovery request。连续10个包没有回应，会广播一个discovery request,如此反复。**AP没有与AC建立CAPWAP隧道时，会处于idle状态**。

2. 如果AP和AC不在同一网段，AP会先验证网络的连通性，然后向AC单播{三层？}CAPWAP协议的discovery request。连续10个包没有回应，会广播一个discovery request,如此反复。

3. AP发送的discovery request报文里有WTP Board Data选项，该选项有vender、AP型号、AP的SN、AP的MAC，AC可以根据此选项回应相关报文。

## AP配置
```sh
[系统视图]# ap-address mode static #这条命令是AP配置静态IP的前置条件
[系统视图]# ap-adress static ip-address {ip} {pre-mask} #需要重启才生效
[系统视图]# ap-address static ac-list {ip} #需要重启才生效
[系统视图]# ap-mode-switch fat ftp {file-name} {ftp-server-ip} #把AP切换为FAT模式
[系统视图]# ap-mode-switch check #检查设备文件系统状态是否允许形态切换，8030DN，8130DN不需要该命令
[系统视图]# ap-mode-switch prepare #切换设备文件系统状态使FIT AP可以切换为FAT AP
```

# WLAN基本业务配置流程图

![WLAN基本业务配置流程图](https://download.huawei.com/mdl/imgDownload?uuid=d8dbfa7ba914404bb6b09946ee89e44e.png)

# WLAN模板引用关系图

![WLAN模板引用关系图](https://download.huawei.com/mdl/imgDownload?uuid=08091058512b4755a3a6d408ae0a262d.png)
	