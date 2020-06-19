[TOC]

# 第一节、Hive 基本概念

## 1. 什么是Hive

>   Hive 是基于Hadoop的一种**数据仓库工具**，能**将结构化的数据文件映射为一张表**，并提供**类SQL**的**查询**功能
>
>   **本质就是将HQL转化为MapReduce程序，我们大部分的任务就是写HQL**

**学习Hive的时候时刻想成MapReduce！！！**

## 2. Hive的优缺点

### 优点

1.  采用类SQL语法，简单容易上手
2.  避免学习MapReduce，减少学习成本
3.  延时执行高，擅长离线计算常用于数据分析
4.  延时执行高，所以擅长处理大数据，对小数据没有优势
    5.  可自定义函数

### 缺点

1.  表达能力有限
    -   迭代计算无法表达
    -   数据挖掘方面不擅长
2.  效率较低
    -   生成的MapReduce不够智能
    -   调优较为困难，粒度较粗

## 3. Hive的架构原理

![image-20200617154011484](https://i.loli.net/2020/06/19/PcyjtTZMk8CI9Jb.png)

## 4. Hive和数据库的区别比较

|          | 数据库                                                       | Hive                                                         |
| -------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 存储位置 | 块设备或者本地文件系统                                       | HDFS                                                         |
| 数据更新 | 经常修改                                                     | 读多写少，不建议修改数据                                     |
| 索引     | 针对列建立索引可以对于少量的特定条件数据访问具有非常高的效率 | 不能建立索引，加载数据时不会对数据进行任何处理，查询时会暴力扫描整个数据 |
| 执行     | 通常有自己的执行引擎                                         | 通过Hadoop提供的MapReduce来执行                              |
| 执行延迟 | 仅数据少时延迟低                                             | 无论数据多少延迟都高                                         |
| 扩展性   | 因ACID语义限制，Oracle数据库最多只有100台                    | 数据存于HDFS，所以扩展性与HDFS一致，2009年雅虎最多达时到4000个节点左右 |
| 数据规模 | 规模较小                                                     | 可以利用MapReduce进行并行计算，所以支持很大的规模数据        |

# 第二节、Hive的安装和配置

## 一、下载安装

### 1. 下载Hive

>   查看跟Hadoop版本对应的Hive版本并下载

### 2. 安装Hive

-   下载tar.gz包直接解压

-   下载Hive项目并自己用maven打包

### 3. 基本配置

-   配置HIVE环境变量并添加到PATH中
-   在Hive/conf下生成hive-env.sh，修改里面的HADOOP_HOME和HIVE_CONF_DIR

>   集群不是一个人使用，所以需要在HDFS中的/user/hive/warehouse和/tmp给予对应用户或组的写入权限
>
>   或者在hdfs-site.xml中关闭权限检查
>
>   ```xml
>   <property>
>   	<name>dfs.permissions.enable</name>
>   	<value>false</value>
>   </property>
>   ```

### 4. 启动Hive

1.  Hive是把数据存在HDFS，所以要先启动HDFS
2.  Hive底层是MapReduce，MapReduce是在Yarn上运行，所以要再启动Yarn
3.  启动Hive客户端

### 5. 加载本地数据到Hive

```SQL
create table xxx(xxx int,xxxx string) row format delimited fields terminated by "\t"; --  建表规定字段间分割符为缩进
load data local inpath "$PWD" into table xxx; -- 加载数据到Hive中
```

### 6. Hive元数据(MetaData)配置到MySQL

-   因为MetaData默认存在Hive自带的derby数据库中，derby是单用户数据库，从而导致同时只能一个用户登录到Hive中，现在需要安装MySQL，把MetaData存到MySQL中来解决此问题

-   在MySQL中配置user可以远程登录，使集群内的机器都能使用MySQL来存储MetaData实现多Hive客户端登录

-   驱动拷贝到Hive依赖库/hive/lib中

-   修改hive-site.xml

    ```xml
    <?xml version="1.0"?>
    <?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
    <configuration>
    	<property>
    	  <name>javax.jdo.option.ConnectionURL</name>
    	  <value>jdbc:mysql://MySQL主机名:3306/metastore?createDatabaseIfNotExist=true</value>
    	  <description>JDBC connect string for a JDBC metastore</description>
    	</property>
    
    	<property>
    	  <name>javax.jdo.option.ConnectionDriverName</name>
    	  <value>com.mysql.jdbc.Driver</value>
    	  <description>Driver class name for a JDBC metastore</description>
    	</property>
    
    	<property>
    	  <name>javax.jdo.option.ConnectionUserName</name>
    	  <value>root</value>
    	  <description>username to use against metastore database</description>
    	</property>
    
    	<property>
    	  <name>javax.jdo.option.ConnectionPassword</name>
    	  <value>你的MySQL密码</value>
    	  <description>password to use against metastore database</description>
    	</property>
    </configuration>
    ```

    ## 二、Hive的使用

    ### 1. Hive两种执行HQL方式

    1.  ```shell
        hive -e "你的HQL"
        ```

    2.  ```shell
        hive -f 你的HQL文件
        ```

    ### 2. 退出Hive
- exit&quit
    在新版的hive中没区别了，在以前的版本是有的：

    exit:先隐性提交数据，再退出；

    quit:不提交数据，退出；