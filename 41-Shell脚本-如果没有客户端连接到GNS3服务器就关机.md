<!--markdown-->**因为cron不会读取系统中PATH环境变量，所以cron引用的脚本中，建议使用绝对路径执行命令，不然可能会找不到命令。**

使用shell脚本配合crontab，每30分钟一次是否有客户端连接到检查GNS3服务，如果有连接则退出脚本，如果没有链接则关机。

1. 编写脚本/etc/gns3r.sh

```
#!/usr/bin/bash
LOCAL_IP=$(/usr/sbin/ifconfig |grep 'inet 192.168.222.' |awk '{print $2}')
DATE=$(date -I)

# if gns3 client disconnected time more than 1200s(20m) return 1, else return 0
function gns3_client_disconnected {
    LIMIT_TIME=$(expr 20 \\* 60)
    NEW_CLIENT=$(cat /var/log/gns3/gns3.log |grep $DATE |grep "New client has connected to controller WebSocket"|wc -l)
    DISC_CLIENT=$(cat /var/log/gns3/gns3.log |grep $DATE |grep "Client has disconnected from controller WebSocket"|wc -l)
    if [ $NEW_CLIENT -eq $DISC_CLIENT ];then
        #Disconnected time
        DIS_TIME=$(date -d $(cat /var/log/gns3/gns3.log |grep "Client has disconnected from controller WebSocket" |tail -n 1 |awk '{print $2}') +%s)
        NOW=$(date +%s)

        if [ $(($NOW - $DIS_TIME)) -ge $LIMIT_TIME ];then
            echo "GNS3 clients disconnected time more than $LIMIT_TIME seconds"
            return 1
        else
            echo "GNS3 clients disconnected time less than $LIMIT_TIME seconds"
        fi
    else
        echo "client has connected"
    fi
    return 0
}

# if has ssh connected return 1, else return 0
function ssh_connected {
    if [ -n "$(netstat -nt4 |grep "$LOCAL_IP:22")" ];then
        echo "ssh clinets online"
        return 1
    else
        echo "no ssh clients online"
        return 0
    fi
}

# uptime in minutes
if [ -n "$(/usr/bin/uptime -p |grep hour)" ];then
    UPTIME=$(expr $(/usr/bin/uptime -p |awk '{print $2}') \\* 60 + $(uptime -p |awk '{print $(NF-1)}'))
else
    UPTIME=$(expr $(/usr/bin/uptime -p |awk '{print $(NF-1)}'))
fi

#
# main script
#
if [ $UPTIME -ge 20 ];then
    ssh_connected
    if [ $? -eq 0 ];then
        gns3_client_disconnected
        if [ $? -eq 1 ];then
            echo "could shutdown"
            /usr/sbin/shutdown
        else
            echo "cannt shutdown"
        fi
    else
        echo "cannt shutdown"
    fi
else
    echo "uptime less than 20 minutes, cannt shutdown"
fi

```
2. 把gns3r.sh添加到root用户的crontab中
    ```
    crontab -u root -e
    */30 * * * * /etc/gns3r.sh
    ```	