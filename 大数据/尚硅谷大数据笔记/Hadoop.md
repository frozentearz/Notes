[TOC]

### Hadoop框架

#### 一、Hadoop 三大版本

1. Apache 版本

    > 最基础的版本，入门最好

2. Cloudera 版本

    > 公司常用，简称CDH，旨在解决Hadoop生态圈内不同框架之间的兼容性问题。

3. Hortonworks 版本

    > 成立之初吸纳了Hadoop工程师贡献的80%源代码，所以文档特别全面

#### 二、Hadoop 的优势

1. 高可靠性
2. 高扩展性
3. 高效性
4. 高容错性

#### **三、Hadoop 组成** :star:

|           | Hadoop distributed file System | MapReduce          |                |
| --------- | ------------------------------ | ------------------ | -------------- |
| hadoop1.x | HDFS                           | MR<计算和资源调度> |                |
| hadoop2.x | HDFS                           | MR<仅计算>         | YARN<资源调度> |

##### 1. HDFS —— Hadoop distributed file System

1. NameNode(nn)

    > 存储文件的元数据(属性数据，如文件名，文件属性，目录结构)，以及每个文件的块列表和块所在的DataNode等

2. DataNode(dn)

    > 存储具体文件块数据，以及数据的校验和

3. Secondary NameNode(2nn)

    > 给NameNode干活的（待补充）

##### 2. YARN

1. ResourceManager(rm)

    > 负责整个集群的资源调度

2. NodeManager(nn)

    > 负责一个节点的资源调度

3. ApplicationMaster

    > 任务分配调度

4. Container

    > 对任务运行环境的抽象，封装了一个节点的资源，一个节点资源的抽象

##### 3. MapReduce —— 基于硬盘的离线计算框架

1. Map 阶段

    > 分——把数据拆分，然后并行处理 **问：这个并行处理计算的行为是Map做的吗？还是说要调度给其他什么东西做？**

2. Reduce 阶段

    > 合—— 把处理完的数据合并汇总 **问：处理完的数据是从哪里拿？**

#### 四、大数据技术生态体系

![image-20200601014418210](https://i.loli.net/2020/06/01/GAsaJTOxESY41gp.png)