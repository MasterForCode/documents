1. 已有rocketmq服务

2. 新建spring boot项目，引入以下依赖

    ```xml
    <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
    </dependency>

    <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
    </dependency>
    <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
    </dependency>

    <dependency>
            <groupId>org.apache.rocketmq</groupId>
            <artifactId>rocketmq-client</artifactId>
            <version>${rocketmq.version}</version>
    </dependency>   
    ```

3. 配置文件

    ```yml
    apache:
        rocketmq:
            #消费者的配置
            consumer:
            pushConsumer: testConsumer
            #生产者的配置
            producer:
            producerGroup: testProducer
            #Nameserver的地址,这里配置你MQ安装的机器上的IP就好，我这里在本机安装的
            namesrvAddr: 192.168.0.110:9876
    ```

4. 新建生产者Producer.java

    ```java
    package top.soliloquize.rocket_mq;

    import org.apache.commons.lang3.time.StopWatch;
    import org.apache.rocketmq.client.exception.MQBrokerException;
    import org.apache.rocketmq.client.exception.MQClientException;
    import org.apache.rocketmq.client.producer.DefaultMQProducer;
    import org.apache.rocketmq.client.producer.SendResult;
    import org.apache.rocketmq.common.message.Message;
    import org.apache.rocketmq.remoting.common.RemotingHelper;
    import org.apache.rocketmq.remoting.exception.RemotingException;
    import org.springframework.beans.factory.annotation.Value;
    import org.springframework.stereotype.Component;

    import javax.annotation.PostConstruct;
    import java.io.UnsupportedEncodingException;

    /**
    * @author wb
    * @date 2019/7/18
    */
    @Component
    public class Producer {

        /**
        * 生产者的组名
        */
        @Value("${apache.rocketmq.producer.producerGroup}")
        private String producerGroup;

        private DefaultMQProducer producer;
        /**
        * NameServer 地址
        */
        @Value("${apache.rocketmq.namesrvAddr}")
        private String namesrvAddr;

        @PostConstruct
        public void defaultMQProducer() {

            //生产者的组名
            producer = new DefaultMQProducer(producerGroup);
            //指定NameServer地址，多个地址以 ; 隔开
            producer.setNamesrvAddr(namesrvAddr);
            producer.setVipChannelEnabled(false);
            try {
                producer.start();
                System.out.println("-------->:producer启动了");
            } catch (MQClientException e) {
                e.printStackTrace();
            }
        }

        public String send(String topic, String tags, String body) throws InterruptedException, RemotingException, MQClientException, MQBrokerException, UnsupportedEncodingException {
            Message message = new Message(topic, tags, body.getBytes(RemotingHelper.DEFAULT_CHARSET));
            StopWatch stop = new StopWatch();
            stop.start();
            SendResult result = producer.send(message);
            System.out.println("发送响应：MsgId:" + result.getMsgId() + "，发送状态:" + result.getSendStatus());
            stop.stop();
            return "{\"MsgId\":\"" + result.getMsgId() + "\"}";
        }
    }

    ```

5. 新建Consumer.java

    ```java
    package top.soliloquize.rocket_mq;

    import org.apache.rocketmq.client.consumer.DefaultMQPushConsumer;
    import org.apache.rocketmq.client.consumer.listener.ConsumeConcurrentlyStatus;
    import org.apache.rocketmq.client.consumer.listener.MessageListenerConcurrently;
    import org.apache.rocketmq.common.consumer.ConsumeFromWhere;
    import org.apache.rocketmq.common.message.Message;
    import org.springframework.beans.factory.annotation.Value;
    import org.springframework.boot.CommandLineRunner;
    import org.springframework.stereotype.Component;

    /**
    * @author wb
    * @date 2019/7/18
    */
    @Component
    public class Consumer implements CommandLineRunner {

        /**
        * 消费者
        */
        @Value("${apache.rocketmq.consumer.pushConsumer}")
        private String pushConsumer;

        /**
        * NameServer 地址
        */
        @Value("${apache.rocketmq.namesrvAddr}")
        private String namesrvAddr;


        /**
        * 初始化RocketMq的监听信息，渠道信息
        */
        public void messageListener() {

            DefaultMQPushConsumer consumer = new DefaultMQPushConsumer("SpringBootRocketMqGroup");

            consumer.setNamesrvAddr(namesrvAddr);
            try {

                // 订阅PushTopic下Tag为push的消息,都订阅消息
                consumer.subscribe("testTopic", "push");

                // 程序第一次启动从消息队列头获取数据
                consumer.setConsumeFromWhere(ConsumeFromWhere.CONSUME_FROM_FIRST_OFFSET);
                //可以修改每次消费消息的数量，默认设置是每次消费一条
                consumer.setConsumeMessageBatchMaxSize(1);

                //在此监听中消费信息，并返回消费的状态信息
                consumer.registerMessageListener((MessageListenerConcurrently) (msgs, context) -> {

                    // 会把不同的消息分别放置到不同的队列中
                    for (Message msg : msgs) {

                        System.out.println("接收到了消息：" + new String(msg.getBody()));
                    }
                    return ConsumeConcurrentlyStatus.CONSUME_SUCCESS;
                });

                consumer.start();

            } catch (Exception e) {
                e.printStackTrace();
            }
        }

        @Override
        public void run(String... args) throws Exception {
            this.messageListener();
        }
    }
    ```

6. 新建测试类TestController.java

    ```java
    package top.soliloquize.rocket_mq;

    import org.apache.rocketmq.client.exception.MQBrokerException;
    import org.apache.rocketmq.client.exception.MQClientException;
    import org.apache.rocketmq.remoting.exception.RemotingException;
    import org.springframework.beans.factory.annotation.Autowired;
    import org.springframework.web.bind.annotation.RequestMapping;
    import org.springframework.web.bind.annotation.RequestParam;
    import org.springframework.web.bind.annotation.RestController;

    import java.io.UnsupportedEncodingException;

    /**
    * @author wb
    * @date 2019/7/18
    */
    @RestController
    public class TestController {
        @Autowired
        private Producer producer;

        @RequestMapping("/push")
        public String pushMsg(@RequestParam String msg) {
            try {
                return producer.send("testTopic", "push", msg);
            } catch (InterruptedException e) {
                e.printStackTrace();
            } catch (RemotingException e) {
                e.printStackTrace();
            } catch (MQClientException e) {
                e.printStackTrace();
            } catch (MQBrokerException e) {
                e.printStackTrace();
            } catch (UnsupportedEncodingException e) {
                e.printStackTrace();
            }
            return "ERROR";
        }
    }
    ```

7. 在rocketmq服务上新建topic，为上文用到的testTopic

8. 启动后在浏览器上输入http://127.0.0.1:8080/push?msg=hello，页面提示{"MsgId":"AC100AB660C618B4AAC2XXXXXXXX"}就表示消息发送成功啦。