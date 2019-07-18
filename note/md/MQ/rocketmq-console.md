1. 下载

    ```bash
    $ git clone -b release-rocketmq-console-1.0.0 https://github.com/apache/rocketmq-externals.git
    ```

2. 修改rocketmq-externals/rocketmq-console/src/main/resources/application.properties 

    ```properties
    #管理后台访问上下文路径，默认为空，如果填写，一定要前面加“/”，后面不要加，否则启动报错
    server.contextPath=/rocketmq
    #访问端口
    server.port=8080
    #spring.application.index=true
    spring.application.name=rocketmq-console
    spring.http.encoding.charset=UTF-8
    spring.http.encoding.enabled=true
    spring.http.encoding.force=true
    #logback配置文件路径
    logging.config=classpath:logback.xml
    #if this value is empty,use env value rocketmq.config.namesrvAddr  NAMESRV_ADDR | now, you can set it in ops page.default localhost:9876
    #Name Server地址，修改成你自己的服务地址
    rocketmq.config.namesrvAddr=192.168.0.110:9876
    #if you use rocketmq version < 3.5.8, rocketmq.config.isVIPChannel should be false.default true
    rocketmq.config.isVIPChannel=
    #rocketmq-console's data path:dashboard/monitor
    rocketmq.config.dataPath=/tmp/rocketmq-console/data
    #set it false if you don't want use dashboard.default true
    rocketmq.config.enableDashBoardCollect=true
    ```

3. 使用idea中maven下的plugins下的jar打包

4. 运行

    ```cmd
    java -jar -jar target/rocketmq-console-ng-1.0.0.jar --rocketmq.config.namesrvAddr='192.168.0.110:9876'
    ```

5. 访问

    ```http
    http://localhost:8080
    ```