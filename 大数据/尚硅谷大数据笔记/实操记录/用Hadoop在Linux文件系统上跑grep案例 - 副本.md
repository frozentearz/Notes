#### 官方 grep 案例

0. 不需要配置文件，使用默认配置

1. 在 hadoop 安装路径下创建一个`input`文件夹

    ```shell
    mkdir input
    ```

2. 将 hadoop 的 xml 文件复制到 input (XML在 *recourse/_conf* 文件夹下)

    ```shell
    cp /etc/hadoop*.xml input
    ```

3. 执行 share 目录下的 MapReduce 程序

    ```shell
    cd bin
    hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples.2.7.2.jar grep input output 'dfs[a-z.]+'
    ```

4. 查看输出

    ```shell
    cat output/*
    ```

---

思考与问题：

