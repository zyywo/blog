<!--markdown-->查看 Linux 发行版名称和版本号

* ### lsb_release
  > LSB（Linux Standard Base, Linux 标准库）能够打印发行版的具体信息，包括发行版名称、版本号、代号等。
    
    ```commandline
    # lsb_release -a
    No LSB modules are available.
    Distributor ID: Ubuntu
    Description: Ubuntu 16.04.3 LTS
    Release: 16.04
    Codename: xenial
    ```
  
* ### /etc/*-release 文件
  >release 文件通常被视为操作系统的标识。在 /etc 目录下放置了很多记录着发行版各种信息的文件，每个发行版都各自有一套这样记录着相关信息的文件。
  
    ```commandline
    # cat /etc/issue
    # cat /etc/issue.net
    # cat /etc/os-release
    ```
    
* ### uname 命令
  >uname（unix name 的意思） 是一个打印系统信息的工具，包括内核名称、版本号、系统详细信息以及所运行的操作系统等等。

* ### /proc/version 文件
  >这个文件记录了 Linux 内核的版本、用于编译内核的 gcc 的版本、内核编译的时间，以及内核编译者的用户名。
  
* ### dmesg 命令
  >dmesg（display message, 展示信息 或 driver message, 驱动程序信息）是大多数类 Unix 操作系统上的一个命令，用于打印内核的消息缓冲区的信息。
  
    ```commandline
    # dmesg | grep "Linux"
    [ 0.000000] Linux version 4.12.14-300.fc26.x86_64 ([email protected]) (gcc version 7.2.1 20170915 (Red Hat 7.2.1-2) (GCC) ) #1 SMP Wed Sep 20 16:28:07 UTC 2017
    [ 0.001000] SELinux: Initializing.
    [ 0.001000] SELinux: Starting in permissive mode
    [ 0.470288] SELinux: Registering netfilter hooks
    [ 0.616351] Linux agpgart interface v0.103
    [ 0.630063] usb usb1: Manufacturer: Linux 4.12.14-300.fc26.x86_64 ehci_hcd
    [ 0.688949] usb usb2: Manufacturer: Linux 4.12.14-300.fc26.x86_64 ohci_hcd
    [ 2.564554] SELinux: Disabled at runtime.
    [ 2.564584] SELinux: Unregistering netfilter hooks
    ```
    
参考文档：[https://linux.cn/article-9586-1.html](https://linux.cn/article-9586-1.html)	