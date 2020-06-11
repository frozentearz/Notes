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
5. NameNode根据**机架感知**返回若干节点给客户端
6. 客户端获取节点后选择一个节点(DataNode1)建立文件传输通道
7. DataNode向其他节点(DataNode2)请求建立文件传输通道，二号DataNode再向其他节点(DataNode3)建立传输通道
8. 等待最后一个DataNode全部建立传输通道后，客户端在本地把第一块需要上传的Block拆成一个个64K大小的packet，通过PIPE LINE传输给DataNode
9. 客户端与dataNode1传输完毕，dataNode1则返回一个成功答应
10. 客户端收到答应后再向NameNode发送第一块传输完成信息，如果有第二块则重复以上流程
11. NameNode收到第一块传输完成后会对集群内的副本数进行数量校验，如果不足指定副本数则自动进行异步的同步

## 一、下载读取数据流