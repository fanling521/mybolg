# Spark SQL

## 什么是Spark SQL

Spark SQL是Spark用来处理结构化数据的一个模块，它提供了2个编程抽象：`DataFrame`和`DataSet`，并且作为分布式SQL查询引擎的作用，它是将Spark SQL转换成RDD。

### Spark SQL的特点

易整合，统一的数据访问方式，兼容Hive，标准的数据连接。

### 什么是DataFrame

与RDD类似，DataFrame也是一个分布式数据容器。DataFrame是为数据提供了Schema的视图，DataFrame也是懒执行的，性能上比RDD要高。

### 什么是DataSet

DataSet是Dataframe API的一个扩展，是Spark最新的数据抽象，样例类被用来在Dataset中定义数据的结构信息，样例类中每个属性的名称直接映射到DataSet中的字段名称。Dataframe是Dataset的特列，DataFrame=Dataset[Row] ，所以可以通过as方法将Dataframe转换为Dataset。

## Spark SQL编程

### DataFrame

在Spark SQL中SparkSession是创建DataFrame和执行SQL的入口。

创建DataFrame有三种方式：通过Spark的数据源进行创建；从一个存在的RDD进行转换；还可以从Hive Table进行444，查询返回。

（1）从Spark数据源进行创建

```scala
scala> val df=spark.read.text("file:///home/fanl/data/emp.txt")
df: org.apache.spark.sql.DataFrame = [value: string]

scala> df.show
```

（2）创建一个临时表，查询

```scala
// 注意：临时表是Session范围内的，Session退出后，表就失效了
df.createOrReplaceTempView("emp")
//查询全表
val sqlDf=spark.sql("select * from emp")
sqlDf.show
```

（3）创建全局表

```scala
df.createGlobalTempView("emp")
spark.sql("select * from global_temp.emp").show
```

### DataSet

Dataset是具有强类型的数据集合，需要提供对应的类型信息。

```scala
// 创建样式类
case class Person(name:String,age:Int)
// 创建DataSet
val personDS=Seq(Person("Tom",12)).toDS
// 创建临时表等等操作
```

#### RDD转DataSet

```scala
val per=sc.textFile("file:///home/fanl/data/person.txt")
case class Person(name:String,age:Int)
val ds_per=per.map(x=>{val par=x.split(" ");Person(par(0),par(1).toInt)}).toDS
```

#### DataSet转RDD

```scala
//上例子
val ds_per=per.map(x=>{val par=x.split(" ");Person(par(0),par(1).toInt)}).toDS
ds_per: org.apache.spark.sql.Dataset[Person] = [name: string, age: int]
//转RDD
val rdd_per2=ds_per.rdd
rdd_per2: org.apache.spark.rdd.RDD[Person] = MapPartitionsRDD[5] at rdd at <console>:30
```

### DataFrame与DataSet

#### DataFrame转DataSet

DataFrame没有指定字段类型，需要在给出每一列的类型后，使用as方法，新增一个样式类，使用`df.as[T]`

注意：在使用一些特殊的操作时，一定要加上 `import spark.implicits._` 不然toDF、toDS无法使用

没有设置字段类型的，需要将

#### DataSet转DataFrame

DataSet转DataFrame很简单，调用`.toDF`

#### 区别和共性

### Scala编程

```xml
    <dependencies>
        <dependency>
            <groupId>org.apache.spark</groupId>
            <artifactId>spark-core_2.11</artifactId>
            <version>2.2.1</version>
        </dependency>
        <dependency>
            <groupId>org.apache.spark</groupId>
            <artifactId>spark-sql_2.11</artifactId>
            <version>2.2.1</version>
        </dependency>
    </dependencies>
```

代码参考，包括了命名，转换，查询条件等

```scala
def main(args: Array[String]): Unit = {
    //创建SparkConf()并设置App名称
    val spark = SparkSession.builder().master("spark://hadoop01:7077").appName("sparksqldemo").getOrCreate()
    //导入隐式转换
    //读取本地文件
    val df = spark.read
      .csv("hdfs://hadoop01:8020/user/fanl/person.txt")
    //创建临时表
    df.createOrReplaceTempView("person")
    //修改列名
    val schemas = Seq("name", "age")
    val dfRenamed = df.toDF(schemas: _*)
    //将age列转为Int
    // 转成DataSet
    spark.sql("select * from person").show
    import spark.implicits._
    val ds = dfRenamed.as[Person]
    //创建临时表
    ds.createOrReplaceTempView("person2")
    spark.sql("select * from person2").show
    spark.sql("select * from person2 where age > 15").show
    spark.stop()
  }

//样式类
case class Person(name: String, age: String)
```

### 自定义函数

在Shell窗口中可以通过spark.udf功能用户可以自定义函数。

```scala
spark.udf.register("addName", (x:String)=> "Name:"+x)
spark.sql("Select addName(name), age from people")
```

弱类型用户自定义聚合函数：通过继承`UserDefinedAggregateFunction`来实现用户自定义聚合函数。

强类型用户自定义聚合函数：通过继承`Aggregator`来实现强类型自定义聚合函数。

## Spark SQL数据源

### 通用加载/保存

Spark SQL的默认数据源为`Parquet`格式，修改配置项`spark.sql.sources.default`，可修改默认数据源格式。

可以通过SparkSession提供的`read.load`方法用于通用加载数据，使用`write`和`save`保存数据。

```scala
val peopleDF = spark.read.format("json").load("examples/src/main/resources/people.json")
peopleDF.write.format("parquet").save("hdfs://hadoop01:8020/namesAndAges.parquet")
```

可以直接读取SQL

```scala
val peopleDF = spark.read.format("json").load("examples/src/main/resources/people.json")
peopleDF.write.format("parquet").save("hdfs://hadoop01:8020/namesAndAges.parquet")
```

Spark SQL 能够自动推测 JSON数据集的结构，并将它加载为一个`Dataset[Row]`，可以通过`SparkSession.read.json()`去加载一个 一个JSON 文件。

这个JSON文件不是一个传统的JSON文件，每一行都得是一个JSON串。

```json
{"name":"Michael"}
{"name":"Andy", "age":30}
{"name":"Justin", "age":19}
```

读取数据库

```scala
val jdbcDF = spark.read.format("jdbc").option("url", "jdbc:mysql://centos7:3306/rdd").option("dbtable", " rddtable").option("user", "root").option("password", "hive").load()
// 保存
jdbcDF.write
.format("jdbc")
.option("url", "jdbc:mysql://centos7:3306/rdd")
.option("dbtable", "rddtable2")
.option("user", "root")
.option("password", "hive")
.save()
```

### 集成Hive

当Spark应用程序需要访问Hive的元数据库的时候，也就是说需要读取Hive中的表的时候，就要集成Hive。

**第一步**：只需要把hive-site.xml放在Spark的conf中即可。

```bash
[fanl@centos7 hive-1.1.0-cdh5.14.2]$ ln -s /opt/modules/cdh5.14.2/hive-1.1.0-cdh5.14.2/conf/hive-site.xml /opt/modules/cdh5.14.2/spark-2.2.1-bin-2.6.0-cdh5.14.2/conf/
```

**第二步**：把MySQL的驱动包放在Spark的jars里面

**第三步**：如果你hive-site.xml里面配置了metastore服务，则先开启metastore服务

```bash
[fanl@centos7 hive-1.1.0-cdh5.14.2]$ nohup bin/hiveserver2 &
[fanl@centos7 hive-1.1.0-cdh5.14.2]$ nohup bin/hive --service metastore &
# 启动spark-sql
[fanl@centos7 spark-2.2.1-bin-2.6.0-cdh5.14.2]$ bin/spark-sql
spark-sql (default)> show databases;
```

SparkSQL的`thiftserver`服务

```bash
[fanl@centos7 spark-2.2.1-bin-2.6.0-cdh5.14.2]$ bin/start-thriftserver.sh
```

Scala编程

```xml
<dependency>
  <groupId>org.apache.spark</groupId>
  <artifactId>spark-hive-thriftserver_2.11</artifactId>
  <version>2.2.1</version>
</dependency>
```

```scala
def main(args: Array[String]): Unit = {
    //1.添加驱动包和驱动类
    val driver = "org.apache.hive.jdbc.HiveDriver"
    Class.forName(driver)
    //2.创建连接对象
    val url = "jdbc:hive2://centos7:10000"
    val connect = DriverManager.getConnection(url,"xiaoming","123456")
    //3.sql语句
    connect.prepareStatement("use shop").execute()
    var pstmt =  connect.prepareStatement("select empno,ename,sal from emp")
    var rs = pstmt.executeQuery()
    while(rs.next()){
      println(s"empno = ${rs.getInt("empno")},ename = ${rs.getString("ename")},sal = ${
        rs.getDouble("sal")
      }")
    }
    rs.close()
    pstmt.close()
    println("============================================================")
    pstmt =  connect.prepareStatement("select empno,ename,sal from emp where sal > ?")
    pstmt.setDouble(1,1500.00)
    rs = pstmt.executeQuery()
    while(rs.next()){
      println(s"empno = ${rs.getInt("empno")},ename = ${rs.getString("ename")},sal = ${
        rs.getDouble("sal")
      }")
    }
    pstmt.close()
    rs.close()
    connect.close()
  }

```

