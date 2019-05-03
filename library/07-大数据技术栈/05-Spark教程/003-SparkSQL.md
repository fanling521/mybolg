# SparkSQL

## 什么是SparkSQL

Spark SQL是Spark用来处理结构化数据的一个模块，它提供了2个编程抽象：`DataFrame`和`DataSet`，并且作为分布式SQL查询引擎的作用，它是将Spark SQL转换成RDD。

### Spark SQL的特点

易整合，统一的数据访问方式，兼容Hive，标准的数据连接

### 什么是DataFrame

与RDD类似，DataFrame也是一个分布式数据容器。DataFrame是为数据提供了Schema的视图，DataFrame也是懒执行的，性能上比RDD要高。

### 什么是DataSet

DataSet是Dataframe API的一个扩展，是Spark最新的数据抽象，样例类被用来在Dataset中定义数据的结构信息，样例类中每个属性的名称直接映射到DataSet中的字段名称。Dataframe是Dataset的特列，DataFrame=Dataset[Row] ，所以可以通过as方法将Dataframe转换为Dataset。

## Spark SQL编程

### DataFrame

在Spark SQL中SparkSession是创建DataFrame和执行SQL的入口。

创建DataFrame有三种方式：通过Spark的数据源进行创建；从一个存在的RDD进行转换；还可以从Hive Table进行查询返回。

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

