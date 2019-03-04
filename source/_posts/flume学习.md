---
title: flume学习整理
date: 2017-12-15 11:56:22
tags:
---

flume学习整理

#### 一、介绍

flume是分布式、可靠的、可用性好的服务，用于收集、聚合、异动大量日志数据，基于流计算的简单灵活框架，用于在线分析。

#### 二、特点

1. 优点
   * 可以和任何集中式存储进行集成
   * 输入数据速率大于写入存储目的地的速率时，flume会进行缓存。
   * flume提供上下文路由（数据流路线）
   * flume中的事务基于channel，使用了两个事务模型（sender，receive），确保消息被可靠发送
   * flume高效收集web server的log到hdfs
   * 可以高效获取数据
   * 导入大量数据
   * flume支持大量的source和destination类型
   * flume支持多级跳跃，source和destination的扇入和扇出
   * flume水平可扩展
2. put的问题
   * 同一时刻只能传输一个文件
   * put处理静态文件

#### 三、架构

1. 描述
   在数据生成器运行的节点上启动单独的flume agent，来收集数据，数据收集器收集数据，推送到hdfs
2. flume event
   事件是flume的传输单元，主要是byte[]，可以包含一些header信息，在source和destination之间
3. flume agent 
   每个agent是一个独立的java进程，从客户端（其他agent）接收数据，然后转发到下一个destination（sink/agent）
4. agent包含三个组件
   * source  源头 
     从事件生成器接收数据，以事件的形式，传给一个或多个channel
   * channel 通道
     从source中接收flume event ，作为临时存放地，缓存到buffer，直到sink将其消费掉，是source和sink之间的桥梁，channel是事务的，可以和多个source和sink协同
   * sink 沉槽
     存放数据到hdfs，从channel中消费event，并分发给destination，sink的destination也可以是另一个agent或者hdfs。
     一个flume的agent可以有多个source，channel，sink
5. flume的附件组件
   * interceptor
   * channel selector
   * sink processors

#### 四、安装

1. 下载flume，apache-flume-1.6.0-src.tar.gz
2. tar -zxvf apache-flume-1.6.0-src.tar.gz
3. 配置环境变量 profile
4. 验证安装 flume-ng version
5. 配置flume   conf/
   flume-env.sh  配置java_home
6. 配置flume
   * 命名agent组件
   * 描述配置source
   * 描述配置channel
   * 描述配置sink
   * 绑定source，sink到channel

#### 五、使用

1. 命令行模式 /flume.properites

   ```shell
   # Name the components on this agent
   a1.sources = r1
   a1.sinks = k1
   a1.channels = c1
   
   # Describe/configure the source
   a1.sources.r1.type = netcat
   a1.sources.r1.bind = localhost
   a1.sources.r1.port = 44444
   
   # Describe the sink
   a1.sinks.k1.type = logger
   
   # Use a channel which buffers events in memory
   a1.channels.c1.type = memory
   a1.channels.c1.capacity = 1000
   a1.channels.c1.transactionCapacity = 100
   
   # Bind the source and sink to the channel
   a1.sources.r1.channels = c1
   a1.sinks.k1.channel = c1
   ```

2. 运行flume agent
   flume-ng agent --conf . --conf-file flume.properties --name a1 -Dflume.root.logger=INFO,console
   --conf  配置文件所在目录
   --conf-file 配置文件
   --name 代理名称（agent名称）
   -D额外参数

3. 核心类考察
   source ，生成event ，调用channelprocessor的方法，将事件put到channel中
   channel，连接source（event producer）和sink（event consumer），本质上channel就是buffer，支持事务处理，保证原子性，channel是线程安全的（同步）
   sink，连接到channel，消费里面的事件，将其发送到目的地，有很多相应的sink类型，sink可以根据sinkgroup和sinkprocessor进行分组，sink的process()方法只能由一个线程访问。

4. avro客户端使用

   * 配置avro源 avro-r.properties

     ```sh
     a1.sources = r1
     a1.channels = c1
     a1.sources.r1.type = avro
     a1.sources.r1.channels = c1
     a1.sources.r1.bind = 0.0.0.0
     a1.sources.r1.port = 4141
     ```

   * 启动
     flume-ng agent --conf .   --conf-file avro-r.properites  -n a1 -Dflume.root.logger=INFO,console

   * 通过flume-ng avro-client命令发送avro数据消息
     flume-ng avro-client -H localhost -p 8888 -F a.txt -R 头文件名称

5. 将flume文件存放在zk中，从zk中读取

   ```java
   import org.apache.zookeeper.WatchedEvent;
   import org.apache.zookeeper.Watcher;
   import org.apache.zookeeper.ZooKeeper;
   import org.apache.zookeeper.data.Stat;
   
   
   import java.io.FileInputStream;
   
   public class SavePropertiesToZk {
       public static void main(String[] args) throws Exception {
           String conn = "s101:2181,s102:2181,s103:2181";
           ZooKeeper zk = new ZooKeeper(conn, 5000, new Watcher() {
               @Override
               public void process(WatchedEvent watchedEvent) {
   
               }
           });
   
           Stat s = new Stat();
           zk.getData("/flume/a1", false, s);
           int v = s.getAversion();
           FileInputStream fis = new FileInputStream("/conf/flume.properties");
           byte[] data = new byte[fis.available()];
           fis.read(data);
           fis.close();
           zk.setData("/flume/a1", data, v);
       }
   }
   
   ```

   * 在flume-ng中通过--z指定节点位置，提取属性
     flume-ng -z s101:2181,s102:2181 -p/flume -n a1 -Dflume.root.logger=INFO,console

6. exec方式
   略

7. spooling方式

   ```sh
   #定义agent组件
   a1.sources = r1
   a1.sinks =s1
   a1.channels=c1
   
   #描述sources组件
   a1.sources.r1.type =spooldir
   a1.sources.r1.spoolDir =/yu
   a1.sources.r1.fileHeader =true
   
   #描述sink组件
   a1.sinks.s1.type = org.apache.flume.sink.kafka.KafkaSink
   a1.sinks.s1.kafka.topic = temp
   a1.sinks.s1.kafka.bootstrap.servers = slave1:9092
   a1.sinks.s1.kakfa.flumeBatchSize = 20
   a1.sinks.s1.kafka.producer.acks =1
   a1.sinks.s1.kafka.producer.linger.ms=1
   
   #描述channel组件
   a1.channels.c1.type = memory
   a1.channels.c1.capacity =1000
   a1.channels.c1.transactionCapacity = 100
   
   #绑定source、sink到channel
   a1.sources.r1.channels =c1
   a1.sinks.s1.channel =c1
   滚动文件，作用监控一个文件夹中生产的文件，去指定目录下提取日志
   ```

8. tcp源
   略（找官网）

9. UDP源
   略（找官网）

   ```sh
   import java.net.DatagramPacket;
   import java.net.DatagramSocket;
   import java.net.InetAddress;
   
   public class FlumeUDP {
       public static void main(String[] args) throws Exception{
           DatagramSocket socket=new DatagramSocket();
           byte[] data="hello".getBytes();
           DatagramPacket packet=new DatagramPacket(data,data.length);
           packet.setAddress(InetAddress.getByName("s100"));
           packet.setPort(8888);
       }
   }
   ```

10. http源
    走http协议，默认使用json格式处理器，火狐可以发送json数据。

    ```java
    代码模拟
    import java.io.OutputStream;
    import java.net.HttpURLConnection;
    import java.net.URL;
    
    public class FlumeHttp {
        public static void main(String[] args) throws Exception {
            URL url = new URL("http://s100:8888");
            HttpURLConnection conn = (HttpURLConnection) url.openConnection();
            conn.setDoOutput(true);
            conn.setRequestMethod("post");
            OutputStream os = conn.getOutputStream();
            os.write("".getBytes());
            os.close();
        }
    }
    ```

11. stress压力源
    略

12. 自定义源

    * 自定义类继承AbstractEventDrivenSource
    * 导出jar mvn package
    * 复制jar到flume/lib
    * 创建cus-r.prooerties
      a1.sources.r1.type=类名全路径

13. flume的sink hdfs

    ```sh
    a1.channels = c1
    a1.sinks = k1
    a1.sinks.k1.type = hdfs
    a1.sinks.k1.channel = c1
    a1.sinks.k1.hdfs.path = /flume/events/%y-%m-%d/%H%M/%S
    a1.sinks.k1.hdfs.filePrefix = events-
    a1.sinks.k1.hdfs.round = true
    a1.sinks.k1.hdfs.roundValue = 10
    a1.sinks.k1.hdfs.roundUnit = minute
    a1.sinks.k1.hdfs.UselocalTimeStamp=true
    # 启动flume   flume-ng  agent --conf  . --conf-file hdsf-sink.properties -n a1
    ```

14. flume 的sink hive

    ```sh
    # 创建表
    create table weblogs ( id int , msg string )
        partitioned by (continent string, country string, time string)
        clustered by (id) into 5 buckets
        stored as orc;
        
    
    a1.channels = c1
    a1.channels.c1.type = memory
    a1.sinks = k1
    a1.sinks.k1.type = hive
    a1.sinks.k1.channel = c1
    a1.sinks.k1.hive.metastore = thrift://127.0.0.1:9083
    a1.sinks.k1.hive.database = logsdb
    a1.sinks.k1.hive.table = weblogs
    a1.sinks.k1.hive.partition = asia,%{country},%y-%m-%d-%H-%M
    a1.sinks.k1.useLocalTimeStamp = false
    a1.sinks.k1.round = true
    a1.sinks.k1.roundValue = 10
    a1.sinks.k1.roundUnit = minute
    a1.sinks.k1.serializer = DELIMITED
    a1.sinks.k1.serializer.delimiter = "\t"
    a1.sinks.k1.serializer.serdeSeparator = '\t'
    a1.sinks.k1.serializer.fieldnames =id,,msg
    ```

    需要报把hive的类库导入到flume中 /flume/lib flume-sink-hive

15. flume 的sink avro

    ```shZ
    a1.channels = c1
    a1.sinks = k1
    a1.sinks.k1.type = avro
    a1.sinks.k1.channel = c1
    a1.sinks.k1.hostname = 10.10.10.10
    a1.sinks.k1.port = 4545
    ```

    source 一般是bind ，sink一般是hostname
    主要收集web中的日志，最后将数据上传到hdfs，先将数据收集到统一的agent上，然后agent在上传到hdfs上。

16. flume 的sink hbase

    ```sh
    a1.channels = c1
    a1.sinks = k1
    a1.sinks.k1.type = hbase
    a1.sinks.k1.table = foo_table
    a1.sinks.k1.columnFamily = bar_cf
    a1.sinks.k1.serializer = org.apache.flume.sink.hbase.RegexHbaseEventSerializer
    a1.sinks.k1.channel = c1
    ```

    需要把hbase-client.jar 导入到flume/lib目录

17. flume的sink自定义

    * 写一个类继承AbstractSink
    * 导出jar包，放到flume/lib 目录下，mvn package
    * 配置

18. flume的channel jdbc
    jdbc的channel主要是derby和mysql
    mysql通道没有实现，需要自己实现

    * 写一个类继承AbstractChannel
    * 导出jar包，放到flume/lib下
    * 配置

    ```java
    import org.apache.flume.ChannelException;
    import org.apache.flume.Event;
    import org.apache.flume.Transaction;
    import org.apache.flume.channel.AbstractChannel;
    import org.apache.flume.event.SimpleEvent;
    
    import java.sql.Connection;
    import java.sql.DriverManager;
    import java.sql.PreparedStatement;
    import java.sql.ResultSet;
    
    public class CustomChannel extends AbstractChannel {
        static {
            try {
                Class.forName("com.mysql.jdbc.Driver");
    
            } catch (Exception e) {
    
            }
        }
    
        @Override
        public void put(Event event) throws ChannelException {
            Transaction ts = getTransaction();
            try {
                ts.begin();
                Connection conn = DriverManager.getConnection("jdbc:mysql://s100:3306/test", "root", "password");
                String msg = new String(event.getBody());
                PreparedStatement ppst = conn.prepareStatement("insert into t1(msg) value ('" + msg + "')");
                ppst.executeUpdate();
                ppst.close();
                ts.commit();
            } catch (Exception e) {
                ts.rollback();
            }
            ts.close();
        }
    
        @Override
        public Event take() throws ChannelException {
            Transaction ts = getTransaction();
            try {
                ts.begin();
                Connection conn = DriverManager.getConnection("jdbc:mysql://s100:3306/test", "root", "password");
                PreparedStatement ppst = conn.prepareStatement("select id ,msg from t1");
                conn.getAutoCommit(false);
                ResultSet rs = ppst.executeQuery();
                int id = 0;
                String msg = null;
                if (rs.next()) {
                    id = rs.getInt(1);
                    msg = rs.getString(2);
                }
                conn.prepareStatement("delete from t1 where id=" + id).executeUpdate();
                conn.commit();
                rs.close();
                ppst.close();
                SimpleEvent e = new SimpleEvent();
                e.setBody(msg.getBytes());
                return e;
    
            } catch (Exception e) {
                ts.rollback();
            }
            ts.close();
            return null;
        }
    
        @Override
        public Transaction getTransaction() {
            return new Transaction() {
                @Override
                public void begin() {
                    
                }
    
                @Override
                public void commit() {
    
                }
    
                @Override
                public void rollback() {
    
                }
    
                @Override
                public void close() {
    
                }
            };
        }
    }
    
    ```

19. flume的channel  filechannel
    a1.channel.c1.type=file

20. flume的可溢出的内存通道

    ```sh
    a1.channels = c1
    a1.channels.c1.type = SPILLABLEMEMORY
    a1.channels.c1.memoryCapacity = 10000
    a1.channels.c1.overflowCapacity = 1000000
    a1.channels.c1.byteCapacity = 800000
    a1.channels.c1.checkpointDir = /mnt/flume/checkpoint
    a1.channels.c1.dataDirs = /mnt/flume/data
    ```

21. 其他
    复制channel selector ，事件发给集群，关联多个通道，每个通道都要存放副本

    ```sh
    # channel selector configuration
    agent_foo.sources.avro-AppSrv-source1.selector.type = multiplexing
    agent_foo.sources.avro-AppSrv-source1.selector.header = State
    agent_foo.sources.avro-AppSrv-source1.selector.mapping.CA = mem-channel-1
    agent_foo.sources.avro-AppSrv-source1.selector.mapping.AZ = file-channel-2
    agent_foo.sources.avro-AppSrv-source1.selector.mapping.NY = mem-channel-1 file-channel-2
    agent_foo.sources.avro-AppSrv-source1.selector.optional.CA = mem-channel-1 file-channel-2
    agent_foo.sources.avro-AppSrv-source1.selector.mapping.AZ = file-channel-2
    agent_foo.sources.avro-AppSrv-source1.selector.default = mem-channel-1
    ```

    sink processors 提高负载均衡的功能

    ```sh
    a1.sinkgroups = g1
    a1.sinkgroups.g1.sinks = k1 k2
    a1.sinkgroups.g1.processor.type = load_balance
    ```

    使用拦截器

    ```sh
    时间戳拦截器，结果会在头上加上事件戳
    a1.sources = r1
    a1.channels = c1
    a1.sources.r1.channels =  c1
    a1.sources.r1.type = seq
    a1.sources.r1.interceptors = i1
    a1.sources.r1.interceptors.i1.type = timestamp
    ```

    ```sh
    主机拦截器
    a1.sources = r1
    a1.channels = c1
    a1.sources.r1.interceptors = i1
    a1.sources.r1.interceptors.i1.type = host
    ```

    

