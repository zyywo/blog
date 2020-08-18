<!--markdown-->参考网站：[Server World](https://www.server-world.info/en/)，需要翻墙。

# 查看系统信息与日志

`screenfetch`：查看系统信息

`dmidecode`：查看系统信息

`dmidecode -t memory `：查看内存信息

`lsb_release -a`：查看系统信息

`hostname -I`：查看主机的IP地址

`ethtool eth0`：查看网卡的信息

`fdisk`、`df`：查看硬盘信息

**查看日志**

`journalctl`: 系统日志

`/var/log/secure`：安全信息、用户登陆日志

`/var/log/cron` cron的日志路径

`journalctl -u cron` 查看关于cron的日志

**查看开机启动项**：

`systemctl list-unit-files --type=service | grep enabled`

**查看系统开机时各项服务的耗时**

`systemd-analyze blame`

**查看系统开机时间**

`last reboot`

`who -b`

`who -r`

**查处Apache服务是否开启**

```sh
systemctl is-enabled httpd
```

**SELinux**

`sestatus`：查看selinux状态

`setenforce 0`：临时关闭selinux，重启后会再开启

编辑`/etc/selinux/config`文件，设置`SELINUX=disabled`，重启后会永久关闭selinux

参考：[Linux 系统开机启动项清理](http://blog.jobbole.com/112384/)

**一些命令**：

`visudo`：编辑sudo用户

`dpkg -l | grep ^rc | awk '{print $2}' | xargs dpkg -P`：清理已经卸载了的包的配置信息

[5个经典有趣的linux命令：带日期的history，测硬盘写入速度，查找最大的文件，获取文件的详细信息](http://www.codeceo.com/article/5-interest-linux-command-tips.html)

`rename -n "s/-.*//" *`：批量前缀重命名

`du -sh * | sort -n`：统计当前目录大小，并按大小排序

**关于Linux的分区**

`/`：根目录，根据我的经验，在只有“/”和"home"两个分区时，15G的根目录就满足个人日常使用。

`/home`：用户家目录，建议单独分区

`/opt`：可以存放自己安装的软件，建议单独分区

##### systemd服务

systemd的服务分为`系统服务`和`用户服务`。

系统服务的默认执行者是root，可以通过user=的方式指定其他用户。

用户服务的执行者是用户，无法更改。

#### awk命令

使用`-F`参数改变分隔符，如：`awk -F '-'`。

#### find命令 exec

```sh
find / -type f -exec ls -l {} \\;
```

# 系统设置、系统管理

**日期管理**

```sh
timedatectl
```

**用户管理**

usermod

`useradd 用户名`：添加用户

`passwd 用户名`：修改用户密码

**把Debian家目录改为英文**:

>编辑`.config/user-dirs.dirs`文件，把里面的路径改为英文，并新建英文目录即可

**密钥管理**：

>使用软件`seahorse`可以图形化管理密钥，另外命令`apt-key add file.asc`或`gpg --import file.asc`可以安装密钥

**NetworkManager**：

关闭NetworkManager的方法：编辑/etc/NetworkManager/NetworkManager.conf文件，把managerd的值改为false

`nmcli`是它的CLI工具

## 设置永久的IP地址

### Ubuntu 18.04 LTS
```
oot@dlp:~# vi /etc/netplan/01-netcfg.yaml

# This file describes the network interfaces available on your system
# For more information, see netplan(5).
network:
  version: 2
  renderer: networkd
  ethernets:
    ens3:
      dhcp4: no
      # IP address/subnet mask
      addresses: [10.0.0.30/24]
      # default gateway
      gateway4: 10.0.0.1
      nameservers:
        # name server this host refers
        addresses: [10.0.0.10,10.0.0.11]
      dhcp6: no

# apply settings

root@dlp:~# netplan apply
root@dlp:~# ip addr

1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: ens3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 52:54:00:19:e8:09 brd ff:ff:ff:ff:ff:ff
    inet 10.0.0.30/24 brd 10.0.0.255 scope global ens3
       valid_lft forever preferred_lft forever
    inet6 fe80::5054:ff:fe19:e809/64 scope link
       valid_lft forever preferred_lft forever
```
### Debian 10
```
 root@dlp:~# vi /etc/network/interfaces

# This file describes the network interfaces available on your system
# and how to activate them. For more information, see interfaces(5).

source /etc/network/interfaces.d/*

# The loopback network interface
auto lo
iface lo inet loopback

# The primary network interface
allow-hotplug ens2
# comment out
#iface ens2 inet dhcp

# add static settings
iface ens2 inet static
# IP address
address 10.0.0.30
# network address
network 10.0.0.0
# subnet mask
netmask 255.255.255.0
# broadcast address
broadcast 10.0.0.255
# default gateway
gateway 10.0.0.1
# name server
dns-nameservers 10.0.0.10

root@dlp:~# systemctl restart networking ifup@ens2
root@dlp:~# ip addr

1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: ens2: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 52:54:00:87:3e:e4 brd ff:ff:ff:ff:ff:ff
    inet 10.0.0.30/24 brd 10.0.0.255 scope global ens2
       valid_lft forever preferred_lft forever
    inet6 fe80::5054:ff:fe87:3ee4/64 scope link
       valid_lft forever preferred_lft forever
```
### CentOS 8
```
 # if you did not set Host Name during installation, set it like follows

[root@localhost ~]# hostnamectl set-hostname dlp.srv.world
# display devices

[root@localhost ~]# nmcli device

DEVICE  TYPE      STATE      CONNECTION
ens2    ethernet  connected  ens2
lo      loopback  unmanaged  --

# set IPv4 address

[root@localhost ~]# nmcli connection modify ens2 ipv4.addresses 10.0.0.30/24

# set gateway

[root@localhost ~]# nmcli connection modify ens2 ipv4.gateway 10.0.0.1

# set DNS

[root@localhost ~]# nmcli connection modify ens2 ipv4.dns 10.0.0.1

# set manual for static setting (it's [auto] for DHCP)

[root@localhost ~]# nmcli connection modify ens2 ipv4.method manual

# restart the interface to reload settings

[root@localhost ~]# nmcli connection down ens2; nmcli connection up ens2

Connection 'ens2' successfully deactivated (D-Bus active path: /org/freedesktop/NetworkManager/ActiveConnection/1)
Connection successfully activated (D-Bus active path: /org/freedesktop/NetworkManager/ActiveConnection/2)

# show settings

[root@localhost ~]# nmcli device show ens2

GENERAL.DEVICE:                         ens2
GENERAL.TYPE:                           ethernet
GENERAL.HWADDR:                         52:54:00:D0:8F:0B
GENERAL.MTU:                            1500
GENERAL.STATE:                          100 (connected)
GENERAL.CONNECTION:                     ens2
GENERAL.CON-PATH:                       /org/freedesktop/NetworkManager/ActiveC>
WIRED-PROPERTIES.CARRIER:               on
IP4.ADDRESS[1]:                         10.0.0.30/24
IP4.GATEWAY:                            10.0.0.1
IP4.ROUTE[1]:                           dst = 10.0.0.0/24, nh = 0.0.0.0, mt = 1>
IP4.ROUTE[2]:                           dst = 0.0.0.0/0, nh = 10.0.0.1, mt = 100
IP4.DNS[1]:                             10.0.0.10
IP6.ADDRESS[1]:                         fe80::5054:ff:fed0:8f0b/64
IP6.GATEWAY:                            --
IP6.ROUTE[1]:                           dst = fe80::/64, nh = ::, mt = 100
IP6.ROUTE[2]:                           dst = ff00::/8, nh = ::, mt = 256, tabl>

# show state

[root@localhost ~]# ip addr show

1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: ens2: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 52:54:00:d0:8f:0b brd ff:ff:ff:ff:ff:ff
    inet 10.0.0.30/24 brd 10.0.0.255 scope global noprefixroute ens2
       valid_lft forever preferred_lft forever
    inet6 fe80::5054:ff:fed0:8f0b/64 scope link noprefixroute
       valid_lft forever preferred_lft forever
```
## 使用ifconfig设置临时IP地址

`ifconfig eth0 192.168.0.2 netmask 255.255.255.0` 设置IP地址和掩码

`route add default gw 192.168.0.1` 设置网关

使用这个方法设置的IP和网关是临时的，重启后就会失效。

# shell脚本

**变量自增**

    a=1
    let a++
    let a+=1
    ((a++))

**查出并杀死所有wireshark的进程**：

```sh
sudo kill `ps -ef | grep wireshark | grep -v grep | awk '{print $1}'`
```

**杀死所有的ping进程**：

```sh
for i in `ps -x | grep ping | awk '{print $1}'`; do kill $i; done
```

**for语句输出1-15，共15个数字**：

```sh
for i in {1..15}; do echo $i; done
```
  
#### 终端命令行快捷方式
- `ctrl + K`， `crtl + U`， `ctrl + W`， `ctrl + Y`
   ```
   ctrl+K 把光标后面（右面）的命令剪切掉，
   ctrl+U 把光标前面（左面）的命令剪切掉，
   ctrl+W 把光标前面的一个字（以空格分界）剪切掉，
   ctrl+Y 把剪切的内容粘贴回来
   ```
- `ctrl + X + E`
   ```
   开启一个临时文本编辑器，用来编辑需要换行的命令，这样编辑会更直观
   ```

- `alt + . (点)`
   ```
   上一个命令的参数
   ```
   
#### grep的使用

grep（global regular expression print，全局正则表达式输出）

- 在文件中（或多个文件中）查找

      #在/etc/passwd中查找单词"root"
      zyy@Lap:~$ grep root /etc/passwd
      root:x:0:0:root:/root:/bin/bash
      zyy@Lap:~$
      
      #使用 -n 参数在结果中显示行号
      zyy@Lap:~$ grep -n zyy /etc/group
      18:cdrom:x:24:zyy
      19:floppy:x:25:zyy
      62:zyy:x:1000:
      67:wireshark:x:126:zyy
      zyy@Lap:~$ 
      
      #使用^符号输出以某开头的行，使用$符号输出以某结尾的行
      zyy@Lap:~$ grep ^zyy /etc/group
      zyy:x:1000:
      zyy@Lap:~$
      
- 使用 -l 参数列出包含指定模式的文件名

- -v 参数使用反选模式

- 使用 -r 参数递归查找

- 使用 -i 忽略大小写

#### 一些软件

tsocks, putty, certbot（申请SSL证书）	