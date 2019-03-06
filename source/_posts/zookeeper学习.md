zookeeper学习总结

#### 一、介绍

轻量级集群框架，协同服务，保证高可靠、高可用，集中式服务，用于配置信息，名称服务，分布式同步处理树形层次结构，znode，动物管理员

#### 二、zk组件

1. client
   向server周期性的发送消息，表明自己还活着，server向client回应确认消息，client没有收到回应，自动重定向消息到其他server
2. server
   一个zk节点，向client提供所有服务，通知client ，server是活着的
3. ensemble
   一组zk节点，需要最小值3
4. leader
   特殊的zk节点，zk集群启动时，推选leader，leader在follower故障时进行处理，有一半坏掉了，集群就挂了
5. follower（随从）
   听命于leader
6. zk的namespace的等级结构
   * 驻留在内存中
   * 树上的每一个节点都是znode。
   * 每个znode都有name，而且用/分隔
   * 每个znode存放的数据不能超过1m
   * 每个节点都有stat对象
     version 与之关联的数据发生改变时，版本增加
     acl：权限
     timestamp：时间戳
     datalength：长度
7. zk的znode节点类型
   * 持久节点
   * 临时节点
   * 序列节点

#### 三、zk安装

1. 下载
   zookeeper-3.4.10.tar.gz

2. 安装jdk

3. 配置环境变量 profile

4. 配置zk

   * 考察文件/conf/zoo_sample.conf
     tickTime=2000   每次心跳的毫秒数
     initLimit=10 开始同步阶段的心跳数
     synclimit=5 发送请求和得到确定之间可以传递的心跳数
     dataDir=/tmp/zookeeper 存放zk的快照目录
     clientport=2181  客户端连接的端口
     maxclientcnxns=60 客户端最大连接数
     autopurge.snapretainCoun=3在dataDir保留的快照数
     autopurge.purgeInterval=1  丢弃任务的间隔时间

5. 部署zk

   1. 单机版

      * 创建配置/conf/zoo.cfg
        tickTime=2000
        dataDir=/home/root/zk/data
        clientPort=2181
        initLimit=5
        synclimit=2
      * 启动zkserver
        zkServer.sh start
      * 启动zkClient
        zkCli.sh start

   2. zk完全分布式

      * 确定主机
        a101,s102,s103

      * 配置myid
        dataDir目录。/home/root/zk/myid,每台myid不一样，101机器的是101,102机器的是102

      * 配置zoo.cfg
        tickTime=2000
        dataDir=/home/root/zk/data
        clientPort=2181
        initLimit=5
        synclimit=2
        server.101=s101:2888:3888
        server.102=s102:2888:3888
        server.103=s103:2888:3888

      * 每台机器上单独启动
        zkServer.sh start
        zkServer.sh stop

        zkServer.sh status  查看节点状态

#### 四、使用

1. 客户端连接服务器
   zkCli.sh -server s101:2181 
   ls /  查看结构
   get /文件  查看数据
   create /文件名  "内容"   创建节点
   set /文件名  “文件内容”  修改内容
   dataversion 数据版本
   set /文件名  “内容”  版本号  
   delete /目录   删除节点，该节点下不能有内容
   rmr /目录  递归删除

2. 通过zk的api访问zk集群

   ```java
   import org.apache.zookeeper.*;
   import org.apache.zookeeper.data.ACL;
   
   import java.util.ArrayList;
   import java.util.List;
   
   public class ConnZookeeper {
       public static void main(String[] args) throws Exception {
           String conn = "s101:2181,s102:2181,s103:2181";
           ZooKeeper zk = new ZooKeeper(conn, 5000, new Watcher() {
               @Override
               public void process(WatchedEvent watchedEvent) {
               }
           });
           //创建权限列表
           ACL acl = new ACL(ZooDefs.Perms.ALL, ZooDefs.Ids.ANYONE_ID_UNSAFE);
           List<ACL> acls = new ArrayList<>();
           acls.add(acl);
           zk.create("/root", "helloworld".getBytes(), acls, CreateMode.PERSISTENT);
       }
   }
   
   ```

   修改znode

   ```java
           zk.setData("/root","hi".getBytes(),1);
   ```

   读取数据

   ```java
   import org.apache.zookeeper.WatchedEvent;
   import org.apache.zookeeper.Watcher;
   import org.apache.zookeeper.ZooDefs;
   import org.apache.zookeeper.ZooKeeper;
   import org.apache.zookeeper.data.ACL;
   import org.apache.zookeeper.data.Stat;
   
   import java.util.ArrayList;
   import java.util.List;
   
   public class GetZKData {
       public static void main(String[] args) throws Exception {
           String conn = "s101:2181,s102:2181,s103:2181";
           ZooKeeper zk = new ZooKeeper(conn, 5000, new Watcher() {
               @Override
               public void process(WatchedEvent watchedEvent) {
               }
           });
           //创建权限列表
           ACL acl = new ACL(ZooDefs.Perms.ALL, ZooDefs.Ids.ANYONE_ID_UNSAFE);
           List<ACL> acls = new ArrayList<>();
           acls.add(acl);
           //创建状态对象，在getDate时候，接收节点的状态信息
           Stat s = new Stat();
           byte[] bytes = zk.getData("/root", null, s);
           System.out.println(bytes);
       }
   }
   ```

   创建临时节点,程序运行结束就消失

   ```java
    zk.create("/root", "helloworld".getBytes(), acls, CreateMode.EPHEMERAL);
   ```

   创建序列节点

   ```java
    zk.create("/root", "helloworld".getBytes(), acls, CreateMode.PERSISTENT_SEQUENTIAL);
   ```

   得到孩子节点

   ```java
   List<String> list=zk.getChildren("/",null);
   for(String s:list){
       System.out.println(s);
   }
   ```

   删除节点

   ```java
   zk.delete("/root",1);
   ```

   创建节点

   ```java
   zk.create("/root","hello".getBytes(),ZooDefs.Ids.OPEN_ACL_UNSAFE,CreateMode.PERSISTENT);
   ```

#### 五、leader推选过程

1.  所有节点在相同的路径下创建各自的临时序列化节点，每个节点都有自己对应的znode。
2. 拥有最小编号的zk是leader其他的zk为follower。
3. 每个follower观察紧比自己小的那个node
4. 如果leader下线，则它对应的节点就会删除
5. 观察者会发现leader被删除
6. 观察者查找是否还有最下的节点，如果有，最小值节点成为leader，否则观察者为leader

自动容灾一般要用zk