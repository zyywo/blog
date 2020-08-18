<!--markdown-->
这个脚本通过wol启动内网的主机，每次运行都会把路由器上的MAC地址表更新到mac_list文件，如果MAC地址已经存在则跳过这个MAC。

mac_list中每行是一个MAC，MAC后可用逗号隔开，添加一个注释，如`4c:72:b9:e3:18:a1,Hasee_Laptop`，也可不添加注释。

```
#!/bin/sh
macs_file="/data/mac_list"

if [ -e $macs_file ]
then
    #echo "OK, find $macs_file"
    break
else
    #echo "Not find $macs_file, will create it"
    touch $macs_file
fi

macs=`cat $macs_file`


＃这个函数用来检查一个MAC是否已经存在mac_list文件中
findx(){
    c=0
    for mac in $macs
    do
        if [ $1 = ${mac::17} ]
        then
            let c++
        fi
    done

    return $c
}

＃遍历路由器的MAC表项，如果是新的MAC就添加到mac_list文件
for arp in `cat /proc/net/arp |grep br-lan | awk '{print $4}'`
do
    findx $arp
    if [ $? -eq 0 ]
    then
        echo $arp >> $macs_file
    fi
done

i=0
for mac in $macs
do
    let i++
    echo "$i.  $mac"
done

read -p "Select: " ch
if [ $ch -gt $i ]
then
    echo "Bad select!"
else
    target=`head -n $ch $macs_file | tail -1 | awk -F ',' '{print $1}'`
fi

＃如果用255.255.255.255作为目的地，路由器可能不会发送数据包，所以最好用接口的广播地址
bcast_addr=`ifconfig br-lan |grep Bcast: |awk '{print $3}' |awk -F ':' '{print $2}'`
/data/usr/bin/wol $target -h $bcast_addr
```	