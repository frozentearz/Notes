#### Map把数据拆分后对数据进行并行处理，是Map来处理还是会交给其他组件计算

>   Map 处理，因为 Map 本身就是一个计算框架

#### 处理完的数据Reduce是从哪里获取到的呢？Map还是其他组件？

>   Map

#### 单节点在不改dfs.replication的情况下会在本地保存三份数据吗？为什么？

>   //TODO 待实操

#### 如果先启动nodemanager再启动resourcemanager会报错吗？

>   //TODO 待实操

#### 为什么HDFS的块的大小不能设置太大也不能设置太小

>   -   太小
>       -   会增加寻址读取开销
>
>   -   太大
>       -   会让HDFS重启变得很慢
>       -   会增加MapReduce的xxx(待补充)

#### 为什么hadoop2.x版本的块大小默认设置为128M？

>   1.  寻址时间固定为10ms
>   2.  寻址时间为传输时间的1%为最佳状态，所以 10ms/0.01=1s 为最佳传输时间
>   3.  而目前的磁盘传输速率普遍是100M/s
>   4.  最佳为每一秒传输一个块，则最佳块大小为100M，取整得128M

#### hadoop-fs、hadoop-dfs与hdfs-dfs命令的区别是什么？

>   -   hadoop fs 适用范围最广，可以操作HDFS、本地文件系统、HFTP等
>   -   hadoop dfs 用于操作HDFS，但是已经不推荐使用
>   -   hdfs dfs 用于操作HDFS，推荐使用

#### edits.inprogress是namenode启动之前存在吗？

> 启动后创建的，最新记录的操作日志