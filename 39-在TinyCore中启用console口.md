<!--markdown-->**tinycore的版本为10.1，文中的路径可能有误**

1. 编辑extlinux.conf文件，路径: `/mnt/sda1/tce/boot/extlinux/extlinux.conf`
    ```
        serial 0 115200
        display boot.msg
        default microcore
        label microcore
                kernel /boot/vmlinuz
                initrd /boot/core.gz
                append loglevel=3 console=ttyS0,115200 console=tty0
    ```
2. 解压core.gz，路径：`/mnt/sda1/tce/boot/core.gz`
    ```
        mkdir file_system
        cd file_system
        zcat ../boot/tinycore.gz | sudo cpio -i -H newc -d
    ```
3. 修改inittab文件，路径：`file_system/etc/inittab`
    ```
        # /etc/inittab: init configuration for busybox init.
        # Boot-time system configuration/initialization script.
        #
        ::sysinit:/etc/init.d/rcS
        
        # /sbin/getty respawn shell invocations for selected ttys.
        tty1::respawn:/sbin/getty -nl /sbin/autologin 38400 tty1
        #tty2::respawn:/sbin/getty 38400 tty2
        #tty3::respawn:/sbin/getty 38400 tty3
        #tty4::askfirst:/sbin/getty 38400 tty4
        #tty5::askfirst:/sbin/getty 38400 tty5
        #tty6::askfirst:/sbin/getty 38400 tty6
        
        ttyS0::respawn:/sbin/getty 38400 ttyS0 xterm
        
        # Stuff to do when restarting the init
        # process, or before rebooting.
        ::restart:/etc/init.d/rc.shutdown
        ::restart:/sbin/init
        ::ctrlaltdel:/sbin/reboot
        ::shutdown:/etc/init.d/rc.shutdown
    ```
4. 修改securetty文件，路径：`file_system/etc/securetty`
    ```
        # /etc/securetty: List of terminals on which root is allowed to login.
        #
        console
        
        # For people with serial port consoles
        ttyS0
        
        # Standard consoles
        tty1
        tty2
        tty3
        tty4
        tty5
        tty6
        tty7
    ```
5. 修改issue文件（可选），路径：`file_system/etc/issue`，这个文件中的内容会在等待输入用户名时出现
    ```
        Core Linux
        
        username 'gns3', password 'gns3'
        Run filetool.sh -b if you want to save your changes
    ```
6. 重新压缩并替换core.gz
    ```
        cd file_system
        find | sudo cpio -o -H newc | gzip -2 > ../boot/tinycore.gz
    ```
参考:
    
[Serial Port console with syslinux question? ](http://forum.tinycorelinux.net/index.php/topic,11088.msg58088.html#msg58088)

[Login at /dev/ttyS0](http://forum.tinycorelinux.net/index.php/topic,11088.msg58088.html#msg58088)

[Need a Root Password to Change Files](http://forum.tinycorelinux.net/index.php?topic=9204.0)	