<!--markdown-->**因为cron不会读取系统中PATH环境变量，所以cron引用的脚本中，建议使用绝对路径执行命令，不然可能会找不到命令。**

使用shell脚本配合crontab，每30分钟一次检查GNS3服务是否与外部有通信，如果有通信则退出，如果没有通信则关机。

1. 编写脚本/etc/gns3r.sh

        ```
        #如果GNS3服务没有与外部通信，就关闭主机。
        if /usr/bin/uptime -p | grep hour
        then
                UPTIME=$(expr $(/usr/bin/uptime -p | awk '{print $2}') \\* 60 + $(uptime -p | awk '{print $(NF-1)}'))
        else
                UPTIME=$(/usr/bin/uptime -p | awk '{print $(NF-1)}')
        fi

        echo "--------------------" >> /home/gns3/uptime
        echo $UPTIME >> /home/gns3/uptime

        if [ $UPTIME -ge 20 ]
        then
                echo "grater than 20 minutes" >> /home/gns3/uptime
                LOCAL_IP=$(/usr/sbin/ifconfig | grep 'inet '| grep -v '127.0.0.1' | grep -v '172.17.0.1' | awk '{print $2}')

                if netstat -nt4 | grep $LOCAL_IP:3080
                then
                        echo "GNS3 Clients online, No shutdown"
                else
                        if netstat -nt4 | grep $LOCAL_IP:22
                        then
                                echo "SSH Clients online, No shutdown"
                        else
                                /usr/sbin/shutdown now
                        fi
                fi
        fi
        ```
2. 把gns3r.sh添加到root用户的crontab中
    ```
    crontab -u root -e
    */30 * * * * /etc/gns3r.sh
    ```	