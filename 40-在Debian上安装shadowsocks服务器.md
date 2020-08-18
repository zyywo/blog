<!--markdown--># 安装shadowsocks-libv
略
# 把shadowsocks添加为systemd服务
1. 编辑`/etc/systemd/system/shadowsocks.service`文件

    ```
[Unit]
Description=Shadowsocks server
After=network.target

[Service]
TimeoutStartSec=120
ExecStart=/usr/local/bin/ss-server -c /etc/shadowsocks.json

[Install]
WantedBy=multi-user.target
    ```

# 编辑shadowsocks的配置
1. 假设配置文件的路径为`/etc/shadowsocks.json`

    ```
{
  "server": "0.0.0.0",
  "server_port": 8388,
  "password": "PASSWORD",
  "method": "chacha20-ietf-poly1305"
}
    ```

# 启用shadowsocks服务

```
systemctl enable shadowsockes
systemctl start shadowsocks
```	