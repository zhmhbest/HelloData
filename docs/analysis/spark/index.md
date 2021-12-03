<link rel="stylesheet" href="https://zhmhbest.gitee.io/hellomathematics/style/index.css">
<script src="https://zhmhbest.gitee.io/hellomathematics/style/index.js"></script>

# [Spark](../index.html)

<!--
https://archive.apache.org/dist/spark/
https://mirrors.tuna.tsinghua.edu.cn/apache/spark/

http://spark.apachecn.org/#/
-->

## 环境搭建

- [Jdk-8u202](https://www.oracle.com/java/technologies/javase/javase8-archive-downloads.html)
- [Scala-2.12.2](https://www.scala-lang.org/download/2.12.2.html)
- [Hadoop-2.7.7](https://archive.apache.org/dist/hadoop/common/hadoop-2.7.7/)

```bash
# 依赖的环境变量
JAVA_HOME=
HADOOP_HOME=
```

## Hello

```txt
.
├── pom.xml
└── src
    └── main
        ├── resources
        │   └── csv
        │       └── Order.csv
        └── scala
            └── org
                └── example
                    └── HelloSpark.scala
```

`pom.xml`

```xml
<!-- ... -->
<dependencies>
    <dependency>
        <groupId>org.apache.spark</groupId>
        <artifactId>spark-core_2.12</artifactId>
        <version>3.0.0</version>
    </dependency>
    <dependency>
        <groupId>org.apache.spark</groupId>
        <artifactId>spark-sql_2.12</artifactId>
        <version>3.0.0</version>
    </dependency>
</dependencies>

<build>
    <sourceDirectory>${project.basedir}/src/main/scala</sourceDirectory>
    <!--<testSourceDirectory>${project.basedir}/src/test/scala</testSourceDirectory>-->
    <plugins>
        <plugin>
            <groupId>net.alchim31.maven</groupId>
            <artifactId>scala-maven-plugin</artifactId>
            <version>3.2.2</version>
            <executions>
                <execution>
                    <id>compile-scala</id>
                    <phase>compile</phase>
                    <goals>
                        <goal>add-source</goal>
                        <goal>compile</goal>
                    </goals>
                </execution>
                <execution>
                    <id>test-compile-scala</id>
                    <phase>test-compile</phase>
                    <goals>
                        <goal>add-source</goal>
                        <goal>testCompile</goal>
                    </goals>
                </execution>
            </executions>
        </plugin>
    </plugins>
</build>
<!-- ... -->
```

`HelloSpark.scala`

```scala
package org.example
import org.apache.spark.sql.{DataFrameReader, SparkSession}

object HelloSpark {
    def main(args: Array[String]): Unit = {
        val spark = SparkSession.builder
            .config("spark.driver.host", "localhost")
            .appName("AppName")
            .master("local")
            .getOrCreate
        spark.sparkContext.setLogLevel("ERROR")
        val dfReader: DataFrameReader = spark.read.format("csv")
        val csvFile = this.getClass.getClassLoader.getResource("./csv/Order.csv").getFile
        val df = dfReader.csv(csvFile)
        df.foreach(r => {
            println(r)
        })
    }
}
```
