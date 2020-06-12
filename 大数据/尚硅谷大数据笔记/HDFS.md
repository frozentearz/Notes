[TOC]

# 第一节、HDFS 介绍

## 一、HDFS 概念

>   HDFS (Hadoop Distributed File System) 是一个 **分布式** 的 **文件系统**

## 二、HDFS 优缺点

### 1. 优点

	1. 高容错性
 	2. 适合大数据处理
 	3. 流式数据访问，能保证数据一致性
 	4. 可构建在廉价机器上，通过多副本机制，提高可靠性

### 2. 缺点

1.  不适合低延时的数据访问，比如毫秒级的存储数据

2.  无法高效对大量小文件进行存储

    >   原因：
    >
    >   -   会占用大量 NameNode 内存来存小文件的元数据，而 NameNode 总内存有限
    >   -   小文件存储的寻址时间会超过读取时间，违反了HDFS的设计原则

3.  不支持并发写入和文件随机修改

    >   原因：
    >
    >   -   一个文件只能有一个写，不允许多线程同时写入
    >   -   仅支持数据 append (追加)，不支持文件的随机修改

## 三、HDFS 的组成架构

![image-20200602145631874](https://i.loli.net/2020/06/02/hbBHs68S4RGqdrD.png)

### 1. HDFS 的组件

#### NameNode(nn)

>   存储文件的元数据(属性数据，如文件名，文件属性，目录结构)，以及每个文件的块列表和块所在的 DataNode 等

#### DataNode(dn)

> 存储具体文件块数据，以及数据的校验和

#### Secondary NameNode(2nn)

> 在 NameNode 忙的时候帮忙同步镜像文件和 log 日志

### 2. HDFS 的文件块 —— Block

>   HDFS 的文件在物理上是分块存储的，块的大小可通过 `dfs.blocksize` 设定
>
>   -   hadoop2.x 版本中默认块的大小是 **128M**
>   -   老版本是 64M

**问：为什么HDFS的块的大小不能设置太大也不能设置太小？** [点此查看](./学习大数据中遇到的问题/Question.md#为什么HDFS的块的大小不能设置太大也不能设置太小)

**问：为什么hadoop2.x版本的块大小默认设置为128M？** [点此查看](./学习大数据中遇到的问题/Question.md#为什么hadoop2.x版本的块大小默认设置为128M？)

# 第二节、HDFS 使用

## 一、HDFS 的Shell操作

```shell
hadoop fs xxx
```

**问：hadoop fs 命令和前面提到的 hdfs dfs 命令有区别吗？** [点此查看](./学习大数据中遇到的问题/Question.md#hadoop-fs、hadoop-dfs与hdfs-dfs命令的区别是什么？)

1.  -moveFromLocal 从本地剪贴到hdfs
2.  -copyFromLocal 从本地复制到hdfs
3.  -copyFToLocal [-get] 从hdfs复制到本地
4.  -getmerge 合并下载多个文件
5.  -appendToFile 追加一个文件到已存在的文件末尾

## 二、用Java调用HDFS的API进行开发

### 1. 环境准备

1. 下载安装 JDK 并配置环境变量
2. 下载对应系统所需要的 hadoop jar 包
3. 配置 `HADOOP_HOME` 并添加 `HADOOP_HOME\bin` 到 `PATH` 环境变量
4. 可下载 Maven 并配置
5. 可优化相关 IDEA 设置

### 2. 开始开发测试

1. 创建 maven 项目，导入相关依赖：`junit`、`log4j`、`hadoop-common`、`hadoop-client`、`hadoop-hdfs`
2. 配置 log4j
3. 创建测试类
4. 编写代码，[进入实操](./实操记录/用java对hadoopAPI进行开发测试.md)

# 第三节、HDFS的数据流 :star:

## 一、上传写入数据流

![image-20200611204640473](https://i.loli.net/2020/06/11/QSwC4bI1gPatsGl.png)

1. 客户端向NameNode发起上传文件请求
2. NameNode响应可以上传文件
3. 客户端收到响应后再本地对文件分块
4. 客户端向NameNode请求上传第一个块文件
5. NameNode根据 [**机架感知**](#三机架感知) 返回若干节点给客户端
6. 客户端获取节点后选择一个节点(DataNode1)建立文件传输通道
7. DataNode向其他节点(DataNode2)请求建立文件传输通道，二号DataNode再向其他节点(DataNode3)建立传输通道
8. 等待最后一个DataNode全部建立传输通道后，客户端在本地把第一块需要上传的Block拆成一个个64K大小的packet，通过PIPE LINE传输给DataNode
9. 客户端与dataNode1传输完毕，dataNode1则返回一个成功答应
10. 客户端收到答应后再向NameNode发送第一块传输完成信息，如果有第二块则重复以上流程
11. NameNode收到第一块传输完成后会对集群内的副本数进行数量校验，如果不足指定副本数则自动进行异步的同步

## 二、下载读取数据流

![image-20200612003433437](https://i.loli.net/2020/06/12/1qPN3Q9HTUmzpeC.png)

1. 客户端向NameNode请求要下载的文件
2. NameNode通过查询元数据找到文件块所在的DateNode地址
3. 挑选一台就近的几点，然后读取数据
4. DataNode开始向客户端以Packet为单位传输数据
5. 客户端以Packet为单位接收，先在本地缓存，然后写入目标文件

## 三、机架感知

1.  低版本的机架感知规则

    >   1.  第一个副本选择客户端所在的节点上，如果客户端不在集群内则随机选择一个
    >   2.  第二个副本选择在第一个副本的不同机架的随机节点
    >   3.  第三个副本选择在第二个副本相同机架的随机节点

2.  2.7.2版本-新版本规则

    >   1.  第一个副本选择客户端所在的节点上，如果客户端不在集群内则随机选择一个
    >   2.  第二个副本选择在第一个副本的相同机架的随机节点
    >   3.  第三个副本选择其他机架的随机节点

# 第四节、NameNode与SecondaryNameNode

## 一、NameNode与SecondaryNameNode工作机制

![image-20200612135247234](D:/Program Files/Typora/upload/image-20200612135247234.png)

### 第一阶段：NameNode启动

1.  第一次启动NameNode格式化后，创建Fsimage和Edits文件。如果不是第一次启动，直接加载编辑日志和镜像文件到内存，加载时为[安全模式](#五NameNode的安全模式)(只读)

2.  客户端对元数据进行增删改的请求
3.  NameNode记录操作日志，更新滚动日志
4.  NameNode在内存中对数据进行增删改

### 第二阶段：Secondary NameNode工作

1.  Secondary NameNode询问NameNode是否需要CheckPoint。直接带回NameNode是否检查结果
2.  Secondary NameNode请求执行CheckPoint
3.  NameNode滚动正在写的Edits日志
4.  将滚动前的编辑日志和镜像文件拷贝到Secondary NameNode
5.  Secondary NameNode加载编辑日志和镜像文件到内存，并合并
6.  生成新的镜像文件fsimage.chkpoint
7.  拷贝fsimage.chkpoint到NameNode
8.  NameNode将fsimage.chkpoint重新命名成fsimage。

## 二、Fsimage与Edits解析

>   Fsimage：NameNode内存中元数据序列化后形成的文件
>
>   Edits：记录客户端更新元数据信息的每一步操作（可通过Edits运算出元数据）

-   用oiv命令查看Fsimage文件
-   用oev查看Edits文件

## 三、CheckPoint时间设置

1.  通常情况下，SecondaryNameNode每隔一小时执行一次

    ```xml
    <property>
      <name>dfs.namenode.checkpoint.period</name>
      <value>3600</value>
    </property>
    ```

2.  一分钟检查一次操作次数，当操作次数达到1百万时，SecondaryNameNode执行一次

    ```xml
    <property>
      <name>dfs.namenode.checkpoint.txns</name>
      <value>1000000</value>
    <description>操作动作次数</description>
    </property>
    
    <property>
      <name>dfs.namenode.checkpoint.check.period</name>
      <value>60</value>
    <description> 1分钟检查一次操作次数</description>
    </property >
    ```

## 四、NameNode故障处理

>   NameNode故障后，可以采用如下两种方法恢复数据

### 方法一：将SecondaryNameNode中数据拷贝到NameNode存储数据的目录

1.  kill -9 NameNode进程
2.  删除NameNode存储的数据
3.  拷贝SecondaryNameNode中数据到原NameNode存储数据目录
4.  重新启动NameNode

### 方法二：使用-importCheckpoint选项启动NameNode守护进程，从而将SecondaryNameNode中数据拷贝到NameNode目录中

1.  修改hdfs-site.xml

    ```xml
    <property>
      <name>dfs.namenode.checkpoint.period</name>
      <value>120</value>
    </property>
    
    <property>
      <name>dfs.namenode.name.dir</name>
      <value>/opt/module/hadoop-2.7.2/data/tmp/dfs/name</value>
    </property>
    ```

2.  kill -9 NameNode进程

3.  删除NameNode存储的数据

4.  如果SecondaryNameNode不和NameNode在一个主机节点上，需要将SecondaryNameNode存储数据的目录拷贝到NameNode存储数据的平级目录，并删除in_use.lock文件

5.  导入检查点数据（等待一会ctrl+c结束掉）

    ```shell
    bin/hdfs namenode -importCheckpoint
    ```

6.  启动NameNode

## 五、NameNode的安全模式

### 1. 概述

![image-20200612143452597](https://i.loli.net/2020/06/12/Y6vQcafOBXZtDrA.png)

### 2. 基本语法

>   集群处于安全模式，不能执行重要操作（写操作）。集群启动完成后，自动退出安全模式。

-   bin/hdfs dfsadmin -safemode get     （功能描述：查看安全模式状态）

-   bin/hdfs dfsadmin -safemode enter  （功能描述：进入安全模式状态）

-   bin/hdfs dfsadmin -safemode leave  （功能描述：离开安全模式状态）

-   bin/hdfs dfsadmin -safemode wait    （功能描述：等待安全模式状态）

## 六、NameNode多目录配置