1. 安装gcc和ruby

    ```bash
    [root@localhost ~]# yum -y install centos-release-scl
    [root@localhost ~]# yum -y install devtoolset-9-gcc devtoolset-9-gcc-c++ devtoolset-9-binutils
    [root@localhost ~]# scl enable devtoolset-9 bash
    [root@localhost ~]# echo "source /opt/rh/devtoolset-9/enable" >> /etc/profile
    [root@localhost ~]# gcc -v
    ```

2. 到[官网](http://download.redis.io/releases/)下载最新版本

3. 将文件解压并复制到创建的目录中

    ```bash
    [root@localhost ~]# tar -zxvf redis-6.0.5.tar.gz 
    [root@localhost ~]# mv redis-6.0.5 /usr/local/redis
    ```

4. 编译安装

    ```bash
    [root@localhost ~]# cd /usr/local/redis/
    [root@localhost redis]#  make && make install
    ```

5. 创建目录

    ```bash
    [root@localhost redis]#  mkdir -p logs
    [root@localhost redis]#  mkdir -p redis-6379
    ```

6. 配置

    ```bash
    [root@localhost redis]# cd  /usr/local/redis/redis-6379 && vi redis.conf
    ```

    ```bash
    # 端口6379
    port 6379  
    # 默认ip为127.0.0.1，需要改为其他节点机器可访问的ip，否则创建集群时无法访问对应的端口，无法创建集群
    bind 192.168.0.122 
    # redis后台运行
    daemonize yes   
    # 请求超时，默认15秒，可自行设置
    cluster-node-timeout 8000   
    # 开启aof持久化模式，每次写操作请求都追加到appendonly.aof文件中
    appendonly yes 
    # 每次有写操作的时候都同步 
    appendfsync always  
    # redis服务日志
    logfile "/usr/local/redis/logs/redis.log" 
    # pidfile文件对应6379
    pidfile /var/run/redis_6379.pid  
    # 禁用危险命令
    rename-command FLUSHALL ""
    rename-command FLUSHDB ""
    rename-command KEYS ""
    ```

7. 启动

    ```bash
    [root@localhost redis-6379]# redis-server /usr/local/redis/redis-6379/redis.conf
    ```

8. 连接

    ```bash
    [root@localhost redis-6379]# redis-cli -h 192.168.0.122 -p 6379
    ```
