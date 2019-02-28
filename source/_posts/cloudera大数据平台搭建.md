---
title: cloudera大数据平台搭建流程
tags:
---
cloudera大数据平台搭建流程

# 基础环境

* 软件环境

| NO   | 软件名称         | 版本                                            |
| ---- | ---------------- | ----------------------------------------------- |
| 1    | 操作系统         | centos7 64位                                    |
| 2    | jdk              | jdk-8u181-linux-x64.tar.gz                      |
| 3    | cloudera manager | cloudera-manager-centos7-cm5.15.1_x86_64.tar.gz |
| 4    | cdh              | CDH-5.15.1-1.cdh5.15.1.p0.4-el7                 |
| 5    | 数据库           | mysql-5.7.18-1.el7.x86_64.rpm-bundle.tar        |
| 6    | jdbc             | mysql-connector-java-5.1.43.jar                 |

* 配置规划

| NO   | 机器名称 | IP        | 配置         | 用途                |
| ---- | -------- | --------- | ------------ | ------------------- |
| 1    | master   | 10.2.47.1 | 16C/32G/500G | master，cm，mysqldb |
| 2    | slave1   | 10.2.47.2 | 16C/32G/500G | slave               |
| 3    | slave2   | 10.2.47.3 | 16C/32G/500G | slave               |
| 4    | slave3   | 10.2.47.4 | 16C/32G/500G | slave               |
| 5    | slave4   | 10.2.47.5 | 16C/32G/500G | slave               |

* 软件资源

  1. jdk

  下载地址：[jdk-8u181-linux-x64.tar.gz](https://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html)

  2. CM

  下载地址：[cloudera-manager-centos7-cm5.15.1_x86_64.tar.gz](http://archive-primary.cloudera.com/cm5/cm/5/cloudera-manager-centos7-cm5.15.1_x86_64.tar.gz)

  3. cdh包

  下载地址：[manifest.json](http://archive.cloudera.com/cdh5/parcels/5.15.1/manifest.json)

  下载地址：[CDH-5.15.1-1.cdh5.15.1.p0.4-el7.parcel.sha1](http://archive.cloudera.com/cdh5/parcels/5.15.1/CDH-5.15.1-1.cdh5.15.1.p0.4-el7.parcel.sha1)

  下载地址：[CDH-5.15.1-1.cdh5.15.1.p0.4-el7.parcel](http://archive.cloudera.com/cdh5/parcels/5.15.1/CDH-5.15.1-1.cdh5.15.1.p0.4-el7.parcel)

  4. jdbc连接jar包

  下载地址：[mysql-connector-java-5.1.43.jar](http://central.maven.org/maven2/mysql/mysql-connector-java/5.1.43/mysql-connector-java-5.1.43.jar)

# 基础环境配置

1. jdk安装配置

   * 卸载系统自带jdk

   ```sh
   rpm -qa | grep java
   [root@localhost ~]# rpm -qa | grep java
   python-javapackages-3.4.1-5.el7.noarch
   java-1.7.0-openjdk-headless-1.7.0.51-2.4.5.5.el7.x86_64
   tzdata-java-2014b-1.el7.noarch
   javapackages-tools-3.4.1-5.el7.noarch
   java-1.7.0-openjdk-1.7.0.51-2.4.5.5.el7.x86_64
   删除全部，noarch可不删除
   [root@localhost ~]# rpm -e --nodeps java-1.7.0-openjdk-headless-1.7.0.51-2.4.5.5.el7.x86_64
   [root@localhost ~]# rpm -e --nodeps java-1.7.0-openjdk-1.7.0.51-2.4.5.5.el7.x86_64
   ```

   * 安装jdk

   ```sh
   [root@localhost soft]# tar -zxvf jdk-8u181-linux-x64.tar.gz 
   [root@localhost soft]# mv jdk1.8.0_181 jdk
   配置环境变量
   [root@localhost soft]# vim /etc/profile
   export JAVA_HOME=/soft/jdk
   export PATH=.:$JAVA_HOME/bin:$PATH
   ```

   2. 安装mysql

   ```mysql
   安装前先卸载mariadb 
   # rpm -qa | grep mariadb
   # rpm -e -nodeps mariadb-libs-5.5.35-3.el7.x86_64
   [root@localhost soft]#tar -xvf mysql-5.7.18-1.el7.x86_64.rpm-bundle.tar
   按common–>libs–>client–>server->devel的顺序安装：
   # rpm -ivh mysql-community-common-5.7.18-1.el7.x86_64.rpm
   # rpm -ivh mysql-community-libs-5.7.18-1.el7.x86_64.rpm
   # rpm -ivh mysql-community-client-5.7.18-1.el7.x86_64.rpm
   # rpm -ivh mysql-community-server-5.7.18-1.el7.x86_64.rpm
   # rpm -ivh mysql-community-devel-5.7.18-1.el7.x86_64.rpm 
   # rpm -ivh  mysql-community-libs-compat-5.7.18-1.el7.x86_64.rpm
   启动数据库
   # systemctl start mysqld 
   查看状态： 
   # systemctl status mysqld
   修改mysql初始密码
   1、先修改配置文件/etc/my.cnf令MySQL跳过登录时的权限检验，在[mysqld]下加入一行：
   skip-grant-tables
   2、重启MySQL
   #service mysqld restart
   3、免密码登录MySQL。
   #mysql
   4、mysql客户端执行如下命令，修改root密码
   mysql>  use mysql;
   mysql> UPDATE user SET authentication_string = password('your-password') WHERE host = 'localhost' AND user = 'root';
   mysql> select host,user, authentication_string, password_expired from user; 
   mysql> update user set password_expired='N' where password_expired='Y' //密码不过期
   mysql> update user set host='%' where user='root' and host='localhost'; //远程可访问
   mysql> flush privileges; //刷新
   mysql> exit;//退出
   5、修改配置文件/etc/my.cnf删除此前新增那一行skip-grant-tables，并重启MySQL(这一步非常重要，不执行可能导致严重的安全问题)
   #service mysqld restart //重启 Mysql
   后期会出现 == 初始化数据库错误 ==：
   在这个环节，出现的问题较多，但总的来说，是与数据库参数配置，和帐号权限配置有关。
   如后面 在执行scm_prepare_database.sh脚本时，出现的错误：
   
   java.sql.SQLException: Your password does not satisfy the current policy requirements
   
   一般是由密码策略级别问题引发：
   可以通过 my.cnf 配置文件关闭 validate_password 插件。
   通过修改/etc/my.cnf 目录下配置文件，修改设置密码策略的级别，只需要在[mysqld]下添加一行 
   validate_password = off
   编辑完配置文件后，重启mysqld服务即可生效。
   
   ```

   ![mysql](/images/cloudera/mysql.jpg)

   3. mysql卸载

   ```
    rpm -qa | grep -i mysql
    [root@localhost local]# rpm -e MySQL-server-5.6.17-1.el6.i686
   [root@localhost local]# rpm -e MySQL-client-5.6.17-1.el6.i686
   c)删除mysql服务
   [root@localhost local]# chkconfig --list | grep -i mysql
   [root@localhost local]# chkconfig --del mysql
   d)删除分散mysql文件夹
   [root@localhost local]# whereis mysql 或者 find / -name mysql
   mysql: /usr/lib/mysql /usr/share/mysql
   清空相关mysql的所有目录以及文件
   rm -rf /usr/lib/mysql
   rm -rf /usr/share/mysql
   rm -rf /usr/my.cnf
   ```



   3. 源码安装mysql

   ```
   下载mysql源码mysql-5.5.17.tar.gz  http://dev.mysql.com/get/Downloads/MySQL-5.6/mysql-5.6.22.tar.gz/from/http://cdn.mysql.com/
   yum方式安装相关依赖包  yum -y install cmake bison git ncurses-devel gcc gcc-c++
   -创建一个用户名为mysql的用户并加入mysql用户组 
   # groupadd mysql
   # useradd -g mysql mysql
   解压mysql-5.6.38.tar.gz，并且创建mysql安装目录和数据库文件存放目录
   # tar zxvf mysql-5.6.38.tar.gz 
   # mkdir /usr/local/mysql
   # mkdir /usr/local/mysql/data
   # cd mysql-5.6.38/
   # cmake -DCMAKE_INSTALL_PREFIX=/usr/local/mysql -DMYSQL_UNIX_ADDR=/usr/local/mysql/mysql.sock -DDEFAULT_CHARSET=utf8 -DDEFAULT_COLLATION=utf8_general_ci -DWITH_INNOBASE_STORAGE_ENGINE=1 -DWITH_ARCHIVE_STORAGE_ENGINE=1 -DWITH_BLACKHOLE_STORAGE_ENGINE=1 -DMYSQL_DATADIR=/usr/local/mysql/data -DMYSQL_TCP_PORT=3306 -DMYSQL_USER=mysql -DENABLE_DOWNLOADS=1
   
    进入/mysql目录执行
   make && make install
   修改目录属主权限
   # chown -R mysql:mysql /usr/local/mysql/data/
   # chown -R mysql:mysql /usr/local/mysql/
   创建MySQL Server系统表
   # cd /usr/local/mysql/
   # scripts/mysql_install_db --user=mysql --datadir=/usr/local/mysql/data
    配置启动脚本
   cp /usr/local/mysql/support-files/mysql.server /etc/init.d/mysqld
   添加配置文件vim /etc/my.cnf
   [mysql]
   # 设置mysql客户端默认字符集
   default-character-set=utf8 
   socket=/var/lib/mysql/mysql.sock
   [mysqld]
   skip-name-resolve
   #设置3306端口
   port = 3306 
   socket=/var/lib/mysql/mysql.sock
   # 设置mysql的安装目录
   basedir=/usr/local/mysql
   # 设置mysql数据库的数据的存放目录
   datadir=/usr/local/mysql/data
   # 允许最大连接数
   max_connections=200
   # 服务端使用的字符集默认为8比特编码的latin1字符集
   character-set-server=utf8
   # 创建新表时将使用的默认存储引擎
   default-storage-engine=INNODB 
   lower_case_table_names=1
   max_allowed_packet=16M
   
   #忘记密码时可取消注释，无密码登陆
   #skip-grant-tables
   添加环境变量
   vim /etc/profile
   export MYSQL_HOME=/usr/local/mysql
   export PATH=.:$MYSQL_HOME/bin:$PATH
   source /etc/profile
   设置开机启动：
   chkconfig mysqld on
   启动服务：
   service mysqld start
   设置初始密码
   1、使用空的初始密码登录mysql账号：
   mysql-uroot -p
    
   2、修改root密码：
   mysql> update user set Password=password("123456") where User='root';
   Query OK, 4 rows affected (0.01 sec)
   Rows matched: 4  Changed: 4  Warnings: 0
    
   mysql> flush privileges;
   Query OK, 0 rows affected (0.04 sec)
    
   mysql> select Host,User,password from user where user='root';
   +-----------------------+------+-------------------------------------------+
   | Host                  | User | password                                  |
   +-----------------------+------+-------------------------------------------+
   | localhost             | root | *5626ED34B75C6C508BA2A3D0A4F6E4C58823138C |
   | localhost.localdomain | root | *5626ED34B75C6C508BA2A3D0A4F6E4C58823138C |
   | 127.0.0.1             | root | *5626ED34B75C6C508BA2A3D0A4F6E4C58823138C |
   | ::1                   | root | *5626ED34B75C6C508BA2A3D0A4F6E4C58823138C |
   +-----------------------+------+-------------------------------------------+
   4 rows in set (0.00 sec)
   ```



   3. 修改机器名称

   ```sh
   方法1：hostnamectl set-hostname master
   方法2：vi /etc/hostname 
   master                
   :wq
   [root@bogon ~]# reboot
   
   ```

   4. 配置hosts

   ```host
   #vim /etc/hosts
   10.2.47.1 master
   10.2.47.2 slave1
   10.2.47.3 slave2
   10.2.47.4 slave3
   10.2.47.5 slave4
   ```

   5. 设置防火墙（所有节点）

   ```sh
   # firewall-cmd  --state（查询防火墙状态） 
   #systemctl stop firewalld.service （关闭防火墙） 
   #systemctl start firewalld.service （开启防火墙）
   #systemctl disable firewalld.service （禁止firewall开机启动）
   //禁止开机启动
   systemctl disable firewalld
   Removed symlink /etc/systemd/system/multi-user.target.wants/firewalld.service.
   Removed symlink /etc/systemd/system/dbus-org.fedoraproject.FirewallD1.service.
   
   设置防火墙策略，在所有节点执行下面脚本（执行前要启动防火墙）(可省略)
   //集群机器间可以相互访问
   firewall-cmd --permanent --add-rich-rule="rule family='ipv4' source address='10.2.47.1' port protocol='tcp' port='0-65535' accept"
   firewall-cmd --permanent --add-rich-rule="rule family='ipv4' source address='10.2.47.2' port protocol='tcp' port='0-65535' accept"
   firewall-cmd --permanent --add-rich-rule="rule family='ipv4' source address='10.2.47.3' port protocol='tcp' port='0-65535' accept"
   firewall-cmd --permanent --add-rich-rule="rule family='ipv4' source address='10.2.47.4' port protocol='tcp' port='0-65535' accept"
   firewall-cmd --permanent --add-rich-rule="rule family='ipv4' source address='10.2.47.5' port protocol='tcp' port='0-65535' accept" 
   //设置可以访问的端口
   firewall-cmd --permanent --add-port=0-65535/tcp
   firewall-cmd --reload
   
   ```

   6. 配置ssh免密登录

   ```sh
   #cd~ //进入到 根目录
   # ssh-keygen -t rsa
    一路回车，生成无密码的密钥对。
   # cp id_rsa.pub authorized_key
   # ssh-copy-id -i slave1
   # ssh-copy-id -i slave2
   # ssh-copy-id -i slave3
   # ssh-copy-id -i slave4
   ```

   7. 关闭selinux（所有节点）

   ```sh
   关闭linux SELINUX安全内核
   # setenforce 0 （临时生效）
   修改 /etc/selinux/config 下的 SELINUX=disabled （重启后永久生效）
   # vi /etc/selinux/config
   内容增加：
   SELINUX=disabled
   # reboot
   查看SELINUX 是否关闭：
   #sestatus
   ```

   8. 修改内核参数(所有节点)

   ```sh
   1.设置swappiness，控制运行时内存的相对权重，Cloudera 建议将 swappiness 设置为 10：
   //查看swappiness
   # cat /proc/sys/vm/swappiness
   //永久性修改，执行下面两条命令
   # sysctl -w vm.swappiness=10
   # echo vm.swappiness = 10 >> /etc/sysctl.conf 
   
   2.关闭透明大页面：
   自CentOS6版本开始引入了Transparent Huge Pages(THP)，从CentOS7版本开始，该特性默认就会启用。尽管THP的本意是为提升内存的性能，不过某些数据库厂商还是建议直接关闭THP，否则可能会导致性能出现下降。
   首先查看透明大页是否启用，[always] never表示已启用，always [never]表示已禁用：
   # cat /sys/kernel/mm/transparent_hugepage/defrag
   [always] madvise never
   # cat /sys/kernel/mm/transparent_hugepage/enabled
   [always] madvise never
   以上状态就说明是启用的。
   临时关闭（重启机器会变回默认开启状态）：
   # echo never > /sys/kernel/mm/transparent_hugepage/defrag
   #echo never > /sys/kernel/mm/transparent_hugepage/enabled
   永久关闭：
   //编辑/etc/rc.d/rc.local
   # vi /etc/rc.d/rc.local
   //在文件后添加下面内容:
   if test -f /sys/kernel/mm/transparent_hugepage/enabled; then
   echo never > /sys/kernel/mm/transparent_hugepage/enabled
   fi
   if test -f /sys/kernel/mm/transparent_hugepage/defrag; then
   echo never > /sys/kernel/mm/transparent_hugepage/defrag
   fi
   保存退出，然后赋予rc.local文件执行权限：
   #chmod +x /etc/rc.d/rc.local
   
   3.修改文件句柄数：
   修改系统文件句柄数限制：
   //查看文件句柄数，显示1024，显然太小
   # ulimit -n 
   1024
   //修改限制
   #vi /etc/security/limits.conf 
   //在文件后加入下面内容：
   
   * soft nofile 100000
   * hard nofile 100000
   修改后需要重启机器。
   
   
   ```

   9. 其他安装与配置

   ```sh
   # yum  -y  install psmisc MySQL-python at bc bind-libs bind-utils cups-client cups-libs cyrus-sasl-gssapi cyrus-sasl-plain ed fuse fuse-libs httpd httpd-tools keyutils-libs-devel krb5-devel libcom_err-devel libselinux-devel libsepol-devel libverto-devel mailcap noarch mailx mod_ssl openssl-devel pcre-devel postgresql-libs python-psycopg2 redhat-lsb-core redhat-lsb-submod-security  x86_64 spax time zlib-devel
   #yum install -y python-lxml
   #yum install krb5-devel cyrus-sasl-gssapi cyrus-sasl-deve libxml2-devel libxslt-devel mysql mysql-devel openldap-devel python-devel python-simplejson sqlite-devel
   
   # chmod +x /etc/rc.d/rc.local
   # yum -y install rpcbind
   # systemctl start rpcbind
   # echo "systemctl start rpcbind" >> /etc/rc.d/rc.local
   
   可能出现问题如下：
   Loaded plugins: fastestmirror, langpacks
   Repository base is listed more than once in the configuration
   Repository updates is listed more than once in the configuration
   Repository extras is listed more than once in the configuration
   Repository centosplus is listed more than once in the configuration
   证明源有问题，只需把源更新成mirror.gree.com即可
   ```

   10. 配置NTF服务

   ```sh
   集群中所有主机必须保持时间同步，如果时间相差较大会引起各种问题。 具体建设过程如下：所有节点安装相关组件：
   # yum install  ntp  ntpdate  -y
   NTP服务端（主节点）：
   1. 查找时间同步服务器http://www.pool.ntp.org/zone/asia：
   打开网址，内容如下：
   2. 编辑 /etc/ntp.conf：
   # vi /etc/ntp.conf
   //在文件中输入上面网页内容：
   server 10.2.8.44 
   server 10.2.47.1
   
   server 0.asia.pool.ntp.org
   server 1.asia.pool.ntp.org
   server 2.asia.pool.ntp.org
   server 3.asia.pool.ntp.org
   3.启动ntp服务
   # systemctl start  ntpd
   4. 配置开机启动：
   # systemctl enable  ntpd.service  
   
   注意：如果ntpd 开机启动失效，有可能是因为安装了chronyd 并且是开机自启状态，所以导致ntpd开机自启失败。
   # 查看  chronyd设置状态
   # systemctl status chronyd
   表示没问题
   ● chronyd.service - NTP client/server
      Loaded: loaded (/usr/lib/systemd/system/chronyd.service; enabled; vendor preset: enabled)
      Active: failed (Result: signal) since Sat 2018-10-13 11:00:50 CST; 43s ago
    Main PID: 1163 (code=killed, signal=KILL)
   
   Oct 13 10:30:33 master chronyd[1163]: Linux kernel major=3 minor=10 patch=0
   Oct 13 10:30:33 master chronyd[1163]: hz=100 shift_hz=7 freq_scale=1.00000000 nomin...l=2
   Oct 13 10:31:33 master systemd[1]: Started NTP client/server.
   以面表明，chronyd显示为开机关闭状态。
   
   将chronyd设为禁用状态：
   # systemctl disable chronyd.service
   此时，NTP的服务开机自启动完成！ 
   5. 检查是否设置成功：
   # ntpq  -p
   //更新时间
   [root@master ~]# ntpq -p
        remote           refid      st t when poll reach   delay   offset  jitter
   ==============================================================================
   
   
   *10.2.8.44       10.4.20.100      4 u  113  256  377    1.136    3.295   1.089
    master          .INIT.          16 u    - 1024    0    0.000    0.000   0.000
   
   #timedatectl 
   
   [root@master ~]# timedatectl 
         Local time: Sat 2018-10-13 14:00:35 CST
     Universal time: Sat 2018-10-13 06:00:35 UTC
           RTC time: Sat 2018-10-13 06:00:35
          Time zone: Asia/Shanghai (CST, +0800)
        NTP enabled: no
   NTP synchronized: yes
    RTC in local TZ: no
         DST active: n/a
   
   
   NTP客户端（所有从节点）：
   6. 远程客户端时间同步测试
   # date
   # ntpdate 10.2.47.1
   10.2.47.1是NTP服务端IP，显示如下信息，测试成功：
   [root@slave1 ~]# date
   Sat Oct 13 14:00:50 CST 2018
   [root@slave1 ~]# ntpdate 10.2.47.1
   13 Oct 14:01:08 ntpdate[5973]: adjust time server 10.2.47.1 offset -0.192486 sec
   
   7. 客户端设置计划任务，每30分钟同步时间
   #crontab -e 
   //加入内容：
   0-59/30 * * * * /usr/sbin/ntpdate 10.2.47.1 && /sbin/hwclock -w
   8. 设置定时任务开机启动
   //设置开机启动
   # systemctl enable crond.service
   
   //查看状态
   # systemctl status crond
   
   ```

   ![ntp](/images/cloudera/ntp.jpg)

# 安装CM

1. 传包、解压

```sh
在主节点上下载相关软件包，这里将软件包下载到/soft目录下。
将CM解压到/opt/目录
[root@master soft]# tar -zxvf cloudera-manager-centos7-cm5.15.1_x86_64.tar.gz  -C /opt/
#ls /opt/
[root@master opt]# ls
cloudera  cm-5.15.1  rh

```

2. 创建数据库

```
在主节点上：
# mysql -h127.0.0.1 -uroot -p   //加参数-h127.0.0.1 指定本机方式，否则可能不允许执行grant
Enter password:          \\输入数据库密码
//在MariaDB [(none)]>命令状态输入下面脚本：
create database hive DEFAULT CHARSET utf8 COLLATE utf8_general_ci;
create database amon DEFAULT CHARSET utf8 COLLATE utf8_general_ci;
create database hue DEFAULT CHARSET utf8 COLLATE utf8_general_ci;
create database monitor DEFAULT CHARSET utf8 COLLATE utf8_general_ci;
create database oozie DEFAULT CHARSET utf8 COLLATE utf8_general_ci;
grant all privileges on *.* to root@localhost identified by 'root' with grant option;
grant all on *.* to root@"%" Identified by "root";
flush privileges;
exit;
//复制Mysql JDBC包到/opt/cm-5.15.1/share/cmf/lib/目录
[root@master soft]# cp mysql-connector-java-5.1.43.jar /opt/cm-5.15.1/share/cmf/lib/

[root@master soft]# /opt/cm-5.15.1/share/cmf/schema/scm_prepare_database.sh  mysql cm -h10.2.47.1 -uroot -pa464999578 --scm-host 10.2.47.1 scm scm scm


```

3. 创建用户

```sh
在所有节点上执行：
[root@master ~]# useradd --system --home=/opt/cm-5.15.1/run/cloudera-scm-server/  --no-create-home --shell=/bin/false --comment "Cloudera SCM User" cloudera-scm

```

4. 制作本地yum源

```sh
在主节点上：
//进入软件包目录
[root@master ~]# cd /soft/
//拷贝三个文件到/opt/cloudera/parcel-repo/目录
[root@master soft]# cp CDH-5.15.1-1.cdh5.15.1.p0.4-el7.parcel CDH-5.15.1-1.cdh5.15.1.p0.4-el7.parcel.sha1 manifest.json /opt/cloudera/parcel-repo/
//进入/opt/cloudera/parcel-repo/目录
#cd /opt/cloudera/parcel-repo/
//修改文件名
[root@master parcel-repo]# mv CDH-5.15.1-1.cdh5.15.1.p0.4-el7.parcel.sha1 CDH-5.15.1-1.cdh5.15.1.p0.4-el7.parcel.sha
此时/opt/cloudera/parcel-repo/目录下文件：
[root@master parcel-repo]# ls
CDH-5.15.1-1.cdh5.15.1.p0.4-el7.parcel      manifest.json
CDH-5.15.1-1.cdh5.15.1.p0.4-el7.parcel.sha


```

5. 拷贝jar包

```sh
在所有节点上：
//如果没有/usr/share/java/目录，则创建
# cp mysql-connector-java-5.1.43-bin.jar /usr/share/java/mysql-connector-java.jar
在主节点上：
//将mysql JDBC jar 包拷贝到  /opt/cm-5.15.1/share/cmf/lib/ 目录
[root@master soft]# cp mysql-connector-java-5.1.43.jar /opt/cm-5.15.1/share/cmf/lib/
```

6. 修改cloudera-scm-agent配置

```sh
在主节点上，修改/opt/cm-5.15.1/etc/cloudera-scm-agent/config.ini文件：
//将config.ini server_host=localhost 内容改为server_host=10.2.47.1
# sed -i "s/server_host=localhost/server_host=10.2.47.1/" /opt/cm-5.13.1/etc/cloudera-scm-agent/config.ini
在主节点上：
//将cm-5.15.1 打包，并复制到其他节点
#cd /opt 
#tar czf cm-5.15.1.tar.gz  cm-5.15.1/
//复制到其他节点
[root@master opt]# scp cm-5.15.1.tar.gz root@slave1:/opt/
[root@master opt]# scp cm-5.15.1.tar.gz root@slave2:/opt/
[root@master opt]# scp cm-5.15.1.tar.gz root@slave3:/opt/
[root@master opt]# scp cm-5.15.1.tar.gz root@slave4:/opt/

 在所有从节点上解压：
//将cm-5.15.1 包解压
#cd opt 
#tar -xzvf cm-5.15.1.tar.gz 
//解压后删除
#rm -rf cm-5.15.1.tar.gz
```

7. 启动CM Server和Agent

```sh
在主节点上，启动cloudera-scm-server：
[root@master opt]# /opt/cm-5.15.1/etc/init.d/cloudera-scm-server start
启动过程较慢，可通过/opt/cm-5.15.1/log/cloudera-scm-server日志，查看启动过程。
在所有节点上，启动cloudera-scm-agent：
[root@master opt]# /opt/cm-5.15.1/etc/init.d/cloudera-scm-agent start
```

8. 访问CM

```sh
地址：http://master:7180    http://10.2.47.1:7180
账号：admin
密码：admin
显示如下：
```

![cloudera manager]()

# 安装CDH

1. 登录后界面接受协议：

![hello CM](/images/cloudera/cmhello.jpg)

2. 选择CM版本，本文选择Cloudera Express免费版。

![version](/images/cloudera/version.jpg)

3. 指定主机

```sh
在搜索主机名和IP地址框输入slave[1-4]，这里输入的内容支持正则表达式。输入后点【搜索】按钮，出现机器列表：
```

![address](/images/cloudera/address.jpg)

选择“当前管理的主机“选择项卡，点【继续】

![nodes](/images/cloudera/address.jpg)

4. 选择CDH版本

```sh
这里需要选择制作本地源时的版本，如果选择别的版本的就会去官网下载，那样安装速度会很慢。
```

![source version](/images/cloudera/version1.jpg)

点【继续】按钮，进入安装界面

3. 检查主机

```
在搜索主机名和IP地址框输入slave[1-4]，这里输入的内容支持正则表达式。输入后点【搜索】按钮，出现机器列表,点击继续
```

* 出现cdh5.15.1新版无法显示，此时我们需要做httpd服务

```sh
安装http和启动http服务
#rpm -qa |grep httpd
#yum install -y httpd
#chkconfig httpd on
#service httpd start
创建parcels文件夹
#mkdir /var/www/html/parcels
#cd /var/www/html/parcels
# cp CDH-5.15.1-1.cdh5.15.1.p0.4-el7.parcel CDH-5.15.1-1.cdh5.15.1.p0.4-el7.parcel.sha manifest.json  /var/www/html/parcels
在网页中输入 http://10.2.47.1/parcels

```

* 出现找不到存储包,解决如下:

![save](/images/cloudera/save.jpg)

  点击使用parcel 更多选项更改远程parcel存储库URL`http://10.2.47.1/parcels/`

安装

![](/images/cloudera/clouderainstall.jpg)

* 出现“主机运行状态不良”错误

![buliang](/images/cloudera/buliang.jpg)

解决方案如下

```sh
遇到节点“主机运行状态不良”的提示，解决办法是删除故障节点Agent服务cm_guid文件：
#rm -rf /opt/cm-5.13.1/lib/cloudera-scm-agent/cm_guid
重新启动故障节点Agent服务：
#/opt/cm-5.13.1/etc/init.d/cloudera-scm-agent restart
重启故障节点Agent服务后，故障消失：
注：出现故障原因是，因为我之前在故障节点启动过cloudera-scm-agent服务。
```



5. 检查主机，全部通过

![check](/images/cloudera/check.jpg)

6. 选择安装的服务，全部安装

![install](/images/cloudera/all.jpg)



7. 角色分配

![fenpei](/images/cloudera/分配1.jpg)

![fenpei2](/images/cloudera/分配2.jpg)

![fenpei3](/images/cloudera/分配3.jpg)

8. 数据库设置

```
指定的数据库名称，与原来创建的数据库（hive,monitor,oozie,hue）保持一致，并输入对应的数据库用户名和密码，为了简便这里用的是root账号名和密码。
测试连接，Hue 测试报错：Unable to verify database connection：
原因是缺少Mysql mysql-community-libs-compat 安装包，安装后，问题解决：
//进入安装包所在目标
# cd /data/mysql/ 
# rpm -ivh  mysql-community-libs-compat-5.7.18-1.el7.x86_64.rpm
测试成功，继续

```

![](/images/cloudera/test.jpg)

9. 集群设置，默认配置，直接点击继续

![](/images/cloudera/1.jpg)

![2](/images/cloudera/2.jpg)

![3](/images/cloudera/3.jpg)

![4](/images/cloudera/4.jpg)

![5](/images/cloudera/5.jpg)

10. 开始安装

* 安装spark显示找不到JAVA_HOME,解决方案如下：

```
#mkdir -p /usr/java
#ln -s /soft/jdk /usr/java/default
刷新问题解决。
```

* 出现hive数据库驱动找不到,解决方案如下：

```
因为缺少mysql驱动
[root@master soft]# cp /soft/mysql-connector-java-5.1.43.jar /opt/cloudera/parcels/CDH-5.15.1-1.cdh5.15.1.p0.4/lib/hive/lib/
问题解决。
```

* 出现oozie数据库驱动找不到,解决方案如下：

```
因为缺少mysql驱动
[root@master soft]# cp /soft/mysql-connector-java-5.1.43.jar /opt/cloudera/parcels/CDH-5.15.1-1.cdh5.15.1.p0.4/lib/oozie/lib/
[root@master soft]# cp /soft/mysql-connector-java-5.1.43.jar /opt/cloudera/parcels/CDH-5.15.1-1.cdh5.15.1.p0.4/lib/oozie/libext/

```

11. 安装完成

![finally](/images/cloudera/finally.jpg)

第一次安装完成后，会出现一些配置的警告信息。这些可以根据提示信息更改。

* 警告处理

发现节点有异常信息，显示所有主机，发现节点运行状态不良，按照 主机->所有主机->进入节点查看：

![ntperr](/images/cloudera/ntperr.jpg)

选择“配置“页签，拉到页面底部，修改”主机时钟偏差阈值“，设为”从不“，点【保存更改】，异常消失。

![](/images/cloudera/ntpconf.jpg)

![ntp](/images/cloudera/更改1.jpg)

* 未能连接到 Host Monitor

![hostno](/images/cloudera/hostno.jpg)

```
解决方案：
后台tail -f /opt/cm-5.15.1/log/cloudera-scm-server/cloudera-scm-server.log 日志报错信息：
com.cloudera.cmon.MgmtServiceLocatorException: Could not find a HOST_MONITORING nozzle from SCM.
此问题原因：有些网上说是由文件句柄数限制引起，所以按照网上说明进行了修改：
在主节点上，修改/opt/cm-5.15.1/etc/cloudera-scm-agent/config.ini文件：
//查看文件句柄数，显示1024，显然太小
# ulimit -n 
1024
//修改限制
#vi /etc/security/limits.conf 
//在文件后加入下面内容：
* soft nofile 100000
* hard nofile 100000
此步骤需要重启机器生效，可以设置完后再重启。

```

