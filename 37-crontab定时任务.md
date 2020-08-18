<!--markdown-->
**cron引用的脚本中，建议使用绝对路径执行命令，不然可能会找不到命令。**

使用`crontab -l`查看用户的定时任务

使用`crontab -e`编辑用户的定时任务

用户所建立的crontab文件中，每一行都代表一项任务，每行的每个字段代表一项设置，它的格式共分为六个字段，前五段是时间设定段，第六段是要执行的命令段，格式如下：

![crontab](https://raw.githubusercontent.com/zyywo/zyywo.pic/master/crontab-2.png "crontab的格式")


修改crontab文件后，不会立即生效，需要过段时间(2分钟内)才会重新加载文件，日志内可以看到EDIT和RELOAD的时间

cron的日志路径：`/var/log/cron`

**查看关于cron的日志**
journalctl  -u cron

表示时间的特殊值：

- `@reboot`：表示在机器重启后执行 

在任务前加`#`符号表示注释。
	