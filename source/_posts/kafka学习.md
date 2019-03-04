kafka学习总结

#### 一、介绍

kafka类似于消息系统的发布订阅数据流，以分布式副本集群的方式存储数据流，实时处理数据，构建实时数据流通道，水平可伸缩，容错，速度快

1. MQ：消息队列。
2. JMS：java message server java消息服务，消息中间件，主要是解耦作用。
3. queue：队列模式
4. p2p：点对点模式
5. topic：主题模式  publish-subscribe  发布订阅模式
6. kafka是主题模式，可以实现队列效果

特点：

1. 巨量数据，TB级别
2. 高吞吐量，支持每秒百万消息
3. 分布式，支持在多个server之间进行消息分区
4. 多客户端支持，和多语言进行协调。

#### 二、安装kafka

1. 下载kafka_2.11-0.10.2.1.tgz

2. 解压 tar -zxvf kafka_2.11-0.10.2.1.tgz 

3. 重命名  mv kafka_2.11-0.10.2.1   kafka

4. 体验kafka
   启动zk：zookeeper-server-start.sh  config/zookeeper.properties &
   启动kafka服务器：kafka-server-start.sh config/server.properties &
   创建主题：kafka-topics.sh --create  --zookeeper localhost:2181  --replication-factor 1 --partitions 1 --topic test
   列出主题：kafka-topics.sh --list --zookeeper localhost:2181
   发送消息：kafka-console-producer.sh --broker-list localhost:9092  --topic test
   启动消费者：kafka-console-consumer.sh --zookeeper:2181 --topic test --from-beginning

5. 搭建多个broker的kafka集群

   * 创建多个server配置文件 cp server.properties server-1.properties   server-2.properties 
     broker.id=1
     listeners=PLAINTEXT://:9093
     log.dir=/temp/kafka-log
   * 启动server
     kafka-server-start.sh server-1.properties &
     kafka-server-start.sh server-2.properties &
   * 创建主题：kafka-topics.sh --create  --zookeeper localhost:2181  --replication-factor 3 --partitions 1 --topic test
   * 查看主题内容：kafka-topics.sh --describe --zookeeper localhost:2181 --topic test
   * 发送新消息给主题：kafka-console-producer.sh --broker-list localhost:2181 --topic test
   * 消费消息：kafka-console-consumer.sh --zookeeper:2181 --topic test --from-beginning

6. 容错测试

   * 找到server-1进程  ps -Af |grep server-1
   * 杀死进程server-1    kill  pid
   * 查看主题描述：kafka-topics.sh --describe --zookeeper localhost:2181 --topic test

7. kafka组件介绍
   zk  协同系统
   broker  代理kafka server，并不维护哪个消费者消费了消息
   producer  生产者
   consumer  消费者，维护了消费的消息状态
   topic  主题
   consumer group 消费者组，每个组中只有一个消费者可以消费消息
   每个消费者各成一组就是主题模式，所有消费者在一个组，就是队列模式。

8. kafka核心部分

   * 消息缓存和filesystem的存储，数据 被立刻写入os内核页
   * 消息被消费后，kafka长时间驻留消息，如有必要可以重复消费
   * 可以设置网络过载
   * 消费者保持消息的元数据
   * 消费者状态默认在zk中保存，也可以放到其他的oltp中
   * kafka的生产和消费是pull-push模式
   * kafka没有主从模式，所有的broker地位相同，broker的数据均在zk中维护，并在producer和consumer之间共享
   * kafka的负载均衡策略允许producer动态发现broker
   * producer维护了broker的连接池
   * producer可以选择同步或一部的方式向broker发送消息

9. kafka消息压缩

   * producer压缩消息，consumer解压缩消息
   * 压缩的消息没有深度限制
   * 在message的header中有一个压缩类型，低两位表示压缩类型，0表说不压缩
   * kafka镜像，将源集群数据副本化到target

10. kafka完全分布式集群

    * 使用外部zk集群存放kafka的数据
      启动s101-s103 的zk zkServer.sh start

    * 复制s100上的kafka+profile到s101-s103
      rsync -vrl kafka  root@s101:/soft/kafka

    * 配置文件conf/server.properties 
      broker.id=101
      log.dirs=XXXX

      zookeeper.connect=s101:2181,s102:2181,s103:2181

    * 创建XXXX文件夹 mkdir -p  XXXX

    * 启动kafka kafka-server-start.sh   config/server.properties &

11. kafka在zk中的znode
    controller    相当于leader
    topic  主题
    admin 删除的主题

12. 创建主题
    kafka-topics.sh --create  --zookeeper s101:2181  --replication-factor 3 --partitions 3 --topic test

13. 删除主题
    删除主题前要先启用主题删除 delete.topic.enable=true
    kafka-topics.sh --delete --zookeeper s101:2181 --topic test

14. 生产者发送消息
     kafka-console-producer.sh --broker-list s101:2181 --topic test
    生产者不在zk中注册，消费者在zk中注册，不同的分区都有自己的leader ，文件夹结构：主题+分区号
    副本数无法修改，只能在创建topic时指定，可以给副本指定不同的broker上去

15. kafka特点
    ![](/images/kafka/副本.jpg)
    kafka的分区相当于hdfs的block块，，每个分区都有3个副本。副本对应broker。
    重新指派分区：kafka-topics.sh --alter  --zookeeper  s101:2181 --replica-assignment 101:101,102:102,103:103 --topic test

16. kafka 的副本机制
    每个分区有N的副本，可以承受（N-1）节点故障，每个副本都有自己的leader，其余的都是follow
    zk中存放分区的leader和all replication信息
    每个副本存储消息的部分数据在本地log和offset，周期性的同步到磁盘上，确保消息全部写到副本或者其中一个。
    leader故障时，消息或者写到本地log，或者在producer收到确认消息前，重新发分区给new leader

17. kafka支持的副本模型

    * 同步复制
      生产者从zk中找leader，并发送消息，消息立即写到本地log，而且follow开始pull消息，每个follow将消息写到各自的本地log后，向leader发送回执，leader在收到所有的follow确认回执，再向producer发送确认消息
    * 异步复制
      leader的本地log写入完成后，即向producer发送确认回执
      组件
      broker    ---------  zk
      tpoic       ---------  zk
      producer -------- no zk
      consumer ------- zk

#### 二、代码演练

1. 编写生产者oldAPI

   ```java
   public class KafkaOldProducer {
       private static Producer<Integer, String> producer;
       private static final Properties props = new Properties();
   
       public static void main(String[] args) {
           props.put("metadata.broker.list", "s101:9092");
           props.put("serialize.class", "kafka.serialzer.StringEncoder");
           props.put("request.requisred.acks", "1");
           producer = new Producer<Integer, String>(new ProducerConfig(props));
           KeyedMessage<Integer, String> msg = new KeyedMessage<Integer, String>("test", "hello");
           producer.send(msg);
       }
   }
   ```

2. 编写生产者 new API

   ```java
   import org.apache.kafka.clients.producer.KafkaProducer;
   import org.apache.kafka.clients.producer.Producer;
   import org.apache.kafka.clients.producer.ProducerRecord;
   
   import java.util.Properties;
   
   public class KafkaNewProducer {
       public static void main(String[] args) {
           Properties props = new Properties();
           props.put("bootstrap.servers", "192.168.1.171:9092");
           props.put("acks", "all");
           props.put("retries", 0);
           props.put("batch.size", 16384);
           props.put("linger.ms", 1);
           props.put("buffer.memory", 33554432);
           props.put("key.serializer", "org.apache.kafka.common.serialization.StringSerializer");
           props.put("value.serializer", "org.apache.kafka.common.serialization.StringSerializer");
           // props.put("partitioner.class","com.yu.kafka.KafkaPartitioner");
           Producer<String, String> producer = new KafkaProducer<String, String>(props);
           for (int i = 0; i < 10; i++) {
               producer.send(new ProducerRecord<String, String>("mytopic", Integer.toString(i), Integer.toString(i)));
   
           }
           producer.close();
       }
   }
   ```

3. kafkaProducer发送消息分析
   生产者是线程安全的，维护了本地线程池，send方法时异步的，ack=all导致记录完全提交时阻塞

4. 自定义分区函数

   * 使用带有分区函数的生产者发送消息

     ```java
     public class KafkaPartitioner implements Partitioner {
     
         public KafkaPartitioner(VerifiableProperties p){}
         public int partition(String topic, Object key, byte[] keyBytes, Object value, byte[] valueBytes, Cluster cluster) {
             return 1;
         }
     
         public void close() {
     
         }
     
         public void configure(Map<String, ?> configs) {
     
         }
     }
     
     ```

   * 在生产者函数中指定分区函数`props.put("partitioner.class","com.yu.kafka.KafkaPartitioner");`

5. 编写消费者

   ```java
   import kafka.consumer.Consumer;
   import kafka.consumer.ConsumerConfig;
   import kafka.consumer.ConsumerIterator;
   import kafka.consumer.KafkaStream;
   import kafka.javaapi.consumer.ConsumerConnector;
   
   import java.util.HashMap;
   import java.util.List;
   import java.util.Map;
   import java.util.Properties;
   
   public class KafkaNewConsumer {
       public static void main(String[] args) {
           //创建属性并添加
           Properties props = new Properties();
           props.put("zookeeper.connect", "s101:2181");
           props.put("group.id", "g1");
           props.put("zookeeper.session.timeout.ms", "500");
           props.put("zookeeper.sync.time.ms", "250");
           props.put("suto.commit.interval.ms", "1000");
           //创建消费者配置对象
           ConsumerConfig config = new ConsumerConfig(props);
   //        创建连接器
           ConsumerConnector conn = Consumer.createJavaConsumerConnector(config);
           //创建主题map
           Map<String, Integer> topicMap = new HashMap<>();
           topicMap.put("test", 1);
           Map<String, List<KafkaStream<byte[], byte[]>>> map = conn.createMessageStreams(topicMap);
           List<KafkaStream<byte[], byte[]>> list = map.get("test");
           for (KafkaStream<byte[], byte[]> s : list) {
               ConsumerIterator<byte[], byte[]> it = s.iterator();
               while (it.hasNext()) {
                   String msg = new String(it.next().message());
               }
           }
           conn.shutdown();
       }
   }
   ```

6. 消费介绍
   修改/consumers/g1/offsets/test3/(0,1,2) 里面的值就可以从头开始消费，以上内容在zk中
   从哪里开始消费取决于偏移量，偏移量存在zk中
   消费者消费的偏移量记载的是消费者组消费的该分区的消息的数量，你消费一次则zk中存储的值就会变化一次
   将消息分到不同的主题上，每个主题对应一种类型的数据（mysql，hdfs，xml）等

7. kafka作为flume的sink
   a1.Channels.c1.parseAsFlumeEvent=false  必须要添加，其他的配置看官网。

8. kafka其他指令

   * 关闭kafka  kafka-server-stop.sh
   * 查看主题偏移量  kafka-consumer-offset-checker.sh  --zookeeper s101:2181 --group g1 --topic test;