<!--markdown-->Install GNS3 server:
sudo dpkg --add-architecture i386
sudo apt update
sudo apt install libc6:i386
sudo apt install libssl-dev:i386
sudo ln -s /lib/i386-linux-gnu/libcrypto.so.1.1 /lib/i386-linux-gnu/libcrypto.so.4

sudo apt install python3-pip qemu qemu-kvm qemu-utils libvirt-clients libvirt-daemon-system virtinst curl gnupg2
或
sudo apt install python3-pip qemu qemu-kvm qemu-utils libvirt-clients libvirt-daemon-system virtinst apt-transport-https ca-certificates curl gnupg2 software-properties-common

sudo pip3 install gns3-server
sudo cp init/gns3.service.systemd /etc/systemd/system/gns3.service
sudo systemctl enable gns3

安装docker-ce,设置国内的Docker hub源:
\	参考1：https://docs.docker.com/install/linux/docker-ce/debian/#install-docker-ce-1
\	参考2：https://yq.aliyun.com/articles/110806
\	参考3：http://mirrors.ustc.edu.cn/help/dockerhub.html
\	curl -fsSL https://get.docker.com | bash -s docker --mirror Aliyun
安装vpc,ubridge，dynamips：
\	参考1：https://docs.gns3.com/1QXVIihk7dsOL7Xr7Bmz4zRzTsJ02wklfImGuHwTlaA4/index.html#h.l56h7z55hmxm
\	Add the following lines to your /etc/apt/sources.list:
\	deb http://ppa.launchpad.net/gns3/ppa/ubuntu bionic main
\	deb-src http://ppa.launchpad.net/gns3/ppa/ubuntu bionic main 
\	Get the GPG key:
\	sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys F88F6D313016330404F710FC9A2FD067A2E3EF7B
\	Refresh your metadata, and ONLY install the following two packages
\	sudo apt update
\	sudo apt install dynamips ubridge vpcs
\	然后为了安全把上面的ppa源注释掉
Add your user to the following groups:
kvm libvirt docker ubridge wireshark

如果安装python的psutil包出错：`Python.h：没有那个文件或目录`，可以尝试安装python3-dev解决：`sudo apt install python3-dev`。
	