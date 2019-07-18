> [下载](https://dev.mysql.com/downloads/mysql/)

> 卸载旧MySql服务

**删除前做好备份**
* 停止服务，以管理员身份打开cmd，运行net stop [服务名]
* 通过卸载工具卸载MySQL（安装版本需执行该操作，免安装版本不需要）
* 卸载服务，运行sc delete MySQL
* 清理注册表
* 1.windows + r
    2.输入regedit
    3.删除\HKEY_LOCAL_MACHINE\SYSTEM\ControlSet001\services\eventlog\Application\MySQL下MySQL整个目录
* 删除旧文件
> 解压下载文件到指定目录

* 在解压目录下创建my.ini文件
* 添加配置信息

``` bash
[mysqld]
# 设置3306端口
port=3306
# 设置mysql的安装目录
basedir=D:\\soft\\mysql-8.0.13-winx64   # 切记此处一定要用双斜杠\\，单斜杠我这里会出错。
# 设置mysql数据库的数据的存放目录
datadir=D:\\soft\\mysql-8.0.13-winx64\\data   # 此处同上
# 允许最大连接数
max_connections=200
# 允许连接失败的次数。这是为了防止有人从该主机试图攻击数据库系统
max_connect_errors=10
# 服务端使用的字符集默认为UTF8
character-set-server=utf8
# 创建新表时将使用的默认存储引擎
default-storage-engine=INNODB
# 默认使用“mysql_native_password”插件认证
default_authentication_plugin=mysql_native_password
[mysql]
# 设置mysql客户端默认字符集
default-character-set=utf8
[client]
# 设置mysql客户端连接服务端时默认使用的端口
port=3306
default-character-set=utf8
```

> 初始化
* 以管理员身份运行cmd命令，并将路径换到mysql的bin目录下
* 初始化数据库，运行命令mysqld --initialize --console，记下控制台输出数据，该数据中有初始化密码
* 安装服务，运行命令mysqld --install [服务名] 其中服务名可以不写，默认是mysql 
* 启动MySQL，运行命令net start [服务名]该服务名要和安装的服务名相同
* 登陆MySQL，运行命令mysql -uroot -p回车后输入初始化时获得的密码
* 修改密码，运行alter user 'root'@'localhost' IDENTIFIED BY 'wangBin_123';密码中最好有大小写英文、数字、特殊字符否则又可能在某些版本出错
* 修改密码加密方式，运行alter user 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'wangBin_123';
* 修改时区，运行set global time_zone='+8:00';