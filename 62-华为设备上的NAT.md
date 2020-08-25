<!--markdown-->## NAT
### NAT类型
**静态一对一映射**
**基于地址池的一对一映射（no-pat）**
  从地址池取出一个地址供私有IP访问公网，实际上仍然是一对一的映射。只做地址转换，不做端口转换
**基于地址池的多对一映射（NPAT）**
  地址和端口都做转换。
**easyIP**
  直接使用出口路由器接口的IP地址做NAT，不需要额外购买公网IP。
**nat server**
  将服务器映射到外网供外网访问。

### NAT在路由器上的部署
**静态一对一映射**
  [对外接口]ip address 200.1.1.1 24
  [对外接口]nat static global 200.1.1.100 inside 192.168.1.1 netmask 32
**动态地址池的一对一映射**
  #定义nat地址池
  [系统视图]nat address-group 1 200.1.1.100 200.1.1.200
  #定义acl，用于匹配允许nat的内网地址段
  [系统视图]acl 2000
  [ACL视图]rule 5 permit source 192.168.1.0 0.0.0.255
  #接口下配置nat
  [对外接口]nat outbound 2000 address-group 1 no-pat
  **EasyIP**
  [系统视图]acl 2000
  [acl视图]rule 5 permit source 192.168.1.0 0.0.0.255
  [对外接口g0/0/1]nat outbound 2000 interface g0/0/1
  **nat server**
  [对外接口]nat server protocol tcp global 200.1.1.100 8080 inside 192.168.1.100:80
### NAT在防火墙上的部署
略略略...	