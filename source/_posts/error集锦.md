error集锦

* err1

```
Exception in thread "main" java.lang.NoClassDefFoundError: storm/kafka/BrokerHosts
    at java.lang.Class.getDeclaredMethods0(Native Method)
    at java.lang.Class.privateGetDeclaredMethods(Class.java:2570)
    at java.lang.Class.getMethod0(Class.java:2813)
    at java.lang.Class.getMethod(Class.java:1663)
    at sun.launcher.LauncherHelper.getMainMethod(LauncherHelper.java:494)
    at sun.launcher.LauncherHelper.checkAndLoadMain(LauncherHelper.java:486)
Caused by: java.lang.ClassNotFoundException: storm.kafka.BrokerHosts
    at java.net.URLClassLoader$1.run(URLClassLoader.java:366)
    at java.net.URLClassLoader$1.run(URLClassLoader.java:355)
    at java.security.AccessController.doPrivileged(Native Method)
    at java.net.URLClassLoader.findClass(URLClassLoader.java:354)
    at java.lang.ClassLoader.loadClass(ClassLoader.java:425)
    at sun.misc.Launcher$AppClassLoader.loadClass(Launcher.java:308)
    at java.lang.ClassLoader.loadClass(ClassLoader.java:358)

```

原因：storm的server端的lib包中缺少jar包   

==一定要注意版本，本文用的kafka_2.11-0.10.0.0，storm1.1.0==

把maven本地库中的`storm-kafka-1.1.0.jar`复制到$STORM_HOME/lib下

* err2

```
Exception in thread "main" java.lang.NoClassDefFoundError: kafka/api/OffsetRequest
        at org.apache.storm.kafka.KafkaConfig.<init>(KafkaConfig.java:44)
        at org.apache.storm.kafka.SpoutConfig.<init>(SpoutConfig.java:42)
        at Kafka2StormTopologyMain.main(Kafka2StormTopologyMain.java:16)
Caused by: java.lang.ClassNotFoundException: kafka.api.OffsetRequest
        at java.net.URLClassLoader$1.run(URLClassLoader.java:366)
        at java.net.URLClassLoader$1.run(URLClassLoader.java:355)
        at java.security.AccessController.doPrivileged(Native Method)
        at java.net.URLClassLoader.findClass(URLClassLoader.java:354)
        at java.lang.ClassLoader.loadClass(ClassLoader.java:425)
        at sun.misc.Launcher$AppClassLoader.loadClass(Launcher.java:308)
        at java.lang.ClassLoader.loadClass(ClassLoader.java:358)
        ... 3 more

```

解决:
把maven本地库中的 *kafka_2.11-0.10.0.0.jar* 复制到 **$STORM_HOME/lib**下

* err3

```
Exception in thread "main" java.lang.NoClassDefFoundError: scala/ScalaObject
        at java.lang.ClassLoader.defineClass1(Native Method)
        at java.lang.ClassLoader.defineClass(ClassLoader.java:800)
        at java.security.SecureClassLoader.defineClass(SecureClassLoader.java:142)
        at java.net.URLClassLoader.defineClass(URLClassLoader.java:449)
        at java.net.URLClassLoader.access$100(URLClassLoader.java:71)
        at java.net.URLClassLoader$1.run(URLClassLoader.java:361)
        at java.net.URLClassLoader$1.run(URLClassLoader.java:355)
        at java.security.AccessController.doPrivileged(Native Method)
        at java.net.URLClassLoader.findClass(URLClassLoader.java:354)
        at java.lang.ClassLoader.loadClass(ClassLoader.java:425)
        at sun.misc.Launcher$AppClassLoader.loadClass(Launcher.java:308)
        at java.lang.ClassLoader.loadClass(ClassLoader.java:358)
        at org.apache.storm.kafka.KafkaConfig.<init>(KafkaConfig.java:44)
        at org.apache.storm.kafka.SpoutConfig.<init>(SpoutConfig.java:42)
        at Kafka2StormTopologyMain.main(Kafka2StormTopologyMain.java:16)

```

解决:
把maven本地库中的 *scala-library-2.11.8.jar* 复制到 **$STORM_HOME/lib**下

* err4

```
Exception in thread "main" java.lang.NoClassDefFoundError: com/google/common/base/Strings
        at org.apache.storm.kafka.KafkaSpout.declareOutputFields(KafkaSpout.java:184)
        at org.apache.storm.topology.TopologyBuilder.getComponentCommon(TopologyBuilder.java:383)
        at org.apache.storm.topology.TopologyBuilder.createTopology(TopologyBuilder.java:135)
        at Kafka2StormTopologyMain.main(Kafka2StormTopologyMain.java:27)

作者：zhouhaolong1
链接：https://www.jianshu.com/p/70c3a7f56386
來源：简书
简书著作权归作者所有，任何形式的转载都请联系作者获得授权并注明出处。
```

解决:
把maven本地库中的 *guava-15.0.jarr* 复制到 **$STORM_HOME/lib**下

* err5

```
java.lang.NoClassDefFoundError: org/apache/curator/RetryPolicy  
```

解决:
把maven本地库中的curator-client-2.6.0.jar和 *curator-framework-2.6.0.jar* 复制到 **$STORM_HOME/lib**下

* err6

```
java.lang.NoClassDefFoundError: org/json/simple/JSONValue
```

解决:
把maven本地库中的 *json-simple-1.1.jar* 复制到 **$STORM_HOME/lib**下

* err7

```
java.lang.NoClassDefFoundError: com/yammer/metrics/Metrics at kafka.metrics.KafkaMetricsGroup
$class.newTimer(KafkaMetricsGroup.scala:52) at
kafka.consumer.FetchRequestAndResponseMetrics.newTimer(FetchRequestAndResponseStats.scala:25)
at kafka.consumer.FetchRequestAndResponseMetrics.<init>
(FetchRequestAndResponseStats.scala:26) at kafka.consumer.FetchRequestAndResponseStats.<init>
(FetchRequestAndResponseStats.scala:37) at
kafka.consumer.FetchRequestAndResponseStatsRegistry$$anonfun$2.apply
(FetchRequestAndResponseStats.scala:50) ...

```

解决:
把maven本地库中的 *metrics-core-2.2.0.jar* 复制到 **$STORM_HOME/lib**下

* err8

```
java.lang.NoSuchMethodError: org.apache.zookeeper.ZooKeeper.<init>
```

解决:
把maven本地库中的 *zookeeper-3.4.6.jar* 复制到 **$STORM_HOME/lib**下