# Clickhouse学习总结

## 一、数据类型

```
1、整型：固定长度类型，包括有符号整型和无符号整型
有符号整型范围（-2^(n-1)~2^(n-1)-1）:
Int8:[-128-127](byte)
Int16:[-32768-32767](short)
Int32:[-2147483648-147483647](int)
Int64:[-9223372036854775808-9223372036854775807](long)
无符号整型范围（0~2^(n-1)）
UInt8:[0-255]
UInt16:[0-65535]
UInt32:[0-4294967295]
UInt64:[0-18446744073709551615]

2、浮点型
Float32(float)
Float64(double)

3、布尔型
没有单独的类型来存储布尔型值，可以使用UInt8来表示，限制值为0或1

4、decimal型（有符号浮点数，可在加、减和乘法运算过程中保持精度。对于除法、最低有效数字会被丢弃）
Decimal32(s)，相当于Decimal(9-s,s),有效位数为1~9，s标识小数位长度
Decimal64(s)，相当于Decimal(18-s,s),有效位数为1~18
Decimal128(s)，相当于Decimal(38-s,s),有效位数为1~38
使用场景：一般金额字段、汇率、利率等字段为了保证小数点精度，都使用Decimal进行存储

5、字符串
String：字符串可以任意长度，可以包含任意字符集，包含空字节
FixedString(N)：固定长度N的字符串，N必须是严格的正自然数。当服务端读取长度小于N的字符串的时候，通过在字符串结尾添加空字节来达到N字节长度，当服务端读取长度大于N的字符串的时候，将返回错误消息。
使用场景：名称、文字描述、字符型编码。固定长度的可以保存一些定长的内容，比如一些编码，性别等

6、枚举类型
Enum8：用'String'=Int8对描述
Enum16：用'String'=Int16对描述
eg:
创建一个带有一个枚举Enum8('hello'=1,'world'=2)类型的列
create table t_enum(
	x Enum8('hello'=1, 'world'=2)
)ENGINE=TinyLog;
这个x列只能存储类型定义中列出的值：'hello'或'world'，插入其他值会抛出异常
insert into t_enum8 values('hello')('world');
insert into t_enum8 values('a'); 报错，插入的值没有定义
如果要查看对应行的数值，则必须将Enum值转换为整数类型
select cast(x, 'Int8') from t_enum8;

7、时间类型
Date32接受年-月-日的字符串，如'2019-12-16'
Datetime接受年-月-日 时:分:秒的字符串，如'2019-12-16 10:10:10'
Datetime64接受年-月-日 时:分:秒.亚秒的字符串，如'2019-10-10 10:10:10.10'
日期类型，用两个字节存储，表示从1970-01-01到当前的日期值

8、数组
Array(T)：由T类型元素组成的数组
T可以是任意类型，包含数组类型，但不推荐使用多维数组，在MergeTree表中存储多维数组
创建数组方式1，使用array函数
select array(1,3) as x, toTypeName(x);
创建数组方式2，使用方括号
select [1,2] as x, toTypeName(x);

9、Map
Map(key type,value type)可以存储key:value键值对类型的数据
eg：
CREATE TABLE t_map(
	m Map(String, Int32)
)ENGINE=Memory;
INSERT INTO t_map values({'key1': 1, 'key2': 2})({'key3': 3});
查看key为'key1'的所有值，如key没有匹配则返回0
SELECT m['key1'] FROM t_map; 
查看所有的key
SELECT m.keys FROM t_map;
查看所有的value
SELECT m.values FROM t_map;

10、元组
Tuple(t1,t2,...)
不能在表中进行存储，可以用于临时列分组
eg：
SELECT tuple(1,3,'a') AS x, toTypeName(x);
将tuple类型转换为map类型
SELECT CAST((['a','b','c'], [1,2,3]),'Map(String, Int8)') AS x, toTypeName(x);
```

## 二、表引擎

### 2.1、MergeTree

```mysql
MergeTree系列的引擎被设计用于插入极大量的数据到一张表当中，数据可以以数据片段的形式一个接着一个的快速写入，数据片段在后台按照一定的规则进行合并。相比在插入时不断修改已存储的数据，这种策略会高效很多。
任何一个批次的数据写入都会产生一个临时分区，不会纳入到任何一个已有的分区，写入后的某个时刻（大概10到15分钟），clickhouse会自动执行合并操作，把临时分区的数据合并到已有分区中。
主要特点：
	存储的数据按照主键排序
	如果指定了分区键可以使用分区，在相同数据集和相同结果集的情况下，clickhouse中某些带分区的操作会比普通操作更快。查询中指定了分区键时，clickhouse会自动截取分区数据，加快查询效率。
建表方式：
CREATE TABLE [IF NOT EXISTS] [db.]table_name [ON CLUSTER cluster]
(
    name1 [type1] [DEFAULT|MATERIALIZED|ALIAS expr1] [TTL expr1],
    name2 [type2] [DEFAULT|MATERIALIZED|ALIAS expr2] [TTL expr2],
    ...
    INDEX index_name1 expr1 TYPE type1(...) GRANULARITY value1,
    INDEX index_name2 expr2 TYPE type2(...) GRANULARITY value2
) ENGINE = MergeTree()
ORDER BY expr
[PARTITION BY expr]
[PRIMARY KEY expr]
[SAMPLE BY expr]
[TTL expr [DELETE|TO DISK 'xxx'|TO VOLUME 'xxx'], ...]
[SETTINGS name=value, ...];

ENGINE：引擎名和参数

ORDER BY（必选）：排序键，可以是一组列的元组或任意表达式，如果没有使用PRIMARY KEY显示指定主键，clickhouse会使用排序键作为主键，如果不需要排序，可以使用ORDER BY tuple()。
设定了分区内的数据按照哪些字段顺序进行有序保存
主键必须是order by字段的前缀字段，如order by字段是(id,sku_id)，那么主键必须是id或(id,sku_id)

PARTITION BY：分区键，大多数情况下，不需要使用分区键，即使需要使用，也不需要使用比月更细粒度的分区键。分区不会加快查询。不要使用客户端指定分区标识符或分区字段名称来对数据进行分区（而是将分区字段标识作为order by表达式的第一列来指定分区）。要按月分区，可以使用表达式toYYYYMM(date_column)，date_column是一个Date类型的列，分区名格式会是"YYYYMM"

PRIMARY KEY:如果要选择与排序键不同的主键，可以在这里指定。默认情况下主键和排序键相同，因此大部分情况下不需要专门指定一个PRIMARY KEY子句。
他只提供数据的一级索引，但是却不是唯一约束。
主键的设定主要依据是查询语句中的where条件。根据条件通过对主键进行某种二分查找，能够定位到对应的索引，避免全表扫描。

TTL：指定行存储的持续时间并定义数据片段在硬盘和卷上的移动逻辑的规则列表。表达式中必须存在至少一个Date或Datetime类型的列。如：TTL date + INTERVAL 1 DAY。规则类型(DELETE|TO DISK 'XXX'|TO VOLUME 'XXX')指定了当满足条件（到达指定时间）时所要执行的动作：移除过期的行，还是将数据片段（如果数据片段中的所有行都满足表达式的话）移动到指定的磁盘(TO DISK 'XXX')或卷(TO VOLUME 'XXX')。默认的规则是移除（DELETE）。可以在列表中指定多个规则，但最多只能有一个DELETE的规则。

列TTL：当列中的值过期时，clickhouse会将他们替换成该列数据类型的默认值，如果数据片段中列的所有值均已过期，则clickhouse会从文件系统中的数据片段中删除此列。TTL子句不用用于主键字段
eg：
CREATE TABLE example_table
(
    d DateTime,
    a Int TTL d + INTERVAL 1 MONTH,
    b Int TTL d + INTERVAL 1 MONTH,
    c String
)
ENGINE = MergeTree
PARTITION BY toYYYYMM(d)
ORDER BY d;

表TTL：可以设置一个用于移除过期行的表达式，以及多个用于在磁盘或卷上自动移除数据片段的表达式。当表中的行过期时，clickhouse会删除所有对应的行。对于数据片段的转移特性，必须所有的行都满足转移条件。
格式：
TTL expr
    [DELETE|TO DISK 'xxx'|TO VOLUME 'xxx'][, DELETE|TO DISK 'aaa'|TO VOLUME 'bbb'] ...
    [WHERE conditions]
    [GROUP BY key_expr [SET v1 = aggr_func(v1) [, v2 = aggr_func(v2) ...]] ]
GROUP BY：聚合过期的行
WHERE：指定哪些过期的行会被删除或聚合。GROUP BY 表达式必须是表主键的前缀。如果某列不是GROUP BY 表达式的一部分，也没有在SET从句中显示引用，结果行中相应的列的值是随机的
eg：
CREATE TABLE example_table
(
    d DateTime,
    a Int
)
ENGINE = MergeTree
PARTITION BY toYYYYMM(d)
ORDER BY d
TTL d + INTERVAL 1 MONTH DELETE,
    d + INTERVAL 1 WEEK TO VOLUME 'aaa',
    d + INTERVAL 2 WEEK TO DISK 'bbb';
eg：
创建一张表，设置一个月后数据过期，这些过期的行中日期为星期一的删除：
CREATE TABLE table_with_where
(
    d DateTime,
    a Int
)
ENGINE = MergeTree
PARTITION BY toYYYYMM(d)
ORDER BY d
TTL d + INTERVAL 1 MONTH DELETE WHERE toDayOfWeek(d) = 1;
eg：
创建一张表，设置过期的列会被聚合。列x包含每组行中的最大值，y为最小值，d为可能任意值。
CREATE TABLE table_for_aggregation
(
    d DateTime,
    k1 Int,
    k2 Int,
    x Int,
    y Int
)
ENGINE = MergeTree
ORDER BY (k1, k2)
TTL d + INTERVAL 1 MONTH GROUP BY k1, k2 SET x = max(x), y = min(y);
```

### 2.2、ReplacingMergeTree

```mysql
该引擎会删除排序键值相同的重复项。数据去重只会在合并期间进行，合并会在后台一个不确定的时间进行，因此在使用数据的时候可能数据仍未被处理。不能保证没有重复数据的出现。
去重只会在分区内部进行去重，不能跨分区去重

CREATE TABLE [IF NOT EXISTS] [db.]table_name [ON CLUSTER cluster]
(
    name1 [type1] [DEFAULT|MATERIALIZED|ALIAS expr1],
    name2 [type2] [DEFAULT|MATERIALIZED|ALIAS expr2],
    ...
) ENGINE = ReplacingMergeTree([ver])
[PARTITION BY expr]
[ORDER BY expr]
[SAMPLE BY expr]
[SETTINGS name=value, ...]

参数：
ver：版本列，类型为UInt*,Date,DateTime，可选参数。在数据合并的时候，会从所有具有相同排序键的行中选择一行留下；
如果ver列未指定，则保留最后一条数据
如果ver列指定，则保留ver最大的版本。
```

### 2.3、SummingMergeTree

```mysql
在合并SummingMergeTree表的数据片段时，会把所有具有相同主键的行合并为一行，该行包含了被合并的行中具有数值数据类型的列的汇总值。

CREATE TABLE [IF NOT EXISTS] [db.]table_name [ON CLUSTER cluster]
(
    name1 [type1] [DEFAULT|MATERIALIZED|ALIAS expr1],
    name2 [type2] [DEFAULT|MATERIALIZED|ALIAS expr2],
    ...
) ENGINE = SummingMergeTree([columns])
[PARTITION BY expr]
[ORDER BY expr]
[SAMPLE BY expr]
[SETTINGS name=value, ...]

参数：
columns：包含了将要被汇总的列的列名的元组，可选参数。所选的列必须是数值类型，并且不可位于主键中。

当数据被插入表中时，他们将被原样保存。clickhouse定期合并插入的数据片段，并在这个时候对所有具有相同主键的行中的列进行汇总，将这些行替换为包含汇总数据的一行记录。
```

### 2.4、TinyLog

```
用于将数据存储在磁盘上，每列都存储在单独的压缩文件中。写入时，数据将附加到文件末尾。
并发数据访问不受限制：
	如果同时从表中读取并在不同的查询中写入，则读取操作将抛出异常。
	如果同时写入多个查询中的表，则数据将被破坏。
这种表引擎的典型用法是write-once：首先只写入一次数据，然后根据需要多次读取。
```

## 三、函数

### 3.1、类型转换函数

```

```

