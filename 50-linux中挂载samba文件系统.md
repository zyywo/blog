<!--markdown--> 1. 安装cifs依赖：

    `sudo apt install cifs-utils`

 2. 创建挂载点文件夹

    `mkdir /home/zyy/openwrt_share`

 3. 编辑`/etc/fstab`，添加挂载点

    `//openwrt.lan/share\	/home/zyy/openwrt_share\	  cifs guest,vers=1.0,uid=zyy,gid=zyy  0\	0`
  
    `cifs`是指挂载的文件系统；

    `guest`选项是匿名登陆；

    `vers=1`选项是使用的samba版本号；

    `uid=zyy,gid=zyy`选项是指定文件系统的用户ID和组ID；

 4. 挂载文件系统

    `sudo mount -a`

卸载文件系统可以使用下面的方法：

`sudo umount /home/zyy/openwrt_share `

参考：

[Auto-mount Samba / CIFS shares via fstab on Linux](http://timlehr.com/auto-mount-samba-cifs-shares-via-fstab-on-linux/)	