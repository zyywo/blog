<!--markdown-->转载自：[win7，Ubuntu 12.04 双系统修改启动项顺序三方法](http://www.cnblogs.com/Duahanlang/p/3811465.html)

本文所涉及的方法都是在Ubuntu的安装时将引导加载程序grub安装在了整个硬盘（即MBR内），即开机以grub引导。

### 方法1
在Ubuntu终端下输入：
```
sudo mv /etc/grub.d/30_os-prober /etc/grub.d/08_os-prober
sudo update-grub
```
该命令是将etc文件夹下的grub.d文件夹下的30_os-prober文件改名为08_os-prober。（08可以改为06~09都可以）。 Ubuntu的启动项相关文件名为“10_....”这样就可以将win7的启动项放在Ubuntu前面，即启动项列表的第一个。由于引导程序默认启动第 一个启动项，所以这样就可以先启动win7了。注意修改完后更新一下grub才能生效（即命令sudo update-grub）。

### 方法2
在Ubuntu终端下输入：
```
sudo gedit /etc/default/grub
sudo update-grub
```
在打开的文本中修改“GRUB_DEFAULT=0”这一项。比如win7在启动项列表中为第5项，则将0改为4。就是win7在启动项列表中的项数减1。
这里还可以修改该在启动项列表等待的时间，即修改“GRUB_TIMEOUT=所要等待的秒数”，-1表示不倒计时。
修改完后按`Ctrl`+`X`，会提示是否保存，输入`Y`，提示保存的文件名，还是原来的grub文件，所以直接回车确定。
`sudo update-grub`，更新一下grub。

### 方法3
在Ubuntu终端下输入：
```
sudo gedit /boot/grub/grub.cfg
```
这个方法是编辑 /boot/grub/grub.cfg文件，

在打开的文本中修改set default="0” 这一项。比如win7在启动项列表中为第5项，则将0改为4。就是win7在启动项列表中的项数减1。

这里还可以修改该在启动项列表等待的时间，即修改“set timeout=10”，-1表示不倒计时。
修改完后按`Ctrl`+`s`，保存后直接关闭。	