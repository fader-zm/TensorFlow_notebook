### 5 Hive 安装部署

- Hive 安装前需要安装好 JDK 和 Hadoop。配置好环境变量。

- 下载Hive的安装包 http://archive.cloudera.com/cdh5/cdh/5/ 并解压

  ```shell
   tar -zxvf hive-1.1.0-cdh5.7.0.tar.gz  -C ~/bigdata/
  ```

- 进入到 解压后的hive目录 找到 conf目录, 修改配置文件

  ```shell
  cp hive-env.sh.template hive-env.sh
  vi hive-env.sh
  ```

  在hive-env.sh中指定hadoop的路径![1559298631982](hive2.assets/1559298631982.png)

  ![1559298045140](hive2.assets/1559298045140.png)

  ```shell
  HADOOP_HOME=/home/hadoop/app/hadoop-2.6.0-cdh5.7.0
  ```

- 配置环境变量

  - ```shell
    vi ~/.bash_profile
    ```

  - ```shell
    export HIVE_HOME=/home/hadoop/app/hive-1.1.0-cdh5.7.0
    export PATH=$HIVE_HOME/bin:$PATH
    ```

  - ```shell
    source ~/.bash_profile
    ```

- 根据元数据存储的介质不同，分为下面两个版本，其中 derby 属于内嵌模式。实际生产环境中则使用 mysql 来进行元数据的存储。

  - 内置 derby 版： 
    bin/hive 启动即可使用
    缺点：不同路径启动 hive，每一个 hive 拥有一套自己的元数据，无法共享

  - mysql 版： 

    - 安装mysql

      - 1，下载mysql的安装源

        ```
        wget http://repo.mysql.com/mysql-community-release-el7-5.noarch.rpm
        ```

      - 2，安装用来配置mysql的yum源的rpm包

        ```
        sudo rpm -ivh mysql-community-release-el7-5.noarch.rpm
        ```

      - 3，安装mysql

      - ```
        sudo yum install mysql-server
        ```

      - 4，开启mysql服务

        `service mysqld start`

      - 5，修改用户目录拥有者

        ```
        sudo chown -R hadoop:hadoop /var/lib/mysql
        ```

      - 6，查看和修改密码

        ```
        mysql -uroot -p
        默认密码没有，直接敲回车
        进入mysql后：
        mysql > use mysql;
        mysql > update user set password=password('root') where user='root';
        mysql > exit;
        修改完成后，重启mysql服务
        service mysqld restart
        ```

    - 创建数据库：

      ```
      create database hive charset=latin1;
      #由于数据库版本为5.6，较低，导致上述的数据集必须为latin1，否则会造成一些问题。如果数据库版本是5.7，那么就可以随意设置charset=utf8。
      ```

    - 上传 mysql驱动到 hive安装目录的lib目录下

      mysql-connector-java-5.*.jar

    - vi conf/hive-site.xml 配置 hive的元数据信息存储位置(存储到Mysql数据库中)

      ```xml-dtd
      <?xml version="1.0" encoding="UTF-8" standalone="no"?>
      <?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
      <configuration>
      <!-- 插入以下代码 -->
          <property>
              <name>javax.jdo.option.ConnectionUserName</name>
              <value>root</value><!-- 指定mysql用户名 -->
          </property>
          <property>
              <name>javax.jdo.option.ConnectionPassword</name>
              <value>root</value><!-- 指定mysql密码 -->
          </property>
         <property>
              <name>javax.jdo.option.ConnectionURL</name>mysql
              <value>jdbc:mysql://127.0.0.1:3306/hive</value>
          </property><!-- 指定mysql数据库地址 -->
          <property>
              <name>javax.jdo.option.ConnectionDriverName</name>
              <value>com.mysql.jdbc.Driver</value><!-- 指定mysql驱动 -->
          </property>
      </configuration>
      ```