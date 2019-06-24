## 二 美多业务数据导入

### 2.1 导入数据到HDFS

- 建立hdfs目录:

```shell
hadoop fs -mkdir /project2-meiduo-rs
```

- 建立以下sh文件mysqlToHDFS.sh，可将mysql中数据库meiduo_mall里指定表全部导入hdfs中：

``` shell
#!/bin/bash

array=(tb_goods tb_goods_category tb_goods_specification tb_sku tb_sku_specification tb_specification_option)

for table_name in ${array[@]};
do
    sqoop import \
        --connect jdbc:mysql://localhost/meiduo_mall \
        --username root \
        --password root \
        --table $table_name \
        --m 5 \
        --target-dir /project2-meiduo-rs/meiduo_mall/$table_name
done
```

- 执行脚本:

```shell
chmod +x mysqlToHDFS.sh
source mysqlToHDFS.sh
```

- 查看导入数据结果

```shell
hadoop fs -ls /project2-meiduo-rs/meiduo_mall
SLF4J: Class path contains multiple SLF4J bindings.
SLF4J: Found binding in [jar:file:/root/bigdata/hadoop-2.9.1/share/hadoop/common/lib/slf4j-log4j12-1.7.25.jar!/org/slf4j/impl/StaticLoggerBinder.class]
SLF4J: Found binding in [jar:file:/root/bigdata/apache-hive-2.3.4-bin/lib/log4j-slf4j-impl-2.6.2.jar!/org/slf4j/impl/StaticLoggerBinder.class]
SLF4J: See http://www.slf4j.org/codes.html#multiple_bindings for an explanation.
SLF4J: Actual binding is of type [org.slf4j.impl.Log4jLoggerFactory]
Found 6 items
drwxr-xr-x   - root supergroup          0 2018-12-19 17:30 /project2-meiduo-rs/meiduo_mall/tb_goods
drwxr-xr-x   - root supergroup          0 2018-12-19 17:30 /project2-meiduo-rs/meiduo_mall/tb_goods_category
drwxr-xr-x   - root supergroup          0 2018-12-19 17:30 /project2-meiduo-rs/meiduo_mall/tb_goods_specification
drwxr-xr-x   - root supergroup          0 2018-12-19 17:30 /project2-meiduo-rs/meiduo_mall/tb_sku
drwxr-xr-x   - root supergroup          0 2018-12-19 17:31 /project2-meiduo-rs/meiduo_mall/tb_sku_specification
drwxr-xr-x   - root supergroup          0 2018-12-19 17:31 /project2-meiduo-rs/meiduo_mall/tb_specification_option
```

- 使用spark读取数据

注意默认情况下，导入数据，每列数据之间是用逗号分隔，因此可以如同csv文件一样进行读取

```python
import os
# 配置spark driver和pyspark运行时，所使用的python解释器路径
PYSPARK_PYTHON = "/home/hadoop/miniconda3/envs/datapy365spark23/bin/python"
JAVA_HOME='/home/hadoop/app/jdk1.8.0_191'
# 当存在多个版本时，不指定很可能会导致出错
os.environ["PYSPARK_PYTHON"] = PYSPARK_PYTHON
os.environ["PYSPARK_DRIVER_PYTHON"] = PYSPARK_PYTHON
os.environ['JAVA_HOME']=JAVA_HOME
```

```python
# spark配置信息
from pyspark import SparkConf
from pyspark.sql import SparkSession

SPARK_APP_NAME = "TransferMySQLToHDFS"
SPARK_URL = "spark://192.168.199.188:7077"

conf = SparkConf()    # 创建spark config对象
config = (
	("spark.app.name", SPARK_APP_NAME),    # 设置启动的spark的app名称，没有提供，将随机产生一个名称
	("spark.executor.memory", "2g"),    # 设置该app启动时占用的内存用量，默认1g
	("spark.master", SPARK_URL),    # spark master的地址
    ("spark.executor.cores", "2"),    # 设置spark executor使用的CPU核心数
)

conf.setAll(config)

# 利用config对象，创建spark session
spark = SparkSession.builder.config(conf=conf).getOrCreate()
```

- 通过spark读取hdfs上的数据

```python
ret = spark.read.csv("hdfs://hadoop000:8020/project2-meiduo-rs/meiduo_mall/tb_goods")
ret
```

- 显示结果:

```shell
DataFrame[_c0: string, _c1: string, _c2: string, _c3: string, _c4: string, _c5: string, _c6: string, _c7: string, _c8: string, _c9: string, _c10: string]
```

- 展示数据

```python
ret.show()
ret.select("_c3", "_c10").show(100)
```

- 终端显示结果

```shell
+--------------------+--------------------+--------------------+--------------------+----+----+----+----+----+----+--------------------+
|                 _c0|                 _c1|                 _c2|                 _c3| _c4| _c5| _c6| _c7| _c8| _c9|                _c10|
+--------------------+--------------------+--------------------+--------------------+----+----+----+----+----+----+--------------------+
|                   1|2018-04-11 16:01:...|2018-04-25 12:09:...|Apple MacBook Pro...|   1|   1|   1|   4|  45| 157|<h1 style="text-a...|
|<p>它纤薄如刃，轻盈如羽，却又比...|                null|                null|                null|null|null|null|null|null|null|                null|
|<p><img alt="" sr...|                null|                null|                null|null|null|null|null|null|null|                null|
|<p><img alt="" sr...|                null|                null|                null|null|null|null|null|null|null|                null|
|<p><img alt="" sr...|                null|                null|                null|null|null|null|null|null|null|                null|
|<p><img alt="" sr...|                null|                null|                null|null|null|null|null|null|null|                null|
|<p><img alt="" sr...|                null|                null|                null|null|null|null|null|null|null|                null|
|<p><img alt="" sr...|                null|                null|                null|null|null|null|null|null|null|                null|
|       <p>&nbsp;</p>|       <h2>包装清单</h2>|                null|                null|null|null|null|null|null|null|                null|
|<p>MacBook Air 电源...|<p>&nbsp;<strong>...|                null|                null|null|null|null|null|null|null|                null|
|<p>1、Mac 电脑整机及所含附...|                null|                null|                null|null|null|null|null|null|null|                null|
|如因质量问题或故障，凭厂商维修中心...|                null|                null|                null|null|null|null|null|null|null|                null|
|(注:如厂家在商品介绍中有售后保障的说明|则此商品按照厂家说明执行售后保障服...|                null|                null|null|null|null|null|null|null|                null|
|              <br />|                null|                null|                null|null|null|null|null|null|null|                null|
|品牌官方网站：<a href="h...|                null|                null|                null|null|null|null|null|null|null|                null|
|售后服务电话：4006668800...|                null|                null|                null|null|null|null|null|null|null|                null|
|<p><strong>正品行货</...|                null|                null|                null|null|null|null|null|null|null|                null|
|<p>京东商城向您保证所售商品均为...|                null|                null|                null|null|null|null|null|null|null|                null|
|<p><strong>全国联保</...|                null|                null|                null|null|null|null|null|null|null|                null|
|<p>凭质保证书及京东商城发票，可...|                null|                null|                null|null|null|null|null|null|null|                null|
+--------------------+--------------------+--------------------+--------------------+----+----+----+----+----+----+--------------------+
only showing top 20 rows

+--------------------+--------------------+
|                 _c3|                _c10|
+--------------------+--------------------+
|Apple MacBook Pro...|<h1 style="text-a...|
|                null|                null|
|                null|                null|
|                null|                null|
|                null|                null|
|                null|                null|
|                null|                null|
|                null|                null|
|                null|                null|
|                null|                null|
|                null|                null|
|                null|                null|
|                null|                null|
|                null|                null|
|                null|                null|
|                null|                null|
|                null|                null|
|                null|                null|
|                null|                null|
|                null|                null|
|                null|                null|
|                null|                null|
| Apple iPhone 8 Plus|<p><img alt="" sr...|
|                null|                null|
|                null|                null|
|                null|                null|
|                null|                null|
|                null|                null|
|                null|                null|
|                null|                null|
|                null|                null|
|                null|                null|
|                null|                null|
|                null|                null|
|                null|                null|
|                null|                null|
|                null|                null|
|                null|                null|
|                null|                null|
|                null|                null|
|                null|                null|
|                null|                null|
|  华为 HUAWEI P10 Plus|<p><img alt="" sr...|
|                null|                null|
|                null|                null|
|                null|                null|
|                null|                null|
|                null|                null|
|                null|                null|
|                null|                null|
|                null|                null|
|                null|                null|
|                null|                null|
|                null|                null|
|                null|                null|
|                null|                null|
|                null|                null|
|                null|                null|
|                null|                null|
|【次日达】泰火薄快充电宝手机壳无线...|                null|
|GoPro hero7运动相机水下...|                null|
|川宇 USB3.0高速多功能合一T...|                null|
|腾讯听听 9420  智能音箱/音...|                null|
|苹果XS Max无线充电器iPho...|                null|
|behringer 百灵达X32数...|                null|
|ESCASE 荣耀畅玩8c手机壳 ...|                null|
|FDY FW3C 5C 蓝牙无线W...|                null|
|摩托罗拉（Motorola） 威泰...|                null|
|沃丁（WODING） 亚马逊 Am...|                null|
|科凌（keling） F8全波段收...|                null|
|科威盛（kevsen） Q3对讲机...|                null|
|小米（MI） 小米手环3代nfc版...|                null|
|摩托罗拉（Motorola） MA...|                null|
|C&C 儿童相机数码卡通照相机玩具...|                null|
|全景看房婚礼旅行！理光 THETA...|                null|
|欧兴 KLG5.0 对讲机 待机1...|                null|
|徕卡（Leica）C-LUX新款数...|                null|
|爱国者（aigo）DPF81/83...|                null|
|徕卡Leica数码相机V-LUX ...|                null|
|佳能（Canon）PowerSho...|                null|
|飚王（SSK）SCRM331二合一...|                null|
|佳能（Canon）小型数码相机 2...|                null|
|莱彩(RICH)E006儿童礼物照...|                null|
|爱国者（aigo）DPF81/83...|                null|
|震荡波（ZDB）全格式10寸数码相...|                null|
|松下（Panasonic）ZS11...|                null|
|绿巨能（llano） TF/SD读...|                null|
|绿联 多功能二合一读卡器USB3....|                null|
|乐仕泰 type-c多功能合一读卡...|                null|
|尼康（Nikon）COOLPIX ...|                null|
|飚王（SSK）SCRM060 灵动...|                null|
|BSN type-c tf卡读卡器...|                null|
|OCG 苹果6s/7/8plus高...|                null|
|绿联 USB3.0读卡器多功能合一...|                null|
|闪迪内存卡储存卡SD卡EOSM6E...|                null|
|墨一（MOYi） 苹果手机读卡器 ...|                null|
|绿巨能（llano） 多功能合一手...|                null|
|Type-C读卡器 锌合金type...|                null|
|沣标（FB） 存储卡SD卡 TF卡...|                null|
|川宇 3.0USB读卡器TF/SD...|                null|
+--------------------+--------------------+
only showing top 100 rows

```

- 经过观察发现 结果中有很多null值  但对比mysql数据库中的显示发现是加载数据有问题

![1545630559725](/img/1545630559725.png)

![1545630651922](/img/1545630651922.png)

实际是数据换行的问题 

### 2.2 导入数据到Hive

- 使用sqoop 将mysql数据导入hive

  清除注释, \后不要有空格

``` shell
#!/bin/bash

array=(tb_goods tb_goods_category tb_goods_specification tb_sku tb_sku_specification tb_specification_option)

for table_name in ${array[@]};
do
        sqoop import \
                --connect jdbc:mysql://localhost/meiduo_mall \
                --username root \
                --password root \
                --table $table_name \
                --m 5 \
                --hive-import \ #导入到hive
                --create-hive-table  \ #创建hive表
                --hive-drop-import-delims \ # 去掉多余的 制表符换行符
                --warehouse-dir /user/hive/warehouse \ # 默认路径不需要指定
                --hive-table $table_name 
done
```

- `--hive-import`就是指明往hive里面导入

  `--warehouse-dir`指明hdfs上hive数据库的路径

  重点注意`--hive-drop-import-delims`选项，会把如"\n \r"等字符自动处理掉，避免出现数据与字段不对应的问题

  加可执行权限`chmod +x mysqlToHive.sh`

  执行`source mysqlToHive.sh`

* sqoop把mysql导入hive时报错：Could not load org.apache.hadoop.hive.conf.HiveConf.

```
# vi ~/.bash_profile
export HADOOP_CLASSPATH=$HADOOP_CLASSPATH:/home/app/hive/lib/*  # hive/lib/* 的路径
# source ~/.bash_profile
```

- 查看结果

```shell
hadoop fs -ls /user/hive/warehouse
```

- 终端显示

```shell
SLF4J: Class path contains multiple SLF4J bindings.
SLF4J: Found binding in [jar:file:/root/bigdata/hadoop-2.9.1/share/hadoop/common/lib/slf4j-log4j12-1.7.25.jar!/org/slf4j/impl/StaticLoggerBinder.class]
SLF4J: Found binding in [jar:file:/root/bigdata/apache-hive-2.3.4-bin/lib/log4j-slf4j-impl-2.6.2.jar!/org/slf4j/impl/StaticLoggerBinder.class]
SLF4J: See http://www.slf4j.org/codes.html#multiple_bindings for an explanation.
SLF4J: Actual binding is of type [org.slf4j.impl.Log4jLoggerFactory]
Found 6 items
drwxr-xr-x   - root supergroup          0 2018-12-20 11:09 /user/hive/warehouse/tb_goods
drwxr-xr-x   - root supergroup          0 2018-12-20 11:10 /user/hive/warehouse/tb_goods_category
drwxr-xr-x   - root supergroup          0 2018-12-20 11:10 /user/hive/warehouse/tb_goods_specification
drwxr-xr-x   - root supergroup          0 2018-12-20 11:10 /user/hive/warehouse/tb_sku
drwxr-xr-x   - root supergroup          0 2018-12-20 11:11 /user/hive/warehouse/tb_sku_specification
drwxr-xr-x   - root supergroup          0 2018-12-20 11:12 /user/hive/warehouse/tb_specification_option

```

- Spark整合Hive

Spark SQL是兼容Hive SQL的，但为了Spark能访问Hive中的数据信息，还需要一定的配置：

1.将Hive的hive-site.xml复制到Spark的conf配置目录下：

```shell
cp /home/hadoop/app/hive-1.1.0-cdh5.7.0/conf/hive-site.xml /home/hadoop/app/spark-2.3.0-bin-2.6.0-cdh5.7.0/conf/
```

2.在spark/conf/spark-env.sh下，新增：

```shell
export HIVE_HOME=/home/hadoop/app/hive-1.1.0-cdh5.7.0/
export SPARK_CLASSPATH=$HIVE_HOME/lib/mysql-connector-java-5.1.36.jar:$SPARK_CLASSPATH
```

- pyspark中从Hive数据库进行数据加载
  - 注意配置：
    - 配置中加入("hive.metastore.uris", "thrift://localhost:9083")配置项，告诉spark使用hive已有的metastore，否则spark会尝试自己建立一个新的metastore

    - 创建SparkSession时，调用enableHiveSupport方法开启对Hive的操作接口

    - 在访问hive的元数据之前 需要打开hive的元数据服务

      ```shell
      hive --service metastore&
      ```

```python
# spark配置信息
# 注意：添加
from pyspark import SparkConf
from pyspark.sql import SparkSession

SPARK_APP_NAME = "transMeiDuoData"
SPARK_URL = "spark://192.168.199.188:7077"

conf = SparkConf()    # 创建spark config对象
config = (
	("spark.app.name", SPARK_APP_NAME),    # 设置启动的spark的app名称，没有提供，将随机产生一个名称
	("spark.executor.memory", "2g"),    # 设置该app启动时占用的内存用量，默认1g
	("spark.master", SPARK_URL),    # spark master的地址
    ("spark.executor.cores", "2"),    # 设置spark executor使用的CPU核心数
    ("hive.metastore.uris", "thrift://localhost:9083"),    # 配置hive元数据的访问，否则spark无法获取hive中已存储的数据
)
# 查看更详细配置及说明：https://spark.apache.org/docs/latest/configuration.html

conf.setAll(config)

# 利用config对象，创建spark session
spark = SparkSession.builder.config(conf=conf).enableHiveSupport().getOrCreate()
```

- spark列出所有的数据库

```python
spark.catalog.listDatabases()
```

- 显示结果

```shell
[Database(name='default', description='Default Hive database', locationUri='hdfs://0.0.0.0:9000/user/hive/warehouse')]
```

- spark列出所有的表

```python
spark.catalog.listTables("default")
```

- 显示结果

```shell
[Table(name='tb_goods', database='default', description='Imported by sqoop on 2018/11/20 11:09:54', tableType='MANAGED', isTemporary=False),
 Table(name='tb_goods_category', database='default', description='Imported by sqoop on 2018/11/20 11:10:11', tableType='MANAGED', isTemporary=False),
 Table(name='tb_goods_specification', database='default', description='Imported by sqoop on 2018/11/20 11:10:30', tableType='MANAGED', isTemporary=False),
 Table(name='tb_sku', database='default', description='Imported by sqoop on 2018/11/20 11:10:54', tableType='MANAGED', isTemporary=False),
 Table(name='tb_sku_specification', database='default', description='Imported by sqoop on 2018/11/20 11:11:28', tableType='MANAGED', isTemporary=False),
 Table(name='tb_specification_option', database='default', description='Imported by sqoop on 2018/11/20 11:12:04', tableType='MANAGED', isTemporary=False)]
```

- spark列出表中所有的结果

```python
spark.sql("select * from tb_goods").show()  # DataFrame
```

- 显示结果

```shell
+---+--------------------+--------------------+--------------------+-----+--------+--------+------------+------------+------------+--------------------+--------------------+--------------------+
| id|         create_time|         update_time|                name|sales|comments|brand_id|category1_id|category2_id|category3_id|         desc_detail|           desc_pack|        desc_service|
+---+--------------------+--------------------+--------------------+-----+--------+--------+------------+------------+------------+--------------------+--------------------+--------------------+
|  1|2018-04-11 16:01:...|2018-04-25 12:09:...|Apple MacBook Pro...|    1|       1|       1|           4|          45|         157|<h1 style="text-a...|<h2>包装清单</h2><p>M...|<p>&nbsp;<strong>...|
|  2|2018-04-14 02:09:...|2018-04-25 11:51:...| Apple iPhone 8 Plus|    3|       1|       1|           1|          38|         115|<p><img alt="" sr...|<h3>包装清单</h3><p>采...|<p>&nbsp;<strong>...|
|  3|2018-04-14 03:03:...|2018-04-25 11:51:...|  华为 HUAWEI P10 Plus|    1|       8|       2|           1|          38|         115|<p><img alt="" sr...|<h3>包装清单</h3><p>手...|<p>&nbsp;<strong>...|
|  4|2018-11-01 10:43:...|2018-11-01 10:43:...|【次日达】泰火薄快充电宝手机壳无线...|    0|       0|       1|           1|          39|         126|                    |                    |                    |
|  5|2018-11-01 10:43:...|2018-11-01 10:43:...|GoPro hero7运动相机水下...|    0|       0|       1|           2|          40|         132|                    |                    |                    |
|  6|2018-11-01 10:43:...|2018-11-01 10:43:...|川宇 USB3.0高速多功能合一T...|    0|       0|       1|           3|          41|         140|                    |                    |                    |
|  7|2018-11-01 10:43:...|2018-11-01 10:43:...|腾讯听听 9420  智能音箱/音...|    0|       0|       1|           3|          42|         143|                    |                    |                    |
|  8|2018-11-01 10:44:...|2018-11-01 10:44:...|苹果XS Max无线充电器iPho...|    0|       0|       1|           1|          39|         123|                    |                    |                    |
|  9|2018-11-01 10:44:...|2018-11-01 10:44:...|behringer 百灵达X32数...|    0|       0|       1|           3|          42|         146|                    |                    |                    |
| 10|2018-11-01 10:44:...|2018-11-01 10:44:...|ESCASE 荣耀畅玩8c手机壳 ...|    0|       0|       1|           1|          39|         119|                    |                    |                    |
| 11|2018-11-01 10:44:...|2018-11-01 10:44:...|FDY FW3C 5C 蓝牙无线W...|    0|       0|       1|           3|          42|         143|                    |                    |                    |
| 12|2018-11-01 10:44:...|2018-11-01 10:44:...|摩托罗拉（Motorola） 威泰...|    0|       0|       1|           1|          38|         118|                    |                    |                    |
| 13|2018-11-01 10:44:...|2018-11-01 10:44:...|沃丁（WODING） 亚马逊 Am...|    0|       0|       1|           3|          42|         143|                    |                    |                    |
| 14|2018-11-01 10:44:...|2018-11-01 10:44:...|科凌（keling） F8全波段收...|    0|       0|       1|           3|          42|         144|                    |                    |                    |
| 15|2018-11-01 10:44:...|2018-11-01 10:44:...|科威盛（kevsen） Q3对讲机...|    0|       0|       1|           1|          38|         118|                    |                    |                    |
| 16|2018-11-01 10:44:...|2018-11-01 10:44:...|小米（MI） 小米手环3代nfc版...|    0|       0|       1|           3|          43|         147|                    |                    |                    |
| 17|2018-11-01 10:44:...|2018-11-01 10:44:...|摩托罗拉（Motorola） MA...|    0|       0|       1|           1|          38|         118|                    |                    |                    |
| 18|2018-11-01 10:44:...|2018-11-01 10:44:...|C&C 儿童相机数码卡通照相机玩具...|    0|       0|       1|           2|          40|         128|                    |                    |                    |
| 19|2018-11-01 10:44:...|2018-11-01 10:44:...|全景看房婚礼旅行！理光 THETA...|    0|       0|       1|           2|          40|         128|                    |                    |                    |
| 20|2018-11-01 10:44:...|2018-11-01 10:44:...|欧兴 KLG5.0 对讲机 待机1...|    0|       0|       1|           1|          38|         118|                    |                    |                    |
+---+--------------------+--------------------+--------------------+-----+--------+--------+------------+------------+------------+--------------------+--------------------+--------------------+
only showing top 20 rows
```

- spark查询hive数据

```python
spark.sql("select * from tb_goods").select("name", "desc_detail").show()
```

- 显示结果

```shell
+--------------------+--------------------+
|                name|         desc_detail|
+--------------------+--------------------+
|Apple MacBook Pro...|<h1 style="text-a...|
| Apple iPhone 8 Plus|<p><img alt="" sr...|
|  华为 HUAWEI P10 Plus|<p><img alt="" sr...|
|【次日达】泰火薄快充电宝手机壳无线...|                    |
|GoPro hero7运动相机水下...|                    |
|川宇 USB3.0高速多功能合一T...|                    |
|腾讯听听 9420  智能音箱/音...|                    |
|苹果XS Max无线充电器iPho...|                    |
|behringer 百灵达X32数...|                    |
|ESCASE 荣耀畅玩8c手机壳 ...|                    |
|FDY FW3C 5C 蓝牙无线W...|                    |
|摩托罗拉（Motorola） 威泰...|                    |
|沃丁（WODING） 亚马逊 Am...|                    |
|科凌（keling） F8全波段收...|                    |
|科威盛（kevsen） Q3对讲机...|                    |
|小米（MI） 小米手环3代nfc版...|                    |
|摩托罗拉（Motorola） MA...|                    |
|C&C 儿童相机数码卡通照相机玩具...|                    |
|全景看房婚礼旅行！理光 THETA...|                    |
|欧兴 KLG5.0 对讲机 待机1...|                    |
+--------------------+--------------------+
only showing top 20 rows
```

### 2.3 Spark直接访问MySQL数据库

- spark可以直接访问MySQL业务数据库，获取数据，但是这样可能会导致业务数据库压力比较大，数据量较大时，不推荐这样处理

```python
import os
# 配置spark driver和pyspark运行时，所使用的python解释器路径
PYSPARK_PYTHON = "/home/hadoop/miniconda3/bin/python3"
JAVA_HOME='/home/hadoop/app/jdk1.8.0_181'
# 当存在多个版本时，不指定很可能会导致出错
os.environ["PYSPARK_PYTHON"] = PYSPARK_PYTHON
os.environ["PYSPARK_DRIVER_PYTHON"] = PYSPARK_PYTHON
os.environ['JAVA_HOME']=JAVA_HOME

# 把mysql-connector-java-5.1.36.jar放入指定目录下
# 如果是使用spark直连MySQL数据库，读取数据，需要使用mysql链接的jar包，这里预先下载并放在了指定目录
os.environ["PYSPARK_SUBMIT_ARGS"] = "--jars /home/hadoop/app/hive-1.1.0-cdh5.7.0/lib/mysql-connector-java-5.1.36.jar pyspark-shell"
```

- 直连访问mysql读取数据
  - 这种方式数据没有经过hdfs，而是直接在MySQL数据库上进行操作，因此如果数据量较大，后续操作会对MySQL产生较大压力，使用时一定要注意对mysql数据带来的影响

```python
ret = spark.read.format("jdbc").options(url="jdbc:mysql://localhost/meiduo_mall",
                                       driver="com.mysql.jdbc.Driver",
                                       dbtable="tb_goods",
                                        user="root",
                                       password="root").load()

ret
```

- 显示结果

```shell
DataFrame[id: int, create_time: timestamp, update_time: timestamp, name: string, sales: int, comments: int, brand_id: int, category1_id: int, category2_id: int, category3_id: int, desc_detail: string, desc_pack: string, desc_service: string]
```

```python
ret.show()
```

- 显示结果

```shell
+---+--------------------+--------------------+--------------------+-----+--------+--------+------------+------------+------------+--------------------+--------------------+--------------------+
| id|         create_time|         update_time|                name|sales|comments|brand_id|category1_id|category2_id|category3_id|         desc_detail|           desc_pack|        desc_service|
+---+--------------------+--------------------+--------------------+-----+--------+--------+------------+------------+------------+--------------------+--------------------+--------------------+
|  1|2018-04-11 16:01:...|2018-04-25 12:09:...|Apple MacBook Pro...|    1|       1|       1|           4|          45|         157|<h1 style="text-a...|<h2>包装清单</h2>

...|<p>&nbsp;<strong>...|
|  2|2018-04-14 02:09:...|2018-04-25 11:51:...| Apple iPhone 8 Plus|    3|       1|       1|           1|          38|         115|<p><img alt="" sr...|<h3>包装清单</h3>

...|<p>&nbsp;<strong>...|
|  3|2018-04-14 03:03:...|2018-04-25 11:51:...|  华为 HUAWEI P10 Plus|    1|       8|       2|           1|          38|         115|<p><img alt="" sr...|<h3>包装清单</h3>

...|<p>&nbsp;<strong>...|
|  4| 2018-11-01 10:43:14| 2018-11-01 10:43:14|【次日达】泰火薄快充电宝手机壳无线...|    0|       0|       1|           1|          39|         126|                    |                    |                    |
|  5| 2018-11-01 10:43:25| 2018-11-01 10:43:25|GoPro hero7运动相机水下...|    0|       0|       1|           2|          40|         132|                    |                    |                    |
|  6| 2018-11-01 10:43:58| 2018-11-01 10:43:58|川宇 USB3.0高速多功能合一T...|    0|       0|       1|           3|          41|         140|                    |                    |                    |
|  7| 2018-11-01 10:43:59| 2018-11-01 10:43:59|腾讯听听 9420  智能音箱/音...|    0|       0|       1|           3|          42|         143|                    |                    |                    |
|  8| 2018-11-01 10:44:00| 2018-11-01 10:44:00|苹果XS Max无线充电器iPho...|    0|       0|       1|           1|          39|         123|                    |                    |                    |
|  9| 2018-11-01 10:44:05| 2018-11-01 10:44:05|behringer 百灵达X32数...|    0|       0|       1|           3|          42|         146|                    |                    |                    |
| 10| 2018-11-01 10:44:13| 2018-11-01 10:44:13|ESCASE 荣耀畅玩8c手机壳 ...|    0|       0|       1|           1|          39|         119|                    |                    |                    |
| 11| 2018-11-01 10:44:11| 2018-11-01 10:44:11|FDY FW3C 5C 蓝牙无线W...|    0|       0|       1|           3|          42|         143|                    |                    |                    |
| 12| 2018-11-01 10:44:12| 2018-11-01 10:44:12|摩托罗拉（Motorola） 威泰...|    0|       0|       1|           1|          38|         118|                    |                    |                    |
| 13| 2018-11-01 10:44:14| 2018-11-01 10:44:14|沃丁（WODING） 亚马逊 Am...|    0|       0|       1|           3|          42|         143|                    |                    |                    |
| 14| 2018-11-01 10:44:16| 2018-11-01 10:44:16|科凌（keling） F8全波段收...|    0|       0|       1|           3|          42|         144|                    |                    |                    |
| 15| 2018-11-01 10:44:25| 2018-11-01 10:44:25|科威盛（kevsen） Q3对讲机...|    0|       0|       1|           1|          38|         118|                    |                    |                    |
| 16| 2018-11-01 10:44:24| 2018-11-01 10:44:24|小米（MI） 小米手环3代nfc版...|    0|       0|       1|           3|          43|         147|                    |                    |                    |
| 17| 2018-11-01 10:44:25| 2018-11-01 10:44:25|摩托罗拉（Motorola） MA...|    0|       0|       1|           1|          38|         118|                    |                    |                    |
| 18| 2018-11-01 10:44:29| 2018-11-01 10:44:29|C&C 儿童相机数码卡通照相机玩具...|    0|       0|       1|           2|          40|         128|                    |                    |                    |
| 19| 2018-11-01 10:44:30| 2018-11-01 10:44:30|全景看房婚礼旅行！理光 THETA...|    0|       0|       1|           2|          40|         128|                    |                    |                    |
| 20| 2018-11-01 10:44:29| 2018-11-01 10:44:29|欧兴 KLG5.0 对讲机 待机1...|    0|       0|       1|           1|          38|         118|                    |                    |                    |
+---+--------------------+--------------------+--------------------+-----+--------+--------+------------+------------+------------+--------------------+--------------------+--------------------+
only showing top 20 rows
```

