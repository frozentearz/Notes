[TOC]

# 第一节、Hadoop 框架介绍

## 一、Hadoop 三大版本

1. Apache 版本

    > 最基础的版本，入门最好

2. Cloudera 版本

    > 公司常用，简称CDH，旨在解决Hadoop生态圈内不同框架之间的兼容性问题

3. Hortonworks 版本

    > 成立之初吸纳了Hadoop工程师贡献的80%源代码，所以文档特别全面

## 二、Hadoop 的优势

1. 高可靠性
2. 高扩展性
3. 高效性
4. 高容错性

## **三、Hadoop 组成** :star:

|           | Hadoop distributed file System | MapReduce          | YARN           |
| --------- | ------------------------------ | ------------------ | -------------- |
| hadoop1.x | HDFS                           | MR<计算和资源调度> |                |
| hadoop2.x | HDFS                           | MR<仅计算>         | YARN<资源调度> |

### 1. HDFS —— Hadoop distributed file System

1. NameNode(nn)

    > 存储文件的元数据(属性数据，如文件名，文件属性，目录结构)，以及每个文件的块列表和块所在的DataNode等

2. DataNode(dn)

    > 存储具体文件块数据，以及数据的校验和

3. Secondary NameNode(2nn)

    > 给NameNode干活的

### 2. YARN

1. ResourceManager(rm)

    > 负责整个集群的资源调度

2. NodeManager(nn)

    > 负责一个节点的资源调度

3. ApplicationMaster

    > 任务分配调度

4. Container

    > 对任务运行环境的抽象，封装了一个节点的资源，一个节点资源的抽象

### 3. MapReduce —— 基于硬盘的离线计算框架

1. Map 阶段

    > 分——把数据拆分，然后并行处理 
    >
    > **问：Map把数据拆分后对数据进行并行处理，是Map来处理还是会交给其他组件计算？** [点此查看](./学习大数据中遇到的问题/Question.md#Map把数据拆分后对数据进行并行处理，是Map来处理还是会交给其他组件计算)

2. Reduce 阶段

    > 合—— 把处理完的数据合并汇总 
    >
    > **问：处理完的数据Reduce是从哪里获取到的呢？Map还是其他组件？** [点此查看](./学习大数据中遇到的问题/Question.md#处理完的数据Reduce是从哪里获取到的呢？Map还是其他组件？)

## 四、大数据技术生态体系

![image-20200601014418210](https://i.loli.net/2020/06/01/GAsaJTOxESY41gp.png)

# 第二节、Hadoop 环境搭建(虚拟机)

## 一、操作系统准备

1.  准备一份原镜像(硬盘给50G，后期改硬盘大小很难改、NAT网络模式)

2.  克隆虚拟机(注意目录结构规划)

3.  修改被克隆的机器的静态 ip

    >   ```shell
    >   sudo vim /etc/udev/reles.d/70-persistent-net.reles
    >   ```
    >
    >   因为克隆虚拟机会导致 MAC 地址一致，从而导致自带的第一个网卡无效
    >
    >   需要删除无效网卡，新 MAC 地址网卡改名为 eth0，复制新 MAC 地址到：
    >
    >   ```shell
    >   vim /etc/stsconfig/network-scripts/ifconfig-eth0
    >   ```
    >
    >   覆盖其中无效的 MAC 地址
    >
    >   再修改其中静态 ip 为所需要的 ip

4.  修改主机名

    ```shell
    sudo vim /etc/sysconfig/network # 修改其中 HOSTNAME 为所需要的主机别名
    ```
    
5. 关闭防火墙

    ```shell
    # CentOS 6.5
    servcie iptables status/restart/stop/start # 临时操作防火墙
    chkconfig iptables off # 永久关闭防火墙
    # CentOS 7.2
    systemctl status/restart/stop/start/disable/enable firewalld.service
    ```

6. 创建用户，给权限

    ```shell
    su -
    useradd xxx
    passwd xxx
    id xxx
    vim /etc/sudoers # 在第92行添加一条跟root一致的配置
    ```

7. 创文件夹放大数据生态圈组件，改权限和所有者

    ```shell
    cd /opt
    mkdir module software
    chown xxx:xxx module/ software/
    ```

8.  SSH 测试连接

## 二、软件组件安装

### 1. JDK 安装

​	网上教程太多，略过

### 2. Hadoop 安装

1. 下载&解压(好像需要编译，暂时先拿老师提供的已编译版本解压)

2. 配置环境变量 `HADOOP_HOME`，添加 `HADOOP_HOME\bin` 和 `HADOOP_HOME\sbin` 到 `PATH`

### 3. Hadoop 目录结构

- bin —— hadoop 组件调用命令(hadoop、hdfs、mapred、yarn)
- etc —— hadoop 配置文件
- lib —— 依赖包，主要是so文件，系统相关文件
- sbin —— hadoop 启动停止等程序
- share —— 文档及各个组件的工具包

# 第三节、Hadoop 的使用

## 1. Hadoop 三种运行模式

### 本地(独立)模式  [Local (Standalone) Mode](https://hadoop.apache.org/docs/stable/hadoop-project-dist/hadoop-common/SingleCluster.html#Standalone_Operation)

> 在单节点上作为单个Java进程运行，对于调试很有用

<small>ps: 可以使用本地模式尝试运行官方[grep案例](./实操记录/用Hadoop在Linux文件系统上跑grep案例.md)和[wordcount案例](./实操记录/用Hadoop在Linux文件系统上跑wordcount案例.md)</small>

### 伪分布式模式  [Pseudo-Distributed Mode](https://hadoop.apache.org/docs/stable/hadoop-project-dist/hadoop-common/SingleCluster.html#Pseudo-Distributed_Operation)

> 以伪分布式模式在单节点上运行

#### 配置 HDFS

 1. hadoop-env.sh

    配置具体的 `JAVA_HOME` 地址，因为跨节点可能会读不到对方节点的环境变量

 2. core-site.xml

    - 配置 fs.defaultFS，默认是使用 Linux 的文件系统。现在修改为 HDFS 文件系统

        ```xml
        <property>
            <name>fs.defaultFS</name>
            <value>hdfs://HOSTNAME:9000</value>
            <description>The name of the default file system.  A URI whose
            scheme and authority determine the FileSystem implementation.  The
            uri's scheme determines the config property (fs.SCHEME.impl) naming
            the FileSystem implementation class.  The uri's authority is used to
            determine the host, port, etc. for a filesystem.</description>
        </property>
        ```

    - 配置 hadoop.tmp.dir，默认目录在 /tmp 下，而操作系统在硬盘空间不足时会删除 /tmp 下的文件，从而导致 Hadoop 丢失数据，所以安全起见在 hadoop 安装路径下新建 data/tmp 用于保存数据

        ```xml
        <property>
            <name>hadoop.tmp.dir</name>
            <value>${HADOOP_HOME}/data/tmp</value>
            <description>A base for other temporary directories.</description>
        </property>
        ```

 3. hdfs-site.xml

    配置 dfs.replication，默认是3，指 HDFS 的副本数量，单节点建议改为1

    ```xml
    <property>
        <name>dfs.replication</name>
        <value>1</value>
        <description>Default block replication. 
          The actual number of replications can be specified when the file is created.
          The default is used if replication is not specified in create time.
        </description>
    </property>
    ```
    
    **问：单节点在不改的情况下会在本地保存三份数据吗？为什么？** [点此查看](./学习大数据中遇到的问题/Question.md#单节点在不改dfs.replication的情况下会在本地保存三份数据吗？为什么？)

#### 启动 HDFS

1. 格式化 NameNode —— 为了生成工作空间，格式化之前防止 ClusterId 不一致需要删除 /data 以及 /log 目录

    ```shell
    bin/hdfs namenode -format
    ```

2. 启动 NameNode

    ```shell
    sbin/hadoop-daemon.sh start namenode
    ```

3. 启动 DataNode

    ```shell
    sbin/hadoop-daemon.sh start datanode
    ```

4. 查看是否正常启动

    - jsp
    - ip:50070 (hadoop的web页面)

#### 配置 YARN 与 MapReduce (MapReduce 需要在 YARN 上运行)

1. yarn-env.sh

    修改 `JAVA_HOME`，原理同上

2. yarn-site.xml

    - 配置 yarn.nodemanager.aux-services，指获取 reduce 的数据获取方式，目前使用 shuffle 方式

        ```xml
        <property>
            <description>A comma separated list of services where service name should only
              contain a-zA-Z0-9_ and can not start with numbers</description>
            <name>yarn.nodemanager.aux-services</name>
            <value>mapreduce_shuffle</value>
        </property>
        ```

    - 配置 yarn.resourcemanager.hostname，指定 YARN 的 ResourceManager 的地址，默认0.0.0.0

        ```xml
        <property>
            <description>The hostname of the RM.</description>
            <name>yarn.resourcemanager.hostname</name>
            <value>你的HOSTNAME</value>
        </property>  
        ```

3. mapred-env.sh

    配置 `JAVA_HOME`，同上

4. mapred-site.xml (copy from mapred-site.xml.template)

    配置 mapreduce.framework.name，指定 MapReduce 运行在 YARN 上

    ```xml
    <property>
        <name>mapreduce.framework.name</name>
        <value>yarn</value>
        <description>The runtime framework for executing MapReduce jobs.
        Can be one of local, classic or yarn.
        </description>
    </property>
    ```

#### 启动 YARN

0. 检查 NameNode 和 DataNode 已经启动，关闭的要先关闭 YARN 再关闭 HDFS

1. 启动 YARN (RecourseManager和NodeManager)

    ```shell
    yarn-daemon.sh start resoucemanager
    yarn-daemon.sh start nodemanager
    ```

    **问：如果先启动nodemanager再启动resourcemanager会报错吗？** [点此查看](./学习大数据中遇到的问题/Question.md#如果先启动nodemanager再启动resourcemanager会报错吗？)

2. 查看是否启动

    - jsp
    - ip:8088/cluster (yarn的web页面)

#### 配置历史服务器

1. mapred-site.xml

    - 配置 mapreduce.jobhistory.address 

        ```xml
        <property>
            <name>mapreduce.jobhistory.address</name>
            <value>IP地址或主机名:10020</value>
            <description>MapReduce JobHistory Server IPC host:port</description>
        </property>
        ```

    - 配置 mapreduce.jobhistory.webapp.address 

        ```xml
        <property>
            <name>mapreduce.jobhistory.webapp.address</name>
            <value>IP地址或主机名:19888</value>
            <description>MapReduce JobHistory Server Web UI host:port</description>
        </property>
        ```

#### 启动历史服务器

```shell
sbin/mr-jobhistory-daemon.sh start historyserver
```

#### 配置日志服务器

1. yarn-site.xml

    - 配置 yarn.log-aggregation-enable，打开日志聚集功能，默认关闭

      ```xml
      <property>
          <description>Whether to enable log aggregation. Log aggregation collects
            each container's logs and moves these logs onto a file-system, for e.g.
            HDFS, after the application completes. Users can configure the
            "yarn.nodemanager.remote-app-log-dir" and
            "yarn.nodemanager.remote-app-log-dir-suffix" properties to determine
            where these logs are moved to. Users can access the logs via the
            Application Timeline Server.
          </description>
          <name>yarn.log-aggregation-enable</name>
          <value>true</value>
      </property>
      ```

    - 配置 yarn.log-aggregation.retain-seconds，设置日志保留时间为7天

      ```xml
      <property>
          <description>How long to keep aggregation logs before deleting them.  -1 disables. 
          Be careful set this too small and you will spam the name node.</description>
          <name>yarn.log-aggregation.retain-seconds</name>
          <value>604800</value>
      </property> 
      ```

#### 启动日志服务器

1. 重启 YARN

    ```shell
    yarn-daemon.sh restart resoucemanager
    yarn-daemon.sh restart nodemanager
    ```

2. 重启历史服务器

    ```shell
    mr-jobhistory-daemon.sh restart historyserver
    ```

3. 查看 log 

    IP:19888

### 全分布式运行 [Fully-Distributed Mode](https://hadoop.apache.org/docs/stable/hadoop-project-dist/hadoop-common/SingleCluster.html#Fully-Distributed_Operation)

> 搭建一个完整的集群

#### 配置全分布式运行

1. 
