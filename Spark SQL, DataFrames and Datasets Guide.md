#Spark SQL, DataFrames and Datasets Guide

注明：翻译过程中参考了[《Spark 官方文档》Spark SQL, DataFrames 以及 Datasets 编程指南](http://ifeve.com/spark-sql-dataframes/)。

官方文档链接是：[Spark SQL, DataFrames and Datasets Guide](http://spark.apache.org/docs/latest/sql-programming-guide.html)

在翻译之前先说下个人理解的RDD和。。的关系，这有助于理解本文。

##Overview

Spark SQL是spark中处理结构化数据的模块，不像底层的Spark RDD API，Spark SQL的接口提供了更多的关于数据和正在执行的任务的结构信息。在Spark内部，Spark SQL利用这些额外的信息来执行更多的优化。

现在有三种方式可以和Spark SQL交互:SQL语句、DataFrame API和Datasets API。当Spark计算结果时，用的执行引擎是一样的，独立于你使用了怎么样的api或者语言来描述计算过程。

这样的底层统一的方式意味着开发者在这些API中很方便的切换，并且这些API提供了最自然的方式来执行RDD的转换操作。

本文中的所有例子都使用Spakr发布版本的样例数据，并且可以在**spark-shell**，**pyspark shell**,或者**sparkR shell**中运行。

##SQL

Spark SQL的一种用法就是执行SQL查询，用最基本的SQL语法或者HiveQL。
Spark SQL也可以从已安装的Hive系统中读取数据。关于更详细的如何配置这个特性的说明请参考：[Hive Tablees](http://spark.apache.org/docs/latest/sql-programming-guide.html#hive-tables)这一节。如果用其它编程语言运行SQL，那么会以DataFrame的方式返回结果。你也可以使用[command line](http://spark.apache.org/docs/latest/sql-programming-guide.html#running-the-spark-sql-cli)和[JDBC/ODBC](http://spark.apache.org/docs/latest/sql-programming-guide.html#running-the-thrift-jdbcodbc-server)的方式使用Spark SQL接口。

## DataFrames

DataFrame就是用命名的列来组织的分布式数据集合，概念上等同于关系型数据库中的表，或者是R和python中的data frame，只不过DataFrame在底层做了更多的优化。DataFrame可以从不同的数据源([sources](http://spark.apache.org/docs/latest/sql-programming-guide.html#data-sources))构造数据,比如结构化的数据文件，Hive中的表，外部数据库，或者已有的RDD。

##Datasets

Dataset是Spark1.6中新加的实验性质的接口，就是为了利用RDD的各种便利(强类型，强大的使用lambda表达式的能力)和Spark SQL优化后的执行引擎的便利。Dataset可以从JVM对象构建，而后就可以使用各种函数式变换(transformations)，比如：map、flatMap、filter等等。

统一的Dataset API支持scala和java，但是还不支持python，但是python自身语言的动态特性，Datasets的许多便利处也能享受到，比如，你可以通过row.columnName来访问行的字段。对python的完整支持会在未来加进来。

##Getting Started

Spark SQL所有功能的入口点是都是[SQLContext](http://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.sql.SQLContext)类，或者它的子类。通过SparkContext就可以创建一个基本的SQLContext。

	val sc: SparkContext // An existing SparkContext.
	val sqlContext = new org.apache.spark.sql.SQLContext(sc)

	// this is used to implicitly convert an RDD to a DataFrame.
	import sqlContext.implicits._

除了创建SQLContext外，你也可以创建HiveContext，HiveContext是SQLContext的超集，除了SQLContext的功能外，HiveContext还提供了完整HiveQL解析语法，访问Hive UDFs还有从Hive tables读取数据的能力。为了使用HiveContex，你并不需要安装Hive，而且SparkSQL能用的数据源，HiveContext也能用。HiveContext是单独打包的就是为了避免在默认的Spark版本中包含所有的Hive的依赖。如果这些依赖对你来说都不是问题(不会造成依赖冲突)，那么建议你使用Spark1.3版本或者更早，因为后续的版本关注把SQLContext升级到和HiveContext同等地位。

spark.sql.dialect选项用来选择具体的解析查询语句的SQL变种，这个参数可以用SQLContext中的setConf方法来改变，也可以通过SQL语句的SET key=value命令来指定。该配置目前唯一的可选值就是”sql”，这个变种使用一个Spark SQL自带的简易SQL解析器。而对于HiveContext，spark.sql.dialect 默认值为”hiveql”，当然你也可以将其值设回”sql”。仅就目前而言，HiveSQL解析器支持更加完整的SQL语法，所以大部分情况下，推荐使用HiveContext。

##Creating DataFrames

Spark应用可以用SQLContext从[existing RDD](http://spark.apache.org/docs/latest/sql-programming-guide.html#interoperating-with-rdds)、Hive表或者[data sources](http://spark.apache.org/docs/latest/sql-programming-guide.html#data-sources)。

下面的例子就是从一个json文件来创建DataFrame。

	val sc: SparkContext // An existing SparkContext.
	val sqlContext = new org.apache.spark.sql.SQLContext(sc)

	val df = sqlContext.read.json("examples/src/main/resources/people.json")

	// Displays the content of the DataFrame to stdout
	df.show()

## DataFrame Operations

DataFrame提供了操作结构化数据的领域特定语言，比如scala、java、python、R。

这里我们给出用DataFrame处理结构化数据的基础例子。

    val sc: SparkContext // An existing SparkContext.
    val sqlContext = new org.apache.spark.sql.SQLContext(sc)
    
    // Create the DataFrame
    val df = sqlContext.read.json("examples/src/main/resources/people.json")
    
    // Show the content of the DataFrame
    df.show()
    // age  name
    // null Michael
    // 30   Andy
    // 19   Justin
    
    // Print the schema in a tree format
    df.printSchema()
    // root
    // |-- age: long (nullable = true)
    // |-- name: string (nullable = true)
    
    // Select only the "name" column
    df.select("name").show()
    // name
    // Michael
    // Andy
    // Justin
    
    // Select everybody, but increment the age by 1
    df.select(df("name"), df("age") + 1).show()
    // name(age + 1)
    // Michael null
    // Andy31
    // Justin  20
    
    // Select people older than 21
    df.filter(df("age") 21).show()
    // age name
    // 30  Andy
    
    // Count people by age
    df.groupBy("age").count().show()
    // age  count
    // null 1
    // 19   1
    // 30   1

DataFrame的完整操作列表请参考：[API Documentation](http://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.sql.DataFrame)。

除了简单的字段引用和表达式支持外，DataFrame还提供了丰富的工具库，比如：字符串操作，日期处理，常用的数学表达式等。完整列表见：[DataFrame Function Reference](http://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.sql.functions$)。

##Running SQL Queries Programmatically

SQLContext.sql能够使Spark应用用编程的方式运行SQL查询并且以DataFrame的形式返回结果。

	val sqlContext = ... // An existing SQLContext
	val df = sqlContext.sql("SELECT * FROM table")

##Creating Datasets

Datasets和RDD类似，不过它不用java序列化或者Kryo，而是用专门的编码器([Encoder](http://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.sql.Encoder))来序列化跨网络的处理的传输。如果这个编码器和标准序列化都能把对象转字节，那么编码器就可以根据代码动态生成，并使用一种特殊数据格式，这种格式下的对象不需要反序列化回来，就能允许Spark进行操作，如过滤、排序、哈希等。

    // Encoders for most common types are automatically provided by importing sqlContext.implicits._
    val ds = Seq(1, 2, 3).toDS()
    ds.map(_ + 1).collect() // Returns: Array(2, 3, 4)
    
    // Encoders are also created for case classes.
    case class Person(name: String, age: Long)
    val ds = Seq(Person("Andy", 32)).toDS()
    
    // DataFrames can be converted to a Dataset by providing a class. Mapping will be done by name.
    val path = "examples/src/main/resources/people.json"
    val people = sqlContext.read.json(path).as[Person]

##Interoperating with RDDs

Spark SQL提供了两种方法将RDD转为DataFrame。

第一种方式：使用反射机制，推导包含指定类型对象RDD的schema。这种基于反射机制的方法使代码更简洁，而且如果你事先知道数据schema，推荐使用这种方式；

第二种方式：编程方式构建一个schema，然后应用到指定RDD上。这种方式更啰嗦，但如果你事先不知道数据有哪些字段，或者数据schema是运行时读取进来的，那么你很可能需要用这种方式。

##Inferring the Schema Using Reflection

Spark SQL的Scala接口支持自动将包含case class对象的RDD转为DataFrame。对应的case class定义了表的schema。case class的参数名通过反射，映射为表的字段名。case class还可以嵌套一些复杂类型，如Seq和Array。RDD隐式转换成DataFrame后，可以进一步注册成表。随后，你就可以对表中数据使用SQL语句查询了。

    // sc is an existing SparkContext.
    val sqlContext = new org.apache.spark.sql.SQLContext(sc)
    // this is used to implicitly convert an RDD to a DataFrame.
    import sqlContext.implicits._
    
    // Define the schema using a case class.
    // Note: Case classes in Scala 2.10 can support only up to 22 fields. To work around this limit,
    // you can use custom classes that implement the Product interface.
    case class Person(name: String, age: Int)
    
    // Create an RDD of Person objects and register it as a table.
    val people = sc.textFile("examples/src/main/resources/people.txt").map(_.split(",")).map(p => Person(p(0), p(1).trim.toInt)).toDF()
    people.registerTempTable("people")
    
    // SQL statements can be run by using the sql methods provided by sqlContext.
    val teenagers = sqlContext.sql("SELECT name, age FROM people WHERE age >= 13 AND age <= 19")
    
    // The results of SQL queries are DataFrames and support all the normal RDD operations.
    // The columns of a row in the result can be accessed by field index:
    teenagers.map(t => "Name: " + t(0)).collect().foreach(println)
    
    // or by field name:
    teenagers.map(t => "Name: " + t.getAs[String]("name")).collect().foreach(println)
    
    // row.getValuesMap[T] retrieves multiple columns at once into a Map[String, T]
    teenagers.map(_.getValuesMap[Any](List("name", "age"))).collect().foreach(println)
    // Map("name" -> "Justin", "age" -> 19)

##Programmatically Specifying the Schema

如果不能事先通过case class定义schema（例如，记录的字段结构是保存在一个字符串，或者其他文本数据集中，需要先解析，又或者字段对不同用户有所不同），那么你可能需要按以下三个步骤，以编程方式的创建一个DataFrame：

1. 从已有的RDD创建一个包含Row对象的RDD
2. 用StructType创建一个schema，和步骤1中创建的RDD的结构相匹配
3. 把得到的schema应用于包含Row对象的RDD，调用这个方法来实现这一步：SQLContext.createDataFrame

例如：

    // sc is an existing SparkContext.
    val sqlContext = new org.apache.spark.sql.SQLContext(sc)
    
    // Create an RDD
    val people = sc.textFile("examples/src/main/resources/people.txt")
    
    // The schema is encoded in a string
    val schemaString = "name age"
    
    // Import Row.
    import org.apache.spark.sql.Row;
    
    // Import Spark SQL data types
    import org.apache.spark.sql.types.{StructType,StructField,StringType};
    
    // Generate the schema based on the string of schema
    val schema =
      StructType(
    schemaString.split(" ").map(fieldName => StructField(fieldName, StringType, true)))
    
    // Convert records of the RDD (people) to Rows.
    val rowRDD = people.map(_.split(",")).map(p => Row(p(0), p(1).trim))
    
    // Apply the schema to the RDD.
    val peopleDataFrame = sqlContext.createDataFrame(rowRDD, schema)
    
    // Register the DataFrames as a table.
    peopleDataFrame.registerTempTable("people")
    
    // SQL statements can be run by using the sql methods provided by sqlContext.
    val results = sqlContext.sql("SELECT name FROM people")
    
    // The results of SQL queries are DataFrames and support all the normal RDD operations.
    // The columns of a row in the result can be accessed by field index or by field name.
    results.map(t => "Name: " + t(0)).collect().foreach(println)

##Data Sources

Spark SQL支持基于DataFrame操作一系列不同的数据源。DataFrame既可以当成一个普通RDD来操作，也可以将其注册成一个临时表来查询。把DataFrame注册为table之后，你就可以基于这个table执行SQL语句了。本节将描述加载和保存数据的一些通用方法，包含了不同的Spark数据源，然后深入介绍一下内建数据源可用选项。

##Generic Load/Save Functions

在最简单的情况下，所有操作都会以默认类型数据源来加载数据（默认是Parquet，除非修改了spark.sql.sources.default 配置）。

    val df = sqlContext.read.load("examples/src/main/resources/users.parquet")
    df.select("name", "favorite_color").write.save("namesAndFavColors.parquet")

##Manually Specifying Options

你也可以手动指定数据源，并设置一些额外的选项参数。数据源可由其全名指定（如，org.apache.spark.sql.parquet），而对于内建支持的数据源，可以使用简写名（json, parquet, jdbc）。任意类型数据源创建的DataFrame都可以用下面这种语法转成其他类型数据格式。

    val df = sqlContext.read.format("json").load("examples/src/main/resources/people.json")
    df.select("name", "age").write.format("parquet").save("namesAndAges.parquet")

##Run SQL on files directly

Spark SQL还支持直接对文件使用SQL查询，不需要用read方法把文件加载进来。

    val df = sqlContext.sql("SELECT * FROM parquet.`examples/src/main/resources/users.parquet`")

##Save Modes