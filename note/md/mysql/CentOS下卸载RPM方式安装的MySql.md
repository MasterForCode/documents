> 检查安装的MySql组件
``` bash
[root@localhost ~]# rpm -qa | grep -i mysql
# 安装了以下组件
mysql-community-libs-8.0.13-1.el7.x86_64
mysql-community-libs-compat-8.0.13-1.el7.x86_64
mysql80-community-release-el7-1.noarch
mysql-community-common-8.0.13-1.el7.x86_64
mysql-community-client-8.0.13-1.el7.x86_64
mysql-community-server-8.0.13-1.el7.x86_64
```

> 关闭MySql服务
``` bash
[root@localhost ~]# service mysqld stop
```

> 卸载MySql各类组件
``` bash
[root@localhost ~]# yum remove mysql-community-server-8.0.13-1.el7.x86_64
[root@localhost ~]# yum remove mysql-community-libs-8.0.13-1.el7.x86_64
[root@localhost ~]# yum remove mysql80-community-release-el7-1.noarch
[root@localhost ~]# yum remove mysql-community-common-8.0.13-1.el7.x86_64
```

> 收集MySql对应的文件夹信息并删除
``` bash
[root@localhost ~]# find / -name mysql
[root@localhost ~]# rm -rf  /var/lib/mysql/
```

> 删除用户及用户组
``` bash
[root@localhost ~]# userdel mysql
[root@localhost ~]# groupdel mysql
```