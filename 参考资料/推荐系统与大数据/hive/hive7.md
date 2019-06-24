## 四 hive综合案例

- 内容推荐数据处理

  ![](img/hive3.png)

  - 需求
    - 根据用户行为以及文章标签筛选出用户最感兴趣(阅读最多)的标签

- 相关数据

  ```
  11,101,2018-12-01 06:01:10
  22,102,2018-12-01 07:28:12
  33,103,2018-12-01 07:50:14
  11,104,2018-12-01 09:08:12
  22,103,2018-12-01 13:37:12
  33,102,2018-12-02 07:09:12
  11,101,2018-12-02 18:42:12
  35,105,2018-12-03 09:21:12
  22,104,2018-12-03 16:42:12
  77,103,2018-12-03 18:31:12
  99,102,2018-12-04 00:04:12
  33,101,2018-12-04 19:10:12
  11,101,2018-12-05 09:07:12
  35,102,2018-12-05 11:00:12
  22,103,2018-12-05 12:11:12
  77,104,2018-12-05 18:02:02
  99,105,2018-12-05 20:09:11
  ```

  - 文章数据

  ```
  101,http://www.itcast.cn/1.html,kw8|kw1
  102,http://www.itcast.cn/2.html,kw6|kw3
  103,http://www.itcast.cn/3.html,kw7
  104,http://www.itcast.cn/4.html,kw5|kw1|kw4|kw9
  105,http://www.itcast.cn/5.html,
  ```

- 数据上传hdfs

  ```shell
  hadoop fs -mkdir /tmp/demo
  hadoop fs -mkdir /tmp/demo/action
  ```

- 创建外部表

  - 用户行为表

  ```sql
  drop table if exists actions;
  CREATE EXTERNAL TABLE actions(
      user_id STRING,
      article_id STRING,
      ts STRING
  )
  ROW FORMAT delimited fields terminated by ','
  LOCATION '/tmp/demo/action';
  ```

  - 文章表

  ```sql
  drop table if exists articles;
  CREATE EXTERNAL TABLE articles(
      article_id STRING,
      url STRING,
      kws array<STRING>
  )
  ROW FORMAT delimited fields terminated by ',' 
  COLLECTION ITEMS terminated BY '|' 
  LOCATION '/tmp/demo/article/';
  /*
  kws array<STRING>  数组的数据类型
  COLLECTION ITEMS terminated BY '|'  数组的元素之间用'|'分割
  */
  ```

  - 查看数据

  ```sql
  select * from actions;
  select * from articles;
  ```

  - 分组查询每个用户的浏览记录

    - collect_set/collect_list作用:
      - 将group by中的某列转为一个数组返回
      - collect_list**不去重**而collect_set**去重**
    - collect_set

    ```sql
    select user_id,collect_set(article_id) from actions group by user_id;
    ```

    ```shell
    11      ["101","104"]
    22      ["102","103","104"]
    33      ["103","102","101"]
    35      ["105","102"]
    77      ["103","104"]
    99      ["102","105"]
    ```

    - collect_list

    ```sql
    select user_id,collect_list(article_id) from actions group by user_id;
    ```

    ```shell
    11      ["101","104","101","101"]
    22      ["102","103","104","103"]
    33      ["103","102","101"]
    35      ["105","102"]
    77      ["103","104"]
    99      ["102","105"]
    
    ```

    ```
    - sort_array: 对数组排序
    
    ​```sql
    
    select user_id,sort_array(collect_list(article_id)) as contents from actions group by user_id;
    
    ​```
    
    ​``` shell
    11      ["101","104","101","101"]
    22      ["102","103","104","103"]
    33      ["103","102","101"]
    35      ["105","102"]
    77      ["103","104"]
    99      ["102","105"]
    ​```
    ```

- 查看每一篇文章的关键字 lateral view explode

  - 将一行数据拆分成多行数据，在此基础上可以对拆分的数据进行聚合

  ```sql
  #lateral view:建立侧视图
  #explode：hive内置函数，用于将数组内容分解开
  #t:侧视图的名称
  select article_id,kw from articles lateral view explode(kws) t as kw;#这种方式默认会剔除空值 '105 NULL'
  select article_id,kw from articles lateral view outer explode(kws) t as kw;
  ```

  ```shell
  101     kw8
  101     kw1
  102     kw6
  102     kw3
  103     kw7
  104     kw5
  104     kw1
  104     kw4
  104     kw9
  ```

  ```shell
  101     kw8
  101     kw1
  102     kw6
  102     kw3
  103     kw7
  104     kw5
  104     kw1
  104     kw4
  104     kw9
  105     NULL
  #含有outer
  ```

- 根据文章id找到用户查看文章的关键字

  - 原始数据

  ```shell
  101     http://www.itcast.cn/1.html     ["kw8","kw1"]
  102     http://www.itcast.cn/2.html     ["kw6","kw3"]
  103     http://www.itcast.cn/3.html     ["kw7"]
  104     http://www.itcast.cn/4.html     ["kw5","kw1","kw4","kw9"]
  105     http://www.itcast.cn/5.html     []
  ```

  ```sql
  select a.user_id, b.kw from actions as a left outer JOIN 
  (select article_id,kw from articles lateral view outer explode(kws) t as kw) 
  b on (a.article_id = b.article_id) order by a.user_id;
  ```

  ```shell
  11      kw1
  11      kw8
  11      kw5
  11      kw1
  11      kw4
  11      kw1
  11      kw9
  11      kw8
  11      kw1
  11      kw8
  22      kw1
  22      kw7
  22      kw9
  22      kw4
  22      kw5
  22      kw7
  22      kw3
  22      kw6
  33      kw8
  33      kw1
  33      kw3
  33      kw6
  33      kw7
  35      NULL
  35      kw6
  35      kw3
  77      kw9
  77      kw1
  77      kw7
  77      kw4
  77      kw5
  99      kw3
  99      kw6
  99      NULL
  ```

- 根据文章id找到用户查看文章的关键字并统计频率

  ```sql
  select a.user_id, b.kw,count(1) as weight from actions as a left outer JOIN 
  (select article_id,kw from articles lateral view outer explode(kws) t as kw) 
  b on (a.article_id = b.article_id) group by a.user_id,b.kw order by a.user_id,weight desc;
  ```

  ```shell
  11      kw1     4
  11      kw8     3
  11      kw5     1
  11      kw9     1
  11      kw4     1
  22      kw7     2
  22      kw9     1
  22      kw1     1
  22      kw3     1
  22      kw4     1
  22      kw5     1
  22      kw6     1
  33      kw3     1
  33      kw8     1
  33      kw7     1
  33      kw6     1
  33      kw1     1
  35      NULL    1
  35      kw3     1
  35      kw6     1
  77      kw1     1
  77      kw4     1
  77      kw5     1
  77      kw7     1
  77      kw9     1
  99      NULL    1
  99      kw3     1
  99      kw6     1
  ```

- 将用户查看的关键字和频率合并成 key:value形式

  ```sql
  select a.user_id, concat_ws(':',b.kw,cast (count(1) as string)) as kw_w from actions as a left outer JOIN 
  (select article_id,kw from articles lateral view outer explode(kws) t as kw) 
  b on (a.article_id = b.article_id) group by a.user_id,b.kw;
  ```

  ```shell
  11      kw1:4
  11      kw4:1
  11      kw5:1
  11      kw8:3
  11      kw9:1
  22      kw1:1
  22      kw3:1
  22      kw4:1
  22      kw5:1
  22      kw6:1
  22      kw7:2
  22      kw9:1
  33      kw1:1
  33      kw3:1
  33      kw6:1
  33      kw7:1
  33      kw8:1
  35      1
  35      kw3:1
  35      kw6:1
  77      kw1:1
  77      kw4:1
  77      kw5:1
  77      kw7:1
  77      kw9:1
  99      1
  99      kw3:1
  99      kw6:1
  ```

- 将用户查看的关键字和频率合并成 key:value形式并按用户聚合

  ```sql
  select cc.user_id,concat_ws(',',collect_set(cc.kw_w)) from (select a.user_id, concat_ws(':',b.kw,cast (count(1) as string)) as kw_w from actions as a left outer JOIN 
  (select article_id,kw from articles lateral view outer explode(kws) t as kw) 
  b on (a.article_id = b.article_id) group by a.user_id,b.kw) as cc group by cc.user_id;
  ```

  ```shell
  11      kw1:4,kw4:1,kw5:1,kw8:3,kw9:1
  22      kw1:1,kw3:1,kw4:1,kw5:1,kw6:1,kw7:2,kw9:1
  33      kw1:1,kw3:1,kw6:1,kw7:1,kw8:1
  35      1,kw3:1,kw6:1
  77      kw1:1,kw4:1,kw5:1,kw7:1,kw9:1
  99      1,kw3:1,kw6:1
  ```

- 将上面聚合结果转换成map

  ```sql
  select cc.user_id,str_to_map(concat_ws(',',collect_set(cc.kw_w))) as wm from
  (select a.user_id, concat_ws(':',b.kw,cast (count(1) as string)) as kw_w from actions as a left outer JOIN (select article_id,kw from articles lateral view outer explode(kws) t as kw) b on (a.article_id = b.article_id) group by a.user_id,b.kw
  ) as cc group by cc.user_id;
  ```

  ```shell
  11      {"kw1":"4","kw4":"1","kw5":"1","kw8":"3","kw9":"1"}
  22      {"kw1":"1","kw3":"1","kw4":"1","kw5":"1","kw6":"1","kw7":"2","kw9":"1"}
  33      {"kw1":"1","kw3":"1","kw6":"1","kw7":"1","kw8":"1"}
  35      {"1":null,"kw3":"1","kw6":"1"}
  77      {"kw1":"1","kw4":"1","kw5":"1","kw7":"1","kw9":"1"}
  99      {"1":null,"kw3":"1","kw6":"1"}
  ```

- 将用户的阅读偏好结果保存到表中

  ```sql
  create table user_kws as 
  select cc.user_id,str_to_map(concat_ws(',',collect_set(cc.kw_w))) as wm
  from(
  select a.user_id, concat_ws(':',b.kw,cast (count(1) as string)) as kw_w 
  from actions as a 
  left outer JOIN (select article_id,kw from articles
  lateral view outer explode(kws) t as kw) b
  on (a.article_id = b.article_id)
  group by a.user_id,b.kw
  ) as cc 
  group by cc.user_id;
  ```

- 从表中通过key查询map中的值

  ```sql
  select user_id, wm['kw1'] from user_kws;
  ```

  ```shell
  11      4
  22      1
  33      1
  35      NULL
  77      1
  99      NULL
  ```

- 从表中获取map中所有的key 和 所有的value

  ```sql
  select user_id,map_keys(wm),map_values(wm) from user_kws;
  ```

  ```shell
  11      ["kw1","kw4","kw5","kw8","kw9"] ["4","1","1","3","1"]
  22      ["kw1","kw3","kw4","kw5","kw6","kw7","kw9"]     ["1","1","1","1","1","2","1"]
  33      ["kw1","kw3","kw6","kw7","kw8"] ["1","1","1","1","1"]
  35      ["1","kw3","kw6"]       [null,"1","1"]
  77      ["kw1","kw4","kw5","kw7","kw9"] ["1","1","1","1","1"]
  99      ["1","kw3","kw6"]       [null,"1","1"]
  ```

- 用lateral view explode把map中的数据转换成多列

  ```sql
  select user_id,keyword,weight from user_kws lateral view explode(wm) t as keyword,weight;
  ```

  ```shell
  11      kw1     4
  11      kw4     1
  11      kw5     1
  11      kw8     3
  11      kw9     1
  22      kw1     1
  22      kw3     1
  22      kw4     1
  22      kw5     1
  22      kw6     1
  22      kw7     2
  22      kw9     1
  33      kw1     1
  33      kw3     1
  33      kw6     1
  33      kw7     1
  33      kw8     1
  35      1       NULL
  35      kw3     1
  35      kw6     1
  77      kw1     1
  77      kw4     1
  77      kw5     1
  77      kw7     1
  77      kw9     1
  99      1       NULL
  99      kw3     1
  99      kw6     1
  ```