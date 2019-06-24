## 三 商品数据处理

### 3.1 数据处理目标

- 处理目标: 将要用来提取关键词(标签)的数据全部整合到一张表中, 将电商产品的所有详细描述组合成一个文本
- 数据整合目的: 对电商类型的产品进行关键词提取，如TF·IDF、TextRank等等，又或者直接进行分词

### 3.2 查看当前商品元数据表

- 查看数据库中的表

```python
spark.catalog.listTables("default")
```

- 显示结果

```shell
[Table(name='hive_sku_detail', database='default', description=None, tableType='MANAGED', isTemporary=False),
 Table(name='sku_detail', database='default', description=None, tableType='MANAGED', isTemporary=False),
 Table(name='tb_goods', database='default', description='Imported by sqoop on 2018/11/20 11:09:54', tableType='MANAGED', isTemporary=False),
 Table(name='tb_goods_category', database='default', description='Imported by sqoop on 2018/11/20 11:10:11', tableType='MANAGED', isTemporary=False),
 Table(name='tb_goods_specification', database='default', description='Imported by sqoop on 2018/11/20 11:10:30', tableType='MANAGED', isTemporary=False),
 Table(name='tb_sku', database='default', description='Imported by sqoop on 2018/11/20 11:10:54', tableType='MANAGED', isTemporary=False),
 Table(name='tb_sku_specification', database='default', description='Imported by sqoop on 2018/11/20 11:11:28', tableType='MANAGED', isTemporary=False),
 Table(name='tb_specification_option', database='default', description='Imported by sqoop on 2018/11/20 11:12:04', tableType='MANAGED', isTemporary=False)]
```

- 查看表中的信息

```python
print("tb_goods表：")
spark.sql("select count(*) tb_goods from tb_goods").show()
```

- 显示结果

```shell
tb_goods表：
+--------+
|tb_goods|
+--------+
|  326160|
+--------+
```

- 查询tb_goods表中的内容

``` python
spark.sql("select * from tb_goods").show()
```

- 显示结果

``` shell
+---+--------------------+--------------------+--------------------+-----+--------+--------+------------+------------+------------+--------------------+--------------------+--------------------+
| id|         create_time|         update_time|                name|sales|comments|brand_id|category1_id|category2_id|category3_id|         desc_detail|           desc_pack|        desc_service|
+---+--------------------+--------------------+--------------------+-----+--------+--------+------------+------------+------------+--------------------+--------------------+--------------------+
|  1|2018-04-11 16:01:...|2018-04-25 12:09:...|Apple MacBook Pro...|    1|       1|       1|           4|          45|         157|<h1 style="text-a...|<h2>包装清单</h2><p>M...|<p>&nbsp;<strong>...|
|  2|2018-04-14 02:09:...|2018-04-25 11:51:...| Apple iPhone 8 Plus|    3|       1|       1|           1|          38|         115|<p><img alt="" sr...|<h3>包装清单</h3><p>采...|<p>&nbsp;<strong>...|
|  3|2018-04-14 03:03:...|2018-04-25 11:51:...|  华为 HUAWEI P10 Plus|    1|       8|       2|           1|          38|         115|<p><img alt="" sr...|<h3>包装清单</h3><p>手...|<p>&nbsp;<strong>...|
|  4|2018-11-01 10:43:...|2018-11-01 10:43:...|【次日达】泰火薄快充电宝手机壳无线...|    0|       0|       1|           1|          39|         126|                    |                    |                    |
|  5|2018-11-01 10:43:...|2018-11-01 10:43:...|GoPro hero7运动相机水下...|    0|       0|       1|           2|          40|         132|                    |                    |   
+---+--------------------+--------------------+--------------------+-----+--------+--------+------------+------------+------------+--------------------+--------------------+--------------------+
only showing top 20 rows
```

- 查询tb_goods_category表

```python
print("tb_goods_category表：")
spark.sql("select count(*) tb_goods_category from tb_goods_category").show()
```

- 显示表中数据

```shell
tb_goods_category表：
+-----------------+
|tb_goods_category|
+-----------------+
|              544|
+-----------------+
```

- 查询tb_goods_category表中内容

```python
spark.sql("select * from tb_goods_category").show()
```

- 显示表中数据

```python
+---+--------------------+--------------------+----+---------+
| id|         create_time|         update_time|name|parent_id|
+---+--------------------+--------------------+----+---------+
|  1|2018-04-09 08:03:...|2018-04-09 08:03:...|  手机|     null|
|  2|2018-04-09 08:04:...|2018-04-09 08:04:...|  相机|     null|
|  3|2018-04-09 08:04:...|2018-04-09 08:04:...|  数码|     null|
|  4|2018-04-09 08:05:...|2018-04-09 08:05:...|  电脑|     null|
|  5|2018-04-09 08:05:...|2018-04-09 08:05:...|  办公|     null|
+---+--------------------+--------------------+----+---------+
only showing top 20 rows
```

- 查询tb_goods_specification表数据数量

``` python
print("tb_goods_specification表：")
spark.sql("select count(*) tb_goods_specification from tb_goods_specification").show()
```

- 显示结果

```shell
tb_goods_specification表：
+----------------------+
|tb_goods_specification|
+----------------------+
|                368959|
+----------------------+
```

- 查询tb_goods_specification表中内容

```python
spark.sql("select * from tb_goods_specification").show()
```

- 显示查询结果

``` shell
+---+--------------------+--------------------+----+--------+
| id|         create_time|         update_time|name|goods_id|
+---+--------------------+--------------------+----+--------+
|  1|2018-04-11 17:20:...|2018-04-11 17:20:...|屏幕尺寸|       1|
|  2|2018-04-11 17:21:...|2018-04-11 17:21:...|  颜色|       1|
|  3|2018-04-11 17:22:...|2018-04-11 17:22:...|  版本|       1|
|  4|2018-04-14 02:10:...|2018-04-14 02:10:...|  颜色|       2|
|  5|2018-04-14 02:10:...|2018-04-14 02:10:...|  内存|       2|
+---+--------------------+--------------------+----+--------+
only showing top 5 rows
```

- 查询tb_sku_specification表数据数量

```python
print("tb_sku_specification表：")
spark.sql("select count(*) tb_sku_specification from tb_sku_specification").show()
```

- 显示结果

```shell
tb_sku_specification表：
+--------------------+
|tb_sku_specification|
+--------------------+
|             2616364|
+--------------------+
```

- 查询tb_sku_specification表的详细内容

```python
spark.sql("select * from tb_sku_specification").show()
```

- 显示结果

```shell
+---+--------------------+--------------------+---------+------+-------+
| id|         create_time|         update_time|option_id|sku_id|spec_id|
+---+--------------------+--------------------+---------+------+-------+
|  1|2018-04-11 17:53:...|2018-04-11 17:53:...|        1|     1|      1|
|  2|2018-04-11 17:56:...|2018-04-11 17:56:...|        4|     1|      2|
|  3|2018-04-11 17:56:...|2018-04-11 17:56:...|        7|     1|      3|
|  4|2018-04-12 07:11:...|2018-04-12 07:11:...|        1|     2|      1|
|  5|2018-04-12 07:11:...|2018-04-12 07:11:...|        3|     2|      2|
+---+--------------------+--------------------+---------+------+-------+
only showing top 5 rows
```

- 查询tb_sku表数据数量

```python
print("tb_sku表：")
spark.sql("select count(*) tb_sku from tb_sku").show()
```

- 显示结果

``` shell
tb_sku表：
+------+
|tb_sku|
+------+
|326173|
+------+
```

- 查询tb_sku表的详细内容

```python
spark.sql("select * from tb_sku").show()
```

- 显示查询结果

```shell
+---+--------------------+--------------------+--------------------+--------------------+-------+----------+------------+-----+-----+--------+-----------+-----------+--------+--------------------+
| id|         create_time|         update_time|                name|             caption|  price|cost_price|market_price|stock|sales|comments|is_launched|category_id|goods_id|   default_image_url|
+---+--------------------+--------------------+--------------------+--------------------+-------+----------+------------+-----+-----+--------+-----------+-----------+--------+--------------------+
|  1|2018-04-11 17:28:...|2018-04-25 11:09:...|Apple MacBook Pro...|【全新2017款】MacBook ...|11388.0|   10350.0|     13388.0|    5|    5|       1|       true|        157|       1|http://image.meid...|
|  2|2018-04-12 06:53:...|2018-04-23 11:44:...|Apple MacBook Pro...|【全新2017款】MacBook ...|11398.0|   10388.0|     13398.0|    0|    1|       0|       true|        157|       1|http://image.meid...|
|  3|2018-04-14 02:14:...|2018-04-14 17:26:...|Apple iPhone 8 Pl...|选【移动优惠购】新机配新卡，198...| 6499.0|    6300.0|      6598.0|   10|    0|       0|       true|        115|       2|http://image.meid...|
|  4|2018-04-14 02:20:...|2018-04-14 17:27:...|Apple iPhone 8 Pl...|选【移动优惠购】新机配新卡，198...| 7988.0|    7888.0|      8088.0|    8|    2|       0|       true|        115|       2|http://image.meid...|
|  5|2018-04-14 02:45:...|2018-04-14 17:27:...|Apple iPhone 8 Pl...|选【移动优惠购】新机配新卡，198...| 6688.0|    6588.0|      6788.0|   10|    0|       0|       true|        115|       2|http://image.meid...|
```

- 查询tb_specification_option表的数据数量

```python
print("tb_specification_option表：")
spark.sql("select count(*) tb_specification_option from tb_specification_option").show()
```

- 显示结果

``` shell
tb_specification_option表：
+-----------------------+
|tb_specification_option|
+-----------------------+
|                2616353|
+-----------------------+
```

- 查询tb_specification_option表的详细内容

```python
spark.sql("select * from tb_specification_option").show()
```

- 显示查询结果

```shell
+---+--------------------+--------------------+--------------------+-------+
| id|         create_time|         update_time|               value|spec_id|
+---+--------------------+--------------------+--------------------+-------+
|  1|2018-04-11 17:22:...|2018-04-11 17:22:...|              13.3英寸|      1|
|  2|2018-04-11 17:24:...|2018-04-11 17:24:...|              15.4英寸|      1|
|  3|2018-04-11 17:24:...|2018-04-11 17:24:...|                 深灰色|      2|
|  4|2018-04-11 17:24:...|2018-04-11 17:24:...|                  银色|      2|
|  5|2018-04-11 17:25:...|2018-04-11 17:25:...| core i5/8G内存/256G存储|      3|
|  6|2018-04-11 17:25:...|2018-04-11 17:25:...| core i5/8G内存/128G存储|      3|
|  7|2018-04-11 17:25:...|2018-04-12 07:12:...| core i5/8G内存/512G存储|      3|
|  8|2018-04-14 02:11:...|2018-04-14 02:11:...|                  金色|      4|
|  9|2018-04-14 02:11:...|2018-04-14 02:11:...|                 深空灰|      4|
| 10|2018-04-14 02:11:...|2018-04-14 02:11:...|                  银色|      4|
+---+--------------------+--------------------+--------------------+-------+
only showing top 10 rows
```

### 3.3 数据需求分析

- 用到的表字段情况

- **tb_goods**            **tb_goods_category **   **tb_goods_specification**   **tb_sku_specification**   **tb_sku**               

  id                         id(分类ID)                       id(属性ID)                              id                                     id

  create_time       create_time                  create_time                            create_time                 create_time

  update_time      update_time                update_time                          update_time               update_time               

   name                 **name（分类名字）    name（属性名字）**              option_id                      **name**

  sales                   parent_id                      goods_id                                 sku_id                          **caption**

  comments                                                                                                spec_id                         **price**

  brand_id                                                                                                                                       cost_price

  **category1_id**                                        **tb_specification_option**                                        market_price

  **category2_id**                                                id                                                                             stock

  **category3_id**                                             value                                                                          sales

  desc_detail                                               spec_id                                                                       comments

  desc_pack                                                                                                                                     is_launched

  desc_service                                                                                                                                category_id

  ​                                                                                                                                                      **goods_id**

- 目标：以tb_sku表为基础，将其他信息合并

  合并后保留字段：sku_id | name | caption | category1_id | category2_id | category3_id | price | cost_price | market_price | specification | category1 | category2 | category3 

  - sku_id | name | caption | price | cost_price | market_price 来自tb_sku表
  - category1_id | category2_id | category3_id 来自tb_goods表
  - category1 | category2 | category3 根据tb_goods表中的类别ID对应tb_goods_category表获得
  - specification字段将以tb_sku_specifition表对照tb_goods_specification表和tb_specification_option表获得

  这里商品推荐将以SKU为最小单位，如果以GOODS为最小单位，那么应当以tb_goods表为基础进行处理

### 3.4 连接tb_sku表和tb_goods表

- sku和1、2、3级分类合并

``` python
sql = '''
select id sku_id, name, caption, price, cost_price, market_price, goods_id from tb_sku
'''
# 选取tb_sku表中需要保留的数据
spark.sql(sql).show()
```

- 显示结果

``` shell
+------+--------------------+--------------------+-------+----------+------------+--------+
|sku_id|                name|             caption|  price|cost_price|market_price|goods_id|
+------+--------------------+--------------------+-------+----------+------------+--------+
|     1|Apple MacBook Pro...|【全新2017款】MacBook ...|11388.0|   10350.0|     13388.0|       1|
|     2|Apple MacBook Pro...|【全新2017款】MacBook ...|11398.0|   10388.0|     13398.0|       1|
|     3|Apple iPhone 8 Pl...|选【移动优惠购】新机配新卡，198...| 6499.0|    6300.0|      6598.0|       2|
|     4|Apple iPhone 8 Pl...|选【移动优惠购】新机配新卡，198...| 7988.0|    7888.0|      8088.0|       2|
|     5|Apple iPhone 8 Pl...|选【移动优惠购】新机配新卡，198...| 6688.0|    6588.0|      6788.0|       2|
|     6|Apple iPhone 8 Pl...|选【移动优惠购】新机配新卡，198...| 7988.0|    7888.0|      7988.0|       2|
|     7|Apple iPhone 8 Pl...|选【移动优惠购】新机配新卡，198...| 6688.0|    6588.0|      6788.0|       2|
|     8|Apple iPhone 8 Pl...|选【移动优惠购】新机配新卡，198...| 7988.0|    7888.0|      7988.0|       2|
|     9|华为 HUAWEI P10 Plu...|wifi双天线设计！徕卡人像摄影！...| 3388.0|    3288.0|      3388.0|       3|
|    10|华为 HUAWEI P10 Plu...|wifi双天线设计！徕卡人像摄影！...| 3788.0|    3588.0|      3888.0|       3|
+------+--------------------+--------------------+-------+----------+------------+--------+
only showing top 10 rows
```

- 连接tb_sku和tb_goods，匹配每一个sku对应1、2、3级分类

``` python
sql2 = '''
select a.id sku_id, name, caption, price, cost_price, market_price, goods_id, category1_id, category2_id, category3_id 
from 
tb_sku as a
join
(select id, category1_id, category2_id, category3_id from tb_goods) as b
where a.goods_id=b.id
'''
spark.sql(sql2).show()
```

- 显示结果

```shell
+------+--------------------+--------------------+-------+----------+------------+--------+------------+------------+------------+
|sku_id|                name|             caption|  price|cost_price|market_price|goods_id|category1_id|category2_id|category3_id|
+------+--------------------+--------------------+-------+----------+------------+--------+------------+------------+------------+
|     1|Apple MacBook Pro...|【全新2017款】MacBook ...|11388.0|   10350.0|     13388.0|       1|           4|          45|         157|
|     2|Apple MacBook Pro...|【全新2017款】MacBook ...|11398.0|   10388.0|     13398.0|       1|           4|          45|         157|
|     3|Apple iPhone 8 Pl...|选【移动优惠购】新机配新卡，198...| 6499.0|    6300.0|      6598.0|       2|           1|          38|         115|
|     4|Apple iPhone 8 Pl...|选【移动优惠购】新机配新卡，198...| 7988.0|    7888.0|      8088.0|       2|           1|          38|         115|
|     5|Apple iPhone 8 Pl...|选【移动优惠购】新机配新卡，198...| 6688.0|    6588.0|      6788.0|       2|           1|          38|         115|
|     6|Apple iPhone 8 Pl...|选【移动优惠购】新机配新卡，198...| 7988.0|    7888.0|      7988.0|       2|           1|          38|         115|
|     7|Apple iPhone 8 Pl...|选【移动优惠购】新机配新卡，198...| 6688.0|    6588.0|      6788.0|       2|           1|          38|         115|
|     8|Apple iPhone 8 Pl...|选【移动优惠购】新机配新卡，198...| 7988.0|    7888.0|      7988.0|       2|           1|          38|         115|
|     9|华为 HUAWEI P10 Plu...|wifi双天线设计！徕卡人像摄影！...| 3388.0|    3288.0|      3388.0|       3|           1|          38|         115|
|    10|华为 HUAWEI P10 Plu...|wifi双天线设计！徕卡人像摄影！...| 3788.0|    3588.0|      3888.0|       3|           1|          38|         115|
+------+--------------------+--------------------+-------+----------+------------+--------+------------+------------+------------+
only showing top 10 rows
```

- 匹配1、2、3级分类的中文描述

``` python
# 方式1：利用sql语句的方式
sql3 = '''
select * from(
    select a.id sku_id,name, caption, price, cost_price, market_price, goods_id, category1_id, category2_id, category3_id 
    from 
    tb_sku as a
    join
    (select id, category1_id, category2_id, category3_id from tb_goods) as b
    where a.goods_id=b.id
) as tb
join
(select id, name category1 from tb_goods_category) as c
where tb.category1_id=c.id
'''
# 这里把一级分类匹配上
spark.sql(sql3).show()
# 但如果还要继续匹配二级三级，使用sql语句的话，代码可读性很差，因此这里最终使用dataframe的api来完成分类剩余的sql操作
```

- 显示结果

```shell
+------+--------------------+--------------------+-------+----------+------------+--------+------------+------------+------------+---+---------+
|sku_id|                name|             caption|  price|cost_price|market_price|goods_id|category1_id|category2_id|category3_id| id|category1|
+------+--------------------+--------------------+-------+----------+------------+--------+------------+------------+------------+---+---------+
|     1|Apple MacBook Pro...|【全新2017款】MacBook ...|11388.0|   10350.0|     13388.0|       1|           4|          45|         157|  4|       电脑|
|     2|Apple MacBook Pro...|【全新2017款】MacBook ...|11398.0|   10388.0|     13398.0|       1|           4|          45|         157|  4|       电脑|
|     3|Apple iPhone 8 Pl...|选【移动优惠购】新机配新卡，198...| 6499.0|    6300.0|      6598.0|       2|           1|          38|         115|  1|       手机|
|     4|Apple iPhone 8 Pl...|选【移动优惠购】新机配新卡，198...| 7988.0|    7888.0|      8088.0|       2|           1|          38|         115|  1|       手机|
|     5|Apple iPhone 8 Pl...|选【移动优惠购】新机配新卡，198...| 6688.0|    6588.0|      6788.0|       2|           1|          38|         115|  1|       手机|
|     6|Apple iPhone 8 Pl...|选【移动优惠购】新机配新卡，198...| 7988.0|    7888.0|      7988.0|       2|           1|          38|         115|  1|       手机|
|     7|Apple iPhone 8 Pl...|选【移动优惠购】新机配新卡，198...| 6688.0|    6588.0|      6788.0|       2|           1|          38|         115|  1|       手机|
|     8|Apple iPhone 8 Pl...|选【移动优惠购】新机配新卡，198...| 7988.0|    7888.0|      7988.0|       2|           1|          38|         115|  1|       手机|
|     9|华为 HUAWEI P10 Plu...|wifi双天线设计！徕卡人像摄影！...| 3388.0|    3288.0|      3388.0|       3|           1|          38|         115|  1|       手机|
|    10|华为 HUAWEI P10 Plu...|wifi双天线设计！徕卡人像摄影！...| 3788.0|    3588.0|      3888.0|       3|           1|          38|         115|  1|       手机|
+------+--------------------+--------------------+-------+----------+------------+--------+------------+------------+------------+---+---------+
only showing top 10 rows
```

- 方式2：利用dataframe的sql化API方式

```python
sql2 = '''
select a.id sku_id, name, caption, price, cost_price, market_price, goods_id, category1_id, category2_id, category3_id 
from 
tb_sku as a
join
(select id, category1_id, category2_id, category3_id from tb_goods) as b
where a.goods_id=b.id
'''

ret = spark.sql(sql2)
# 合并一级分类
cate_df = spark.sql("select id, name category1 from tb_goods_category")
ret = ret.join(cate_df,[ret.category1_id==cate_df.id])

# 合并二级分类
cate_df = spark.sql("select id, name category2 from tb_goods_category")
ret = ret.join(cate_df,[ret.category2_id==cate_df.id])

# 合并三级分类
cate_df = spark.sql("select id, name category3 from tb_goods_category")
ret = ret.join(cate_df,[ret.category3_id==cate_df.id])

ret.select("sku_id,name,caption,price,cost_price,market_price,category1,category2,category3".split(",")).show()
```

- 显示结果

```shell
+------+--------------------+--------------------+-------+----------+------------+---------+---------+---------+
|sku_id|                name|             caption|  price|cost_price|market_price|category1|category2|category3|
+------+--------------------+--------------------+-------+----------+------------+---------+---------+---------+
|     1|Apple MacBook Pro...|【全新2017款】MacBook ...|11388.0|   10350.0|     13388.0|       电脑|     电脑整机|      笔记本|
|     2|Apple MacBook Pro...|【全新2017款】MacBook ...|11398.0|   10388.0|     13398.0|       电脑|     电脑整机|      笔记本|
|     3|Apple iPhone 8 Pl...|选【移动优惠购】新机配新卡，198...| 6499.0|    6300.0|      6598.0|       手机|     手机通讯|       手机|
|     4|Apple iPhone 8 Pl...|选【移动优惠购】新机配新卡，198...| 7988.0|    7888.0|      8088.0|       手机|     手机通讯|       手机|
|     5|Apple iPhone 8 Pl...|选【移动优惠购】新机配新卡，198...| 6688.0|    6588.0|      6788.0|       手机|     手机通讯|       手机|
|     6|Apple iPhone 8 Pl...|选【移动优惠购】新机配新卡，198...| 7988.0|    7888.0|      7988.0|       手机|     手机通讯|       手机|
|     7|Apple iPhone 8 Pl...|选【移动优惠购】新机配新卡，198...| 6688.0|    6588.0|      6788.0|       手机|     手机通讯|       手机|
|     8|Apple iPhone 8 Pl...|选【移动优惠购】新机配新卡，198...| 7988.0|    7888.0|      7988.0|       手机|     手机通讯|       手机|
|     9|华为 HUAWEI P10 Plu...|wifi双天线设计！徕卡人像摄影！...| 3388.0|    3288.0|      3388.0|       手机|     手机通讯|       手机|
|    10|华为 HUAWEI P10 Plu...|wifi双天线设计！徕卡人像摄影！...| 3788.0|    3588.0|      3888.0|       手机|     手机通讯|       手机|
+------+--------------------+--------------------+-------+----------+------------+---------+---------+---------+
only showing top 10 rows
```

- 匹配每一个SKU的specification并选出对应的文字描述
  - 连接tb_sku_specification表和tb_specification_option

``` python
sql = '''
select option_id, sku_id, a.spec_id, option 
from 
tb_sku_specification as a 
join 
(select id, value option from tb_specification_option) as b 
where a.option_id=b.id
'''
spark.sql(sql).sort("sku_id").show()
```

显示结果

```shell
+---------+------+-------+-------------------+
|option_id|sku_id|spec_id|             option|
+---------+------+-------+-------------------+
|        4|     1|      2|                 银色|
|        7|     1|      3|core i5/8G内存/512G存储|
|        1|     1|      1|             13.3英寸|
|        1|     2|      1|             13.3英寸|
|        7|     2|      3|core i5/8G内存/512G存储|
|        3|     2|      2|                深灰色|
|       11|     3|      5|               64GB|
|        8|     3|      4|                 金色|
|        8|     4|      4|                 金色|
|       12|     4|      5|              256GB|
+---------+------+-------+-------------------+
only showing top 10 rows
```

- 将sql1的结果再和tb_goods_specification表连接，并选出specification的文字描述和对应sku_id

```python
sql2 = '''
select sku_id, specification, option from (
    select option_id, sku_id, a.spec_id, option from tb_sku_specification as a 
    join 
    (select id, value option from tb_specification_option) as b
    where a.option_id=b.id
) as tb 
join 
(select id, name specification from tb_goods_specification) as c
where tb.spec_id=c.id
'''
spark.sql(sql2).sort("sku_id").show()
```

- 显示结果

```shell
+------+-------------+-------------------+
|sku_id|specification|             option|
+------+-------------+-------------------+
|     1|           颜色|                 银色|
|     1|           版本|core i5/8G内存/512G存储|
|     1|         屏幕尺寸|             13.3英寸|
|     2|           颜色|                深灰色|
|     2|           版本|core i5/8G内存/512G存储|
|     2|         屏幕尺寸|             13.3英寸|
|     3|           内存|               64GB|
|     3|           颜色|                 金色|
|     4|           颜色|                 金色|
|     4|           内存|              256GB|
+------+-------------+-------------------+
only showing top 10 rows
```

- 将刚才sql2的结果以SKU为单位进行合并
  - 先使用sql的concat方法可以对列进行合并
  - 然后对数据进行group by后使用collect_set进行聚合操作，收集每一列非重复数据，再用concat或concat_ws方法对列进行合并
  - 相关API
    - [pyspark.sql.functions.concat](https://spark.apache.org/docs/2.2.2/api/python/pyspark.sql.html?highlight=concat#pyspark.sql.functions.concat)
    - [pyspark.sql.functions.concat_ws](https://spark.apache.org/docs/2.2.2/api/python/pyspark.sql.html?highlight=concat#pyspark.sql.functions.concat_ws)
    - [pyspark.sql.functions.collect_set](https://spark.apache.org/docs/2.2.2/api/python/pyspark.sql.html?highlight=concat#pyspark.sql.functions.collect_set)
- 合并列：concat

```python
sql3 = '''
select sku_id, concat(specification,":",option) as temp from (
    select option_id, sku_id, a.spec_id, option from tb_sku_specification as a 
    join 
    (select id, value option from tb_specification_option) as b
    where a.option_id=b.id
) as tb 
join 
(select id, name specification from tb_goods_specification) as c
where tb.spec_id=c.id
'''
spark.sql(sql3).sort("sku_id").show()
```

显示结果

```shell
+------+--------------------+
|sku_id|                temp|
+------+--------------------+
|     1|版本:core i5/8G内存/5...|
|     1|               颜色:银色|
|     1|         屏幕尺寸:13.3英寸|
|     2|版本:core i5/8G内存/5...|
|     2|              颜色:深灰色|
|     2|         屏幕尺寸:13.3英寸|
|     3|               颜色:金色|
|     3|             内存:64GB|
|     4|            内存:256GB|
|     4|               颜色:金色|
+------+--------------------+
only showing top 10 rows
```

- 聚合后合并行：collect_set

```python
sql4 = '''
select sku_id, concat_ws(",", collect_set(temp)) as specification  
from 
    (select sku_id, concat(specification,":",option) as temp from 
            (select option_id, sku_id, a.spec_id, option from tb_sku_specification as a 
            join 
            (select id, value option from tb_specification_option) as b  
            where a.option_id=b.id
            ) as tb 
        join 
        (select id, name specification from tb_goods_specification) as c
    where tb.spec_id=c.id
    ) 
group by sku_id
'''
spark.sql(sql4).sort("sku_id").show()
```

显示结果

```shell
+------+--------------------+
|sku_id|       specification|
+------+--------------------+
|     1|屏幕尺寸:13.3英寸,版本:co...|
|     2|屏幕尺寸:13.3英寸,颜色:深灰...|
|     3|       颜色:金色,内存:64GB|
|     4|      颜色:金色,内存:256GB|
|     5|      颜色:深空灰,内存:64GB|
|     6|     内存:256GB,颜色:深空灰|
|     7|       内存:64GB,颜色:银色|
|     8|      内存:256GB,颜色:银色|
|     9|      版本:64GB,颜色:钻雕金|
|    10|     版本:128GB,颜色:钻雕金|
+------+--------------------+
only showing top 10 rows
```

- 对合并后的行数据进行排序：sort_array

```python
sql5 = '''
select sku_id, concat_ws(",", sort_array(collect_set(temp))) as specification  from (
    select sku_id, concat(specification,":",option) as temp from (
        select option_id, sku_id, a.spec_id, option from tb_sku_specification as a 
        join 
        (select id, value option from tb_specification_option) as b
        where a.option_id=b.id
    ) as tb 
    join 
    (select id, name specification from tb_goods_specification) as c
    where tb.spec_id=c.id
) group by sku_id
'''
spark.sql(sql5).sort("sku_id").show()
```

显示结果

```shell
+------+--------------------+
|sku_id|       specification|
+------+--------------------+
|     1|屏幕尺寸:13.3英寸,版本:co...|
|     2|屏幕尺寸:13.3英寸,版本:co...|
|     3|       内存:64GB,颜色:金色|
|     4|      内存:256GB,颜色:金色|
|     5|      内存:64GB,颜色:深空灰|
|     6|     内存:256GB,颜色:深空灰|
|     7|       内存:64GB,颜色:银色|
|     8|      内存:256GB,颜色:银色|
|     9|      版本:64GB,颜色:钻雕金|
|    10|     版本:128GB,颜色:钻雕金|
+------+--------------------+
only showing top 10 rows
```

- 对前面的两个结果进行合并，获得最终sku_detail表

```python
# a.category合并
sql2 = '''
select a.id sku_id, name, caption, price, cost_price, market_price, goods_id, category1_id, category2_id, category3_id 
from 
tb_sku as a
join
(select id, category1_id, category2_id, category3_id from tb_goods) as b
where a.goods_id=b.id
'''

ret = spark.sql(sql2)
# 合并一级分类
cate_df = spark.sql("select id, name category1 from tb_goods_category")
ret = ret.join(cate_df,[ret.category1_id==cate_df.id])

# 合并二级分类
cate_df = spark.sql("select id, name category2 from tb_goods_category")
ret = ret.join(cate_df,[ret.category2_id==cate_df.id])

# 合并三级分类
cate_df = spark.sql("select id, name category3 from tb_goods_category")
ret = ret.join(cate_df,[ret.category3_id==cate_df.id])

# b.
sql5 =  '''
select sku_id, concat_ws(",", sort_array(collect_set(temp))) as specification  from (
    select sku_id, concat(specification,":",option) as temp from (
        select option_id, sku_id, a.spec_id, option from tb_sku_specification as a 
        join 
        (select id, value option from tb_specification_option) as b
        where a.option_id=b.id
    ) as tb 
    join 
    (select id, name specification from tb_goods_specification) as c
    where tb.spec_id=c.id
) group by sku_id
'''
# 避免sku_id冲突，这里改写一下名称
specification_df = spark.sql(sql5).withColumnRenamed("sku_id", "sku_id2")

sku_detail = ret.join(specification_df, ret.sku_id==specification_df.sku_id2, "outer")

sku_detail = sku_detail.select("goods_id,sku_id,category1_id,category1,category2_id,category2,category3_id,category3,name,caption,price,cost_price,market_price,specification".split(","))

sku_detail.show()
```

显示结果

```shell
+--------+------+------------+---------+------------+---------+------------+---------+--------------------+--------------------+------+----------+------------+--------------------+
|goods_id|sku_id|category1_id|category1|category2_id|category2|category3_id|category3|                name|             caption| price|cost_price|market_price|       specification|
+--------+------+------------+---------+------------+---------+------------+---------+--------------------+--------------------+------+----------+------------+--------------------+
|     135|   148|           3|       数码|          41|     数码配件|         140|      读卡器|随身厅 WPOS-3 高度集成业务...|      享包邮！正品保证，购物无忧！|2999.0|    2999.0|      2999.0|                null|
|     451|   463|           3|       数码|          41|     数码配件|         140|      读卡器|飞花令 安卓手机读卡器Type-c...|您身边的私人定制：【联系客服告知型...|   7.8|       7.8|         7.8|颜色:Type-C TF卡 读卡器...|
|     458|   471|           3|       数码|          41|     数码配件|         140|      读卡器|【包邮】飞花令 安卓外置手机读卡器...|micro usb/V8 TF卡读...|  15.8|      15.8|        15.8|版本:360N5 N4S N4 N...|
|     483|   496|           3|       数码|          41|     数码配件|         140|      读卡器|品胜（PISEN） 全能读卡器迷你...|【京东配送·快速送达】提供一年质保...|  29.0|      29.0|        29.0|颜色:SD读卡器,颜色:TF读卡器...|
|     820|   833|           3|       数码|          41|     数码配件|         140|      读卡器|LEXAR 雷克沙（Lexar） ...|                    | 160.0|     160.0|       160.0|          版本:25合一读卡器|
|    1075|  1088|           2|       相机|          40|     摄影摄像|         135|     数码相框|青美 壁挂广告机65寸安卓网络广告...|                    |2699.0|    2699.0|      2699.0|版本:触摸版,版本:非触摸,颜色:...|
|    1225|  1238|           3|       数码|          41|     数码配件|         140|      读卡器|dyplay苹果手机相机读卡器三合...|                    | 128.0|     128.0|       128.0|                null|
|    1329|  1342|           3|       数码|          41|     数码配件|         140|      读卡器|绿联（UGREEN） Type-C...|支持读取安防监控/单反相机sd/t...|  39.0|      39.0|        39.0|颜色:Type-C+USB双卡单读...|
|    1567|  1580|           2|       相机|          40|     摄影摄像|         135|     数码相框|HNM 19英寸高清壁挂数码相框1...|                    | 988.0|     988.0|       988.0|版本:10英寸白色钢化玻璃8G卡,...|
|    1578|  1591|           3|       数码|          41|     数码配件|         140|      读卡器|kisdisk  读卡器四合一US...|         高速接口 四合一读卡器| 198.0|     198.0|       198.0|版本:128G卡,版本:256G卡...|
+--------+------+------------+---------+------------+---------+------------+---------+--------------------+--------------------+------+----------+------------+--------------------+
only showing top 10 rows
```

- 将sku_detail数据写入hive表

```python
# 将spark的dataframe注册为临时表，以便能对其使用sql语句
sku_detail.registerTempTable("tempTable")
# 查看临时表结构
spark.sql("desc tempTable").show()
```

- 显示结果

```shell
+-------------+---------+-------+
|     col_name|data_type|comment|
+-------------+---------+-------+
|     goods_id|      int|   null|
|       sku_id|      int|   null|
| category1_id|      int|   null|
|    category1|   string|   null|
| category2_id|      int|   null|
|    category2|   string|   null|
| category3_id|      int|   null|
|    category3|   string|   null|
|         name|   string|   null|
|      caption|   string|   null|
|        price|   double|   null|
|   cost_price|   double|   null|
| market_price|   double|   null|
|specification|   string|   null|
+-------------+---------+-------+
```

- 创建表并写入数据到Hive

```python
# 由于当前开启了hive sql功能，因此这里先创建表
spark.sql("DROP TABLE IF EXISTS `sku_detail`")

sql = '''
CREATE TABLE `sku_detail` (
goods_id INT,
sku_id INT,
category1_id INT,
category1 STRING,
category2_id INT,
category2 STRING,
category3_id INT,
category3 STRING,
name STRING,
caption STRING,
price DOUBLE,
cost_price DOUBLE,
market_price DOUBLE,
specification STRING
)
'''
spark.sql(sql)
# 写入数据到hive表
spark.sql("INSERT INTO sku_detail SELECT * FROM tempTable")
```

- 检查hive表sku_detail中的数据

```python
spark.sql("select * from sku_detail").count()
```

- 显示结果

```shell
326173
```

```python
spark.sql("select * from sku_detail").show()
```

- 显示结果:

```shell
+--------+------+------------+---------+------------+---------+------------+---------+--------------------+--------------------+------+----------+------------+--------------------+
|goods_id|sku_id|category1_id|category1|category2_id|category2|category3_id|category3|                name|             caption| price|cost_price|market_price|       specification|
+--------+------+------------+---------+------------+---------+------------+---------+--------------------+--------------------+------+----------+------------+--------------------+
|     135|   148|           3|       数码|          41|     数码配件|         140|      读卡器|随身厅 WPOS-3 高度集成业务...|      享包邮！正品保证，购物无忧！|2999.0|    2999.0|      2999.0|                null|
|     451|   463|           3|       数码|          41|     数码配件|         140|      读卡器|飞花令 安卓手机读卡器Type-c...|您身边的私人定制：【联系客服告知型...|   7.8|       7.8|         7.8|颜色:Type-C TF卡 读卡器...|
|     458|   471|           3|       数码|          41|     数码配件|         140|      读卡器|【包邮】飞花令 安卓外置手机读卡器...|micro usb/V8 TF卡读...|  15.8|      15.8|        15.8|版本:360N5 N4S N4 N...|
|     483|   496|           3|       数码|          41|     数码配件|         140|      读卡器|品胜（PISEN） 全能读卡器迷你...|【京东配送·快速送达】提供一年质保...|  29.0|      29.0|        29.0|颜色:SD读卡器,颜色:TF读卡器...|
|     820|   833|           3|       数码|          41|     数码配件|         140|      读卡器|LEXAR 雷克沙（Lexar） ...|                    | 160.0|     160.0|       160.0|          版本:25合一读卡器|
|    1075|  1088|           2|       相机|          40|     摄影摄像|         135|     数码相框|青美 壁挂广告机65寸安卓网络广告...|                    |2699.0|    2699.0|      2699.0|版本:触摸版,版本:非触摸,颜色:...|
|    1225|  1238|           3|       数码|          41|     数码配件|         140|      读卡器|dyplay苹果手机相机读卡器三合...|                    | 128.0|     128.0|       128.0|                null|
|    1329|  1342|           3|       数码|          41|     数码配件|         140|      读卡器|绿联（UGREEN） Type-C...|支持读取安防监控/单反相机sd/t...|  39.0|      39.0|        39.0|颜色:Type-C+USB双卡单读...|
|    1567|  1580|           2|       相机|          40|     摄影摄像|         135|     数码相框|HNM 19英寸高清壁挂数码相框1...|                    | 988.0|     988.0|       988.0|版本:10英寸白色钢化玻璃8G卡,...|
|    1578|  1591|           3|       数码|          41|     数码配件|         140|      读卡器|kisdisk  读卡器四合一US...|         高速接口 四合一读卡器| 198.0|     198.0|       198.0|版本:128G卡,版本:256G卡...|
|    1632|  1645|           2|       相机|          40|     摄影摄像|         135|     数码相框|爱国者（aigo） 数码相框 DP...|21.5寸 商业广告机 家用大屏相...|1699.0|    1699.0|      1699.0|版本:官方标配,版本:官方标配+金...|
|    1816|  1829|           3|       数码|          41|     数码配件|         140|      读卡器|金士顿（Kingston） USB...|11月1日钜惠来袭，11.11元限...|  69.9|      69.9|        69.9|颜色:USB3.1双接口Micro...|
|    1946|  1959|           2|       相机|          40|     摄影摄像|         128|     数码相机|理光（Ricoh） THETA 全...|【11.11京东全球好物节】影像钜...|1599.0|    1599.0|      1599.0|颜色:THETA SC 白色,颜色...|
|    2109|  2122|           1|       手机|          39|     手机配件|         126|     移动电源|贝视特苹果笔记本充电宝移动电源QC...|如需更大功率容量移动电源可点击了解...| 399.0|     399.0|       399.0|颜色:太空银20000毫安/DC数...|
|    2129|  2142|           1|       手机|          39|     手机配件|         126|     移动电源|戈派 无线磁吸充电宝迷你超薄应急移...|                    | 258.0|     258.0|       258.0|颜色:三合一，珍珠白,颜色:四合一...|
|    2353|  2366|           1|       手机|          39|     手机配件|         126|     移动电源|赋电 充电宝超薄小巧 苹果安卓迷你...|领券下单立减3元，吸附式充电宝，购...|  89.9|      89.9|        89.9|版本:型号,颜色:01款苹果6/6...|
|    2646|  2659|           1|       手机|          39|     手机配件|         126|     移动电源|OISLE苹果专用无线充电宝 ip...|换手机不用换背夹 无线充电 苹果快...| 109.0|     109.0|       109.0|版本:iphone 5/5S/SE...|
|    2853|  2866|           1|       手机|          38|     手机通讯|         118|      对讲机|宝锋（BAOFENG） BF-UV...|★1111特惠★自驾游户外专属【手...| 159.0|     159.0|       159.0|颜色:5R三代（亲民）,颜色:一代...|
|    3162|  3175|           1|       手机|          38|     手机通讯|         118|      对讲机|Motorola 摩托罗拉T8对讲...|摩托罗拉免执照对讲机 商务手台 送...| 480.0|     480.0|       480.0|        颜色:MOTO商务系列T|
|    3736|  3749|           1|       手机|          38|     手机通讯|         118|      对讲机|ZASTONE 即时通D9000车...|送安装6件套 50W大功率 带中继...|2598.0|    2598.0|      2598.0|颜色:中文版 吸盘天线套餐,颜色:...|
+--------+------+------------+---------+------------+---------+------------+---------+--------------------+--------------------+------+----------+------------+--------------------+
only showing top 10 rows
```

