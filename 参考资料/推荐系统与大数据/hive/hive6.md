## 三 Hive 函数

### 3.1 内置运算符

在 Hive 有四种类型的运算符：

- 关系运算符

- 算术运算符

- 逻辑运算符

- 复杂运算

  (内容较多，见《Hive 官方文档》》)

### 3.2 内置函数

https://cwiki.apache.org/confluence/display/Hive/LanguageManual+UDF

- 简单函数: 日期函数 字符串函数 类型转换 
- 统计函数: sum avg distinct
- 集合函数
- 分析函数
- show functions;  显示所有函数
- desc function 函数名;
- desc function extended 函数名;  # extended: 扩展

### 3.3 Hive 自定义函数和 Transform

- UDF

  - 当 Hive 提供的内置函数无法满足你的业务处理需要时，此时就可以考虑使用用户自定义函数（UDF：user-defined function）。

  - **TRANSFORM**,and **UDF** and **UDAF**

    it is possible to plug in your own custom mappers and reducers

     A UDF is basically only a transformation done by a mapper meaning that each row should be mapped to exactly one row. A UDAF on the other hand allows us to transform a group of rows into one or more rows, meaning that we can reduce the number of input rows to a single output row by some custom aggregation.

    **UDF**：就是做一个mapper，对每一条输入数据，映射为一条输出数据。

    **UDAF**:就是一个reducer，把一组输入数据映射为一条(或多条)输出数据。

    一个脚本至于是做mapper还是做reducer，又或者是做udf还是做udaf，取决于我们把它放在什么样的hive操作符中。放在select中的基本就是udf，放在distribute by和cluster by中的就是reducer。

    We can control if the script is run in a mapper or reducer step by the way we formulate our HiveQL query.

    The statements DISTRIBUTE BY and CLUSTER BY allow us to indicate that we want to actually perform an aggregation.

    User-Defined Functions (UDFs) for transformations and even aggregations which are therefore called User-Defined Aggregation Functions (UDAFs)

- UDF示例(运行java已经编写好的UDF)

  - 在hdfs中创建 /user/hive/lib目录

    ```shell
    hadoop fs -mkdir /user/hive/lib
    ```

  - 把 hive目录下 lib/hive-contrib-hive-contrib-1.1.0-cdh5.7.0.jar 放到hdfs中

    ```shell
    hadoop fs -put hive-contrib-1.1.0-cdh5.7.0.jar /user/hive/lib/
    ```

  - 把集群中jar包的位置添加到hive中

    ```shell
    hive> add jar hdfs:///user/hive/lib/hive-contrib-1.1.0-cdh5.7.0.jar;
    ```

  - 在hive中创建**临时**UDF(mapper)

    ```sql
    hive> CREATE TEMPORARY FUNCTION row_sequence as 'org.apache.hadoop.hive.contrib.udf.UDFRowSequence'；
    ```

  - 在之前的案例中使用**临时**自定义函数(函数功能: 添加自增长的行号)

    ```sql
    select row_sequence(),* from emp;
    ```

  - 创建**非临时**自定义函数

    ```
    CREATE FUNCTION test.row_sequence as 'org.apache.hadoop.hive.contrib.udf.UDFRowSequence' using jar 'hdfs:///user/hive/lib/hive-contrib-1.1.0-cdh5.7.0.jar';
    ```

- Python UDF

  - 准备案例环境

    - 创建表

      ```sql
      CREATE table u(fname STRING,lname STRING);
      ```

    - 向表中插入数据

      ```sql
      insert into table u values('George','washington');
      insert into table u values('George','bush');
      insert into table u values('Bill','clinton');
      insert into table u values('Bill','gates');
      ```

  - 编写map风格脚本

    ```python
    import sys
    for line in sys.stdin:
        line = line.strip()
        fname , lname = line.split('\t')
        l_name = lname.upper()
        print('\t'.join([fname, str(l_name)]))
    ```

  - 通过hdfs向hive中ADD file

    - 加载文件到hdfs

      ```shell
      hadoop fs -put udf.py /user/hive/lib/
      ```

    - hive从hdfs中加载python脚本

      ```shell
      hive>ADD FILE hdfs:///user/hive/lib/udf.py;
      ```

  - Transform

    ```sql
    SELECT TRANSFORM(fname, lname) USING 'udf.py' AS (fname, l_name) FROM u;
    
    SELECT TRANSFORM(fname, lname) USING 'python3 udf.py' AS (fname, l_name) FROM u;
    ```

- Python UDAF