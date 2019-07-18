1. [安装jdk](https://soliloquize.top/2018/12/10/centos%e4%b8%8b%e5%ae%89%e8%a3%85jdk1-8/)

2. [下载rocketmq](http://rocketmq.apache.org/dowloading/releases/)

3. 解压

    ```bash
    # 如果没有unzip命令需要执行yum install unzip
    [root@localhost ~]# unzip rocketmq-all-4.5.1-bin-release.zip
    ```

4. 修改配置文件

    * 修改broker.conf文件

        ```bash
        [root@localhost ~]# vi rocketmq-all-4.5.1-bin-release/conf/broker.conf 

        # 添加如下内容
        brokerIP1 = 192.168.0.110
        autoCreateTopicEnable = true  # 线上环境应该设为false
        ```

    * 修改runbroker.sh文件中的JVM配置

        ```bash
        [root@localhost ~]# vi rocketmq-all-4.5.1-bin-release/bin/runbroker.sh 
        # 修改如下
        JAVA_OPT="${JAVA_OPT} -server -Xms256m -Xmx256m -Xmn256m"
        JAVA_OPT="${JAVA_OPT} -XX:MaxDirectMemorySize=256m"
        ```

    * 修改runserver.sh文件中JVM配置

        ```bash
        [root@localhost ~]# vi rocketmq-all-4.5.1-bin-release/bin/runserver.sh 
        # 修改内容如下
        AVA_OPT="${JAVA_OPT} -server -Xms128m -Xmx128m -Xmn128m -XX:MetaspaceSize=128m -XX:MaxMetaspaceSize=256m"
        ```

5. 启动(可能需要开启9876、10909、10911端口)

    * 后台启动NameServer
        
        ```bash
        [root@localhost ~]# nohup sh rocketmq-all-4.5.1-bin-release/bin/mqnamesrv -n 192.168.0.110:9876 &
        # 查看日志，看是否启动成功,文件是启动后自动生成的
        tail -f ~/logs/rocketmqlogs/namesrv.log
        ```

    * 后台启动Broker

        ```bash
        [root@localhost ~]# nohup sh rocketmq-all-4.5.1-bin-release/bin/mqbroker -n 192.168.0.110:9876 -c rocketmq-all-4.5.1-bin-release/conf/broker.conf &
        # 查看日志，看是否启动成功
	    tail -f ~/logs/rocketmqlogs/broker.log 
        ```

    * 查看启动

        ```bash
        [root@localhost ~]# jps
        ```

6. 关闭

    ```bash
    [root@localhost ~]# sh rocketmq-all-4.5.1-bin-release/bin/mqshutdown broker
    [root@localhost ~]# sh rocketmq-all-4.5.1-bin-release/bin/mqshutdown namesrv
    ```