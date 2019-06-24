## 七 基于内容的商品相似度计算与商品召回

- 网站初期，由于没有大量的用户数据，因此初期的离线推荐将以基于物品画像的召回推荐为主，

  主要实现逻辑：

  根据商品关键词对应的权重值结合该关键词对应的词向量进行加权求平均计算出该商品的向量值

  每一个sku对应n个关键词  

  ``` python
  words=['数码', '数码配件', '读卡器', 'WPOS', '高度', '业务', '智能', '终端', '森锐', '触摸屏', '收银机', '身份', '包邮', '正品', '购物']
  ```

  每一个关键词有一个词向量

  ```shell
  [Row(word='钟爱', vector=DenseVector([-0.0042, -0.1172, 0.0594, 0.0564, -0.049, -0.0457, 0.0411, 0.1067, -0.0022, -0.0802, -0.0047, 0.0994, -0.0352, 0.0873, -0.0881, -0.0294, 0.044, 0.0465, 0.0676, 0.024, 0.0667, -0.0318, -0.1308, 0.0375, 0.062, -0.0315, 0.1234, 0.0362, -0.0514, 0.1064, -0.0207, -0.0979, 0.0573, -0.0375, -0.1729, 0.1023, 0.0792, 0.0389, 0.0515, 0.0372, 0.0506, -0.1057, 0.0342, -0.1603, 0.1454, 0.0428, -0.0016, -0.0058, -0.0609, -0.0648, -0.0104, -0.0365, -0.1157, 0.0437, 0.064, 0.0896, 0.0412, 0.048, 0.1512, -0.0691, -0.047, -0.1193, -0.0817, 0.0354, 0.2027, 0.0355, -0.0444, -0.0089, -0.0672, 0.0502, -0.062, -0.0532, -0.0295, -0.1187, -0.0217, 0.0015, 0.0039, -0.1415, 0.0448, 0.0175, -0.1728, -0.0365, -0.058, 0.0222, -0.1564, 0.1083, 0.0279, 0.0307, 0.0911, 0.0291, 0.0404, -0.0636, 0.1131, 0.0489, 0.0226, 0.0851, 0.1362, 0.091, -0.0671, -0.0095])),
  ```

  每一个关键词都有一个权重

  ``` shell
  +------+--------+----+-------------------+-------------------+-------------------+
  |sku_id|industry| tag|           textrank|              tfidf|            weights|
  +------+--------+----+-------------------+-------------------+-------------------+
  |   148|    电子产品|  智能|  0.853869407929065|0.12920924724659527|0.49153932758783014|
  |   148|    电子产品|WPOS| 0.9474997114660046| 0.6672418695993603| 0.8073707905326825|
  |   148|    电子产品|  包邮| 0.5962385663111377| 0.2752684393351877| 0.4357535028231627|
  |   148|    电子产品|  购物|  0.442940968516803| 0.2408169399138654| 0.3418789542153342|
  ```

  

  通过sku对应的关键词 以及关键词的权重 关键词的词向量 计算出一个sku的向量

  计算不同的sku之间的向量相似度 作为召回商品的依据

  利用相似度得出每件件商品TOP-N与之相似的商品

### 7.1 加载词频权重数据

- 加载之前的计算结果

```python
sku_tag_weights = spark.sql("select * from sku_tag_merge_weights")
sku_tag_weights.show()
```

展示查询结果

```shell
+------+--------+-------+-------------------+-------------------+-------------------+
|sku_id|industry|    tag|           textrank|              tfidf|            weights|
+------+--------+-------+-------------------+-------------------+-------------------+
|    85|    电子产品|     数码|0.37669191799893376|0.07434212900804543| 0.2255170235034896|
|   172|    电子产品|   USB2| 0.5297582014310518| 0.4902682570089824| 0.5100132292200171|
|   182|    电子产品|     型号| 0.9112570559313992| 0.4417105502575884| 0.6764838030944937|
|   190|    电子产品|    雷克沙|0.18932069864134093|0.25239101355870736|0.22085585610002414|
|   271|    电子产品|   数码配件|0.18629417960331815|0.16435125309725987|  0.175322716350289|
|   282|    电子产品|    香槟色|0.20939011016154468|  0.450895660720322|0.33014288544093334|
|   305|    电子产品|     星空|0.25201085335696144|0.32303206716234273| 0.2875214602596521|
|   312|    电子产品|   SONY| 0.6481662914614708| 0.4221730662623697| 0.5351696788619202|
|   326|    电子产品|     川宇| 0.3939664586744098| 0.8599491424894472| 0.6269578005819285|
|   334|    电子产品| TOPSSD|0.37769613311191574| 0.9027798790978679| 0.6402380061048918|
|   334|    电子产品|     天硕|0.31446175905424484| 0.9027798790978679| 0.6086208190760564|
|   351|    电子产品|   数码配件| 0.3284858938211362|0.43142203938030715|0.37995396660072167|
|   370|    电子产品|     功能|  0.171914010773732|0.20335407147331752|0.18763404112352478|
|   403|    电子产品|   数码配件| 0.3543503711641944| 0.2465268796458898|0.30043862540504207|
|   410|    电子产品|MicroSD| 0.3341093557715508|0.35569624565338087| 0.3449028007124658|
|   414|    电子产品|     颜色| 0.1632912209999795|0.03465489849573508|0.09897305974785729|
|   441|    电子产品|    手机卡|0.09101143769017656|0.21072675335984295|0.15086909552500977|
|   441|    电子产品|     数码|0.09553860210195766|0.04556453068235043|0.07055156639215404|
|   450|    电子产品|   数码配件|0.41273510820561926|0.21571101969015358| 0.3142230639478864|
|   456|    电子产品|     数码| 0.2580242294098149|0.06726192624537444|0.16264307782759468|
+------+--------+-------+-------------------+-------------------+-------------------+
only showing top 20 rows
```

- 查看一个品类的关键词权重结果

```python
spark.sql("SELECT * FROM sku_tag_merge_weights where sku_id=1 order by weights").show()
```

显示查询结果

```shell
+------+--------+-------+-------------------+--------------------+-------------------+
|sku_id|industry|    tag|           textrank|               tfidf|            weights|
+------+--------+-------+-------------------+--------------------+-------------------+
|     1|    电子产品|     颜色|0.23858124571960931|0.012129214473507281| 0.1253552300965583|
|     1|    电子产品|     官方|0.40193232407753093|  0.1290345523159126|0.26548343819672177|
|     1|    电子产品|     产品| 0.4465411654517351| 0.09150413938225072| 0.2690226524169929|
|     1|    电子产品|     特惠| 0.3683345433126666|   0.212171847060278| 0.2902531951864723|
|     1|    电子产品|     版本| 0.5631617412377677| 0.06297913325415283| 0.3130704372459603|
|     1|    电子产品|     i5|0.47596461597376005|  0.1554620507339392|0.31571333335384966|
|     1|    电子产品|     内存|  0.508557740569568| 0.14093529366222984|0.32474651711589897|
|     1|    电子产品|     屏幕| 0.5488410475104375| 0.21232883306051142|0.38058494028547446|
|     1|    电子产品|     电脑| 0.6740172924015514| 0.09269926353711627|0.38335827796933386|
|     1|    电子产品|     即发| 0.5133667671128688| 0.26525609786407256|0.38931143248847067|
|     1|    电子产品|     尺寸| 0.6342523535440957| 0.18900585201996106|0.41162910278202836|
|     1|    电子产品|     才华|0.40417007695625207|  0.5004314021995202|0.45230073957788614|
|     1|    电子产品|     黑五|0.46778188068617316|  0.5004314021995202|0.48410664144284665|
|     1|    电子产品|     银色|  0.684377750753916|  0.3039150706380271|0.49414641069597154|
|     1|    电子产品|   core| 0.5574657990181027| 0.46577404317152293| 0.5116199210948128|
|     1|    电子产品|     英寸| 0.8975902158426171| 0.19777846894227954| 0.5476843423924483|
|     1|    电子产品|    笔记本|  0.899406809710101|  0.2816696831885774| 0.5905382464493392|
|     1|    电子产品|    Pro| 0.8695484060070439| 0.38615852650639787| 0.6278534662567209|
|     1|    电子产品|  Apple| 0.8817364088520955| 0.46631090759580734| 0.6740236582239514|
|     1|    电子产品|MacBook|                1.0|  0.5903233645581719| 0.7951616822790859|
+------+--------+-------+-------------------+--------------------+-------------------+
```

### 7.2 加载Word2vec模型

- 从hdfs中把模型加载出来

``` python
from pyspark.ml.feature import Word2VecModel
word2vec_model = Word2VecModel.load("hdfs://localhost:9000/电子产品.word2vec_model")
vectors = word2vec_model.getVectors()
# 这里注意，由于部分词在wrod2vec模型中不存在，因此必须使用inner join，舍弃掉这次，他们数量较少，就忽略不计
_ = sku_tag_weights.join(vectors, sku_tag_weights.tag==vectors.word, "inner")
_.show()
```

- 显示结果

``` shell
+------+--------+-------+-------------------+-------------------+-------------------+-------+--------------------+
|sku_id|industry|    tag|           textrank|              tfidf|            weights|   word|              vector|
+------+--------+-------+-------------------+-------------------+-------------------+-------+--------------------+
|    85|    电子产品|     数码|0.37669191799893376|0.07434212900804543| 0.2255170235034896|     数码|[0.05912682041525...|
|   172|    电子产品|   USB2| 0.5297582014310518| 0.4902682570089824| 0.5100132292200171|   USB2|[-0.1141790449619...|
|   182|    电子产品|     型号| 0.9112570559313992| 0.4417105502575884| 0.6764838030944937|     型号|[-0.8850123286247...|
|   190|    电子产品|    雷克沙|0.18932069864134093|0.25239101355870736|0.22085585610002414|    雷克沙|[-0.1313853859901...|
|   271|    电子产品|   数码配件|0.18629417960331815|0.16435125309725987|  0.175322716350289|   数码配件|[-0.2159668058156...|
|   282|    电子产品|    香槟色|0.20939011016154468|  0.450895660720322|0.33014288544093334|    香槟色|[-0.1126879602670...|
|   305|    电子产品|     星空|0.25201085335696144|0.32303206716234273| 0.2875214602596521|     星空|[0.28189748525619...|
|   312|    电子产品|   SONY| 0.6481662914614708| 0.4221730662623697| 0.5351696788619202|   SONY|[-0.1847716569900...|
|   326|    电子产品|     川宇| 0.3939664586744098| 0.8599491424894472| 0.6269578005819285|     川宇|[-0.1949329227209...|
|   334|    电子产品| TOPSSD|0.37769613311191574| 0.9027798790978679| 0.6402380061048918| TOPSSD|[-0.0161089580506...|
|   334|    电子产品|     天硕|0.31446175905424484| 0.9027798790978679| 0.6086208190760564|     天硕|[-0.0332020446658...|
|   351|    电子产品|   数码配件| 0.3284858938211362|0.43142203938030715|0.37995396660072167|   数码配件|[-0.2159668058156...|
|   370|    电子产品|     功能|  0.171914010773732|0.20335407147331752|0.18763404112352478|     功能|[0.17380681633949...|
|   403|    电子产品|   数码配件| 0.3543503711641944| 0.2465268796458898|0.30043862540504207|   数码配件|[-0.2159668058156...|
|   410|    电子产品|MicroSD| 0.3341093557715508|0.35569624565338087| 0.3449028007124658|MicroSD|[-0.0490555875003...|
|   414|    电子产品|     颜色| 0.1632912209999795|0.03465489849573508|0.09897305974785729|     颜色|[0.00411114702001...|
|   441|    电子产品|    手机卡|0.09101143769017656|0.21072675335984295|0.15086909552500977|    手机卡|[-0.1219766363501...|
|   441|    电子产品|     数码|0.09553860210195766|0.04556453068235043|0.07055156639215404|     数码|[0.05912682041525...|
|   450|    电子产品|   数码配件|0.41273510820561926|0.21571101969015358| 0.3142230639478864|   数码配件|[-0.2159668058156...|
|   456|    电子产品|     数码| 0.2580242294098149|0.06726192624537444|0.16264307782759468|     数码|[0.05912682041525...|
+------+--------+-------+-------------------+-------------------+-------------------+-------+--------------------+
only showing top 20 rows
```

- 展示表中结果

```python
print(_.count())
sku_tag_weights.count()
```

显示结果

``` shell
1140439
1172403
```

### 7.3 计算商品相似度

- 使用每个词的综合权重乘以每个词的词向量

```python
sku_tag_vector = _.rdd.map(lambda r:(r.sku_id, r.tag, r.weights*r.vector)).toDF(["sku_id", "tag","vector"])
sku_tag_vector.show()
```

显示结果

``` shell
+------+-------+--------------------+
|sku_id|    tag|              vector|
+------+-------+--------------------+
|    85|     数码|[0.01333410454927...|
|   172|   USB2|[-0.0582328234302...|
|   182|     型号|[-0.5986965058535...|
|   190|    雷克沙|[-0.0290172319018...|
|   271|   数码配件|[-0.0378638870371...|
|   282|    香槟色|[-0.0372031283570...|
|   305|     星空|[0.08105157660438...|
|   312|   SONY|[-0.0988841883341...|
|   326|     川宇|[-0.1222147164901...|
|   334| TOPSSD|[-0.0103135671827...|
|   334|     天硕|[-0.0202074556195...|
|   351|   数码配件|[-0.0820574445237...|
|   370|     功能|[0.03261207532459...|
|   403|   数码配件|[-0.0648847702723...|
|   410|MicroSD|[-0.0169194095194...|
|   414|     颜色|[4.06892799643887...|
|   441|    手机卡|[-0.0184025048013...|
|   441|     数码|[0.00417148979608...|
|   450|   数码配件|[-0.0678617514344...|
|   456|     数码|[0.00961656805449...|
+------+-------+--------------------+
only showing top 20 rows
```

- 创建临时表

``` python
sku_tag_vector.registerTempTable("tempTable")

def map(row):
    x = 0
    for v in row.vectors:
        x += v
    #  将平均向量作为sku的向量
    return row.sku_id, x/len(row.vectors)

sku_vector = spark.sql("select sku_id, collect_set(vector) vectors from tempTable group by sku_id").rdd.map(map).toDF(["sku_id", "vector"])
sku_vector.show()
```

显示结果

``` shell
+------+--------------------+
|sku_id|              vector|
+------+--------------------+
|    26|[0.01235398195028...|
|    29|[0.21760844687031...|
|   474|[-0.0423007059418...|
|   964|[0.08240807672413...|
|  1677|[0.02065021965251...|
|  1697|[0.39568690077127...|
|  1806|[0.05530353810490...|
|  1950|[0.13603717113579...|
|  2040|[-0.0108802742841...|
|  2214|[-0.0480597909252...|
|  2250|[0.03186294420201...|
|  2453|[-0.0384253501587...|
|  2509|[-0.0221520914506...|
|  2529|[-0.0486537793846...|
|  2927|[-0.0213679434397...|
|  3091|[-0.0477536702739...|
|  3506|[-0.1093108035922...|
|  3764|[0.01326956604482...|
|  4590|[0.16880469210238...|
|  4823|[0.13785943123954...|
+------+--------------------+
only showing top 20 rows
```

- 计算皮尔逊相关系数

```python
from pyspark.mllib.stat import Statistics
# print(sku_vector.where("sku_id=1").select("vector").first().vector)
v1 = sku_vector.where("sku_id=1").select("vector").first().vector
v2 = sku_vector.where("sku_id=2").select("vector").first().vector

sc = spark.sparkContext

x = sc.parallelize(v1)  # rdd
y = sc.parallelize(v2)
# 
Statistics.corr(x, y, method="pearson")
# 约0.97
```

- 计算余弦相似度

```python
import numpy as np
vector1 = v1
vector2 = v2
# 余弦相似度
np.dot(vector1,vector2)/(np.linalg.norm(vector1)*(np.linalg.norm(vector2)))
# 0.97
```

- 查询sku_vector的条目数量

``` python
sku_vector.count()   # 6w6 * 6w6   44亿
```

显示结果

``` shell
66651
```

- 为计算sku相似度准备数据

```python
#sku_vector.withColumnRenamed("sku_id", "sku_id2").withColumnRenamed("vector", "vector2")
#sku_vector.join(sku_vector.withColumnRenamed("sku_id", "sku_id2").withColumnRenamed("vector", "vector2"))
temp_df = sku_vector.withColumnRenamed("sku_id", "sku_id2").withColumnRenamed("vector", "vector2")
import time
start = time.time()

### 注意注意注意：这里一定不要用inner join，因为内连接由于会剔除条左右表中不存在的条目，因此它会有过滤操作，在数据量极其大的情况下非常慢，注意是非常慢
print(sku_vector.join(temp_df, sku_vector.sku_id!=temp_df.sku_id2, how="inner").count())
print(time.time()-start)
# 66651 * 66651 约36亿条目
# 这里其实本身是一个自连接，不会有不存在的条目，所以直接用outer，提升执行效率
temp_df = sku_vector.withColumnRenamed("sku_id", "sku_id2").withColumnRenamed("vector", "vector2")
import time
start = time.time()
print(sku_vector.join(temp_df, sku_vector.sku_id!=temp_df.sku_id2, how="outer").count())
print(time.time()-start)
# 66651 * 66651 约44亿条目
```

- 

``` python
sku_vector.join(temp_df, sku_vector.sku_id!=temp_df.sku_id2, how="outer").show(100)
```

- 计算物品两两余弦相似度

``` python
import numpy as np
# 创建一个临时的DataFrame 
temp_df = sku_vector.withColumnRenamed("sku_id", "sku_id2").withColumnRenamed("vector", "vector2")
# 把sku_vector 和 临时的DataFrame 做JOIN  每一条记录 sku_id 和 sku_id2 都是不同的
# 1 2 vector1 vector2
# 1 3 vector1 vector3
# 1 4 vector1 vector4
#........................
sku_vector_join = sku_vector.join(temp_df, sku_vector.sku_id!=temp_df.sku_id2, how="outer")

def mapPartitions(partition):
    for row in partition:
        vector1 = row.vector
        vector2 = row.vector2
        
        sim = np.dot(vector1,vector2)/(np.linalg.norm(vector1)*(np.linalg.norm(vector2)))
        
        yield row.sku_id, row.sku_id2, float(sim)

similarity = sku_vector_join.rdd.mapPartitions(mapPartitions).toDF(["sku_id", "sku_id2", "sim"])
similarity
```

显示计算结果 

```python
similarity.show()
```

终端显示

```shell
+------+-------+--------------------+
|sku_id|sku_id2|                 sim|
+------+-------+--------------------+
|    26|     29| 0.31983941722824993|
|    26|    474|  0.1923222912419553|
|    26|    964|  0.2150093035916437|
|    26|   1677|  0.1619193116837042|
|    26|   1697| 0.11683355244602506|
|    26|   1806|  0.1404007057243507|
|    26|   1950| 0.04163347845281761|
|    26|   2040|  0.3258932783082057|
|    26|   2214| 0.14748466518194572|
|    26|   2250|   0.072558953233589|
|    26|   2453| 0.15444468268515185|
|    26|   2509|  0.2075858797438308|
|    26|   2529| 0.09926985028799892|
|    26|   2927|  0.2502524514154315|
|    26|   3091| 0.05759395308873295|
|    26|   3506|0.005805630735091046|
|    26|   3764|  0.3108594219056489|
|    26|   4590|  0.3535770338801637|
|    26|   4823|  0.3520366415295813|
|    26|   4894| 0.38996569066820563|
+------+-------+--------------------+
only showing top 20 rows
```

- 由于条目数实在太多，如果这里进行如查询、分组等操作需要大量的计算，对硬件要求很高，所以以下代码没能跑成功

```python
similarity.where("sku_id=1").show()
```

显示结果

``` shell
---------------------------------------------------------------------------
KeyboardInterrupt                         Traceback (most recent call last)
<ipython-input-11-8d8704b2f321> in <module>
----> 1 similarity.where("sku_id=1").show()

~/miniconda2/envs/bigdata/lib/python3.6/site-packages/pyspark-2.2.2-py3.6.egg/pyspark/sql/dataframe.py in show(self, n, truncate)
    334         """
    335         if isinstance(truncate, bool) and truncate:
--> 336             print(self._jdf.showString(n, 20))
    337         else:
    338             print(self._jdf.showString(n, int(truncate)))

~/miniconda2/envs/bigdata/lib/python3.6/site-packages/py4j-0.10.7-py3.6.egg/py4j/java_gateway.py in __call__(self, *args)
   1253             proto.END_COMMAND_PART
   1254 
-> 1255         answer = self.gateway_client.send_command(command)
   1256         return_value = get_return_value(
   1257             answer, self.gateway_client, self.target_id, self.name)

~/miniconda2/envs/bigdata/lib/python3.6/site-packages/py4j-0.10.7-py3.6.egg/py4j/java_gateway.py in send_command(self, command, retry, binary)
    983         connection = self._get_connection()
    984         try:
--> 985             response = connection.send_command(command)
    986             if binary:
    987                 return response, self._create_connection_guard(connection)

~/miniconda2/envs/bigdata/lib/python3.6/site-packages/py4j-0.10.7-py3.6.egg/py4j/java_gateway.py in send_command(self, command)
   1150 
   1151         try:
-> 1152             answer = smart_decode(self.stream.readline()[:-1])
   1153             logger.debug("Answer received: {0}".format(answer))
   1154             if answer.startswith(proto.RETURN_MESSAGE):

~/miniconda2/envs/bigdata/lib/python3.6/socket.py in readinto(self, b)
    584         while True:
    585             try:
--> 586                 return self._sock.recv_into(b)
    587             except timeout:
    588                 self._timeout_occurred = True

KeyboardInterrupt: 
```

- 考虑使用foreachPartition方法，遍历每一行数据，进行运算, 将结果直接存入redis

  - foreachPartition属于action运算操作，而mapPartitions是在Transformation，
  - 使用: 
    - mapPartitions可以获取返回值，继续在返回RDD上做其他的操作
    - foreachPartition是action操作没有返回值，数据处理的最后一步, 如将计算结果落地到hdfs中

  

- 每一个商品只保留与它相似的TOP100个商品的sku_id和对应的相似度，那么这里的结果其实就是一个基于商品相似度的一个召回结果集

```python
import numpy as np
import gc
import redis
temp_df = sku_vector.withColumnRenamed("sku_id", "sku_id2").withColumnRenamed("vector", "vector2")
sku_vector_join = sku_vector.join(temp_df, sku_vector.sku_id!=temp_df.sku_id2, how="outer")

def foreachPartition(partition):
    client = redis.StrictRedis()
    
    for row in partition:
        vector1 = row.vector
        vector2 = row.vector2
        
        # 余弦相似度计算公式
        sim = np.dot(vector1,vector2)/(np.linalg.norm(vector1)*(np.linalg.norm(vector2)))
        #zcard 返回有序集 key 的基数。返回值
		#当 key 存在且是有序集类型时，返回有序集的基数。 当 key 不存在时，返回 0
        if client.zcard(row.sku_id) < 100: #如果存入redis中的数量没超过100直接保存
            client.zadd(row.sku_id, float(sim), row.sku_id2)
        else:#如果存入redis中的数量超过100 比较min_sim值 只保留比较大的
            # 取出当前redis中与sku_id相似度最小的sku_id2的相似度值
            key = client.zrange(row.sku_id, 0, 0)
            min_sim = client.zscore(row.sku_id, key[0]) if len(key) == 1 else None
        
            if min_sim is None:
                client.zadd(row.sku_id, float(sim), row.sku_id2)
            else:
                if sim > min_sim:# 如果当前保存的sku 相似度比已经缓存的相似度还要大 
                    #删除相似度小的
                    client.zrem(row.sku_id, key[0])
                    #保留相似度大的
                    client.zadd(row.sku_id, float(sim), row.sku_id2)
        # 
            del key
            del min_sim
        del vector1
        del vector2
        del sim
        del row
        gc.collect()
        # 这里为了节约内存开销，手动进行内存回收，避免内存泄漏等问题

sku_vector_join.foreachPartition(foreachPartition)

# 但注意这里，由于计算量比较大， 非常耗时，预计4核8g需要一天时间才能跑完
# 这里大家可能会想，光是计算6w条电子产品的相似就要这这么久时间，那么如果全部大类都要计算完，或者说数量量加倍了，该如何处理：
    # 其实大家思考一下，对于商品之间的相似度计算，其实往往只需要一开始对所有的商品进行两两计算，比如6w条，那么计算量就是6w*6w
    # 但是其实后面如果有新增商品，只需要对新增商品与其他商品的相似度进行计算就可以，那么新增商品后的计算量是1*6w
    # 也就是我们只需要在一开始把所有的都算一遍，之后更新的时候，采取的是增量更新
    # 或者根据业务发展情况，每隔几个月做一次全量更新，期间就只做增量更新

# 一下报错，是因为手动关闭程序导致的，因为这里没有等全部算出结果，只算了一部分，就关闭了
```

### 7.4 其它计算相似度的方法

- 如pyspark.ml.stat.Correlation、pyspark.mllib.stat.Statistics 但这些的相似度计算都只能完成所有列两两之间的相似度计算，如果是计算所有行两两之间的相似度，就需要自行去实现

```python
from pyspark.ml.linalg import Vectors
from pyspark.ml.stat import Correlation

dataset = [[Vectors.dense([1, 0, 0, -2])],
[Vectors.dense([4, 5, 0, 3])],
[Vectors.dense([6, 7, 0, 8])],
[Vectors.dense([9, 0, 0, 1])],
[Vectors.dense([1, 0, 0, -2])]]
dataset = spark.createDataFrame(dataset, ['features'])
dataset.show()
pearsonCorr = Correlation.corr(dataset, 'features', 'pearson').show(truncate=False)
# print(str(pearsonCorr).replace('nan', 'NaN'))

spearmanCorr = Correlation.corr(dataset, 'features', method='spearman').show()
# print(str(spearmanCorr).replace('nan', 'NaN'))
```

显示结果

``` shell
+------------------+
|          features|
+------------------+
|[1.0,0.0,0.0,-2.0]|
| [4.0,5.0,0.0,3.0]|
| [6.0,7.0,0.0,8.0]|
| [9.0,0.0,0.0,1.0]|
|[1.0,0.0,0.0,-2.0]|
+------------------+

+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
|pearson(features)                                                                                                                                                                                                                                                      |
+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
|1.0                 0.2522120576380154  NaN  0.5517643975445569  
0.2522120576380154  1.0                 NaN  0.9262057975392757  
NaN                 NaN                 1.0  NaN                 
0.5517643975445569  0.9262057975392757  NaN  1.0                 |
+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+

+--------------------+
|  spearman(features)|
+--------------------+
|1.0              ...|
+--------------------+
```

- 

```python
from __future__ import print_function

import numpy as np

from pyspark import SparkContext
from pyspark.mllib.stat import Statistics

if __name__ == "__main__":
    sc = spark.sparkContext

    # $example on$
    seriesX = sc.parallelize([1.0, 2.0, 3.0, 3.0, 5.0])  # a series
    # seriesY must have the same number of partitions and cardinality as seriesX
    seriesY = sc.parallelize([11.0, 22.0, 33.0, 33.0, 555.0])

    # Compute the correlation using Pearson's method. Enter "spearman" for Spearman's method.
    # If a method is not specified, Pearson's method will be used by default.
    print("Correlation is: " + str(Statistics.corr(seriesX, seriesY, method="pearson")))

    data = sc.parallelize(
        [np.array([1.0, 10.0, 100.0]), np.array([2.0, 20.0, 200.0]), np.array([5.0, 33.0, 366.0]), np.array([5.0, 33.0, 366.0])]
    )  # an RDD of Vectors

    # calculate the correlation matrix using Pearson's method. Use "spearman" for Spearman's method.
    # If a method is not specified, Pearson's method will be used by default.
    print(Statistics.corr(data, method="pearson"))
```

显示结果

``` shell
Correlation is: 0.8500286768773007
[[1.         0.98473193 0.99316078]
 [0.98473193 1.         0.99832152]
 [0.99316078 0.99832152 1.        ]]

```

- 

```python
from pyspark.mllib.stat import Statistics
from pyspark.mllib.linalg import Vectors
rdd = sc.parallelize([Vectors.dense([1, 0, 0, -2]), Vectors.dense([4, 5, 0, 3]),Vectors.dense([6, 7, 0,  8]), Vectors.dense([9, 0, 0, 1]), Vectors.dense([9, 0, 0, 1])])
pearsonCorr = Statistics.corr(rdd)
pearsonCorr
```

显示结果

``` shell
array([[ 1.        , -0.16524238,         nan,  0.24090545],
       [-0.16524238,  1.        ,         nan,  0.89613897],
       [        nan,         nan,  1.        ,         nan],
       [ 0.24090545,  0.89613897,         nan,  1.        ]])
```

