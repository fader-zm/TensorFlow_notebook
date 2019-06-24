## 六 商品关键词的词向量计算(word2vec)

### 6.0 word2vec原理简介

- word2vec是google在2013年推出的一个NLP(Natural Language Processing自然语言处理) 工具，它的特点是将所有的词向量化，这样词与词之间就可以定量的去度量他们之间的关系，挖掘词之间的联系。
- one-hot vector VS. word vector
  - 用向量来表示词并不是word2vec的首创
  - 最早的词向量是很冗长的，它使用是词向量维度大小为整个词汇表的大小，对于每个具体的词汇表中的词，将对应的位置置为1。
  - 比如下面5个词组成词汇表，词"Queen"的序号为2， 那么它的词向量就是(0,1,0,0,0)同样的道理，词"Woman"的词向量就是(0,0,0,1,0)。

![](img/word2vec1.png)

- one hot vector的问题

  - 如果词汇表非常大，如达到万级别，每个词都用万维的向量来表示浪费内存。这样的向量除了一个位置是1，其余位置全部为0，过于稀疏，需要降低词向量的维度
  - 难以发现词之间的关系，以及难以捕捉句法（结构）和语义（意思）之间的关系
  - Dristributed representation可以解决One hot representation的问题，它的思路是通过训练，将每个词都映射到一个较短的词向量上来。所有的这些词向量就构成了向量空间，进而可以用普通的统计学的方法来研究词与词之间的关系。这个较短的词向量维度一般需要我们在训练时指定。
  - 比如下图我们将词汇表里的词用"Royalty(王位)","Masculinity(男性气质)", "Femininity(女性气质)"和"Age"4个维度来表示，King这个词对应的词向量可能是(0.99,0.99,0.05,0.7)(0.99,0.99,0.05,0.7)。当然在实际情况中，我们并不一定能对词向量的每个维度做一个很好的解释。

  ![](img/word2vec2.png)

- 有了用Dristributed representation表示的较短的词向量，就可以较容易的分析词之间的关系，比如将词的维度降维到2维，用下图的词向量表示我们的词时，发现：vec{King} - vec{Man} + vec{Woman} = vec{Queen}

![](img/word2vec3.png)

- 什么是word vector（词向量）

  - 每个单词被表征为多维的浮点数，每一维的浮点数的数值大小表示了它与另一个单词之间的“距离”，表征的结果就是语义相近的词被映射到相近的集合空间上，好处是这样单词之间就是可以计算的：

  <table>
      <th>
      <td> animal </td>
      <td> pet </td>
      </th>
  <tr>
   <td> dog </td>
   <td> -0.4 </td>
   <td> 0.02 </td>
  </tr>
  <tr>
   <td> lion </td>
   <td> 0.2 </td>
   <td> 0.35 </td>
  </tr>
  </table>

  animal那一列表示的就是左边的词与animal这个概念的”距离“

- #### Word2Vec

  **效果演示**

  ```python
  from pyspark.sql.functions import format_number as fmt
  #findSynonyms 找到跟‘笔记本’相似的20个词
  #fmt保留5位有效数字
  model.findSynonyms("荣耀", 20).show()
  #返回结果
  +---------+------------------+
  |     word|        similarity|
  +---------+------------------+
  |     play|0.8511283993721008|
  |      v10|0.8239492177963257|
  |       麦芒|0.8086176514625549|
  |       青春|0.8035994172096252|
  |     Nova|0.7878020405769348|
  |mate10pro|0.7805472612380981|
  |      p10|0.7785272002220154|
  |      p20| 0.778145968914032|
  |       华为|0.7774022221565247|
  |   p20pro|0.7765063643455505|
  |  p10plus|0.7755928635597229|
  |      V10|  0.77093505859375|
  |   mate10|0.7679151892662048|
  |   nova2s|0.7678438425064087|
  |    mate8|0.7622326612472534|
  |    nova2|0.7591176629066467|
  |   Nova3i|0.7514565587043762|
  |   note10|0.7510180473327637|
  |     AL00|0.7456992864608765|
  |   nova3e| 0.744728684425354|
  +---------+------------------+
  ```

  ##### Word2Vec中两个重要模型（了解）

  - 原理：拥有差不多上下文的两个单词的意思往往是相近的

  - **Continuous Bag-of-Words(CBOW)**

    - 功能：通过上下文预测当前词出现的概率

    - 原理分析

      假设文本如下：“the florid <u>prose of</u> **the** <u>nineteenth century.</u>”

      想象有个滑动窗口，中间的词是关键词，两边为相等长度的文本来帮助分析。文本的长度为7，就得到了7个one-hot向量，作为神经网络的输入向量，训练目标是：最大化在给定前后文本情况下输出正确关键词的概率，比如给定("prose","of","nineteenth","century")的情况下，要最大化输出"the"的概率，用公式表示就是

      P("the"|("prose","of","nineteenth","century"))

    - 特性

      - hidden layer只是将权重求和，传递到下一层，是线性的

  - **Continuous Skip-gram**

    - 功能：根据当前词预测上下文
    - 原理分析
      - 和CBOW相反，则我们要求的概率就变为P(Context(w)|w)

- 总结：word2vec算法可以计算出每个词语的一个词向量，我们可以用它来表示该词的语义层面的含义

### 6.1 合并数据并分词

- 将sku信息合并到一起

```python
sku_detail = spark.sql("select * from sku_detail")
# sku_detail.show()
# 电子产品
electronic_product = sku_detail.where("category1_id<=5 and category1_id>=1")

from pyspark.sql.functions import concat_ws
# 合并多列数据
sentence_df = electronic_product.select("sku_id", "category1_id",\
    concat_ws(\
              ",", \
              electronic_product.category1, \
              electronic_product.category2, \
              electronic_product.category3, \
              electronic_product.name, \
              electronic_product.caption, \
              electronic_product.price, \
              electronic_product.specification\
             ).alias("summary")
)
```

- 分词

```python
def words(partitions):
    import os

    import jieba
    import jieba.posseg as pseg
    import codecs

    abspath = "/home/hadoop/"

    # 结巴加载用户词典
    userDict_path = os.path.join(abspath, "data/all.txt")
    jieba.load_userdict(userDict_path)

    # 停用词文本
    stopwords_path = os.path.join(abspath, "data/baidu_stopwords.txt")

    def get_stopwords_list():
        """返回stopwords列表"""
        stopwords_list = [i.strip()
                          for i in codecs.open(stopwords_path).readlines()]
        return stopwords_list

    # 所有的停用词列表
    stopwords_list = get_stopwords_list()

    def cut_sentence(sentence):
        """对切割之后的词语进行过滤，去除停用词，保留名词，英文和自定义词库中的词，长度大于2的词"""
        # print(sentence,"*"*100)
        # eg:[pair('今天', 't'), pair('有', 'd'), pair('雾', 'n'), pair('霾', 'g')]
        seg_list = pseg.lcut(sentence)
        seg_list = [i for i in seg_list if i.flag not in stopwords_list]
        filtered_words_list = []
        for seg in seg_list:
            # print(seg)
            if len(seg.word) <= 1:
                continue
            elif seg.flag == "eng":
                if len(seg.word) <= 2:
                    continue
                else:
                    filtered_words_list.append(seg.word)
            elif seg.flag.startswith("n"):
                filtered_words_list.append(seg.word)
            elif seg.flag in ["x", "eng"]:  # 是自定一个词语或者是英文单词
                filtered_words_list.append(seg.word)
        return filtered_words_list
    
    for row in partitions:
        yield (cut_sentence(row.summary),)

doc = sentence_df.rdd.mapPartitions(words)
doc = doc.toDF(["words"])
doc
```



### 6.2 Word2Vec模型训练

- 创建Word2Vec对象 并训练模型

```python
from pyspark.ml.feature import Word2Vec
#训练出的模型 词向量的维度是多少
word2Vec = Word2Vec(vectorSize=100, inputCol="words", outputCol="model")

model = word2Vec.fit(doc)
```

- Word2Vec模型使用

```python
from pyspark.sql.functions import format_number as fmt
#findSynonyms 找到跟‘笔记本’相似的20个词
#fmt 格式化, 保留5位有效数字
model.findSynonyms("笔记本",20).select("word",fmt("similarity",5).
alias("similarity")).show()
model.findSynonyms("荣耀", 20).show()
```

- 显示结果

``` shell
+-----------+----------+
|       word|similarity|
+-----------+----------+
|      笔记本电脑|   0.67314|
|        台式机|   0.63248|
|       台式电脑|   0.61965|
|     Laptop|   0.60712|
|      Air14|   0.58734|
|         墨舞|   0.58601|
|        XPS|   0.58482|
|        超轻薄|   0.58225|
|   ThinkPad|   0.57702|
|       adol|   0.57685|
|       V110|   0.57237|
|   MateBook|   0.57236|
|        UIY|   0.56889|
|  EliteBook|   0.56689|
|       X0AA|   0.56436|
|     Lenovo|   0.56426|
|       CUSO|   0.56023|
|         常压|   0.55886|
|      XPS13|   0.55733|
|ThinkCentre|   0.55493|
+-----------+----------+

+---------+------------------+
|     word|        similarity|
+---------+------------------+
|     play|0.8511283993721008|
|      v10|0.8239492177963257|
|       麦芒|0.8086176514625549|
|       青春|0.8035994172096252|
|     Nova|0.7878020405769348|
|mate10pro|0.7805472612380981|
|      p10|0.7785272002220154|
|      p20| 0.778145968914032|
|       华为|0.7774022221565247|
|   p20pro|0.7765063643455505|
|  p10plus|0.7755928635597229|
|      V10|  0.77093505859375|
|   mate10|0.7679151892662048|
|   nova2s|0.7678438425064087|
|    mate8|0.7622326612472534|
|    nova2|0.7591176629066467|
|   Nova3i|0.7514565587043762|
|   note10|0.7510180473327637|
|     AL00|0.7456992864608765|
|   nova3e| 0.744728684425354|
+---------+------------------+
```

### 6.3 保存模型

- 保存模型到hdfs上

```python
model.save("hdfs://localhost:9000/project2-meiduo-rs/models/电子产品.word2vec_model")
from pyspark.ml.feature import Word2VecModel
model = Word2VecModel.load("hdfs://localhost:9000/project2-meiduo-rs/models/电子产品.word2vec_model")
vectors = model.getVectors()
vectors
```

- 显示结果

```shell
DataFrame[word: string, vector: vector]
```

- 显示模型中的结果

```python
vectors.count()
```

显示结果

``` shell
18103
```

```python
vectors.head(100)
```

显示结果

```shell
[Row(word='钟爱', vector=DenseVector([-0.0042, -0.1172, 0.0594, 0.0564, -0.049, -0.0457, 0.0411, 0.1067, -0.0022, -0.0802, -0.0047, 0.0994, -0.0352, 0.0873, -0.0881, -0.0294, 0.044, 0.0465, 0.0676, 0.024, 0.0667, -0.0318, -0.1308, 0.0375, 0.062, -0.0315, 0.1234, 0.0362, -0.0514, 0.1064, -0.0207, -0.0979, 0.0573, -0.0375, -0.1729, 0.1023, 0.0792, 0.0389, 0.0515, 0.0372, 0.0506, -0.1057, 0.0342, -0.1603, 0.1454, 0.0428, -0.0016, -0.0058, -0.0609, -0.0648, -0.0104, -0.0365, -0.1157, 0.0437, 0.064, 0.0896, 0.0412, 0.048, 0.1512, -0.0691, -0.047, -0.1193, -0.0817, 0.0354, 0.2027, 0.0355, -0.0444, -0.0089, -0.0672, 0.0502, -0.062, -0.0532, -0.0295, -0.1187, -0.0217, 0.0015, 0.0039, -0.1415, 0.0448, 0.0175, -0.1728, -0.0365, -0.058, 0.0222, -0.1564, 0.1083, 0.0279, 0.0307, 0.0911, 0.0291, 0.0404, -0.0636, 0.1131, 0.0489, 0.0226, 0.0851, 0.1362, 0.091, -0.0671, -0.0095])),
 Row(word='伙伴', vector=DenseVector([-0.0746, -0.1684, -0.1954, 0.2206, -0.1033, -0.1933, 0.2222, 0.0283, -0.1717, -0.2701, 0.0136, 0.1946, 0.0564, 0.2032, 0.1733, 0.1299, -0.0916, 0.0493, 0.0804, -0.0972, 0.1325, -0.0258, 0.0042, 0.1495, 0.0971, -0.0784, 0.0907, -0.0505, 0.2231, -0.0511, 0.017, 0.0087, 0.1827, 0.0345, -0.2545, -0.0504, 0.1547, 0.1605, -0.2079, -0.1262, -0.0303, 0.0107, -0.0945, -0.131, 0.2295, 0.0171, -0.348, 0.0882, 0.1833, -0.3242, 0.0511, 0.1928, 0.2684, 0.0111, -0.0415, 0.1187, -0.0957, 0.1475, 0.1049, -0.036, -0.2077, -0.1302, -0.2307, -0.018, -0.0608, 0.0445, -0.0832, 0.109, -0.1278, 0.3188, -0.0544, -0.0383, -0.1464, -0.0834, -0.1821, -0.1381, -0.0254, -0.1361, 0.1039, -0.0894, -0.0167, 0.0282, -0.1594, 0.1166, -0.0241, 0.016, 0.0903, 0.0703, 0.0231, -0.0764, -0.0044, 0.0959, 0.2174, 0.0524, -0.3269, 0.1416, -0.1383, 0.1814, -0.0663, 0.0594])),
 Row(word='uhd566', vector=DenseVector([0.1238, -0.0663, -0.0386, 0.0775, 0.0022, 0.0395, -0.0613, -0.0223, 0.263, -0.002, 0.0239, 0.0319, -0.0432, -0.0152, 0.0922, -0.0286, -0.0906, -0.1479, 0.0675, 0.0472, -0.0331, -0.1508, 0.0504, 0.061, -0.1183, -0.0068, -0.0457, 0.0152, -0.0443, 0.0346, 0.0911, 0.1484, 0.0277, 0.1795, -0.1104, 0.0219, -0.0937, -0.0436, 0.0251, 0.073, -0.0405, 0.0311, 0.2154, -0.0432, 0.0041, 0.0263, -0.2014, -0.0253, 0.0003, 0.0992, -0.026, 0.0727, 0.1734, 0.0674, -0.012, 0.1411, 0.1518, 0.006, 0.0182, -0.0038, 0.0228, 0.0746, -0.0456, -0.0468, -0.0329, 0.0693, -0.0815, -0.041, 0.0038, 0.0035, 0.028, -0.061, -0.044, -0.1879, -0.0387, -0.0837, 0.0671, 0.0075, -0.0778, -0.2301, -0.1174, -0.0614, -0.0083, 0.0501, -0.0658, 0.1055, -0.0899, 0.0025, -0.1419, 0.0972, 0.0204, -0.1028, -0.0501, -0.1447, 0.0417, -0.0065, -0.1218, 0.0964, 0.1925, -0.0311])),
 ..... ....
 ]
```

