<!--markdown-->##### 使用x11vnc建立VNC服务器

###### 安装x11vnc
>apt install x11vnc

###### 使用x11vnc
> x11vnc -ncache 10 -forever -shared -bg -rfbport 5900 -rfbauth /root/.vnc/passwd -auth /var/run/lightdm/root/:0 -display :0

###### 创建系统服务x11vnc.service
>vim /lib/systemd/system/x11vnc.service

```
[Unit]
Description=start x11vnc at startup
#注释
After=multi-user.target

[Service]
#WorkingDirectory=
ExecStart=/usr/bin/x11vnc -ncache 10 -forever -shared -bg -rfbport 5900 -rfbauth /root/.vnc/passwd -auth /var/run/lightdm/root/:0 -display :0
Restart=on-failure
#or always

[Install]
WantedBy=multi-user.target
```

###### 启用服务x11vnc.service
```
systemctl enable x11vnc.service
systemctl daemon-reload
systemctl start x11vnc.service
```

###### 检查服务状态x11vnc.service
{%highlight shell linenos %}
systemctl status x11vnc.service
journalctl [-f] -u x11vnc.service
{%endhighlight%}

##### 使用noVNC建立基于web的vnc客户端
###### 下载noVNC并解压 https://kanaka.github.io/noVNC/
###### 使用noVNC连接到VNC服务器 ./noVNC/utils/launch.sh --vnc localhost:5900
###### 通过noVNC主机的IP登陆noVNC <noVNC_host>:6080/vnc.html

##### 说明
x11vnc参数

>x11vnc -noxrecord -forever -rfbport 5900 -rfbauth /root/.vnc/passwd -auth /var/lib/lightdm/.Xauthority -display :0

-auth可用值
```
/var/run/lightdm/root/:0
/var/lib/lightdm/.Xauthority
```

	