# SparklyrIntroduction
spraklyr的入门介绍以及使用
Sparklyr
1	官方介绍
1.1	概述
Sparklyr是一个R语言通往spark的接口
1.	他为连接到sprak提供了一个完成的dplyr的后端
2.	可以过滤并且聚合spark的数据集，然后将这些数据集导入R的环境进行分析和可视化
3.	可以使用来自R的spark分布式机器学习库
4.	创建调用完整的Spark API并且扩展，并且为Spark包提供接口
2	安装
2.1	系统架构
2.1.1	环境要求
框架名称	要求的版本	作用
JDK	1.8 or later versions	其他框架运行的必须的环境
R	3.0以上	R语言运行的环境
RStudio Server	1.2.以上	连接spark集群可视化界面IDE
Spark	2.3	做分布式运算
Hadoop	2.6以上	大数据环境

2.1.2	R环境搭建
首先要在集群中的机器上搭建R的环境以及RStudio Server
1.	安装前所需的各种包
yum install gcc gcc-c++
yum install gcc-gfortran
yum install readline-devel
yum install libXt-deve
yum install fonts-chinese tcl tcl-devel tclx tk tk-devel
yum install mesa-libGLU mesa-libGLU-devel

2.	安装R
下载R-3.2.3.tar.gz
创建R的安装目录 /usr/local/R
解压R-3.2.3.tar.gz 然后配置
./configure --enable-R-shlib=yes --with-libpng-x=no --with-tcltk --prefix=/usr/local/R
配置环境：
R_HOME=/usr/local/R
PATH=$PATH:R_HOME/bin

如果不是用自行编译方法还可以使用yum源安装
yum install R
安装完成后在termimal中输入R启动成功
3.	RStudio Server 安装
https://www.rstudio.com/products/rstudio/download-server/(官网示例)
 
在页面访问RStudio ServerIDE地址  http://ServerIp:8787/
3	连接spark集群
3.1	加载连接spark所需的一些包
加载sparklyr包
>install.packages("sparklyr")  #从CRAN下载sparklyr包
>library(sparklyr)           #加载sparklyr包

加载与自己集群spark版本相同的spark软件包到R环境当中
>spark_install(version = "2.3.2")
注：此方法可能连接不上apache的仓库，如果下载失败则需要手动离线下载tgz软件包到本地。
从apache官网手动下载相关的spark_hadoop依赖包：
http://spark.apache.org/downloads.html
然后上传该软件包到机器上 spark_install_tar(tarfile = "文件路径")到此成功配置sparklyr的环境





3.2	连接local spark
示例：
conf$`sparklyr.cores.local` <- 4
conf$`sparklyr.shell.driver-memory` <- "1G"
conf$spark.memory.fraction <- 0.9
sc <- spark_connect(master = "local",  config = conf)

注：单机版不需要装RStudio Server 直接可以使用桌面版本的RStudio即可。
3.3	连接spark集群版本方法（yarn-cluster）
首先需要当前R语言环境在集群的节点中，然后根据节点配置相关的环境变量
>Sys.setenv(JAVA_HOME="/usr/local/jdk1.8.0_181")
>Sys.setenv(SPARK_HOME = '/usr/hdp/3.1.0.0-78/spark2')
>Sys.setenv(YARN_CONF_DIR = '/usr/hdp/3.1.0.0-78/hadoop-yarn/conf')
可以通过设置某些Spark属性的值来自定义与Spark的连接。在sparklyr，可以使用config函数中的参数设置Spark属性spark_connect()。
默认情况下，spark_connect()使用spark_config()的默认配置。但是可以自定义，如下面的示例代码所示。由于无限数量的可能组合，spark_config()仅包含基本配置，因此很可能需要其他设置才能正确连接到群集。这其中conf的设置相当于spark—submit里面的参数设置
R连接spark相当于提交一个spark任务。

示例：
conf <- spark_config()
conf$spark.executor.memory <- "300M"
conf$spark.executor.cores <- 2
sc <- spark_connect(master = "yarn-client", 
                       config = conf)
连接成功之后就可以在sparkWebUI上看到此任务了 任务名称默认为sparklyr。
4	使用dplyr对数据进行操纵
4.1	概述
dplyr包是一个用于处理结构化数据的R包，一般情况下使用dplyr包作为操作SparkDataFrames的接口。它可以：
1.选择，过滤和汇总数据
2.使用窗口函数（例如用于采样）
3.在DataFrame上执行连接
4.将Spark中的数据收集到R中
4.2	操作数据
4.2.1	读取数据
您可以使用以下函数将数据读入Spark DataFrames：
spark_read_csv： 读取CSV文件并提供与dplyr兼容的数据源
spark_read_json  读取JSON文件并提供与dplyr兼容的数据源
spark_read_parquet 读取镶木地板文件并提供与dplyr兼容的数据源
相关参数可以参考官网apl文档  （https://spark.rstudio.com/reference/）

无论数据的格式如何，Spark都支持从各种不同的数据源中读取数据。这些包括存储在HDFS（hdfs://...），S3（s3n://...）或Spark工作节点可用的本地文件（file://..）上的数据
这些函数中的每一个都返回对Spark DataFrame的引用	


对于R已经存在的数据集可以使用copy_to来复制到spark集群当中并且使用spark集群来进行运算
如:
library(nycflights13)
flights <- copy_to(sc, flights, "flights")
对flights队形进行操作就是在spark集群中进行的操作





4.2.2		dplyr Verbs操作

Verbs是用于操作数据的dplyr 命令，当连接到Spark DataFrame时，dplyr会将这些命令转化为SparkSql的语句由spark执行。以下是五个动词及其相对应的SQL命令
select 〜 SELECT
filter 〜 WHERE
arrange 〜 ORDER
summarise 〜 aggregators: sum, min, sd, etc.
mutate 〜 operators: +, *, log, etc.
如：
select(flights, year:day, arr_delay, dep_delay)
相当于sql：
selecy year,month,day, dep_delay from flights 
4.2.3	使用sql操作
还可以直接对Spark集群中的表执行SQL查询。spark_connection对象实现了一个用于Spark的DBI接口，因此可以使用dbGetQuery执行SQL，并将结果作为R数据帧返回:，实际上作为不太理解R语言的开发者 用sql会降低使用门槛，推荐使用sql来对数据集进行操作，具体的操作如下
library(DBI)
dbGetQuery(sc,”select * from flights”)




4.2.4	收集到R
收集到R 相当于spark就是把数据手机到driver的道理一样， 你可以使用collect()函数将得出的数据收集到R终端 ，然后对这些数据进行可视化的展示
如：
c4 <- flights %>%
  filter(month == 5, day == 17, carrier %in% c('UA', 'WN', 'AA', 'DL')) %>%
  select(carrier, dep_delay, air_time, distance) %>%
  arrange(carrier) %>%
  mutate(air_time_hours = air_time / 60) 
c4 %>%
  group_by(carrier) %>%
  summarize(count = n(), mean_dep_delay = mean(dep_delay))
carrierhours <- collect(c4)

4.2.5	写数据
将分析结果或在Spark集群上生成的表保存到持久存储中通常很有用。在许多情况下，最好的选择是使用spark_write_parquet 函数将表写入 Parquet文件
如：
spark_write_parquet(tbl, "hdfs://hdfs.company.org:9000/hdfs-path/data")

这会将tbl R变量引用的Spark DataFrame写入给定的HDFS路径。您可以使用 spark_read_parquet 函数将同一个表读回到后续的Spark会话中：
tbl <- spark_read_parquet(sc, "data", "hdfs://hdfs.company.org:9000/hdfs-path/data")






5	.Spark机器学习库（MLlib）
sparklyr为Spark的分布式机器学习库提供绑定。特别是，sparklyr允许您访问spark.ml包提供的机器学习例程。与sparklyr的dplyr界面一起，您可以轻松地在Spark上创建和调整机器学习工作流程，完全在R中编排。
sparklyr提供了三个函数系列，可以与Spark机器学习一起使用：
用于分析数据的机器学习算法（ml_*）
用于处理各个特征的特征变换器（ft_*）
用于操作Spark DataFrames的函数（sdf_*）

具有sparklyr的分析工作流程可能由以下阶段组成：
1.通过sparklyr dplyr接口执行SQL查询，
2.使用函数sdf_*和ft_*函数系列来生成新列或对数据集进行分区，
3.从ml_*函数族中选择合适的机器学习算法来建模数据，
4.检查模型拟合的质量，并使用它来使用新数据进行预测。
5.收集结果以便在R中进行可视化和进一步分析

可以通过一ml_*组函数从sparklyr访问Spark的机器学习库 如：
 

例如聚类分析例子：
kmeans_model <- iris_tbl %>%
  select(Petal_Width, Petal_Length) %>%
  ml_kmeans(centers = 3)
6	基于分布式R的计算
6.1	概述
Sparklyr支持在spark集群中通过分布式大规模的运行任意的R代码通过spark_apply()方法，当需要使用在spark中没有的包而R中有的包时尤其有用，
spark_apply()可以将R函数应用于spark对象，通常来说是DataFrame对象。对于spark来说对象是分区的，因此对象可以是跨机器分布。可以使用spark_apply默认分区，也可以使用group_by参数自行定义分区。但重要的一点是 您使用的R函数必须返回另外一个DataFrame对象。Spark在每个分区上进行计算然后输出单个的Spark DataFrame。

6.2	简单例子

sdf_len(sc, 5, repartition = 1) %>%
  spark_apply(function(e) I(e))
## # Source:   table<sparklyr_tmp_378c2e4fb50> [?? x 1]
## # Database: spark_connection
##      id
##   <dbl>
## 1     1
## 2     2
## 3     3
## 4     4
## 5     5
您的R功能应设计为在R 数据帧（DataFrame）上运行。传递的R函数spark_apply需要一个DataFrame，并返回一个可以转换为DataFrame的对象

6.3	分区
Spark将按散列或范围对数据进行分区，以便可以在群集中分布。在下面的示例中，我们创建了两个分区并计算每个分区中的行数。然后我们打印每个分区中的第一条记录。
trees_tbl <- sdf_copy_to(sc, trees, repartition = 2)
trees_tbl %>%
  spark_apply(function(e) nrow(e), names = "n")
## # Source:   table<sparklyr_tmp_378c15c45eb1> [?? x 1]
## # Database: spark_connection
##       n
##   <int>
## 1    16
## 2    15

通常情况下spark_apply()，从输入Spark数据框中派生列名。使用names参数重命名或添加新列。
trees_tbl %>%
  spark_apply(
    function(e) data.frame(2.54 * e$Girth, e),
names = c("Girth(cm)", colnames(trees)))

## # Source:   table<sparklyr_tmp_378c14e015b5> [?? x 4]
## # Database: spark_connection
##    `Girth(cm)` Girth Height Volume
##          <dbl> <dbl>  <dbl>  <dbl>
##  1      21.082   8.3     70   10.3
##  2      22.352   8.8     63   10.2
##  3      27.178  10.7     81   18.8
##  4      27.940  11.0     66   15.6
##  5      28.194  11.1     80   22.6
##  6      28.702  11.3     79   24.2
##  7      28.956  11.4     76   21.4
##  8      30.480  12.0     75   19.1
##  9      32.766  12.9     85   33.8
## 10      34.798  13.7     71   25.7
## # ... with more rows
6.4	分组
在某些情况下，您可能希望将R函数应用于数据中的特定组。例如，假设您要针对特定子组计算回归模型。要解决此问题，您可以指定group_by参数。此示例计算iris物种中的行数，然后拟合每个物种的简单线性模型。
iris_tbl <- sdf_copy_to(sc, iris)
iris_tbl %>%
  spark_apply(nrow, group_by = "Species")

## # Source:   table<sparklyr_tmp_378c1b8155f3> [?? x 2]
## # Database: spark_connection
##      Species Sepal_Length
##        <chr>        <int>
## 1 versicolor           50
## 2  virginica           50
## 3     setosa           50

iris_tbl %>%
  spark_apply(
    function(e) summary(lm(Petal_Length ~ Petal_Width, e))$r.squared,
    names = "r.squared",
group_by = "Species")
## # Source:   table<sparklyr_tmp_378c30e6155> [?? x 2]
## # Database: spark_connection
##      Species r.squared
##        <chr>     <dbl>
## 1 versicolor 0.6188467
## 2  virginica 0.1037537
## 3     setosa 0.1099785

6.5	使用R中的packages
使用spark_apply() 你可以使用R中的packages来调用spark进行计算，例如你可以使用broom包中的函数对数据进行计算：
spark_apply(
  iris_tbl,
  function(e) broom::tidy(lm(Petal_Length ~ Petal_Width, e)),
  names = c("term", "estimate", "std.error", "statistic", "p.value"),
  group_by = "Species")
## # Source:   table<sparklyr_tmp_378c5502500b> [?? x 6]
## # Database: spark_connection
##      Species        term  estimate std.error statistic      p.value
##        <chr>       <chr>     <dbl>     <dbl>     <dbl>        <dbl>
## 1 versicolor (Intercept) 1.7812754 0.2838234  6.276000 9.484134e-08
## 2 versicolor Petal_Width 1.8693247 0.2117495  8.827999 1.271916e-11
## 3  virginica (Intercept) 4.2406526 0.5612870  7.555230 1.041600e-09
## 4  virginica Petal_Width 0.6472593 0.2745804  2.357267 2.253577e-02
## 5     setosa (Intercept) 1.3275634 0.0599594 22.141037 7.676120e-27
## 6     setosa Petal_Width 0.5464903 0.2243924  2.435422 1.863892e-02
 
