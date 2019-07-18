# 系统架构

![](https://upload-images.jianshu.io/upload_images/4325076-2b07397a5b80633c.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000/format/webp)

* NameServer提供服务发现和路由。 每个 NameServer 记录完整的路由信息，提供等效的读写服务，Broker启动后将自己注册至NameServer；随后每隔30s定期向NameServer上报Topic路由信息。每个 Broker 与NameServer 集群中的所有节点都会建立长连接。

* Producer 与 NameServer 集群中的其中一个节点（随机选择）建立长连接，定期从 NameServer 获取 Topic 路由信息，并向提供 Topic 服务的 Broker Master 建立长连接，且定时向 Broker 发送心跳。Producer 只能将消息发送到 Broker master。

* Consumer 可同时和提供Topic服务的master和Slave建立长连接，即在master节点宕机时，消费者可以从slave节点读取消息。

# 数据存储

RocketMQ的数据存储主要有三个内容：ConsumeQueue、CommitLog和IndexFile

ConsumeQueue是消息的逻辑队列，是由20字节定长的二进制数据单元组成，其中commitLogOffset(8 byte)、msgSize(4 byte)、tagsHashCode(8 byte)；每个Topic和QueuId对应一个ConsumeQueue；单个文件大小约5.72M，每个文件由30W条数据组成，每个文件默认大小为600万个字节，当一个ConsumeQueue类型的文件写满了，则写入下一个文件。

CommitLog是消息存放的实际物理位置，每个Broker下所有的Topic下的消息队列共用同一个CommitLog的日志数据文件来存储，所有RocketMQ的写入是顺序的。单个CommitLog文件的默认大小为1G。

IndexFile即消息索引，如果一个消息包含key值的话，会使用IndexFile存储消息索引，其每个单元的数据构成为keyHash(4 byte)、commitLogOffset(8 byte)、timestamp(4 byte)、nextIndexOffset(4byte)。IndexFile主要是用来根据key来查询消息。

![](https://upload-images.jianshu.io/upload_images/4325076-15cc1e5277f58759.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000/format/webp)

* Producer端发送消息最终写入的是CommitLog，写入CommitLog有同步刷盘和异步刷盘两种方式：

    1. 同步刷盘：只有在消息真正持久化至磁盘后，Broker端才会真正地返回给Producer端一个成功的ACK响应。

    2. 异步刷盘：只要消息写入PageCache即可将成功的ACK返回给Producer端。

* Consumer端先从ConsumeQueue读取持久化消息的offset，随后再从CommitLog中进行读取消息的真正实体内容。所以实际上读取操作是随机而不是顺序的，所以这也是消费速度是比Kafka低的原因。

# 生产者

RocketMQ发送消息有三种方式

* 同步

    消息发送后，等待服务端的ack响应，这种方式最可靠，但效率最低

* 异步

    消息发送注册回调函数，不需等待服务端的响应

* 单向

    消息发送后，不关心服务端是否成功接受

如果Producer发送消息失败，会自动重试，重试的策略：

* 重试次数 < retryTimesWhenSendFailed（可配置）

* 总的耗时（包含重试n次的耗时） < sendMsgTimeout（发送消息时传入的参数）

* 同时满足上面两个条件后，Producer会选择另外一个队列发送消息

# 消费者

RocketMQ消费消息主要有两种方式：

* pull模式

    由消费者客户端主动向服务端拉取消息。

    一般情况下，如果我们没有控制好pull的频率，频率过低时，则可能消费速度太低导致消息的积压，频率过高时，则可能发送过多无效或低效pull请求，增加了服务端负载。

    为了解决这个问题，RocketMQ在没有足够的消息时（如服务端没有可消费的消息），并不会立即返回响应，而是保持并挂起当前请求，待有足够的消息时在返回。并且我们需要指定offset的起点和终点，并且需要我们自己保存好本次消费的offset点，下次消费的时候好从上次的offset点开始拉取消息。

    pull模式我们并不经常使用。

* push模式

    由服务端主动地将消息推送给消费者。

    push模式下，慢消费的情况可能导致消费者端的缓冲区溢出。

    但是在RocketMQ中并不是真正的push，而是基于长轮询的pull模式的来实现的伪push。具体的实现是：Consumer端每隔一段时间主动向broker发送拉消息请求，broker在收到Pull请求后，如果有消息就立即返回数据，Consumer端收到返回的消息后，再回调消费者设置的Listener方法。如果broker在收到Pull请求时，消息队列里没有数据，broker端会阻塞请求直到有数据传递或超时才返回。

# 消费重试

即消费失败后，隔一段时间重新消费该消息。

* 重试队列

    RocketMQ会为每个消费组都设置一个Topic名称为%RETRY%+consumerGroup的重试队列（这里需要注意的是，这个Topic的重试队列是针对消费组，而不是针对每个Topic设置的），用于暂时保存因为各种异常而导致Consumer端无法消费的消息。Consumer端出现异常失败时，失败的消息会重新发送给服务端的重试队列。

* 死信队列

    重试队列中超过配置的“最大重试消费次数”后就会移入到这个死信队列中。在RocketMQ中，SubscriptionGroupConfig配置常量默认地设置了两个参数，一个是retryQueueNums为1（重试队列数量为1个），另外一个是retryMaxTimes为16（最大重试消费的次数为16次）。Broker端通过校验判断，如果超过了最大重试消费次数则会将消息移至这里所说的死信队列。这里，RocketMQ会为每个消费组都设置一个Topic命名为%DLQ%+consumerGroup的死信队列。

    一般在实际应用中，移入至死信队列的消息，需要人工干预处理。

* RocketMQ的的默认延迟级别分为16个，所以一条消息最大的重试次数为16；

    ```java
    // 源码位置：org.apache.rocketmq.store.config.MessageStoreConfig.class
    // 如需修改，则需要修改broker的配置，官方并不建议修改
    private String messageDelayLevel = "1s 5s 10s 30s 1m 2m 3m 4m 5m 6m 7m 8m 9m 10m 20m 30m 1h 2h";

    //如果我们在发送消息时设置消息的延迟级别为3，则表示消息10s后才能被消费者发现
    msg.setDelayTimeLevel(3);
    ```

* RocketMQ的Message中的reconsumeTimes属性，表示该消息当前已重试的的次数，我们可以通过如下方法来控制重试的次数

    ```java
    int reconsumeTimes = msg.getReconsumeTimes();
    // 只重试三次
    if(reconsumeTimes >= 3){
        // 表示消息已成功消费，不在放入重试队列中
        return ConsumeConcurrentlyStatus.CONSUME_SUCCESS;
    } else {
        // 表示消息消费失败，放入重试队列中，指定延迟时间后重新消费
        return ConsumeConcurrentlyStatus.RECONSUME_LATER;
    }
    ```

    RocketMQ自身的重试机制，默认消息的初始延迟级别就为3

    ```java
    // 源码位置：org.apache.rocketmq.client.impl.consumer.ProcessQueue.class
    public void cleanExpiredMsg(DefaultMQPushConsumer pushConsumer) {
        ...
        pushConsumer.sendMessageBack(msg, 3);
        ...
    }
    ```