<!--markdown-->
#### centos7设置全局代理
```commandline
在/etc/profile最后添加
proxy=http://proxy_ip:port
http_proxy=$proxy
https_proxy=$proxy
HTTP_PROXY=$proxy
HTTPS_PROXY=$proxy
export http_proxy https_proxy HTTP_PROXY HTTPS_PROXY

保存后执行立即生效:
sorce /etc/profile 
```

#### 代理自动配置(proxy auto-config) PAC
- 安装genpac
    
    ```sh
    sudo pip install genpac
    pip install --upgrade genpac
    ```

- 调用在线gfwlist列表生成本地autoproxy.pac文件
```commandline
genpac --pac-proxy "SOCKS5 127.0.0.1:7070" --output="autoproxy.pac" 
```

- 在火狐中使用autoproxy.pac

#### 火狐设置代理
在设置页（about:preferences）找到网络代理

如果需要，在（about:config）有更加详细的设置

比如：force-generic-ntlm-v1	