
# HIVE编程指南

## 变量和属性
* hivevar:用户自定义变量,可读/可写
* hiveconf:hive相关配置属性,可读/可写
* system:java定义的配置属性,可读/可写
* env:shell环境定义的环境变量,只可读
* Hive变量内部是以Java字符串的方式存储的,用户可以在查询中引用变量.hive会先使用变量值替换掉查询的变量引用,然后才会将查询语句提交给查询处理器.
* 在CLI中,可以使用SET命令显示或修改变量值.

```sql
--定义month用户变量
set hivevar:month=12;
--查询变量
set hivevar:month;

--使用
select...from...where month = ${hivevar:month}

--定义多个参数
set hivevar:ids = (1,2,3,4)

--使用
select...from...where id in ${hivevar:ids}

--执行命令时传参

hive -hivevar month=12 -e 'select...from...where month=${hivevar:month}'

```

## 在hive中使用shell/hadoop命令
* shell命令前加上!并且以;结尾即可

```sql
hive> ! echo 'hello world';
hive> ! pwd;

```

* hadoop命令前加dfs并且以;结尾

```sql
hive> dfs -ls /

```

## 显示字段名称

* set hive.cli.print.header=true;

## 基本数据类型
* TINEINT : 1byte有符号整数
* SMALLINT : 2byte有符号整数
* INT : 4byte有符号整数
* BIGINT : 8byte有符号整数
* BOOLEAN : 布尔类型,true或false
* FLOAT : 单精度浮点数,4byte
* DOUBLE : 双精度浮点数,8byte
* STRING : 字符序列(单双引号包裹)
* TIMESTAMP : 整数,浮点数或字符串
* ARRAY : 数组
* MAP : 键值对
* STRUCT : 结构体,与对象类似,都可以通过点符号访问元素内容.
	* 例如name STRUCT{first String,last String},访问name.first

> float和double的区别
> 
> float表示单精度浮点数在机内占4个字节，用32位二进制描述。 float型定义的数据末尾必须有"f"或"F",以示区分。
double表示双精度浮点数在机内占8个字节，用64位二进制描述。
> 
> double 和 float 的区别是double精度高，有效数字16位，float精度7位。从左边第一个非0数字开始计数.
> 但double消耗内存是float的两倍，double的运算速度比float慢得多
> 
> 因为小数在计算机中存储的多数为近似值，虽然符点数的表示方法（IEEE754标准）可以使得四字节或八字节表示很大的数据，但是，由于有的小数不能完全转换成纯粹相等的二进制数，所以，计算机只能保存最接近的其值的二进制小数。
> 
> [浮点数表示](https://blog.csdn.net/dreamer2020/article/details/24158303)
> 


* 创建复杂类型的示例:

```sql

CREATE TABLE IF NOT EXISTS employees(
	name STRING,
	salary FLOAT,
	subordinates ARRAY<STRING> COMMENT '下属员工名单',
	deductions MAP<STRING, FLOAT> COMMENT '扣款明细',
	address STRUCT<street:STRING, city:STRING, state:String>
);

```
## 分隔符
* \n, 用于分割记录,每行都是一条记录,换行符可以分割记录.
* ^A(\001), 用于分割字段.用户还可以使用'\t'(制表键)作为字段分隔符.
* ^B(\002), 用于分割集合中的元素.例如,数组,字典中的元素间隔
* ^C(\003), 用于分割字段中键和值

* 在定义分隔符前都要加上**ROW FORMAT DELIMITED**

```sql

CREATE TABLE IF NOT EXISTS employees(
	name STRING COMMENT '姓名',
	salary FLOAT COMMENT '工资',
	subordinates ARRAY<STRING> COMMENT '下属名单',
	deducitons MAP<STRING,FLOAT> COMMENT '扣款明细',
	address STRUCT<street:STRING, city:STRING, state:String> COMMENT '住址'
)
--定义分割
ROW FORMAT DELIMITED
LINES TERMINATED BY '\n'
FIELDS TERMINATED BY '\001'
COLLECTION ITEMS TERMINATED BY '\002'
MAP KEYS TERMINATED BY '\003'
STORED BY textfile;

```

## 读时模式
* hive不会在数据加载时进行验证,而是在查询时进行验证,这就是读时模式.
* 那么如果模式和文件内容并不匹配会怎么样?用NULL值进行补充.

## 数据定义

* DDL（Data Definition Language）：数据定义语言，用来定义数据库对象：库、表、列等；
* DML（Data Manipulation Language）：数据操作语言，用来定义数据库记录（数据）；
* DCL（Data Control Language）：数据控制语言，用来定义访问权限和安全级别；
* DQL（Data Query Language）：数据查询语言，用来查询记录（数据）。

### 数据库DATEBASE
* 创建

```sql
CTEATE DATEBASE IF NOT EXISTS financials 
COMMENT ''
WITH DBPROPERTIES('creator' = 'zyh', 'date' = '20190101');

```

* 查看: SHOW DATEBASES/SHOW DATEBASES LIKE 'h.*'
* 查看描述: DESCRIBE DATEBASE EXTENDED financials;
* 使用: USE default;
* 删除: DROP DATEBASE IF EXISTS financials;

> 数据库删除前,数据库中的表一定要删除完毕才可以执行这条语句.要是想都自动完成要使用CASCADE关键字.
> 
> DROP DATEBASE IF EXISTS financials CASCADE;
> 

* 配置显示当前数据库:set hive.cli.print.current.db=true;
* 修改数据库信息: ALTER DATEBASE financials SET DBPROPERTIES('edited-by' = 'zyh')

### 表
* 创建表

```sql
CREATE TABLE IF NOT EXISTS db.table(
	id INT COMMENT '员工ID',
	name STRING COMMENT '员工姓名',
	salay FLOAT COMMENT '工资',
	subordinates ARRAY<STRING> COMMENT '子员工名单',
	deducitions MAP<STRING,FLOAT> COMMENT '扣款明细',
	address STRUCT<street:STRING, city:STRING, state:STRING> COMMENT '员工地址'
)
ROW FORMAT DELIMITED
LINES TERMINATED BY '\n'
FIELDS TERMINATED BY '\001'
COLLECTION ITEMS TERMINATED BY '\002'
MAP KEYS TERMINATED BY '\003'
COMMENT '员工信息表'
TBLPROPERTIES('creator'='zyh','create_time'='20190101')
STORED BY ORC
LOCATION '/path1/path2/path3';

```

> hive 会自动添加两个表属性TBLPROPERTIES:一个是last_modified_by,其保存着最后修改这个表的用户名,另一个是last_modified_time,其保存这最后一个修改的时间.
> 
> 如果没有定义TBLPROPERTIES,那么hive就不自动添加这两个属性到TBLPROPERTIES

* 复制表模式:CREATE TABLE IF NOT EXISTS db.table1 LIKE db.table2;
* 列举数据库中所有的表: SHOW TABLES/SHOW TABLES 'empl.*';
* 查看表属性: DESCIBLE EXTENDED/FORMATTED db.table;
* 删除表:DROP TABLE IF EXISTS table

### 修改表
* 表重命名: ALTER TABLE old_name RENAME TO new_name;
* 分区修改
	* 添加分区: ALTER TABLE t ADD PARTITION(year=2012) 
	* 删除分区: ALTER TABLE t DROP PARTITION(year=2012)
* 修改列信息
	* 添加列: ALTER TABLE t ADD COLUMNS(id INT, name STRING);
	* 替换列: ALTER TABLE t REPLACE COLUMNS(new_id INT);
	* 修改列: ALTER TABLE t CHANGE COLUMN old_col new_col_name new col_type

> 注意
> 
> 在使用REPLACE修改列的时候是拿新定义的列信息对原始列进行修改,本例中是删除掉id,name后添加new_id.
> 
> 在修改列的时候 先写上原有列名,然后输入新列名字和类型. 关键字COLUMN可选.也可以添加COMMENT ''

* 修改表属性: ALTER TABLE t SET PROPERTIES('note' = 'new');
* 修改存储属性: ALTER TABLE t PARTITION(year=2012) SET FILEFORMAT SEQUENCEFILE
* 修改分桶: ALTER TABLE t CLUSTERED BY(id) INTO 46 BUCKETS SORT BY(id)


### 外部表EXTERNAL TABLE
* 创建外部表

```sql
CREATE EXTERNAL TABLE ex_t(
	...
)
ROW FORMAT DELIMITED
...
LOCATION '/path';

```

> 外部表后面的LOCATION子句用于告诉Hive数据位于哪个路径下

* 外部表是外部的,Hive并非认为完全拥有这份数据.删除表并不会删除掉这份数据,只不过描述表的元数据信息会被删除掉.

### 分区表
* 创建分区表

```sql

CREATE TABLE IF NOT EXISTS db.table(
	id INT,
	...
)
COMMENT '分区表'
PARTITIONED BY(country STRING);

```

* 分区字段一旦创建好,表现得就和普通的字段一样.
* 对数据进行分区,最重要的原因是为了更快的查询.
* 查看分区: SHOW PARTITIONS table;
* 在严格模式下,如果表有分区,是不能执行一个不包含分区的查询的.

> 设置严格模式
>
>set hive.mapred.mode=strict;

* [动态分区](https://www.cnblogs.com/sunpengblog/p/10396442.html)

* 添加分区: ALTER TABLE table ADD PARTITION(year=2012);
* 外部表也可以添加分区,使用方法一样.

### 表存储格式
* hive自带三种存储格式:textfile,sequencefile,rcffile.
* sequencefile和rcffile都是使用二进制编码和压缩来优化磁盘空间使用和I/O宽带新能的.

* hive使用一个inputformat对象将输入流分割成记录,然后使用一个outputformat对象来将记录格式化为输出流,再使用一个SerDe在读数据时将记录解析成列,在写数据时将列编码成记录.
 
> 记录编码是通过一个inputformat对象来控制的.
>
>例如textfile是hive使用了一个名为org.apache.hadoop.mapred.TextInputFormat的java类.
>
>为了保持完整性,hive还使用一个叫做outputformat的对象来将查询的输出写入到文件中或者控制台.
>
> textfile的outputformat是HiveIgnoreKeyTextOutputFormat

* 用户可以指定第三方的输入输出格式以及SerDe

```sql
CREATE TABLE kst
PARTITIONED BT(ds STRTING)
--指定了使用的SerDe
ROW FORMAT SERDE 'com.linkedin.haivvreo.AvroSerDe'
--指定用于输入和输出格式的java类
STORED AS
INPUTFORMAT 'com.linkedin.haivvreo.AvroContainerInputFormat'
OUTPUTFORMAT 'com.linkedin.haivvreo.AvroContainerOutputFormat'

``` 

### 分桶表
* 同分区表一样将字段再次进行拆分到不同文件中.
* 与分区表不同的是,分区表使用的新定义的字段作为分区字段,而分桶表则是用表已有字段进行分桶排序的.
* CLUSTERED BY (col) 对字段进行分桶, 
* SORT BY(col) 分桶时进行排序,可选.
* INTO n BUCKETS分成多少桶

```sql

CREATE TABLE c_t(
	id INT,
	name STRING
	age INT
)
CLUSTERED BY (id,name) INTO 96 BUCKETS
SORT BY (age)

```

## 数据操作(DML)

### LOAD装载数据
* hive中没有行级别的数据插入/更新/删除操作,往表中转载数据的唯一途径是使用大量的数据装载操作
* 装载数据 LOAD DATE [LOCAL] INPATH '' INTO TABLE 

```sql
LOAD DATE LOACL INPATH ''
OVERWRITE INTO TABLE t
PARTITION BY(year=2012)

```
> 通常情况下指定的路径应该是一个目录,而不是单个独立的文件.Hive会将所有文件都拷贝到这个目录中.
> 
> 如果文件是本地文件则需要用到LOCAL关键字,hive会从本地复制一份数据到存储文件中.如果文件是HDFS则不需要使用LOCAL关键字,hive会找到文件并将其地址换到hive的存储文件中,也就说不复制数据,直接移动数据.
> 

* 装载操作不会验证用户装载的数据和表的模式是否匹配.

### INSERT插入数据

```sql
--向单个表插入
INSERT [INTO/OVERWRITE] TABLE t PARTITION(year=2012)
SELECT...

--向多个表插入
FROM t
INSERT INTO TABLE t1 PARTITION(year=2012) SELECT... WHERE
INSERT OVERWRITE TABLE t2 SELECT... WHERE

```

* 静态分区:分区值是确定的

```sql
INSERT INTO TABLE t PARTITION(year=2012)
SELECT...FROM...WHERE...

```

* 动态分区:分区值是不确定的

```sql
INSERT INTO TABLE t partition(year)
SELECT...,year FROM...WHERE...

```

> 上面例子,year是决定分区的字段,在指定partition的时候给入字段并不用填值.分区值由select的最后一个值决定这条记录被分到哪个分区.在分区字段前的所有字段应与要插入的表的模式匹配.

* 动态分区和静态分区混合使用:静态分区必须在动态分区前

```sql
INSERT INTO TABLE t partition(year=2012,month)
--处理t中的匹配字段外只有动态分区字段,没有静态分区字段
SELECT...,month FROM...WHERE...

```

* hive默认是不开启动态分区的,在使用动态分区前需要配置hive的参数.
	* set hive.exec.dynamic.partition=true;--启动动态分区
	* set hive.exec.dynamic.partition.mode=nonstrict;--允许所有分区都是动态的
	* set hive.exec.max.dynamic.partitions=1000;--允许创建的最大分区数

### CTAS

```sql

CREATE TALBE T AS
SELECT...FROM...WHERE
```

* 这个功能不能用于外部表,一般用于从一个大的宽表选取部分需要的数据集.

### 导出数据
* 用户可以使用INSERT...DIRECTORY来导出数据

```sql

INSERT OVERWRITE LOCAL DIRECTORY '/path'
SELECT...FROM...WHERE

```


## DQL

### 行转列: explode(col) + lateral view

```sql
select t1.col1, subview.col2
from table t1
lateral view explode(col) subview as col2
where ...

```

> 相当于原始表t1和explode(col)生成的表进行笛卡尔积,然后筛选出对应的t1:col1,col记录输出
> 
> 一般explode与split连用,split将字符串切割成为数组,explode则将数组拆成行.

### 列转行: collect_set/collect_list

```sql
select a,collect_set(b)
from table1
where b <= 0
group by a;

```

* 将某一列值收集成一个数组.collect_set对数据进行去重,而collect_list则不去重.
* 一般collecti_set/list与array_sort/concat_ws连用.
	* array_sort是对数组进行排序
	* concat_ws则是将数组进行拼接成字符串

#### 思考:如何将两列值转化成字典

1. 将两列合并:str1 = concat_ws('\003',col1,col2)
2. 将合并后的值收集成一个数组:array = collect_list(str)
3. 将合并后的数组连接成字符串: str2 =concat_ws('\002',array)
4. 将合并后的字符串分割成字典:str_to_map(str2,'\002','\003')

> 注意
> 
> concat_ws()的使用,当后边参数不是array时,用于每行记录各个列连接.当后边参数是单个数组的时候,则用于将整个数组连接成一个字符串.
> 
> str_to_map()函数则是将规定好的字符串根据指定的分割符进行分割转化字典.
> 

### hive本地查询模式

* 对于WHERE语句中过滤条件只是分区字段这种情况,且没有聚合.无需MR,只是对数据记录进行过滤.
* 文件属性配置:set hive.exec.mode.local.auto=true;

### 关于浮点数的比较
* 浮点数比较的一个常见陷阱出现在不同类型间作比较的时候(也就是float和double比较).
* 当用户写一个浮点数时,hive会将该值保存为double类型,即便不需要那么多有效位数也同样如此.例如:0.2,hive会自动将0.2保存为double而不是float类型.
* 在浮点数表示法中,0.2存储为float类型时为:0.2000001,double类型时为0.200000000001,略大于最近似的精确值.而不是0.2000000和0.200000000000
* 如果用float类型的数字比较double类型时,hive中会发生隐式转化,将低精度自动转化为高精度.这样不会丢精度,如果高精度转低精度会丢失精度.
* 所以0.20000001的float类型会自动转化成为0.200000100000的double类型.转化后double类型是大于hive中自动保存的0.2double值的 0.200000100000 > 0.200000000001.

* 解决方式
	1. 如果要使用浮点数进行比较的时候要定义相同类型,避免隐式转化.
		* 例如上例中:0.2hive会自动存储为doule,改成显示定义为float.cast(0.2 as float)
	2. 任何比较都避免使用浮点数.将数字都乘以倍数转化为整数.
		* 例如:金钱相关,以分为单位,而不是元.

### LIKE和RLIKE
* LIKE为模式匹配:%为任意个任意字符,.为单个任意字符
* RLIKE为正则匹配
	* .表示和任意的字符匹配
	* *表示重复左边的字符
	* x|y表示和x或者y匹配
	* 例如:RLIKE '.\*(Chicago|Ontario).\*'表示为中间包含Chicago或Ontario的字符串.

### with as
* [with as用法1](https://blog.csdn.net/Abysscarry/article/details/81322669)
* [with as用法2](https://blog.csdn.net/high2011/article/details/85335954)

### 连接
* hive不支持不等值连接,只支持等值连接.
* 对于多表连接,大多数情况下,hive会对没对JOIN连接对象启动一个MR任务.从左到右,首先启动一个完成第一个表和第二表的连接操作,然后再启动一个MR任务完成上个输出和第三个表的连接操作.
* 如果每个ON子句都使用到了相同键作为一个JOIN连接键,hive会通过一个优化可以在同一MR任务中连接所有表,而不是将连接过程拆分成多个任务.HIVE会同时假定最后一张表是最大的那个表,在对每行记录进行连接操作时,它会尝试将其他表缓存起来,然后扫描最后一个表进行计算.因此用户需要将表按照大小从左到右进行排序.用户并非总是要将最大的表放置在查询语句的最后边,hive中还提供了标记机制显示的告知查询优化器哪张表是大表.

```sql
SELECT /*+STREAMTABLE(s)*/ 
	   s.id,s.name,d.id
FROM stock s 
JOIN dividends d ON s.id = d.id
WHERE s.symbol = 'AAPL';
```

* 注意,在写连接前,最好将每个表中符合条件的记录先筛选出来再进行JOIN操作.因为hive中执行顺序是先JOIN后再次进行筛选.所以减少JOIN量是决定速度的关键因素.

#### 半连接 semi join
* hive中不支持自查询,但是可以使用left semi join半连接来代替

```sql
SELECT s.id,s.name
FROM table1 s
LEFT SEMI JOIN table2 t
ON s.id = t.id

```
> 半连接,将s表中符合on条件的记录输出,这里并不会输出t表中的记录.注意跟left join进行区分.
> 
> left semi join通常比join更加高效,因为对于左边中的记录,一旦在右边表中匹配到,会立刻停止扫描.而join则会继续匹配,继续寻找下一条记录.
> 

#### map-side join
* 如果所有表中只有一张表是小表,那么可以在最大的表通过mapper的时候将小表完全放到内存中.hive可以在map端执行连接过程.hive可以在内存中的小表进行逐一匹配,从而省略掉常规连接操作所需要的reduce过程.
* 在hive0.7版本前可以使用标记的方式/\*+MAPJOIN(table)\*/来标记map-join的小表.
* 在v0.7之后废弃掉这种方式,但是也可以使用.如果用户不加这种标记则需要启动设置hive.auto.conver.JOIN=true,这样hive才会在必要的时候启动这个优化.默认是false.此外还需要配置一下能够使用map-join表的大小:hive.mapjoin.smalltable.filesize=25000000;
* 分桶表可以使用map-join的优化,但是条件是on中的键是分桶的键,而且一张表的分桶个数是另一张表的若干倍.同样需要设置参数进行开启hive.optimize.bucketmapJOIN=true

### 排序
1. order by:全局排序
2. sort by:只对每个reducer中的数据进行排序,一般与distrubute by连用.

* distrubute by控制map的输出在reducer中是如何划分的.默认情况下,MR计算框架会依据map输入的键计算相应的hash值,然后按照得到的哈希值将键值对均匀的分发到多个reducer中去.如果我们希望具有相同的某个值的数据被分到同一个reducer中去处理,可以使用distribute by来保证具有相同目标值的记录会被分发到同一个reducer中去,然后同sort by来对同一个reducer中的数据进行排序.

```sql
SELECT s.ymd, s.symbol, s.price
FROM stock s
DISTRUBUTE BY s.symbol
SORT BY s.symbol desc, s.price asc;

```
> 注意,distribute by一定要写在sort by前边.
> 

* cluster by = distribute by + sort by. 当分发的值和排序的值相同时可以简写成为cluster by

### 类型转化

* 显示转换:cast(col as type)
* [隐式转换](https://www.iteblog.com/archives/892.html)

### 数据抽样
* tablesample
	* tablesample(100 rows)
	* tablesample(100M)
	* tablesample(0.1 percent)
	* tablesample(bucket 3 out of 16 on col/rand())
		* 如果要进行随机抽样 on 后边要使用rand()函数进行随机分桶
		* 进行分桶抽样的时候,如果分桶关键字跟clustered by分桶的分桶关键字相同的时候,会在原有桶上进行抽样.

```sql
SELECT *
FROM table1 TABLESAMPLE(100 ROWS) t;
```
> 注意,tablesample只能用在实体表上,并不能用在临时表中.
> 

* order by rand() limit:会对所有数据添加一个0-1的随机值,然后进行排序.

```sql
SELECT *
FROM table1
ORDER BY rand()
LIMIT 10;

```

### 窗口函数

* [窗口函数](http://lxw1234.com/search/窗口函数)
* [窗口函数和cube](https://www.jianshu.com/p/9fda829b1ef1?from=timeline)
* hive是不支持count(distinct) over()的窗口函数的.但是我们可以使用size(collect_set() over())来代替完成

``` sql
select date, uid
	    count(distinct uid) over(partition by date),--不要这样写,不支持
	    size(collect_set(uid) over(partition by date)) --支持.
from table
```



## 视图

* 创建视图: CREATE VIEW 

```sql
CTEATE VIEW IF NOT EXISTS v1(time, part)
COMMENT 'TIME AND PARTS FOR SHIPMENTS'
TBLPROPERTIES('creator'='me')
AS SELECT ...
FROM ...
WHERE ...

```

> 注意,这里定义的col后边没有类型,后边定义col可选

* 复制视图: CREATE VIEW v1 LIKE v2
* 删除视图: DROP VIEW IF EXISTS v1
* 视图实际上并不会具体化任何实际数据,只是对使用的查询语句固化过程.
* 视图是只读的,只允许改变TBLPROPERTIES中的属性.

## 模式设计

### 按照时间进行分区
* Hive中分区的功能是非常有用的,这是因为Hive通常要对输入进行全盘扫描,来满足查询条件.通过创建很多的分区确实可以优化一些查询.

```sql
CREATE TABLE weblogs(
	url STRING,
	time long
)
PARTITIONED BY(dt INT, city STRING);
```

* 但是并不是分区越多就越好.HDFS用于设计存储数百万的大文件,而非数十亿的小文件.使用过多分区可能导致一个问题就是会创建大量的非必须的Hadoop文件和文件夹.如果每天都创建大量的文件,最终会超出NAMENODE对系统元数据的处理能力.
* MR会将一个任务(job)转化成为多个任务(task).默认情况下,每个task都是一个新的JVM实例,都需要开启和销毁的开销.对于小文件,每个文件都会对应一个task,这样会导致开启销毁JVM实例的时间远远超过数据处理时间.
* 按照时间范围进行分区是一个好的策略.分区数量是可以预估的,增长也是均匀的.每个五分区的数据量大小也是很均匀的.

### 同一份数据多种处理

```sql
FROM t1
INSERT OVERWRITE TABLE t2 SELECT ... WHERE ...
INSERT INTO TABLE t3 SELECT ... WHERE ...
```
> 仅仅读取一次t1表,就能完成t2,t3表的插入.
> 

### 分桶表数据存储
* 分区提供一个隔离数据和优化查询的便利的方式.但是,并非所有数据集都可以形成合理的分区,不能确定合适的划分大小.分桶时将数据集分解成更易于管理的若干部分的另一个技术.

```sql
--分区表
CREATE TABLE weblog(
	url STRING,
	source_ip STRING
) 
PARTITIONED BY(dt STRING, user_id INT)

--分桶表
CREATE TABLE weblog(
	url STRING,
	source_ip STRING,
	user_id INT
)
PARTITIONED BY(dt STRING)
CLUSTERED BY(user_id) INTO 96 BUCKETS;
```
> 上例,我们需要对日期和user_id对web的日志进行分开管理,如果采用日期和user_id进行分区就会产生大量的分区,所以采用分区并不合适.如果对user_id进行分桶,我们可以定义固定的个分桶文件.多个用户会被分到同一个桶中,方便管理.
> 

* 分桶的优点:桶的数量是固定的,没有数据波动,并不会产生大量的分区文件.方便抽样和执行高效的map-side join.

* 在为分桶表导入数据的时候,需要提前为任务配置一个正确的分桶表的reducer个数(等于分桶的个数)才能保证数据的正确填充.
	* 使用hive.enforce.bucketing=true属性强制reducer等于桶的个数.
	* 自己配置reducer个数等于桶的个数:set mapred.reduce.tasks=96

```sql
SET hive.enforce.bucketing=true;

FROM t
INSERT OVERWRITE/INTO TABLE weblog PARTITION(dt='20190101')
SELECT url, ip, user_id FROM s WHERE dt='20190101'

```

## 调优
### 使用EXPLAIN
* hive将一条查询语句分为多个token(符号)金额literal(字面值).
* 一个Hive任务会包含一个或多个stage(阶段),不同的阶段间会存在着依赖关系.
* 一个stage可以是一个MR任务,也可以是一个抽样阶段,或者一个合并阶段,还可以是一个limit阶段,以及hive需要的其他某个任务的一个阶段.默认情况下,hive会一次只执行一个stage(阶段),但是在开启并行执行后,可以同时进行多个阶段.

* [explain示例](https://www.cnblogs.com/skyl/p/4737411.html)

### 参数调优
* 限制调整:SET hive.limit.optimize.enable=true;

> 在使用limit的时候需要执行整个查询语句,然后返回部分结果,这种情况下执行整个查询通常是浪费的,在配置限制调整后,当使用limit的死后,对原数据进行抽样.

* 本地模式: SET hive.exec.mode.local.auto=true;在适当的时候自动启动模式

> 有时hive的输入数据非常小时,可以通过在本地模式单台计算机上处理所有任务,执行时间可以明显缩短.

* 并行执行:SET hive.exec.parallel=true;

> hive会将一个查询转化成一个或者多个阶段.在默认情况下,hive一次只会执行一个阶段,某些不相互依赖的阶段可以平行执行的,可以缩短整个job的时间.

* 严格模式:SET hive.mapred.mode=strict;
	* 分区表WHERE条件后必须要有分区字段
	* ORDER BY语句后必须要有LIMIT语句
	* 限制笛卡尔积


* 调整mapper和reducer个数
	* hive.exec.reducer.bytes.per.reducer:单个reducer处理数据量
	* mapred.reducer.tasks:执行该job使用的reducer个数
	* hive.exec.reducers.max:执行一个任务使用的最大的reducer个数
	

> hive通过将查询换分成一个或者多个MR任务达到并行的目的.每个任务都具有多个mapper和reducer任务,其中至少有一些是可以并行执行的.确定最佳的mapper个数和reducer个数取决于多个变量,例如:输入的数据量的大小和对数据执行的操作类型等.但是不是任务数量越多越好,保持平衡是有必要的,如果有太多的mapper和reducer任务,就会导致启动阶段,调度和运行job过程中产生过多的开销,如果设置的数量太少那么可能没有充分利用好集群内在的并行性.
> 
> hive是按照输入的数据量大小来确定reducer个数的.属性hive.exec.reducer.bytes.per.reducer的默认值是1GB.通常默认情况下是比较合适的.有些情况下查询的map阶段会产生比实际输入数据量要多得多的数据.如果map阶段产生的数据量非常多,那么根据输入的数据量大小来确定的reducer个数就显得有些少了.同样在map阶段过滤掉输入数据集中很大一部门数据而这时需要少量的reducer就满足计算了.
> 
> 一个快速的进行验证的方式就是将reducer个数设置成固定的值,而无需Hive来计算得到这个值hive的默认reducer个数是3.可以通过设置属性mapred.reducer.tasks的值为不同的值来确定是使用较多还是较少的reducer来缩短执行时间.需要记住,受外部因素影响,需要考虑启动和调度map和reduce任务的时间,特此额是job比较小的时候.
> 
> 在共享集群上处理大任务时,为了控制资源利用情况,属性hive.exec.reducers.max也十分重要.一个hadoop集群可以提供的map和reduce资源个数是固定的.某个大的job可能会消耗完所有的插槽,从而导致其他job无法执行.通过这个属性可以阻止某个查询消耗太多的reducer资源.这个值的大小为 总reducer槽位个数*1.5/执行中查询的平均个数
> 

* JVM重用:mapred.job.reuse.jvm.num.tasks=n;

> JVM重用是hadoop调优参数的内容,其对hive的性能具有非常大的影响,特别是对于很难避免小文件的场景或task特别多的场景,这些场景大多数的任务的执行时间都比较短.
> 
> hadoop的默认配置通常是使用派生JVM来执行map和reduce任务的.这时JVM的启动过程可能会造成相当大的开销,尤其是执行的job包含有成百上千的task任务的情况.JVM重用可以使得JVM实例在同一个job中重新使用N次.
> 
> 缺点是,开启JVM重用将会一直占用使用到的task插槽,以便进行重用,直到任务完成后才释放.如果有不平衡的job中有几个reduce task执行时间比其他reduce task消耗时间多得多的话,保留的JVM就会一直闲着无法被其他job使用,直到所有task都结束了才会释放.
> 

* 动态分区调整
	* hive.exec.dynamic.partition.mode=strict;严格模式下,动态分区前至少有一个静态分区
	* hive.exec.max.dynamic.partitions=n;最大动态分区使用的分区数

* 推测执行:mapred.map.tasks.speculative.execution=true;

> 推测执行是hadoop中的一个功能,其可以触发执行一些重复的任务(task).但是会因为对重复的数据进行计算而导致消耗更多的计算资源,不过这个功能的目标是通过加快获取单个task结果以及进行侦测将执行慢的tasktrack加入到黑名单中提高整体的任务执行效率.
> 
> 启动推测执行会对输入量很大的需要执行长时间的任务带来很大负担,推荐关掉.
> 

* 开启中间压缩:hive.exec.compress.intermediate=true

> 对中间数据进行压缩可以减少job中map和reduce之间的数据传输量.对于中间数据压缩,选择一个低CPU开下的编/解码器要比选择一个压缩率较高的编/解码器重要得多.
> 

* 最终输出结果压缩:hive.exec.compress.output=true,默认为false
	* 如果设置为true后,需要为其制定一个编/解码器:mapred.output.compression.codec=org.apache.hadoop.io.compress.GzipCodec

* [hive优化十个原则](https://www.cnblogs.com/sandbank/p/6408762.html)
* [关于数据倾斜问题的解决](https://segmentfault.com/a/1190000009166436)
* [关于map/reduce的参数配置1](https://www.cnblogs.com/wcwen1990/p/7601252.html)
* [关于map/reduce的参数配置2](https://blog.csdn.net/zhong_han_jun/article/details/50814246)


### 数据倾斜

#### join数据倾斜
1. 异常值null跟任何值关联都失败,hive会起一个reduce专门处理关联失败的数据,可能导致改reudce处理的数据大.
	* 解决方式
		* 如果业务需要null值,将null值随机赋值打散nvl(col,rand())或者分开处理
		* 不需要null直接过滤掉
2. 关联键类型不统一导致关联失败.一个关联键为int类型,另一个关联键为string.可能导致类型隐式转换不一样关联不上的情况.
	* 解决方式:cast将类型统一,最好少依赖类型隐式转换.

#### group by数据倾斜
* 在group by的时候数据分布不均匀,导致某个reduce处理的数据量远大于别的reduce.
* 解决方式
	0. 尽量减少关联表的数据量.
	1. 开启均衡负载参数:set hive.groupby.skewindata=true.当选项设定为 true，生成的查询计划会有两个 MR Job。第一个 MR Job 中，Map 的输出结果集合会随机分布到 Reduce 中，每个 Reduce 做部分聚合操作，并输出结果，这样处理的结果是相同的 Group By Key 有可能被分发到不同的 Reduce 中，从而达到负载均衡的目的；第二个 MR Job 再根据预处理的数据结果按照 Group By Key 分布到 Reduce 中（这个过程可以保证相同的 Group By Key 被分布到同一个 Reduce 中），最后完成最终的聚合操作。
	2. 开启combiner:set hive.map.aggr=true
	3. 大表join小表:转化为mapjoin.
	4. 大表join大表:找到大key需要将其分配的数据散.
		* 如果大key是静态的且可少量,直接单独处理即可然后与其他数据union all
		* 如果大key是动态的且量比较大,需要维护一个大key的临时表.将大key和普通key分开处理.最好能转化成mapjoin.

[大表join大表数据倾斜优化](https://www.cnblogs.com/shaosks/p/9491905.html)

[hive数据倾斜方法总结](https://www.cnblogs.com/kongcong/p/7777092.html)

#### HQL数据倾斜
* HQL尽量将数据控制到最小.在连接前加上分区剪裁和分桶剪裁.
* count(distinct)在hive中屏蔽用户设置的reducer个数且只使用一个reducer,所以在处理大量数据的时候会异常缓慢.尽量改写成为sum() from(group by())

[count(distinct)原理](https://blog.csdn.net/oracle8090/article/details/80760233)



## 用户自定义函数

* 查看用户自定义函数: show functions;
* 查看某个函数: describe function concat;
* 使用UDF函数
	1. 将自定义函数打包成jar文件,上传到集群
	2. 在hive中添加add jar /path/xxx.jar,注意jar路径不需要引号引起
	3. 创建临时函数:create temporary function myfunc as '包名.主函数名'
		* 注意,这里只是临时函数,仅仅作用于当前会话.
	4. 如果频繁需要需要将jar文件添加到$HOME/.hiverc中.
* 删除函数 drop temporary function;
* 用户自定义函数分为UDF/UDAF/UDTF三种.

## Streaming-TRANSFORM
* hive是通过利用或扩展hadoop组件来运行的,通过抽象InputFormat,OutputFormat来定义数据的输入输出.Streaming提供了另一种处理数据的方式,在Streaming job中,会为外部进程开启一个I/O管道,然后数据会被传给这个进程,其会从标准输入中读取数据,然后通过标准输出来写数据结果,最后返回到Streaming API job中.
* 在使用关键字TRANSFORM来使用Streaming功能.

```sql
SELECT TANSFORM(col1,col2)
USING '/bin/cat' AS new_col1,new_col2
FROM table


--定义输出格式
SELECT TANSFORM(col1,col2)
USING '/bin/cat' AS (
	new_col1 STRING,
	new_col2 INT
) ROW FORMAT DELIMITED
LINES TERMINATED BY '\n' 
...
FROM table

```

* 在写UDTF的时候,一般transform 与 distribute + sort联合使用更有效果,因为distribute+sort能将固定数据分到同一个reduce中,然后做处理.

## 示例,计算用户在线时长

### 工作内容
* 计算某个用户某台设备的在线时长

### 内容解读
* 数据中并没有统计累计在线时长的,所以需要自己计算.
* 计算在线时长需要定义怎么才算在线?
	* 如果在线肯定会有接口请求数据
	* 在线的隐式含义是连续的接口请求,如何才算连续?
		* 定义上下两次接口之间的间隔时间大小才认定是否有效连续.
			* 间隔时间长,认为断开重连,不是本次在线间断->结束
			* 间隔时间短,可认为在线,可认为是连续请求.我们这定义为300秒,也就是5分钟.
	* 计算本次在线最开始接口请求的时间和最后接口请求的时间之差就是在线时长.
* 缺点
	* 有的中途切换并不能认证为同一次在线,导致中间间隔时间多加.
	* 有的页面停留时间过长,导致认为两次在线,导致中间间隔时间没加.
	* 总之跟定义连续接口访问的时间限制大小有很大关系.
		* 因为我们现在app的ugc内容以图片为主,所以停留时间不长,5分钟已经扩的很开.
		* 需要根据业务来指定

### 思路
1. hive本身没有类似UDF可以使用,所以需要自己定义函数来完成计算.
2. 定义什么函数?计算在线时长是聚合操作,以用户分组.
	* UDAF -- java
	* transform -- python
3. 根据同一用户,同一台设备分组.
4. 计算在定义的有效时间内,前后次请求的差值.可以认为这个差值就是本次点击的在线时长
5. 遇到超出有效时间的连续请求可认定为断开连接,本次在线结束.
6. 结束时,汇总所有接口之间差值就是在线时长.

### 代码
* 因为我现在公司的技术栈没有java,则采用transform的方式来计算.
* 下面是transform的具体用法sql部分
* transform语法,hql部分

```sql
	add file transform1.py;
	add file transform2.py;
	-- 插入新表
	from(
	 	select ... from table where ...
	) tmp
	insert overwrite table1 partition(dt = '')
	select transform(tmp.*) -- 使用transform,tmp.*是传入的数据
	using 'python transform1.py' -- 执行transform的文件
	as( -- transform输出结果
		col1 int,
		col2 string,
		...
	)
	row format -- 定义transform输出格式
	delimited fields terminated by '\001'
	collection items terminated by '\002'
	map keys terminated by '\003'
	
	insert overwrite table1 partition(dt = '')
	select transform(tmp.*) -- 使用transform,tmp.*是传入的数据
	using 'python transform2.py' -- 执行transform的文件
	...


	-- 数据转换	
	select transform(t.*) -- 使用transform,tmp.*是传入的数据
	using 'python transform.py' -- 执行transform的文件
	as( -- transform输出结果
		col1 int,
		col2 string,
		...
	)
	from(
		select ... from table where ...
	) t
		
```

* user_time.hql

``` sql

-- user_time.hql
ADD FILE transform.py; --添加transform脚本

FROM
    (
        SELECT
            device_id,
            uid,
            path,
            request_timestamp
        FROM ods.api_access_log acl
        WHERE partition_date={{DATE}} --{{DATE}}是参数
        DISTRIBUTE BY device_id --用设备ID分桶
        SORT BY device_id, request_timestamp --排序
) tmp --基表,用于收集要处理的数据
INSERT OVERWRITE TABLE dw.dw_user_use_time --要插入的表partition(partition_date={{DATE}}) --要插入表的分区
SELECT TRANSFORM(tmp.*) -- transform语法,传入tmp的所有数据
USING 'python transform.py' -- transform.py 和 hql 在一个文件夹下
AS( -- transform输出
    device_id string,
    user_id bigint,
    use_time int
)
ROW FORMAT -- transform输出格式
DELIMITED FIELDS TERMINATED BY '\001'
COLLECTION ITEMS TERMINATED BY '\002'
MAP KEYS TERMINATED BY '\003'

```
  
* transform.py内容

``` python

import os
import sys

	for line in sys.stdin: 
		# sys.stdin 是select transform(tmp.*)传输进来的
		
		line = line.rstrip('\n') # 去除行末尾的换行符
		col1,col2,col3 = line.split()
		
	... # 处理过程
	
	# transform输出  必须要用print输出
	# 如果是array,map结构的拼成字符串即可,在hql输出的时候用row format规定格式分割
	print '\t'.join(map(str,[col1, col2, col3]))
	

```

* 问题解决的transform.py

```python 

#!/usr/bin/python
#encoding:utf-8
# transform.py

import sys
import time

define USER_LEAVE_INTERVAL = 300 # 定义有效时间

def process(data):
    total_cost_time = 0 # 总的使用时间
    prev_request_time = None # 上一个请求时间
    user_id = None # 记录当前处理的用户ID
    device_id = None # 记录当前处理的设备ID
    
    for parts in data:
        try:
            device_id, uid, path, request_time = parts
            request_time = long(request_time)
            
            if prev_request_time is None: # 初始化
                prev_request_time = request_time 
                user_id = uid 
                device_id = device_id
                continue
           
            cost = request_time - prev_request_time # 计算前后两个请求时间差值
            prev_request_time = request_time # 更新上一个请求时间为当前时间
            if cost > 0 and cost < USER_LEAVE_INTERVAL: # 差值时间有效
                total_cost_time += cost # 加入总的时间
        except:
            continue
    if total_cost_time > 0: # 输出
        print "\001".join(map(str, [device_id, uid, total_cost_time]))
        
def main():

    old_device_id = None # 用于标识上一条数据的设备ID
    buffers = [] # 用户保存要传入处理过程的列表
    for line in sys.stdin: # 获取transform的输出
        line = line.rstrip('\n') 
        if not line: # 空行略过
            continue
        parts = line.split() # 切分数据
        if len(parts) != 4: # 如果传入的数据为脏数据,略过
            continue
        device_id = parts[0] # 获取设备ID

        if device_id == "": # 设备ID为空,略过
            continue

        if device_id != old_device_id: # 如果当前设备ID 不等于 上一条数据的设备ID
            if old_device_id: # 且上一条数据设备ID不为None,说明上个设备数据取完(tmp.*传入的数据是按照设备ID排序的)
                process(buffers) # 对这个设备ID的数据进行处理
            old_device_id = device_id # 更新设备ID为当前设备ID
            buffers = [] # 置空

        buffers.append(parts) # 设备ID相同时直接加入buffer
    else:
        if buffers:
            process(buffers)

if __name__ == "__main__":
    main()

```

### 使用窗口函数解决用户在线时长问题
* 窗口函数lag和lead是取当前函数的前/后n条数据
* [使用详解](http://lxw1234.com/archives/2015/04/190.htm)

* 思路:我们可以通过lag窗口函数将当前访问记录和上个访问记录进行连接.然后计算前后两条访问时间的间隔.然后对间隔进行累加就是最终的在线时长.

```sql

select device_id,uid,path,sum(last_request_timestamp - request_timestamp) as use_time
from 
(
	SELECT
	   device_id,
	   uid,
	   path,
	   request_timestamp,
	   lag(request_timestamp,1, 0) over(partition by device_id,uid,path order by request_timestamp) as last_request_timestamp
	FROM ods.api_access_log acl
	WHERE partition_date={{DATE}} --{{DATE}}是参数
) t 
where last_request_timestamp - request_timestamp <= 300 

```

[关于页面统计学习博客](https://blog.csdn.net/zhaolq1024/article/details/81081710)


### 连续登陆天数

```sql
	 --uid,flag,连续端起始时间,连续端结束时间,连续天数,连续段内所有天
    select uid,flag, min(date) start_date, max(date) end_date, count(1) days, collect_set(date) as content
    from
    (
        select uid, date, (date - row_number() over(partition by uid order by date)) flag
        from 
        (
            select distinct uid,from_unixtime(add_time,'yyyyMMdd') as date
            from nice_live_kkgoo.user_show_sid
            where from_unixtime(add_time,'yyyyMMdd') between 20190601 and 20190630
        ) t 
    ) t 
    group by 1,2
```

### 同时在线人数

* 我们有这样的一张使用时间的表

```sql
create table user_use_time(
   id int comment "使用记录ID",
	uid int comment "用户id",
	start_time string comment "用户使用开始时间",
	end_time string comment "用户使用结束时间"
)
...


user_id	login_time	logout_time	
11111	12:00	12:02
11112   12:01	12:03
11113   12:00	12:08
11114   12:00	12:03
11115   12:03	12:04
11116   11:00   13:00

```

* 开始时间和结束时间是一个明确的时间,并且只记录了这个使用记录的开始时间和结束时间,中间经历的时间没有记录.求:当天每分钟的最大同时在线人数.


```sql
with user_use_time as 
(
    select '11111' as user_id,	'12:00' as start_time,	'12:02' as end_time
    union all
    select '11112' as user_id,	'12:01' as start_time,	'12:03' as end_time
    union all
    select '11113' as user_id,  '12:00' as start_time,	'12:08' as end_time
    union all
    select '11114' as user_id,  '12:00' as start_time,	'12:03' as end_time
    union all
    select '11115' as user_id,  '12:03' as start_time,	'12:04' as end_time
    union all
    select '11116' as user_id,  '11:00' as start_time,	'13:00' as end_time
) --模拟数据

select num, --在线人数
       time,--开始时间
       lead(time, 1) over(order by time)--结束时间
from 
(
    select distinct time, num --因为使用窗口函数计算的,需要去重
    from(
        select time,--开始时间
               sum(num) over(order by time) as num --同时在线人数
        from 
        (
        	  --每个开始时间,是人数增加.+1
            select user_id, 
            	      start_time as time, 
            	      1 as num
            from user_use_time
            
            union all
            
            --每个结束时间,是人数减少.-1
            select user_id, 
                   end_time as time, 
                   -1 as num
            from user_use_time
        ) t
    ) t 
) t 
```

* 结果展示

```
在线人数/开始时间/结束时间
1	11:00	12:00
4	12:00	12:01
5	12:01	12:02
4	12:02	12:03
3	12:03	12:04
2	12:04	12:08
1	12:08	13:00
0	13:00	NULL

```

* 注意:这个结束时间是用lead窗口函数匹配出来的,而开始时间则是每个用户的使用开始时间和结束时间出现的记录.我们可以理解为当一个用户有了开始使用或者结束使用这个动作,那么在线人数会因为他的动作改变,每个改变就会产生一条记录,每条记录都有改变的时间阶段:开始时间是改变的时间(登录/退出的时间),结束时间是下一个改变的时间.

## 宏(macro)
### 描述
* 在编写HQL的过程，很多逻辑需要反复的使用。这是我们可以通过宏对这段重复使用的逻辑进行提炼。* 宏的作用相当于编程过程中的函数，将一些常用的语句进行封装便于后续的重复多次使用。* 显而易见，开发过程比UDF更加简单快捷。* 同时UDF是Java编写的，代码中的堆变量内存回收不受开发者控制，同时UDF还是嵌套在HQL中执行的，对于规模较大的表，比较容易造成OOM。Hive-0.12.0及以后版本支持宏的使用。

### 语法
#### 创建宏
```sql
create temporary macro [if not exists] macro_name( [ col_name col_type, ... ] ) expression;宏的参数可以为若干个，宏的返回值为expression的返回值。
```

```sql
--示例
create temporary macro fixed_number() 42; --42create temporary macro if not exists simple_add(int x, int y) x + y;create temporay macro nest_add(int x) x + fixed_number(); --macromacro--macrocase-whencreate temporary macro case_when()casewhen ... then ...when ... then ...else ...end;
```

#### 删除宏

```sql
drop temporary macro [if exists] macro_name;
```

#### 一些常用的例子
1. 空值转空串
	* 业务需要：将空串''视为NULL，需要补足
	* 使用场景：在使用nvl或者coalesce时，如果前一个参数为空串‘’，则无法取到后边的参数。

```sql
create temporary macro empty2null(string x) if(trim(x) = '', null, x);coalesce(empty2null(x1),empty2null(x2),...,'UNKNOWN');
```

2. NULL转空串
	* 使用场景：当使用concat拼接两个字段时，只有其中一个为NULL，则输出也为NULL。

```sql
create temporary macro null2empty(x string) if(x is null, '', x);concat(null2empty(str1), null2empty(str2));
```

3. 判断NULL和空串

```sql
create temporary macro is_ne(x string) nvl(trim(x), '') = '';--NULLtruefalse trim(x)null''
```

4. 数据倾斜解决方法
	* 使用场景：当字段存在大量NULL值或者空串时，可能会引发数据倾斜问题，应该把key转化为随机字符串，使得该字段均匀分布到各个reduce中。

```sql
create temporary macro ne2rand(x string)	case		when is_ne(x) then concat('hive',rand())		else x	end;
```

5. 有关日期的计算

```sql
--计算月的第一天
--create temporary macro first_day(dt string) trunc(dt,'mm'); dtyyyy-dd-mm--create temporary macro first_day_last_month(dt string) trunc(add_months(dt, -1), 'mm');


--计算月的最后一天
--create temporary macro last_day(dt string) last_day(dt);--create temporary macro last_day_last_month(dt string) last_day(add_months(dt, -1));
```