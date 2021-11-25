[![License](http://img.shields.io/:license-Apache_v2-blue.svg)](LICENSE)
# spark-radiant

spark-radiant is Apache Spark Performance and Cost Optimizer Project.

### Providing Spark as a Service with your own Fork of Spark

One of the major challenges in providing Spark as a Service is to effectively maintain own code base, while also keeping up with the many and frequent changes in the Open Source Spark.
Apache Spark releases its new version in every six months. In order to further build upon this new version 
of Spark, one needs to merge changes to the Spark branch, work on the codebase and resolve conflicts. 
This entire process requires a lot of efforts, and is also Time consuming.

### Why do organizations prefer maintaining their own fork of Spark?

Below are the few scenarios for maintaining own fork
1) There are certain features that are only applicable to the use case of the organization.
2) The feature request is important for majority of the customers, and open source community does not accept feature requests.
3) Last but most important, the organization does not want to open source the change for the competitive reason.

### What if ?

1) we had the liberty to keep up with the pace of changes in Apache Spark Master.
2) we can release Spark the same day the new version of Apache Spark releases.
3) we have our own Spark Optimizer Project that can be integrated with the Runtime Spark.
4) we can maintain our own project and update as per the new changes in Spark.
5) what if we need not modify our project with any new release by Apache Spark

### Turning what if into reality !

Spark-Radiant is Apache Spark Performance and Cost Optimizer. The product, Spark-Radiant will help optimize performance
and cost considering catalyst optimizer rules, Enhanced Auto-Scaling in Spark, Collecting Important Metrics related to spark job,
BloomFilter Index in Spark etc. 

Maintaining Spark-Radiant is much easier than maintaining the Spark Fork. With minimal changes in the Spark Fork, and
considerable optimizations achieved using Spark-Radiant, we can easily reduce the time needed to release Spark as soon
as the new version of the same is released by Apache Spark.

## How to use Spark-Radiant

**Build** 

This project used the Maven Build.

git clone https://github.com/SaurabhChawla100/spark-radiant.git

cd spark-radiant

mvn clean install -DskipTests / mvn clean package -DskipTests 

**Add the Dependency to your project**

Build the project locally and add the dependency to your project

or

Use the current released version from the maven central.

For Sql Optimization(Ready for use)

```

<dependency>
    <groupId>io.github.saurabhchawla100</groupId>
    <artifactId>spark-radiant-sql</artifactId>
    <version>1.0.2</version>
</dependency>
```

For Core Optimization (WIP)
```

<dependency>
    <groupId>io.github.saurabhchawla100</groupId>
    <artifactId>spark-radiant-core</artifactId>
    <version>1.0.2</version>
</dependency>

```

### running Spark job

Publish the spark-radiant-sql-1.0.2.jar, spark-radiant-core-1.0.2.jar to maven central
```
./bin/spark-shell --packages "io.github.saurabhchawla100:spark-radiant-sql:1.0.2,io.github.saurabhchawla100:spark-radiant-core:1.0.2"

./bin/spark-submit 
 --packages "io.github.saurabhchawla100:spark-radiant-sql:1.0.2,io.github.saurabhchawla100:spark-radiant-core:1.0.2"
 --class com.test.spark.examples.SparkTestDF /spark/examples/target/scala-2.12/jars/spark-test_2.12-3.1.1.jar 
 ```

### Pre-requisite
spark-radiant have the below prerequisites

a) This is supported with spark-3.0.x and latter version of spark.   
b) Supported scala version 2.12.x.

### Use spark-radiant at runtime
This spark-radiant project has 2 modules, you can use those modules in your project

1) **spark-radiant-sql** - This contains the optimization related to Spark Sql Performance Improvement

   a) **Using Dynamic Filtering in Spark** - Please refer the docs for [Dynamic Filtering](docs/dynamicFilterInSpark.md).

   b) **Using Size based Join ReOrdering in Spark** - Please refer the docs for [SizeBasedJoinReOrdering](docs/SizeBasedJoinOrderingRerInSpark.md).

   c) **Add multiple value in the withColumn Api of Spark** - Till now you can add one by one columns
   using withColumn call with respect to spark dataframe. If we need to multiple column in dataframe there
   is need to call this withColumn again and again.
   For example
   `
   df.withColumn("newCol", lit("someval")).withColumn("newCol1", lit("someval1")).withColumn("newCol2", lit("someval2"))
   `

   This will add the projection in each call of withColumn, resulting in the exception (stackoverflow error).
   [SPARK-26224][SQL] issue of withColumn while using it multiple times

   We can add all the new col in one call in one single projection using the method useWithColumnsOfSpark

        import com.spark.radiant.sql.api.SparkRadiantSqlApi
        val sparkRadiantSqlApi = new SparkRadiantSqlApi()
        val withColMap = Map("newCol" -> lit("someval"), "newCol1" -> lit("someval1"), "newCol2" -> lit("someval2"))
        val df1 = sparkRadiantSqlApi.useWithColumnsOfSpark(withColMap, inputDF)

   d) **UnionReuseExchangeOptimizeRule** - This rule works for scenarios when union is
      present with aggregation having same grouping column. The union is between the same table/datasource.
      In this scenario instead of scan twice the table/datasource because of the child of Union,
      There will be one scan of table/datasource, and the other child of union will reuse this scan.This feature is enabled
      using --conf spark.sql.optimize.union.reuse.exchange.rule=true

      ```
         import com.spark.radiant.sql.api.SparkRadiantSqlApi
         // adding Extra optimizer rule
         val sparkRadiantSqlApi = new SparkRadiantSqlApi()
         sparkRadiantSqlApi.addOptimizerRule(spark)
      ```
     
    #### In PySpark
   
      ``` 
        In PySpark
   
              ./bin/pyspark --packages io.github.saurabhchawla100:spark-radiant-sql:1.0.2
      
               // Importing the extra Optimizations rule
               from sparkradiantsqlpy import SparkRadiantSqlApi
               SparkRadiantSqlApi(spark).addExtraOptimizerRule()
                    
              or
         
              // Importing the extra Optimizations rule
              spark._jvm.com.spark.radiant.sql.api.SparkRadiantSqlApi().addOptimizerRule(spark._jsparkSession) 
   
      ```

   e) **ExchangeOptimizeRule** - This optimizer rule works for scenarios where partial aggregate exchange is
      present and also the exchange which is introduced by SMJ and other join that add the shuffle exchange, So in total
      there are 2 exchange present in the executed plan and cost of creating both exchange are almost same. In that scenario
      we skip the exchange created by the partial aggregate and there will be only one exchange and partial and complete
      aggregation done on the same exchange, now instead of having 2 exchange there will be only one exchange. This can be
      enabled using --conf spark.sql.skip.partial.exchange.rule=true
      
      ```
         import com.spark.radiant.sql.api.SparkRadiantSqlApi
         // adding Extra optimizer rule
         val sparkRadiantSqlApi = new SparkRadiantSqlApi()
         sparkRadiantSqlApi.addOptimizerRule(spark)
      ```
   #### In PySpark
      
      ``` 
         In PySpark
   
              ./bin/pyspark --packages io.github.saurabhchawla100:spark-radiant-sql:1.0.2
      
               // Importing the extra Optimizations rule
               from sparkradiantsqlpy import SparkRadiantSqlApi
               SparkRadiantSqlApi(spark).addExtraOptimizerRule()
                    
              or
         
              // Importing the extra Optimizations rule
              spark._jvm.com.spark.radiant.sql.api.SparkRadiantSqlApi().addOptimizerRule(spark._jsparkSession) 

      ```
   
   f) **ExplodeOptimizeRule** - This optimizer rule works for scenarios where Explode is present with aggregation,
   So there will be exchange after partial aggregation and there are scenarios where cost of partial aggregate + exchange
   is high. In those scenarios it's better to have exchange first and then apply both partial aggregate and complete
   aggregate on the exchange. This can be enabled using --conf spark.sql.optimize.explode.rule=true

      ```
         import com.spark.radiant.sql.api.SparkRadiantSqlApi
         // adding Extra optimizer rule
         val sparkRadiantSqlApi = new SparkRadiantSqlApi()
         sparkRadiantSqlApi.addOptimizerRule(spark)
      ``` 
   #### In PySpark

      ``` 
         In PySpark
              ./bin/pyspark --packages io.github.saurabhchawla100:spark-radiant-sql:1.0.2
      
               // Importing the extra Optimizations rule
               from sparkradiantsqlpy import SparkRadiantSqlApi
               SparkRadiantSqlApi(spark).addExtraOptimizerRule()
                    
              or
         
              // Importing the extra Optimizations rule
              spark._jvm.com.spark.radiant.sql.api.SparkRadiantSqlApi().addOptimizerRule(spark._jsparkSession) 

      ```

   g)  **BloomFilter Index** - This is WIP

2) **spark-radiant-core** - This contains the optimization related to total cost optimization.
   
   a) **Metrics Collector** - SparkJobMetricsCollector is used for collecting the important metrics to Spark Driver Memory,
        Stage metrics, Task Metrics. This is enabled by using the configuration
        --conf spark.extraListeners=com.spark.radiant.core.SparkJobMetricsCollector and providing the jars in the class path 
        using --jars spark-radiant-core-1.0.3-SNAPSHOT.jar
      
   ```
   Spark-Radiant Metrics Collector
   Total Time taken by Application:: 895 sec
   
   *****Driver Metrics*****
   Time spend in the Driver: 307 sec
   Percentage of time spend in the Driver: 34. Try adding more parallelism to the Spark job for Optimal Performance
   
   *****Stage Info Metrics*****
   Stage Info Metrics Stage Id:0
   {
   "Stage Id": 0,
   "Number of Task": 10,
   "Total Executors ran to complete all Task": 2,
   "Stage Completion Time": 858 ms,
   "Average Task Completion Time": 139 ms
   "Stage Skew info": Skew task in not present in this stage
   }
   Stage Info Metrics Stage Id:1
   {
   "Stage Id": 1,
   "Number of Task": 10,
   "Total Executors ran to complete all Task": 2,
   "Stage Completion Time": 53 ms,
   "Average Task Completion Time": 9 ms
   "Stage Skew info": Skew task in not present in this stage
   }
   Stage Info Metrics Stage Id:2
   {
   "Stage Id": 2,
   "Number of Task": 100,
   "Total Executors ran to complete all Task": 4,
   "Stage Completion Time": 11206 ms,
   "Average Task Completion Time": 221 ms
   "Stage Skew info": List({
   "Task Id": 0,
   "Executor Id": 3,
   "Number of records read": 11887,
   "Number of shuffle read Record": 11887,
   "Number of records write": 0,
   "Number of shuffle write Record": 0,
   "Task Completion Time": 10656 ms
   }, {
   "Task Id": 4,
   "Executor Id": 1,
   "Number of records read": 11847,
   "Number of shuffle read Record": 11847,
   "Number of records write": 0,
   "Number of shuffle write Record": 0,
   "Task Completion Time": 10013 ms
   })
   }
    ```
   
   b) **Enhanced Autoscaling in Spark**- This is WIP

### Bug reports / New Feature Request

You can raise new feature request and Bug on this [issuelink](https://github.com/SaurabhChawla100/spark-radiant/issues).
