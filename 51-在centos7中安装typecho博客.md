<!--markdown--># 关闭SELINUX
    
编辑`/etc/selinux/config`文件，设置`SELINUX=disabled`，重启后会永久关闭selinux。

`setenforce 0`：临时关闭selinux，重启后会再开启

# 安装http服务器
   
    yum install httpd

   **开启httpd服务器**    
    systemctl enable httpd
    systemctl start httpd

# 安装并设置mysql数据库
    yum install mariadb-server mariadb

编辑`/etc/my.cnf`，add follows within [mysqld] section
   
    [mysqld]
    character-set-server=utf8
    bind-address=127.0.0.1

**开启mysql服务器**
   
    systemctl enable mariadb
    systemctl start mariadb

**Initial Settings for MariaDB**

    mysql_secure_installation

**使用root用户登陆mysql数据库，创建typecho用的数据库**

    mysql -u root -p
    
    CREATE DATABASE typecho;  #创建typecho用的数据库

    SHOW DATABASES;  #验证结果

# 安装php和用到的组件

    yum install php php-mbstring php-mysql php-gd curl curl-devel

# 安装typecho

   **解压typecho压缩包**
    
    tar -xf 1.1-17.10.30-release.tar.gz

   **把typecho安装文件夹放到http根目录下**
    
    mv build/* /var/www/html/

   **修改html目录与了目录的所有者**

    chown -R apache:apache /var/www/html

至此结束，访问服务器的80端口就可以了。

[typecho官网](http://typecho.org/)

[Typecho主题 - Initial](https://github.com/jielive/initial)	