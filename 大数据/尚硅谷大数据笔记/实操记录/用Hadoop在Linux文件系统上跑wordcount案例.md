#### 官方 wordcount 案例

0. 不需要配置文件，使用默认配置

1. 准备数据

    ```shell
    mkdir wcinput
    echo "xiaoming xiaoming xiaohong xiaobai xiaobai xiaobai" > ./wcinput/wc.input
    ```

2. 执行任务

    ```shell
    hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples.2.7.2.jar wordcount wcinput/wc.input wcoutput
    ```

3. 查看输出

    ```shell
    cat wcoutput/*
    ```

---

思考与问题：

