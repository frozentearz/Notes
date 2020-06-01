[TOC]

### 第一节、Hadoop 框架介绍

#### 一、Hadoop 三大版本

1. Apache 版本

    > 最基础的版本，入门最好

2. Cloudera 版本

    > 公司常用，简称CDH，旨在解决Hadoop生态圈内不同框架之间的兼容性问题

3. Hortonworks 版本

    > 成立之初吸纳了Hadoop工程师贡献的80%源代码，所以文档特别全面

#### 二、Hadoop 的优势

1. 高可靠性
2. 高扩展性
3. 高效性
4. 高容错性

#### **三、Hadoop 组成** :star:

|           | Hadoop distributed file System | MapReduce          | YARN           |
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

    > 合—— 把处理完的数据合并汇总 **问：老师讲的是把Map处理完的数据通过Reduce汇总，但是确定处理数据的执行者是Map吗？还是Map仅仅只是一个间接者，需要分给其他组件跑完之后从其他组件汇总数据**

#### 四、大数据技术生态体系

![image-20200601014418210](https://i.loli.net/2020/06/01/GAsaJTOxESY41gp.png)

### 第二节、Hadoop 环境搭建(虚拟机)

#### 一、操作系统准备

1.  准备一份原镜像(硬盘给50G，后期改硬盘大小很难改、NAT网络模式)

2.  克隆虚拟机(注意目录结构规划)

3.  修改被克隆的机器的静态 ip

    >   ```shell
    >   sudo vim /etc/udev/reles.d/70-persistent-net.reles
    >   ```
    >
    >   删除其中一个网卡，保留eth0

4.  修改主机名

5.  关闭防火墙

6.  创建用户，给权限

7.  创文件夹放大数据生态圈组件，改权限和所有者