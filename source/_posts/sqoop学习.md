sqoop学习

#### 一、介绍

sqoop是RDBMS和hdfs之间进行数据导入导出的工具，高效的数据导入导出工具。

#### 二、安装

1. sqoop2.X安装
   sqoop2.X包含两部分，server和client
   server：所有客端的入口点
   client：可以安装在任何节点上

2. server安装
   * 确保hadoop可用
   * 确保HADOOP_HOME环境变量可用
   * core-site.xml
     hadoop.proxyuser.sqoop2.hosts=*
     hadoop.proxyuser.sqoop2.groups=*
   * 第三方jar包，mysql驱动放到sqoop/tools/lib下即可
   * 配置PATH
   * 修改sqoop.properties
   * 仓库初始化  sqoop2-tool upgrade
   * 验证是否初始化成功   sqoop2-tool  verify
   * server 启动和停止   sqoop2-server start/stop

3. client安装
   * 将文件拷贝到对应的客户端
   * 启动客户端shell  sqoop2-shell
   * 连接到自己的server  set server --host s101  --port  12000 --webapp  sqoop
   * 验证连接  show version --all
   * 查看sqoop注册的连接器  show connector
   * 创建连接（mysql）  create link --connector  generic-jdbc-connector  然后输入一些，driver，url等参数
   * 创建hdfs连接器
     create link --connector  hdfs-connector  一些参数
   * 显示所有连接
     show link -all  显示细节
     show link
   * 创建作业
     create job -f    “创建的mysql连接器的名字”    -t    “创建的hdfs连接器的名字”  接下来是参数
   * 显示job
     show job  -all
   * 启动作业
     start job  -- name “job的名称” 

#### 三、sqoop 1.46使用

1. 安装
2. 配置
   * 环境变量  SQOOP_HOME
   * sqoop-env.sh  保证和hadoop协同
     export HADOOP_COMMON_HOME=/usr/hadoop
     export HADOOP_MAPRED_HOME=/usr/hadoop
3. 配置mysql驱动程序，放到sqoop/lib目录下
4. 验证sqoop安装
   sqoop-version
5. sqoop  导入  从mysql------>hdfs
   * 查看帮助  sqoop help
   * 查看特定指令帮助   sqoop  help  指令
   * sqoop import --connect jdbc:mysql://s100:3306/test --driver com.mysql.jdbc.Driver --username root     --password password  --table  mytest  --target-dir  hdfs://s100:8020/user/root/ --where "id">"1" --increment append（增量导入可选）--check-column id （检查列）
6. sqoop  将mysql中所有表导入到hdfs
   sqoop import-all-tables  --connect jdbc:mysql://s100:3306/test --driver com.mysql.jdbc.Driver --username root     --password password    默认使用表名作为目录
7. export 导出  hdfs------>mysql
   sqoop export  --connect jdbc:mysql://s100:3306/test --driver com.mysql.jdbc.Driver --username root     --password password  --table  mytest  --export-dir  hdfs://s100:8020/user/root/ --columns id ,name,age
8. 导入mysql数据到hive中
   sqoop import--connect jdbc:mysql://s100:3306/test --driver com.mysql.jdbc.Driver --username root     --password password  --table  mytest  --hive-import   --hive-overwrite  --hive-table myhive（hive中的表名）
9. 将mysql数据导入到hbase（不要使用它的创建表），需要启动好hbase
   sqoop import--connect jdbc:mysql://s100:3306/test --driver com.mysql.jdbc.Driver --username root     --password password  --table  mytest --hbase-table us1:t1 --hbase-row-key  id --column-family f1
10. 创建作业
    sqoop job --create myjob  sqoop import--connect jdbc:mysql://s100:3306/test --driver com.mysql.jdbc.Driver --username root     --password password  --table  mytest --hbase-table us1:t1 --hbase-row-key  id --column-family f1
11. 查看所有job列表
    sqoop job --list
12. 查看指定job
    sqoop job --show myjob
13. 启动job
    sqoop job -exec myjob
14. 删除job
    sqool job --delete myjob
15. sqoop的增量导入
    * check-column 检查字段，该字段不能使用varchar类型
    * increment  append/lastmodified
    * --last-value 最后的值，指定之前导入时候的最大值，指的是mysql中的值，把mysql表中新增加的内容导入到其他的数据库中