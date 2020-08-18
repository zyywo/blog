<!--markdown-->用户生成密钥对，把公钥放在服务器上用户的认证文件中，用户登陆时使用私钥认证。

# 得到密钥对
生成openssh格式的公钥和私钥：可以使用ssh-keygen和putty-gen

# Debian系服务器端设置
编辑/etc/ssh/sshd_config
>设置 PubkeyAuthentication yes

重启ssh服务 systemctl restart ssh

在用户的家目录下创建.ssh文件夹（注意所有者是用户，权限是755）

把公钥安装在.ssh文件夹内的authorized_keys中（注意所有者是用户，权限是644）。

安装方法：cat 公钥 >authorized_keys 或 公钥重命名为authorized_keys 或 ssh-copy-id hostname

最好禁用root使用密码登陆

# 小米路由器的Dropbear config

[openwrt dropbear configuration](https://wiki.openwrt.org/doc/uci/dropbear)

[Setup authorized_keys](https://wiki.openwrt.org/oldwiki/dropbearpublickeyauthenticationhowto)

    vi /etc/config/dropbear
    
    config dropbear
        option PasswordAuth 'off'
        option RootPasswordAuth 'off'
        option Port '22'
        
公钥存放目录：

    cd /etc/dropbear/authorized_keys
    chmod 0600 authorized_keys	