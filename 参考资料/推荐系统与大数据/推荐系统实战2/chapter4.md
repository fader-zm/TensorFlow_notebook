## 四 商品关键词提取

- 根据商品SKU详情表, 对商品进行关键词提取, 关键词提取的第一步就是要分词 

### 4.1 jieba分词

- 停用词

  - 分词之后要去掉的词

- 载入词典

  - 自定义的词典，以便包含 jieba 词库里没有的词。虽然 jieba 有新词识别能力，但是自行添加新词可以保证更高的正确率
  - 用法： jieba.load_userdict(file_name) # file_name 为文件类对象或自定义词典的路径
  - 词典格式和 `dict.txt` 一样，一个词占一行；每一行分三部分：词语、词频（可省略）、词性（可省略），用空格隔开，顺序不可颠倒。`file_name` 若为路径或二进制方式打开的文件，则文件必须为 UTF-8 编码。
  - 词频省略时使用自动计算的能保证分出该词的词频。
  - 自定义词典示例

  ```shell
  创新办 3 i
  云计算 5
  凱特琳 nz
  台中
  ```

- 词性

  https://www.cnblogs.com/csj007523/p/7773027.html

```python
import os
import jieba
import jieba.posseg as pseg
import codecs

abspath = "/home/hadoop/"
userDict_path = os.path.join(abspath,'data/all.txt') # 用户自定义词典
# jieba加载用户词典
jieba.load_userdict(userDict_path)
stopwords_path = os.path.join(abspath,'data/baidu_stopwords.txt') #停用词词典

def get_stopwords_list():
    """返回stopwords列表"""
    stopwords_list = [i.strip()
                      for i in codecs.open(stopwords_path).readlines()]
    return stopwords_list

# 所有的停用词列表
stopwords_list = get_stopwords_list()

# 分词
def cut_sentence(sentence):
    """对切割之后的词语进行过滤，去除停用词，保留名词，英文和自定义词库中的词，长度大于2的词"""
    # print(sentence,"*"*100)
    # eg:[pair('今天', 't'), pair('有', 'd'), pair('雾', 'n'), pair('霾', 'g')]
    seg_list = pseg.lcut(sentence)
    #过滤停用词
    seg_list = [i for i in seg_list if i.word not in stopwords_list]
    filtered_words_list = []
    for seg in seg_list:
        # print(seg)
        if len(seg.word) <= 1:
            continue
        elif seg.flag == "eng":
            if len(seg.word) <= 2:
                # 如果是英文 长度<=2的过滤掉
                continue
            else:
                filtered_words_list.append(seg.word)
        elif seg.flag.startswith("n"):
            # 如果是名词的 加入到分词的结果中
            filtered_words_list.append(seg.word)
        elif seg.flag in ["x", "eng"]:  # 是自定一个词语或者是英文单词
            # 是自定一个词语或者是英文单词
            filtered_words_list.append(seg.word)
    return filtered_words_list
```

- 利用jieba分词

```python
sentence = '''
攀升（IPASON）H68 i7 8700K/Z370/GTX1070Ti 8G/16G DDR4水冷游戏台式DIY组装电脑京东自营游戏主机UPC京东自营，自带内存】OMG战队训练指定用机，自带16G内存，GTX1070Ti 8G！六核处理器+GTX1050Ti游戏主机，点此抢购
'''
cut_sentence(sentence)
```

- 显示结果

```shell
['IPASON',
 'H68',
 'i7',
 '8700K',
 'Z370',
 'GTX1070Ti',
 '16G',
 'DDR4',
 '水冷',
 '游戏',
 '台式',
 'DIY',
 '电脑',
 '京东',
 '游戏',
 'UPC',
 '京东',
 '自带',
 '内存',
 'OMG',
 '战队',
 '自带',
 '16G',
 '内存',
 'GTX1070Ti',
 '处理器',
 'GTX1050Ti',
 '游戏']
```

### 4.2 商品归类

- 对于电商来说，同一个关键词在不同品类数据之间通常都具有不同的含义，往往需要根据品类特性，分别进行关键词的解析提取。当我们进行商品关键词提取等处理时，都是要根据不同的类别进行独立的处理
  - 电子产品中的“苹果”和生鲜中的“苹果”，意义不同
  - 推荐类似物品时，通常也是推荐相同类别的产品：用户正在浏览图书，那么相关的物品推荐，通常是不会出现食品的

- 查看当前商品数据所有一级分类

```python
spark.sql("select * from tb_goods_category where parent_id is null").show(100)
```

- 显示结果

```shell
+---+--------------------+--------------------+----+---------+
| id|         create_time|         update_time|name|parent_id|
+---+--------------------+--------------------+----+---------+
|  1|2018-04-09 08:03:...|2018-04-09 08:03:...|  手机|     null|
|  2|2018-04-09 08:04:...|2018-04-09 08:04:...|  相机|     null|
|  3|2018-04-09 08:04:...|2018-04-09 08:04:...|  数码|     null|
|  4|2018-04-09 08:05:...|2018-04-09 08:05:...|  电脑|     null|
|  5|2018-04-09 08:05:...|2018-04-09 08:05:...|  办公|     null|
|  6|2018-04-09 08:05:...|2018-04-09 08:05:...|家用电器|     null|
|  7|2018-04-09 08:05:...|2018-04-09 08:05:...|  家居|     null|
|  8|2018-04-09 08:05:...|2018-04-09 08:05:...|  家具|     null|
|  9|2018-04-09 08:05:...|2018-04-09 08:05:...|  家装|     null|
| 10|2018-04-09 08:05:...|2018-04-09 08:05:...|  厨具|     null|
| 11|2018-04-09 08:06:...|2018-04-09 08:06:...|  男装|     null|
| 12|2018-04-09 08:06:...|2018-04-09 08:06:...|  女装|     null|
| 13|2018-04-09 08:06:...|2018-04-09 08:06:...|  童装|     null|
| 14|2018-04-09 08:06:...|2018-04-09 08:06:...|  内衣|     null|
| 15|2018-04-09 08:06:...|2018-04-09 08:06:...|  女鞋|     null|
| 16|2018-04-09 08:08:...|2018-04-09 08:08:...|  箱包|     null|
| 17|2018-04-09 08:09:...|2018-04-09 08:09:...|  钟表|     null|
| 18|2018-04-09 08:09:...|2018-04-09 08:09:...|  珠宝|     null|
| 19|2018-04-09 08:09:...|2018-04-09 08:09:...|  男鞋|     null|
| 20|2018-04-09 08:09:...|2018-04-09 08:09:...|  运动|     null|
| 21|2018-04-09 08:09:...|2018-04-09 08:09:...|  户外|     null|
| 22|2018-04-09 08:11:...|2018-04-09 08:11:...|  房产|     null|
| 23|2018-04-09 08:11:...|2018-04-09 08:11:...|  汽车|     null|
| 24|2018-04-09 08:11:...|2018-04-09 08:11:...|汽车用品|     null|
| 25|2018-04-09 08:11:...|2018-04-09 08:11:...|  母婴|     null|
| 26|2018-04-09 08:11:...|2018-04-09 08:11:...|玩具乐器|     null|
| 27|2018-04-09 08:56:...|2018-04-09 08:56:...|  食品|     null|
| 28|2018-04-09 08:56:...|2018-04-09 08:56:...|  酒类|     null|
| 29|2018-04-09 08:56:...|2018-04-09 08:56:...|  生鲜|     null|
| 30|2018-04-09 08:56:...|2018-04-09 08:56:...|  特产|     null|
| 31|2018-04-09 08:56:...|2018-04-09 08:56:...|  图书|     null|
| 32|2018-04-09 08:56:...|2018-04-09 08:56:...|  音像|     null|
| 33|2018-04-09 08:56:...|2018-04-09 08:56:...| 电子书|     null|
| 34|2018-04-09 08:56:...|2018-04-09 08:56:...|  机票|     null|
| 35|2018-04-09 08:56:...|2018-04-09 08:56:...|  酒店|     null|
| 36|2018-04-09 08:56:...|2018-04-09 08:56:...|  旅游|     null|
| 37|2018-04-09 08:57:...|2018-04-09 08:57:...|  生活|     null|
+---+--------------------+--------------------+----+---------+
```

- 但如何具体给商品归类，其实不是由我们随意决定的，而是由产品设计等人员针对产品特性来划分，以下划分仅供参考：

  - 电子产品: 手机、相机、数码、电脑、办公   ==>   1-5
  - 家居产品：家用电器、家居、家具、家装、厨具   ==>   6-10
  - 服饰产品：男装、女装、童装、内衣、女鞋、箱包、钟表、珠宝、男鞋、运动、户外   ==>   11-21
  - 资产产品：房产、汽车、汽车用品   ==>   22-24
  - 母婴用品：母婴、玩具乐器   ==>   25-26
  - 食用产品：食品、酒类、生鲜、特产   ==>   27-30
  - 影音图书产品：图书、音像、电子书   ==>   31-33
  - 旅游出行产品：机票、酒店、旅游、生活   ==>   34-37

  所以这里我们需要对这几大类别的商品，分别进行关键词的提取工作

- 查询sku详情

```python
sku_detail = spark.sql("select * from sku_detail")
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
only showing top 20 rows
```

- 以电子产品为例对一级分类id是1到5的商品，即电子产品进行关键词提取

```python
# 电子产品
electronic_product = sku_detail.where("category1_id<=5 and category1_id>=1")
electronic_product.show()
```

显示结果

``` shell
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
only showing top 20 rows
```

- 数据整合 为了让数据表现出足够的特征，这里我们把一个商品所有的详细信息都拼接为一个长文本字符串

```python
# 利用concat_ws方法，将多列数据合并为一个长文本内容
from pyspark.sql.functions import concat_ws
# 合并多列数据
# summary字段就是拼接的长文本字符串
ret = electronic_product.select("goods_id", "sku_id", "category1_id", "category2_id", "category3_id",\
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
ret.show()
```

- 显示结果

```shell
+--------+------+------------+------------+------------+--------------------+
|goods_id|sku_id|category1_id|category2_id|category3_id|             summary|
+--------+------+------------+------------+------------+--------------------+
|     135|   148|           3|          41|         140|数码,数码配件,读卡器,随身厅 W...|
|     451|   463|           3|          41|         140|数码,数码配件,读卡器,飞花令 安...|
|     458|   471|           3|          41|         140|数码,数码配件,读卡器,【包邮】飞...|
|     483|   496|           3|          41|         140|数码,数码配件,读卡器,品胜（PI...|
|     820|   833|           3|          41|         140|数码,数码配件,读卡器,LEXAR...|
|    1075|  1088|           2|          40|         135|相机,摄影摄像,数码相框,青美 壁...|
|    1225|  1238|           3|          41|         140|数码,数码配件,读卡器,dypla...|
|    1329|  1342|           3|          41|         140|数码,数码配件,读卡器,绿联（UG...|
|    1567|  1580|           2|          40|         135|相机,摄影摄像,数码相框,HNM ...|
|    1578|  1591|           3|          41|         140|数码,数码配件,读卡器,kisdi...|
|    1632|  1645|           2|          40|         135|相机,摄影摄像,数码相框,爱国者（...|
|    1816|  1829|           3|          41|         140|数码,数码配件,读卡器,金士顿（K...|
|    1946|  1959|           2|          40|         128|相机,摄影摄像,数码相机,理光（R...|
|    2109|  2122|           1|          39|         126|手机,手机配件,移动电源,贝视特苹...|
|    2129|  2142|           1|          39|         126|手机,手机配件,移动电源,戈派 无...|
|    2353|  2366|           1|          39|         126|手机,手机配件,移动电源,赋电 充...|
|    2646|  2659|           1|          39|         126|手机,手机配件,移动电源,OISL...|
|    2853|  2866|           1|          38|         118|手机,手机通讯,对讲机,宝锋（BA...|
|    3162|  3175|           1|          38|         118|手机,手机通讯,对讲机,Motor...|
|    3736|  3749|           1|          38|         118|手机,手机通讯,对讲机,ZASTO...|
+--------+------+------------+------------+------------+--------------------+
only showing top 20 rows
```

### 4.3 基于TextRank提取关键词 

- TextRank原理简介
  - TextRank是基于PageRank算法的一种用来做关键词提取的算法，也可以用于提取短语和自动摘要
    - PageRank设计之初是用于计算Google的网页排名
    - 以该公司创办人拉里·佩奇（Larry Page）之姓来命名
    - 体现网页的相关性和重要性，在搜索引擎优化操作中经常被用来评估网页优化的成效因素之一
    - PageRank通过互联网中的超链接关系来确定一个网页的排名，其公式是通过一种投票的思想来设计的：如果我们要计算网页A的PageRank值（以下简称PR值），那么我们需要知道有哪些网页链接到网页A，也就是要首先得到网页A的入链，然后通过入链给网页A的投票来计算网页A的PR值。
    - 当某些高质量的网页指向网页A的时候，那么网页A的PR值会因为这些高质量的投票而变大
    - 网页A被较少网页指向或被一些PR值较低的网页指向的时候,A的PR值也不会很大，这样可以合理地反映一个网页的质量水平
  - [TextRank ](http://web.eecs.umich.edu/~mihalcea/papers/mihalcea.emnlp04.pdf)算法是一种用于文本的基于图的排序算法。其基本思想来源于谷歌的 [PageRank](http://ilpubs.stanford.edu:8090/422/1/1999-66.pdf)算法, 通过把文本分割成若干组成单元(单词、句子)并建立图模型, 利用局部词汇之间关系（共现窗口）和投票机制对文本中的重要成分进行排序, 仅利用单篇文档本身的信息即可实现关键词提取、文摘。 TextRank不需要事先对多篇文档进行学习训练, 因其简洁有效而得到广泛应用。



- 使用jieba中文分词自带的textrank方法进行处理

```python
ret.where("category1_id=4").first()
```

显示结果

``` shell
Row(goods_id=29588, sku_id=29601, category1_id=4, category2_id=45, category3_id=158, summary='电脑,电脑整机,游戏本,戴尔DELL灵越游匣Master15.6英寸游戏笔记本电脑(i5-7300HQ 8G 128GSSD+1T GTX1050Ti 4G独显)红,【GTX1050Ti 4G独显】帧率高稳定性强运行更畅快,IPS防眩光显示屏全面还原游戏战场！,7099.0,版本:游戏笔记本电脑,颜色:i5 8G GTX1050Ti PCIe 黑,颜色:i5 8G GTX1050Ti 白,颜色:i5 8G GTX1050Ti 高色域,颜色:i5 8G GTX1060 6G PCIe 黑,颜色:i5 8G GTX1060 6G 白,颜色:i5 8G GTX1060 6G 高色域,颜色:i5 8G GTX1060 6G 黑,颜色:i5-7300HQ 128G+1T GTX1050Ti 红,颜色:i7 16G GTX1060 白,颜色:i7 8G GTX1050Ti 白,颜色:i7 GTX1050Ti 高色域,颜色:i7 GTX1060 高色域,颜色:i9 16G GTX1060 白')
```

- 结巴使用示范
  - 创建一个类继承 jieba.analyse.TextRank
  - 重写pairfilter函数 在这里做词语过滤
  - 创建对象 调用textrank方法

```python
import os

# if os.path.exists("/tmp/jieba.cache"):
#     print("exists")
#     os.remove("/tmp/jieba.cache")

import jieba
import jieba.analyse
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

class TextRank(jieba.analyse.TextRank):
    def __init__(self, window=20, word_min_len=2):
        super(TextRank, self).__init__()
        self.span = window  # 窗口大小
        self.word_min_len = word_min_len  # 单词的最小长度
        # 要保留的词性，根据jieba github ，具体参见https://github.com/baidu/lac
        self.pos_filt = frozenset(
            ('n', 'x', 'eng', 'f', 's', 't', 'nr', 'ns', 'nt', "nw", "nz", "PER", "LOC", "ORG"))

    def pairfilter(self, wp):
        """过滤条件，返回True或者False"""

        if wp.flag == "eng":
            if len(wp.word) <= 2:
                return False
		#如果不是停用词 词性输入保留列表中的 切单词长度>2 这些词都留下
        if wp.flag in self.pos_filt and len(wp.word.strip()) >= self.word_min_len \
                and wp.word.lower() not in stopwords_list:
            return True

text =  '''电脑,电脑整机,游戏本,戴尔DELL灵越游匣Master15.6英寸游戏笔记本电脑(i5-7300HQ 8G 128GSSD+1T GTX1050Ti 4G独显)红,【GTX1050Ti 4G独显】帧率高稳定性强运行更畅快,IPS防眩光显示屏全面还原游戏战场！,7099.0,版本:游戏笔记本电脑,颜色:i5 8G GTX1050Ti PCIe 黑,颜色:i5 8G GTX1050Ti 白,颜色:i5 8G GTX1050Ti 高色域,颜色:i5 8G GTX1060 6G PCIe 黑,颜色:i5 8G GTX1060 6G 白,颜色:i5 8G GTX1060 6G 高色域,颜色:i5 8G GTX1060 6G 黑,颜色:i5-7300HQ 128G+1T GTX1050Ti 红,颜色:i7 16G GTX1060 白,颜色:i7 8G GTX1050Ti 白,颜色:i7 GTX1050Ti 高色域,颜色:i7 GTX1060 高色域,颜色:i9 16G GTX1060 白'''       
    
textrank_model = TextRank(window=10, word_min_len=2)
allowPOS = ('n', "x", 'eng', 'nr', 'ns', 'nt', "nw", "nz", "c")
tags = textrank_model.textrank(text, topK=20, withWeight=True, allowPOS=allowPOS, withFlag=False)
print(tags)        
```

- 使用TextRank提取所有关键词

``` python
from functools import partial

def _mapPartitions(partition, industry):
    
    import os

    import jieba
    import jieba.analyse
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

    class TextRank(jieba.analyse.TextRank):
        def __init__(self, window=20, word_min_len=2):
            super(TextRank, self).__init__()
            self.span = window  # 窗口大小
            self.word_min_len = word_min_len  # 单词的最小长度
            # 要保留的词性，根据jieba github ，具体参见https://github.com/baidu/lac
            self.pos_filt = frozenset(
                ('n', 'x', 'eng', 'f', 's', 't', 'nr', 'ns', 'nt', "nw", "nz", "PER", "LOC", "ORG"))

        def pairfilter(self, wp):
            """过滤条件，返回True或者False"""

            if wp.flag == "eng":
                if len(wp.word) <= 2:
                    return False

            if wp.flag in self.pos_filt and len(wp.word.strip()) >= self.word_min_len \
                    and wp.word.lower() not in stopwords_list:
                return True
    textrank_model = TextRank(window=10, word_min_len=2)
    allowPOS = ('n', "x", 'eng', 'nr', 'ns', 'nt', "nw", "nz", "c")
    
    for row in partition:
        tags = textrank_model.textrank(row.summary, topK=20, withWeight=True, allowPOS=allowPOS, withFlag=False)
        for tag in tags:
            yield row.sku_id, industry, tag[0], tag[1]

mapPartitions = partial(_mapPartitions, industry="电子产品")

sku_tag_weights = ret.rdd.mapPartitions(mapPartitions)
sku_tag_weights = sku_tag_weights.toDF(["sku_id", "industry", "tag","weights"])
sku_tag_weights  
```

显示结果

```shell
DataFrame[sku_id: bigint, industry: string, tag: string, weights: double]
```

- textrank结果显示

```python
sku_tag_weights.where("tag='i5'").orderBy("weights").show()
```

显示结果

``` shell
+------+--------+---+-------------------+
|sku_id|industry|tag|            weights|
+------+--------+---+-------------------+
| 42090|    电子产品| i5|0.08546975192173897|
| 43265|    电子产品| i5|0.10551986636215893|
| 37795|    电子产品| i5|0.11665840899649133|
| 31619|    电子产品| i5|0.11913525223903051|
| 42450|    电子产品| i5|0.11919319831950932|
| 29746|    电子产品| i5|0.12156392019404358|
| 42539|    电子产品| i5|0.12200729167516883|
| 37640|    电子产品| i5|0.12349789949353178|
| 37711|    电子产品| i5|0.12349789949353178|
| 37636|    电子产品| i5|0.12349789949353178|
| 37664|    电子产品| i5|0.12349789949353178|
| 37489|    电子产品| i5|0.12403634476677387|
| 37288|    电子产品| i5|0.12502307694727494|
| 37270|    电子产品| i5| 0.1259262008416576|
| 36980|    电子产品| i5| 0.1259262008416576|
| 37342|    电子产品| i5| 0.1268697472853674|
| 37224|    电子产品| i5| 0.1268697472853674|
| 36332|    电子产品| i5| 0.1268697472853674|
| 43205|    电子产品| i5|0.12737637312201944|
| 37159|    电子产品| i5|0.12784211950916827|
+------+--------+---+-------------------+
only showing top 20 rows
```

### 4.4 将提取的关键词存入hive中

- 显示出分词数量结果

``` python
sku_tag_weights.count()
```

- 显示结果

``` shell
1174703
```

- 创建临时表

```python
sku_tag_weights.registerTempTable("tempTable")
spark.sql("desc tempTable").show()
```

显示结果

``` shell
+--------+---------+-------+
|col_name|data_type|comment|
+--------+---------+-------+
|  sku_id|   bigint|   null|
|industry|   string|   null|
|     tag|   string|   null|
| weights|   double|   null|
+--------+---------+-------+
```

- 创建hive表

```python
sql = """CREATE TABLE IF NOT EXISTS sku_tag_weights(
sku_id INT,
industry STRING,
tag STRING,
weights DOUBLE
)"""
spark.sql(sql)
```

- 从临时表中查询数据插入表中

``` python
spark.sql("INSERT INTO sku_tag_weights SELECT * FROM tempTable")
spark.sql("SELECT * FROM sku_tag_weights").show()
```

显示结果

```shell
+------+--------+----+-------------------+
|sku_id|industry| tag|            weights|
+------+--------+----+-------------------+
|   148|    电子产品|  高度|                1.0|
|   148|    电子产品|  终端| 0.9880227098783998|
|   148|    电子产品|WPOS| 0.9474997114660046|
|   148|    电子产品|  身份| 0.8859077272889758|
|   148|    电子产品| 触摸屏| 0.8677093692671172|
|   148|    电子产品|  森锐| 0.8643336666795045|
|   148|    电子产品| 收银机| 0.8606879409613701|
|   148|    电子产品|  智能|  0.853869407929065|
|   148|    电子产品|  业务| 0.8500761345577008|
|   148|    电子产品| 读卡器| 0.6114834155265747|
|   148|    电子产品|  正品| 0.5977867682869656|
|   148|    电子产品|  包邮| 0.5962385663111377|
|   148|    电子产品|数码配件| 0.4880302016489459|
|   148|    电子产品|  数码|0.48644615569271077|
|   148|    电子产品|  购物|  0.442940968516803|
|   463|    电子产品| 读卡器|                1.0|
|   463|    电子产品|  颜色| 0.7811873889257711|
|   463|    电子产品|  安卓| 0.5651375469154993|
|   463|    电子产品|  手机|  0.458740094560001|
|   463|    电子产品|  电脑| 0.4219429919380977|
+------+--------+----+-------------------+
only showing top 20 rows
```

### 4.5 基于TFIDF提取关键词

- 由于TFIDF的求值需要根据全体数据进行处理，这里将使用spark中TFIDF的相关模块

  先分词，然后分别计算词的TF和IDF值

  TF = 当前文档某关键词的个数/当前文档的关键词总个数   单词在文档中出现的频率

  - 如某文档共有100个词(含重复)，其中”python“出现了5次，那么该文档中”python“的TF值为：5/100 = 0.05

  IDF = log(总文档个数/(含有某关键词的文档个数+1))，这里+1是为了防止分母为0

  - 如共100篇文档，其中5篇含有”python“，那么"python"的IDF值：math.log(100/6) = 2.81

  TFIDF = TF * IDF

- ```python
  import math
  math.log(100/6)
  ```


- 计算TF 需要先计算词语出现的次数 spark 中提供了CountVectorizer来统计词语出现次数

- 基于CountVectorizer的结果， 可以利用Spark 提供的IDF模块计算 IDF值
- TF= CountVectorizer统计出的词语出现次数/文档长度
- 将上两步的结果相乘得到TFIDF值

**CountVectorizer使用介绍**

① 创建DataFrame 准备数据 

② 创建CountVectorizer

③ 调用fit函数创建模型

④ 通过transform获取结果

```python
from pyspark.ml.feature import CountVectorizer

# CountVectorizer对数据集中的单词进行个数统计
# Input data: Each row is a bag of words with a ID.
df = spark.createDataFrame([
    (0, "a b c g h".split(" ")),
    (1, "a b b c a d e f".split(" "))
], ["id", "words"])

# fit a CountVectorizerModel from the corpus.
# vocabSize: 最多保留的单词个数
# minDF：最小的出现次数，即词频
cv = CountVectorizer(inputCol="words", outputCol="features", vocabSize=100, minDF=1.0)

model = cv.fit(df)
print("数据集中的词：", model.vocabulary)

result = model.transform(df)
result.show(truncate=False)
```

显示结果:

``` shell
数据集中的词： ['a', 'b', 'c', 'f', 'g', 'd', 'e', 'h']
+---+------------------------+-------------------------------------------+
|id |words                   |features                                   |
+---+------------------------+-------------------------------------------+
|0  |[a, b, c, g, h]         |(8,[0,1,2,4,7],[1.0,1.0,1.0,1.0,1.0])      |
|1  |[a, b, b, c, a, d, e, f]|(8,[0,1,2,3,5,6],[2.0,2.0,1.0,1.0,1.0,1.0])|
+---+------------------------+-------------------------------------------+
```

- 分词并统计个数函数

```python
def words(partitions):
    # 先分词
    
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
        yield row.sku_id, cut_sentence(row.summary)

doc = ret.rdd.mapPartitions(words)
doc = doc.toDF(["sku_id", "words"])
doc
```

- 使用CountVectorizer 统计词频

```python
from pyspark.ml.feature import CountVectorizer

# 6w * 20
# 这里我们将所有出现过的词都统计出来，这里最多会有6w * 20个词
cv = CountVectorizer(inputCol="words", outputCol="rawFeatures", vocabSize=60000*20, minDF=1.0)

cv_model = cv.fit(doc)
cv_result= cv_model.transform(doc)
cv_result.show()
```

- 终端显示（耗时较长）

```shell
+------+--------------------+--------------------+
|sku_id|               words|         rawFeatures|
+------+--------------------+--------------------+
|   148|[数码, 数码配件, 读卡器, W...|(42513,[7,10,36,9...|
|   463|[数码, 数码配件, 读卡器, 飞...|(42513,[0,2,3,5,1...|
|   471|[数码, 数码配件, 读卡器, 包...|(42513,[0,1,5,10,...|
|   496|[数码, 数码配件, 读卡器, 品...|(42513,[0,5,10,13...|
|   833|[数码, 数码配件, 读卡器, L...|(42513,[1,10,36,5...|
|  1088|[摄影, 数码相框, 青美, 壁挂...|(42513,[0,1,9,48,...|
|  1238|[数码, 数码配件, 读卡器, d...|(42513,[10,22,36,...|
|  1342|[数码, 数码配件, 读卡器, 绿...|(42513,[0,5,10,36...|
|  1580|[摄影, 数码相框, HNM, 英...|(42513,[1,2,4,9,1...|
|  1591|[数码, 数码配件, 读卡器, k...|(42513,[1,3,10,36...|
|  1645|[摄影, 数码相框, 爱国者, a...|(42513,[1,4,17,20...|
|  1829|[数码, 数码配件, 读卡器, 金...|(42513,[0,10,36,5...|
|  1959|[摄影, 数码相机, 理光, Ri...|(42513,[0,6,9,13,...|
|  2122|[手机, 手机配件, 移动电源, ...|(42513,[0,5,22,24...|
|  2142|[手机, 手机配件, 移动电源, ...|(42513,[0,5,22,23...|
|  2366|[手机, 手机配件, 移动电源, ...|(42513,[0,1,2,5,1...|
|  2659|[手机, 手机配件, 移动电源, ...|(42513,[0,1,2,5,9...|
|  2866|[手机, 手机, 通讯, 对讲机,...|(42513,[0,5,21,43...|
|  3175|[手机, 手机, 通讯, 对讲机,...|(42513,[0,5,21,23...|
|  3749|[手机, 手机, 通讯, 对讲机,...|(42513,[0,5,11,11...|
+------+--------------------+--------------------+
only showing top 20 rows
```

- 显示词频统计结果

```python
print(cv_model.vocabulary)
len(cv_model.vocabulary)
```

- 显示结果

```shell
['颜色', '版本', '黑色', '电脑', '英寸', '手机', '套装', '智能', '办公', '白色', '数码', '套餐', '鼠标', '京东', '内存', '游戏', '型号', '高清', '键盘', '产品', 'i5', '官方', '耳机', '苹果', '无线', '容量', '金属', '原装', 'i7', '文具', '电脑配件', '固态', '蓝色', '小米', '支架', '手机配件', '平台', '平板', '读卡器', '华为', '耗材', '主板', '蓝牙', '镜头', '尺寸', '红色', '下单', '银色', '机械', '硬盘', '摄影', 'U盘', '经典', '数据线', '安卓', 'USB', '钢化', '充电器', '手环', '电池', '内存卡', '摄像头', '家用', '墨盒', '电子', '笔记本', '金色', 'CPU', '麦克风', '客服', '电源', '粉色', '荣耀', '商品', '秒杀', 'DDR4', '打印机', '新品', '专业', 'HDMI', '佳能', '专用', '鼠标垫', '手表', '学生', '商务', 'USB3', '配件', '台式机', '领券', '彩色', '网通', 'WIFI', 'Type', '存储卡', '影音', '计算器', '语音', '键鼠套', '正品', 'RGB', '静音', '华硕', '保护套', '话筒', 'i3', '牧马人', '办公设备', '新款', '青轴', '显示器', '蓝光', '机器人', '酷睿', '儿童',......]

42443
```

### 4.7 IDF值计算

```python
from pyspark.ml.feature import IDF
idf = IDF(inputCol="rawFeatures", outputCol="features")

idfModel = idf.fit(cv_result)
rescaledData = idfModel.transform(cv_result)

rescaledData.select("words", "features").show()
```

- 显示结果

``` shell
+--------------------+--------------------+
|               words|            features|
+--------------------+--------------------+
|[数码, 数码配件, 读卡器, W...|(42443,[7,10,38,9...|
|[数码, 数码配件, 读卡器, 飞...|(42443,[0,2,3,5,1...|
|[数码, 数码配件, 读卡器, 包...|(42443,[0,1,5,10,...|
|[数码, 数码配件, 读卡器, 品...|(42443,[0,5,10,13...|
|[数码, 数码配件, 读卡器, L...|(42443,[1,10,38,5...|
|[摄影, 数码相框, 青美, 壁挂...|(42443,[0,1,9,50,...|
|[数码, 数码配件, 读卡器, d...|(42443,[10,23,38,...|
|[数码, 数码配件, 读卡器, 绿...|(42443,[0,5,10,38...|
|[摄影, 数码相框, HNM, 英...|(42443,[1,2,4,9,1...|
|[数码, 数码配件, 读卡器, k...|(42443,[1,3,10,38...|
|[摄影, 数码相框, 爱国者, a...|(42443,[1,4,17,21...|
|[数码, 数码配件, 读卡器, 金...|(42443,[0,10,38,5...|
|[摄影, 数码相机, 理光, Ri...|(42443,[0,6,9,13,...|
|[手机, 手机配件, 移动电源, ...|(42443,[0,5,23,25...|
|[手机, 手机配件, 移动电源, ...|(42443,[0,5,23,24...|
|[手机, 手机配件, 移动电源, ...|(42443,[0,1,2,5,1...|
|[手机, 手机配件, 移动电源, ...|(42443,[0,1,2,5,9...|
|[手机, 手机, 通讯, 对讲机,...|(42443,[0,5,22,45...|
|[手机, 手机, 通讯, 对讲机,...|(42443,[0,5,22,24...|
|[手机, 手机, 通讯, 对讲机,...|(42443,[0,5,11,11...|
+--------------------+--------------------+
only showing top 20 rows
```

- 利用idf属性，获取每一个词的idf值，这里每一个值与cv_model.vocabulary中的词一一对应

```python
idfModel.idf.toArray()
```

- 显示结果

``` shell
array([ 0.24258429,  1.25958267,  1.40733883, ..., 10.41409315,
       10.41409315, 10.41409315])
```

- 将关键词和idf结果放在一起

```python
keywords_list_with_idf = list(zip(cv_model.vocabulary, idfModel.idf.toArray()))
keywords_list_with_idf
```

- 显示结果

``` shell
[('颜色', 0.2425842894701456),
 ('版本', 1.2595826650830566),
 ('黑色', 1.407338832221065),
 ('电脑', 0.9269926353711626),
 ('英寸', 1.9777846894227953),
 ('手机', 1.3972161773351852),
 ('套装', 2.1746317022809443),
 ('智能', 1.938138708698929),
...   ......
]
```

### 4.8 TFIDF值计算

- 展示一行数据

```python
row = rescaledData.first()
row
```

- 终端显示

```python
Row(words=['数码', '数码配件', '读卡器', 'WPOS', '高度', '业务', '智能', '终端', '森锐', '触摸屏', '收银机', '身份', '包邮', '正品', '购物'], rawFeatures=SparseVector(42443, {7: 1.0, 10: 1.0, 38: 1.0, 99: 1.0, 195: 1.0, 216: 1.0, 356: 1.0, 422: 1.0, 647: 1.0, 2923: 1.0, 4425: 1.0, 7473: 1.0, 13946: 1.0, 14562: 1.0, 24286: 1.0}), features=SparseVector(42443, {7: 1.9381, 10: 1.4125, 38: 3.4038, 99: 2.9766, 195: 3.4514, 216: 3.6123, 356: 4.1244, 422: 4.7829, 647: 5.8446, 2923: 6.8587, 4425: 7.7399, 7473: 8.0627, 13946: 9.1613, 14562: 9.1613, 24286: 10.0086}))
```

- row.rawFeatures是一个向量类型

```python
# row.rawFeatures.indices中每一个元素是当前行每一个关键词 在所有关键词表中的索引
row.rawFeatures.indices
```

- 显示结果

``` shell
array([    7,    10,    38,    99,   195,   216,   356,   422,   647,
        2923,  4425,  7473, 13946, 14562, 24286], dtype=int32)
```

- 统计一行有几个关键词

```python
row.rawFeatures.values #当前行所有关键词出现的次数的集合
row.rawFeatures.values.sum() #可以计算出一行有多少个关键词
```

显示结果

```shell
array([1., 1., 1., 1., 1., 1., 1., 1., 1., 1., 1., 1., 1., 1., 1.])
```



``` python
row.rawFeatures[7] #获取第7个关键词出现的次数
```

显示结果

``` shell
1.0
```

- tfidf统计结果思路分析
  - 我们已经 处理好了两份数据 keywords_list_with_idf（关键词和idf结果合并后的数据） cv_result （词频统计的结果）如何通过这两份数据计算TF-IDF值
    - keywords_list_with_idf 中已经包含了每个词的idf值
    - 只需计算出每个词的tf值即可
    - tf = 文本出现次数/文档长度
      - 通过rdd的map操作 可以将每一行的数据拿到 
      - 通过这一行数据的 row.rawFeatures.values.sum() 可计算出一行有多少个关键词（文档长度）
      - 遍历这一行数据的 row.rawFeatures.indices 获取索引 这个索引与keywords_list_with_idf 的索引一一对应
      - 通过row.rawFeatures 传入索引获取单词出现次数 /文档长度 可计算tf值
      - 通过索引 到keywords_list_with_idf 中找到idf值
      - tf*idf 可以算出该行每个词语的tf-idf值

- tfidf统计结果

```python
from functools import partial
#tf 就是词库中的某个词在 当前文章 中出现的频率
def _tfidf(partition, kw_list):
    for row in partition:
        words_length = row.rawFeatures.values.sum()
        for index in  row.rawFeatures.indices:
            word, idf = kw_list[int(index)] 
            tf = row.rawFeatures[int(index)]/words_length   # 计算TF值
            tfidf = float(tf)*float(idf)    # 计算该词的TFIDF值
            yield row.sku_id, word, tfidf

# 使用partial为函数预定义要传入的参数
tfidf = partial(_tfidf, kw_list=keywords_list_with_idf)            
            
keyword_tfidf = cv_result.rdd.mapPartitions(tfidf)
keyword_tfidf = keyword_tfidf.toDF(["sku_id","keyword", "tfidf"])
keyword_tfidf.show()
```

- 显示结果

``` shell
+------+-------+-------------------+
|sku_id|keyword|              tfidf|
+------+-------+-------------------+
|   148|     智能|0.12920924724659527|
|   148|     数码|0.09416669674352422|
|   148|    读卡器|0.22691875231942266|
|   148|     正品| 0.1984394909665287|
|   148|   数码配件| 0.2300917543361638|
|   148|     购物| 0.2408169399138654|
|   148|     包邮| 0.2752684393351877|
|   148|    触摸屏|0.31885875801848024|
|   148|     高度| 0.3896366762502419|
|   148|    收银机|0.45724967270727696|
|   148|     终端|  0.515996300178136|
|   148|     业务|  0.537514526329006|
|   148|     森锐| 0.6107553455735467|
|   148|     身份| 0.6107553455735467|
|   148|   WPOS| 0.6672418695993603|
|   463|     颜色|0.08984603313709096|
|   463|     黑色|0.20849464181052813|
|   463|     电脑|0.17166530284651157|
|   463|     手机|0.20699498923484225|
|   463|     数码|0.05231483152418012|
+------+-------+-------------------+
only showing top 20 rows
```

- 分组查看tfidf结果

```python
keyword_tfidf.orderBy("tfidf", ascending=False).show()
```

显示结果

``` shell
+------+-------+------------------+
|sku_id|keyword|             tfidf|
+------+-------+------------------+
| 65304|     钥匙|16.866725445032152|
| 46934|    K22|15.970125524670596|
| 64669|     研钵|15.621139728147854|
| 23128|     木纹|13.619016619083395|
| 23350|     木纹|13.619016619083395|
| 46559|    XAD|13.354329091785555|
| 10349|     条线|13.255835415734486|
| 53158|     畸变|13.191231766589008|
|  4507|   W88D|13.136324307737405|
| 61486|     单排| 13.02289314127927|
| 65283|    抢答器|12.954087439685217|
| 51841|     纯铜| 12.68401586837053|
| 65127|    钥匙盘|12.663736560299217|
| 46847|    低碳钢| 12.38866944982758|
| 46643|X45X100|12.307564634298311|
| 65127|     铁环|12.149775344114998|
| 46349|    XAD|11.975349457307699|
| 66679|     卡位|11.935691239580011|
| 65127|     钥匙|11.869177165022624|
| 46848|     水道|11.650520419971981|
+------+-------+------------------+
only showing top 20 rows
```



### 4.9 将TFIDF结果存入hive中

- 把tfidf结果放到临时表中

```python
keyword_tfidf.registerTempTable("tempTable")
spark.sql("desc tempTable").show()
```

- 显示结果

``` shell
+--------+---------+-------+
|col_name|data_type|comment|
+--------+---------+-------+
|  sku_id|   bigint|   null|
| keyword|   string|   null|
|   tfidf|   double|   null|
+--------+---------+-------+
```

- 创建hive表

```python
sql = """CREATE TABLE IF NOT EXISTS sku_tag_tfidf(
sku_id INT,
tag STRING,
weights DOUBLE
)"""
spark.sql(sql)
```

- 将数据从临时表中插入到hive表中

```python
spark.sql("INSERT INTO sku_tag_tfidf SELECT * FROM tempTable")
```

- 查询结果

``` python
spark.sql("select * from sku_tag_tfidf").show()
```

``` shell
+------+----+-------------------+
|sku_id| tag|            weights|
+------+----+-------------------+
|   148|  智能|0.12920924724659527|
|   148|  数码|0.09416669674352422|
|   148| 读卡器|0.22691875231942266|
|   148|  正品| 0.1984394909665287|
|   148|数码配件| 0.2300917543361638|
|   148|  购物| 0.2408169399138654|
|   148|  包邮| 0.2752684393351877|
|   148| 触摸屏|0.31885875801848024|
|   148|  高度| 0.3896366762502419|
|   148| 收银机|0.45724967270727696|
|   148|  终端|  0.515996300178136|
|   148|  业务|  0.537514526329006|
|   148|  森锐| 0.6107553455735467|
|   148|  身份| 0.6107553455735467|
|   148|WPOS| 0.6672418695993603|
|   463|  颜色|0.08984603313709096|
|   463|  黑色|0.20849464181052813|
|   463|  电脑|0.17166530284651157|
|   463|  手机|0.20699498923484225|
|   463|  数码|0.05231483152418012|
+------+----+-------------------+
only showing top 20 rows
```

