### 2.2 Hive的内部表和外部表

<table>
  <tr>
    <th></th>
    <th>内部表(managed table)</th>
    <th>外部表(external table)</th>
  </tr>
  <tr>
    <td> 概念 </td>
    <td> 创建表时无external修饰 </td>
    <td> 创建表时被external修饰 </td>
  </tr>
  <tr>
    <td> 数据管理 </td>
    <td> 由Hive自身管理 </td>
    <td> 由HDFS管理 </td>
  </tr>
  <tr>
    <td> 数据保存位置 </td>
    <td> hive.metastore.warehouse.dir  （默认：/user/hive/warehouse） </td>
    <td> hdfs中任意位置 </td>
  </tr>
  <tr>
    <td> 删除时影响 </td>
    <td> 直接删除元数据（metadata）及存储数据 </td>
    <td> 仅会删除元数据，HDFS上的文件并不会被删除 </td>
  </tr>
  <tr>
    <td> 表结构修改时影响 </td>
    <td> 修改会将修改直接同步给元数据  </td>
    <td> 表结构和分区进行修改，则需要修复（MSCK REPAIR TABLE table_name;）</td>
  </tr>
</table>

- 案例

  - 创建一个外部表student2

    在对应目录A下有数据，创建外部表时指定对应的目录A，数据会被外部表自动加载，数据不被清除。

  ```sql
  CREATE EXTERNAL TABLE student2 (classNo string, stuNo string, score int) row format delimited fields terminated by ',' location '/tmp/student';
  ```

- 显示表信息

  ```sql
  desc formatted student;
  ```

- 向外部表传入数据

  ```shell
  直接向hdfs响应的文件夹上传数据即可
  hadoop fs -put ~/Desktop/student.txt /tmp/student
  ```

- 删除表查看结果

  ```sql
  drop table student2;
  ```

  注：内部表删除表后，元数据和数据均被删除，外部表删除表后，只有元数据被删除。

- 再次创建外部表 student2

- 不插入数据直接查询查看结果

  ```
  select * from student2;
  ```