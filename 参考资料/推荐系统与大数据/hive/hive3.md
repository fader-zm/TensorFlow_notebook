## 二 Hive 基本操作

### 2.1 Hive HQL操作初体验

- 启动hive

  在已经配置了hive的情况下，在任意位置执行指令`hive`，即可进入hive的交互式环境中。

- 创建数据库

  ```sql
  CREATE DATABASE test;
  ```

- 显示所有数据库

  ```sql
  SHOW DATABASES;
  ```

- 使用数据库

  ```
  use test;
  ```

- 显示当前数据库

  ```
  select current_database();  # current: 当前
  ```

- 设置hive属性在命令行显示当前数据库

  ```sql
  set hive.cli.print.current.db=true;
  ```

- 创建表

  ```
  CREATE TABLE student(classNo string, stuNo string, score int) row format delimited fields terminated by ',';
  ```

  - row format delimited fields terminated by ','  指定了字段的分隔符为逗号，所以load数据的时候，load的文本也要为逗号，否则加载后为NULL。hive只支持单个字符的分隔符

- 查看表信息

  ```sql
  desc formatted student;  # format: 格式
  ```

- 将数据load到表中

  - 在本地文件系统创建一个如下的文本文件：/home/hadoop/data/student.txt

    ```
    C01,N0101,82
    C01,N0102,59
    C01,N0103,65
    C02,N0201,81
    C02,N0202,82
    C02,N0203,79
    C03,N0301,56
    C03,N0302,92
    C03,N0306,72
    ```

  - ```sql
    #local:本地文件系统
    #去掉local，指的是hdfs文件系统
    # overwrite: 将文件覆盖掉
    hive>load data local inpath 'file:///home/hadoop/data/student.txt'overwrite into table student;
    ```

  - 这个命令将student.txt文件复制到hive的warehouse目录中，这个目录由hive.metastore.warehouse.dir配置项设置，默认值为/user/hive/warehouse。Overwrite选项将导致Hive事先删除student目录下所有的文件, 并将文件内容映射到表中。
    Hive不会对student.txt做任何格式处理，因为Hive本身并不强调数据的存储格式。

- 查询表中的数据 跟SQL类似

  ```sql
  hive>select * from student;
  ```

- 分组查询group by和统计 count

  ```sql
  hive>select classNo,count(score) from student where score>=60 group by classNo;
  ```

  从执行结果可以看出 hive把查询的结果变成了MapReduce作业通过hadoop执行

```
C01	2
C02	3
C03	2
```

