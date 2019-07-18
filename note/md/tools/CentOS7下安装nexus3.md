1. 安装jdk1.8+

2. [下载nexus3](https://www.sonatype.com/download-oss-sonatype)

3. * 上传到CentOS中
   
   * 创建文件夹
   ```bash
    [root@localhost ~]# mkdir /usr/local/nexus
   ``` 
   
   * 解压
   ```bash
    [root@localhost ~]# tar -zxvf nexus-3.14.0-04-unix.tar.gz -C /usr/local/nexus/
   ``` 
   
   * 进入nexus的bin目录下指定jdk路径
   ```bash
    [root@localhost ~]# cd /usr/local/nexus/nexus-3.14.0-04/bin/
    [root@localhost bin]# vi nexus
    # 打开注释并填写jdk路径
    INSTALL4J_JAVA_HOME_OVERRIDE=/usr/local/java/jdk1.8.0_191
   ```
   
   * 添加使用用户
   ```bash
    [root@localhost bin]# vi nexus.rc
    # 打开注释添加用户
    run_as_user="root"
    # 默认端口为8081，可在nexus-default.properties 中修改
   ```
   
   * 启动
   ```bash
    [root@localhost bin]# ./nexus run &
   ```
   
   * 开启默认端口的防火墙
   ```bash
    [root@localhost bin]# firewall-cmd --zone=public --add-port=8081/tcp --permanent
    success
    # 重新加载防火墙配置
    [root@localhost bin]# firewall-cmd --reload
   ```
   
   * 登陆
   > 1. 在浏览器中输入地址，http://192.168.0.112:8081(修改成自己的地址)
   > 2. 默认用户名为admin,默认密码为admin123 
   
   * 设置成开机自启动
   ```bash
    [root@localhost bin]# vi /usr/lib/systemd/system/nexus.service

    # 内容如下，修改成自己的安装路径
    [Unit]
    Description=nexus service
    After = network.target

    [Service]
    Type=forking
    LimitNOFILE=65536
    ExecStart=/usr/local/nexus/nexus-3.14.0-04/bin/nexus start
    ExecReload=/usr/local/nexus/nexus-3.14.0-04/bin/nexus restart
    ExecStop=/usr/local/nexus/nexus-3.14.0-04/bin/nexus stop
    Restart=on-abort


    [Install]
    WantedBy=multi-user.target
    # 服务加入开机启动 
    [root@localhost bin]# systemctl enable /usr/lib/systemd/system/nexus.service 

    # 重新加载配置文件
    [root@localhost bin]# systemctl daemon-reload
   ```

   * 自定义数据日志等存储位置(选做)
   ```bash
    [root@localhost bin]# vi bin/nexus.vmoptions
   ```
   * 手动更新索引
   1. [下载nexus-maven-repository-index.gz](http://repo.maven.apache.org/maven2/.index/nexus-maven-repository-index.gz)
   2. [下载 nexus-maven-repository-index.properties](http://repo.maven.apache.org/maven2/.index/nexus-maven-repository-index.properties)
   3. [下载 indexer-cli-6.0.0.jar](http://central.maven.org/maven2/org/apache/maven/indexer/indexer-cli/6.0.0/indexer-cli-6.0.0.jar)
   ```bash
    # 新建一个临时目录
    [root@localhost bin]# mkdir temp & cd temp
    # 将下载的三个文件上传到该目录下,然后执行
    [root@localhost temp]# java -jar indexer-cli-6.0.0.jar -u nexus-maven-repository-index.gz -d ./indexer
    # 将indexer下生成的文件复制到指定目录
    [root@localhost temp]# cp ./indexer/* /usr/local/nexus/nexus-3.14.0-04/sonatype-work/nexus3/indexer/central-ctx
   ```
   * 使用maven
   1. 配置全局的setting.xml
   ```xml
    <!-- 配置server -->
    <servers>
        <!-- 正式版 -->
        <server>
        <id>releases</id>
        <username>admin</username>
        <password>admin123</password>
        </server>
        <!-- 开发版 -->
        <server>
        <id>snapshots</id>
        <username>admin</username>
        <password>admin123</password>
        </server>
    </servers>
    <!-- 配置repositories，修改成自己的地址 -->
    <repositories>
        <repository>
            <id>maven-central</id>
            <name>maven-central</name>
            <url>http://192.168.0.102:8081/repository/maven-central/</url>
            <snapshots>
                <enabled>true</enabled>
                <!-- 开发版本 拉取周期：每次都拉取 -->
                <updatePolicy>always</updatePolicy>
            </snapshots>
            <releases>
                <enabled>true</enabled>
            </releases>
        </repository>
    </repositories>
   ```
   2. 配置发布，在pom.xml中添加如下配置
   ```xml
    <distributionManagement>
        <repository>
            <!-- id需要与setting.xml server中配置的id一致,地址修改成自己的 -->
            <id>releases</id>
            <name>maven-releases</name>
            <url>http://192.168.0.112:8081/repository/maven-releases/</url>
        </repository>
        <snapshotRepository>
            <id>snapshots</id>
            <name>maven-snapshots</name>
            <url>http://192.168.0.112:8081/repository/maven-snapshots/</url>
        </snapshotRepository>
    </distributionManagement>
   ```
   3. maven-releases默认不允许提交，在http://192.168.0.112:8081/#admin/repository/repositories:maven-releases下将Deployment policy修改成Allow redeploy(ip地址修改成自己的)

    * 使用npm
        1. 点击最左侧菜单Repositories, 然后点击Create repository按钮
        2. 选择npm(proxy),Name输入:npm-proxy, remote storage 填写 https://registry.npm.taobao.org 用于将包情求代理到该地址，点击最下方的Create repository按钮确定
        3. 再次点击Create repository按钮，增加npm(hossted)Name输入:npm-breeze用于存放自己的私有包，点击最下方的Create repository按钮确定
        4. 再次点击Create repository按钮，增加npm(group)Name输入: npm-all,下面Member repositories里选择之前添加的2个,移动到右边，点击最下方的Create repository按钮确定
        5. npm config set registry http://192.168.0.112:8081:8081/repository/npm-all/这里的url在仓库npm-all右边有获取url
