# IoTDB集成EMR Spark

### 测试环境：

emr 5.33.1
Spark 2.4.7
openjdk 1.8.0_312
IoTDB 0.12.4

### 一、安装IoTDB，编译相关依赖包

编译打包iotdb-jdbc-0.13.0-SNAPSHOT-jar-with-dependencies.jar和spark-iotdb-connector-0.12.4.jar依赖包

```
git clone https://github.com/apache/iotdb.git
cd iotdb/spark-iotdb-connector
mvn clean scala:compile compile install
cd ../jdbc
mvn clean package -DskipTests -Pget-jar-with-dependencies
```

在target目录下，复制iotdb-jdbc-0.13.0-SNAPSHOT-jar-with-dependencies.jar和spark-iotdb-connector-0.12.4.jar待用

下载IoTDB 二进制Release,启动IoTDB server

```
wget https://www.apache.org/dyn/closer.cgi/iotdb/0.12.4/apache-iotdb-0.12.4-all-bin.zip
unzip [apache-iotdb-0.12.4-all-bin.zip](https://www.apache.org/dyn/closer.cgi/iotdb/0.12.4/apache-iotdb-0.12.4-all-bin.zip)
cd apache-iotdb-0.12.4-all-bin/
`nohup sbin/start-server.sh >/dev/null 2>&1 &
cd lib`
```

复制libthrift-0.14.1.jar待用

IoTDB示例数据创建参考官方的[快速上手](https://iotdb.apache.org/zh/UserGuide/Master/QuickStart/QuickStart.html)

### 二、配置EMR Spark，加载IoTDB数据

创建EMR 5.33.1集群，过程略。
用hadoop用户ssh登录emr spark

复制iotdb-jdbc-0.13.0-SNAPSHOT-jar-with-dependencies.jar和spark-iotdb-connector-0.12.4.jar到/usr/lib/spark/jars

默认EMR Spark 2.4.7的libthrift版本是0.9.3, 需要用IoTDB目录下lib目录下的libthrift-0.14.1.jar替换

```
sudo cp iotdb-jdbc-0.13.0-SNAPSHOT-jar-with-dependencies.jar /usr/lib/spark/jars/
sudo cp spark-iotdb-connector-0.12.4.jar /usr/lib/spark/jars/
sudo cp libthrift-0.14.1.jar /usr/lib/spark/jars/
cd /usr/lib/spark/jars/
sudo mv libthrift-0.9.3.jar bak.libthrift-0.9.3.jar
spark-shell 
scala> val df = spark.read.format("org.apache.iotdb.spark.db").option("url","jdbc:iotdb://172.31.2.191:6667/").option("sql","SELECT * FROM root.ln.wf01.wt01").load
df: org.apache.spark.sql.DataFrame = [Time: bigint, root.ln.wf01.wt01.temperature: float ... 1 more field]

scala> df.show()
+----+-----------------------------+------------------------+                   
|Time|root.ln.wf01.wt01.temperature|root.ln.wf01.wt01.status|
+----+-----------------------------+------------------------+
| 100|                         null|                    true|
| 200|                        20.71|                   false|
+----+-----------------------------+------------------------+
```




