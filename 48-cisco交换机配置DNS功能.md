<!--markdown--># 交换机开启DNS功能，使交换机能够解析域名

```
ip domain lookup  开启域名解析功能
ip name-server {ip-address}  解析域名时用的服务器IP
ip domain name {domain-name}  默认的域名
```
## 说明
`ip domain name {domain-name}`  默认的域名，系统会在名称后面加上这个域名，比如，设置`ip domain name lab`，`ping router1`时如果DNS服务器一直对router1没有回应，系统就会尝试解析router1.lab。	