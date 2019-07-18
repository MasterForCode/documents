> 下载MySql的rpm文件
``` bash
# yum仓库下载MySql
[root@localhost ~]# yum localinstall https://repo.mysql.com//mysql80-community-release-el7-1.noarch.rpm
```

> 安装MySql
``` bash
# yum安装MySQL
[root@localhost ~]# yum install mysql-community-server
```

> 启动MySql
``` bash
[root@localhost ~]# service mysqld start
```

> 查看MySql服务运行状态
``` bash
[root@localhost ~]# service mysqld status
```

> 查看初始密码
``` bash
[root@localhost ~]# grep 'temporary password' /var/log/mysqld.log
```

> 修改密码
``` sql
<!-- 先登录-- >
mysql> ALERT USER 'root'@'localhost' IDENTIFIED BY 'wangbin_123';
```