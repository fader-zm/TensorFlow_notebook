### 2.3 分区表

- 什么是分区表

  - 随着表的不断增大，对于新纪录的增加，查找，删除等(DML)的维护也更加困难。对于数据库中的超大型表，可以通过把它的数据分成若干个小表，从而简化数据库的管理活动，对于每一个简化后的小表，我们称为一个单个的分区。
  - hive中分区表实际就是对应hdfs文件系统上独立的文件夹，该文件夹内的文件是该分区所有数据文件。
  - 分区可以理解为分类，通过分类把不同类型的数据放到不同的目录下。
  - 分类的标准就是分区字段，可以一个，也可以多个。
  - 分区表的意义在于优化查询。查询时尽量利用分区字段。如果不使用分区字段，就会全部扫描。

- 创建分区表

  ```sql
  create table emp (name string,salary bigint) partitioned by (dt string) row format delimited fields terminated by ',' lines terminated by '\n' stored as textfile;  # partitioned: 分区 terminated: 终止
  ```

- 添加分区

  ```
  alter table emp add if not exists partition(dt='2018-12-01');
  ```

- 查看表的分区

  ```sql
  show partitions emp;
  ```

- 加载数据到分区

  employee.txt

  ```
  tom,123
  jerry,234
  bob,345
  mike,456
  ```

  ```
   
  ```

- 如果重复加载同名文件，不会报错，会自动创建一个*_copy_1.txt

- 外部分区表即使有分区的目录结构, 也必须要通过hql添加分区, 才能看到相应的数据

  ```shell
  hadoop fs -mkdir /user/hive/warehouse/test.db/emp/dt=2018-12-04
  hadoop fs -copyFromLocal /home/hadoop/data/employee.txt /user/hive/warehouse/test.db/emp/dt=2018-12-04/employee.txt
  ```

  - 此时查看表中数据发现数据并没有变化, 需要通过hql添加分区

    ```
    alter table emp add if not exists partition(dt='2018-12-04');
    ```

  - 此时再次查看才能看到新加入的数据

- 总结

  - 利用分区表方式减少查询时需要扫描的数据量
    - 分区字段不是表中的列, 数据文件中没有对应的列
    - 分区仅仅是一个目录名
    - 查看数据时, hive会自动添加分区列
    - 支持多级分区, 多级子目录

### 2.4 动态分区

- hive提供了一个动态分区功能，其可以基于查询参数的位置去推断分区的名称，从而建立分区。

- 在写入数据时自动创建分区(包括目录结构)

- 创建表

  ```
  create table emp2 (name string,salary bigint) partitioned by (dt string) row format delimited fields terminated by ',' lines terminated by '\n' stored as textfile;
  ```

- 导入数据

  ```sql
  insert into table emp2 partition(dt) select name,salary,dt from emp;
  ```

- 使用动态分区需要设置参数

  ```shell
  set hive.exec.dynamic.partition.mode=nonstrict;
  ```